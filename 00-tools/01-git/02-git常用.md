[toc]

<extoc></extoc>



## git初始化

```shell
git init 

#  添加推送到远程的地址
git remote add gitee https://gitee.com/ivanl001/nebulas_doc.git

git add .
git commit -m 'init'

# git push gitee
git push --set-upsteam gitee master

#  从当前分支直接新建分支并切换到新分支上
git checkout -b pages
# 推送，并设置推送的分支
git push --set-upstream gitee pages
```

