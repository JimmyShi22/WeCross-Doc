## 配置文件
WeCross通过配置文件管理Stub以及每个Stub中的跨链资源，启动时首先加载配置文件，根据配置去初始化各个Stub以及相应的资源，如果配置出错，则启动失败。

```eval_rst
.. note::
    - WeCross在开发阶段尝试了XML、YAML等不同的配置了文件格式，在综合易用性、灵活性、功能性以及需求契合性等多方面因素的考量后，最终采用了Toml的配置格式。
    - Toml是一种语义化配置文件格式，可以无二义性地转换为一个哈希表，支持多层级配置，无缩进和空格要求，配置容错率高。
```

### 配置结构

WeCross的配置分为根配置和子配置两级，根配置负责P2P、Tomcat Server等和WeCross服务相关的必要信息；子配置主要有两个重要的配置项，分别是Stub对应区块链客户端的建连配置以及区块链上的资源列表信息。如果子配置缺省，WeCross仍能启动成功，只是不能提供任何跨链服务。

WeCross根配置文件名为`wecross.toml`，子配置文件名为`stub.toml`，配置的目录结构如下：

```
.
├── log4j2.xml   // 日志配置文件，无需更改
├── stubs         
│   ├── bcos
│   │   └── stub.toml
│   └── fabric
│       └── stub.toml
└── wecross.toml
```

### 根配置

WeCross的根配置示例文件`wecross-sample.toml`编译后位于`WeCross/dist/conf`目录，使用前需拷贝成指定文件名`wecross.toml`。

```bash
cd dist
cp conf/wecross-sample.toml conf/wecross.toml
```

配置示例如下：

```toml
[common]
    network = 'payment'
    visible = true

[stubs]
    path = 'classpath:stubs'

[server]
    address = '127.0.0.1'
    port = 8250

[p2p]
    listenIP = '0.0.0.0'
    listenPort = 25500
    caCert = 'classpath:p2p/ca.crt'
    sslCert = 'classpath:p2p/node.crt'
    sslKey = 'classpath:p2p/node.key'
    peers = ['127.0.0.1:25501','127.0.0.1:25502']

[test]
    enableTestResource = false
```

根配置有五个配置项，分别是`[common]`、`[stubs]`、`[server]`、`[p2p]`以及`[test]`，各个配置项含义如下：

- [common] 通用配置
  - network：字符串；跨链网络标识符；通常一种跨链业务/应用为一个跨链网络
  - visible：布尔；可见性；标明当前跨链网络下的资源是否对其他跨链网络可见
- [stubs] Stub配置
  - path：字符串；Stub配置的根目录；WeCross从该目录下去加载各个Stub的配置
- [server] Tomcat Server配置
  - address：字符串；本机IP地址；WeCross通过Spring Boot内置的Tomcat启动Web服务
  - port：整型；WeCross服务端口；需要未被占用
- [p2p] 组网配置
  - listenIP：字符串；监听地址；一般为'0.0.0.0'
  - listenPort ：整型；监听端口；WeCross节点之间的消息端口
  - caCert ：字符串；根证书路径；拥有相同根证书的WeCross节点才能互相通讯
  - sslCert ：字符串；节点证书路径；WeCross节点的证书
  - slKey ：字符串；节点私钥路径；WeCross节点的私钥
  - peers：字符串数组；peer列表；需要互相连接的WeCross节点列表
- [test] 测试配置
  - enableTestResource：布尔；测试资源开关；如果开启，那么即使没有配置Stub的资源信息，也可以根据测试资源体验WeCross的部分功能。

**注：**  


1. WeCross启动时会把`conf`目录指定为classpath，若配置项的路径中开头为`classpath:`，则以`conf`为相对目录。
2.  `[p2p]`配置项中的证书和私钥可以通过build_cert.sh脚本生成，-h可查看帮助信息。使用示例如下：

   ```bash
   # 生成根证书ca.crt
   sh build_cert.sh -c
   
   # 生成节点证书和私钥node.crt和node.key
   # 必须先生成根证书ca.crt，才能生成节点证书和私钥
   # 该命令还会生成node.nodeid，主要用于P2P出错时的程序调试，可以忽略
   sh build_cert.sh -n
   
   # 批量生成节点证书和私钥
   # -C后面为数量
   sh build_cert.sh -n -C 10
   ```
3. 若通过build_wecross.sh脚本生成的项目，那么已自动帮忙配置好了`wecross.toml`，包括P2P的配置，其中Stub的根目录默认为`stubs`。

### 子配置

子配置即每个Stub的配置，是WeCross跨链业务的核心，配置了Stub和区块链交互所需的信息，以及注册了各个链需要参与跨链的资源。WeCross启动后会在`wecross.toml`中所指定的Stub的根目录下去遍历所有的一级目录，目录名即为Stub的名字，不同的目录代表不同的链，然后尝试读取每个目录下的stub.toml`文件。

目前WeCross支持的Stub类型包括：[FISCO BCOS](https://github.com/FISCO-BCOS/FISCO-BCOS)、[Fabric](https://github.com/hyperledger/fabric)和[JDChain]()

#### FISCO BCOS

配置示例如下：

```toml
[common]
    stub = 'bcos' # stub must be same with directory name
    type = 'BCOS'

[smCrypto]
    # boolean
    enable = false

[account]
    accountFile = 'classpath:/stubs/bcos/0xa1ca07c7ff567183c889e1ad5f4dcd37716831ca.pem'
    password = ''  # if you choose .p12, then password is required


[channelService]
    timeout = 60000  # millisecond
    caCert = 'classpath:/stubs/bcos/ca.crt'
    sslCert = 'classpath:/stubs/bcos/sdk.crt'
    sslKey = 'classpath:/stubs/bcos/sdk.key'
    groupId = 1
    connectionsStr = ['127.0.0.1:20200']

# resources is a list
[[resources]]
    # name cannot be repeated
    name = 'HelloWorldContract'
    type = 'BCOS_CONTRACT'
    contractAddress = '0x8827cca7f0f38b861b62dae6d711efe92a1e3602'
[[resources]]
    name = 'FirstTomlContract'
    type = 'BCOS_CONTRACT'
    contractAddress = '0x584ecb848dd84499639fbe2581bfb8a8774b485c'
```

配置方法详见[FISCO BCOS Stub配置]()

#### Fabric

配置示例如下：

```toml
[common]
    stub = 'fabric' # stub must be same with directory name
    type = 'FABRIC'

# fabricServices is a list
[fabricServices]
     channelName = 'mychannel'
     orgName = 'Org1'
     mspId = 'Org1MSP'
     orgUserName = 'Admin'
     orgUserKeyFile = 'classpath:/stubs/fabric/5895923570c12e5a0ba4ff9a908ed10574b475797b1fa838a4a465d6121b8ddf_sk'
     orgUserCertFile = 'classpath:/stubs/fabric/Admin@org1.example.com-cert.pem'
     ordererTlsCaFile = 'classpath:/stubs/fabric/tlsca.example.com-cert.pem'
     ordererAddress = 'grpcs://127.0.0.1:7050'
     
[peers]
    [peers.a]
        peerTlsCaFile = 'classpath:/stubs/fabric/tlsca.org1.example.com-cert.pem'
        peerAddress = 'grpcs://127.0.0.1:7051'
    [peers.b]
         peerTlsCaFile = 'classpath:/stubs/fabric/tlsca.org2.example.com-cert.pem'
         peerAddress = 'grpcs://127.0.0.1:9051'   
           
# resources is a list
[[resources]]
    # name cannot be repeated
    name = 'HelloWorldContract'
    type = 'FABRIC_CONTRACT'
    chainCodeName = 'mycc'
    chainLanguage = "go"
    peers=['a','b']
[[resources]]
    name = 'FirstTomlContract'
    type = 'FABRIC_CONTRACT'
    chainLanguage = "go"
    chainCodeName = 'cscc'
    peers=['a','b']
```

配置方法详见[Fabric Stub配置]()

#### JDChain

配置示例如下：

```toml
[common]
    stub = 'jd' # stub must be same with directory name
    type = 'JDCHAIN'

# jdServices is a list
[[jdServices]]
     privateKey = '0000000000000000'
     publicKey = '111111111111111'
     password = '222222222222222'
     connectionsStr = '127.0.0.1:18081'
[[jdServices]]
     privateKey = '0000000000000000'
     publicKey = '111111111111111'
     password = '222222222222222'
     connectionsStr = '127.0.0.1:18082'

# resources is a list
[[resources]]
    # name cannot be repeated
    name = 'HelloWorldContract'
    type = 'JDCHAIN_CONTRACT'
    contractAddress = '0x38735ad749aebd9d6e9c7350ae00c28c8903dc7a'
[[resources]]
    name = 'FirstTomlContract'
    type = 'JDCHAIN_CONTRACT'
    contractAddress = '0x38735ad749aebd9d6e9c7350ae00c28c8903dc7a'
```

配置方法详见[JDChain Stub配置]()