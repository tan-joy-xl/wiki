

# macos系统nginx环境搭建

## nginx下载安装-通过homebrew包管理器安装

### 

#### homebrew包管理器下载

复制以下指令到终端执行
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

##### or

https://brew.sh/



#### 下载nginx

```shell
brew install nginx
```

#### 检查安装是否成功

```shell
nginx -V
```

![image-20201209151645880](/Users/tanxl/Library/Application Support/typora-user-images/image-20201209151645880.png)

安装成功后，执行nginx -V会展示出nginx的系列配置项

```shell
安装目录：--prefix=/usr/local/Cellar/nginx/1.17.3_1
启动脚本：--sbin-path=/usr/local/Cellar/nginx/1.17.3_1/bin/nginx
配置文件：--conf-path=/usr/local/etc/nginx/nginx.conf
```

检查好之后，cd到启动脚本目录/usr/local/Cellar/nginx/1.17.3_1/bin下执行nginx指令

```shell
cd /usr/local/Cellar/nginx/1.17.3_1/bin
启动：nginx
重启：nginx -s reload
```

启动后，到浏览器访问localhost:8080，出现一下页面说明nginx搭建成功。

![image-20201209153637452](/Users/tanxl/Library/Application Support/typora-user-images/image-20201209153637452.png)







## nginx配置

nginx所有的配置都在nginx.conf文件中

#### 查看配置文件nginx.conf路径

```shell
nginx -V
```

![image-20201214154735982](/Users/tanxl/Library/Application Support/typora-user-images/image-20201214154735982.png)



#### 查看||编辑 配置文件

```shell
vim /usr/local/etc/nginx/nginx.conf 
```

```shell
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```



#### URL匹配规则

```shell
location [=|~|~*|^~|@] /uri/ {
  ...
} 

= : 表示精确匹配后面的url
~ : 表示正则匹配，但是区分大小写
~* : 正则匹配，不区分大小写
^~ : 表示普通字符匹配，如果该选项匹配，只匹配该选项，不匹配别的选项，一般用来匹配目录
@ : "@" 定义一个命名的 location，使用在内部定向时，例如 error_page
上述匹配规则的优先匹配顺序：

= 前缀的指令严格匹配这个查询。如果找到，停止搜索；
所有剩下的常规字符串，最长的匹配。如果这个匹配使用 ^~ 前缀，搜索停止；
正则表达式，在配置文件中定义的顺序；
如果第 3 条规则产生匹配的话，结果被使用。否则，使用第 2 条规则的结果。
```



#### 默认server-localtion配置

```shell
server {
        listen       80; # 监听的端口，默认为80端口
        server_name  www.test.com; # 监听的ip地址，也可以是域名
        location / {
            root   html; # 路径，可在这里配置静态资源路径
            index  index.html index.htm; # 文件
        }
    }
# 用户访问 www.test.com，访问到服务器html/index.html或html/index.htm文件
# 有代理配置 proxy_pass 则会代理到代理配置地址
```



## 代理配置

#### proxy_pass代理可以代理到单个服务，也可以代理到服务集群

```shell
http {
	...
		# 配置服务集群
		upstream test {
				server localhost: 7777;
				server localhost: 8888;
				server localhost: 9999;
				...
		}
		
    server {
        listen       80;
        server_name  localhost;
        location / {
            #proxy_pass http://www.baidu.com/; # 代理到具体单个服务
            proxy_pass http://nginx_test/; # 代理到nginx_test服务集群
        }
    }
	...
 }
```

## nginx负载均衡策略（6种）

#### 轮询	默认方式

```shell
# 将请求挨个分派
upstream test {
		server localhost: 7777;
		server localhost: 8888;
		server localhost: 9999 max_fails=3 fail_timeout=20s;
		server localhost: 9000 backup;
}
# 9次请求，7777端口接收到3次；8888接收到3次；9999接收到3次；第十次请求由7777端口接收；
```

| 服务配置参数 | 解释                                                         |
| ------------ | ------------------------------------------------------------ |
| fail_timeout | 与max_fails结合使用。                                        |
| max_fails    | 设置在fail_timeout参数设置的时间内最大失败次数，如果在这个时间内，所有针对该服务器的请求都失败了，那么认为该服务器会被认为是停机了， |
| fail_time    | 服务器会被认为停机的时间长度,默认为10s。                     |
| backup       | 标记该服务器为备用服务器。当主服务器停止时，请求会被发送到它这里。 |
| down         | 标记服务器永久停机了。                                       |

#### weight	权重方式

```shell
# weight默认值为1;
upstream test {
		server localhost: 7777;
		server localhost: 8888 weight=2;
		server localhost: 9999 weight=2;
}
# 10次请求，7777--2次；8888、9999--4次；
```



#### ip_hash	依据ip分配方式

```shell
# 指定负载均衡器按照基于客户端IP的分配方式
upstream test {
		ip_hash; # 保证每个访客固定访问一个后端服务器
		server localhost: 7777;
		server localhost: 8888;
		server localhost: 9999;
}
# 这个方法确保了相同的客户端的请求一直发送到相同的服务器，以保证session会话。这样每个访客都固定访问一个后端服务器，可以解决session不能跨服务器的问题。
```

#### least_conn	最少连接方式

```shell
# 把请求发送给连结数较少的后端服务器。
# 轮询是将请求平均分发给每个服务器，使得它们的负载大致相同，但是有的请求占用的时间比较长，分到较长占用的请求的服务器不能及时处理，会造成该服务器负载较高。这种情况下，使用这种策略可以达到更好的负载均衡效果。
upstream test {
		least_conn; # 把请求转发给连接数较少的后端服务器
		server localhost: 7777;
		server localhost: 8888;
		server localhost: 9999;
}
# 此负载均衡策略适合请求处理时间长短不一造成服务器过载的情况。
```

#### fair（第三方）	响应时间方式

```
upstream test {
		server localhost: 7777;
		server localhost: 8888;
		server localhost: 9999;
		fair; # 实现响应时间短的优先分配
}
```

#### url_hash（第三方）	依据URL分配方式

```
upstream test {
		hash $request_uri; # 实现每个url定向到同一个后端服务器
		server localhost: 7777;
		server localhost: 8888;
		server localhost: 9999;
}
```



## 最后附上一个较为完整的配置文件以便大家参考

```sh
########### 每个指令必须有分号结束。#################
#user administrator administrators;  #配置用户或者组，默认为nobody nobody。
#worker_processes 2;  #允许生成的进程数，默认为1
#pid /nginx/pid/nginx.pid;   #指定nginx进程运行文件存放地址
error_log log/error.log debug;  #制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emerg
events {
    accept_mutex on;   #设置网路连接序列化，防止惊群现象发生，默认为on
    multi_accept on;  #设置一个进程是否同时接受多个网络连接，默认为off
    #use epoll;      #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
    worker_connections  1024;    #最大连接数，默认为512
}
http {
    include       mime.types;   #文件扩展名与文件类型映射表
    default_type  application/octet-stream; #默认文件类型，默认为text/plain
    #access_log off; #取消服务日志    
    log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
    access_log log/access.log myFormat;  #combined为日志格式的默认值
    sendfile on;   #允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。
    sendfile_max_chunk 100k;  #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上限。
    keepalive_timeout 65;  #连接超时时间，默认为75s，可以在http，server，location块。

    upstream mysvr {   
      server 127.0.0.1:7878;
      server 192.168.10.121:3333 backup;  #热备
    }
    error_page 404 https://www.baidu.com; #错误页
    server {
        keepalive_requests 120; #单连接请求上限次数。
        listen       4545;   #监听端口
        server_name  127.0.0.1;   #监听地址       
        location  ~*^.+$ {       #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
           #root path;  #根目录
           #index vv.txt;  #设置默认页
           proxy_pass  http://mysvr;  #请求转向mysvr 定义的服务器列表
           deny 127.0.0.1;  #拒绝的ip
           allow 172.18.5.54; #允许的ip           
        } 
    }
}
```

