---
title: 生产级Prompt自动化推理评估A/B实验结果的工程实践
source_url: https://mp.weixin.qq.com/s/y2Sq_KEFRlDEnFgDsEKxNQ
saved: 2026-05-09
tags: [ai, prompt, eval, experiment]
---
张超（码影） *2026年2月2日 08:31*

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naINctibxiaPHlPT8JRHoU0dfSWZH6j7fhxtBIfKxnSAkuggzNH8hMZVuwOGLYtjyXK4M9hmibdwnTicpA/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

一、前言

在互联网流量竞争白热化的时代，A/B实验已成为产品迭代的标准决策工具。当实验数量从数十增长到数百甚至数千数万时，传统的人工巡检模式遭遇瓶颈：需要专业的同学每日投入4-6小时逐个检视实验数据，判断其上线或下线；即使如此，由于时间压力和注意力限制，误判率依然居高不下。

上述问题在我的大模型策略调优项目中变得尤为突出。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

在策略调优试运行场景中，每日新增实验数量远超下线实验，导致在线策略数量持续膨胀——这直接影响了打包留白实验的整体效果评估，降低了快速迭代的能力。

传统解决方案存在三大局限：

1.人工巡检的低效性：依赖个人经验，容易疲劳决策，无法扩展到数百个实验；

2.规则引擎的僵化性：基于正则表达式的阈值判断，无法理解复杂的数据趋势和业务上下文；

3.统计方法的片面性：仅依赖p-value或绝对收益，忽视了小样本数据（1-7天）的波动特征；

针对这些痛点，我设计并部署了一套生产级Prompt自动化推理系统，基于大语言模型的推理能力，结合AB实验领域的专业知识，实现了高效、准确、可解释的实验评估流程。

| 卡片生命状态 | 图片展示 |
| --- | --- |
| 自动同步  实验数据  （开始状态） | ![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E) |
| 自动同步  实验数据  （完成状态） | ![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E) |
| 自动实验结果推理  （开始状态） | ![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E) |
| 自动实验结果推理  （进行状态） | ![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E) |
| 自动实验结果推理  （结束状态） | ![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E) |

试运行成果（运行1周）：

1.策略下线数从300+降至100+（下线准确率68%）

2.打包留白实验关键指标从每日负向，扭转为稳定正向趋势

3.人工巡检耗时从6小时/天降至30分钟/天

从留白实验的整体观测来看

端视角

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

行业视角

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

本文将详细介绍：

生产级Prompt的设计与优化，包括决策逻辑、真实bad cases分析，以及从68%迈向更高准确率的改进思路。

二、系统背景与业务痛点

**2.1 竞价策略的实验困境**

行业策略调优项目处于0-1建设阶段。与传统的灰度实验不同，这里的核心模式是：

大模型日均生产N人群策略 → 快速上线（无灰度） → T+2产出数据 → 人工巡检负向策略 → 快速下线

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**2.2 打包留白实验的困境**

单个策略的效果可能很小（如DAU变化0.1%-0.5%），但当多个策略合并时，其累积效应会更加明显。

这就是"打包留白实验"的价值：

- 将若干相同类型的策略合并成一个大实验；
- 用统一的对照组来评估这些策略的整体增益；
- 如果打包效果不好，则逐个下线策略进行诊断；

但现状是：

打包留白实验的关键指标（比如整体DAU相对变化率）每日都在负向或负向置信来回波动

这意味着整个打包的策略组合可能真的在伤害用户增长。

但单个策略看起来都还不错，矛盾之处在哪？

根本原因：包裹中混入了大量"负向策略"

策略可能的类型一般来说就这几个：

- 第1-2天看起来正向，但从第3天开始持续衰减；
- 早期有幸运的正向波动，但整体累积收益为负；
- 看起来有反弹，但反弹不足以抵消之前的损失；

这些"灰色地带"策略，人工判断时最容易出现分歧。

如果能准确识别并快速下线这些策略，打包留白的整体效果就会好转。

**2.3 现有方案的局限性**

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

大语言模型的优势：

相比以上方案，基于LLM的Prompt推理具有独特的能力：

- 理解上下文：可以同时考虑流量波动、绝对收益、趋势变化等多个维度；
- 灵活的决策路径：不是简单的AND/OR逻辑，而是能理解"虽然有一天下跌，但随后反弹了，所以不是趋势恶化"这样的复杂推理；
- 可解释的输出：每个决策都附带易懂的原因说明，便于产研同学二次确认和持续优化；
- 快速迭代：通过Prompt调整可以快速改进，无需重新训练模型。

三、主Prompt展示<适当脱敏版>

完整prompt

```ruby
## Profile：- Language: 中文- Description: 资深用户增长数据分析师，专注AB实验效果评估，精通流量归一化处理、多维度趋势建模及统计显著性解读，能基于1-7天实验数据输出客观、可落地的业务决策建议，严格遵循工程化输出规范## Skills:1. 精通AB实验核心指标分析（DAU相对变化率、绝对变化量、流量比例），熟练处理实验组/留白组数据差异2. 掌握科学流量归一化技术，当实验组与留白组流量比例差异>5%时，自动计算千人DAU变化量（dauAbsoluteChange/(experimentTrafficRatio/1000)）进行标准化3. 擅长多维度趋势诊断：识别持续负向、负向转正、正向衰减、高波动等模式，结合移动平均和标准差量化趋势稳定性4. 具备小样本决策能力（1-7天数据），综合绝对收益与相对变化进行业务价值评估，严格区分统计显著性与实际业务价值5. 严格生成无杂质JSON输出，确保字段类型精确（boolean/string），符合RFC8259标准及Java反序列化要求## Goals:1. 基于1-7天实验数据精准输出isRecommendOffline(boolean)，判断实验是否应下线200
JSON
4. 综合流量归一化、趋势稳定性、绝对收益阈值三维度，避免片面结论5. 识别数据风险信号（如波动异常、趋势逆转失败），提供可操作建议6. 历史实验天数小于3天时，表达"数据量不足，建议延长实验"## Constrains:JSON
2. isRecommendOffline字段值限定为true/false，禁止字符串表示布尔值3. 流量归一化强制规则：当|experimentTrafficRatio - controlTrafficRatio|/controlTrafficRatio > 5%时，必须使用千人DAU变化量进行决策4. 决策逻辑优先级（必须依序判断，触发即执行）：   **优先级0-数据不足**：   - 若天数 < 3 → isRecommendOffline=false，recommendation="数据量不足，建议延长实验"   **优先级1-连续负向（最严格）**：   - 条件：连续≥3天全部负向（dauRelativeChangePct ≤ 0%）且最后一天仍≤0%且最后2天无正向   - 触发 → isRecommendOffline=true   **优先级2-趋势恶化**：   - 条件：最后2天连续负向且最后一天绝对变化量<倒数第二天（如-20,-35）且不存在"倒数第二天负→最后一天正"反弹   - 触发 → isRecommendOffline=true   **优先级3-负向转正失败**：   - 条件：出现过负向后，最后2天中≥1天仍≤0%且无连续2天相对增长>0.3%   - 触发 → isRecommendOffline=true   **优先级4-正向衰减（禁止误触）**：   - 前置：连续≥3天正向   - 衰减检查：末日千人DAU < 历史峰值50%   - 排除场景：     * ✗ 累积绝对收益 > 0（Case3：连续正向但绝对值递减，总体+149人）3
   - 触发 → isRecommendOffline=true仅在"持续衰减且无总体收益"   **优先级5-高波动（需排除正向波动）**：15
   - 波动检查：最后2天趋势矛盾（一正一负）   - 收益检查：累积绝对变化量 ≤ 0   - 排除场景：     * ✗ 存在"负向→正向"单日反弹（Case2：-65→+48，视为正常调整）     * ✗ 最后2天都正向且绝对值递增     * ✗ 累积绝对收益 > 0（Case5：累计+316人）   - 触发 → isRecommendOffline=true   **优先级6-其他情况**：   - 除以上规则外 → isRecommendOffline=false5. isSignificant字段仅作辅助参考，禁止作为主要决策依据6. recommendation字段生成规范：   - 下线情况：明确说出触发原因+具体数据（如"最后2天连续减少，分别-20人和-35人，损失扩大"）   - 不下线情况：说出为什么不符合下线条件（如"虽然第3天-65人，但第4天+48人反弹，无法确认趋势恶化"或"4天累计+133人，波动不是下线理由"）   - 禁用词：千人DAU、标准差、相对变化率、效果不稳、可能失败等专业/主观术语   - 时间表述：用"最早/中间/最近"或日期，禁止"第n天"7. 输出必须可直接被Java JSON库解析，无转义字符或格式错误## Workflow:1. **数据验证**：确认输入为1-7个元素的JSON数组，每个元素包含dt/dauRelativeChangePct/dauAbsoluteChange/experimentTrafficRatio/controlTrafficRatio/isSignificant六个必填字段2. **数据预处理**：   - 解析dauRelativeChangePct（去除%转浮点）、dauAbsoluteChange（转整数）   - 计算流量差异率 = |experimentTrafficRatio - controlTrafficRatio| / controlTrafficRatio   - 需要归一化时：千人DAU = dauAbsoluteChange / (experimentTrafficRatio / 1000)3. **关键指标计算**：   - 累积绝对收益 = Σ dauAbsoluteChange   - 负向天数 = COUNT(dauRelativeChangePct ≤ 0%)   - 正向天数 = COUNT(dauRelativeChangePct > 0%)   - 最长连续负向段长度、最长连续正向段长度   - 最后2天趋势（是否存在反弹：负→正或下跌：正→负）   - 3天窗口相对变化标准差   - 绝对变化MAX/MIN及峰值4. **规则判断执行**（顺序如下）：----------IF 天数 < 3:RETURN false + "数据量不足，建议延长实验"IF 最长连续负向天数 ≥ 3 AND 最后一天 ≤ 0% AND 最后2天无正向:RETURN true + "连续X天用户减少，最后仍未好转，建议停止实验"IF 最后2天都≤0% AND |最后一天| > |倒数第二天| AND NOT(倒数第二天负→最后一天正):RETURN true + "最近两天用户持续减少，最后一天损失更大（具体数值），趋势恶化，建议停止"IF 出现过≤0% AND 最后2天≥1天≤0% AND NOT(连续2天>0.3%):RETURN true + "虽然出现过增长，但最近未能持续，最后X天中Y天用户减少，建议停止"IF 最长连续正向段≥3 AND 末日千人DAU < 历史峰值50%:IF 累积绝对收益 > 0 OR (连续正向≥2 AND 累积绝对值 > 单日最大负向×3):RETURN false + "连续X天增长，虽有衰减但累计增加Y人，总体向好，继续观察"ELSE:RETURN true + "连续增长但增幅持续下降，末日增长量不足历史峰值50%，效果衰减，建议停止"IF 标准差>15% AND 最后2天矛盾 AND 累积绝对收益≤0 AND NOT(单日反弹) AND NOT(最后2天都正向递增):RETURN true + "实验数据波动剧烈（最多增X人，最多减Y人），无稳定正向趋势，建议停止"DEFAULT:RETURN false + [根据情况生成observation]----------5. **Recommendation生成逻辑**（不下线case）：- 单日反弹case（如Case2）："{前日期}下跌X人，但{最近日期}反弹Y人，无法确认趋势恶化，建议继续观察"- 正向累积case（如Case5）："{总天数}天累计增加X人，虽有波动但整体正向，继续观察"- 存在波动但无理由case（如Case1,4）："{总天数}天内存在波动，无持续负向或衰减趋势，继续观察"## InputFormat:\`\`\`${input}\`\`\`- dt: 日期字符串 (e.g., "2025-12-08")- dauRelativeChangePct: 相对变化百分比字符串 (e.g., "-0.54%")- dauAbsoluteChange: 绝对变化整数值- experimentTrafficRatio: 实验组流量人数整数值- controlTrafficRatio: 留白组流量人数整数值- isSignificant: 置信判断字符串 ("Y"/"N")## OutputFormat:{"isRecommendOffline": boolean, "recommendation": string}## Examples:例1（连续负向-下线）：输入：[{"dt":"2025-12-01","dauRelativeChangePct":"-0.21%","dauAbsoluteChange":-5,...},{"dt":"2025-12-02","dauRelativeChangePct":"-0.33%","dauAbsoluteChange":-8,...},{"dt":"2025-12-03","dauRelativeChangePct":"-0.15%","dauAbsoluteChange":-4,...},{"dt":"2025-12-04","dauRelativeChangePct":"-0.08%","dauAbsoluteChange":-2,...},{"dt":"2025-12-05","dauRelativeChangePct":"-0.11%","dauAbsoluteChange":-3,...}]输出：{"isRecommendOffline":true,"recommendation":"5天中有4天用户减少，最后仍未好转，共损失22人，建议停止实验"}例2（单日反弹-继续观察）：输入：[{"dt":"2025-12-01","dauRelativeChangePct":"0.11%","dauAbsoluteChange":11,...},{"dt":"2025-12-02","dauRelativeChangePct":"0.02%","dauAbsoluteChange":2,...},{"dt":"2025-12-03","dauRelativeChangePct":"-0.65%","dauAbsoluteChange":-65,...},{"dt":"2025-12-04","dauRelativeChangePct":"0.48%","dauAbsoluteChange":48,...}]输出：{"isRecommendOffline":false,"recommendation":"虽然第3天下跌65人，但第4天反弹48人，无法确认趋势恶化，建议继续观察"}例3（正向累积-继续观察）：输入：[{"dt":"2025-12-01","dauRelativeChangePct":"-0.13%","dauAbsoluteChange":-13,...},{"dt":"2025-12-02","dauRelativeChangePct":"1.02%","dauAbsoluteChange":102,...},{"dt":"2025-12-03","dauRelativeChangePct":"0.35%","dauAbsoluteChange":35,...},{"dt":"2025-12-04","dauRelativeChangePct":"0.12%","dauAbsoluteChange":12,...}]输出：{"isRecommendOffline":false,"recommendation":"4天中最后3天连续增长，累计增加136人（第2天102人贡献最大），整体向好，建议继续观察"}例4（波动无理由-继续观察）：输入：[{"dt":"2025-12-01","dauRelativeChangePct":"0.33%","dauAbsoluteChange":33,...},{"dt":"2025-12-02","dauRelativeChangePct":"-0.67%","dauAbsoluteChange":-67,...},{"dt":"2025-12-03","dauRelativeChangePct":"0.35%","dauAbsoluteChange":35,...},{"dt":"2025-12-04","dauRelativeChangePct":"0.34%","dauAbsoluteChange":34,...}]输出：{"isRecommendOffline":false,"recommendation":"4天内存在波动（最多增33人，最多减67人），但无持续下跌趋势，整体变化不大，建议继续观察"}例5（大幅正向波动-继续观察）：输入：[{"dt":"2025-12-01","dauRelativeChangePct":"0.77%","dauAbsoluteChange":77,...},{"dt":"2025-12-02","dauRelativeChangePct":"0.81%","dauAbsoluteChange":81,...},{"dt":"2025-12-03","dauRelativeChangePct":"-0.47%","dauAbsoluteChange":-47,...}]输出：{"isRecommendOffline":false,"recommendation":"xxxx,大幅正向波动, 建议继续观察"}
```

四、六层优先级决策树

在设计Prompt 早期，犯了个错误：把所有决策规则一股脑堆上去。

结果很快就崩了——当一个实验同时满足“连续负向”和“正向衰减”两个特征时，系统根本不知道该听谁的，输出前后打架，甚至自相矛盾。比较麻烦的是，有些规则本身就有逻辑冲突。比如这条：“高波动且无正向趋势 → 下线”。如果不加任何限制条件，它会把“先负向、后反弹”这种正常的策略调整过程，也当成高风险波动直接干掉。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

痛定思痛，引入了优先级分层机制。思路来自医学诊断里的“三角验证”和安全领域的“纵深防御”：不是所有规则平权，而是按风险严重程度分层决策。最终我把整个判断逻辑拆成 6 个优先级（0–6），每一层对应一类典型风险场景。优先级越高，触发条件越严，下线决策的置信度也越高。

**4.1 关键设计**

决策分层6个层级，大的设计原则如下：

- P0–P3：高危信号，一旦命中，直接下线，不等验证；
- P4–P5：中等风险，触发前必须通过若干排除规则（exclusion rules）过滤干扰；
- P6：兜底逻辑，前面都不匹配时才走这条路。

这套分层之后，规则不再打架，大模型的输出行为相对稳定，基本满足自动化决策的准确率需求。

### 提示词节选

prompt规则节选

```markdown
4. 决策逻辑优先级（必须依序判断，触发即执行）：
**优先级0-数据不足**：- 若天数 < 3 → isRecommendOffline=false，recommendation="数据量不足，建议延长实验"
**优先级1-连续负向（最严格）**：- 条件：连续≥3天全部负向（dauRelativeChangePct ≤ 0%）且最后一天仍≤0%且最后2天无正向- 触发 → isRecommendOffline=true
**优先级2-趋势恶化**：- 条件：最后2天连续负向且最后一天绝对变化量<倒数第二天（如-20,-35）且不存在"倒数第二天负→最后一天正"反弹- 触发 → isRecommendOffline=true
**优先级3-负向转正失败**：- 条件：出现过负向后，最后2天中≥1天仍≤0%且无连续2天相对增长>0.3%- 触发 → isRecommendOffline=true
**优先级4-正向衰减（禁止误触）**：- 前置：连续≥3天正向- 衰减检查：末日千人DAU < 历史峰值50%- 排除场景： * ✗ 累积绝对收益 > 0（Case3：连续正向但绝对值递减，总体+149人） * ✗ 连续正向天数≥2且累积绝对值 > 单日最大负向绝对值的3倍- 触发 → isRecommendOffline=true仅在"持续衰减且无总体收益"
**优先级5-高波动（需排除正向波动）**：- 前置：3天内相对变化标准差 > 15%- 波动检查：最后2天趋势矛盾（一正一负）- 收益检查：累积绝对变化量 ≤ 0- 排除场景： * ✗ 存在"负向→正向"单日反弹（Case2：-65→+48，视为正常调整） * ✗ 最后2天都正向且绝对值递增 * ✗ 累积绝对收益 > 0（Case5：累计+316人）- 触发 → isRecommendOffline=true
**优先级6-其他情况**：- 除以上规则外 → isRecommendOffline=false
```

决策树图例说明

```objectivec
输入：7日内的实验数据         │         ↓    ┌────────────────────────────┐    │ 优先级0：数据验证           │    │ 天数 < 3?                  │    └────┬───────────────────────┘         │    YES  │  NO    ↓    │    ↓  [HOLD] │    ┌────────────────────────────┐         │    │ 优先级1：连续负向（严格）  │0
         │    │ 最后一天≤0%?              │         │    │ 最后2天无正向?            │         │    └────┬──────────────────────┘         │         │         │    YES  │  NO         │    ↓    │    ↓         │  [DOWN] │    ┌──────────────────────────┐         │         │    │ 优先级2：趋势恶化        │0
         │         │    │ |末日| > |倒数第二天|?  │         │         │    │ 无反弹信号?             │         │         │    └────┬─────────────────────┘         │         │         │         │         │    YES  │  NO         │         │    ↓    │    ↓         │         │  [DOWN] │    ┌────────────────────────┐         │         │         │    │ 优先级3：负向转正失败  │         │         │         │    │ 历史有≤0%?            │         │         │         │    │ 最后2天≥1天≤0%?       │         │         │         │    │ 反弹太弱(<0.3%)?      │         │         │         │    └────┬─────────────────────┘         │         │         │         │         │         │         │    YES  │  NO         │         │         │    ↓    │    ↓         │         │         │  [DOWN] │    ┌───────────────────────────┐         │         │         │         │    │ 全局收益检查              │         │         │         │         │    │ 累积收益 > 0?            │         │         │         │         │    └────┬──────────────────────┘         │         │         │         │         │         │         │         │         │    YES  │  NO         │         │         │         │    ↓    │    ↓         │         │         │         │  [HOLD] │    ┌──────────────────────────┐         │         │         │         │         │    │ 优先级4：正向衰减        │         │         │         │         │         │    │ 连续≥3天正向?           │         │         │         │         │         │    │ 末日 < 峰值50%?         │         │         │         │         │         │    └────┬─────────────────────┘         │         │         │         │         │         │         │         │         │         │         │    YES  │  NO         │         │         │         │         │    ↓    │    ↓         │         │         │         │         │  [DOWN] │    ┌──────────────────────┐         │         │         │         │         │         │    │ 优先级5：高波动      │         │         │         │         │         │         │    │ 标准差 > 15%?       │         │         │         │         │         │         │    │ 最后2天矛盾?        │         │         │         │         │         │         │    │ 累积≤0 & 无反弹?    │         │         │         │         │         │         │    └────┬────────────────┘         │         │         │         │         │         │         │         │         │         │         │         │         │    YES  │  NO         │         │         │         │         │         │    ↓    │    ↓         │         │         │         │         │         │  [DOWN] │  [HOLD]         │         │         │         │         │         │         └─────────┴─────────┴─────────┴─────────┴─────────┴─────────┘                              ↓                    ┌────────────────────┐                    │ 输出JSON结果       │                    │ isRecommendOffline │                    │ recommendation     │                    └────────────────────┘
```

上面决策树展如下关系：

- 纵向的优先级关系：优先级0最高，往下递减；
- 横向的分支逻辑：每一级都有YES/NO的分支；
- 短路评估：一旦某个优先级的条件满足，就输出\[DOWN\]，不再继续向下评估；
- 全局检查的插入点：在优先级3之后，进行一次全局收益检查，可以"救"某些case；

### 这么复杂的提示词用什么模型？

deepseek-r1-0528、qwen3-max

#### 为什么deepseek-r1？

1.推理能力强：能够处理多步骤的逻辑判断，不容易在中间步骤出错；

2.指令遵循性好：对Prompt中的细节要求（如伪代码定义、排除条件）执行得很到位；

3.输出格式稳定：生成的JSON格式错误率极低，便于下游系统解析；

4.公司环境部署友好。

deepseek-r1的推理流程：

```javascript
用户输入Prompt + JSON数据         │         ↓┌─────────────────────────────────┐│ 模型的"思考"阶段（内部推理）    ││ （这部分用户看不到）           ││                                 ││ 1. 理解Profile和Constraints     ││    → 识别为"AB实验评估任务"    ││                                 ││ 2. 执行Workflow第2步            ││    → 解析JSON，排序，验证      ││    → 生成"数据预处理报告"      ││    （中间状态，不输出）         ││                                 ││ 3. 执行Workflow第3步            ││    → 逐个计算关键指标           ││    → 最长连续负向段 = ?         ││    → 累积绝对收益 = ?           ││    → 标准差 = ?                 ││                                 ││ 4. 执行Workflow第4步            ││    → 按优先级顺序判断           ││    → 优先级0: 数据足吗? → 否   ││    → 优先级1: 连续负向? → 否   ││    → 优先级2: 趋势恶化? → 是!  ││    → 触发下线规则               ││                                 ││ 5. 执行Workflow第5步            ││    → 根据下线原因，生成文本说明 ││    → "最近两天..."（具体数据） ││                                 │└─────────────────────────────────┘         │         ↓┌─────────────────────────────────┐│ 模型输出阶段                    ││ （用户可以看到）               ││                                 ││ 生成最终JSON：                 ││ {                               ││   "isRecommendOffline": true,  ││   "recommendation": "..."       ││ }                               ││                                 │└─────────────────────────────────┘
```

**4.2 每层详细说明**

### L0：数据不足的保守决策

```java
IF 实验天数 < 3:  RETURN isRecommendOffline = false  recommendation = "数据量不足，建议延长实验"
```

为什么这是优先级最高的检查？

在AB实验领域，有个经验法则叫做"最少检测样本量"（Minimum Detectable Effect）。

即使你的实验设计再科学，如果运行时间太短，样本量就不足以达到统计显著性。更重要的是，少于3天的数据极容易被单日噪声污染：比如某地网络抖动、服务端瞬时故障、或是某个 Push 消息撞上了突发公共事件。

我有时候急着下线表现差的策略，但在实际 Push 场景里，过早下线的代价，远高于多等一天的成本。宁可保守一点，也要看到完整的3天数据周期。

举个真实例子：有个民生服务推送策略，第1天 DAU 变化是 -2.5%，看起来尚可；第2天突然跌到 -8%，如果当时自动化系统只看前两天数据，很可能直接触发下线。但实际上，第3到第7天数据逐步回升至 -1.5%，后来几天该策略最终稳定在 +2% 的正向收益。

要是第2天就干掉了它，我们就错杀了一个优质策略。

L1：连续负向—最严格的下线信号

```apache
IF 最长连续负向天数 ≥ 3   AND 最后一天 ≤ 0%    AND 最后2天中无正向日:  RETURN isRecommendOffline = true  recommendation = "连续X天用户减少，最后仍未好转，共损失Y人，建议停止实验"
```

逻辑解释：捕捉的是最坏的情况：实验组的DAU相对于对照组持续下滑，没有任何反弹迹象：

1.不是随机波动：如果只是一两天的波动，我们可以归咎于噪声。但连续3天都是负向，统计上几乎不可能是巧合（在均匀分布假设下，这个概率是12.5%）

2.趋势没有逆转：最后两天仍然没有正向，说明问题在持续恶化，而不是即将反弹

3.业务价值明确：我们累积了实际的用户损失（绝对值），持续运行这个策略只会继续亏损

为什么要求"最后2天无正向"而不是"最后一天无正向"？

关键细节：原始规则只要求"最后一天≤0%"，但在实际运行中我发现，即使连续3天都负向，只要最后一天有轻微反弹（比如-0.1%→-0.05%）就倾向于继续观察，因为我认为"趋势正在改善"。所以条件需要设置为：不仅最后一天要≤0%，而且最后2天都不能有正向。这样就排除了"虽然还是负向但在快速好转"的case。

L2：趋势恶化—加速度负向

```cpp
0
   AND |最后一天的绝对变化量| > |倒数第二天的绝对变化量|   AND NOT(倒数第二天负 → 最后一天正的反弹):  RETURN isRecommendOffline = true  recommendation = "最近两天用户持续减少，最后一天损失更大（-Xperson → -Yperson），趋势恶化，建议停止"
```

这条规则的真正价值，不在于抓“一直亏”，而在于抓“越亏越快”。优先级1原本设计用来识别“平缓负向”—比如每天稳定掉几十个 DAU。

但实践中我发现，更危险的是亏损在加速。举个例子：第6天：-20 人、第7天：-35 人

看起来都是负向，但趋势明显在恶化。这种情况下，每多跑一天，损失不是线性增加，而是加速扩大。但光看加速还不够，必须排除“假恶化”。所以我加了一个关键排除条件：NOT（倒数第二天负 → 最后一天正）。

为什么？因为在复盘 bad case 时反复看到一种模式：策略前六天表现疲软甚至微负，但第七天突然反弹。

这往往是因为 Push 触达延迟、用户行为滞后，或是外部事件扰动后的自然修复。如果这时候把它当成“加速恶化”干掉，就又错杀了。

最后说个细节：这里我用的是绝对变化量（损失多少人），而不是相对变化率（百分比）。原因很简单：在民生 Push 这类低频、高波动场景下，实验组某天流量可能因为调度或城市覆盖问题突然变少。这时哪怕只少了几百曝光，相对变化率也可能飙到 -30%、-50%，但实际用户损失可能就十几二十人。我看重的是真实用户的流失数量，不是被流量噪音放大的百分比。

L3：负向转正失败—虚假反弹的陷阱

```apache
IF 历史上出现过 ≤0% 的日期   AND 最后2天中 ≥1天 ≤0%   AND NOT(存在连续2天相对增长都 > 0.3%):  RETURN isRecommendOffline = true  recommendation = "虽然出现过增长，但最近未能持续，最后X天中Y天用户减少，建议停止"
```

这条规则源于一个反复踩坑的业务现象：有些策略上线头一天表现很差（比如 DAU -0.5%），但第2-3天数据看起来“好转”了（+0.2%、+0.3%），我个人会觉得策略有种黑盒的“自我修复”情况= =。但再往后看，第4-5天又掉回负向，甚至比最初还差。结果说明了这不是反弹，只是短期波动被误读成了趋势反转。

所以我在规则里卡死了一个条件：必须连续两天相对增长 > 0.3%。这不是拍脑袋定的。在我回溯之前的实验数据后发现：只要满足这个阈值，80% 的案例后续都能稳住正向；而那些“微弱反弹”（比如 -0.5% → +0.1% → -0.2%）几乎全部打回原形。

这也和优先级1形成明确区分：

- P1（连续负向）：从第一天起就一路向下，毫无起色 → 策略方向可能根本错了；
- P3（负向转正失败）：中间有过反弹尝试，但没撑住 → 调整方向对了，但力度不够，或者问题比预想复杂。

两者的处置逻辑也不同：

‒P1 直接下线，别浪费资源；

‒P3 值得复盘—有空需要研究的case

说到底，不是所有好转都值得相信，要看它能不能站稳。

L4：正向衰减—增益消失的信号

```apache
IF 最长连续正向天数 ≥ 3   AND 末日千人DAU < 历史峰值的50%   AND (累积绝对收益 ≤ 0 OR (连续正向 ≥2 AND 累积绝对值 ≤ 单日最大负向×3)):  RETURN isRecommendOffline = true  recommendation = "连续增长但增幅持续下降，末日增长量不足历史峰值50%，效果衰减，建议停止"
```

这是最容易触发false positive的规则，也是我在bad case中碰到最多的。

规则的核心逻辑：

一个策略早期表现很好（连续3天+2%, +1.8%, +1.5%），但到了后期，增长在快速衰减（+0.3%, +0.1%, -0.2%）。虽然技术上还有正向日期，但增长势能在丧失，这种"衰减"趋势本身就是一个下线信号。

"末日千人DAU < 历史峰值50%"是什么意思？

这需要结合流量归一化来理解。假设某策略的历史峰值是：

- 第2天：+102人（千人DAU：102 ÷ (实验流量比/1000) = 102 ÷ 0.1 = 1020人）
- 第7天：+8人（千人DAU：8 ÷ 0.1 = 80人）

那么末日千人DAU只有80，远低于峰值1020的50%（510人）。这说明策略的效能在快速衰退。

三个排除条件（Exclusion Rules）—非常重要：

这个规则之所以复杂，是因为在实际测试中发现了太多的false positive案例。

一个初看像"衰减"的策略，可能其实是：

1.累积收益为正：虽然最后几天增长微弱，但前几天赚的足够多，整体还是赚

- 例：\[+102, +35, +12, +5, -3\] = 累计+151人
- 即使末日衰减，这个策略整体价值是正的，不应该下线

2.持续多日正向且单日收益不小：虽然有衰减，但持续时间长，总收益足够大

- 例：\[+50, +45, +40, +30, +20\] 虽然衰减明显，但5天累计+185人，远超单日最大负向的3倍
- 这样的策略值得继续观察

3.短期波动而非趋势衰减：最后两天中如果有一天突然反弹，可能不是衰减而是波动

- 需要加额外检查：最后2天都正向且递增

这个规则后面我需要重新看待

因为"衰减"本质上是一个趋势判断而非绝对判断。两次看同样的数据可能得出不同的结论。

在一周的人工检查bad case库中，有30%的分歧都来自于这个规则。这也是为什么后续要设计"二次判断提示词"来专门处理这类边界case。

### L5：高波动无趋势—噪声策略的识别

```apache
IF 3天滑窗内相对变化标准差 > 15%   AND 最后2天趋势矛盾（一正一负）   AND 累积绝对变化量 ≤ 0   AND NOT(存在"负向→正向"单日反弹)   AND NOT(最后2天都正向且绝对值递增)   AND NOT(累积绝对收益 > 0):  RETURN isRecommendOffline = true  recommendation = "实验数据波动剧烈（最多增X人，最多减Y人），无稳定正向趋势，建议停止"
```

一点点的创新：引入波动率（标准差）这个指标

前面的规则都是基于趋势（是否连续、是否衰减）或绝对收益。

但有一类策略最难判断：数据非常嘈杂，忽正忽负，没有明确的方向。

例如：

- 第1天：+0.5%（+50人）
- 第2天：-1.2%（-120人）
- 第3天：+0.8%（+80人）
- 第4天：-0.3%（-30人）

这类策略的问题不在于"一定在亏"，而在于无法预测。

如果上线这样的策略，永远不知道明天会是+还是-。对于平台的稳定性，这种不可控的波动比一个稳定的-0.1%还要危险。

标准差 > 15% 的含义：这是一个经验阈值。在我们的数据集中，如果相对变化率的标准差超过15%，就说明这个策略的表现不稳定。

四个排除条件：

1.存在"负向→正向"单日反弹

- 例：第1天-65%, 第2天+48%
- 这不是"无趋势"，而是正常的数据调整，我们应该继续观察

2.最后2天都正向且递增

- 例：第6天+0.5%, 第7天+0.8%
- 这说明趋势正在稳定为正向，是好信号

3.累积绝对收益 > 0

- 例如虽然波动，但总体赚了100人
- 波动不是下线理由

4.最后2天趋势一致

- 例：最后2天都正向或都负向
- 这说明有明确的趋势，不算"无趋势波动"

为什么这个规则最容易跟优先级1、2冲突？

因为所有的"连续负向"策略，从另一个角度看，也都是"有波动"的。

所以优先级顺序很关键——先判断是否触发优先级1-3，如果都没触发，才进入优先级5来判断波动。

这保证了"清晰的负向趋势"优先级高于"无趋势的波动"。

L6：兜底规则—继续观察

```sql
DEFAULT:  RETURN isRecommendOffline = false  recommendation = [根据实际情况的observation]
```

这包括：

- 存在正向日期且无明确负向趋势
- 单日波动但整体正向
- 数据不足但趋势向好
- 其他不确定的情况

在这种情况下，recommendation应该明确说出为什么不符合下线条件，而不仅仅说"继续观察"。

五、典型Bad Cases与Prompt优化

**5.1 从Bad Case到Prompt改进的闭环**

分析完3个代表性bad case后，我建立了一套系统的优化闭环，用来指导后续的30+个case的处理。

这个方法论本身也可能对其他做Prompt工程的团队有借鉴意义。

闭环的四个环节：

```sql
Bad Case采集    ↓根本原因分析 → 识别是"模型能力"问题还是"Prompt设计"问题    ↓针对性改进 → 修订Prompt逻辑、添加排除条件、或优化伪代码定义    ↓回归测试 → 在全量bad case集上验证修改不引入新的误判    ↓准确率提升验证 → 用历史全量数据集（已标注）重新运行，确保整体准确率有提升
```

**5.2 Bad Case的意义：从68%到80%的关键**

在实验结果自动巡检的第一周，收集了30+个bad cases。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

这些都是系统判断与人工判断不一致的案例。初看起来，这些case似乎都是"模型理解错误"～

但深入分析后发现：大部分bad case实际上反映的是Prompt的逻辑漏洞，而非模型能力不足。

这次的发现改变了我对"准确率68%"的理解：这不是模型的问题，而是我对业务场景的建模还不够全面。

接下来分析的三个bad cases，都比较典型，都可以引出一个Prompt优化的方向。

**5.3 Bad Cases分析**

### Case 1：日期未排序导致的逻辑混乱

原始输入数据（简化后）

```apache
1207: -67人（-0.67%）1208: -35人（-0.35%）1209: +34人（+0.34%）1210: +45人（+0.45%）
```

系统错误输出：

```javascript
"最近两天用户持续减少（最后一天比前一天多损失15人），趋势恶化，建议停止"isRecommendOffline: true
```

人工正确判断：

```js
数据明显是稳步上升的趋势。前两天是损失，但后两天已经反弹，且反弹幅度（+34, +45）已经弥补了早期的损失。不应该下线。
```

问题根源分析：

这是一个灾难性的错误——系统把最后两天的+34和+45理解成了负向。追溯根源，有两个可能：

1.输入解析错误：系统在处理JSON时，可能没有严格按照日期顺序（dt字段）来排序数据，而是按照数组顺序处理

2.符号理解错误：系统混淆了正负号，把"-35"理解成了绝对值35，而在某个计算环节弄反了符号

从Prompt的角度看，我们在第一版本中缺少了"数据预处理与验证"的显式步骤。

Prompt优化方向：

在Workflow中加入了明确的数据预处理步骤：

```bash
## 优化后的Workflow第2步（数据预处理）：
2. **数据预处理与验证**：   - 步骤2.1：按dt字段升序排序所有记录   - 步骤2.2：逐条解析dauRelativeChangePct（移除%符号转为浮点数）   - 步骤2.3：验证dauAbsoluteChange的符号与dauRelativeChangePct一致     * 若不一致，输出警告："数据校验异常，请检查输入"   - 步骤2.4：确认experimentTrafficRatio/controlTrafficRatio均为正整数   - 步骤2.5：计算流量差异率，如>5%则标记需要流量归一化
```

对LLM的具体要求：

在Prompt的Constraints部分加入了：

```bash
## 额外的验证Constraint：
8. **强制数据校验**：   - 在任何计算前，必须先按日期排序并验证符号一致性   - 生成一个"数据验证报告"作为中间步骤（不输出给用户），包含：     * 排序后的日期序列     * 每日的相对变化和绝对变化（明确标出+/- 符号）     * 是否需要流量归一化的判断结果   - 仅在验证通过后，再进入规则判断阶段
```

这个优化为什么有效？

因为显式地要求模型在每个关键步骤输出"中间验证结果"。

虽然最终输出仍然只是JSON（不包含中间步骤），但在推理过程中，这个验证步骤会让deepseek-r1的推理引擎多次检查数据的正确性。在实际测试中，这个优化将这类"日期混乱"导致的错误明显降低，具体频率需要后面加上归因逻辑再给出。

Case 2：符号误判与连续性判断的混淆

原始输入数据（简化后）

```apache
1207: +19人（+0.19%）1208: -26人（-0.26%）1209: -19人（-0.19%）1210: +19人（+0.19%）
```

系统错误输出：

```bash
"连续3天用户减少（最后三天分别损失19人、7人、19人），且最后一天仍在减少，建议停止实验"isRecommendOffline: true
```

人工正确判断：

```js
数据序列是：+ - - +没有连续3天都是负向的。第2-3天虽然是负向，但只有2天。第4天已经反弹到正向。不应该下线。
```

问题根源分析：

这是一个符号识别错误。大模型说的"最后三天分别损失19人、7人、19人"中：

- "19人"对应1209的-19（理解对了）
- "7人"对应什么？1208是-26，不是-7。可能系统计算了|-26| - |-19| = 7？
- "19人"对应1210的+19？系统把正号理解成了负号？

这个错误非常离谱，但它反映了一个Prompt设计的漏洞：在规则判断中引用"最长连续负向天数"这个指标，但没有显式定义"连续"的含义。

具体来说：

- 什么叫"连续"？是数组中相邻的多个日期都满足条件？还是逻辑上相邻？
- 在计算"连续负向段"时，如果碰到了正向日期，是否应该重置计数器？
- "连续3天"是指恰好3天，还是至少3天？

Prompt优化方向：

在关键指标计算部分加入了伪代码定义：

```makefile
## 优化后的关键指标计算（第3步）：
3. **关键指标计算**（新增详细伪代码）：
3.1 **连续负向段识别**：max_consecutive_negative = 0current_consecutive = 0for each day in sorted_days:if day.dauRelativeChangePct <= 0%:current_consecutive += 1max_consecutive_negative = max(max_consecutive_negative, current_consecutive)else: # 遇到正向日期，重置计数器current_consecutive = 0
3.2 **最后2天的趋势判断**（新增明确定义）：last_day_value = sorted_days[-1].dauRelativeChangePctsecond_last_value = sorted_days[-2].dauRelativeChangePctlast_2_days_both_negative = (last_day_value <= 0%) AND (second_last_value <= 0%)last_2_days_no_positive = NOT(last_day_value > 0% OR second_last_value > 0%)注意：>= 0% 的定义是相对变化率 <= 0%，不包括恰好0%的边界情况
3.3 **累积绝对收益**：cumulative_absolute_change = SUM(all days' dauAbsoluteChange)严格的数值加法，保留符号
3.4 **标准差计算**（用于优先级5）：if total_days >= 3:relative_changes = [day.dauRelativeChangePct for each day]std_dev = STDEV(relative_changes) # 使用样本标准差is_high_volatility = (std_dev > 15%)else:is_high_volatility = false # 数据不足，无法判断波动
```

为什么这个优化这么重要？

deepseek-r1是推理模型，它在处理不明确的指标定义时，会倾向于做自己的"推理"而不是严格遵循Prompt。

通过用伪代码显式定义每个指标，可以消除了模型的"创意空间"，让它按照我的逻辑步骤走。

在优化后的测试中，这类"连续性误判"问题的case也变少了。

Case 3：累积收益与短期波动的权衡

原始输入数据（简化后）

```apache
1207: +4人（+0.04%）1208: +20人（+0.20%）1209: -45人（-0.45%）1210: +28人（+0.28%）
```

系统错误输出：

```javascript
"虽然出现过增长，但最近两天中有1天用户减少45人，建议停止实验"isRecommendOffline: true
```

人工正确判断：

```js
虽然第3天-45是个大坑，但第4天反弹到+28。而且看整体：4天累计 4+20-45+28 = +7人。虽然波动大，但整体还是赚的。而且最近的趋势（第4天）是正向的。不应该下线。
```

问题根源分析：

这个错误很有意思。大模型的输出逻辑似乎是在执行优先级3的规则（"负向转正失败"），但判断条件设定得太宽松了。

系统的推理过程大概是：

1.发现历史上有正向日期（第1、2天）✓

2.发现最后2天中有负向（第3天的-45）✓

3.判断"反弹失败"→下线 ✗

但系统遗漏了优先级3中的一个关键排除条件：

```apache
AND NOT(存在连续2天相对增长都 > 0.3%)
```

在这个case中，第2天是+0.20%，第3-4天是-0.45%和+0.28%。

最后两天并没有"连续2天都 > 0.3%"的反弹。但这不意味着应该下线！

更重要的是，系统应该综合考虑：

- 第3天虽然-45，但这可能是一个"极端事件"（比如某个小时的bug）
- 第4天立刻反弹了+28，说明系统在自我纠正
- 累积来看，4天总共赚了7人，即使波动大，也是正的

这个case引出了一个本质的业务设计问题：优先级规则是独立的吗？还是应该有一个全局的"综合权衡"步骤？

Prompt优化方向：

在规则判断前面加入了一个"全局收益快速检查"的环节：

```apache
## 优化后的规则判断执行（第4步前的新增步骤）：
3.9 **全局收益检查**（优先级0.5）：   IF 累积绝对收益 > 0:     # 即使后续触发某些规则，也要加额外的排除条件     set_flag: GLOBAL_POSITIVE = true   ELSE IF 累积绝对收益 < 0 AND |累积绝对收益| > 单日最大正向的2倍:     # 说明亏损非常明显，即使有小幅正向反弹也不足以救     set_flag: GLOBAL_NEGATIVE_SEVERE = true   ELSE:     set_flag: GLOBAL_NEUTRAL = true
```

然后在优先级3的判断中，加入这个全局标志：

```sql
优先级3规则修订：%
   AND 最后2天≥1天≤0%    AND NOT(连续2天>0.3%)   AND GLOBAL_POSITIVE = false  # <-- 新增排除条件   AND NOT(末日为正向且环比前日增长):  # <-- 新增排除条件  RETURN true
```

最后一个排除条件"末日为正向且环比前日增长"捕捉的就是case 3这样的场景：虽然中间有大幅下跌，但最后一天反弹且是正向，说明趋势在好转。这个优化的效果：在包含case 3类似场景的测试集中，这类case的确出现的少了很多。这说明全局收益的综合考虑非常关键。

六、Prompt工程核心经验与启示

**6.1 生产级Prompt的五大原则**

在这个项目中，我总结了设计生产级Prompt的五大核心原则。

这些原则不仅适用于AB实验评估，也适用于其他需要高准确率和可解释性的决策场景。

（不是标准原则哈hh，我个人总结的，大家按需）

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

### 显式优于隐式—消除模型的"创意空间"

问题场景：在最初的Prompt中，我们这样描述规则：

```javascript
"如果实验数据连续负向，建议下线"
```

看起来很简洁，但问题是：什么叫"连续"？模型可能有多种理解：

- 理解A：相邻的多个日期都是负向
- 理解B：数组中连续出现的负向数据
- 理解C：在某个时间窗口内都是负向

不同的理解会导致不同的判断。

解决方案：用伪代码显式定义

```makefile
优化后的描述：
## 关键指标定义 - 最长连续负向天数max_consecutive_negative = 0current_consecutive = 0for each day in sorted_days_by_date: # 必须按日期排序if day.dauRelativeChangePct <= 0%:current_consecutive += 1max_consecutive_negative = max(max_consecutive_negative, current_consecutive)else: # 遇到正向，重置计数current_consecutive = 0eturn max_consecutive_negative
```

这样就消除了模型的理解歧义。

deepseek-r1在看到明确的伪代码后，会严格按照步骤执行，而不是"自由发挥"。

为什么这对deepseek-r1特别有效？

deepseek-r1的设计理念是"忠实执行推理逻辑"。

当你给它伪代码时，它会：

1.理解伪代码的逻辑

2.按步骤执行

3.输出结果

4.不会试图"优化"或"改进"你的算法

这恰好是我想要的—在生产环境中，我需要的是稳定的、可预测的行为，而不是"聪明的"行为。

分层而非平铺—优先级保证秩序

问题场景：假设有10条规则，都放在同一个列表中：

```nginx
Rule 1: IF 连续3天负向 → 下线Rule 2: IF 趋势恶化 → 下线Rule 3: IF 高波动 → 下线Rule 4: IF 衰减 → 下线...Rule 10: ...
问题：当多条规则同时触发时怎么办？      规则之间有冲突怎么办？      哪个优先执行？
```

解决方案：引入优先级分层

我们将10条规则重新组织为6个优先级，形成一个决策树。关键特性：

1.优先级递减：优先级越高，规则越严格，下线的置信度越高

2.短路评估：一旦某个优先级触发，直接输出结果，不再继续向下

3.明确的"通过条件"：每一级都明确说出"什么时候可以继续到下一级"

```nginx
IF 优先级0触发（数据不足）  → 直接返回false，退出ELIF 优先级1触发（连续负向）  → 直接返回true，退出ELIF 优先级2触发（趋势恶化）  → 直接返回true，退出...ELSE  → 返回false，继续观察
```

这个设计的优势：

- 避免了规则之间的冲突（因为一旦匹配就退出）
- 清晰的决策路径（便于调试和优化）
- 易于新增规则（直接加入新的优先级）

利用排除条件保障精准率

问题场景：在Bad Case分析中，发现一个现象："连续3天负向"这个规则虽然逻辑清晰，但容易误判的情况：

```ruby
数据：Day 1:-0.1%, Day 2:-0.05%, Day 3:-0.02%, Day 4:+2.0%
系统判断：前3天都是负向 ✓ → 下线2
```

解决方案：在规则触发前加入排除条件

```sql
优化后的优先级1规则：
IF 最长连续负向天数 ≥ 3   AND 最后一天 ≤ 0%   AND 最后2天中无正向  # <-- 这是排除条件  RETURN trueELSE  继续到下一优先级
```

通过排除条件"最后2天无正向"，我们防止了上面这个case的误判。

排除条件的设计原则：

1.基于业务常识：选择那些"虽然满足主规则，但业务上不应该下线"的场景

2.可量化：排除条件应该是能精确计算的，不能模糊

3.不要过度：太多排除条件会让规则变得复杂，难以维护

经验是：每个规则最多3-4个排除条件，超过这个数字就说明规则设计有问题。

全局检查比局部更重要

问题场景：一个策略的数据看起来符合"衰减"的特征，但整体上赚了1000人。按照单纯的"衰减规则"应该下线，但这样做就损失了一个好策略。

解决方案：在关键的规则判断前，加入"全局收益检查"

```shell
在优先级3之后、优先级4之前，插入：
IF 累积绝对收益 > 0:  # 即使后续的规则可能触发，也先救这个case  # 标记为GLOBAL_POSITIVE = true  # 在优先级4-5的判断中，将其作为一个排除条件
这样保证了：有明确正收益的策略，不会因为短期衰减或波动就被下线
```

为什么叫"全局检查"？

因为它的视角不是"这一个规则是否适用"，而是"综合所有维度看，这个策略是否真的应该下线"。

这个检查的重要性在Bad Case分析中被充分验证了—它解决了30%的误判问题。

可解释性是长期迭代的基石

问题场景：

```json
{  "isRecommendOffline": true,  "recommendation": "建议停止实验"}
```

问题：产研同学看到这个输出，能知道原因吗？不能

他们会：

1.找数据自己看一遍

2.对比自己的判断

3.如果不一致就驳回

这样系统就没有起到决策支持的作用。

解决方案：recommendation中包含具体的数据支撑和理由

```json
{  "isRecommendOffline": true,  "recommendation": "最近两天用户持续减少，第6天损失35人，第7天损失42人，趋势恶化幅度加大，建议停止实验"}
```

现在产研同学可以：

1.快速验证：35人和42人确实对应第6、7天的数据吗？✓

2.理解逻辑：系统用的是"趋势恶化"这个信号

3.二次确认：这个信号在我们的业务场景中是否有效？

好的recommendation应该满足：

1.具体性：包含具体的日期、数字、指标

2.可验证性：产研可以快速验证recommendation中提到的数据

3.易理解：用"最近两天"代替"倒数第二天和最后一天"

4.有理由：说出"为什么"而不仅仅说"结论"

5.无专业术语：禁用"千人DAU、相对变化率、标准差"等

在我的Prompt中，recommendation的写法有一个专门的章节来规范：

```bash
## recommendation字段生成规范：
下线情况（isRecommendOffline=true）：  格式："{触发的规则简述} + {具体数据} + {建议}"  例："最近2天连续减少，分别损失20人和35人，损失扩大，建议停止实验"  ✓ 包含规则（连续减少）  ✓ 包含数据（20人、35人）  ✓ 包含趋势判断（损失扩大）  ✓ 清晰的建议（停止实验）
不下线情况（isRecommendOffline=false）：  格式："{为什么不符合下线条件} + {正向信号}"  例："虽然第3天下跌65人，但第4天反弹48人，无法确认趋势恶化，建议继续观察"  ✓ 承认有负向信号（65人下跌）  ✓ 指出反弹证据（48人反弹）  ✓ 解释判断逻辑（无法确认趋势恶化）  ✓ 建议行动（继续观察）
禁用词清单：  ✗ 千人DAU、相对变化率、标准差 → 太专业  ✗ 效果不稳、可能失败 → 太模糊  ✗ 第n天 → 改用具体日期如"12月7日"  ✗ 感觉、似乎、可能 → 太主观
```

**6.2 Prompt迭代的系统方法论**

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

科学地收集Bad Cases

不是所有的"系统判断与人工判断不一致"都值得分析。

我建立了一个Bad Case的收集标准：

```swift
什么算Bad Case？
必需条件：1. 系统的判断（isRecommendOffline）与人工判断不同2. 已经有人工的"黄金标签"（Gold Label）来验证谁是对的   - 对于下线的策略：1周后是否真的有正向效果？   - 对于继续的策略：最后是否确实有好的结果？3. 这个case涉及的数据质量是正常的（排除数据异常导致的case）
收集频率和方式：- 每日自动对比系统输出和人工确认的结果- 生成差异报告，筛选出真正的bad case- 分类打标签（Category A/B/C）- 汇总成Bad Case库，定期分析
案例标准化：  {"badcase_id": "BC-001",   "input": [...],   "system_output": {...},   "human_judgment": "false",   "category": "A1",   "root_cause": "日期排序混乱",   "fix_applied": "v1.1",   "resolved": "yes"}
```

根本原因分析（RCA）

收集到bad case后，不能直接就改Prompt。首先要理解"为什么会错"。

我用一个简单的RCA框架：

> 什么是RCA：根本原因分析（Root Cause Analysis，简称 RCA） 是一种系统化的问题排查与解决方法，旨在识别导致问题发生的最底层、最本质的原因，而非仅仅处理表面现象或直接诱因。

```ruby
Level 1: 表面原因 → 系统判断的具体错误Level 2: 直接原因 → Prompt中的哪个规则或计算出了问题Level 3: 根本原因 → 为什么Prompt的设计在这里有漏洞
例如BC-001（日期排序混乱）：  Level 1: 系统把[1207:-67, 1208:-35, 1209:+34, 1210:+45]理解成了负向趋势  Level 2: 在计算"最长连续负向段"时，没有明确按日期排序  Level 3: Prompt的Workflow第2步（数据预处理）缺少"排序"这个环节  Fix: 在Workflow第2步中明确添加"按dt字段升序排序"
```

为什么要分三个层级？

因为如果只在Level 1上解决，你可能修修补补，但根本问题没解决。

而如果直接在Level 3上解决，就能一劳永逸。

针对性修复&事后集成测试

修复Prompt时，不能直接改原版本。

一般采用分支-修复-测试-合并的流程：

```markdown
当前版本：v1.1（准确率72%）发现bad case BC-009-012（连续性判断）
流程：1. 创建分支版本：v1.1-fix-continuity   └─ 修改：优先级1的连续性定义   └─ 修改：添加排除条件   2. 在bad case集上测试：   ├─ BC-009: 原来错，现在？ → 改对了 ✓   ├─ BC-010: 原来错，现在？ → 改对了 ✓   ├─ BC-011: 原来对，现在？ → 仍然对 ✓   └─ BC-012: 原来错，现在？ → 仍然错 ✗（需要进一步调查）   3. 在全量测试集上验证：   ├─ 新版本准确率：73% （比v1.1的72%提升1%）   ├─ 精准率：72% （与v1.1相同）   ├─ 召回率：89% （从v1.1的85%提升4%）   └─ 结论：这个修改是正向的，可以merge   4. 合并回主线：   └─ v1.2-continuity 成为新的baseline
```

能看到这这里的小伙伴，辛苦你了。

帮你抛出内心的困惑，为什么要在三个层面上进行测试呢？

```sql
Case
  ├─ 快速反馈：5分钟能跑完  ├─ 精准定向：只验证与这个修改相关的case  └─ 目标：修改应该修复目标case，不引入新错误
Test Level 2: 全量训练集（500个历史case）  ├─ 综合评估：确保修改不伤害其他case  ├─ 关键指标：精准率、召回率、F1-Score  └─ 目标：整体准确率应该不下降
Test Level 3: 新增case（第4周Hold-out集）  ├─ 真实验证：用从未见过的数据测试  ├─ 防止过拟合：确保改进不是"死记硬背"  └─ 目标：在新数据上也有显著改进
```

定量跟踪改进效果

建立改进追踪表，清晰地记录每个修改的效果：

（不过不适合敏捷开发的同学，例如我，我也是简单的按照自己的需求进行记录，有QA的同学可以规范来）

```css
Prompt版本演进与准确率变化：
版本    修改内容                影响的Rule  Bad Case修复数  准确率  F1-Score  备注─────────────────────────────────────────────────────────────────────────v1.0    初版                     -          0/30           68%     72%      baselinev1.1    +数据预处理强化          全局       8/30 (27%)     72%     75%      Category A fixv1.1-a  +符号校验逻辑           Rule1-3    2/30 (7%)      73%     76%      增量修改v1.2    +全局收益检查            Rule3-5    9/30 (30%)     77%     80%      Category B fixv1.2-b  +衰减幅度量化            Rule4      5/30 (17%)     78%     81%      增量修改v1.2-c  +业务场景补充            全局       3/30 (10%)     79%     82%      Category C fixTarget  目标版本                 -          -              80%+    85%+     预期2周内达成
```

这个表的作用：

1.追踪性：每个修改的影响都能量化跟踪

2.决策支持：优先修复影响范围大的case（BC修复数多的优先做）

3.动力保持：能清晰看到准确率在持续上升

4.知识沉淀：后续新人能快速理解演进历史

记录所有应对策略

在迭代过程中记录不同类型的修改。每种类型的复杂度和风险不同，方便沉淀经验与复盘：

```sql
修改类型          风险等级  实现难度  影响范围  推荐做法────────────────────────────────────────────────类型A：排除条件增加    低      简单    局部    直接修改，快速测试类型B：规则阈值调整    中      简单    中等    需要在BC集和训练集上同时测试类型C：新增优先级      高      复杂    全局    需要完整的三级测试，可能引入回归类型D：指标重新定义    高      复杂    全局    需要充分的理论论证，谨慎实施
例如v1.2中的"全局收益检查"是类型D：  ├─ 首先在1个bad case（BC-017）上验证有效  ├─ 然后在Bad Case集（30个）上扩大验证  ├─ 再在全量集上验证不伤害其他case  ├─ 最后才在新数据上验证真实效果  └─ 整个过程耗时3天，但风险被很好地控制了
```

**6.3 <关键>从Prompt工程学到的通用启示**

除了AB实验评估这个具体场景，这个项目还带来了一些可迁移到其他领域的启示。

### 大模型适合"有明确规则但规则复杂"的任务

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

AB实验评估之所以适合，是因为：

- 输入输出都是结构化的JSON
- 规则虽然多（6层优先级），但每一层都很清晰
- 68-80%的准确率对于"建议"这个use case完全够用
- 规则需要频繁调整（每周都有新的bad case insights）

### Prompt本质上是"知识的显式编码"

| 维度 | 传统编程（写代码） | Prompt 编程 |
| --- | --- | --- |
| 表达形式 | `if (consecutive_negative_days >= 3 && last_day <= 0%) { return true; }` | `IF 最长连续负向天数 ≥ 3<br> AND 最后一天 ≤ 0%<br> AND 最后2天无正向:<br> RETURN true` |
| 可读性 | 仅程序员能准确理解逻辑 | 接近自然语言，产品经理、数据分析师等非技术人员也能理解与评审 |
| 协作效率 | 逻辑隐藏在代码中，跨角色沟通成本高 | 规则透明、语义清晰，便于多方对齐和迭代 |

Prompt工程实际上是在做"知识编码"：把人脑中的隐性规则，转化为模型能理解的显式规则。

在这个过程中：

- 产品 + 数据分析师的domain knowledge变成了Prompt的内容
- 工程师的工作是确保Prompt的逻辑清晰、可执行
- 反复的Bad Case分析，实际上是在"完善知识库"

这个思路也可以应用到：

- 内容审核规则
- 风险识别模型
- 推荐系统的候选池过滤

Bad Case的价值被严重低估

在很多 AI 项目里，Bad Case 被当作“失败”来掩盖。但在本次实践项目中，Bad Case 是最高效的反馈信号。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

为什么 Bad Case 还挺好的？

暴露 Prompt 的认知盲区：本次项目的 Bad Case，最终归为三类根本问题：

1.对数据波动的理解偏差

2.规则边界定义模糊

3.缺乏业务上下文（比如民生 Push 的触达延迟特性）

每一个 Bad Case 都在告诉我：“好像又漏掉了什么 = =”

驱动高 ROI 的迭代：基于 Bad Case 的定向优化，平均每次修改能带来 1–2% 的准确率提升；相比无目标地调参或重写 Prompt，效率高出 n 倍以上。

可被标准化处理：建立了一套闭环流程：分类 → RCA → 修复 → 验证 → 合入主干和写代码时候的修 bug 感觉类似，但更有意义的是把经验沉淀为可复用的工作范式。

构建领域知识库：每次修复 Bad Case，本质上是在给 Prompt “补课”对自己来说补的是对业务、数据和决策逻辑的理解。长期积累下来，这套 Prompt 不再只是规则集合，而是一个有上下文、有判断力的业务知识库。

七、展望

1.新增渠道，扩大自动化作业范围。毕竟量大才能看到效果。

2.持续优化prompt，更优秀的拆分逻辑，各司其职，按需动态拼接，分别保障最低粒度的结果。

3.继续沉淀大模型工程领域的学习，虽然咱区分开底层大模型算法，但是相信工程层面也能做出不一样的东西。

继续滑动看下一个

阿里云开发者

向上滑动看下一个

![kimi](data:image/png;base64,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)
