[toc]

参考文档：tensorflow安装相关

https://www.cnblogs.com/ming-4/p/11516728.html



## 1, anaconda安装

官网安装即可



## 2, anaconda中python环境变量添加

> C:\Users\admin\Anaconda3\python.exe



## 3, 解决window10下输入python直接跳转到microsoft store的问题

删除环境变量中的这个变量：

```shell
%USERPROFILE%\AppData\Local\Microsoft\WindowsApps
```



## 4, anaconda中pip不能使用方法解决

* 默认使用pip的时候报错：

* ```python
  File "/usr/bin/pip3", line 11, in <module>
      sys.exit(main())
  TypeError: 'module' object is not callable
  ```

* 解决办法：

> ```sh
> python -m pip install --upgrade pip
> python -m pip install *
> ```



## 5, anaconda环境下安装tensorflow

```shell
python -m pip install tensorflow-gpu
```



## 6, 解决tensorflow-gpu下的问题

```python
Could not load dynamic library 'cudart64_100.dll'; dlerror: cudart64_100.dll not found
```

解决方案：https://www.joe0.com/2019/10/19/how-resolve-tensorflow-2-0-error-could-not-load-dynamic-library-cudart64_100-dll-dlerror-cudart64_100-dll-not-found/

具体办法：

我电脑没有navidia显卡，所以不支持gpu加速，只能写在之前的gpu版本，重新安装cpu版本。

如果是有navidia显卡的，可以按照上面链接中方案进行操作



## 7, mac上查看tensorflow的安装版本和安装路径

```python
import tensorflow as tf
tf.__version__  # version ID
tf.__path__      # installation path
```

