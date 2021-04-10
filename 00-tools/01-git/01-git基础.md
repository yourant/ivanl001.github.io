[toc]



### rebase是做什么的？



### 命令行merge分支？







### git版本回退

#### 查看commit日志

```shell
git log --pretty=oneline
```



#### 本地仓库往前回退一个版本

```shell
git reset --hard HEAD^
```



#### 本地仓库回退到指定版本

```shell
git reset --hard 51ece7a6074122fe17e9d7d08ffdb97f2678b461
```



#### 如果想要重新回到后续版本

```shell
 git reflog
 git reset --hard ****
```



#### 总结

```shell
现在总结一下：

HEAD指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令git reset --hard commit_id。
穿梭前，用git log可以查看提交历史，以便确定要回退到哪个版本。
要重返未来，用git reflog查看命令历史，以便确定要回到未来的哪个版本。
```



#### 另外git远程仓库回退

```shell
#  找到版本号
git log --pretty=oneline

# 本地回退到该版本
git reset --hard ***

# 强制回退远程仓库到当前版本
git push -f
```



### git 解决冲突

#### 1，git push的时候发现冲突

```shell
# git 如果在push的时候发现push不上去，先拉取一次
git pull

# 拉取之后会有提示，说文件有冲突， 通过status命令查看有冲突的文件
git status

# 然后vim该文件进行修改即可

# 修改完成后,重新把文件进行添加到暂存区，并提交即可
git add .
git commit -m 'merge finished'
git push

```

#### 2, git pull拉取的时候发现有冲突

这个和上面的区别是，先commit, 再pull会支流直流并合并

但是先stash然后pull，再然后push，就不会创建支流。

> 写代码的时候，还未提交，需要拉取远程代码，发现拉取不下来，有冲突

```shell
# 先把当前写的代码stash一下
git add .
git stash

# 再pull的时候就可以了
git pull

# 拉取下来后本地文件变成远程文件了，需要使用git stash pop把存储的数据弹出
git stash list
git stash pop

# 然后会有提示冲突，解决冲突即可
# 解决冲突

git add .
git commit -m 'finished(no merge)'
git push
```





### git的stash的作用

参考文档：

https://blog.csdn.net/ForMyQianDuan/article/details/78750434

https://blog.csdn.net/kindergarten_sir/article/details/109829115



`git stash ----将当前工作区和暂存区进度保存`

> 1, 当你正在当前分支修改代码时候，如果当前分支别人修改了东西并提交到远程，但是需要你放弃自己写的东西，调整一下远程上对方的代码的时候，就可以通过stash把自己的代码保存起来，然后pull对方代码，直接调整，push出去的时候，自己写的代码不会push出去。
>
> 然后再通过stash pop再恢复自己的代码，接着写。
>
> stash相当于是临时存储自己的临时代码，push到远程的操作不会push当前自己的临时代码
>
> 2, 当你在一个分支上创建一个文件，如果直接切到其他分支上，该文件依然会存在。如果stash后就可以避免这种问题





git stash 保存当前工作区和暂存区进度（默认只会保存加入到版本管理的文件，即未被追踪的文件不会存储）
git stash 需要在git add之前执行(这个在命令行的时候好像需要先add才能stash有效)

* 保存

  ```sql
  git stash ----将当前工作区和暂存区进度保存
  ```

* 查看保存列表

  ```sql
  git stash list ----查看所有保存的记录列表，记录列表前有{序号}可用于删除或取出。
  ```

  

* 不可重复恢复

  ```sql
  git stash pop stash@{序号} ----恢复，{序号}是可选项，不选会全部恢复。恢复后会从git stash list中删除掉被恢复的项。
  ```

  

* 可重复恢复

  ```sql
  git stash apply stash@{序号} ----恢复，{序号}是可选项，不选会全部恢复。恢复后不会从git stash list中删除掉被恢复的项。
  ```

  

* 删除某项

  ```sql
  git stash drop stash@{序号} ----删除git stash list中的某项保存，{序号}是可选项。
  ```

  

* 删除所有

  ```sql
  git stash clear ----删除git stash list中的所有保存。
  ```

  

### git切换分支

#### 查看远程分支

```shell
git branch -a
```



#### 查看本地分支

```shell
git branch
```



切换分支

```shell
git checkout dev
```



### tag的作用

```shell
tag就像是一个里程碑一个标志一个点，branch是一个新的征程一条线；
tag是静态的，branch要向前走；
稳定版本备份用tag，新功能多人开发用branch（开发完成后merge到master）。
```