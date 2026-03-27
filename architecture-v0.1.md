# 负熵知识复利系统 (Negative Entropy)
## 架构设计文档 v0.1 - 讨论稿

---

## 1. 系统愿景

> **"对抗遗忘的负熵武器"**

将学习对话转化为可复利增长的知识资产，通过语义图谱、间隔重复和语境注入，实现知识的持久化和自我强化。

---

## 2. 核心概念模型

### 2.1 知识单元 (Knowledge Unit)
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
  embeddings?: number[];         // 向量嵌入（用于语义搜索）
  
  // 记忆属性
  mastery: number;               // 掌握度 0-1
  lastReviewed: number;          // 上次复习时间
  reviewCount: number;           // 复习次数
  
  // 元数据
  tags: string[];
  domain: string;                // 领域分类（如：C++、算法）
}
```

### 2.2 对话会话 (Chat Session)
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
  
  // SM-2/FSRS参数
  interval: number;              // 当前间隔天数
  repetition: number;            // 重复次数
  easinessFactor: number;        // 简易度因子
  nextReviewDate: number;        // 下次复习时间
  
  // 统计
  history: ReviewLog[];
}
```

### 2.4 语义链接 (Semantic Link)
```typescript
interface SemanticLink {
  sourceId: string;
  targetId: string;
  type: 'prerequisite' | 'related' | 'extends' | 'contrasts';
  weight: number;                // 关联强度 0-1
  explanation: string;           // 为什么有关联
  autoDetected: boolean;         // 是否自动检测
}
```

---

## 3. 功能模块设计

### 3.1 知识提取引擎 (Knowledge Extraction Engine)

**职责**：从对话文本中自动识别和提取结构化知识

**流程**：
1. **文本分块**：将长对话分割为有意义的片段
2. **实体识别**：使用LLM识别关键概念、代码、机制
3. **结构化提取**：生成KnowledgeUnit
4. **向量化**：计算embedding用于语义搜索

**Prompt设计**（草稿）：
```
你是一个知识提取专家。分析以下学习对话，提取所有值得长期记忆的知识点。

对于每个知识点，输出：
- 标题：简洁的概念名称
- 类型：concept|code|mechanism|definition
- 内容：详细的解释（Markdown格式）
- 关键代码片段（如有）
- 相关概念（在对话中提到的其他概念）
- 难度评估：1-5

对话内容：
{conversation}

输出JSON格式：
{
  "units": [...]
}
```

### 3.2 语义图谱引擎 (Semantic Graph Engine)

**职责**：构建和维护知识之间的关联关系

**核心算法**：
1. **共现链接**：同一对话中提到的概念自动建立弱链接
2. **语义相似度**：基于embedding计算概念间相似度
3. **显式链接**：用户在UI中手动创建的链接（权重更高）
4. **层级推导**：通过传递闭包推断间接关系

**图遍历**：BFS/DFS用于知识导航和关联推荐

### 3.3 记忆综合器 (Memory Synthesizer) - SRS模块

**职责**：基于间隔重复算法管理复习调度

**算法选型**（待研究确认）：
- **SM-2**：经典算法，简单可靠
- **FSRS**：Free Spaced Repetition Scheduler，更现代的自适应算法

**闪卡生成Prompt**：
```
基于以下知识点，生成3-5个用于主动回忆的闪卡。

要求：
- 问题必须能触发深层理解，而非简单记忆
- 包含代码相关的概念性问题
- 难度分布：2个简单、2个中等、1个困难

知识点：{unit}

输出Anki格式：
Q: ...
A: ...
```

### 3.4 语境注入引擎 (Context Injection Engine)

**职责**：在用户发起新对话时，智能检索并注入相关历史知识

**工作流程**：
1. **查询分析**：提取用户新问题的关键词和意图
2. **向量检索**：在知识库中搜索语义相似的units
3. **相关性排序**：结合时间衰减和掌握度评分排序
4. **Prompt构建**：将相关知识格式化为隐性上下文

**Prompt注入模板**：
```
[系统知识背景]
用户之前学习过以下内容，在回答时请自然地将这些知识联系起来：

- {related_unit_1.title}: {related_unit_1.summary}
- {related_unit_2.title}: {related_unit_2.summary}
...

[用户问题]
{user_query}
```

---

## 4. 数据持久化设计（待定）

### 4.1 存储方案对比

| 方案 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| SQLite | 成熟、查询强大、单文件 | 图查询需手动实现 | 关系型数据为主 |
| JSON文件 | 简单、人类可读、版本友好 | 大数据量性能差 | 配置、小规模数据 |
| IndexedDB | 浏览器原生、容量大 | API复杂、无关系查询 | Web/PWA应用 |
| 图数据库(Neo4j) | 图查询强大 | 重量级、嵌入式复杂 | 大规模图谱 |

### 4.2 推荐方案（待讨论）
**Tauri + SQLite**:
- 使用SQLite存储核心实体数据（units, sessions, flashcards）
- 使用JSON文件存储图谱关系（便于版本控制）
- 在内存中构建图索引用于快速遍历

---

## 5. 技术栈方案（待确认）

### 方案A：Tauri + React + TypeScript
```
前端: React + TailwindCSS + 设计系统
桌面: Tauri (Rust)
存储: SQLite (通过tauri-sql插件)
AI: OpenAI/Claude API
可视化: D3.js / Cytoscape.js (知识图谱)
```

**优点**：
- 轻量级（<10MB）
- 原生性能
- 安全的本地文件系统访问
- 可使用Rust实现性能敏感模块

### 方案B：Next.js + PWA
```
全栈: Next.js (App Router)
存储: IndexedDB + localStorage
部署: Vercel / 本地静态导出
```

**优点**：
- 开发效率高
- 可Web可桌面
- 热更新方便

---

## 6. MVP功能范围（建议）

### Phase 1: 核心闭环
- [ ] 对话界面（类似已有UI）
- [ ] 知识提取（基础Prompt工程）
- [ ] 知识库列表和详情页
- [ ] 基础闪卡生成

### Phase 2: 连接与复习
- [ ] 语义相似度搜索
- [ ] 知识图谱可视化（基础版）
- [ ] SRS复习调度
- [ ] 每日复习提醒

### Phase 3: 智能化
- [ ] 语境自动注入
- [ ] 智能知识链接建议
- [ ] 学习统计和熵值计算
- [ ] 导出功能（Anki、Markdown）

---

## 7. 待讨论问题

1. **存储方案**：SQLite vs JSON vs 混合？
2. **SRS算法**：SM-2 vs FSRS？
3. **Embedding模型**：OpenAI vs 本地模型？
4. **图谱可视化**：D3.js vs Cytoscape.js vs 自研？
5. **AI提供商**：OpenAI vs Claude vs 本地LLM？
6. **是否开源**：个人工具 vs 开源社区？

---

*文档状态：草稿 - 等待技术研究结果*
