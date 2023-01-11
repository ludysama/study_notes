# 直接doker pull拉取我制作好的镜像
```
docker pull ludysama369/blumnet:origin
```
以下记录的是复现的过程，鉴于镜像已经制作完成，可以直接略过下面的步骤，拉取我制作好的镜像直接开始实验
镜像中包含有需要用到的主干网络预训练权重、sk1491数据集（目前看代码是没有使用额外的旋转翻折等数据增广，数据量很小）


## 1、拉取基础的docker镜像
按照blumnet官方的说法 （https://github.com/cong-yang/BlumNet）使用的环境在torch1.9 cuda10.2以上，因此直接拉取一个相近的docker torch镜像
拉取镜像的网站是： https://hub.docker.com/r/pytorch/pytorch/tags
我选择的版本是 pytorch/pytorch:1.9.1-cuda11.1-cudnn8-devel，因为之前做类似的复现工作中使用过这个版本，能正常把相关但不相同的代码跑起来，所以本着能用就不换的原则，还是继续使用这个版本。

拉取镜像，等了二十多分钟搞定了，嫌慢的可以试试换源
```
(base) ludy_ubuntu@ludy-ubuntu-desktop:~$ docker pull pytorch/pytorch:1.9.1-cuda11.1-cudnn8-devel
1.9.1-cuda11.1-cudnn8-devel: Pulling from pytorch/pytorch
f22ccc0b8772: Pull complete 
3cf8fb62ba5f: Pull complete 
e80c964ece6a: Pull complete 
8a451ac89a87: Pull complete 
c563160b1f64: Pull complete 
596a46902202: Pull complete 
aa0805983180: Pull complete 
5718c3da35a0: Pull complete 
003637b0851a: Pull complete 
18a42526ee57: Pull complete 
32c20b0bdcb3: Pull complete 
7a199c750eab: Pull complete 
Digest: sha256:fd8fcd6e1196d8965657b04e7dfb666046063904b767c1fd75df8039fe0ada17
Status: Downloaded newer image for pytorch/pytorch:1.9.1-cuda11.1-cudnn8-devel
docker.io/pytorch/pytorch:1.9.1-cuda11.1-cudnn8-devel
```

## 2、创建容器
然后创建容器，基于前车之鉴，创建的时候把数据盘也加进去，这里我加了两块数据盘，格式是 -v 宿主路径:容器内路径，起个名字blumnet --gpus all是必须的，不然无法在容器中使用gpu资源
```
docker run -it --gpus all -v /ssd/ubuntu_docker:/ssd -v /chia/f0:/hdd --name blumnet pytorch/pytorch:1.9.1-cuda11.1-cudnn8-devel
```

### 2.x 后续提交镜像时需要注意的事项
接下来可以进入容器内进行开发了，这点对于vscode开发环境还是很轻松的
![image](https://user-images.githubusercontent.com/62829345/210067627-00f3ccfd-691d-4df3-942a-ea4f1de7014c.png)

首先远程连接宿主机（或者直接在宿主机安装vscode，本地操作），接下来在远程开发组件的箭头位置，选择containers，找到刚刚启动的容器，进入即可。
不过使用vscode登录容器后，制作镜像时会把vscode server的一些文件也保留在容器中，制作好的镜像再新建容器会有vscode server认证失败的问题，注意提交镜像前将 /root/.vscode-server 文件夹删掉

## 3、容器内安装git
执行
```
apt-get update
```
更新软件安装列表，然后执行
```
apt-get install git
```
安装git到容器
## 4、clone blumnet项目
在/root文件夹（默认用户是root，所以相当于/home）执行
```
git clone https://github.com/cong-yang/BlumNet.git
```
## 5、创建环境
容器内装好pylance插件，可以高亮看包装没装，另外对照requirements.txt文件查漏补缺，把环境补全

```
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```


## 6、编译算子
```
cd models/ops
sh make.sh
python test.py
```
很神奇的是这次居然没有爆算子无法编译的错，离谱，之前在这里卡了很久

## 7、下载模型权重
项目的readme中有写一个sh文件执行就可以下载权重
不过试了试，网络问题很难下下来，最后在一台科学上网的电脑上访问sh文件中的地址把权重下载到本地，然后传给docker容器。
曲线救国完成此步骤

## 8、开始训练
按照项目readme进行接下来的步骤吧

## 后记
在这个docker下，如果需要使用diffvg做渲染器，安装的时候会报一个missing PYTHON_INCLUDE_DIRS的错误，尝试了几把，解决的办法是安装一个python-dev
```
apt-get install python-dev
```
