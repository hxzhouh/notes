# Golang 通过 Consul 实现分布式锁
# Consul 是什么
Consul 是一个支持多数据中心分布式高可用的服务发现和配置共享的服务软件,由 HashiCorp 公司用 Go 语言开发, 基于 Mozilla Public License 2.0 的协议进行开源. Consul 支持健康检查,并允许 HTTP 和 DNS 协议调用 API 存储键值对.
命令行超级好用的虚拟机管理软件 vgrant 也是 HashiCorp 公司开发的产品.
一致性协议采用 Raft 算法,用来保证服务的高可用. 使用 GOSSIP 协议管理成员和广播消息, 并且支持 ACL 访问控制.
# Consul 的使用场景
* docker 实例的注册与配置共享
* coreos 实例的注册与配置共享
* vitess 集群
* SaaS 应用的配置共享
* 与 confd 服务集成，动态生成 nginx 和 haproxy 配置文件
# Consul 的优势
* 使用 Raft 算法来保证一致性, 比复杂的 Paxos 算法更直接. 相比较而言, zookeeper 采用的是 Paxos, 而 etcd 使用的则是 Raft.
* 支持多数据中心，内外网的服务采用不同的端口进行监听。 多数据中心集群可以避免单数据中心的单点故障,而其部署则需要考虑网络延迟, 分片等情况等. zookeeper 和 etcd 均不提供多数据中心功能的支持.
* 支持健康检查. etcd 不提供此功能.
* 支持 http 和 dns 协议接口. zookeeper 的集成较为复杂, etcd 只支持 http 协议.
* 官方提供web管理界面, etcd 无此功能.
综合比较, Consul 作为服务注册和配置管理的新星, 比较值得关注和研究.
# Consul 的角色
client: 客户端, 无状态, 将 HTTP 和 DNS 接口请求转发给局域网内的服务端集群.
server: 服务端, 保存配置信息, 高可用集群, 在局域网内与本地客户端通讯, 通过广域网与其他数据中心通讯. 每个数据中心的 server 数量推荐为 3 个或是 5 个.
# 什么是分布式锁
**分布式锁**，是控制[分布式系统](https://zh.wikipedia.org/w/index.php?title=%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F&action=edit&redlink=1 "分布式系统（页面不存在）")之间同步访问共享[资源](https://zh.wikipedia.org/wiki/%E8%B5%84%E6%BA%90 "资源")的一种方式。在分布式系统中，常常需要协调他们的动作。如果不同的系统或是同一个系统的不同主机之间共享了一个或一组资源，那么访问这些资源的时候，往往需要[互斥](https://zh.wikipedia.org/wiki/%E4%BA%92%E6%96%A5 "互斥")来防止彼此干扰来保证一致性，在这种情况下，便需要使用到分布式锁。
目前几乎很多大型网站及应用都是分布式部署的，分布式场景中的数据一致性问题一直是一个比较重要的话题。分布式的CAP理论告诉我们“任何一个分布式系统都无法同时满足一致性（Consistency）、可用性（Availability）和分区容错性（Partition tolerance），最多只能同时满足两项。”所以，很多系统在设计之初就要对这三者做出取舍。在互联网领域的绝大多数的场景中，都需要牺牲强一致性来换取系统的高可用性，系统往往只需要保证“最终一致性”，只要这个最终时间是在用户可以接受的范围内即可。
在很多场景中，我们为了保证数据的最终一致性，需要很多的技术方案来支持，比如分布式事务、分布式锁等。有的时候，我们需要保证一个方法在同一时间内只能被同一个线程执行。
在这里我们使用Consul来管理分布式锁。Consul内置了服务注册与发现框 架、分布一致性协议实现、健康检查、Key/Value存储、多数据中心方案，不再需要依赖其他工具（比如[ZooKeeper](http://tonybai.com/tag/zookeeper)等）。
### Sessions
[](https://o6rr5e4by.qnssl.com/wp-content/uploads/2017/01/consul-sessions-56bc7006.png)
session是一个远程进程和consul节点之间的链接，它由一个远程进程和可以显式无效或由于健康检查机制。根据会话配置，创建与已失效会话锁摧毁或释放。
#### Health checks
Consul支持多种检查 （如 HTTP、 TCP 等）。在session创建过程中可以定义的健康检查列表。这些检查将用于确定是否sessio需要使之失效。
#### TTL
除了健康检查，会话也具有内置支持的 TTL。当 TTL 过期session被视为无效。远程进程负责更新session之前 TTL 过期。
### Golang API
[Consul API client](https://godoc.org/github.com/hashicorp/consul/api) 提供一个方便的抽象，session和 K/V 存储。有是一个锁结构与锁定、 解锁和破坏的方法。也有用于帮助创建锁实例方法。API 客户端还负责更新会话。
#### Creating the Consul client
``` golang
client, err := api.NewClient(&api.Config{Address: "127.0.0.1:8500"})
```
#### Creating Lock instance
```golang
type LockOptions struct {
    Key string // Must be set and have write permissions
    Value []byte // Optional, value to associate with the lock
    Session string // Optional, created if not specified
    SessionOpts *SessionEntry // Optional, options to use when creating a session
    SessionName string // Optional, defaults to DefaultLockSessionName (ignored if SessionOpts is given)
    SessionTTL string // Optional, defaults to DefaultLockSessionTTL (ignored if SessionOpts is given)
    MonitorRetries int // Optional, defaults to 0 which means no retries
    MonitorRetryTime time.Duration // Optional, defaults to DefaultMonitorRetryTime
    LockWaitTime time.Duration // Optional, defaults to DefaultLockWaitTime
    LockTryOnce bool // Optional, defaults to false which means try forever
}
```
LockOptions 是所有可能的选项的容器，可以用于设置键和值、 定制会话或设置TTL。
```golang
opts := &api.LockOptions{
    Key: "webhook_receiver/1",
    Value: []byte("set by sender 1"),
    SessionTTL: "10s",
    SessionOpts: &api.SessionEntry{
    Checks: []string{"check1", "check2"},
    Behavior: "release",
},
}
lock, err := client.LockOpts(opts)
```
另一种常用的方法是 LockKey，它创建一个锁与所有选项设置为默认条目名称除外。
```golang
    lock, err := client.LockKey("webhook_receiver/1")
```
#### Acquiring lock
```golang
stopCh := make(chan struct{})
lockCh, err := lock.Lock(stopCh)
if err != nil {
    panic(err)
}
cancelCtx, cancelRequest := context.WithCancel(context.Background())
req, _ := http.NewRequest("GET", "https://example.com/webhook", nil)
req = req.WithContext(cancelCtx)
go func() {
http.DefaultClient.Do(req)
select {
case <-cancelCtx.Done():
log.Println("request cancelled")
default:
log.Println("request done")
err = lock.Unlock()
if err != nil {
log.Println("lock already unlocked")
}
}
}()
go func() {
<-lockCh
cancelRequest()
}()
```