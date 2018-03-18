从运行bitcoind开始

99.99%的人都是从比特币入门区块链的, 剩下那个当然是中本聪啦😈

个人觉得搞这个东西还是要熟悉一些概念后亲自跑一跑程序, 操作一下, 不然很难有深刻的印象(搞计算机技术好像都是这一个套路, 可能其他行业也是)

源码安装

废话不多说，开搞，建议开个虚拟机，因为依赖环境比较多，别把你宿主机搞得不兼容了，毕竟C++库不是随便装的

    git clone https://github.com/bitcoin/bitcoin.git
    #这个是我当时用的稳定版
    git checkout v0.16.0 

接着直接运行这个shell脚本

    pi@raspberrypi:~/project/bitcoin $ ./autogen.sh
    configuration failed, please install autoconf first
    
    #装autoconf
    sudo apt-get install autoconf

再来一遍 ./autogen.sh， 可能会报如下错误

    /usr/share/automake-1.15/am/depend2.am: error: am__fastdepCXX does not appear in AM_CONDITIONAL
    /usr/share/automake-1.15/am/depend2.am:   The usual way to define 'am__fastdepCXX' is to add 'AC_PROG_CXX'
    /usr/share/automake-1.15/am/depend2.am:   to 'configure.ac' and run 'aclocal' and 'autoconf' again
    /usr/share/automake-1.15/am/depend2.am: error: AMDEP does not appear in AM_CONDITIONAL
    /usr/share/automake-1.15/am/depend2.am:   The usual way to define 'AMDEP' is to add one of the compiler tests
    /usr/share/automake-1.15/am/depend2.am:     AC_PROG_CC, AC_PROG_CXX, AC_PROG_OBJC, AC_PROG_OBJCXX,
    /usr/share/automake-1.15/am/depend2.am:     AM_PROG_AS, AM_PROG_GCJ, AM_PROG_UPC
    /usr/share/automake-1.15/am/depend2.am:   to 'configure.ac' and run 'aclocal' and 'autoconf' again
    Makefile.am: error: C++ source seen but 'CXX' is undefined
    Makefile.am:   The usual way to define 'CXX' is to add 'AC_PROG_CXX'
    Makefile.am:   to 'configure.ac' and run 'autoconf' again.
    parallel-tests: installing 'build-aux/test-driver'
    autoreconf: automake failed with exit status: 1

这是因为缺少libtool

    sudo apt-get install libtool

索性一波全装了吧

    #索性一波全装上
    sudo apt-get install build-essential libtool autotools-dev automake pkg-config libssl-dev libevent-dev bsdmainutils python3
    
    #C++的boost库
    sudo apt-get install libboost-system-dev libboost-filesystem-dev libboost-chrono-dev libboost-program-options-dev libboost-test-dev libboost-thread-dev
    
    ##如果上面装了编不过去
    sudo apt-get install libboost-all-dev
    
    #qt的东西，如果你需要图形界面的话
    sudo apt-get install libqt5gui5 libqt5core5a libqt5dbus5 qttools5-dev qttools5-dev-tools libprotobuf-dev protobuf-compiler
    

安装Berkeley DB  4.8版本，我编译装的5.2，这个只牵扯到钱包的兼容性，不过在我的ubuntu上钱包运行的也正常

    #官方说这个地址有编译好的ubuntu Berkeley DB4.8
    https://launchpad.net/~bitcoin/+archive/bitcoin
    #添加repo
    sudo apt-get install software-properties-common
    sudo add-apt-repository ppa:bitcoin/bitcoin
    sudo apt-get update
    sudo apt-get install libdb4.8-dev libdb4.8++-dev

编译

    #忽略libdb的版本检查, 不然不给过, 当然4.8版本可以不加
    #include 和 lib路径如果是用apt-get 安装的libdb可以不加
    #建议加上--prefix指定个单独的目录
    ./configure --with-incompatible-bdb LDFLAGS="-L /home/username/bitcoin/db4/lib/" CPPFLAGS="-I /home/username/bitcoin/db4/include/" --prefix=<path>

成功生成MakeFile文件后当然是

    sudo make && make install

在安装路径下有如下文件就安装成功啦

    ├── bin
    │   ├── bench_bitcoin
    │   ├── bitcoin-cli
    │   ├── bitcoind
    │   ├── bitcoin-tx
    │   └── test_bitcoin
    ├── include
    │   └── bitcoinconsensus.h
    ├── lib
    │   ├── libbitcoinconsensus.a
    │   ├── libbitcoinconsensus.la
    │   ├── libbitcoinconsensus.so -> libbitcoinconsensus.so.0.0.0
    │   ├── libbitcoinconsensus.so.0 -> libbitcoinconsensus.so.0.0.0
    │   ├── libbitcoinconsensus.so.0.0.0
    │   └── pkgconfig
    │       └── libbitcoinconsensus.pc
    └── share
        └── man
            └── man1
                ├── bitcoin-cli.1
                ├── bitcoind.1
                └── bitcoin-tx.1



其中bin/bitcoind 是节点服务程序, 运行起来你就建立了一个比特币节点，而且默认会在~/.bitcoin下同步全网链上数据, 少说也有几百个G吧, 最好还是外挂个硬盘，在你准备存储比特币网络数据的文件夹下创建bitcoin.conf, 并且写入你的json-rpc账户密码

    rpcuser=your_pass_username
    rpcpassword=your_password

启动bitcoind

    ./bitcoind -datadir=<your_data_path>

同时可以看下<your_data_path>下面的debug.log, 大概能看出来点bitcoind大概启动时都干了点什么，几个线程都是干嘛的

    17:41:59 Bitcoin Core version v0.16.0 (release build)
    17:41:59 InitParameterInteraction: parameter interaction: -whitelistforcerelay=1 -> setting -whitelistrelay=1
    17:41:59 Assuming ancestors of block 0000000000000000005214481d2d96f898e3d5416e43359c145944a909d242e0 have valid signatures.
    17:41:59 Setting nMinimumChainWork=000000000000000000000000000000000000000000f91c579d57cad4bc5278cc
    17:41:59 Using the 'sse4' SHA256 implementation

当然用top 命令也能看

    1311488 307976  18992 S  0.0  7.7   0:12.96 bitcoind
    1311488 307976  18992 S  0.0  7.7   0:00.00 bitcoin-scriptc
    1311488 307976  18992 S  0.0  7.7   0:00.29 bitcoin-schedul
    1311488 307976  18992 S  0.0  7.7   0:00.00 bitcoin-http
    1311488 307976  18992 S  0.0  7.7   0:00.00 bitcoin-httpwor
    1311488 307976  18992 S  0.0  7.7   0:00.00 bitcoin-httpwor
    1311488 307976  18992 S  0.0  7.7   0:00.00 bitcoin-httpwor
    1311488 307976  18992 S  0.0  7.7   0:00.00 bitcoin-httpwor
    1311488 307976  18992 S  0.0  7.7   0:29.62 bitcoind
    1311488 307976  18992 S  0.0  7.7   0:00.00 bitcoin-torcont
    1311488 307976  18992 S  0.0  7.7   0:01.61 bitcoin-net
    1311488 307976  18992 S  0.0  7.7   0:00.01 bitcoin-addcon
    1311488 307976  18992 S  0.0  7.7   0:00.04 bitcoin-opencon
    1311488 307976  18992 D  0.0  7.7   0:20.56 bitcoin-msghand

一些补充

我们的节点同步全网数据需要很长时间, 有资源的同学可以去搞一份拷贝, 或者知道有临近的高速全节点的, 可以手动加入到我们节点连接中去

我个人把节点部署到了<树莓派> 上, 对你没看错, 可以跑得起来, 系统都是debian系的, 安装过程差不多, 折腾折腾就玩了, 切记make的时候用一个核, 2,3,4我都试过跑不到两分钟就得挂, 这玩意还是很脆弱的, 同步数据和操作基本都是耗在io上, 所以拿来搞还省电, 多好

如果你在树莓派上挂载NTFS格式硬盘, 就算是777的权限, 文件系统也是无法写入的

     #安装这个工具
     sudo apt-get install ntfs-3g
     #在/etc/fstab中添加
     /dev/sda1 /mnt/myusbdrive ntfs-3g defaults,noexec,umask=0000 0 0
     #重启就ok



开始与节点交互

bin/bitcoin-cli 是一个命令行工具，通过json-rpc与我们的节点进程交互，可以通过以下两个参数设置节点ip和port，默认本地

      -rpcconnect=<ip>
           Send commands to node running on <ip> (default: 127.0.0.1)
    
      -rpcport=<port>
           Connect to JSON-RPC on <port> (default: 8332 or testnet: 18332)

查看我们运行节点的基本信息

    ./bitcoin-cli  -getinfo
    {
      "version": 160000,           #客户端版本编号
      "protocolversion": 70015,    #协议编号
      "walletversion": 159900,     #钱包编号
      "balance": 0.00000000,       #余额
      "blocks": 411418,            #已同步的区块高度
      "timeoffset": -13,
      "connections": 8,            #节点连接数
      "proxy": "",                 #代理
      "difficulty": 194254820283.444, #pow 难度
      "testnet": false,
      "keypoololdest": 1520946917,
      "keypoolsize": 1000,         #地址池大小
      "unlocked_until": 0,         #钱包解锁到期时间
      "paytxfee": 0.00000000,
      "relayfee": 0.00001000,
      "warnings": ""
    }
    

获取账户所有地址

    ./bitcoin-cli getaddressesbyaccount ''                                
    [
      "3AjyTTHRsGELnRxvcSsVKdKHkY4zRwzkcQ",
      "3EgCto1Qfa9VhBmMFqbg6hSEYmdMfCxMKV",
      "3LmfCAA2KyyXE7gX314ZGVgTUJvvx59RRd"
    ]

获取对应地址的私钥WIF格式

    ./bitcoin-cli dumpprivkey 3AjyTTHRsGELnRxvcSsVKdKHkY4zRwzkcQ
    KyjMQnhdUTbC4psCVjahyvrKkKYvU89ogFnFF1Nm184bp87evhhT

因为dumpprivkey命令是提取钱包中的私钥, 是getnewaddress命令生成的, 并不是反推出来的(不要想多了), 所以要运行walletpassphrase命令解锁钱包

获得一笔交易代码

     ./bitcoin-cli getrawtransaction fe28050b93faea61fa88c4c630f0e1f0a1c24d
    0082dd0e10d369e13212128f33
    01000000010000000000000000000000000000000000000000000000000000000000000000ffffffff0804ffff001d02fd04ffffffff0100f2052a01000000434104f5eeb2b10c944c6b9fbcfff
    94c35bdeecd93df977882babc7f3a2cf7f5c81d3b09a68db7f0e04f21de5d4230e75e6dbe7ad16eefe0d4325a62067dc6f369446aac00000000

解码一笔交易，获得一笔交易的详细信息

	这里面可以看到此交易是区块上第一个交易，是系统对于矿工的奖励50个btc，如果你看一个最近产生的区块，奖励已经是25个btc了，如果是普通用户交易，vin内部字段也是不一样的，这是基于btc的TUXO模型，这个后面的文章会详细介绍

    ./bitcoin-cli  decoderawtransaction 01000000010000000000000000000000000000000000000000000000000000000000000000ffffffff0804ffff001d02fd04ffffffff0100f2052a01000000434104f5eeb2b10c944c6b9fbcfff94c35bdeecd93df977882babc7f3a2cf7f5c81d3b09a68db7f0e04f21de5d4230e75e6dbe7ad16eefe0d4325a62067dc6f369446aac00000000
    {
      "txid": "fe28050b93faea61fa88c4c630f0e1f0a1c24d0082dd0e10d369e13212128f33",
      "hash": "fe28050b93faea61fa88c4c630f0e1f0a1c24d0082dd0e10d369e13212128f33",
      "version": 1,
      "size": 135,
      "vsize": 135,
      "locktime": 0,
      "vin": [
        {
          "coinbase": "04ffff001d02fd04",
          "sequence": 4294967295
        }
      ],
      "vout": [
        {
          "value": 50.00000000,
          "n": 0,
          "scriptPubKey": {
            "asm": "04f5eeb2b10c944c6b9fbcfff94c35bdeecd93df977882babc7f3a2cf7f5c81d3b09a68db7f0e04f21de5d4230e75e6dbe7ad16eefe0d4325a62067dc6f369446a OP_CHECKSIG",
            "hex": "4104f5eeb2b10c944c6b9fbcfff94c35bdeecd93df977882babc7f3a2cf7f5c81d3b09a68db7f0e04f21de5d4230e75e6dbe7ad16eefe0d4325a62067dc6f369446aac",
            "reqSigs": 1,
            "type": "pubkey",
            "addresses": [
              "1BW18n7MfpU35q4MTBSk8pse3XzQF8XvzT"
            ]
          }
        }
      ]
    }
    



获得这笔交易所在区块的信息

    ./bitcoin-cli  getblock 00000000c937983704a73af28acdec37b049d214adbda81d7e2a3dd146f6ed09
    {
      "hash": "00000000c937983704a73af28acdec37b049d214adbda81d7e2a3dd146f6ed09",
      "confirmations": 386154,
      "strippedsize": 216,
      "size": 216,
      "weight": 864,
      "height": 1000,
      "version": 1,
      "versionHex": "00000001",
      "merkleroot": "fe28050b93faea61fa88c4c630f0e1f0a1c24d0082dd0e10d369e13212128f33",
      "tx": [
        "fe28050b93faea61fa88c4c630f0e1f0a1c24d0082dd0e10d369e13212128f33"
      ],
      "time": 1232346882,
      "mediantime": 1232344831,
      "nonce": 2595206198,
      "bits": "1d00ffff",
      "difficulty": 1,
      "chainwork": "000000000000000000000000000000000000000000000000000003e903e903e9",
      "previousblockhash": "0000000008e647742775a230787d66fdf92c46a48c896bfbc85cdc8acc67e87d",
      "nextblockhash": "00000000a2887344f8db859e372e7e4bc26b23b9de340f725afbf2edb265b4c6"
    }
    

通过高度获取区块哈希

	我们获取高度为1000的区块的哈希值，可以看到结果是和上面那条命令的哈希值相同

    ./bitcoin-cli  getblockhash 1000
    00000000c937983704a73af28acdec37b049d214adbda81d7e2a3dd146f6ed09

	借此我们看下中本聪😈创世区块的内容

    ./bitcoin-cli getbloackhash 0
    000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f
    
    ./bitcoin-cli getblock 000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f
    {
      "hash": "000000000019d6689c085ae165831e934ff763ae46a2a6c172b3f1b60a8ce26f",
      "confirmations": 1,
      "strippedsize": 285,
      "size": 285,
      "weight": 1140,
      "height": 0,
      "version": 1,
      "versionHex": "00000001",
      "merkleroot": "4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b",
      "tx": [
        "4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b"
      ],
      "time": 1231006505,
      "mediantime": 1231006505,
      "nonce": 2083236893,
      "bits": "1d00ffff",
      "difficulty": 1,
      "chainwork": "0000000000000000000000000000000000000000000000000000000100010001"
    }
    

	接下来我们想获取这个交易的详细信息，可是系统提示创世块禁止检索，这是因为创世交易没有添加进交易数据库中，所以从你本身的节点是检索不到的，至于为什么，这是中本聪😈自己的设计

    ./bitcoin-cli -datadir=/media/liuyang/My\ Passport/bitcoin_data/ getrawtransaction 4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b
    
    error code: -5
    error message:
    The genesis block coinbase is not considered an ordinary transaction and cannot be retrieved

	google了一下有人说这笔交易被硬编码到了最近的bitcoin core client中, 这是一份json dump,  有空看下这块的代码，找找写在哪里了.

    {
        "txid" : "4a5e1e4baab89f3a32518a88c31bc87f618f76673e2cc77ab2127b7afdeda33b",
        "version" : 1,
        "locktime" : 0,
        "vin" : [
            {
                "coinbase" : "04ffff001d0104455468652054696d65732030332f4a616e2f32303039204368616e63656c6c6f72206f6e206272696e6b206f66207365636f6e64206261696c6f757420666f722062616e6b73",
                "sequence" : 4294967295
            }
        ],
        "vout" : [
            {
                "value" : 50.00000000,
                "n" : 0,
                "scriptPubKey" : {
                    "asm" : "04678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5f OP_CHECKSIG",
                    "hex" : "4104678afdb0fe5548271967f1a67130b7105cd6a828e03909a67962e0ea1f61deb649f6bc3f4cef38c4f35504e51ec112de5c384df7ba0b8d578a4c702b6bf11d5fac",
                    "reqSigs" : 1,
                    "type" : "pubkey",
                    "addresses" : [
                        "1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa"
                    ]
                }
            }
        ]
    }



总结

本篇我们从编译安装比特币节点开始，用命令行工具与我们搭建的节点进行了一些简单的交互，简单地探索了一下链上区块，其实还有非常多的操作没有介绍，这些大家可以自己多尝试一下, JSON-RPC返回的json格式数据也非常有益于我们编程去做一些链上数据处理，比如地址关联分析，交易关系可视化等都非常有趣.

如果觉得有所收获， 不要说话， 请用币砸我 :)

狗狗币地址:

DDTVkcxGWBbtMp6ArPYmXRvRTryVRQVn3U

ERC20代币地址:

0x807ed647c8572C368976497EA4cb3eA812533b53

