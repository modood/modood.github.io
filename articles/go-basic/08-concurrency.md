『Go 语法基础』之并发
=====================

Table of contents
-----------------

*   [goroutine](#goroutine)
*   [channel](#channel)
*   [select](#select)

## goroutine

goroutine 类似一个线程，但是 goroutine 是由 go 自己调度，而不是由操作系统调度。

goroutine 很容易创建且开销较小。最终多个 goroutine 将会在同一个底层的系统线程上运行。
这也常称之为 M:N 线程模型，因为我们有 M 个应用线程（goroutine）运行在 N 个系统线程上。

启动一个 goroutine：

```go
go func() {
    fmt.Println("processing")
}()
```

## channel

goroutine 可以让代码并发执行，并发执行的代码需要协同。

channel 是一个通信管道，它用于 goroutine 之间传递数据。

从一个信道接收或者发送数据时会阻塞，信道是共享状态的唯一方式，确保在任意时刻只有一个 goroutine 可以访问一个特定的数据。

```go
// 创建一个信道
c := make(chan int)

// 发送数据
CHANNEL <- DATA

// 接收数据
VAR := <-CHANNEL
```

信道拥有内置额度缓存能力，可以更好地处理数据突然飙升。使用 make 创建一个通道时，可以指定通道的长度：

```go
c := make(chan int, 100)
```

使用内置函数 close 可以关闭一个信道，通常情况下无需关闭它们。只有在需要告诉接收者没有更多的数据的时候才有必要进行关闭，例如中断一个 range。
接收者可以通过赋值语句的第二参数来测试信道是否已经被关闭。

使用 for range 不断从 channel 接收值，直到它被关闭：

```go
for i := range c {
    fmt.Println(i)
}
```

关闭一个信道：

```go
close(c)
```

从一个信道接收数据并检查是否已被关闭：

```go
v, ok := <- c
```

## select

select 用于管理多个信道。

给定多个信道，如果有多个信道都可用时，select 随机选择其中的一个，如果没有可用的通道，若提供了 default 语句则立即执行该分支，
否则将会阻塞直到有一个信道可用。

示例，没有信道可用时丢弃数据（不建议）：

```go
select {
case c <- rand.Int():
  // do something
default:
  fmt.Println("dropped")
}
```

示例，没有信道可用一段时间后超时处理（建议）：

```go
select {
case c <- rand.Int():
  // do something
case <-time.After(time.Millisecond * 100):
  fmt.Println("timed out")
}
```

