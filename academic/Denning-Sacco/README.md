# Denning-Sacco（对称密钥版）

在 NSSK 的基础上加入时间戳的密钥分发协议。

- 文档：本库根据原论文（Denning & Sacco, CACM 24(8), 1981）写成的 RFC 风格规范。
- 专家模型：ProVerif 没有现成的对称密钥版模型，本库对照 SPORE 上 S. Delaune 的 Tamarin 模型写成。`main.pv` 是基础模型，`main-compromise-A/B.pv` 分别加入 A、B 一方的长期密钥泄露，`tamarin-aligned.pv` 与 Tamarin 模型逐条对齐。
