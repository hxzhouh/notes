

从 VB.net、Java、C# 和 Python 开始转到 Go开发的时候，我对Go语言层级的模式的缺乏有点懊恼，这促使我花了一点时间找出容易表达的那些模式。

这里是一些通用的模式的集合，以及我发现的最容易表示它们的方式。

## 装饰器(Decorator)

这个特性在大部分的编程语言中都有广泛的应用， 使用某种效果或者属性来加强一个函数或者方法的功能。

如果你熟悉python, 你可能属性下面的代码：

```python
@login_required
@app.route('/private')
def get_secret():
	# code here
```

或者c#中类似的代码：

```c#
[Authorize(Roles = "User")]
public class SecretManager: Controller
{
	[Route("/private")]
	public ActionResult GetSecret()
	{
		// Code here
	}
}
```

这段代码是说当你访问`/private`路径的数据时，你需要身份验证。

上面的这种编程风格有几个危险而令人讨厌的问题：

- 很容易忘记注解（annotation），或者放错地方
- 为了理解代码的正确性，你需要额外地了解这种编程语言的特性（比如注解顺序是否有相关性）
- 不容易发现注解定义的地方
- 控制逻辑隐藏在注释中

Go使用另外一种方式：

```go
Authenticate(http.HandlerFunc(w http.ResponseWriter,r *http.Response){
 // Code here
})
```
一个简单的Authenticate实现如下:
```go
func Authenticate(h http.Handler) http.Handler {
	return http.HandlerFunc(w http.ResponseWriter,r *http.Response){
		if !isAuth(r) {
			w.WriteHeader(http.StatusForbidden)
			w.Write(forbiddenMsg)
			return
		}
		h.ServeHTTP(w, r)
	}
}
```

此外，这种模式允许Go开发者装饰整个类型，而不仅仅是函数。你可以为`(http.ServeMux).ServeHTTP`增加点东西，例如增加一些缺省的Header:

```go
var securityHeaders = map[string]string{
	"Strict-Transport-Security": "max-age=31536000; includeSubDomains",
	"X-XSS-Protection":          "1; mode=block",
	"X-Frame-Options":           "SAMEORIGIN",
	"X-Content-Type-Options":    "nosniff",
}

type secureMux http.ServeMux

func (s *secureMux) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	for key, value := range securityHeaders {
		w.Header().Set(key, value)
	}
	s.ServeMux.ServeHTTP(w, r)
}

func setup() {
	// Cast the new ServeMux to a secureMux.
	serveMux := secureMux(http.NewServeMux())

	// Register handlers.
	serveMux.Handle("/private", Authenticate(myAuthHandler))
	serveMux.Handle("/settings", Authenticate(myAuthHandler))
	serveMux.Handle("/", greetingsHandler)

	// Use serveMux
	srv := &http.Server{Handler: serveMux /* more configs here */}
	if err := srv.ListenAndServe(); err != nil {
		log.Println(err)
	}
}
```

这个例子在初始阶段发现缺少的身份认证调用是很容易的：所有的handler都在一个函数中隐式地调用，而不是handler定义的地方所有的路由都在一个地方进行处理，这样你就不容易遗漏。

## 单例(Singleton)

单例是一个通用的表达方式，来表示只存在程序中某个地方的某个东西。它可以延迟初始化，也可以启动时就初始化，依赖这个值何时初始化。非延迟初始化的单例一般实现为全局变量，它们可以在 `init` 函数中初始化，或者在声明的时候初始化：

```go
var myOnlyInstance *myType
func init() {
	// Prepare things that are needed here

	myOnlyInstance = newMyType()
}
```

如果没有初始化的前置条件，用下面的方式更好：

var myOnlyInstance = newMyType()

这很平常， 我以前使用的大部分语言都这样做。 唯一的一个大的不同点是 Java/C#中这个变量需要是一个类的静态变量(static)。

Go保证 init 函数会在 main 函数之前被执行，所以可以保证这些值可以在使用之前已经被初始化了。

我们来谈谈延迟初始化(lazy)，Java中的方式如下：

```java
public class MyType {
	static MyType single;
	public static MyType getSingle() {
		if (single == null) {
			single = new MyType();
		}
		return single;
	}
}
```

我看到这样的写法有几百次了，但是这里有个问题： 这里有个并发问题。如果多个并发访问getSingle, 这个值可能被初始化多次，也就不再是单例了。为了解决这个问题，需要加上`synchronized`关键字，但是这又影响性能。

> 译者按: 这个Java中的一个经典问题， Java实现正确的单例模式至少有5中写法: 静态变量初始化、synchronized、双重检查、静态内部类、枚举类型

这种模式在Go中可以使用`sync.Once`实现:

```go
// The zero value for Once is ready to use
var oSingle sync.Once
var single *myType

func getSingle() *myType {
	oSingle.Do(func(){ single = newMyType() })
	return single
}
```

这有三个好处:

- 保证有且只有一次调用初始化
- 并发访问会被阻塞，知道初始化完成
- 初始化完成后调用很快

如果你对性能比较关心，可能注意到两个月前的一个更新可以将方法调用减少到 0.5纳秒。

使用装饰模式和单例模式，你可以巧妙地将一个不线程安全的API转换成安全的API。

例如， `exec.Cmd`不允许调用`Wait`方法[多次](https://github.com/golang/go/issues/28461)，并发访问的时候怎么办呢？你可以像这样进行包装：

```go
type multiWaitableCmd struct {
	exec.Cmd // Promote all Cmd methods
	o   sync.Once
	err error
}

// Wait decorates `(*exec.Cmd).Wait` with a `(*sync.Once).Do()` call
func (mwc *multiWaitableCmd) Wait() error {
	mwc.o.Do(func() { mwc.err = mwc.Cmd.Wait() })
	return mwc.err
}
```

## 静态成员（static member）

这个模式只用在很稀少的场景，你需要某个struct类型的所有的实例需要共享同一个值。 我最近偶然发现的一个例子是出于调试目的：一个接口`I`实现了`Kind() string`函数，用来在日志系统中打印类型。

这太容易实现了，而且也不会弄乱包命名空间：

```go
type myImpl struct{}

func (*myImpl) Kind() string {
	return "Best implementation"
}
```

注意`Kind()`需要一个指针类型的receiver，但是它并没有为这个receiver命名，所以很清晰的表明我们并不使用类型实例。

## 信号量(Semaphore)

因为我一直对并发模式感兴趣，我很惊讶Go居然没有信号量类型。

当然使用channel来模拟信号量是很容易的:

```go
// Semaphore can be created with `make(Semaphore, size)`
type Semaphore chan struct{}

// Lock uses channels send operations to emulate an "acquire".
func (s Semaphore) Lock()   { s <- struct{}{} }
// Lock uses channels receive operations to emulate a "release".
func (s Semaphore) Unlock() { <-s }
```

注意信号量实现了`sync.Locker`接口。这段代码工作很正常， 往一个 unbuffered channel中发送一个值会被阻塞，知道其它的goroutine调用了Unlock方法，从channel中读取了这个值。如果channel的buffer设置为1, 我们就取得了类似Mutex的效果。

一个更有效带权重的实现可以在Go的实验包[找到](https://godoc.org/golang.org/x/sync/semaphore)。

## 错误组(Errgroup)

有时候你想创建多个goroutine，让它们并行地工作，当遇到某种错误或者你不像再输出了，你可能想取消整个goroutine。

为了取得这个效果你可以使用 sync.Waitgroup 和 context.Context 是可行的，但是这得需要很多冗余代码，我建议使用[errgroup.Errgroup](https://godoc.org/golang.org/x/sync/errgroup)。

例子：

```go
import "golang.org/x/sync/errgroup"

   ......

eg, ctx := errgroup.WithContext(context.TODO())
for _, w := range work {
		w := w
		eg.Go(func() error {
			// Do something with w and
			// listen for ctx cancellation
		})
}
// If any of the goroutines returns an error ctx will be
// canceled and err will be non-nil.
if err := eg.Wait(); err != nil {
	return err
}
```