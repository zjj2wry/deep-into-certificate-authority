author: zhengjiajin
summary: deep into certificate authority
id: ./
categories: https,ssl client auth
feedback link: https://github.com/zjj2wry/deep-into-certificate-authority

# deep-into-certificate-authority

## https 
### overview
https 的作用是提供对网站服务器的身份认证(**Server Auth**)，保护交换数据的隐私与完整性。 理解 https 之前需要一些预备知识，比如加密，签名，密钥等。

- **对称密钥:** 加密和解密使用相同的密钥
- **非对称密钥:** 分成公钥和私钥，公钥是可以公开的，私钥是私有的。公钥和私钥是唯一的一对，比如使用公钥加密，则只能使用对应的私钥解密
- **加密:** 对信息加密，即使数据被获取了对方也无法知道里面的内容
- **签名:** 对信息签名，通过签名验证信息有没有被篡改

### usage
https 协议中大量的使用了非对称密钥，后面会详细的描述 https 的流程，理解密钥的用法会帮助理解为什么客户端或者服务端需要这个类型密钥，为什么它们具有加密或者签名数据的能力。

- **对称密钥**
    - 使用相同的密钥加密 (encrypt) 和解密 (decrypt) 数据
    - 使用相同的密钥签名 (sign) 和验证 (verify) 签名

- **非对称密钥**
    - **使用公钥加密 (encrypt) 数据，私钥解密 (decrypt) 数据**
    - **使用私钥签名 (sign) 数据，公钥验证 (verify) 签名**


### workflow
https 的作用是客户端验证服务器端是不是一个安全的服务，简称 **Server Auth**。https 的作用是数据在传输的过程中会加密数据，传输的数据即使被获取了对方也无法知道里面的内容。
![https](imgs/https.svg)

**certificate signings:**

1. **generated private server key:** 首先服务端提供 https 前需要一对密钥，服务端提前生成私钥是一种常见的申请 csr 的方式，这里的重点是私钥是不需要给 ca 的
2. **certificate signing request:**  server 使用自己的私钥生成 csr，csr 中包含了用户的基本信息，比如 common name, email 等以及公钥的信息，在生成 csr 的时候已经有一对密钥了
3. **return signed public server cert:**  ca 也有自己的一对密钥，ca 使用自己的私钥去对 csr 签名生成证书 (**私钥签名 (sign) 数据**) ，证书包含了 csr 中的信息以及下发证书的 issuer 和签名(signature) 等信息
4. **use public ca cert verify server cert:**  ca 的证书是公开的，客户端和服务端都可以获取到 ca 证书，服务端可以使用ca 证书去验证自己拿到到证书是否合法，前提是 ca 是被信任的。 (**公钥验证 (verify) 签名**) 


**basic tls:**
证书签名在服务端提供服务前已经做好了，下面描述客户端和服务端之间的工作流。

Positive
: (async)every body can get public ca cert: 客户端一样可以从 ca 得到 public ca cert，通常它已经被加载在浏览器里了

1. **request `https://server-domain.com`:**  客户端像服务端的域名发送一个请求
2. **response public server cert:**  服务端接受到请求后，返回证书，证书和公钥一样是可以公开的
3. **use public ca cert verify public server cert:**  客户端使用 ca 证书验证服务端的证书，保证获取到的证书是合法的，被 ca 签名过的。如果验证失败，则会显示 https 警告
4. **generated random symmetric key:**  客户端生成一个随机的对称密钥
5. **transfer encrypted symmetric key:**  客户端使用服务端的证书对随机密钥加密并发送给服务端端。 (**公钥加密 (encrypt) 数据**) 
6. **use private server key to decrypt encrypted key:**  服务端使用自己的私钥解密数据，获取对称密钥。 (**私钥解密 (decrypt) 数据**) 
7. **use symmetric key to encryt content:**  服务端使用对称密钥加密消息发送给客户端。 (**对称密钥加密 (encrypt) 数据**) 
8. **use symmetric key to descrypt content:**  客户端使用对称密钥解密数据。 (**对称密钥解密 (decrypt) 数据**) 

通过上面的流程可以看到 https 的流程会用到两对非对称密钥，一对是 ca 的，一对是服务器端的，ca 的密钥主要用来对证书签名，ca 可以用自己的私钥签名无数个证书。服务器端的证书是用来加密和解密对称密钥的。下面可以引出两个问题: 

**1. 为什么需要证书签名 ?**
假设没有证书签名会发生什么，在通信的时候服务器端会发送证书给客户端，客户端不知道证书是不是服务端的，是不是合法的。所以这个时候引入了 ca，ca 是一个公开的，公认的合法机构，客户端和服务端都信任它，服务端去 ca 签名自己的证书，客户端通过 ca 给他的证书(里面包含了 ca 的公钥)验证服务端的证书。通过这样客户端和服务端相互信任。

**2. 为什么最终使用对称密钥交换数据 ?**
我们知道可以使用公钥加密数据，私钥解密数据，假设客户端也有一对密钥，服务端能通过某种方式信任客户端的公钥，它们一样可以实现加密通信。看 https 的流程其实是使用非对称密钥交换对称密钥的一个过程，为什么需要这么做？因为使用对称密钥性能比非对称密钥的性能要更好，因为对称密钥更小，且算法更容易处理。


## ssl client authentication
![client-auth](imgs/client-auth.svg)
## openssl certificate authority
## golang https
## golang two-way tls
## kubernetes x509 authentication
## kubernetes bootstrap token
## cert-manager with let's encrypt
## kubernetes tls ingress
