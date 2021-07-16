[toc]

# 第一种方式(在部分情况下不好使用)

## maven依赖

```xml
<!--javaMail 发邮件-->
<dependency>
    <groupId>javax.mail</groupId>
    <artifactId>javax.mail-api</artifactId>
    <version>1.5.6</version>
</dependency>
<dependency>
    <groupId>com.sun.mail</groupId>
    <artifactId>javax.mail</artifactId>
    <version>1.5.3</version>
</dependency>
```





## MailUtils工具类

```java
package azazie.com.azazieoptimizerjava.utils;

import azazie.com.azazieoptimizerjava.model.common.IMResultModel;

import javax.mail.*;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;
import java.util.Properties;

/**
 * #Author : admin
 * #Date   : 10/15/2019-8:56 AM
 * #Desc   : 发邮件工具类, 网络上抄的，可以正常使用
 **/
public class MailUtils {

    private static final String USER = "ivanl001@163.com"; // 发件人称号，同邮箱地址
    private static final String PASSWORD = "********"; // 如果是qq邮箱可以使户端授权码，或者登录密码

    /**
     * @param to    收件人邮箱
     * @param text  邮件正文
     * @param title 标题
     */
    /* 发送验证信息的邮件 */
    public static IMResultModel sendMail(String to, String text, String title) {
        IMResultModel resultModel = new IMResultModel();
        resultModel.setErrorResult("初始化");

        try {
            final Properties props = new Properties();
            props.put("mail.smtp.auth", "true");
            props.put("mail.smtp.host", "smtp.163.com");

            // 发件人的账号
            props.put("mail.user", USER);
            //发件人的密码
            props.put("mail.password", PASSWORD);

            // 构建授权信息，用于进行SMTP进行身份验证
            Authenticator authenticator = new Authenticator() {
                @Override
                protected PasswordAuthentication getPasswordAuthentication() {
                    // 用户名、密码
                    String userName = props.getProperty("mail.user");
                    String password = props.getProperty("mail.password");
                    return new PasswordAuthentication(userName, password);
                }
            };
            // 使用环境属性和授权信息，创建邮件会话
            Session mailSession = Session.getInstance(props, authenticator);
            // 创建邮件消息
            MimeMessage message = new MimeMessage(mailSession);
            // 设置发件人
            String username = props.getProperty("mail.user");
            InternetAddress form = new InternetAddress(username);
            message.setFrom(form);

            // 设置收件人
            InternetAddress toAddress = new InternetAddress(to);
            message.setRecipient(Message.RecipientType.TO, toAddress);

            // 设置邮件标题
            message.setSubject(title);

            // 设置邮件的内容体
            message.setContent(text, "text/html;charset=UTF-8");
            // 发送邮件
            Transport.send(message);

            resultModel.setOKResult("消息发送成功");
        } catch (Exception e) {
            resultModel.setErrorResult(e.getMessage());
        }
        return resultModel;
    }
}
```





## 测试使用

```java
// 做测试用
@Test
public void mailSendTest() {
    IMResultModel resultModel = MailUtils.sendMail("dfzhang@i9i8.com", "你好，这是一封测试邮件，无需回复。", "测试邮件");
    if (resultModel.getStatus().equals("ok")) {
        System.out.println("发送成功");
    }
}
```





# 第二种方式(springboot)

## 2.1, maven依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```



## 2.2, 代码中使用

```java
//这里其实没错，问题不大，误报
@Autowired
private JavaMailSender mailSender;

//发送邮件
public void sendEmail() {
	SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
    // 设置收件人，寄件人
    simpleMailMessage.setTo(new String[]{"dfzhang@i9i8.com"});
    simpleMailMessage.setFrom("ivanl001@163.com");
    simpleMailMessage.setSubject("job重试2*3次依然失败：" + ":\n" + e);
    simpleMailMessage.setText("报表系统失败邮件：" + e.getMessage());
    // 发送邮件
    mailSender.send(simpleMailMessage);
    log.info("失败邮件已发送...");     
}
```

