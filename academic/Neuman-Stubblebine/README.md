# Neuman-Stubblebine

基于 nonce 的重复双向认证协议：四条消息完成首次认证，之后 A 凭票据 `{A, K_ab, T_b}K_kb` 用三条消息向 B 重新认证，不再找 KDC。

- 文档：本库根据原论文（Neuman & Stubblebine, ACM SIGOPS OSR 27(2), 1993）写成的 RFC 风格规范。
- 专家模型：来自 AutoSM 基准集的 SAPIC+ 模型导出的 ProVerif 文件。`reference_autosm.pv` 是原样导出（每条消息带标签）；`main.pv` 去掉了这些标签。
