# TLS 1.3 draft-18：0-RTT

TLS 1.3 草案 18 的 0-RTT 早期数据：客户端在握手完成前，就用预共享密钥（PSK）加密发送数据。

- 文档：draft-ietf-tls-tls13-18（2016 年 10 月）。
- 专家模型：从 reftls（Bhargavan, Blanchet, Kobeissi, IEEE S&P 2017）的 draft-18 模型中截出的 0-RTT 部分。
- 标准答案：`subprotocols/zero-rtt/`，有两个场景：PSK 保密、PSK 泄露。
