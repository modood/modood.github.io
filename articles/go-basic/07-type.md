『Go 语法基础』之类型
=====================

Table of contents
-----------------

*   [基本概念](#基本概念)
*   [类型转换](#类型转换)
*   [类型断言](#类型断言)
*   [类型选择](#类型选择)

## 基本概念

> 静态类型

变量的静态类型是通过其声明定义的类型。

> 动态类型

接口类型的变量有一个独特的动态类型，这是在运行时存储在变量中的值的实际类型，在执行过程中可能会有所不同。

> 基本类型

每个类型 T 都有一个 基本类型。若 T 为预声明类型或类型字面，其相应的基本类型为 T 本身。例如：预声明类型 `string`，类型字面 `[]byte`

否则 T 的基本类型为其类型声明中所依据的基本类型。例如：

```
type T1 string
type T2 T1
type T3 []T1
type T4 T3
```

以上 string，[]T1 的基本类型为其本身，分别为 string，[]T1。

的基本类型为 T1 和 T2 的基本类型为 string。T3 和 T4 的基本类型为 []T1 。

## 类型转换

> 表达式 T(v) 将值 v 转换为类型 T 。

```go
var i int = 42
var f float64 = float64(i)
```

> 字符串和字节数组

```go
stra := "the spice must flow"
byts := []byte(stra)
strb := string(byts)
```

> 整型到字符串

使用 strconv.Itoa 函数或 strconv.FormatInt 函数

```
func Itoa(i int) string
func FormatInt(i int64, base int) string
```
```go
var i int = 1
var s1, s2 string

s1 = strconv.Itoa(i)
s2 = strconv.FormatInt(int64(i), 10)
```

> 字符串到整型

使用 strconv.Atoi 函数或 strconv.ParseInt 函数

```
func Atoi(s string) (int, error)
func ParseInt(s string, base int, bitSize int) (i int64, err error)
```
```go
var s string = "1"
var i1 int
var i2 int64
var err error

i1, err = strconv.Atoi(s)
i2, err = strconv.ParseInt(s, 10, 0)
```

## 类型断言

x 为接口类型的表达式，断言 x 不为 nil 且存储于 x 中的值其类型为 T：

```
x.(T)
```

若 T 为非接口类型，x.(T) 断言 T 实现了 x 接口。

```go
var a interface{} = 1
fmt.Println(a.(int))
```

若 T 为接口类型，x.(T) 断言 x 的动态类型实现了接口 T。

```go
var err error = errors.New("something wrong")
fmt.Println(err.(error))
```

类型断言作为表达式时，若断言成立，表达式的值为 x 的值且其类型为 T。若断言不成立，就会出现运行时恐慌（panic）。例如：

```go
var a interface{} = 1
fmt.Printf("type:%T, value: %v\n", a.(int), a.(int)) // type:int, value: 1
```
```go
var a interface{} = 1
fmt.Println(a.(string))

/*
panic: interface conversion: interface {} is int, not string

goroutine 1 [running]:
main.main()
        /home/mwn/Desktop/modood/note/05.zhuanlan/main.go:9 +0xfa
exit status 2
*/
```

类型断言用于赋值和初始化时，其断言结果为 (T, bool) 的值对，这种方式不会出现运行时恐慌。
若断言成立，则返回 (x.(T), true)。若断言不成立，则返回 (T 的零值，false)。例如：

```go
var i interface{} = 1
v, ok := i.(int)
fmt.Printf("%t, type:%T, value:%v\n", ok, v, v) // true, type:int, value:1
```
```go
var i interface{} = "foobar"
v, ok := i.(int)
fmt.Printf("%t, type:%T, value:%v\n", ok, v, v) // false, type:int, value:0
```

## 类型选择

1.  类型选择使用保留字 type 而非实际类型。
2.  类型选择中不允许使用 fallthrough 语句。

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

类型选择监视可包含一个短变量声明，在 case 列表只有一个类型的子句中刚好匹配上时，该变量即拥有此类型。

```go
switch i := x.(type) {
case nil:
  // i 的类型为 x 的类型 interface{}
case int:
  // i 的类型为 int
case float64:
  // i 的类型为 float64
case func(int) float64:
  // i 的类型为 func(int) float64
case bool, string:
  // i 的类型为 x 的类型 interface{}
default:
  // i 的类型为 x 的类型 interface{}
}
```

可重写为：

```go
if x == nil {
  i := x
  // i 的类型为 x 的类型 interface{}
} else if i, ok := x.(int); ok {
  // i 的类型为 int
} else if i, ok := x.(float64); ok {
  // i 的类型为 float64
} else if i, ok := x.(func(int) float64); ok {
  // i 的类型为 func(int) float64
} else {
  _, isBool := x.(bool)
  _, isString := x.(string)
  if isBool || isString {
    i := x
    // i 的类型为 x 的类型 interface{}
  } else {
    i := x
    // i 的类型为 x 的类型 interface{}
  }
}
```

