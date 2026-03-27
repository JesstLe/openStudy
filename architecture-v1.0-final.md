# 负熵知识复利系统 (Negative Entropy)
## 架构设计文档 v1.0 - 最终版

---

## 文档状态

| 项目 | 状态 |
|------|------|
| 版本 | v1.0 |
| 日期 | 2026-03-27 |
| 状态 | 架构冻结，待开发 |
| 作者 | Sisyphus + 多代理研究 |

---

## 1. 执行摘要

### 1.1 系统愿景

> **"对抗遗忘的负熵武器"**

将学习对话转化为可复利增长的知识资产，通过**语义图谱**、**间隔重复**和**语境注入**三大核心机制，实现知识的持久化和自我强化。

### 1.2 核心创新

1. **知识提取自动化**：AI自动从对话中提取结构化知识点
2. **语义网络构建**：基于嵌入向量自动发现知识关联
3. **FSRS记忆优化**：使用最先进的间隔重复算法
4. **语境隐式注入**：AI回复自然引用用户知识背景

### 1.3 目标用户

正在学习编程/技术的个人学习者，特别是：
- 每天进行AI辅助学习
- 希望建立长期知识积累
- 需要对抗遗忘曲线

---

## 2. 系统架构概览

### 2.1 技术栈决策（已确认）

| 层级 | 技术选型 | 理由 |
|------|----------|------|
| **UI层** | React 18 + TypeScript + TailwindCSS | 组件化、类型安全、设计系统友好 |
| **桌面层** | Tauri 2.0 (Rust) | 轻量(8-15MB)、高性能、完全本地 |
| **后端层** | Rust (rusqlite, ollama-rs) | 内存安全、AI推理快、原生性能 |
| **存储** | SQLite + Markdown | 关系查询 + 人类可读 + 版本控制友好 |
| **AI** | Ollama + qwen2.5:7b + BGE-M3 | 本地运行、中文优化、隐私安全 |
| **图谱可视化** | React Flow | React生态、交互丰富、性能优秀 |
| **SRS算法** | FSRS (ts-fsrs) | 现代化、自适应、预测准确 |

### 2.2 系统架构图

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
                    ┌─────────────┼─────────────┐
                    ▼             ▼             ▼
              ┌─────────┐   ┌─────────┐   ┌─────────┐
              │ SQLite  │   │Ollama   │   │ BGE-M3  │
              │  .db    │   │ (本地)  │   │(嵌入模型)│
              └─────────┘   └─────────┘   └─────────┘
```

---

## 3. 核心概念模型

### 3.1 实体关系图

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  ChatSession     │     │ KnowledgeUnit    │     │  Flashcard       │
├──────────────────┤     ├──────────────────┤     ├──────────────────┤
│ id: UUID         │◄────┤ id: UUID         │────►│ id: UUID         │
│ title: String    │     │ title: String    │     │ unit_id: UUID    │
│ messages: []     │     │ type: Enum       │     │ front: String    │
│ entropy: Float   │     │ content: MD      │     │ back: String     │
│ created_at: TS   │     │ embedding: []    │     │ srs_state: FSRS  │
│ updated_at: TS   │     │ mastery: Float   │     │ due: Date        │
└──────────────────┘     │ source: {}       │     └──────────────────┘
                         │ tags: []         │              │
                         └──────────────────┘              │
                                  │                        │
                                  │    ┌───────────────────┘
                                  │    │
                                  ▼    ▼
                         ┌──────────────────┐
                         │  SemanticLink    │
                         ├──────────────────┤
                         │ source_id: UUID  │
                         │ target_id: UUID  │
                         │ type: Enum       │
                         │ weight: Float    │
                         │ auto: Boolean    │
                         └──────────────────┘
```

### 3.2 数据模型定义

#### ChatSession（对话会话）

```typescript
interface ChatSession {
  id: string;                    // UUID v4
  title: string;                 // 会话标题（AI生成）
  summary: string;               // 会话摘要
  messages: Message[];           // 消息列表
  extractedUnits: string[];      // 提取的知识单元ID
  entropy: number;               // 信息密度评分 0-1
  createdAt: number;             // Unix timestamp
  updatedAt: number;
}

interface Message {
  id: string;
  role: 'user' | 'assistant' | 'system';
  content: string;
  timestamp: number;
  contextInjected?: boolean;     // 是否注入了上下文
  contextSourceIds?: string[];   // 注入的知识源ID
}
```

#### KnowledgeUnit（知识单元）

```typescript
interface KnowledgeUnit {
  id: string;                    // UUID v4
  type: 'concept' | 'code' | 'mechanism' | 'definition' | 'example';
  title: string;                 // 实体名称（如：智能指针）
  content: string;               // 核心内容（Markdown格式）
  summary: string;               // AI生成的一行摘要
  
  source: {
    sessionId: string;           // 来源对话ID
    timestamp: number;           // 创建时间
    rawText: string;             // 原始对话片段
  };
  
  // 语义连接
  embedding: number[];           // BGE-M3向量 (768维，已归一化)
  relatedIds: string[];          // 显式关联的知识ID
  
  // 记忆属性
  mastery: number;               // 掌握度 0-1
  lastReviewed: number;          // 上次复习时间
  reviewCount: number;           // 复习次数
  
  // 元数据
  tags: string[];
  domain: string;                // 领域（如：C++、算法、系统编程）
  difficulty: 1|2|3|4|5;         // 难度评估
  
  createdAt: number;
  updatedAt: number;
}
```

#### SemanticLink（语义链接）

```typescript
interface SemanticLink {
  id: string;
  sourceId: string;              // 源节点ID
  targetId: string;              // 目标节点ID
  type: LinkType;
  weight: number;                // 关联强度 0-1
  explanation: string;           // 关联解释（为什么相关）
  autoDetected: boolean;         // 是否自动检测
  detectionMethod: 'embedding' | 'cooccurrence' | 'manual';
  createdAt: number;
}

type LinkType = 
  | 'prerequisite'   // 前置知识
  | 'extends'        // 扩展
  | 'related'        // 相关
  | 'contrasts'      // 对比
  | 'cooccurrence'   // 共现
  | 'instance_of';   // 实例
```

#### Flashcard（闪卡）- FSRS数据模型

```typescript
interface Flashcard {
  id: string;
  unitId: string;                // 关联的知识单元
  
  // 卡片内容
  front: string;                 // 问题面（支持Markdown）
  back: string;                  // 答案面（支持Markdown）
  tags: string[];                // 题型标签
  
  // FSRS核心参数
  due: number;                   // 下次复习时间戳（核心字段）
  stability: number;             // 记忆稳定性（天）
  difficulty: number;            // 卡片难度 1-10
  elapsedDays: number;           // 已经过去的天数
  scheduledDays: number;         // 计划间隔天数
  reps: number;                  // 复习次数
  lapses: number;                // 遗忘次数（连续失败）
  state: CardState;              // 学习状态
  
  // 元数据
  generatedBy: 'ai' | 'manual';  // 生成方式
  generationPrompt?: string;     // 生成时使用的Prompt
  
  createdAt: number;
  updatedAt: number;
}

type CardState = 0 | 1 | 2 | 3;  // 0=New, 1=Learning, 2=Review, 3=Relearning

type ReviewRating = 1 | 2 | 3 | 4;  // 1=Again, 2=Hard, 3=Good, 4=Easy

interface ReviewLog {
  id: string;
  cardId: string;
  rating: ReviewRating;          // 用户评分
  reviewedAt: number;            // 复习时间
  reviewDurationMs: number;      // 复习用时（毫秒）
  
  // 复习前状态（用于分析）
  prevState: {
    due: number;
    stability: number;
    difficulty: number;
  };
  
  // 复习后状态
  nextState: {
    due: number;
    stability: number;
    difficulty: number;
    scheduledDays: number;
  };
}
```

---

## 4. 数据库设计

### 4.1 存储策略：SQLite + Markdown混合架构

```
📁 KnowledgeVault/                          # 知识库根目录
├── 📄 .ne/                                  # 系统数据库
│   ├── vault.db                             # SQLite主数据库
│   └── embeddings.idx                       # 向量索引（HNSW）
├── 📁 sessions/                             # 对话会话（原始记录）
│   ├── 2024-03-27-cpp-pointer.md            # Markdown格式
│   └── ...
├── 📁 units/                                # 知识单元（原始内容）
│   ├── concept_smart-pointer.md
│   ├── code_unique-ptr-example.md
│   └── ...
├── 📁 flashcards/                           # 闪卡（可导出Anki）
└── 📄 graph.json                            # 图谱关系缓存（快速加载）
```

### 4.2 SQLite Schema

```sql
-- ============================================
-- 核心表结构
-- ============================================

-- 对话会话表
CREATE TABLE chat_sessions (
    id TEXT PRIMARY KEY,
    title TEXT NOT NULL,
    summary TEXT,
    extracted_unit_ids TEXT,     -- JSON数组
    entropy REAL DEFAULT 0,
    created_at INTEGER NOT NULL,
    updated_at INTEGER NOT NULL
);

-- 消息表（可选，或存JSON文件）
CREATE TABLE messages (
    id TEXT PRIMARY KEY,
    session_id TEXT NOT NULL,
    role TEXT NOT NULL,          -- 'user' | 'assistant' | 'system'
    content TEXT NOT NULL,
    timestamp INTEGER NOT NULL,
    context_injected BOOLEAN DEFAULT FALSE,
    context_source_ids TEXT,     -- JSON数组
    FOREIGN KEY (session_id) REFERENCES chat_sessions(id)
);

-- ============================================
-- 知识图谱核心表
-- ============================================

CREATE TABLE knowledge_units (
    id TEXT PRIMARY KEY,
    type TEXT NOT NULL,          -- 'concept' | 'code' | 'mechanism' | 'definition'
    title TEXT NOT NULL,
    content TEXT,                -- Markdown格式
    summary TEXT,
    
    -- 来源信息
    source_session_id TEXT,
    source_timestamp INTEGER,
    source_raw_text TEXT,
    
    -- 嵌入向量（BLOB存储，768个float32 = 3072 bytes）
    embedding BLOB,
    
    -- 记忆属性
    mastery REAL DEFAULT 0,
    last_reviewed INTEGER,
    review_count INTEGER DEFAULT 0,
    
    -- 元数据
    tags TEXT,                   -- JSON数组
    domain TEXT,
    difficulty INTEGER,
    
    created_at INTEGER NOT NULL,
    updated_at INTEGER NOT NULL,
    
    FOREIGN KEY (source_session_id) REFERENCES chat_sessions(id)
);

-- 语义链接表（属性图模型）
CREATE TABLE semantic_links (
    id TEXT PRIMARY KEY,
    source_id TEXT NOT NULL,
    target_id TEXT NOT NULL,
    type TEXT NOT NULL,
    weight REAL NOT NULL,
    explanation TEXT,
    auto_detected BOOLEAN DEFAULT TRUE,
    detection_method TEXT,
    created_at INTEGER,
    
    FOREIGN KEY (source_id) REFERENCES knowledge_units(id),
    FOREIGN KEY (target_id) REFERENCES knowledge_units(id),
    UNIQUE(source_id, target_id, type)
);

-- ============================================
-- SRS闪卡表（FSRS算法）
-- ============================================

CREATE TABLE flashcards (
    id TEXT PRIMARY KEY,
    unit_id TEXT NOT NULL,
    
    -- 卡片内容
    front TEXT NOT NULL,
    back TEXT NOT NULL,
    tags TEXT,                   -- JSON数组
    
    -- FSRS核心状态
    due INTEGER NOT NULL,        -- 下次复习时间（Unix timestamp）
    stability REAL DEFAULT 0,    -- 记忆稳定性（天）
    difficulty REAL DEFAULT 0,   -- 难度 1-10
    elapsed_days INTEGER DEFAULT 0,
    scheduled_days INTEGER DEFAULT 0,
    reps INTEGER DEFAULT 0,
    lapses INTEGER DEFAULT 0,
    state INTEGER DEFAULT 0,     -- 0=New, 1=Learning, 2=Review, 3=Relearning
    
    -- 元数据
    generated_by TEXT DEFAULT 'ai',
    generation_prompt TEXT,
    
    created_at INTEGER NOT NULL,
    updated_at INTEGER NOT NULL,
    
    FOREIGN KEY (unit_id) REFERENCES knowledge_units(id)
);

-- 复习日志（用于算法优化和分析）
CREATE TABLE review_logs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    card_id TEXT NOT NULL,
    rating INTEGER NOT NULL,     -- 1=Again, 2=Hard, 3=Good, 4=Easy
    reviewed_at INTEGER NOT NULL,
    review_duration_ms INTEGER,
    
    -- 复习前状态
    prev_due INTEGER,
    prev_stability REAL,
    prev_difficulty REAL,
    
    -- 复习后状态
    next_due INTEGER,
    next_stability REAL,
    next_difficulty REAL,
    scheduled_days INTEGER,
    
    FOREIGN KEY (card_id) REFERENCES flashcards(id)
);

-- ============================================
-- 索引优化
-- ============================================

-- 知识单元搜索索引
CREATE INDEX idx_units_domain ON knowledge_units(domain);
CREATE INDEX idx_units_tags ON knowledge_units(tags);
CREATE INDEX idx_units_mastery ON knowledge_units(mastery);

-- 语义链接索引
CREATE INDEX idx_links_source ON semantic_links(source_id);
CREATE INDEX idx_links_target ON semantic_links(target_id);
CREATE INDEX idx_links_weight ON semantic_links(weight) WHERE weight > 0.5;

-- 闪卡核心索引（FSRS查询优化）
CREATE INDEX idx_cards_due ON flashcards(due);  -- 每日待复习查询
CREATE INDEX idx_cards_unit ON flashcards(unit_id);
CREATE INDEX idx_cards_state ON flashcards(state);

-- 复习日志索引
CREATE INDEX idx_logs_card ON review_logs(card_id, reviewed_at);
CREATE INDEX idx_logs_date ON review_logs(reviewed_at);

-- ============================================
-- FTS5全文搜索（知识单元内容搜索）
-- ============================================

CREATE VIRTUAL TABLE unit_search USING fts5(
    title, 
    content, 
    summary,
    content='knowledge_units',
    content_rowid='rowid'
);

-- 触发器：自动同步全文索引
CREATE TRIGGER unit_search_insert AFTER INSERT ON knowledge_units BEGIN
    INSERT INTO unit_search(rowid, title, content, summary) 
    VALUES (new.rowid, new.title, new.content, new.summary);
END;

CREATE TRIGGER unit_search_update AFTER UPDATE ON knowledge_units BEGIN
    UPDATE unit_search SET 
        title = new.title, 
        content = new.content, 
        summary = new.summary 
    WHERE rowid = new.rowid;
END;

CREATE TRIGGER unit_search_delete AFTER DELETE ON knowledge_units BEGIN
    DELETE FROM unit_search WHERE rowid = old.rowid;
END;
```

---

## 5. 核心模块设计

### 5.1 知识提取引擎

**职责**：从对话文本中自动识别和提取结构化知识

**工作流程**：
```
对话文本 → 分段 → LLM提取 → 向量化 → 去重检测 → 存储
```

**Prompt模板**：

```typescript
const KNOWLEDGE_EXTRACTION_PROMPT = `你是一位知识提取专家，擅长从学习对话中提取结构化知识。

## 任务
分析以下学习对话，提取所有值得长期记忆的知识点。

## 输出格式
返回JSON格式：
{
  "units": [
    {
      "title": "简洁的概念名称（如：智能指针）",
      "type": "concept|code|mechanism|definition|example",
      "content": "详细解释（Markdown格式，代码用代码块）",
      "summary": "一句话摘要",
      "difficulty": 1-5,
      "tags": ["相关标签"],
      "relatedConcepts": ["对话中提到的其他概念"]
    }
  ]
}

## 提取规则
1. 只提取真正有价值的技术知识点
2. 代码必须保留完整语法和注释
3. 概念需要包含：定义、原理、用途、示例
4. 机制需要包含：工作流程、关键步骤、注意事项
5. 难度评估基于概念深度和学习曲线
6. relatedConcepts用于构建知识图谱

## 对话内容
{conversation}

请提取最多10个最重要的知识点。`;
```

**实现伪代码**：

```rust
// Rust后端实现
pub struct KnowledgeExtractionEngine {
    ollama_client: OllamaClient,
    embedding_model: BGEM3,
}

impl KnowledgeExtractionEngine {
    pub async fn extract_from_session(
        &self, 
        session: &ChatSession
    ) -> Result<Vec<KnowledgeUnit>, Error> {
        // 1. 分段处理长对话
        let segments = self.segment_conversation(&session.messages);
        
        let mut units = Vec::new();
        
        for segment in segments {
            // 2. LLM提取
            let prompt = KNOWLEDGE_EXTRACTION_PROMPT
                .replace("{conversation}", &segment);
            
            let response = self.ollama_client
                .generate(&prompt, "qwen2.5:7b")
                .await?;
            
            let extracted: ExtractedUnits = serde_json::from_str(&response)?;
            
            // 3. 向量化
            for unit_data in extracted.units {
                let content = format!("{} {}", unit_data.title, unit_data.content);
                let embedding = self.embedding_model.encode(&content).await?;
                
                // 4. 去重检测
                let similar = self.find_similar(&embedding, 0.9).await?;
                if similar.is_empty() {
                    let unit = KnowledgeUnit::new(unit_data, embedding, session.id);
                    units.push(unit);
                }
            }
        }
        
        Ok(units)
    }
}
```

### 5.2 语义图谱引擎

**职责**：构建和维护知识之间的关联关系

**链接发现策略**：

| 方法 | 权重 | 触发条件 | 计算方式 |
|------|------|----------|----------|
| 嵌入相似度 | 0.5 | 余弦相似度 > 0.75 | embedding_cosine_similarity |
| 共现链接 | 0.3 | 同一会话出现 | 共现频率 |
| 显式链接 | 1.0 | 用户手动创建 | [[WikiLink]] 解析 |
| 传递推导 | 0.2 | A→B, B→C | 图遍历 |

**向量相似度计算（Rust）**：

```rust
use faiss::{IndexFlatIP, IndexHNSWFlat};

pub struct SemanticGraphEngine {
    embedding_index: IndexHNSWFlat,  // HNSW索引，支持百万级向量
    units: HashMap<String, KnowledgeUnit>,
    links: Vec<SemanticLink>,
}

impl SemanticGraphEngine {
    /// 查找相似的知识单元
    pub fn find_similar(
        &self, 
        query_embedding: &[f32], 
        threshold: f32,
        top_k: usize
    ) -> Vec<(String, f32)> {
        // BGE-M3向量已归一化，内积 = 余弦相似度
        let distances = self.embedding_index.search(query_embedding, top_k);
        
        distances.into_iter()
            .filter(|(_, dist)| *dist >= threshold)
            .map(|(id, dist)| (id.to_string(), dist))
            .collect()
    }
    
    /// 自动发现链接
    pub async fn auto_discover_links(&mut self, unit: &KnowledgeUnit) {
        // 1. 嵌入相似度链接
        let similar = self.find_similar(&unit.embedding, 0.75, 10);
        for (other_id, similarity) in similar {
            if other_id != unit.id {
                let link = SemanticLink {
                    source_id: unit.id.clone(),
                    target_id: other_id,
                    link_type: LinkType::Related,
                    weight: similarity,
                    auto_detected: true,
                    detection_method: "embedding",
                };
                self.links.push(link);
            }
        }
        
        // 2. 共现链接（同一会话）
        let cooccurring = self.find_cooccurring_in_session(&unit.source.session_id);
        for other in cooccurring {
            if other.id != unit.id {
                let link = SemanticLink {
                    source_id: unit.id.clone(),
                    target_id: other.id,
                    link_type: LinkType::Cooccurrence,
                    weight: 0.3,
                    auto_detected: true,
                    detection_method: "cooccurrence",
                };
                self.links.push(link);
            }
        }
    }
    
    /// 获取知识网络（用于可视化）
    pub fn get_knowledge_network(
        &self, 
        center_id: &str, 
        depth: usize
    ) -> KnowledgeNetwork {
        // BFS遍历获取子图
        let mut visited = HashSet::new();
        let mut nodes = Vec::new();
        let mut edges = Vec::new();
        let mut queue = VecDeque::new();
        
        queue.push_back((center_id.to_string(), 0));
        visited.insert(center_id.to_string());
        
        while let Some((current_id, current_depth)) = queue.pop_front() {
            if current_depth > depth {
                break;
            }
            
            if let Some(unit) = self.units.get(&current_id) {
                nodes.push(unit.clone());
                
                // 获取相关链接
                let related = self.links.iter()
                    .filter(|l| l.source_id == current_id || l.target_id == current_id)
                    .filter(|l| l.weight > 0.5)  // 只取强链接
                    .take(5);  // 限制每个节点的链接数
                
                for link in related {
                    edges.push(link.clone());
                    let neighbor_id = if link.source_id == current_id {
                        &link.target_id
                    } else {
                        &link.source_id
                    };
                    
                    if !visited.contains(neighbor_id) {
                        visited.insert(neighbor_id.clone());
                        queue.push_back((neighbor_id.clone(), current_depth + 1));
                    }
                }
            }
        }
        
        KnowledgeNetwork { nodes, edges }
    }
}
```

### 5.3 记忆综合器（SRS模块）

**职责**：基于FSRS算法管理闪卡复习调度

**FSRS核心实现**：

```typescript
// 使用官方 ts-fsrs 库
import { createFSRS, Card, Rating, State } from 'ts-fsrs';

export class SRSEngine {
  private fsrs = createFSRS({
    w: [0.4, 0.6, 2.4, 2.2, 4.2, 0.5, 0.5, 1.2], // 默认优化权重
    enableFuzz: true,  // 启用模糊化，避免卡片聚集
  });

  /// 获取今日待复习卡片
  async getDueCards(userId: string): Promise<Flashcard[]> {
    const now = Date.now();
    
    return db.flashcards
      .where('due')
      .belowOrEqual(now)
      .and(card => card.state !== State.New)
      .sortBy('due');
  }

  /// 提交复习评分
  async submitReview(
    cardId: string, 
    rating: Rating,
    reviewTimeMs: number
  ): Promise<void> {
    const card = await db.flashcards.get(cardId);
    if (!card) throw new Error('Card not found');
    
    const now = new Date();
    
    // 调用FSRS算法计算新状态
    const result = this.fsrs.repeat(card, now);
    const newCardState = result[rating].card;
    
    // 保存复习日志
    await db.reviewLogs.add({
      cardId,
      rating,
      reviewedAt: now.getTime(),
      reviewDurationMs: reviewTimeMs,
      prevState: {
        due: card.due,
        stability: card.stability,
        difficulty: card.difficulty,
      },
      nextState: {
        due: newCardState.due.getTime(),
        stability: newCardState.stability,
        difficulty: newCardState.difficulty,
        scheduledDays: result[rating].scheduledDays,
      },
    });
    
    // 更新卡片状态
    await db.flashcards.update(cardId, {
      due: newCardState.due.getTime(),
      stability: newCardState.stability,
      difficulty: newCardState.difficulty,
      elapsedDays: newCardState.elapsedDays,
      scheduledDays: result[rating].scheduledDays,
      reps: card.reps + 1,
      state: newCardState.state,
      updatedAt: now.getTime(),
    });
    
    // 如果评分是Again，增加遗忘计数
    if (rating === Rating.Again) {
      await db.flashcards.update(cardId, {
        lapses: card.lapses + 1,
      });
    }
  }

  /// 生成学习统计
  async getLearningStats(userId: string, days: number = 30): Promise<Stats> {
    const startDate = Date.now() - days * 24 * 60 * 60 * 1000;
    
    const logs = await db.reviewLogs
      .where('reviewedAt')
      .above(startDate)
      .toArray();
    
    const totalReviews = logs.length;
    const correctReviews = logs.filter(l => l.rating >= Rating.Good).length;
    const retentionRate = totalReviews > 0 ? correctReviews / totalReviews : 0;
    
    const dailyStats = this.aggregateByDay(logs);
    
    return {
      totalReviews,
      retentionRate,
      dailyStats,
      // ...更多统计
    };
  }
}
```

**闪卡生成Prompt**：

```typescript
const FLASHCARD_GENERATION_PROMPT = `你是一位教育专家，擅长将知识点转化为高效的闪卡。

## 任务
基于以下知识点，生成3-5个主动回忆闪卡。

## 知识点
{unit_content}

## 闪卡设计要求

### 问题类型分布
1. **概念定义型**："什么是X？" 或 "X的定义是？"
2. **机制解释型**："X的工作原理是什么？" 或 "X是如何实现的？"
3. **对比分析型**："X和Y的区别是什么？" 或 "为什么选择X而不是Y？"
4. **代码分析型**："这段代码的输出是什么？" 或 "解释这段代码的作用"
5. **应用场景型**："在什么情况下使用X？" 或 "X的典型应用场景是？"

### 质量标准
- 问题必须具体、可回答
- 答案必须准确、完整
- 避免可以从问题中直接猜到答案的提示
- 代码示例保持语法正确
- 难度分布：1个基础 + 2个中等 + 1-2个困难

## 输出格式
返回JSON：
{
  "flashcards": [
    {
      "front": "问题（简洁明了）",
      "back": "答案（准确完整）",
      "type": "concept|mechanism|comparison|code|application",
      "difficulty": 1-5,
      "tags": ["标签"]
    }
  ]
}`;
```

### 5.4 语境注入引擎（RAG模块）

**职责**：在用户发起新对话时，智能检索并注入相关历史知识

**分层检索策略**：

```
用户输入
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│ 第一层：核心知识（高相关）                                │
│ - 相似度 > 0.85                                          │
│ - 最近30天的知识                                          │
│ - 同一领域的知识                                          │
│ - 注入：必须引用                                          │
└─────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│ 第二层：背景知识（中相关）                                │
│ - 相似度 0.75-0.85                                       │
│ - 相关但较旧的知识                                        │
│ - 前置概念（通过图谱链接发现）                            │
│ - 注入：可选引用                                          │
└─────────────────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│ 第三层：参考知识（低相关）                                │
│ - 相似度 0.65-0.75                                       │
│ - 仅当上下文窗口充足时注入                                │
└─────────────────────────────────────────────────────────┘
```

**Prompt注入模板**（隐式注入）：

```typescript
const CONTEXT_INJECTION_TEMPLATE = `[系统指令]
你是用户的个人学习助手，熟悉用户的知识背景和学习历史。

[对话历史摘要]
{recent_conversation_summary}

[相关背景知识]
以下是与当前问题相关的用户已学内容，请在回答中自然地联系这些知识：

{core_knowledge}

{background_knowledge}

[用户当前问题]
{user_query}

[回复要求]
- 基于用户的知识背景进行个性化解释
- 使用用户熟悉的概念进行类比和连接
- 指出新知识与用户已有知识的联系
- 如果发现用户已有知识中存在误解，请温和地纠正`;

/// 格式化注入内容
function formatKnowledgeForInjection(
  units: KnowledgeUnit[],
  level: 'core' | 'background'
): string {
  const header = level === 'core' 
    ? '▸ 核心相关知识\n'
    : '▸ 扩展背景知识\n';
  
  const formatted = units.map(u => {
    const mastery = u.mastery > 0.8 ? '（已掌握）' : 
                    u.mastery > 0.5 ? '（熟悉）' : '（学习中）';
    return `▸ ${u.title}${mastery}\n  ${u.summary}`;
  }).join('\n\n');
  
  return header + formatted;
}
```

**实现逻辑**：

```rust
pub struct ContextInjectionEngine {
    embedding_model: BGEM3,
    graph_engine: Arc<SemanticGraphEngine>,
    db: Database,
}

impl ContextInjectionEngine {
    /// 为用户查询构建上下文注入
    pub async fn build_context(
        &self,
        user_query: &str,
        current_session_id: &str,
    ) -> Result<ContextPayload, Error> {
        // 1. 查询向量化
        let query_embedding = self.embedding_model.encode(user_query).await?;
        
        // 2. 第一层检索：向量相似度
        let core_candidates = self.graph_engine
            .find_similar(&query_embedding, 0.75, 20)
            .await?;
        
        // 3. 过滤和排序
        let mut core_units = Vec::new();
        let mut background_units = Vec::new();
        
        for (unit_id, similarity) in core_candidates {
            let unit = self.db.knowledge_units.get(&unit_id).await?;
            
            // 时间衰减因子
            let days_old = (now() - unit.created_at) / (24 * 60 * 60 * 1000);
            let time_decay = (-0.01 * days_old as f32).exp();  // 30天后衰减到0.74
            
            // 掌握度调整（优先推荐未完全掌握的知识）
            let mastery_bonus = 1.0 - unit.mastery * 0.5;  // 未掌握的知识得分更高
            
            let adjusted_score = similarity * time_decay * mastery_bonus;
            
            if adjusted_score > 0.6 {
                core_units.push((unit, adjusted_score));
            } else if adjusted_score > 0.4 {
                background_units.push((unit, adjusted_score));
            }
        }
        
        // 4. 第二层检索：图谱扩展
        let graph_extended = self.expand_via_graph(&core_units, 1).await?;
        background_units.extend(graph_extended);
        
        // 5. 去重和限制数量
        core_units.truncate(3);      // 最多3个核心知识
        background_units.truncate(5); // 最多5个背景知识
        
        // 6. 构建Prompt
        let core_text = format_knowledge_list(&core_units.iter().map(|(u, _)| u).collect::<Vec<_>>()
        );
        let background_text = format_knowledge_list(
            &background_units.iter().map(|(u, _)| u).collect::<Vec<_>>()
        );
        
        let prompt = CONTEXT_INJECTION_TEMPLATE
            .replace("{core_knowledge}", &core_text)
            .replace("{background_knowledge}", &background_text)
            .replace("{user_query}", user_query);
        
        Ok(ContextPayload {
            prompt,
            injected_unit_ids: core_units.iter()
                .map(|(u, _)| u.id.clone())
                .collect(),
        })
    }
    
    /// 通过图谱扩展发现相关知识
    async fn expand_via_graph(
        &self,
        seed_units: &[(KnowledgeUnit, f32)],
        depth: usize,
    ) -> Result<Vec<(KnowledgeUnit, f32)>, Error> {
        let mut extended = Vec::new();
        let mut visited: HashSet<String> = seed_units.iter()
            .map(|(u, _)| u.id.clone())
            .collect();
        
        for (unit, base_score) in seed_units {
            // 获取直接关联
            let links = self.graph_engine.get_links(&unit.id).await?;
            
            for link in links.iter().filter(|l| l.weight > 0.5) {
                let neighbor_id = if link.source_id == unit.id {
                    &link.target_id
                } else {
                    &link.source_id
                };
                
                if !visited.contains(neighbor_id) {
                    visited.insert(neighbor_id.clone());
                    
                    if let Ok(neighbor) = self.db.knowledge_units.get(neighbor_id).await {
                        // 图谱链接的权重衰减
                        let graph_score = base_score * link.weight * 0.7;
                        extended.push((neighbor, graph_score));
                    }
                }
            }
        }
        
        Ok(extended)
    }
}
```

---

## 6. 前端模块设计

### 6.1 页面路由

| 路由 | 页面 | 功能描述 |
|------|------|----------|
| `/` | Dashboard | 主仪表盘，熵值、统计、今日待办 |
| `/chat` | Chat | 对话界面，支持语境注入 |
| `/chat/:id` | ChatSession | 具体对话会话 |
| `/units` | KnowledgeBase | 知识库列表，支持搜索筛选 |
| `/unit/:id` | KnowledgeDetail | 知识详情，显示关联 |
| `/graph` | KnowledgeGraph | 知识图谱可视化 |
| `/review` | Review | 每日复习界面 |
| `/settings` | Settings | 系统设置、AI配置 |

### 6.2 状态管理（Zustand）

```typescript
// stores/knowledgeStore.ts
interface KnowledgeState {
  units: KnowledgeUnit[];
  links: SemanticLink[];
  selectedUnitId: string | null;
  
  // Actions
  loadUnits: () => Promise<void>;
  addUnit: (unit: KnowledgeUnit) => void;
  updateUnit: (id: string, updates: Partial<KnowledgeUnit>) => void;
  selectUnit: (id: string | null) => void;
  searchUnits: (query: string) => Promise<KnowledgeUnit[]>;
}

export const useKnowledgeStore = create<KnowledgeState>((set, get) => ({
  units: [],
  links: [],
  selectedUnitId: null,
  
  loadUnits: async () => {
    const units = await invoke<KnowledgeUnit[]>('get_all_units');
    set({ units });
  },
  
  addUnit: (unit) => {
    set((state) => ({ units: [...state.units, unit] }));
  },
  
  searchUnits: async (query) => {
    return await invoke('search_units', { query });
  },
  
  // ...
}));

// stores/reviewStore.ts
interface ReviewState {
  dueCards: Flashcard[];
  currentCardIndex: number;
  sessionStats: {
    total: number;
    correct: number;
    timeSpent: number;
  };
  
  loadDueCards: () => Promise<void>;
  submitReview: (rating: Rating, timeMs: number) => Promise<void>;
  skipCard: () => void;
}

export const useReviewStore = create<ReviewState>((set, get) => ({
  dueCards: [],
  currentCardIndex: 0,
  sessionStats: { total: 0, correct: 0, timeSpent: 0 },
  
  loadDueCards: async () => {
    const cards = await invoke<Flashcard[]>('get_due_cards');
    set({ dueCards: cards, currentCardIndex: 0 });
  },
  
  submitReview: async (rating, timeMs) => {
    const { dueCards, currentCardIndex, sessionStats } = get();
    const currentCard = dueCards[currentCardIndex];
    
    await invoke('submit_review', {
      cardId: currentCard.id,
      rating,
      reviewTimeMs: timeMs,
    });
    
    set({
      currentCardIndex: currentCardIndex + 1,
      sessionStats: {
        ...sessionStats,
        total: sessionStats.total + 1,
        correct: rating >= 3 ? sessionStats.correct + 1 : sessionStats.correct,
        timeSpent: sessionStats.timeSpent + timeMs,
      },
    });
  },
  
  // ...
}));
```

### 6.3 图谱可视化（React Flow）

```tsx
// components/KnowledgeGraph.tsx
import ReactFlow, {
  Background,
  Controls,
  MiniMap,
  Node,
  Edge,
  useNodesState,
  useEdgesState,
} from 'reactflow';
import 'reactflow/dist/style.css';

interface KnowledgeGraphProps {
  centerUnitId: string;
  depth?: number;
}

export const KnowledgeGraph: React.FC<KnowledgeGraphProps> = ({
  centerUnitId,
  depth = 2,
}) => {
  const [nodes, setNodes, onNodesChange] = useNodesState([]);
  const [edges, setEdges, onEdgesChange] = useEdgesState([]);
  
  useEffect(() => {
    loadGraph(centerUnitId, depth);
  }, [centerUnitId, depth]);
  
  const loadGraph = async (centerId: string, depth: number) => {
    const network = await invoke<KnowledgeNetwork>('get_knowledge_network', {
      centerId,
      depth,
    });
    
    // 转换为React Flow格式
    const flowNodes: Node[] = network.nodes.map((unit, index) => ({
      id: unit.id,
      data: { 
        label: unit.title,
        summary: unit.summary,
        type: unit.type,
      },
      position: calculatePosition(index, network.nodes.length),
      style: {
        background: getNodeColor(unit.type),
        border: '1px solid #00FF41',
        color: '#fff',
        fontFamily: 'Space Grotesk',
      },
    }));
    
    const flowEdges: Edge[] = network.edges.map((link) => ({
      id: link.id,
      source: link.sourceId,
      target: link.targetId,
      animated: link.autoDetected,
      style: {
        stroke: getEdgeColor(link.type),
        strokeWidth: link.weight * 3,
      },
      label: link.type,
    }));
    
    setNodes(flowNodes);
    setEdges(flowEdges);
  };
  
  const onNodeClick = (event: React.MouseNodeEvent, node: Node) => {
    // 加载该节点的知识详情
  };
  
  return (
    <div style={{ width: '100%', height: '600px' }}>
      <ReactFlow
        nodes={nodes}
        edges={edges}
        onNodesChange={onNodesChange}
        onEdgesChange={onEdgesChange}
        onNodeClick={onNodeClick}
        fitView
      >
        <Background color="#3b4b37" gap={20} size={1} />
        <Controls />
        <MiniMap 
          nodeColor={(node) => getNodeColor(node.data?.type)}
          maskColor="rgba(0, 0, 0, 0.8)"
        />
      </ReactFlow>
    </div>
  );
};

// 辅助函数
const calculatePosition = (index: number, total: number) => {
  const angle = (index / total) * 2 * Math.PI;
  const radius = 200 + (index % 3) * 100;
  return {
    x: Math.cos(angle) * radius + 400,
    y: Math.sin(angle) * radius + 300,
  };
};

const getNodeColor = (type: string) => {
  switch (type) {
    case 'concept': return '#004a77';
    case 'code': return '#00530e';
    case 'mechanism': return '#604100';
    default: return '#201F1F';
  }
};

const getEdgeColor = (type: string) => {
  switch (type) {
    case 'prerequisite': return '#00FF41';  // 绿色
    case 'extends': return '#00A2FD';       // 蓝色
    case 'related': return '#FFBA38';       // 琥珀色
    default: return '#84967e';
  }
};
```

---

## 7. MVP开发路线图

### Phase 1: 核心闭环（2-3周）
**目标**：实现"对话 → 提取 → 存储 → 浏览"的基础闭环

- [x] 技术选型确认
- [x] 架构设计完成
- [ ] 项目初始化（Tauri + React + Rust）
- [ ] SQLite数据库初始化
- [ ] 对话界面（复刻UI设计）
- [ ] 简单知识提取（硬编码Prompt）
- [ ] 知识库列表页
- [ ] Markdown文件导出

**验收标准**：
- 可以进行对话
- AI能提取知识点并显示
- 知识点能在列表中查看

### Phase 2: 连接与复习（2-3周）
**目标**：建立知识关联，实现间隔重复

- [ ] BGE-M3嵌入模型集成
- [ ] 语义相似度搜索
- [ ] 知识图谱可视化（基础版）
- [ ] ts-fsrs库集成
- [ ] 闪卡生成Prompt
- [ ] 每日复习界面

**验收标准**：
- 知识之间有连线显示
- 每日能看到待复习卡片
- 复习后调度正确

### Phase 3: 智能化（2-3周）
**目标**：实现语境注入，系统开始"理解"用户

- [ ] 语境自动注入功能
- [ ] 智能知识链接推荐
- [ ] 学习统计和熵值计算
- [ ] Anki格式导出
- [ ] 设置界面（模型配置、复习参数）

**验收标准**：
- AI回复会自然引用历史知识
- 系统熵值可计算和显示
- 可导出到Anki

### Phase 4: 优化与扩展（持续）
- [ ] 性能优化（大数据量响应）
- [ ] 自定义提取模板
- [ ] 云端同步（可选）
- [ ] 社区分享功能

---

## 8. 部署与配置

### 8.1 本地开发环境

```bash
# 1. 安装依赖
# - Node.js 18+
# - Rust 1.70+
# - Ollama (本地LLM)

# 2. 克隆项目
git clone https://github.com/user/negative-entropy.git
cd negative-entropy

# 3. 安装前端依赖
cd src-ui
npm install

# 4. 安装后端依赖
cd ../src-tauri
cargo build

# 5. 启动Ollama并拉取模型
ollama pull qwen2.5:7b
ollama pull bge-m3

# 6. 启动开发服务器
cd ..
npm run tauri dev
```

### 8.2 配置文件

```json
// ~/.config/negative-entropy/config.json
{
  "ai": {
    "provider": "ollama",
    "model": "qwen2.5:7b",
    "embeddingModel": "bge-m3",
    "apiUrl": "http://localhost:11434"
  },
  "srs": {
    "algorithm": "fsrs",
    "newCardsPerDay": 20,
    "reviewsPerDay": 100,
    "targetRetention": 0.9
  },
  "ui": {
    "theme": "dark",
    "language": "zh-CN",
    "fontSize": 14
  },
  "sync": {
    "enabled": false,
    "provider": null
  }
}
```

---

## 9. 风险评估与缓解

| 风险 | 影响 | 可能性 | 缓解措施 |
|------|------|--------|----------|
| **本地LLM性能不足** | 高 | 中 | 支持云端API回退；优化Prompt减少token；使用更小模型 |
| **嵌入向量存储过大** | 中 | 中 | 使用量化压缩；定期清理低质量知识；分页加载 |
| **初次冷启动慢** | 中 | 高 | 预加载关键数据；懒加载非关键模块；显示加载进度 |
| **数据丢失** | 高 | 低 | 自动备份；版本控制；导出功能；SQLite WAL模式 |
| **图谱可视化卡顿** | 中 | 中 | 限制节点数量；使用Canvas渲染；分层加载 |
| **SRS算法初期不准确** | 低 | 高 | 使用默认参数；积累足够数据后优化；允许用户调整 |

---

## 10. 附录

### 10.1 参考资源

- **FSRS算法**：https://github.com/open-spaced-repetition/fsrs4anki
- **ts-fsrs库**：https://github.com/open-spaced-repetition/ts-fsrs
- **BGE嵌入模型**：https://huggingface.co/BAAI/bge-m3
- **Tauri文档**：https://tauri.app/
- **React Flow**：https://reactflow.dev/
- **Ollama**：https://ollama.com/

### 10.2 术语表

| 术语 | 说明 |
|------|------|
| **负熵** | 对抗知识遗忘的系统目标 |
| **SRS** | Spaced Repetition System，间隔重复系统 |
| **FSRS** | Free Spaced Repetition Scheduler，现代间隔重复算法 |
| **Embedding** | 文本的向量表示，用于语义计算 |
| **RAG** | Retrieval-Augmented Generation，检索增强生成 |
| **HNSW** | Hierarchical Navigable Small World，高效向量索引算法 |

### 10.3 性能指标目标

| 指标 | 目标值 | 说明 |
|------|--------|------|
| 冷启动时间 | < 2s | 应用启动到可操作 |
| 知识提取延迟 | < 5s | 对话到知识提取完成 |
| 向量搜索延迟 | < 100ms | 千级知识库搜索 |
| 图谱渲染帧率 | > 30fps | 百级节点流畅交互 |
| 闪卡复习加载 | < 500ms | 下一张卡片显示 |
| 安装包大小 | < 50MB | 包含所有依赖 |

---

*文档结束 - 负熵知识复利系统 v1.0 架构设计*
