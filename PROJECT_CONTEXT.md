# 负熵知识复利系统 - 项目上下文

## 项目概述

**项目名称**: Negative Entropy（负熵）  
**类型**: iOS App Store应用  
**愿景**: 对抗遗忘的知识复利系统，将学习对话转化为语义图谱、闪卡和持久记忆

---

## 核心功能

1. **知识提取**: AI自动从学习对话中提取结构化知识
2. **语义图谱**: 基于嵌入向量构建知识关联网络
3. **间隔重复**: FSRS算法驱动的闪卡复习系统
4. **语境注入**: RAG驱动的对话上下文增强

---

## 技术栈决策

### 最终选择
- **平台**: iOS App Store
- **UI框架**: SwiftUI
- **数据存储**: SwiftData (内置CloudKit同步)
- **AI集成**: 多提供商API（默认Kimi2.5）
- **图谱可视化**: SpriteKit
- **架构模式**: MVVM

### 支持的AI提供商
- Kimi (默认，支持多模态)
- OpenAI
- MiniMax
- Google Gemini
- 智谱GLM
- 自定义API

---

## 关键设计决策

| 决策项 | 选择 | 理由 |
|--------|------|------|
| 云同步 | ✅ CloudKit | 苹果原生，审核友好 |
| 开源 | ✅ MIT License | 社区友好 |
| 数据加密 | ❌ 不需要 | 简化架构 |
| 默认AI模型 | Kimi2.5 | 多模态支持好 |
| SRS算法 | FSRS | 现代化、自适应 |

---

## 项目文件结构

```
negative-entropy/
├── README.md                    # 项目介绍
├── LICENSE                      # MIT License
├── docs/                        # 文档
│   ├── architecture-v2.0-ios.md # 完整架构文档
│   └── PRD.md                   # 产品需求文档
├── NegativeEntropy/             # Xcode项目
│   ├── App/
│   │   ├── NegativeEntropyApp.swift
│   │   └── ContentView.swift
│   ├── Views/
│   │   ├── Chat/
│   │   ├── Knowledge/
│   │   ├── Graph/
│   │   ├── Review/
│   │   └── Settings/
│   ├── ViewModels/
│   ├── Services/
│   │   ├── AIService.swift
│   │   ├── KnowledgeService.swift
│   │   ├── SRSService.swift
│   │   └── SyncService.swift
│   ├── Models/
│   │   ├── KnowledgeUnit.swift
│   │   ├── Flashcard.swift
│   │   ├── SemanticLink.swift
│   │   └── ChatSession.swift
│   └── Utils/
└── Resources/
    └── Prompts/
```

---

## 开发路线图

### Phase 1: 基础框架（3-4周）
- [ ] Xcode项目初始化
- [ ] SwiftData模型配置
- [ ] 基础UI组件（复刻设计系统）
- [ ] 对话界面

### Phase 2: AI集成（2-3周）
- [ ] AI服务抽象层
- [ ] 多提供商支持
- [ ] 知识提取功能
- [ ] 多模态输入

### Phase 3: 知识管理（2-3周）
- [ ] 知识库列表/详情
- [ ] 嵌入向量计算
- [ ] 语义相似度搜索
- [ ] 知识图谱可视化

### Phase 4: SRS复习（2-3周）
- [ ] FSRS算法实现
- [ ] 闪卡生成
- [ ] 复习界面
- [ ] 学习统计

### Phase 5: 发布（2周）
- [ ] iCloud同步配置
- [ ] App Store准备
- [ ] TestFlight测试
- [ ] 提交审核

---

## 参考文档

- `architecture-v2.0-ios.md` - 完整技术架构
- UI设计源文件 - `C:/Users/Administrator/Downloads/stitch (1)/stitch/`

---

## 开发环境

- **开发机**: Mac ( Sonoma 14.0+ )
- **IDE**: Xcode 15.0+
- **最低部署**: iOS 17.0+
- **语言**: Swift 5.9+

---

## GitHub仓库

**Repository**: https://github.com/[username]/negative-entropy

**克隆命令**:
```bash
git clone https://github.com/[username]/negative-entropy.git
cd negative-entropy
```

---

## 备注

- 项目最初考虑Tauri桌面方案，后改为iOS原生
- 设计系统采用"Terminal Authority"赛博朋克风格
- 数据模型参考了FSRS算法和知识图谱最佳实践

---

*文档生成时间: 2026-03-27*  
*对话会话: Sisyphus + 用户*
