[toc]



## 1, 初期计算指标

>  以下指标目前全部以设备为准，一台独立的设备为一个用户，除了订单等跟具体用户id关联的数据外，其他非用户id相关数据均用设备id当作独立用户



日活，周活，月活

新增设备(每日), --------活跃表中中存在新增表中不存在即是新增用户

每日留存用户，留存率--------1, 2, 3, 4, 7, 14, 30-------活跃用户/新增用户

每周留存用户(不是日留存直接相加，是会去重)



沉默用户

本周回流用户(上周之前活跃过，上周未活跃，本周重新活跃)



流失用户(最近7日无活跃的用户)



连续3周活跃用户

最近7日连续3天活跃用户



gat 



device_imei

idfa  -----  ----- 

(idfv)







Appevent

1，app事件

同一用户需要在该事件范围内提供一个相同的event_id

打开

进入后台

关闭



-活跃设备，活跃用户，用户时长



Pageevent    



2，页面曝光事件pageview及

组件曝光事件

-需要同时提供页面标签，页面url和上一页面标签及上一页面url



同一用户需要有相同的sessionid来计算用户访问轨迹及访问深度





首页算一个页面

列表页全一个页面

详情页每个商品为一个页面

加车页

加车成功页

付款页

付款成功页



pv uv 时长 

人均点击次数

bounceRate()

退出率







-页面Uv pv 



组件曝光事件



(页面时长)------解决办法



3，用户点击事件pageclick



4，推送事件



5, 错误日志



页面加载开始？加载结束？？加载失败？？？



问题1:比如进入首页，那么会有一个pathview事件和若干组件曝光事件，是否批量发送？





渠道？？？







页面

channel， source， medium

download， open



{
	"pdl": 1,
	"pf":  1,
	"cv": "1.0.0",
	"di": "ab31343ddfasdf843djkfda",
	"am": "appStore",
	"dm": "iphone 6s",
	"ov": "7s",
	"dr": "1920*1080",
	"ln": 0.00,
	"la": 0.00,
	"lt": "2017-06-01 12:21:13",
	"ip": "127.0.0.1",

}







pl int product_line 产品线, 1代表azazie，2代表blushmark，3代表ppmianliao
p  int platform 平台, 1代表pc, 2代表mweb, 3代表ios， 4代表android 

cv string client_version 客户端版本号
di string device_imei 设备唯一标识
am string app_market 下载的市场
dm string device_model 设备模型，如手机型号，浏览器类型等
ov string os_version 系统版本
dr string device_resolution 设备分辨率
ln double lng经度
la double lat 纬度
lt string log_time, 时间





/*
1:iOS
2:Android
3:M站(Html5)
4:WAP
5:PC
6:TV
7:Web
8:Mac
9:WP
*/

Zhangdf_123









v=1&
ds=client&
tid=UA-153534498-2&
cid=00000000-0000-0000-0000-000000000000&
dh=https%3A%2F%2Fapi-p.blushmark.com%2F&
dp=home&
geoid=US&
uid=743035fe365799d0215c15e3c3e15a0d&
sr=812%C3%97375&
ua=com.lebbay.blushmark%2F1.1.1%20(x86_64%3B%20iOS%2013.4%3B%20Scale%2F3&
utc=home&
utv=app_request_track&
t=timing&
utt=3187&
utl=https%3A%2F%2Fapi-p.blushmark.com%2Fv1%2Fproducts%2FmostPopular%2Flist



v=1&
ds=client&
tid=UA-153534498-2&
cid=00000000-0000-0000-0000-000000000000&
dh=https%3A%2F%2Fapi-p.blushmark.com%2F&
dp=goods%2Flist&
geoid=US&
uid=743035fe365799d0215c15e3c3e15a0d&
sr=812%C3%97375&
ua=com.lebbay.blushmark%2F1.1.1%20(x86_64%3B%20iOS%2013.4%3B%20Scale%2F3&
ec=ListPage_Pettie_View&
ea=event&
t=event&
el=3_23


//
v=1&
ds=client&
tid=UA-153534498-2&
cid=00000000-0000-0000-0000-000000000000&
dh=https%3A%2F%2Fapi-p.blushmark.com%2F&
dp=home&
geoid=US&
uid=743035fe365799d0215c15e3c3e15a0d&
sr=812%C3%97375&
ua=com.lebbay.blushmark%2F1.1.1%20(x86_64%3B%20iOS%2013.4%3B%20Scale%2F3&
utc=home&
utv=app_request_track&
t=timing&
utt=3187&
utl=https%3A%2F%2Fapi-p.blushmark.com%2Fv1%2Fproducts%2FmostPopular%2Flist