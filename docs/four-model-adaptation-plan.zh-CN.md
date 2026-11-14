# codex-team-mode 四模型适配报告与调整计划

日期：2026-11-14（Asia/Shanghai）  
上游：[oil-oil/codex-team-mode](https://github.com/oil-oil/codex-team-mode)  
个人 Fork：[yuyuwen84-hash/codex-team-mode](https://github.com/yuyuwen84-hash/codex-team-mode)  
分析基线：`8eab25b9961b06b7946a06a6432146291f4a5341`

## 1. 范围和结论

本文记录 Fork 的四模型调整方案与实施结果。按用户最新要求，**只根据模型能力调整模型选择和推理档位，保留上游 Explorer、Executor、Reviewer、ExpertAdvisor 的名称、职责和调度方向。** default 继续作为上游可选哨兵。本报告不建议新增、拆分、改名或重新定义角色，也不迁入本地另一套角色体系。

按用户确认的分工，推荐起点：**Explorer 用 Luna / medium；Executor 用 Terra / high；Reviewer 保持 Sol / high；ExpertAdvisor 继续按次选模，困难问题优先 Astra，推理档位按任务决定。** Executor 的 high 是待验证起点；优先控制日常实施成本，再依据验收、返工和总用量评估效果。ExpertAdvisor 的其他模型选择仍限定在原职责范围内，不为使用某模型而制造专家任务。

这是一份能力匹配方案，不是四模型性能实测结论。仓库模板、说明和相关测试已按方案修改；尚未安装到本地 Codex 环境，也未逐模型验证实际派发。

## 2. 可用模型与证据

已核对当前会话可选模型、本地模型目录和非秘密配置字段。本地 provider 是 `magpie`，配置默认值为 Sol / high；配置允许选择模型，不等于本轮已经逐一验证四条转发链路。

| 模型 | 本环境配置 ID | 本地目录声明支持的推理档位 | 能力定位与本计划的用途 |
| --- | --- | --- | --- |
| GPT-6 Astra | `auto/gpt-6-astra` | low、medium、high、xhigh、max、ultra | 官方定位为最困难的工作；用于 ExpertAdvisor 的困难决策、建模和反复未解问题。 |
| GPT-6.1 Sol | `auto/gpt-6.1-sol` | low、medium、high、xhigh、max、ultra | 官方定位为复杂工作；用于 Reviewer 深入核查，也可在 ExpertAdvisor 的原有专业任务中按需选择。 |
| GPT-6.1 Terra | `auto/gpt-6.1-terra` | none、low、medium、high、xhigh、max、ultra | 官方定位包括视觉推理、写作和范围清楚的代码修复；作为 Executor 默认模型，也可在原有 ExpertAdvisor 范围内按需选择。 |
| GPT-6 Luna | `auto/gpt-6-luna` | none、low、medium、high、xhigh、max | 官方定位为聚焦、批量任务；继续用于 Explorer 的明确代码定位。 |

能力依据是已实际读取的 [OpenAI GPT-6 使用指南](https://developers.openai.com/api/docs/guides/latest-model)、[官方选型指南](https://developers.openai.com/api/docs/guides/model-selection) 和 [官方子代理文档](https://learn.chatgpt.com/docs/agent-configuration/subagents)。选型指南明确将范围清楚的代码修复列为 Terra 的适用场景，并要求提供明确输入、要求和结果验证方式；子代理文档建议 Sol 处理复杂任务、Luna 处理范围窄的工作。这些定位支持以 Terra 承担边界清楚的实施、Sol 负责独立审查，再以真实结果验证分工。

这些官方定位不能证明 Magpie 的实际价格、延迟或路由结果；本报告不估算节省比例，也不声称 Terra 或 Astra 在所有任务上优于 Sol。

## 3. 逐角色调整

| 原角色 | 上游配置 | 建议配置 | 具体理由与职责保持 |
| --- | --- | --- | --- |
| Explorer | Luna / medium，模板请求 Fast | `auto/gpt-6-luna` / medium | 大型代码库中的文件、入口和调用路径定位通常有清楚边界，继续使用 Luna。保持只读代码定位，不扩成通用研究或跨模块排错角色。 |
| Executor | Luna / xhigh | `auto/gpt-6.1-terra` / high（待验证起点） | 任务由主代理明确目标与写入范围，Terra 适用于范围清楚的代码修复；结合测试与独立审查，以日常实施总成本和验收结果评估效果。保留围绕分配结果自行实施与验证的合同。 |
| Reviewer | Sol / high | `auto/gpt-6.1-sol` / high | 保持深度核查、独立上下文、稳定产物审查和必要时保存 Markdown 报告；不把实施和审查合成一次自检。 |
| ExpertAdvisor | 不固定模型与推理档位 | 继续不固定；按任务选择 Astra、Terra，必要时 Sol | 保留复杂决策、建模、复杂自动化、反复失败问题的原有职责，以及按次选模机制。 |
| default（可选） | Luna / low | `auto/gpt-6-luna` / low | 继续仅拦截漏传角色的派发；不增加业务职责，也不默认更换现有安装。 |

Explorer 的 Fast 设置本轮不作为能力调整项更改，速度档与模型能力分开判断；当前 provider 是否接受、是否实际生效，后续安装时依据宿主核对。本报告不承诺 Fast 可用或相应费用。

### Executor 为什么从 Luna / xhigh 改成 Terra / high

Executor 接收目标和写入范围明确的实施任务，主代理负责拆解与衔接，Reviewer 负责独立审查。这与官方列出的 Terra 适用条件相符：明确输入、范围清楚的代码修复，以及可执行的结果验证。Terra 具备工具调用、Shell 和 Apply Patch 支持；不能因为它突出视觉和写作能力，就推断其无法承担代码实施。

按用户的成本目标，将调用频繁的实施工作默认交给 Terra / high，将 Sol 集中用于独立审查。high 是待验证起点，没有实测证据证明 Terra 在所有工程任务上足够，也没有证据证明这些任务必须默认使用 Sol。最终比较的是完成并通过验收的总成本，包括重试、返工、审查开销和耗时；本报告不对 Magpie 的单价和节省比例作未经核实的推算。

复杂跨模块改动、隐含约束多或持续排错可能需要更强模型，但一次普通报错不足以改变默认配置。先让 Executor 根据测试和审查意见修复；确有困难决策或反复未解问题时，再按上游流程调用 ExpertAdvisor，困难问题优先 Astra。若代表性任务反复显示 Terra 不能满足验收，再单独评估 Executor 的模型配置，不增加角色，也不把实现任务转移给 Reviewer。

### ExpertAdvisor 如何使用 Astra 和 Terra

| 专家任务的主要难点 | 建议按次模型 | 起始推理档位 | 调整条件 |
| --- | --- | --- | --- |
| 复杂架构取舍、困难根因、约束冲突、反复失败后的独立分析 | `auto/gpt-6-astra` | xhigh | 只有实际证据说明仍需更深推理时再评估 max / ultra，不默认开启最高档。 |
| 以图像、截图、页面状态理解为主要难点的复杂自动化；原任务中确有专家价值的视觉表达或写作 | `auto/gpt-6.1-terra` | medium | 根据状态跟踪、工具操作和输出验收决定是否升 high；包含复杂业务逻辑时重新判断主难点。 |
| 工程工具执行与验证占主导的专业建模或复杂自动化 | `auto/gpt-6.1-sol` | high | 若 Executor 已能承担，不重复发起专家咨询；必要性由主代理判断。 |

ExpertAdvisor 不强制绑定 Astra，也不变成专门的视觉或文案角色。Terra 是其按次模型选项之一。若某任务需要“更强模型”的专家咨询，不能仅因切换到 Terra 就声称增强了所有推理能力，应说明视觉或写作能力与任务的匹配。

## 4. 模型配置和派发方式

固定角色同时写明实际 model ID 与 effort，避免继承到不适合的档位。ExpertAdvisor 继续省略这两个字段，派发时指定。下列示例已落实到仓库模板，尚未安装到本地运行环境：

```toml
# agents/Explorer.toml：仅列与本轮有关的字段
name = "Explorer"
model = "auto/gpt-6-luna"
model_reasoning_effort = "medium"
```

```toml
# agents/Executor.toml
name = "Executor"
model = "auto/gpt-6.1-terra"
model_reasoning_effort = "high"
```

```toml
# agents/Reviewer.toml
name = "Reviewer"
model = "auto/gpt-6.1-sol"
model_reasoning_effort = "high"
```

```text
困难决策：agent_type="ExpertAdvisor",
          model="auto/gpt-6-astra", reasoning_effort="xhigh",
          fork_turns="none"

视觉占主导的专家自动化：agent_type="ExpertAdvisor",
          model="auto/gpt-6.1-terra", reasoning_effort="medium",
          fork_turns="none"
```

官方文档说明，角色文件固定的 model / effort 优先于派发参数。所以不能临时给固定的 Executor 传 Astra，并声称已换模型；按次选模继续走未固定的 ExpertAdvisor。`auto/` 是本环境配置的 provider 路由标识，其他用户使用本 Fork 时需按实际模型目录配置。

主代理继续使用用户选定模型；本计划不自动切换它。保留上游按需分派、最少必要代理数量、独立审查、明确写入所有权和子代理不继续派生的方向。

## 5. 仓库修改清单

| 文件 | 已落实到仓库的改动 |
| --- | --- |
| `agents/Explorer.toml` | 模型 ID 对齐本环境，保留 Luna / medium 和原职责。 |
| `agents/Executor.toml` | 模型改为 Terra，effort 改为 high（待验证起点），保持原名称与职责正文。 |
| `agents/Reviewer.toml` | 模型 ID 对齐，保持 Sol / high 和原职责。 |
| `agents/ExpertAdvisor.toml` | 保持不固定模型，不添加新的固定职责或角色拆分。 |
| `agents/default.toml` | 仅对齐模型 ID，保持可选哨兵行为。 |
| `skills/team-mode/SKILL.md` | 同步 Executor 默认 Terra / high，并补充 ExpertAdvisor 的模型选择依据，保留原调度和职责方向。 |
| `skills/team-mode/references/custom-agents.md` | 更新模型与档位表，说明固定角色优先级、provider ID 及 ExpertAdvisor 的 Astra / Terra 选择。 |
| `skills/team-mode/references/evaluation.md` | 补充四模型适配的任务验收与实际运行记录要求，不引入新的调度门槛。 |
| `README.md`、`README.zh-CN.md`、`skills/team-mode/README.md` | 同步能力匹配表和适配来源，不改四角色结构。 |
| `tests/test_agent_profiles.py` | 更新模型 ID 和 Executor 档位断言，保留原有角色合同；检查 ExpertAdvisor 仍未固定模型。 |

模型选择说明采用一处主表，其他文档引用，避免多处角色配置逐渐不一致。不新增角色文件、不改角色大小写、不将本地角色覆盖进上游结构。本次也不把权限、安装体系、并发策略或费用脚本重构纳入能力调整方案。

## 6. 验证与后续计划

1. **静态检查（已完成）**：解析五份 TOML，核对 model ID 与支持的 effort；固定角色设置符合上表，ExpertAdvisor 继续不固定模型。与上游基线逐字段比较，角色名称、description、developer_instructions、权限字段、速度档和昵称均保持原样。
2. **已有回归（已完成）**：更新模型与档位断言后运行仓库测试，23 项中 22 项通过、1 项跳过、0 失败；Skill 格式校验通过。原有职责边界的测试保留。
3. **实际调用**：在新会话可识别这些角色后，按实际任务依次核对 Explorer / Luna / medium、Executor / Terra / high、Reviewer / Sol / high，以及 ExpertAdvisor 按次模型和档位；困难问题优先验证 Astra，其他选项只在有实际需要时验证。
4. **结果验收**：Explorer 给出可核对的主要入口；Executor 完成预期结果与相关验证；Reviewer 独立审查稳定产物；Astra 专家给出有依据的决策或建模结果；Terra 专家任务按实际图像、页面状态和保存后的结果验收。
5. **调整依据**：比较同类实际任务的通过率、重试、返工、审查开销、耗时和可归属 token，保留能满足验收的较低成本配置。有可信 provider 费率时再计算实际费用，用证据决定是否改 effort 或模型。四个模型无需同时运行，也不为填表重复执行真实工作。

当前会话总活跃代理上限是 4，包含主代理；后续验证遵守宿主限制。这是执行约束，不要求重写上游调度。角色是否可用、权限和速度档是否生效，最终以当前宿主工具合同与实际记录为准。

## 7. 本轮结果与限制

- 已在账号 `yuyuwen84-hash` 下完成真正的 Fork，页面显示 `forked from oil-oil/codex-team-mode`，创建时 main 与分析基线一致。
- 已读取上游说明、五个角色文件、Skill、配置与评估文档、诊断脚本及测试，核对四模型目录和当前会话选项。
- 已运行未修改上游基线测试和修改后的回归测试，两次均为：**23 项，22 项通过，1 项跳过，0 失败**。静态验证结果不代表四模型真实任务验收已通过。
- 已按用户确认的分工将报告更新为 Explorer / Luna / medium、Executor / Terra / high、Reviewer / Sol / high，ExpertAdvisor 保持按次选模且困难问题优先 Astra。结论、理由、配置示例、修改清单和验证计划均已同步；现已修改仓库角色模板、Skill、说明和相关测试，本地已安装配置未变。
- 未逐一调用四模型，未测试模型性能、转发连通性、真实价格或 Fast 生效情况。
- 已通过登录浏览器完成 Fork，并将角色配置、Skill、说明、测试与本报告上传至远程 [`codex/four-model-adaptation` 分支](https://github.com/yuyuwen84-hash/codex-team-mode/tree/codex/four-model-adaptation)。发布通过网页分目录提交完成；此前 GitHub 集成写入受限与 Git HTTPS 缺少非交互凭据的问题，已通过获准的浏览器上传方式完成交付。Fork 的 `main` 尚未合并此分支，查看或获取本次适配时应明确选择该分支。

另观察到：上游用量脚本以裸模型名查价，未覆盖本环境四个 `auto/` ID 以及当前 Terra 费率。现有 token 统计与费用估算应分别理解；未知费用不能当成零。此问题只记录为适配时的诊断限制，**不纳入本轮模型能力调整的实现清单**，也不根据未经核实的官方费率推算 Magpie 账单。

仓库修改、静态验证和远程分支发布已完成。后续仍需在获准安装后，以新会话进行真实任务验收；本报告不将仓库发布表述为已部署或四模型实测通过。

## 8. 来源

1. [上游中文说明（固定基线）](https://github.com/oil-oil/codex-team-mode/blob/8eab25b9961b06b7946a06a6432146291f4a5341/README.zh-CN.md)。
2. [上游角色模板](https://github.com/oil-oil/codex-team-mode/tree/8eab25b9961b06b7946a06a6432146291f4a5341/agents)。
3. [上游调度 Skill](https://github.com/oil-oil/codex-team-mode/blob/8eab25b9961b06b7946a06a6432146291f4a5341/skills/team-mode/SKILL.md)。
4. [OpenAI GPT-6 使用指南](https://developers.openai.com/api/docs/guides/latest-model)，2026-11-14 已实际读取；支持四模型的能力定位。
5. [OpenAI 官方子代理文档](https://learn.chatgpt.com/docs/agent-configuration/subagents)，2026-11-14 已实际读取；支持 Sol / Luna 的任务建议及模型配置优先级。
6. 当前环境模型目录、非秘密 Codex 配置字段及当前会话工具合同。本文未附个人完整配置、认证信息或原始会话日志。
7. [OpenAI 官方选型指南](https://developers.openai.com/api/docs/guides/model-selection) 与 [Terra 模型说明](https://developers.openai.com/api/docs/models/gpt-6.1-terra)，2026-11-14 已实际读取；分别支持范围清楚的代码修复定位与工具能力说明。
