# 在虚拟机中的Ubuntu（v18.04）上配置 hyperledger fabric 开发环境

## 准备工作：

### 更改源：

```sh
sudo gedit /etc/apt/sources.list
```

上面的命令会打开一个文本文件，删除所有内容，把下面的粘贴进去

```sh
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

如果时区是正确的，不需要这一步：

```sh
sudo apt install ntpdate
sudo ntpdate cn.pool.ntp.org
sudo cp /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
sudo hwclock --systohc
```

**重启**

### 安装基础工具：

```sh
sudo apt -y install git
sudo apt -y install make
sudo apt -y install gcc
sudo apt -y install curl
sudo apt -y install tree
sudo apt -y install jq
sudo apt -y install net-tools
sudo apt install axel # 多线程下载工具
```

## 安装 docker：

```sh
sudo apt install -y docker.io
sudo apt install -y docker-compose
```

### 设置 docker 用户组：

```sh
sudo gpasswd -a ${USER} docker
sudo service docker restart
newgrp - docker
```

### 设置docker镜像源：

```sh
sudo gedit /etc/docker/daemon.json
```

上面的命令会打开一个文本文件，把下面的粘贴进去

```json
{
  "registry-mirrors" : ["http://f1361db2.m.daocloud.io"]
}
```

重启

## 安装 go1.9.7 开发环境：

删除已安装版本：

```sh
sudo rm -rf /usr/local/go
```

下载：

```sh
wget -c https://dl.google.com/go/go1.9.7.linux-amd64.tar.gz
```

[所有版本下载地址链接](https://golang.google.cn/dl/)

解压：

```sh
sudo tar -C /usr/local -xzf go1.9.7.linux-amd64.tar.gz go
```

添加环境变量：

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

安装 go 插件和包：

```sh
###### clone
git clone https://github.com/golang/protobuf    $GOPATH/src/github.com/golang/protobuf

git clone https://github.com/uudashr/gopkgs     $GOPATH/src/github.com/uudashr/gopkgs
git clone https://github.com/karrick/godirwalk  $GOPATH/src/github.com/karrick/godirwalk
git clone https://github.com/pkg/errors         $GOPATH/src/github.com/pkg/errors

git clone https://github.com/go-delve/delve     $GOPATH/src/github.com/go-delve/delve
git clone https://github.com/rogpeppe/godef     $GOPATH/src/github.com/rogpeppe/godef

git clone https://github.com/golang/tools           $GOPATH/src/golang.org/x/tools
git clone https://github.com/golang/lint            $GOPATH/src/golang.org/x/lint

git clone https://github.com/mdempsky/gocode        $GOPATH/src/github.com/mdempsky/gocode
git clone https://github.com/ramya-rao-a/go-outline $GOPATH/src/github.com/ramya-rao-a/go-outline
git clone https://github.com/acroca/go-symbols      $GOPATH/src/github.com/acroca/go-symbols
git clone https://github.com/stamblerre/gocode      $GOPATH/src/github.com/stamblerre/gocode
git clone https://github.com/sqs/goreturns          $GOPATH/src/github.com/sqs/goreturns

###### install
go install github.com/golang/protobuf/protoc-gen-go

go install github.com/uudashr/gopkgs/cmd/gopkgs
go install github.com/go-delve/delve/cmd/dlv
go install github.com/rogpeppe/godef

go install golang.org/x/tools/cmd/guru
go install golang.org/x/tools/cmd/gorename
go install golang.org/x/lint/golint

go install github.com/mdempsky/gocode
go install github.com/ramya-rao-a/go-outline
go install github.com/acroca/go-symbols
go install github.com/stamblerre/gocode
go install github.com/sqs/goreturns
```

## node.js v8.9.4 安装：

运行命令：

```sh
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
source ~/.bashrc
nvm install 8.9.4
```

## Fablic 安装：

### 删除已安装版本：

```sh
rm -rf $GOPATH/src/github.com/hyperledger/*
```

### 安装 release-1.1版本：

```sh
mkdir -p $GOPATH/src/github.com/hyperledger
git clone -b release-1.1 https://github.com/hyperledger/fabric.git $GOPATH/src/github.com/hyperledger/fabric
git clone -b release-1.1 https://github.com/hyperledger/fabric-samples.git $GOPATH/src/github.com/hyperledger/fabric-samples

curl https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric/hyperledger-fabric/linux-amd64-1.1.0/hyperledger-fabric-linux-amd64-1.1.0.tar.gz
mkdir -p $GOPATH/src/github.com/hyperledger/fabric/release/linux-amd64
tar -C $GOPATH/src/github.com/hyperledger/fabric/release/linux-amd64/ -xzf hyperledger-fabric-linux-amd64-1.1.0.tar.gz
tar -C $GOPATH/src/github.com/hyperledger/fabric-samples -xzf hyperledger-fabric-linux-amd64-1.1.0.tar.gz

mkdir -p $GOPATH/src/github.com/hyperledger/fabric/build/docker/gotools/bin
cp $GOPATH/bin/protoc-gen-go $GOPATH/src/github.com/hyperledger/fabric/build/docker/gotools/bin
```

### 添加环境变量：

```sh
export $PATH=$PATH:$GOPATH/src/github.com/hyperledger/fabric/release/linux-amd64/bin
```

下载 docker 镜像：

```sh
docker pull hyperledger/fabric-peer:$x86_64-1.1.0
docker pull hyperledger/fabric-orderer:$x86_64-1.1.0
docker pull hyperledger/fabric-ca:$x86_64-1.1.0
docker pull hyperledger/fabric-tools:$x86_64-1.1.0
docker pull hyperledger/fabric-ccenv:$x86_64-1.1.0
docker pull hyperledger/fabric-baseimage:$x86_64-0.4.6
docker pull hyperledger/fabric-baseos:$x86_64-0.4.6
docker pull hyperledger/fabric-couchdb:$x86_64-0.4.6
docker pull hyperledger/fabric-kafka:$x86_64-0.4.6
docker pull hyperledger/fabric-zookeeper:$x86_64-0.4.6
```

## 验证配置成功

运行以下命令：

```sh
peer version
orderer version
cryptogen version
configtxgen -version
configtxlator version
```
