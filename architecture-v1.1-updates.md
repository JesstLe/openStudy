# 负熵知识复利系统 (Negative Entropy)
## 架构更新 v1.1 - 配置决策确认

---

## 1. 决策确认清单

| 决策项 | 用户选择 | 架构影响 |
|--------|----------|----------|
| **云同步** | ✅ 支持 | 需要设计同步模块、冲突解决策略 |
| **开源** | ✅ 是 | 选择开源协议、社区贡献指南 |
| **数据加密** | ❌ 不需要 | 简化架构，无需端到端加密 |
| **AI模型** | 用户自定义API + 默认kimi2.5 | 多提供商支持、配置界面 |

---

## 2. AI配置模块更新

### 2.1 支持的AI提供商

```typescript
// 支持的API提供商列表
const AI_PROVIDERS = {
  kimi: {
    name: 'Kimi',
    baseUrl: 'https://api.moonshot.cn/v1',
    defaultModel: 'kimi2.5',
    supportsMultimodal: true,
    apiKeyEnv: 'KIMI_API_KEY',
  },
  openai: {
    name: 'OpenAI',
    baseUrl: 'https://api.openai.com/v1',
    defaultModel: 'gpt-5.4',
    supportsMultimodal: true,
    apiKeyEnv: 'OPENAI_API_KEY',
  },
  minimax: {
    name: 'MiniMax',
    baseUrl: 'https://api.minimax.chat/v1',
    defaultModel: 'minimax2.7',
    supportsMultimodal: false,
    apiKeyEnv: 'MINIMAX_API_KEY',
  },
  gemini: {
    name: 'Google Gemini',
    baseUrl: 'https://generativelanguage.googleapis.com/v1',
    defaultModel: 'gemini-3.1-pro',
    supportsMultimodal: true,
    apiKeyEnv: 'GEMINI_API_KEY',
  },
  zhipu: {
    name: '智谱GLM',
    baseUrl: 'https://open.bigmodel.cn/api/paas/v4',
    defaultModel: 'glm-5',
    supportsMultimodal: true,
    apiKeyEnv: 'ZHIPU_API_KEY',
  },
  custom: {
    name: '自定义',
    baseUrl: '',  // 用户填写
    defaultModel: '',  // 用户填写
    supportsMultimodal: false,  // 用户标记
    apiKeyEnv: '',
  },
  ollama: {
    name: 'Ollama (本地)',
    baseUrl: 'http://localhost:11434',
    defaultModel: 'qwen2.5:7b',
    supportsMultimodal: false,
    apiKeyEnv: null,  // 本地无需API Key
  },
} as const;
```

### 2.2 配置数据结构

```typescript
interface AIConfig {
  provider: keyof typeof AI_PROVIDERS;
  
  // 基础配置
  baseUrl: string;
  apiKey: string;  // 加密存储（系统级）
  model: string;
  
  // 模型参数
  temperature: number;  // 0.0 - 2.0，默认0.7
  maxTokens: number;    // 默认4096
  topP: number;         // 默认1.0
  
  // 功能开关
  features: {
    multimodal: boolean;       // 是否支持图片输入
    streaming: boolean;        // 是否启用流式响应
    functionCalling: boolean;  // 是否支持工具调用
  };
  
  // 特定用途配置
  usage: {
    chat: string;        // 对话模型
    extraction: string;  // 知识提取模型
    embedding: string;   // 嵌入模型（可选单独配置）
  };
}

// 默认配置（kimi2.5）
const DEFAULT_AI_CONFIG: AIConfig = {
  provider: 'kimi',
  baseUrl: 'https://api.moonshot.cn/v1',
  apiKey: '',  // 用户首次启动时配置
  model: 'kimi2.5',
  temperature: 0.7,
  maxTokens: 4096,
  topP: 1.0,
  features: {
    multimodal: true,
    streaming: true,
    functionCalling: true,
  },
  usage: {
    chat: 'kimi2.5',
    extraction: 'kimi2.5',
    embedding: 'kimi2.5',  // kimi支持嵌入
  },
};
```

### 2.3 多模态支持

```typescript
// 消息内容类型
interface MessageContent {
  type: 'text' | 'image_url' | 'file';
  text?: string;
  imageUrl?: {
    url: string;  // base64或URL
    detail?: 'low' | 'high' | 'auto';
  };
  file?: {
    name: string;
    mimeType: string;
    content: string;  // base64
  };
}

// 多模态消息
interface MultimodalMessage {
  role: 'user' | 'assistant' | 'system';
  content: string | MessageContent[];  // 支持文本或混合内容
}

// 使用示例
const multimodalPrompt: MultimodalMessage = {
  role: 'user',
  content: [
    { type: 'text', text: '分析这张代码截图中的问题' },
    { 
      type: 'image_url', 
      imageUrl: { 
        url: 'data:image/png;base64,iVBORw0KGgo...',
        detail: 'high'
      }
    }
  ]
};
```

### 2.4 AI客户端抽象层

```rust
// Rust后端：AI客户端 trait
#[async_trait]
pub trait AIClient: Send + Sync {
    async fn chat(&self, messages: &[Message]) -> Result<String, AIError>;
    async fn chat_stream(&self, messages: &[Message]) -> Result<Stream<String>, AIError>;
    async fn embed(&self, texts: &[String]) -> Result<Vec<Vec<f32>>, AIError>;
    async fn extract_knowledge(&self, conversation: &str) -> Result<Vec<KnowledgeUnit>, AIError>;
    fn supports_multimodal(&self) -> bool;
}

// OpenAI兼容格式客户端
pub struct OpenAICompatibleClient {
    config: AIConfig,
    client: reqwest::Client,
}

#[async_trait]
impl AIClient for OpenAICompatibleClient {
    async fn chat(&self, messages: &[Message]) -> Result<String, AIError> {
        let response = self.client
            .post(format!("{}/chat/completions", self.config.base_url))
            .header("Authorization", format!("Bearer {}", self.config.api_key))
            .json(&json!({
                "model": self.config.model,
                "messages": messages,
                "temperature": self.config.temperature,
                "max_tokens": self.config.max_tokens,
                "top_p": self.config.top_p,
            }))
            .send()
            .await?;
        
        // 解析响应...
        Ok(content)
    }
    
    // ... 其他方法实现
}

// 客户端工厂
pub struct AIClientFactory;

impl AIClientFactory {
    pub fn create(config: &AIConfig) -> Box<dyn AIClient> {
        match config.provider {
            "openai" | "kimi" | "minimax" | "zhipu" => {
                Box::new(OpenAICompatibleClient::new(config))
            }
            "gemini" => Box::new(GeminiClient::new(config)),
            "ollama" => Box::new(OllamaClient::new(config)),
            "custom" => Box::new(OpenAICompatibleClient::new(config)),
            _ => panic!("Unknown provider: {}", config.provider),
        }
    }
}
```

### 2.5 配置界面设计

```tsx
// 设置页面：AI配置
export const AISettings: React.FC = () => {
  const [config, setConfig] = useAtom(aiConfigAtom);
  const [testResult, setTestResult] = useState<TestResult | null>(null);
  
  const handleProviderChange = (provider: string) => {
    const defaultConfig = AI_PROVIDERS[provider];
    setConfig({
      ...config,
      provider,
      baseUrl: defaultConfig.baseUrl,
      model: defaultConfig.defaultModel,
      features: {
        ...config.features,
        multimodal: defaultConfig.supportsMultimodal,
      },
    });
  };
  
  const testConnection = async () => {
    try {
      const result = await invoke('test_ai_connection', { config });
      setTestResult({ success: true, latency: result.latency });
    } catch (e) {
      setTestResult({ success: false, error: e.message });
    }
  };
  
  return (
    <SettingsSection title="AI配置">
      {/* 提供商选择 */}
      <Select
        label="AI提供商"
        value={config.provider}
        onChange={handleProviderChange}
        options={Object.entries(AI_PROVIDERS).map(([key, p]) => ({
          value: key,
          label: p.name,
        }))}
      />
      
      {/* API配置 */}
      <Input
        label="API地址"
        value={config.baseUrl}
        onChange={(v) => setConfig({ ...config, baseUrl: v })}
        placeholder="https://api.example.com/v1"
      />
      
      <Input
        label="API Key"
        type="password"
        value={config.apiKey}
        onChange={(v) => setConfig({ ...config, apiKey: v })}
        placeholder="sk-..."
      />
      
      <Input
        label="模型名称"
        value={config.model}
        onChange={(v) => setConfig({ ...config, model: v })}
        placeholder="kimi2.5"
      />
      
      {/* 模型参数 */}
      <Slider
        label="Temperature"
        value={config.temperature}
        min={0}
        max={2}
        step={0.1}
        onChange={(v) => setConfig({ ...config, temperature: v })}
      />
      
      <NumberInput
        label="Max Tokens"
        value={config.maxTokens}
        min={1}
        max={8192}
        onChange={(v) => setConfig({ ...config, maxTokens: v })}
      />
      
      {/* 功能开关 */}
      <Switch
        label="启用多模态"
        checked={config.features.multimodal}
        onChange={(v) => setConfig({ ...config, features: { ...config.features, multimodal: v } })}
      />
      
      <Switch
        label="流式响应"
        checked={config.features.streaming}
        onChange={(v) => setConfig({ ...config, features: { ...config.features, streaming: v } })}
      />
      
      {/* 测试连接 */}
      <Button onClick={testConnection} variant="secondary">
        测试连接
      </Button>
      
      {testResult && (
        <Alert type={testResult.success ? 'success' : 'error'}>
          {testResult.success 
            ? `连接成功！延迟: ${testResult.latency}ms` 
            : `连接失败: ${testResult.error}`}
        </Alert>
      )}
    </SettingsSection>
  );
};
```

---

## 3. 云同步模块设计

### 3.1 同步策略

**方案选择**：双向同步 + 最后写入优先（Last Write Wins）

```
┌─────────────┐         ┌─────────────┐
│  Device A   │◄───────►│  Device B   │
│  (本地)     │  云端   │  (本地)     │
└──────┬──────┘  协调   └──────┬──────┘
       │                       │
       └───────────┬───────────┘
                   ▼
           ┌───────────────┐
           │   Cloud API   │
           │ (WebDAV/S3/   │
           │  自建服务器)   │
           └───────────────┘
```

### 3.2 同步协议

```typescript
// 同步元数据
interface SyncMetadata {
  deviceId: string;           // 设备唯一标识
  lastSyncAt: number;         // 上次同步时间
  syncVersion: number;        // 同步版本号（递增）
}

// 数据变更记录
interface ChangeLog {
  id: string;
  table: string;              // 变更的表名
  recordId: string;           // 记录ID
  operation: 'CREATE' | 'UPDATE' | 'DELETE';
  data: any;                  // 变更后的数据
  timestamp: number;          // 变更时间
  deviceId: string;           // 变更设备
}

// 同步请求
interface SyncRequest {
  deviceId: string;
  lastSyncAt: number;
  changes: ChangeLog[];       // 本地变更
}

// 同步响应
interface SyncResponse {
  success: boolean;
  serverChanges: ChangeLog[]; // 服务端变更
  conflicts: Conflict[];      // 冲突列表
  newSyncAt: number;
  syncVersion: number;
}

// 冲突
interface Conflict {
  recordId: string;
  table: string;
  localChange: ChangeLog;
  serverChange: ChangeLog;
  resolution: 'LOCAL' | 'SERVER' | 'MERGE';
}
```

### 3.3 支持的同步后端

```typescript
// 同步提供商配置
interface SyncProvider {
  type: 'webdav' | 's3' | 'custom';
  name: string;
  config: WebDAVConfig | S3Config | CustomConfig;
}

// WebDAV（推荐个人用户）
interface WebDAVConfig {
  url: string;                // 如: https://dav.jianguoyun.com/dav/
  username: string;
  password: string;
  rootPath: string;           // 同步根目录，如: /negative-entropy/
}

// S3兼容（推荐技术用户）
interface S3Config {
  endpoint: string;           // 如: https://s3.amazonaws.com
  bucket: string;
  region: string;
  accessKeyId: string;
  secretAccessKey: string;
  prefix: string;             // 对象前缀
}

// 自定义API
interface CustomConfig {
  baseUrl: string;
  apiKey: string;
  endpoints: {
    sync: string;
    upload: string;
    download: string;
  };
}
```

### 3.4 Rust同步引擎

```rust
pub struct SyncEngine {
    db: Arc<Database>,
    provider: Box<dyn SyncProvider>,
    device_id: String,
}

#[async_trait]
pub trait SyncProvider: Send + Sync {
    async fn upload(&self, path: &str, data: &[u8]) -> Result<(), SyncError>;
    async fn download(&self, path: &str) -> Result<Vec<u8>, SyncError>;
    async fn list(&self, prefix: &str) -> Result<Vec<String>, SyncError>;
    async fn delete(&self, path: &str) -> Result<(), SyncError>;
    async fn last_modified(&self, path: &str) -> Result<Option<u64>, SyncError>;
}

impl SyncEngine {
    /// 执行同步
    pub async fn sync(&self) -> Result<SyncResult, SyncError> {
        // 1. 获取本地变更
        let local_changes = self.get_local_changes().await?;
        
        // 2. 获取上次同步时间
        let last_sync = self.db.get_sync_metadata().await?.last_sync_at;
        
        // 3. 从云端获取变更
        let remote_changes = self.fetch_remote_changes(last_sync).await?;
        
        // 4. 冲突检测与解决
        let (to_upload, to_apply, conflicts) = 
            self.resolve_conflicts(local_changes, remote_changes).await?;
        
        // 5. 上传本地变更
        for change in to_upload {
            self.upload_change(&change).await?;
        }
        
        // 6. 应用远程变更
        for change in to_apply {
            self.apply_change(&change).await?;
        }
        
        // 7. 更新同步时间
        let now = current_timestamp();
        self.db.update_sync_metadata(now).await?;
        
        Ok(SyncResult {
            uploaded: to_upload.len(),
            downloaded: to_apply.len(),
            conflicts,
        })
    }
    
    /// 冲突解决策略
    async fn resolve_conflicts(
        &self,
        local: Vec<ChangeLog>,
        remote: Vec<ChangeLog>,
    ) -> Result<(Vec<ChangeLog>, Vec<ChangeLog>, Vec<Conflict>), SyncError> {
        let mut to_upload = Vec::new();
        let mut to_apply = Vec::new();
        let mut conflicts = Vec::new();
        
        // 按记录ID分组
        let local_map: HashMap<_, _> = local.into_iter()
            .map(|c| ((c.table.clone(), c.record_id.clone()), c))
            .collect();
        let remote_map: HashMap<_, _> = remote.into_iter()
            .map(|c| ((c.table.clone(), c.record_id.clone()), c))
            .collect();
        
        // 处理所有涉及的记录
        let all_keys: HashSet<_> = local_map.keys()
            .chain(remote_map.keys())
            .collect();
        
        for key in all_keys {
            match (local_map.get(key), remote_map.get(key)) {
                // 本地有，远程没有 → 上传
                (Some(local), None) => to_upload.push(local.clone()),
                
                // 本地没有，远程有 → 下载
                (None, Some(remote)) => to_apply.push(remote.clone()),
                
                // 都有 → 冲突检测
                (Some(local), Some(remote)) => {
                    if local.timestamp > remote.timestamp {
                        // 本地更新 → 上传
                        to_upload.push(local.clone());
                    } else if remote.timestamp > local.timestamp {
                        // 远程更新 → 下载
                        to_apply.push(remote.clone());
                    }
                    // 时间戳相同 → 无冲突
                }
                _ => {}
            }
        }
        
        Ok((to_upload, to_apply, conflicts))
    }
    
    /// 应用远程变更到本地数据库
    async fn apply_change(&self, change: &ChangeLog) -> Result<(), SyncError> {
        match change.operation {
            Operation::Create | Operation::Update => {
                self.db.upsert(&change.table, &change.record_id, &change.data).await?;
            }
            Operation::Delete => {
                self.db.delete(&change.table, &change.record_id).await?;
            }
        }
        
        // 记录变更已应用
        self.db.mark_change_applied(change.id.clone()).await?;
        
        Ok(())
    }
}

// WebDAV实现示例
pub struct WebDAVProvider {
    client: reqwest::Client,
    config: WebDAVConfig,
}

#[async_trait]
impl SyncProvider for WebDAVProvider {
    async fn upload(&self, path: &str, data: &[u8]) -> Result<(), SyncError> {
        let url = format!("{}{}", self.config.url, path);
        
        self.client
            .request(reqwest::Method::from_bytes(b"PUT")?, &url)
            .basic_auth(&self.config.username, Some(&self.config.password))
            .body(data.to_vec())
            .send()
            .await?
            .error_for_status()?;
        
        Ok(())
    }
    
    async fn download(&self, path: &str) -> Result<Vec<u8>, SyncError> {
        let url = format!("{}{}", self.config.url, path);
        
        let response = self.client
            .get(&url)
            .basic_auth(&self.config.username, Some(&self.config.password))
            .send()
            .await?;
        
        Ok(response.bytes().await?.to_vec())
    }
    
    // ... 其他方法
}
```

### 3.5 增量同步优化

```rust
// 增量同步：只传输变更部分
pub struct DeltaSync {
    // 数据分块，用于增量传输
    chunk_size: usize,
}

impl DeltaSync {
    /// 计算数据差异
    pub fn compute_delta(
        &self,
        old_data: &[u8],
        new_data: &[u8],
    ) -> Vec<DataChunk> {
        // 使用rsync算法或类似方案
        // 只传输变化的块
        todo!()
    }
    
    /// 应用差异
    pub fn apply_delta(
        &self,
        base_data: &[u8],
        delta: &[DataChunk],
    ) -> Vec<u8> {
        todo!()
    }
}

// 压缩传输
pub fn compress_data(data: &[u8]) -> Vec<u8> {
    use flate2::{Compression, write::GzEncoder};
    use std::io::Write;
    
    let mut encoder = GzEncoder::new(Vec::new(), Compression::default());
    encoder.write_all(data).unwrap();
    encoder.finish().unwrap()
}
```

### 3.6 同步界面

```tsx
// 设置页面：同步配置
export const SyncSettings: React.FC = () => {
  const [syncConfig, setSyncConfig] = useAtom(syncConfigAtom);
  const [syncStatus, setSyncStatus] = useState<SyncStatus | null>(null);
  
  const handleSyncNow = async () => {
    setSyncStatus({ status: 'syncing', progress: 0 });
    
    try {
      const result = await invoke('sync_now');
      setSyncStatus({ 
        status: 'success', 
        lastSync: Date.now(),
        message: `同步完成: ${result.uploaded} 上传, ${result.downloaded} 下载`
      });
    } catch (e) {
      setSyncStatus({ status: 'error', error: e.message });
    }
  };
  
  return (
    <SettingsSection title="云同步">
      {/* 同步提供商选择 */}
      <Select
        label="同步方式"
        value={syncConfig.provider}
        onChange={(v) => setSyncConfig({ ...syncConfig, provider: v })}
        options={[
          { value: 'webdav', label: 'WebDAV (坚果云/OwnCloud)' },
          { value: 's3', label: 'S3兼容存储' },
          { value: 'disabled', label: '禁用同步' },
        ]}
      />
      
      {syncConfig.provider === 'webdav' && (
        <>
          <Input
            label="WebDAV地址"
            value={syncConfig.webdav?.url}
            onChange={(v) => setSyncConfig({
              ...syncConfig,
              webdav: { ...syncConfig.webdav, url: v }
            })}
            placeholder="https://dav.jianguoyun.com/dav/"
          />
          
          <Input
            label="用户名"
            value={syncConfig.webdav?.username}
            onChange={(v) => setSyncConfig({
              ...syncConfig,
              webdav: { ...syncConfig.webdav, username: v }
            })}
          />
          
          <Input
            label="密码"
            type="password"
            value={syncConfig.webdav?.password}
            onChange={(v) => setSyncConfig({
              ...syncConfig,
              webdav: { ...syncConfig.webdav, password: v }
            })}
          />
        </>
      )}
      
      {/* 自动同步设置 */}
      <Switch
        label="自动同步"
        checked={syncConfig.autoSync}
        onChange={(v) => setSyncConfig({ ...syncConfig, autoSync: v })}
      />
      
      {syncConfig.autoSync && (
        <Select
          label="同步频率"
          value={syncConfig.syncInterval}
          onChange={(v) => setSyncConfig({ ...syncConfig, syncInterval: parseInt(v) })}
          options={[
            { value: '300', label: '5分钟' },
            { value: '900', label: '15分钟' },
            { value: '3600', label: '1小时' },
          ]}
        />
      )}
      
      {/* 同步按钮和状态 */}
      <Button 
        onClick={handleSyncNow}
        loading={syncStatus?.status === 'syncing'}
      >
        立即同步
      </Button>
      
      {syncStatus?.status === 'success' && (
        <Alert type="success">
          {syncStatus.message}
          <br />
          上次同步: {formatTime(syncStatus.lastSync)}
        </Alert>
      )}
      
      {syncStatus?.status === 'error' && (
        <Alert type="error">同步失败: {syncStatus.error}</Alert>
      )}
    </SettingsSection>
  );
};
```

---

## 4. 开源策略

### 4.1 开源协议选择

**推荐：MIT License**

理由：
- 最宽松的协议，允许商业使用、修改、分发
- 只需保留版权声明
- 适合工具类应用
- 社区友好，易于贡献

```
MIT License

Copyright (c) 2026 Negative Entropy Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
```

备选：Apache 2.0（如果需要专利保护）

### 4.2 项目结构（开源版）

```
negative-entropy/
├── LICENSE                 # MIT License
├── README.md              # 项目介绍
├── CONTRIBUTING.md        # 贡献指南
├── CODE_OF_CONDUCT.md     # 行为准则
├── SECURITY.md            # 安全政策
│
├── docs/                  # 文档
│   ├── architecture/      # 架构文档
│   ├── api/              # API文档
│   └── guides/           # 用户指南
│
├── src-ui/               # 前端源码
│   ├── src/
│   ├── package.json
│   └── README.md
│
├── src-tauri/            # 后端源码 (Rust)
│   ├── src/
│   ├── Cargo.toml
│   └── README.md
│
├── resources/            # 资源文件
│   ├── prompts/         # Prompt模板
│   ├── icons/           # 图标
│   └── themes/          # 主题
│
├── tests/               # 测试
├── scripts/             # 构建脚本
└── .github/             # GitHub配置
    ├── workflows/       # CI/CD
    ├── ISSUE_TEMPLATE/  # Issue模板
    └── PULL_REQUEST_TEMPLATE.md
```

### 4.3 贡献指南要点

```markdown
# CONTRIBUTING.md

## 如何贡献

### 报告Bug
1. 使用最新版本复现问题
2. 搜索现有Issues，避免重复
3. 使用Bug Report模板创建Issue

### 提交功能请求
1. 描述使用场景
2. 说明期望行为
3. 考虑向后兼容性

### 提交代码
1. Fork仓库
2. 创建功能分支 (`git checkout -b feature/amazing-feature`)
3. 提交更改 (`git commit -m 'Add amazing feature'`)
4. 推送到分支 (`git push origin feature/amazing-feature`)
5. 创建Pull Request

## 开发规范

### 代码风格
- Rust: 使用 `cargo fmt` 和 `cargo clippy`
- TypeScript: 使用项目配置的 ESLint

### 提交信息规范
- `feat:` 新功能
- `fix:` 修复Bug
- `docs:` 文档更新
- `style:` 代码格式
- `refactor:` 重构
- `test:` 测试
- `chore:` 构建/工具

### 测试要求
- 新功能必须包含测试
- 保持测试覆盖率 > 80%
```

---

## 5. 更新后的完整架构图

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
│  │     SRS      │  │    Store     │  │   AI Client  │               │
│  │   Engine     │  │   (SQLite)   │  │   Factory    │               │
│  │  间隔重复    │  │  数据持久化  │  │ 多提供商支持 │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    Sync Engine (云同步)                      │   │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐             │   │
│  │  │  WebDAV    │  │    S3      │  │   Custom   │             │   │
│  │  │  Provider  │  │  Provider  │  │  Provider  │             │   │
│  │  └────────────┘  └────────────┘  └────────────┘             │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
                    ┌─────────────┼─────────────┐
                    ▼             ▼             ▼
              ┌─────────┐   ┌──────────┐   ┌──────────┐
              │ SQLite  │   │   AI     │   │  Cloud   │
              │  .db    │   │  APIs    │   │  Sync    │
              └─────────┘   │(Kimi/     │   │ Server   │
                          │OpenAI/etc) │   └──────────┘
                          └──────────┘
```

---

## 6. 配置文件示例

```json
// ~/.config/negative-entropy/config.json
{
  "version": "1.0.0",
  
  "ai": {
    "provider": "kimi",
    "baseUrl": "https://api.moonshot.cn/v1",
    "apiKey": "sk-...",
    "model": "kimi2.5",
    "temperature": 0.7,
    "maxTokens": 4096,
    "features": {
      "multimodal": true,
      "streaming": true,
      "functionCalling": true
    },
    "usage": {
      "chat": "kimi2.5",
      "extraction": "kimi2.5",
      "embedding": "kimi2.5"
    }
  },
  
  "srs": {
    "algorithm": "fsrs",
    "newCardsPerDay": 20,
    "reviewsPerDay": 100,
    "targetRetention": 0.9
  },
  
  "sync": {
    "enabled": true,
    "provider": "webdav",
    "autoSync": true,
    "syncInterval": 900,
    "webdav": {
      "url": "https://dav.jianguoyun.com/dav/negative-entropy/",
      "username": "user@example.com",
      "password": "encrypted..."
    }
  },
  
  "ui": {
    "theme": "dark",
    "language": "zh-CN",
    "fontSize": 14,
    "showSyncStatus": true
  }
}
```

---

## 7. 下一步行动

1. **初始化开源仓库**
   - 创建GitHub仓库
   - 添加LICENSE (MIT)
   - 添加README和CONTRIBUTING

2. **项目脚手架**
   - 初始化Tauri项目
   - 配置React + TypeScript
   - 添加基础UI组件库

3. **数据库初始化**
   - 创建SQLite schema
   - 添加迁移脚本

4. **AI模块开发**
   - 实现AI Client Factory
   - 添加kimi2.5默认配置
   - 测试多模态功能

5. **同步模块开发**
   - 实现WebDAV Provider
   - 添加同步界面

---

*架构更新完成 - 配置决策已确认，可开始开发*
