
# Hbase 基础操作

## Create 创建表
   create 'test_kail',{NAME=>'report'}
 
## Put 新增数据
   put 'test_kail','123456','report:simple','hehehahaheihei'
 
## Get 获取数据
get 'test_kail','123456'
get 'test_kail','123456','report'
get 'test_kail','123456','report:all'