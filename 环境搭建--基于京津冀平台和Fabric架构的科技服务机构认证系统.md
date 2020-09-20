# 环境搭建--基于京津冀平台和Fabric架构的科技服务机构认证系统

### 1、更换下载源

更换apt的下载源，因为官方下载源很慢，需要更换到国内的镜像站

#### 1.1、进入/etc/apt/目录

```shell
cd etc/apt
```

#### 1.2、备份sources.list文件

```shell
sudo cp /etc/apt/sources.list /etc/apt/sources.list_backup
```

#### 1.3、打开sources.list文件并添加阿里云镜像

```shell
sudo vi /etc/apt/sources.list

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

1.4、更新apt-get

```shell
sudo apt-get update
sudo apt-get upgrade
```

### 2、环境安装

#### 2.1、必备软件安装

```shell
sudo apt-get install vim
sudo apt-get install git
sudo apt-get install curl
sudo apt-get install wget
sudo apt-get install python-pip
```

#### 2.2、Go安装

```shell
#进入根目录
cd ~

#下载压缩包
wget https://studygolang.com/dl/golang/go1.14.9.linux-amd64.tar.gz

#解压压缩包
tar -xzf go1.14.9.linux-amd64.tar.gz

#删除压缩包
rm -rf go1.14.9.liunx-amd64.tar.gz

#设置权限，并把根目录下的go文件夹移动到/usr/local/目录下
sudo chmod 777 /usr/local/
sudo mv go /usr/local

#修改环境变量
vi ~/.bashrc

#设置环境变量
export  PATH=$PATH:/usr/local/go/bin
export  GOROOT=/usr/local/go
export  GOPATH=$HOME/go
export  PATH=$PATH:$HOME/go/bin

#更新环境变量
source ~/.bashrc
#查看go是否安装成功
go version

#查看go环境变量是否设置成功
go env
```

#### 2.3、docker安装

```shell
#查询docker是否安装
docker version

#更新apt包索引
sudo apt-get update

#下载安装工具
sudo apt-get install apt-transport-https ca-certificates software-properties-common

#添加官方密钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

#加入apt仓库
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

#下载docker-ce
sudo apt-get update
sudo apt-get install docker-ce

#验证版本
docker version

#将非root组加入docker组
sudo groupadd docker
sudo gpasswd -a $USER docker    #将当前用户添加到用户组
#sudo usermod -aG docker XXX(XXX是当前用户名) 

#重启docker服务
sudo service docker restart
#切换当前会话到新group或重启会话
newgrp - docker


#添加阿里云docker镜像
sudo mkdir -p /etc/docker     
sudo vim /etc/docker/daemon.json

#将以下内容写入文件中
{
 "registry-mirrors": ["https://obou6wyb.mirror.aliyuncs.com"]
}

#重启docker
sudo systemctl daemon-reload 
sudo systemctl restart docker 
docker version
```

#### 2.4、docker-compose安装

```yaml
#下载docker-compose
sudo curl -L https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose

#允许其他用户执行compose相关命令
sudo chmod +x /usr/local/bin/docker-compose

#验证版本
docker-compose -version
```

#### 2.5、下载Fabric源码，镜像，示例等

```shell
#创建并进入hyperledger目录
mkdir -p $GOPATH/src/github.com/hyperledger
cd $GOPATH/src/github.com/hyperledger

#下载fabric源码
git clone https://github.com/hyperledger/fabric.git

#将fabric切换至1.4版本
cd fabric
git branch -a
git checkout release-1.4

#下载fabric镜像以及示例
#cd $GOPATH/src/github.com/hyperleger/fabric/scripts
#中间会stop几次，重新运行即可
#./bootstrap.sh
#因为运行bootstrap.sh文件需要下载一些二进制文件，但是这些文件又存放在国外的网站上，所以下载十分缓慢，不能通过直接运行bootstrap.sh来安装fabric所需的工具和镜像。运行该文件主要进行了3个步骤，可以分开操作
#步骤一
cd $GOPATH/src/github.com/hyperledger/fabric/scripts
git clone https://github.com.cnpmjs.org/hyperledger/fabric-samples.git
git branch -a
git checkout release-1.4
#步骤二
#查看version和ca-version,然后手动下载对应版本
vim bootstrap.sh
cd $GOPATH
wget https://github.com/hyperledger/fabric/releases/download/v1.4.8/hyperledger-fabric-linux-amd64-1.4.8.tar.gz
wget https://github.com/hyperledger/fabric-ca/releases/download/v1.4.8/hyperledger-fabric-ca-linux-amd64-1.4.8.tar.gz
#解压压缩包后会生成/bin文件夹，存放工具
tar -xzf hyperledger-fabric-linux-amd64-1.4.8.tar.gz
tar -xzf hyperledger-fabric-ca-linux-amd64-1.4.8.tar.gz
#步骤三
#下载docker镜像
cd $GOPATH/src/github.com/hyperledger/fabric/scripts
sudo chmod 777 bootstrap.sh
sudo ./bootstrap.sh -s -b

#查看安装的镜像
docker images
```

### 3、测试fabric环境

```
#进入示例目录
cd $GOPATH/src/github.com/hyperledger/fabric/scripts/fabric-samples/first-network/

#生成创世区块，通道，证书等
./byfn.sh generate

#启动网络
./byfn.sh up

#停止网络
./byfn.sh down
```

