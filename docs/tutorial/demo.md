本demo将搭建一个WeCross跨连网络，连接FISCO BCOS和Fabric区块链。并通过WeCross控制台，对不同的链上资源进行操作。以理解WeCross的原理。

## 搭建Demo

``` bash
bash build.sh # 耗时10分钟左右
```

部署成功！能看到demo网络的架构，FISCO BCOS和Fabric通过各自的WeCross Router相连。（输入Y，回车，进入WeCross控制台）

``` 
[INFO] Success! WeCross demo network is running. Framework:

      FISCO BCOS                    Fabric
     (4node pbft)              (first-network)
   (HelloWorld.sol)               (abac.go)
          |                           |
          |                           |
    WeCross Router <----------> WeCross Router
(127.0.0.1-8250-25500)      (127.0.0.1-8251-25501)
           |
           |
    WeCross Console

Start console? [Y/n]
```

## 操作

**查看资源**

进入控制台，用`listResources`命令查看WeCross跨连网络中的所有资源。可看到有两个资源

* payment.bcos.HelloWeCross
  * 对应于**FISCO BCOS**链上的[HelloWorld.sol](https://github.com/FISCO-BCOS/console/blob/master/contracts/solidity/HelloWorld.sol)合约
* payment.fabric.abac
  * 对应于**Fabric**网络上的[abac.go](https://github.com/hyperledger/fabric-samples/blob/v1.4.4/chaincode/abac/go/abac.go)合约

``` groovy
[WeCross]> listResources
path: payment.bcos.HelloWeCross, type: BCOS2.0, distance: 0
path: payment.fabric.abac, type: Fabric1.4, distance: 1
```

**查看账户**

用`listAccounts`命令查看WeCross Router上已存在的账户，操作资源时用相应账户进行操作

``` groovy
[WeCross]> listAccounts
[
 {
  name=fabric_user1,
  type=Fabric1.4
 },
 {
  name=bcos_user1,
  type=BCOS2.0
 },
 {
  name=bcos_default_account,
  type=BCOS2.0
 },
 {
  name=fabric_default_account,
  type=Fabric1.4
 }
]
```



**操作资源：payment.bcos.HelloWeCross**

读资源：`call path 账户名 接口名 （参数列表）`

> 调用HelloWeCross合约中的get接口

``` groovy
[WeCross]> call payment.bcos.HelloWeCross bcos_user1 get
Result: [] // 查询为空，未set任何数据
```

写资源：`sendTransaction path 返回值类型列表 接口名 （参数列表）`

> 调用HelloWeCross合约中的set接口

``` groovy
[WeCross]> sendTransaction payment.bcos.HelloWeCross bcos_user1 set Tom
Txhash  : 0x21a412a1eb5239f2da9d40d09d11ce0107a5d82d113f1ecb315f2aa5bd3cc0cd
BlockNum: 2
Result  : [Tom]  // 将Tom给set进去

[WeCross]> call payment.bcos.HelloWeCross bcos_user1 get
Result: [Tom] // 再次get，Tom已set
```

**操作资源：payment.fabric.abac**

读资源：`call path 账户名 接口名 （参数列表）`

> 调用abac合约中的query接口

``` groovy
[WeCross]> call payment.fabric.abac fabric_user1 query a
Result: [90] // 初次query，a的值为90
```

写资源：`sendTransaction path 返回值类型列表 接口名 （参数列表）`

> 调用abac合约中的invoke接口

``` groovy
[WeCross]> sendTransaction payment.fabric.abac fabric_user1 invoke a b 10
Txhash  : db44b064c54d4dc97f01cdcd013cae219f7849c329f38ee102853344d8f0004d
BlockNum: 5
Result  : [] 

[WeCross]> call payment.fabric.abac fabric_user1 query a
Result: [80] // 再次query，a的值变成80
```

WeCross Console是基于WeCross Java SDK开发的跨连应用。在跨连网络搭建后，可基于WeCross Java SDK开发更多的跨连应用，通过统一的接口对各种链上的资源进行操作。

### 跨链转账

WeCross基于[哈希时间锁合约](../routine/htlc.html)实现了异构链之间资产的原子互换。可通过脚本`htlc_config.sh`完成相关配置，并体验跨链转账。

```shell
# 请确保demo已搭建完毕，并在demo根目录执行
bash htlc_config.sh
```

跨链转账涉及两条链、两个用户、四个账户，两条链上的资产转出者各自通过WeCross控制台创建一个[转账提案](../routine/htlc.html)，之后router会自动完成跨链转账。

* 创建转账提案
- BCOS链的资产转出者操作步骤

```shell
# 假设当前在demo根目录
cd WeCross-Console
bash start.sh
[WeCross]> newHTLCTransferProposal payment.bcos.htlc bcos_sender bea2dfec011d830a86d0fbeeb383e622b576bb2c15287b1a86aacdba0a387e11 9dda9a5e175a919ee98ff0198927b0a765ef96cf917144b589bb8e510e04843c true 0x55f934bcbe1e9aef8337f5551142a442fdde781c 0x2b5ad5c4795c026514f8317c7a215e218dccd6cf 700 2000010000 Admin@org1.example.com User1@org1.example.com 500 2000000000

Txhash: a0c48eb7d1ca3a01ddf3563aeb6a1829f23dd0d778e7de2ce22406d1e84ba00f
BlockNum: 56
Result: create a htlc transfer proposal successfully
```

- Hyperledger Fabric链的资产转出者操作步骤

```shell
# 在demo目录新打开一个终端
cd WeCross-Console-8251
bash start.sh
[WeCross]> newHTLCTransferProposal payment.fabric.htlc fabric_admin bea2dfec011d830a86d0fbeeb383e622b576bb2c15287b1a86aacdba0a387e11 null false 0x55f934bcbe1e9aef8337f5551142a442fdde781c 0x2b5ad5c4795c026514f8317c7a215e218dccd6cf 700 2000010000 Admin@org1.example.com User1@org1.example.com 500 2000000000

Txhash: 0x40ae8e2e284de813f8b071e0261e627ddc4d91e365e63f222638db9b1a70d05a
BlockNum: 123
Result: create a htlc transfer proposal successfully
```

* 查询转账结果

当两个资产转出者都创建完提案后，router开始执行调度，并完成跨链转账。一次跨链转账存在5-25s的交易时延，主要取决于两条链的TPS和机器的软硬件性能。可在各自的WeCross控制台查询资产是否到账。

- 查询BCOS链上资产接收者余额
```shell
[WeCross]> call payment.bcos.htlc bcos_sender balanceOf 0x2b5ad5c4795c026514f8317c7a215e218dccd6cf
Result: [700]
```

- 查询Hyperledger Fabric链上资产接收者余额
```shell
[WeCross]>  call payment.fabric.htlc fabric_admin balanceOf User1@org1.example.com
Result: [500]
```