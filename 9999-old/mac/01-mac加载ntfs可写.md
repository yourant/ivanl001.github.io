> 简单来讲，就是把移动硬盘挂在到已有文件上即可
>
> big sur 可用

```shell
# 查看自己磁盘名称
ls -l /Volumes/

# 找到自己的磁盘后，比如我的是“张不二-2T”，通过diskutil查看该盘的Device Node
diskutil info /Volumes/张不二-2T

# 通过上命令，查到磁盘的Device Node是：Device Node:  /dev/disk2s2
# 先退出磁盘
hdiutil eject /Volumes/张不二-2T

# 创建一个用于挂在的目录(如果已存在该目录，可以不创建)
mkdir /Users/ivanl001/myMobileDisk

# 通过命令挂在即可
sudo mount_ntfs -o rw,nobrowse /dev/disk2s2 /Users/ivanl001/myMobileDisk
```



