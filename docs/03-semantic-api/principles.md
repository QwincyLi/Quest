# 语义化API设计原则

> **核心理念**: 用人类和AI都能理解的概念描述游戏，而非底层技术细节

---

## 三大核心原则

### 原则1: 声明式（Declarative）

**描述"是什么"而非"怎么做"**

```typescript
// ❌ 命令式（How）- 告诉引擎怎么做
const node = new cc.Node('Enemy');
node.setPosition(100, 200, 0);
node.addComponent(cc.Sprite);
node.addComponent(AIComponent);
// ... 50行代码

// ✅ 声明式（What）- 告诉引擎要什么
const enemy = await quest.create({
  type: 'enemy',
  position: { x: 100, y: 200 },
  threat: 'medium',
});
```

**优势**: AI理解意图，自动推理实现细节

---

### 原则2: 语义化（Semantic）

**用概念而非技术细节**

```typescript
// ❌ 技术细节
enemy.aggroRange = 15;      // AI疑问：为什么是15？
enemy.attackDamage = 25;    // AI疑问：为什么是25？

// ✅ 语义概念
enemy = quest.create({
  type: 'enemy',
  threat: 'medium',        // AI理解：中等威胁
  temperament: 'aggressive', // AI理解：攻击性强
});
// AI推理 → aggroRange: 20, damage: 24
```

**语义层次**:
```
第3层: 自然语言      "创建一个会飞的龙Boss"
第2层: 语义化API     { type: 'boss', species: 'dragon', abilities: ['fly'] }
第1层: 技术实现      new Node() + FlyComponent + DragonAI...
```

---

### 原则3: 可组合（Composable）

**模块化和复用**

```typescript
// 基础模板
const baseEnemy = {
  type: 'enemy',
  behaviors: ['attackable'],
};

// 组合扩展
const flyingEnemy = {
  ...baseEnemy,
  abilities: ['fly'],
};

const magicEnemy = {
  ...flyingEnemy,
  abilities: [...flyingEnemy.abilities, 'cast-spell'],
};

// 使用
const wizardBoss = await quest.create({
  ...magicEnemy,
  threat: 'boss',
});
```

---

## 辅助原则

### 容错性
```typescript
// 提供合理默认值
const char = await quest.create({
  type: 'character',
  // name: 自动生成
  // position: 默认(0,0,0)
  // health: 根据type推理
});
```

### 渐进式增强
```typescript
// 简单 → 复杂，逐步增加细节
const enemy1 = quest.create({ type: 'enemy' });
const enemy2 = quest.create({ type: 'enemy', threat: 'high' });
const enemy3 = quest.create({ type: 'enemy', threat: 'high', overrides: { health: 500 } });
```

### 自解释性
```typescript
// 代码即文档
const shop = quest.create({
  type: 'building',
  purpose: 'shop',          // 一眼看出是商店
  shopkeeper: 'blacksmith', // 一眼看出是铁匠铺
});
```

---

## 核心价值

### 对比传统API

```
传统API: 创建巡逻守卫
  → 50行代码
  → 需要理解Node、Component、Animation等概念
  → 30分钟开发时间
  → AI生成成功率: 60%

语义化API:
  → 1行代码
  → 只需理解"守卫"、"巡逻"等概念
  → 30秒开发时间
  → AI生成成功率: 95%

代码减少: 98%
时间减少: 99%
AI友好度: +58%
```

---

完整设计规范见：[语义化API完整文档](complete.md)
