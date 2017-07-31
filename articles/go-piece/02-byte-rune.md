『Go 化整为零』之字符和字节
===========================

Table of contents
-----------------

*   [字符和字节](#字符和字节)
*   [UTF-8 编码](#utf-8-编码)
*   [byte 和 rune](#byte-和-rune)

字符和字节
----------

先了解一下字符集和编码相关的知识：

*   ASCII 字符集

    只包括 127 个常见字符，也就是大小写英文字母、数字和一些符号。

    优点：一个字符占一个字节。

    缺点：能表示的字符数量少，无法表示各国的文字（因此中国指定了 GB2312 字符集）。

*   Unicode 字符集

    把所有语言都统一到一套编码里，并且完全兼容所有 ASCII 字符。

    优点：可以表示所有字符。

    缺点：一个字符占两个字节，包括普通的 ASCII 字符，造成浪费。

*   UTF-8 编码

    UTF-8 是一种传输格式，以一个字节（8 位）为单位传输字符。

    使用 UTF-8 传输字符时，需要对所有字符进行编码，如果是 ASCII 字符（占一个字节），仅需一个字节即可容纳；
    如果是非 ASCII 字符的 Unicode 字符（占两个字节），则需要截断以字节为单位进行传输。

    优点：可以表示所有语言和文字，在存储和传输上更节约空间、提高性能。

    **需要注意区分：** ASCII, Unicode 和 GB2312 等属于字符集，而 UTF-8, UTF-16, Utf-32 等编码是存储和传输上面的概念。

UTF-8 编码
----------

*   编码时

    如果是 ASCII 字符（占一个字节），仅需一个字节即可容纳。

    如果是非 ASCII 字符的 Unicode 字符（占两个字节），则需要截断，因此需要增加额外的标志位来描述截断信息，所以需要三个字节方可容纳。

*   解码时

    根据每个字节标志位的信息确定一个字符所占的字节数，将被截断的多个字节重新还原为一个字符。

*   标志位

    根据需要使用的字节数，编码时的标志位如下：

    *   需要一个字节：`0.......`
    *   需要两个字节：`110.....` `10......`
    *   需要三个字节：`1110....` `10......` `10......`
    *   需要四个字节：`11110...` `10......` `10......` `10......`
    *   ...（依此类推）

*   示例一

    字母“A”的 Unicode 编码十进制表示为：**65**，十六进制表示为：41，
    二进制表示为：1000001，占用一个字节。
    如果使用 UTF-8 编码不需要截断，编码后为：<span style="color:red">0</span>1000001。

*   示例二

    汉字“啊”的 Unicode 编码十进制表示为：**21834**，十六进制表示为：554A，
    二进制表示为：01010101 01001010，占用两个字节。

    如果使用 UTF-8 编码则需要截断，并且需要额外的一个字节作为标志位来描述截断信息，所以一共需要三个字节。

    编码后为：<span style="color:red">1110</span>0101 <span style="color:red">10</span>010101 <span style="color:red">10</span>001010，分别转换为十进制为：**229 149 138**。

byte 和 rune
-------------

在 go 语言中，内置类型 byte 和 rune 语义上分别表示字节和字符。

```go
// byte is an alias for uint8 and is equivalent to uint8 in all ways. It is
// used, by convention, to distinguish byte values from 8-bit unsigned
// integer values.
type byte = uint8

// rune is an alias for int32 and is equivalent to int32 in all ways. It is
// used, by convention, to distinguish character values from integer values.
type rune = int32
```

分别使用 `[]byte` 和 `[]rune` 来保存字符串 "A啊" 即可看出两者的区别：

```go
a := []byte("A啊")
b := []rune("A啊")

fmt.Println(len(a), a)                        // output: 4 [65 229 149 138]
fmt.Println(len(b), b)                        // output: 2 [65 21834]

fmt.Println(string([]byte{a[1]}))             // output:
fmt.Println(string([]byte{a[1], a[2], a[3]})) // output: 啊
fmt.Println(string([]rune{b[1]}))             // output: 啊
```

因为 `byte` 为 `uint8` 的别名，占 8 位内存。`rune` 为 `int32` 的别名，占 32 位内存。上述代码中

`len(a)` 为 4，因此变量 a 一共占用 8 * 4 = 32 位内存。

`len(b)` 为 2，因此变量 b 一共占用 32 * 2 = 64 位内存。

因此可以得出结论：使用 `[]byte` 更节省内存。

但是使用 `[]byte` 无法简单地通过下标直接访问到汉字等非 ASCII 字符，而使用 `[]rune` 则可以。

可以使用 Unicode 编码十进制值或单引号表示的字符字面量对 `[]byte` 或 `[]rune` 的单个元素赋值。例如：

```go
c := []byte("AAAA")
d := []rune("啊")

fmt.Println(string(c), c) // output: AAAA [65 65 65 65]
fmt.Println(string(d), d) // output: 啊 [21834]

c[0] = 97
c[1] = 'a'
c[2] = c[2] + 32
c[3] = 'A' + 32
d[0] = '哈'

fmt.Println(string(c), c) // output: aaaa [97 97 97 97]
fmt.Println(string(d), d) // output: 哈 [21704]
```

使用单引号表示字符时支持偏移指定整数，例如字符 'a' 的 Unicode 编码值为 97，'A' 的 Unicode 编码值为 65，相差 32。
因此使用 `'A' + 32` 的结果即为 `'a'`

