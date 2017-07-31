『Go 语法基础』之语句
=====================

Table of contents
-----------------

*   [前言](#前言)
*   [循环（for）](#循环for)
*   [迭代（range）](#迭代range)
*   [跳出（break, continue, goto）](#跳出break-continue-goto)
*   [条件（if, else）](#条件if-else)
*   [分支（switch, case, fallthrough, default）](#分支switch-case-fallthrough-default)
*   [函数相关（return, defer）](#函数相关return-defer)
*   [go 程（go, select）](#go-程go-select)

## 前言

go 所有关键字中语句相关的关键字有以下：

```
if else for range switch select case fallthrough default
continue break defer go goto return
```

## 循环（for）

for 语句包含三个组成部分：初始化语句、循环条件表达式和后置语句。

初始化语句中定义的变量的作用域仅在 for 语句范围之内。

for 语句的三个组成部分，并不需要用 `()` 括起来，但是循环体必须使用 `{}` 括起来。

> while 循环？

for 语句的初始化语句和后置语句是可选的，省略掉的话将相当于其它编程语言中的 while 循环。

```go
for sum < 1000 {
    sum += sum
}
```

> 死循环？

for 语句的三个组成部分都省略掉的话，就是一个死循环。例如：

```go
for {
    // do something
}
```

## 迭代（range）

带 range 子句的 for 语句可以用于遍历字符串、数组、切片、映射和信道。

| 目标 | 第一个值 | 第二个值 |
|:-----|:---------|:---------|
| 字符串      | 下标(int)   | Unicode 码点的值（rune） |
| 数组、切片  | 下标（int） | 元素（特地类型） |
| 映射        | 键          | 值 |
| 信道        | 值          | 无 |

## 跳出（break, continue, goto）

break 语句终止最内层的 for、 switch 或 select 语句的执行。

continue 语句开始下一次最内层 for 循环的迭代。

goto 语句用于将控制转移到指定标签相应的语句。

## 条件（if, else）

if 语句可以像 for 循环一样在条件之前执行一个简单初始化语句，
初始化语句定义的变量的作用域仅在 if/else 范围之内。

if 语句的条件不需要用 `()` 括起来，但是条件执行体必须使用 `{}` 括起来。

```go
if err := json.Unmarshal([]byte(blob), &zoo); err != nil {
    log.Fatal(err)
}
```

## 分支（switch, case, fallthrough, default）

switch 的条件从上到下的执行，当匹配成功的 case 执行完成后停止(除非以 fallthrough 语句结束）。

如果匹配都不成功默认执行 default 的内容。

> 长串的 if ... else if ... else if ... else ？

省略掉 switch 语句的条件表达式可以用更清晰的形式代替长串的 if/else 链。例如：

```go
t := time.Now()
switch {
case t.Hour() < 12:
    fmt.Println("Good morning!")
case t.Hour() < 17:
    fmt.Println("Good afternoon.")
default:
    fmt.Println("Good evening.")
}
```

> 类型选择

类型选择比较类型而非值。该表达式为使用保留字 type 而非实际类型的类型断言的形式：

```go
switch a.(type) {
    case int:
        fmt.Printf("a is now an int and equals %d\n", a)
    case bool, string:
        // ...
    default:
        // ...
}
```

## 函数相关（return, defer）

defer 语句会延迟函数的执行直到上层函数返回，一般用于资源的清理工作等。

当程序执行进入了恐慌过程（panic），也会执行 defer，因此处理恐慌的 recover 函数必须在 defer 中使用。

return 语句终止函数的执行，并可选地提供一个或多个返回值。

## go 程（go, select）

go 语句将函数调用作为独立的 goroutine 来启动。程序的执行并不等待 go 语句调用的函数完成。
goroutine 之间的通信使用信道（channel）。

select 语句使得一个 goroutine 在多个通讯操作上等待。select 会阻塞，直到条件分支中的某个可以继续执行。

