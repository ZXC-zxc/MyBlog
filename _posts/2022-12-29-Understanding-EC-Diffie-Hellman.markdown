---
layout: post
title:  "理解ECDH(EC-Diffie-Hellman)"
date:   2022-12-29 16:09:51 +0800
categories: ECDH ECC 
usemathjax: true
---

&emsp;&emsp;如果你曾经开发过 Web 服务器，你很可能遇到过Elliptic-curve Diffie–Hellman (ECDH) 或Elliptic-curve Diffie–Hellman Ephemeral (ECDHE) 密码套件。你可能想知道这些套件实现了什么以及它们是如何工作的。我们今天就来讨论一下这个问题。  
&emsp;&emsp;想要了解 ECDH，我们首先必须深入研究标准的Diffie-Hellman 密钥交换协议。早在 1976 年，它就是最早被设计出来的[公钥协议](https://medium.com/@pierrephilip/intro-into-cryptography-192a91d5ba59#26fe)之一，但至今仍在广泛使用。它以 Whitfield Diffie 和 Martin Hellman 的名字命名——他们都是在加密领域中留下自己印记的杰出密码学家，他们也是发明公钥密码学的三人团队的成员。  
&emsp;&emsp;一般来说，当我们进行加密（对称加密）时，你需要通过使用安全通道（可以是任何形式的传输）在两方之间交换加密密钥。但是这种方法有一个固有的缺陷，如果另一个第三方秘密的进入了安全通道，他们就可以截获密钥。进而使用这个密钥来解密双方之间的加密数据，使加密操作无效。
![](../../../../../images/useless-lock.webp)
## **密码学密钥**
&emsp;&emsp;在密码学中，密钥是在加密算法中所使用的一串字符，用于更改原始数据以使其看起来像是随机出现的数据。与物理上的钥匙一样，它锁定（加密）之后的数据只有使用相同密钥的人才能解锁（解密）它。还有一点很重要，需要特别注意，相同的数据，相同的密钥，一定是相同的结果，每一次！  
&emsp;&emsp;密钥通常会符合特定的位大小。在使用[对称加密算法](https://medium.com/@pierrephilip/intro-into-cryptography-192a91d5ba59#523f)时，比如说[AES 256](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)，密钥大小从 128 位到 256 位不等。在 DH 中，建议的密钥大小超过 2000 位，而在 ECDH 中，使用 250 位的密钥就可以实现相同级别的安全性。查看 [Keylength 网站](https://www.keylength.com/en/8/)，可以更好地了解密钥大小与其相关实现之间的关系。  
## **Diffie-Hellman*（迪菲-赫尔曼）*算法**
&emsp;&emsp;Diffie-Hellman算法允许上述两方交换他们的秘密数据，而无需安全通道来传输。事实上，它可以用于任何不安全的通道。然后，交换产生的秘密数据可用于生成[对称加密](https://medium.com/@pierrephilip/intro-into-cryptography-192a91d5ba59#523f)的密钥，并用于后续的加密通信。  
&emsp;&emsp;正如我在[上一篇文章](https://medium.com/@pierrephilip/intro-into-cryptography-192a91d5ba59#71c2)中提到的，我略过了一些基础知识，但在此处我们将进行一下详细的介绍。我们通过使用颜色类比实际上的非常大的数字来说明公钥交换的概念。    
![](../../../../../images/Key exchange.webp)
<center><small><i>密钥交换过程</i></small></center>  
&emsp;&emsp;假设第三方检查这个不安全通信的交换过程。他们只会看到常见的颜色（在本例中为黄色）和第一组混合颜色（浅橙色和浅蓝色）。第三方如果想计算最终颜色，将非常困难。在实际使用中，非常大的数字会代替颜色，这在计算上也需要付出非常昂贵的代价。尝试计算最终颜色是不可行的，即使对于现代超级计算机也是如此。
## **分解说明**
1. 我生成一个素数***p***,一个数字***g***，***g***是***p-1***的[互素数](https://en.wikipedia.org/wiki/Coprime_integers)。然后我告诉你这两个数字。
2. 你选择一个秘密数字***a***，然后你通过***g^a modulo p***计算出***A***，然后再把***A***返回给我。
3. 我会和你做同样的事，我也选择一个秘密数字***b***，然后通过***g^b modulo p***计算出***B***，然后我把***B***传给你。
4. 用我给你的数再做同样的操作 ***B^a modulo p***
5. 我也会对A做同样的操作 ***A^b modulo p***  

我们第4、5步计算出的数字是一样的。  
&emsp;&emsp;我们在第 4 步和第 5 步之后拥有了相同的数字，我们可以使用该数作为对称加密的密钥。上面的数学原理实际上可以归结为[模指数](https://en.wikipedia.org/wiki/Modular_exponentiation)的特性。  
![](../../../../../images/Agreed upon.webp)
<center><small><i>握手成功</i></small></center>  
## **Elliptic-curve Diffie–Hellman*（椭圆曲线迪菲-赫尔曼）*算法**
&emsp;&emsp;现在我们知道常规的 Diffie-Hellman 密钥交换协议是如何工作的了，接下来我们可以开始Elliptic-curve Diffie-Hellman。它们的概念基本上是相同的，整个进程也相似，但是，ECDH使用[代数曲线](https://en.wikipedia.org/wiki/Algebraic_curve)来生成各方使用的密钥。此外，双方还需要事先就椭圆曲线参数达成一致。使用椭圆曲线也比使用普通 DH 时所需的大数运算要快得多。椭圆曲线的[离散对数问题](https://en.wikipedia.org/wiki/Discrete_logarithm)也比[普通离散对数](https://en.wikipedia.org/wiki/Discrete_logarithm)问题更难解决。这意味着我们可以使用比 DH 更小的密钥。  
## **什么是椭圆曲线**
&emsp;&emsp;在[数学](https://en.wikipedia.org/wiki/Mathematics)上，椭圆曲线是亏格（[*genus*](https://en.wikipedia.org/wiki/Genus_of_an_algebraic_curve)）为 1 的光滑([*smooth*](https://en.wikipedia.org/wiki/Nonsingular_variety))投影([*projective*](https://en.wikipedia.org/wiki/Projective_variety))代数曲线，其上有一个指定的点 O。在不同于 2 和 3 的[特征域](https://en.wikipedia.org/wiki/Characteristic_(algebra))上的每条椭圆曲线都可以描述为以下形式的等式：y² = x³ + ax + b 表达的[平面代数曲线](https://en.wikipedia.org/wiki/Plane_algebraic_curve)。  
一条真正的椭圆曲线看起来像是这个样子：  
![](../../../../../images/elliptic curve.webp)
<center><small><i>基本椭圆曲线</i></small></center>  
## **ECDH是怎样工作的**
&emsp;&emsp;首先你需要先确认双方选择使用哪条[椭圆曲线](https://safecurves.cr.yp.to/)。一但选定了曲线，[域参数](https://en.wikipedia.org/wiki/Elliptic-curve_cryptography#Domain_parameters)也就被选定了。有些曲线比其他曲线更安全。我在本段开头添加的椭圆曲线链接显示了哪些曲线可以安全的使用。这与在正常的 Diffie-Hellman 中选择好的素数作用相同。  
&emsp;&emsp;以下是指定的域参数：  
***E*** ——椭圆曲线本身  
***G*** ——***E***上面的基点  
## **分解说明**  
1. 我生成一个随机数***a***作为私钥  
2. 我通过计算***aG***生成公钥***A***
3. 你生成一个随机数***b***作为私钥
4. 你通过计算***bG***生成公钥***B***
5. 我们交换公钥
6. 我通过***aB***计算***K***
7. 你通过***bA***计算***K***  

&emsp;&emsp;这样我们在第 6 步和第 7 步就有了相同的数字，然后我们可以使用该数字作为对称加密的秘密。这种椭圆曲线密钥交换很难破解，因为攻击者需要破解离散对数问题。  
## **为什么使用椭圆曲线**  
&emsp;&emsp;可以看出，ECDH 和 DH 之间并没有太大区别，实现起来也不太复杂。我们应该记住，对于发生的大量计算（例如具有大量流量的网站）的场景，使用椭圆曲线而不是大质数所节省的时间将会累加起来。在小型网站上可能看不出差异，但流量越大，性能的提升就越明显。



