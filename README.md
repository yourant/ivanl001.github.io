[toc]

## github-page + gitbook + travis 自动构建blog手册



> 参考文档：
>
> 开启github page功能： https://pages.github.com/
>
> gitbook+github page搭建：https://github.com/riskers/blog/issues/48
>
> gitbook搭建及插件学习等：https://snowdreams1006.github.io/



### 1, 创建仓库并开启page功能

> https://pages.github.com/



### 2, 初始化项目

> 注意：以下配置文件需要根据自己的github的项目做相关部分内容的更改

#### 2.1, clone出项目

```shell
git clone https://github.com/ivanl001/ivanl001.github.io.git
```

#### 2.2, 在项目中添加配置文件等



##### .gitignore

```shell
# 忽略本地测试的node模块代码
node_modules
#  忽略gitbook本地测试生成的代码
_book

# mac忽略文件
.DS_Store
```



##### README.md

> > 必要的README.md文件, 这个里面是对项目的描述等内容

```shell
vim README.md


## README.md
### 这里是项目的说明文档,内容自定义
```

##### SUMMARY.md

> > SUMMARY.md : 这个是gitbook的目录文档，通过这个文件进行目录编写

```shell
vim SUMMARY.md

* [简介](README.md)
```

##### book.json

> >book.json : 这个是gitbook的配置文件，在里面配置gitbook的基本配置及插件等
> >
> >mygitalk中配置了proxy，因为默认的gitalk中使用的代理不能用，使用的时候会报错。

```shell
vim book.json

{
    "title": "不二技术博客",
    "author": "不二",
    "description": "不二 搭建的 Gitbook 个人博客",
    "language": "zh-hans",
    "gitbook": "3.2.3",
    "links": {
        "sidebar": {
            "github": "https://github.com/ivanl001",
	        "gitlab":"https://gitlab.com/ivanl001",
	        "gitee":"https://gitee.com/ivanl001"
        }
    },
    "plugins": [
        "toc",
        "-lunr",
        "-search",
        "search-plus",
        "splitter",
        "-sharing",
        "sharing-plus",
        "expandable-chapters-small",
        "anchor-navigation-ex",
        "edit-link-plus",
        "code",
        "chart",
        "favicon-absolute",
        "github-buttons",
        "advanced-emoji",
        "sitemap-general",
        "copyright",
        "tbfed-pagefooter",
        "mygitalk",
        "donate",
        "icp",
        "diff",
        "simple-mind-map",
        "hide-element",
        "audio_image",
        "mermaid-gb3",
        "baidu-tongji-with-multiple-channel",
        "google-tongji-with-multiple-channel"
    ],


    "pluginsConfig": {

        "donate": {
            "wechat": "./pic/wechatpay.jpg",
            "alipay": "./pic/alipay.jpg",
            "title": "",
            "button": "赏",
            "alipayText": "支付宝",
            "wechatText": "微信"
        },

        "favicon-absolute": {
            "favicon": "./pic/favicon.ico",
            "appleTouchIconPrecomposed152": "./pic/apple-touch-icon-precomposed-152.png"
        },

        "toc": {
            "addClass": true,
            "className": "toc"
        },


        "copyright": {
            "site": "https://ivanl001.github.io",
            "author": "不二",
            "website": "不二技术博客",
            "image": "",
            "copyProtect": true
        },

        "github-buttons": {
            "buttons": [{
                "user": "ivanl001",
                "repo": "ivanl001.github.io.git",
                "type": "star",
                "size": "small"
            }]
        },

        "tbfed-pagefooter": {
            "copyright": "&copy 不二",
            "modify_label": "文件修订时间: ",
            "modify_format": "YYYY-MM-DD HH:mm:ss"
        },

        "sharing": {
            "douban": true,
            "facebook": false,
            "google": false,
            "hatenaBookmark": false,
            "instapaper": false,
            "line": false,
            "linkedin": false,
            "messenger": false,
            "pocket": false,
            "qq": true,
            "qzone": true,
            "stumbleupon": false,
            "twitter": false,
            "viber": false,
            "vk": false,
            "weibo": true,
            "whatsapp": false,
            "all": [
                "facebook", "google", "twitter",
                "weibo", "instapaper", "linkedin",
                "pocket", "stumbleupon"
            ]
        },


        "edit-link-plus": {
            "base": {
              "ivanl001.github.io":"https://github.com/ivanl001/ivanl001.github.io/edit/master",
              "ivanl001.gitlab.io":"https://gitlab.com/ivanl001/ivanl001.gitlab.io/edit/master",
              "ivanl001.gitee.io":""
            },
            "defaultBase": "https://github.com/ivanl001/ivanl001.github.io/edit/master",
            "label": "编辑本页"
        },

        "chart": {
            "type": "c3"
        },


        "sitemap-general": {
            "prefix": "https://ivanl001.github.io/"
        },

        
        "mygitalk": {
            "clientID": "3d8b5499512dfc769dcc",
            "clientSecret": "72387e663a746fd79685ed71a74a99e0c27ba530",
            "repo": "ivanl001.github.io",
            "owner": "ivanl001",
            "admin": ["ivanl001"],
            "distractionFreeMode": false,
            "proxy": "https://shielded-brushlands-08810.herokuapp.com/https://github.com/login/oauth/access_token"

        },


        "icp": {
            "number": "沪ICP备00000000号(无)",
            "link": "http://www.beian.gov.cn/"
        },


        "hide-element": {
            "elements": ["a.gitbook-link[href='https://www.gitbook.com']"]
        },


        "simple-mind-map":{
            "preset": "colorful"
        }
    }
}

```

##### .travis.yml

> > .travis.yml : travis的自动部署配置文件

```shell
vim .travis.yml

language: node_js
node_js:
  - "10"
cache: npm

notifications:
  email:
    recipients:
      - ivanl001@163.com # 设置通知邮件
    on_success: change
    on_failure: always

install:
  - npm install -g gitbook-cli
  - gitbook install

script:
  - gitbook build

after_script:
  - cd _book
  - git init
  - git remote add origin https://${REF}
  - git add .
  - git commit -m "Updated By Travis-CI With Build $TRAVIS_BUILD_NUMBER For Github Pages"
  - git push --force --quiet "https://${TOKEN}@${REF}" master:pages

branches:
  only:
    - master

env:
  global:
    - REF=github.com/ivanl001/ivanl001.github.io.git # 设置 github 地址

```

### 3, 项目提交并准备

#### 3.1, 提交项目到master分支

```shell
git add .
git commit -m 'github page + gitbook + travis项目初始化'
git push
```

#### 3.2, 创建pages分支提交gitbook生成静态网页

```shell
# 当前分支生成pages分支
git checkout -b pages
git add .
git commit -m 'branch page created'
git push --set-upstream origin pages
```

#### 3.3,  github配置pages分支为github page分支

![](https://learningnotebookv1-1302566743.cos.ap-nanjing.myqcloud.com/img/20210409112717.png)

### 4,  git配置token及gitalk的密钥

#### 4.1, token(给travis提交代码用)

![image-20210409113100037](https://learningnotebookv1-1302566743.cos.ap-nanjing.myqcloud.com/img/image-20210409113100037.png)

#### 4.2, client_id 及secret key(给gitalk用)

![image-20210409113418324](https://learningnotebookv1-1302566743.cos.ap-nanjing.myqcloud.com/img/image-20210409113418324.png)



### 5, 最后配置

https://travis-ci.com/

#### 5.1, 配置git的token方便提交代码到github pages分支

![image-20210409113727887](https://learningnotebookv1-1302566743.cos.ap-nanjing.myqcloud.com/img/image-20210409113727887.png)



#### 5.2, 回到book.json中修改mygitalk的配置client_id和secret key

```shell
"mygitalk": {
            "clientID": "3d8b5499512dfc769dcc",
            "clientSecret": "72387e663a746fd79685ed71a74a99e0c27ba530",
            "repo": "ivanl001.github.io",
            "owner": "ivanl001",
            "admin": ["ivanl001"],
            "distractionFreeMode": false,
            "proxy": "https://shielded-brushlands-08810.herokuapp.com/https://github.com/login/oauth/access_token"

        }
```



### 6, 去travis中重新build

> 查看日志，如无问题，则代表ok 

进入页面验证是否ok:

https://ivanl001.github.io/



### 7, 完成

下次提交代码, travis中会自动构建并提交静态网页到pages分支

