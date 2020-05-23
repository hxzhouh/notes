# 1. RPC 入门

### 1.1 RPC 框架原理

### 1.2 业界主流的 RPC 框架

## 1.3 gRPC 简介

### 1.3.1 gRPC 概览

### 1.3.2 gRPC 特点

# 2. gRPC 服务端创建

## 2.1 服务端创建业务代码

## 2.2 服务端创建流程

## 2.3 服务端 service 调用流程

# 3. 源码分析

## 3.1 主要类和功能交互流程

### 3.1.1 gRPC 请求消息头处理

### 3.1.2 gRPC 请求消息体处理和服务调用

### 3.1.3 gRPC 响应消息处理

## 3.2 源码分析

### 3.2.1 Netty 服务端创建

### 3.2.2 服务实例创建和绑定

```go
type Server struct {
   opts serverOptions

   mu     sync.Mutex // guards following
   lis    map[net.Listener]bool
   conns  map[transport.ServerTransport]bool // 服务端传输端口，grpc 内部包。
   serve  bool
   drain  bool
   cv     *sync.Cond          // signaled when connections close for GracefulStop
   m      map[string]*service // service name -> service info
   events trace.EventLog

   quit               *grpcsync.Event
   done               *grpcsync.Event
   channelzRemoveOnce sync.Once
   serveWG            sync.WaitGroup // counts active Serve goroutines for GracefulStop

   channelzID int64 // channelz unique identification number
   czData     *channelzData
}
```

```go
type ServerTransport interface {
   // HandleStreams receives incoming streams using the given handler.
    // 用指定的方法处理输入流
   HandleStreams(func(*Stream), func(context.Context, string) context.Context)

   // WriteHeader sends the header metadata for the given stream.
   // WriteHeader may not be called on all streams.
    // 写mateData。
   WriteHeader(s *Stream, md metadata.MD) error

   // Write sends the data for the given stream.
   // Write may not be called on all streams.
   Write(s *Stream, hdr []byte, data []byte, opts *Options) error

   // WriteStatus sends the status of a stream to the client.  WriteStatus is
   // the final call made on a stream and always occurs.
   WriteStatus(s *Stream, st *status.Status) error

   // Close tears down the transport. Once it is called, the transport
   // should not be accessed any more. All the pending streams and their
   // handlers will be terminated asynchronously.
   Close() error

   // RemoteAddr returns the remote network address.
   RemoteAddr() net.Addr

   // Drain notifies the client this ServerTransport stops accepting new RPCs.
   // Drain通知客户端此ServerTransport停止接受新的RPC。
   Drain()

   // IncrMsgSent increments the number of message sent through this transport.
    // 统计功能。
   IncrMsgSent()

   // IncrMsgRecv increments the number of message received through this transport.
    // 统计功能
   IncrMsgRecv()
}
```

### 3.2.3 service 调用

## 3.3 服务端线程模型