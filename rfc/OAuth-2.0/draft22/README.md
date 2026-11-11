# OAuth 2.0 draft-22

OAuth 2.0 的授权码模式：客户端把资源所有者重定向到授权服务器，拿回授权码，再在令牌端点换取访问令牌。

- 文档：draft-ietf-oauth-v2-22（2011 年 9 月）。
- 专家模型：开源项目 denadai2/formalVerificationOAuthProverif（学生项目，未发表）。`main.pv` 是授权码模式；另两份是上游的变体：不加密信道、合并两种令牌（未完成）。
- 标准答案：`subprotocols/authorization-code/`。
