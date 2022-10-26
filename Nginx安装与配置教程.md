## 一、Nginx安装与配置详细教程
### 1. 安装nginx
#### Debian/Ubuntu  

    aot install -y nginx
### 2. 重启nginx(可选)  
    systemctl restart nginx
### 3. 检查nginx状态(可选)  
    systemctl status nginx
### 4. nginx的配置文件位于  
    /etc/nginx/sites-available/default
>或者  

    /etc/nginx/nginx.conf
### 5. 复制以下任意一份配置文件替换`default`文件中的所有内容，并且按照注释中的提示修改对应的内容(可选)  
    server {

        listen 80 default_server;

        listen [::]:80 default_server;
       

        server_name mydomain.me; #这里修改成自己的域名即可。

        location / {

                # First attempt to serve request as file, then

                # as directory, then fall back to displaying a 404.

                try_files $uri $uri/ =404;

        }
    }
### 6. 复制以下任意一份配置文件替换`nginx.conf`文件中http{}里的内容，并且按照注释中的提示修改对应的内容  
>配置1：

    server {
        listen 443 ssl;
        listen [::]:443 ssl;
        
        server_name 你的域名;  #你的域名
        ssl_certificate       /usr/local/etc/v2ray/server.crt; 
        ssl_certificate_key   /usr/local/etc/v2ray/server.key;
        ssl_session_timeout 1d;
        ssl_session_cache shared:MozSSL:10m;
        ssl_session_tickets off;
        
        ssl_protocols         TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers off;
        
        location / {
            proxy_pass https://伪装网址; #伪装网址，国内能打开的正常网站
            proxy_ssl_server_name on;
            proxy_redirect off;
            sub_filter_once off;
            sub_filter "伪装网址" $server_name;   #这里也要修改
            proxy_set_header Host "伪装网址";   #这里也要修改
            proxy_set_header Referer $http_referer;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header User-Agent $http_user_agent;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto https;
            proxy_set_header Accept-Encoding "";
            proxy_set_header Accept-Language "zh-CN";
        }
        
        
        location /Ray123 {   #路径，和v2ray的配置保持一致
            proxy_redirect off;
            proxy_pass http://127.0.0.1:12345;  #端口，和v2ray的配置保持一致
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
    
    server {
        listen 80;
        server_name 你的域名;    #你的域名
        rewrite ^(.*)$ https://${server_name}$1 permanent;
    }
>配置2：

    server {
        listen  443 ssl;
        ssl on;
        ssl_certificate       /etc/v2ray/v2ray.crt;
        ssl_certificate_key   /etc/v2ray/v2ray.key;
        ssl_protocols         TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers           HIGH:!aNULL:!MD5;
        server_name           mydomain.me;
            location /ray/ { # 与 V2Ray服务端 配置中的 path 保持一致
            proxy_redirect off;
            proxy_pass http://127.0.0.1:10000;#这个端口和服务端保持一致
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $http_host;
            
            # Show realip in v2ray access.log
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
    }
### 7. 如果使用的是配置2就需要将你的html文件或者静态网站放入以下nginx的静态网站目录，放入之前记得先删除目录中nginx默认的欢迎页面文件
    /usr/share/nginx/html/
>或者(二选一即可)

    /var/www/html/
### 8.  负载均衡配置
    stream {
        upstream hk { #以下的三个都是负载的目标服务器
            server hknf.mftop.xyz:42194;
            server hk.mftop.xyz:55689;
            server hk.lookmv.xyz:44562;
        }
        server { #以下信息配置的是本机的监听端口
            listen 11041;
            listen 11041 udp;
            proxy_pass hk; #proxy_pass对应的是upstream的名称，如果存在多个upstream组，表示由哪个组负载传过来的流量
        }
    }
### 9. 负载均衡的配置可以搭配配置1或配置2共同在`nginx.conf`使用。也可以使用以下命令单独新建一个专门负责负载均衡的配置文件目录
    mkdir -p /etc/nginx/tcpconf.d
### 10. 在新建的这个目录下再新建一个负责负载均衡配置文件,将上边负载均衡的配置复制粘贴到新增的这个配置文件中并且按照注释中的提示修改对应的内容。
    vi /etc/nginx/tcpconf.d/ssrproxy.conf
>ssrproxy是配置文件的名字，最好能清楚的表示你负载的项目，方便以后维护，比如可以命名为v2rayproxy或者trojanproxy都行，重点是`.conf`这个后缀别忘了就行
### 11. 执行以下命令将tcpconf.d这个目录下所有的`.conf`后缀的配置文件引入`nginx.conf`主配置文件中
    echo "include /etc/nginx/tcpconf.d/*.conf;" >> /etc/nginx/nginx.conf
### 12. nginx重新加载配置文件(只要修改了nginc配置文件,最后都需要执行下面的命令，无论是修改了`nginx.conf`或者是`default`)
    systemctl reload nginx
### 13. 编辑定时任务
    crontab -e
### 14. 将nginx重启设置为定时任务(可选，有些服务器给的是动态IP时需要设置)
    0 */6 * * * systemctl restart nginx
### 15. 设置ngnx服务为开机自动启动(可选)
    systemctl enable nginx.service
## 二、Nginx配置详细解读教程
### 1. 标准配置
    # global                                  #全局配置
    worker_processes    1;

    # events                                  #events模块
    events {
        worker_connections  1024;
    }

    # http                                    #http模块
    http {
        include       mime.types;
        default_type  application/octet-stream;
        sendfile        on;
        keepalive_timeout   65;

        # virtual host                        #虚拟主机
        server {
            listen        80;
            server_name   localhost;

            location / {
                root    html;
                index   index.html index.html;
            }
            error_page    500 502 503 504   /50x.html;
            location = /50x.html {
                root   html;
            }
        }
    }
### 2. 最简配置
    events  {}
    http    {
        server  {
            return  200 "welcome";
        }
    }
### 3. 将标准配置拆分开解读，首先看全局配置
    # global                                  #全局配置
    user www www;   #这里定义了运行nginx的用户和用户组
    worker_processes    1;  #这里是设置nginx的worker进程数量，根据cpu来设置这个数量，默认是1，比较安全保险的设置是auto
    
    #以下三条配置是表示将nginx的三种类型的日志信息统一输出到同一个目录路径下的error.log文件中
    error_log /var/log/nginx/error.log info;
    error_log /var/log/nginx/error.log debug;
    error_log /var/log/nginx/error.log error;

    pid /var/run/nginx.pid; #这里配置的是nginx主进程的id

    worker_rlimit_nofile 65535; #这里表示每一个nginx进程允许打开文件的最大数量
### 4.events配置
    events {
        use epoll;    #这个配置是申明网络io模型的选用
        worker_connections  1024;   #这里表示单个进程的最大连接数
    }
### 5. http模块配置
    http {
        #以下4条配置设置的时候一定要谨慎，切勿设置的太大或者太小，设置的太小会导致nginx不断的使用io,影响它的性能；设置的太大会造成安全问题，容易被攻击者打开过多的连接而系统无法分配足够的缓存处理等等
        client_header_buffer_size 1k;   #当接收到客户端的请求时nginx首先会让这个配置处理请求体中header的值，当header的值超过这里设置的大小时nginx就会将请求header交给下面这条配置处理
        lsrge_client_header_buffers 21k;    #当请求体中header的值超过这里设置的大小时nginx就会返回code400错误
        client_body_buffer_size 16k;    #这里是处理客户端请求中body中的内容大小的，当body中的内容超过这里设置的大小时就会交给下面这条配置处理
        client_max_body_size 8m;    #当客户端请求中body的值超过这里设置的大小时nginx就会返回一条包含413的错误

        sendfile on;    #可以简单的理解为开启了一个高效文件传输模式。sendfile指的是nginx是否调用sendfile函数传输文件，如果不用sendfile函数发送文件会先分配一个本地的缓存区-->再将对象数据检索和复制到本地缓存区-->再从本地缓存区复制到socket缓存区。sendfille函数不用复制整个对象，而是传递了指针，直接传递给了socket缓存区，所以可以非常高效的传输文件，但是这个配置也不是适用于所有的应用场景的，一般对于普通的应用、磁盘io负载比较轻的应用可以设置为on
        tcp_nopush on;  #此配置也是和网络io相关，简单来说是为了一次性优化数据的发送量。一般和sendfile一起配合设为开启
        tcp_nodelay on; #此配置是关闭了一个防止流量堵塞算法所带来的一些暂时的死锁，大概有一个200毫秒的优化，一般和sendfile一起配合设为开启
        
        #keepalive也就是长链接，这两条是针对客户端(也就是发起请求方)设置的长链接，nginx有两种keepalive
        keepalive_requests 100000;  #这条配置是指同一条链接最大的请求数量，nginx默认设置的是100，一般都会调大这个值，特别是如果你设置了SSL，调大这个值可以明显的缩短响应时间，提高nginx的性能
        keepalive_timeout 120;  #此配置单位是秒，表示连接的空闲时间2分钟，2分钟之内的请求可以重用相同的连接
    }
### 6. server模块配置
    server {
        listen 80;  #这里表示监听本机nginx接收请求的端口
        server_name example.org www.example.org;    #这里代表请求的域名，一般会根据应用场景切割划分不同的虚拟主机，切割的方式可以基于ip或者ip+port分别配置server模块，工作中常用的是基于server_name划分，这里的值呢其实就是htpp请求过来的时候header里面host的值，这里可以是全名或者是通配符或者是正则表达式

        #这个location是对应的这个server模块分类下，对更精细的路径进行不同的配置它可以是一个本地的文件路径，也可以是其他服务器的一个服务路径，这个location后面跟的是通配符和正则表达式，这里举了三个location的例子，第一个location代表的是图片类的请求，
        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$ {
            expires 10d;    #表示这个图片类的请求的缓存时间有效期是10天
        }
        #第二个location代表所有对应这个域名的请求都会被nginx映射到本地的文件系统
        location / {
            root /usr/share/nginx/html; #这是文件系统的路径
            index index.html;   #nginx返回了一个index.html的文件
        }
        #第三个location代表所有以app开头的路径都会被反向代理到其他的服务器
        location /app/ {
            proxy_pass http://127.0.0.1:88;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-FOR $proxy_add_x_forwarded_for;
        }
    }
### 7. server模块的location路径匹配
>`=`精确匹配; `~`正则匹配，区分大小写; `~*`正则匹配，不分大小写; `~^`普通字符匹配; `^~`普通字符匹配 搜索停止; ` `默认匹配 

    #精确匹配优先级最高
    location = /app/home {
        [ configuration A ]
    }
    #正则表达式匹配优先级次之，当有多个正则匹配路径的时候判断符合这个URL路径的时候它不会马上选择B,他会继续判断下一个，判断完毕确定C是最符合这个匹配的就会选择配置C
    location /images/ {
        [ configuration B ]
    }
    location ~ /images/ {
        [ configuration C ]
    }
    location ~ /documents {
        [ configuration D ]
    }
    #如果多个URL路径匹配都是正则表达式匹配，他会按照书写的顺序选择最后一个
    location ~ /documents/Abc {
        [ configuration E ]
    }
    如果都是普通字符匹配，他会选择URL路径最长的一个
### 8. upstream模块配置
    http {
        upstream cluster1 {
            #这里将三个服务器放在一个名字叫做cluster1的upstream模块中
            server 192.168.1.163:8080;
            server 192.168.1.164:8080;
            server 192.168.1.165:8080;
        }
        server {

            #这里表示将所有以/app1/开头的URL请求反向代理到cluster1这个负载服务器集群上
            location /app1/ {
                proxy_pass http://cluster1/;
            }
        }
    }
>例 不要纠结这里面为什么有这么多一样的名字叫tomcat

    upstream tomcat {
            server host.docker.internal:8080;
            server host.docker.internal:8081;
            server host.docker.internal:8082;
        }
        server {
            listen      80;
            server_name localhost;

            location /tomcat {
                proxy_pass http://tomcat/;
            }
        }
>upstream有三种负载均衡策略

#### 轮询：顾名思义按排列顺序每请求一次换一台服务器
    upstream cluster1 {
        server 192.168.1.163:8080;
        server 192.168.1.164:8080;
        server 192.168.1.165:8080;
    }
#### 权重：顾名思义哪个权重大就优先使用哪个
    upstream cluster1 {
        server 192.168.1.163:8080 weight = 1;
        server 192.168.1.164:8080 weight = 5;
        server 192.168.1.165:8080 weight = 10;
    }
#### ip_hash：顾名思义请求中ip的hash结果决定使用哪个
    upstream cluster1 {
        ip_hash;
        server 192.168.1.163:8080
        server 192.168.1.164:8080
        server 192.168.1.165:8080
    }