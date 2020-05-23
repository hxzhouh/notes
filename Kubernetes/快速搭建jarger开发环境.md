docker pull jaegertracing/all-in-one

```bash
docker run -d --name jaeger \
  -e COLLECTOR_ZIPKIN_HTTP_PORT=9411 \
  -p 5775:5775/udp \
  -p 6831:6831/udp \
  -p 6832:6832/udp \
  -p 5778:5778 \
  -p 16686:16686 \
  -p 14268:14268 \
  -p 14250:14250 \
  -p 9411:9411 \
  jaegertracing/all-in-one:latest
```

# consul

**单机**

\# docker获取consul并创建容器的步骤

```bash
# docker pull consul
# docker run --name consul -d -p 8500:8500 -p 8600:8600/udp consul
```

**集群**

\# 建立consul集群命令步骤

\# 建立第一个容器，并启动第一个consul服务

\# docker run --name consul1 -d -p 8500:8500 -p 8300:8300 -p 8301:8301 -p 8302:8302 -p 8600:8600 consul agent -server -bootstrap-expect 2 -ui -bind=0.0.0.0 -client=0.0.0.0

\# 上诉命令字段解析

\#8500 http 端口，用于 http 接口和 web ui

\#8300 server rpc 端口，同一数据中心 consul server 之间通过该端口通信

\#8301 serf lan 端口，同一数据中心 consul client 通过该端口通信

\#8302 serf wan 端口，不同数据中心 consul server 通过该端口通信

\#8600 dns 端口，用于服务发现

\#-bbostrap-expect 2: 集群至少两台服务器，才能选举集群leader

\#-ui：运行 web 控制台

\#-bind： 监听网口，0.0.0.0 表示所有网口，如果不指定默认未127.0.0.1，则无法和容器通信

\#-client ： 限制某些网口可以访问

\# 获取第一个容器IP地址

\# docker inspect --format "{{ .NetworkSettings.IPAddress }}" consul1

\# 输出是：172.17.0.2

\# 启动第二个consul服务：consul2， 并加入consul1（使用join命令）

\# docker run --name consul2 -d -p 8501:8500 consul agent -server -ui -bind=0.0.0.0 -client=0.0.0.0 -join 172.17.0.2

\# 启动第三个consul服务：consul3， 并加入consul1（使用join命令）

\# docker run --name consul3 -d -p 8502:8500 consul agent -server -ui -bind=0.0.0.0 -client=0.0.0.0 -join 172.17.0.2

\#(同样的步骤，可以启动第四，第五甚至更多的consul服务)

\# 宿主机浏览器访问：http://localhost:8500 或者 http://localhost:8501 或者 http://localhost:8502

 