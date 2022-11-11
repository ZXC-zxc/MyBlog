---
layout: post
title:  "EdDSA，Ed25519，Ed25519-IETF，Ed25519ph，Ed25519ctx，HashEdDSA，PureEdDSA 都是些什么？"
date:   2022-11-03 16:09:51 +0800
categories: eddsa ed25519 
usemathjax: true
---
## **爱德华兹曲线(Edwards-curve)数字签名EdDSA**
&emsp;&emsp;你已经听说过**EdDSA**了吧，这个闪亮的新签名方案（它真的是新的吗？它从2008开始就一直存在了，快醒醒吧！）  
&emsp;&emsp;自从创建以来，EdDSA 已经发展了很多，也有了一些标准化的过程。它几乎注定要被 NIST 在 [FIPS 186-5](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-5-draft.pdf) 规范中采用！
我们先来说一些定义：
* EdDSA 代表着爱德华兹曲线数字签名算法(*Edwards-curve Digital Signature Algorithm*)。顾名思义，它应该与扭曲爱德华兹曲线(*twisted Edwards curves*)（一种椭圆曲线）一起使用。这个名称可能有一些欺骗性，因为它不是基于数字签名算法 (DSA)，而是基于 [Schnorr 签名](https://www.cryptologie.net/article/193/schnorrs-signature-and-non-interactive-protocols/)！
* Ed25519算法的名字结合了 EdDSA 和 Edwards25519（一条曲线有点等同于 Curve25519，但后来发现，而且性能更高）。   
 
EdDSA、Ed25519 和更安全的 Ed448 都是 [RFC 8032](https://tools.ietf.org/html/rfc8032) 中定义的。  

## **RFC 8032定义的EdDSA**

RFC 8032 相较于[原始论文](https://ed25519.cr.yp.to/ed25519-20110926.pdf)，拓展中了一些新的内容：
* 它在验证期间指定了延展性检查，以防止不怀好意的人从你的现有签名中伪造出额外的有效签名。每当有人谈论 ***Ed25519-IETF*** 时，他们的意思可能是“具有延展性检查的Ed25519算法”。
* 它指定了一些 Ed25519 ***变体***，这也是这篇有了这篇文章的原因。
* 可能还有一些我忘了写的东西。  

&emsp;&emsp;如果要使用论文中定义的原始Ed25519算法签名，您应该执行以下操作：
1. 计算nonce值: ***HASH(nonce_key\|\|message)***
2. 计算承诺(*commitment*)值R : ***R = [nonce]G*** ，其中G是群的基点
3. 计算挑战(*challenge*)值: ***HASH(commitment\|\|public_key\|\|message)***
4. 计算证明(*proof*)值:***S = nonce + challenge × signing_key***
5. 签名就是: ***(R,S)***  

&emsp;&emsp;其中***Hash***是SHA-512函数。

![](../../../../../images/eddsa.jpg)
&emsp;&emsp;在高层次上讲，这类似于 Schnorr 签名，但他还是有如下的区别：  
* ***nonce***是使用固定的 ***nonce_key***（派生自私钥和消息 M）确定性地（而不是概率地）生成的。这是 Ed25519 的一个很酷的功能：它可以防止你重复使用相同的 nonce 两次。
* ***challenge***的计算不仅使用commitment和要签名的消息，还需要使用***签名者的公钥***。你知道为什么吗？  

&emsp;&emsp;重要提示：请注意，此处的消息在传递给签名算法之前不需要进行哈希处理，因为它已经成为了签名算法的一部分来哈希处理。
## **PureEdDSA, ContextEdDSA 与 HashEdDSA**

以下是 RFC 中实际指定的变体：
* ***PureEdDSA***，当配合 Edwards25519 曲线使用时缩写为 ***Ed25519***。
* ***HashEdDSA***，当配合 Edwards25519 曲线使用时缩写为 ***Ed25519ph***（其中 ***ph*** 代表“prehash”）。
* 其他没有名字的东西我们都称之为 ***ContextEdDSA***，当配合 Edwards25519 曲使用时，定义为 ***Ed25519ctx***。  
  
这三个变体可以共享相同的密钥。它们仅在签名和验证算法上有所不同。  
顺便提一下，***Ed448*** 有点不同，所以下面的内容将专注于 EdDSA 和 Edwards25519 曲线。  
&emsp;&emsp;***Ed25519*** (或者 ***pureEd25519***) 我在上面一部分已经描述过了，搞定！  
&emsp;&emsp;***Ed25519ctx***（或者 ***ContextEd25519***）是在 ***pureEd25519***基础上进行了一些额外的修改：  
哈希函数，也就是我在上面的签名协议中描述的 ***HASH(.)*** 被重新定义为   
***HASH(x) = SHA-512(some_encoding(flag, context) || x)***，在其中：
* ***flag*** 设置为 ***0***
* ***context*** 是一个上下文字符串（仅使用于 ***Ed25519ctx***）  

&emsp;&emsp;换句话说，签名算法中的两次哈希的计算现在包含了一些前缀。 （你可以很直观的看到这些变体彼此完全不兼容。）  
&emsp;&emsp;现在，你可以看到 ***ContextEd25519*** 最大区别在于它要求与 ***Ed25519*** 进行一些隔离。
&emsp;&emsp;***Ed25519ph (或者HashEd25519)***, 最后来说说它, 它基于***ContextEd25519***，并做了如下改动：
* ***flag*** 设置为 ***1***
* ***context***现在是可选的，但建议保留
* 参与哈希的不再是消息本身，而换成了消息的哈希（规范说哈希必须是 SHA-512，但我猜它实际上可以是其他算法）  

所以，最大的区别是就是现在需要做两次哈希了。
![](../../../../../images/ed25519-double-hash.jpg)
## **为什么有HashEdDSA，又为什么需要两次哈希?**
&emsp;&emsp;首先，预哈希很糟糕，因为它破坏了Ed25519签名算法的抗碰撞性。在 PureEdDSA 中，我们假设我们采用原始消息而不是哈希。 （虽然这并不能总是正确的，因为函数的调用者可以放入任何想放的东西）。这样的话，哈希函数上的碰撞其实就无关紧要了（虽然它会让你创建一个可以验证两个不同消息的签名），因为你还需要找到nonce的冲突（它是通过私钥计算的随机性密钥）。  
&emsp;&emsp;但是，一旦你对消息进行预哈希，那么只要发现哈希冲突就足以获得一个可以验证两条消息的签名。  
&emsp;&emsp;因此，**如果可能，你就应该使用 PureEdDSA**。并且要正确的使用它（传递正确的信息。）
那么HashEdDSA 是怎么一回事呢？  
最早介绍该算法的  [***EdDSA for more curves***](https://cryptojedi.org/papers/eddsa-20150704.pdf) 论文有这样的说法：  
&emsp;&emsp;*HashEdDSA 的主要动机是因为存储问题（与许多设计好的签名应用并没有关系），当使用PureEdDSA计算 M 的签名时，需要从与 M 一样长的缓冲区中读取 M 两次，因此不支持长消息在小内存时使用的“Init-Update-Final”接口。每个常用的哈希函数 H0 都支持长消息在小内存时使用“Init-Update-Final”接口，因此 H0-EdDSA 签名也支持长消息在小内存时使用“Init-Update-Final”接口。但是请注意，类似的长消息流式验证意味着验证者传递了来自于攻击者伪造的消息。所以协议设计者将长消息拆分成短消息进行签名才是最安全的，而且这种拆分还解决了存储问题。*  
## **我为什么要研究这么晦涩的问题？**
&emsp;&emsp;
因为我正在写[一本书](https://www.manning.com/books/real-world-cryptography?a_aid=Realworldcrypto)，很高兴能解释一下在 Ed25519 身上到底发生了什么。

本文翻译自：
https://www.cryptologie.net/article/497/eddsa-ed25519-ed25519-ietf-ed25519ph-ed25519ctx-hasheddsa-pureeddsa-wtf/
