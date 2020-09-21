# 实战--fabric---商业信息征信系统

# 1、生成证书文件

- 新建工程目录文件夹education

  ```shell
  mkdir business/
  cd business
  ```

- step1 生成配置文件

```shell
#将模版文件重定向生成配置文件
cryptogen showtemplate >  crypto-config.yaml
```

- step2  修改配置文件 crypto-config.yaml

  ```yaml
  # ---------------------------------------------------------------------------
  # "OrdererOrgs" - Definition of organizations managing orderer nodes
  # ---------------------------------------------------------------------------
  OrdererOrgs:
    # ---------------------------------------------------------------------------
    # Orderer
    # ---------------------------------------------------------------------------
    - Name: Orderer
      Domain: innovation.com
      EnableNodeOUs: true
      Specs:
        - Hostname: orderer
  
  # ---------------------------------------------------------------------------
  # "PeerOrgs" - Definition of organizations managing peer nodes
  # ---------------------------------------------------------------------------
  PeerOrgs:
    # ---------------------------------------------------------------------------
    # Org1
    # ---------------------------------------------------------------------------
    - Name: Org1
      Domain: org1.innovation.com
      EnableNodeOUs: true
      Template:
        Count: 2
      Users:
        Count: 2
  
    # ---------------------------------------------------------------------------
    # Org2: See "Org1" for full specification
    # ---------------------------------------------------------------------------
    - Name: Org2
      Domain: org2.innovation.com
      EnableNodeOUs: true
      Template:
        Count: 2
      Users:
        Count: 2
  ```

- step3  生成证书文件

  ```shell
  cryptogen generate --config=crypto-config.yaml
  #进入文件夹，可查看目录文件
  tree ordererOrganizations/
  #查看peer文件
  tree peerOrganizations/peerOrganizations/
  ```

  Sepcs和Template指定用户的区别（可分开使用，也可以联合使用）

  - Sepcs是可以指出确定的域名

  - Template是按照0开始排列 peer0,peer1

  ```shell
   Specs:
        - Hostname: orderer           #子域名，可访问 orderer.example.com,对应一个节点
    Template:
    		Count: 1
  ```

# 2、生成创世块文件与通道文件,锚节点更新文件

### 2.1、编写配置文件configtx.yaml

文件名固定为configtx.yaml

>  将之前下载的示例下的`configtx.yaml`拷贝到当前目录下：

```shell
cp ~/go/src/github.com/hyperledger/fabric/scripts/fabric-samples/first-network/configtx.yaml ~/go/src/github.com/business/
```

### 2.2、生成排序服务的创始块文件

- 首先在主文件夹business下创建channel-artifacts文件夹（为后面docker-compose作准备）

  ```shell
  mkdir channel-artifacts
  ```

- 在主文件下之执行生成创始块命令（一定要在configtx.yaml文件的同级目录下），生成genesis.block文件

- 根据文件最后配置的不同，选择不同的profile，比如官方命令为：

- **此处的通道ID和之后的通道ID名不能一样，此处也可以不设置，默认为`testchainid`**

  ```shell
  FABRIC_CFG_PATH=$PWD configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block -channelID businessfirstchannel
  ```

### 2.3、生成通道文件

- 主文件夹下执行生成通道文件的命令,生成channel.tx文件，可指定channelD，通道名字,如果不指定默认为`mychannel`

  ```shell
  #设置当前通道ID
  export CHANNEL_NAME=businesschannel
  #查询是否设置成功
  echo $CHANNEL_NAME
  #生成通道文件
  FABRIC_CFG_PATH=$PWD configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID $CHANNEL_NAME
  ```

### 2.4、生成锚节点更新文件

- 需要为**每个组织**各生成一份锚节点更新文件

  ```shell
  FABRIC_CFG_PATH=$PWD configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org1MSP
  ```

  ```shell
  FABRIC_CFG_PATH=$PWD configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID $CHANNEL_NAME -asOrg Org2MSP
  ```

命名解读：

  - -profile：指定configtx.yaml文件中profiles中的组织名
  - -outputAnchorPeersUpdate：指定生成锚节点更新文件的文件名
  - -channelID: 指定锚节点所属通道，通道名为之前生成通道文件时命名的
  - -asOrg: 指定锚节点所属的组织名

# 3、修改docker-compose.yml文件

## 3.1、修改yml文件

从下载的fabric-samples中复制docker-compose-cli.yaml文件进行修改

```
cp ~/go/src/github.com/hyperledger/fabric/scripts/fabric-samples/basic-network/docker-compose.yml ~/go/src/github.com/business/docker-compose.yml
```

## 3.2、测试网络环境

启动docker-compose，查看所有容器运行情况，查看最终结果是否创建4个peer节点，1个orderer节点，1个CA节点容器。

```shell
#在docker-compose.yml文件同级目录下运行以下命令
docker-compose up -d
```

运行结果显示：

```
         Name                              Command                                 State            Ports         
--------------------------------------------------------------------------------
ca.innovation.com        sh -c fabric-ca-server                   Up        0.0.0.0:7054->7054/tcp
                                               sta ...                                                
couchdb                             tini -- /docker-                                 Up       4369/tcp, 0.0.0.0:5984
                                               entrypoint ...                                                       ->5984/tcp, 9100/tcp  
orderer.innovation.com   orderer                                          Up       0.0.0.0:7050->7050/tcp
peer0.org1.innovation.   peer node start                            Up       0.0.0.0:7051->7051/tcp
com                                                       ,                     
                                                                                                                            0.0.0.0:7053->7053/tcp
peer0.org2.innovation.   peer node start                              Up      0.0.0.0:7251->7251/tcp
com                                                       ,                     
                                                                                                                             0.0.0.0:7253->7253/tcp
peer1.org1.innovation.   peer node start                               Up      0.0.0.0:7151->7051/tcp
com                                                       ,                     
                                                                                                                             0.0.0.0:7153->7053/tcp
peer1.org2.innovation.   peer node start                                Up      0.0.0.0:7351->7351/tcp
com                                                       ,                     
                                                                                                                                0.0.0.0:7353->7353/tcp
```

运行完毕后如果不需要一定要将网络环境关闭

```shell
#关闭并删除所有容器
docker-compose down
```

# 4、Fabric-SDK-Go

## 4.1、配置config.yaml

创建一个新的 config.yaml 配置文件给应用程序所使用的 Fabric-SDK-Go 配置相关参数及 Fabric 组件的通信地址

首先，进入项目的根目录中创建一个 `config.yaml` 文件并编辑

创建配置文件的时候,有两个文件可以进行参考...

```
$GOPATH/src/github.com/hyperledger/fabric-sdk-go/test/fixtures/config/config_test.yaml
```

```
$GOPATH/src/github.com/hyperledger/fabric-sdk-go/pkg/core/config/testdata/template/config.yaml
```

#######仔细修改config.yaml文件，对比docker-compose.yml文件#################

## 4.2、定义Fabric-SDK结构体

在项目的根目录下添加一个名为 `sdkInit` 的新目录，我们将在这个文件夹中创建 SDK，并根据配置信息创建应用通道

```shell
$ mkdir sdkInit
```

为了方便管理 Hyperledger Fabric 网络环境，我们将在 `sdkInit` 目录中创建一个 `fabricInitInfo.go`  的源代码文件，用于定义一个结构体，包括 Fabric SDK 所需的各项相关信息

```shell
$ vim sdkInit/fabricInitInfo.go
```

fabricInitInfo.go文件内容如下：

```go
package sdkInit

import (
	"github.com/hyperledger/fabric-sdk-go/pkg/client/resmgmt"
)

type InitInfo struct {
	ChannelID     string
	ChannelConfig string
	OrgAdmin      string
	OrgName       string
	OrdererOrgName	string
	OrgResMgmt *resmgmt.Client

	ChaincodeID	string
	ChaincodeGoPath	string
	ChaincodePath	string
	UserName	string
}
```

结构体中包含成员解释如下：

- **ChannelID：**通道名称
- **ChannelConfig：**通道交易配置文件所在路径
- **OrgName：**组织名称
- **OrgAdmin：**组织管理员名称
- **OrdererOrgName：**Orderer名称
- **OrgResMgmt：**资源管理端实例

声明好结构体中的相应成员之后，就可以使用 `fabric-sdk-go` 相关的API来创建SDK实例，并使用该SDK实例进行一系列的操作，如：

- 创建通道
- 将组织中的peers加入已创建的通道中
- 安装链码
- 实例化链码
- 创建客户端实例

## 4.3、创建SDK

首先，我们先来完成SDK的创建及通道的创建，通道创建完成之后，就可以将peers加入到该通道中。

在 `sdkInit` 目录下新创建一个名为 `start.go` 的源代码文件利用 vim 编辑器进行编辑：

```shell
$ vim sdkInit/start.go
```



# 5、满足依赖

在运行应用程序之前，需要对Golang源代码进行编译，需要下载相应的第三方包。Golang官方最初只提供了包管理的go get工具，它将下载的第三方包保存到GOPATH的src目录下，项目一般由很多来源不同的第三方包构成，所以Golang1.5版本之后增加了一个新的发现包的方法，使用dep工具，在vendor目录中处理这些依赖关系，但经过测试golang.org中包被墙了，下载时会报错

%%%%%%%%%%%%%%%%%%%%%%%%代补充%%%%%%%%

因此此处采用go module模块

## 1.1、go module

go module 是go官方自带的go依赖管理库，可以将某个项目下的所有依赖整理成一个go.mod文件，里面写入依赖版本等，

- 初始化

```shell
#使用modules功能，对应off,auto
export GO111MODULE=on
#设置代理
export GOPROXY=https://goproxy.io
cd 目录名
#项目第一次使用GO MODULE 创建一个go.mod文件，标识了项目名和go的版本
go mod init 项目名
```

- 检测依赖

  ```shell
  go mod tidy
  ```

  检测该目录下所有引入的依赖，写入go.mod文件，写入后go.mod文件会有变动：

  ```mod
  module kong
  
  go 1.14
  
  require (
  	github.com/Shopify/sarama v1.27.0 // indirect
  	github.com/hashicorp/go-version v1.2.1 // indirect
  	github.com/hyperledger/fabric v2.1.1+incompatible
  	github.com/hyperledger/fabric-amcl v0.0.0-20200424173818-327c9e2cf77a // indirect
  	github.com/hyperledger/fabric-sdk-go v1.0.0-beta3
  	github.com/sykesm/zap-logfmt v0.0.3 // indirect
  	go.uber.org/zap v1.15.0 // indirect
  )
  ```

  此时依赖还没有下载

- 下载依赖

将依赖下载至本地

```shell
go mod download
```

依赖包会全部下载至GOPATH下的pkg/mod文件夹中，同时会在根目录下生成go.sum文件，该文件是依赖的详细依赖

GO MODULE 常用命令

```shell
go mod init                            #初始化go.mod
go mod tidy                           #更新依赖文件
go mod download              #下载依赖文件
go mod vendor                    #将依赖转移至本地vendor文件
go mod edit                          #手动修改依赖文件
go mod verify                       #校验依赖
```



## 1.2、注意事项

- 设置GOLAND开启GO MODULE

- ```
  GOPROXY=http://goproxy.cn,direct
  ```

  

![](/home/liuhui/文档/hyperledge  Fabric/galand-module.png)

- 导入本地依赖包

  使用go module 之后，不再从GOPATH下开始读取路径，而是从项目名下开始读取路径，不然会报错，此处项目名为kong

  ```
  import (
  	"fmt"
  	"kong/web"
  	"kong/sdkInit"
  	"kong/service"
  	"kong/web/controller"
  	"os"
  )
  ```

# 2、Makefile整合shell命令

- 检测是否安装make工具

  ```shell
  make --version
  ```

- 安装make工具

  ```shell
  sudo apt install make
  ```

- 进入项目根目录并创建Makefile的文件并编辑

```shell
cd 项目根目录
vim Makefile
```

编写内容

```makefile
##########GO MODULE
module：
	@echo "download module"
	@export GO111MODULE=on
	@export GOPROXY=https://goproxy.io
	@cd ~/home/liuhui/go/src/github.com/kongyixueyuan.com/kongyixueyuan
	@go mod init kong
	@go mod tidy
	@go mod download
	@echo "download module done"

##########Start Hyperledger fabric network
env-up:
	@echo "Start environment....."
	@cd ~/home/liuhui/go/src/github.com/kongyixueyuan.com/kongyixueyuan/fixtures && docker-compose up -d
	@echo "Environment up "
	
##########Down Hyperledger fabric network
env-down:
	@echo "Stop environment....."
	@cd ~/home/liuhui/go/src/github.com/kongyixueyuan.com/kongyixueyuan/fixtures && docker-compose down
	@echo "Environment down"

#########RUN app
run:
	@echo "Start app......."
	@./kong
	
#########CLEAN ENV
clean:env-down
	@echo "Clean up......"
	@rm -rf /tmp/kongyixueyuan-* kongyixueyuan
    @docker rm -f -v `docker ps -a --no-trunc | grep "kongyixueyuan" | cut -d ' ' -f 1` 2>/dev/null || true
    @docker rmi `docker images --no-trunc | grep "kongyixueyuan" | cut -d ' ' -f 1` 2>/dev/null || true
    @echo "Clean up done"
```

