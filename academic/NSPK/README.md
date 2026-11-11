# NSPK：Needham-Schroeder 公钥协议

Needham-Schroeder 公钥认证协议的七消息原始形式：双方先向认证服务器取对方公钥，再跑三条消息的 nonce 握手。

- 文档：本库根据原论文（Needham & Schroeder, CACM 21(12), 1978）和 SPORE 条目写成的 RFC 风格规范。
- 专家模型：ProVerif 2.05 自带的例子 `NeedhamSchroederPK.pv`（`formal/proverif/main.pv`）；`scenarios/admission_closed.pv` 去掉了攻击者注册公钥的进程，其余不变。
- 已知攻击：Lowe 中间人攻击（G. Lowe, IPL 1995）。
