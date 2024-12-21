---
title:  阿里云域名+Nginx Proxy Manager配置SSL证书全流程 
subtitle:
date: 2024-12-11T21:41:18+08:00
slug: Cv5a9gA3
tags:
  - Linux
  - NginxProxyManager
categories:
  - 系统管理
---

本篇文章介绍：域名托管在阿里云，使用nginx proxy manager配置SSL证书的操作流程。

nginx proxy manager部署，可以看我的这篇文章：[https://www.idev.ink/archives/zSA7KJ8k](https://www.idev.ink/archives/zSA7KJ8k)

下面直接开始操作

#### 阿里云DNS API的获取

##### 创建子账号

前往：https://ram.console.aliyun.com 申请子账号。

![F8328333-00D1-4637-9932-AFEAB4E2AF68.png](https://img.idev.ink/2024/12/11/F8328333-00D1-4637-9932-AFEAB4E2AF68.png)

名字任意命名，下面勾上：使用永久 AccessKey 访问
![AA9D2F2E-5AD5-4CD2-8754-345E8A22F603.png](https://img.idev.ink/2024/12/11/AA9D2F2E-5AD5-4CD2-8754-345E8A22F603.png)

然后确定

![5ADB7A45-10FC-4758-A881-7FAF089AE82D.png](https://img.idev.ink/2024/12/11/5ADB7A45-10FC-4758-A881-7FAF089AE82D.png)

把AccessKey ID和Secret都复制下来备用。

##### 子账号授权

点击授权，新增授权：
![F15DAC2A-5100-473D-9EFA-1CEDDB8E8C7D.png](https://img.idev.ink/2024/12/11/F15DAC2A-5100-473D-9EFA-1CEDDB8E8C7D.png)

勾选刚才新建的用户，授予两个权限：`AliyunHTTPDNSFullAccess`  `AliyunDNSFullAccess`

![E085DAAF-9264-4AF1-A1CC-3A35226E537D.png](https://img.idev.ink/2024/12/11/E085DAAF-9264-4AF1-A1CC-3A35226E537D.png)

然后确定，aliyun DNS API 获取完毕。

#### 使用 nginx proxy manager 使用阿里云域名为网站部署证书

登录nginx proxy manager 控制台 点击`SSL Certificates`

![B2F1212A-B64E-40B9-999B-01B5E74261EF.png](https://img.idev.ink/2024/12/11/B2F1212A-B64E-40B9-999B-01B5E74261EF.png)

点击 Add SSL Certificate

![4C741D10-A009-4C68-AA5A-1F01C11B89BB.png](https://img.idev.ink/2024/12/11/4C741D10-A009-4C68-AA5A-1F01C11B89BB.png)

如图所示我设置了一个通配符证书，DNS服务商选阿里云，把下面的API ID和值替换成实际值，点save。
![2E53D4D0-6AF6-4D4F-A760-FBF0CC40B411.png](https://img.idev.ink/2024/12/11/2E53D4D0-6AF6-4D4F-A760-FBF0CC40B411.png)

成功后如图所示：
![6FA23FE0-008C-444C-9A65-0706DF79EC46.png](https://img.idev.ink/2024/12/11/6FA23FE0-008C-444C-9A65-0706DF79EC46.png)

在部署的其他服务中就可以直接使用了
![69AEA531-E9EC-451E-94F0-786880AB847A.png](https://img.idev.ink/2024/12/11/69AEA531-E9EC-451E-94F0-786880AB847A.png)

