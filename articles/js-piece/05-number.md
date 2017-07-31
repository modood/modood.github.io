『JavaScript 化整为零』之数值
=============================

Table of contents
-----------------

*   [二进制浮点数算术标准（IEEE 754）](#二进制浮点数算术标准ieee-754)
*   [数值转换](#数值转换)
*   [数值取整](#数值取整)

二进制浮点数算术标准（IEEE 754）
-------------------------------

先看一下以下代码会输出什么：

```js
console.log((123).toString());  // 123
console.log(123.0.toString());  // 123
console.log(123..toString());   // 123
console.log(123 .toString());   // 123
```

为什么这种代码也能跑通？这些点是怎么回事？

其实是因为 JavaScript 中的数值类型是基于[二进制浮点数算术标准（IEEE 754）](https://zh.wikipedia.org/wiki/IEEE_754)的，
JavaScript 中的数字是用 IEEE 754 双精度 64 位浮点数 来存储的（没有整型）。

因此上面的几种方式是全等的，最终结果都是：

```js
console.log((123.0).toString());   // 123
```

再看一段代码：

```js
console.log(0.1 + 0.2);                   // 0.30000000000000004
Math.pow(2, 53)+1 === Math.pow(2, 53);    // true
```

可以发现 JavaScript 无法正确处理十进制的小数，而且在处理大整数的时候存在精度丢失现象。
其实这也是因为 IEEE 754 这个规范引起的，这种问题不仅仅存在于 JavaScript 中，比如 C 语言中也是一样。

如果想详细了解原因可以参考这三篇文章：

*   [JavaScript 中小数和大整数的精度丢失](https://lifesinger.wordpress.com/2011/03/07/js-precision/)
*   [浮点数计算为什么不精确](http://brooch.me/2016/11/17/浮点数计算为什么不精确)
*   [js为什么不能正确处理小数运算？](http://m.jb51.net/article/77148.htm)

这种问题可以通过指定精度来避免，例如： `(0.1 * 10 + 0.2 * 10)/10 === 0.3`

数值转换
--------

通常情况下就是字符串转数字。

1.  强制类型转换 Number()

    ```js
    console.log(Number('10'));             // 10
    console.log(Number('3.14'));           // 3.14
    console.log(Number('3.14.111'));       // NaN
    console.log(Number('abc'));            // NaN
    console.log(Number(''));               // 0
    console.log(Number(false));            // 0
    console.log(Number(null));             // 0，需要注意
    console.log(Number(undefined));        // NaN，需要注意
    console.log(Number(NaN));              // NaN
    console.log(Number(new Date()));       // 1500429884017, 需要注意，等价于 new Date().getTime()
    ```

2.  利用 JavaScript 的弱类型转换。

    可以使用三种形式：正数符号 +，减法运算符 -，乘法运算符 *。

    转换失败返回 NaN，这种形式同时适应于转换为整型和浮点型。

    ```js
    console.log(+'10');                   // 10
    console.log(+'3.14');                 // 3.14
    console.log(+'3.14.111');             // NaN
    console.log(+'abc');                  // NaN
    console.log(+'');                     // 0
    console.log(+false);                  // 0
    console.log(+null);                   // 0，需要注意
    console.log(+undefined);              // NaN，需要注意
    console.log(+NaN);                    // NaN
    console.log(+new Date());             // 1500429884017, 需要注意，等价于 new Date().getTime()

    // 换一种使用方式，运行结果和上面一样
    console.log('10' - 0);                // 10
    console.log('3.14' - 0);              // 3.14
    console.log('3.14.111' - 0);          // NaN
    console.log('abc' - 0);               // NaN
    console.log('' - 0);                  // 0
    console.log(false - 0);               // 0
    console.log(null - 0);                // 0，需要注意
    console.log(undefined - 0);           // NaN，需要注意
    console.log(NaN - 0);                 // NaN
    console.log(new Date() - 0);          // 1500429884017, 需要注意，等价于 new Date().getTime()

    // 换一种使用方式，运行结果和上面一样
    console.log('10' * 1);                // 10
    console.log('3.14' * 1);              // 3.14
    console.log('3.14.111' * 1);          // NaN
    console.log('abc' * 1);               // NaN
    console.log('' * 1);                  // 0
    console.log(false * 1);               // 0，需要注意
    console.log(null * 1);                // 0，需要注意
    console.log(undefined * 1);           // NaN
    console.log(NaN * 1);                 // NaN
    console.log(new Date() * 1);          // 1500429884017，需要注意，等价于 new Date().getTime()
    ```

3.  使用 eval 函数（不建议）。

    转换失败会导致运行时异常，这种形式强烈不建议，在这里指出只是表示有这种形式。

    ```js
    console.log(eval('10'));              // 10
    console.log(eval('3.14'));            // 3.14
    console.log(eval('3.14.111'));        // undefined:1, SyntaxError: Unexpected number
    console.log(eval('abc'));             // ReferenceError: abc is not defined
    console.log(eval(''));                // undefined
    ```

4.  使用 parseInt(Number.parseInt), parseFloat(Number.parseFloat) 进行转换。

    转换失败返回 NaN，关于 parseInt 需要注意几点：

    1.  字符串开头的空白符会被忽视掉。例如：

        ```js
        parseInt('      1')； // 1
        ```

    2.  非数字非空白字符开头的字符串会返回 NaN。例如：

        ```js
        parseInt('a1'); // NaN
        ```

    3.  第一个非数字字符之后的所有内容都会被忽视掉（包括小数点，因此会向下取整）。例如：

        ```js
        parseInt('123foobar'); // 123
        parseInt('10.234'); // 10
        ```

数值取整
--------

1.  使用 parseInt(Number.parseInt)

    使用方法见上方。

2.  使用位运算。

    可以使用两种形式：~~ 和 |0

    小数点后面的内容会被忽视（向下取整），转换失败返回 0，因此一个函数参数为整数类型需要设置默认值为 0 时，
    使用这种方式特别有用。

    ```js
    console.log(~~'10');                  // 10
    console.log(~~'3.14');                // 3
    console.log(~~'3.14.111');            // 0
    console.log(~~'abc');                 // 0
    console.log(~~'');                    // 0
    console.log(~~false);                 // 0
    console.log(~~null);                  // 0
    console.log(~~undefined);             // 0
    console.log(~~NaN);                   // 0
    console.log(~~new Date());            // 1488441002，需要注意，这个不是准确的时间戳！

    // 换一种使用方式，运行结果和上面一样
    console.log('10'|0);                  // 10
    console.log('3.14'|0);                // 3
    console.log('3.14.111'|0);            // 0
    console.log('abc'|0);                 // 0
    console.log(''|0);                    // 0
    console.log(false|0);                 // 0
    console.log(null|0);                  // 0
    console.log(undefined|0);             // 0
    console.log(NaN|0);                   // 0
    console.log(new Date()|0);            // 1488441002，需要注意，这个不是准确的时间戳！
    ```

3.  使用 `Math.ceil`, `Math.floor`, `Math.round`,  `Math.trunc`

    它们之间的区别在于：

    ```
    Math.ceil               向上取整
    Math.floor              向下取整
    Math.round              四舍五入
    Math.trunc              丢弃小数，ES6 新增
    ```

