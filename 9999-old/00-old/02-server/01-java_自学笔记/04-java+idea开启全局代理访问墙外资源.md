```java
// 直接添加jvm全局环境变量即可
//开发环境下需要添加代理免得后面访问不了google analytics
System.setProperty("socksProxyHost", "192.168.2.66");
System.setProperty("socksProxyPort", "9006");
```

