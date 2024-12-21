---
title:  halo博客邮件通知设置：使用阿里云SMTP邮件推送服务 
subtitle:
date: 2024-12-08T21:41:18+08:00
slug: X1OPK9Tf
tags:
  - SMTP
  - 邮件推送
  - 博客邮件推送
categories:
  - 系统管理
---

阿里云的邮件推送服务免费额度200封，超出免费额度的发信将消费资源包（消费完自动转为按量计费）需要更多资源需要购买资源包。非常便宜，个人使用几乎等于免费。

前置条件：需要备案域名
申请地址：https://www.aliyun.com/product/directmail

#### 添加发信域名

需要备案域名

域名最好使用单独的二级域名来做发信域名，比如这里使用fr.idev.ink作为发信域名。

![28525D31-4879-429C-9019-116C659202CB.png](https://img.idev.ink/2024/12/08/28525D31-4879-429C-9019-116C659202CB.png)

#### 添加发信地址和回信地址

发信地址就是实际发信的邮箱账号，回信地址是用户收到邮件后回信的接收邮箱，添加一个私人邮箱。

![FBE73067-C1E2-4FA8-964A-AB22D92256C8.png](https://img.idev.ink/2024/12/08/FBE73067-C1E2-4FA8-964A-AB22D92256C8.png)

![546B359A-52E2-4C08-B041-BB53ACD0B284.png](https://img.idev.ink/2024/12/08/546B359A-52E2-4C08-B041-BB53ACD0B284.png)

**SMTP服务地址： smtpdm.aliyun.com ，SMTP服务端口号：25或80或465(SSL加密)。**

#### halo博客配置

然后进入halo后台点击设 置 - > 通知设置，填入所需信息即可。

![440F0C3C-037A-4531-A692-99177D7232A1.png](https://img.idev.ink/2024/12/08/440F0C3C-037A-4531-A692-99177D7232A1.png)

![D13FADC4-6D34-4E37-9337-3AA091935631.png](https://img.idev.ink/2024/12/08/D13FADC4-6D34-4E37-9337-3AA091935631.png)

