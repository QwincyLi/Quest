# Quest引擎文档导航

欢迎来到Quest AI原生游戏引擎文档中心！

---

## 📖 快速开始

刚接触Quest？从这里开始：

1. **[5分钟了解Quest](01-architecture/overview.md)** - 架构总览
2. **快速入门**（开发中）
3. **第一个游戏**（开发中）

---

## 📚 核心文档

### 🏗️ 架构设计

**从这里了解Quest的整体设计**

- [架构总览](01-architecture/overview.md) ⭐ 推荐先读
- [完整架构文档](01-architecture/complete.md) - 详细版
- [技术栈对比](01-architecture/tech-stack.md) - 为什么选择这些技术
- 技术决策记录（开发中）

**关键概念**:
- 五层架构设计
- AI驱动型引擎定位
- Cocos 4 + 语义化API封装策略

---

### 🤖 Agent系统

**了解Quest的AI核心**

- [Agent系统完整文档](02-agent-system/complete.md) ⭐ 核心文档

**包含内容**:
- Multi-Agent架构（6个专家Agent）
- Skill系统（Markdown动态加载）
- MCP集成（3200+工具）
- OpenRouter集成
- 自修改工作流
- 质量评估管道

**关键特性**:
- 动态工作流（运行时可修改）
- Skill热加载
- Agent间协作
- 自我优化

---

### 🎨 语义化API

**Quest的核心创新**

- [语义化API完整文档](03-semantic-api/complete.md) ⭐ 核心创新

**设计原则**:
1. 声明式 - 描述"是什么"而非"怎么做"
2. 语义化 - 用概念而非技术细节
3. 可组合 - 模块化和复用

**核心价值**:
```typescript
// 传统API: 50行代码
const node = new cc.Node();
node.setPosition(...);
// ... 48行

// 语义化API: 1行
const hero = await quest.create({ type: 'character' });

// 代码减少: 98%
```

---

### 📘 API参考手册

**完整的API文档**

- [API参考完整文档](04-api-reference/complete.md)

**核心API**:
- `quest.create()` - 创建游戏对象
- `quest.createScene()` - 创建场景
- `quest.modify()` - 修改对象
- `quest.generateAsset()` - 生成资产
- `quest.savePrompt()` - 保存提示词

**使用示例**:
- 完整游戏开发流程
- 对话式开发示例
- Prompt-as-Source工作流
- Multi-Agent协作示例

---

## 🎓 使用指南

**实战教程和最佳实践**

- 快速开始（开发中）
- 平台跳跃游戏教程（开发中）
- RPG游戏教程（开发中）
- 开发自定义Skill（开发中）
- 部署发布指南（开发中）

---

## 🚀 实施指南

**开发Quest引擎本身**

- [实施路线图](06-implementation/roadmap.md) - 12个月开发计划

**包含内容**:
- MVP/Alpha/Beta三阶段规划
- 10个里程碑详细计划
- 资源需求（人员、成本）
- 风险管理和缓解策略
- Gantt甘特图

**关键里程碑**:
- M1-M3: MVP阶段（3个月） - 核心验证
- M4-M6: Alpha阶段（3个月） - 功能完善
- M7-M10: Beta阶段（6个月） - 生产就绪

---

## 🔬 参考资料

- [参考文献索引](07-references/index.md)

**包含**:
- AI原生开发范式（PDD、AI-Native Patterns）
- 学术研究（6篇论文）
- 技术基础（MCP、Cocos 4）
- 经典理论（DSL、声明式编程）
- 相关框架和引擎

---

## 📖 文档阅读路径

### 路径1: 快速了解（30分钟）
1. [架构总览](01-architecture/overview.md)
2. [语义化API示例](03-semantic-api/complete.md#使用示例)
3. [快速开始](05-guides/) （开发中）

### 路径2: 深入学习（3小时）
1. [完整架构文档](01-architecture/complete.md)
2. [Agent系统](02-agent-system/complete.md)
3. [语义化API设计](03-semantic-api/complete.md)
4. [API参考](04-api-reference/complete.md)

### 路径3: 准备开发Quest（5小时）
1. 以上全部文档
2. [实施路线图](06-implementation/roadmap.md)
3. [技术栈对比](01-architecture/tech-stack.md)
4. [参考资料](07-references/index.md)

---

## 🆘 常见问题

### Q: Quest和Unity Muse有什么区别？
A: Unity Muse是传统引擎的AI插件（路线1），Quest是AI驱动的全新引擎（路线2）。Quest重构了整个工作流，而非仅添加AI辅助。

### Q: 为什么选择Cocos 4？
A: MIT协议、跨平台、2D/3D都支持、TypeScript技术栈统一。详见[技术栈对比](01-architecture/tech-stack.md)

### Q: 什么是语义化API？
A: 用"角色"、"威胁等级"等人类概念描述游戏，而不是"setPosition"、"addComponent"等技术细节。AI能直接理解和生成。详见[语义化API](03-semantic-api/complete.md)

### Q: 开发周期多长？
A: 预计12个月到1.0版本。MVP 3个月可用。详见[路线图](06-implementation/roadmap.md)

---

## 🤝 贡献文档

发现文档问题？欢迎贡献！

- 提交Issue
- 创建PR
- 加入Discord讨论

---

**文档版本**: v1.0.0  
**最后更新**: 2026-03-19
