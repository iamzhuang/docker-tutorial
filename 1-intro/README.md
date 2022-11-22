# Contents  
1. Docker是什么？
1.1. 麻烦的环境部署
1.2. 虚拟机（Virtual Machine）
1.3. Linux容器（Container）
1.4. Docker是什么？
1.5. Docker的用途
2. 运行第一个Docker容器
2.1. 安装Docker
2.2. image文件
2.3. 官方镜像 Hello World
2.4. 建立自己的容器
3. 【Docker教程】大纲
Docker自从2013年诞生至今，早已成为了开发者必不可少的一项技术。但很多人并不清楚Docker是什么，解决了哪些问题，好处是什么？本章就来详细解释，帮助大家更好地理解Docker，创建并运行第一个Docker容器。

## Docker是什么？
### 麻烦的环境部署
在软件开发中，最麻烦的事情之一就是环境配置。在正常情况下，如果要保证程序能运行，我们需要设置好操作系统，以及各种库和组件的安装。

举例来说，要运行一个Python程序，计算机必须要有 Python 引擎，还需要安装好程序的各种依赖，甚至还要配置特定的环境变量。假设你有两个程序都需要部署在同一个服务器上，一个需要软件是基于Python2.0，一个是Python3.0，那么在部署上就很容易造成混乱。因为不同版本的Python模块可能互不兼容，况且不同开发环境上的库也需要额外的配置。如果要部署很多程序，而开发环境和部署环境各不相同的话，可想而知配置得多么麻烦。

为了更好地将软件从一个环境移植到另一个环境上，必须从根源上解决问题，那么如何在移植软件的时候，将一模一样的原始环境迁移过来呢？

### 虚拟机（Virtual Machine）
虚拟机是移植环境的一种解决方案。虚拟机本质上也是一个软件，在这个软件中，我们可以运行另一种操作系统。比如我们想要在 MacOS 上运行 Linux 系统，我们就在电脑上安装 Linux 镜像，并使用虚拟机打开此镜像，就能创建出一个镜中镜了。这个方案非常方便，想要新环境，就安装镜像，然后使用虚拟机打开，不想要直接删除。但是这个方案有几个缺点：

占用资源多：虚拟机需要安装整个操作系统，自然会消耗大量内存和硬盘空间。如我们只需要运行1MB的软件，有时候也不得不安装几个G的环境才能运行。
运行步骤冗余：虚拟机安装的是完整的系统，每次运行程序都需要按部就班，打开系统、登入用户等等之类麻烦的步骤，很不方便。
运行速度慢：为了运行特定环境中的软件，虚拟机必须先运行系统，而系统占用的资源往往很多（网络，GUI，IO等等），自然也会影响运行速度。
### Linux容器（Container）
为了解决虚拟机存在的这些缺点，Linux发展出了另一种虚拟化的技术：Linux容器。Linux 容器不是模拟一个完整的操作系统，而是对进程进行隔离。或者说，就是在正常进程的外面套用了一个保护层。对于容器里面的进程来说，它接触到的各种资源都是虚拟的，从而实现与底层程序的隔离。由于容器是进程级别的，相比虚拟机有更多优势：

占有资源少：容器只占用需要的资源，不占用那些用不到的资源。相比于虚拟机安装完整的操作系统，容器需要消耗的空间自然就少了很多。
资源利用率高：虚拟机都是独享资源，电脑需要为每个虚拟环境单独分配资源，不仅仅占用空间大，而且资源的利用率很低。而容器之间可以共享资源，最大化资源的利用率。
运行速度快：容器里面的应用就是底层系统的一个进程，所以启动容器相当于直接运行本机的一个进程，而不是一个完整并臃肿的操作系统，自然就快很多。
### Docker是什么？
Docker属于Linux容器的一种封装，提供简单易用的容器使用接口，它也是目前最流行的Linux容器解决方案。Docker 将软件代码和其依赖，全打包在一个文件中。运行单个文件，就会生成虚拟容器。在这个虚拟容器中，不管本地的操作系统是如何的不同，此容器都能照常运行。

简而言之，Docker的接口非常简单，可以帮助用户更好地创建和使用容器，让相同的代码在不同的环境上正常运行。

### Docker的用途
Docker目前主要有以下三个用途：

提供一次性的环境：本地测试别人的软件、持续集成的时候提供单元测试和构建的环境。
提供弹性的云服务：因为Docker容器可以随时启动或关闭，所以非常适合动态规划和缩容。
组建微服务构架：通过多个容器，服务的部署能更加灵活，帮助实现微服务构架。
## 运行第一个Docker容器
### 安装Docker
Docker是一个开源的商业产品，有两个版本：社群版（Community Edition）和企业本（Enterprise Edition）。企业版中包含了一些收费服务，个人开发者一般用不到。下面我们就来下载并使用社区版。

Docker CE 的安装可以参看官网网站：https://docs.docker.com/get-docker/，请根据自己操作系统的类型选择相对应的版本。

安装完成后，运行下面的命令，验证是否安装成功。
```
$ docker version
```
确认 Docker 安装完毕后，我们可以使用以下命令运行 Docker：
```
$ sudo service docker start
```
使用 Mac 的朋友直接打开下载好的 Docker 软件即可。

### image文件
Docker 把应用程序及其依赖，打包在 image 文件里面。只有通过这个文件，才能生成 Docker 容器。image 文件可以看作是容器的设计蓝图。Docker 根据 image 文件生成容器的实例。同一个 image 文件，可以生成多个同时运行的容器实例。

image 是二进制文件，在实际开发中，一个 image 文件往往通过继承另一个 image 文件，加上一些个性化设置而生成。举例来说，你可以在 Ubuntu 的 image 基础上，往里面加入 Apache 服务器，形成你的 image。

#### 列出本机所有的 image 文件
```
$ docker image ls
```
#### 删除特定的 image
```
$ docker image rm [imageName]
```
image 文件是通用的，一台机器的 image 文件拷贝到另一台机器，照样可以使用。一般来说，为了节省时间，我们应该尽量使用别人制作好的 image 文件，而不是自己制作。即使要定制，也应该基于别人的 image 文件进行加工，而不是从零开始制作。

为了方便分享，image 文件制作完成后，可以上传到网上的仓库。Docker 的官方仓库 Docker Hub 是最重要、最常用的 image 仓库。此外，公开分享自己制作的 image 文件也是可以的。

### 官方镜像 Hello World
下面，我们通过最简单的 image “hello word”，来感受一下 Docker 的易用性。

首先我们运行下面的命令，将 image 从仓库抓取到本地：
```
$ docker image pull hello-world
```
抓取成功以后，使用下面的指令，就能在本机看到这个 image 文件了：
```
$ docker image ls
```
运行这个 image：
```
$ docker container run hello-world
```
docker container run 命令会根据 image 的设定，生成一个正在运行的容器实例。

注意，docker container run 命令具有自动抓取 image 文件的功能。如果发现本地没有指定的 image 文件，就会从云端仓库自动抓取。因此，前面的 docker image pull 命令并不是必需的步骤。如果运行成功，你就会在屏幕上读到类似下面的输出：
```
$ docker container run hello-world
```
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
......
```
输出这段提示以后，hello world 就会停止运行，容器自动终止。

有些容器不会自动终止，比如一些容器提供的是服务：像是Ubuntu的image，就可以在命令行体验Ubuntu系统：
```
$ docker container run -it ubuntu bash
```
对于那些不会自动终止的容器，我们可以使用 docker container kill 命令终止。首先我们可以使用 docker container ls 找到你要终止容器的id，然后使用 kill 命令终止容器：
```
$ docker contianer ls 

$ docker container kill [containerId]
```
### 建立自己的容器
学会使用官方的 image 文件以后，接下来的问题就是，如何生成自己的 image 文件？这里就需要使用 Dockerfile 文件，Dockerfile 是一个文本文件，用来配置 image 的具体内容。Docker 会根据该文件生成二进制的 image 文件。

下面我们在本地编辑一个 Python 文件 hello.py，这个文件的功能是使用 Python 打印出一行字符串：
```
print('Hello World :)')
```
然后，我们可以在这个项目的相同路径中，新建一个文本文件 .dockerignore，写入以下内容：
```
__pycache__
env
```
这段代码表示，__pycache__ 和 env 这两个路径被排除，不会被打包进入 image 文件。如果你没有路径要排除，这个文件可以不用建。

然后，在项目的根目录下，新建一个文件文件 Dockerfile，写入下面的内容：
```
FROM python3.
COPY . /app
WORKDIR /app
python3 main.py
```
上面的代码一共五行，含义如下：

#将 image 文件继承与官方的3.7版本的Python
```
FROM python:3.7
```
#将当前目录下的所有文件（除了 .dockerignore 排除的路径），都拷贝进 image 文件的 /app 目录。
```
COPY . /app
```
#指定接下来的工作路径为 /app
```
WORKDIR /app 
```
#在 /app 目录下，运行 python 文件
CMD python3 hello.py 
有了 Dockerfile 之后，就可以使用 docker image build 命令创建 image 文件了：
```
$ docker image build -t python-app .
or 
$ docker image build -t python-app:0.0.1 .
```
这段命令行的意思是，-t 参数用来指定 image 文件的名字，后面还可以用冒号指定标签。如果不指定，默然的标签是latest。最后的那个点表示 Dockerfile 文件使用的路径，上面的例子是当前路径，所以是一个点。

如果运行成功，就能使用 docker image ls 看到新的image文件了。

创建好 image 之后，就能使用 docker container run 运行 image，并生成容器了。
```
$ docker container run python3-app
```
```
Hello World :)
```
如果一切正常，运行上面的命令后，就会返回输出内容，这就代表运行完成啦。

## 【Docker教程】大纲
相信大家读到这里，已经对Docker有一个基本的了解，那么之后还会有三章关于Docker更多详细的介绍，帮助大家更好地掌握Docker技术，并在教程结束后，能够将它用于日常开发。以下便是接下来三章的内容：

### Docker常用命令：

使用Docker为Python后端服务建立容器，了解更多Docker常用的命令

Docker Compose：学习使用Docker Compose运行包含多容器的Docker应用

DockerHub：学习DockerHub，将本地容器放入云端，更好地管理和发布容器