# Quest API设计文档

> **API风格**: 语义化、声明式、AI友好  
> **主要语言**: TypeScript  
> **版本**: v1.0.0-alpha

---

## 目录

- [1. API概览](#1-api概览)
- [2. 核心API](#2-核心api)
- [3. 高级API](#3-高级api)
- [4. 编辑器API](#4-编辑器api)
- [5. 使用示例](#5-使用示例)

---

## 1. API概览

### 1.1 API层次结构

```
quest                          # 全局命名空间
├─ create()                   # 创建游戏对象
├─ createScene()              # 创建场景
├─ modify()                   # 修改对象
├─ find()                     # 查找对象
├─ delete()                   # 删除对象
├─ generateAsset()            # 生成资产
├─ savePrompt()               # 保存提示词
├─ loadPrompt()               # 加载提示词
│
├─ semantic.*                 # 语义化API（主要）
│  ├─ create()
│  ├─ createScene()
│  └─ ...
│
├─ api.*                      # 直接API（中级）
│  ├─ createNode()
│  ├─ addComponent()
│  └─ ...
│
└─ raw.*                      # 原生API（高级）
   └─ cocos.*                 # Cocos 4原生API
```

### 1.2 快速开始

```typescript
// 安装
npm install @quest/engine

// 初始化
import { quest } from '@quest/engine';

await quest.initialize({
  canvas: document.getElementById('game-canvas'),
  openRouterKey: process.env.OPENROUTER_API_KEY,
});

// 创建第一个游戏对象
const hero = await quest.create({
  type: 'character',
  name: 'Hero',
  appearance: 'warrior',
  position: 'center',
  behaviors: ['walkable', 'attackable'],
});

// 创建场景
const scene = await quest.createScene({
  name: 'Level1',
  environment: {
    lighting: 'afternoon',
    atmosphere: 'peaceful',
  },
  objects: [
    hero,
    { type: 'enemy', threat: 'low', count: 3 },
    { type: 'prop', subtype: 'treasure-chest', position: 'random' },
  ],
});

// 运行游戏
await quest.run(scene);
```

---

## 2. 核心API

### 2.1 quest.create()

**签名**:
```typescript
function create<T extends GameObjectType>(
  descriptor: GameObjectDescriptor<T>,
  options?: CreateOptions
): Promise<GameObject<T>>;
```

**参数**:

```typescript
interface GameObjectDescriptor<T = any> {
  // 必需
  type: GameObjectType;
  
  // 可选
  name?: string;
  appearance?: AppearanceDescriptor;
  position?: PositionDescriptor;
  behaviors?: BehaviorDescriptor[];
  attributes?: AttributesDescriptor;
  overrides?: Record<string, any>;
  metadata?: Record<string, any>;
}

interface CreateOptions {
  // 调试选项
  debug?: boolean;           // 显示推理过程
  
  // 生成选项
  model?: string;            // 指定LLM模型
  temperature?: number;      // 创意度
  
  // 性能选项
  optimize?: boolean;        // 自动优化
  budget?: PerformanceBudget; // 性能预算
}
```

**返回值**:

```typescript
interface GameObject<T = any> {
  id: string;
  type: T;
  name: string;
  
  // Cocos 4 Node引用
  node: cc.Node;
  
  // 语义描述符（可用于修改）
  descriptor: GameObjectDescriptor<T>;
  
  // 方法
  modify(changes: Partial<GameObjectDescriptor<T>>): Promise<void>;
  destroy(): Promise<void>;
  getComponent<C>(type: ComponentType): C | null;
  addBehavior(behavior: BehaviorDescriptor): Promise<void>;
}
```

**使用示例**:

```typescript
// 基础用法
const enemy = await quest.create({
  type: 'enemy',
  threat: 'medium',
});

// 完整用法
const boss = await quest.create({
  type: 'boss',
  name: 'ForestKing',
  
  appearance: {
    model: {
      prompt: '巨大的树人，身上覆盖青苔和藤蔓',
      style: 'low-poly',
    },
    scale: 'huge',
  },
  
  position: {
    relativeTo: 'player',
    distance: 'far',
    direction: 'north',
  },
  
  behaviors: [
    {
      type: 'behavior-tree',
      tree: {
        root: {
          type: 'sequence',
          children: [
            { type: 'condition', check: 'player-in-range' },
            { type: 'action', action: 'summon-minions' },
            { type: 'action', action: 'root-attack' },
          ],
        },
      },
    },
  ],
  
  attributes: {
    health: 'boss',
    damage: 'devastating',
    defense: 'high',
  },
  
  overrides: {
    health: 5000,  // 精确控制生命值
  },
}, {
  debug: true,
  optimize: true,
});

// 调试输出:
// [SemanticMapper] type='boss' → 推理基础属性
// [SemanticMapper] threat='boss' → health=[800,1500], damage=[100,200]
// [SemanticMapper] override detected → health=5000
// [AssetGenerator] 生成模型: 树人Boss
// [Optimizer] 优化多边形: 45K → 38K
// [Evaluation] 质量评分: 92/100
```

---

### 2.2 quest.createScene()

**签名**:
```typescript
function createScene(
  descriptor: SceneDescriptor,
  options?: SceneOptions
): Promise<Scene>;
```

**参数**:

```typescript
interface SceneDescriptor {
  name: string;
  
  // 环境
  environment?: {
    timeOfDay?: TimeOfDay;
    weather?: Weather;
    lighting?: LightingDescriptor;
    atmosphere?: Atmosphere;
    ambientSound?: string | SoundPrompt;
    skybox?: string | 'procedural';
  };
  
  // 地形
  terrain?: TerrainDescriptor | TerrainPrompt;
  
  // 对象
  objects?: (GameObjectDescriptor | BatchObjectDescriptor)[];
  
  // 区域
  zones?: ZoneDescriptor[];
  
  // 导航
  navigation?: {
    generateNavMesh?: boolean;
    walkableAreas?: BoundingBox[];
  };
}

interface BatchObjectDescriptor {
  type: GameObjectType;
  count: number;
  distribution?: 'grid' | 'random' | 'cluster' | 'path';
  area?: { width: number; height: number; depth?: number };
  spacing?: number;
  density?: 'sparse' | 'medium' | 'dense';
}
```

**使用示例**:

```typescript
// 完整场景示例
const rpgTown = await quest.createScene({
  name: 'StarterTown',
  
  environment: {
    timeOfDay: 'afternoon',
    weather: 'clear',
    lighting: 'warm',
    atmosphere: 'peaceful',
    ambientSound: 'town-bustle',
  },
  
  terrain: {
    type: 'flat',
    material: 'cobblestone',
  },
  
  zones: [
    {
      name: 'town-square',
      area: { min: [0, 0], max: [50, 50] },
      purpose: 'safe',
      objects: [
        { type: 'building', subtype: 'fountain', position: 'center' },
        { type: 'npc', occupation: 'merchant', count: 3 },
        { type: 'prop', subtype: 'bench', count: 5, distribution: 'path' },
      ],
    },
    
    {
      name: 'residential',
      area: { min: [50, 0], max: [100, 50] },
      purpose: 'story',
      objects: [
        { type: 'building', subtype: 'house', count: 8, distribution: 'grid' },
        { type: 'npc', occupation: 'villager', count: 10, distribution: 'random' },
      ],
    },
    
    {
      name: 'outskirts',
      area: { min: [100, 0], max: [150, 50] },
      purpose: 'combat',
      objects: [
        { type: 'enemy', species: 'wolf', threat: 'low', count: 5 },
        { type: 'prop', subtype: 'tree', count: 20, distribution: 'random' },
      ],
    },
  ],
  
  navigation: {
    generateNavMesh: true,
    walkableAreas: [
      { min: [0, 0], max: [150, 50] },  // 整个区域可行走
    ],
  },
});
```

---

### 2.3 quest.modify()

**签名**:
```typescript
function modify(
  target: string | GameObject,
  changes: Partial<GameObjectDescriptor>,
  options?: ModifyOptions
): Promise<void>;
```

**使用示例**:

```typescript
// 修改对象
await quest.modify('enemy-001', {
  threat: 'high',  // 提升威胁等级
});

// 通过对话修改
用户: "让这个敌人更强一点"
Quest内部:
  → 识别当前选中对象
  → 调用modify({ threat: 'high' })
  → 重新推理数值
  → 更新GameObject
```

---

### 2.4 quest.find()

**签名**:
```typescript
function find(query: SemanticQuery): Promise<GameObject[]>;
```

**语义查询类型**:

```typescript
type SemanticQuery =
  | { type: GameObjectType }                    // 按类型
  | { name: string }                            // 按名称
  | { tag: string }                             // 按标签
  | { within: BoundingBox }                     // 按区域
  | { hasComponent: string }                    // 按组件
  | { attribute: { [key: string]: any } }       // 按属性
  | { prompt: string };                         // 自然语言查询（AI）

// 示例
const enemies = await quest.find({ type: 'enemy' });
const nearbyObjects = await quest.find({ within: playerBounds });

// AI语义查询
const result = await quest.find({
  prompt: '找出所有会飞的敌人',
});
// AI理解 → 查询 hasComponent('FlyComponent')
```

---

### 2.5 quest.generateAsset()

**签名**:
```typescript
function generateAsset(prompt: AssetPrompt): Promise<Asset>;
```

**AssetPrompt类型**:

```typescript
interface AssetPrompt {
  // 资产类型
  type: 'texture' | 'model' | 'audio' | 'animation';
  
  // 描述（核心）
  description: string;
  
  // 艺术风格
  style?: ArtStyle;
  
  // 约束条件
  constraints?: {
    // 纹理约束
    maxSize?: number;              // 最大尺寸（像素）
    format?: 'png' | 'jpg' | 'webp';
    
    // 模型约束
    maxTriangles?: number;         // 最大多边形数
    maxTextureSize?: number;       // 最大纹理尺寸
    
    // 音频约束
    duration?: number;             // 时长（秒）
    format?: 'mp3' | 'ogg' | 'wav';
    
    // 性能预算
    budget?: 'low' | 'medium' | 'high';
  };
  
  // 参考资产
  references?: string[];
  
  // 技术参数
  technical?: {
    pbr?: boolean;                 // 是否PBR材质
    lod?: boolean;                 // 是否生成LOD
    mipmaps?: boolean;             // 是否生成Mipmap
  };
}

type ArtStyle = 
  | 'pixel'
  | 'cartoon'
  | 'realistic'
  | 'low-poly'
  | 'hand-drawn'
  | 'anime'
  | 'cel-shaded';
```

**使用示例**:

```typescript
// 生成纹理
const texture = await quest.generateAsset({
  type: 'texture',
  description: '中世纪石质城墙，有青苔和裂纹',
  style: 'realistic',
  constraints: {
    maxSize: 1024,
    format: 'webp',
  },
  technical: {
    pbr: true,
    mipmaps: true,
  },
});

// 生成3D模型
const sword = await quest.generateAsset({
  type: 'model',
  description: '传奇级别的火焰长剑',
  style: 'low-poly',
  constraints: {
    maxTriangles: 2000,
    budget: 'medium',
  },
  technical: {
    pbr: true,
    lod: true,
  },
});

// 生成音效
const soundEffect = await quest.generateAsset({
  type: 'audio',
  description: '剑击中金属盾牌的声音',
  constraints: {
    duration: 1.5,
    format: 'ogg',
  },
});
```

---

### 2.6 quest.savePrompt() / loadPrompt()

**签名**:
```typescript
function savePrompt(prompt: PromptAsset): Promise<string>;
function loadPrompt(promptId: string): Promise<PromptAsset>;
function diffPrompts(v1: string, v2: string): Promise<PromptDiff>;
```

**PromptAsset类型**:

```typescript
interface PromptAsset {
  id?: string;                    // 自动生成
  type: 'scene' | 'character' | 'behavior' | 'asset';
  content: string;                // 提示词内容
  result?: any;                   // 生成结果
  version?: string;               // 版本号
  metadata?: {
    author?: string;
    tags?: string[];
    description?: string;
    createdAt?: Date;
    modifiedAt?: Date;
  };
}
```

**使用示例**:

```typescript
// 保存提示词
const scene = await quest.createScene({
  name: 'MyForest',
  environment: { lighting: 'dusk' },
  objects: [/* ... */],
});

const promptId = await quest.savePrompt({
  type: 'scene',
  content: 'forest-scene',
  result: scene,
  metadata: {
    tags: ['forest', 'outdoor', 'peaceful'],
    description: '黄昏时分的森林场景',
  },
});
// 返回: "forest-scene-001@v1.0.0"

// 加载提示词
const loaded = await quest.loadPrompt('forest-scene-001@v1.0.0');
const regenerated = await quest.createScene(loaded.result.descriptor);

// 修改提示词（创建新版本）
await quest.modifyPrompt('forest-scene-001', {
  change: '增加更多树木',
});
// 自动保存为: "forest-scene-001@v1.1.0"

// 对比版本
const diff = await quest.diffPrompts(
  'forest-scene-001@v1.0.0',
  'forest-scene-001@v1.1.0'
);

console.log(diff);
// {
//   contentDiff: "增加更多树木",
//   resultDiff: {
//     added: [{ type: 'tree', count: 10 }],
//     removed: [],
//     modified: []
//   },
//   metadata: { ... }
// }
```

---

## 3. 高级API

### 3.1 quest.compose()

**组合多个描述符**:

```typescript
function compose(...descriptors: Partial<GameObjectDescriptor>[]): GameObjectDescriptor;
```

**使用示例**:

```typescript
// 定义基础模板
const baseEnemy = {
  type: 'enemy',
  behaviors: ['attackable', 'collidable'],
};

const flyingEnemy = {
  ...baseEnemy,
  abilities: ['fly'],
  behaviors: [...baseEnemy.behaviors, 'flyable'],
};

const magicEnemy = {
  ...flyingEnemy,
  abilities: [...flyingEnemy.abilities, 'cast-spell'],
  attributes: {
    mana: 100,
  },
};

// 使用组合
const wizardBoss = await quest.create(
  quest.compose(
    magicEnemy,
    { threat: 'boss' },
    { name: 'ArcaneOverlord' }
  )
);
```

---

### 3.2 quest.batch()

**批量创建对象**:

```typescript
function batch(operation: BatchOperation): Promise<GameObject[]>;

interface BatchOperation {
  operation: 'create' | 'modify' | 'delete';
  targets: any[];
  options?: BatchOptions;
}
```

**使用示例**:

```typescript
// 批量创建敌人
const enemies = await quest.batch({
  operation: 'create',
  targets: [
    { type: 'enemy', species: 'goblin', count: 10 },
    { type: 'enemy', species: 'orc', count: 5 },
    { type: 'enemy', species: 'troll', count: 2 },
  ],
  options: {
    distribution: 'random',
    area: { width: 100, height: 100 },
  },
});

// 批量修改
await quest.batch({
  operation: 'modify',
  targets: enemies.map(e => e.id),
  changes: { threat: 'high' },  // 全部提升威胁等级
});
```

---

### 3.3 quest.ai.chat()

**AI对话API**（编辑器主要使用）:

```typescript
interface AIChat {
  send(message: string): Promise<AIResponse>;
  stream(message: string): AsyncGenerator<string>;
  getHistory(): Message[];
  clearHistory(): void;
}

const quest.ai = {
  chat: AIChat,
  agent: AgentControl,
  memory: MemoryControl,
};
```

**使用示例**:

```typescript
// 发送消息
const response = await quest.ai.chat.send(
  '创建一个森林Boss战场景'
);

console.log(response);
// {
//   message: '已创建森林Boss战场景',
//   generatedAssets: [
//     { type: 'scene', id: 'scene-001', name: 'ForestBossArena' },
//     { type: 'boss', id: 'boss-001', name: 'ForestKing' },
//   ],
//   preview: 'cocos4://preview/scene-001',
// }

// 流式对话（实时反馈）
for await (const chunk of quest.ai.chat.stream('添加更多树木')) {
  console.log(chunk);
  // "正在"
  // "添加"
  // "树木"
  // "..."
  // "完成！"
}
```

---

## 4. 编辑器API

### 4.1 编辑器扩展API

```typescript
// quest-editor.d.ts

declare namespace QuestEditor {
  // 注册自定义面板
  function registerPanel(config: PanelConfig): void;
  
  // 注册自定义工具
  function registerTool(config: ToolConfig): void;
  
  // 注册自定义Skill
  function registerSkill(skill: SkillDefinition): void;
  
  // 监听事件
  function on(event: EditorEvent, handler: EventHandler): void;
}

interface PanelConfig {
  id: string;
  title: string;
  icon?: string;
  position: 'left' | 'right' | 'bottom';
  component: React.ComponentType;
}

interface ToolConfig {
  id: string;
  name: string;
  icon: string;
  category: string;
  execute: (context: ToolContext) => Promise<void>;
}
```

**使用示例**:

```typescript
// 创建自定义编辑器面板
QuestEditor.registerPanel({
  id: 'my-custom-panel',
  title: '自定义工具',
  position: 'right',
  component: MyCustomPanel,
});

// 创建自定义工具
QuestEditor.registerTool({
  id: 'batch-rename',
  name: '批量重命名',
  icon: 'edit',
  category: 'utilities',
  execute: async (context) => {
    const selected = context.getSelection();
    
    for (const obj of selected) {
      await quest.modify(obj.id, {
        name: `${obj.name}_renamed`,
      });
    }
  },
});

// 监听编辑器事件
QuestEditor.on('object-selected', (event) => {
  console.log('用户选中了:', event.object);
});

QuestEditor.on('scene-loaded', (event) => {
  console.log('场景加载完成:', event.scene);
});
```

---

## 5. 使用示例

### 5.1 完整游戏开发流程

```typescript
// ============================================
// 示例: 开发一个简单的RPG游戏
// ============================================

// 1. 创建项目
const project = await quest.createProject({
  name: 'MyRPG',
  template: 'rpg-starter',
});

// 2. 创建主角
const hero = await quest.create({
  type: 'character',
  name: 'Hero',
  class: 'warrior',
  
  appearance: {
    prompt: '年轻的战士，棕色头发，穿着皮甲',
    style: 'pixel',
  },
  
  attributes: {
    health: 100,
    mana: 50,
    strength: 15,
    agility: 10,
    intelligence: 8,
  },
  
  behaviors: [
    'walkable',
    'attackable',
    'use-items',
  ],
  
  skills: [
    { name: 'Slash', damage: 'medium', cooldown: 1 },
    { name: 'Heavy Strike', damage: 'strong', cooldown: 5 },
  ],
});

// 3. 创建第一个场景（新手村）
const starterVillage = await quest.createScene({
  name: 'StarterVillage',
  
  environment: {
    timeOfDay: 'morning',
    atmosphere: 'peaceful',
    ambientSound: 'village-ambience',
  },
  
  terrain: {
    type: 'flat',
    material: 'grass',
  },
  
  objects: [
    // 主角出生点
    { ...hero, position: 'center' },
    
    // 村庄建筑
    { type: 'building', subtype: 'inn', position: { x: -20, y: 0 } },
    { type: 'building', subtype: 'shop', position: { x: 20, y: 0 } },
    { type: 'building', subtype: 'blacksmith', position: { x: 0, y: 20 } },
    
    // NPC
    {
      type: 'npc',
      name: 'VillageElder',
      occupation: 'quest-giver',
      position: { x: 0, y: 0 },
      dialogue: {
        greeting: '欢迎来到新手村，年轻的冒险者！',
        quests: [
          {
            id: 'first-quest',
            title: '清理史莱姆',
            description: '村外有史莱姆骚扰，去消灭10只',
            rewards: { exp: 100, gold: 50 },
          },
        ],
      },
    },
    
    {
      type: 'npc',
      name: 'Merchant',
      occupation: 'merchant',
      position: { x: 20, y: 0 },
      inventory: [
        { item: 'health-potion', price: 20, stock: 10 },
        { item: 'sword', price: 100, stock: 3 },
      ],
    },
  ],
});

// 4. 创建战斗场景
const battleField = await quest.createScene({
  name: 'ForestBattleField',
  
  environment: {
    timeOfDay: 'afternoon',
    atmosphere: 'tense',
  },
  
  objects: [
    // 敌人
    { type: 'enemy', species: 'slime', threat: 'trivial', count: 10 },
    { type: 'enemy', species: 'wolf', threat: 'low', count: 5 },
    
    // Boss
    {
      type: 'boss',
      name: 'ForestKing',
      species: 'tree-giant',
      threat: 'boss',
      phases: [
        { health: '100-70%', pattern: 'melee' },
        { health: '70-30%', pattern: 'summon-minions' },
        { health: '30-0%', pattern: 'berserk' },
      ],
    },
  ],
});

// 5. 创建游戏逻辑
const gameLogic = await quest.generateCode({
  prompt: `
创建RPG游戏的核心逻辑：
- 战斗系统（回合制或实时？由AI决定）
- 经验和升级系统
- 物品和背包系统
- 任务追踪系统
  `,
  framework: 'quest-semantic-api',
});

// 6. 测试游戏
await quest.playtest({
  scene: starterVillage,
  automated: true,
  testCases: [
    '玩家能否移动',
    '能否与NPC对话',
    '能否接受任务',
    '能否战斗',
  ],
});

// 7. 导出发布
await quest.build({
  platforms: ['web', 'ios', 'android'],
  optimization: 'aggressive',
  output: './dist',
});
```

---

### 5.2 对话式开发示例

```typescript
// Quest编辑器对话界面

用户: "我想做个塔防游戏"

AI: "好的！让我为你创建塔防游戏的基础框架。包括：
     1. 游戏场景（路径+据点）
     2. 敌人波次系统
     3. 防御塔类型
     4. UI界面
     
     你想要什么风格？像素风、卡通风还是写实风？"

用户: "像素风"

AI: [开始生成...]
    ✅ 创建游戏场景
    ✅ 生成3种防御塔（箭塔、炮塔、魔法塔）
    ✅ 生成5种敌人（史莱姆、哥布林、兽人、巨魔、飞龙）
    ✅ 生成波次系统（10波，难度递增）
    ✅ 生成UI界面（血条、金币、升级按钮）
    ✅ 评估质量（93分）
    
    "塔防游戏已创建完成！点击预览查看效果。"
    
用户: "箭塔的攻击速度太慢了"

AI: [调用modify()]
    ✅ 识别对象：arrow-tower
    ✅ 修改属性：attackSpeed从1.0 → 1.5
    
    "已提升箭塔攻击速度50%，请查看效果。"

用户: "很好！现在添加一个Boss关卡"

AI: [创建新场景]
    ✅ 生成Boss（巨型史莱姆王）
    ✅ 设计Boss技能（分裂、召唤小怪）
    ✅ 创建Boss专属场景
    ✅ 调整难度曲线
    
    "Boss关卡已添加！你的塔防游戏现在有11个关卡了。"

// 整个过程耗时: ~5分钟
// 传统开发: 2-3天
// 效率提升: 500-1000倍
```

---

### 5.3 提示词驱动的资产管理

```typescript
// ============================================
// Prompt-as-Source工作流
// ============================================

// 1. 初始创建（保存提示词）
const dragon = await quest.create({
  type: 'boss',
  species: 'dragon',
  threat: 'extreme',
});

await quest.savePrompt({
  id: 'dragon-boss',
  type: 'character',
  content: dragon.descriptor,
});

// 2. 迭代修改（版本控制）
// v1.0.0 → v1.1.0
await quest.modifyPrompt('dragon-boss', {
  change: '增加火焰喷吐技能',
});

// v1.1.0 → v1.2.0
await quest.modifyPrompt('dragon-boss', {
  change: '降低飞行速度',
});

// v1.2.0 → v1.3.0
await quest.modifyPrompt('dragon-boss', {
  change: '添加第二阶段（破甲后变狂暴）',
});

// 3. 版本对比
const diff = await quest.diffPrompts(
  'dragon-boss@v1.0.0',
  'dragon-boss@v1.3.0'
);

console.log(diff.changes);
// [
//   '+ 火焰喷吐技能',
//   '- 飞行速度: 6 → 4',
//   '+ 狂暴阶段'
// ]

// 4. 回滚版本
await quest.rollbackPrompt('dragon-boss', 'v1.1.0');
// 重新生成v1.1.0版本的龙

// 5. 分支开发
await quest.forkPrompt('dragon-boss@v1.3.0', 'ice-dragon');
await quest.modifyPrompt('ice-dragon', {
  change: '改为冰龙，使用寒冰技能',
});

// 现在有两个变体:
// - dragon-boss: 火龙
// - ice-dragon: 冰龙（fork自火龙）
```

---

### 5.4 Multi-Agent协作示例

```typescript
// ============================================
// 复杂任务的Multi-Agent协作
// ============================================

用户: "创建一个完整的RPG村庄场景，包括商店、任务、NPC对话"

// Quest内部流程:

// 1. MasterAgent分解任务
const plan = await masterAgent.decompose({
  task: '创建RPG村庄',
  requirements: ['商店', '任务', 'NPC对话'],
});

// plan = {
//   subtasks: [
//     { agent: 'scene', task: '创建村庄场景布局' },
//     { agent: 'npc', task: '设计5个NPC角色' },
//     { agent: 'dialogue', task: '生成NPC对话树' },
//     { agent: 'code', task: '实现商店系统代码' },
//   ]
// }

// 2. 并行执行（4个Agent同时工作）
const [sceneResult, npcResult, dialogueResult, codeResult] = 
  await Promise.all([
    sceneAgent.execute(plan.subtasks[0]),
    npcAgent.execute(plan.subtasks[1]),
    dialogueAgent.execute(plan.subtasks[2]),
    codeAgent.execute(plan.subtasks[3]),
  ]);

// 3. Agent间通信（通过黑板）
// NPCAgent告诉DialogueAgent: "我创建了铁匠NPC"
await blackboard.write('npc-list', npcResult.npcs);

// DialogueAgent读取并生成相应对话
const npcs = await blackboard.read('npc-list');
const dialogues = await dialogueAgent.generateForNPCs(npcs);

// 4. MasterAgent整合结果
const integrated = await masterAgent.integrate([
  sceneResult,
  npcResult,
  dialogueResult,
  codeResult,
]);

// 5. EvaluationAgent评估
const evaluation = await evalAgent.evaluate(integrated);

// evaluation = {
//   score: 89,
//   issues: [
//     { type: 'performance', message: 'NPC数量过多，建议减少' },
//   ],
//   suggestions: [
//     '将部分NPC改为静态装饰',
//   ]
// }

// 6. 自动优化（如果需要）
if (evaluation.score < 80) {
  const optimized = await optimizationAgent.optimize(
    integrated,
    evaluation.issues
  );
}

// 最终输出给用户
```

---

### 5.5 Skill开发示例

```typescript
// ============================================
// 开发自定义Skill
// ============================================

// 1. 创建Skill Markdown文件
// skills/custom/loot-system-generator.skill.md

/*
---
id: loot-system-generator
name: 战利品系统生成器
category: game-systems
version: 1.0.0
tools:
  - create_item
  - create_loot_table
  - balance_economy
---

# 战利品系统生成器

## 描述
根据游戏设计，生成完整的战利品掉落系统。

## 参数
- gameType: string - 游戏类型
- difficultyLevels: number - 难度级别数
- itemCategories: string[] - 物品分类

## Prompt

你是游戏经济系统设计专家。请设计战利品掉落系统：

游戏类型: {gameType}
难度级别: {difficultyLevels}
物品分类: {itemCategories}

生成以下内容:
1. 物品列表（JSON格式）
2. 掉落表（每个敌人的掉落配置）
3. 稀有度分布曲线
4. 经济平衡建议

使用MCP工具：
- create_item: 创建物品
- create_loot_table: 创建掉落表
- balance_economy: 平衡经济
*/

// 2. 注册Skill
await quest.registerSkill({
  path: './skills/custom/loot-system-generator.skill.md',
});

// 3. 使用Skill
const lootSystem = await quest.useSkill('loot-system-generator', {
  gameType: 'action-rpg',
  difficultyLevels: 5,
  itemCategories: ['weapon', 'armor', 'consumable', 'material'],
});

console.log(lootSystem);
// {
//   items: [
//     { id: 'sword-001', name: 'Iron Sword', rarity: 'common', ... },
//     { id: 'potion-001', name: 'Health Potion', rarity: 'common', ... },
//     // ... 100+ items
//   ],
//   lootTables: {
//     'slime': { 'potion-001': 0.3, 'material-001': 0.5 },
//     'goblin': { 'sword-001': 0.1, 'gold': 0.8 },
//     // ...
//   },
//   economyBalance: {
//     inflationRate: 'low',
//     recommendations: [...]
//   }
// }
```

---

### 5.6 混合模式开发

```typescript
// ============================================
// 语义化API + 传统代码混合开发
// ============================================

// 90%使用语义化API（AI生成）
const game = await quest.create({
  type: 'game',
  genre: 'platformer',
  
  scenes: [
    // AI生成场景
    await quest.createScene({
      prompt: '第一关：森林入口',
      difficulty: 'easy',
    }),
  ],
  
  characters: [
    // AI生成角色
    { type: 'character', class: 'ninja', abilities: ['dash', 'throw-shuriken'] },
  ],
});

// 10%使用传统代码（精确控制）
const customBehavior = quest.raw.createComponent({
  name: 'CustomBossAI',
  
  // 手写TypeScript代码
  update(dt: number) {
    // 精确的AI逻辑
    if (this.health < this.maxHealth * 0.3) {
      this.enterBerserkMode();
    }
    
    // 自定义攻击模式
    if (this.phaseTimer > 10) {
      this.switchAttackPattern();
    }
  },
});

// 附加到Boss
await quest.modify('boss-001', {
  overrides: {
    customAI: customBehavior,
  },
});

// 混合模式的优势:
// ✅ 大部分用AI生成（快）
// ✅ 关键逻辑手写（精确）
// ✅ 两者无缝集成
```

---

## 附录: TypeScript类型定义完整版

```typescript
// @quest/engine/types/index.d.ts

declare module '@quest/engine' {
  // === 核心API ===
  export namespace quest {
    function initialize(config: QuestConfig): Promise<void>;
    function create<T extends GameObjectType>(
      descriptor: GameObjectDescriptor<T>,
      options?: CreateOptions
    ): Promise<GameObject<T>>;
    
    function createScene(
      descriptor: SceneDescriptor,
      options?: SceneOptions
    ): Promise<Scene>;
    
    function modify(
      target: string | GameObject,
      changes: Partial<GameObjectDescriptor>,
      options?: ModifyOptions
    ): Promise<void>;
    
    function find(query: SemanticQuery): Promise<GameObject[]>;
    function delete(target: string | GameObject): Promise<void>;
    
    function generateAsset(prompt: AssetPrompt): Promise<Asset>;
    function savePrompt(prompt: PromptAsset): Promise<string>;
    function loadPrompt(promptId: string): Promise<PromptAsset>;
    
    // AI接口
    namespace ai {
      const chat: AIChat;
      const agent: AgentControl;
      const memory: MemoryControl;
    }
    
    // 语义化API
    namespace semantic {
      // 同quest.*，但更明确
    }
    
    // 直接API
    namespace api {
      function createNode(config: NodeConfig): Promise<cc.Node>;
      function addComponent<C>(node: cc.Node, type: ComponentType): C;
    }
    
    // 原生API
    namespace raw {
      export import cocos = cc;
    }
  }
  
  // === 类型导出 ===
  export * from '@quest/engine/types';
}
```

---

**文档版本**: v1.0.0  
**更新日期**: 2026-03-19  
**状态**: API设计规范
