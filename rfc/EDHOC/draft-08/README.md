# EDHOC draft-08

EDHOC（Ephemeral Diffie-Hellman Over COSE）的早期草案：为受限设备设计的三消息认证密钥交换，有两种认证方式。

- 文档：draft-selander-ace-cose-ecdhe-08（2018 年 3 月）。
- 专家模型：原本是一个同时包含两种认证方式的 ProVerif 模型，这里按认证方式拆成两份 `main.pv`，每份只保留本方式的进程和查询。原模型的出处库里没有记录。
- 标准答案：`subprotocols/handshake/asymmetric/`（签名认证）、`subprotocols/handshake/symmetric/`（预共享密钥认证）。
