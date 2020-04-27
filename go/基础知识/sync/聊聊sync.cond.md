> Cond的主要作用就是获取锁之后，wait()方法会等待一个通知，来进行下一步锁释放等操作，以此控制锁合适的释放，释放频率等。适用于在并发环境下goroutine的等待和通知。

> 本文基于源码 113.5 

# 例子

先来个例子看看cond 的用法

https://play.golang.org/p/7JrAUyTIXpP

``` go 
package main

import (
	"fmt"
	"sync"
	"time"
)

var locker = new(sync.Mutex)
var cond = sync.NewCond(locker)

func main() {
	for i := 0; i < 40; i++ {
		go func(x int) {
			cond.L.Lock()         //获取锁
			defer cond.L.Unlock() //释放锁
			cond.Wait()           //等待通知,阻塞当前goroutine
			fmt.Println(x)
			time.Sleep(time.Second * 1)

		}(i)
	}
	time.Sleep(time.Second * 1)
	fmt.Println("Signal...")
	cond.Signal() // 下发一个通知给已经获取锁的goroutine
	time.Sleep(time.Second * 1)
	cond.Signal() // 3秒之后 下发一个通知给已经获取锁的goroutine
	time.Sleep(time.Second * 3)
	cond.Broadcast() //3秒之后 下发广播给所有等待的goroutine
	fmt.Println("Broadcast...")
	time.Sleep(time.Second * 60)
}
```

输出

```bash 
Signal...
0
2
Broadcast...
19
....
```

# 源码分析



![](https://blog-image-1253555052.cos.ap-guangzhou.myqcloud.com/20200424171644.png)

## Cond

``` go
type Cond struct {
    // noCopy可以嵌入到结构中，在第一次使用后不可复制,使用go vet作为检测使用
	noCopy noCopy
	// L is held while observing or changing the condition,
    // 可以根据需求初始化成不同的锁
	L Locker
	//// 通知列表,调用Wait()方法的goroutine会被放入list中,每次唤醒,从这里取出
	notify  notifyList
     // 复制检查,检查cond实例是否被复制
	checker copyChecker
}
```

## notifyList

``` go
type notifyList struct {
	wait   uint32
	notify uint32
	lock   uintptr
	head   unsafe.Pointer
	tail   unsafe.Pointer
}
```

## 函数

### NewCond

``` go 
func NewCond(l Locker) *Cond {
	return &Cond{L: l}
}
```



Cond的构造函数，用于初始化Cond，参数为Locker实例初始化,传参数的时候必须是引用或指针,比如&sync.Mutex{}或new(sync.Mutex)，不然会报异常

### Wait

``` go
func (c *Cond) Wait() {
    // 检查c是否是被复制的，如果是就panic
	c.checker.check()
	// 将当前goroutine加入等待队列
	t := runtime_notifyListAdd(&c.notify)
	// 解锁
	c.L.Unlock()
	// 等待队列中的所有的goroutine执行等待唤醒操作
	runtime_notifyListWait(&c.notify, t)
	c.L.Lock()
}
```



等待自动解锁c.L和暂停执行调用goroutine。恢复执行后,等待锁c.L返回之前。与其他系统不同，等待不能返回，除非通过广播或信号唤醒。

因为c。当等待第一次恢复时，L并没有被锁定，调用者通常不能假定等待返回时的条件是正确的。相反，调用者应该在循环中等待:

### Signal

```go
func (c *Cond) Signal() {
    // 检查c是否是被复制的，如果是就panic
	c.checker.check()
	// 通知等待列表中的一个 
	runtime_notifyListNotifyOne(&c.notify)
}
```

唤醒等待队列中的一个goroutine，一般都是任意唤醒队列中的一个goroutine，为什么没有选择FIFO的模式呢？这是因为FiFO模式效率不高，虽然支持，但是很少使用到。

### Broadcast

```go
func (c *Cond) Broadcast() {
    // 检查c是否是被复制的，如果是就panic
	c.checker.check()
	// 检查c是否是被复制的，如果是就panic
	runtime_notifyListNotifyAll(&c.notify)
}
```

唤醒等待队列中的所有goroutine。



# 一个真实的例子

我们来看看k8s中使用Cond实现的[FIFO](https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/client-go/tools/cache/fifo.go)，它是如何处理条件的消费的。

```go
func (f *FIFO) Pop(process PopProcessFunc) (interface{}, error) {
	f.lock.Lock()
	defer f.lock.Unlock()
	for {
		for len(f.queue) == 0 {
			// When the queue is empty, invocation of Pop() is blocked until new item is enqueued.
			// When Close() is called, the f.closed is set and the condition is broadcasted.
			// Which causes this loop to continue and return from the Pop().
			if f.IsClosed() {
				return nil, FIFOClosedError
			}

			f.cond.Wait()
		}
		id := f.queue[0]
    f.queue = f.queue[1:]
    ...
	}
}

func NewFIFO(keyFunc KeyFunc) *FIFO {
	f := &FIFO{
		items:   map[string]interface{}{},
		queue:   []string{},
		keyFunc: keyFunc,
	}
	f.cond.L = &f.lock
	return f
}
```

Cond共用了FIFO的lock，在Pop时，会先加锁 `f.lock.Lock()`，而在`f.cond.Wait()`前，会先检查`len(f.queue)`是否为0，防止2种情况：

1. 如上面的例子3，条件已经满足，不需要wait
2. 唤醒时满足，但被其他goroutine捷足先登，阻塞在`f.lock`的加锁中；当获得了锁，加锁成功以后，`f.queue`已经被消费为空，直接访问`f.queue[0]`会访问越界。