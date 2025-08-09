# Wordflow アプリケーション設計書

## 1. システム全体設計

### 1.1 アーキテクチャ概要
```
┌─────────────────────────────────────────┐
│           Wordflow macOS App            │
├─────────────────────────────────────────┤
│ SwiftUI Presentation Layer              │
│ ├─ DocumentEditor View                  │
│ ├─ ProjectManager View                  │
│ ├─ Dashboard View                       │
│ └─ Settings View                        │
├─────────────────────────────────────────┤
│ Business Logic Layer (MVVM)             │
│ ├─ DocumentViewModel                    │
│ ├─ ProjectViewModel                     │
│ ├─ AnalyticsViewModel                   │
│ └─ SettingsViewModel                    │
├─────────────────────────────────────────┤
│ Data Layer (Repository Pattern)         │
│ ├─ DocumentRepository                   │
│ ├─ ProjectRepository                    │
│ ├─ AnalyticsRepository                  │
│ └─ SettingsRepository                   │
├─────────────────────────────────────────┤
│ Persistence Layer                       │
│ ├─ SwiftData Models                     │
│ ├─ FileSystem Integration               │
│ └─ Backup & Export                      │
└─────────────────────────────────────────┘
```

### 1.2 技術スタック詳細
**コア技術**:
- **プラットフォーム**: macOS 14.0+ (Sonoma)
- **言語**: Swift 5.9+
- **UI**: SwiftUI
- **データ**: SwiftData (Core Data)
- **アーキテクチャ**: MVVM + Repository Pattern

**DELAX共通技術遺産統合**:
- **SwiftUIQualityKit**: 品質管理・バグ検出システム
- **swift-ui-components**: 再利用可能UIコンポーネント
- **claude-integration**: AI支援開発ワークフロー

## 2. データモデル設計

### 2.1 SwiftDataモデル設計

```swift
// 文書モデル
@Model
final class Document {
    var id: UUID
    var title: String
    var content: String
    var createdAt: Date
    var updatedAt: Date
    var wordCount: Int
    var characterCount: Int
    var estimatedReadingTime: Int // 分
    
    // 関係
    var project: Project?
    var tags: [Tag] = []
    var sessions: [WritingSession] = []
    
    // メタデータ
    var documentType: DocumentType
    var status: DocumentStatus
    var priority: Priority
    
    init(title: String, content: String = "", project: Project? = nil) {
        self.id = UUID()
        self.title = title
        self.content = content
        self.createdAt = Date()
        self.updatedAt = Date()
        self.project = project
        self.documentType = .markdown
        self.status = .draft
        self.priority = .medium
        updateCounts()
    }
    
    private func updateCounts() {
        self.wordCount = content.components(separatedBy: .whitespacesAndNewlines)
            .filter { !$0.isEmpty }.count
        self.characterCount = content.count
        self.estimatedReadingTime = max(1, wordCount / 200) // 200 words/minute
    }
}

// プロジェクトモデル
@Model
final class Project {
    var id: UUID
    var name: String
    var description: String
    var createdAt: Date
    var updatedAt: Date
    var deadline: Date?
    var targetWordCount: Int?
    var status: ProjectStatus
    
    // 関係
    var documents: [Document] = []
    var tasks: [Task] = []
    var milestones: [Milestone] = []
    
    // 計算プロパティ
    var totalWordCount: Int {
        documents.reduce(0) { $0 + $1.wordCount }
    }
    
    var progress: Double {
        guard let target = targetWordCount, target > 0 else { return 0 }
        return min(1.0, Double(totalWordCount) / Double(target))
    }
    
    init(name: String, description: String = "") {
        self.id = UUID()
        self.name = name
        self.description = description
        self.createdAt = Date()
        self.updatedAt = Date()
        self.status = .planning
    }
}

// 執筆セッションモデル（生産性分析用）
@Model
final class WritingSession {
    var id: UUID
    var startTime: Date
    var endTime: Date?
    var wordCountStart: Int
    var wordCountEnd: Int?
    var document: Document?
    
    // 計算プロパティ
    var duration: TimeInterval {
        guard let end = endTime else { return Date().timeIntervalSince(startTime) }
        return end.timeIntervalSince(startTime)
    }
    
    var wordsWritten: Int {
        guard let end = wordCountEnd else { return 0 }
        return max(0, end - wordCountStart)
    }
    
    var wordsPerMinute: Double {
        let minutes = duration / 60
        return minutes > 0 ? Double(wordsWritten) / minutes : 0
    }
}

// タグモデル
@Model
final class Tag {
    var id: UUID
    var name: String
    var color: String // Hex color code
    var documents: [Document] = []
    
    init(name: String, color: String = "#007AFF") {
        self.id = UUID()
        self.name = name
        self.color = color
    }
}

// 列挙型
enum DocumentType: String, CaseIterable, Codable {
    case markdown = "markdown"
    case plainText = "plainText"
    case richText = "richText"
}

enum DocumentStatus: String, CaseIterable, Codable {
    case draft = "draft"
    case inReview = "inReview"
    case completed = "completed"
    case published = "published"
    case archived = "archived"
}

enum ProjectStatus: String, CaseIterable, Codable {
    case planning = "planning"
    case active = "active"
    case onHold = "onHold"
    case completed = "completed"
    case cancelled = "cancelled"
}

enum Priority: String, CaseIterable, Codable {
    case low = "low"
    case medium = "medium"
    case high = "high"
    case urgent = "urgent"
}
```

### 2.2 データ関係設計
```
Project (1) ←→ (N) Document
Document (N) ←→ (N) Tag
Document (1) ←→ (N) WritingSession
Project (1) ←→ (N) Task
Project (1) ←→ (N) Milestone
```

## 3. UI/UX設計

### 3.1 全体UI構造
```
WindowGroup {
    ContentView() // NavigationSplitView
    ├─ Sidebar
    │  ├─ Projects Section
    │  ├─ Recent Documents
    │  ├─ Tags Section
    │  └─ Analytics
    ├─ Main Content
    │  ├─ Document List (Middle)
    │  └─ Document Editor (Detail)
    └─ Inspector Panel (Optional)
       ├─ Document Info
       ├─ Writing Statistics
       └─ Session Analytics
}
```

### 3.2 主要ビュー設計

**DocumentEditorView** (DELAX swift-ui-components活用)
```swift
struct DocumentEditorView: View {
    @Bindable var document: Document
    @State private var isPreviewMode = false
    @State private var currentSession: WritingSession?
    
    var body: some View {
        HSplitView {
            // メインエディター
            TextEditor(text: $document.content)
                .font(.system(.body, design: .serif))
                .lineSpacing(4)
                .padding()
                .onChange(of: document.content) { updateDocumentStats() }
            
            // プレビューパネル（オプション）
            if isPreviewMode {
                MarkdownPreview(content: document.content)
                    .frame(minWidth: 300)
            }
        }
        .toolbar {
            DocumentToolbar(
                document: document,
                isPreviewMode: $isPreviewMode,
                currentSession: $currentSession
            )
        }
        .inspector(isPresented: .constant(true)) {
            DocumentInspector(document: document)
        }
    }
}
```

**ProjectManagerView**
```swift
struct ProjectManagerView: View {
    @Query private var projects: [Project]
    @State private var selectedProject: Project?
    
    var body: some View {
        NavigationSplitView {
            // プロジェクト一覧
            List(projects, selection: $selectedProject) { project in
                ProjectCard(project: project)
            }
        } detail: {
            if let project = selectedProject {
                ProjectDetailView(project: project)
            } else {
                Text("Select a project")
            }
        }
    }
}
```

**DashboardView** (分析・統計画面)
```swift
struct DashboardView: View {
    @Query private var sessions: [WritingSession]
    @Query private var documents: [Document]
    
    var body: some View {
        ScrollView {
            LazyVStack(spacing: 20) {
                // 今日の統計
                TodayStatsCard(sessions: todaySessions)
                
                // 週間進捗
                WeeklyProgressChart(sessions: weekSessions)
                
                // 生産性分析
                ProductivityAnalysis(sessions: sessions)
                
                // 目標進捗
                GoalProgressView(documents: documents)
            }
            .padding()
        }
    }
}
```

### 3.3 DELAX UIコンポーネント活用
**統合予定のコンポーネント**:
- **DelaxTextEditor**: バグレポート統合テキストエディター
- **DelaxProgressCard**: 進捗表示カード
- **DelaxAnalyticsChart**: 分析グラフコンポーネント
- **DelaxQualityIndicator**: 品質指標表示

## 4. システム機能設計

### 4.1 コア機能アーキテクチャ

**文書管理システム**
```swift
class DocumentRepository: ObservableObject {
    private let modelContext: ModelContext
    
    func create(title: String, project: Project? = nil) -> Document
    func update(_ document: Document)
    func delete(_ document: Document)
    func search(query: String) -> [Document]
    func filterByStatus(_ status: DocumentStatus) -> [Document]
    func filterByProject(_ project: Project) -> [Document]
    func export(_ document: Document, format: ExportFormat) throws -> URL
}
```

**プロジェクト管理システム**
```swift
class ProjectRepository: ObservableObject {
    private let modelContext: ModelContext
    
    func create(name: String, description: String) -> Project
    func update(_ project: Project)
    func delete(_ project: Project)
    func calculateProgress(_ project: Project) -> Double
    func getProjectStats(_ project: Project) -> ProjectStats
    func setDeadline(_ project: Project, deadline: Date)
}
```

**分析システム**
```swift
class AnalyticsRepository: ObservableObject {
    private let modelContext: ModelContext
    
    func startWritingSession(for document: Document) -> WritingSession
    func endWritingSession(_ session: WritingSession)
    func getDailyStats(date: Date) -> DailyStats
    func getWeeklyStats(week: Date) -> WeeklyStats
    func getProductivityTrends(period: TimePeriod) -> [ProductivityData]
    func getGoalProgress() -> GoalProgress
}
```

### 4.2 ワークフロー最適化機能

**集中モード**
```swift
class FocusMode: ObservableObject {
    @Published var isActive = false
    @Published var sessionType: FocusSessionType = .pomodoro
    @Published var remainingTime: TimeInterval = 0
    
    func startSession(type: FocusSessionType, duration: TimeInterval)
    func pauseSession()
    func endSession()
    func configureEnvironment(hideDistractions: Bool, enableNotifications: Bool)
}
```

**自動保存・バックアップ**
```swift
class AutoSaveManager: ObservableObject {
    private var autoSaveTimer: Timer?
    private var backupManager: BackupManager
    
    func startAutoSave(interval: TimeInterval = 30)
    func performBackup()
    func scheduleBackup(interval: TimeInterval = 3600) // 1時間毎
    func exportToiCloudDrive()
}
```

## 5. DELAX共通技術遺産統合設計

### 5.1 SwiftUIQualityKit統合
```swift
// App初期化時に統合
@main
struct WordflowApp: App {
    @StateObject private var qualityManager = DelaxQualityManager()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(qualityManager)
                .onAppear {
                    // SwiftUIQualityKit初期化
                    qualityManager.initialize()
                    qualityManager.startMonitoring()
                }
        }
        .modelContainer(sharedModelContainer)
    }
}
```

### 5.2 品質監視統合
```swift
class DelaxQualityManager: ObservableObject {
    @Published var qualityScore: Double = 1.0
    @Published var detectedIssues: [QualityIssue] = []
    
    func initialize() {
        // SwiftUIQualityKitセットアップ
    }
    
    func startMonitoring() {
        // リアルタイム品質監視開始
    }
    
    func reportIssue(_ issue: QualityIssue) {
        // バグレポート処理
    }
}
```

### 5.3 Claude統合ワークフロー
```swift
// 開発支援機能（開発時のみ）
class ClaudeIntegration {
    func generateDocumentTemplate(type: DocumentType) -> String
    func suggestImprovement(for content: String) -> [String]
    func analyzeWritingPattern(sessions: [WritingSession]) -> WritingAnalysis
    func optimizeUI(component: String) -> SwiftUICode
}
```

## 6. パフォーマンス設計

### 6.1 最適化戦略
- **遅延読み込み**: 大きなドキュメントのページング
- **インクリメンタル検索**: リアルタイム検索結果
- **バックグラウンド処理**: 統計計算・バックアップ
- **メモリ管理**: 適切なモデルライフサイクル

### 6.2 パフォーマンス目標
- **起動時間**: 3秒以内
- **文書切り替え**: 200ms以内
- **検索結果**: 500ms以内
- **自動保存**: バックグラウンドで非同期実行

## 7. セキュリティ・プライバシー設計

### 7.1 データ保護
- **App Sandbox**: macOS標準のセキュリティモデル
- **暗号化**: 機密文書の自動暗号化オプション
- **アクセス制御**: ファイルシステム権限管理
- **データ復旧**: 定期バックアップと履歴管理

### 7.2 プライバシー設計
- **ローカル処理**: データはローカル環境で完結
- **透明性**: データ使用方法の明確な説明
- **ユーザー制御**: データ共有・分析のオプトイン

## 8. 拡張性設計

### 8.1 プラグインアーキテクチャ（将来）
```swift
protocol WordflowPlugin {
    var identifier: String { get }
    var name: String { get }
    var version: String { get }
    
    func initialize(context: PluginContext)
    func processDocument(_ document: Document) -> Document
    func contributeUI() -> [PluginUIComponent]
}
```

### 8.2 API設計（将来）
- **Export API**: 外部アプリとの連携
- **Import API**: 他のツールからのデータ移行
- **Webhook**: 外部サービス統合
- **Scripting**: 自動化スクリプト対応

## 9. 品質保証設計

### 9.1 テスト戦略
- **単体テスト**: ビジネスロジック・データモデル
- **統合テスト**: SwiftData永続化・ファイルI/O
- **UIテスト**: 主要ユーザーフロー
- **パフォーマンステスト**: 大きなドキュメント処理

### 9.2 DELAX品質保証統合
- **SwiftUIQualityKit**: 自動品質監視
- **バグレポート**: ユーザーフィードバック収集
- **品質メトリクス**: 継続的な品質測定
- **自動修正**: 一般的な問題の自動解決

## 10. 実装優先順位

### 10.1 MVP（Minimum Viable Product）
1. **基本文書管理**: CRUD操作
2. **シンプルエディター**: プレーンテキスト編集
3. **プロジェクト機能**: 基本的なプロジェクト管理
4. **自動保存**: データ損失防止

### 10.2 フェーズ2（拡張機能）
1. **マークダウン対応**: 構文ハイライト・プレビュー
2. **分析機能**: 執筆統計・生産性分析
3. **集中モード**: ポモドーロタイマー統合
4. **高度な検索**: 全文検索・フィルタリング

### 10.3 フェーズ3（最適化）
1. **DELAX統合完了**: 全コンポーネント統合
2. **パフォーマンス最適化**: 大きなプロジェクト対応
3. **エクスポート機能**: 多様な形式対応
4. **アクセシビリティ**: VoiceOver完全対応

この設計書は、requirements.mdで定義された要件を満たし、DELAX共通技術遺産を効果的に活用する包括的な設計を提供します。