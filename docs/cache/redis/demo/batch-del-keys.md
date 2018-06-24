
# 批量删除 Key

## 遍历文件中的 Keys 进行删除

``` bash
# 如果你 Key 是在Windows 下整理的，需要先删除每一行尾的 \r 字符，
# dos2unix keys.txt
# dos2unix -n keys.txt  keys_no_r.txt
# cat keys.txt | tr -d '\r' > keys_no_r.txt

# 
for line in `cat keys_no_r.txt` ; do redis-cli del $line; done
```

## 一次删除文件中的多个键
``` bash
# 10个10个的删除
xargs -a keys.txt | xargs -n10 redis-cli del
```

## `SCAN` 删除
``` bash
redis-cli --scan --pattern "*" | xargs -n 2 redis-cli del
```

## `KEYS *` 删除
**不建议使用**
``` bash
redis-cli keys \* | xargs redis-cli del
```

## Read More
[超大批量删除redis中无用key](http://blog.csdn.net/mengxianhua/article/details/51076165)