

#  背景

同一主机同一进程不同的线程，如何同步访问一段代码块呢？

Java有

```java
synchronizedsynchronized(this) {}
```

Golang有sync工具包

```go
var mutex sync.Mutex//加锁mutexmutex.Lock()
do Something···
//解锁mutex
mutex.Unlock()
```



PHP因为PHP没有多线程的概念，对PHP而言，普遍的是多进程，PHP的多个多进程之间同步访问，可以通过文件锁来实现。

```php
$fp = fopen("logs/lock.l", "a+");
if (flock($fp, LOCK_EX)) { // 进行排它型锁定

do something....

flock($fp, LOCK_UN); // 释放锁定
} else {
echo "锁正在被其他程序占用";
}
fclose($fp);
```

不同的语言对于同步访问的问题，会有不同的解决方案，还可以通过共享内存来实现，但是这些解决方案的本质都是通过锁来保持同步的。现在企业对外提供的服务，一般都是以集群的方式来做的，那么多机之前该怎么去协同同步呢？答案很简单，用**分布式锁**来解决。接下来尝试用Redis实现一个分布式锁。

## 加锁



加锁的时候，对特定的资源加锁，例如：秒杀场景为库存余额加锁等等。lockKey就代表库存，要标识这个锁的身份，所以就要用一个唯一IDuniqLockValue来标识。如果一个任务过长，我们要设定锁的过期时间$ttl。

```lua
//表示库存 这把锁
$lockKey = "PRODUCT_LOCK_KEY";

//获取一个唯一ID，来表明这个把锁是谁加的
$uniqLockValue = getUniqueId();

//加上一把锁 setnx 是当key不存在的时候，可以添加成功。
$ret = $redis->setnx($lockKey,$uniqLockValue);
if($ret){
//加锁成功 设置过期时间
$ttl = 3; // 单位：s
$redis->expire($lockKey,$ttl);
}else{
//表明锁有其他人在使用
}
```



## 解锁

解锁的逻辑如下：获取锁，验证锁的身份是否是自己加上的，如果是自己加上的并且没有过期就解锁，否则，那就解锁失败。

```lua
$lockKey = "PRODUCT_LOCK_KEY";
$currentTime = time();
if(($uniqLockValue == $redis->get($lockKey)) && $redis->ttl($lockKey) <= $currentTime){
	$redis->del($key);
	return true;
}else{
	return false;
}
```

示例代码：

``` php
class RedisLock {
const ORDER_LOCK_KEY_PRE = 'PRODUCET_KEY_LOCK';
//锁过期时间
const LOCK_KEY_EXPIRE = 5;

//类静态实例
private static $objInstance = null;

// redis服务实例
protected $objRedis = null;

private function __construct() {
$this->objRedis = new Redis();
$this->objRedis->connect('127.0.0.1', 6379);
}

public static function getInstance() {
if (is_null(self::$objInstance)) {
self::$objInstance = new self();
}
return self::$objInstance;
}

/**
* 加锁
* @param $strKey string redis key值
* @return bool
*/
public function lock($strKey, $force = true) {
//表示库存 这把锁
$lockKey = "PRODUCT_LOCK_KEY";

//获取一个唯一ID，来表明这个把锁是谁加的
$uniqLockValue = uniqid();

//加上一把锁 setnx 是当key不存在的时候，可以添加成功。
$ret = $this->objRedis->setnx($lockKey,$uniqLockValue);
// 如果锁存在 但是 已经过期了
if($ret || ($this->objRedis->ttl($lockKey) == -2 && $this->objRedis->getSet($lockKey,$uniqLockValue))){
//加锁成功 设置过期时间
$this->objRedis->expire($lockKey,self::LOCK_KEY_EXPIRE);
return true;
}
return false;
}
/**
* 释放锁, 如果指定key则释放指定key，否则全部释放
* @param $strKey string
*/
public function unLock($strKey = '') {
$lockKey = "PRODUCT_LOCK_KEY";
$currentTime = time();
if(($uniqLockValue == $this->objRedis->get($lockKey)) && $this->objRedis->ttl($lockKey) != -2){
$redis->del($key);
return true;
}else{
return false;
}
}
private function __clone () {
}
}
```

其实这样实现一把锁是存在一些问题的，其中的很多操作都不能保证其原子性，在平常的情况下不容易显现出来，接下来我带大家再进一步优化这把锁。

// todo ttl

