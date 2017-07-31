『Go 语法基础』之结构体
=======================

Table of contents
-----------------

*   [定义](#定义)
*   [初始化](#初始化)
*   [构造函数？](#构造函数)
*   [方法](#方法)
*   [组合](#组合)
*   [标注](#标注)

## 定义

定义结构体类型：
```go
type Book struct {
  Name  string
  Price int
}
```

## 初始化

使用字面量初始化：

```go
gopl := Book{"The Go Programming Language", 60}
```
```go
gopl := Book{
  Name:  "The Go Programming Language",
  Price: 60,
}
```
```go
gopl := Book{
  Name: "The Go Programming Language",
}
gopl.Price = 60
```

使用内置函数 new 分配所需要的内存，并返回指针（`new(Book)` 和 `&Book{}` 是等效的）：

```go
gopl := new(Book)
gopl.Name = "The Go Programming Language"
gopl.Price = 60
```
```
gopl := &Book{}
gopl.Name = "The Go Programming Language"
gopl.Price = 60
```

## 构造函数？

Go 中没有构造函数的概念，可以通过一个函数返回一个相应类型的实例：
```go
type Book struct {
  Name  string
  Price int
}

func New(name string, price int) *Book {
  return &Book{Name: name, Price: price}
}
```

## 方法

函数定义时指定一个类型作为方法接收者：

```go
type Book struct {
  Name  string
  Price int
}

func (b *Book) Rise(value int) {
  b.Price += value
}
```

## 组合

示例代码：

```go
type Animal struct {
  Voice string
}

func (a *Animal) Say() {
  fmt.Println(a.Voice)
}

func (a *Animal) Eat() {
  fmt.Println("Eating")
}

type Cat struct {
  *Animal
  Name string
}

func (c *Cat) Eat() {
  fmt.Println("Cat is Eating")
}

func main() {
  tom := &Cat{
    Animal: &Animal{"Meow~~"},
    Name:   "Tom",
  }
  tom.Say()
  tom.Eat()
}
```

**如果一个结构体定义的时候嵌入了另一个结构体并且没有指定名字（称为匿名字段），则外层结构体可以间接调用内层结构体的字段和方法。
如果指定了名字，则不可以调用。**

例如将 Cat 定义的部分改为以下代码，程序编译时会报错：

```go
type Cat struct {
  Animal *Animal
  Name   string
}
```
```
tom.Say undefined (type *Cat has no field or method Say))
```

**如果间接调用的方法使用了内层结构体的属性，则需要在初始化外层结构体的同时将内层结构体视为外层结构体的一个字段并且初始化，
这个字段的名字为内层结构体定义时的名称。**

例如在 tom 初始化时没有初始化内层结构体 Animal，程序运行时会报错：

```go
tom := &Cat{
  Name:   "Tom",
}
```
```
panic: runtime error: invalid memory address or nil pointer dereference
[signal SIGSEGV: segmentation violation code=0x1 addr=0x0 pc=0x47b694]

goroutine 1 [running]:
main.(*Animal).Say(0x0)
exit status 2
```

**如果外层结构体定义了和内层结构体的名称一样的方法，则该方法会被外层结构体重写。**

例如这里的 Eat 方法会输出：
```
Cat is Eating
```

## 标注

标注可通过反射接口获得。

```
type config struct {
  Host string `yaml:"host"`
  Port uint16 `yaml:"port"`
}
```
