---
layout: post
title: Https 详解 [网络]
category: Android
tags:
  - NetWork
abbrlink: 882ae1dd
keywords:
  - 网络
  - Https
  - Http
  - TLS/SSL
  - 加密
  - 网络通信
  - 网络安全
date: 2017-12-13 22:56:00
location: 杭州尚妆
photos: http://olx4t2q6z.bkt.clouddn.com/18-2-1/45157165.jpg
---

超文本传输安全协议（HTTPS，常称为HTTP over TLS/SSL）是一种通过计算机网络进行安全通信的传输协议。HTTPS经由HTTP进行通信，但利用SSL/TLS来加密数据包。HTTPS开发的主要目的，是提供对网站服务器的身份认证，保护交换数据的隐私与完整性。


<!--more-->


本文主要介绍  ：

- Https 如何保证数据传输的安全 
- CA 的存在及其安全性
- 证书验证流程
- Https 握手流程
- Android 下进行 Https 访问


## 前言

TCP (Transmission Control Protoco) 传输层控制协议

TLS (Transport Layer Security) 传输层安全协定

SSL (Secure Socket Layer) 安全套接层

HTTP(Hypertext Transfer Protocol) 基于 TCP 协议，无连接，每次连接只处理一个请求，结束后断开连接；无状态，无法保持用户状态，使用 cookie 和 session 解决。

HTTPS(HTTP over TLS/SSL) 安全的 http 协议，HTTP 协议和TCP 协议之间增加了 TLS/SSL 保证数据的安全传输。

### 关于 TLS/SSL

```

  1994年，NetScape公司设计了SSL协议（Secure Sockets Layer）的1.0版，但是未发布。

  1995年，NetScape公司发布SSL 2.0版，很快发现有严重漏洞。

  1996年，SSL 3.0版问世，得到大规模应用。

  1999年，互联网标准化组织ISOC接替NetScape公司，发布了SSL的升级版TLS 1.0版。

  2006年和2008年，TLS进行了两次升级，分别为TLS 1.1版和TLS 1.2版。最新的变动是2011年TLS 
1.2的修订版。

  TLS 1.0通常被标示为SSL 3.1，TLS 1.1为SSL 3.2，TLS 1.2为SSL 3.3。

  目前，应用最广泛的是TLS 1.0，接下来是SSL 3.0。但是，主流浏览器都已经实现了TLS 1.2的支持。

```

## Https 安全性

HTTP 协议的不安全性体现在 3 个方面：

风险 | 描述 | https解决方案
:-- |:--|:--
窃听风险|攻击者可以获知消息内容|消息加密
篡改风险|攻击者可以篡改消息内容|消息摘要
冒充风险|攻击者可以冒充其他人参与通信|CA 身份认证
CA 不可信|信任的 CA 乱签发证书|证书锁  


### 窃听/嗅探

指的是路由上的攻击者，可以偷窥到传输的消息内容。

解决方案：

1. 使用对称加密算法加密通信内容，窃听者获取到消息也无法识别，存在问题 -> 密钥传递的安全性，在网络上面的通信双方都是陌生人，无法识别身份，密钥要通过网络传输时很有可能被窃取。❌

2. 使用非对称加密算法加密通信内容，发布的公钥用来加密，私钥用来解密，即使公钥被窃取，依然无法解密消息内容。存在问题 -> 速度慢，消耗大；公钥被公开，如果回发私钥加密的信息，任何持有公钥的人都可以解密。❌

3. 最终，消息内容仍旧使用对称加密算法来加密，但是前期对称加密的密钥交换使用非对称加密来进行，客户端使用服务端公钥加密对称加密的密钥，这样就只有拥有私钥的服务端可以获取到加密内容，由于对称加密密钥长度有限，加密的时间可以忽略不计。✅

以上，可以防止嗅探的问题，路由上面的攻击者即使获取到消息也无法识别消息的内容。

### 消息篡改

消息加密以后攻击者无法获取消息内容的含义，但是可以篡改消息内容，篡改之后接收方也无法感知。

解决方案：

- 采用 [消息摘要（见文末注脚）](#消息摘要) 算法可以验证数据的完整性，我们将发送的消息进行摘要，连同消息一起发送给接收方，接收方拿到消息之后对消息做同样的摘要处理，对比摘要结果，即可知道消息有没有被篡改。

以上，可以解决消息完整性和真实性的问题。

### 中间人攻击

**中间人攻击**（Man-in-the-middle Attack，MITM）指的是攻击者在链路上伪装自己，与通讯双方分别建立联系，并交换其所收到的数据，使通讯的两端认为他们正在通过一个私密的连接与对方直接对话，但事实上整个会话都被攻击者完全控制。

作为 **A** 和 **B** 通信路由上的攻击者 **M**，作为中间人伪造自己的身份。A 向 B 请求用于通信的 `PK_A`，但是被中间人 M 截获，他伪造生成了假的公钥 `PK_M`，发送给了 A，同时向 B 请求并获取了 B 的公开密钥 `PK_B`，原来安全的通信过程 A(使用 `PK_B` 加密) -> 安全 -> B(使用 `SK_B` 解密)，现在变成了 A(使用 `PK_M` 加密) -> M(使用 `SK_M` 解密获取明文内容，再用 `PK_B` 加密，可能篡改数据) -> B(使用 `SK_B` 解密)。

出现问题的原因在于，密钥在交换的初期是不安全的，网络上的通信双方，无法确定对方的身份，即无法获悉当前的公钥是不是自己想要的公钥。

解决方案：

- 引入第三方公正作[信用背书](#信用背书)，第三方公正具有权威性，他和通信双方没有关系，接收方无条件信任公证机构，也就会信任他签名的信息。这种机构被称为 CA（Certificate Authority）机构，即数字证书认证机构。CA 机构与浏览器和操作系统厂商合作，将公钥内置在浏览器和操作系统中，也就是不走网络传输了，这样一定程度上保证了公钥不会被窃取篡改。

- 服务端 B 将自己的消息（消息内容大致包括电子签证机关的信息、公钥用户信息、公钥、权威机构的签字和有效期等）进行摘要之后，发给 CA 机构 S 签发证书，S 使用自己的私钥对消息摘要加密，形成证书，B 将证书和消息内容发送给 A，A 收到后，发现是 S 的证书，同时对 S 是信任的，则会使用 S 的公钥对证书进行解密（这里涉及证书链的验证，其实要更加复杂），获取到消息摘要，同时对收到的消息进行摘要，对比，如果一致则说明内容没有被篡改，是可信的，因为生成加密数据的私钥只有 CA 机构才有，这一过程称为验证数字签名。

以上，可以解决中间人攻击的文图，另外涉及到非对称加密的两种应用场景，详细介绍见文末[非对称加密应用场景](#非对称加密应用场景)。


### CA 错误签发

受信任的 CA（证书颁发机构）有好几百个，他们成为整个网站身份认证过程中一个较大的攻击面。实际上，目前由于 CA 失误导致错误签发证书，以及个别 CA 出于某些目的（如监控加密流量）故意向第三方随意签发证书这两种情况时有发生。现有的证书信任链机制最大的问题是，任何一家受信任的 CA 都可以签发任意网站的站点证书，这些证书在客户端看来，都是合法的，是可以通过验证的。

解决方案：

- 证书锁（Certificate Pinning），证书锁是为了防范由「伪造或不正当手段获得网站证书」造成的中间人攻击。
 
- 证书锁类似于 `HPKP` 技术（下面有简单介绍），给予我们主动选择信任 CA 的权利。它的工作原理就是使用预先设置的证书指纹和服务器传过来的证书链中的证书指纹进行匹配，只要有任何一对指纹匹配成功，则认为是一次合法的连接，否则禁止本次链接。

- 也就是说，使用证书锁之后，不是所有被系统信任的 CA 都可以通过验证，只有我保存了指纹的一些 CA 签发的证书才可以，比如有 100 个 CA 机构，但我就信任其中一个，我可以保证这个 CA 不会乱签发证书，那我就只保存这个 CA 的指纹，即使攻击者从其他的 99 个 CA 签发证书，对我进行攻击，也无法完成连接。

- 证书锁定增加了安全性，但限制了你的服务器团队升级TLS证书的能力。

以上，可以解决 CA 机构签发证书权威性不足的问题，相关解决方案详见文末[签发证书权威性问题](#签发证书权威性问题)

🔥  **综上，对称加密通信 + 非对称加密交换密钥 + CA 认证身份 + 证书锁锁定证书指纹 共同保证了 https 的安全性。**

## CA 安全性

作为全网 https 连接的权威公正，CA 的安全性至关重要。

### 安全保存 CA及证书验证

从根 CA 开始到直接给客户发放证书的各层次 CA，都有其自身的密钥对。CA 中心的密钥对一般由硬件加密服务器在机器内直接产生，并存储于加密硬件内，或以一定的加密形式存放于密钥数据库内。加密备份于 IC 卡或其他存储介质中，并以高等级的物理安全措施保护起来。密钥的销毁要以安全的密钥冲写标准，彻底清除原有的密钥痕迹。需要强调的是，根 CA 密钥的安全性至关重要，它的泄露意味着整个公钥信任体系的崩溃，所以 CA 的密钥保护必须按照最高安全级的保护方式来进行设置和管理。CA的私钥是自己靠上述方法保管的，不对外公开。

CA 的公钥是厂商跟浏览器和操作系统合作，把公钥默认装到浏览器或者操作系统环境里。比如 firefox 就自己维护了一个可信任的 CA 列表，而 chrome 和 IE 使用的是操作系统的 CA 列表。

### 证书链

现在大的 CA 都会有证书链，证书链的好处一是安全，保持根 CA 的私钥离线使用。第二个好处是方便部署和撤销，即如果证书出现问题，只需要撤销相应级别的证书，根证书依然安全。
根 CA 证书都是自签名，即用自己的公钥和私钥完成了签名的制作和验证。而证书链上的证书签名都是使用上一级证书的密钥对完成签名和验证的。


### 证书验证

证书是否是信任的有效证书

1. 是否信任 ：接收方内置了信任根证书的公钥，需要证书是不是这些信任根证书签发的或者信任根证书的二级证书机构颁发的。

2. 是否有效：证书是否在有效期内。

3. 是否合法：对方是不是上述证书的合法持有者，证明对方是否持有证书的对应私钥。验证方法两种，一种是对方签个名，我用证书验证签名；另外一种是用证书做个信封，看对方是否能解开。

4. 是否吊销：验证是否吊销可以采用黑名单方式或者OCSP方式。黑名单就是定期从CA下载一个名单列表，里面有吊销的证书序列号，自己在本地比对一下就行。优点是效率高。缺点是不实时。OCSP是实时连接CA去验证，优点是实时，缺点是效率不高。
 
### 自签名证书如何验证

自签名证书是自己给自己签发的证书，也就是说自己做自己的 CA 机构，为自己担保，因此无法内置在系统当中，因此我们通常会在客户内置一个证书文件，自己进行校验。

简单来说，握手流程需要两对密钥对：

1. 一对 CA 的密钥对 `JKS_A`，由 CA 机构维护，通常他的公钥内置在 os 中，用来签名服务端信息摘要，保证服务端公钥的真实性，避免中间人攻击。
2. 一对服务器的密钥对 `JKS_B`，他是握手过程中随机生成的，然后用「它的公钥及其他内容」的摘要去向 CA 实时签发证书，用来进行对称密钥的加密传输。

自签名证书就是不走 CA 机构，而是自己生成一对密钥对 `JKS_C`，他的作用就好比 CA 的密钥对 `JKS_A`，也是为了保证公钥的真实性，握手过程和原来一样，只是我们不需要去 CA 签发证书了，用自己的 `JKS_C` 签发就可以了；同样因为 `JKS_C` 是我们自己的密钥对，公钥没有被内置在 os 中，所以此时需要我们自己把 `cert` 文件（`JKS_C` 的公钥）放到本地，自己完成原本由 os 完成的 CA 校验任务。


### 双向验证

双向验证指的是，不光客户端要验证来自服务器的连接是不是可靠，服务器也要验证客户端。

服务端也会内置一套受信任的 CA 证书列表，用于验证客户端证书的真实性。验证过程和客户端验证服务端类似。

 

## Https 握手

单向验证握手过程：

- 1、Client Hello  ➡️

> 客户端向服务器发送握手信息，告知自己支持的加密算法、摘要算法、安全层协议版本、随机数 `Random-Secret-C`。

- 2、Server Hello  ⬅️

> 服务端随机生成本次握手需要的非对称加密的密钥对（私钥+公钥），将来用来传输对称加密密钥。

> 服务端生成消息，内容包含随机数 `Random-Secret-S`，确定的一组加密算法和摘要算法，服务端公钥，域名信息等。

> 服务端对信息内容摘要，使用摘要的信息向 CA 机构申请的签名证书。

> 服务端向客户端发送消息和申请的证书。

> **如果需要双向验证的话，请求客户端证书。**

- 3、验证服务端证书，提取服务端公钥  ➡️

> 客户端从信任证书列表中发现是受信任的证书，会首先验证证书是否被信任、有效性、合法性等信息，验证过程参照上面的 **证书验证**。

> 验证通过，客户端使用 `CA` 机构的公钥对证书解密，拿到消息的摘要，对真正的消息内容进行摘要，对比确定消息没有被篡改，则取出服务端公钥。

> **如果需要双向验证的话，向服务端发送自己的证书。**

> 客户端生成随机数字 `Pre-Master-Secret`，将其进行摘要处理，使用服务端公钥对消息和摘要结果加密，发送给服务器，并发送一个编码改变通知，说明以后将会开始加密通信。

- 4、生成对称加密密钥  ⬅️

> **如果需要双向验证的话，首先验证客户端证书，验证过程类似客户端验证，验证失败则断开连接**

> 服务器使用私钥对收到的信息解密，对消息进行摘要对比无误，则说明对称加密的密钥没有被篡改，然后使用 `Random-Secret-C`,`Random-Secret-S`,`Pre-Master-Secret` 生成最终将要进行对称加密通信的密钥 `Master-Secret`。

> 服务器使用 `Master-Secret` 加密一段握手信息及其摘要，发送给客户端，并发送一个编码改变通知，说明以后将会开始加密通信。

- 5、客户端验证加密结果，握手结束 👌

> 客户端使用 `Random-Secret-C`,`Random-Secret-S`,`Pre-Master-Secret` 生成同样的对称加密密钥 `Master-Secret`，使用密钥解密，并验证信息摘要，没有问题则握手结束。

> 后面的通信将会使用新生成的对称加密密钥加密进行。

图解：

![https](http://olx4t2q6z.bkt.clouddn.com/17-12-19/94724225.jpg)

 
## Q&A

Q：为什么要有 3 个随机数？

> 不管是客户端还是服务器，都需要随机数，这样生成的密钥才不会每次都一样。由于SSL协议中证书是静态的，因此十分有必要引入一种随机因素来保证协商出来的密钥的随机性。对于RSA密钥交换算法来说，Pre-Master-Secret 本身就是一个随机数，再加上hello消息中的随机，三个随机数通过一个密钥导出器最终导出一个对称密钥。

> Pre-Master 的存在在于 SSL 协议不信任每个主机都能产生完全随机的随机数，如果随机数不随机，那么 Pre-Master-Secret 就有可能被猜出来，那么仅适用 Pre-Master-Secret 作为密钥就不合适了，因此必须引入新的随机因素，那么客户端和服务器加上 Pre-Master-Secret 三个随机数一同生成的密钥就不容易被猜出了，一个伪随机可能完全不随机，可是是三个伪随机就十分接近随机了，每增加一个自由度，随机性增加的可不是一。"


Q：放在 Android 客户端的 cert 文件是啥？

> 是自签名证书的公钥，自签名证书就是自己做自己的 CA 机构，服务端会自己维护一个密钥对 JKS，他就相当于 CA 的签发证书的密钥对，在握手过程中服务器不需要走 CA 申请签名证书了，自己签发就可以，原本 CA 的公钥被内置在 os 中，CA 的验证不需要我们来关注，但是现在是自签名的，自己做自己的 CA，所以 JKS 的公钥我们要内置在客户端中，自己完成验证过程，替代原来 os 的验证。

## 证书工具-keytool


提取证书内容为一个字符串

```
keytool -printcert -rfc -file [srca.cer]
```

生成密钥对

```
keytool -genkey -alias [march_server] -keyalg RSA -keystore [march_server.jks] -validity 3600 -storepass [123456]
```

读取密钥对信息

```
keytool -list -v -keystore [srca.jks] -storepass [123456] 
```

提取公钥，签发 `cert` 文件

```
keytool -export -alias march_server -file march_server.cer -keystore march_server.jks -storepass 123456
```

将客户端公钥导入客户端密钥对中，主要是服务器不能使用 `cert` 证书，需要导入到 `jks` 文件中

```
keytool -import -alias [march_client] -file [march_client.cer] -keystore [march_client_for_server.jks]
```


## 搭建 https 服务

这部分内容参考了 [CSDN-hongyang-Android Https 完全解析](http://blog.csdn.net/lmj623565791/article/details/48129405)，这里做一个汇总和整理。

借助 `tomcat` 搭建简单的 `https` 服务，主要是用来测试，之前使用 `12306` 的证书测试，但是前段时间 `12306` 证书换掉了，现在已经是正式证书了，所以不得以需要自己搭建一个简单的服务来做访问测试。

- 搭建单向验证的 https 服务

首先我们准备好密钥和证书，使用 `keytool` 工具生成服务端 `march_server.jks` 密钥对，然后提取服务端公钥 `march_server.cert`。

配置服务，主要是配置 `tomcat/conf/server.xml` 文件，添加一个如下的 `Connector`。

```
<!-- 
  https 测试服务
  clientAuth 和 truststoreFile 配置服务器验证客户端
  keystoreFile 和 keystorePass 配置客户端验证服务器
  -->
  <Connector
    
    clientAuth="false"
    
    keystoreFile="/Users/march/Documents/march_server.jks" 
    keystorePass="123456" 

    disableUploadTimeout="true" 
    enableLookups="true" 
    SSLEnabled="true" 
    acceptCount="100"  
    maxSpareThreads="75" 
    maxThreads="200" 
    minSpareThreads="5" 
    protocol="org.apache.coyote.http11.Http11NioProtocol" 
    scheme="https" 
    secure="true" 
    port="8443" 
    sslProtocol="TLS"/>
```

- 搭建双向验证的 https 服务

再生成客户端密钥对 `march_client.jks`，然后签发客户端公钥 `march_client.cert`，为了能在服务端使用，将 `march_client.cert` 导入到新的密钥对 `march_client_for_server.jks`。使用的命令都可以在上一节找到，生成这几个密钥，是为了搭建后面的服务作准备。
 
更改 `conf/server.xml` 配置

```
<!-- 
  https 测试服务
  clientAuth 和 truststoreFile 配置服务器验证客户端
  keystoreFile 和 keystorePass 配置客户端验证服务器
   -->
  <Connector 
    
    clientAuth="true"
    truststoreFile="/Users/march/Documents/march_client_for_server.jks" 
    
    keystoreFile="/Users/march/Documents/march_server.jks" 
    keystorePass="123456" 

    disableUploadTimeout="true" 
    enableLookups="true" 
    SSLEnabled="true" 
    acceptCount="100"  
    maxSpareThreads="75" 
    maxThreads="200" 
    minSpareThreads="5" 
    protocol="org.apache.coyote.http11.Http11NioProtocol" 
    scheme="https" 
    secure="true" 
    port="8443" 
    sslProtocol="TLS"/>
```

此时浏览器已经不能访问


## Android 进行 Https 访问

如果你的证书是购买的 CA 签发的证书，是可以直接进行 https 访问的，不需要做什么特殊处理，下面主要介绍使用自签名证书的情况。

启用双向验证时，Android 客户端也不能直接使用 `march_client.jks` 需要转为 `bks` 文件，使用下面的工具。[jks 转 bks 工具](https://sourceforge.net/projects/portecle/files/latest/download?source=files)


注：这部分不是说如何发请求，只是讨论怎么配置证书和密钥对去发起 Https 访问。

在 Android 中进行 Https 访问，主要牵扯到以下几个关键类

1. TrustManagerFactory，它主要是用来导入自签名证书，用来验证来自服务器的连接。
2. KeyManagerFactory，当开启双向验证时，用来导入客户端的密钥对。
3. SSLContext，SSL 上下文，使用上面的两个类进行初始化，就是个上下文环境。

😠 都让开，我要贴代码了！


### SSLContext

为了简单起见，我们先来看看如何生成 SSLContext

```java
/**
 * 创建 SSLContext 
 * @param keyManagerFactory 用于双向验证，不需要可以直接为空
 * @param trustManagerFactory 用于导入证书
 * @return SSLContext
 */
private SSLContext getSSLContext(@Nullable KeyManagerFactory keyManagerFactory,
        TrustManagerFactory trustManagerFactory)
        throws NoSuchAlgorithmException, KeyManagementException {
    if (trustManagerFactory == null)
        return null;
    SSLContext sslContext = SSLContext.getInstance("TLS");
    KeyManager[] keyManagers = null;
    if (keyManagerFactory != null) {
        keyManagers = keyManagerFactory.getKeyManagers();
    }
    sslContext.init(keyManagers, trustManagerFactory.getTrustManagers(), new SecureRandom());
    return sslContext;
}
```


### TrustManagerFactory

导入客户端证书，生成 TrustManagerFactory

```java
/**
 * 导入客户端证书，生成 TrustManagerFactory
 * @param serverCertInputStreams 证书的输入流
 * @return TrustManagerFactory
 */
private TrustManagerFactory getTrustManagerFactory(InputStream... serverCertInputStreams)
        throws KeyStoreException, CertificateException, NoSuchAlgorithmException, IOException {
    if (serverCertInputStreams == null || serverCertInputStreams.length == 0) {
        return null;
    }
    CertificateFactory certificateFactory = CertificateFactory.getInstance("X.509");
    // 为证书设置一个keyStore，并将证书放入 keyStore 中
    KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());
    keyStore.load(null, null);
    int index = 0;
    for (InputStream inputStream : serverCertInputStreams) {
        String alias = Integer.toString(index++);
        Certificate certificate = certificateFactory.generateCertificate(inputStream);
        keyStore.setCertificateEntry(alias, certificate);
    }
    // 创建 TrustManagerFactory
    TrustManagerFactory trustManagerFactory = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
    trustManagerFactory.init(keyStore);
    TrustManager[] trustManagers = trustManagerFactory.getTrustManagers();
    if (trustManagers.length != 1 || !(trustManagers[0] instanceof X509TrustManager)) {
        throw new IllegalStateException("Unexpected default trust managers:"
                + Arrays.toString(trustManagers));
    }
    return trustManagerFactory;
}
```

### KeyManagerFactory

双向验证时，配置客户端密钥对，生成 KeyManagerFactory

```java
/**
 * 双向验证时，配置客户端密钥对，生成 KeyManagerFactory
 *
 * @param clientCertInputStream 密钥对输入流
 * @param passWd                密钥对密码
 * @return KeyManagerFactory
 */
private KeyManagerFactory getKeyManagerFactory(InputStream clientCertInputStream, String passWd)
        throws KeyStoreException, CertificateException, NoSuchAlgorithmException, IOException, UnrecoverableKeyException {
    if (clientCertInputStream == null || passWd == null) {
        return null;
    }
    //本地证书 初始化keystore
    KeyStore clientKeyStore = KeyStore.getInstance(KeyStore.getDefaultType());
    clientKeyStore.load(clientCertInputStream, passWd.toCharArray());
    KeyManagerFactory keyManagerFactory = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
    keyManagerFactory.init(clientKeyStore, passWd.toCharArray());
    return keyManagerFactory;
}
```

### Init Connection

这样我们就拿到了已经生成好的 SSLContext，我们需要的 KeyManagerFactory，TrustManagerFactory 都已经加到 SSLContext 里面了。

如果是用 `HttpsURLConnection`

```java
connection.setSSLSocketFactory(sslContext.getSocketFactory());
```
如果是用 `OkHttp`

```java
// 这个方法已经过时了
okHttpClientBuilder.sslSocketFactory(sslContext.getSocketFactory());

// 新的方法需要一个 X509TrustManager，那我们就从 TrustManagerFactory 取出来给他
TrustManagerFactory trustManagerFactory = getTrustManagerFactory(serverCertInputStreams);
KeyManagerFactory keyManagerFactory = getKeyManagerFactory(clientCertInputStream, passWd);
if (trustManagerFactory != null) {
    X509TrustManager x509TrustManager = (X509TrustManager) trustManagerFactory.getTrustManagers()[0];
    SSLContext sslContext = getSSLContext(keyManagerFactory, trustManagerFactory);
    if (x509TrustManager != null && sslContext != null) {
        okHttpClientBuilder.sslSocketFactory(sslContext.getSocketFactory(), x509TrustManager);
    }
}
```

### HostnameVerifier

另外 `HostnameVerifier` 也是必要的，这个 `HttpsURLConnection` 和 `OkHttp` 没啥差别

```java
connection.setHostnameVerifier(new HostnameVerifier() {
    @Override
    public boolean verify(String hostname, SSLSession session) {
        return true;
    }
});

okHttpClientBuilder.hostnameVerifier(new HostnameVerifier() {
    @Override
    public boolean verify(String hostname, SSLSession session) {
        return true;
    }
});
```


## 巨人的肩膀

[CSDN 如何理解HTTP协议的 “无连接，无状态” 特点？](http://blog.csdn.net/tennysonsky/article/details/44562435)
	
[知乎 一个故事让你彻底理解 Https](https://zhuanlan.zhihu.com/p/31880655)

[简书 HTTPS 是如何保证安全的？](http://www.jianshu.com/p/b894a7e1c779)

[个人博客-饿了么Android-聊聊 Android HTTPS 的使用姿势](http://dieyidezui.com/talk-about-android-https/)

[个人博客-细说 CA 和证书](http://www.barretlee.com/blog/2016/04/24/detail-about-ca-and-certs/)

[个人博客-https详解](http://luojinping.com/2016/04/17/https%E8%AF%A6%E8%A7%A3/) 概要介绍

[CSDN-hongyang-Android Https 完全解析](http://blog.csdn.net/lmj623565791/article/details/48129405)

[阮一峰-SSL/TLS协议运行机制的概述](http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html)

	


## 注脚

<span id="消息摘要"/>
**消息摘要（Message Digest）**

指的是将长度不固定的参数作为输入参数，运行特定 hash 函数，生成固定长度的输出，这个输出就是消息的摘要，常用算法有 `MD5` 和 `SHA1`，摘要算法是单向不可逆的，即无法从摘要重新反向出原消息的内容。


<span id="信用背书"/>
**信用背书**

票据的收款人或持有人在转让票据时，在票据背面签名或书写文句的手续。背书的人就会对这张支票负某种程度、类似担保的偿还责任。


<span id="非对称加密应用场景"/>
**非对称加密的两种应用场景**

1. 某用户使用自己的私钥对数据加密，任何人都可以使用公钥对数据进行解密，因为私钥只有该用户持有，则说明该数据一定出自于该用户。公众可以用这一方法验证内容是否完整，是否被篡改，接受者可以认定该内容出自该用户，该用户也无法抵赖，这被称作数字签名。
2. 某用户使用公开的公钥对数据进行加密，那么可以保证只有发布公钥的一方可以对数据进行解密，别人无法获取数据的内容，保证数据传递的安全。


<span id="签发证书权威性问题"/>

**应对 CA 签发证书权威性问题的解决方案**

1. Google 推动的 Certificate Transparency 技术，它旨在通过开放的审计和监控系统，提高 HTTPS 网站证书安全性。CT 技术能改善这种情况，但 CT 还没有普及，现阶段浏览器不能贸然阻断没有提供 SCT 信息的 HTTPS 网站。[CT 介绍](https://imququ.com/post/certificate-transparency.html)

2. HTTP Public Key Pinning 技术 [HPKP 介绍](https://imququ.com/post/http-public-key-pinning.html)

