# VerdictLab

VerdictLab 是 VerdictBench 的课程子集，供课程项目 20、22 使用，收了 15 个协议。每个协议带三样东西：

- 协议文档：原件，以及转换好的 Markdown；
- 专家写的 ProVerif 模型；
- 标准答案：Orrery 各步骤用的 JSON。

每个协议回答的都是同一类问题：**协议 P 在假设 A 下，安全目标 G 成立吗？**

## 收录的协议

| 协议 | 标准答案所在目录 |
|---|---|
| `academic/NSPK` | 协议根目录 |
| `academic/Neuman-Stubblebine` | 协议根目录 |
| `academic/NSSK` | 协议根目录 |
| `academic/Denning-Sacco` | 协议根目录 |
| `academic/Otway-Rees` | 协议根目录 |
| `academic/wide-mouth-frog` | 协议根目录 |
| `academic/Yahalom` | 协议根目录 |
| `academic/Kao-Chow` | 协议根目录 |
| `rfc/OAuth-2.0/draft22` | `subprotocols/authorization-code/` |
| `rfc/TLS13/draft18` | `subprotocols/zero-rtt/` |
| `rfc/TLS13/draft20` | `subprotocols/handshake/reftls/` |
| `rfc/TLS13/8446` | `subprotocols/handshake/reftls/` |
| `rfc/EDHOC/draft-08` | `subprotocols/handshake/asymmetric/`、`symmetric/` |
| `rfc/EDHOC/draft-12` | `subprotocols/handshake/dh/` |
| `rfc/IKEV2` | `subprotocols/ike-sa/signature/` |

## 目录结构

```text
<来源>/<协议>/[<版本>/]
  protocol.yaml               协议名和子协议清单
  README.md                   协议是什么、文档和专家模型的来源
  original/main/              协议文档原件（txt、xml、html 或 pdf）
  converted/
    main/<主文档>.md            协议文档（images/ 是插图）
    refs/<文档>.md              [部分有] 该协议自带的引用文档
    document_set.json         文档集
  [subprotocols/<子协议>/[<选择>/]]     有子协议时，下面的标准答案放在这一层
    flow/
      principals.json         参与方
      values.json             值表
    claims/
      setup.json              验证设定
      expected.json           专家的问题和判定
    attacks/attacks.json      [仅 NSPK] 攻击标注的示例
    adjudication.json         [部分有] 专家模型和文档的偏差审计
    formal/proverif/main.pv   专家 ProVerif 模型（有的还有 scenarios/ 或几份辅助 .pv/.pvl）
public/converted/             多个协议共用的引用文档（RFC、NIST 等），每份只存一次
```

## 各文件的含义

| 文件 | 内容 | 对应 Orrery 的步骤 |
|---|---|---|
| `converted/document_set.json` | 主文档和它读到的引用文档（`library:public/...` 指向 `public/`），以及每节的标签、子协议划分、文献表里每条对到哪份文件 | preprocess |
| `flow/principals.json` | 参与方（`principals`），以及每个值在运行前由谁持有（`values`） | extract_principals |
| `flow/values.json` | 协议里每个值从哪来（`source.by`：`new` 新生成、`let` 派生、`const`/`free` 常量或公开名等）、谁持有（`holders`），附文档出处 | 建模 |
| `claims/setup.json` | 验证设定，分三部分：<br>• `assumptions`：假设空间，每个对象在每根轴上取一档，`default` 是默认档；<br>• `properties`：安全目标，按目标词表写；<br>• `claims`：文档声称哪个目标在什么条件下成立（`expect`），带原文引用 | extract_setup |
| `claims/expected.json` | 标准答案：<br>• `scenarios`：命名的场景，每个场景是一组假设档位，即“世界”；<br>• `goals`：安全目标；<br>• `questions`：每条问“目标 × 场景”，`verdict` 是专家模型跑出的判定，`evidence` 指向所用的 `.pv`；<br>• `unexpressed`：文档里说了、词表表达不了的结论 | 评估打分 |
| `attacks/attacks.json` | 攻击标注的格式示例，只有 NSPK 有：名称、概要、文献出处；`refutes` 列出它推翻的“目标 × 场景”格子，都是 `expected.json` 里判为 false 的 | attacks |
| `adjudication.json` | 专家模型哪里偏离了文档：矛盾、建模惯例、简化等 | 无（供人读） |
| `formal/proverif/*.pv` | 专家模型，`expected.json` 的判定都由它跑出来，有疑问时以它为准 | 无（对照基准） |

`names` 字段在 setup.json 和 expected.json 里都有，解释每个名字指什么。

几个词：

- **轴**：假设的一个维度，例如：
  - `data`：某个值攻击者能否拿到；
  - `channel`：某条链路能否被读写；
  - `admission`：攻击者能否注册成合法参与方；
  - `key_reuse`、`sessions` 等。

  档位从弱到强。
- **目标类**：`secret`（保密）、`correspondence`（认证/对应性）、`correlation`、`equivalence`、`unreachable`。
- 所有 JSON 都按词表版本 `vocab_version` 4.0.0 写。

## Orrery 的运行流程

Orrery 从协议文档出发，最后给出每个“目标 × 场景”的结论。下面的产物和本库的标准答案一一对照：

1. **convert**：原件转成 Markdown（对照 `converted/main/`）
2. **preprocess**：建文档集（对照 `document_set.json`）
3. **extract_principals**：提取参与方和各自运行前持有的值（对照 `principals.json`）
4. **建模**：写出协议诚实运行的 ProVerif 模型，并整理每个值的来历（对照 `values.json`）
5. **extract_setup**：读出假设空间、安全目标和文档声称（对照 `setup.json`）
6. **check_model**：把固定假设写进模型，得到根模型
7. **write_applier / write_injectors**：
   - 施加器把一组可变假设施加到根模型上，得到场景模型；
   - 注入器写入各目标的 ProVerif 查询。
8. **verify**：每个场景交给 ProVerif 求解，得到判定（对照 `expected.json` 的 `verdict`）
9. **cross_check / attacks**：
   - 判定和文档声称对照；
   - 把判为 false 的格子背后的攻击复现成逐步脚本。

模型链条：协议模型 → 根模型（加固定假设）→ 场景模型（加一组可变假设）→ 注入后的模型（加查询）→ ProVerif。

## 说明

- 每个协议带主文档的原件和转换好的 Markdown；引用文档只有 Markdown。IETF 文档可以自由转载。
