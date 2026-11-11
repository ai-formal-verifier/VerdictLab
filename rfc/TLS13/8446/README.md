# TLS 1.3：RFC 8446

TLS 1.3 正式标准的握手。

- 文档：RFC 8446（2018 年 8 月）。
- 专家模型：reftls（Inria-Prosecco/reftls）的 RFC 8446 模型。`Inria-Prosecco-reftls/pv/` 是上游原件，`main.pv` 是两份原件的拼接，`generated/` 是实际运行的文件。
- 标准答案：`subprotocols/handshake/reftls/`，对应模型实际覆盖的组合：PSK + (EC)DHE、服务器证书认证、带 0-RTT。RFC 本身不允许这个组合。
