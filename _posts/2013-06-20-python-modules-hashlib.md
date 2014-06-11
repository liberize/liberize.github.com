---
layout: post
title: "Python 模块学习之 hashlib"
keywords: ["Python", "hashlib"]
description: "Python 模块学习之 hashlib"
category: "tech"
tags: ["python", "module"]
---
{% include JB/setup %}

### 1、数据加密

1. 公钥加密：又称非对称加密，由一个公钥和一个私钥组成。实际传输数据时，由接收方首先生成一个公钥私钥对，当发送方要发送数据时，首先向接收方索要公钥，使用公钥对数据进行加密，接收方收到加密的数据后使用私钥进行解密。公钥算法的速度很慢，不适合用来加密大量数据。

2. 私钥加密：又称对称加密，数据的加密和解密使用相同的私钥。具有密钥的任意一方都可以使用该密钥解密数据，因此私钥必须保密。实际传输数据时，由一方使用私钥对数据进行加密，另一方接受到数据后使用相同的私钥进行解密。私钥本身可以事先通过公钥加密传输给对方。私钥算法的速度很快，没有数据量限制。

3. 数据签名：使用公钥算法，用来验证数据来源。由发送方对消息进行散列创建信息摘要，然后使用私钥对信息摘要进行加密以创建个人签名，接收方使用公钥对数据进行解密以恢复消息摘要，然后使用相同的哈希算法来散列消息，如果接收方计算出的信息摘要与从发送方接收到的信息摘要相同，则可以确定该消息来自发送方，并且数据未被修改过。

*更详细内容请参考：<http://msdn.microsoft.com/zh-cn/library/92f9ye3s.aspx>*

### 2、hashlib

hashlib是一个提供了多种hash（散列）算法的库。hash的作用是创建信息摘要，因此是单向的。

常用hash算法：

1. MD4：MD4(RFC 1320)是 MIT 的Ronald L. Rivest在 1990 年设计的，MD 是 Message Digest（消息摘要） 的缩写。它适用在32位字长的处理器上用高速软件实现——它是基于 32位操作数的位操作来实现的。
2. MD5：MD5(RFC 1321)是 Rivest 于1991年对MD4的改进版本。它对输入仍以512位分组，其输出是4个32位字的级联，与 MD4 相同。MD5比MD4来得复杂，并且速度较之要慢一点，但更安全，在抗分析和抗差分方面表现更好。
3. SHA-1及其他：SHA1是由NIST NSA设计为同DSA一起使用的，它对长度小于264的输入，产生长度为160bit的散列值，因此抗穷举(brute-force)性更好。SHA-1 设计时基于和MD4相同原理,并且模仿了该算法。

此处主要演示MD5：

```python
import hashlib
md5 = hashlib.md5()
md5.update('This is a test')
print md5.digest()		# 加密结果（二进制）
print md5.hexdigest()	# 加密结果用十六进制字符串表示
```

输出：

```python
'\xce\x11NE\x01\xd2\xf4\xe2\xdc\xea>\x17\xb5F\xf39'
'ce114e4501d2f4e2dcea3e17b546f339'
```

可以看到hexdigest结果是digest结果的十六进制字符串表示。实际中常用的是hexdigest形式。

简单写法：

```python
import hashlib
hashlib.new("md5", "This is a test").hexdigest()
hashlib.md5("This is a test").hexdigest()
```

输出：

```python
'ce114e4501d2f4e2dcea3e17b546f339'
'ce114e4501d2f4e2dcea3e17b546f339'
```

SHA：

```python
hashlib.sha1("This is a test").hexdigest()
hashlib.sha224("This is a test").hexdigest()
hashlib.sha256("This is a test").hexdigest()
hashlib.sha384("This is a test").hexdigest()
hashlib.sha512("This is a test").hexdigest()
```
