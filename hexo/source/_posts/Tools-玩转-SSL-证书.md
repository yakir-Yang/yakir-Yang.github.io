---
title: 玩转 SSL 证书
date: 2018-08-09 23:41:26
tags:
categories: Tools
---



## Introduction

Openssl 是一个很牛逼的工具，基本能搞定 PKI & HTTPS 证书相关的事情。这篇博文归类了一堆常用的命令，全部都是关于 key & csr & crt。

本文分成两部分，第一部分对 SSL/TLS 证书最核心的几个名词概念，第二部分则是一系列 openssl 常用命令。



<!-- more -->

----



## Key, CSR, CRT, CA 都是什么？

### 什么是 KEY、CRT ？
KEY 是指密钥，通过一定长度的随机数，经过指定的加密算法处理，得到后的一个文本密钥文件。文件中既包含了 公钥 Publib Key & Private key 两个部分。

密钥文件包含 *PRIVATE KEY* 字符，格式如下 （例子中采用 RSA 算法加密）：

```
-----BEGIN RSA PRIVATE KEY-----
...............
-----END RSA PRIVATE KEY-----
```

CRT 是指证书，其与 KEY 是一对的。KEY 密钥文件需要严格保密，而 CRT 证书文件却是公开的。以 PC 浏览器上网为例，浏览器通过向服务器 (如 https://github.com) 索取其 server.crt 证书，然后与浏览器内置的受信的 ca.crt 证书进行验证。如果校验成功，则浏览器会提示网站通过 https 连接是安全的。

证书文件包含 CERTIFICATE 字符，格式如下：

```
-----BEGIN CERTIFICATE-----
...............
-----END CERTIFICATE-----
```


### 什么是 CSR ？

如果你想要一个 CA (Certificate Authority) 认证机构为你颁发CRT 证书的话，那你需要先提供一个 CSR 请求文件。这个 CSR 请求文件中，会包含 Public Key 公钥，以及一些申请者信息 (DN *Distingusied Name*)。

DN 信息中最最重要的部分，就是 **Common Name (CN)**，这个成员需要与后续安放 CRT 证书的服务器，其主机域名  Fully Qualified Domain Name (FQDN) 完全一致才行。
请求文件包含 CERTIFICATE REQUEST 字符，格式如下

```
-----BEGIN CERTIFICATE REQUEST-----
...............
-----END CERTIFICATE REQUEST-----
```



### 什么是 CA ？

CA 全称为 Certificate Authotiry，证书授权中心，又被成为根证书。它类似于工商管理局，给公司企业颁发营业执照的。它主要有两大主要特性：

- CA 本身是受国际认可，受信任的
- CA 需要给它受信任的对象颁发证书

证书长啥样？其实你的电脑中有一堆 CA 证书。你可以看一看：

- Chrome 浏览器：设置 -> 高级 -> 管理证书 -> 授权中心
- Ubuntu: `/etc/ssl/certs`



### CA 的证书 ca.crt 与 SSL Server 的证书 Server.crt 是什么关系呢？

1. SSL Server 自己生成一个 密钥 KEY，内置包含 Private Key/Public Key， 私钥加密，公钥解密；
2. Server 利用 密钥 生成一个请求文件 server.csr，请求文件中包含有 Server 的一些信息，如域名/申请者/公钥等； 
3. Server 将请求文件 server.csr 递交给 CA，然后 CA 对其验明正身后，将用 ca.key 和 server.csr 加密生成 server.crt  证书；
4. 由于 ca.key 和 ca.crt 是一对，于是以后客户端通过使用 ca.crt 就可以解密认证 server.crt 证书了。

在实际应用中：如果 SSL Client 想要校验 SSL server。那么 SSL server 必须要将他的证书 server.crt 传给 client。然后 client 用 ca.crt 去校验 server.crt 的合法性。如果是一个钓鱼网站，那么 CA 是不会给他颁发合法 server.crt 证书的，这样 client 用 ca.crt 去校验，就会失败。比如浏览器作为一个 client，你想访问合法的淘宝网站([https://www.taobao.com](https://www.taobao.com/)，结果不慎访问到 [https://wwww.jiataobao.com](https://wwww.jiataobao.com/) ，那么浏览器将会检验到这个假淘宝钓鱼网站的非法性，提醒用户不要继续访问！这样就可以保证了client的所有https访问都是安全的。

### 什么是 SSL/TLS 单向认证，双向认证 ？

**单向认证**，指的是只有一个对象校验对端的证书合法性。通常都是 client 来校验服务器的合法性。那么 client 需要一个 ca.crt，服务器需要 server.crt，server.key；

**双向认证**，指的是相互校验，服务器需要校验每个 client，client 也需要校验服务器。server 需要 server.key、server.crt、ca.crt；client 需要 client.key、client.crt、ca.crt；



----



## Openssl 常用命令

### 生成 CSRs 请求文件

生成 CSR 请求文件的时候，需要填写 DN Distingusied Name 信息。openssl 默认是通过 shell 交互让用户输入，如下：

> Country Name (2 letter code) [AU]: **US**
>
> State or Province Name (full name) [Some-State]: **New York**
>
> Locality Name (eg, city) []: **Brooklyn**
>
> Organization Name (eg, company) [Internet Widgits Pty Ltd]: **Example Brooklyn Company**
>
> Organizational Unit Name (eg, section) []:**Technology Division**
>
> Common Name (e.g. server FQDN or YOUR name) []: **examplebrooklyn.com**
>
> Email Address []:

如果你不想通过交互的方式，来填写这些额外的信息。那则可以使用 `-subj`  选项，如下：

>  `-subj "/C=US/ST=New York/L=Brooklyn/O=Example Brooklyn Company/CN=examplebrooklyn.com"`

接下来的部分将会教大家如何生成与查看 CSR 文件。



**产生一个新的 Private Key 和 CSR**

如果你想使用 HTTPS (HTTP over TLS) 来加密你自己的 Apache HTTP or Nginx web server，并且你想通过 CA 来帮你解决 SSL 认证的问题。那采用以下命令，创建一个的密钥 (`domain.key`) 和一个对应的 CSR 请求文件 (`domain.csr`)：

```
openssl req \
        -newkey rsa:2048 -nodes -keyout domain.key \
        -out domain.csr
```

 其中 `-newkey rsa:2048` 参数指定了密钥应该通过 RSA 算法产生，长度为 2048 bit。而 `-nodes` 选项指出产生的密钥不需要额外通过 password 进一步加密。`-new` 选项在这里是默认选项，既表明需要产生一个 CSR 请问文件。

<br>



**通过已有的 Private Key 来产生 CSR**

当你已经有了一个 Private Key，而且你想通过这个 Key 来向 CA 请求证书的话。那采用以下命令，通过已有的密钥 (`domain.key`) 来产生 CSR 文件 (`domain.csr`)：

```
openssl req \
        -key domain.key \
        -new -out domain.csr
```

填写完 Distingusied Name 信息后，将会得到 CSR 请求文件。其中 `-key` 选项是为了指明需要使用的 Existing Private Key (`domain.key`) ，而 `-new` 选项是为了表明需要产生一个 CSR 请求文件。

> 如果产生 CSR 请求文件，所需的 Distingusied Name 的信息 以及 Private Key 密钥都不变的情况下，那最后得到的 CSR 文件内容也不会变化。不论产生多少次，或者在不同操作系统上，最后得到的 CSR 内容也完全一致。

<br>



**通过已有的 CRT 和 Private Key 来产生 CSR**

当你想要找 CA 更新已有的 CRT 证书的时候，当时又找不到当初的 CSR 请求文件，那你就可以采用以下命令，从 CRT 证书中提取出所需要的 Distingusied Name 信息，与 Private Key 合成出 CSR 文件。

以下命令通过已有的 Private Key (`domain.key`) 和 CRT 证书 (`domain.crt`)，来生成所需的 CSR 请求文件 (`domain.csr`)

```
openssl x509 \
        -in domain.crt \
        -signkey domain.key \
        -x509toreq -out domain.csr
```

其中 `-x509toreq` 选项既表明，想通过一个 X509 的 CRT 证书来生成一个 CSR 请求文件。

<br>



### 生成 CRTs 证书文件

如果你想要使用 SSL 来保证安全的连接，但是又不想去找 public CA 机构购买证书的话，那你就可以自己来生成 Self-signed Root CA 证书。

Self-signed 的证书也可以用来加密连接，只不过你的用户在访问你的网站的时候，将会提示出警告，说你的网站是不被信任的。因此当你开发的服务，并不需要提供给其他用户使用的时候 (e.g. non-production or non-public servers)，你才可以使用 Self-signed 形式的证书。



**生成 Self-Signed 的证书**

以下命令将会生成一个 2048-bits 的 Private Key (`self-ca.key`) 和 Self-Signed 的 CRT 证书 (`self-ca.crt`)：

```
openssl req \
        -newkey rsa:2048 -nodes -keyout self-ca.key \
        -x509 -days 365 -out self-ca.crt
```

其中`-x509` 选项是为了告诉 `req`，生成一个 self-signed 的 X509 证书。而 `-days 365` 表明生成的证书有效时间为 365 天。这条命令执行过程中，会产生一个临时的 CSR 文件，但执行结束后就被删除了。

<br>



**通过已有的密钥 Private Key 来产生 Self-Signed 证书**

以下命令是通过已有的 Private Key (`self-ca.key`)，来生成一个 Self-Signed 的 CRT 证书 (`self-ca.crt`)：

```
openssl req \
        -key self-ca.key
        -new \
        -x509 -days 365 -out self-ca.key
```

<br>



**通过已有的密钥 Private Key & 请求文件 CSR 来产生 Self-Signed 证书**

以下命令是通过已有的 Private Key (`self-ca.key`) 以及请求文件 (`self-ca.csr`)，来生成一个 Self-Signed 的 CRT 证书 (`self-ca.crt`)：

```
openssl x509 \
       -signkey self-ca.key \
       -in self-ca.csr \
       -req -days 365 -out self-ca.crt
```

<br>



**通过已有的请求文件 CSR，通过 Self-Signed CA 来签发 CRT 证书**

以下命令是通过已有的 Self-Signed CA 的证书 (`self-ca.crt`) 和密钥 (`self-ca.key`)，和请求文件 CSR (`domain.csr`) 来签发生成 CRT 证书 (`domain.crt`)：

```
openssl x509 \
        -req \
        -CAcreateserial -days 365 \
        -CA self-ca.crt -CAkey self-ca.key \
        -in nginx.csr -out nginx.crt
```

<br>



### 校验证书

Openssl 默认生成的证书相关的文件都是 PEM 格式编码的，这部分通过 openssl 命令来解析这些文件。

**查看请求文件 CSR**

```
openssl req -text -noout -verify -in domain.csr
```

**查看证书文件 CRT**

```
openssl x509 -text -noout -in domain.crt
```

**校验证书文件 CRT 合法性**

以下命令来校验 `domain.crt` 证书，是否是由 `ca.crt` 证书签发出来的：

```
openssl verify -verbose -CAFile ca.crt domain.crt
```

**校验 Private Key & CSR & CRT 三者是否是匹配**

以下命令是分别提取出 Private Key (`domain.key`) & CSR (`domain.csr`) & CRT (`domain.crt`) 三者中包含的 Public Key，然后通过 md5 运算检查是否一致：

```
openssl rsa -noout -modulus -in domain.key | openssl md5
openssl req -noout -modulus -in domain.csr | openssl md5
openssl x509 -noout -modulus -in domain.crt | openssl md5
```

<br>



### CRT 格式转换

上面通过 X509 证书生成的证书，都是 ASCII PEM 格式进行编码的。然而证书也可以转换成其他格式，有些格式能够将 Private Key & CSR & CRT 三者全部打包在同一个文件中。



#### PEM 格式 vs DER 格式

The DER format is typically used with Java.

- PEM 格式转成 DER 格式

```
openssl x509 \
        -in domain.crt \
        -outform der -out domain.der
```

- DER 格式转成 PEM 格式

```
openssl x509 \
        -inform der -in domain.der \
        -out domain.crt
```



#### PEM 格式 vs PKCS7 格式

> PKCS7 files, also known as P7B, are typically used in Java Keystores and Microsoft IIS (Windows). They are ASCII files which can contain certificates and CA certificates.

- PEM 格式转成 PKCS7 格式

```
openssl crl2pkcs7 -nocrl \
        -certfile domain.crt \
        -certfile ca-chain.crt \
        -out domain.p7b
```

- PKCS7 格式转成 PEM 格式

```
openssl pkcs7 \
        -in domain.p7b \
        -print_certs -out domain.crt
```

如果你的 PKCS7 文件中包含了多个证书文件 (e.g. `domain.crt` & `ca.crt`) ，那上面命令生成的 PEM 文件中将同时包含所有被打包的证书。



#### PEM 格式 vs PKCS12 格式

> PKCS12 files, also known as PFX files, are typically used for importing and exporting certificate chains in Micrsoft IIS (Windows).

- PEM 格式转成 PKCS12 格式

```
openssl pkcs12 \
        -inkey domain.key \
        -in domain.crt \
        -export -out domain.pfx
```

- PKCS12 格式转成 PEM 格式

```
openssl pkcs12 \
        -in domain.pfx \
        -nodes -out domain.combined.crt
```



----



## 示例案例

我自己搭建的 Nginx Web Server 想要提供 HTTPS 连接方式，但是觉得不想花钱去找知名 CA 证书中心 (e.g. AWS Certificate Manager) 购买证书。于是我想通过 Self-Signed 的方式来解决我的需求：

1. 先生成 Self-Signed CA 证书：

```
openssl req \
        -newkey rsa:2048 -nodes -keyout self-ca.key \
        -x509 -days 365 -out self-ca.crt
```

2. 生成 Nginx Web Server 需要的 Private Key & CSR ：

```
openssl req \
        -newkey rsa:2048 -nodes -keyout nginx.key \
        -out nginx.csr
```

3. 利用 Self-Signed CA 证书，来签发业务所需要的 CRT 证书：

```
openssl x509 \
        -req -in nginx.csr \
        -out nginx.crt \
        -CAcreateserial -days 365 \
        -CA self-ca.crt -CAkey self-ca.key
```

经过了上面 3 个步奏后，你就得到了 `self-ca.crt` & `self-ca.key` ，以及由它签发出来的 `nginx.crt` & `nginx.key` 。 

如何使用上面 4 个文件：首先将 `nginx.crt` & `nginx.key` 部署在 Nginx Web Server 上，然后将 `self-ca.crt` 发布给客户端即可。客户端就可以通过 `self-ca.crt` 与业务服务器建立 SSL/TLS 安全连接。



## Reference

 [SSL-TLS 双向认证(一) -- SSL-TLS工作原理](https://blog.csdn.net/espressif/article/details/78541410)

[OpenSSL Essentials: Working with SSL Certificates, Private Keys and CSRs](https://www.digitalocean.com/community/tutorials/openssl-essentials-working-with-ssl-certificates-private-keys-and-csrs)



<font color="red"> **转载请注明出处：** http://kyang.cc/ </font>

 