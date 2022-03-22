# 连入服务器

> **写在开头**：

> 本文内容将长久在[cuberJ/ServerandDocker: 配置服务器连接和docker (github.com)](https://github.com/cuberJ/ServerandDocker)更新，若你此刻阅读的文档为PDF或者Markdown文件，可以在上述网址继续获取后续的更新内容！
>
> 对于Linux操作不熟练的话，可以参考蒋砚军老师的本科Linux课程，我自己这门课的笔记放在了[cuberJ/LinuxNotes: Linux环境基础指令使用 (github.com)](https://github.com/cuberJ/LinuxNotes)
>
> Git的操作笔记：[cuberJ/HowToUseGit: Github使用操作](https://github.com/cuberJ/HowToUseGit)

首先需要先从师兄师姐那里申请账号，密码，端口号和服务器所在的IP地址，再进行以下操作

## Mac用户

1. 打开系统自带的terminal，输入指令：

   ```shell
   格式为：ssh -p 端口号 用户名@IP地址
   例如：端口号为1000，用户名zhangsan，IP地址:192.168.1.1
   ssh -p 1000 zhangsan@192.168.1.1
   ```

2. 回车后，terminal会跳转至服务器登录页面，要求输入密码，输入登记的密码即可登录



## Windows用户

1. ~~ssh连接器可以考虑gitbash？~~~
2. 我自己用的<a href='https://termius.com/'>Termius</a>，感觉还可以（至少写这个推荐的时候处于高级版试用期中，等过了试用期再看情况来改评价吧🤡  更新：现在已经是文档写完后的第二年3月15日了，距离第一次写这段话都过去小半年了，依然还没用Windows跑过服务器，我是🕊石锤了）



## 服务器连接的一些情况

1. 如果terminal长时间没有输入信息，可能会断联。这时候不用慌，因为服务器大概率在一直运行，只是终端掉线了，只需要重新连接就行，基本不用担心没保存的东西丢失
   1. 当然，没事就:w总是一个好习惯
2. 实验室内部的不成文规定：文件尽量不要放在home文件夹下：
   1. 常见的错误：执行cd指令之后，bash会停留在自己的用户目录下，但是用户目录也是在home下的子目录，所以建议不要在自己的目录下面存啥大文件
   2. 尽量将大文件堆到store文件夹下



# Docker配置

由于一群人在一个系统里跑代码，环境随便配会混乱，所以每个人采用docker的方式运行自己的程序



## docker环境配置

该条目由管理员阅读并配置服务器Docker环境

version：2021年11月6日版

采用服务器T630，该服务器上Docker已完成配置



## docker使用

参考链接：[SSH连接docker容器配置pycharm远程调试_﹎厡.唻-CSDN博客](https://blog.csdn.net/u012620515/article/details/89856072)

注意：docker是容器，是一个动态的环境；image是镜像，是一个静态的环境。简单来说，镜像是类，容器是实例。

镜像是只读文件，容器是在镜像的基础上，创建出的一个基于该镜像的可运行环境。我们大部分的时间都是基于docker在调试代码的



### 查看服务器上的现有镜像及容器

```shell
docker images # 查看镜像
docker ps -a # 查看容器
```



### 拉取Python镜像

```shell
docker pull python:3.7.4 #拉取Python3.7.4版本的镜像
docker images  # 可以查看自己刚刚拉取的镜像。如果之前已经拉取过，则该镜像不会更新为现在的时间，而是保留老版本的时间
```



### 创建运行容器

为了能够运行程序，需要在这个docker中使用pip安装Python运行所需的依赖环境和依赖包，并放入运行的代码。最终配置完成后，将该docker打包为image即可迁移

```shell
# -it：-i和-t的结合，感觉就是如下图，直接进入容器的命令行模式。

# –name：自定义容器名称，不用的话会自动分配一个名称。

# -v： 将本地文件夹~/Pycharm/PythonDockerTest与容器文件夹/home/cairenjie共享。

# python:3.7.4：要运行的镜像名+TAG

# -p :将服务器的8034端口与docker的22号端口映射上，这样就可以直接通过8034号端口将Python文件传入docker里

# bash：进入容器命令行。

# --gpus all :调用服务器上的显卡用于训练
docker run --name py2test -p 8034:22 --gpus all -v ~/Pycharm/PythonDockerTest:/home/cairenjie -it python:3.7.4 bash
```

进入后是下图的样子：（由于实际创建的文件夹名称不同，故与展示的指令效果不完全一致）

![image-20211109155843384](image-20211109155843384.png)

<font color='red'>注意：由于容器内的文件夹是映射的，所以与系统中实际绑定的文件夹text_similar在docker中无法删除，需要在实际的系统中删除</font>

在本地文件夹中vim一个hello.py文件，编写一个最简单的Python程序，由于共享文件夹，则docker中的对应目录下也能直接看到该py文件

直接执行`python hello.py`

效果如下：

![image-20211109160127370](image-20211109160127370.png)



>#### **<u>一些容易踩坑的地方：</u>**
>
>1. docker中的环境很简陋，如果只拉取了Python镜像，那么**<u>甚至无法用vim或者vi</u>**去编辑文件。需要使用Vim的话还需要自己安装（见容器SSH配置）
>
>2. 配置端口映射：建议服务器的端口选用8000以上，可以直接尝试上述的`docker run xxxx`指令，如果端口已经被占用，则系统会直接报错无法创建docker，就可以选用另一个端口去尝试;
>
>  1. 或者可以采用下面的指令：
>
>     ```shell
>     netstat -tunple | grep 端口号
>     ```
>
>  2. <font color = red>不要用自己从师兄师姐那里申请账号时获得的端口号作为映射端口。</font>不过就算用了，你也会发现报错，因为这个端口已经被你自己的ssh连接占了
>
>5. **<u>端口映射信息以及GPU配置信息等均建议创建的时候就配置完成，否则后期修改很繁琐</u>**
>
>6. 镜像的选择很重要，一定要三思，不然搭了一天之后发现换个镜像1小时就完事也不是不可能



### 容器SSH配置

到了这里, pycharm依然无法连接docker,因为docker里尚不支持ssh连接。故需要在docker里配置ssh。进入容器，在容器中执行下面两个指令

```ssh
apt-get update
apt-get install openssh-server openssh-client
```

两个指令都会噼里啪啦蹦出来一堆东西，遇到选择就输入Y完事

由于后续需要使用Vim配置信息，所以我们要先安装Vim

```shell
apt install vim
```

安装完成后执行指令编辑配置文件

```shell
vim /etc/ssh/sshd_config
```

将文件中的这三行信息做修改：（文件内容很多，建议esc在命令行模式下用/方式寻找关键字）

![image-20211109170640812](image-20211109170640812.png)

```
#PermitRootLogin without-password 改为 PermitRootLogin yes
#PasswordAuthentication yes 改为 PasswordAuthentication yes
UsePAM yes改为UsePAM no
```

> 备注：
>
> 1. 第一个without-password可能不一定是这个选项，我遇到的是叫prohibit-password还是类似的啥，删的太快忘了，但是无所谓，不管是啥，都换成yes就可以了
> 2. 第一个和第二个配置信息的#都要删除

配置完成后执行

```shell
service ssh restart
# 记得重新配置root用户的密码
passwd root
```

理论上这时候就完成配置了。新建一个端口，去测试连接一下

```shell
ssh -p [docker在服务器上选用的端口] root@[服务器的IP地址]
另一种方式是通过Pycharm的Tools -> Development -> congiuration里加一个SFTP连接测试，Username填root，password填自己刚刚设置的密码，端口是容器映射的那个端口（8034）
```



### 退出容器

在容器命令行内输入exit即可

```shell
exit

# 如果在本地环境，则输入以下指令
docker stop pytest_crj
```

或者Ctrl + D

如果只是想临时返回本地界面而不结束容器，则采用快捷键`Ctrl + P + Q`



### 重启容器

```shell
docker restart f7c249c7529a #这里的编号是Docker的ID
# 或者可以输入名称
docker restart pytest_crj
```



### 删除容器

```shell
docker rm docker的ID
```

删除后，有的时候会出bug（目前只出现过一次，尚未复现成功），服务器上与docker共享的目录可能会失去写权限，变成只读

猜测是因为docker创建的时候失败（比如端口映射的时候映射到了一个使用中的端口），锁定了文件夹

这时候找管理员申请赋权就可以解决了



### 重新进入容器

由于容器短时间内无法配置完环境，所以经常需要重新进入docker容器。重启后的容器也不会自动进入，需要手动进入

查看现在正在运行的容器：

```shell
docker ps -a
```

找到自己的容器，确认第一列的CONTAINER_ID号。这里我的ID号为`f7c249c7529a`

执行指令

```shell
docker exec -it f7c2 bash # ID号可以不用输入全部，只需要输入前缀，并且该前缀独一无二即可
# 或者采用attach
docker attach pytest_crj # 注意，attach方式下不用输入bash
```



### 查看容器的挂载目录信息

如果时间久了记不得自己的容器与服务器上哪一个目录映射，通过下面的两个指令均可以查询

```shell
docker inspect docker的名字 | grep Mounts -A 20
docker inspect docker的编号 | grep Mounts -A 20
```

效果如下：

![image-20211109155327192](image-20211109155327192.png)





## Pycharm连接服务器

神经网络训练中，如何使用pycharm专业版连接服务器



### Pycharm专业版获取途径

1. ~~盗版是绝对要谴责的，但是如果能偷偷用那就没法谴责了对吧？~~
2. 在校生身份，从jetbrains官网申请在校大学生免费使用Pycharm专业版。在此之前需要获取一个edu.cn结尾的邮箱，用该邮箱申请即可。整个过程无需科学上网



### 连接至服务器

1. ![image-20211106110350028](image-20211106110350028.png)

   选择`tools - deployment - configuration `

2. 左上角+号新建SFTP连接，连接名称自行命名，不重要

3. ![image-20211106110613746](image-20211106110613746.png)

   p选择`SSH - configuration`后面的省略号，配置IP和端口

   ![image-20211106110742957](image-20211106110742957.png)

   配置信息包括IP地址，端口号，用户名以及密码。配置后可以测试连通性。如图表示测试成功

4. 其他的信息（默认根目录是服务器的根目录，建议选择自己的账号所在的目录作为根目录）录入完成后，



### 配置解释器

进入解释器配置界面

![image-20211109172446602](image-20211109172446602.png)

点击圈出的设置符号，选择add

![image-20211109172514040](image-20211109172514040.png)

选择SSH Interpreter，Host设置为服务器的IP地址，Port选择当时与Docker映射的服务器端口号。Username为root，点击右下方next，进入下一栏输入密码

配置映射的目录之后，等待Pycharm将文件同步到服务器即可

最后在Pycharm运行代码，解释器为服务器中的Docker

![image-20211109172757661](image-20211109172757661.png)



### 自动同步

![image-20220315105548187](image-20220315105548187.png)



## 常见问题(持续更新中)

#### 我的pycharm同步服务器上的文件夹和项目Python解释器之后，没法通过setting里的interpreter安装第三方库怎么办？

> 1. 如果用的是conda，直接在服务器里运行`conda install numpy（假如要装numpy）`即可
>
>    > 修改conda源：
>    >
>    > 1. `vim ~/.condarc`
>    >
>    > 2. 在里面输入如下内容：
>    >
>    >    ```
>    >    channels:
>    >      - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
>    >      - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
>    >      - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
>    >      - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
>    >    ssl_verify: true
>    >    ```
>    >
>    > 3. 保存退出，重新运行conda install即可
>
> 2. 如果用的是pip……暂时没遇到这个情况，希望其他人能补充一下

#### 我看到镜像里标注了cuda，Python，怎么查版本号和安装路径？

1. Python：
   1. 安装路径：`which python3.7`
   2. 版本：`python3`直接就展示版本了
2. anaconda：`conda --version`



#### 大文件和训练数据怎么下载到服务器里？

1. 一般是下载到本地，然后通过pycharm或者scp指令上传到服务器



#### 镜像怎么选择啊？

1. 一般如果用pytorch作为训练模型的话，建议优先选择有pytorch的镜像，不然安装cuda，anaconda之类的玩意会无比糟心

2. TensorFlow，还没用过，等一个用过的人来帮我更新这一段……

3. python这种一般优先度最低，因为很好装……而且有pytorch了，谁家镜像还不搭配一个Python对吧？

4. 如果服务器上没有合适的镜像，就去到dockerhub找一个。实验室里不愿透露姓名的男子刘某建议选用`cuda+版本号`作为搜索关键字

   

#### 为啥我pycharm在连接docker的时候显示connect refuse？

1. 情况一：你自己密码记错了，建议上服务器执行命令`passwd 用户名(一般docker里是root)`重置密码
2. 情况二：你没设置docker里的ssh或者没有把ssh设置为开机自启动，[参考这里重新配置ssh或者重启ssh](###容器SSH配置)
3. 情况三（很少见）：服务器上的与docker相连的映射端口被关闭了，向管理员申请打开端口



#### 如果我跑训练模型时间很久，导致ssh断开连接，再次重进服务器的时候，输出的训练信息在界面上都丢失了怎么办？

采用shell重定向输出的方式输出到log.txt文件中

```shell
python3 train.py > log.txt 2>&1
```

如果想同时在命令行界面和文件中都能看到输出的结果：

```
python3 train.py | tee log.txt
```

如果想跑模型的过程中干点别的，就用&的方式将运行代码的进程放在后台

```shell
python3 train.py > log.txt 2>&1 &
```







# 常用指令

## 查看服务器当前状态的指令

```shell
# 查看显卡占用情况
nvtop
nvidia-smi
# 查看磁盘占用情况
df -h
```



## 服务器维修

### Network is unreachable

PC3是115服务器（门口两台一起放着的服务器里靠外面的那一台），204服务器是PC4（门口两台一起的服务器里贴着墙的那一台）

多半是无线模块掉了，步骤如下：

1. 执行ifconfig，如果最后一个段落（`w1x646e啥啥啥`）里inet addr没有显示ip地址，说明是无线模块掉了

2. 走到实验室找到服务器，把服务器屁股上的无线模块拔下来重新插一下（就是带一根家用WiFi一样天线的）

3. 然后执行如下命令：

   ```shell
   先进入/home/xinngximing/rt18821CU文件夹
   
   make
   # 执行之后界面里最后一行出现ERROR 2 是正常情况
   sudo make install
   sudo modprobe 8821cu
   sudo usb_modeswitch -KW -v 0bda -p 1a2b
   # 这个时候执行nmcli c s(是connect show的缩写）AP_Wired2WireLess_2E15那个网绿色代表已经连上了，不用再管，白色需要激活
   sudo nmcli connection up AP_Wired2WireLess_2E15
   
   ```

4. 如果需要切换显示器界面，按一下连着的转换器切换到PC4即可。如果坏掉的是204服务器，有可能需要用网线连接到交换机上，通过有线地址外部访问维修

   转换器如图所示

    ![image-20220322161004156](image-20220322161004156.png)

# NLP学习路径

记录一下自己入门的流程，以防以后基本功丢了没法拾回来

## 入门

**<u>完全零基础的入门学习路线</u>**

### 书目

1. 《深度学习的数学》：适合完全不懂什么是神经网络的人阅读



### 教程

1. transformer：[Transformer论文逐段精读【论文精读】_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1pu411o7BE/)
   1. [Transformer论文逐段精读【论文精读】 - 哔哩哔哩 (bilibili.com)](https://www.bilibili.com/read/cv13759416?spm_id_from=333.788.b_636f6d6d656e74.70)
2. Bert：
3. 图神经网络：
