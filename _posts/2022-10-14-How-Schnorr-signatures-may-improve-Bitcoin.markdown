---
layout: post
title:  "Schnorr签名会怎样改进比特币"
date:   2022-10-14 16:09:51 +0800
categories: bitcoin schnorr
usemathjax: true
---

&emsp;&emsp;当我阅读 Blockstream 的 [MuSig](https://eprint.iacr.org/2018/068.pdf) 这篇论文时，我试图想象这对于像我一样的比特币用户意味着什么。我发现 Schnorr 签名的某些特性非常棒且方便，但另一些特性却有些烦人。在这里，我想与您分享我的想法。首先，我们快速回顾一下：  
# **ECDSA签名算法(*Elliptic Curve Digital Signature Algorithm*)**