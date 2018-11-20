# [www.minminmsn.com](https://www.minminmsn.com) 
> ##### 是否受够了各博客平台之间频繁切换，精心设计的语言被无辜替换，辛苦维护的博文偶尔遭到无情的误杀，铺天盖地的广告疯狂的轰炸。烦了！倦了！够了！终于我要出手了，现在给你个配方就可以从零开始创建自己的博客，只需要花点儿银子（平均每月百来元即可小玩一把）就能解决以上所有痛点，在自己的地盘自己做主，随意撒野！
> ##### 具体主要包含如下几步（欢迎各位疯狂点赞、收藏、转载，打CALL）：

### 域名注册
选择一个有代表性的域名，比如：minminmsn.com  
参考：https://buy.cloud.tencent.com/domain?from=console


### 域名备案
个人站点备案，借助平台还是很方便的，大概20工作日左右可完成（根据大陆法所有域名需要备案，否则后果自负）  
参考：https://console.cloud.tencent.com/beian


### 证书申请
竟然有域名型免费版可以免费试用一年（窃喜）  
https://buy.cloud.tencent.com/ssl


### 架构设计
#### 效果图
![](https://github.com/minminmsn/accesslog-analysis-alarm/blob/master/images/minminmsn.jpg)

#### 架构图
#### Nginx(证书、跳转、流控、反向代理）    
       |
#### Docker源站（安全、缓存插件）    
       |
#### Docker数据库（mysql）    
       |
#### 定期备份（脚本）    


### 部署配置
#### Nginx部署
```
wget http://nginx.org/download/nginx-1.14.1.tar.gz
tar zxvf nginx-1.14.1.tar.gz
cd nginx-1.14.1/
./configure --with-http_ssl_module 
make
make install 
```
#### Nginx配置
```
user  nobody;
worker_processes  auto;

error_log  logs/error.log;

pid        logs/nginx.pid;

#偶尔调大点就行，个人博客访问量大绝对有问题
events {
    worker_connections  10240;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  logs/access.log  main;

    sendfile        on;
    tcp_nopush     on;
    keepalive_timeout  65;

    #开启压缩节省服务器流量
    gzip  on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_comp_level 2;
    gzip_types text/htm application/x-javascript text/css image/jpeg image/png;
    gzip_vary off;
    
    #配置限流放置轻微恶搞
    limit_req_status 418;
    limit_conn_status 418;
    limit_req_zone  $binary_remote_addr zone=one:10m rate=3r/s;
    limit_conn_zone $binary_remote_addr zone=addr:10m;

    server {
        listen 80;
        server_name minminmsn.com m.minminmsn.com www.minminmsn.com;
        return 301 https://wwww.minminmsn.com$request_uri;
    }

    #注意填上申请的证书及加密算法的安全性
    server {
        listen       443 ssl;
        server_name  www.minminmsn.com;

        limit_req zone=one burst=5 nodelay;
        limit_conn  addr 5;

        ssl_certificate      ssl/minminmsn.crt;
        ssl_certificate_key  ssl/minminmsn.key;
        ssl_prefer_server_ciphers   on;
        ssl_ciphers "ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:AES256-GCM-SHA384:AES128-GCM-SHA256:AES256-SHA256:AES128-SHA256:AES256-SHA:AES128-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4";
        ssl_protocols               TLSv1.3 TLSv1.2 TLSv1.1 TLSv1;
        ssl_session_cache           shared:SSL:10m;
        ssl_session_timeout         60m;
        
        #wordpress默认不支持ssl，nginx反向代理配置后还需要安装插件ssl-insecure-content-fixer才行
        location / {
            proxy_pass   http://127.0.0.1:8080;
            proxy_http_version 1.1;
            proxy_set_header X-Forwarded-Host $host;
            proxy_set_header X-Forwarded-Server $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto https;
            proxy_set_header X-Forwarded-Port 443;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header Host $host;
            client_max_body_size    30M;
            client_body_buffer_size 256k;
            proxy_connect_timeout       60s;
            proxy_read_timeout          600s;
            proxy_send_timeout          600s;
        }
    }
}
```
#### Docker部署
```
docker可以当做一个可以开启暂停重启关闭的程序，只要数据保存到外部不丢失，使用起来非常方便快捷
docker pull mysql:5.7
docker pull wordpress:latest
docker run --name mysql  -v /yourpath/db:/var/lib/mysql  -e MYSQL_ROOT_PASSWORD=your@password --restart=always   -d mysql:5.7
docker run --name wordpress -v /yourpath/www:/var/www/html  --restart=always  --link mysql:mysql -p 8080:80 -d wordpress
```

### 主题选型
> （选择能Hold住且清新脱俗的）
Chilly是一个响应式，多功能的WordPress主题。灵活的它，适合显示机构、博客、商业、公司或作品集。定制是容易和直接的，提供的选项能让您设置您的站点，以完全符合您所期望的在线存在。请访问此链接https://wordpress.org/themes/spicepress/。


### 前端设计（简洁明快博雅）
+ #### 主题
> ##### 三杯水
+ #### 开源背景图
> ##### 支持开源
+ #### 一首小诗
> ##### 兰德的生与死
+ #### 一张小图
> ##### 博雅塔
+ #### 分类栏目
> ##### 九九归一
+ #### 日历
> ##### 三的倍数是发博日


### 插件选型（选简单好用的）
+ #### AddToAny Share Buttons
可以复杂链接分享给国内外常用的社交平台
+ #### Limit Login Attempts Reloaded
防止暴力破解博客账号密码
+ #### Mobile Menu
在不同的终端都能很好的展示你的博客
+ #### WP Editor.md
或许这是一个WordPress中最好，最完美的Markdown编辑器
+ #### WP Super Cache
博客性能优化，缓存静态资源，访问加速
+ #### WP 统计
后台统计，可对地域及点击率进行分析
+ #### SSL-insecure-content-fixer
帮助您清理并修复 WordPress 站点的 HTTPS 不安全内容

### 原创内容（九九归一，支持原创）
+ #### 佛学
> ##### 博大精深有搞头
+ #### 哲学
> ##### 自由自在无边界
+ #### 开发
> ##### 山不过来我过去
+ #### 心学
> ##### 燃烧你的小宇宙
+ #### 技术
> ##### 术是第一生产力
+ #### 数据
> ##### 数据满足好奇心
+ #### 杂谈
> ##### 亦正亦邪冰与火
+ #### 读书
> ##### 终身跨学科提问
+ #### 运维
> ##### 运筹于帷幄之中




