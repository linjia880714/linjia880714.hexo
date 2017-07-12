---
title: Javamail使用gmail发送邮件
date: 2017-06-16 14:32:32
tags: javamail
categories:
- IT
- java
- other
---


代码先贴上,代码参考[https://gist.github.com/brunocesarsilva/12a529f7f752f2853b9f](https://gist.github.com/brunocesarsilva/12a529f7f752f2853b9f)

```xml
<dependency>
    <groupId>javax.mail</groupId>
    <artifactId>javax.mail-api</artifactId>
    <version>1.5.1</version>
</dependency>

<dependency>
    <groupId>com.sun.mail</groupId>
    <artifactId>javax.mail</artifactId>
    <version>1.5.5</version>
</dependency>
```

```java
import javax.mail.*;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;
import java.util.Date;
import java.util.Properties;
import java.util.logging.Level;
import java.util.logging.Logger;

public class SendEmailGmail {

    private static final Logger LOGGER = Logger.getAnonymousLogger();

    private static final String SERVIDOR_SMTP = "smtp.gmail.com";
    private static final int PORTA_SERVIDOR_SMTP = 587;
    private static final String CONTA_PADRAO = "xxxx@gmail.com";
    private static final String SENHA_CONTA_PADRAO = "xxxx";

    private final String from = "xxxx@gmail.com";
    private final String to = "xxxx@sleepace.net";

    private final String subject = "Teste";
    private final String messageContent = "Teste de Mensagem";

    public void sendEmail() {
        final Session session = Session.getInstance(this.getEmailProperties(), new Authenticator() {

            @Override
            protected PasswordAuthentication getPasswordAuthentication() {
                return new PasswordAuthentication(CONTA_PADRAO, SENHA_CONTA_PADRAO);
            }

        });

        try {
            final Message message = new MimeMessage(session);
            message.setRecipient(Message.RecipientType.TO, new InternetAddress(to));
            message.setFrom(new InternetAddress(from));
            message.setSubject(subject);
            message.setText(messageContent);
            message.setSentDate(new Date());
            Transport.send(message);
        } catch (final MessagingException ex) {
            LOGGER.log(Level.WARNING, "Erro ao enviar mensagem: " + ex.getMessage(), ex);
        }
    }

    public Properties getEmailProperties() {
        final Properties config = new Properties();
        config.put("mail.smtp.auth", "true");
        config.put("mail.smtp.starttls.enable", "true");
        config.put("mail.smtp.host", SERVIDOR_SMTP);
        config.put("mail.smtp.port", PORTA_SERVIDOR_SMTP);
        //这个参数会把和邮箱服务器的交互过程都打印出来
        config.put("mail.debug", "true");
        return config;
    }

    public static void main(final String[] args) {
        new SendEmailGmail().sendEmail();
    }

}
```

报错
```
javax.mail.AuthenticationFailedException: 535-5.7.8 Username and Password not accepted. Learn more at
535 5.7.8  https://support.google.com/mail/?p=BadCredentials y8sm3034037pge.0 - gsmtp
	at com.sun.mail.smtp.SMTPTransport$Authenticator.authenticate(SMTPTransport.java:826)
	at com.sun.mail.smtp.SMTPTransport.authenticate(SMTPTransport.java:761)
	at com.sun.mail.smtp.SMTPTransport.protocolConnect(SMTPTransport.java:685)
	at javax.mail.Service.connect(Service.java:317)
	at javax.mail.Service.connect(Service.java:176)
	at javax.mail.Service.connect(Service.java:125)
	at javax.mail.Transport.send0(Transport.java:194)
	at javax.mail.Transport.send(Transport.java:124)
	at office365.SendEmailGmail.sendEmail(SendEmailGmail.java:43)
	at office365.SendEmailGmail.main(SendEmailGmail.java:59)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:147)
```

你会发现你输入了正确的用户名和密码还是报错。

那是因为google开启了两步验证功能，所以如果要使用javamail发送邮件的话，需要一个应用专用密码（app password），这样即使密码被盗取也无法登录到账号

![](Javamail使用gmail发送邮件.md/01.png)
![](Javamail使用gmail发送邮件.md/02.png)
![](Javamail使用gmail发送邮件.md/03.png)
谷歌的官方文档[Sign in using App Passwords](https://support.google.com/accounts/answer/185833?hl=en&ctx=ch_DisplayUnlockCaptcha)

__注意注意: office365也有个小坑，如果邮箱地址有大写，要全部小写才能发送成功，不然会一直提示授权失败，其他邮箱没有测试过，如果一直授权失败应该就是大小写的原因__