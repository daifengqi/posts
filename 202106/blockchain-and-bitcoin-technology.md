# 比特币与区块链技术

# Bitcoin And Blockchain

## 2021 年 6 月 17 日

## Jun 17 2021



> 参考：[肖臻的比特币课程](https://www.bilibili.com/video/BV1Vt411X7JF)。



### 密码学中的哈希函数（cryptographic hash function）

**哈希函数的两个性质：**

1. collision resistance（哈希碰撞抗性）：哈希碰撞的定义是，x不等于y，而H(x)等于H(y)；

   哈希碰撞是不可避免的，因为输入空间是远大于输出空间的（通常是`2^256`）。

   另外，没有高效地人为制造的哈希碰撞的方法，即给定H(x)，你只能随机地去找y，使得H(x)等于H(y)。

   <u>应用</u>：message digest

> MD5曾经是一个很流行的Hash函数，但是现在已经有人为地制造MD5哈希碰撞的方法，所以不再推荐使用MD5算法。

2. hiding（隐蔽性）

   哈希函数的计算过程是**单向不可逆**的，即根据输出H(x)，是无法反推出x的，即哈希值没有泄露关于输入值的信息。

   <u>应用</u>：digital commitment/equivalent (of a sealed envelop)

> 如果有个股神说自己可以预测明天哪只股票可以涨停，别人要求证明这个能力，那么就应该把预测结果封装在一个信封里（sealed envelop)，到第二天打开，才能证明正确性，否则，如果提前说了预测结果，那么可能会影响股市。
>
> 要怎么<u>封印</u>这个预测结果呢？答案是使用哈希函数。

注意，以上两个性质都可以通过蛮力求解（brute-force）的方式破解，要抵抗暴力破解，哈希函数的输入空间应该具备如下性质：

- 输入域足够大

  输入空间不够的解决办法：在输入x后面拼接一个随机数nonce，以达到输入空间大且随机分布的性质，即`H(x || nonce)`，其中`||`是字符串拼接符。

- 分布足够均匀（熵），用上课的话来说，就是"High min-entropy"。

> “High min-entropy” means that the distribution is “very spread out”, so that no particular value is chosen with more than negligible probability.

**比特币中的哈希函数**

比特币用到的哈希函数还需要第三个性质：puzzle friendly。

哈希值的计算是事先不可预测的，光是看输入x，是猜不出结果H(x)，所以如果想要输出的哈希值是落在某个范围之内的，那没有什么好办法，只能一个一个输入随机去试，看哪个输入恰好是落在要求的范围之内的。

puzzle friendly这个性质和collision resistance有一点像，它们确实有一定的联系，但不完全一致。

### 签名

**比特币用到的密码学有两个关键技术，一个是哈希，另一个就是签名。**上一节我们讲了哈希，另一个就是签名。

**比特币怎么做到去中心化呢？开户的过程不是到某个机构，比如银行去证明，而是由个人创建自己的公钥/私钥对，一个公钥/私钥对就代表了一个账户。**

> 公钥私钥的概念来自于非对称加密（asymmetric encryption algorithm）体系。最早的加密体系都是对称加密（没有那个a）体系，即加密解密用的都是同一个key，使用对称加密的优点是速度快，但缺点是密钥的分发不是很方便。
>
> 非对称加密的缺点是速度慢。非对称加密的通信是不需要知道对方的私钥的，只需要通过对方的公开的公钥来加密，然后把信息发送给对方，对方用私钥解密，就可以。
>
> 公钥就相当于账号，私钥就相当于账户密码。

公钥/私钥技术就是签名的基础：在比特币交易中，我用私钥来签名发起的交易（比如转账给别人），别人用公钥来验证。

有一种理论上的攻击方式是，不停地创造公钥/私钥，拿公钥去比特币账户上比对，如果有相同的公钥，那么因为你也有私钥，你就有了比特币的交易权。这种攻击方式仅存在于理论上，因为对于256位的公钥来说，创建出相同公钥的概率是1/2^256，这个概率比地球爆炸的概率还低。

注意一个概念，a good source of randomness，因为计算机生成的都是伪随机数，而伪随机数的生成依赖于随机元，这个随机元在两个地方都很重要，

- 公私钥对的生成
- 每一次交易的签名

如果随机元的source被发现，那么依然可以得到生成的随机数，那就"全完了"。

### 比特币中的数据结构

**一、哈希指针**：普通的指针存放的是某个结构体在内存中的地址（起始位置），哈希指针除了存地址之外，还要保存地址的哈希值。这样做的好处是，从哈希指针中不只能找到结构体的位置，还可以检测结构体的内容是否被篡改，因为我们保存了哈希值。

比特币最核心的结构就是区块链，那么区块链和普通的链表有什么区别呢？

- 用哈希指针代替了普通指针；

通过这样的数据结构可以实现**tamper-evident log**，我只要保存最后一个哈希值，就可以知道前面的值是否被改动。

有了这一个性质，比特币中的一些节点就不一定要保存整条区块链的内容，比如说它可以只保存最近的几千个区块，以前的就不用存了，那么如果用到以前的区块怎么办呢，向其他节点要就是了。**那怎么知道其他节点给到的信息是正确的呢？计算区块的哈希值和自己保存的哈希值是否一致就可以了**

**二、Merkle tree**：Merkle tree和binary tree的区别之一是，用哈希指针代替了普通指针（指向左子节点和右子节点）。Merkle tree的子节点是数据库（data block），内部的叶节点和根节点都是哈希指针。根节点的哈希称为root hash，只要保存根哈希值，就能检测数中任何数据的修改。

**Merkle tree相对于哈希指针链表有更高的效率，它使用树的结构代替了链表结构，使得验证的时间复杂度由O(n)下降到O(logn)。**

在区块链的block header里面，有Merkle tree的根哈希值，而没有交易的具体内容；交易的列表储存在block body里面。Merkle tree有什么用呢，

- Merkle proof （又名 proof of membership/inclusion）

> 全节点（full node）和轻节点（light node）：一般手机上的钱包都是轻节点，轻节点只储存block header，block header上只有根哈希值；要在轻节点上证明一笔交易，需要先找到交易所在的位置，然后验证从交易往上一直到根节点的路径，这条验证路径就被称为Merkle proof。
>
> 证明方需要提供Merkle proof。

![merkle](https://cescdf.com/image/merkleroot.png)

一次Merkle proof需要轻节点向其他全节点发送请求，返回一些Hash值，就上图的例子而言，就是那三个红色的`H()`。

collision resistance性质保证了制造不出其他节点的哈希碰撞，所以Merkle proof是无法造假的。

- proof of non-membership

Merkle tree已经可以做到证明存在性，那么如何证明一笔交易不在Merkle tree里呢？一个直观的想法是，传递整个Merkle tree。即使这样，证明的复杂度也是O(n)级别的。

我们可以对data block，即叶节点，按照某个顺序排序（可以直接按哈希值排序）。这种排好序的Merkle tree，称为Sorted Merkle tree。

**由于比特币不需要证明不存在性，所以比特币只用了Merkle tree，而没有用Sorted Merkle tree。**

> 哈希指针在哪些场合可以替代普通指针？只要无环就可以，因为有环会存在哈希值循环依赖的问题。

### 协议：输入输出

> 思考：一个中心化的数字货币体系是否只需要非对称加密，即私钥/公钥体系，央行用私钥加密，老百姓用公钥解密，来确定一个货币是signed by central bank的，这里并不需要区块链。

这样的数字货币体系是可以的吗？不行，因为会有双花问题（double-spending）。这就是相对于传统货币，数字货币面临的主要挑战。

要改进这个问题，可以引入一个数据结构，央行可以维护一个大的数据库，一张Map，通过数字货币的编号映射到持有人。每次支付都需要同时验证货币真实性和持有人，这就解决了双花问题。

这又引入了另一个问题，即这是一个中心化的体系，每一次支付都需要在央行的表中做一次查询，这是不切实际的。因此，我们要做一个去中心化的协议。

> 通过上面的描述，比特币的技术核心就在于解决了双花和中心化的问题。

一个去中心化的货币需要解决两个货币：谁发货币？怎么验证交易有效性？

**比特币通过区块链的结构解决了“验证”问题。这个结构通过分布式的节点由全体参与方共同维护。**

比特币的每次交易中都有输入和输出两个部分，

- 输入：说明币的来源（避免双花）

- 输出：给出收款人公钥的哈希

比特币体系中，收款地址是通过公钥的哈希推算出来的，地址就相当于银行账号。若A要给B转账，B就要通过某些比特币以外的渠道告诉A自己的公钥。

收款人B也要知道A的公钥，实际上，所有节点都要知道A的公钥，这样才能验证币确实是A持有的。问题来了，怎么才能知道A的公钥？写在交易的输入里就行！，所以

- 输入：说明币的来源（避免双花），以及付款人的公钥（验证）

**输入里付款人的公钥必须和来源的交易块输出里的收款人的公钥对上，这样才能证明“自己花自己的钱”，避免冒充花别人的钱。**所以，

**验证一次交易的过程：**拼接输入（付款）的脚本和上次交易（币的来源）的输出脚本，拼接起来的脚本运行一次，如果是脚本运行正确的那么校验通过。

> 为什么input脚本要在output脚本之前？因为在压入栈计算时，要先通过后压入的output得到公钥的哈希，然后解开input上的签名，才能验证付款人是否确实持有这一块的币。

所以，要理解一句话：配对的input和output脚本不是来自同一个区块的。

### 协议：区块结构

Block Header：宏观信息，比如用的比特币哪个版本的协议，区块链中指向前一个区块的指针，整个Merkle tree的根哈希值，还有两个和挖坑相关的，难度阈值target的编码nBit和随机值nonce。**注意，前一个哈希一般只算区块的块头。**

Block Body：交易列表（transaction list）

> 全节点（full node）保存所有区块信息，轻节点（light node）只保存block header的信息，一般来说，轻节点无法独自验证一笔交易。比特币中有大量的轻节点。

问题：在分布式的节点中，账本的内容怎样取得分布式共识（一致性）。

> 分布式系统中的不可能结论（imposiibility result）：
>
> - FLP：FLP是三个作者姓名的开头，内容是，在一个异步的系统里，网络传输时延没有上限，哪怕一个成员是faulty的，也无法达成共识。
> - CAP Theorem：C指Consistency，A指Availability，P指Partition tolerance，任何一个分布式系统，这三个性质中只能满足其中两个，这是一个“不可能三角”。
>
> 分布式共识的一个比较著名的协议是Paxos，这个协议能保证一致性（Consistency）和可用性（Availablity）。某些情况下（概率很小），Paxos协议会一直无法达成共识。

比特币要解决的共识协议，需要解决一个问题：在小部分有恶意的（malicious） 节点的情况下，当保证大部分节点是正确时，协议是一致的。

- 方式一：全体投票，多数通过（联盟链可以采用）

这个方法有几个问题，最主要的是女巫攻击（Sybil attack），只要有一个计算机不停地产生公私钥对，这就可以获取很多投票权（membership），从而操纵结果。

**比特币中如何解决这个问题：1. 通过算力投票。**

引入一个概念，longest valid chain（最长合法链）。比特币的区块应该接在最长合法链上，否则属于forking attack（分叉攻击）。

在最长合法链的前提下，如果出现了两个等长的分叉，每个节点都接收自己先接到的那个区块，暂时维持一段时间，然后看哪个区块是“往下扩展的区块”，当一个节点继续沿着某个区块往下扩展，就表明认可了链上的所有区块。

**2. block reward**

orphan block（没有后续链的区块），或者不在最长合法链上的区块，是不会得到回报的，或者说得到的回报是无效的，所以矿工都希望在最长合法链上继续挖矿，这就避免了在某些恶意节点上的分叉上挖矿的问题，因为大多数诚实的节点是不会接受的。

当投票是靠算力保证时，投票的权重不会因为更多的账户而改变，这避免了sybil attack。所以有一句老话说，Bitcoin is secured by mining。只要大多数算力是诚实的，那么这个系统就是安全的。只要有恶意的节点不超过50%，恶意节点产生的block就不会写入最长合法链里（因为大多数诚实节点不会接受这个block继续延伸），这对攻击者的损失是很大的，不仅没得到block reward，还浪费了算力。

比特币协议中需要等6个confirmation，才认为（6个之前的）前面的交易是不可篡改的，平均时间也就是等1个小时。

> Coinbase Transaction：一个区块的Merkle tree控制的交易列表中有一个特殊的交易，即Coinbase transaction，这个交易就是矿工给自己发的钱（block reward），这里面可以写任何东西，比如人生感想：挖矿不容易，且挖且珍惜。
>
> 通过调整Coinbase transaction的值，就可以调整Merkle tree root hash，也就能影响最终区块的哈希，通过调整这个hash也许能够找到满足target的哈希值。
>
> 所以，比特币的挖矿过程实际上只有两重循环，外循环是coinbase transaction影响的merkle tree的哈希，内循环是nonce，矿工就是通过这两重循环来找到符合条件的hash值。

### 比特币系统的实现：UTXO

**UTXO（Unspent Transaction Output）：所有没有被花掉的交易的输出组成的集合。**

注意，同一个交易里因为有多个输出，所以同一个交易里的有些输出可能还在UTXO里，有些已经不在了。

**UTXO的每个元素需要提供两个信息：**

1. **产生这个输出的交易的哈希值**
2. **该元素在这个交易里是第几个输出**

这两个信息就可以定位到这笔没被花的钱。

UTXO存在的目的是为了检测Double-spending。

全节点要在内存中维护UTXO，以便快速检测Double-spending。事实上，一个中心化的账本要维护的一个数据结构就是UTXO。

 每个交易会消耗掉一些输出，同时也会产生一些新的输出，所以每次交易都会更新UTXO。

**total inputs = total outputs + (transaction fee）**

一个交易可能有多个来自不同地址的输入（这就需要多个签名），也可能有多个输出，这时候，必须保证，要么输入的总金额等于输出的总金额，要么等于输出加付给矿工的交易费。

**交易费是比特币系统设计的第二个激励机制。**

> 每隔21万个区块回报减半，10分钟产生一个区块，所以大约每4年（4年是一个计算出来的属性）减半一次。

### 比特币系统的实现：数学性质

一、指数分布的无记忆性

每次尝试哈希值都是一次伯努利试验，伯努利过程是无记忆的，当试验次数足够大时，可以用Poisson过程来近似，而Poisson分布的等待时间和是指数分布，指数分布具有无记忆性。

所以，这个性质告诉我们，不管你挖矿尝试了多少分钟，之后尝试的时间和已经消耗的工作量是没有关系的。或者，这叫做puzzle friendly的性质，这恰恰是挖矿公平性的保证。

否则，如果，挖到矿的概率和已经做过的工作量成正相关，那么算力不平衡将会被放大，导致不公平。

二、几何过程的和上限

> 每21万（210000）个区块回报减半，第一个是50个回报。

比特币最终能产生的总量计算：

21万个区块 * 50个比特币 + 21万个区块 * 25个比特币  + ...

= 21万 * 50个 * （1 + 1/2 + 1/4...）

= 21万 * 50个 * 2

= 2100万个 （21 * 10 ……6）

### 比特币网络（The BitCoin Network）<-分布式

在应用层，是比特币区块链协议（BitCoin Block chain），在网络层，是P2P Overlay Network。**比特币的每个网络节点都是平等的**。

**关键性质，**

- simple, robust but not efficient（简单、鲁棒而不是高效）
- flooding（随机的拓扑结构，与距离无关，这么做也是为了robust，所以向邻居转账和向其他国家转账的速度是差不多的）
- best effort（尽最大努力的交付）

**加入/退出：**加入节点网络只需要告诉一个seed node（种子节点），然后它会告诉你可以连接哪些节点；退出节点时不需要做任何事情，只需要关闭程序，其他节点一段时间没有接收到你的消息，就会自动把你抹去。

**维护交易：**每个节点要维护一个等待上链的交易的集合，交易产生的时候先被验证有效，如果有效就会被转发，然后就会被邻居节点记录。注意，如果同时产生了冲突的交易，那么每个节点以先收到的有效交易为准，之后哪个交易能上到最长合法链就看运气了。比特币协议对区块大小的限制是1m。

**转发区块：**节点收到交易后打包到区块上，挖到矿之后就转发block，节点只有第一次收到block时会转发，避免endless loop。

> flooding：消息传播采取flooding的方式，节点第一次收到消息的时候，做两件事情，
>
> 1. 转发给邻居节点，
> 2. 记录，已经收到这个交易了，下次收到的时候就不做任何处理。

### 挖矿和挖矿难度

> H(block header) <= target

挖矿难度和目标阈值是成<u>反比</u>的。

比特币的出块时间是10分钟，以太坊只要15秒，所以出块速度是bit的40倍。这种情况下，以太坊就需要一些新的共识协议，它不能丢弃orphan block，而是也要给它们一些奖励。

> ***比特币*每产生*2016个区块*调整一次挖矿难度。**

2016个区块，每个区块十分钟，一天有60*24分钟，所以每隔14天调整一次挖矿难度。

> target = target * (actual time / expected time)，其中expected time 是两星期（14天）。实际时间比预期时间长了，就要调高阈值，降低难度。

**调整挖矿难度的目的是：适应算力的增长，使得出块时间保持稳定。**

**🌟全节点和轻节点**

*全节点：*

1. 一直在线
2. 本地硬盘上维护完整的区块链信息
3. 内存里维护UTXO集合
4. 监听比特币网络上的每个交易信息，验证每个交易的合法性
5. 决定哪些交易被打包到区块里
6. 监听别的矿工挖出来的区块，验证区块的合法性
7. **挖矿**：决定沿着哪条链挖下去，当出现等长分叉时，选择最先听到的。

*轻节点（SPV：simplified payment vertification）：*

1. 不是一直在线
2. 只保存block header
3. 只保存与自己相关的交易
4. 只能验证与自己相关的交易合法性
5. 无法检测网上发布的区块正确性
6. 可以验证挖矿的难度（发布的区块是否符合难度要求）
7. 只能检测哪个是<u>最长链</u>，但是不知道哪个是<u>最长合法链</u>

**🌟比特币交易的保证**

1. 密码学的性质，保证了只有私钥的人才能转账；
2. 系统中拥有大多数算力的矿工是诚实的，不会接受那些没有合法签名的交易。

只有这两条同时保障，才能确保交易合法性。

### 比特币脚本

每个交易的输入需要说明币的来源，即txid，是之前的交易的哈希值，vout表示之前的交易的第几个输出，

```json
"vin": [{
  "txid": "...",
  "vout": 0,
  "scriptSig": {
    "asm": "...",
    "hex": "..."
  },
}]
```

签名scriptSig是证明有权花这个钱。

交易的输出如下，

```json
"voud": [{
  "value": ...,
  "n": 0,
  "scriptPubKey": {
  "asm": "DUP HASH160 ... EQUALVERIFY CHECKSIG",
  "hex": "...",
  "reqSigs": 1,
  "type": "pubkeyhash",
  "addresses": ["..."]
	},
},{
  "value" : ...,
  "n": 1
  // ...
}
]
```

交易也可以有多个输出，通过索引序号n控制分别给谁转多少。

三种形式：

> 注意，一个输入通常与两个输出脚本有关系，一个来自以前的区块，一个是现在的区块。

**🌟P2PK（Pay to Public Key）**

input script:

- PUSHDATA(Sig)

output script:

- PUSHDATA(PubKey)
- CHECKSIG

*！注意，上面这个公钥（PubKey）是和签名（Sig）的私钥配对的，也就是说output scirpt里提供的是付款人的公钥，是来自上一笔交易*

**🌟P2PKH（Pay to Public Key Hash）**

input script:

- PUSHDATA(Sig)
- PUSHDATA(PubKey)

output script:

- DUP
- HASH160
- PUSHDATA(PubKeyHash)
- EQUALVERIFY
- CHECKSIG

这是最常用的。

**🌟P2SH（Pay to Script Hash）**

input script:

- ...
- PUSHDATA(Sig)
- ...
- PUSHDATA(serialized redeemScript)

output script

- HASH160
- PUSHDATA(redeemScriptHash)
- EQUAL

这是最复杂的，收款人提供一个赎回脚本的哈希。

> 验证分为两步：
>
> 1⃣️：验证**序列化的赎回脚本**是否和输出脚本中的哈希值匹配
>
> 2⃣️：执行**反序列化的赎回脚本**，验证输入脚本中给出的签名是否正确
>
> 赎回脚本有以下几种形式：
>
> 1. P2PK
> 2. P2PKH
> 3. 多重签名

一个用P2SH实现P2PK的例子如下

redeemScript

- PUSHDATA(PubKey)
- CHECKSIG

input script

- PUSHDATA(Sig)
- PUSHDATA(serialized redeemScript)

output script

- HASH160
- PUSHDATA(redeemScriptHash)
- EQUAL

注意，P2SH的验证一定是两阶段的，第一阶段是验证序列化的赎回脚本，第二阶段把赎回脚本反序列化，然后执行、验证。

**P2SH最常用的情景是多重签名**。

本质是把复杂度从输出脚本转移到了赎回脚本，赎回脚本的哈希在输出脚本里提供，也就是说，由收款人提供。

### 比特币分叉（fork）

state fork（临时性分叉）

1. forking attack（分叉攻击）

2. deliberate fork（人为故意分叉）

protocol fork（协议升级带来的分叉）

- hard fork
- soft fork

比如说，有些人认为1m区块的限制太小了，算一算，比特币一个区块最多1m，也就是一百万比特，一个交易大概250比特，所以一个区块大概4000个交易；一个区块十分钟，再摊下来，大概就是每秒钟7笔交易。**这个速度是非常低的。**

**🌟硬分叉（hard fork）**

假设，有人发布了一个软件更新，把block size limit从1m增加到4m，然后系统中的大多数算力都承认这个更新，那么系统运行起来后新的节点大多数都是4m的（更新后的）节点类型，所以（新规则下的）最长合法链会被新规则定义的链替代，这时分叉就会变成永久性的，

- 更新后的大部分节点会沿着新的最长合法链挖
- 没更新的少部分节点会继续沿着按以前规则定义的区块挖

**以上情况就会永久性地出现两条链，而且双方都只挖自己的fork，平行运行，所以叫hard fork。**

> hard fork会导致社区分裂。

**🌟软分叉（hard fork）**

典型的例子是，让block size limit减少，比如到0.5m，大多数系统承认这个更新，新节点认为0.5m是限制，旧节点认为1m是限制。

这时候会导致一种情况，旧节点如果不更新软件的话，还是会认为新分叉是合法的，所以会继续沿着挖，然而新节点不认同旧节点挖的区块，**所以旧节点做的工作就白费了**。

**这种分叉总是临时性的，旧节点必须立刻更新，不然它做的工作都不会被认可。所以，系统不会有永久性的软分叉。**

> 实际中可能会有软分叉的情况：赋予新的规则。