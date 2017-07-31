『Go 语法基础』之映射
=====================

Table of contents
-----------------

*   [映射](#映射)
*   [创建](#创建)
*   [检索](#检索)
*   [遍历](#遍历)
*   [内建函数](#内建函数)

## 映射

映射是一个同种类型元素的无序组

## 创建

动态增长：
```go
m := make(map[string]int)
```

设置映射的初始大小，可以获得一定的性能提升：
```go
m := make(map[string]int, 100)
```

使用字面量创建的同时初始化：
```go
m := map[string]int{
	"tom":  100,
	"jack": 200,
}
```

## 检索

通过双赋值检测某个键存在：
```go
m := make(map[string]int)
m["tom"] = 100

x, ok := m["tom"]    // 100, true
y, ok := m["nobody"] // 0, false
```

## 遍历

for 循环的 range 可以对 map 进行遍历。

```
m := map[string]int{
  "tom":  100,
  "jack": 200,
}
for k, v := range m {
  fmt.Println(k, v)
}
```

## 内建函数

```
len         映射长度（键的数量）
make        映射创建，可以接收一到两个参数，第二个参数用于初始化长度
delete      映射删除某个键
```
