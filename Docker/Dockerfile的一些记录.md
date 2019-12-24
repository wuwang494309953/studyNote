#### Mysql

**1.**创建mysql的时候执行一段sql脚本

```dockerfile
FROM mysql:5.7
COPY ./saber_authr.sql /docker-entrypoint-initdb.d
```

emmm,docker的mysql容器下有个**docker-entrypoint-initdb.d**的目录，只要将脚本放进去就会在启动时执行了



#### Nginx

**1.**nginx部署前端项目

**dockerfile**

 ```dockerfile
FROM nginx
#配置文件替换一下
ADD ./conf.d/default.conf /etc/nginx/conf.d/
#前端资源复制一下(这里是vue的包)
COPY ./dist /dist
 ```

**conf.d/default.conf**

```conf
upstream zuul1  {
    server zuul:8080;
}

server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location /ng {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

	# 项目配置
    location / {
        root   /dist;
        index  index.html index.htm;
    }
    
    # 请求转发
    location /v1 {
    	proxy_pass http://zuul1/v1;
        proxy_set_header Host $host;
    }
}
```

