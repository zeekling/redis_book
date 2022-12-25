
# siphash算法

SipHash是一个伪随机函数（PRF）系列，为短消息的速度而优化。SipHash算法是由Jean-Philippe Aumasson和Daniel J. Bernstein
在2012年设计的，可以对hash的DoS攻击的进行防御。

SipHash算法的特点：

- 与以前的加密算法相比，如基于通用散列的MACS，在短消息上更简单、更快速。
- 在性能上与不安全的非加密算法（如fhhash）具有竞争力。
- 密码学上是安全的，尽管领先的密码学家进行了多个密码分析项目，没有发现弱点。
- 经过实战检验，在操作系统（Linux内核、OpenBSD、FreeBSD、FreeRTOS）、语言（Perl、Python、Ruby等）、
  库（OpenSSL libcrypto、Sodium等）和应用程序（Wireguard、Redis等）中成功整合。

作为一个安全的伪随机函数（又称有密钥的哈希函数），SipHash也可以作为一个安全的消息认证码（MAC）。
但SipHash不是通用的无密钥散列函数（如BLAKE3或SHA-3）意义上的散列。因此，SipHash应始终与秘密密钥一起使用，以确保安全。

# 变体

默认的SipHash是SipHash-2-4：它接受一个128位的密钥，进行2轮压缩，4轮最终确定，并返回一个64位的标签。
变体可以使用不同的轮数。例如，我们提出SipHash-4-8作为一个保守的版本。

以下版本没有在论文中描述，但为了满足应用的需要而设计和分析的。

- SipHash-128返回一个128位的标签，而不是64位。有指定轮数的版本是SipHash-2-4-128，SipHash4-8-128，以此类推。
- HalfSipHash使用32位的字，而不是64位，接受64位的密钥，并返回32位或64位的标签。例如，HalfSipHash-2-4-32有2个压缩轮，4个终结轮，并返回一个32位标签。


# 安全性

(Half)SipHash-c-d，c≥2，d≥4，有望为任何具有相同密钥和输出大小的函数提供最大的PRF安全性。

标准的PRF安全目标允许攻击者访问由攻击者自适应选择的信息上的SipHash输出。

安全性受到密钥大小的限制（SipHash为128位），因此搜索2s密钥的攻击者有2s-128的机会找到SipHash密钥。
安全性也受到输出大小的限制。特别是，当SipHash被用作MAC时，如果t是该标签的字节级别大小，盲目尝试2s标签的攻击者将以2s-t的概率成功。



