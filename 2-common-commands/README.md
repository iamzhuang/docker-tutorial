# Contents   
1. 容器参数
2. 容器端口映射 Container Ports
3. Docker 数据管理（Manage Data in Docker）
   
    3.1. 挂载 Bind Mount

    3.2. 数据卷 Docker Volume
4. Docker 命令总结
关于 Docker Image 和 Container 的基本概念，已经在上一章介绍过了，在本章中，我们通过运行由不同容器组成的三个项目，进一步掌握Docker最常用的命令。

### 容器参数
首先通过一个小项目复习一下如何建立一个Docker容器。先下载一个公开的GitHub Repo：
```
git clone https://github.com/turingplanet/docker-tutorial.git
cd docker-tutorial/2-common-commands/pyramid
```
其中有两个文件，第一个是 pyramind.py：
```
import sys

def print_pyramid(level: int):
    for i in range(level):
        new_line_str = ''
        new_line_str += ' ' * (level - 1 - i)
        new_line_str += '*' * ((i+1) * 2 - 1)
        print(new_line_str)

input_level = int(sys.argv[1])
print_pyramid(input_level)
```
此文件可以根据运行时传入的参数，打印出不同层数的金字塔，另一个文件是 Dockerfile：
```
FROM python:3.7
COPY ./pyramid.py /pyramid.py
ENTRYPOINT ["python3", "pyramid.py"]
CMD ["5"]
```
此文件规定了如何为容器设置环境，Docker Engine通过分析Dockerfile的内容，按部就班地创建Docker Image。

Dockerfile 第三行 ENTRYPOINT 命令的目的和 CMD 一样，都是在指定容器启动时的程序和参数，而 CMD 指定的是容器的默认命令，其命令会被覆盖。

在这个容器中，如果没有输入额外命令的话，容器最后会默认运行
```
python3 pyramid.py 5：
```
```
$ docker build . --tag pyramid
$ docker run pyramid
```
运行后，命令台会自动打印出五层的金字塔：
```

    *
   ***
  *****
 *******
*********
```
如果我们运行时定义好参数，那么就会打印出与参数对应层数的金字塔：
```
$ docker run pyramid 7
```
```
      *
     ***
    *****
   *******
  *********
 ***********
*************
```
以上就是对运行自定义容器和设定默认命令的基本内容，接下来学习对应本地端口和容器中的端口。

### 容器端口映射 Container Ports
如果一个容器需要暴露端口才能运行，那么我们需要将本地端口映射到容器的端口后，才能从本地访问到程序。以下的命令则会把 my-app 容器中的端口 3000 映射到本地端口 5000：
```
$ docker run -p 5000:3000 my-app
```
接下来通过两个容器来深入理解端口映射，一个是前端程序，一个是后端程序，源代码存在 docker-tutorial/2-common-commands/webapp 下。

后端程序存在 backend 中，其中有四个文件，book.json 储存一本书籍的信息：
```
{
    "title": "Book Title",
    "description": "Description",
    "link": "turingplanet.org"
}
```
而 server.py 负责开启后端 API：
```
from flask import Flask, render_template
from flask_cors import CORS # need to mention
import json

app = Flask(__name__)
CORS(app)

@app.route('/book')
def json_file():
    file = open('./book.json')
    json_data = json.load(file)
    return json_data

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```
此文件会将本地的端口5000打开，开启服务器后，如果在游览器中输入 localhost:5000/book，那么就会获得 json 文件中的内容。

另外两个文件中， requirements.txt 定义了此Python程序需要安装的依赖，而 Dockerfile 定义了创建这个后端容器的具体步骤：
```
FROM python:3.7
COPY . /app 
WORKDIR /app 
RUN ["pip3", "install", "-r", "requirements.txt"]
EXPOSE 5000
CMD ["python3", "server.py"]
```
Dockerfile中的第四行RUN命令，目的是安装此程序需要的依赖，接下来一行的命令 EXPOSE 5000 表示将端口 5000 打开，最后的 CMD 命令运行 server 程序。

请使用以下的命令创建Docker image：
```
$ docker build . --tag web-app-backend
```
然后运行通过映射端口运行我们的容器：
```
$ docker run -p 4000:5000 web-app-backend
```
运行后，容器中的 5000 端口会对应到本地的 4000 端口。这个时候如果打开本地游览器，然后输入 localhost:4000/book ，就会返回书籍的信息。

另一个程序是前端程序，在 2-common-commands/webapp/frontend 目录下，其中包含了一个文件夹和三个文件：templates, server.py, requirements.txt, 和 Dockerfile。其中 server.py 负责将 templates 中的html 文件传给服务器的根目录：
```
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def home():
    return render_template('index.html')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=3000)
Dockerfile 中规定了将容器的端口 3000 打开，并运行 server.py：

FROM python:3.7
COPY . /app 
WORKDIR /app 
RUN ["pip3", "install", "-r", "requirements.txt"]
EXPOSE 3000
ENTRYPOINT ["python3", "server.py"]
```
要注意的是，index.html中会从 localhost:5000/book 路径中获得书籍的标题并展示在网页中。

所以如果你使用以下的命令运行前端程序的话，可以在游览器中输入 localhost:3000 看到网页，但网页中是不会显示书籍的标题，因为后端程序运行在本地的端口是4000：
```
$ docker build . --tag web-app-frontend
$ docker run -p 3000:3000 web-app-frontend
```
如果要让网站正常显示书籍的标题，首先停止后端的容器，再使用以下的命令重新开启后端容器：
```
$ docker run -p 5000:5000 web-app-backend
```
此时打开游览器，输入 localhost:3000 ，网站就会正常显示书籍标题。

### Docker 数据管理（Manage Data in Docker）
Docker container 并不能永远储存数据，一旦容器被停止，那么其中的文件也无法被访问了。同时，如果我们想要把一个容器中的数据转移到其他容器中也是非常困难的。为了解决在容器中无法存储永久数据的问题，Docker 提供了两种解决方案，一种是 Bind mount，另一种是 volume。

#### 挂载 Bind Mount
使用 Bind Mount 可以将本地的一个文件夹与 Docker 容器中的另一个文件夹同步，就是说对本地指定的文件夹上做的任何操作都会同步到容器中的文件夹，反之亦然。

那么就通过最后一个程序来了解挂载，此项目是在 2-commond-commands/volume 文件夹下面，其中有两个文件夹和三个文件。文件夹 templates 中包含前端页面的内容，而 volume_data 中有一个存储书籍信息的 JSON 文件。在三个文件中，Dockerfile 包含了建立容器的步骤，requirementext.txt 指定了需要安装的依赖信息，server.py 负责服务器的创建：
```
from flask import Flask, render_template
import json

app = Flask(__name__)

@app.route('/book')
def json_file():
    file = open('./volume_data/book.json')
    json_data = json.load(file)
    return json_data

@app.route('/')
def home():
    return render_template('index.html')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=3000)
```    
然后使用以下命令创建镜像，并运行容器：
```
$ docker build . --tag volume-app-frontend 
$ docker run -p 3000:3000 -v /Users/enoch/code_hub/1.turing_planet/docker-tutorial/2-common-commands/volume/volume_data:/app/volume_data volume-app-frontend
```
要注意到的是，如果要让本地文件夹与容器中的文件夹同步，则需要在运行时使用 -v <local_folder_path>:<container_folder_path> 参数，定义好同步文件夹的绝对路径和容器文件夹的路径。

运行成功后，打开游览器，输入 localhost:3000 就会显示网页，此时如果修改 -v 指定的本地文件夹中 volume_data/book.json 中书籍标题的内容，网页显示的内容也会改变。

#### 数据卷 Docker Volume
另一种存储数据的方式是 Docker Volume，和 Bind Mount 不同的是，我们不需要在本地提前建好文件夹，只需要指定 volume 的名字，Docker 则会自动创造出一个文件夹，用来储存 volume 对应的数据。

我们可以使用 volume create 创建 volume，使用 inspect 查看 volume 的具体信息：
```
$ docker volume create book-data
$ docker volume inspect book-data
[
    {
        "CreatedAt": "2020-12-18T20:33:53Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/book-data/_data",
        "Name": "book-data",
        "Options": {},
        "Scope": "local"
    }
]
```
如果想要让 Docker 容器和 volume 同步，只需要使用 -v <volume_name>:<container_folder_path> 定义好 volume 名字和容器文件夹的位置：
```
$ docker run -p 3000:3000 -v book-data:/app/volume_data volume-app-frontend
```
如果想要改变 volume 中的文件内容，则需要开启 ubuntu 容器并与之相同的 volume 同步，然后安装编辑器修改其中的文件即可：
```
$ docker run -v book-data:/book-data -it ubuntu
$ root: apt update
$ root: apt install vim
$ root: vim /book-data/book.json
```
在容器中修改书籍信息后，volume 内的数据也会改变，随之程序容器内的文件内容也会改变，所以最后网页的内容显示也会不同。

### Docker 命令总结
以上就是对 Docker 常用功能的介绍，以下是其他一些常见命令的总结：

####  Images
```
$ docker build --tag my-app:1.0 .
$ docker images # listing images
$ docker save -o <path_for_generated_tar_file> <image_name> # Save the image as a tar file
$ docker load -i <path_to_tar_file> # Load the image into Docker
$ docker rmi my-app:1.0 # Remove a docker image
$ docker rmi --force my-app:1.0 
$ docker rmi $(docker images -a -q) # Remove all images that are not associated with existing containers
$ docker rmi $(docker images -a -q) -f # same as above, but forces the images associated with running containers to be also be removed
```
####  Containers
```
$ docker run my-app:1.0
$ docker ps # view a list of running containers
$ docker ps -a # includes stopped containers
$ docker run -p 8000:3000 my-app:1.0 # Exposes port 3000 in a running container, and maps to port 8000 on the host machine
$ docker rm $(docker ps -a -q)  # removes all containers
$ docker rm $(docker ps -a -q) -f  # same as above, but forces running containers to also be removed
```
####  Volumes
```
$ docker run -d --name my-app -v volume-name:/usr/src/app my-app:1.0
$ docker run -p 3000:3000 -v <load_absolute_path>:<docker_absolute_path> volume-app-frontend
$ docker volume ls 
$ docker volume prune # Removes unused volumes
```