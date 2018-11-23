『JavaScript 化整为零』之类型
=============================

Table of contents
-----------------

*   [数据类型](#数据类型)
*   [类型判断](#类型判断)
*   [特殊类型判断](#特殊类型判断)
*   [类型转换](#类型转换)

## 数据类型

在 JavaScript 常见的比较独立的数据类型有：

字符串, 数字, 布尔值, 对象, 数组, 函数，NaN, null, undefined 等。

实际上 NaN 可以归属为数字类型，数组和函数等也可以归属为对象类型。

## 类型判断

*   使用 `typeof`

    使用 typeof 判断的结果为字符串并且只有六种情况：`number`, `string`, `boolean`, `object`, `function`, `undefined`。

    ```js
    typeof null; // object, 需要注意
    typeof []; // object, 需要注意
    ```

*   使用 `instanceof`

    `foo instanceof bar` 判断对象 bar 是否在对象 foo 的 prototype 链上。

    ```js
    1 instanceof Number; // false，需要注意
    new Number() instanceof Number; // true
    new Number() instanceof Object; // true
    ```

*   使用 `constructor`

    查看某实例的构造函数。

    ```js
    'foobar'.constructor === String; // true
    ```

*   根据特点类型具有的特性来判断

    例如 arguments 参数有一个 callee 属性指向当前执行的函数。

    ```js
    // is.js 源码
    // is a given value Arguments?
    is.arguments = function(value) {    // fallback check is for IE
        return toString.call(value) === '[object Arguments]' ||
            (value != null && typeof value === 'object' && 'callee' in value);
    };
    ```

*   使用 `Object.prototype.toString`

    使用这种方式判断数据类型比较准确，**推荐**。

    ```js
    // is.js 源码
    // is a given value Date Object?
    is.date = function(value) {
        return toString.call(value) === '[object Date]';
    };
    ```

## 特殊类型判断

*   判断数组

    可以根据数组的特性来判断（不建议）

    ```js
    function isArray (o){
        return  o &&
                typeof o === 'object' &&
                typeof o.length === 'number' &&
                typeof o.splice === 'function' &&
                !(o.propertyIsEnumerable('length'));
    }
    ```

    使用 `Object.prototype.toString` 判断（建议，这是 isjs 源码）

    ```js
    // is.js 源码
    // is a given value Array?
    is.array = Array.isArray || function(value) {    // check native isArray first
        return toString.call(value) === '[object Array]';
    };
    ```

*   判断 undefined

    变量的值为 undefined 的情况有以下几种：

    *   未初始化的变量
    *   不存在的属性
    *   无返回值的函数调用
    *   多余的形参

    注意不要使用 undefined 直接判断 undefined，因为 undefined 是一个预定义标识符，可以被重新声明。比如：

    ```js
    var undefined = 1;

    var i = 1;

    if (i === undefined) {
        console.log(i, true);   // 输出 1 true
    }
    ```

    应该使用 void 运算符来判断，void 运算符计算一个表达式不返回值，因此结果为 undefined。

    ```js
    // is.js 源码
    // is a given value undefined?
    is.undefined = function(value) {
        return value === void 0;
    };
    ```

*   判断 null

    可以直接使用 null 进行判断。

    ```js
    function isNull(value) {
        return value === null
    }
    ```

*   判断 NaN

    NaN 不等于 NaN，因此可以根据这个特性来判断。

    ```js
    // is.js 源码
    // is a given value NaN?
    is.nan = function(value) {
        return value !== value;
    };
    ```

    也可以使用全局的 isNaN 函数和 Number.isNaN 函数判断。

    **注意** 全局的 isNaN 函数在判断一个值时，如果该值不是数值类型，会自动先转换为数值类型来判断，
    这种行为会导致可能得到的结果并不是你想要的，比如非空字符串在尝试转换为数值时会转换为 NaN。
    可以把 isNaN  的实现看成这样：

    ```js
    isNaN = function(value) {
        Number.isNaN(Number(value));
    }
    ```

    而 Number.isNaN 函数将参数转换成数字，只有在参数是真正的数字类型，且值为 NaN 的时候才会返回 true。

    ```js
    isNaN(NaN); // true
    isNaN('foobar'); // true
    isNaN(undefined); // true
    isNaN({}); // true
    isNaN(new Number(NaN)); // true

    Number.isNaN(NaN); // true
    Number.isNaN('foobar'); // false
    Number.isNaN(undefined); // false
    Number.isNaN({}); // false
    Number.isNaN(new Number(NaN)); // false, 需要注意
    ```

    因此使用 `Number.isNaN` 来判断最靠谱，但是 `Number.isNaN` 是 ES6 新加特性，因此如果在不支持的环境下，
    使用表达式 (x != x) 来判断更加可靠，不建议用 `isNaN`，除非你就是想利用 `isNaN` 这种怪异行为实现特别的功能，
    比如：检测函数的参数是可运算的等。

## 类型转换

*   强制类型转换

    基本类型强制类型转换可以使用其构造函数： `Number()`, `String()`, `Boolean()` 等。

    使用的时候需要格外注意，比如：

    ```js
    Number('1'); // 1
    Number('a'); // NaN
    Number(''); // 0
    ```

*   转字符串

    第一种方法：使用 `toString` 函数转换，如果当前对象重新实现了 `toString` 函数则会调用，
    否则会沿着原型链找，最后调用 `Object.prototype.toString`。

    ```js
    var a = 1;

    console.log(a.toString()); // 1

    Number.prototype.toString = function () {
      return 'number: ' + this;
    }

    console.log(a.toString()); // number: 1
    ```

    第二种方法：使用 `JSON.stringify` 函数序列化也会返回一个字符串实现转换的效果。例如：

    ```js
    var o = { name: 'tom', email: 'tom@xxx.com' };
    var s = JSON.stringify(o);

    console.log(typeof s, s); // string {"name":"tom","email":"tom@xxx.com"}
    ```

    第三种方法：利用 JavaScript 的弱类型转换字符串。使用运算符 + 的时候，如果操作数有字符串
    类型的变量，则会认为是字符串连接操作，因此会自动将所有操作数转换为字符串，因此可以通过
    运算符 + 加一个空字符达到转换字符串的效果。例如：

    ```js
    var n = 100;

    var s = n + '';

    console.log(typeof s, s); // string 100
    ```

*   转数字

    我在另外一篇文章里详细介绍了数字类型的转换：[『JavaScript 化整为零』之数值](https://github.com/modood/modood.github.io/blob/master/articles/js-piece/05-number.md)

*   转布尔值

    使用 `!!` 双重取反即可转换为布尔值，效果和强制类型转换 Boolean() 是一样的，只是一种编程技巧。
    需要注意的是在 JavaScript 中假值只有这几个：false, undefined, null, NaN, 0, ''。

    ```js
    console.log(!!1); // true
    console.log(!!'foobar'); // true
    console.log(!!{}); // true
    console.log(!![]); // true
    console.log(!!undefined); // false
    console.log(!!null); // false
    console.log(!!NaN); // false
    console.log(!!false); // false
    console.log(!!''); // false
    console.log(!!0); // false
    ```

*   自动转换

    需要注意以下几种特殊的值在特定环境下转换结果也不同：

    | 值 | 字符串操作环境 | 数字运算环境 | 逻辑运算环境 | 对象操作环境 |
    |:---|:---------------|:-------------|:-------------|:-------------|
    | undefined | 'undefined' | NaN | false | Error |
    | null | 'null' | 0 | false | Error |
    | NaN | 'NaN' | 不转换 | false | Number |
    | 0 | '0' | 不转换 | false | Number
    | '' | 不转换 | 0 | false | String |
    | true | 'true' | 1 | 不转换 | Boolean |
    | false | 'false' | 0 | 不转换 | Boolean |
    | 对象 | .toString() | valueOf() 或 toString() 或 NaN | true | 不转换 |

