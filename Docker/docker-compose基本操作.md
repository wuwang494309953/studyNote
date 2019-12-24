### Docker-compose

​		记录一个自己写的项目编写的`docker-compose.yml`

**docker-compose.yml**

```yaml
version: '3'
services:
  eureka-docker:
    image: 119.23.29.13/saber/saber-eureka:latest
    ports:
      - '8501:8501'
    volumes:
      - '/www/logs:/www/logs'
  saber-auth:
    image: 119.23.29.13/saber/saber-authr-service:latest
    depends_on:
      - eureka-docker
    volumes:
      - '/www/logs:/www/logs'
  zuul:
    image: 119.23.29.13/saber/saber-zuul:latest
    depends_on:
      - saber-auth
    ports:
      - '8070:8080'
    volumes:
      - '/www/logs:/www/logs'
  web:
    image: 119.23.29.13/saber/saber-web:v2
    depends_on:
      - zuul
    ports:
      - '8071:80'
  saber-mysql:
    image: 119.23.29.13/saber/saber-mysql:v2
    restart: always
    ports:
      - '3306:3306'
    command: [
      '--character-set-server=utf8mb4',
      '--collation-server=utf8mb4_unicode_ci'
    ]
    environment:
      MYSQL_ROOT_PASSWORD: 123456
```

---

#### 指令

**build**	-	构建容器

**可选参数**

> --force-rm              Always remove intermediate containers.
> -m, --memory MEM        Set memory limit for the build container.
> --no-cache              Do not use cache when building the image.
> --no-rm                 Do not remove intermediate containers after a successful 
> --pull                  Always attempt to pull a newer version of the image.

**config**	-	验证 Compose 文件格式是否正确

**down**	-	此命令将会停止 up 命令所启动的容器，并移除网络

**kill**	-	停止容器

**logs**	-	日志打印

**up**	-	该命令十分强大，它将尝试自动完成包括构建镜像，（重新） 创建服务，启动服务，并关联服务相关容器的一系列操作。链接的服务都将会被自动启动，除非已经处于运行状态