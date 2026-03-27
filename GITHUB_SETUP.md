# 如何设置GitHub仓库

## 1. 在GitHub创建仓库

1. 访问 https://github.com/new
2. 填写信息：
   - **Repository name**: `negative-entropy`
   - **Description**: A knowledge compounding system for iOS
   - **Visibility**: Public (开源) 或 Private (私有)
   - ✅ Initialize with README (可选)
   - ✅ Add .gitignore: Swift
   - ✅ Choose a license: MIT

3. 点击 **Create repository**

## 2. 在Windows上推送代码

```bash
# 进入项目目录
cd E:\Documents\WorkSpace\openStudy

# 初始化git
git init

# 添加所有文档
git add .

# 提交
git commit -m "Initial commit: Architecture docs and project context"

# 关联远程仓库（替换YOUR_USERNAME）
git remote add origin https://github.com/YOUR_USERNAME/negative-entropy.git

# 推送
git branch -M main
git push -u origin main
```

## 3. 在Mac上拉取

```bash
# 克隆到Mac
cd ~/Documents
git clone https://github.com/YOUR_USERNAME/negative-entropy.git

# 进入项目
cd negative-entropy

# 查看文档
open PROJECT_CONTEXT.md
open architecture-v2.0-ios.md
```

## 4. Xcode项目初始化（在Mac上）

```bash
# 创建Xcode项目文件夹
mkdir NegativeEntropyApp
cd NegativeEntropyApp

# 用Xcode创建项目后，推送回GitHub
git add .
git commit -m "feat: Initialize Xcode project"
git push
```

## 5. 工作流建议

**Windows** (文档/设计):
- 更新PRD、架构文档
- 设计素材
- 推送: `git push origin main`

**Mac** (代码开发):
- 拉取: `git pull origin main`
- Xcode开发
- 推送: `git push origin main`

## 6. 分支策略（可选）

```bash
# 功能开发用分支
git checkout -b feature/chat-interface
# 开发...
git commit -m "feat: Add chat interface"
git push origin feature/chat-interface

# GitHub上创建Pull Request合并到main
```

---

**提示**: 记得把 `YOUR_USERNAME` 替换为您的实际GitHub用户名！
