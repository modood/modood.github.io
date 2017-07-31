『Go 语法基础』之声明
=====================

Table of contents
-----------------

*   [前言](#前言)
*   [包声明（package）](#包声明package)
*   [函数声明（func）](#函数声明func)
*   [导入声明（import）](#导入声明import)
*   [类型声明（type）](#类型声明type)
*   [常量和变量声明（const, var）](#常量和变量声明const-var)

## 前言

go 所有关键字中声明相关的关键字有以下：

```
package import const var type func
```

## 包声明（package）

使用 package 声明一个包，按照惯例：

1.  每个包单独放置在一个目录下。
2.  包导入路径的最后一个目录应该与包名一致（命令源码除外）。

命令源码包名必须为 main，并且必须声明一个 main 函数作为可执行程序的入口。

## 函数声明（func）

使用 func 声明函数：

```go
func add(x int, y int) int {
    return x + y
}
```

两个特殊的函数（main, init）：

1.  main 函数、init 函数不能接受任何参数，并且无返回值。
2.  main 函数一般在 main 包中声明，作为可执行程序的入口，只能声明一次。
3.  init 函数可以在任何包中声明，用于包的初始化操作，可以声明多次（不建议，因为多个 init 函数会按照不确定的顺序执行）。

内置函数有：cap, len, make, new 等

## 导入声明（import）

使用 import 导入一个包：

```go
import "fmt"
```

如果导入多个包，应该使用打包的导入语句：

```go
import (
    "fmt"
    "math"
)
```

1. 导入一个从未使用的包是非法的。
2. 首字母大写的名称是被导出的。

```
import   "lib/math"  // 导入 math 包
import m "lib/math"  // 导入 math 包并且重命名为 m
import . "lib/math"  // math 中声明的可导出标识符无需指定 math 即可直接访问
import _ "lib/math"  // 执行 math 包的初始化函数（init），而不涉及任何可导出标识符的使用
```

## 类型声明（type）

使用 type 声明类型：

```go
type Deleted bool
```

声明多个类型也可以使用打包的类型声明语句：

```go
type (
    Deleted bool
    Book    struct {
        Name  string
        Price int
    }
)
```

1.  预声明的类型：数值（int, int8, ...）、字符串（string）、布尔值（bool）、错误（error）等
2.  变量的静态类型是其声明定义的类型
3.  接口类型的变量有一个独特的动态类型，它是在运行时存储在变量中的值的实际类型。

## 常量和变量声明（const, var）

使用 const 声明常量，使用 var 声明变量。

```go
const Pi = 3.14
var name string
```

声明多个常量或变量可以使用打包的声明语句：

```go
const (
    PI = 3.14
    K  = 0.916
)

var (
    name string
    age  uint
)
```

使用打包的常量声明，如果没有指定初始化表达式，则默认与上一个常量声明一致，例如：

```
const (
	c0 = iota
	c1
	c2
)
```
等价于：

```go
const (
	c0 = iota
	c1 = iota
	c2 = iota
)
```

iota 用于在打包的 const 声明中表示连续的整数常量。有两个特点：

1.  在每个 const 声明中重置为 0。
2.  每个常量声明后 iota 加一。

```go
const (
  c0     = iota        // iota 为 0, c0 值为 0
  c1, c2 = iota, iota  // iota 为 1, c1, c2 的值都为 1
  c3     = iota * iota // iota 为 2, c3 的值为 4
  c4                   // iota 为 3, c4 = iota * iota，c4 的之为 9
  c5     = iota        // iota 为 4, c5 的值为 4
)

const (
  c6 = iota // iota 为 0
  c7        // iota 为 1
)

func main() {
  fmt.Println(c0) // output: 0
  fmt.Println(c1) // output: 1
  fmt.Println(c2) // output: 1
  fmt.Println(c3) // output: 4
  fmt.Println(c4) // output: 9
  fmt.Println(c5) // output: 4
  fmt.Println(c6) // output: 0
  fmt.Println(c7) // output: 1
}
```

常量声明时必须初始化。预定义常量有：true, false, iota, nil。

变量声明时如果未初始化，其值将被设置为指定类型的零值：

```
布尔值为：false
整型为：0
浮点型为：0.0
字符串为：""
指针、函数、接口、切片、映射、信道        nil
```

可以使用一个表达式初始化常量和变量，会自动推导类型。

```go
var message = "Hello World"
```

在函数体中，可以使用短变量声明（:=）省略 var，并且自动推导类型。
使用短变量声明必须至少有一个是新声明的变量。

```go
func main() {
    bytes, err := ioutil.ReadFile("config/default.yml")
    if nil != err {
        log.Fatalln("ReadFile: ", err.Error())
    }
}
```

