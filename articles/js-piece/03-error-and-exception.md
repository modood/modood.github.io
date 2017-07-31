『JavaScript 化整为零』之错误和异常处理
=======================================

Table of contents
-----------------

*   [错误](#错误)
    *   [错误类型](#错误类型)
    *   [错误对象的属性](#错误对象的属性)
    *   [错误传递](#错误传递)
        *   [把错误传递给回调函数](#把错误传递给回调函数)
        *   [以事件的方式触发](#以事件的方式触发)
        *   [以异常的方式抛出](#以异常的方式抛出)
*   [异常](#异常)
    *   [运行时异常](#运行时异常)
    *   [Promise](#Promise)
*   [参考资料](#参考资料)

# 错误

## 错误类型

在 JavaScript 中错误都是普通对象，JavaScript 标准库自带的[错误构造函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Error) 有：

*   Error
*   EvalError
*   InternalError
*   RangeError
*   ReferenceError
*   SyntaxError
*   TypeError
*   URIError

Error 是最通用的错误类型，其它常见的错误类型有 `ReferenceError`, `SyntaxError`, `TypeError`：

```js
console.log(,);                // SyntaxError: Unexpected token ,
var a = fooooooobar;           // ReferenceError: fooooooobar is not defined
console.log(undefined.foobar); // TypeError: Cannot read property 'foobar' of undefined
```

## 错误对象的属性

错误对象包含一些属性可以查看，标准的属性有：

*   name：错误名称
*   message：错误描述信息

而各厂商扩展的属性有：

*   number：错误码
*   fileName：文件名称
*   lineNumber：行号
*   columnNumber：列号
*   stack：函数调用栈

在 Node.js 中，函数调用栈可以使用 `Error.captureStackTrace` 函数进行裁剪。

## 错误传递

当我们创建了一个错误对象后，通常需要将其传递出去，比如函数运行出错需要将错误信息返回给函数调用者。

错误传递的方式主要有三种：

*   把错误传递给回调函数
*   以异常的方式抛出
*   在事件的方式触发

### 把错误传递给回调函数

这种形式通常称为回调函数的 Error-first 模式，Node.js 依赖异步代码保证快速性能，因此这种形式在 Node.js
中最常见，用户传进来一个回调函数（callback），之后当某个异步操作完成后调用这个 callback。两条规则：

>   1.  回调函数的第一个参数保留给一个错误 error 对象，如果有错误发生，错误将通过第一个参数 err 返回。
>   2.  回调函数的第二个参数为成功响应的数据保留，如果没有错误发生，err 将被设置为 null，成功的数据将从第二个参数返回。

举个例子：

```js
var fs = reqire('fs');

fs.readFile('/foo.txt', function(err, data) {
    if (err) {
        // error handling
    }

    // do something
});
```

实际上这只是一种约定俗成的编码规范，建议遵守。

### 以事件的方式触发

这种形式比较适用于可能会产生多个错误或多个结果的复杂操作。

比如，有一个请求一边从数据库取数据一边把数据发送回客户端，而不是等待所有的结果一起到达。
在这个例子里，没有用 callback，而是返回了一个 EventEmitter，每个结果会触发一个 data 事件，
当所有结果发送完毕后会触发 end 事件，出现错误时会触发一个 error 事件。例如：

```js
var http = reqire('http');
var querystring = reqire('querystring');

var postData = querystring.stringify({
  'msg' : 'Hello World!'
});

var options = {
  hostname: '127.0.0.1',
  port: 80,
  path: '/upload',
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
    'Content-Length': Buffer.byteLength(postData)
  }
};

var req = http.request(options, function (res) {
  console.log(`STATUS: ${res.statusCode}`);
  console.log(`HEADERS: ${JSON.stringify(res.headers)}`);
  res.setEncoding('utf8');

  res.on('data', function (chunk) {
    console.log(`BODY: ${chunk}`);
  });

  res.on('end', function () {
    console.log('No more data in response.');
  });
});

req.on('error', function (e) {
  console.log(`problem with request: ${e.message}`);
});

// write data to request body
req.write(postData);
req.end();
```

### 以异常的方式抛出

错误对象使用 throw 关键字抛出后就产生了一个异常，例如：

```js
try {
  throw new Error('something wrong');
} catch (e) {
  console.log(e); // Error: something wrong
}
```

异常和错误是有区别的，错误对象被抛出后变成了异常，而异常的值不一定是错误对象，
字符串、数字或别的任意一种数据类型的值被抛出后也会产生异常，比如：

```js
try {
  throw 'hahahhh';
} catch (e) {
  console.log(e); // hahahhh
}
```

但是 **强烈建议** 所有异常抛出都应该抛出一个错误对象！！

# 异常

异常可以通过 `throw` 抛出， 使用 `try/catch` 语句来捕获。

## 运行时异常

最常见的：

```js
console.log(,);                   // Uncaught SyntaxError: Unexpected token ,
var a = fooooooobar;              // Uncaught ReferenceError: fooooooobar is not defined
console.log(undefined.foobar);    // Uncaught TypeError: Cannot read property 'foobar' of undefined
```

Node.js 里的同步函数通常不会产生运行失败。例外：

```js
var foo = JSON.parse(undefined);  // Uncaught SyntaxError: Unexpected token u in JSON at position 0
var bar = decodeURI('%');         // Uncaught URIError: URI malformed
eval(',');                        // Uncaught SyntaxError: Unexpected token ,
```

使用这几个函数时必须用 `try-catch` 语句尝试捕获异常。

如果有异常而没有被捕获，通常情况下会导致程序崩溃。

Node.js 中异常也可能会被 domains 或者进程级的 uncaughtException 捕捉到。例如：

```js
process.on('uncaughtException', function (err) {
  console.log(err);
  console.log(err.stack)；
});
```

## Promise

在 Promise 中，异常可以通过 `Promise.reject()` 抛出，使用 `.catch()` 进行捕获

```js
new Promise(function (resolve, reject) {
  doSomething(function (err, result) {
    if (err) return reject(err);
    return resolve(result);
  })
})
.then(function (r) {
  console.log(r);
})
.catch(function (err) {
  console.log(err);
});
```

### 参考资料

*   [Error Handling in Node.js](https://www.joyent.com/node-js/production/design/errors)
*   [NodeJS 错误处理最佳实践](https://cnodejs.org/topic/55714dfac4e7fbea6e9a2e5d)

