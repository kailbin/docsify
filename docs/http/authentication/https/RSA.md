
# RSA

- 对极大整数做**因数分解**的难度决定了RSA算法的可靠性

## 非对称加密算法

- 乙方生成两把密钥（公钥和私钥）。公钥是公开的，任何人都可以获得，私钥则是保密的。
- 甲方获取乙方的公钥，然后用它对信息加密。
- 乙方得到加密后的信息，用私钥解密。
- **公钥可以解密私钥加密的数据**，**私有也可以解密公钥加密的数据**

## 在线测试 公钥/私钥 的 加密/解密

- [在线生成密钥对](http://www.ssleye.com/web/pass_double)
- [在线 RSA 公钥加密解密](http://tool.chacuo.net/cryptrsapubkey)
- [在线  RSA 私钥加密解密](http://tool.chacuo.net/cryptrsaprikey)


## Read More

- 阮一峰的网络日志 [RSA算法原理（一）](http://www.ruanyifeng.com/blog/2013/06/rsa_algorithm_part_one.html)
- 阮一峰的网络日志 [RSA算法原理（二）](http://www.ruanyifeng.com/blog/2013/07/rsa_algorithm_part_two.html)