## 描述一下“以太坊上一笔交易上链的完整过程”

- 以太坊使用共识算法LMD-GHOST+随机抽签，算出全网最重链头，并且随机
从Validator中选出唯一Proposer
- Proposer本地创建Block，从内存池中选出最优交易列表，打包入块
- Proposer用自己的私钥签名并广播Block
- 其他节点收到Block后，进行共识验证。即抽签结果、是否是最重链头、状态转换/签名/Gas/根hash是否合法
- 验证通过后Attestation Committee会对新的Block投attestations（含 LMD 投票 + FFG 投票）。
聚合后写入下一个块的attestations字段
- 下一个Proposer重复第一步，此时上一块以作为最重链头，如果连续两轮2/3Validator见证
则对应的checkpoint被finalized，不可回滚
