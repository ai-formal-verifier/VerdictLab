# TLS 1.3 draft-20：完整握手

TLS 1.3 草案 20 的完整握手：(EC)DHE 和 PSK 密钥交换、0-RTT 和 1-RTT 数据。

- 文档：draft-ietf-tls-tls13-20（2017 年 4 月）。
- 专家模型：reftls（Bhargavan, Blanchet, Kobeissi, IEEE S&P 2017）的 draft-20 模型。上游拆成 `tls-lib-draft20.pvl` 和 `tls13-draft20-only.pv` 两个文件，`main.pv` 是两者原样拼接。
- 标准答案：`subprotocols/handshake/reftls/`，有三个场景：凭据都安全、服务器私钥泄露、PSK 泄露。
