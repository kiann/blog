## Java、android、js通用aes和rsa通用加解密库

>简单介绍

一直想有时间整理一下有关加解密部分的东西，之前明文开发和明文传递数据习惯了，总不是长久之计，可以迟迟没有动手。年后终于有时间了，原本想理清各种加密算法之后有个规范的文档就可以了，结果百度一番，各种实现方式都有，所谓的多平台通用要么没有运行成功要么不够简洁，哪就自己造轮子吧。

说干就干。

>aes通用
    
aes算法应该算是最主流的对称加密算法，简单来说就是给出要加密的明文和一个长度16的密钥，使用该密钥加密数据，而想要解密只需要提供相同的密钥即可，优点是速度快，缺点是同一个密钥，有些场景并不合适。选用了AES-128-CBC加密模式（因为该模式最容易实现全平台通用）。具体用法看[这里](https://github.com/kiann/SampleCode/blob/master/java、android、js通用AES-Util/README.md)。

其中js实现使用了crypts-js.js为核心，并且提供了单文件版本，具体可以看[这里](https://github.com/kiann/SampleCode/tree/master/java、android、js通用AES-Util/js)，是我已经上传的代码，可以直接下载单文件版本使用，也有源码。而对应的java和Android版本可以看[这里](https://github.com/kiann/SampleCode/tree/master/java、android、js通用AES-Util/java)，因为java没有zeropadding这种补位方式，所以只能选择nopadding然后手动实现。

*注意：具体使用的时候可以使用提供的密钥生成方法，也可以手动定义密钥，唯一的要求是仅支持字母和数字，且长度必须16位。为了在网络传输的时候方便，被加密之后的密文没有使用常见的base64编码而是使用了16进制大写字符串。*
如下
```
8A8AD3F003BB877FBC8B93E23623338BCDAD13248986513D49CC46DA7C3E8960B8F8ACCD6341076AE770F7C817603021506379F0A546C7CE36370846D226D90AEC3D0B8C47EF4384D24EBC87042FB0E3FBAB1BFFC72B22D4B2E675ECF0869CF1F8FBEFABE71EFAE397F14DB98369F3792E8FEE5B18C4EF423FB0BD8B4F38870221F891C9E7BEBD889FCB6439049A7364A84A8A59ED17575B84D1C65DD703177E8830C9E80255561FC33370FA7F7742C34D35C86E3A2D01AF91661D7A649F35D138083C87A49CBC7B307913D523CD4E8384230E7043A9A8EA86828C0BFF9ABAFB8CB724489A61380AAEDDC897995B609F972F9FAEED68FCDAEAA1A88292A082EB2B81BBA80E4C4D1262126F058575EEC1FC6A075D14E20F1F93FAC7C18C47C6987BADCCCCBC115F132DF1B4E3EE9DC4EA86B781BC031987F97701ACDEF20282EEB73822934E91A40F10AD840480B2432101A51A6831A76E8CCD5BF60CACBE4FE6A6832EF9873FF2393F16F884F0C670050A8ABC835BD5E9E4530AE22FDD2EC543
```

>rsa通用

rsa算法也是主流的非对称加密算法，不同于对称加密，非对称加密天生就是用来应对密钥不安全的场景的。它提供一对密钥分别称为公钥和密钥，使用的时候将公钥部署在不安全的客户端，而将私钥放在安全的服务端，被公约加密的数据能够被私玥解密。具体的实现原理基于了一个非常简单的数学原理，计算两个质数的乘积很容易，但是根据乘积反向推算这两个质数却非常难。

用法看[这里](https://github.com/kiann/SampleCode/blob/master/java、android、js通用RSA-Util/README.md)。

js同样使用crypto-js.js为核心，提供单文件版本，在[这里](https://github.com/kiann/SampleCode/tree/master/java、android、js通用RSA-Util/js)可以下载写好的代码以及源码。因为限制，也因为用处不大，所以js版本并没有提供密钥对生成方法，仅java和Android支持。

java和Android的对应版本见[这里](https://github.com/kiann/SampleCode/tree/master/java、android、js通用RSA-Util/java)。

*注意：为了方便传输，rsa工具类从密钥对到密文都使用的是16进制大写字符串，所以通过其他方式生成的密钥对并不能直接使用，需要转成16进制字符串格式，或者直接使用提供的方法生成密钥对。js也必须使用对应的密钥对。*

>其他

最后唠叨一点跟实际代码没什么关系而跟业务流程有关的东西。

首先是rsa中的密钥对的使用，虽然公密钥都能用来加密也都能用来解密，但是因为强度要求的不同，还是应该严格按照公钥加密私钥解密来，本工具也仅支持该方式。

其次是实际业务中加密方式应该怎么用。
* 如果是简单加密场景，可以用与用户有关的信息作为密钥进行aes加密，这应该最简单了，有一定的安全性且容易实现，即使被破解也只是暴露一个用户信息而不是全库。
* 如果加密场景比较复杂，应该使用aes+rsa结合的方式，aes简单高效但是不够安全，rsa足够安全但是效率较低，结合使用才最安全。具体方式如下：
    假设是浏览器和服务器（c-s模式类似），用户加载页面时直接将rsa公钥传递给浏览器，当浏览器需要提交加密信息时，浏览器生成一个aes密钥并用该密钥加密信息，再然后用公钥将该aes密钥也加密，最后把这些信息都提交给服务器，服务器收到后先解密拿到aes密钥，再用aes密钥解密信息。而之后服务端需要传递加密信息给浏览器时，直接用上次传递过来的aes密钥加密数据后给客户端，客户端用aes解密。当然使用的过程中涉及到密钥保持的问题，需要自行考虑。
* 再说一条，加解密必须配合身份校验活着鉴权校验一起使用，否则意义不大。

有问题请指正，欢迎交流。