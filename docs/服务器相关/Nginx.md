### 

## 安装nginx

```shell
# 安装
yum install nginx

```

## nginx常见指令

```shell
# 启动nginx
systemctl start nginx 
# 查看nginx状态
systemctl status nginx
# 设置开机启动nginx
systemctl enable nginx
# 重启nginx
systemctl restart nginx
```

## nginx配置多个前端项目（直接访问打包之后的index.html）

```nginx
# 配置history的项目一 可以是vue或react
server {
  listen       80;
  listen       [::]:80;
  server_name  vue.huangjianxin.cn;  # 访问的域名
  # root         /usr/share/nginx/html;

  location / {
    root /root/vue;
    index index.html;
    try_files $uri $uri/ /index.html;  // 关键代码其它请求默认找回index.html 配置history路由
  }

  # Load configuration files for the default server block.
  include /etc/nginx/default.d/*.conf;

    error_page 404 /404.html;
        location = /404.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
    
# 配置history的项目二 可以是vue或react
server {
  listen       80;
  listen       [::]:80;
  server_name  react.huangjianxin.cn;  # 访问的域名
  # root         /usr/share/nginx/html;

  location / {
    root /root/react;
    index index.html;
    try_files $uri $uri/ /index.html;  // 关键代码其它请求默认找回index.html 配置history路由
  }

  # Load configuration files for the default server block.
  include /etc/nginx/default.d/*.conf;

    error_page 404 /404.html;
        location = /404.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```

## 基于端口配置

```nginx
server {
        listen 8000;
        
        location / { 
                root /data/web-a/dist;
                index index.html;
        }
}

# nginx 80端口配置 （监听a二级域名）
server {
        listen  80;
        server_name a.fly.com;
        
        location / {
                proxy_pass http://localhost:8000; #转发
        }
}
```
