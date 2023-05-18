



# 漏洞介绍：

  SpringCloudFunction是SpringBoot开发的一个Servless中间件(FAAS),支持基于SpEL的函数式动态路由.当Spring Cloud Function 启用动态路由functionRouter时, HTTP请求头spring.cloud.function.routing-expression参数存在SPEL表达式注入漏洞, 攻击者可通过该漏洞进行远程命令执行.

  

话不多说， 直接复现！  

  

```
影响的版本：
```

  

# 搭环境：

  

使用Spring Initializer一步到位

![图片](https://mmbiz.qpic.cn/mmbiz_png/bkEQtib9Aglty9rMfBmYjMS9tleA8ic3Gt0vaxdFovyv1AJoicKYibFx9YJCE43EX1Yd1dxm5m4Ctln71Wsw7kyfkg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)  

  

我选择的是jdk1.8

![图片](https://mmbiz.qpic.cn/mmbiz_png/bkEQtib9Aglty9rMfBmYjMS9tleA8ic3GtdW253hj0Day17hAniaR6Vt5pN3HSYgkVEyPTib4XLlb6DuDia8CAoB2SQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

选择指定的模块就完事了

![图片](https://mmbiz.qpic.cn/mmbiz_png/bkEQtib9Aglty9rMfBmYjMS9tleA8ic3GtFcCaRLct4jdCVT8CNn5b9twibiaibEjiam2MJ1MkLicTdBh41CV5RCMxghw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/bkEQtib9Aglty9rMfBmYjMS9tleA8ic3Gt39lkpwicEUGMibkUnur6ichjP8YcicMoKqnc4lYwtVy1KAl1theG56CqCw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

  

改一下pom.xml 修改成指定的Spring-cloud-function版本

```
 <groupId>org.springframework.cloud</groupId>
```

burp抓包打！

路径/functionRouter  
  

```
Payload:
```

  

拿下！  

![图片](https://mmbiz.qpic.cn/mmbiz_png/bkEQtib9Aglty9rMfBmYjMS9tleA8ic3GtklOb0AyAe2Z90OcSpFQMSMc2icamPu3cibqc1kpxNnNAjDJ6s4dtEerQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)