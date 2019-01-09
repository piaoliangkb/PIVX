> 官方链接地址 
> 
> [https://pivx.org/knowledge-base/masternode-setup-guide/](https://pivx.org/knowledge-base/masternode-setup-guide/)

1. 首先，我们要生成masternode的privatekey，将生成的key当作第一步的结果。

**pivx-cli masternode genkey**

```bash
zl@zl-VirtualBox:~$ sudo pivx-cli masternode genkey
92MsoEkmbjSyRmXoNap9nn6LPDKD9SPqNP2ExSsEqhwAQ4Twe7n
```

2. 新建一个账户，并获得他的地址，masternode需要10000PIV的抵押，我们之后需要把PIV转入到这个账户中。我们这里的account名字就叫mn1.

**pivx-cli getaccountaddress /accountname/**

```bash
zl@zl-VirtualBox:~$ sudo pivx-cli getaccountaddress mn1
yPrQan4aEuxvbMJnWfdBsnCB5TYxWKNBgS
```

3. 向第二部生成的账户地址内转入10000PIV，保证不多也不少。

**pivx-cli sendtoaddress /address/ /count/**

```bash
zl@zl-VirtualBox:~$ sudo pivx-cli sendtoaddress yPrQan4aEuxvbMJnWfdBsnCB5TYxWKNBgS 10000
636e0aecc9b17a7422edfecee4930a82f8f00eeec3de69d4101f880f319e219a
```

随后我们可以通过listtransactions指令，看到交易信息

**pivx-cli listtransactions**

```bash
  {
    "account": "mn1",
    "address": "yeRKVCMDiwpS2hddkM1kXT5HbT5rtoYXes",
    "category": "receive",
    "amount": 10000.00000000,
    "vout": 1,
    "confirmations": 0,
    "bcconfirmations": 0,
    "txid": "d01e2b5a8d88a556bf5118d6d859c58838372b91255f682a3f79e48ecc5ebf5f",
    "walletconflicts": [
    ],
    "time": 1546998225,
    "timereceived": 1546998225
  },
  {
    "account": "",
    "address": "yeRKVCMDiwpS2hddkM1kXT5HbT5rtoYXes",
    "category": "send",
    "amount": -10000.00000000,
    "vout": 1,
    "fee": -0.00047300,
    "confirmations": 0,
    "bcconfirmations": 0,
    "txid": "d01e2b5a8d88a556bf5118d6d859c58838372b91255f682a3f79e48ecc5ebf5f",
    "walletconflicts": [
    ],
    "time": 1546998225,
    "timereceived": 1546998225
  }

```

但此时我们交易的confirmations为0，没有经过确认。无法在官方指南的第四步中通过masternode outputs指令看到信息。

在这里我们进行一个POW区块的生成，来进行交易信息的确认。

4. 区块生成后，我们可以看到交易的confirmations变为1.之后我们执行masternode outputs指令

**masternode outputs**

```bash
zl@zl-VirtualBox:~$ sudo pivx-cli masternode outputs
[
  {
    "txhash": "636e0aecc9b17a7422edfecee4930a82f8f00eeec3de69d4101f880f319e219a",
    "outputidx": 1
  }
]

```

5. 我们进入到PIVX的文件目录，打开masternode.conf配置文件，要输入的信息如下所示

```
 <Name of Masternode(Use the name you entered earlier for simplicity)> <Unique IP address>:51472 <The result of Step 1> <Result of Step 4> <The number after the long line in Step 4>
 
 Example: MN1 31.14.135.27:51472 892WPpkqbr7sr6Si4fdsfssjjapuFzAXwETCrpPJubnrmU6aKzh c8f4965ea57a68d0e6dd384324dfd28cfbe0c801015b973e7331db8ce018716999 1
```

至此我们配置好了masternode.conf文件。

6. 接下来配置pivx.conf文件

```
server=1
logtimestamps=1
masternode=1
externalip=192.168.152.3
masternodeaddr=192.168.152.3:52888
masternodeprivkey=92MsoEkmbjSyRmXoNap9nn6LPDKD9SPqNP2ExSsEqhwAQ4Twe7n

```

7. 停止PIVX core，之后重新启动。我们通过pivx-cli来启动masternode服务，却看到masternode input必须经过15个确认才可以

**startmasternode local false**

```bash
zl@zl-VirtualBox:~$ sudo pivx-cli startmasternode local false
Masternode input must have at least 15 confirmations

```

那么继续启动POW挖矿，证明数到达要求之后，重新启动masternode

**startmasternode local false**


```bash
zl@zl-VirtualBox:~$ sudo pivx-cli startmasternode local false
Masternode successfully started
```

-----------------------------------------------------------------------

问题：上述guide的结果，真的让我们正常启动masternode了吗？

```bash
zl@zl-VirtualBox:~$ sudo pivx-cli getmasternodestatus

error: {"code":-1,"message":"Masternode not found in the list of available masternodes. Current status: Not capable masternode: Obfuscation Masternode List doesn't include our Masternode, shutting down Masternode pinging service! CTxIn(COutPoint(636e0aecc9b17a7422edfecee4930a82f8f00eeec3de69d4101f880f319e219a, 1), scriptSig=)"}
```

对应debug console显示的信息

```bash
CActiveMasternode::SendMasternodePing() - Relay Masternode Ping vin = CTxIn(COutPoint(636e0aecc9b17a7422edfecee4930a82f8f00eeec3de69d4101f880f319e219a, 1), scriptSig=)
CActiveMasternode::ManageStatus() - Error on Ping: Obfuscation Masternode List doesn't include our Masternode, shutting down Masternode pinging service! CTxIn(COutPoint(636e0aecc9b17a7422edfecee4930a82f8f00eeec3de69d4101f880f319e219a, 1), scriptSig=)

```

该问题对应**activemasternode.cpp**中的函数**SendMasternodePing**

**Masternode is not registered in the network**

```cpp
line 230 -> 234

// Seems like we are trying to send a ping while the Masternode is not registered in the network
errorMessage = "Obfuscation Masternode List doesn't include our Masternode, shutting down Masternode pinging service! " + vin.ToString();
status = ACTIVE_MASTERNODE_NOT_CAPABLE;
notCapableReason = errorMessage;
return false;
```

### 解决方案

我们通过**startmasternode alias false mn1** 可以正常启动masternode结点

```bash
zl@zl-VirtualBox:/usr/local/bin$ sudo pivx-cli startmasternode alias false mn1
{
  "overall": "Successfully started 0 masternodes, failed to start 1, total 1",
  "detail": [
    {
      "alias": "mn1",
      "result": "failed",
      "errorMessage": "Sync in progress. Must wait until sync is complete to start Masternode"
    }
  ]
}
```

此时显示节点需要同步，那么我们检查一下masternode同步信息，通过指令
**pivx-cli mnsync status**

```bash
zl@zl-VirtualBox:/usr/local/bin$ sudo pivx-cli mnsync status
{
  "IsBlockchainSynced": false,
  "lastMasternodeList": 0,
  "lastMasternodeWinner": 0,
  "lastBudgetItem": 0,
  "lastFailure": 0,
  "nCountFailures": 0,
  "sumMasternodeList": 0,
  "sumMasternodeWinner": 0,
  "sumBudgetItemProp": 0,
  "sumBudgetItemFin": 0,
  "countMasternodeList": 0,
  "countMasternodeWinner": 0,
  "countBudgetItemProp": 0,
  "countBudgetItemFin": 0,
  "RequestedMasternodeAssets": 1,
  "RequestedMasternodeAttempt": 1
}

```

显示blockchainsynced = false，即区块链未同步。

我们在这里修改masternode-sync.cpp中的代码，可以看到如果 我们最后一个区块的生成时间+1小时 < 当前时间的话，blockchainsynced就设置为false。

由于我们的测试区块还无法一直生成区块，所以我们先把这里注释掉。

```cpp
// if the last block time plus 1 hour < gettime(), turns up that the chain is not new
// so in the real PIVX network, it should update
// we delete this to let the IsBlockSynced flag = true
/*
if (pindex->nTime + 60 * 60 < GetTime())
    return false;
*/
```

修改之后我们编译重新启动masternode，**pivx-cli stratmasternode alias false mn1**，显示有正常的masternode。我们pivxd的console里边也在显示pivx client不断地ping向masternode。

```bash
zl@zl-VirtualBox:~/projects/PIVX$ sudo pivx-cli masternode count
{
  "total": 1,
  "stable": 1,
  "obfcompat": 1,
  "enabled": 1,
  "inqueue": 1,
  "ipv4": 0,
  "ipv6": 0,
  "onion": 0
}

```

```bash
zl@zl-VirtualBox:~/projects/PIVX$ sudo pivx-cli masternode status
{
  "txhash": "636e0aecc9b17a7422edfecee4930a82f8f00eeec3de69d4101f880f319e219a",
  "outputidx": 1,
  "netaddr": "192.168.152.3:52888",
  "addr": "yPrQan4aEuxvbMJnWfdBsnCB5TYxWKNBgS",
  "status": 4,
  "message": "Masternode successfully started"
}

```

```bash
CActiveMasternode::SendMasternodePing() - Relay Masternode Ping vin = CTxIn(COutPoint(636e0aecc9b17a7422edfecee4930a82f8f00eeec3de69d4101f880f319e219a, 1), scriptSig=)
CActiveMasternode::SendMasternodePing() - Relay Masternode Ping vin = CTxIn(COutPoint(636e0aecc9b17a7422edfecee4930a82f8f00eeec3de69d4101f880f319e219a, 1), scriptSig=)
CActiveMasternode::SendMasternodePing() - Relay Masternode Ping vin = CTxIn(COutPoint(636e0aecc9b17a7422edfecee4930a82f8f00eeec3de69d4101f880f319e219a, 1), scriptSig=)
ResendWalletTransactions()
CActiveMasternode::SendMasternodePing() - Relay Masternode Ping vin = CTxIn(COutPoint(636e0aecc9b17a7422edfecee4930a82f8f00eeec3de69d4101f880f319e219a, 1), scriptSig=)
```

-----------------------------------------------------------------------

## 待解决的问题

1. 如何连接两个masternode
   
   使用**pivx-cli masternodeconnect 192.168.152.5:52888**无效。

2. 如何让mnsync = true, mnsync = true之后，就可以进行正常的staking生成POSblock了。