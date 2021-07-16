[toc]



## 1, 安装node

```shell
# node安装：官网下载直接双击安装即可
# centos上使用wget获取需要的版本即可

# 验证
node -v
```





## 2, 安装vue

```shell
sudo npm install -g @vue/cli
sudo npm uninstall -g @vue/cli

vue -V
```



## 3, npm换成淘宝镜像

```shell
npm config get registry

npm config set registry https://registry.npm.taobao.org

npm config get registry
```







## 4, 创建项目

```shell
vue ui
```









lint使用：

```shell
npm run lint
```



