# NSSK：Needham-Schroeder 对称密钥协议

借助可信服务器分发会话密钥的对称密钥协议，A、B 各自和服务器共享长期密钥。

- 文档：本库根据原论文（Needham & Schroeder, CACM 21(12), 1978）写成的 RFC 风格规范。
- 专家模型：ProVerif 2.05 自带的例子 `NeedhamSchroederSK.pv`（`main.pv`）和 `NeedhamSchroederSK-comp.pv`（`main_pfs.pv`，旧会话密钥会泄露）。
