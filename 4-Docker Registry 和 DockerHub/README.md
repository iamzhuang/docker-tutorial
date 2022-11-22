# Docker Registry和DockerHub
通过前几章的学习，大家已经了解了Docker是什么，掌握了Docker常用的命令，比如端口映射，Docker Volume 和 Docker compose 等等。在这一章中，我们就来学习如何使用 Docker Registry 在本地存储镜像，最后再使用 DockerHub 通过云管理容器。

## Docker Registry
Docker Registry 是存储Docker image的仓库，它在Docker生态环境中的位置如下图所示：


运行 Docker pull，Docker push时，实际上就是和 Docker Registry 通信。Docker pull 负责从 Docker registry 将存好的 image 下载到本地，而 Docker push 是将自己创建的 image 上传到 Docker registry 中，Docker registry 本质上就是存放 image 的仓库。

接下来我们试一下在本地开启 Docker Registry，首先运行以下的命令：
```
$ docker run -d -p 5000:5000 --name registry registry:2
```
然后运行以下的命令将抓下来的镜像 ubuntu，放到本地的Docker registry中：
```
$ docker pull ubuntu
$ docker tag ubuntu localhost:5000/my-ubuntu
$ docker push localhost:5000/my-ubuntu
```
接着我们将本地的镜像删除：
```
$ docker image remove ubuntu
$ docker image remove localhost:5000/my-ubuntu
```
最后再将镜像从registry中下载下来，并运行这个镜像：
```
$ docker pull localhost:5000/my-ubuntu
$ docker run -it localhost:5000/my-ubuntu
```
如果要停止并删除registry，可以使用以下命令：
```
$ docker container stop registry && docker container rm -v registry
```
## Docker Hub
Docker Hub 是一个由Docker公司运行和管理的基于云的储存库。它是一个在线存储库，Docker 镜像可以由其他用户发布和使用。

如何使用 Docker Hub 呢，首先是登入这里创建自己的 Docker ID。

然后创建一个自己的仓库：
```
登入 Docker Hub
点击在欢迎界面中点击 Create a Repository
将你的仓库命名为 <your-username>/my-private-repo
设定 visibility 为 Private
然后点击 Create，你就创建好第一个 repository 了
```
接下来就可以准备将本地的镜像扔到 Docker Hub 上了。
```
为你的项目创建好 Dockerfile
创建镜像：运行 docker build -t <your_username>/my-private-repo
测试镜像：运行 docker run <your_username>/my-private-repo
在Docker Desktop上使用Docker ID和密码登入
上传镜像到 DockerHub：运行 docker push <your_username>/my-private-repo
登入 Docker Hub，你就会看到 Tags 下有 latest 的标签。
```
若要将自己的镜像下载下来，可以遵循以下的步骤：
```
删除本地的镜像。
运行 docker pull <your_user_name>/my-private-repo
```