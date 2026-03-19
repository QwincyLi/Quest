# Quest术语表

快速查找Quest相关术语的定义。

---

## A

### AI-Native（AI原生）
从零为AI设计的系统，而非传统系统加AI插件。

**例子**: Quest是AI原生引擎，Unity Muse是AI辅助型。

### AI驱动型引擎
技术路线2，AI占据核心地位，人类保持控制权。效率提升5-10倍。

---

## B

### BehaviorTree（行为树）
AI决策系统，树状结构描述NPC行为逻辑。

### Blackboard（黑板系统）
Agent间共享数据的机制，类似团队白板。

---

## C

### Cocos 4
MIT协议的开源游戏引擎，Quest的底层渲染引擎。

---

## D

### DSL (Domain-Specific Language)
领域特定语言，为特定问题域设计的语言。

**参考**: Martin Fowler的DSL设计理论

### Declarative API（声明式API）
描述"要什么"而非"怎么做"的API设计。

**例子**: Kubernetes的YAML配置

---

## E

### Evaluation Pipeline（评估管道）
自动化的质量检查系统，包含性能/语义/一致性/可玩性检查。

---

## F

### Fluent Interface（流畅接口）
链式调用的API设计，读起来像自然语言。

```typescript
quest.createCharacter('Hero')
  .withAppearance('warrior')
  .atPosition(10, 20)
  .build();
```

---

## G

### Gateway（网关）
中心化的控制节点，管理所有请求和Agent调度。

**参考**: OpenClaw的Gateway模式

---

## M

### MCP (Model Context Protocol)
Anthropic提出的AI工具协议，"AI的USB-C"。

**作用**: 统一AI与外部工具的连接标准。

### Multi-Agent
多个专家Agent协作完成复杂任务。

**Quest的Agent**: Master、Scene、NPC、Dialogue、Code、Evaluation

---

## O

### OpenRouter
统一的LLM API网关，支持300+模型。

**优势**: 一次集成，随时切换模型。

---

## P

### Prompt-as-Source
提示词作为资产的"源代码"，而非中间产物。

**传统**: PSD → PNG → 导入引擎  
**Quest**: Prompt → 直接生成运行时资产

### PDD (Prompt-Driven Development)
提示词驱动开发，提示词是开发的源头。

**参考**: [PDD框架](https://github.com/promptdriven/pdd/)

---

## S

### Semantic API（语义化API）
用人类和AI都能理解的概念描述游戏，而非底层技术细节。

**例子**: 
```typescript
// 语义化
{ type: 'enemy', threat: 'medium' }

// 非语义化
{ health: 100, damage: 25, aggroRange: 15 }
```

### Skill
可复用的AI能力，用Markdown定义。

**格式**: 元数据 + Prompt + 工具声明 + 示例

---

## T

### TypeScript
Quest的主要编程语言，全栈统一。

---

## W

### Workflow（工作流）
Agent执行任务的流程图。

**Quest特性**: 自修改工作流（运行时可修改）

---

## 技术路线

### 路线1: AI辅助型
传统引擎 + AI插件。代表：Unity Muse

### 路线2: AI驱动型 ⭐
AI为核心，人类控制。代表：**Quest**

### 路线3: AI自主型
AI完全自主。代表：DeepMind Genie

### 路线4: AI共生型
人机深度协作。未来方向。

---

## 概念对比

| Quest概念 | 传统引擎等价物 | 优势 |
|----------|--------------|------|
| Prompt资产 | PSD/FBX源文件 | 可版本控制，无二进制 |
| 语义化API | 底层引擎API | AI可理解，代码减少98% |
| Skill | 代码库/插件 | 动态加载，Markdown定义 |
| Multi-Agent | 单线程执行 | 并行协作，效率高 |
| 评估管道 | 手动测试 | 自动化，质量稳定 |

---

**相关文档**:
- [架构总览](../01-architecture/overview.md)
- [语义化API](../03-semantic-api/principles.md)
- [Agent系统](../02-agent-system/complete.md)
