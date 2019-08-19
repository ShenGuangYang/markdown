# SpringBoot 2.x 集成mail



## 思路

我们要做的其实就是把Java程序作为一个客户端，然后通过配置SMTP协议去连接我们所使用的发送邮箱（from）对应的SMTP服务器，然后通过SMTP协议，将邮件转投到目标邮箱（to）对应的SMTP服务器，最后将该邮件分发到目标邮箱



## 引入POM

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```



## 配置文件

运营商对应的host

|     运营商     |        host        | 授权码 |
| :------------: | :----------------: | ------ |
|   个人QQ邮箱   |    smtp.qq.com     | 生成   |
|    163邮箱     |    smtp.163.com    | 密码   |
|  腾讯企业邮箱  | smtp.exmail.qq.com | 密码   |
| 阿里的企业邮箱 | smtp.mxhichina.com | 密码   |


```properties
# 这里的host对应是上面的几大运营商的STMP服务器的host
spring.mail.host=smtp.163.com
spring.mail.username=****@163.com
# 这里的password对应的就是上面的授权码
spring.mail.password=*** 
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
spring.mail.properties.mail.smtp.starttls.required=true
spring.mail.default-encoding=UTF-8
```



## 代码

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.Map;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class MailEntity {

    /**
     * 邮件发送人
     */
    private String from;

    /**
     * 邮件接收人
     */
    private String to;

    /**
     * 邮件主题
     */
    private String subject;

    /**
     * 邮件内容
     */
    private String content;

    /**
     * 邮件主题
     */
    private String type;

    /**
     * 发送邮件模板时的模板文件名
     */
    private String templateName;

    /**
     * 模板参数
     */
    private Map<String,Object> variables;

    /**
     * 附件地址
     */
    private String attachPath;

}
```





```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.FileSystemResource;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.stereotype.Component;

import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;
import java.io.File;

@Component
public class IMailServiceImpl implements IMailService {

    @Autowired
    private JavaMailSender mailSender;

    @Value("${spring.mail.from}")
    private String from;

    /**
     * 发送文本邮件
     *
     * @param to
     * @param subject
     * @param content
     */
    @Override
    public void sendSimpleMail(String to, String subject, String content) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(from);
        message.setTo(to);
        message.setSubject(subject);
        message.setText(content);
        mailSender.send(message);
    }

    @Override
    public void sendSimpleMail(String to, String subject, String content, String... cc) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(from);
        message.setTo(to);
        message.setCc(cc);
        message.setSubject(subject);
        message.setText(content);
        mailSender.send(message);
    }

    /**
     * 发送HTML邮件
     * @param to
     * @param subject
     * @param content
     */
    @Override
    public void sendHtmlMail(String to, String subject, String content) throws MessagingException {
        MimeMessage message = mailSender.createMimeMessage();

        //true表示需要创建一个multipart message
        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        helper.setFrom(from);
        helper.setTo(to);
        helper.setSubject(subject);
        helper.setText(content, true);

        mailSender.send(message);
    }

    @Override
    public void sendHtmlMail(String to, String subject, String content, String... cc) {

    }


    /**
     * 发送带附件的邮件
     * @param to
     * @param subject
     * @param content
     * @param filePath
     */
    public void sendAttachmentsMail(String to, String subject, String content, String filePath) throws MessagingException {
        MimeMessage message = mailSender.createMimeMessage();

        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        helper.setFrom(from);
        helper.setTo(to);
        helper.setSubject(subject);
        helper.setText(content, true);

        FileSystemResource file = new FileSystemResource(new File(filePath));
        String fileName = filePath.substring(filePath.lastIndexOf(File.separator));
        helper.addAttachment(fileName, file);

        mailSender.send(message);
    }

    @Override
    public void sendAttachmentsMail(String to, String subject, String content, String filePath, String... cc) {

    }


    /**
     * 发送正文中有静态资源（图片）的邮件
     *
     * @param to
     * @param subject
     * @param content
     * @param rscPath
     * @param rscId
     */
    public void sendResourceMail(String to, String subject, String content, String rscPath, String rscId) throws MessagingException {
        MimeMessage message = mailSender.createMimeMessage();

        MimeMessageHelper helper = new MimeMessageHelper(message, true);
        helper.setFrom(from);
        helper.setTo(to);
        helper.setSubject(subject);
        helper.setText(content, true);

        FileSystemResource res = new FileSystemResource(new File(rscPath));
        helper.addInline(rscId, res);

        mailSender.send(message);
    }

    @Override
    public void sendResourceMail(String to, String subject, String content, String rscPath, String rscId, String... cc) {

    }
}
```



```java
import javax.mail.MessagingException;

public interface IMailService {
    /**
     * 发送文本邮件
     * @param to
     * @param subject
     * @param content
     */
    public void sendSimpleMail(String to, String subject, String content);

    public void sendSimpleMail(String to, String subject, String content, String... cc);

    /**
     * 发送HTML邮件
     * @param to
     * @param subject
     * @param content
     * @throws MessagingException
     */
    public void sendHtmlMail(String to, String subject, String content) throws MessagingException;

    public void sendHtmlMail(String to, String subject, String content, String... cc);

    /**
     * 发送带附件的邮件
     * @param to
     * @param subject
     * @param content
     * @param filePath
     * @throws MessagingException
     */
    public void sendAttachmentsMail(String to, String subject, String content, String filePath) throws MessagingException;

    public void sendAttachmentsMail(String to, String subject, String content, String filePath, String... cc);

    /**
     * 发送正文中有静态资源的邮件
     * @param to
     * @param subject
     * @param content
     * @param rscPath
     * @param rscId
     * @throws MessagingException
     */
    public void sendResourceMail(String to, String subject, String content, String rscPath, String rscId) throws MessagingException;

    public void sendResourceMail(String to, String subject, String content, String rscPath, String rscId, String... cc);

}
```



```java
@Data
public class JsonResult<T> {

    private int code;
    private String msg;
    private T data;


    public JsonResult() {
        this.code = 0;
        this.msg = "success";
    }


    public JsonResult(int code, String msg) {
        this.code = code;
        this.msg = msg;
    }


    public JsonResult(T data) {
        this.data = data;
        this.code = 0;
        this.msg = "success";
    }


    public JsonResult(T data, String msg) {
        this.data = data;
        this.code = 0;
        this.msg = msg;
    }
}
```



```java
import com.study.mail.service.IMailService;
import com.study.mail.vo.JsonResult;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.thymeleaf.TemplateEngine;
import org.thymeleaf.context.Context;

@RestController
@RequestMapping("email")
public class EmailController {

    @Autowired
    private IMailService mailService;
    @Autowired
    private TemplateEngine templateEngine;

    @RequestMapping("index")
    public JsonResult index(){
        try {
            mailService.sendSimpleMail("434539550@qq.com","这是一封普通的邮件","这是一封普通的SpringBoot测试邮件");
        }catch (Exception ex){
            ex.printStackTrace();
            return new JsonResult(-1,"邮件发送失败!!");
        }
        return new JsonResult();
    }

    @RequestMapping("/htmlEmail")
    public JsonResult htmlEmail(){
        try {
            mailService.sendHtmlMail("434539550@qq.com","这是一HTML的邮件","<body style=\"text-align: center;margin-left: auto;margin-right: auto;\">\n"
                    + "	<div id=\"welcome\" style=\"text-align: center;position: absolute;\" >\n"
                    +"		<h3>欢迎使用IJPay -By Javen</h3>\n"
                    +"		<span>https://github.com/Javen205/IJPay</span>"
                    + "		<div\n"
                    + "			style=\"text-align: center; padding: 10px\"><a style=\"text-decoration: none;\" href=\"https://github.com/Javen205/IJPay\" target=\"_bank\" ><strong>IJPay 让支付触手可及,欢迎Start支持项目发展:)</strong></a></div>\n"
                    + "		<div\n" + "			style=\"text-align: center; padding: 4px\">如果对你有帮助,请任意打赏</div>\n"
                    + "		<img width=\"180px\" height=\"180px\"\n"
                    + "			src=\"https://oscimg.oschina.net/oscnet/8e86fed2ee9571eb133096d5dc1b3cb2fc1.jpg\">\n"
                    + "	</div>\n" + "</body>");
        }catch (Exception ex){
            ex.printStackTrace();
            return new JsonResult(-1,"邮件发送失败!!");
        }
        return new JsonResult();
    }

    @RequestMapping("/attachmentsMail")
    public JsonResult attachmentsMail(){
        try {
            String filePath = "/Users/Javen/Desktop/IJPay.png";
            mailService.sendAttachmentsMail("434539550@qq.com", "这是一封带附件的邮件", "邮件中有附件，请注意查收！", filePath);
        }catch (Exception ex){
            ex.printStackTrace();
            return new JsonResult(-1,"邮件发送失败!!");
        }
        return new JsonResult();
    }

    @RequestMapping("/resourceMail")
    public JsonResult resourceMail(){
        try {
            String rscId = "IJPay";
            String content = "<html><body>这是有图片的邮件<br/><img src=\'cid:" + rscId + "\' ></body></html>";
            String imgPath = "/Users/Javen/Desktop/IJPay.png";
            mailService.sendResourceMail("434539550@qq.com", "这邮件中含有图片", content, imgPath, rscId);

        }catch (Exception ex){
            ex.printStackTrace();
            return new JsonResult(-1,"邮件发送失败!!");
        }
        return new JsonResult();
    }

    @RequestMapping("/templateMail")
    public JsonResult templateMail(){
        try {
            Context context = new Context();
            context.setVariable("project", "IJPay");
            context.setVariable("author", "Javen");
            context.setVariable("url", "https://github.com/Javen205/IJPay");
            String emailContent = templateEngine.process("emailTemp", context);

            mailService.sendHtmlMail("434539550@qq.com", "这是模板邮件", emailContent);
        }catch (Exception ex){
            ex.printStackTrace();
            return new JsonResult(-1,"邮件发送失败!!");
        }
        return new JsonResult();
    }
}

```

