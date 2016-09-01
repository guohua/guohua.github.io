---
layout:     post
category:   "Visual C++"
title:      "Cryptlib 库编程经验总结"
subtitle:   ""
date:       2016-08-24
author:     "郭华"
tags:
    - Visual C++
    - Cryptlib
---

# 将一个缓冲区编码
* 首先声明一个字符串来存放编码的结果
```cpp
string str;
```

* 接着声明一个编码器(HexEncoder 可替换为 Base64Encoder 等)对象，并通过 StringSink 类关联两者
```cpp
HexEncoder encoder(new StringSink(str));
```

* 将数据放入编码器中
```cpp
byte buf[1024];
...
encoder.Put(buf, sizeof(buf));
SetDlgItemText(IDC_EDIT_TEST, spk.data());
```

* 调用 MessageEnd 函数结束编码
```cpp
encoder.MessageEnd();
```

# 以 Transformation 结尾的几个类的区别
BufferedTransformation、BlockTransformation、StreamTransformation、HashTransformation 这几个类都是对输入的缓冲区进行某种运算后输出，但是有一些差别：
1. BufferedTransformation: 是对缓冲区进行某种变换，这种变换式可以还原的。
2. BlockTransformation: 基本与 BufferedTransformation 相同，不同之处在于它是以块为单位。
3. StreamTransformation: 与 BufferedTransformation 差不多相同，不同之处在于是以流式方式，如网络套结字。
4. HashTransformation: 是对缓冲区计算哈希值，这种变换是不可还原的。

# Sink 类及其派生类的作用
Sink 类及其派生类，以下简称 Sink 类，它们的作用是在真实的数据和缓冲区变换类之间起一个连接的作用，Sink类由BufferedTransformation派生，但是不能产生任何输入输出，所有对它的操作都是对它所指向的数据的操作。每个Sink类必须与一个正式的数据关联起来。
1. ArraySink: 操作字节数据。
2. StringSink: 操作字符串数据。
3. ArrayXorSink: 操作字节数据，但是会对缓冲区进行异或运算。
4. FileSink: 操作文件。

# SecByteBlock 和 ByteQueue 的区别
1. 两个类都描述了一个字节为单位的缓冲区，所不同的是 SecByteBlock 所分配的缓冲区时连续的，而 ByteQueue 则是用链表实现的。
1. 使用 SecBlock 类可以动态分配指定字节数的内容，并在类析构时自动释放，而且可以作为指定类的指针使用，有些方面类似于 auto_ptr。

# 椭圆曲线加密(ECIES)、签名(ECDSA)算法
## ECIES
构造方法如下：
```cpp
AutoSeededRandomPool rng;// 随机数产生器
ECIES<EC2N>::Decryptor cpriv(rng, ASN1::sect193r1());// 私钥，自己保留
ECIES<ECP>::Encryptor cpub(cpriv);// 公钥，提供给用户
```
**注意：**椭圆曲线加密算法是拥有公钥的人将数据加密后，密文发送给自己，自己来解密，所以不适用于注册码的生成

## ECDSA
以下描述一个自定义的椭圆曲线的构造过程（以GF(p)上的椭圆曲线为例），参数可从以下网址得到 <http://www.cryptomathic.dk/labs/ellipticcurvedemo.html>
```cpp:n
//
Integer modulus("199999999999999999999999980586675243082581144187569");
// a、b为椭圆曲线参数
Integer a("659942,b7261b,249174,c86bd5,e2a65b,45fe07,37d110h");
Integer b("3ece7d,09473d,666000,5baef5,d4e00e,30159d,2df49ah");
// 计算基点G的两个参数x、y
Integer x("25dd61,4c0667,81abc0,fe6c84,fefaa3,858ca6,96d0e8h");
Integer y("4e2477,05aab0,b3497f,d62b5e,78a531,446729,6c3fach");
Integer r("100000000000000000000000000000000000000000000000151");
Integer k(2);
// 私有密钥
Integer d("76572944925670636209790912427415155085360939712345");

// 椭圆曲线
ECP ec(modulus, a, b);
// P为基点G
ECP::Point P(x, y);
// 计算基点G
P = ec.Multiply(k, P);
// Q为公开密钥
ECP::Point Q(ec.Multiply(d, P));
ECIES<ECP>::Decryptor cpriv(ec, P, r, d);
ECIES<ECP>::Encryptor cpub(cpriv);
ECDSA<ECP, SHA>::Signer spriv(cpriv);
ECDSA<ECP, SHA>::Verifier spub(spriv);
ECDH<ECP>::Domain ecdhc(ec, P, r, k);
ECMQV<ECP>::Domain ecmqvc(ec, P, r, k);
```

## 去掉自校验
打开文件 fipstest.cpp，找到函数 DoPowerUpSelfTest，将
```cpp
void DoPowerUpSelfTest(const char *moduleFilename, const byte *expectedModuleMac)
{
     g_powerUpSelfTestStatus = POWER_UP_SELF_TEST_NOT_DONE;
```

改为：
```cpp
void DoPowerUpSelfTest(const char *moduleFilename, const byte *expectedModuleMac)
{
#if 1
     g_powerUpSelfTestStatus = POWER_UP_SELF_TEST_PASSED;

     return;
#endif

     g_powerUpSelfTestStatus = POWER_UP_SELF_TEST_NOT_DONE;
```

## 一些概念
### 椭圆曲线上的点的阶(Order of point)
椭圆曲线的签名的长度是由基点P的阶的长度决定的，和素数的长度无关。所以椭圆曲线可以选取素数较大的类型。

### 椭圆曲线的密钥
私钥是一个大数，而公钥则是一个点
