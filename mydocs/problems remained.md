# 需要解决的几个问题

## 1. RequestedMasternodeAssets 1

重现问题：

pivxd启动参数为**pivxd -printtoconsole -dubug**

需要添加-debug命令

出错日志：

```bash
CMasternodeSync::Process() - tick 5471 RequestedMasternodeAssets 1
CMasternodeSync::Process() - tick 5476 RequestedMasternodeAssets 1
CMasternodeSync::Process() - tick 5481 RequestedMasternodeAssets 1
CMasternodeSync::Process() - tick 5486 RequestedMasternodeAssets 1
CObfuscationPool::CheckTimeout() -- Session timed out (30s) -- resetting
CMasternodeSync::Process() - tick 5491 RequestedMasternodeAssets 1
CMasternodeSync::Process() - tick 5496 RequestedMasternodeAssets 1
CMasternodeSync::Process() - tick 5501 RequestedMasternodeAssets 1
CMasternodeSync::Process() - tick 5506 RequestedMasternodeAssets 1
CMasternodeSync::Process() - tick 5511 RequestedMasternodeAssets 1
CMasternodeSync::Process() - tick 5516 RequestedMasternodeAssets 1
CObfuscationPool::CheckTimeout() -- Session timed out (30s) -- resetting
CMasternodeSync::Process() - tick 5521 RequestedMasternodeAssets 1
CMasternodeSync::Process() - tick 5526 RequestedMasternodeAssets 1

```

错误原因分析：

Masternode sync状态一直保持在 **MASTERNODE_SYNC_SPORKS**=1状态

1. **masternode-sync.h**文件中定义了各个状态对应的数值

    ```cpp
    #define MASTERNODE_SYNC_INITIAL 0
    #define MASTERNODE_SYNC_SPORKS 1
    #define MASTERNODE_SYNC_LIST 2
    #define MASTERNODE_SYNC_MNW 3
    #define MASTERNODE_SYNC_BUDGET 4
    #define MASTERNODE_SYNC_BUDGET_PROP 10
    #define MASTERNODE_SYNC_BUDGET_FIN 11
    #define MASTERNODE_SYNC_FAILED 998
    #define MASTERNODE_SYNC_FINISHED 999
    ```

2. **masternode-sync.cpp**中272行对结点进行遍历，如果处在**MASTERNODE_SYNC_SPORKS=1**的状态,需要**RequestedMasternodeAttempt**变量>=2才可进入到next asserts.每有一个结点**RequestedMasternodeAttempt**都会+1.猜测需要>2个结点才能进行到masternode-sync的下一步。

    ```cpp
    BOOST_FOREACH (CNode* pnode, vNodes) {
        // ..............

        //set to synced
        if (RequestedMasternodeAssets == MASTERNODE_SYNC_SPORKS) {
            // log here

            if (pnode->HasFulfilledRequest("getspork")) continue;
            pnode->FulfilledRequest("getspork");

            pnode->PushMessage("getsporks"); //get current network sporks
            if (RequestedMasternodeAttempt >= 2) GetNextAsset();
            RequestedMasternodeAttempt++;
            // LOGPRINTF

            return;
        }
    ```

3. 等待多个节点网络的测试进行mnsync的下一步。

## 2. nMasternodeCountDrift

通过**MasternodeCountDrift()**函数返回这个值，在**chainparams.cpp**中默认值为20，testnet中的默认值为4.

用来动态调整主节点奖励的一个参数

## 3. masternodeconnect 无法正常连接

配置好masternode之后，两台机器都通过

```bash
pix-cli startmasternode alias false mn1
```

的方式正常启动masternode,使用

```bash
pivx-cli masternodeconnect ip：port
```

的方式无法连接，没有日志打印。

## 4. 如何启动POS mining

可以通过**getstakingstatus**命令查看staking的状况，目前只差mnsync = true即可启动。

等待调试如何让masternode正常同步。