# 『密码学应用』之公钥指纹

<!-- vim-markdown-toc GFM -->

* [基础概述](#基础概述)
* [中间人攻击](#中间人攻击)
* [Go 语言示例](#go-语言示例)

<!-- vim-markdown-toc -->


## 基础概述

公钥指纹（Public Key Fingerprint）通过对公钥应用加密散列函数获得，常用于防止中间人攻击。

## 中间人攻击

有时在登录到一台主机时，发现服务器公钥改变了。
这种情况可能是由于中间人攻击导致，但更多的情况下，是因为主机被重建，生成了新的 SSH 密钥。

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:sYNNR1L6T5cSEG4BndqtCDhJEI0eB9LamBTkuIue3+0.
Please contact your system administrator.
Add correct host key in /home/xxx/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /home/xxx/.ssh/known_hosts:40
  remove with: ssh-keygen -f "/home/xxx/.ssh/known_hosts" -R xxx.com
ECDSA host key for [xxx.com] has changed and you have requested strict checking.
Host key verification failed.
```

为了避免中间人（man-in-the-middle）攻击，在通过 SSH 首次连接某台主机时，SSH 会获取主机公钥并请求保存。
保存后，在以后每次连接时都会验证所连主机公钥与保存的公钥是否匹配，如果不匹配，会给出警告说明密钥变更。

SSH2 支持的密钥类型：RSA, DSA, ECDSA, ED25519。

一个典型 RSA 公共密钥的长度会在 1024 位以上，通过对公钥进行散列生成较短的指纹，可用于验证一个很长的公共密钥。

```
SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8
```
```
MD5:16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48
```

## Go 语言示例

```go
package main

import (
	"crypto/md5"
	"crypto/sha256"
	"encoding/base64"
	"encoding/hex"
	"fmt"
)

func main() {
	// reference: [What is a SSH key fingerprint and how is it generated?](https://superuser.com/a/714195)
	// $ ssh-keygen -f foo
	// Generating public/private rsa key pair.
	// Enter passphrase (empty for no passphrase):
	// Enter same passphrase again:
	// Your identification has been saved in foo.
	// Your public key has been saved in foo.pub.
	// The key fingerprint is:
	// 65:30:38:96:35:56:4f:64:64:e8:e3:a4:7d:59:3e:19 andrew@localhost

	// $ ssh-keygen -E md5 -lf foo.pub
	// 2048 MD5:65:30:38:96:35:56:4f:64:64:e8:e3:a4:7d:59:3e:19 andrew@localhost (RSA)

	// $ ssh-keygen -lf foo.pub
	// 2048 SHA256:I+KtMScfLGEuY8pjFC5AE0YxgdBtKOJK7UXD6trPtnU andrew@localhost (RSA)

	// $ cat foo.pub
	// ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDEbKq5U57fhzQ3SBbs3NVmgY2ouYZfPhc6cXBNEFpRT3T100fnbkYw+EHi76nwsp+uGxk08kh4GG881DrgotptrJj2dJxXpWp/SFdVu5S9fFU6l6dCTC9IBYYCCV8PvXbBZ3oDZyyyJT7/vXSaUdbk3x9MeNlYrgItm2KY6MdHYEg8R994Sspn1sE4Ydey5DfG/WNWVrzFCI0sWI3yj4zuCcUXFz9sEG8fIYikD9rNuohiMenWjkj6oLTwZGVW2q4wRL0051XBkmfnPD/H6gqOML9MbZQ8D6/+az0yF9oD61SkifhBNBRRNaIab/Np7XD61siR8zNMG/vCKjFGICnp andrew@localhost

	// $ cat foo.pub | cut -d' ' -f2 | base64 -d | md5sum
	// $ echo 'AAAAB3NzaC1yc2EAAAADAQABAAABAQDEbKq5U57fhzQ3SBbs3NVmgY2ouYZfPhc6cXBNEFpRT3T100fnbkYw+EHi76nwsp+uGxk08kh4GG881DrgotptrJj2dJxXpWp/SFdVu5S9fFU6l6dCTC9IBYYCCV8PvXbBZ3oDZyyyJT7/vXSaUdbk3x9MeNlYrgItm2KY6MdHYEg8R994Sspn1sE4Ydey5DfG/WNWVrzFCI0sWI3yj4zuCcUXFz9sEG8fIYikD9rNuohiMenWjkj6oLTwZGVW2q4wRL0051XBkmfnPD/H6gqOML9MbZQ8D6/+az0yF9oD61SkifhBNBRRNaIab/Np7XD61siR8zNMG/vCKjFGICnp' | base64 -d | md5sum
	// 6530389635564f6464e8e3a47d593e19  -

	// $ cat foo.pub | cut -d' ' -f2 | base64 -d | sha256sum
	// $ echo 'AAAAB3NzaC1yc2EAAAADAQABAAABAQDEbKq5U57fhzQ3SBbs3NVmgY2ouYZfPhc6cXBNEFpRT3T100fnbkYw+EHi76nwsp+uGxk08kh4GG881DrgotptrJj2dJxXpWp/SFdVu5S9fFU6l6dCTC9IBYYCCV8PvXbBZ3oDZyyyJT7/vXSaUdbk3x9MeNlYrgItm2KY6MdHYEg8R994Sspn1sE4Ydey5DfG/WNWVrzFCI0sWI3yj4zuCcUXFz9sEG8fIYikD9rNuohiMenWjkj6oLTwZGVW2q4wRL0051XBkmfnPD/H6gqOML9MbZQ8D6/+az0yF9oD61SkifhBNBRRNaIab/Np7XD61siR8zNMG/vCKjFGICnp' | base64 -d | sha256sum
	// 23e2ad31271f2c612e63ca63142e4013463181d06d28e24aed45c3eadacfb675  -

	var pub = "AAAAB3NzaC1yc2EAAAADAQABAAABAQDEbKq5U57fhzQ3SBbs3NVmgY2ouYZfPhc6cXBNEFpRT3T100fnbkYw+EHi76nwsp+uGxk08kh4GG881DrgotptrJj2dJxXpWp/SFdVu5S9fFU6l6dCTC9IBYYCCV8PvXbBZ3oDZyyyJT7/vXSaUdbk3x9MeNlYrgItm2KY6MdHYEg8R994Sspn1sE4Ydey5DfG/WNWVrzFCI0sWI3yj4zuCcUXFz9sEG8fIYikD9rNuohiMenWjkj6oLTwZGVW2q4wRL0051XBkmfnPD/H6gqOML9MbZQ8D6/+az0yF9oD61SkifhBNBRRNaIab/Np7XD61siR8zNMG/vCKjFGICnp"
	bytes, err := base64.StdEncoding.DecodeString(pub)
	if err != nil {
		fmt.Println(err)
	}

	m := md5.Sum(bytes)
	fmt.Println(hex.EncodeToString(m[:])) // 6530389635564f6464e8e3a47d593e19
	fmt.Printf("%x\n", m)                 // 6530389635564f6464e8e3a47d593e19

	s := sha256.Sum256(bytes)
	fmt.Println(hex.EncodeToString(s[:]))                   // 23e2ad31271f2c612e63ca63142e4013463181d06d28e24aed45c3eadacfb675
	fmt.Printf("%X\n", s)                                   // 23E2AD31271F2C612E63CA63142E4013463181D06D28E24AED45C3EADACFB675
	fmt.Printf("%x\n", s)                                   // 23e2ad31271f2c612e63ca63142e4013463181d06d28e24aed45c3eadacfb675
	fmt.Printf("%#x\n", s)                                  // 0x23e2ad31271f2c612e63ca63142e4013463181d06d28e24aed45c3eadacfb675
	fmt.Println(base64.StdEncoding.EncodeToString(s[:]))    // I+KtMScfLGEuY8pjFC5AE0YxgdBtKOJK7UXD6trPtnU=
	fmt.Println(base64.URLEncoding.EncodeToString(s[:]))    // I-KtMScfLGEuY8pjFC5AE0YxgdBtKOJK7UXD6trPtnU=
	fmt.Println(base64.RawStdEncoding.EncodeToString(s[:])) // I+KtMScfLGEuY8pjFC5AE0YxgdBtKOJK7UXD6trPtnU
	fmt.Println(base64.RawURLEncoding.EncodeToString(s[:])) // I-KtMScfLGEuY8pjFC5AE0YxgdBtKOJK7UXD6trPtnU

	// reference: [Generating the SHA hash of a string using golang](https://stackoverflow.com/a/10701951)
	// the best practices on conversions to strings:
	// 1. you never store a SHA as a string in a database, but as raw bytes
	// 2. when you want to display a SHA to a user, a common way is Hexadecimal
	// 3. when you want a string representation because it must fit in an URL or in a filename,
	//    the usual solution is Base64, which is more compact
}
```

