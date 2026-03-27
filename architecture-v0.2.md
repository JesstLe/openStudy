# 负熵知识复利系统 (Negative Entropy)
## 架构设计文档 v0.2 - 集成研究成果

---

## 1. 系统愿景

> **"对抗遗忘的负熵武器"**

将学习对话转化为可复利增长的知识资产，通过语义图谱、间隔重复和语境注入，实现知识的持久化和自我强化。

---

## 2. 核心概念模型

### 2.1 知识单元 (KnowledgeUnit)
```typescript
interface KnowledgeUnit {
  id: string;                    // UUID v4
  type: 'concept' | 'code' | 'mechanism' | 'definition';
  title: string;                 // 实体名称
  content: string;               // 核心内容（Markdown）
  summary: string;               // AI生成的一行摘要
  source: {
    sessionId: string;           // 来源对话ID
    timestamp: number;           // 创建时间
    rawText: string;             // 原始对话片段
  };
  
  // 图谱连接
  relatedIds: string[];          // 关联知识ID
  embedding: number[];           // 向量嵌入（bge-base-zh, 768维）
  
  // 记忆属性
  mastery: number;               // 掌握度 0-1
  lastReviewed: number;          // 上次复习时间
  reviewCount: number;           // 复习次数
  
  // 元数据
  tags: string[];
  domain: string;                // 领域分类（如：C++、算法）
}
```

### 2.2 对话会话 (ChatSession)
```typescript
interface ChatSession {
  id: string;
  title: string;
  summary: string;               // AI生成的会话摘要
  messages: Message[];
  extractedUnits: string[];      // 提取的知识单元ID列表
  entropy: number;               // 会话熵值（信息密度评分）
  createdAt: number;
  updatedAt: number;
}

interface Message {
  id: string;
  role: 'user' | 'assistant' | 'system';
  content: string;
  timestamp: number;
  // 系统注入的上下文标记
  contextInjected?: boolean;
  contextSourceIds?: string[];
}
```

### 2.3 闪卡 (Flashcard)
```typescript
interface Flashcard {
  id: string;
  unitId: string;                // 关联的知识单元
  
  // SRS算法数据
  front: string;                 // 问题面
  back: string;                  // 答案面
  difficulty: 1|2|3|4|5;         // 难度等级
  tags: string[];                // 题型标签
  
  // FSRS参数（推荐算法）
  due: number;                   // 下次复习时间戳
  stability: number;             // 记忆稳定性
  difficulty: number;            // 卡片难度 (1-10)
  elapsedDays: number;           // 已经过去的天数
  scheduledDays: number;         // 计划间隔天数
  reps: number;                  // 复习次数
  lapses: number;                // 遗忘次数
  state: State;                  // 学习状态 (New/Learning/Review/Relearning)
  
  // 统计
  history: ReviewLog[];
}

type State = 0 | 1 | 2 | 3;      // 0=New, 1=Learning, 2=Review, 3=Relearning

interface ReviewLog {
  date: number;
  rating: Rating;                // 1=Again, 2=Hard, 3=Good, 4=Easy
  elapsedDays: number;
  scheduledDays: number;
}
```

### 2.4 语义链接 (SemanticLink)
```typescript
interface SemanticLink {
  sourceId: string;
  targetId: string;
  type: 'prerequisite' | 'related' | 'extends' | 'contrasts' | 'cooccurrence';
  weight: number;                // 关联强度 0-1
  explanation: string;           // 为什么有关联
  autoDetected: boolean;         // 是否自动检测
  detectionMethod: 'embedding' | 'cooccurrence' | 'manual';
  createdAt: number;
}
```

---

## 3. 技术架构决策

### 3.1 推荐技术栈：Tauri + React + Rust

基于研究分析，推荐采用 **Tauri + React + TypeScript** 技术栈：

**核心优势**：
- **轻量级**：安装包仅 8-15MB（Electron的1/10）
- **高性能**：Rust后端处理AI推理、embedding计算快2-3倍
- **完全本地**：SQLite本地存储，数据完全自主可控
- **AI集成**：通过 ollama-rs 可无缝调用本地LLM（Ollama）

**完整技术栈**：
```
前端层:
  - React 18 + TypeScript
  - TailwindCSS (实现设计系统)
  - React Flow (知识图谱可视化)
  - Zustand (状态管理)

桌面层:
  - Tauri 2.0 (Rust)
  
后端层 (Rust):
  - rusqlite (SQLite操作)
  - ollama-rs (本地LLM调用)
  - ndarray + faiss-rs (向量计算)
  - petgraph (图算法)

AI/ML:
  - Ollama (本地模型服务)
  - BGE-M3 (中文嵌入模型)
  - llama.cpp (可选本地推理)
```

### 3.2 存储方案：SQLite + Markdown混合

**双层存储架构**：

```
📁 KnowledgeVault/                          # 知识库根目录
├── 📄 .ne/                                  # 系统数据库
│   └── vault.db                             # SQLite主数据库
├── 📁 sessions/                             # 对话会话
│   ├── 2024-03-27_001.md                    # 原始对话记录
│   └── ...
├── 📁 units/                                # 知识单元
│   ├── concept_pointer.md
│   ├── code_smart-ptr.md
│   └── ...
├── 📁 flashcards/                           # 闪卡（可选Anki导出）
└── 📄 graph.json                            # 图谱关系缓存
```

**SQLite表结构**：

```sql
-- 知识单元主表
CREATE TABLE knowledge_units (
    id TEXT PRIMARY KEY,
    type TEXT NOT NULL,
    title TEXT NOT NULL,
    content TEXT,
    summary TEXT,
    source_session_id TEXT,
    source_timestamp INTEGER,
    embedding BLOB,              -- 序列化的向量 (768*4 bytes)
    mastery REAL DEFAULT 0,
    last_reviewed INTEGER,
    review_count INTEGER DEFAULT 0,
    domain TEXT,
    created_at INTEGER NOT NULL
);

-- 语义链接表（属性图模型）
CREATE TABLE semantic_links (
    source_id TEXT NOT NULL,
    target_id TEXT NOT NULL,
    type TEXT NOT NULL,
    weight REAL NOT NULL,
    explanation TEXT,
    auto_detected BOOLEAN DEFAULT 1,
    detection_method TEXT,
    created_at INTEGER,
    PRIMARY KEY (source_id, target_id, type),
    FOREIGN KEY (source_id) REFERENCES knowledge_units(id),
    FOREIGN KEY (target_id) REFERENCES knowledge_units(id)
);

-- 闪卡表（FSRS算法）
CREATE TABLE flashcards (
    id TEXT PRIMARY KEY,
    unit_id TEXT NOT NULL,
    front TEXT NOT NULL,
    back TEXT NOT NULL,
    difficulty INTEGER,
    due INTEGER NOT NULL,        -- 下次复习时间
    stability REAL,
    elapsed_days INTEGER DEFAULT 0,
    scheduled_days INTEGER DEFAULT 0,
    reps INTEGER DEFAULT 0,
    lapses INTEGER DEFAULT 0,
    state INTEGER DEFAULT 0,     -- 0=New, 1=Learning, 2=Review, 3=Relearning
    FOREIGN KEY (unit_id) REFERENCES knowledge_units(id)
);

-- 复习日志
CREATE TABLE review_logs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    card_id TEXT NOT NULL,
    rated_date INTEGER NOT NULL,
    rating INTEGER NOT NULL,     -- 1=Again, 2=Hard, 3=Good, 4=Easy
    elapsed_days INTEGER,
    scheduled_days INTEGER,
    FOREIGN KEY (card_id) REFERENCES flashcards(id)
);

-- FTS5全文搜索
CREATE VIRTUAL TABLE unit_search USING fts5(
    title, content, summary,
    content='knowledge_units',
    content_rowid='rowid'
);
```

---

## 4. 核心模块设计

### 4.1 知识提取引擎 (Knowledge Extraction)

**工作流程**：
```
对话文本 → 文本分块 → LLM提取 → 向量化 → 存储
```

**Prompt模板**：
```javascript
const EXTRACTION_PROMPT = `你是一个知识提取专家，擅长从学习对话中提取结构化知识。

任务：分析以下对话，提取所有值得长期记忆的知识点。

对于每个知识点，请输出：
{
  "units": [
    {
      "title": "简洁的概念名称",
      "type": "concept|code|mechanism|definition",
      "content": "详细解释（Markdown格式，包含代码块）",
      "summary": "一句话摘要",
      "difficulty": 1-5,
      "tags": ["相关标签"],
      "relatedConcepts": ["对话中提到的其他概念"]
    }
  ]
}

对话内容：
{conversation}

要求：
1. 只提取真正有价值的技术知识点
2. 代码必须保留完整的语法和注释
3. 难度评估基于概念深度和学习曲线
4. relatedConcepts用于构建知识图谱`;
```

**实现要点**：
- 使用 Ollama 调用本地模型（如 qwen2.5:7b 或 llama3.1:8b）
- 对话按主题自动分段，每段独立提取
- 提取后自动计算 BGE 嵌入向量
- 与现有知识库进行去重检测（相似度 > 0.9 视为重复）

### 4.2 语义图谱引擎 (Semantic Graph)

**链接发现策略**：

| 方法 | 触发条件 | 权重 | 说明 |
|------|----------|------|------|
| **共现链接** | 同一会话中共同出现 | 0.3 | 自动建立弱链接 |
| **嵌入相似度** | 余弦相似度 > 0.75 | 0.4 | 语义相近概念 |
| **显式链接** | 用户手动创建 | 0.9 | 双向链接 [[概念]] |
| **层级推导** | 传递闭包 | 0.2 | A→B, B→C 则 A→C |

**向量相似度计算**：
```rust
use faiss::IndexFlatIP;  // 内积 = 余弦相似度（归一化后）

// BGE-M3 生成的向量已经是归一化的
fn find_similar_units(query_embedding: &[f32], top_k: usize) -> Vec<( String, f32 )> {
    // 使用 HNSW 索引进行近似最近邻搜索
    // 返回 (unit_id, similarity_score)
}
```

**图可视化**：使用 React Flow 实现交互式知识图谱
- 节点：知识单元（颜色按领域区分）
- 边：语义链接（粗细表示权重）
- 交互：点击节点展开详情、拖拽重新布局、缩放导航

### 4.3 记忆综合器 - SRS模块 (Memory Synthesizer)

**算法选择：FSRS (Free Spaced Repetition Scheduler)**

相比经典SM-2算法，FSRS优势：
- 基于记忆稳定性建模，预测更准确
- 支持4级评分（Again/Hard/Good/Easy）
- 开源实现完整，社区活跃

**FSRS核心公式**：
```python
# 记忆稳定性更新
S'(r) = S * (e^(W_6 * (r - W_7)) + W_8 * (r - 0.5) + 1)

# 下次复习时间 = 当前稳定性 * 难度调整
next_interval = stability * difficulty_adjustment
```

**闪卡生成Prompt**：
```javascript
const FLASHCARD_PROMPT = `基于以下知识点，生成3-5个主动回忆闪卡。

知识点：
{unit_content}

要求：
1. 问题必须触发深层理解，避免简单事实记忆
2. 包含概念解释、代码分析、机制对比等类型
3. 难度分布：1个基础、2个中等、1-2个困难
4. 使用Anki格式输出

输出格式：
Q: [问题]
A: [答案]
Tags: [类型标签]

Q: ...`;
```

**复习调度UI**：
- 每日待复习卡片列表
- 卡片翻转动画（正面问题 → 背面答案）
- 四级评分按钮（Again/Hard/Good/Easy）
- 统计仪表盘（记忆保持率、学习曲线）

### 4.4 语境注入引擎 (Context Injection)

**工作流程**：
```
用户输入 → 查询向量化 → 向量检索 → 相关性排序 → Prompt组装 → 发送LLM
```

**分层检索策略**：
1. **核心层（高相关）**：相似度 > 0.85，最近7天的知识
2. **背景层（中相关）**：相似度 0.75-0.85，同一领域的知识
3. **参考层（低相关）**：相似度 0.65-0.75，相关但较旧的知识

**Prompt注入模板**（隐式注入）：
```javascript
const CONTEXT_INJECTION_TEMPLATE = `[系统指令]
你是用户的个人学习助手，熟悉用户的知识背景和学习历史。

[对话历史]
{recent_messages}

[相关背景]
以下是与当前问题相关的用户已学内容，请在回答中自然地联系这些知识：

{core_knowledge}  // 核心层，必须引用

{background_knowledge}  // 背景层，可选引用

[当前问题]
用户问：{user_query}

[回复要求]
- 基于用户的知识背景进行个性化解释
- 使用用户熟悉的概念进行类比
- 指出新知识与用户已有知识的联系`;
```

**注入内容格式化**：
```
▸ 智能指针（unique_ptr/shared_ptr）
  独占所有权 vs 引用计数，实现RAII自动内存管理

▸ 悬空指针问题  
  指向已释放内存的指针，现代C++应使用智能指针避免
```

---

## 5. 系统架构图

```
┌─────────────────────────────────────────────────────────────────────┐
│                         UI Layer (React)                            │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │
│  │  Chat    │ │Knowledge │ │  Graph   │ │ Review   │ │ Settings │  │
│  │ 对话界面 │ │  知识库  │ │  图谱    │ │  复习    │ │  设置    │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘  │
│                           Zustand Store                             │
└─────────────────────────────────────────────────────────────────────┘
                                  │
                         Tauri IPC (Commands)
                                  │
┌─────────────────────────────────────────────────────────────────────┐
│                      Backend Layer (Rust)                           │
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │  Extraction  │  │    Graph     │  │    RAG       │               │
│  │    Engine    │  │   Engine     │  │   Engine     │               │
│  │  知识提取    │  │  图谱管理    │  │  语境注入    │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │     SRS      │  │    Store     │  │    AI        │               │
│  │   Engine     │  │   (SQLite)   │  │   Client     │               │
│  │  间隔重复    │  │  数据持久化  │  │  (Ollama)    │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
                                  │
                    ┌─────────────┼─────────────┐
                    ▼             ▼             ▼
              ┌─────────┐   ┌─────────┐   ┌─────────┐
              │ SQLite  │   │Ollama   │   │ BGE-M3  │
              │  .db    │   │ (本地)  │   │(嵌入模型)│
              └─────────┘   └─────────┘   └─────────┘
```

---

## 6. MVP开发路线图

### Phase 1: 核心闭环 (2-3周)
**目标**：实现"对话 → 提取 → 存储 → 复习"的基础闭环

- [ ] 对话界面（复刻UI设计）
- [ ] 本地SQLite数据库初始化
- [ ] 基础知识提取（简单Prompt）
- [ ] 知识库列表和详情页
- [ ] Markdown文件导出

**交付物**：可记录学习对话并提取知识点的最小可用版本

### Phase 2: 连接与复习 (2-3周)
**目标**：建立知识关联，实现间隔重复

- [ ] BGE嵌入模型集成
- [ ] 语义相似度搜索
- [ ] 知识图谱可视化（基础版）
- [ ] FSRS算法实现
- [ ] 闪卡生成和复习界面

**交付物**：知识开始有连接，支持每日复习

### Phase 3: 智能化 (2-3周)
**目标**：实现语境注入，系统开始"理解"用户

- [ ] 语境自动注入功能
- [ ] 智能知识链接推荐
- [ ] 学习统计和熵值计算
- [ ] Anki格式导出
- [ ] 设置界面（模型配置、复习参数）

**交付物**：AI回答会自然引用用户的历史知识

### Phase 4: 优化与扩展 (持续)
- [ ] 性能优化（大数据量下的响应速度）
- [ ] 插件系统（允许自定义提取模板）
- [ ] 多端同步（可选WebDAV/iCloud）
- [ ] 社区分享（知识模板、闪卡组）

---

## 7. 关键决策确认

基于研究成果，以下关键决策已确定：

| 决策项 | 选择 | 理由 |
|--------|------|------|
| **技术栈** | Tauri + React + Rust | 轻量、高性能、完全本地 |
| **存储** | SQLite + Markdown | 关系查询 + 人类可读 |
| **SRS算法** | FSRS | 现代化、自适应、准确度高 |
| **嵌入模型** | BGE-M3 (本地) | 中文优化、隐私安全、无需API费用 |
| **本地LLM** | Ollama + qwen2.5:7b | 中文能力强、运行效率高 |
| **图谱可视化** | React Flow | React生态、交互丰富 |
| **向量搜索** | HNSW (faiss-rs) | 毫秒级百万级向量检索 |

---

## 8. 待确认问题

在启动开发前，还需确认：

1. **数据隐私级别**：是否需要端到端加密？
2. **同步需求**：是否需要云端同步？还是纯本地？
3. **开源策略**：是否开源？开源协议选择？
4. **AI模型偏好**：是否有偏好的本地模型？（qwen/llama/mistral等）
5. **导入导出**：需要支持哪些外部格式？（Anki、Obsidian、Notion等）

---

*文档状态：v0.2 已集成研究成果 - 等待SRS研究结果*
