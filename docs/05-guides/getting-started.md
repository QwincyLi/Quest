# Quest快速开始指南

> 5分钟创建你的第一个AI生成游戏

---

## 安装（开发中）

```bash
# 安装Quest CLI
npm install -g @quest/cli

# 验证安装
quest --version
```

---

## 创建第一个项目

```bash
# 创建新项目
quest new my-first-game

# 进入项目目录
cd my-first-game

# 启动编辑器
quest dev
```

---

## 第一次对话

Quest编辑器启动后，你会看到AI对话界面。试试这些：

### 示例1: 创建简单场景

```
你: "创建一个森林场景"

Quest AI: 
  ✅ 已创建森林场景
  - 20棵树木
  - 草地地形
  - 黄昏光照
  - 鸟鸣环境音
  
  [预览窗口实时显示场景]
```

### 示例2: 添加角色

```
你: "添加一个主角，战士类型"

Quest AI:
  ✅ 已添加角色 "Hero"
  - 外观: 战士铠甲
  - 能力: 行走、跳跃、攻击
  - 位置: 场景中心
```

### 示例3: 添加敌人

```
你: "添加3个哥布林敌人，会巡逻"

Quest AI:
  ✅ 已添加3个哥布林
  - 威胁等级: 低
  - 行为: 方形路径巡逻
  - AI: 检测到玩家会追击
```

---

## 运行游戏

点击预览窗口的"运行"按钮，或：

```bash
quest play
```

---

## 发布游戏

```bash
# 构建Web版本
quest build --platform web

# 构建移动版
quest build --platform ios
quest build --platform android

# 一次性构建所有平台
quest build --all
```

---

## 下一步

- 📚 [完整教程](tutorial-platformer.md)
- 📘 [API文档](../04-api-reference/complete.md)
- 🎨 [语义化API](../03-semantic-api/complete.md)

---

**注**: Quest目前处于架构设计阶段，CLI工具和编辑器尚未开发。预计2026年6月MVP版本发布。
