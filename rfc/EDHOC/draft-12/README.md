# EDHOC draft-12

EDHOC 在 IETF LAKE 工作组的草案：三消息认证密钥交换，双方各自选择签名或静态 DH 认证。

- 文档：draft-ietf-lake-edhoc-12（2021 年 10 月）。
- 专家模型：Jacomme, Klein, Kremer, Racouchot, *A comprehensive, formal and automated analysis of the EDHOC protocol*, USENIX Security 2023 的 SAPIC+ 模型（charlie-j/edhoc-formal-analysis），由 tamarin-prover 1.7.1 导出成 ProVerif。每条引理一份 `.pv`，没有 `main.pv`。
- 标准答案：`subprotocols/handshake/dh/`。
