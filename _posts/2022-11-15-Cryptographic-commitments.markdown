---
layout: post
title:  "密码学中的承诺"
date:   2022-11-03 16:09:51 +0800
categories: commitments zkp 
usemathjax: true
---

&emsp;&emsp;承诺(*commitments*)是密码学中的一个有用工具，通常与零知识证明一起使用。原则上讲，承诺被用于确保以正确的方式来选择某些值。但是这个描述并不是很清楚，所以让我们考虑一个具体的案例：“石头剪刀布”游戏。  
&emsp;&emsp;Alice和Bob是“石头剪刀布”游戏的两个玩家。想象一下他们通过互联网玩这个游戏的场景，*如果没有可信赖的第三方，他们怎么能公平地玩这个游戏呢？* ***公平***一词是指没有人能在知晓对方的选择后再决定他/她自己的选择。理想情况下，如果他们能同时表达自己的选择，这就是一场公平的比赛。但是众所周知，同步在网络中是一个非常不切实际的棘手问题。  
&emsp;&emsp;那么，在这种情况下，该如何设计协议呢？对于数据隐藏性，[对称/非对称加密方案](https://medium.com/@kylehuang_96433/symmetric-asymmetric-encryption-5d8d4f6d80f1)可能是一种直观的加密工具；对于[不可否认性](https://medium.com/@kylehuang_96433/digital-signature-c601da3c0c02)，可能需要数字签名。而且使用这两个工具可能在事先也需要通过可信任的第三方的认证，比如密钥分发中心（KDC *key distribution center* ）。  
&emsp;&emsp;在这篇文章中，我们将介绍另一种强大的加密工具来解决这个问题，即密码学中的承诺。承诺协议有两个阶段，提交阶段(*commit*)和开启(*open*)阶段。在提交阶段，证明者提交***com***以隐藏他/她的秘密***secret***以及随机数***r***；经过一些确认后，证明者将***secret***和***r***都公开给验证者；最后，验证者可以验证集合 ***{commit, secret, r}*** 的有效性。  

![](../../../../../images/commitment.png)
<center><small><i>密码学中的承诺</i></small></center> 
&emsp;&emsp;如上所示，承诺框架中有 3 个算法，其中开启算法只是显示 ***secret*** 和 ***r***，不进行任何计算。承诺不依赖于对称/非对称密钥，因此也不存在相应的密钥管理问题，如密钥分发或 KDC。通过承诺，密码学将保证两个属性：  
***隐藏性(*Hiding*)：***  
&emsp;&emsp;在提交阶段，验证者不会从提交的 ***com*** 中获取任何关于 ***secret*** 和 ***r*** 的信息。  
***绑定性(*Binding*)：***  
&emsp;&emsp;在开启阶段，证明者无法开启另一个秘密 ***{secret’, r’} !={secret, r}*** 来使得 ***verify(com, secret’, r’) = True*** 成立。  

![](../../../../../images/commitment2.png)
<center><small><i>应用了密码学中的承诺的“石头剪刀布”</i></small></center> 
&emsp;&emsp;以前面提到的“石头剪刀布”游戏为例，我们会发现，使用承诺会比加密和签名的方案更好。假设***secretA***、***secretB***分别属于***{‘Rock’、‘Paper’、‘Scissor’}***中的一个，它们分别表示Alice和Bob的选择，首先，Alice 和 Bob 在提交阶段交换他们的承诺，如上图所示。通过承诺的***隐藏性***，在提交阶段不会泄露任何值，双方都无法作弊。之后，在他们确认收到对方的承诺后，就进入了开启阶段。两名玩家向对方透露他们的***secret***和 ***r***，以便验证承诺。通过承诺的***绑定性***，证明者在做出承诺后不能改变他/她的想法；也就是说，密码学保证了“公开开启的秘密”与“承诺中的秘密”相同。因此，该承诺保证了两方之间的公平竞争。  
### **实际上的结构**
&emsp;&emsp;到目前为止，也许承诺听起来很神奇，但实际上实现起来并不困难。有几个著名的实现，例如基于哈希的承诺（*hash-based commitment*）、ElGamal 承诺(*ElGamal commitment*)和 Pederson 承诺(*Pederson commitment*)。接下来，我们利用[哈希函数的特性](https://medium.com/@kylehuang_96433/hash-function-a0912682d236)来构造一个简单的承诺。  
***com = Commit(secret, r)***  
```c
r = random(512) // 一个512 bits的随机数来作为熵
com = hash(secret, r)
return com
```
&emsp;&emsp;哈希函数的***单向性***(*one-wayness*)直接保证了承诺的***隐藏性***。  
***T / F = Verify(com, secret, r)***
```c
return (com == hash(secret, r))
```
&emsp;&emsp;哈希函数的***抗碰撞性***(*collision-resistant*)本质上确保了承诺的***绑定性***。  
&emsp;&emsp;至此，密码学上的承诺的基本知识已经介绍完毕。在接下来的零知识证明相关的故事中，还会介绍一些更高级的承诺方案。

本文翻译自：
https://kyle-crypto.medium.com/cryptographic-commitments-55ccc4f297c9