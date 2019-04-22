# 在虚拟机中的Ubuntu（v18.04）上配置 hyperledger fabric 开发环境

## 准备工作：

### 更改源：

```
sudo gedit /etc/apt/sources.list
```

上面的命令会打开一个文本文件，删除所有内容，把下面的粘贴进去（不用之前搜到的源了，那个源安装curl时有问题）

```
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```

保存之后，运行下面的命令：
```sh
sudo apt-get -y update
```

### 调整时区：
```sh
sudo apt install ntpdate
sudo ntpdate cn.pool.ntp.org 
sudo cp /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
sudo hwclock --systohc
```
**重启**

### 安装必要工具：
```sh
sudo apt install git
sudo apt install make
sudo apt install gcc
sudo apt install curl
sudo apt-get install tree
sudo apt-get install jq
```

## 安装 docker：
```sh
sudo apt install docker.io
sudo apt install docker-compose
```

设置 docker 用户组：
```sh
sudo gpasswd -a ${USER} docker
sudo service docker restart
newgrp - docker
```

设置docker镜像源：
```sh
sudo gedit /etc/docker/daemon.json
```

上面的命令会打开一个文本文件，把下面的粘贴进去
```json
{
  "registry-mirrors" : ["https://docker.mirrors.ustc.edu.cn"]
}
```

## 安装 go1.12.4：

把 go1.12.4.linux-amd64.tar.gz 复制到 Home 文件夹

运行命令：
```sh
sudo tar -C /usr/local -xzf go1.12.4.linux-amd64.tar.gz go
sudo gedit /etc/profile
```

在打开的文件末尾加：

```sh
##golang
export GOPATH=$HOME/go
export GOROOT=/usr/local/go
export GOARCH=amd64
export GOOS=linux
export GOTOOLS=$GOROOT/pkg/tool
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
##golang end
```

## node.js v8.9.4 安装：

运行命令：
```sh
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
source ~/.bashrc
nvm install 8.9.4
sudo gedit /etc/profile
```

在打开的文件末尾加：
```sh
##nvm
export NVM_DIR="$HOME/.nvm"
##nvm end
```

## Fablic 安装：

**重启**

把 fabric.zip 复制到 Home 文件夹

依次运行命令：

```sh
sudo mkdir -p $GOPATH/src/github.com/hyperledger
cd $GOPATH/src/github.com/hyperledger
sudo unzip ~/fabric.zip > /dev/null
go get github.com/golang/protobuf/protoc-gen-go

mkdir -p $GOPATH/src/github.com/hyperledger/fabric/build/docker/gotools/bin
cp $GOPATH/bin/protoc-gen-go $GOPATH/src/github.com/hyperledger/fabric/build/docker/gotools/bin


cd $GOPATH/src/github.com/hyperledger/fabric
make release
make docker

sudo cp $GOPATH/src/github.com/hyperledger/fabric/release/linux-amd64/bin/* /usr/local/bin
sudo chmod -R 775 /usr/local/bin/configtxgen
sudo chmod -R 775 /usr/local/bin/configtxlator
sudo chmod -R 775 /usr/local/bin/cryptogen
sudo chmod -R 775 /usr/local/bin/peer
sudo chmod -R 775 /usr/local/bin/orderer
```
