# 《PyTorch自动驾驶...》第八章
---

## 2026_03_16
先跳过前面章节，直接先学习这一章，因为即将入职地平线，相关工作有联系。

### Docker
关于Docker一直都疏于练习，同时也缺少相关的学习环境。

Docker的使用主要分为三个步骤：**Docker Repository**/ **Docker Image**/ **Docker Container**

Docker Repository类似于Github的仓库，同样可以**pull**和**push** **Docker Images**。

Docker Image可以认为是一个软件环境的模版。
例如，有一个深度学习论文的实现代码，要求在`Ubuntu 16.04 CUDA 7.5 PyTorch 0.4.1 和 Numpy 1.16`环境下运行。
即使本地系统是`Ubuntu 20.04`，利用Docker也可以创建一个复合要求的Docker Image。

至于`Docker Container`，可以理解成一个`Docker Image`的实例 Instance。

### Docker Image 管理
Docker Image的管理主要有两种方式：**Dockerfile**和**Docker Hub**。

#### Dockerfile
Dockerfile是一个文本文件，包含了一系列的命令和指令，用于构建Docker Image。通过编写Dockerfile，可以定义所需的软件环境、依赖关系和配置，从而自动化地创建Docker Image。
例如，以下是一个简单的Dockerfile示例：
```Dockerfile
FROM nvidia/cuda:11.3.1-base-ubuntu20.04
RUN apt update
RUN apt install -y python3
RUN apt install -y python3-pip
RUN python3 -m pip install --upgrade pip
RUN pip install numpy
RUN pip3 install torch torchvision --extra-index-url https://download.pytorch.org/whl/cu113
```
将上面的代码复制到一个名为`Dockerfile`的记事本中，然后在终端中运行以下命令来构建Docker Image：
```bash
docker build .
```
也可以使用一下命令指定具体的Dockerfile进行构建：
```bash
docker build -f /home/luan/Desktop/Dockerfile .
```
Docker会优先在Dockerhub上查找是否存在符合要求的Docker Image，如果存在则直接下载并使用；如果不存在，则会根据Dockerfile中的指令逐步构建Docker Image。

为本地的Docker Image打上tag：
```bash
docker build -t pytorch:0.4.1 .
```
按照惯例，Docker Image的命名格式为`<repository>:<tag>`，其中`repository`是镜像的名称，`tag`是镜像的版本号或标签。

#### 管理Docker Image
可以使用以下命令来管理Docker Image：
- 列出本地的Docker Image：
```bash
docker images
```
执行此命令后，会显示本地所有的Docker Image，包括它们的仓库名称、标签、镜像ID、创建时间和大小等信息。
例如下面：
```
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
<none>             <none>               123456789abc        2 hours ago             1.2GB
test                latest              123456789abc        2 hours ago             1.2GB
pytorch             0.4.1               123456789abc        2 hours ago             1.2GB
```
不给Image标注Tag也是可以的，但是每个Image都会被分配一个唯一的镜像ID。

有些镜像的体积是很大的，可以通过以下命令删除：
```bash
docker image rm <image_id>
# 旧版命令： docker rmi <image_id>
```
除了通过`Dickerfile`构建Docker Image之外，还可以直接从Docker Hub上拉取现成的Docker Image：
```bash
docker pull pytorch/pytorch:0.4.1-cuda7.5-cudnn7-runtime
```

### Docker Container 管理
开启一个新的`Docker Container`：
```bash
# -it参数：-i表示交互式，-t表示分配一个伪终端
docker run -it test:2.0 /bin/bash
```
这条命令会基于`test:2.0`这个Docker Image创建一个新的Docker Container，并在其中启动一个交互式的bash终端。
换句话说，用户已经穿越到了这个Docker Container中，可以在其中执行各种命令，就像在本地系统中一样。
例如，在Docker Container中可以执行以下命令来查看当前的系统信息：
```bash
uname -a
```
这条命令会显示当前系统的内核版本、操作系统类型和其他相关信息。

如果不用`-it`参数，直接运行以下命令：
```bash
docker run test:2.0 ls
```
用户会看到执行结果，但无法进入到Docker Container中停留，命令执行完后就会推出Docker Container。

要查看当前正在运行的Docker Container，可以使用以下命令：
```bash
docker ps
# -a参数：显示所有的Docker Container，包括正在运行的和已经停止的
docker ps -a
```
这条命令会列出所有正在运行的Docker Container，包括它们的容器ID、镜像名称、命令、创建时间、状态和端口等信息。
例如下面是一个示例输出：
```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
123456789abc        test:2.0            "/bin/bash"         2 hours ago         Up 2 hours                              test_container
```
在已打开的Docker Container中，仍可以使用以下命令来运行`ls`命令：
```bash
docker exec <container_id> ls
```
简而言之，`docker run`命令用于创建和启动一个新的Docker Container，而`docker exec`命令用于在已经运行的Docker Container中执行命令。

### 在Docker Container中加载本地文件夹
Docker Container模仿ssh的方式和本地硬盘进行数据交换。
在运行一个新的Docker Container时，可以使用`-v`参数来挂载 (Mount) 本地文件夹到Docker Container中。

例如，想要将本地目录`/home/luan/Desktop`挂载到Docker Container中的`/workspace`目录，可以使用以下命令：
```bash
docker run -it -v /home/luan/Desktop:/workspace test:2.0 /bin/bash
```
执行以上命令后，文件系统内出现了一个新目录`/workspace`，里面正是本地目录`/home/luan/Desktop`的内容。
在Docker Container中对`/workspace`目录进行的任何修改都会直接反映到本地目录`/home/luan/Desktop`中，反之亦然。
这种挂载方式非常方便，可以让用户在Docker Container中直接访问和修改本地文件，而无需进行复杂的文件传输操作。

### 基于VS Code的Docker开发环境
从原理上说，Docker Container和一个远程服务器类似，甚至可以通过ssh连接到Docker Container中进行开发。
VS Code提供了一个名为`Remote - Containers`的扩展，可以让用户直接在Docker Container中进行开发，而无需离开VS Code的界面。

首先，安装`Docker`和`Remote - Containers`扩展。
然后在源代码文件夹下创建一个隐藏文件夹`.devcontainer`，同时在其中创建一个名为`devcontainer.json`的文件，内容如下：
```json
{
    "name": "Test Container",
    "context": "..",
    "image": "nvidia/cuda:11.3.1-base-ubuntu20.04",
    "runArgs": [
        "--gpus", "all",
        "--user", "root",
        "-v", "/home/luan/Desktop:/workspace"
        "-e", "SHELL=/bin/bash"
    ]
}
```
其中，`name`字段指定了Docker Container的名称，`context`字段指定了Dockerfile所在的目录，`image`字段指定了要使用的Docker Image，`runArgs`字段指定了运行Docker Container时的参数。

参数中，
`--gpus all`表示允许Docker Container访问所有的GPU资源，
`--user root`表示以root用户身份运行Docker Container，
`-v /home/luan/Desktop:/workspace`表示将本地目录挂载到Docker Container中，
`-e SHELL=/bin/bash`表示设置环境变量SHELL为/bin/bash。

这样，一个最简单的配置就算完成了，单击高亮的`Reopen in Container`按钮，VS Code会自动构建Docker Image并启动Docker Container，同时将当前的源代码文件夹挂载到Docker Container中。
之后，用户就可以在Docker Container中进行开发了，所有的操作都会直接反映到本地文件系统中，非常方便。

---

## 2026_03_17

### 配置C++开发环境

#### 一个简单的CMake项目

假设待编译的C++源码是main.cpp，里面是一个C++的Hello World程序。
要是用CMake来构建这个项目，需要在同一目录下创建一个名为`CMakeLists.txt`的文件，
这是CMake的项目管理文件，CMake会以此文件为入口。

一个最简单的CMakeLists.txt文件内容如下：
```cmake
# 最低版本要求
cmake_minimum_required(VERSION 3.9)
# 使用C++17标准
set(CMAKE_CXX_STANDARD 17)
# 将项目命名为test
project(test)
# 将mian.cpp链接到demo程序
add_executable(demo main.cpp)
```
完成`CMakeLists.txt`文件的编写后，在终端中执行以下命令来构建项目：
```bash
mkdir build && cd build
cmake ..
make
```

我对C++不太熟悉，所以这里从我个人角度，详细地解释一下以上每一步给C++项目带来的改变。

`cmake_minimum_required()`和`set()`这两个比较好理解，没什么歧义。

而`project()`这个命令理论上是可以省略的，起到一个**标签**的作用，告诉CMake这个项目的名字是什么，后续如果有多个CMakeLists.txt文件，CMake就可以通过这个标签来区分不同的项目。可以用`${PROJECT_NAME}`来引用这个标签。

`add_executable(demo main.cpp)`这个命令是告诉CMake要构建一个可执行文件，名字叫demo，源文件是main.cpp。

编写完以上的CMakeLists.txt文件后，执行`mkdir build && cd build`。
我一开始好奇的点在于，这一步的意义在哪里，为什么要创建一个新的build目录来构建项目，而不是直接在当前目录下构建呢？
原来在后面的构造后，会产生一大堆构建文件和中间文件，如果直接在当前目录下构建，会把这些文件和源代码混在一起，显得很乱。
所以，创建一个新的build目录来构建项目，可以把构建文件和中间文件都放在这个目录下，这样就不会污染源代码目录了。

然后执行`cmake ..`，这个命令是告诉CMake要使用上一级目录中的CMakeLists.txt文件来构建项目。
执行完这个命令之后，`build/`目录下就会生成一些构建文件，例如`Makefile`，这些文件是CMake根据CMakeLists.txt文件生成的，用于指导后续的构建过程。

最后执行`make`命令，这个命令是告诉系统要根据生成的构建文件来构建项目。
执行完这个命令之后，`build/`目录下就会生成一个可执行文件，名字叫demo，这个文件就是我们最终的构建结果，可以直接运行这个文件来查看输出的结果.




