# 『密码学应用』之非对称加密

<!-- vim-markdown-toc GFM -->

* [基础概述](#基础概述)
    * [公开密钥基础设施（Public Key Infrastructure，PKI）](#公开密钥基础设施publickeyinfrastructurepki)
    * [公钥加密标准（Public Key Cryptography Standards, PKCS）](#公钥加密标准public-key-cryptography-standards-pkcs)
* [密钥管理](#密钥管理)
    * [私钥生成](#私钥生成)
* [密钥格式](#密钥格式)
    * [PEM（Privacy Enhanced Mail）](#pemprivacy-enhanced-mail)
    * [DER（Distinguished Encoding Rules）](#derdistinguished-encoding-rules)
    * [PKCS#1](#pkcs1)
    * [PKCS#8](#pkcs8)
    * [golang 密钥相关数据结构](#golang-密钥相关数据结构)
    * [golang 密钥解析相关函数](#golang-密钥解析相关函数)
* [分块和填充](#分块和填充)
    * [`RSA_PKCS1_PADDING`](#rsa_pkcs1_padding)
    * [`RSA_PKCS1_OAEP_PADDING`](#rsa_pkcs1_oaep_padding)
* [示例：`RSAES-OAEP`/`RSAES-PKCS1-v1_5` 加解密 Go 语言实现](#示例rsaes-oaeprsaes-pkcs1-v1_5-加解密-go-语言实现)
* [参考资料](#参考资料)

<!-- vim-markdown-toc -->

## 基础概述

![非对称加密](images/04-crypto-asymmetric-cryptography.jpeg)

又称“公开密钥加密（Public-key cryptography）”

使用公钥加密明文，只有对应的私钥持有者才可以解密得到明文

常见算法：
*   RSA（Ron Rivest, Adi Shamir, Leonard Adleman）
*   椭圆曲线加密算法（Elliptic Curve Cryptography, ECC）
*   国密算法 SM2：基于椭圆曲线加密算法

本文主要介绍 RSA 的使用。

### 公开密钥基础设施（Public Key Infrastructure，PKI）

公开密钥基础设施（Public Key Infrastructure，PKI）是一种遵循标准的利用公钥加密技术为电子商务的开展提供一套安全基础平台的技术和规范。
支持公开密钥管理并能支持认证、加密、完整性和可追究性服务的基础设施。

*   签证机构（Certification Authority，CA）：负责签发证书
*   注册机构（Registration  Authority，RA）：负责受理用户申请
*   公钥证书（Public Key Certificate，PKC）

用户只要获得其他用户的证书，就可以获得其他用户的公钥。
公钥证书有 CA 的签名，攻击者不能伪造合法的公钥证书。
证书不需要保密，可以在公网上分发，从而实现公钥的安全分配。

**第一代 PKI 标准：**

```
RSA         公钥密码标准（Public-Key Cryptography Standards，PKCS）
IETF        公钥基础设施 X.509（Public Key Instructure X.509，PKIX）
ITU-T       X.509
```

**第二代 PKI 标准：**

```
Microsoft, Verisign, webMethods:

XML 密钥管理规范（XML Key Management Specification，XKMS）
*   XML密钥信息服务规范（XML Key Information  Service Specification，X-KISS）
*   XML密钥注册服务规范（XML Key Registration Service Specification，X-KRSS）
```

**X.509 标准：**

| 编码      | 后缀           | 备注                                         |
|-----------|----------------|----------------------------------------------|
| PKCS#12   | .P12 .PFX      | 二进制方式，私钥/公钥/证书，通常包含保护密码 |
| X.509 DER | .DER .CER .CRT | 二进制形式，证书，不含私钥                   |
| X.509 PAM | .PEM           | Base64 编码文本形式                          |

### 公钥加密标准（Public Key Cryptography Standards, PKCS）

公钥加密标准的设计与发布皆由 RSA 信息安全公司所制定。

> 详情请查阅：<https://zh.wikipedia.org/wiki/公钥加密标准>

| 标准     | 版本 | 名称                                                                    |
|----------|------|-------------------------------------------------------------------------|
| PKCS #1  | 2.1  | RSA 密码编译标准（RSA Cryptography Standard）                           |
| PKCS #2  | -    | 撤销                                                                    |
| PKCS #3  | 1.4  | DH 密钥协议标准（Diffie-Hellman key agreement Standard）                |
| PKCS #4  | -    | 撤销                                                                    |
| PKCS #5  | 2.0  | 密码基植加密标准（Password-based Encryption Standard）                  |
| PKCS #6  | 1.5  | 证书扩展语法标准（Extended-Certificate Syntax Standard）                |
| PKCS #7  | 1.5  | 密码消息语法标准（Cryptographic Message Syntax Standard）               |
| PKCS #8  | 1.2  | 私钥消息表示标准（Private-Key Information Syntax Standard）             |
| PKCS #9  | 2.0  | 选择属性格式（Selected Attribute Types）                                |
| PKCS #10 | 1.7  | 证书申请标准（Certification Request Standard）                          |
| PKCS #11 | 2.20 | 密码设备标准接口（Cryptographic Token Interface (Cryptoki)）            |
| PKCS #12 | 1.0  | 个人消息交换标准（Personal Information Exchange Syntax Standard）       |
| PKCS #13 | -    | 椭圆曲线密码学标准（Elliptic curve cryptography Standard）              |
| PKCS #14 | -    | 拟随机数产生器标准（Pseudo-random Number Generation）                   |
| PKCS #15 | 1.1  | 密码设备消息格式标准（Cryptographic Token Information Format Standard） |

## 密钥管理

### 私钥生成

使用 openssl 生成 rsa 私钥：

```
$ openssl genrsa -out rsa_private_key.pem 2048
```

从私钥提取公钥：

```
$ openssl rsa -in rsa_private_key.pem -pubout -out public_key.pem
```

某些编程语言需要 pkcs8 格式的密钥，将刚才生成的私钥转为 PKCS8 格式：

```
$ openssl pkcs8 -topk8 -inform PEM -in rsa_private_key.pem -outform PEM -nocrypt -out private_key.pem
```

## 密钥格式

### PEM（Privacy Enhanced Mail）

> Base64 encoded DER certificates or keys, with additional header and footer lines

```
-----BEGIN CERTIFICATE-----
BASE64 ENCODED DATA
-----END CERTIFICATE-----
```

### DER（Distinguished Encoding Rules）

> A binary format for keys or certificates. It is a message transfer syntax specified by the ITU in X.690.

### PKCS#1

> The RSA Private/Public key PEM file is specific for RSA keys.

```
-----BEGIN RSA PRIVATE KEY-----
-----END RSA PRIVATE KEY-----
```
```
-----BEGIN RSA PUBLIC KEY-----
-----END RSA PUBLIC KEY-----
```

### PKCS#8

> Because RSA is not used exclusively inside X509 and SSL/TLS,
> a more generic key format is available in the form of PKCS#8,
> that identifies the type of private/public key and contains the relevant data.

> The encrypted PKCS#8 encoded data start and ends with the tags:

```
-----BEGIN ENCRYPTED PRIVATE KEY-----
-----END ENCRYPTED PRIVATE KEY-----
```

> The unencrypted PKCS#8 encoded data starts and ends with the tags:

```
-----BEGIN PRIVATE KEY-----
-----END PRIVATE KEY-----
```

```
-----BEGIN PUBLIC KEY-----
-----END PUBLIC KEY-----
```

### golang 密钥相关数据结构

**PEM**

```go
package pem
import "encoding/pem"

// A Block represents a PEM encoded structure.
//
// The encoded form is:
//    -----BEGIN Type-----
//    Headers
//    base64-encoded Bytes
//    -----END Type-----
// where Headers is a possibly empty sequence of Key: Value lines.
type Block struct {
    Type    string            // The type, taken from the preamble (i.e. "RSA PRIVATE KEY").
    Headers map[string]string // Optional headers.
    Bytes   []byte            // The decoded bytes of the contents. Typically a DER encoded ASN.1 structure.
}
```

**RSA**

```go
package rsa
import "crypto/rsa"

// A PrivateKey represents an RSA key.
type PrivateKey struct {
    PublicKey            // public part.
    D         *big.Int   // private exponent
    Primes    []*big.Int // prime factors of N, has >= 2 elements.

    // Precomputed contains precomputed values that speed up private
    // operations, if available.
    Precomputed PrecomputedValues
}

// A PublicKey represents the public part of an RSA key.
type PublicKey struct {
    N   *big.Int // modulus
    E   int      // public exponent
}
```

**DSA**

```go
package dsa
import "crypto/dsa"

// PrivateKey represents a DSA private key.
type PrivateKey struct {
    PublicKey
    X   *big.Int
}

// PublicKey represents a DSA public key.
type PublicKey struct {
    Parameters
    Y   *big.Int
}
```

**ECDSA**

```go
package ecdsa
import "crypto/ecdsa"

// PrivateKey represents an ECDSA private key.
type PrivateKey struct {
    PublicKey
    D   *big.Int
}

// PublicKey represents an ECDSA public key.
type PublicKey struct {
    elliptic.Curve
    X, Y *big.Int
}
```

### golang 密钥解析相关函数

```go
package pem
import "encoding/pem"

func Encode(out io.Writer, b *Block) error
func Decode(data []byte) (p *Block, rest []byte)
```

```go
package x509
import "crypto/x509"

// pem
func IsEncryptedPEMBlock(b *pem.Block) bool
func DecryptPEMBlock(b *pem.Block, password []byte) ([]byte, error)
func EncryptPEMBlock(rand io.Reader, blockType string, data, password []byte, alg PEMCipher) (*pem.Block, error)

// ecdsa
func MarshalECPrivateKey(key *ecdsa.PrivateKey) ([]byte, error)
func ParseECPrivateKey(der []byte) (*ecdsa.PrivateKey, error)

// rsa/pkcs1
func MarshalPKCS1PrivateKey(key *rsa.PrivateKey) []byte
func MarshalPKCS1PublicKey(key *rsa.PublicKey) []byte
func ParsePKCS1PrivateKey(der []byte) (*rsa.PrivateKey, error)
func ParsePKCS1PublicKey(der []byte) (*rsa.PublicKey, error)

// pkcs8
func MarshalPKCS8PrivateKey(key interface{}) ([]byte, error)
func MarshalPKIXPublicKey(pub interface{}) ([]byte, error)
func ParsePKCS8PrivateKey(der []byte) (key interface{}, err error)
func ParsePKIXPublicKey(derBytes []byte) (pub interface{}, err error)
```

## 分块和填充

### `RSA_PKCS1_PADDING`

```go
package rsa
import "crypto/rsa"

// EncryptPKCS1v15 encrypts the given message with RSA and the padding
// scheme from PKCS#1 v1.5.  The message must be no longer than the
// length of the public modulus minus 11 bytes.
//
// The rand parameter is used as a source of entropy to ensure that
// encrypting the same message twice doesn't result in the same
// ciphertext.
//
// WARNING: use of this function to encrypt plaintexts other than
// session keys is dangerous. Use RSA OAEP in new protocols.
func EncryptPKCS1v15(rand io.Reader, pub *PublicKey, msg []byte) ([]byte, error)

// DecryptPKCS1v15 decrypts a plaintext using RSA and the padding scheme from PKCS#1 v1.5.
// If rand != nil, it uses RSA blinding to avoid timing side-channel attacks.
//
// Note that whether this function returns an error or not discloses secret
// information. If an attacker can cause this function to run repeatedly and
// learn whether each instance returned an error then they can decrypt and
// forge signatures as if they had the private key. See
// DecryptPKCS1v15SessionKey for a way of solving this problem.
func DecryptPKCS1v15(rand io.Reader, priv *PrivateKey, ciphertext []byte) ([]byte, error)
```

### `RSA_PKCS1_OAEP_PADDING`

```go
package rsa
import "crypto/rsa"

// EncryptOAEP encrypts the given message with RSA-OAEP.
//
// OAEP is parameterised by a hash function that is used as a random oracle.
// Encryption and decryption of a given message must use the same hash function
// and sha256.New() is a reasonable choice.
//
// The random parameter is used as a source of entropy to ensure that
// encrypting the same message twice doesn't result in the same ciphertext.
//
// The label parameter may contain arbitrary data that will not be encrypted,
// but which gives important context to the message. For example, if a given
// public key is used to decrypt two types of messages then distinct label
// values could be used to ensure that a ciphertext for one purpose cannot be
// used for another by an attacker. If not required it can be empty.
//
// The message must be no longer than the length of the public modulus minus
// twice the hash length, minus a further 2.
func EncryptOAEP(hash hash.Hash, random io.Reader, pub *PublicKey, msg []byte, label []byte) ([]byte, error)

// DecryptOAEP decrypts ciphertext using RSA-OAEP.

// OAEP is parameterised by a hash function that is used as a random oracle.
// Encryption and decryption of a given message must use the same hash function
// and sha256.New() is a reasonable choice.
//
// The random parameter, if not nil, is used to blind the private-key operation
// and avoid timing side-channel attacks. Blinding is purely internal to this
// function – the random data need not match that used when encrypting.
//
// The label parameter must match the value given when encrypting. See
// EncryptOAEP for details.
func DecryptOAEP(hash hash.Hash, random io.Reader, priv *PrivateKey, ciphertext []byte, label []byte) ([]byte, error)
```

## 示例：`RSAES-OAEP`/`RSAES-PKCS1-v1_5` 加解密 Go 语言实现

```go
package rsautil

import (
	"crypto/rand"
	"crypto/rsa"
	"crypto/sha256"
	"crypto/x509"
	"encoding/pem"
	"errors"
	"io/ioutil"
)

func LoadPublicKey(filepath string) (*rsa.PublicKey, error) {
	block, err := loadPEMBlock(filepath)
	if err != nil {
		return nil, err
	}

	return parsePublicKey(block.Bytes)
}

func LoadPrivateKey(filepath string) (*rsa.PrivateKey, error) {
	block, err := loadPEMBlock(filepath)
	if err != nil {
		return nil, err
	}

	return parsePrivateKey(block.Bytes)
}

func loadPEMBlock(filepath string) (*pem.Block, error) {
	prvPemBytes, err := ioutil.ReadFile(filepath)
	if err != nil {
		return nil, err
	}

	block, _ := pem.Decode(prvPemBytes)
	if block == nil {
		return nil, errors.New("no pem data is found")
	}

	return block, nil
}

func parsePublicKey(der []byte) (*rsa.PublicKey, error) {
	var err error
	var parsedKey interface{}
	if parsedKey, err = x509.ParsePKCS1PublicKey(der); err != nil {
		if parsedKey, err = x509.ParsePKIXPublicKey(der); err != nil {
			return nil, err
		}
	}

	pub, ok := parsedKey.(*rsa.PublicKey)
	if !ok {
		return nil, errors.New("unable to parse RSA public key")
	}

	return pub, nil
}

func parsePrivateKey(der []byte) (*rsa.PrivateKey, error) {
	var err error
	var parsedKey interface{}
	if parsedKey, err = x509.ParsePKCS1PrivateKey(der); err != nil {
		if parsedKey, err = x509.ParsePKCS8PrivateKey(der); err != nil {
			return nil, err
		}
	}

	prv, ok := parsedKey.(*rsa.PrivateKey)
	if !ok {
		return nil, errors.New("unable to parse RSA private key")
	}

	return prv, nil
}

func EncryptPKCS1v15(pub *rsa.PublicKey, origindata []byte) ([]byte, error) {
	return rsa.EncryptPKCS1v15(rand.Reader, pub, origindata)
}

func DecryptPKCS1v15(prv *rsa.PrivateKey, ciphertext []byte) ([]byte, error) {
	return rsa.DecryptPKCS1v15(rand.Reader, prv, ciphertext)
}

func EncryptOAEP(pub *rsa.PublicKey, origindata []byte) ([]byte, error) {
	return rsa.EncryptOAEP(sha256.New(), rand.Reader, pub, origindata, nil)
}

func DecryptOAEP(prv *rsa.PrivateKey, ciphertext []byte) ([]byte, error) {
	return rsa.DecryptOAEP(sha256.New(), rand.Reader, prv, ciphertext, nil)
}
```

## 参考资料

*   [公钥密码学标准 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E5%85%AC%E9%92%A5%E5%AF%86%E7%A0%81%E5%AD%A6%E6%A0%87%E5%87%86)
*   [PKCS #1 v2.2: RSA Cryptography Standard](https://www.emc.com/collateral/white-papers/h11300-pkcs-1v2-2-rsa-cryptography-standard-wp.pdf)
*   [IETF rfc8017](https://tools.ietf.org/html/rfc8017)
*   [ASN.1 key structures in DER and PEM](https://tls.mbed.org/kb/cryptography/asn1-key-structures-in-der-and-pem)
*   [How to load rsa keys in go](https://gist.github.com/jshap70/259a87a7146393aab5819873a193b88c)
