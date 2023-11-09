---
title: Golang基础笔记(下)
date: 2021-11-16 13:59:00
tags:
categories:
- Golang
---

# 一、接口
## 例子
定义测试接口  testInterface/mock/main.go
```go
package mock

type Retriever struct {
    Contents string
}

// 给结构体添加方法
func (r Retriever) Get(url string) string {
    return r.Contents
}
```
<!--more-->

定义真实接口   testInterface/real/main.go
```go
package real

import (
    "net/http"
    "net/http/httputil"
    "time"
)

type Retriever struct {
    UserAgent string
    TimeOut time.Duration
}

func (r Retriever) Get(url string) string  {
    res,err := http.Get(url)
    if err != nil {
        panic(err)
    }

    result,err := httputil.DumpResponse(res,true)
    if err != nil {
        panic(err)
    }

    // 读完 Response 的 body 后需要关掉
    res.Body.Close()

    return string(result)
}
```

使用两个接口
```go
package main

import (
    "fmt"
    "testInterface/mock"
    "testInterface/real"
)

// 定义接口
type Retriver interface {
    Get(url string) string
}

// 定义方法，调用接口
func download(r Retriver) string {
    return r.Get("https://baidu.com")
}

func main() {
    var r Retriver

    r = mock.Retriever{Contents: "this is a mock"}
    fmt.Println(download(r))

    var newR Retriver
    newR = real.Retriever{}
    fmt.Println(download(newR))

}
```

## 例子
```go
package main

import (
    "fmt"
)

type Phone interface {
    call()
}

type NokiaPhone struct {
}

func (nokiaPhone NokiaPhone) call() {
    fmt.Println("I am Nokia, I can call you!")
}

type IPhone struct {
}

func (iPhone IPhone) call() {
    fmt.Println("I am iPhone, I can call you!")
}

func main() {
    var phone Phone

    phone = new(NokiaPhone)
    phone.call()

    phone = new(IPhone)
    phone.call()

}
```

# 二、函数式编程 
## 闭包实现一个累加器
```go
//    闭包（实现0加到i，结果是多少）

func adder() func(int) int {
    sum := 0
    return func(v int) int {
        sum += v
        return sum
    }
}

func main() {
    a := adder()
    for i := 0; i < 10; i ++{
        fmt.Printf("0 加到 %d 是 %d\n",i,a(i))
    }
}

//0 加到 0 是 0
//0 加到 1 是 1
//0 加到 2 是 3
//0 加到 3 是 6
//0 加到 4 是 10
//0 加到 5 是 15
//0 加到 6 是 21
//0 加到 7 是 28
//0 加到 8 是 36
//0 加到 9 是 45
```

## 正统函数式编程，实现上述例子
```go
type iAdder func(int) (int, iAdder)

func adder(base int) iAdder {
    return func(v int) (int,iAdder) {
        return base + v, adder(base + v)
    }
}

func main() {
    a := adder(0)
    for i := 0; i < 10; i ++{
        var s int
        s,a = a(i)
        fmt.Printf("0 加到 %d 是 %d\n",i,s)
    }
}
```

## 利用闭包实现，斐波那契数列
```go
// fibonacci（i = 前面两个数之和）

func fibonacci() func() int {
    a,b := 0,1
    return func() int {
        a,b = b, a + b
        return a
    }
}

func main() {
    f := fibonacci()
    fmt.Println(f()) // 1
    fmt.Println(f()) // 1
    fmt.Println(f()) // 2
    fmt.Println(f()) // 3
    fmt.Println(f()) // 5
    fmt.Println(f()) // 8
}
```

## 函数实现接口
```go
// fibonacci（i = 前面两个数之和）

type intGen func() int

func (g intGen) Read (p []byte) (n int, err error)  {
    next := g()
    if next > 10000 { // 当数量大于10000就退出
        return 0,io.EOF
    }
    s := fmt.Sprintf("%d\n",next)

    // TODO: incorrect if p is too small
    return strings.NewReader(s).Read(p)
}

func printFileContent(reader io.Reader)  {
    scanner := bufio.NewScanner(reader)
    for scanner.Scan() {
        fmt.Println(scanner.Text())
    }
}

func fibonacci() intGen {
    a,b := 0,1
    return func() int {
        a,b = b, a + b
        return a
    }
}

func main() {
    f := fibonacci()
    printFileContent(f)
}
```

# 三、错误处理和资源管理
## defer 调用
### 例子
```go
func main() {
    fmt.Println(1)
    defer fmt.Println(3)
    fmt.Println(2)
    defer fmt.Println(4)
    // 1 2 4 3
    // defer函数 会在 主函数结束前调用
    // defer 语句是按 先进后出 顺序执行
}
```

### 例子
```go
func main() {
    file,err := os.Create("haha.txt")
    if err != nil {
        panic(err)
    }
    defer file.Close() // 关闭文件

    writer := bufio.NewWriter(file)
    defer writer.Flush() // 写入文件

    for i := 0; i < 100; i ++ {
        fmt.Fprintln(writer,i)
    }
}
```

## 错误处理
```go
func main() {
    file,err := os.OpenFile("haha.txt",os.O_EXCL|os.O_CREATE,0666)
    if err != nil {
        // 若 error 是 *os.PathError
        if patherr,ok := err.(*os.PathError);ok{
            fmt.Println(patherr.Op)
            fmt.Println(patherr.Path)
            fmt.Println(patherr.Err)
        } else {
            // 新建 error
            err = errors.New("other error")
            panic(err)
        }
        return
    }
    defer file.Close() // 关闭文件
}
```

# 四、测试与性能调优
## 表格驱动测试
main.go
```go
package main

func Add(a,b int) int {
    return a + b
}

func main()  {
}
```
add_test.go
```go
package main

import "testing"

func TestAdd(t *testing.T) {
    tests := []struct{ a,b,c int} {
        {1,2,3},
        {1,5,3},
        {3,3,6},
    }

    for _,tt := range tests{
        if res := Add(tt.a,tt.b); res != tt.c{
            t.Errorf("计算错误：%d + %d = %d，而不是 %d",tt.a,tt.b,res,tt.c)
        }
    }
}
```
开始测试
```shell
go test .
```

## 代码覆盖率测试
```shell
go test -coverprofile=cover.out ./internal/... 
go tool cover -html=cover.out
```
*(./internal/...) 表示internal下所有的测试代码*

## 代码性能测试(基准测试)
```go
func BenchmarkAdd(b *testing.B) {
    var a,aa int
    a = 1000
    aa = 2000

    for i := 0; i < b.N; i++ {
        if res := Add(a,aa); res != 3000{
            b.Errorf("计算错误：%d + %d = %d，而不是 %d",a,aa,res,3000)
        }
    }
}
```

测试命令
```shell 
go test -bench .
```

## 利用 pprof 进行性能调优
### 安装 [Graphviz](https://graphviz.org/download/)
### 查看性能
```shell
go test -bench . -cpuprofile cpu.out 
go tool pprof cpu.out
web
```

# 五、Goroutine（协程）
## goroutine 概述
* 轻量级“线程”
* 非抢占式多任务处理，由协程主动交出控制权
* 编译器 / 解释器 / 虚拟机层面的多任务
* 多个协程可能在一个或多个线程上运行（线程数量一般不大于机器核数）

## goroutine 并发例子
```go
func main()  {
    for i := 0; i < 1000; i++ {
        go func(j int) {
            //for {
                fmt.Println("i am routine",j)　　　　　　　　　　// runtime.Gosched() // 手动交出控制权
            //}
        }(i)
    }
    time.Sleep(time.Millisecond)
}
```

## goroutine 可能会切换的点
>解释：任何函数加上go就能送到调度器运行，调度器会在合适的位置进行协程的切换
* I/O,select
* channel
* 等待锁
* 函数调用（有时）
* runtime.Gosched()

## 检测数据访问冲突（检测race condition）
代码
```go
func main()  {
    a := 1
    for i := 0; i < 1000; i++ {
        go func(j int) { //race condition
            for {
                a++
            }
        }(i)
    }
    time.Sleep(time.Millisecond)
}
```
检测命令
```shell
go run -race test.go
```

# 六、Channel（通道） 

## 简单收发channel
```go
func main()  {
    chanDemo()
}

func chanDemo() {
    c := make(chan int)
    go func() {
        for {
            n := <- c // 接收 channel 的数据
            fmt.Println(n)
        }
    }()

    // 发送 channel 数据
    c <- 1
    c <- 2

    time.Sleep(time.Millisecond)
}
```

## channel 批量收发数据
```go
func main()  {
    chanDemo()
}

func chanDemo() {
    // 定义 channel 数组
    var channels [10]chan int

    // 批量收数据
    for i := 0; i < 10; i++ {
        channels[i] = make(chan int)
        go worker(i,channels[i])
    }

    // 批量发数据
    for i := 0; i < 10; i++ {
        channels[i] <- i + 'a'
    }

    time.Sleep(time.Millisecond)
}

func worker (i int, c chan int){
    for {
        n := <- c // 接收 channel 的数据
        fmt.Printf("接收来自 %d 通道，数据%v\n",i,n)
    }
}
```

## channel 通道类型
```go
// 双向通道
var a chan int

// 仅发送类型
var b chan<- int

//仅接收类型
var c <-chan int
```

## channel 的缓冲区
```go
func main()  {
    bufferedChan()
}

func bufferedChan() {
    c := make(chan int,3) // 给通道设定缓冲区
    go worker(0,c)
    c <- 11
    c <- 22
    c <- 33
    close(c) // 关闭通道

    time.Sleep(time.Millisecond)
}

func worker (i int, c chan int){
    // 若从通道收不到数据就退出
    //for {
    //    n,ok := <- c
    //    if !ok {
    //        break
    //    }
    //    fmt.Printf("接收来自 %d 通道，数据%v\n",i,n)
    //}

    // 同理，若通道有数据就打印
    for n := range c {
        fmt.Printf("接收来自 %d 通道，数据%v\n",i,n)
    }
}
```

## channel 等待所有 goroutine 结束
```go
func main()  {
    chanDemo()
}

type workStruct struct {
    in chan int
    done chan bool
}

func chanDemo() {
    // 定义 channel 数组
    var channels [10]workStruct
    for i := 0; i < 10; i++ {
        channels[i] = workStruct{
            in : make(chan int),
            done: make(chan bool),
        }
    }

    // 批量收数据
    for i,w := range channels{
        go worker(i,w)
    }

    // 批量发数据
    for i,w := range channels{
        w.in <- 'a' + i
    }

    // 当接收完 channels 里面的 done ，表示 channel 执行完毕
    for _,w := range channels{
        <-w.done
        close(w.in)
        close(w.done)
    }

    fmt.Printf("执行后续操作")
}

func worker (i int, c workStruct){
    // 同理，若通道有数据就打印
    for n := range c.in {
        fmt.Printf("接收来自 %d 通道，数据%v\n",i,n)
        c.done <- true
    }
}
```

## WaitGroup 等待所有 goroutine 结束
```go
func main()  {
    chanDemo()
}

type workStruct struct {
    in chan int
    wg *sync.WaitGroup
}

func chanDemo() {
    var wg sync.WaitGroup

    // 定义 channel 数组
    var channels [10]workStruct
    for i := 0; i < 10; i++ {
        channels[i] = workStruct{
            in : make(chan int),
            wg: &wg,
        }
    }

    wg.Add(10)

    // 批量收数据
    for i,w := range channels{
        go worker(i,w)
    }

    // 批量发数据
    for i,w := range channels{
        w.in <- 'a' + i
    }

    wg.Wait()

    fmt.Printf("执行后续操作")
}

func worker (i int, c workStruct){
    // 若通道有数据就打印
    for n := range c.in {
        fmt.Printf("接收来自 %d 通道，数据%v\n",i,n)
        c.wg.Done()
    }
}
```

## select 接收或发送某个 channel 的值
```go
func main() {
    var c1, c2 = generator(), generator()
    for {
        select {
        case n := <-c1:
            fmt.Println("c1里面来了数据", n)
        case n := <-c1:    
            fmt.Println("走这里", n) 
        case n := <-c2:
            fmt.Println("c2里面来了数据", n)
        }
    }
}

func generator() chan int {
    out := make(chan int)
    go func() {
        i := 0
        for {
            time.Sleep(time.Duration(rand.Intn(100)) * time.Millisecond)
            out <- i
            i++
        }
    }()
    return out
}
```
## select 语法概述
* 每个 case 都必须是一个通信
* 所有 channel 表达式都会被求值
* 所有被发送的表达式都会被求值
* 如果任意某个通信可以进行，它就执行，其他被忽略。
* 如果有多个 case 都可以运行，Select 会随机公平地选出一个执行。其他不会执行。
* 否则：
     - 如果有 default 子句，则执行该语句。
     - 如果没有 default 子句，select 将阻塞，直到某个通信可以运行；Go 不会重新对 channel 或值进行求值

## 传统同步机制
```go
type atomicInt struct {
    value int
    lock sync.Mutex
}

func (a *atomicInt) increment() {
    a.lock.Lock()
    defer a.lock.Unlock()
    a.value++
}

func (a *atomicInt) get() int {
    a.lock.Lock()
    defer a.lock.Unlock()
    return int(a.value)
}

func main() {
    var a atomicInt
    a.increment()
    go func() {
        a.increment()
    }()
    time.Sleep(time.Millisecond)
    fmt.Println(a.get())
}
```
