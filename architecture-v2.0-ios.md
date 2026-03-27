# 负熵知识复利系统 (Negative Entropy)
## iOS架构设计 v2.0 - SwiftUI + SwiftData

---

## 重要变更说明

**原方案（Tauri + React + Rust）→ 新方案（SwiftUI + Swift）**

原因：
- iOS App Store严格要求使用原生技术栈
- Tauri不支持iOS（仅支持桌面macOS/Windows/Linux）
- SwiftUI是苹果官方推荐的iOS UI框架
- 必须通过App Store审核指南

---

## 1. iOS技术栈

| 层级 | 技术 | 理由 |
|------|------|------|
| **UI** | SwiftUI | iOS原生、App Store首选 |
| **架构** | MVVM | SwiftUI推荐模式 |
| **存储** | SwiftData | iOS 17+原生ORM、自动CloudKit同步 |
| **AI** | URLSession + 自定义API | 支持多提供商（Kimi/OpenAI等） |
| **同步** | CloudKit | 苹果原生、审核友好 |
| **图表** | SpriteKit | 原生物理引擎适合知识图谱 |

---

## 2. SwiftUI组件映射（React → SwiftUI）

### 2.1 核心组件对照表

| React组件 | SwiftUI对应 |
|-----------|-------------|
| `div` | `VStack`, `HStack`, `ZStack` |
| `p` / `span` | `Text` |
| `button` | `Button` |
| `input` | `TextField` |
| `img` | `Image` |
| `scroll` | `ScrollView`, `List` |
| `nav` | `NavigationStack` |
| Material Icons | SF Symbols (`Image(systemName: "icon")`) |

### 2.2 设计系统配色（SwiftUI）

```swift
// 设计系统配色扩展
extension Color {
    // Primary - Matrix Green
    static let neoGreen = Color(red: 0, green: 1, blue: 0.255)
    static let neoGreenDark = Color(red: 0, green: 0.816, blue: 0.161)
    
    // Secondary - Cyber Blue
    static let neoBlue = Color(red: 0, green: 0.635, blue: 0.992)
    
    // Tertiary - Alert Amber
    static let neoAmber = Color(red: 1, green: 0.729, blue: 0.22)
    
    // Surface
    static let neoSurface = Color(red: 0.075, green: 0.075, blue: 0.075)
    static let neoSurfaceLow = Color(red: 0.11, green: 0.106, blue: 0.106)
    static let neoSurfaceContainer = Color(red: 0.125, green: 0.122, blue: 0.122)
}
```

### 2.3 主要视图结构

```swift
// 主Tab导航（对应React的BottomNav）
struct ContentView: View {
    @State private var selectedTab = 0
    
    var body: some View {
        TabView(selection: $selectedTab) {
            DashboardView()
                .tabItem {
                    Image(systemName: "grid.view")
                    Text("枢纽")
                }
                .tag(0)
            
            GraphView()
                .tabItem {
                    Image(systemName: "network")
                    Text("图谱")
                }
                .tag(1)
            
            ChatView()
                .tabItem {
                    Image(systemName: "bubble.left.fill")
                    Text("对话")
                }
                .tag(2)
            
            ReviewView()
                .tabItem {
                    Image(systemName: "rectangle.stack.fill")
                    Text("复习")
                }
                .tag(3)
            
            SettingsView()
                .tabItem {
                    Image(systemName: "gear")
                    Text("设置")
                }
                .tag(4)
        }
        .tint(.neoGreen)
        .preferredColorScheme(.dark)
    }
}
```

---

## 3. SwiftData数据模型

### 3.1 核心实体

```swift
import SwiftData

// 知识单元
@Model
class KnowledgeUnit {
    @Attribute(.unique) var id: UUID
    var type: KnowledgeType
    var title: String
    var content: String
    var summary: String
    
    var sourceSessionId: UUID?
    var sourceTimestamp: Date
    
    var embedding: Data?  // Float数组序列化
    var mastery: Double
    var lastReviewed: Date?
    var reviewCount: Int
    
    var tags: [String]
    var domain: String
    var difficulty: Int
    
    var createdAt: Date
    var updatedAt: Date
    
    // 关系
    @Relationship(deleteRule: .cascade, inverse: \Flashcard.unit)
    var flashcards: [Flashcard]?
    
    @Relationship(deleteRule: .nullify, inverse: \SemanticLink.source)
    var outgoingLinks: [SemanticLink]?
}

enum KnowledgeType: String, Codable, CaseIterable {
    case concept = "概念"
    case code = "代码"
    case mechanism = "机制"
    case definition = "定义"
    case example = "示例"
}

// 闪卡（FSRS算法）
@Model
class Flashcard {
    @Attribute(.unique) var id: UUID
    var front: String
    var back: String
    var tags: [String]
    
    // FSRS参数
    var due: Date
    var stability: Double
    var difficulty: Double
    var elapsedDays: Int
    var scheduledDays: Int
    var reps: Int
    var lapses: Int
    var state: CardState
    
    var createdAt: Date
    
    @Relationship(deleteRule: .cascade, inverse: \ReviewLog.card)
    var reviewLogs: [ReviewLog]?
    
    var unit: KnowledgeUnit?
}

enum CardState: Int, Codable {
    case new = 0
    case learning = 1
    case review = 2
    case relearning = 3
}

enum ReviewRating: Int, Codable {
    case again = 1
    case hard = 2
    case good = 3
    case easy = 4
}

// 复习日志
@Model
class ReviewLog {
    var rating: ReviewRating
    var reviewedAt: Date
    var reviewDurationMs: Int
    var prevStability: Double
    var nextStability: Double
    var card: Flashcard?
}

// 语义链接
@Model
class SemanticLink {
    @Attribute(.unique) var id: UUID
    var type: LinkType
    var weight: Double
    var autoDetected: Bool
    var createdAt: Date
    var source: KnowledgeUnit?
    var target: KnowledgeUnit?
}

enum LinkType: String, Codable {
    case prerequisite = "前置"
    case related = "相关"
    case extends = "扩展"
    case contrasts = "对比"
}
```

### 3.2 App配置

```swift
@main
struct NegativeEntropyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: [
            KnowledgeUnit.self,
            ChatSession.self,
            ChatMessage.self,
            SemanticLink.self,
            Flashcard.self,
            ReviewLog.self
        ])
    }
}
```

---

## 4. AI服务（多提供商）

### 4.1 支持的提供商

```swift
enum AIProvider: String, CaseIterable {
    case kimi = "Kimi"           // 默认，支持多模态
    case openai = "OpenAI"
    case minimax = "MiniMax"
    case gemini = "Gemini"
    case zhipu = "智谱"
    case custom = "自定义"
}

struct AIServiceConfig {
    var provider: AIProvider
    var baseURL: String
    var apiKey: String
    var model: String
    var temperature: Double = 0.7
    var maxTokens: Int = 4096
    var supportsMultimodal: Bool = false
}

// 默认配置：Kimi2.5
let defaultAIConfig = AIServiceConfig(
    provider: .kimi,
    baseURL: "https://api.moonshot.cn/v1",
    apiKey: "",
    model: "kimi2.5",
    supportsMultimodal: true
)
```

### 4.2 AI客户端实现

```swift
protocol AIServiceProtocol {
    func chat(messages: [ChatMessage]) async throws -> String
    func chatStream(messages: [ChatMessage]) -> AsyncThrowingStream<String, Error>
    func extractKnowledge(from conversation: String) async throws -> [KnowledgeUnit]
}

class OpenAICompatibleService: AIServiceProtocol {
    private let config: AIServiceConfig
    private let session: URLSession
    
    init(config: AIServiceConfig) {
        self.config = config
        self.session = URLSession(configuration: .default)
    }
    
    func chat(messages: [ChatMessage]) async throws -> String {
        let url = URL(string: "\(config.baseURL)/chat/completions")!
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.setValue("Bearer \(config.apiKey)", forHTTPHeaderField: "Authorization")
        
        let body: [String: Any] = [
            "model": config.model,
            "messages": messages.map { ["role": $0.role.rawValue, "content": $0.content] },
            "temperature": config.temperature,
            "max_tokens": config.maxTokens
        ]
        
        request.httpBody = try JSONSerialization.data(withJSONObject: body)
        
        let (data, _) = try await session.data(for: request)
        let response = try JSONDecoder().decode(ChatResponse.self, from: data)
        
        return response.choices.first?.message.content ?? ""
    }
    
    func extractKnowledge(from conversation: String) async throws -> [KnowledgeUnit] {
        let prompt = extractKnowledgePrompt(conversation: conversation)
        let messages = [ChatMessage(role: .system, content: prompt)]
        let response = try await chat(messages: messages)
        // 解析JSON...
        return []
    }
}
```

---

## 5. FSRS算法（Swift）

```swift
class FSRSCalculator {
    private let w = [0.4, 0.6, 2.4, 2.2, 4.2, 0.5, 0.5, 1.2]
    
    func calculate(card: Flashcard, rating: ReviewRating, now: Date = Date()) -> ReviewResult {
        let newDifficulty = min(10, max(1, card.difficulty + w[4 + (rating.rawValue - 1)]))
        
        let newStability: Double
        if card.state == .new {
            newStability = w[rating.rawValue - 1]
        } else {
            let retrievability = pow(1 + Double(card.elapsedDays) / card.stability, -1.0)
            newStability = card.stability * (1 + w[0] * (11 - newDifficulty) * pow(retrievability, -w[1]))
        }
        
        let scheduledDays = max(1, Int(round(newStability)))
        let nextDue = Calendar.current.date(byAdding: .day, value: scheduledDays, to: now)!
        
        return ReviewResult(
            due: nextDue,
            stability: newStability,
            difficulty: newDifficulty,
            scheduledDays: scheduledDays
        )
    }
}

struct ReviewResult {
    let due: Date
    let stability: Double
    let difficulty: Double
    let scheduledDays: Int
}
```

---

## 6. iCloud同步

```swift
import CloudKit

class CloudKitSyncManager {
    private let container = CKContainer.default()
    private let database: CKDatabase
    
    init() {
        database = container.privateCloudDatabase
    }
    
    func checkAccountStatus() async -> CKAccountStatus {
        try? await container.accountStatus() ?? .couldNotDetermine
    }
    
    func syncToCloud(unit: KnowledgeUnit) async throws {
        let record = CKRecord(recordType: "KnowledgeUnit")
        record["id"] = unit.id.uuidString
        record["title"] = unit.title
        record["content"] = unit.content
        record["createdAt"] = unit.createdAt
        
        try await database.save(record)
    }
}
```

---

## 7. App Store准备清单

### 7.1 必需配置

- [ ] **Bundle ID**: com.yourcompany.negative-entropy
- [ ] **Capabilities**: iCloud (CloudKit), Push Notifications
- [ ] **App Groups**: 用于数据共享
- [ ] **PrivacyInfo.xcprivacy**: 隐私清单
- [ ] **ATS**: 只允许HTTPS

### 7.2 隐私清单示例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN">
<plist version="1.0">
<dict>
    <key>NSPrivacyCollectedDataTypes</key>
    <array>
        <dict>
            <key>NSPrivacyCollectedDataType</key>
            <string>NSPrivacyCollectedDataTypeOtherData</string>
            <key>NSPrivacyCollectedDataTypeTracking</key>
            <false/>
        </dict>
    </array>
</dict>
</plist>
```

---

## 8. 开发路线图（iOS）

| Phase | 时长 | 主要任务 |
|-------|------|----------|
| **Phase 1** | 3-4周 | SwiftUI项目初始化、基础UI组件、对话界面 |
| **Phase 2** | 2-3周 | AI服务层、多提供商支持、知识提取 |
| **Phase 3** | 2-3周 | 知识库、嵌入向量、图谱可视化（SpriteKit）|
| **Phase 4** | 2-3周 | FSRS算法、闪卡、复习界面 |
| **Phase 5** | 2周 | iCloud同步、App Store提交 |

---

## 9. 文件清单

| 文件 | 说明 |
|------|------|
| `architecture-v1.0-final.md` | 原始Tauri架构（桌面版） |
| `architecture-v1.1-updates.md` | 配置决策更新 |
| `architecture-v2.0-ios.md` | **本文档** - iOS SwiftUI架构 |

---

**关键技术点：**

1. **SwiftUI** 替代 React + Tailwind
2. **SwiftData** 替代 SQLite（内置CloudKit同步）
3. **URLSession** 处理AI API调用
4. **SpriteKit** 实现知识图谱可视化
5. **CloudKit** 实现iCloud同步

**下一步**：是否需要开始搭建iOS项目脚手架？
