# Quest项目设置指南

> 如何克隆、构建和开发Quest项目

---

## 📥 克隆项目

### 方式1: 完整克隆（推荐）

```bash
# 克隆Quest项目，包含Cocos4 submodule
git clone --recursive https://github.com/你的用户名/Quest
cd Quest
```

`--recursive`标志会自动克隆所有Git Submodule（包括Cocos 4引擎）。

### 方式2: 分步克隆

```bash
# 1. 克隆主项目
git clone https://github.com/你的用户名/Quest
cd Quest

# 2. 初始化submodule
git submodule init

# 3. 下载submodule内容
git submodule update
```

---

## 🎮 关于Cocos 4引擎Submodule

### 什么是Git Submodule？

Git Submodule允许你在一个Git仓库中包含另一个Git仓库作为子目录。

**Quest的使用方式**:
```
Quest仓库（主项目）
  └── engine/ （Submodule → github.com/QwincyLi/engine）
```

### 为什么使用Submodule？

✅ **优势**:
- Cocos 4引擎独立管理（有自己的Git历史）
- 可以单独更新引擎版本
- Quest主仓库保持轻量
- 清晰的版本锁定

❌ **如果不用Submodule**:
- 需要将整个Cocos 4代码复制到Quest
- Quest的Git历史会包含Cocos 4的所有提交
- 更新Cocos 4很麻烦

### Submodule当前状态

```bash
# 查看submodule状态
git submodule status

# 输出示例:
# 1a2b3c4 engine (v4.0.0-alpha.7)
```

---

## 🔄 更新Cocos 4引擎

### 从你的fork更新

```bash
# 进入engine目录
cd engine

# 拉取最新代码
git pull origin v4.0.0

# 返回Quest根目录
cd ..

# Quest主项目记录更新
git add engine
git commit -m "chore: update Cocos4 engine to latest"
```

### 从Cocos官方上游同步

```bash
cd engine

# 添加上游仓库（一次性操作）
git remote add upstream https://github.com/cocos/cocos4

# 拉取上游更新
git fetch upstream

# 合并到你的fork
git merge upstream/v4.0.0

# 推送到你的fork
git push origin v4.0.0

# 返回并提交Quest更新
cd ..
git add engine
git commit -m "chore: sync Cocos4 from upstream"
```

---

## 🛠️ 构建Cocos 4引擎

### 前置要求

```bash
# 检查Node.js版本（需要 >=14）
node --version

# 检查npm版本
npm --version
```

### 编译Cocos 4

```bash
# 进入engine目录
cd engine

# 安装依赖
npm install

# 构建引擎
npm run build

# 返回Quest根目录
cd ..
```

**预计时间**: 首次构建约10-20分钟

---

## 📦 Quest项目设置（未来）

当Quest源代码开发后：

### 安装依赖

```bash
# 在Quest根目录
npm install
# 或使用pnpm（推荐）
pnpm install
```

### 开发模式

```bash
# 启动所有开发服务
npm run dev

# 单独启动
npm run dev:editor    # 启动编辑器
npm run dev:agent     # 启动Agent后端
```

### 构建

```bash
# 构建所有包
npm run build

# 构建顺序:
# 1. engine (Cocos 4)
# 2. adapter (Cocos4适配器)
# 3. semantic-api (语义化API)
# 4. agent (Agent系统)
# 5. editor (编辑器)
```

---

## 🔍 验证安装

### 检查Submodule

```bash
# 查看submodule状态
git submodule status

# 应该看到:
# <commit-hash> engine (v4.0.0-alpha.7)
```

### 检查Cocos 4

```bash
cd engine

# 查看Cocos 4版本
cat package.json | grep version

# 查看远程仓库
git remote -v
# origin	https://github.com/QwincyLi/engine (fetch)
# origin	https://github.com/QwincyLi/engine (push)
```

---

## ⚠️ 常见问题

### Q1: 克隆时忘记`--recursive`怎么办？

```bash
# 事后初始化submodule
git submodule init
git submodule update
```

### Q2: Submodule显示为空目录？

```bash
# 强制更新
git submodule update --init --recursive --force
```

### Q3: 如何移除Submodule？

```bash
# 不推荐，但如果需要:
git submodule deinit engine
git rm engine
rm -rf .git/modules/engine
```

### Q4: Submodule更新后冲突？

```bash
cd engine
git status
# 解决冲突
git add .
git commit
cd ..
git add engine
```

---

## 📚 相关资源

- [Git Submodule官方文档](https://git-scm.com/book/en/v2/Git-Tools-Submodules)
- [Cocos 4官方文档](https://docs.cocos.com/)
- [Quest文档](docs/README.md)

---

## 🆘 获取帮助

遇到问题？

- 查看[PROJECT_STATUS.md](PROJECT_STATUS.md)了解当前状态
- 阅读[文档](docs/README.md)
- 提交Issue（未来）
- 加入Discord（未来）

---

**最后更新**: 2026-03-19
