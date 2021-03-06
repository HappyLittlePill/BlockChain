### 区块链技术与应用

---

> 学习北京大学肖臻老师《区块链技术与应用》公开课笔记
>
> 视频地址：https://www.bilibili.com/video/BV1Vt411X7JF

- [区块链技术与应用](#区块链技术与应用)
  - [01-课程介绍](#01-课程介绍)
  - [02-BTC-密码学原理](#02-btc-密码学原理)
  - [03-BTC-数据结构](#03-btc-数据结构)
  - [04-BTC-协议](#04-btc-协议)
  - [05-BTC-实现](#05-btc-实现)
  - [06-BTC-网络](#06-btc-网络)
  - [07-BTC-挖矿难度](#07-btc-挖矿难度)
  - [08-BTC-挖矿](#08-btc-挖矿)
  - [09-BTC-比特币脚本](#09-btc-比特币脚本)
  - [10-BTC-分叉](#10-btc-分叉)
  - [11-BTC-问答](#11-btc-问答)
  - [12-BTC-匿名性](#12-btc-匿名性)
  - [13-BTC-思考](#13-btc-思考)
  - [14-ETH-以太坊概述](#14-eth-以太坊概述)
  - [15-ETH-账户](#15-eth-账户)
  - [16-ETH-状态树](#16-eth-状态树)
  - [17-ETH-交易树和收据树](#17-eth-交易树和收据树)
  - [18-ETH-GHOST](#18-eth-ghost)
  - [19-ETH-挖矿算法](#19-eth-挖矿算法)
  - [20-ETH-难度调整](#20-eth-难度调整)
  - [21-ETH-权益证明](#21-eth-权益证明)
  - [22-ETH-智能合约](#22-eth-智能合约)
  - [23-ETH-TheDAO](#23-eth-thedao)
  - [24-ETH-反思](#24-eth-反思)
  - [25-ETH-美链](#25-eth-美链)
  - [26-ETH-总结](#26-eth-总结)

#### 01-课程介绍

课程大纲

![20200916201905](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200916203640.png)

![image-20200719160610964](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200916204010.png)

#### 02-BTC-密码学原理

- hash 函数

  x—>H(x)

  hash 碰撞不可避免(collision free：想要人为制造 hash 碰撞是不可行的)

  - Collision resistance：抗碰撞性——内容无法篡改

    1. 用于作信息摘要(message digest)：没有办法篡改内容而又不被检测出来
    2. 没有办法用纯数学、纯理论的方法证明哪一个 hash 函数是不会被碰撞的 一切都是基于经验的

  - hiding：隐藏性——通过 H(x)无法获知 x

    hash 函数的计算过程是单向的 不可逆的

    - 不被蛮力破解的条件：

      1. 输入空间足够大
      2. 各输入可能性分布均匀

    - 用途：

      和 Collision resistance 的性质一起作数字签名(digital commitment / digital equivalent of a sealed envelope)

      <u>实际操作：H（x || nonce）</u>

      x 的取值空间小，在其后附加一个随机数 nonce，使得输入空间足够大，足够随机

  ——以上是密码学中 hash 函数的两个基本性质，比特币中用到的 hash 函数还要求具有下面第三个性质——

  - puzzle friendly:hash 值的计算结果事先无法预知——用作工作量证明（proof of work）

  比特币中用的 hash 函数——SHA256

- 签名

  - 比特币中的账户管理

    - 在本地创建一对公、私钥对(非对称加密体系 asymmetric encryption algorithm)

      注意：加、解密用的是一个人的公钥和私钥

    - 公钥——账号

      私钥——账户密码

  - 比特币是加密货币(crypto-currency)，但实际是不加密的。

    在发布信息时，用自己的私钥作签名，别人可用我的公钥来验证

  - 比特币中用的签名算法，不仅在生成公、私钥时要有好的随机源，之后每次签名时也需要有好的随机源(否则可能泄露私钥)

<u>总结：比特币系统中，一般是先对 message 取 hash，再对 hash 值签名。</u>

#### 03-BTC-数据结构

普通指针：存放结构体在内存中的首地址

Hash 指针：不仅存放地址，还存放结构体的 hash 值(不光能找到结构体的位置，还能检测出该结构体是否被篡改)

区块链：一个个区块组成的链表

- 与普通链表的区别：
  - 用 hash 指针代替普通指针：牵一发而动全身，任何一个区块的篡改，都会影响最后的 hash 指针和系统中对不上

![image-20200720162854939](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200916204019.png)

默克尔树 Merkle tree：类似于二叉树

![image-20200720153455872](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200916235454.png)

![image-20200720153839323](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200916235459.png)

由根 hash 指针确定区块是否遭到了篡改

tx：transaction 交易

Block header:仅存放 root hash，没有交易的具体内容

Block body:存放交易的列表

比特币中的节点分为两类：

全节点：包含 Block header 和 Block body

轻节点：只包含 Block header

![image-20200720163310091](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200916235510.png)

轻节点向全节点请求 merkle proof，全节点返回给它图中标红的 hash 值，之后轻节点可以在本地计算出图中绿色的 hash 值

merkel proof 指的是：上图中 tx、所有绿色的 H()和红色的 H()。即从待证明 tx 到根 hash 的一条路径所涉及到的所有 H()

merkle tree 的作用：提供 merkle proof，证明某一交易是否存在于 merkle tree 中。

(这种证明也叫做 proof of membership / proof of inclusion) 时间复杂度：$Θ(log(n))$ 高效的。

拓展：证明 proof of non-membership

1. 把整棵树传给轻节点。时间复杂度：$Θ（n）$
2. 对每一个叶节点的交易取一次 hash，再排序，对比根 hash。时间复杂度:$Θ(log(n))$

![image-20200720172931583](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200916235529.png)

<u>总结：比特币中两种基本数据结构——区块链、merkle tree 它们都是由 hash 指针来构造。</u>

只要数据结构是无环的，都可以用 hash 指针来代替普通指针，否则会出现循环依赖。

![image-20200720162744195](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200916235529.png)

#### 04-BTC-协议

Double spending attack：双花攻击/花两次攻击

<u>去中心化货币需解决两个问题：</u>

1. 数字货币的发行：谁有发行权？发行多少？什么时候发行？
2. 验证交易的有效性：如何防止 Double spending attack？

![image-20200721140344039](%E5%8C%BA%E5%9D%97%E9%93%BE%E6%8A%80%E6%9C%AF%E4%B8%8E%E5%BA%94%E7%94%A8.assets/image-20200721140344039.png)

- Block header

  - version：用的是哪个比特币版本的协议
  - Hash of previous block header：指向前一个区块的指针(只对 block header 部分取 hash)

  ![image-20200721141959296](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200916235546.png)

  - merkle root hash：整颗 merkle tree 的根 hash 值(保障 block body 里 transaction list 是无法被更改的)
  - target：挖矿的难度 目标阈值的编码 nBits H(block header)≤target
  - nonce：随机数

- Block body

  - Transaction list：交易的列表

- Full node(fully validating node)：全节点（很少）

- Light node：轻节点（绝大多数）
  
  - 无法独立验证交易的合法性



分布式的共识(distributed consensus)——分布式的 hash 表(distributed hash table)

FLP impossibility result 不可能结论

异步系统中，网络传输时延没有上限，只要有一个节点是错误的，就无法达成共识。

CAP Theorem：任何的分布式系统中，以下 3 个性质最多只能满足 2 个，不可能都满足

- Consistency 一致性
- Availability 可用性
- Partition tolerance 分区容忍性

比特币中的共识 Consensus in BitCoin——靠算力来投票

投票资格 membership

联盟链 hyperleger fabric

女巫攻击 sybil attatck

最长合法链 longest valid chain

分叉攻击 forking attack

![image-20200721155156618](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200916235552.png)

![image-20200721155434289](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200916235555.png)

出块奖励 block reward——coinbase transaction——唯一产生比特币的途径

Hash rate 越高-算力越高

挖矿——争夺记账权

#### 05-BTC-实现

比特币采用基于交易的账本模式(Transaction-based ledger)

优势：隐私保护性好

缺点：需要说明币的来源

全节点需要维护 UTXO 数据结构

UTXO(unspent transaction output)：还没被花出去的交易的输出——检测 double spending

![image-20200722140854247](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200916235605.png)

UTXO 集合的每个元素要给出：

- 产生这个输出的交易的 hash 值

- 它在这个交易里是第几个输出

total inputs = total outputs

Transaction fee(交易费)——比特币系统第二个奖励(一般很少)

规定：每隔 21 万个区块，出块奖励减半；每隔 10min 产生一个区块。

$（21万×10分钟）/（60分钟×24小时×365天）≈ 4年$

大约每隔 4 年，出块奖励就减半。

以太坊采用基于账户的账本模式(Account-based ledger)

系统需要显式记录账户上有多少币

Bernoulli trial：a random experiment with binary outcome

Bernoulli process：a sequence of independent Bernoulli trails

- 无记忆性(memoryless)：前面试验的结果对后面试验的结果不会产生影响

poisson process

出块时间服从指数分布(exponential distribution)——性质:memoryless、progress free(挖矿公平性的保障——算力强 10 倍的矿工，挖到矿的概率也始终强 10 倍。否则超过 10 倍，就构成不成比例的差异)

![image-20200722212149798](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200916235616.png)

比特币的总量

集合集 geometric series

$21万×50+21万×25+21万×12.5+···$

$=21万×50（1+1/2+1/4+···）$

$=2100万$

Bitcoin is secured by mining.

挖矿的这个过程对于维护比特币系统的安全性是至关重要的。

防范分叉攻击的办法：多等几个确认（比特币中缺省的是 6 个确认消息，平均 1h）

![image-20200722220714687](%E5%8C%BA%E5%9D%97%E9%93%BE%E6%8A%80%E6%9C%AF%E4%B8%8E%E5%BA%94%E7%94%A8.assets/image-20200722220714687.png)

![image-20200722220815274](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200916235638.png)

比特币协议规定，每个区块的大小是有限制的，最多不超过 1M。

selfish mining：自己挖到下一个区块后先不发布，然后继续沿着自己刚挖到的区块继续往下挖，一直挖下去，直到监听到已经有快赶上自己当前链长时，再将自己所有区块发布出去，率先成为最长链，以达到减少竞争，故意浪费他人算力的目的。（前提是自己的算力足够强，也存在风险）

- 是分叉攻击的一种手段

![image-20200722220654513](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200916235647.png)

- 减少竞争

![image-20200722220558835](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200916235656.png)

![image-20200722220534832](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200916235700.png)

#### 06-BTC-网络

The BitCoin Network

Application layer：Bitcoin block chain

Network layer：P2P overlay network

比特币工作在应用层

在比特币 P2P 网络中，所有的节点都是平等的，没有超级节点(super node/master node)

seed node 种子节点

加入一个网络首先要知道并联系一个种子节点，种子节点会告诉它所知道的网络中其他节点，节点间通过 tcp 协议进行通信。离开时无需进行其他操作，直接退出应用程序，其他节点没有收到你的消息就会自动删除。

比特币网络设计原则：简单、鲁棒，而非高效。simple,robust,but not efficient.

消息的传播在网络中采取 flooding 的方式，当第时一次收到某消息，记录下该消息已被接收，再传递给邻居节点，下一次再收到同一个消息，就不会再转发给邻居节点。邻居节点的选取是随机的，不考虑下层拓扑结构，牺牲效率，增强鲁棒性。

比特币网络的传播是属于 best effort——去中心化系统中面临的实际问题：

- 一个交易发布出去不可能被每一个节点收到
- 不同节点收到这个交易的顺序是不一样的
- 网络传播存在延迟，这个延迟有时会很长
- 有的节点不一定按照比特币协议的规定进行转发(该转发的不转发导致别的节点收不到、转发不合法交易)

#### 07-BTC-挖矿难度

H(block header) ≤ target

target 越小，挖矿难度越大

调整挖矿难度，就是调整目标空间在整个输出空间中所占的比例。

比特币采用的 hash 是 SHA-256 取值空间 2<sup>256</sup>

difficulty = difficulty_1_target/target

difficulty_1_target：难度为 1 时的目标阈值（挖矿难度最小就是 1，对应一个非常大的数）

difficulty 与 target 成反比

不断调整挖矿难度，是为了保持出块时间在一个常数范围波动，不能过短，否则容易造成一个区块后多分叉，不易达成共识，易遭受 51% attack！好的节点被多分叉分散了算力，而坏的节点则集中算力制造最长链，形成所谓的 51% attack。

![image-20200722235820559](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917000218.png)

如何调整挖矿难度：每隔 2016 个区块，要重新调整目标阈值(约为每 2 星期调整一次)

<u>调整目标阈值的公式：target = target ×(actual time/expected time)​</u>

actual time：time spent mining the last 2016 blocks 最近产生 2016 个区块所实际花费的时间

expected time：要保持 10min 的出块时间间隔，2016 个区块理论上要花费的时间(2016×10min)

注：

1. target 阈值的调整是有上下限的，增大/减小幅度不能超过 4 倍
2. target 是 256bit，即 32 个字节，实际在 block header 中，仅保存的是 nBits(target 的编码)，只占 4 个字节
3. 如果有人想要故意调低 target 的难度，方便自身挖矿，则他产出的区块的 block header 里的 nBits 编码是不会被绝大多数节点所接受的
4. 挖矿难度 ≠ target 目标阈值 二者是成反比关系（实际比特币系统源代码中用的是目标阈值 target）

![image-20200723002551945](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917000226.png)

所以要区分图中公式和本节前部分下划线处 target 计算公式的区别

#### 08-BTC-挖矿

![image-20200723162313183](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917000230.png)

![image-20200723162658865](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917000239.png)

比特币网络中大多是轻节点

比特币是如何保障安全的？

- 密码学上的保障（没有私钥，就无法伪造签名）

- 共识机制（占据绝大多数算力的矿工都遵守比特币协议的规定）

挖矿设备的演化：通用—>专用

1. CPU：个人计算机 CPU 通用计算 但只用到了 CPU 很少的功能部件 硬盘内存的使用比也极小 性价比低

2. GPU：主要用于通用并行计算，例如深度学习、大量矩阵乘法。浮点数运算的部件被闲置了，因为挖矿主要是整数运算，只有在深度学习中梯度计算才会用到大量浮点数。所以，GPU 虽然比 CPU 效率高，但仍然有很大浪费

3. ASIC 芯片：Application specific integrated circut

   专门为挖矿设计的芯片，只能用来计算 hash 值

   一种 ASIC 芯片只能为一种特定货币挖矿(专用性），除非是两个加密货币都是用的一个 mining puzzle，这叫做 merge mining

一些新的加密货币采用 Alternative mining puzzle，是为了抗 ASIC(ASIC resistance)，防止生产矿机的厂商抢先使用新型 ASIC 芯片自己先挖矿获利，破坏了挖矿的民主化。

对于单个矿工而言，收益是不稳定的，且还要承担全节点的全部责任——推动矿池的出现

矿主(pool manager)：全节点的其他职责，给每个矿工分配计算任务

矿工(miner)：只需要负责计算 hash 值

收益分配：按贡献量大小分配(工作量证明)——按每个矿工提交的 share 数(almost valid block)分红

【share：降低挖矿难度后，差不多符合要求的区块。只有作工作量证明一个用处，对矿主来说，没有其他用处】

<u>问：矿工偷出块奖励？</u>

答：不可能。每个矿工的任务是由矿主分配，矿主组织好一个区块交给矿工去计算。发布区块包含的 coinbase transaction 有一个收款人地址，为矿主的地址，矿工私自去发布区块是取不出钱的。

<u>问：矿工私自更改 coinbase transaction 的收款人地址？</u>

答：这样提交上去的 share 的交易列表已经被改变，算出的 merkle tree 的根 hash 也不一样，矿主不会认可。相当于矿工从一开始就单干，与矿池没有任何关系。

<u>问：矿工挖到真正合法的区块不提交反而扔掉，只提交一些平时挖到的 share 作工作量证明？</u>

答：存在该情况。矿池间存在竞争关系，对手矿池可能派矿工故意来捣乱，捣乱矿工还能靠一些平时 share 分到出块奖励。

矿池的危害：

矿工转换矿池是很容易的，只需按矿池的协议与矿主联系。无事时分散算力进行正常工作，矿主靠抽取一定比例出块奖励或交易费作为矿池管理运营费获益。如果想要搞破坏，就会降低这个抽取费，甚至赔本吸引大量矿工，来发动 51%攻击。

恶意的矿主（默认算力强）可以发动：

- 51%攻击【仗着算力强，可以回滚已经支付的比特币重新到自己账户】
- 联合抵制某个账户（boycott）【一旦有某个账户的交易，立马分叉产生新区块，无形中使其他矿工也不敢记录该账户的交易】

矿池的好处：

减轻矿工负担，稳定收入

#### 09-BTC-比特币脚本

**_交易实例_**

![image-20200801151503514](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917000307.png)

![image-20200801153000598](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917000329.png)

![image-20200801163147657](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917000337.png)

![image-20200801164751018](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917000841.png)

![image-20200801165452143](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917000907.png)

在早期的比特币实现中，这两个脚本是拼接在一起，从头到尾执行一遍。后来出于安全因素的考虑，这两个脚本改为分别执行。

首先执行输入脚本，如果没有出错，再执行输出脚本，如果能顺利执行，最后栈顶的结果为非零值，也就是 true，这个交易就是合法的，验证通过。如果执行过程中出现任何错误，这个交易就是非法的。

如果一个交易有多个输入，则每个输入脚本都要和其对应的输出脚本进行匹配，全都验证通过，这个交易才算是合法的。

**_输入输出脚本的几种形式_**

**<u>1、P2PK（Pay to Public Key）</u>**

是最简单的形式，因为 PubKey 是直接在输出脚本里给出的

![image-20200801171417362](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917001052.png)

![image-20200801171630302](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917001104.png)

1. 把输入脚本里提供的签名压入栈

![image-20200801171802520](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917001242.png)

2. 把输出脚本里提供的公钥压入栈

![image-20200801171906149](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917001509.png)

3. 把栈顶这两个元素弹出来 用公钥检查签名是否正确。如果正确，返回 true,说明验证通过。否则执行出错，说明交易是非法的。

![image-20200801172226952](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917001516.png)

![image-20200801213057643](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917001525.png)

**<u>2、P2PKH（Pay to Public Key Hash）</u>**

是最常用的形式

与 P2PK 的区别在于：在输出脚本里，没有明确给出收款人的公钥，给出的是公钥的 hash，公钥是在输入脚本里给出的

输入脚本既要给出公钥，也要给出签名

![image-20200801213935005](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917001536.png)

1. 先把签名压入栈

![image-20200801214210321](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917001540.png)

2. 把公钥压入栈

![image-20200801214235899](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917001544.png)

3. 把栈顶元素复制一遍 所以栈顶又多一个公钥

![image-20200801214316893](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917001551.png)

4. 把栈顶的元素取 hash 然后再压入栈

![image-20200801214421072](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917001603.png)

5. 把输出脚本里提供的公钥的 hash 值压入栈

![image-20200801214505947](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917001607.png)

6. 弹出栈顶的两个元素 比较它们是否相等 如果两个 hash 值相等则从栈顶消失

目的在于：防止有人冒名顶替 用自己的公钥冒充收款人的公钥

![image-20200801214742447](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917001615.png)

![image-20200801215138668](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917001620.png)

7. 弹出栈顶的两个元素 用公钥检查签名是否正确 如果签名正确，整个脚本顺利运行结束，栈顶留下 true。如果整个环节中某一处出错，则交易非法。

![image-20200801215525496](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917001628.png)

![image-20200801220252395](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917001645.png)

**<u>3、P2SH（Pay to Script Hash）</u>**

最复杂的一种脚本形式

输出脚本给出的不是收款人的公钥的 hash，而是收款人提供的脚本的 hash（redeemScript 赎回脚本）

![image-20200801220835230](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917001700.png)

![image-20200801221323776](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917001836.png)

![image-20200801222211013](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917001850.png)

**第一阶段的验证**

1. 首先把输入脚本中的 sig 压入栈

![image-20200801224548417](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917001857.png)

seriRS：serialized RedeemScript

RSH：RedeemScriptHash

2. 把赎回脚本压入栈

![image-20200801224738557](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917002020.png)

3. 取 hash 得到赎回脚本的 hash

![image-20200801224816398](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917002009.png)

4. 把输出脚本里给出的 hash 值压入栈

![image-20200801224932755](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917002251.png)

5. 比较两个 hash 值是否相等 如果相等，这两个 hash 值就从栈顶消失

![image-20200801225042704](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917002309.png)

**第二阶段的验证**

1. 把输入脚本里给出的序列化的赎回脚本反序列化（每个节点自己完成）

![image-20200801225503829](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917002322.png)

2. 执行赎回脚本的内容 首先把 PubKey 压入栈

![image-20200801225526487](%E5%8C%BA%E5%9D%97%E9%93%BE%E6%8A%80%E6%9C%AF%E4%B8%8E%E5%BA%94%E7%94%A8.assets/image-20200801225526487.png)

3. 用 CHECKSIG 验证输入脚本里给出的 signature 的正确性

![image-20200801225633888](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917002329.png)

P2SH 最开始是没有的，是通过软分叉加入。

应用场景：对多重签名的支持

比特币系统中一个输出可能需要多个签名才能取出钱（防止其中某个合伙人私钥泄露被盗币，或是其中某个合伙人忘记私钥而大家都取不出钱来。以此提供安全性保护）

![image-20200731224330201](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917080323.png)

表示：N 个合伙人中，只用给出 M 个合伙人的签名即可

注：输入中给出的 M 个签名的顺序，需要和输出中对应的公钥的顺序相同

**CHECKMULTISIG 的脚本执行（原生脚本）**

1. 两个签名要跟它们在公钥中的顺序一致（第 1 个公钥在第 2 个公钥的前面，所以第 1 个签名也要在第 2 个签名的前面）

![image-20200801144906946](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917080335.png)

2. 把多余的元素 FALSE 压入栈

![image-20200801144931207](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917080902.png)

3. 把两个签名依次压入栈。此时输入脚本执行完毕。

![image-20200801145339053](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917080916.png)

4. 接着把输出脚本的里的阈值 M 压入栈

![image-20200801145458024](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917080947.png)

5. 依次把 3 个公钥压入栈

![image-20200801145619327](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917102850.png)

6. 把 N 的值压入栈

![image-20200801145648008](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917102858.png)

7. 最后执行 CHECKMULTISIG，检查堆栈里是否包含了 3 个签名中的 2 个。如果是，则验证通过。

![image-20200801145858821](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917102905.png)

**用 P2SH 实现多重签名**

把复杂度从输出脚本转移到输入脚本（赎回脚本里）

用户只用知道电商提供的 RedeemScriptHash 就可以了，至于电商采用什么样的多重签名规则是无需知道的。在电商需要花这笔钱时，只用在输入脚本给出对应的签名和序列化的赎回脚本即可。

![image-20200802133035049](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917102911.png)

脚本执行：

1. 先把解决 CHECKMULTISIG 的 bug 的无用元素 false 压入栈

![image-20200802133207960](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917102917.png)

2. 把签名压入栈

![image-20200802133233477](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917102928.png)

3. 把序列化的赎回脚本压入栈

![image-20200802133309619](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917102935.png)

4. 把栈顶元素取 hash

![image-20200802133350102](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917102943.png)

5. 把输出里提供的赎回脚本的 hash 值压入栈

![image-20200802133422336](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917102951.png)

6. 判断两个 hash 值是否相等

![image-20200802133448766](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917102957.png)

7. 把赎回脚本展开后执行

![image-20200802133518546](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917103003.png)

8. 先把 M 压入栈

![image-20200802133607216](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917103121.png)

9. 把 3 个公钥压入栈

![image-20200802133633403](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917103127.png)

10. 把 N 压入栈

![image-20200802133651053](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917103152.png)

11. 检查多重签名的正确性 3 个里面有 2 个是正确的即验证通过

![image-20200802133740186](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917103203.png)

使用 P2SH 作多重签名的实例

现在的多重签名多是采用 P2SH 的形式

![image-20200803004036566](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917103211.png)

**<u>4、Proof of Burn</u>**

是销毁比特币的一种办法

RETURN 后面跟 0 个或多个操作，但都返回 false，不会被执行

![image-20200803005955361](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917103224.png)

应用场景：

- 销毁一定数额的比特币，来换取其他小的币种(AltCoin Alternative Coin)
- 利用区块链不可篡改的特性，来写入一些想要长久保留下来的信息（digital commitment）。例如想要保护自己的知识产权，将知识产权的内容取 hash，放在 RETURN 语句之后。任何用户都可以利用销毁很少一点比特币，换取往区块链里写入一些内容的机会（铸币交易的 coinbase 域也可以实现相同功能，但是只有拥有记账权的人才能往里写东西，是发布区块的人；而 Proof of Burn 是任何人都可以写入，是相当于发布一个交易）

![Snipaste_2020-08-03_01-05-59](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917103242.png)

![image-20200803011033369](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917104507.png)

这种形式的脚本的好处是，矿工看到这种脚本时，知道输出永远不可能兑现，所以就没有必要保存到 UTXO 里面，对全节点友好。

总结：比特币中用的脚本语言是非常简单的，以太坊中智能合约用的脚本语言就复杂得多。 比特币脚本语言不支持循环，就不会产生死循环，也就不会出现停机问题。比特币脚本语言虽然看似简单，实则在比特币应用场景做了很多优化，在密码学方面的功能很强大。

#### 10-BTC-分叉

产生分叉的情况：

- State fork： 比特币区块链上产生了状态分歧

  - 两个矿工差不多同时挖到矿 是偶发性的、客观存在的
  - forking attatck（deliberate fork）分叉攻击 是故意产生的、是人为的

- Protocol fork：比特币协议的内容发生了改变，要修改比特币协议需要软件升级，在去中心化系统中，升级软件无法保证所有节点同时都升级软件

  根据对协议修改的内容的不同，分为：

  - 硬分叉（hard fork）
  - 软分叉（soft fork）

<u>硬分叉</u>

想要对比特币协议中加入新特性，**扩展**新功能。而没有进行软件升级的旧节点是不会认可这些新特性、新功能的，会认为是非法的。

例如：增加区块大小限制（block size limit）1M—>4M

![image-20200804133648907](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917104604.png)

认可新特性的新节点算力占大头，当出现分叉时，更容易使包含 4M 大小的区块成为最长链，且新节点也兼容旧节点，它同样认可 1M 大小的区块，当之后最长链上有 1M 区块时，新节点是可以继续沿着挖下去，而旧节点不会，因为前面出现过 4M 区块，旧节点就会认为整个链是不合法的。最终会出现两条平行链，双方各挖各的，新节点始终沿着上面挖（因为算力强，可以始终保持最长链，区块大小有大有小），旧节点始终沿着下面挖（因为上面链尽管保持最长链，但里面存在有大区块始终会被视作非法链）。

两条链会各加一个 chain ID，防止在另一条链上重放、回滚交易，非法获利。

![image-20200804143353953](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917232441.png)

<u>软分叉</u>

对比特币协议加入一些**限制**，使得原来合法的区块或交易，在新的比特币协议中变得不合法。

例如：减小区块大小 1M—>0.5M

![image-20200804142056987](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917232451.png)

系统**不会有永久性的分叉**（硬分叉是永久性的），旧节点如果不尽快升级软件，很可能很多区块都白挖了。

实际中可能出现软分叉的情况：给某些目前协议中没有规定的域，增加新的含义、赋予新的规则。

如铸币交易中的 coinbase 域，它的前 8 个字节可以作 extra nounce。

还有人提议，可以将 UTXO 集合的内容做成 merkle tree，取根 hash，记录到 coinbase 域后面没用上的空间里，coinbase 交易也会被取 hash，层层向上传递，最终被记录到 block header 里。

比特币历史上著名的软分叉例子——P2SH：Pay to Script Hash

P2SH 的验证要分两个阶段：

1、验证输入脚本里给出的 redeemScript 是正确的

2、执行 redeemScript 脚本的内容，验证输入脚本里的给出的签名是正确的

旧节点只知道第一个阶段的验证，而当第二阶段的验证不通过的情况下，旧节点是算作合法的，新节点认为是非法的。新节点认为合法的交易，旧节点肯定认为是合法的。

**总结：**

旧节点**<u>能</u>**接纳新节点——软分叉

旧节点**<u>不能</u>**接纳新节点——硬分叉

软分叉（soft fork）：只要系统中拥有半数以上的算力更新了软件，系统就不会出现永久性的分叉，可能会有临时性的分叉。

硬分叉（hard fork）：必须是所有的节点都更新了软件，系统中才不会出现永久性的分叉。如果有小部分节点不愿意更新，系统就会被分成两条链。

#### 11-BTC-问答

<u>问：转账交易时，收款方不在线怎么办？</u>

答：无需接收方在线，转账交易只需把该笔交易记录到区块链上即可。

<u>问：全节点收到一个转账交易，接收者的地址是从未听到过的，可能吗？</u>

答：可能。想要创建一个账户只需要在本地产生一对公私钥对即可，不用告诉其他人。只有当这个地址第一次收到钱后，其他节点才会知道这个账户的存在。

<u>问：如果比特币账户的私钥丢失，怎么办？</u>

答：没有办法。如果将私钥存放在数字加密货币交易所、在线钱包 APP 等，则可以通过修改密码找回（前提是进行了身份验证），但安全性较低，容易遭受黑客攻击。相较而言，冷钱包、硬件钱包安全性更高。

<u>问：如果私钥泄露了怎么办？</u>

答：一个账户的公私钥对一旦生成，是无法更改的。只能尽快将账户上的比特币抢先转移到另一个安全的账户上去。

<u>问：转账时写错了收款人的地址怎么办？</u>

答：没有办法。没有办法取消一个已经发布到区块链上的交易。

<u>问：Proof of Burn 里既然 OP_RETURN 始终是返回 false，怎么会被记录到区块链里？</u>

答：判断一个交易能不能被写入到区块链、是不是合法的，是将当前交易的输入脚本和币的来源的输出脚本拼接到一起，在整个验证过程不能产生错误。而 Proof of Burn 里的 OP_RETURN 是写在输出脚本中的，是不会被执行的，只有当想花出这笔钱时，才会用到（实际情况是该 output 永远不会被花出去）。

<u>问：如何区分是哪个矿工最先找到的 nounce？而不存在别的矿工偷用这个区块的 nounce，自己把这个区块又发布出去？</u>

答：发布的区块里有一个 coinbase tx，里面有个收款人地址，是挖到这个矿的矿工的地址， 如果要偷答案，就需要改收款人地址为自己的地址，这时 coinbase tx 的内容也会随之改变，导致 merkle tree 的根 hash 值发生改变，根 hash 和 nounce 都是在 block header 里，根 hash 值发生改变，原来找到的 nounce 也就作废了。所以每个矿工挖到的 nounce 是和他自己的收款地址是绑定在一起的。

<u>问：奖励给矿工打包区块的交易费事先怎么知道会是哪个矿工挖到这个矿？</u>

答：事先不需要知道是哪个矿工会得到这笔交易费。只需要始终保证 total inputs ＞ total outputs，这个差额就是交易费。挖到矿的矿工可以自己把这个差额收集起来，作为交易费。

#### 12-BTC-匿名性

银行如果可以使用化名的话，匿名性比比特币好，因为比特币系统的账本是公开的，任何人都可以查询这些账目，但银行只有工作人员和通过司法手段才可以查询某个账户的交易信息。

破坏比特币匿名性：

1. 一个人可以生成很多个地址账户，但这些地址账户是有可能被关联起来的（不同的账户之间可以进行关联）
2. 生成的地址账户可能会和现实中的真实身份产生关联（资金的转入转出与现实世界产生联系时）
3. 实体世界可以用比特币做交易时

比特币的匿名性并不是绝对的。

根据以往经验，用比特币从事非法交易，大概率都会被查出来，除非这些黑钱不在现实中用出去，一直保持在账户中，否则都会泄露身份信息，所以比特币的匿名性并不是人们想的那么好。

匿名性的实现：

网络层：多路径转发（TOR 洋葱路由）

应用层：coin mixing（在线钱包、交易所）

零知识证明

![image-20200806162352523](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917232504.png)

一个有争议的例子：证明一个账户属于我，即我知道这个账户的私钥。我不能直接给出账户的私钥，但可以给出用这个私钥做出的签名，验证者可用该账户的公钥来进行验证。

争议点在于：“无需透露除该陈述是正确的外的任何信息”，实际上给出了用私钥做的签名。应该根据实际的应用场景，具体情况具体分析。

同态隐藏——零知识证明的基础

![image-20200806163134606](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917232514.png)

解释：

1. 逆否命题：加密函数值 E(x)和 E(y)相同，x 和 y 就相同
2. 加密函数是不可逆的：知道加密后的值，无法推出加密前输入的值（和 hash 函数的 hiding property 是类似的）
3. 同态运算：加密后值的和=求和后再加密、加密后值的积=乘积后再加密

![image-20200809151836672](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917232522.png)

![image-20200809151903249](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917232531.png)

盲签名

在不知道具体内容时，还要对它签名

![image-20200809153111862](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917232539.png)

零币和零钞

证明时，从数学上可以保障这个被花费的币是区块链上以前某个合法存在的币，但不知道具体是哪一个，没法追溯（与比特币的不同之处：比特币系统中必须证明花的币，来自于哪个交易的输出）

在与现实世界交互时，仍有可能暴露身份，不能百分百匿名。

![image-20200809153407534](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917232548.png)

#### 13-BTC-思考

![image-20200814162254521](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917232559.png)

- 哈希指针：hash 值本身就是指针，其性质保障整个区块链的内容是不可篡改的

- 区块恋：截断私钥（把私钥划分成几个部分）降低了账户的安全性。

  【对于多个人的共享账户，更推荐使用多重签名】

  无法花出去的比特币、这些死钱会永久保存在 UTXO 集合里越来越臃肿，给矿工维护这个集合造成麻烦

- 分布式共识：为什么比特币系统能够绕过分布式共识中的那些不可能结论？

  比特币系统并没有真正意义上绕过不可能结论，因为达成共识是不能再做更改了，但比特币系统是可以回滚的，甚至可以回滚到创世纪块。并且很多理论上的不可能结论只是在特定的模型下成立，实际应用中往往在基础模型上进行了改动，不可能结论也就不成立了。

  【分布式系统理论有：在异步（通讯传输的延迟没有上限）的环境中，不可能区分某台远程服务器是垮掉了、死机了，还是运行缓慢、网络拥塞】

- 比特币的稀缺性：通常一个好的货币是应该具有通货膨胀的性质。就像早期拿黄金作为通行货币，黄金产出速度是远比不上财富的增长速度，黄金就会越来越值钱，这就会使早期先买黄金的人越来越富，穷的人就会越来越穷（炒房价同理），对整个社会的积极性和生产活力是不好的。

  比特币的总量就是恒定的。

- 量子计算：就算对加密货币有冲击，也会先冲击传统的金融业 ，量子计算机真正投入使用也还有很长时间。就算会对加密货币造成威胁，也会随之产生量子加密算法。在比特币系统中，公钥不是直接给出的，而是给出公钥的 hash 值作为地址，即时公钥丢失了，也可以用私钥推导出来，但公钥不能推出私钥，这些都是由非对称加密体系得出的。

#### 14-ETH-以太坊概述

比特币是区块链 1.0，以太坊就是区块链 2.0

BitCoin：decentralized currency

Ethereum：decentralized contract

以太坊与比特币的区别：

- 以太坊出块时间大幅缩减，只需十几秒，为适应这种出块时间，设计了基于 ghost 的共识机制
- 以太坊的另一改进是挖矿的 mining puzzle，对内存的要求很高（memory hard），以限制 ASIC 芯片的使用（ASIC resistance）。
- 将来以太坊还会有革命性的改变：用权益证明（proof of stake）来替代工作量证明（proof of work），权益证明是用所占权益比例来投票，而不是靠算力，权益证明是不需要挖矿的。
- 对智能合约（smart contract）的支持

以太币的最小单位：wei

**去中心化货币的优势**：比法定货币（fiat currency）进行跨国转账更方便

**去中心化合约的优势**：在没有统一的司法管辖权情况下，可类似于跨国贸易，去中心化的合约能更好维护合约的有效性和被执行。这种智能合约一旦被发布到区块链上就不可更改 ，保障大家就只能按照代码中指定的规则执行，从一开始就不可能违约

#### 15-ETH-账户

比特币：基于交易的账本

每笔交易必须给出币的来源，没花完的币还要新建一个账户，再转回给自己的新账户

![image-20200815165720218](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917232615.png)

以太坊：基于账户的账本

显式记录账户余额，每次交易不用给出币的来源，会自动查询账户下是否有足够的余额，天然防范 double spending attack

![image-20200815171327580](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917232624.png)

防范重放攻击（replay attack）：用一个计数器 nonce 值来记录某账户从一开始到现在，一共完成了多少笔交易。交易数 nonce 值和余额 balance 值一起由全节点维护。

外部账户（externally owned account）：普通账户 由公私钥对控制

合约账户（smart contract account）：不能主动发起一个交易，可以调用另外一个合约（合约在创建时会返回一个地址，通过地址就可以调用合约）

![image-20200816144232144](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200917232632.png)

以太坊支持智能合约，合约要求参与者有比较稳定的身份，如果有隐私保护的需要，也可以创建多个账户进行不同的交易

#### 16-ETH-状态树

账户地址—账户状态的映射

Addr 账户地址 160bit 通常用 40 个 16 进制数表示

State 账户状态(外部账户和合约账户的状态) 包括余额 balance、交易次数 nonce（合约账户还有代码 code、存储 storage）

不用 hash 表来保存维护(addr,state)键值对的原因在于：尽管对 hash 表的插入、更改操作都很方便，但要创建 merkle tree 来提供 merkle proof 和维护各全节点保持一致性，需要花费的代价太大(每构建一个区块，需要遍历一遍，把所有的以太坊账户都记录到 merkle tree，比只记录交易高出好几个数量级)

不能直接用 merkle tree 来保存(addr,state)的原因在于：没有提供一个高效的查找和更新的方法；每个节点记录账户的顺序不同，算出的根 hash 不同，难以保持一致（比特币中 merkle tree 的叶节点交易顺序虽然也不做要求，但实际是由区块的发布者决定的，一旦挖到这个矿了，交易的顺序就是确定的了）。如果用排序的 merkle tree 也是不可取的，新增一个账户也是代价很大的

trie 树（字典树）结构的特点：

优点：

1. 每个节点的分支数目（branching factor），取决于 key 值的取值范围，在以太坊中是 17 个（0~F 加上结束标志符）
2. trie 的查找效率，取决于 key 的长度，以太坊中 key 值为 40，因为以太坊中的地址都是 40 位的十六进制数（以太坊中地址是由公钥取 hash，把前面部分截断，只保留后面 160bit 得到）
3. trie 是不会出现碰撞的，只要地址不同，就能映射到树中的不同分支
4. 只要输入相同，不论插入时的顺序，最后得到的 trie 树是一样的
5. 更新时的局部性很好，只用改动有变化部分，其余账户都保持不变

缺点：

1. 存储有浪费、查找效率低

   【引入 patricia tree/patricia trie 压缩前缀树 进行路径压缩，树变浅了，访问内存的次数也会减少，效率就提高了。但是如果新插入一个单词，原来压缩的路径可能就需要扩展开来。】

![image-20200816214856451](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain@master/img/20200917232908.png)

![image-20200816214927892](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918103119.png)

【树中插入的键值分布比较稀疏时，路径压缩效果比较好。】

![image-20200816215450806](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918103132.png)

用 patricia trie 之后的结果：

![image-20200816215525266](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918103200.png)

在以太坊中，键值是 160 位地址，整个地址空间非常大，分布就很稀疏，用户账户也很难发生碰撞。

MPT（Merkle Patricia tree）

区块链与链表的区别、merkle tree 与 binary tree 的区别、 patricia tree 与 merkle patricia tree 的区别都是：把普通指针换成了 hash 指针

Modified MPT

![image-20200817162226364](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918103210.png)

1 nibble = 1 个 16 进制数 = 4bit = 半字节

Prefix 前缀区分扩展节点、叶子节点和奇数个、偶数个

每个 key 对应的 value 值是下一个节点地址的 hash 值，用的是 hash 指针

![image-20200817200542305](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918103224.png)

系统中每个全节点，需要维护的不是一颗 MPT，而是每产生一个区块，都要新建一个 MPT，只不过状态树中大部分节点都是共享的，只有少数发生变化的节点要新建分支。

以太坊中为了能够回滚，需要保留历史交易信息，每次的操作只能新增，不能删除。

**以太坊中代码的一些数据结构：**

![image-20200817213236757](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918103254.png)

**区块的结构：**

![image-20200817213453564](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918103333.png)

**区块真正在网上发布的时候，发布的信息：**

![image-20200817213603906](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918103427.png)

状态树中保存的是（key，value）—— key：地址 value：账户的状态

value 要存储在状态树中，需要经历一个 RLP 编码序列化的过程

RLP：Recursive Length Prefix

RLP 只支持一种类型：nested array of bytes 嵌套字节数组

protocol buffer（简称：protobuf）： 做序列化的库

#### 17-ETH-交易树和收据树

交易树和收据树的节点一一对应，以太坊中智能合约比较复杂，增加收据树有利于快速查询一些执行的结果。

从数据结构来说，状态树、交易树、收据树都是 MPT。

在状态树中，键值是账户的地址；在交易树和收据树中，键值是交易在发布区块中的序号。

三棵树中，交易树和收据树是把当前发布区块里的交易组织起来，状态树是把系统中所有账户的状态包含进去。

每个区块的状态树大多是共享节点的，而交易树和收据树每个区块是独立的。

交易树和收据树都可以提供 merkle proof。

Bloom filter 数据结构支持高效的查找，判断某个元素是否在某个大的集合里。有可能出现误报，但不会漏报。该数据结构局限性在于，不支持删除操作。

每个交易执行完后会形成一个收据，收据里就包含 bloom filter，记录交易的类型、地址等信息。区块块头中也有一个总的 bloom filter，总的 bloom filter 是区块里所有交易的 bloom filter 的并集。这样做可以快速过滤掉大量无关的区块。

以太坊和比特币都可以看做是交易驱动的状态机（transaction-driven state machine），以太坊中状态是状态树中所有账户的状态，比特币中的状态可以看做 UTXO 集合。共同点是状态的转移都是确定性的。

区块中的数据结构：

![image-20200818185020190](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918103652.png)

![image-20200818185233097](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918103710.png)

![image-20200818185306988](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918103729.png)

![image-20200818185339471](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918103806.png)

![image-20200818191628096](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918103927.png)

![image-20200818191602389](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918103934.png)

![image-20200818193441230](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918103944.png)

![image-20200818193625087](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918103958.png)

![image-20200818193748902](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918104023.png)

![image-20200818193913339](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918104102.png)

![image-20200818193951336](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918104112.png)

#### 18-ETH-GHOST

比特币和以太坊都是运行在应用层的，以太坊采用基于 ghost 协议的共识机制。

![image-20200819133849124](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918104123.png)

最长链上新产出的区块，可以合并最多两个叔父区块，这样叔父区块可以获得安慰奖励（出块奖励值 ×7/8），合并他们的区块，可以获得额外奖励（每合并一个叔父区块，额外奖励出块奖励值 ×1/32）

![image-20200819143111299](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918104143.png)

![image-20200819145221596](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918104150.png)

鼓励出现分叉后，尽早进行合并。包含叔父区块的话，不论是哪一辈的叔父，额外奖励都是 1/32。

叔父区块只用检查其是否是合法区块，即是否满足挖矿难度，叔父区块里包含的交易不会被检查，也不会被执行。

只有分叉过后的第一个区块能够得到 uncle reward，之后跟在它后面的区块都得不到任何奖励，否则分叉攻击的代价就很小，风险就降低了。如下例中 A 想要回滚本次交易，发动分叉攻击，如果攻击失败，整条链还能被当做叔父链获得安慰奖励的话，分叉攻击付出的代价就大大减小了。所以还是鼓励及时合并分叉。

![image-20200819160552911](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918104423.png)

**以太坊中的实例：**

![image-20200819162404569](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918104432.png)

![image-20200819162947295](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918104506.png)

#### 19-ETH-挖矿算法

挖矿是保障区块链安全的重要手段

以太坊也是 memory hard 的 mining puzzle，设计了一大一小两个数据集（16M cache + 1G dataset,DAG），是为了方便验证，轻节点只需要保存小的数据集，矿工才要保存 1G 的 dataset。

dataset 里的每个元素，都是从 cache 里，按照伪随机的顺序，读取 256 个数，不断进行迭代更新，最后得到一个 hash 值，存入 dataset 里。

![image-20200819211035348](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918104524.png)

![](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918104533.png)

![image-20200820150450681](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918104542.png)

![image-20200820151655383](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918104808.png)

![image-20200820152537316](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918104822.png)

以太坊的挖矿算法 ethash，矿工挖矿需要 1G 内存，对大内存的需求使得以太坊的挖矿主要是 GPU 挖矿，达到了 ASIC resistance。

以太坊最初是有 pre-mining 的，还有个相关概念 pre-sale。

Hashrate：系统中所有矿工每秒计算的 hash 次数

不同加密货币，采用的 mining puzzle 不同，hashrate 是不可比的。例如以太坊中尝试一个 nonce 的工作量要比比特币大得多，所以二者的 hashrate 是不可比的。

以太坊的挖矿算法设计认为， 要尽可能让通用计算设备也能参加，参与者越多，挖矿过程越民主，区块链就越安全。

但也有人反对，觉得应该统一都用 ASIC 芯片来挖矿才更安全，因为 ASIC 芯片价格贵，如果要购入大量 ASIC 芯片来发动攻击，投入的开销大，如果攻击成功导致货币信用受损，市值降低，购买硬件的成本可能都没法挽回，反而得不偿失，因而风险也高。如果通用计算设备都能参与挖矿的话，不法分子可以租用大量云服务器，或者本身就拥有很多服务器，一起集中发动攻击，投入的代价就小得多。

#### 20-ETH-难度调整

比特币每隔 2016 个区块才会调整挖矿难度，以太坊是每个区块都有可能调整挖矿难度。

![image-20200821140456074](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918105556.png)

![image-20200821140942717](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918105606.png)

x 是难度调整幅度最小单位

![image-20200821141458006](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918105614.png)

![image-20200821154615000](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918105622.png)

PoS 协议：proof of stake 权益证明

难度炸弹设计的初衷是，以太坊共识未来会由工作量证明转为权益证明，权益证明不需要挖矿，以前投入巨额代价买矿机的矿工就会赔本，为了不使这些亏本矿工联合起来发动硬分叉攻击导致分裂成两个社区，就设计一个难度炸弹，起初挖矿的难度主要是由出块的时间间隔来决定。而难度炸弹是指数函数，到达某个值时，挖矿难度会呈指数级别增长，这时继续挖矿难度会越来越大，迫使矿工们不得不迁移到 PoS 协议。但 PoS 协议开发难度超乎预期，未在指定时间内发布，而挖矿难度又日趋爆炸增长如下图所示，就出现用 H<sup>‘</sup><sub>i</sub>代替原来的 H<sub>i</sub>，让原来真正的区块序号减少三百万个，为权益证明开发争取了时间。

![image-20200821160015536](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918105631.png)

![image-20200821160238923](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918105641.png)

![image-20200821161503334](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918105653.png)

![image-20200821172547451](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918105707.png)

![image-20200821180443595](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918105731.png)

早期挖矿难度的调整是以稳定出块时间为准的

![image-20200821181150331](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918105744.png)

#### 21-ETH-权益证明

比特币总能耗

![image-20200821182715191](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918105757.png)

![image-20200821182735726](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918105806.png)

![image-20200821182756536](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918105853.png)

以太坊总能耗

![image-20200821182815696](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918105907.png)

![image-20200821182852969](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918105915.png)

把比特币+以太坊消耗的总能耗当做一个整体，在国家能源消耗中排第 34 名

![image-20200821182957684](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918105930.png)

权益证明的基本思想（virtual mining）：比特币和以太坊所采用的的工作量证明共识机制，归根结底比拼的是算力，而算力又归根于财力，财力越大，记账权越大。但工作量证明需要挖矿，挖矿需要花费大量能耗，也需要开发专门的矿机。既然归根结底是比钱多钱少，为什么不直接用财力来代替算力，根据财力的比重，来划分收益的比重。大家都把钱集中起来进行区块链的开发和维护，后期再按前期 pre-mining 时持有货币的数量，来参与投票，分配之后的收益。

权益证明相较于工作量证明的优点：

1. 省去了挖矿的过程，也避免了由此带来的对能耗和自然环境的影响，减少温室气体的排放。
2. 工作量证明的区块链系统，维护它安全的资源不是一个闭环。权益证明是闭环（是按照拥有多少这个币种的币来投票。如果要发动攻击，必须设法获得这个币种发行量一半以上的份额才行，发动攻击的资源只能从这个加密货币系统内部得到）。

权益证明和工作量证明并不是互斥的，有的加密货币采用二者的混合模型，仍要挖矿，但挖矿难度与持有多少币相关。

早期权益证明遇到的挑战：两边下注 nothing at stake

以太坊中准备采用的协议：Casper the Friendly Finality Gadget（FFG）

在准备阶段与工作量证明混合使用，为工作量证明提供 Finality，包含在 Finality 中的交易不会被取消。

Casper 协议引入验证者 validator，要成为 validator 必须投入一点以太币作为保证金。validator 负责推动系统达成共识，投票决定哪条链成为最长合法链，投票权重取决于保证金的大小。

起初挖矿时每 100 个区块称为一个 epoch，决定其能否成为一个 Finality 要进行投票。类似于数据库中 two-phase commit，要经过 2 轮投票 prepare message 和 commit message，两轮投票都要有超过 2/3 验证者才通过（按保证金金额来算）。

而实际不再区分两个 message，每个 epoch 从原来 100 区块减少到 50 个，每个 epoch 只投 1 次票，对于前一个 epoch 来说是 commit message，对于后一个 epoch 来说是 prepare message，前后连续 2 个 epoch 都有超过 2/3 投票，才算有效。

![image-20200821220016116](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918105939.png)

![image-20200821220339853](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918105947.png)

EOS 加密货币——权益证明，完全不用挖矿

采用 DPOS 协议：delegated proof of stake

用投票的方法，选出 21 个超级节点，然后再由超级节点产生区块

#### 22-ETH-智能合约

智能合约是以太坊的精髓，也是和比特币的最大区别

什么是智能合约？
➢ 智能合约是运行在区块链上的一段代码，代码的逻辑定义了合约的内容
➢ 智能合约的帐户保存了合约当前的运行状态
balance： 当前余额
nonce： 交易次数
code：合约代码
storage：存储，数据结构是一棵 MPT
➢Solidity 是智能合约最常用的语言，语法上与 JavaScript 很接近

![image-20200822134803138](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918105955.png)

Solidity 是面向对象的编程语言 强类型语言

contract 类似 c++的类 class

Address 是 Solidity 语言特有的

Mapping:hash 表，保存了地址到 uint 的映射，不支持遍历

Solidity 里的数组可以是固定、也可以是动态改变的

Solidity 语言中定义构造函数 2 种方法：（构造函数只能有一个）

1、定义一个与 contract 同名的函数，可以有参数，但不能有返回值

2、用 constructor 定义一个构造函数

<u>外部账户如何调用智能合约？</u>

![image-20200822135703210](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110005.png)

<u>一个合约如何调用另个合约中的函数？</u>

1、直接调用

![image-20200822140325022](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110012.png)

说明：以太坊中规定，一个交易只能由外部账户发起，合约账户不能够自己主动发起一个交易。所以本例实际为一个外部账户，调用了合约 B 中的函数，再调用合约 A 中的函数。

2、使用 address 类型的 call()函数

![image-20200822141618059](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110026.png)

两种方法的区别：在错误处理上，第一种被调合约返回错误，调用合约也会回滚；第二种被调合约返回错误，调用合约只会返回错误，不会受影响。

3、代理调用 delegatecall()

![image-20200822141847625](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110034.png)

![image-20200822144405288](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110042.png)

**智能合约的创建和运行**
➢ 智能合约的代码写完后，要编译成 bytecode
➢ 创建合约：外部帐户发起一个转账交易到 0x0 的地址

- 转账的金额是 0， 但是要支付汽油费
- 合约的代码放在 data 域里

➢ 智能合约运行在 EVM （Ethereum Virtual Machine）上
➢ 以太坊是一个交易驱动的状态机

- 调用智能合约的交易发布到区块链上后，每个矿工都会执行这个交易，从当前状态确定性地转移到下一个状态

![image-20200822155345623](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110057.png)

每个交易处理具有原子性，要么不处理，要么一次处理完，如果交易执行过程中，汽油费不够，会回滚到交易执行前的状态，且不退已经付的汽油费。

![image-20200822160233164](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110108.png)

![image-20200822161814052](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110132.png)

比特币中限制区块的大小不超过 1M，这是写死在比特币协议之中，不可更改；以太坊中限制区块大小的是 gaslimit，每个矿工在出块时，可以在上一个区块的基础上，上调或下调 1/1024。

![image-20200822200551368](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110140.png)

智能合约执行过程中，任何对状态的修改，都是在改全节点本地的数据结构，只有在合约执行完了，发布到区块链上之后，本地的修改才会变成外部可见的，才会变成区块链上的共识。

问：先执行智能合约？还是先挖矿？

答：先执行。必须先执行完这个区块的所有交易，包括智能合约的交易，这样才能更新 3 颗数（状态树、交易树、收据树），才能知道 block header 里的 3 个根 hash 值（Root、TxHash、ReceiptHash），这样 block header 里的内容才能确定，然后才能尝试各个 nonce。

区块链的安全依赖于，所有的全节点要独立验证发布区块的合法性。

发布到区块链上的交易不一定都是成功执行的，因为及时交易执行失败，还是会被发布出去，这样在本地调用合约的这个账户的汽油费才能被真正扣掉，转给矿工自己。

![image-20200823121920064](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110158.png)

Solidity 不支持多线程，因为多个核对内存访问顺序不同的话，执行结果有可能是不确定的。而以太坊是交易驱动的状态机，要求每个状态是完全确定性的。

任何可能导致状态不确定的操作在以太坊中是不可行的，所以在以太坊中不能使用真正的随机数，都是用的伪随机数，不然在不同的全节点验证都不通过。

![image-20200823122020190](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110205.png)

![image-20200823124051888](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110257.png)

![image-20200823124218150](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110317.png)

![image-20200823162251453](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110326.png)

![image-20200823162450166](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110641.png)

![image-20200823163040201](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110651.png)

![image-20200823171445391](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110700.png)

问：出现一个黑客，利用自己的外部账户，也调用一个合约来参与竞拍，但他写的合约里，没有 fallback()函数。当竞拍结束，需要退回竞拍失败者以太币和把最高出价发送给受益人时，到退款转账给黑客合约账户时就会抛出错误（因为没写 fallback()函数），这个时候因为转账采用的是 transfer 的方式，出现错误会连锁回滚，回滚只会回到 auctionEnd()执行之前，而所有竞拍者转账给竞拍合约账户的交易都已经被写入区块链没法更改，此时所有的人都没法取回以太币，包括受益者。有解决办法吗？

![image-20200823185528698](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110708.png)

答：没有解决办法。

Code is law：智能合约的规则是由代码逻辑决定的，代码一旦发布到区块链上，就不可更改了。好处是没人能够篡改规则，坏处是规则中有漏洞也没法更改。所以智能合约写得不好，就有可能把以太币永久得锁起来，谁也取不出来。

![image-20200823221046801](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110739.png)

右边先账户清零，再转账，转账不成功再恢复的操作，是和其他合约发送交互的经典编程模式。

在区块链上，任何未知的合约，都可能是有恶意的。所以每次向对方转账，或是调用对方某个函数时，需要时刻提防对方合约反调用自己的合约并修改状态。

![image-20200823221132701](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110751.png)

左边账户清零的操作，只有当上一步转账操作结束后才能执行，而上一步的转账操作又陷入了与右边 fallback()函数的递归调用中，根本执行不到清零操作，只要判断条件成立（拍卖合约账户余额>退款给黑客的金额 且 当前调用还剩的汽油费>6000 且 调用栈的深度<500），合约账户就会一直向黑客账户转账。

![image-20200823221252732](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110825.png)

修改方法：先清零，再转账，如果转账失败再恢复。且使用 send 函数替代 call.value，因为 send 函数转账时向对方合约账户发送 2300 汽油费，这点汽油费只够对方合约写个 log，不足以发起下一次调用。

#### 23-ETH-TheDAO

比特币实现了去中心化的货币，以太坊实现了去中心化的合约

DAO：Decentralized Autonomous Organization 去中心化的自治组织

DAC：Decentralized Autonomous Corporation 去中心化的自治公司

二者区别：DAC 是出于盈利目的，DAO 可以是出于非盈利目的公益事业

汽油费的机制很大一方面是为了防范 Dos 攻击。防止恶意攻击者不断发布非法交易，浪费矿工的资源。

本节主要讲述的是 TheDAO 事件，和其所导致的以太坊分叉，可以参考文章：[以太坊分叉的缘由：The DAO 事件究竟是怎么回事？](https://www.8btc.com/article/326359)

#### 24-ETH-反思

<u>反思 1：智能合约真的智能吗？</u>

答：智能合约并不智能，只是按写死的程序自动执行某些操作，一旦写好就没法更改。可以看做是程序合约或自动合约。

<u>反思 2：不可篡改性是一把双刃剑</u>

答：一方面来说，不可篡改性增加了公信力，没有人能够篡改规则；另一方面，如果规则中存在漏洞，想要修补漏洞、软件升级都是很困难的。所以针对 TheDAO 事件，智能合约一旦发布到区块链上也不能阻止别人去调用它，但对于剩下的 2/3 还未被黑客盗走的以太币，可以以同样的黑客手段调用智能合约把剩下的钱转移到安全的账户。

<u>反思 3：没有什么是真正不可篡改的</u>

答：代码是死的，人是活的。如果发生重大事件，迫不得已要修改代码、回滚交易，还是可以做到的。就像 TheDAO 事件最后的解决方案，还是得靠软件升级硬分叉，来退回所有 TheDAO 基金参与者投入的以太币。

<u>反思 4：Solidity 语言设计上有没有问题？</u>

答：Solidity 语言在设计上确实还有很多待改进的地方，但是否应该采用函数式编程语言、形式化证明(formal verification)去证明程序的正确性，还有待探讨。

<u>反思 5：开源真的更安全吗？</u>

答：开源提高了公信力，每一位参与者都能看到程序软件、智能合约中的具体代码，受人民监督。但真正去仔细研究源码的人少之又少，且检查代码本身需要具备相关专业知识，检查出潜在漏洞也不是件容易的事，大部分人还是靠信任有公信力的组织、机构、智能合约来进行交易。

<u>反思 6：去中心化意味着什么？</u>

答：去中心化中存在的分叉恰恰是民主的体现，反对者可以自己发动分叉，维护自己所信仰的体系结构。

<u>反思 7：去中心化 ≠ 分布式</u>

答：一个去中心化的系统必然是分布式的，但分布式系统不一定是去中心化的。

分布式的主要应用是给多台计算机分配不同的任务，最后把所有计算机的执行结果汇总，以期望达到 10 台计算机效率是 1 台计算机效率 10 倍的线性增速（实际情况却远远达不到，最多 6、7 倍），是并行处理的、以求提高效率的。大规模计算服务的云计算就是分布式的主要应用。

比特币、以太坊都是交易驱动的状态机，区块链里的分布式是为了达成共识、统一状态、提高容错的，10 台计算机的效率还没有 1 台计算机的效率高。其中智能合约是用来编写控制逻辑的，只有那些需要在互不信任的实体之间建立共识的操作，才需要写进智能合约里。

#### 25-ETH-美链

美链（Beauty Chain）是一个部署在以太坊上的智能合约，有自己的代币 BEC。

- 没有自己的区块链，代币的发行、转账都是通过调用智能合约中的函数来完成的
- 可以自己定义发行规则，每个账户有多少代币也是保存在智能合约的状态变量里
- ERC 20(Ethereum Request for Comments)是以太坊上发行代币的一个标准，规范了所有发行代币的合约应该实现的功能和遵循的接口
- 美链中有一个叫 batchTransfer 的函数， 它的功能是向多个接收者发送代币，然后把这些代币从调用者的帐户上扣除

**美链里因为 batchTransfer 函数，发起的攻击：**

![image-20200825150242423](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110847.png)

![image-20200825150635573](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110857.png)

![image-20200825150734542](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110907.png)

![image-20200825150749265](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110914.png)

![image-20200825150954699](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110922.png)

反思：在进行数值运算时，一定要考虑溢出的可能性。

![image-20200825151015367](https://cdn.jsdelivr.net/gh/HappyLittlePill/BlockChain/img/20200918110933.png)

Solidity 语言提供了一个 SafeMath 库，调用该库函数进行数值运算，可以轻易检测出是否发生溢出。

#### 26-ETH-总结

成功的商业模式既可以包含中心化成分，也可以包含去中心化的成分。比特币、以太坊只是运用了区块链技术的支付方式，而使用这些去中心化支付方式的商业模式不一定也是去中心化的。

加密货币不应该用在当下常规货币使用得很好的领域，而应该用在已有的支付方式解决得不是很好的领域。如： 全球范围内的支付，缺乏统一的流通货币 information can flow freely on the internet,but payment cannot。未来发展预测支付渠道和信息传播渠道逐渐融合，使得价值交换变得和信息传播一样方便。

判断一种支付手段效率的高低，要在当时的历史条件下去看，跟当时存在的其他支付手段作对比。

