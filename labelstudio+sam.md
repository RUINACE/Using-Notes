# 部署

## 1、conda环境创建与启动

首先创建一个python3.9的环境，修改版本需要调整其他配置

```
conda create -n skin python=3.9 -yConda create -n skin python=3.9 -y

```

conda最好配置下清华源然后再创建环境，可以在下载的时候配置临时清华源或者永久的清华源

```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/mainConda配置—添加通道https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/freeConda配置—添加通道https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/freeConda配置—添加通道https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/rConda配置—添加通道https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/proConda配置—添加通道https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/pro
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2Conda配置—添加通道https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2

```

pip临时使用清华源

```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-packagePIP install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package

```

pip永久配置清华源

```
pip install pip -U   pip安装pip -U
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simplePIP配置set global。index-url https://pypi.tuna.tsinghua.edu.cn/simple

```

## 2、playground下载
playground是运行的环境，需要在playground的label-anything目录下进行运行label-studio，使用git拉取

https://github.com/open-mmlab/playground

## 3、pytorch下载

https://download.pytorch.org/whl/torch_stable.html 在这里下载后安装，先安装torch的whl，然后再torchaudio和torchvision

上述配置需要在远程服务器以及本地PC上做一遍，如果是同一个设备就不需要。

## 4、SAM安装和预训练权重添加

下载源代码与权重，直接复制到浏览器

https://github.com/facebookresearch/segment-anything
https://dl.fbaipublicfiles.com/segment_anything/sam_vit_b_01ec64.pth
https://dl.fbaipublicfiles.com/segment_anything/sam_vit_l_0b3195.pth
https://dl.fbaipublicfiles.com/segment_anything/sam_vit_h_4b8939.pth

SAM相关库安装

```
pip install opencv-python pycocotools matplotlib onnxruntime onnx
pip install timm

```


## 5、安装 Label-Studio 和 label-studio-ml-backend

Label-Studio 就是客户端，用来打标的，label-studio-ml-backend就是服务端

大概流程就是服务器通过label-studio-ml-backend启动后端服务，后端服务就加载模型，当用户在客户端操作的时候，就会向服务端发一张你操作的图像，然后服务端就会通过SAM对你操作的图像进行分割，然后返回给你，你就可以看到标注好的图像了。

首先需要在服务端安装label-studio-ml-backend，然后需要在客户端安装Label-Studio ，我这里就是我自己的win10，安装命令如下

```
pip install label-studio==1.7.3
pip install label-studio-ml==1.0.9

```

## 参考

[从零实现label-studio和SAM进行半自动标注以及踩坑日志_sam标注-CSDN博客](https://blog.csdn.net/qq_43967413/article/details/134308713)
# 问题汇总

## 1、PC上cmd指令，把echo换成set（set是win上的）

## 2、 启动命令

服务端

```
set sam_config=vit_b
set sam_checkpoint=sam_vit_b_01ec64.pth

label-studio-ml start sam --port xxxx --with sam_config=vit_b sam_checkpoint_file=.\sam_vit_b_01ec64.pth out_bbox=True out_mask=False out_poly=False device=cuda:0

```

客户端

```
cd D:\xxxx\playground\label_anything

# 启动虚拟环境
conda activate xxxx

# 启动 
label-studio start

```

## 3、无法导入标签

最简单的方法就是生成json文件，直接传递图片和json标签，用的是convert_with_base64.py，直接upload json文件就行

使用 Base64 编码，图片数据直接嵌入 JSON，原始图片带标签数据位：E:\xxxx\label_studio_import.json

## 4、YOLO直接导出出错

直接导出json文件，然后提取转换

## 5、 导入数据集Json文件时报错找不到total

该错误与上传数据大小超过限制有关。在 Windows CMD 中设置环境变量并启动，可设置为 2147483648（2GB）或 5368709120（5GB）

```
set DATA_UPLOAD_MAX_MEMORY_SIZE=5368709120 # 1GB
label-studio start

```

持续报错

![[label-studio +sam.png]]

修改源码仍然报错，最简单直接的方式就是分割成小文件来上传，用的是虚拟机里面的 transfer_json2.py

## 6、多人共享

### 本机部署方式：必须得在一个网段或者内网穿透

#### 方法1：使用 Label Studio

```
1. 在本机安装并启动：
   
# 启动（默认端口8080） 
label-studio

2. 获取本机 IP 地址：
ipconfig


3. 启动时指定 host（允许外部访问）：
# 允许所有IP访问
label-studio --host 0.0.0.0
# 或指定端口
label-studio --host 0.0.0.0 --port 8080
   

4. 邀请他人：
在 Label Studio Web 界面创建账号
在项目设置中邀请成员（输入对方的邮箱）
对方通过 http://你的IP:8080 访问并登录

```

#### 方法2：使用 Docker

```
# 启动 Label Studio
docker run -it -p 8080:8080 -v $(pwd)/mydata:/label-studio/data heartexlabs/label-studio:latest label-studio

# 允许外部访问
docker run -it -p 8080:8080 --host 0.0.0.0 -v $(pwd)/mydata:/label-studio/data heartexlabs/label-studio:latest label-studio

# 如果防火墙挡住
# Windows: 在防火墙中允许8080端口
# Linux: 
sudo ufw allow 8080
# 或
sudo firewall-cmd --add-port=8080/tcp --permanent

```

### 内网穿透（跨网络访问）

如果需要跨网络访问，可以使用ngrok，但确实很慢很慢

#### 1.在本机安装 ngrok：

```
# Windows: 下载 https://ngrok.com/download
# Linux/Mac:
wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz
tar -xzf ngrok-v3-stable-linux-amd64.tgz
sudo mv ngrok /usr/local/bin/

```

#### 2.注册 ngrok 账号（免费）：

- 访问：https://dashboard.ngrok.com/signup
- 获取 authtoken

```

1. 配置 ngrok：
    ngrok config add-authtoken YOUR_AUTH_TOKEN

2. 启动 Label Studio（本机）：
    label-studio --host 0.0.0.0 --port 8080

3. 启动 ngrok（另一个终端）：
    cd D:\other\ngrok-v3-stable-windows-amd64
    ngrok http 8080

4. 获取公网 URL：
    Forwarding: https://abc123.ngrok-free.app -> http://localhost:8080

5. 分享 URL：
- 把 https://abc123.ngrok-free.app 发给对方
- 本机则是：http://localhost:8080
  
```

### 服务器部署

上面都没办法做到的话就只能服务器部署了
#### a.开启容器，并安装脚本：

```

#!/bin/bash
# Label Studio 直接安装脚本（容器内使用）
set -e
echo "=========================================="
echo "Label Studio 直接安装脚本"
echo "=========================================="

# 检查 Python 是否安装
if ! command -v python3 &> /dev/null; then
    echo "错误: Python3 未安装"
    echo "请先安装 Python3"
    exit 1
fi

echo ""
echo "1. 更新 pip..."
python3 -m pip install --upgrade pip
echo ""
echo "2. 安装 Label Studio..."
pip3 install label-studio
echo ""
echo "3. 创建数据目录..."
mkdir -p ~/label-studio-dataMkdir -p ~/label-studio-data
mkdir -p ~/label-studio-mediaMkdir -p ~/label-studio-media

echo ""   echo ""
echo "=========================================="echo "=========================================="
echo "安装完成！"
echo "=========================================="echo "=========================================="
echo ""   echo ""
echo "启动命令:"
echo "  label-studio start --host 0.0.0.0 --port 8080"Echo "  label-studio start—host 0.0.0.0—port 8080"；
echo ""   echo ""
echo "或者后台运行:"
echo "  nohup label-studio start --host 0.0.0.0 --port 8080 > label-studio.log 2>&1 &"Echo "  nohup label-studio start—host 0.0.0.0—port 8080 > label-studio.log 2>&1 &"；
echo ""   echo ""
echo "查看日志:"
echo "  tail -f label-studio.log"Echo "  tail -f label-studio.log"；
echo ""   echo ""

```

#### b.启动

```
# 方式1：直接启动（前台运行，测试用）
label-studio start --host 0.0.0.0 --port 9600标签工作室启动——主机0.0.0.0——端口9600

# 方式2：后台运行（推荐）
nohup label-studio start --host 0.0.0.0 --port 9600 > label-studio.log 2>&1 &Nohup label-studio start—host 0.0.0.0—port 9600 > label-studio.log 2>&1 &；

```

#### c. 验证端口监听

```
# 检查 9600 端口是否监听
netstat -tlnp | grep 9600netstat  -tlnp  |   grep  96

# 或
ss -tlnp | grep 9600   ss  -tlnp  |   grep  96

```

## 7、本机访问

在浏览器打开：
http://192.168.x.xxx:9600

注意：使用 9600 端口，不是 8080。

## 8、数据集导出

导出数据的时候注意，一般导出yolo模式是没招的，直接导出json然后用 json_to_yolo.py代码，直接会自动解析出图片和代码
