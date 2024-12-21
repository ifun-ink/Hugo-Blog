---
title:  Traefik的完整配置 
subtitle:
date: 2024-11-30T21:41:18+08:00
slug: bny8WNBH
tags:
  - traefik
  - 反向代理
categories:
  - 系统管理
  - 应用部署
---

本文只是给出示例配置，不是详细教程，没有详细解释，后面有时间的话可能会补上。
traefik配置起来比较繁琐，但是一旦配置好使用起来就很方便了。

#### 目录结构：

```
.
├── data
│   ├── certs
│   │   └── acme.json
│   └── config
│       ├── dynamic.yml
│       └── traefik.yml
├── docker-compose.yaml
└── .env
```

`traefik.yml`是静态配置。
`dynamic.yml`是动态配置
`acme.json`是存放证书的文件，
`.env`是环境变量，隐私变量放在这里，比如密码密钥之类的。

创建文件和目录：

```
创建主目录
mkdir -p ./data/certs
mkdir -p ./data/config

创建配置文件：
touch ./data/certs/acme.json
touch ./data/config/dynamic.yml
touch ./data/config/traefik.yml
touch docker-compose.yaml
touch .env
```

证书文件需要特别设置权限：

```
chmod 600 ./data/certs/acme.json
```

下面是配置具体内容：

#### 主配置`docker-compose.yaml`;

```
services:
  traefik:
    container_name: traefik
    image: traefik:v3.2
    ports:
      - "80:80"
      - "443:443"
    environment:
      #cf
      - "CF_DNS_API_TOKEN=${CF_DNS_API_TOKEN}"
      - "TRAEFIK_DASHBOARD_CREDENTIALS=${TRAEFIK_DASHBOARD_CREDENTIALS}"
      #阿里云
      - "ALICLOUD_ACCESS_KEY=${ALICLOUD_ACCESS_KEY}"
      - "ALICLOUD_SECRET_KEY=${ALICLOUD_SECRET_KEY}"
      - "ALICLOUD_REGION_ID=${ALICLOUD_REGION_ID}"
      #腾讯云
      - "TENCENTCLOUD_SECRET_ID=${TENCENTCLOUD_SECRET_ID}"
      - "TENCENTCLOUD_SECRET_KEY=${TENCENTCLOUD_SECRET_KEY}"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock
      - ./data/config/traefik.yml:/traefik.yml:ro
      - ./data/config/dynamic.yml:/dynamic.yml:ro
      - ./data/certs/acme.json:/acme.json:rw   #设置权限：chmod 600 acme.json
    labels:
      - "traefik.enable=true"
      # 中间件定义
      - "traefik.http.middlewares.traefik-auth.basicauth.users=${TRAEFIK_DASHBOARD_CREDENTIALS}"
      # 路由配置
      - "traefik.http.routers.traefik-secure.entrypoints=websecure"
      - "traefik.http.routers.traefik-secure.rule=Host(`dash.host.domain.com`)" #这里是traefik的控制台域名
      - "traefik.http.routers.traefik-secure.middlewares=traefik-auth" #控制台密码验证中间件
      - "traefik.http.routers.traefik-secure.tls=true"
      - "traefik.http.routers.traefik-secure.tls.certresolver=aliyun"
      #域名证书申请，如果有多个域名建议一次性写在下面申请通配符证书，其他服务就可以直接用了，中括号里面的序号依次递增
      - "traefik.http.routers.traefik-secure.tls.domains[0].main=domain.com"
      - "traefik.http.routers.traefik-secure.tls.domains[0].sans=*.domain"
      - "traefik.http.routers.traefik-secure.tls.domains[1].main=domain2.com"
      - "traefik.http.routers.traefik-secure.tls.domains[1].sans=*.domain2.com"
      #多级域名也支持
      - "traefik.http.routers.traefik-secure.tls.domains[2].main=host.domain.com"
      - "traefik.http.routers.traefik-secure.tls.domains[2].sans=*.host.domain.com"     
      - "traefik.http.routers.traefik-secure.service=api@internal"     
    networks:
      - traefik_network
networks:
  traefik_network:  # 定义网络配置
    external: true  # 指定该网络是外部网络
```

先手动创建一个专属网络给所有服务使用：

```
docker network create traefik_network
```

这里环境环境变量定义三种申请证书的API 变量分别是Cloudflare，阿里云，腾讯云。
以及几个示例的域名，按需修改，服务启动时会自动申请证书。

#### 静态配置：`traefik.yml`

```
global:
  checkNewVersion: false               # 禁用版本检查，避免自动提示新版本。
  sendAnonymousUsage: false            # 禁用匿名数据发送，增强隐私性。

log:
  level: ERROR                         # 日志级别，设置为 ERROR 以减少日志噪声。
  # 选项: DEBUG (调试) / INFO (信息) / ERROR (错误)

api:
  dashboard: true                      # 启用 Traefik 的仪表盘。
  insecure: false                      # 生产环境应设置为 false，禁用非安全访问仪表盘。
  debug: false                         # 禁用 API 调试模式，提升性能。

entryPoints:
  web:
    address: :80                       # 定义 HTTP 入口点，监听 80 端口。
  websecure:
    address: :443                      # 定义 HTTPS 入口点，监听 443 端口。

# -- 可选：禁用 TLS 证书验证检查 (不建议在生产环境启用)
# serversTransport:
#   insecureSkipVerify: true           # 跳过后端 TLS 验证，仅用于测试或信任的后端。

providers:
  docker:
    exposedByDefault: false            # 禁止默认公开所有 Docker 容器，需手动设置 `traefik.enable=true` 标签。
    endpoint: 'unix:///var/run/docker.sock' # 使用 Docker 的 UNIX 套接字作为提供者。
  file:
    filename: /dynamic.yml             # 引用动态配置文件，包含中间件、路由等配置。

certificatesResolvers:
  cloudflare:
    acme:
      email: "test@domain.com"           # 使用 ACME 协议注册的电子邮件。
      storage: acme.json               # 证书存储路径。
      caServer: https://acme-v02.api.letsencrypt.org/directory  # 生产环境使用的 ACME 服务器。
      # caServer: https://acme-staging-v02.api.letsencrypt.org/directory # 测试环境 ACME 服务器。
      keyType: EC256                   # 使用 ECDSA P-256 密钥。
      dnsChallenge:                    # 使用 DNS-01 验证方式。
        provider: cloudflare           # Cloudflare DNS 提供商。
        resolvers:
          - "1.1.1.1:53"               # 优先使用 Cloudflare 的公共 DNS 解析。
          - "8.8.8.8:53"               # Google 的公共 DNS 解析。

  aliyun:
    acme:
      email: "test@domain.com"           # 使用 ACME 协议注册的电子邮件。
      storage: acme.json               # 证书存储路径。
      caServer: https://acme-v02.api.letsencrypt.org/directory  # 生产环境 ACME 服务器。
      # caServer: https://acme-staging-v02.api.letsencrypt.org/directory # 测试环境 ACME 服务器。
      keyType: EC256                   # 使用 ECDSA P-256 密钥。
      dnsChallenge:                    # 使用 DNS-01 验证方式。
        provider: alidns               # 阿里云 DNS 提供商。
        resolvers:
          - "1.1.1.1:53"               # Cloudflare DNS。
          - "8.8.8.8:53"               # Google DNS。

  tencentcloud:
    acme:
      email: "test@domain.com"     # 使用 ACME 协议注册的电子邮件。
      storage: acme.json               # 证书存储路径。
      caServer: https://acme-v02.api.letsencrypt.org/directory  # 生产环境 ACME 服务器。
      # caServer: https://acme-staging-v02.api.letsencrypt.org/directory # 测试环境 ACME 服务器。
      keyType: EC256                   # 使用 ECDSA P-256 密钥。
      dnsChallenge:                    # 使用 DNS-01 验证方式。
        provider: tencentcloud         # 腾讯云 DNS 提供商。
        disablePropagationCheck: true  # 禁用验证前的传播检查，提升速度。
        delayBeforeCheck: 100          # 等待时间（毫秒），避免因 DNS 延迟导致验证失败。
        resolvers:
          - "1.1.1.1:53"               # Cloudflare DNS。
          - "8.8.8.8:53"               # Google DNS。
          - "119.29.29.29:53"          # 腾讯云 DNSPod 公共 DNS。
```

#### 动态配置：`dynamic.yml`

```
http:
  routers:
    # 目前没有定义路由器 (routers)，在需要时手动添加。
    # 示例:
    # my-router:
    #   rule: "Host(`example.com`)"
    #   service: my-service
    #   middlewares:
    #     - secured

  services:
    # 目前没有定义服务 (services)，在需要时手动添加。
    # 示例:
    # my-service:
    #   loadBalancer:
    #     servers:
    #       - url: "http://localhost:8080"

  middlewares:
    https-redirectscheme:               # 中间件：强制将 HTTP 重定向为 HTTPS。
      redirectScheme:
        scheme: https                   # 目标协议为 HTTPS。
        permanent: true                 # 使用永久重定向 (HTTP 301)。

    default-headers:                    # 中间件：设置默认安全 HTTP 响应头。
      headers:
        frameDeny: true                 # 禁用页面嵌套 (防点击劫持)。
        browserXssFilter: true          # 启用浏览器的 XSS 过滤器。
        contentTypeNosniff: true        # 禁止 MIME 类型嗅探。
        forceSTSHeader: true            # 启用严格传输安全 (HSTS)。
        stsIncludeSubdomains: true      # HSTS 包括子域名。
        stsPreload: true                # 允许将域名加入 HSTS preload 列表。
        stsSeconds: 15552000            # HSTS 有效期：180 天 (单位为秒)。
        customFrameOptionsValue: SAMEORIGIN  # 允许同源 iframe 嵌套。
        customRequestHeaders:           # 自定义请求头。
          X-Forwarded-Proto: https      # 明确告知后端请求是通过 HTTPS 转发的。

    default-whitelist:                  # 中间件：IP 白名单。
      ipAllowList:
        sourceRange:
        - "10.0.0.0/8"                  # 允许内网 IP 段 (10.x.x.x)。
        - "192.168.0.0/16"              # 允许内网 IP 段 (192.168.x.x)。
        - "172.16.0.0/12"               # 允许内网 IP 段 (172.16.x.x - 172.31.x.x)。

    secured:                            # 中间件：组合多个中间件为链 (chain)。
      chain:
        middlewares:
        - default-whitelist             # 首先执行 IP 白名单过滤。
        - default-headers               # 然后应用安全 HTTP 响应头。
```

#### 环境变量：`.env`

```
# 账号密码 admin/B2BVdeYR1SvBt
 #生成命令：echo $(htpasswd -B -nb "admin" "B2BVdeYR1SvBt") | sed -e s/\\$/\\$\\$/g
TRAEFIK_DASHBOARD_CREDENTIALS="admin:$$2y$$05$$fxf4gv/OBn4K/sFs0L3lYOaB3In6fjWGwArjvoWjYqREihKoOsAMu"

CF_DNS_API_TOKEN="kG9BzwmAYL-hkyqM96W4o"

ALICLOUD_ACCESS_KEY="LTAI5Gc7SMjWY4"
ALICLOUD_SECRET_KEY="ykYpvtqwJEASbRiU"
ALICLOUD_REGION_ID="cn-shanghai"

#腾讯云API密钥: https://console.dnspod.cn/account/token/apikey 
TENCENTCLOUD_SECRET_ID=AKID53xZgcMqrfHiHd
TENCENTCLOUD_SECRET_KEY=QTQ5fYLSK9oHYg
```

#### 启动服务：

```
docker compose up -d
```

查看运行日志：

```
docker compose logs  -f
```

#### 使用方法

这里以Nginx为例来说明用法。
比如我要使用docker compose 启动一个Nginx服务

```
services:
  nginx:
    image: nginx:latest
    container_name: nginx_service
    restart: unless-stopped
    volumes:
      # 挂载 Nginx 配置文件
      - ./nginx.conf:/etc/nginx/nginx.conf
    labels:
      # 启用 Traefik 代理
      - "traefik.enable=true" 
      # 定义路由规则，基于Host来匹配请求
      - "traefik.http.routers.nginx.rule=Host(`nginx.example.com`)"
      - "traefik.http.routers.nginx.tls.certresolver=tencentcloud" #指定证书申请方式，如果主配置已经申请通配符这里可以注释掉
      - "traefik.http.routers.nginx.tls=true" # 启用 TLS
      - "traefik.http.routers.nginx.entrypoints=websecure" #入口使用HTTPS端口
      # 引用动态配置中的中间件，强制 HTTPS 重定向并添加默认头
      - "traefik.http.routers.nginx.middlewares=https-redirectscheme@file,secured@file"
      - "traefik.http.services.nginx.loadbalancer.server.port=80" #监听Nginx端口
    networks:
      - traefik_network
networks:
  traefik_network:
    external: true

```

只需要在原有配置里定义`labels`，如上面所示
然后启动服务：

```
docker compose up -d
```

启动后域名和证书就自动配置完成。

