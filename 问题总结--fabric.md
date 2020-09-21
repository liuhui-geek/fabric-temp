# Fabric 问题总结

1、更新锚节点时

```shell
2020-08-03 12:31:50.857 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
Error: got unexpected status: BAD_REQUEST -- error applying config update to existing channel 'mychannel': error authorizing update: error validating DeltaSet: policy for [Group]  /Channel/Application/Org1MSP not satisfied: signature set did not satisfy policy
```

问题：需要在更新组织一的锚节点时配置一下环境变量，有可能现在处于其他组织的环境变量中

```
export CORE_PEER_ADDRESS=peer0.org1.example.com:7051 
export CORE_PEER_LOCALMSPID=Org1MSP
export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
export CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt 
```



2、使用Fabric CA客户端fabric-ca-client注册用户时

执行注册命令：

```shell
fabric-ca-client enroll -u http://admin:pass@localhost:7054
```

出现以下错误：

```shell
Error：Response from server: Error Code: 20 - Authorization failure
```

用户名称与密码不匹配，需要和fabric-ca-server中设置的名称和密码对应



3、使用go module模块导入包后开始编译

执行命令

```shell
go mod tidy
```

出现以下错误

```
github.com/kongyixueyuan.com/kongyixueyuan/sdkInit: cannot find module providing package github.com/kongyixueyuan.com/kongyixueyuan/sdkInit: module github.com/kongyixueyuan.com/kongyixueyuan/sdkInit: reading https://goproxy.io/github.com/kongyixueyuan.com/kongyixueyuan/sdk%21init/@v/list: 404 Not Found
	server response:
	not found: go list -m -json -versions github.com/kongyixueyuan.com/kongyixueyuan/sdkInit@latest
	: go list -m github.com/kongyixueyuan.com/kongyixueyuan/sdkInit: git ls-remote -q https://github.com/kongyixueyuan.com/kongyixueyuan in /data1/golang/pkg/mod/cache/vcs/e351eab08f462adae74c49f34a995aa3165ed8df1b094047715e4e24b48fe06e: exit status 128:
		fatal: could not read Username for 'https://github.com': terminal prompts disabled
	If this is a private repository, see https://golang.org/doc/faq#git_https for additional information.
kong imports
	github.com/kongyixueyuan.com/kongyixueyuan/service: cannot find module providing package github.com/kongyixueyuan.com/kongyixueyuan/service: module github.com/kongyixueyuan.com/kongyixueyuan/service: reading https://goproxy.io/github.com/kongyixueyuan.com/kongyixueyuan/service/@v/list: 404 Not Found
	server response:
	not found: go list -m -json -versions github.com/kongyixueyuan.com/kongyixueyuan/service@latest
	: go list -m github.com/kongyixueyuan.com/kongyixueyuan/service: git ls-remote -q https://github.com/kongyixueyuan.com/kongyixueyuan in /data1/golang/pkg/mod/cache/vcs/e351eab08f462adae74c49f34a995aa3165ed8df1b094047715e4e24b48fe06e: exit status 128:
		fatal: could not read Username for 'https://github.com': terminal prompts disabled
	If this is a private repository, see https://golang.org/doc/faq#git_https for additional information.
```

是由于本地依赖包的路径因为使用了MODULE而发生了改变，

在go文件中将import的本地包路径进行修改

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

##如果在goland中修改需要设置GO MODULE才能检测错误！

![](/home/liuhui/文档/hyperledge  Fabric/galand-module.png)