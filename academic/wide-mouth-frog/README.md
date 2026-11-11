# Wide-Mouth Frog

最简单的服务器中转密钥分发协议：A 生成会话密钥，带上时间戳经服务器转给 B。

- 文档：本库根据原论文（Burrows, Abadi & Needham, 1989）写成的 RFC 风格规范。
- 专家模型：由 SPORE 上 J. Dreier 的 Tamarin 模型 `Wide_Mouthed_Frog.spthy` 导出成 ProVerif，查询按 ProVerif 写法改写。
