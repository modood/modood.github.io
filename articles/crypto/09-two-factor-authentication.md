# 『密码学应用』之双因素认证

<!-- vim-markdown-toc GFM -->

* [2FA](#2fa)
* [HOTP](#hotp)
* [TOTP](#totp)
    * [K：共享密钥](#k共享密钥)
    * [T：时间窗口](#t时间窗口)
* [谷歌验证器 Google Authenticator - Key Uri Format](#谷歌验证器-google-authenticator---key-uri-format)
* [Go 语言示例](#go-语言示例)
* [参考资料](#参考资料)

<!-- vim-markdown-toc -->

## 2FA

> 双因素认证（Two-factor authentication，2FA）

*   因素一：基于用户所知的信息，例如登录密码
*   因素二：用户具有的设备的身份验证，例如短信验证码、网银动态口令牌、**TOTP 基于时间的一次性密码**等

如果要访问网络，用户必须具备这两个因素。

## HOTP

> [HOTP: An HMAC-Based One-Time Password Algorithm](https://tools.ietf.org/html/rfc4226)

```
HOTP(K,C) = Truncate(HMAC-SHA-1(K,C))

 Symbol  Represents
   -------------------------------------------------------------------
   C       8-byte counter value, the moving factor.  This counter
           MUST be synchronized between the HOTP generator (client)
           and the HOTP validator (server).

   K       shared secret between client and server; each HOTP
           generator has a different and unique secret K.

   T       throttling parameter: the server will refuse connections
           from a user after T unsuccessful authentication attempts.

   s       resynchronization parameter: the server will attempt to
           verify a received authenticator across s consecutive
           counter values.

   Digit   number of digits in an HOTP value; system parameter.
```

**我们可以把操作过程简单描述为三个步骤**

We can describe the operations in 3 distinct steps:

**第一步：生成 HMAC-SHA-1 结果（20 字节长度的字符串）**

Step 1: Generate an HMAC-SHA-1 value

```
HMAC 的哈希函数默认使用 SHA1，基于 K 和 T，通过两阶段运算，算出一个 20-byte 的字符串。

Let HS = HMAC-SHA-1(K,C)  // HS is a 20-byte string
```

**第二步：生成 4 字节长度的字符串（动态裁剪）**

Step 2: Generate a 4-byte string (Dynamic Truncation)

```
Truncate 函数要把 HMAC 生成的 20-byte 字符串，通过一个算法把它转成 4-byte 的字符串。

Let Sbits = DT(HS)                      // DT, defined below, returns a 31-bit string

   DT(String)                           // String = String[0]...String[19]

   Let OffsetBits be the low-order 4 bits of String[19]

   Offset = StToNum(OffsetBits)         // 0 <= OffSet <= 15

   Let P = String[OffSet]...String[OffSet+3]

   Return the Last 31 bits of P
```

**第三步：计算 HOTP 值**

Step 3: Compute an HOTP value

```
将这个 4-byte 的字符串转成数字，然后对 10^6 取模得到一个 6 位数字。

Let Snum  = StToNum(Sbits)   // Convert S to a number in 0...2^{31}-1
Return D = Snum mod 10^Digit // D is a number in the range 0...10^{Digit}-1
```

## TOTP

> [TOTP: Time-Based One-Time Password Algorithm](https://tools.ietf.org/html/rfc6238)

```
TOTP 是基于 HOTP 的：

TOTP = HOTP(K, T)

T = math.floor((Current Unix time - T0) / X)

X   represents the time step in seconds and is a system parameter.
    default value X = 30 seconds
T0  is the Unix time to start counting time steps and is also a system parameter.
    default value is 0, i.e., the Unix epoch

For example, with T0 = 0 and Time Step X = 30
    T = 1 if the current Unix time is 59 seconds
    T = 2 if the current Unix time is 60 seconds
```

### K：共享密钥

1.  验证器客户端和服务器首先协商出一个 secret，然后各自保存
2.  在用户登录的时候，打开验证器，验证器会根据当前时间窗口以及之前协商好的 secret，算出一个数字
3.  用户手动输入这个数字，传输到服务器。
4.  服务器使用同样的函数，算出一个数字，判断与用户输入的数字是否匹配

### T：时间窗口

考虑到网络和人的延迟（从你看到验证器上的数字，到认证服务器接收到你输入的数字之间的延迟），
T 不能特别精确，模糊处理一下，一般每 30s 为一个窗口。

考虑到用户可能在 unix timestamp 29 的时候，输入了这个数字，但不幸网络不好，
到达认证服务器的时候，已经是 unix timestamp 31 了，这时候服务器算出来的数字就会和客户端不同，
为了处理这种情况，RFC 标准里推荐当数字不匹配时，服务器可以多算前一个窗口的数字，但出于安全考虑，最好只多算一个窗口。

## 谷歌验证器 Google Authenticator - Key Uri Format

> [Key Uri Format · google/google-authenticator Wiki](https://github.com/google/google-authenticator/wiki/Key-Uri-Format)

```
otpauth://TYPE/LABEL?PARAMETERS

TYPE
    hotp
    totp

LABEL
    The label is used to identify which account a key is associated with

PARAMETERS
    secret     required    An arbitrary key value encoded in Base32 according to RFC 3548
    issuer     recommended A string value indicating the provider or service this account is associated with
    algorithm  optional    SHA1 (Default), SHA256, SHA512
    digits     optional    Determines how long of a one-time passcode to display to the user. The default is 6.

    counter    required    if type is hotp: The counter parameter is required when provisioning a key
                           for use with HOTP
    period     optional    if type is totp: The period parameter defines a period that a TOTP code
                           will be valid for, in seconds. The default value is 30.
EXAMPLE
    otpauth://totp/GitHub:modood?secret=iamapseudosecret&issuer=GitHub&algorithm=SHA256&digits=8&period=60

```

在线生成：[FreeOTP - Two-Factor Authentication](https://freeotp.github.io/qrcode.html)

## Go 语言示例

```go
// Refer to: https://github.com/rsc/2fa
package main

import (
	"crypto/hmac"
	"crypto/sha1"
	"encoding/base32"
	"encoding/binary"
	"fmt"
	"strings"
	"time"
)

func main() {
	secret := "D6GVU6YIBGE4TISK"

	key, err := base32DecodeKey(secret)
	if err != nil {
		fmt.Println(err)
		return
	}

	code := totp(key, time.Now(), 6)

	fmt.Println(code)
}

func hotp(key []byte, counter uint64, digits int) int {
	h := hmac.New(sha1.New, key)
	binary.Write(h, binary.BigEndian, counter)
	sum := h.Sum(nil)
	v := binary.BigEndian.Uint32(sum[sum[len(sum)-1]&0x0F:]) & 0x7FFFFFFF
	d := uint32(1)
	for i := 0; i < digits && i < 8; i++ {
		d *= 10
	}
	return int(v % d)
}

func totp(key []byte, t time.Time, digits int) int {
	return hotp(key, uint64(t.UnixNano())/30e9, digits)
}

func base32DecodeKey(secret string) ([]byte, error) {
	return base32.StdEncoding.DecodeString(strings.ToUpper(secret))
}
```

## 参考资料

*   [两步验证器是如何工作的](https://mp.weixin.qq.com/s/-NRl2Wx7N-_LK16gQDx0Yg)
