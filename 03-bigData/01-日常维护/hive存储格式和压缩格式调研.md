##### 主流

使用**ORC+snappy**组合！
使用**Parquet+LZO**组合！



```shell
# 调研结果：
# orc存储，相对parquet，orc压缩比更大。
# SNAPPY压缩，原因是orc存储文件默认采用ZLIB压缩。比snappy压缩的小
STORED AS orc tblproperties ("orc.compress"="SNAPPY");
```

