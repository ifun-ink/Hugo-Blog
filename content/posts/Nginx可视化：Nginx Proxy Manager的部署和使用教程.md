---
title:  NginxProxyManager的部署和使用教程 
subtitle:
date: 2024-12-11T21:41:18+08:00
slug: zSA7KJ8k
tags:
  - Linux
  - NginxProxyManager
categories:
  - 系统管理
  - 应用部署
---

官方文档：https://nginxproxymanager.com/guide/

提示：`docker compose`的命令有下面两种格式：

```
docker compose
或者
docker-compose
```

中间带不带杠取决于你的docker-compose安装方式。

##### 下面正式开始安装配置`nginx-proxy-manager`

创建项目目录：

```bash
mkdir nginx-proxy-manager
cd nginx-proxy-manager
touch docker-compose.yml
```

编辑`docker-compose.yml`内容如下：

```yml
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: nginx-proxy-manager
    restart: unless-stopped
    ports:
    - '80:80'
    - '81:81'
    - '443:443'
    volumes:
    - ./data:/data
    - ./letsencrypt:/etc/letsencrypt
    networks:
    - nginx_network # 加入nginx_network 网络

networks:
nginx_network:
external: true # 声明此网络为外部网络，需确保nginx_network网络已事先创建
```

##### 部署

上面使用了一个nginx_network网络，后续部署所有的服务都可以加入此网络内，在同一网络下的服务不需要对外暴露端口，安全性大大提高。
启动服务前需要手动创建这个网络：

```
docker network create nginx_network
```

查看已创建网络：

```
docker network ls
```

然后启动：

```
docker compose up -d
或者
docker-compose up -d
```

可以使用下面命令查看日志

```
docker ps
docker compose logs app
```

`nginx-proxy-manager`控制台默认使用81端口，需要防火墙放行81端口，如果有安全组同样需要放行。

浏览器输入地址：`http://ip:81`进入控制台。

默认管理员用户以及密码如下：

```
Email:    admin@example.com
Password: changeme
```

使用默认用户登录后，系统会立即要求修改详细信息并更改密码。
![2C727715-FF91-4FC2-8C18-097ADCAC992E.png](https://img.idev.ink/2024/12/11/2C727715-FF91-4FC2-8C18-097ADCAC992E.png)

##### 添加域名

1.域名解析：
这里以Cloudflare为例：
![65EBA2FA-9EAC-4069-9309-5935906E6CCB.png](https://img.idev.ink/2024/12/11/65EBA2FA-9EAC-4069-9309-5935906E6CCB.png)

2.创建API
依次点击：右上角头像下拉菜单 -> 我的个人资料 -> API令牌 -> 创建令牌 -> 使用编辑区域模板
![D09EE884-DE40-40B3-BD4E-A46276BC5285.png](https://img.idev.ink/2024/12/11/D09EE884-DE40-40B3-BD4E-A46276BC5285.png)

如下图 所示：特定区域选择指定域名，点下面的继续创建。
![76FB47F9-070F-48EE-97E6-913B7096D7E0.png](https://img.idev.ink/2024/12/11/76FB47F9-070F-48EE-97E6-913B7096D7E0.png)

![11DF81B5-B084-4FF2-83CB-ABCBFA84F21F.png](https://img.idev.ink/2024/12/11/11DF81B5-B084-4FF2-83CB-ABCBFA84F21F.png)

把令牌复制保存。

API：5U5rBj-_BFK5h3gxX904zf9ph5SrxfQiFHM7R-sJ

##### 为`nginx-proxy-manager`设置域名

依次点击：SSL Certificates  -  add SSL Certificate
如图所示：
DNS Provider 选择Cloudflare
域名设置单域名：`nginx.azfum.com` 或通配符`*.azfum.com` 都可以
![80DA9806-313D-4E58-910F-CE1592ABEB00.png](https://img.idev.ink/2024/12/11/80DA9806-313D-4E58-910F-CE1592ABEB00.png)

![DCC7C8DA-E8FD-41B6-AD62-26D234CD0FAC.png](https://img.idev.ink/2024/12/11/DCC7C8DA-E8FD-41B6-AD62-26D234CD0FAC.png)
填入API token 和自己的邮箱（任意邮箱都可以）点击save
成功后如图：
![DDE7092A-80FC-4280-8FD9-2F2917F25798.png](https://img.idev.ink/2024/12/11/DDE7092A-80FC-4280-8FD9-2F2917F25798.png)

设置反向代理：
![633548B5-0670-4BAD-99A4-34A139E4C312.png](https://img.idev.ink/2024/12/11/633548B5-0670-4BAD-99A4-34A139E4C312.png)
点击：New Proxy Host
如图，把域名和IP以及端口填写进去：
![53143223-6620-46DF-9613-B460271E55F1.png](https://img.idev.ink/2024/12/11/53143223-6620-46DF-9613-B460271E55F1.png)
然后点击 SSL：
![C96DCF94-51EE-4A98-9C45-E36E7392834C.png](https://img.idev.ink/2024/12/11/C96DCF94-51EE-4A98-9C45-E36E7392834C.png)
选择证书，把安全选项全部勾选，点save保存。

![AE2B1B23-E239-4E82-9C01-00C70F9CAB10.png](https://img.idev.ink/2024/12/11/AE2B1B23-E239-4E82-9C01-00C70F9CAB10.png)

现在就可以使用域名来访问nginx-proxy-manager控制台了。
![677EB937-9E69-4030-9F5B-13293C169A1D.png](https://img.idev.ink/2024/12/11/677EB937-9E69-4030-9F5B-13293C169A1D.png)

