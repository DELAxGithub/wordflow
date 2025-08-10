# IELTS Writing Practice App - 実装引き継ぎ書

## 📋 プロジェクト概要

### MVP版開発計画
**目標**: 過剰仕様を抑制し、12週間でリリース可能なIELTS特化タイピング練習アプリを実装

### 完了済み設計作業
- ✅ **requirements.md**: MVP版機能要件（51%工数削減）
- ✅ **technical-spec.md**: 簡素化アーキテクチャ（70%複雑度削減）
- ✅ **implementation-plan.md**: 12週間開発計画（40%期間短縮）
- ✅ **ui-design-mvp.md**: 単画面UI設計

---

## 🎯 MVP版核心方針

### 設計哲学
**「必要最小限で使いやすいシンプルなインターフェース」**

#### 重要な削減・後回し項目
- ❌ **F004**: 8種類エラー分類 → 基本カウントのみ
- ❌ **F006**: 4段階習得度管理 → 基本リスト表示のみ
- ❌ **複雑UI**: 多画面構成 → 単一メイン画面
- ❌ **高精度目標**: 数値達成優先 → 計測システム優先

---

## 🏗️ 技術実装要件

### コアアーキテクチャ
```
┌─────────────────────────────────────┐
│ Presentation Layer (SwiftUI)       │
│ └─ BasicTypingPracticeView (Core)  │
├─────────────────────────────────────┤
│ Business Logic (Simplified)        │
│ ├─ TypingTestManager               │
│ ├─ BasicTTSManager                 │
│ └─ BasicErrorCounter               │
├─────────────────────────────────────┤
│ Data Layer (Essential)             │
│ ├─ IELTSTask Model                 │
│ └─ TypingResult Model              │
└─────────────────────────────────────┘
```

### プラットフォーム要件
- **macOS**: 14.0+ (Sonoma)
- **Swift**: 5.9+
- **フレームワーク**: SwiftUI, SwiftData, AVFoundation
- **外部依存**: なし（Native frameworks only）

---

## 📊 データモデル実装

### 1. IELTSTask（MVP版簡素化）
```swift
@Model
final class IELTSTask {
    @Attribute(.unique) var id: UUID
    var taskType: TaskType
    var topic: String
    var modelAnswer: String
    var targetBandScore: Double
    var wordCount: Int
    var createdDate: Date
    var lastUsedDate: Date?
    
    @Relationship(deleteRule: .cascade)
    var typingResults: [TypingResult] = []
}

enum TaskType: String, CaseIterable, Codable {
    case task1 = "task1"  // 150 words
    case task2 = "task2"  // 250 words
}
```

### 2. TypingResult（MVP版簡素化）
```swift
@Model
final class TypingResult {
    @Attribute(.unique) var id: UUID
    var sessionDate: Date
    var testDuration: TimeInterval
    var targetText: String
    var userInput: String
    var finalWPM: Double
    var accuracy: Double
    var completionPercentage: Double
    var basicErrorCount: Int // 簡素化: エラーカウントのみ
    
    @Relationship(deleteRule: .nullify)
    var task: IELTSTask?
}
```

---

## ⚙️ Manager Classes実装

### 1. TypingTestManager（MVP版）
```swift
@MainActor
@Observable
final class TypingTestManager {
    // Test state
    private(set) var isActive: Bool = false
    private(set) var isPaused: Bool = false
    private(set) var remainingTime: TimeInterval = 120 // 2分
    private(set) var elapsedTime: TimeInterval = 0
    
    // Metrics（基本のみ）
    private(set) var currentWPM: Double = 0
    private(set) var accuracy: Double = 100
    private(set) var completionPercentage: Double = 0
    
    // Configuration（緩和）
    let timeLimit: TimeInterval = 120
    let updateInterval: TimeInterval = 0.2 // 200ms（100ms→200ms緩和）
    
    func startTest(with task: IELTSTask)
    func updateInput(_ input: String)
    func pauseTest()
    func resumeTest()
    func endTest() -> TypingResult?
}
```

### 2. BasicTTSManager（大幅簡素化）
```swift
@MainActor
@Observable
final class BasicTTSManager: NSObject {
    private let synthesizer = AVSpeechSynthesizer()
    
    // 基本状態のみ
    private(set) var isPlaying: Bool = false
    private(set) var isPaused: Bool = false
    var playbackSpeed: TTSSpeed = .normal
    
    // MVP版: 全文再生のみ
    func playFullText(_ text: String)
    func pause()
    func resume()
    func stop()
    func setSpeed(_ speed: TTSSpeed)
}

enum TTSSpeed: String, CaseIterable {
    case slow = "0.75x"
    case normal = "1.0x"
    case fast = "1.25x"
}
```

### 3. BasicErrorCounter（大幅簡素化）
```swift
struct BasicErrorCounter {
    func countBasicErrors(input: String, target: String) -> BasicErrorInfo {
        // 簡単な文字比較のみ
        // 詳細分析は v2.0 で実装
    }
}

struct BasicErrorInfo {
    let totalErrors: Int
    let errorRate: Double
    let errorPositions: [Int]
}
```

---

## 🖼️ UI実装要件

### メイン画面構成（1画面のみ）
```
┌─────────────────────────────────────────┐
│           IELTS Typing Practice         │
├─────────────────────────────────────────┤
│  ┌─────────────┬─────────────────────┐   │
│  │ Target Text │    Your Input       │   │
│  │   (45%)     │      (45%)          │   │
│  │             │                     │   │
│  │ [Play TTS]  │  Here you type...   │   │
│  └─────────────┴─────────────────────┘   │
│  ┌─────────────────────────────────────┐   │
│  │   [Start] [Pause] [Stop]            │   │
│  │ ⏱️ 1:23  📊 45WPM  🎯 92%  📈 85% │   │
│  │ (Basic Statistics - 10%)            │   │
│  └─────────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

### UI Components実装
```swift
// 1. メイン画面
struct BasicTypingPracticeView: View

// 2. 基本パネル
struct BasicTargetTextPanel: View
struct BasicInputArea: View
struct BasicStatisticsPanel: View

// 3. デザインシステム（簡素化）
extension Color {
    static let mvpPrimary = Color.blue
    static let mvpSecondary = Color.gray
    static let mvpSuccess = Color.green
    static let mvpError = Color.red
}
```

---

## 📅 実装スケジュール（12週間）

### Phase 1: Core IELTS機能（週1-4）
**Week 1-2**: データモデル・基本アーキテクチャ
- [ ] IELTSTask, TypingResult モデル実装
- [ ] SwiftData統合・Repository実装

**Week 3-4**: タイピング測定エンジン
- [ ] TypingTestManager実装
- [ ] 基本WPM・正確性計算
- [ ] 2分タイマー機能

### Phase 2: TTS・基本分析（週5-8）
**Week 5-6**: TTS システム
- [ ] BasicTTSManager実装
- [ ] 全文再生機能（3速度）
- [ ] US English固定音声

**Week 7-8**: 基本エラー分析
- [ ] BasicErrorCounter実装
- [ ] 文字比較ベース分析
- [ ] 基本統計表示

### Phase 3: UI・データ管理（週9-11）
**Week 9-10**: メインUI実装
- [ ] BasicTypingPracticeView作成
- [ ] 3パネル構成実装
- [ ] リアルタイム統計表示

**Week 11**: データ出力・保存
- [ ] 基本CSV出力
- [ ] 練習結果保存機能

### Phase 4: 最終調整（週12）
**Week 12**: テスト・リリース準備
- [ ] 統合テスト
- [ ] パフォーマンス確認
- [ ] リリース準備

---

## 🎯 重要な実装ポイント

### 1. MVP版削除・後回し機能
**完全に実装しない要素**:
- ❌ 詳細エラー分析（8種類分類）
- ❌ 復習リストシステム
- ❌ 高度なTTS制御（文・フレーズ単位）
- ❌ 複雑な統計ダッシュボード
- ❌ 設定画面
- ❌ アニメーション効果

### 2. 性能要件の現実化
**計測システム重視**:
```swift
struct BasicPerformanceMonitor {
    func measureExecutionTime<T>(_ operation: () throws -> T) -> (result: T, timeMs: Double)
    func logPerformance(_ metric: String, value: Double, unit: String = "ms")
}
```

**目標設定**:
- ❌ 「入力遅延50ms以内」→ ✅ 「実用的応答性（計測後改善）」
- ❌ 「クラッシュ率0.01%未満」→ ✅ 「基本的な安定性」

### 3. データ構造の簡素化
- **IELTSTask**: 基本情報のみ（ModuleType, keyVocabulary削除）
- **TypingResult**: 基本メトリクス（qualityIndex, 詳細分析削除）
- **ErrorResult/ReviewItem**: MVP版では未実装

---

## 🧪 テスト要件

### 基本テストカバレッジ
```swift
// 1. コアロジックテスト
class TypingTestManagerTests: XCTestCase {
    func testWPMCalculation()
    func testAccuracyCalculation()
    func testTimerFunctionality()
}

// 2. データモデルテスト
class IELTSTaskTests: XCTestCase {
    func testTaskCreation()
    func testResultRelationship()
}

// 3. TTS機能テスト
class BasicTTSManagerTests: XCTestCase {
    func testTextPlayback()
    func testSpeedControl()
}
```

### パフォーマンステスト
- 入力応答性測定（目標: 実用的レベル）
- TTS応答時間測定
- メモリ使用量監視

---

## 🚀 デプロイ・リリース要件

### App Store準備
```xml
<!-- App.entitlements（Sandbox設定） -->
<key>com.apple.security.app-sandbox</key>
<true/>
<key>com.apple.security.files.user-selected.read-write</key>
<true/>
```

### 最小動作要件
- **macOS**: 14.0+
- **メモリ**: 100MB以上
- **ディスク**: 50MB以上

---

## 📈 v2.0拡張計画（参考）

### MVP版フィードバック後の実装予定
**高優先**: 
- 詳細エラー分析（8種類分類）
- 文・フレーズ単位TTS
- リアルタイム差分ハイライト

**中優先**:
- 復習リストシステム
- 統計ダッシュボード
- 設定画面

**低優先**:
- アニメーション効果
- 高度なカスタマイズ
- クラウド同期

---

## ⚠️ 実装時の注意事項

### 1. 過剰実装の回避
- MVP仕様書に明記されていない機能は実装しない
- v2.0機能の先行実装は禁止
- シンプル性を最優先

### 2. パフォーマンス
- 計測システムを最初に実装
- 数値目標より実用性を重視
- 最適化は計測結果を見てから実施

### 3. UI/UX
- 単一メイン画面を維持
- 複雑なナビゲーションは追加しない
- システムデフォルト使用（カスタムスタイル最小限）

---

## 📞 引き継ぎ完了チェックリスト

- [ ] 設計仕様書の理解（requirements.md, technical-spec.md, ui-design-mvp.md）
- [ ] MVP版削除機能の確認（v2.0まで実装禁止）
- [ ] 12週間スケジュールの確認
- [ ] 基本パフォーマンス計測システムの理解
- [ ] SwiftUI + SwiftData + AVFoundation構成の確認

---

**総開発工数**: 450時間（元920時間の51%削減）
**開発期間**: 12週間（元20週間の40%短縮）
**成功基準**: IELTS受験者が即座に使える基本的なタイピング練習環境の提供

---

*この引き継ぎ書は、過剰仕様を抑制したMVP版IELTS Writing Practice Appの実装に必要な全ての情報を含んでいます。v2.0での機能拡張を前提とした段階的開発を成功させるため、MVP仕様の厳守をお願いします。*