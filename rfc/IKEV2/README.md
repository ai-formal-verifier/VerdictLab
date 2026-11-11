# IKEv2

IPsec 的密钥交换协议。本库收的是初始交换 IKE_SA_INIT 和 IKE_AUTH，采用签名认证。

- 文档：RFC 7296（2014 年 10 月）。
- 专家模型：NICT AKE Zoo 的 IKEv2 ProVerif 模型（`main.pv`），只建模了 IKE_SA_INIT 和 IKE_AUTH。`reference/` 是从原 PDF 里摘出的原文，`reachability.pv` 是去掉安全查询后的可达性检查。
- 标准答案：`subprotocols/ike-sa/signature/`。
