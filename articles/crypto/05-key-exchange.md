# 『密码学应用』之密钥交换

<!-- vim-markdown-toc GFM -->

* [基础概述](#基础概述)
* [椭圆曲线](#椭圆曲线)
    * [curve25519](#curve25519)
    * [elliptic](#elliptic)
* [椭圆曲线迪菲 - 赫尔曼密钥交换](#椭圆曲线迪菲---赫尔曼密钥交换)
    * [ecdh](#ecdh)
* [Go 语言示例](#go-语言示例)

<!-- vim-markdown-toc -->

## 基础概述

![密钥交换](images/05-crypto-key-exchange.jpeg)

非对称加密在在计算上相当复杂，性能欠佳、远远不比对称加密；

因此往往会创建临时的随机对称秘钥，通过非对称加密进行密钥交换，然后才通过对称加密来传输大量主体的数据。

*   用于加密用户消息的密钥称为 CEK（Contents Encrypting Key）
*   用于加密密钥的密钥则称为 KEK（Key Encrypting Key）

**Diffe-Hellman 密钥交换**是 1976 年由 Whitfield Diffe 和 Martin Hellman 共同发明的一种算法。

通信双方通过交换一些可以公开的信息就能够生成出共享的秘密数字，而这一秘密数字就可以被用作对称密码的密钥。

![Diffe-Hellman](images/05-crypto-diffe-hellman.jpeg)

*   椭圆曲线迪菲 - 赫尔曼密钥交换（Elliptic Curve Diffie–Hellman key Exchange，ECDH）
    *   迪菲 - 赫尔曼密钥交换（Diffie–Hellman key exchange，DH）
    *   椭圆曲线（Elliptic curve，EC）
        *   Curve25519/X25519：蒙哥马利曲线（Montgomery Curve），专用于密钥协商
        *   P-192, P-224, P-256, P-384, P-521 (see [FIPS 186-3](https://csrc.nist.gov/csrc/media/publications/fips/186/3/archive/2009-06-25/documents/fips_186-3.pdf), section D.2.5)，标准椭圆曲线

## 椭圆曲线

### curve25519

```go
package curve25519
import "golang.org/x/crypto/curve25519"
```

### elliptic

```go
package elliptic
import "crypto/elliptic"

func P224() Curve
func P256() Curve
func P384() Curve
func P521() Curve
```

## 椭圆曲线迪菲 - 赫尔曼密钥交换

### ecdh

```go
package ecdh
import "github.com/aead/ecdh"

// KeyExchange is the interface defining all functions
// necessary for ECDH.
type KeyExchange interface {
    // GenerateKey generates a private/public key pair using entropy from rand.
    // If rand is nil, crypto/rand.Reader will be used.
    GenerateKey(rand io.Reader) (private crypto.PrivateKey, public crypto.PublicKey, err error)

    // Params returns the curve parameters - like the field size.
    Params() CurveParams

    // PublicKey returns the public key corresponding to the given private one.
    PublicKey(private crypto.PrivateKey) (public crypto.PublicKey)

    // Check returns a non-nil error if the peers public key cannot used for the
    // key exchange - for instance the public key isn't a point on the elliptic curve.
    // It's recommended to check peer's public key before computing the secret.
    Check(peersPublic crypto.PublicKey) (err error)

    // ComputeSecret returns the secret value computed from the given private key
    // and the peers public key.
    ComputeSecret(private crypto.PrivateKey, peersPublic crypto.PublicKey) (secret []byte)
}
```

## Go 语言示例

```go
package main

import (
	"bytes"
	"crypto/rand"
	"fmt"

	"github.com/aead/ecdh"
)

func main() {
	c25519 := ecdh.X25519()

	privateAlice, publicAlice, err := c25519.GenerateKey(rand.Reader)
	if err != nil {
		fmt.Printf("Failed to generate Alice's private/public key pair: %s\n", err)
	}

	privateBob, publicBob, err := c25519.GenerateKey(rand.Reader)
	if err != nil {
		fmt.Printf("Failed to generate Bob's private/public key pair: %s\n", err)
	}

	secretAlice := c25519.ComputeSecret(privateAlice, publicBob)
	secretBob := c25519.ComputeSecret(privateBob, publicAlice)

	if !bytes.Equal(secretAlice, secretBob) {
		fmt.Printf("key exchange failed - secret X coordinates not equal\n")
	}
}
```

