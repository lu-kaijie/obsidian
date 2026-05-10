---
title: "AI编码实践：从Vibe Coding到SDD"
source: "https://mp.weixin.qq.com/s/W6-e-uSPcGCqQAXxx_PDgA"
author:
  - "[[式遂]]"
published:
created: 2026-05-08
description:
tags:
  - "clippings"
---
式遂 *2026年1月23日 08:31*

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLwpaH3HGNYYR1pVvHLZpIYuia6yMJmgv6frq6zqVeKUCSKicrewAA01eDzsFa4MopK0eicbTRZHgdUQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

阿里妹导读

本文系统回顾了淘特导购团队在AI编码实践中的演进历程，从初期的代码智能补全到Agent Coding再到引入Rules约束，最终探索SDD（Specification Driven Development，规格驱动开发）——以自然语言规格（spec.md）为唯一真理源，驱动代码、测试、文档自动生成，实现设计先行、可测试性内建与文档永不过期。实践中发现SDD理念先进但落地门槛高、工具链不成熟、历史代码集成难，因此团队当前采用融合策略：以轻量级技术方案模板为输入 + Rules严格约束 + Agent Coding高效实现 + AI自动汇总架构文档，形成兼顾规范性、效率与可维护性的AI辅助编程最佳实践。

一、背景

**1.1 业务背景**

生成式AI技术的范式突破正驱动智能开发工具进入超线性演进阶段，主流代码生成工具的迭代周期已从季度级压缩至周级，智能体架构创新推动开发效能持续提升。

淘特导购系统承载着商品推荐、会场投放、活动营销等多样化的业务场景，技术团队面临着需求迭代频繁、代码腐化及团队协作度高的问题，如何提升开发效率、保证代码质量、降低维护成本成为我们面临的重要挑战。正是在这样的背景下，我们开始尝试将AI技术融入到日常开发流程中，探索从传统编码到AI辅助编程的转变之路。

**1.2 AI编程工具的引入**

2024年初，团队开始探索AI编程工具，希望通过AI提升开发效率和代码质量。最初接触的是Aone Copilot（阿里内部AI工具）的代码智能补全功能，后来逐步尝试Agentic Coding、Rules约束、SDD（Specification Driven Development）等多种AI编程模式。本文将详细记录我们的探索历程、实践经验以及对AI编程未来的思考。

二、代码智能补全与单方法改写

**2.1 初识AI编程**

场景1：代码自动补全

```cs
// 开发者输入：public List<ItemCardVO> buildItemCards(List<ContentEntity> entities) {    List<ItemCardVO> result = new ArrayList<>();    // AI自动补全以下代码    for (ContentEntity entity : entities) {        ItemCardVO itemCard = new ItemCardVO();        itemCard.setItemId(entity.getItemId());        itemCard.setItemTitle(entity.getTitle());        itemCard.setItemImg(entity.getPicUrl());        result.add(itemCard);    }    return result;}
```

场景2：单方法重构

```kotlin
// 原始代码（冗长难读）public String getDiscountText(Long finalPrice, Long nnPrice) {    if (finalPrice == null || nnPrice == null) {        return "";    }    if (finalPrice <= nnPrice) {        return "";    }    Long discount = finalPrice - nnPrice;    if (discount <= 0) {        return "";    }    String discountYuan = String.valueOf(discount / 100.0);    return discountYuan + "元";}// AI重构后（简洁优雅）public String getDiscountText(Long finalPrice, Long nnPrice) {    if (finalPrice == null || nnPrice == null || finalPrice <= nnPrice) {        return "";    }    Money discount = Money.ofFen(finalPrice).subtract(Money.ofFen(nnPrice));    if (discount.getCent() <= 0) {        return "";    }    return String.format("%s元", discount.getYuan());}
```

**2.2 初步收益**

效率提升：

- 去年某次商详项目中，代码补全在对象构建、模型转换中减少70-80%的键盘输入。
- 单方法重构速度提升50%。

体验优化：

- 减少了查找API文档的时间
- 避免了拼写错误和语法错误
- 让开发者更专注于业务逻辑

**2.3 遇到的问题**

然而，这个阶段也暴露出一些问题：

1\. 局限于局部优化：只能帮助完成单个方法或代码片段，无法理解整体业务逻辑；

2\. 缺乏上下文理解：不了解项目的架构规范和代码风格；

3\. 无法应对复杂需求：对于跨多个类、多个模块的需求无能为力；

三、Agentic Coding的探索与挑战

**3.1 Agentic Coding的尝试**

带着上面的问题，我们开始尝试Agentic Coding模式，通过编写详细的提示词（Prompt），让AI一次性实现整个功能。

AI工具：Aone Copilot，模型：Qwen3-coder。

典型的Prompt结构：

```markdown
需求：实现NN页面红包模块背景：- 需要展示用户可用的红包列表- 红包按门槛从小到大排序实现要求：1. 创建数据服务类 NnRedPacketDataService，查询用户红包2. 创建模块VO NnRedPacketVO，包含红包列表、总金额等字段3. 创建模块构建器 NnRedPacketModuleBuilder，组装数据技术细节：- 数据服务需要实现 DataService<List<FundQueryDTO>> 接口- 数据服务实现类需要依赖FpProvider，并执行红包查询，参数请参考已有实现- 模块构建器需要继承 BaseModuleBuilder<NnRedPacketVO>- 使用 @Component 注解标记为Spring Bean- 遵循项目代码规范请生成完整的代码。
```

AI生成的代码示例：

```typescript
// 1. 数据服务@Componentpublic class NnRedPacketDataService implements DataService<List<FundQueryDTO>> {    @Autowired    private FpProvider fpProvider;    @Override    public List<FundQueryDTO> execute(InvocationContext context, JSONObject req) {        // 从配置获取红包池ID        String poolIds = req.getString("nnRedPacketPoolIds");        List<Long> fundPoolIds = Arrays.stream(poolIds.split(","))            .map(Long::parseLong)            .collect(Collectors.toList());        // 查询用户红包        return fpProvider.queryUserFundBuyPoolId(            context,             fundPoolIds,             customRuleId,             securityCode        );    }}// 2. 模块VO@Data@Builderpublic class NnRedPacketVO {    private List<RedPacketItem> redPacketList;    private String totalAmount;    private String expandText;}// 3. 模块构建器@Componentpublic class NnRedPacketModuleBuilder extends BaseModuleBuilder<NnRedPacketVO> {    @Override    public String getName() {        return "nnRedPacket";    }    @Override    protected NnRedPacketVO doBuild(InvocationContext context) {        // 获取红包数据        List<FundQueryDTO> funds = ContextUtils            .<List<FundQueryDTO>>getBizResult(                NnRedPacketDataService.class.getSimpleName(),                 context,                 500            )            .orElse(Collections.emptyList());        if (CollectionUtils.isEmpty(funds)) {            return null;        }        // 构建VO        return NnRedPacketVO.builder()            .redPacketList(buildRedPacketList(funds))            .totalAmount(calculateTotalAmount(funds))            .build();    }    private List<RedPacketItem> buildRedPacketList(List<FundQueryDTO> funds) {        return funds.stream()            .map(this::toRedPacketItem)            .collect(Collectors.toList());    }    // ... 其他方法}
```

**3.2 显著的效率提升**

Agentic Coding实现了开发效率的显著优化，通过提示词驱动的自动代码生成替代了传统手动编写，虽然Prompt设计需要额外时间，但综合效率提升效果明显。

**3.3 快速暴露的问题**

然而，在后续的需求迭代中，我们发现了严重的问题：

问题1：代码延续性差

现象： 同样的NN业务，第二次让AI实现时，生成的代码风格完全不同

```typescript
// 第一次生成（简洁风格）private String buildDiscountText(Money discount) {    return String.format("省%s元", discount.getYuan());}// 第二次生成（冗长风格）  private String buildDiscountText(Money discount) {    BigDecimal yuan = BigDecimal.valueOf(discount.getCent())        .divide(BigDecimal.valueOf(100), 2, RoundingMode.HALF_UP);    String yuanStr = yuan.stripTrailingZeros().toPlainString();    return "省" + yuanStr + "元";}
```

影响： 同一个项目内，类似功能的实现方式五花八门，维护成本高

问题2：代码风格不一致

现象： AI不了解项目的代码规范，导致生成的代码风格和存量代码不一致。

问题3：团队协同性差

现象： 不同开发者写的Prompt差异大，生成的代码质量参差不齐

- 新手写的Prompt过于简单，AI生成的代码质量差
- 老手写的Prompt详细但冗长，难以复用
- 缺乏统一的Prompt模板和最佳实践

**3.4 原因分析**

这些问题的根本原因在于：AI缺乏项目特定的上下文和约束

1\. 没有项目规范：AI不知道项目的代码风格、架构模式、命名规范

2\. 没有领域知识：AI不了解淘特导购业务的特定术语和设计模式

3\. 没有历史经验：每次都是"零基础"生成代码，无法从历史代码中学习

这让我们意识到，需要给AI建立"项目规范"和"领域知识"。

四、Rules约束 - 建立AI的"项目规范"

**4.1 引入Rules文件**

我们开始尝试用Rules文件来约束AI的行为，将项目规范、架构模式、领域知识固化下来。

Rules文件体系：

```bash
.aone_copilot/├── rules/│   ├── code-style.aonerule           # 代码风格规范│   ├── project-structure.aonerule    # 项目结构规范│   └── features.aonerule              # 功能实现规范└── tech/    ├── xx秒杀-技术方案.md      # 具体需求的技术方案    └── xx红包模块-技术方案.md
```

**4.2 Rules文件内容示例**

代码风格规范（code-style.aonerule）

```shell
# 代码风格规范
## Java代码规范/类名使用大驼峰命名法（PascalCase）/方法名和变量名使用小驼峰命名法（camelCase）/常量使用全大写，单词间用下划线分隔（CONSTANT_CASE）
## 空值判断- 集合判空统一使用：CollectionUtils.isEmpty() 或 isNotEmpty()- 字符串判空统一使用：StringUtils.isBlank() 或 isNotBlank()- 对象判空统一使用：Objects.isNull() 或 Objects.nonNull()
## 日志规范- 使用 LogUtil 工具类记录日志- 错误日志格式：LogUtil.error("类名, 方法名, 错误描述, 关键参数={}", param, exception)
## 注解使用- Service类使用 @Component 注解- 数据服务实现 DataService<T> 接口- 模块构建器继承 BaseModuleBuilder<T>
```

项目结构规范

```ruby
# 项目结构规范## 包结构com.alibaba.aladdin.app/├── module/              # 模块构建器│   ├── nn/             # NN业务模块│   ├── seckill/        # 秒杀业务模块│   └── common/         # 通用模块├── domain/             # 领域对象│   ├── module/         # 模块VO（继承ModuleObject）│   └── [业务名]/       # 业务领域对象（BO、DTO）├── dataservice/impl/   # 数据服务实现└── provider/           # 外部服务提供者## 命名规范- 数据服务：[业务名]DataService（如 NnRedPacketDataService）- 模块构建器：[业务名]ModuleBuilder（如 NnFeedsModuleBuilder）- 模块VO：[业务名]VO（如 NnRedPacketVO）- 业务BO：[业务名]BO（如 NnRoundFeatureBO）
```

功能实现规范

```swift
# 功能实现规范## 数据服务层- 必须实现 DataService<T> 接口- 使用 @Component 注解- execute方法的第一个参数是 InvocationContext- execute方法的第二个参数是 JSONObject businessReq示例：\`\`\`java@Componentpublic class NnRedPacketDataService implements DataService<List<FundQueryDTO>> {    @Override    public List<FundQueryDTO> execute(InvocationContext context, JSONObject businessReq) {        // 实现逻辑    }}\`\`\`## 模块构建器- 必须继承 BaseModuleBuilder- 使用 @Component 注解- 实现 getName()、doBuild()、bottomTransform() 三个方法- 通过 ContextUtils.getBizResult() 获取数据服务结果示例：\`\`\`@Componentpublic class NnRedPacketModuleBuilder extends BaseModuleBuilder<NnRedPacketVO> {    @Override    public String getName() {        return "nnRedPacket";    }    @Override    protected NnRedPacketVO doBuild(InvocationContext context) {        List<FundQueryDTO> funds = ContextUtils            .<List<FundQueryDTO>>getBizResult(                NnRedPacketDataService.class.getSimpleName(),                context,                500            )            .orElse(Collections.emptyList());        // 构建逻辑    }}\`\`\`
```

**4.3 技术方案模板**

除了Rules文件，我们还为每个需求创建技术方案文档，明确定义需要生成的代码：

技术方案示例（NN红包模块-技术方案.md）：

```shell
## 业务定义NN红包模块用于展示用户在NN业务场景下可用的红包列表。## 业务领域对象无（复用 FundQueryDTO）## 模块领域对象| 对象含义 | 实现方案 | 属性及类型 ||---------|---------|-----------|| NN红包模块VO | 新增 | 1. redPacketList：List<RedPacketItem> - 红包列表<br>2. totalAmount：String - 总金额<br>3. expandText：String - 展开文案 |## 数据服务层| 数据服务定义 | 实现方案 | execute ||------------|---------|---------|| NN红包查询服务 | 新增 | 1. 从配置获取红包池ID列表<br>2. 调用FpProvider查询用户红包<br>3. 过滤可用红包（状态=2，未过期）<br>4. 返回红包列表 |## 模块构建器| 模块构建器定义 | 实现方案 | doBuild逻辑 ||--------------|---------|-------------|| NN红包模块构建器 | 新增 | 1. 获取红包数据<br>2. 过滤门槛>20元的红包<br>3. 按门槛从小到大排序<br>4. 构建VO |
```

**4.4 显著改善的效果**

引入Rules文件后，我们看到了明显的改善：

代码一致性：

- 所有生成的代码都遵循统一的命名规范
- 项目结构清晰，模块划分明确
- 代码风格保持一致

开发效率：

- 技术方案填写时间从2小时降低到20分钟
- 代码实现时间从1天降低到2小时（需要人工收尾）

团队协作：

- 技术方案成为团队共同语言
- Code Review效率提升50%
- 新人上手时间从1周降低到2天

**4.5 依然存在的问题**

虽然Rules带来了显著改善，但仍存在一些问题：

1\. 需求理解不够深入：AI仍然是基于技术方案"翻译"成代码，对业务语义理解有限

2\. 测试质量参差不齐：虽然能生成单测，但测试用例的通过率和覆盖度仍需人工把关

3\. 文档滞后：代码变更后，文档更新容易遗漏

4\. 依赖关系管理：对于复杂的模块依赖关系，AI处理不够优雅

这些问题让我们思考：能否找到一种方式，让AI能更加规范和延续的coding？

五、SDD探索 - 规格驱动开发

**5.1 SDD的引入**

近期，我们开始初步尝试SDD（Specification Driven Development，规格驱动开发），使用了Spec Kit工具链。

SDD的核心理念：

规格是唯一真理源（Single Source of Truth）

- 所有的代码、测试、文档都从规格生成
- 规格即文档，文档永不过期

设计先于实现

- 先用自然语言描述"做什么"（规格）
- 再让AI生成"怎么做"（代码）

可测试性内建

- 规格中明确定义测试用例
- 自动生成完整的单元测试

**5.2 Speckit执行流程**

5.2.1 环境准备

我们主要使用了两种工具：

1\. iflow + qwen3 coder plus + spec kit

2\. qwen + qwen3 coder plus + spec kit

文件体系：

```javascript
├── .specify/│   ├── memory/│   │   └── constitution.md│   ├── scripts/│   └── templates/├── specs/module
│       ├── checklists/│       │   └── requirements.md│       ├── contracts/│       │   └── api-contract.md│       ├── data-model.md│       ├── plan.md│       ├── quickstart.md│       ├── research.md│       └── spec.md└── req/    └── nn-redpacket.md
```

5.2.2 speckit.constitution—制定整个项目的原则

这一步会生成项目全局的宪章，constitution.md

以下是部分节选：

```markdown
## 核心原则
### I. 模块化服务架构所有服务必须遵循模块化设计原则，具有明确的关注点分离和定义良好的接口。每个模块应具有单一职责并可独立部署。模块必须以松耦合和高内聚的方式设计，以增强可维护性和可扩展性，遵循最小依赖原则。### II. 阿里巴巴开发标准所有代码必须遵循阿里巴巴Java开发指南（基于阿里巴巴Java编码规范）。这包括命名约定、异常处理实践、日志标准、安全最佳实践和性能优化模式。代码必须遵守样式一致性要求，以保持代码库的统一性。### III. 质量保证实践全面测试是强制性的：对所有业务逻辑进行单元测试，对服务交互进行集成测试，对API兼容性进行合同测试。代码覆盖率必须保持在80％以上，特别关注关键业务路径。代码质量工具必须集成到CI/CD管道中以执行标准，遵循阿里巴巴开发规范以确保质量和可靠性。### IV. 模块设计原则遵循单一职责原则，每个模块都有一个明确的目的。模块必须以松耦合和高内聚的方式设计，遵循关注点分离原则。模块边界应与业务能力和领域上下文对齐。所有模块都遵循最小依赖原则，仅导入必要的依赖项以减少系统复杂性。### V. 项目架构设计原则本项目采用分层架构设计，通过模块化组织代码，支持淘特投放业务的各种场景需求。架构层次包括：1. **接入层**：处理请求接入和协议转换2. **解决方案层**：业务解决方案的统一入口3. **子解决方案层**：细粒度的业务处理能力4. **模块构建层**：按业务功能划分的模块构建器5. **数据服务层**：负责各种业务数据的获取、处理和封装6. **外部服务层**：负责调用外部服务并进行模型转换7. **领域模型层**：定义核心业务对象和数据传输对象8. **基础设施层**：包含基础组件和框架封装9. **通用模块层**：公共组件和工具类### VI. 依赖管理遵循最小依赖原则：每个模块应只拥有其实际需要的依赖项。避免模块之间的循环依赖。使用依赖注入实现松耦合。定期审核和更新依赖项以最小化安全漏洞。这确保了可维护和高效的代码结构。### VII. 代码风格一致性在整个项目中保持一致的代码风格，使用标准化的格式化规则。所有代码在合并前必须通过静态分析检查。一致地遵循设计模式，并对与标准实践的任何偏差提供清晰的证明。这确保了统一的代码规范和样式，符合项目标准。### VIII. Speckit中文本地化所有speckit相关文件、文档和配置都应使用中文，以支持本地开发团队。\`.specify/\`和\`specs/\`目录中的文件和相关speckit构件必须使用中文，以便本地开发人员更好地理解和维护，同时应为可能服务国际市场面向用户的组件保留国际化支持。## 安全和合规要求所有代码必须符合阿里巴巴的安全标准，并在部署前进行强制性安全审查。必须为所有暴露的端点实现适当的身份验证和授权。敏感数据必须根据内部合规要求进行处理。必须扫描依赖项中的安全漏洞。## 开发工作流程1. 所有代码更改必须遵循标准的阿里巴巴开发工作流程：功能分支、代码审查、自动化测试和CI/CD管道验证。拉取请求必须通过所有测试并获得指定审阅者的批准后才能合并。除非明确批准进行具有迁移计划的破坏性更改，否则所有更改必须向后兼容。每次更改都必须遵循模块设计原则并保持代码风格一致性。2. 所有操作不要创建新分支，而是在当前分支下进行3. 代码生成必须遵循code-generation-prompt.aonerule文件## 治理本宪法凌驾于所有其他开发实践之上，必须在存储库中的所有工作中遵循。对本宪法的任何修改都需要正式文档、团队批准和迁移计划。所有PR和代码审查必须验证是否符合这些原则。
```

5.2.3 speckit.specify—编写规格说明

这一步会新建一个分支，创建spec.md、requirements.md等文件，这里反复修改了多次，主要还是后续的执行不理想的返工。

NN红包模块规格说明（spec.md）：

```markdown
# NN红包模块规格说明## 功能概述NN红包模块用于在NN频道页面展示用户可用的红包列表，帮助用户了解可以使用的优惠。## 功能需求### FR-1: 红包数据获取**描述：** 系统应该能够查询用户在当前NN业务场景下可用的红包**前置条件：**- 用户已登录- 配置了红包池ID（fundPoolIds）- 配置了规则id（customRuleId）- 配置了securityCode**输入：**- userId：用户ID- fundPoolIds：红包池ID列表- customRuleId：自定义规则ID- securityCode：安全码**处理逻辑：**1. 调用FpProvider.queryUserFundBuyPoolId()查询红包2. 过滤条件：   - 红包状态（payStatus）= 2（可使用）   - 红包未过期（当前时间在startTime和endTime之间）   - 红包门槛 <= 配置的amountThreshold（默认20元）**输出：**- 返回符合条件的红包列表**异常处理：**- 如果FpProvider调用失败，返回空列表- 如果用户未登录，返回空列表.........**处理逻辑：**1. 如果红包列表为空，不展示模块（返回null）2. 构建NnRedPacketVO：   - redPacketList：转换每个红包为RedPacketItem   - totalAmount：计算所有红包金额总和   - expandText：从配置获取展开文案**输出：** NnRedPacketVO## 非功能需求### NFR-1: 性能要求- 红包查询超时时间：500ms- 如果超时，返回空列表，不影响页面其他模块### NFR-2: 可扩展性- 支持配置不同的红包池ID- 支持配置不同的门槛限制## 测试用例### TC-1: 正常流程 - 有可用红包**前置条件：**- 用户ID：123456- 配置的红包池ID：[1001, 1002]- FpProvider返回2个可用红包：  - 红包1：金额5元，门槛15元  - 红包2：金额10元，门槛25元**执行步骤：**1. 调用 doBuild(context)**预期结果：**- 返回的VO不为null- redPacketList包含2个红包- totalAmount = "15.00"（5 + 10）- 红包按门槛排序（红包1在前）### TC-2: 边界条件 - 红包门槛超限**前置条件：**- FpProvider返回1个红包，门槛25元- 配置的amountThreshold = 20元（2000分）**预期结果：**- 返回null（因为门槛超限，红包被过滤）......
```

5.2.4 speckit.plan—制定实施计划

这一步会生成data-model.md、api.md等几个文件，最重要的是plan.md

5.2.5 speckit.tasks—任务分解

生成task.md

5.2.6 speckit.implement—实现代码

数据服务实现

```typescript
@Componentpublic class NnRedPacketDataService implements DataService<List<FundQueryDTO>> {    @Autowired    private FpProvider fpProvider;    @Override    public List<FundQueryDTO> execute(InvocationContext context, JSONObject businessReq) {        long userId = context.getUserId();        if (userId <= 0) {            return Collections.emptyList();        }        // 获取配置        String poolIdsStr = businessReq.getString("nnRedPacketPoolIds");        List<Long> fundPoolIds = parsePoolIds(poolIdsStr);        Long customRuleId = businessReq.getLong("customRuleId");        String securityCode = businessReq.getString("securityCode");        // 查询红包        List<FundQueryDTO> funds = fpProvider.queryUserFundBuyPoolId(            context, fundPoolIds, customRuleId, securityCode        );        if (CollectionUtils.isEmpty(funds)) {            return Collections.emptyList();        }        // 过滤可用红包（按规格FR-1的过滤条件）        Long amountThreshold = businessReq.getLong("amountThreshold", 2000L);        return funds.stream()            .filter(fund -> fund.getPayStatus() == 2)  // 可使用状态            .filter(fund -> isNotExpired(fund))         // 未过期            .filter(fund -> fund.getAmountThreshold() <= amountThreshold)  // 门槛限制            .collect(Collectors.toList());    }    private boolean isNotExpired(FundQueryDTO fund) {        long now = System.currentTimeMillis();        return fund.getStartTime() <= now && now <= fund.getEndTime();    }    // ... 其他方法}
```

模块构建器实现

```typescript
@Componentpublic class NnRedPacketModuleBuilder extends BaseModuleBuilder<NnRedPacketVO> {    @Override    public String getName() {        return "nnRedPacket";    }    @Override    protected NnRedPacketVO doBuild(InvocationContext context) {        // 获取红包数据（FR-1）        List<FundQueryDTO> funds = ContextUtils            .<List<FundQueryDTO>>getBizResult(                NnRedPacketDataService.class.getSimpleName(),                context,                500            )            .orElse(Collections.emptyList());        // 如果无可用红包，不展示模块（FR-3）        if (CollectionUtils.isEmpty(funds)) {            return null;        }        // 红包排序（FR-2）        List<FundQueryDTO> sortedFunds = sortFunds(funds, context);        // 构建VO（FR-3）        return NnRedPacketVO.builder()            .redPacketList(buildRedPacketList(sortedFunds))            .totalAmount(calculateTotalAmount(sortedFunds))            .expandText(getExpandText(context))            .build();    }    /**     * 红包排序：NN专属红包优先，其次按门槛从小到大     * 对应规格 FR-2     */    private List<FundQueryDTO> sortFunds(List<FundQueryDTO> funds, InvocationContext context) {        JSONObject extraParams = getExtraParams(context);        String nnPoolIdsStr = extraParams.getString("nnRedPacketPoolIds");        Set<Long> nnPoolIds = parsePoolIds(nnPoolIdsStr).stream()            .collect(Collectors.toSet());        return funds.stream()            .sorted(Comparator                .comparing((FundQueryDTO fund) ->                     nnPoolIds.contains(fund.getFromFundId()) ? 0 : 1)                .thenComparing(FundQueryDTO::getAmountThreshold))            .collect(Collectors.toList());    }    // ... 其他方法}
```

完整的单元测试

```java
@RunWith(MockitoJUnitRunner.class)public class NnRedPacketModuleBuilderTest {    @Mock    private InvocationContext context;    @Mock    private NnRedPacketDataService dataService;    @InjectMocks    private NnRedPacketModuleBuilder builder;    /**     * 测试用例 TC-1: 正常流程 - 有可用红包     */    @Test    public void testBuild_WithAvailableFunds_ShouldReturnVO() {        // 准备数据        List<FundQueryDTO> mockFunds = Arrays.asList(            createFund(1L, 500L, 1500L),   // 5元，门槛15元            createFund(2L, 1000L, 2500L)   // 10元，门槛25元        );        // Mock        when(dataService.execute(any(), any())).thenReturn(mockFunds);        mockContext(context, "nnRedPacketPoolIds", "1,2");        mockBizResult(context, NnRedPacketDataService.class.getSimpleName(), mockFunds);        // 执行        NnRedPacketVO result = builder.doBuild(context);        // 验证（按规格预期结果）        assertNotNull(result);        assertEquals(2, result.getRedPacketList().size());        assertEquals("15.00", result.getTotalAmount());        // 验证排序：门槛低的在前        assertEquals(1500L, result.getRedPacketList().get(0).getAmountThreshold());    }    /**     * 测试用例 TC-2: 边界条件 - 红包门槛超限     */    @Test    public void testBuild_ThresholdExceeded_ShouldReturnNull() {        // 准备数据：门槛25元，超过配置的20元        List<FundQueryDTO> mockFunds = Arrays.asList(            createFund(1L, 500L, 2500L)        );        // Mock（配置amountThreshold=2000，即20元）        JSONObject businessReq = new JSONObject();        businessReq.put("amountThreshold", 2000L);        when(dataService.execute(any(), eq(businessReq))).thenReturn(Collections.emptyList());        mockBizResult(context, NnRedPacketDataService.class.getSimpleName(), Collections.emptyList());        // 执行        NnRedPacketVO result = builder.doBuild(context);        // 验证（按规格预期结果）        assertNull(result);    }    /**     * 测试用例 TC-3: 异常场景 - FpProvider返回null     */    @Test    public void testBuild_ProviderReturnsNull_ShouldReturnNull() {        // Mock        when(dataService.execute(any(), any())).thenReturn(null);        mockBizResult(context, NnRedPacketDataService.class.getSimpleName(), null);        // 执行        NnRedPacketVO result = builder.doBuild(context);        // 验证（按规格预期结果）        assertNull(result);    }    /**     * 测试用例 TC-4: 异常场景 - 用户未登录     */    @Test    public void testBuild_UserNotLogin_ShouldReturnNull() {        // Mock：userId = 0        when(context.getUserId()).thenReturn(0L);        when(dataService.execute(any(), any())).thenReturn(Collections.emptyList());        mockBizResult(context, NnRedPacketDataService.class.getSimpleName(), Collections.emptyList());        // 执行        NnRedPacketVO result = builder.doBuild(context);        // 验证（按规格预期结果）        assertNull(result);    }    // 辅助方法    private FundQueryDTO createFund(Long id, Long amount, Long threshold) {        FundQueryDTO fund = new FundQueryDTO();        fund.setFundId(id);        fund.setAvailableAmount(amount);        fund.setAmountThreshold(threshold);// 可使用
// 1小时前开始
// 1小时后结束
        return fund;    }}
```

**5.3 SDD带来的改进**

5.3.1 一致性显著提升

代码层面：

- 所有代码都严格遵循规格说明，消除了理解偏差
- 不同开发者实现相同规格，代码风格完全一致
- 代码变更时，必须先更新规格，保证文档与代码同步

业务层面：

- 产品、开发、测试对需求的理解高度一致
- 减少了需求理解偏差导致的返工

5.3.2 可测试性大幅提升

测试覆盖：

- 自动生成的测试用例覆盖了所有正常和异常流程
- 测试用例与规格说明一一对应，确保完整性
- 边界条件和异常场景都有明确的测试用例

测试质量：

- Mock方式规范统一，符合项目最佳实践
- 断言准确全面，不会遗漏关键验证点
- 测试代码可读性好，易于维护

5.3.3 可维护性显著改善

文档永不过期：

- 规格说明就是最准确的文档
- 任何变更都先更新规格，再同步代码
- 新人通过阅读规格说明就能快速理解功能

变更影响分析：

- 修改规格时，清晰知道影响哪些代码模块
- 依赖关系在规格中明确定义
- 重构时可以基于规格验证正确性

代码可读性：

- 代码结构清晰，层次分明
- 注释完整准确，与规格保持一致
- 命名规范统一，易于理解

5.3.4 团队协作效率提升

- 新人通过阅读规格说明快速上手
- 跨团队协作时，规格成为统一语言
- 历史需求回溯更容易，规格即完整记录

**5.4 SDD的问题与挑战**

虽然SDD带来了价值，但在实践中也遇到了一些明显的问题：

问题1：规格编写门槛高

现象： 编写高质量的规格说明需要较强的抽象能力和文档编写能力

- 新手往往写不好规格，过于技术化或过于模糊
- 规格模板虽然有，但如何填写仍需要经验
- 不合格的规格对后面的代码实现影响

影响： 对于简单需求，写规格的时间甚至超过直接写代码

问题2：Spec Kit工具链不成熟

遇到的具体问题：

1. 规格解析不准确
	AI有时无法正确理解规格中的复杂逻辑
	需要用非常精确的语言描述，稍有歧义就可能理解错误
2. 代码生成质量不稳定
	相同的规格，不同时间生成的代码质量差异大
	有时生成的代码过于冗长，有时又过于简化
3. 增量更新困难
	规格修改后，很难做到只更新变化的部分
	往往需要重新生成整个文件，导致手工修改的部分丢失

问题3：与现有代码库集成困难

现象： 我们的代码库已经有大量历史代码，SDD更适合从零开始的新项目

- 历史代码缺乏规格说明，无法纳入SDD体系
- 新老代码风格混杂，维护成本反而增加
- 团队一部分人用SDD，一部分人用传统方式，协作困难

问题4：学习成本高

数据：

- 写出合格的第一份规格说明，平均需要3-5次迭代
- 老员工接受度较低，认为"还不如直接写代码快"

**5.5 SDD适用场景分析**

经过3个月的实践，我们总结出SDD的适用场景：

适合使用SDD：

- ✅ 全新的项目或模块
- ✅ 核心业务逻辑，需要长期维护
- ✅ 复杂度高，需要详细设计的功能
- ✅ 多人协作的大型需求
- ✅ 对质量要求极高的场景

不适合使用SDD：

- ❌ 简单的工具函数或配置修改
- ❌ 快速验证的实验性功能
- ❌ 一次性的临时需求
- ❌ 对现有代码的小修改

六、当前最佳实践 - Rules + Agentic Coding + AI文档汇总

**6.1 融合各阶段优势**

核心思路：

1\. 用Rules约束AI

2\. 用技术方案指导实现

3\. 用Agentic Coding快速迭代

4\. 用AI汇总文档保持同步

**6.2 技术方案模板优化**

我们优化了技术方案模板，更加轻量级：

```shell
# [需求名称]-技术方案## 业务定义[简要描述业务背景和目标，1-2句话]## 业务领域对象[如果需要新增/修改BO或DTO，在此说明]## 模块领域对象[需要新增/修改的VO对象]| 对象含义 | 实现方案 | 属性及类型 ||---------|---------|-----------|| [对象名] | 新增/修改 | 1. 字段1：类型 - 说明<br>2. 字段2：类型 - 说明 |## 数据服务层[需要新增/修改的数据服务]| 数据服务定义 | 实现方案 | execute逻辑 ||------------|---------|-----------|| [服务名] | 新增/复用 | 1. 步骤1<br>2. 步骤2 |## 模块构建器[需要新增/修改的模块构建器]| 模块构建器定义 | 实现方案 | doBuild逻辑 ||--------------|---------|-------------|| [构建器名] | 新增/修改 | 1. 获取数据<br>2. 处理逻辑<br>3. 构建VO |
```

特点：

- 比SDD规格更轻量，编写时间从2小时降低到30分钟
- 比纯Agentic Coding更规范，有明确的结构约束
- 聚焦于"做什么"，而非"怎么做"（实现细节交给AI）

**6.3 AI文档汇总机制**

即：让AI自动维护"整体架构与业务逻辑文档"

6.3.1 文档汇总流程

```javascript
完成需求开发 → 提交AI："将本次代码逻辑汇总到汇总文档" → AI分析代码 → AI更新文档
```

Prompt示例：

```markdown
我刚完成了NN红包模块的开发，请分析以下代码：- NnRedPacketDataService.java- NnRedPacketModuleBuilder.java- NnRedPacketVO.java然后将其业务逻辑汇总到"NN业务整体架构与逻辑文档.md"中，确保：1. 描述模块的核心功能和业务价值2. 说明数据流转过程3. 列出关键的业务规则和判断逻辑4. 保持与现有文档的风格一致
```

6.3.2 架构文档结构

```shell
# NN业务整体架构与逻辑文档## 一、业务概述[业务背景、目标、核心价值]## 二、整体架构### 2.1 技术架构[分层架构图、技术栈]### 2.2 模块组成[各个模块的功能和关系]## 三、核心模块详解### 3.1 NN Feeds模块#### 3.1.1 功能说明[模块的核心功能]#### 3.1.2 数据流转[数据从哪里来，经过哪些处理，最终输出什么]#### 3.1.3 关键逻辑[重要的业务规则、计算逻辑、判断条件]#### 3.1.4 代码位置[对应的类和方法]### 3.2 NN红包模块[类似结构]### 3.3 NN吊钩层模块[类似结构]## 四、数据服务层[各个数据服务的功能和依赖]## 五、关键流程[重要的业务流程时序图]## 六、配置说明[配置项说明]## 七、扩展点[未来可能的扩展方向]
```

6.3.3 AI汇总效果示例

AI生成的文档片段：

```shell
### 3.2 NN红包模块#### 3.2.1 功能说明NN红包模块用于在NN频道页面展示用户可用的红包列表，帮助用户了解可以使用的优惠，提升转化率。#### 3.2.2 数据流转用户请求 → NnRedPacketDataService          → 调用FpProvider查询用户红包         → 过滤可用红包（状态=2，未过期，门槛<=20元）         → 返回红包列表         ↓NnRedPacketModuleBuilder         → 获取红包数据         → 红包排序（NN专属优先，门槛从小到大）         → 构建NnRedPacketVO         → 返回前端
```

```php
#### 3.2.3 关键逻辑**1. 红包过滤规则**\`\`\`java// 只展示满足以下条件的红包：// 1. 状态为可使用（payStatus = 2）// 2. 未过期（当前时间在startTime和endTime之间）// 3. 门槛 <= 配置的amountThreshold（默认20元）if (fund.getPayStatus() != 2) return false;if (fund.getStartTime() > now || now > fund.getEndTime()) return false;if (fund.getAmountThreshold() > amountThreshold) return false;**2. 红包排序规则**\`\`\`java// 排序规则：// 1. NN专属红包（配置的nnRedPacketPoolIds）优先// 2. 其次按门槛从小到大排序funds.stream()    .sorted(Comparator        .comparing(fund -> nnPoolIds.contains(fund.getFromFundId()) ? 0 : 1)        .thenComparing(FundQueryDTO::getAmountThreshold))**3. 总金额计算**\`\`\`java// 计算所有可用红包的金额总和String totalAmount = funds.stream()    .map(FundQueryDTO::getAvailableAmount)    .reduce(0L, Long::sum)    .map(PriceUtils::getPriceFenToYuan)    .orElse("0.00");#### 3.2.4 代码位置- 数据服务：\`NnRedPacketDataService\`  - 位置：\`dataservice/impl/nn/NnRedPacketDataService.java\`  - 核心方法：\`execute(InvocationContext, JSONObject)\`- 模块构建器：\`NnRedPacketModuleBuilder\`  - 位置：\`module/nn/NnRedPacketModuleBuilder.java\`  - 核心方法：\`doBuild(InvocationContext)\`、\`sortFunds(List<FundQueryDTO>)\`- 模块VO：\`NnRedPacketVO\`  - 位置：\`domain/module/nn/NnRedPacketVO.java\`#### 3.2.5 配置项\`\`\`json{  "nnRedPacketPoolIds": "1001,1002,1003",  // NN专属红包池ID  "amountThreshold": 2000,                  // 红包门槛上限（分）  "expandText": "展开查看更多"              // 展开文案}
```

七、思考总结

在淘特导购业务的AIcoding实践中，我们经历了从简单代码补全到Agentic Coding，再到基于规则和SDD的编程模式的演进过程。每个阶段都有其价值和局限性：

1\. 初期探索让我们认识到AI在编码辅助方面的潜力，但也暴露了缺乏规范指导的问题；

2\. Agentic Coding提升了功能实现的完整性，但可延续性和一致性仍有不足；

3\. 基于规则的模式有效解决了代码规范和架构一致性问题，成为当前的主要实践方式；

4\. SDD尝试虽然在理念上很有价值，但在实际应用中还需要进一步完善。

虽然在SDD编程方面遇到了一些挑战，但我们认为AI规范化编程是未来发展的方向。团队中的同学正在持续探索和优化：

1\. 完善工具链：改进Spec Kit等工具，提升自动化能力

2\. 优化流程整合：更好地将SDD模式与现有开发流程结合

3\. 降低学习成本：通过培训和实践案例帮助团队成员适应新模式

4\. 持续改进规则：根据实践经验不断完善规则定义

我们相信，通过持续的探索和实践，一定能找到更适合团队的AI辅助编程模式，进一步提升开发效率和代码质量。

**最后**

本文大部分内容由AI根据代码及配置生成。

八、团队介绍

本文作者式遂，来自淘天集团-淘特用户技术团队。团队主要负责淘宝行业&淘特C端链路的研发工作，包含：搜索推荐、互动游戏、导购、交易等基础服务及创新业务。当下我们积极拥抱AI时代，探索智能化在研发提效和业务场景中的无限可能。技术不只是工具，更是为用户创造价值的力量。

继续滑动看下一个

阿里云开发者

向上滑动看下一个

![kimi](data:image/png;base64,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)