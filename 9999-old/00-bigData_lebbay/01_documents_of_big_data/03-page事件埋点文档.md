[toc]

| 文件版本 | 修改人 | 修改内容     | 修改时间   |      |
| -------- | :----- | ------------ | ---------- | ---- |
| V1       | 张丹峰 | 文档初步拟定 | 2020-04-18 |      |
|          |        |              |            |      |
|          |        |              |            |      |



## 0, 现有问题

```shell
ip能不能取
国家能不能取(from_domain)
```





## 1, 文档目的

> 通过该埋点文档，使开发测试获悉如何进行数据埋点，使测试人员知晓埋点字段含义，使数据分析人员了解数据的相关结构从而进行后续数据分析



## 2, 请求数据示例

> 数据请求中的密钥请联系相关文档提供者进行获取



### 2.0, 在线文档

> 说明：该文档目前是在我本地起的一个服务，该服务有时候会关闭，如果打开不，请联系文档提供者进行重新启动

http://192.168.3.182:8080/swagger-ui.html

### 2.1, 请求方式

* post

### 2.2, 请求相关参数

```shell

```



## 3, 响应示例

### 3.1, 成功请求数据响应示例

```json
{
  "status": "ok",
  "msg": "success",
  "data": "ok"
}
```



### 3.2, 失败请求数据响应示例

```json
{
  "status": "error",
  "msg": "签名未通过，请检查",
  "data": null
}
```



## 4, 请求字段解释



### 4.1, 公共字段

* 公共字段参考文档：01-埋点日志通用字段.md



### 4.2, page事件字段



## 问题

```shell
匹配的类型。必须是“pageview”、“screenview”、“event”、“transaction”、“item”、“social”、“exception”、“timing”中的一个。

前一个页面是否要搜集？？？？
```



| 字段 | 类型   | 是否必填 | 参数含义                 | 备注                                                         | 默认值 |
| ---- | ------ | -------- | ------------------------ | ------------------------------------------------------------ | ------ |
| act  | string | true     | 动作                     | 1为view，也就是曝光事件，2是click，也就点击事件              |        |
|      |        |          |                          |                                                              |        |
| sid  | string | true     | session_id               | 网站有，移动端需生成，时效性由产品再定一下                   |        |
| sc   | string | false    | session control          | session控制，控制session时长，start为强制开始，end为强制结束，如果不填，则计算sid的最早最晚时间差 |        |
|      |        |          |                          |                                                              |        |
| dh   | string | true     | 文档主机名(这个应该不必要)   | https://www.azazie.com/                                      |        |
| dp   | string | true     | 当前页面路径             | /all/accessories/colors/cabernet?page=1&view_switch=smaller  |        |
| dt   | string | true     | 当前文档标题             | home, list, detail, addcart, pay, paysuccess                 |        |
|      |        |          |                          |                                                              |        |
| ldt  | double | false    | 加载时长,秒,保留两位小数 | 当act为view的时候才有需要填写                                |        |
|      |        |          |                          |                                                              |        |
| po | string | false    | screen num               | 如果是view，就是第几屏， 如果是点击，就是第几行等                           |        |
|      |        |          |                          |                                                              |        |
| param1 | string | false    | 拓展字段1                |                                                     |        |
| param2 | string | false    | 拓展字段2                |                                                     |        |
| param3 | string | false    | 拓展字段3                |                                                     |        |
| param4 | string | false    | 拓展字段4                |                                                     |        |
| param5 | string | false    | 拓展字段5                |                                                     |        |
| param6 | string | false    | 拓展字段6                |                                                     |        |
| param7 | string | false    | 拓展字段7                |                                                     |        |
| param8 | string | false    | 拓展字段8                |                                                     |        |














## 5, 签名方式

```java
/**
 * 签名生成方法, 方法如下：
 
 * 1，先把参数按照字典升序顺序排列,
 
 * 2，然后拼接参数为name=Ivanl001&age=10&hobby=game形式，
 
 * 3，将所有大写转小写,然后拼接密钥:act=1&am=appstore&c=us&cv=v1.0.1&di=dfakfdafdaslfasdfasfdasfasdfdsa&entry=1&ip=10.0.0.1&ldt=1.22&lgt=1506047606608&nw=wifi&p=1&pl=1&sr=1080*960&secretkey=me02kunyfqcejqbdwlaf0ghs3j2r8wuu
 
 * 4，把待编码字符串进行base64编码:YWN0PTEmYW09YXBwc3RvcmUmYz11cyZjdj12MS4wLjEmZGk9ZGZha2ZkYWZkYXNsZmFzZGZhc2ZkYXNmYXNkZmRzYSZlbnRyeT0xJmlwPTEwLjAuMC4xJmxkdD0xLjIyJmxndD0xNTA2MDQ3NjA2NjA4Jm53PXdpZmkmcD0xJnBsPTEmc3I9MTA4MCo5NjAmc2VjcmV0a2V5PW1lMDJrdW55ZnFjZWpxYmR3bGFmMGdoczNqMnI4d3V1
 
 * 5，将base64编码后的字符串进行md5加密:c34b30f0f77b63ef80f1cc3b841b3a89
 */

//服务端加密源码参考：com.lebbay.common.utils.IMSignUtils
```