『JavaScript 化整为零』之日期和时间
===================================

Table of contents
-----------------

*   [date](#date)
*   [timezone](#timezone)

date
----

以各种姿势使用 Date() 函数获取当前时间，所能得到的结果：

| 使用方式 | 结果 |
|:---------|:-----|
| `new Date`<br/>`new Date()` | 2017-07-15T15:33:43.315Z |
| `+new Date`<br/>`+new Date()`<br/>`new Date().getTime()`<br/>`new Date().valueOf()`<br/>`Date.now()` | 1500132981648 |
| `Date()`<br/>`new Date().toString()` | `'Wed Jul 19 2017 12:05:47 GMT+0800 (CST)'` |
| `new Date().toISOString()` | `'2017-07-19T04:05:47.743Z'` |
| `new Date().toUTCString()` | `'Wed, 19 Jul 2017 04:05:47 GMT'` |
| `new Date().toLocaleString()` | `'7/19/2017, 12:05:47 PM'` |

timezone
--------

`Date.prototype.getTimezoneOffset()`

表示协调世界时（UTC）与本地时区之间的差值，单位为分钟。

如果本地时区晚于协调世界时，则该差值为正值，如果早于协调世界时则为负值。

