+++
author = "一缕殇流化隐半边冰霜"
categories = ["Protocol", "HTTPS", "Cryptography"]
date = 2018-10-07T05:04:00Z
description = ""
draft = false
image = "https://img.halfrost.com/Blog/ArticleTitleImage/106_0.png"
slug = "cipherkey"
tags = ["Protocol", "HTTPS", "Cryptography"]
title = "秘密的实质——密钥"

+++


## 一、为什么需要密钥？

>密码的本质就是将较长的秘密——消息——变成较短的秘密——密钥。  
>									—————布鲁斯 ● 施耐尔《网络信息安全的真相》


在前面几篇文章中，我们知道对称密码，公钥密码，消息认证码，数字签名，公钥证书，这些密码技术都需要一个密钥。密钥保护了信息的机密性。密钥最重要的是**密钥空间的大下**。密钥的长度决定了密钥空间的大小。密钥空间越大，暴力破解越困难。


## 二、什么是密钥？

密钥仅仅是一个比特序列，但是它所具有的价值和明文等价。密钥的种类主要分为以下几种：

### 1. 对称密码的密钥和公钥密码的密钥

在对称加密中，加密和解密都用同一个密钥，也被称为共享密钥密码。

<p align='center'>
<img src='https://img.halfrost.com/Blog/ArticleImage/106_1.png'>
</p>

在公钥密码中，加密和解密都是不同的密钥。用于加密且能公开的密钥称为公钥。用于解密且不能公开的密钥称为私钥。


<p align='center'>
<img src='https://img.halfrost.com/Blog/ArticleImage/106_2.png'>
</p>

### 2. 消息认证码的密钥和数字签名的密钥

在消息认证码中，发送者和接收者使用共享的密钥来进行认证。消息认证码只有持有合法密钥的人才能计算出来。通过对比消息认证码就可以识别消息是否被篡改或者伪装。


<p align='center'>
<img src='https://img.halfrost.com/Blog/ArticleImage/106_3.png'>
</p>


在数字签名中，签名的生成和验证使用不同的密钥。只有持有私钥的本人才能够生成签名，但由于验证签名使用的是公钥，因此任何人都能够验证签名。


<p align='center'>
<img src='https://img.halfrost.com/Blog/ArticleImage/106_4.png'>
</p>


### 3. 用于保密的密钥和用于认证的密钥

- 对称密钥和公钥密码的密钥都是**用于确保机密性的密钥**。如果不知道用于解密的合法密钥，就可以达到保密明文的效果。

- 消息认证码和数字签名所使用的密钥是**用于认证的密钥**。如果不知道合法的密钥，就无法篡改数据，也无法伪装。


### 4. 会话密钥和主密钥

在 HTTPS 中 TLS 握手中仅限于本次通信的一次性密钥，下次就不能使用了。这种每次通信只能使用一次的密钥叫**会话密钥(session key)**。

由于每次会话都会产生新的会话密钥，即使密钥被窃听了，也只会影响本次会话。如果每次都使用相同的密钥叫**主密钥(master key)**。


### 5. 用于加密内容的密钥和用于加密密钥的密钥

加密的对象是用户直接使用的信息(内容)，这个时候密钥被称为**CEK(Contents Encrypting Key)**。用于加密密钥的密钥被称为**KEK(Key Encrypting Key)**。

<p align='center'>
<img src='https://img.halfrost.com/Blog/ArticleImage/106_5.png'>
</p>

会话密钥被作为 CEK 使用。主密钥被作为 KEK 使用的。



## 三、生成、配送、更新密钥

### 1. 生成密钥

- 用随机数生成密钥。生成密钥的最好办法就是**使用随机数**。
- 用口令生成密钥。通常为了防止字典攻击，需要在口令上附加一串称为**盐**(salt)的随机数。这种方法称为基于口令的密码。

### 2. 配送密钥

- 事先共享密钥
- 使用密钥分配中心
- 使用公钥密码
- Diffie-Hellman 密钥交换

### 3. 更新密钥

这种技术通常被用在共享密钥中。在共享密钥进行通信的过程中，定期(例如，每发送 1000 个字)改变密钥。当然，发送者和接收者改变的步调要一致。

在更新密钥的时候，发送者和接收者使用单向散列函数计算当前密钥的散列值，并将这个散列值用作新的密钥。**用当前密钥的散列值作为下一个密钥**。

密钥更新的好处在于，窃听者窃取了每次会话中的密钥，那么这个密钥之后的内容会被解密，但是窃听者无法解密和更新这个密钥之前的通信内容。因为单向散列函数的单向性。这种防止破译过去的通信内容的机制，称为**后向安全**(backward security)。



## 四、Diffie-Hellman 密钥交换

Diffie-Hellman 密钥交换(Diffie-Hellman key exchange) 是 1976 年由 Whitfield Diffie 和 Martin Hellman 共同发明的一种算法。使用这种算法，通信双方仅通过交换一些可以公开的信息就能生成出共享的对称密码的密钥。

虽然这种算法叫“密钥交换”，但是实际上并没有真正的交换密钥，而是通过计算生成出了一个相同的共享密钥。准确的来说，应该叫 Diffie-Hellman 密钥协商(Diffie-Hellman key agreement)

<p align='center'>
<img src='https://img.halfrost.com/Blog/ArticleImage/106_6.png'>
</p>

1. Alice 向 Bob 发送两个质数 P 和 G   
P 是一个非常大的质数，G 是一个较小的数字，称为生成元。P 和 G 可以公开，P 和 G 的生成也可以由任何一方生成。



2. Alice 生成一个随机数 A    
A 是 1~(P-2) 之间的整数。这个数只有 Alice 知道。



3. Bob 生成一个随机数 B  
B 是 1~(P-2) 之间的整数。这个数只有 Bob 知道。




4. Alice 把 (G^A mod P) 的结果发送给 Bob  
这个数被窃听了也没有关系。




5. Bob 把 (G^B mod P) 的结果发送给 Alice  
这个数被窃听了也没有关系。




6. Alice 用 Bob 发过来的数计算 A 次方并求 mod P  
这个数就是最终的共享密钥。

```c
(G^B mod P)^A mod P = G^(B*A) mod P 
                    = G^(A*B) mod P
```

7. Bob 用 Alice 发过来的数计算 B 次方并求 mod P  
这个数就是最终的共享密钥。

```c
(G^A mod P)^B mod P = G^(A*B) mod P
```

至此，A 和 B 计算出来的密钥是一致的。

窃听者能获取到的信息有：P、G、G^A mod P、G^B mod P。通过这 4 个值想计算出 G^(A*B) mod P 是非常困难的。

如果能知道 A 和 B 任意一个数，就可以破解上面所有步骤，并算出最后的共享密钥。但是窃听者只能获取到 G^A mod P、G^B mod P，这里的 mod P 是关键，如果是知道 G^A 也可以算出 A，但是这里是推算不出 A 和 B 的，因为这是有限域(finite field) 上的**离散对数问题**。

>有限域的离散对数问题的复杂度是支撑 Diffie-Hellman 密钥交换的基础。


虽然 DH 密钥交换可以防止破解，但是无法抵御中间人攻击。

中间人可以在 Alice 和 Bob 之间，分别和双方进行 DH 密钥交换，这样中间人可以截获双方的通信信息。DH 防止中间人攻击的方式和公钥密码的方式一致，可以使用数字签名，证书的方式来应对。

IPSec 中使用的 Diffie-Hellman 密钥交换，就是针对了这个中间人攻击进行了改良和扩展。




## 五、基于口令的密码 PBE

基于口令的密码(Password Based Encryption，PBE)是一种根据口令生成密钥并用该密钥进行加密的方法。加密和解密使用相同密钥。

会使用 PBE 的原因是：

1. 如何保密消息呢？直接存硬盘上会被发现，那就加密吧。生成了 CEK 密钥。
2. 如何安全保存 CEK 密钥呢？用另外一个密钥对 CEK 进行加密吧。生成了 KEK 密钥。
3. 如何安全保存 KEK 密钥呢？这样陷入了死循环了，那么用 PBE 口令来生成密钥 KEK 吧
4. 口令容易遭到字典攻击，先加盐，再和加密以后的 CEK 一次保存在硬盘上，KEK 可以丢弃。
5. 最终只要把口令记在脑袋里即可。


<p align='center'>
<img src='https://img.halfrost.com/Blog/ArticleImage/106_7.png'>
</p>


### 1. PBE 加密

PBE 加密主要包括 3 个步骤：

1. 生成 KEK
2. 生成会话密钥并加密
3. 加密消息

PBE 加密以后输出 3 样：

- 盐
- 用 KEK 加密的会话密钥
- 用会话密钥加密的消息

盐和会话密钥需要保存在安全的地方，消息发送给对方。


### 2. PBE 解密

PBE 加密主要包括 3 个步骤：

1. 重建 KEK
2. 解密会话密钥
3. 解密消息



<p align='center'>
<img src='https://img.halfrost.com/Blog/ArticleImage/106_8.png'>
</p>

对比 PBE 加密过程，可以发现，加密过程中使用了 2 次伪随机数生成器，解密过程一次都没有使用。

**盐主要是用来防止字典攻击的**。


## 六、改良 PBE

由于通过口令生成的密钥 KEK 强度不如由伪随机数生成器生成的会话密钥 CEK，就像一个牢固的保险柜的钥匙放在了一个不保险的地方保管，因此使用基于口令的密码 PBE 时，需要盐和加密后 CEK 通过物理方式进行保护。这里提出一种改良的方式。


在生成 KEK 时，通过多次使用单向散列函数来提高安全性。如果把盐和口令再次输入单向散列函数，得到的值再输入单向散列函数，如此进行 1000 次得到的散列值作为 KEK 来使用，是一个不错的方法。

用户进行 1000 次单向散列值的计算并不会消耗太多时间，但是对于攻击者来说，是很大的困难。像这样将单向散列函数进行多次迭代的方法称为拉伸(stretching)。



------------------------------------------------------

Reference：
  
《图解密码技术》        

> GitHub Repo：[Halfrost-Field](HTTPS://github.com/halfrost/Halfrost-Field)
> 
> Follow: [halfrost · GitHub](HTTPS://github.com/halfrost)
>
> Source: [https://halfrost.com/cipherkey/](https://halfrost.com/cipherkey/)



