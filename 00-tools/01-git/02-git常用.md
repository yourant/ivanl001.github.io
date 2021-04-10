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



## git同时推送到多个仓库

```shell
# 我仓库中默认是github, 为original, 这里再添加gitlab仓库
git remote add gitlab https://gitlab.com/bool_learning/learning_notebook.git

# 推送的时候
# 查看远程仓库
git remote
# 查看远程仓库的具体内容
git remote -v 

git push -u origin
git push -u gitlab


git push -u origin master
```

