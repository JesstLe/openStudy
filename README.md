# Negative Entropy

![Platform](https://img.shields.io/badge/platform-iOS%2017.0+-blue)
![Language](https://img.shields.io/badge/language-Swift%205.9-orange)
![License](https://img.shields.io/badge/license-MIT-green)

> A knowledge compounding system that turns learning conversations into semantic graphs, flashcards, and persistent memory to combat forgetfulness.

## 负熵知识复利系统

将学习对话转化为可复利增长的知识资产，通过**语义图谱**、**间隔重复**和**语境注入**三大核心机制，实现知识的持久化和自我强化。

## 核心功能

- 🤖 **智能知识提取** - AI自动从对话中提取结构化知识点
- 🕸️ **语义知识图谱** - 基于嵌入向量自动发现知识关联
- 🃏 **FSRS间隔重复** - 现代化闪卡复习算法
- 💬 **语境注入对话** - AI回复自然引用历史知识

## 技术栈

- **UI**: SwiftUI (iOS原生)
- **数据**: SwiftData + CloudKit (自动同步)
- **AI**: 多提供商支持（默认Kimi2.5）
- **架构**: MVVM

## 系统要求

- iOS 17.0+
- Xcode 15.0+
- macOS 14.0+ (开发)

## 快速开始

```bash
git clone https://github.com/[your-username]/negative-entropy.git
cd negative-entropy
open NegativeEntropy.xcodeproj
```

## 文档

- [架构设计](docs/architecture-v2.0-ios.md)
- [项目上下文](PROJECT_CONTEXT.md)

## 开源协议

MIT License

## 致谢

设计系统: The Kinetic Archive - Terminal Authority  
SRS算法: FSRS (Free Spaced Repetition Scheduler)
