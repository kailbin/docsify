
# 证书申请




[TOC]


## 术语

- CA :  电子商务认证授权机构（Certificate Authority）
- CSR : 证书签名请求(Certificate Signing Request) 是签发证书时的重要材料，它可以保证私钥安全的情况下通过远端CA签发证书。
- CRT : 证书文件（Certificate [sərˈtɪfɪkət]  ）















## 制作 CSR 文件

在申请**数字证书**之前，必须先生成**证书私钥**和**证书请求文件**(CSR,Cerificate Signing Request)，
CSR是您的公钥证书原始文件，包含了您的**服务器信息和您的单位信息**，需要提交给CA认证中心。
在生成CSR文件时**会同时生成私钥文件**，需妥善保管和备份。


生成CSR文件时，一般需要输入以下信息(中文需要UTF8编码)：
- `Organization Name(O)`：申请单位**名称**法定名称，可以是中文或英文
- `Organization Unit(OU)`：申请单位的**所在部门**，可以是中文或英文
- `Country Code(C)`：申请单位**所属国家**，只能是两个字母的国家码，如中国只能是：**CN**
- `State or Province(S)`：申请单位所在**省名或州名**，可以是中文或英文
- `Locality(L)`：申请单位所在**城市名**，可以是中文或英文
- `Common Name(CN)`：申请SSL证书的具体**网站域名**


### 使用 OpenSSL 制作 

``` bash
# -new 指定生成一个新的CSR
# -out certificate.csr 输出文件为 当前目录下 ca.kail.xyz.csr
# - newkey rsa:2048 指定私钥类型和长度
# - nodes指定私钥文件不被加密
# -keyout 生成私钥输出文件为 当前目录下 ca.kail.xyz.key
$ openssl req -new -out ca.kail.xyz.csr -newkey rsa:2048 -nodes -keyout ca.kail.xyz.key

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:shanghai
Locality Name (eg, city) [Default City]:shanghai
Organization Name (eg, company) [Default Company Ltd]:kail
Organizational Unit Name (eg, section) []:kail-ca
Common Name (eg, your name or your server's hostname) []:ca.kail.xyz
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

显示 csr 文件内容
``` bash
openssl req -in ca.kail.xyz.csr -text
```


### 使用 Java keytool 工具制作


#### 使用 Keytool 工具生成 jks(Java Key Store) 文件

jks 是一个密码保护的文件，存放**私钥**和**证书**

``` bash
# -keyalg 指定密钥类型，必须是 RSA
# -alias 指定证书别名，可自定义
# -keysize 指定密钥长度为 2048
# -keystore 指定证书文件保存路径

keytool -genkey -alias ca.kail.xyz -keyalg RSA -keysize 2048 -keystore ./ca.kail.xyz.jks
```
输入命令后会有下面的交互式操作：
``` text
输入密钥库口令:
再次输入新口令:
您的名字与姓氏是什么?
  [Unknown]:  ca.kail.xyz
您的组织单位名称是什么?
  [Unknown]:  dev
您的组织名称是什么?
  [Unknown]:  kail
您所在的城市或区域名称是什么?
  [Unknown]:  Shanghai
您所在的省/市/自治区名称是什么?
  [Unknown]:  Shanghai
该单位的双字母国家/地区代码是什么?
  [Unknown]:  CN
CN=ca.kail.xyz, OU=dev, O=kail, L=Shanghai, ST=Shanghai, C=CN是否正确?
  [否]:  Y

输入 <ca.kail.xyz> 的密钥口令
        (如果和密钥库口令相同, 按回车):
```

#### 通过 jks 文件生成证书请求

``` bash
# sigalg指定摘要算法，使用 SHA256withRSA。
# alias指定别名，必须与 keystore 文件中的证书别名一致。
# keystore指定证书文件。
# file指定证书请求文件(CSR)。

keytool -certreq -sigalg SHA256withRSA -alias ca.kail.xyz -keystore ./ca.kail.xyz.jks -file ./ca.kail.xyz.csr
```

#### 从 jks 文件 获取 私钥

1. 将 JKS 格式证书转换成 PFX 格式
``` bash
keytool -importkeystore -srckeystore ./ca.kail.xyz.jks -destkeystore ./ca.kail.xyz.pfx -srcstoretype JKS -deststoretype PKCS12
```

2. PFX 文件转成 pem 文件
``` bash
openssl pkcs12 -in ca.kail.xyz.pfx -nodes -out ca.kail.xyz.pem
```

3. pem 文件 中提取出私钥
``` bash
openssl rsa -in ca.kail.xyz.pem -out ca.kail.xyz.key
```


> [**主流数字证书都有哪些格式？**](https://help.aliyun.com/knowledge_detail/42214.html) 介绍了各种格式之间的转换









## 自制证书

### keytool

上面 “从 jks 文件 获取 私钥”中的 第2步，生成的 ca.kail.xyz.pem 文件， 里面包含 私钥和证书信息，编辑删除多余的部分 保留 `-----BEGIN CERTIFICATE-----` 和 `-----END CERTIFICATE-----` 中间的内容，后缀名改为 `.csr` 即可。
Windows 下 双击打开，可查看证书内容。

### OpenSSL

```
openssl x509 -req -days 365 -in ca.kail.xyz.csr -signkey ca.kail.xyz.key -out ca.kail.xyz.crt
```














## 申请授信证书

### 通过 SSL.md 免费获取

参见 [快速获取免费SSL证书](https://zhuanlan.zhihu.com/p/30377917)
























## X509 文件扩展名

DER、PEM、CRT和CER这些扩展名经常令人困惑。很多人错误地认为这些扩展名可以互相代替。
尽管的确有时候有些扩展名是可以互换的，但是最好你能确定证书是如何编码的，进而正确地标识它们。正确地标识证书有助于证书的管理。

### 编码 (也用于扩展名)

- `.DER` 扩展名DER用于**二进制**DER编码的证书。比较合适的说法是 **“我有一个DER编码的证书”**，而不是“我有一个DER证书”。
- **`.PEM`** 扩展名PEM用于**ASCII(Base64)编码的各种X.509 v3 证书**。文件开始由一行"—– BEGIN …“开始。

### 常用的扩展名

- **`.CRT`** 扩展名CRT**用于证书**。**证书可以是DER编码，也可以是PEM编码**。扩展名CER和CRT几乎是同义词。这种情况在各种unix/linux系统中很常见。
- `.CER` CRT证书的微软型式。可以用微软的工具把CRT文件转换为CER文件（CRT和CER必须是相同编码的，DER或者PEM）。
    扩展名为CER的文件可以被IE识别并作为命令调用微软的cryptoAPI（具体点就是rudll32.exe cryptext.dll, CyrptExtOpenCER），进而弹出一个对话框来导入并/或查看证书内容。
- `.KEY` 扩展名KEY用于`PCSK#8`的公钥和私钥。这些公钥和私钥可以是DER编码或者PEM编码。

**CRT文件和CER文件只有在使用相同编码的时候才可以安全地相互替代。**

来自： [http://blog.sina.com.cn/s/blog_a9303fd90101jmtx.html](http://blog.sina.com.cn/s/blog_a9303fd90101jmtx.html)












## Read More

- [**如何制作CSR文件?**](https://help.aliyun.com/knowledge_detail/42218.html)
- 阿里云[证书服务](https://help.aliyun.com/product/28533.html)
-
- [快速获取免费SSL证书](https://zhuanlan.zhihu.com/p/30377917)
- [https://ssl.md](https://ssl.md)
-
- [Let's Encrypt，免费好用的 HTTPS 证书](https://imququ.com/post/letsencrypt-certificate.html)
- [acme.sh](https://github.com/Neilpang/acme.sh) 自动更新证书