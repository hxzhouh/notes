Golang的sync包中的Cond实现了一种条件变量，可以使用在多个Reader等待共享资源ready的场景（如果只有一读一写，一个锁或者channel就搞定了）。

Cond的汇合点：多个goroutines等待、1个goroutine通知事件发生。

每个Cond都会关联一个Lock（*sync.Mutex or *sync.RWMutex），当修改条件或者调用Wait方法时，必须加锁，**保护condition**。lock 可以是 *Mutex 或者 *RWMutex

```go
type Cond struct {
   noCopy noCopy

   // L is held while observing or changing the condition
   L Locker

   notify  notifyList
   checker copyChecker
}
```

## 源码解读

![](https://blog-image-1253555052.cos.ap-guangzhou.myqcloud.com/20200331195906.png)

### NewCond

```go
unc NewCond(l Locker) *Cond {
   return &Cond{L: l}
}
```

新建一个Cond条件变量。

### Cond

#### Broadcast

``` go
func (c *Cond) Broadcast()
```

Broadcast会唤醒**所有**等待c的goroutine。

调用Broadcast的时候，可以加锁，也可以不加锁。

#### Signal

```go
func (c *Cond) Signal()
```

Signal只唤醒**1个**等待c的goroutine。

调用Signal的时候，可以加锁，也可以不加锁。

#### Wait

```
func (c *Cond) Wait()
```

`Wait()`会自动释放`c.L`，并挂起调用者的goroutine。之后恢复执行，`Wait()`会在返回时对`c.L`加锁。

除非被Signal或者Broadcast唤醒，否则`Wait()`不会返回。

由于`Wait()`第一次恢复时，`C.L`并没有加锁，所以当Wait返回时，调用者通常并不能假设条件为真。

取而代之的是, 调用者应该在循环中调用Wait。（简单来说，只要想使用condition，就必须加锁。）