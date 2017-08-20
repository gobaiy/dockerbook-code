### 使用Docker测试静态网站

创建sample站点的Dockerfile
```
$ mkdir sample && cd sample
$ touch Dockerfile
```

获取Nginx配置文件
```
$ mkdir nginx && cd nginx
$ wget https://raw.githubusercontent.com/jamtur01/dockerbook-code/master/code/5/sample/nginx/global.conf
$ wget http://raw.githubusercontent.com/jamtur01/dockerbook-code/master/code/5/sample/nginx/nginx.conf
$ cd ..
```

构建新的Nginx镜像
```
$ sudo docker build -t jamtur01/nginx .
```

展示Nginx镜像的构建历史
```
$ sudo docker history jamtur01/nginx
```

下载Sample网站
```
$ mkdir website && cd website
$ wget https://raw.githubusercontent.com/jamtur01/dockerbook-code/master/code/5/sample/website/index.html
$ cd ..
```

构建第一个Nginx测试容器
```
$ sudo docker run -d -p 80 --name website \
  -v $PWD/website:/var/www/html/website \
  jamtur01/nginx nginx
```

`-v`选项允许我们将宿主机的目录作为卷，挂载到容器里。卷可以在容器间共享，即便容器停止，卷里的内容依旧存在

可以通过在目录后面加上`rw`或者`ro`来指定容器内目录的读写状态
```
$ sudo docker run -d -p 80 --name website \
  -v $PWD/website:/var/www/html/website:ro \
  jamtur01/nginx nginx
```

查看Sample网站容器
```
$ sudo docker ps website
```

在宿主机上打开浏览器访问Sample网站

修改宿主机上website目录下的index.html文件，刷新下浏览器，Sample网站内容更新为修改后的状态

### 使用Docker构建并测试Web应用程序

创建Sinatra站点的Dockerfile
```
$ mkdir sinatra && cd sinatra
$ touch Dockerfile
```

构建新的Sinatra镜像
```
$ sudo docker build -t jamtur01/sinatra .
```

下载Sinatra Web应用程序
```
$ cd sinatra
$ wget --cut-dirs=3 -nH -r --reject Dockerfile,index.html --no-parent http://dockerbook.com/code/5/sinatra/webapp/
$ ls -l webapp
```

确保webapp/bin/webapp可以执行
```
$ chmod +x webapp/bin/webapp
```

启动第一个Sinatra容器
```
$ sudo docker run -d -p 4567 --name webapp \
  -v $PWD/webapp:/opt/webapp jamtur01/sinatra
```

检查Sinatra容器的日志
```
$ sudo docker logs webapp
```

跟踪Sinatra容器的日志
```
$ sudo docker logs -f webapp
```

列出Sinatra容器的进程
```
$ sudo docker top webapp
```

检查Sinatra容器的端口映射
```
$ sudo docker port webapp 4567
```

测试Sinatra应用程序
```
$ curl -i -H 'Accept: application/json' \
  -d 'name=Foo&status=Bar' http://localhost:49160/json
```

#### 扩展Sinatra应用程序来使用Redis

下载升级版的Sinatra应用程序
```
$ cd sinatra
$ wget --cut-dirs=3 -nH -r --reject Dockerfile,index.html --no-parent http://dockerbook.com/code/5/sinatra/webapp_redis/
$ ls -l webapp_redis
```

使webapp_redis/bin/webapp文件可执行
```
$ chmod +x webapp_redis/bin/webapp
```

创建Redis镜像的Dockerfile
```
$ mkdir sinatra/redis && cd sinatra/redis
$ touch Dockerfile
```

构建Redis镜像
```
$ sudo docker build -t jamtur01/redis .
```

启动Redis容器
```
$ sudo docker run -d -p 6379 --name redis jamtur01/redis
```

检查Redis容器端口映射
```
$ sudo docker port redis 6379
```

在Ubuntu系统上安装redis客户端，并测试Redis连接
```
$ sudo apt-get install -y redis-tools
$ redis-cli -h 127.0.0.1 -p 49161
```
