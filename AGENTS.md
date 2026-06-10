# 项目知识库

**生成时间:** 2026-03-27
**Commit:** 0fda011
**分支:** main

## 概述

**负熵知识复利系统 (Negative Entropy)** 的纯文档仓库。无源代码，全部文件为架构设计文档和规划物料。计划开发的 iOS 应用将把学习对话转化为语义图谱、FSRS 闪卡和 RAG 语境注入的持久记忆。

## 当前状态

**预实现阶段。** 无 Xcode 工程、无 Swift 代码、无测试、无 CI/CD。仅包含 Markdown 文档和 MIT LICENSE。

## 仓库结构

```
openStudy/
├── architecture-v2.0-ios.md     # 当前活跃架构 (SwiftUI + SwiftData) — 从这里开始
├── architecture-v1.1-updates.md # AI 揢供商配置、同步模块、开源策略
├── architecture-v1.0-final.md   # 原始 Tauri+React+Rust 桌面方案（已废弃）
├── architecture-v0.2.md         # 早期迭代，仅作历史参考
├── architecture-v0.1.md         # 最早草稿，仅作历史参考
├── PROJECT_CONTEXT.md           # 项目愿景、路线图、技术选型
├── README.md                    # 公开 README（中英双语）
├── stitch/                      # UI 设计稿（HTML 模型 + 截图 + 设计规范）
│   ├── negative_entropy/       # DESIGN.md — 设计系统规范
│   ├── dashboard/              # 仪表盘页面设计稿
│   ├── c/                    # 对话页面设计稿
│   └── api_c/               # API 配置页面设计稿
├── GITHUB_SETUP.md              # 仓库上传指南
├── UPLOAD_GUIDE.md              # 上传指南（备选）
└── LICENSE                      # MIT 协议
```

## 查阅索引

| 需求 | 文件 | 备注 |
|------|------|------|
| 当前架构全貌 | `architecture-v2.0-ios.md` | SwiftUI + SwiftData + CloudKit |
| AI 提供商详情 | `architecture-v1.1-updates.md` §2 | 6 个提供商，OpenAI 兼容协议 |
| 数据模型 (Swift) | `architecture-v2.0-ios.md` §3 | KnowledgeUnit, Flashcard, SemanticLink, ReviewLog |
| FSRS 算法 | `architecture-v2.0-ios.md` §5 | Swift 实现，含默认权重 |
| 云同步设计 | `architecture-v1.1-updates.md` §3 | WebDAV/S3/自定义，Last-Write-Wins |
| 语境注入 (RAG) | `architecture-v1.0-final.md` §5.4 | 三层检索（核心/背景/参考） |
| 设计系统规范 | `stitch/negative_entropy/DESIGN.md` | "The Kinetic Archive - Terminal Authority" 完整设计规范 |
| UI 配色系统 | `architecture-v2.0-ios.md` §2.2 | SwiftUI 颜色值；设计规范见 `stitch/negative_entropy/DESIGN.md` |
| UI 页面设计稿 | `stitch/` 目录 | 4 个页面：工作流、仪表盘、对话、API 配置 |
| 计划文件结构 | `PROJECT_CONTEXT.md` | NegativeEntropy/{App,Views,ViewModels,Services,Models,Utils} |
| 开发路线图 | `PROJECT_CONTEXT.md` | 5 个阶段，约 12-15 周 |

## 架构决策

| 决策项 | 选择 | 理由 |
|--------|------|------|
| 平台 | iOS App Store | 面向移动端学习者 |
| UI 框架 | SwiftUI | iOS 原生，App Store 合规 |
| 架构模式 | MVVM | SwiftUI 推荐模式 |
| 数据层 | SwiftData + CloudKit | iOS 17+ 原生 ORM，自动 iCloud 同步 |
| 图谱可视化 | SpriteKit | 原生物理引擎 |
| SRS 算法 | FSRS | 现代、自适应、4 状态 (New/Learning/Review/Relearning) |
| 默认 AI | Kimi2.5 (Moonshot API) | 多模态支持，中文优化 |
| AI 客户端模式 | OpenAI 兼容协议 | 统一接口支持 6+ 提供商 |
| 云同步 | CloudKit（主） | Apple 原生，审核友好 |
| 数据加密 | 不实现 | 简化架构 |
| 许可证 | MIT | 社区友好 |

**历史转向**: 最初设计为 Tauri+React+Rust 桌面应用 (v1.0)。后转向 iOS 原生 (v2.0)，因为 Tauri 不支持 iOS 且 App Store 要求原生技术栈。

## AI 提供商（计划）

所有提供商使用 OpenAI 兼容的 `/chat/completions` 端点模式：

| 提供商 | Base URL | 默认模型 | 多模态 |
|--------|----------|----------|--------|
| Kimi（默认） | `api.moonshot.cn/v1` | kimi2.5 | ✅ |
| OpenAI | `api.openai.com/v1` | gpt-5.4 | ✅ |
| MiniMax | `api.minimax.chat/v1` | minimax2.7 | ❌ |
| Gemini | `generativelanguage.googleapis.com/v1` | gemini-3.1-pro | ✅ |
| 智谱GLM | `open.bigmodel.cn/api/paas/v4` | glm-5 | ✅ |
| 自定义 | 用户配置 | 用户配置 | 不定 |

## 核心数据模型（计划）

**KnowledgeUnit** — 从对话中提取的结构化知识。字段: id, type (concept/code/mechanism/definition/example), title, content (Markdown), summary, embedding (Float 数组序列化为 Data), mastery (0-1), tags, domain, difficulty (1-5)。

**Flashcard** — FSRS 管理的复习卡片。字段: front, back, due, stability, difficulty (1-10), elapsedDays, scheduledDays, reps, lapses, state (new/learning/relearning/review)。

**SemanticLink** — 知识关系边。字段: source, target, type (prerequisite/extends/related/contrasts), weight (0-1), autoDetected。

**ChatSession** — 对话容器。字段: messages, extractedUnits, entropy score。

## 设计系统（计划）

"终端权威" 赛博朋克暗色主题。核心配色:
- 主色: 矩阵绿 `Color(red: 0, green: 1, blue: 0.255)`
- 副色: 赛博蓝 `Color(red: 0, green: 0.635, blue: 0.992)`
- 警示: 琥珀色 `Color(red: 1, green: 0.729, blue: 0.22)`
- 表面: 近黑 `Color(red: 0.075, green: 0.075, blue: 0.075)`

5 个 Tab 导航: 枢纽(Dashboard), 图谱(Graph), 对话(Chat), 复习(Review), 设置(Settings)。

## 编码规范（实现阶段生效）

- **语言**: Swift 5.9+，最低部署 iOS 17.0+
- **UI**: 仅 SwiftUI（除非需要桥接否则不用 UIKit）
- **数据**: SwiftData `@Model` 类，UUID 使用 `@Attribute(.unique)`
- **关系**: 使用 `@Relationship(deleteRule:)` 并显式指定反向键路径
- **异步**: `async/await` + `AsyncThrowingStream` 处理 AI 流式响应
- **API 客户端**: 协议 `AIServiceProtocol` → 具体实现 `OpenAICompatibleService`
- **颜色**: 使用 `Color` 扩展 (neoGreen, neoBlue 等)，禁止硬编码
- **图标**: SF Symbols (`Image(systemName:)`)，不使用自定义资源
- **命名**: 双语 UI 字符串（zh-CN 为主，en 为辅）
- **禁止强制解包**: 避免 `!`，使用 `guard let` / `if let` / `??`
- **不实现数据加密**: 设计决策简化

## 计划文件结构（尚未创建）

```
NegativeEntropy/
├── App/                          # NegativeEntropyApp.swift, ContentView.swift
├── Views/Chat/                   # 对话界面 + 语境注入
├── Views/Knowledge/              # 知识列表、详情、搜索
├── Views/Graph/                  # SpriteKit 知识图谱
├── Views/Review/                 # FSRS 闪卡复习
├── Views/Settings/               # AI 配置、同步配置、偏好
├── ViewModels/                   # MVVM 视图模型
├── Services/AIService.swift      # 多提供商 AI 客户端
├── Services/KnowledgeService.swift # 知识提取 + 管理
├── Services/SRSService.swift     # FSRS 算法实现
├── Services/SyncService.swift    # CloudKit 同步
├── Models/                       # SwiftData @Model 类
└── Utils/                        # 扩展、工具类
```

## 构建命令

暂无构建/测试/lint 命令。Xcode 工程创建后的预期命令：

```bash
# 构建
open NegativeEntropy.xcodeproj
# 或 xcodebuild:
xcodebuild -scheme NegativeEntropy -destination 'platform=iOS Simulator,name=iPhone 15' build

# 全量测试
xcodebuild test -scheme NegativeEntropy -destination 'platform=iOS Simulator,name=iPhone 15'

# 单个测试文件
xcodebuild test -scheme NegativeEntropy -only-testing:NegativeEntropyTests/FlashcardTests
```

## 注意事项

- 所有架构文档为中英双语（中文为主，英文为辅）
- GitHub 远程仓库: `JesstLe/negative-entropy`
- `architecture-v1.0-final.md` 共 1447 行，覆盖原始 Tauri 方案的完整设计 — 领域概念（知识提取、RAG 管线、FSRS）仍有参考价值，但技术栈已废弃
- FSRS 默认权重: `[0.4, 0.6, 2.4, 2.2, 4.2, 0.5, 0.5, 1.2]` — 见 v2.0 §5
- 计划嵌入模型: BGE-M3（768 维），以序列化 Float 数组存储
- RAG 三层检索: 核心 (>0.85 相似度), 背景 (0.75-0.85), 参考 (0.65-0.75)
- 向量搜索自动关联阈值: 余弦相似度 > 0.75
- App Store 要求: 仅 HTTPS (ATS), PrivacyInfo.xcprivacy, CloudKit 能力
