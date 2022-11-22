# Contents  
1. 前后端项目
2. redis项目
3. Docker-compose 常用命令整理
   
在上一章中，我们学习了常用的Docker命令，以及如何运行由多个容器组成的项目。当一个容器变得复杂之后，很可能需要类似以下的命令来运行容器：
```
docker run \
  --detach \
  --name registry \
  --hostname registry \
  --volume $(pwd)/app/registry:/var/lib/registry \
  --publish 5000:5000 \
  --restart unless-stopped \
  registry:latest
```
如果我们有多个容器，若每次都要输入这些命令就很麻烦了。Docker为了解决更好管理多个容器的问题，就推出了 Docker Compose 这个项目，帮助我们更方便地定义和运行多个容器。

Docker Compose 很好使用，我们只需要将所有容器的设置放到一个单独的 docker-compose.yml 文件中。本章我们就通过两个项目来学习Docker Compose，帮助我们深入掌握Docker Compose。首先大家先下载公开的repo，代码都在里面：
```
$ git clone https://github.com/turingplanet/docker-tutorial.git
$ cd docker-tutorial/3-docker-compose/app1
```
## 前后端项目
在第一个项目中，我们有两个容器，一个是backend负责启动服务器，开启后端端口。第二个是frontend，实现前端的展示。如果想要同时启动两个容器，我们可以将设置写在 docker-compose.yml 中：
```
version: "3.9"

services:

  backend:  
    build: ./backend
    ports: 
      - "5000:5000"
    volumes:
      - ./backend:/app

  frontend:
    build: ./frontend
    ports:  
      - "3000:3000"
```
其中参数的含义是：

version “3.9”：代表我们使用的是Docker Compose的3.9版本。

services：这个区域指定了我们会创建的容器，在这个例子中，我们有两个服务，backend和frontend。

backend：这是后端程序的名字。

build：指定了 Dockerfile 的路径，.backend 代表在 docker-compose.yml 所在目录下的 backend 文件夹中。

ports：指定本机端口和容器端口的映射关系。

volumes：制定挂载的参数，相当于 -v 之后的内容。

如果要构建镜像 (image)，只需要输入 docker-compose build 命令，紧接着使用 docker-compose up 启动容器：
```
$ docker-compose build
$ docker-compose up
```
如果要启动特定的服务只需要在 docker-compose up 后面填上服务的名字即可：
```
$ docker-compose up <service1> <service2>
$ docker-compose up backend
```
如果要关闭容器，输入 docker-compose stop 即可。如果要查看当前容器的状态，只需要输入 docker-compose ps：
```

     Name              Command         State     Ports
------------------------------------------------------
app1_backend_1    python3 server.py   Exit 137        
app1_frontend_1   python3 server.py   Exit 137  
```
## redis项目
在第二个项目中，我们会联动两个服务：
```
version: "3.9"

services:

  redis-server:
    image: "redis"

  backend:  
    environment:
      - REDIS_HOST=redis-server
    build: ./backend
    ports: 
      - "5000:5000"
```
第一个服务是 redis 数据库，第二个服务是后端服务器，backend 中有一个环境变量 REDIS_HOST 被设置为 redis-server，这样后端的程序就能连到 redis 容器了：
```
from flask import Flask, render_template
import redis
import os
REDIS_HOST = os.environ.get('REDIS_HOST', '0.0.0.0')

app = Flask(__name__)
cache = redis.Redis(host = REDIS_HOST, port = 6379, db = 0)
cache.set('hits', 0)

@app.route('/')
def get_data():
    hit_num = cache.incr('hits')
    return f'I have been seen {hit_num} times.\n'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```
如果要启动容器，只需要输入 docker-compose up 即可，然后打开游览器，输入 localhost:5000 就能看到后端的信息。

## Docker-compose 常用命令整理
```
$ docker-compose build # Builds the images, does not start the containers.
$ docker-compose up # Builds the images if the images do not exist and starts the containers.
$ docker-compose stop # Stops your containers, but it won't remove them.
$ docker-compose down # Stops your containers, and also removes the stopped containers as well as any networks that were created.
```