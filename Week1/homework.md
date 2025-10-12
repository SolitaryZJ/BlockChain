## 描述一下“以太坊上一笔交易上链的完整过程”

- PoW
  - 矿工节点从mempool中挑选gas最高的交易，打包并本地执行，构建MPT
  - 然后矿工节点通过计算mining puzzle，即调整block header中nonce来算出符合难度阈值的block header
  - 节点私钥签名该block并广播相邻节点
  - 节点收到block后，进行共识验证
  - 验证通过后，节点将block写入数据库并广播P2P网络中


- PoS
  - 以太坊使用共识算法LMD-GHOST+随机抽签，算出全网最重链头，并且随机
    从Validator中选出唯一Proposer
  - Proposer本地创建Block，从内存池中选出最优交易列表，打包入块
  - Proposer用自己的私钥签名并广播Block
  - 其他节点收到Block后，进行共识验证。即抽签结果、是否是最重链头、状态转换/签名/Gas/根hash是否合法
  - 验证通过后Attestation Committee会对新的Block投attestations（含 LMD 投票 + FFG 投票）。
  聚合后写入下一个块的attestations字段
  - 下一个Proposer重复第一步，此时上一块以作为最重链头，如果连续两轮2/3Validator见证
  则对应的checkpoint被finalized，不可回滚
