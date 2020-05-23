> 原文链接:  [Go: How Does Go Stop the World? ](https://medium.com/a-journey-with-go/go-how-does-go-stop-the-world-1ffab8bc8846): 
>            Author :   [Vincent Blanchon](https://medium.com/@blanchon.vincent?source=post_page-----1ffab8bc8846----------------------)

![](https://blog-image-1253555052.cos.ap-guangzhou.myqcloud.com/20200428104804.png)

**本文基于 go 1.13**

在垃圾回收算法中，Stop The Word（STW）是一个很重要的概念，他会中断程序运行，添加写屏障，`以便扫描内存` ,现在一起来看看它内部的原理以及可能存在的问题

# STW

停止程序运行意味着停止所有运行态的`goroutines`,一个简单的例子:

```go
func main() {
   runtime.GC()
}
```

运行垃圾回收算法将触发两个阶段的STW 

> 有关垃圾回收的更多细节，请参考同作者的另外一篇文章*“*[*Go: How Does the Garbage Collector Mark the Memory?*](https://medium.com/a-journey-with-go/go-how-does-the-garbage-collector-mark-the-memory-72cfc12c6976)

## STW的步骤

第一步，抢占所有正在运行的`goroutines`

![](https://blog-image-1253555052.cos.ap-guangzhou.myqcloud.com/20200428164159.png)

第二步，一旦 `goroutines`被抢占，正在运行的`goroutines`将在安全的地方暂停，然后所有的p<sup>[1]</sup>都将被标记为暂停，停止运行任何代码。

![](https://blog-image-1253555052.cos.ap-guangzhou.myqcloud.com/20200428164444.png)

第三步，然后，go调度器将M<sup>[2]</sup>与P分离,并且将M放到空闲列表里面。

![](https://blog-image-1253555052.cos.ap-guangzhou.myqcloud.com/20200428164609.png)

对于在每个M上运行的Goroutines，它们将在全局队列<sup>[3]</sup>>中等待：

![](https://blog-image-1253555052.cos.ap-guangzhou.myqcloud.com/20200428164654.png)

那么，一旦所有的`goroutines`都停止了，那么唯一活跃的`goroutines` （垃圾回收`goroutines`）将会安全的运行，并且在垃圾回收完成后，重新拉起所有的`goroutines`。具体情况，可以通过 go trace查看。

![](https://blog-image-1253555052.cos.ap-guangzhou.myqcloud.com/20200428165140.png)

# System calls

STW时期可能会影响系统调用，因为系统调用可能会在stw时期返回，通过密集执行系统调用的程序来看看怎样处理这种情况，

```go
func main() {
   var wg sync.WaitGroup
   wg.Add(10)   for i := 0; i < 10; i++ {
      go func() {
         http.Get(`https://httpstat.us/200`)
         wg.Done()
      }()
   }   wg.Wait()
}
```

他的trace情况。

![](https://blog-image-1253555052.cos.ap-guangzhou.myqcloud.com/20200428165604.png)

系统调用在STW时期返回，但是现在已经没有P在运行了。所以他会放到全局队列里面,等待STW结束后再运行。

# Latencies

STW的第三步将所有的M与P分离。然而，go将等待调度程序运行、系统调用等自动停止。等待goroutine被抢占应该很快，但是在某些情况下会产生一些延迟，下面是一个极端的例子：

``` go
func main() {
   var t int
   for i := 0;i < 20 ;i++  {
      go func() {
         for i := 0;i < 1000000000 ;i++ {
            t++
         }
      }()
   }

   runtime.GC()
}
```

STW时长达到了2.6S

![](https://blog-image-1253555052.cos.ap-guangzhou.myqcloud.com/20200428172521.png)

没有函数调用的goroutine将不会被抢占，并且它的P在任务结束之前不会被释放。这将迫使STW等待他， 有几种解决方案可改善循环中的抢占，有关其的更多信息，可以查看作者另外一篇文章 [[Go: Goroutine and Preemption).](https://medium.com/a-journey-with-go/go-goroutine-and-preemption-d6bc2aa2f4b7)