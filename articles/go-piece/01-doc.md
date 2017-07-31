『Go 化整为零』之文档
=====================

Table of contents
-----------------

*   [前言](#前言)
*   [文档注释](#文档注释)
    *   [包文档注释](#包文档注释)
*   [使用示例](#使用示例)
    *   [包使用示例](#包使用示例)
    *   [函数使用示例](#函数使用示例)

前言
----

godoc 可以对 go 源码进行处理，并提取包中的文档内容。

文档内容包括两种：

1.  文档注释：出现在顶级声明之前，且与该声明之间没有空行的注释，作为该条目的说明文档。
2.  使用示例：通常是一段示例代码，用于介绍包或函数的基本用法。

文档注释
--------

注释作为文档的通用约定：

1.  每个可导出的顶级声明都应该有文档注释。
2.  文档注释最好是完整的句子，第一句应当以被声明的东西开头，有利于查找文档。

常用小技巧：

1.  缩进的文本以等宽字体显示，适合用于代码片段等。
2.  每个段落以一个空行隔开。
3.  定义 header。例如下面的 `Printing`。

```
/*
	Package fmt implements formatted I/O with functions analogous
	to C's printf and scanf.  The format 'verbs' are derived from C's but
	are simpler.


	Printing

	The verbs:

	General:
		%v		the value in a default format
				when printing structs, the plus flag (%+v) adds field names
		%#v		a Go-syntax representation of the value
		%T		a Go-syntax representation of the type of the value
		%%		a literal percent sign; consumes no value
*/
```

包文档注释
----------

对于包含多个文件的包， 包注释只需出现在其中的任一文件中即可。

一种常用做法是创建一个 `doc.go` 文件，文件内除了一个包声明外，其它内容都是注释，作为包文档。

例如标准库 fmt 包下的 [`doc.go`](https://github.com/golang/go/blob/master/src/fmt/doc.go) 文件。

使用示例
--------

使用示例通常写在 `*_test.go` 文件里（可以跟单元测试放在不同的文件中以区分开来），并且定义一些名称以 `Example` 开头的函数。

包使用示例
----------

如果是一个包含 main 函数的完整示例，通常写在 `example_test.go` 文件中，main 函数名称以 `Example` 代替。

比如标准库 errors 包中的 [`example_test.go`](https://github.com/golang/go/blob/master/src/errors/example_test.go) 文件的内容为：

```go
// Copyright 2012 The Go Authors. All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.

package errors_test

import (
	"fmt"
	"time"
)

// MyError is an error implementation that includes a time and message.
type MyError struct {
	When time.Time
	What string
}

func (e MyError) Error() string {
	return fmt.Sprintf("%v: %v", e.When, e.What)
}

func oops() error {
	return MyError{
		time.Date(1989, 3, 15, 22, 30, 0, 0, time.UTC),
		"the file system has gone away",
	}
}

func Example() {
	if err := oops(); err != nil {
		fmt.Println(err)
	}
	// Output: 1989-03-15 22:30:00 +0000 UTC: the file system has gone away
}
```

最终输出的文档效果为：

>     package main
>
>     import (
>         "fmt"
>         "time"
>     )
>
>     // MyError is an error implementation that includes a time and message.
>     type MyError struct {
>         When time.Time
>         What string
>     }
>
>     func (e MyError) Error() string {
>         return fmt.Sprintf("%v: %v", e.When, e.What)
>     }
>
>     func oops() error {
>         return MyError{
>             time.Date(1989, 3, 15, 22, 30, 0, 0, time.UTC),
>             "the file system has gone away",
>         }
>     }
>
>     func main() {
>         if err := oops(); err != nil {
>             fmt.Println(err)
>         }
>     }

函数使用示例
------------------

单个函数的使用示例可以写在包内的任意 `*_test.go` 文件中，定义一个名称为 `Example` + `函数名称` 的函数即可。

例如需要介绍标准库 errors 包中 New 函数的使用示例，在 [`errors_test.go`](https://github.com/golang/go/blob/master/src/errors/errors_test.go)
文件中定义了以下函数：

```go
func ExampleNew() {
	err := errors.New("emit macho dwarf: elf header corrupted")
	if err != nil {
		fmt.Print(err)
	}
	// Output: emit macho dwarf: elf header corrupted
}
```

输出的文档效果如下（可以看到，Output 注释也会被自动解析）：

> **func New**
>
> Example

> Code:
>
>     err := errors.New("emit macho dwarf: elf header corrupted")
>     if err != nil {
>         fmt.Print(err)
>     }
>
> Output:
>
>     emit macho dwarf: elf header corrupted

如果要介绍函数有多种用法，可以在定义 `Example` 函数的时候加下划线区分，例如：

```go
// The fmt package's Errorf function lets us use the package's formatting
// features to create descriptive error messages.
func ExampleNew_errorf() {
	const name, id = "bimmler", 17
	err := fmt.Errorf("user %q (id %d) not found", name, id)
	if err != nil {
		fmt.Print(err)
	}
	// Output: user "bimmler" (id 17) not found
}
```

输出的文档效果如下：

> **func New**
>
> Example (Errorf)

> The fmt package's Errorf function lets us use the package's formatting
> features to create descriptive error messages.
>
> Code:
>
>     const name, id = "bimmler", 17
>     err := fmt.Errorf("user %q (id %d) not found", name, id)
>     if err != nil {
>         fmt.Print(err)
>     }
>
> Output:
>
>     user "bimmler" (id 17) not found
