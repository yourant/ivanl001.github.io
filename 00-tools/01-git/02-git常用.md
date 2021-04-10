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
#  方法1，push的时候需要单独push
git remote add gitlab https://gitlab.com/bool_learning/learning_notebook.git

#  方法2, push的时候只需要push一次就行
git remote set-url --add origin https://gitlab.com/bool_learning/learning_notebook.git


# 推送的时候
# 查看远程仓库
git remote
# 查看远程仓库的具体内容
git remote -v 

git push -u origin
git push -u gitlab

# 这个命令用于删除现有的git远程仓库
git remote remove gitlab


git push -u origin gitlab
```



> 从下面remote -v命令可以看出, origin 是默认仓库，如果不指定，会使用这个
>
> git remote set-url --add *是添加默认origin仓库。因为我这里先设置了
>
> git remote add origin https://github.com/ivanl001/ivanl001.github.io.git
>
> 然后设置了：
>
> git remote set-url --add origin https://gitlab.com/bool_learning/learning_notebook.git
>
> 所以origin可以同时push到两个仓库

```shell
$ git remote -v
github	https://github.com/ivanl001/ivanl001.github.io.git (fetch)
github	https://github.com/ivanl001/ivanl001.github.io.git (push)
gitlab	https://gitlab.com/bool_learning/learning_notebook.git (fetch)
gitlab	https://gitlab.com/bool_learning/learning_notebook.git (push)
origin	https://github.com/ivanl001/ivanl001.github.io.git (fetch)
origin	https://github.com/ivanl001/ivanl001.github.io.git (push)
origin	https://gitlab.com/bool_learning/learning_notebook.git (push)
```



