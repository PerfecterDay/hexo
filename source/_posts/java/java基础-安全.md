---
title: java基础-安全
date: 2020-04-12  21:32:24
tags: java
category: java基础安全架构
typora-root-url: ..\..
---

Java的安全基础架构主要有以下特点：
+ 实现是互相独立的，java中的安全服务是由 provider 提供的，这些 provider 通过标准接口插入到 java 平台，应用程序可以直接请求这些 provider 提供的安全服务。
+ provider 可跨应用程序进行互操作。具体而言，应用程序未绑定到特定的 provider ，并且 provider 也未绑定到特定的应用程序。
+ 可扩展的， java 中提供了一些基础的安全服务实现，应用程序可以对这些服务进行扩展。且 java 平台也支持安装自定义的 provider

java SPI(Servicec provider Interface)机制：  
是JDK内置的一种服务提供发现机制。SPI是一种动态替换发现的机制， 比如有个接口，想运行时动态的给它添加实现，你只需要添加一个实现。我们经常遇到的就是java.sql.Driver接口，其他不同厂商可以针对同一接口做出不同的实现，mysql和postgresql都有不同的实现提供给用户，而Java的SPI机制可以为某个接口寻找服务实现。

当服务的提供者提供了一种接口的实现之后，需要在classpath下的META-INF/services/目录里创建一个以服务接口命名的文件，这个文件里的内容就是这个接口的具体的实现类。当其他的程序需要这个服务的时候，就可以通过查找这个jar包（一般都是以jar包做依赖）的META-INF/services/中的配置文件，配置文件中有接口的具体实现类名，可以根据这个类名进行加载实例化，就可以使用该服务了。JDK中查找服务实现的工具类是：java.util.ServiceLoader。  

    ServiceLoader<SPITest>  spiTests = ServiceLoader.load(com.company.SPITest.class);
            for (SPITest spiTest : spiTests) {
                System.out.println(spiTest.test());
            }

#### Security Providers
`java.security.Provider`实现了 java 平台下 security provider 的概念。每个 provider 包含一个名字和它实现的安全服务的列表。当有多个 provider 实现了同一个服务时，将会选择最高优先级的 provider 提供的服务。

可以使用引擎类的 `getInstance` 方法从一个底层的 provider 获取安全服务。或者也可以指定 provider name 从指定的 provider 获取安全服务。
![provider选取](/pics/provider-select.jpg)

provider 的安装：
1. 将提供算法实现的 JAR 包放置到 classpath 中即可。
2. 将提供算法实现的 JAR 包放置到扩展路径 `<java-home>/lib/ext`，或者 JRE 的扩展目录：`<jre-home>/lib/ext`

provider 的注册的两种方式：
1. 注册需要在安全配置文件中添加相关信息。java 平台安全相关的配置文件在 `JRE目录/lib/security/java.security`, 可以通过它进行安全相关的配置，也可以调用`java.security `类相关的方法进行设置。还有一个证书相关的文件 `JRE目录/lib/security/cacerts`. MAC 下可以使用`/usr/libexec/java_home`命令查看 java_home。
要注册 provider ，在 java.security 文件中，添加 `security.provider.n=masterClassName`形式的信息。
2. 调用 Security 类的 `addProvider` 或者 `insertProviderAt` 方法动态注册，但是需要合适的权限。

实现一个 provider 的步骤：
1. 编写一个类实现一个 SPI 接口，类必须有一个无参构造函数，部分接口如下：
    + SignatureSpi
    + MessageDigestSpi
    + KeyPairGeneratorSpi
    + SecureRandomSpi
    + AlgorithmParameterGeneratorSpi
    + AlgorithmParametersSpi
    + KeyFactorySpi
    + CertificateFactorySpi
    + KeyStoreSpi
    + CipherSpi
    + KeyAgreementSpi
    + KeyGeneratorSpi
    + MacSpi
    + SecretKeyFactorySpi
    + ExemptionMechanismSpi
2. 给 provider 取一个名字
3. 编写一个 final 类继承自 java.security.Provide ，在该类的构造函数中要调用 super()方法，传入合适的参数。且还要设置合适的属性，将 provider 提供的所有算法服务名字和对应的实现类添加到属性中。
    `put("Signature.SHA256withDSA", "sun.security.provider.DSA")`//该provider中sun.security.provider.DSA类提供 Signature.SHA256withDSA 服务
4. 编译文件并打包到JAR包： `jar cvf <JAR file name> <list of classes, separated by spaces>`
5. 签名 JAR 包。
6. 将 JAR 包安装、注册（见上），可能还需要其他相关的安全设置。

#### 加密算法
+ Java 提供了很多加密算法：
+ Message digest algorithms 消息摘要算法
+ Digital signature algorithms 数字签名算法
+ Symmetric bulk encryption 对称批量加密
+ Symmetric stream encryption 对称流加密
+ Asymmetric encryption 非对称加密
+ Password-based encryption (PBE) 基于密码的加密
+ Elliptic Curve Cryptography (ECC) 椭圆曲线加密
+ Key agreement algorithms 密钥协商算法
+ Key generators 密钥生成
+ Message Authentication Codes (MACs) 消息认证码
+ (Pseudo-)random number generators 伪随机数生成算法
引擎类：一个引擎类提供一个特定安全服务的接口，但是独立于加密算法的实现和 provider 。Java中有如下引擎类：
+ SecureRandom: used to generate random or pseudo-random numbers.
+ MessageDigest: used to calculate the message digest (hash) of specified data.
+ Signature: initialized with keys, these are used to sign data and verify digital signatures.
+ Cipher: initialized with keys, these are used for encrypting/decrypting data. There are various types of algorithms: symmetric bulk encryption (e.g. AES), asymmetric encryption (e.g. RSA), and password-based encryption (e.g. PBE).
+ KeyFactory: used to convert existing opaque cryptographic keys of type Key into key specifications (transparent representations of the underlying key material), and vice versa.
+ SecretKeyFactory: used to convert existing opaque cryptographic keys of type SecretKey into key specifications (transparent representations of the underlying key material), and vice versa. SecretKeyFactorys are specialized KeyFactorys that create secret (symmetric) keys only.
+ KeyPairGenerator: used to generate a new pair of public and private keys suitable for use with a specified algorithm.
+ KeyGenerator: used to generate new secret keys for use with a specified algorithm.
+ KeyAgreement: used by two or more parties to agree upon and establish a specific key to use for a particular cryptographic operation.
+ AlgorithmParameters: used to store the parameters for a particular algorithm, including parameter encoding and decoding.
+ AlgorithmParameterGenerator : used to generate a set of AlgorithmParameters suitable for a specified algorithm.
+ KeyStore: used to create and manage a keystore. A keystore is a database of keys. Private keys in a keystore have a certificate chain associated with them, which authenticates the corresponding public key. A keystore also contains certificates from trusted entities.
+ CertificateFactory: used to create public key certificates and Certificate Revocation Lists (CRLs).
+ CertPathBuilder: used to build certificate chains (also known as certification paths).
+ CertPathValidator: used to validate certificate chains.
+ CertStore: used to retrieve Certificates and CRLs from a repository.

#### Public Key Infrastructure（PKI公钥基础设施）
PKI包含了密钥、证书、公钥加密算法、trusted Certification Authorities (CAs)（信任的认证机构）以及签名的证书。

Java 支持对密钥和证书的持久化存储。 `java.security.KeyStore` 类用来管理密钥/信任证书的存储；`java.security.cert.CertStore`类通常用来存储未信任的证书。

Java 平台包含了标准的PCK11和PCK12，也包含了基于文件的 JKS（Java Key Store），还有 DKS（Domain Key Store，由一系列 keystore组成的逻辑 keystore）. Java中内置了一些权威CA的证书和JKS keystore。存储在`JRE目录/lib/security/cacerts`中，使用 keytool 可以查看.provider SunPKCS11 提供了 PKCS11 的 keystore。

keytool 和 jarsigner 是两个内置的可以用来管理密钥、证书、keystore的工具。  
1. keytool 用来生成和管理 key store:
   1. Create public/private key pairs 生成公私钥
   2. Display, import, and export X.509 v1, v2, and v3 certificates stored as files
   3. Create self-signed certificates 生成自签名证书
   4. Issue certificate (PKCS#10) requests to be sent to CAs
   5. Create certificates based on certificate requests
   6. Import certificate replies (obtained from the CAs sent certificate requests)
   7. Designate public key certificates as trusted 信任公钥证书
   8. Accept a password and store it securely as a secret key
2. jarsigner 用来签名 jar 包文件。