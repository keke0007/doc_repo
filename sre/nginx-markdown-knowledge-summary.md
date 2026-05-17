# Nginx -markdown 知识点维度汇总（Nginx）

> 来源：`shell-markdown/1.md` ~ `shell-markdown/12.md`。  
> 目标：按知识点维度梳理，每个知识点给出示例代码与应用场景。

## 1. Nginx 定位与应用场景

Nginx 是高性能 HTTP 中间件与反向代理，常用于：
- 静态资源服务
- 反向代理
- 负载均衡
- 缓存加速
- 安全防护（WAF、访问控制）

示例（最小可用 Web 服务）：
```nginx
server {
    listen 80;
    server_name _;
    root /usr/share/nginx/html;
    index index.html;
}
```

应用场景：前端静态站、API 网关入口、多服务统一接入层。

## 2. Nginx 安装与编译

常见安装方式：
- `yum/apt` 快速安装
- 源码编译（可控制模块）
- OpenResty（Nginx + Lua 生态）

示例（RPM 仓库安装）：
```bash
cat >/etc/yum.repos.d/nginx.repo <<'EOF'
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=0
enabled=1
EOF

yum install -y nginx
systemctl enable --now nginx
```

应用场景：标准化批量部署；需要精确模块能力时使用编译安装。

## 3. Nginx 配置结构与常用模块

核心结构：
- `main`：全局配置
- `events`：连接模型
- `http`：HTTP 服务层
- `server`：虚拟主机
- `location`：URI 路由

示例（基础结构）：
```nginx
worker_processes auto;

events {
    worker_connections 10240;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    server {
        listen 80;
        server_name example.com;
        location / { root /data/www; index index.html; }
    }
}
```

应用场景：多站点管理、按 URI 切流、统一日志与安全策略。

## 4. 日志与状态监控

重点：
- `access_log` / `error_log`
- `log_format` 自定义字段
- `stub_status` 状态页监控

示例：
```nginx
http {
    log_format main '$remote_addr - $host [$time_local] "$request" $status $body_bytes_sent "$http_user_agent"';
    access_log /var/log/nginx/access.log main;
    error_log  /var/log/nginx/error.log warn;

    server {
        listen 80;
        location /nginx_status {
            stub_status;
            allow 127.0.0.1;
            deny all;
        }
    }
}
```

应用场景：压测对比、错误定位、QPS/连接数观察。

## 5. 目录浏览、下载与访问限制

重点：
- `autoindex` 目录浏览
- `limit_conn` / `limit_req` 限制连接与请求
- 基于 IP 和账号认证控制访问

示例（下载站点 + IP 限制）：
```nginx
location /download/ {
    alias /data/download/;
    autoindex on;
    allow 10.0.0.0/8;
    deny all;
}
```

应用场景：内网软件仓库、受限文件分发、后台接口防滥用。

## 6. 虚拟主机（多站点）

支持：
- 基于域名
- 基于端口
- 别名域名 `server_alias`（等价 `server_name` 多值）

示例（基于域名）：
```nginx
server {
    listen 80;
    server_name a.example.com;
    root /data/www/a;
}

server {
    listen 80;
    server_name b.example.com;
    root /data/www/b;
}
```

应用场景：一台机器承载多个业务站点。

## 7. 静态资源服务优化

重点：
- 静态资源路径映射
- `sendfile`、`tcp_nopush`、`tcp_nodelay`
- 文件压缩 `gzip`
- 浏览器缓存 `expires` / `Cache-Control`

示例：
```nginx
http {
    sendfile on;
    gzip on;
    gzip_types text/css application/javascript application/json image/svg+xml;

    server {
        location /assets/ {
            root /data/www;
            expires 7d;
            add_header Cache-Control "public, max-age=604800";
        }
    }
}
```

应用场景：前端静态资源加速，减少带宽和回源压力。

## 8. CORS 跨域与防盗链

重点：
- `add_header Access-Control-*`
- `valid_referers` + `if ($invalid_referer)`

示例（跨域）：
```nginx
location /api/ {
    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Methods "GET,POST,OPTIONS";
    add_header Access-Control-Allow-Headers "Content-Type,Authorization";
    if ($request_method = OPTIONS) { return 204; }
    proxy_pass http://backend_api;
}
```

示例（防盗链）：
```nginx
location ~* \.(jpg|png|mp4)$ {
    valid_referers none blocked *.example.com;
    if ($invalid_referer) { return 403; }
}
```

应用场景：前后端分离、资源防外站盗用。

## 9. 反向代理

核心指令：
- `proxy_pass`
- `proxy_set_header`
- `proxy_connect_timeout/proxy_read_timeout`
- `proxy_redirect`

示例：
```nginx
location / {
    proxy_pass http://app_cluster;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_read_timeout 60s;
}
```

应用场景：统一入口、隐藏后端、协议转换、网关治理。

## 10. 负载均衡

常用策略：
- 轮询（默认）
- `weight`
- `ip_hash`
- `least_conn`

示例：
```nginx
upstream app_cluster {
    least_conn;
    server 10.0.0.11:8080 weight=3 max_fails=2 fail_timeout=10s;
    server 10.0.0.12:8080 weight=2 max_fails=2 fail_timeout=10s;
}
```

应用场景：分摊流量、提升可用性、滚动发布承载。

## 11. 动静分离

思路：
- 静态请求由 Nginx 直接返回
- 动态请求转发到应用层（PHP/Java 等）

示例：
```nginx
location /static/ {
    root /data/www;
    expires 30d;
}

location / {
    proxy_pass http://app_cluster;
}
```

应用场景：提升吞吐，降低应用服务器负载。

## 12. 代理缓存（Proxy Cache）

关键指令：
- `proxy_cache_path`
- `proxy_cache`
- `proxy_cache_valid`
- `proxy_cache_key`
- `add_header X-Cache-Status $upstream_cache_status`

示例：
```nginx
proxy_cache_path /data/nginx/cache levels=1:2 keys_zone=STATIC:100m max_size=10g inactive=60m use_temp_path=off;

location / {
    proxy_cache STATIC;
    proxy_cache_key $scheme$proxy_host$request_uri;
    proxy_cache_valid 200 304 12h;
    proxy_cache_valid any 10m;
    add_header X-Cache-Status $upstream_cache_status;
    proxy_pass http://app_cluster;
}
```

应用场景：热点页面缓存、降低后端 QPS、提升响应速度。

## 13. Rewrite 与重定向

用途：
- URL 规范化
- 老链接迁移
- 按规则改写路由

示例（301 重定向）：
```nginx
server {
    listen 80;
    server_name old.example.com;
    return 301 https://www.example.com$request_uri;
}
```

示例（rewrite）：
```nginx
location / {
    rewrite ^/user/([0-9]+)$ /profile.php?id=$1 last;
}
```

应用场景：SEO、域名切换、历史 URL 兼容。

## 14. HTTPS 与证书配置

重点：
- `listen 443 ssl http2`
- 证书与私钥
- TLS 协议和加密套件
- HTTP 跳转 HTTPS

示例：
```nginx
server {
    listen 443 ssl http2;
    server_name www.example.com;

    ssl_certificate     /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
}

server {
    listen 80;
    server_name www.example.com;
    return 301 https://$host$request_uri;
}
```

应用场景：生产站点安全加密、满足移动端 ATS/合规要求。

## 15. LNMP（Nginx + PHP-FPM + MySQL）

关键点：
- Nginx 与 PHP-FPM 对接（`fastcgi_pass`）
- PHP-FPM 进程池参数（`pm.*`）
- 慢日志与错误日志

示例：
```nginx
location ~ \.php$ {
    root /data/www;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
}
```

应用场景：PHP 动态网站、电商 CMS、后台管理系统。

## 16. LNMT（Nginx + Tomcat）

关键点：
- Nginx 反向代理到 Tomcat
- 可结合 upstream 做多节点 LB

示例：
```nginx
upstream tomcat_cluster {
    server 10.0.0.21:8080;
    server 10.0.0.22:8080;
}

location / {
    proxy_pass http://tomcat_cluster;
}
```

应用场景：Java Web 服务入口统一、灰度扩容。

## 17. Nginx + Lua（OpenResty）

能力：
- `rewrite/access/content` 阶段编程
- 灰度发布、动态路由、WAF 规则定制

示例（按请求头灰度）：
```nginx
upstream app_v1 { server 10.0.0.11:8080; }
upstream app_v2 { server 10.0.0.12:8080; }

location / {
    access_by_lua_block {
        local canary = ngx.req.get_headers()["X-Canary"]
        if canary == "1" then
            ngx.var.target = "http://app_v2"
        else
            ngx.var.target = "http://app_v1"
        end
    }
    proxy_pass $target;
}
```

应用场景：灰度发布、A/B 测试、细粒度流量控制、安全过滤。

## 18. 性能优化

主要维度：
- 网络：带宽、延迟、丢包
- 系统：CPU、内存、文件句柄、内核参数
- 服务：worker、连接数、缓存、压缩

示例（常见优化项）：
```nginx
worker_processes auto;
worker_rlimit_nofile 65535;

events {
    use epoll;
    worker_connections 20480;
    multi_accept on;
}

http {
    sendfile on;
    keepalive_timeout 65;
    gzip on;
}
```

应用场景：高并发站点、压测调优、吞吐瓶颈治理。

## 19. 压测与性能指标理解

常用工具：
- `ab`
- `wrk`
- `siege`

核心指标：
- QPS/RPS
- 平均响应时间
- 失败请求数
- 吞吐速率

示例（ab）：
```bash
ab -n 10000 -c 200 http://127.0.0.1/index.html
```

应用场景：发布前容量评估、优化前后对比验证。

## 20. 常见问题与高频面试点

重点：
- `server` 匹配优先级
- `location` 匹配优先级（`=`、`^~`、`~`/`~*`、前缀）
- `try_files` 使用
- `root` 与 `alias` 区别
- 获取真实 IP（`X-Forwarded-For` / `real_ip`）

示例（try_files）：
```nginx
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
```

示例（root vs alias）：
```nginx
location /img/ {
    root /data/www;      # 实际路径 /data/www/img/...
}
location /download/ {
    alias /data/files/;  # 实际路径 /data/files/...
}
```

应用场景：解决 404/路由错乱、定位虚拟主机匹配问题、排查反代后客户端 IP 丢失。

## 21. 一套可复用的生产配置骨架

示例：
```nginx
worker_processes auto;
events { worker_connections 10240; }

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile on;
    keepalive_timeout 65;
    gzip on;

    log_format main '$remote_addr - $host [$time_local] "$request" $status $body_bytes_sent';
    access_log /var/log/nginx/access.log main;
    error_log  /var/log/nginx/error.log warn;

    upstream app_cluster {
        least_conn;
        server 10.0.0.11:8080;
        server 10.0.0.12:8080;
    }

    server {
        listen 80;
        server_name www.example.com;
        return 301 https://$host$request_uri;
    }

    server {
        listen 443 ssl http2;
        server_name www.example.com;
        ssl_certificate     /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/privkey.pem;

        location /static/ {
            root /data/www;
            expires 7d;
        }

        location / {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://app_cluster;
        }
    }
}
```

应用场景：企业中小型业务的标准化起步模板。

## 22. 推荐学习路径（按依赖关系）

1. Nginx 基础与配置结构  
2. 静态服务与日志监控  
3. 反向代理、负载均衡、动静分离  
4. 缓存、rewrite、HTTPS  
5. LNMP/LNMT 架构实践  
6. Lua 扩展与安全策略  
7. 性能优化与故障排查  

