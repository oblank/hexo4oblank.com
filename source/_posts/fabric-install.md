---
title: Hypleder Fabric 体验
date: 2018-06-14 17:20:53
tags:
  - fabric
  - blockchain
categories:
  - 开发
---

推荐选用云服务器，按流量付费，使用后及时销毁

注意选择香港区域的服务器，保证网络的联通，避免额外工作，这算是小小的经验吧，我这里选择的是腾讯云香港服务器（4核8G）。

> 国内服务器可以使用ProxyChain4配合代理来无障碍安装软件

注意服务器版本选择，因为要用到Docker，而根据经验CentOS最好使用7+的版本，我们这里选择的是7.4

*根据文档来看，在Windows、Mac环境下都会多做一些额外工作，建议使用CentOS或Ubuntu这样的系统。*


我所用系统内核：
```bash
Linux VM_***_centos 3.10.0-693.el7.x86_64
```

> 按官方文档安装系统环境

http://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html

#### 具体要求如下：
- Docker version 17.06.2-ce or greater.
- Docker Compose version 1.14.0 or greater.
- Go version 1.10.x is required.
- Node.js - version 8.9.x or greater，9.x is not supported at this time.

#### 安装DockerCE社区版

参考官方文档 https://docs.docker.com/install/linux/docker-ce/centos/

确认Docker正常：
```bash
docker --version
Docker version 18.05.0-ce, build f150324
```

#### 安装Docker-compose

参考官方文档：https://docs.docker.com/compose/install/#install-compose

确认Docker-compose正常：
```bash
docker-compose --version
docker-compose version 1.21.2, build a133471
```

#### 安装 Go
Fabric是由Go开发的，智能合约Chaincode也是Go实现的

参考文档：https://golang.org/doc/install?download=go1.10.2.linux-amd64.tar.gz

确定Go正常：
```go
go version
go version go1.10.2 linux/amd64

# test code
go run hello.go 
hello, world
```

#### Node.js 安装
这里使用NVM进行多版本管理

参考：https://github.com/creationix/nvm/blob/master/README.md

确认安装正常：
```bash
nvm install 8.9.4
nvm use v8.9.4

npm -v
5.6.0
```


### 按文档安装Fabric

http://hyperledger-fabric.readthedocs.io/en/latest/install.html

执行：


```
# 如果系统没有GIT则先安装GIT
sudo yum install git -y

# 安装 Fabric
curl -sSL https://goo.gl/6wtTN5 | bash -s 1.1.0
```

没问题的话用`docker images`命令会看下列结果：

```bash
# docker images

REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
hello-world                    latest              e38bc07ac18e        6 weeks ago         1.85kB
hyperledger/fabric-ca          latest              72617b4fa9b4        2 months ago        299MB
hyperledger/fabric-ca          x86_64-1.1.0        72617b4fa9b4        2 months ago        299MB
hyperledger/fabric-tools       latest              b7bfddf508bc        2 months ago        1.46GB
hyperledger/fabric-tools       x86_64-1.1.0        b7bfddf508bc        2 months ago        1.46GB
hyperledger/fabric-orderer     latest              ce0c810df36a        2 months ago        180MB
hyperledger/fabric-orderer     x86_64-1.1.0        ce0c810df36a        2 months ago        180MB
hyperledger/fabric-peer        latest              b023f9be0771        2 months ago        187MB
hyperledger/fabric-peer        x86_64-1.1.0        b023f9be0771        2 months ago        187MB
hyperledger/fabric-javaenv     latest              82098abb1a17        2 months ago        1.52GB
hyperledger/fabric-javaenv     x86_64-1.1.0        82098abb1a17        2 months ago        1.52GB
hyperledger/fabric-ccenv       latest              c8b4909d8d46        2 months ago        1.39GB
hyperledger/fabric-ccenv       x86_64-1.1.0        c8b4909d8d46        2 months ago        1.39GB
hyperledger/fabric-zookeeper   latest              92cbb952b6f8        3 months ago        1.39GB
hyperledger/fabric-zookeeper   x86_64-0.4.6        92cbb952b6f8        3 months ago        1.39GB
hyperledger/fabric-kafka       latest              554c591b86a8        3 months ago        1.4GB
hyperledger/fabric-kafka       x86_64-0.4.6        554c591b86a8        3 months ago        1.4GB
hyperledger/fabric-couchdb     latest              7e73c828fc5b        3 months ago        1.56GB
hyperledger/fabric-couchdb     x86_64-0.4.6        7e73c828fc5b        3 months ago        1.56GB
```

添加PATH环境变理：
```bash
# 具体路径以你实际情况为准
export PATH=$PATH:/root/go/src/fabric-samples/bin
```

#### 运行第一个测试网络

参考：http://hyperledger-fabric.readthedocs.io/en/latest/build_network.html

在`fabric-samples/first-network`目录下执行：

```bash
./byfn.sh generate
./byfn.sh up

# 直到出现下面的提示
========= All GOOD, BYFN execution completed =========== 


 _____   _   _   ____   
| ____| | \ | | |  _ \  
|  _|   |  \| | | | | | 
| |___  | |\  | | |_| | 
|_____| |_| \_| |____/  
```

这个时候再使用使用`docker ps`，会出现如下结果：

```bash
# docker ps
CONTAINER ID        IMAGE                                                                                                  COMMAND                  CREATED             STATUS              PORTS                                              NAMES
61be9101c099        dev-peer1.org2.example.com-mycc-1.0-26c2ef32838554aac4f7ad6f100aca865e87959c9a126e86d764c8d01f8346ab   "chaincode -peer.add…"   3 minutes ago       Up 3 minutes                                                           dev-peer1.org2.example.com-mycc-1.0
eda8b0fd85ff        dev-peer0.org1.example.com-mycc-1.0-384f11f484b9302df90b453200cfb25174305fce8f53f4e94d45ee3b6cab0ce9   "chaincode -peer.add…"   3 minutes ago       Up 3 minutes                                                           dev-peer0.org1.example.com-mycc-1.0
6107fb4406fc        dev-peer0.org2.example.com-mycc-1.0-15b571b3ce849066b7ec74497da3b27e54e0df1345daff3951b94245ce09c42b   "chaincode -peer.add…"   4 minutes ago       Up 4 minutes                                                           dev-peer0.org2.example.com-mycc-1.0
b8204bfc0734        hyperledger/fabric-tools:latest                                                                        "/bin/bash"              4 minutes ago       Up 4 minutes                                                           cli
83ff712ca3bd        hyperledger/fabric-peer:latest                                                                         "peer node start"        4 minutes ago       Up 4 minutes        0.0.0.0:8051->7051/tcp, 0.0.0.0:8053->7053/tcp     peer1.org1.example.com
677a3fc1c6c7        hyperledger/fabric-peer:latest                                                                         "peer node start"        4 minutes ago       Up 4 minutes        0.0.0.0:7051->7051/tcp, 0.0.0.0:7053->7053/tcp     peer0.org1.example.com
193de4b51d5a        hyperledger/fabric-peer:latest                                                                         "peer node start"        4 minutes ago       Up 4 minutes        0.0.0.0:9051->7051/tcp, 0.0.0.0:9053->7053/tcp     peer0.org2.example.com
ebafff254a5b        hyperledger/fabric-orderer:latest                                                                      "orderer"                4 minutes ago       Up 4 minutes        0.0.0.0:7050->7050/tcp                             orderer.example.com
ebe3b1c8f9e8        hyperledger/fabric-peer:latest                                                                         "peer node start"        4 minutes ago       Up 4 minutes        0.0.0.0:10051->7051/tcp, 0.0.0.0:10053->7053/tcp   peer1.org2.example.com

```

简洁模式

```bash
# docker ps --format "{{.ID}}\t{{.Names}}"

61be9101c099    dev-peer1.org2.example.com-mycc-1.0
eda8b0fd85ff    dev-peer0.org1.example.com-mycc-1.0
6107fb4406fc    dev-peer0.org2.example.com-mycc-1.0
b8204bfc0734    cli
83ff712ca3bd    peer1.org1.example.com
677a3fc1c6c7    peer0.org1.example.com
193de4b51d5a    peer0.org2.example.com
ebafff254a5b    orderer.example.com
ebe3b1c8f9e8    peer1.org2.example.com
```

这样就模拟了一个简单的区块链网络，它有1个排序机构（order org），2个节点组织（peer org）每个组织有2个节点（peer）。

上面的组织机构定义在`configtx.yaml`文件中，而各个容器的拓扑关系定义在`crypto-config.yaml`文件中，使用 `./byfn.sh down` 可以关闭这个网络。
