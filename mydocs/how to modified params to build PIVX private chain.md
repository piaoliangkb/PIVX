# 建立PIVX私链需要修改的参数

 No. | Name | Function | FINISHED?
---|---|---|---
 1 | [pchMessageStart](#1) | 节点之间通信message的消息头，进行一致性检测 | Y
 2 | [nDefaultPort](#2) | 区块链网络的通信端口 | Y
 3 | [DNSSeeds](#3) | 通过查询DNSSeeds获得网络的众多节点 | Y
 4 | [base58Prefixes](#4) | 表示币地址的前缀 | Y
 5 | [pnSeed6_main](#5) | IPv6 fixed seeds 硬编码ipv6节点 | Y
 6 | [mapCheckpoints](#6) | checkpoints 区块hash检查 | Y
 7 | [block index error](#7) | 待补充修改细节 | N
 8 | [nLastPOWBlock](#8) | 最后一个POW块 | Y
 9 | [nTargetTimespan nTargetSpacing](#9) | 区块生成周期和难度调整周期 | Y
 10 | [nMaturity](#10) | 成熟度，多少个区块确认之后获得奖励 | Y
 11 | [fMiningRequiresPeers](#11) | 挖矿时是否需要其他节点 | Y
 12 | [nStakeMinAge](#12) | stake时候需要coin已经挖出的最短时间 | Y
 13 | [masternode port](#13) | 主节点端口号 | N
 14 | [masternode-sync lastblocktime check](#14) | 主节点同步IsBlockChainSynced标志位 | Y

<span id='1'></span>

## 1. pchMessageStart （magic number）

```cpp
/**
    * The message start string is designed to be unlikely to occur in normal data.
    * The characters are rarely used upper ASCII, not valid as UTF-8, and produce
    * a large 4-byte int at any alignment.
    */
```

介绍magic number的一个网页：[https://bitcoin.stackexchange.com/questions/43189/what-is-the-magic-number-used-in-the-block-structure/43191#43191](https://bitcoin.stackexchange.com/questions/43189/what-is-the-magic-number-used-in-the-block-structure/43191#43191)

修改方式：

src/chainparams.cpp

```cpp
/*
pchMessageStart[0] = 0x90;
pchMessageStart[1] = 0xc4;
pchMessageStart[2] = 0xfd;
pchMessageStart[3] = 0xe9;
*/
pchMessageStart[0] = 0x46;
pchMessageStart[1] = 0x76;
pchMessageStart[2] = 0x66;
pchMessageStart[3] = 0xbb;
```

<span id='2'></span>

## 2. nDefaultPort (default port number)

修改方式：

src/chainparams.cpp

```cpp
/*
nDefaultPort = 51472;
*/
nDefaultPort = 52888;
```

<span id='3'></span>

## 3. DNSSeeds

修改方式：

src/chainparams.cpp

```cpp
/*
vSeeds.push_back(CDNSSeedData("fuzzbawls.pw", "pivx.seed.fuzzbawls.pw"));     // Primary DNS Seeder from Fuzzbawls
vSeeds.push_back(CDNSSeedData("fuzzbawls.pw", "pivx.seed2.fuzzbawls.pw"));    // Secondary DNS Seeder from Fuzzbawls
vSeeds.push_back(CDNSSeedData("coin-server.com", "coin-server.com"));         // Single node address
vSeeds.push_back(CDNSSeedData("s3v3nh4cks.ddns.net", "s3v3nh4cks.ddns.net")); // Single node address
vSeeds.push_back(CDNSSeedData("178.254.23.111", "178.254.23.111"));           // Single node address
*/

vFixedSeeds.clear();
vSeeds.clear();
```

注释掉原Fixed Seeds和DNSSeeds，并调用两个clear()方法。

<span id='4'></span>

## 4. base58Prefixes

表示币地址的前缀:[https://en.bitcoin.it/wiki/List_of_address_prefixes](https://en.bitcoin.it/wiki/List_of_address_prefixes)

可以在TestNet的代码中见到某些特定前缀是什么，也可以在上述网页看到

```cpp
base58Prefixes[PUBKEY_ADDRESS] = std::vector<unsigned char>(1, 139); // Testnet pivx addresses start with 'x' or 'y'
base58Prefixes[SCRIPT_ADDRESS] = std::vector<unsigned char>(1, 19);  // Testnet pivx script addresses start with '8' or '9'
base58Prefixes[SECRET_KEY] = std::vector<unsigned char>(1, 239);     // Testnet private keys start with '9' or 'c' (Bitcoin defaults)
// Testnet pivx BIP32 pubkeys start with 'DRKV'
base58Prefixes[EXT_PUBLIC_KEY] = boost::assign::list_of(0x3a)(0x80)(0x61)(0xa0).convert_to_container<std::vector<unsigned char> >();
// Testnet pivx BIP32 prvkeys start with 'DRKP'
base58Prefixes[EXT_SECRET_KEY] = boost::assign::list_of(0x3a)(0x80)(0x58)(0x37).convert_to_container<std::vector<unsigned char> >();
// Testnet pivx BIP44 coin type is '1' (All coin's testnet default)
base58Prefixes[EXT_COIN_TYPE] = boost::assign::list_of(0x80)(0x00)(0x00)(0x01).convert_to_container<std::vector<unsigned char> >();
```

修改方式：

src/chainparams.cpp

```cpp
/*
base58Prefixes[PUBKEY_ADDRESS] = std::vector<unsigned char>(1, 30);
base58Prefixes[SCRIPT_ADDRESS] = std::vector<unsigned char>(1, 13);
base58Prefixes[SECRET_KEY] = std::vector<unsigned char>(1, 212);
base58Prefixes[EXT_PUBLIC_KEY] = boost::assign::list_of(0x02)(0x2D)(0x25)(0x33).convert_to_container<std::vector<unsigned char> >();
base58Prefixes[EXT_SECRET_KEY] = boost::assign::list_of(0x02)(0x21)(0x31)(0x2B).convert_to_container<std::vector<unsigned char> >();
// 	BIP44 coin type is from https://github.com/satoshilabs/slips/blob/master/slip-0044.md
base58Prefixes[EXT_COIN_TYPE] = boost::assign::list_of(0x80)(0x00)(0x00)(0x77).convert_to_container<std::vector<unsigned char> >();
*/

base58Prefixes[PUBKEY_ADDRESS] = std::vector<unsigned char>(1, 140);
base58Prefixes[SCRIPT_ADDRESS] = std::vector<unsigned char>(1, 19);
base58Prefixes[SECRET_KEY] = std::vector<unsigned char>(1, 239);
base58Prefixes[EXT_PUBLIC_KEY] = boost::assign::list_of(0x04)(0x35)(0x87)(0xCF).convert_to_container<std::vector<unsigned char> >();
base58Prefixes[EXT_SECRET_KEY] = boost::assign::list_of(0x04)(0x35)(0x83)(0x94).convert_to_container<std::vector<unsigned char> >();
// 	BIP44 coin type is from https://github.com/satoshilabs/slips/blob/master/slip-0044.md
base58Prefixes[EXT_COIN_TYPE] = boost::assign::list_of(0x80)(0x35)(0x87)(0x80).convert_to_container<std::vector<unsigned char> >();

```

<span id='5'></span>

## 5. pnSeed6_main[] (fixed seed nodes)

修改方式：

src/chainparamsseeds.h

注释掉所有的ipv6 fixed Seeds

```cpp
/**
 * List of fixed seed nodes for the bitcoin network
 * AUTOGENERATED by contrib/seeds/generate-seeds.py
 *
 * Each line contains a 16-byte IPv6 address and a port.
 * IPv4 as well as onion addresses are wrapped inside a IPv6 address accordingly.
 */
```

```cpp
static SeedSpec6 pnSeed6_main[] = {
    /*
    {{0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0xff,0xff,0x05,0x84,0x9f,0xb5}, 51472},
    {{0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0xff,0xff,0x12,0xda,0xd0,0x0c}, 51472},
    {{0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0xff,0xff,0x17,0x5c,0x35,0x41}, 51472},
@@ -171,6 +172,7 @@ static SeedSpec6 pnSeed6_main[] = {
    {{0xfd,0x87,0xd8,0x7e,0xeb,0x43,0xfe,0x65,0x5c,0xc5,0x5a,0xcb,0xa1,0xc1,0xcc,0xf3}, 51472},
    {{0xfd,0x87,0xd8,0x7e,0xeb,0x43,0x43,0xc8,0xda,0xb1,0x39,0x1f,0xc5,0x93,0x02,0xee}, 51472},
    {{0xfd,0x87,0xd8,0x7e,0xeb,0x43,0xda,0x92,0x41,0xcc,0xc1,0x19,0x07,0xf8,0xe4,0x44}, 51472}
    */
};
```

<span id='6'></span>

## 6. mapCheckpoints 

修改方式：

src/chainparams.cpp

由于我们新的链未生成区块，而genesis block未修改，所以我们注释掉所有区块的checkpoints，暂时添加一个genesis block的hash。生成区块之后我们可以物理修改这些checkpoints。

```cpp
static Checkpoints::MapCheckpoints mapCheckpoints =
    boost::assign::map_list_of
    (0, uint256("0x0000041e482b9b9691d98eefb48473405c0b8ec31b76df3797c74a78680ef818"));
    /*
    (259201, uint256("1c9121bf9329a6234bfd1ea2d91515f19cd96990725265253f4b164283ade5dd"))
    (424998, uint256("f31e381eedb0ed3ed65fcc98cc71f36012bee32e8efd017c4f9fb0620fd35f6b"))
    (616764, uint256("29dd0bd1c59484f290896687b4ffb6a49afa5c498caf61967c69a541f8191557")) //first block to use modifierV2
    (623933, uint256("c7aafa648a0f1450157dc93bd4d7448913a85b7448f803b4ab970d91fc2a7da7"))
    (791150, uint256("8e76f462e4e82d1bd21cb72e1ce1567d4ddda2390f26074ffd1f5d9c270e5e50"))
    (795000, uint256("4423cceeb9fd574137a18733416275a70fdf95283cc79ad976ca399aa424a443"))
    (863787, uint256("5b2482eca24caf2a46bb22e0545db7b7037282733faa3a42ec20542509999a64"))
    (863795, uint256("2ad866818c4866e0d555181daccc628056216c0db431f88a825e84ed4f469067"))
    (863805, uint256("a755bd9a22b63c70d3db474f4b2b61a1f86c835b290a081bb3ec1ba2103eb4cb"))
    (867733, uint256("03b26296bf693de5782c76843d2fb649cb66d4b05550c6a79c047ff7e1c3ae15"))
    (879650, uint256("227e1d2b738b6cd83c46d1d64617934ec899d77cee34336a56e61b71acd10bb2"))
    (895400, uint256("7796a0274a608fac12d400198174e50beda992c1d522e52e5b95b884bc1beac6"))//block that serial# range is enforced
    (895991, uint256("d53013ed7ea5c325b9696c95e07667d6858f8ff7ee13fecfa90827bf3c9ae316"))//network split here
    (908000, uint256("202708f8c289b676fceb832a079ff6b308a28608339acbf7584de533619d014d"))
    (1142400, uint256("98aff9d605bf123247f98b1e3a02567eb5799d208d78ec30fb89737b1c1f79c5"));
    */
```

<span id='7'></span>

## 7. block index error

修改方式：

src/miner.cpp

[https://github.com/piaoliangkb/PIVX/commit/bf9bf7c4979aa44724e2cc030a6943de27836e16](https://github.com/piaoliangkb/PIVX/commit/bf9bf7c4979aa44724e2cc030a6943de27836e16)

<span id='8'></span>

## 8. nLastPOWBlock

修改方式：

src/chainparams.cpp

修改结束POW挖矿的区块编号

```cpp
/*
nLastPOWBlock = 259200;
*/
nLastPOWBlock = 130; // set this to finish POW early
```

<span id='9'></span>

## 9. nTargetTimespan nTargetSpacing 区块生成周期和难度调整周期

修改方式：

src/chainparams.cpp

```cpp
nTargetTimespan = 40 * 10; // PIVX: 40 blocks to cal a new difficulity
nTargetSpacing = 1 * 10;  // PIVX: 10 seconds for 1 block
```

<span id='10'></span>

## 10. nMaturity 成熟度

修改方式：

src/chainparams.cpp

原代码中为100个block之后才能获得挖矿奖励，这里修改为2.

```cpp
nMaturity = 2; // after this blocks to get the award for miner
```

<span id='11'></span>

## 11. fMiningRequiresPeers 挖矿时是否需要其他节点

修改方式：

src/chainparams.cpp

设置为false之后，单个结点挖矿不需要connection

```cpp
// fMiningRequiresPeers = true; // setit to false in 2018.12.26 for POW test
fMiningRequiresPeers = false; // setit to false in 2018.12.26 for POW test
```

<span id='12'></span>

## 12. nStakeMinAge stake时候需要coin已经挖出的最短时间

修改方式：

src/main.cpp

PIVX设置的可以进行stake的余额的最短时间为1 hour(60 * 60s)，这里修改为10 min(10 * 60s)

```cpp
/*
unsigned int nStakeMinAge = 60 * 60;
*/
unsigned int nStakeMinAge = 10 * 60; // SET THIS TIME TO SPEED UP TIME GAP BETWEEN mintablecoin changed from false to true
```

这一标志影响到stakingstatus。输入指令**pivx-cli getstakingstatus**可以看到如下标志情况：

```bash
zl@zl-VirtualBox:/usr/local/bin$ sudo pivx-cli getstakingstatus
{
  "validtime": true,
  "haveconnections": false,
  "walletunlocked": true,
  "mintablecoins": true,
  "enoughcoins": true,
  "mnsync": false,
  "staking status": false
}
```

其中mintablecoins即为该标志位所述影响。

<span id='13'></span>

## 13. masternode port

修改方式：

src/masternodeconfig.cpp

修改masternode的默认端口，原PIVX为51472，为chainparams.cpp中的默认端口，这里修改为哦我们需要的52888.

问题：masternode 无法通过**pivx-cli masternodeconnect ip:52888**连接，无报错信息但是无法连接上。masternode是否通过该种方式连接？

<img src="https://upload-images.jianshu.io/upload_images/11146099-b226e64a8611a525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" width="50%" height="50%" align=left/>

<span id='14'></span>

## 14. masternode-sync lastblocktime check

修改方式：

src/masternode-sync.cpp

在进行masternode同步的时候，会检查最后一个区块的同步时间，代码如下

```cpp
if (pindex->nTime + 60 * 60 < GetTime())
    return false;
```

如果最后一个区块的时间 + 1 hour < 当前时间，则影响IsBlockChainSynced标志位，可以通过**pivx-cli mnsync status**看到。如果IsBlockChainSynced为false，则mnsync为false，即无法进行后续的POS staking。

由于我们设置的区块数目为100，不能持续生成区块，所以我们需要在做实验的时候暂时把这段代码注释掉来保证IsBlockChainSynced标志位的正常。

```cpp
// if the last block time plus 1 hour < gettime(), turns up that the chain is not new
// so in the real PIVX network, it should update
// we delete this to let the IsBlockSynced flag = true
/*
if (pindex->nTime + 60 * 60 < GetTime())
    return false;
*/
```