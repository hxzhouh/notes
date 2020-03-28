# go 测试

## 测试的必要性

### 测试逻辑是否符合预期

### 监控代码质量

### 可测试性也是好代码的标志

## go 测试工具包

### testing

### 测试代码必须以_test.go结尾

### 测试函数以Test开头

### 正常编译会忽略测试代码

### T

- fail

	- 失败继续执行当前测试函数

- FailNow

	- 失败立即终止当前测试函数

- SkipNow

	- 停止执行当前测试函数

- Log

	- 输出错误信息

- Parallel

	- 与有同样设置的测试函数并行执行

		- t.Parallel

- Error

	- Fail+Log

- Fatal

	- FailNow+Log

### 测试结果缓存

- todo

## 单元测试

### table driven

- golang 默认生成的那种格式

	- 测试数据跟逻辑分开

### testmain

- 执行初始化操作

### example driven

- leetcode风格的测试，打印结果对比

## 性能测试

### 以Benchmark名字为前缀

### go test 默认不会执行 性能测试需要家上 -bench

- go test -bench .

### 如果在性能测试中，需要做一些其他操作，但是又怕这部分操作影响到性能测试的准确性，可以使用timer 暂停计时，

- b.StopTimer()
// do somethings
b.StartTimer()

### memory

- -benchmen

## 代码覆盖率

### go test -cover

## 性能监控

### http/pprof

