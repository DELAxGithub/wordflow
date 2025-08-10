# IELTS Writing Practice App - MVP版実装計画書

## 📋 実装概要

この文書では、IELTS Writing Practice App MVP版の迅速な実装計画、開発マイルストーン、リソース配分を定義します。

### MVP版開発方針
- **シンプル第一**: コア機能に集中、複雑な機能は後回し
- **迅速なリリース**: 3-4ヶ月での市場投入
- **段階的改善**: MVP版でフィードバック収集後、機能を段階的に追加
- **計測重視**: パフォーマンス数値は計測システム実装後に改善

---

## 🚀 全体開発フェーズ

### MVP版 Phase 概要
| Phase | 期間 | 目標 | 重要度 | 成果物 |
|-------|------|------|--------|--------|
| **Phase 1** | 4週間 | Core IELTS機能 | Critical | 基本MVP版 |
| **Phase 2** | 4週間 | 基本TTS・簡易分析 | High | 機能強化版 |
| **Phase 3** | 3週間 | 最小復習・データ出力 | Medium | リリース候補版 |
| **Phase 4** | 1週間 | 最終調整・リリース | High | 製品版 |

### MVP版総開発工数
- **総期間**: 12週間（3ヶ月）
- **総工数**: 450時間
- **平均週間工数**: 38時間
- **開発者**: 1名想定
- **削減効果**: 51%の工数削減、40%の期間短縮

---

## 📅 Phase 1: Core IELTS機能 (Week 1-4) - MVP版簡素化

### 🎯 Phase 1 目標
**IELTS特化タイピング練習の基盤構築（簡素化版）**
- 基本タイピングテスト機能
- 簡素化された測定システム
- 最小限のUI/UX
- 基本データ永続化機能

### Week 1-2: MVP版アーキテクチャ・データ基盤

#### Week 1: MVP版プロジェクト基盤
**目標**: 開発環境とコア構造の構築
```swift
// 作業項目
✅ 新規Xcodeプロジェクト作成
✅ SwiftData統合・データモデル定義  
✅ 基本アーキテクチャ実装(MVVM + Repository)
✅ エラーハンドリング基盤
```

**主要成果物（MVP版簡素化）**:
- [ ] `IELTSApp.swift` - メインアプリケーション
- [ ] `Models/` フォルダ - IELTSTask, TypingResult（簡素化版）
- [ ] `Repositories/` フォルダ - 基本Repository実装
- [ ] `ViewModels/` フォルダ - 簡素化MVVM基盤
- [ ] `Utils/ErrorHandler.swift` - 基本エラーハンドリング

**技術詳細**:
```swift
// IELTSTask.swift - コアデータモデル
@Model
final class IELTSTask {
    @Attribute(.unique) var id: UUID
    var taskType: TaskType
    var topic: String
    var modelAnswer: String
    var targetBandScore: Double
    var wordCount: Int
    var createdDate: Date
    
    @Relationship(deleteRule: .cascade)
    var typingResults: [TypingResult] = []
    
    init(taskType: TaskType, topic: String, modelAnswer: String, targetBandScore: Double) {
        self.id = UUID()
        self.taskType = taskType
        self.topic = topic
        self.modelAnswer = modelAnswer
        self.targetBandScore = targetBandScore
        self.wordCount = modelAnswer.count // MVP版: 簡素化（文字数ベース）
        self.createdDate = Date()
    }
}

// MVP版 TypingResult.swift - 簡素化測定結果モデル
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
    var basicErrorCount: Int // MVP版: 基本エラーカウントのみ
    
    @Relationship(deleteRule: .nullify, inverse: \IELTSTask.typingResults)
    var task: IELTSTask?
    
    init(task: IELTSTask, userInput: String, testDuration: TimeInterval) {
        self.id = UUID()
        self.sessionDate = Date()
        self.task = task
        self.userInput = userInput
        self.testDuration = testDuration
        self.targetText = task.modelAnswer
        // MVP版: 簡素化された計算
        self.calculateBasicMetrics()
    }
    
    // MVP版で削除・後回し
    // ~~qualityIndex~~ → v2.0
    // ~~詳細パフォーマンスメトリクス~~ → v2.0
}
```

#### Week 2: MVP版Repository Pattern & サンプルデータ
**目標**: 基本データアクセス層の完成
```swift
// MVP版作業項目
✅ 基本IELTSTaskRepository実装
✅ 基本TypingResultRepository実装
✅ サンプルIELTS模範解答20問作成（最低限）
✅ 基本データ保存機能
// ~~複雑なマイグレーション機能~~ → v2.0
// ~~詳細な単体テスト~~ → v2.0（基本テストのみ）
```

**技術詳細**:
```swift
// IELTSTaskRepository.swift
protocol IELTSTaskRepositoryProtocol {
    func fetchAll() async throws -> [IELTSTask]
    func fetchByTaskType(_ taskType: TaskType) async throws -> [IELTSTask]
    func fetchByBandScore(_ bandScore: Double) async throws -> [IELTSTask]
    func save(_ task: IELTSTask) async throws
    func delete(id: UUID) async throws
}

class SwiftDataIELTSTaskRepository: IELTSTaskRepositoryProtocol {
    private let modelContext: ModelContext
    
    func fetchAll() async throws -> [IELTSTask] {
        let descriptor = FetchDescriptor<IELTSTask>(
            sortBy: [SortDescriptor(\.lastUsedDate, order: .reverse)]
        )
        return try modelContext.fetch(descriptor)
    }
    
    func fetchByTaskType(_ taskType: TaskType) async throws -> [IELTSTask] {
        let descriptor = FetchDescriptor<IELTSTask>(
            predicate: #Predicate { $0.taskType == taskType },
            sortBy: [SortDescriptor(\.targetBandScore)]
        )
        return try modelContext.fetch(descriptor)
    }
}
```

### Week 3-4: MVP版タイピング測定エンジン

#### Week 3: MVP版基本測定機能
**目標**: 簡素化された測定システムの実装
```swift
// MVP版作業項目
✅ 基本TypingTestManager実装
✅ 基本WPM計算アルゴリズム（総文字数ベース）
✅ 基本正確性計算アルゴリズム
✅ 2分タイマー機能
// ~~品質指数計算~~ → v2.0
// ~~瞬間WPM、平均WPM~~ → v2.0
```

**技術詳細**:
```swift
// TypingTestManager.swift
@MainActor
@Observable
final class TypingTestManager {
    // 測定状態
    private(set) var isActive: Bool = false
    private(set) var remainingTime: TimeInterval = 120 // 2分
    private(set) var currentWPM: Double = 0
    private(set) var accuracy: Double = 100
    private(set) var qualityIndex: Double = 0
    private(set) var completionPercentage: Double = 0
    
    // 内部状態
    private var startTime: Date?
    private var timer: Timer?
    private var currentTask: IELTSTask?
    private var targetText: String = ""
    private var userInput: String = ""
    
    // MVP版設定（緩和）
    private let timeLimit: TimeInterval = 120
    private let updateInterval: TimeInterval = 0.2 // 200ms更新（100ms→200msに緩和）
    
    func startTest(with task: IELTSTask) {
        self.currentTask = task
        self.targetText = task.modelAnswer
        self.startTime = Date()
        self.isActive = true
        self.remainingTime = timeLimit
        
        resetMetrics()
        startTimer()
    }
    
    func updateInput(_ input: String) {
        guard isActive else { return }
        self.userInput = input
        calculateMetrics()
    }
    
    func endTest() -> TypingResult? {
        guard let task = currentTask, let startTime = startTime else { return nil }
        
        isActive = false
        timer?.invalidate()
        
        let testDuration = Date().timeIntervalSince(startTime)
        let result = TypingResult(
            task: task,
            testDuration: testDuration,
            finalWPM: currentWPM,
            accuracy: accuracy
        )
        
        return result
    }
    
    private func calculateMetrics() {
        guard let startTime = startTime else { return }
        
        let elapsedTime = Date().timeIntervalSince(startTime)
        let minutes = elapsedTime / 60.0
        
        // MVP版: 基本WPM計算（総文字数ベース）
        let totalChars = Double(userInput.count)
        currentWPM = minutes > 0 ? (totalChars / 5.0) / minutes : 0
        
        // MVP版: 基本正確性計算
        let correctChars = Double(calculateCorrectCharacters())
        accuracy = totalChars > 0 ? (correctChars / totalChars) * 100 : 100
        
        // ~~品質指数: 削除~~ → v2.0
        
        // 完了率
        let targetLength = Double(targetText.count)
        completionPercentage = targetLength > 0 ? (totalChars / targetLength) * 100 : 0
    }
    
    private func calculateCorrectCharacters() -> Int {
        let target = Array(targetText)
        let input = Array(userInput)
        var correctCount = 0
        
        for (index, char) in input.enumerated() {
            if index < target.count && char == target[index] {
                correctCount += 1
            }
        }
        
        return correctCount
    }
    
    private func startTimer() {
        timer = Timer.scheduledTimer(withTimeInterval: updateInterval, repeats: true) { [weak self] _ in
            self?.updateTimer()
        }
    }
    
    private func updateTimer() {
        guard let startTime = startTime else { return }
        
        let elapsed = Date().timeIntervalSince(startTime)
        remainingTime = max(0, timeLimit - elapsed)
        
        if remainingTime <= 0 {
            _ = endTest()
        }
    }
    
    private func resetMetrics() {
        currentWPM = 0
        accuracy = 100
        qualityIndex = 0
        completionPercentage = 0
        userInput = ""
    }
}
```

#### Week 4: MVP版基本最適化
**目標**: 実用的なパフォーマンスの確保
```swift
// MVP版作業項目
✅ 基本的なパフォーマンス確保
✅ 基本メモリ管理
✅ 簡素化アルゴリズム実装
✅ 基本パフォーマンス監視
// ~~高度な最適化~~ → v2.0
// ~~詳細なストレステスト~~ → v2.0
```

**MVP版基本パフォーマンス監視**:
```swift
// MVP版: 基本的なパフォーマンス計測のみ
struct BasicPerformanceMonitor {
    static let shared = BasicPerformanceMonitor()
    
    func measureExecutionTime<T>(_ operation: () throws -> T) rethrows -> (result: T, timeMs: Double) {
        let startTime = CFAbsoluteTimeGetCurrent()
        let result = try operation()
        let timeElapsed = CFAbsoluteTimeGetCurrent() - startTime
        return (result: result, timeMs: timeElapsed * 1000)
    }
    
    func logPerformance(_ metric: String, value: Double, unit: String = "ms") {
        print("Performance: \(metric) = \(value)\(unit)")
    }
}

// MVP版で削除・後回し
// ~~高度な入力デバウンス~~ → v2.0
// ~~複雑なパフォーマンス監視~~ → v2.0
```

### Week 5-6: メインUI実装

#### Week 5: 練習画面Layout
**目標**: IELTS特化UIの基本構成
```swift
// 作業項目
✅ TypingPracticeView実装 - メイン練習画面
✅ 左側: TargetTextPanel - 模範解答表示
✅ 右側: InputArea - ユーザー入力エリア
✅ 下部: StatisticsPanel - リアルタイム統計
✅ レスポンシブレイアウト対応
```

**UI実装詳細**:
```swift
// TypingPracticeView.swift - メイン練習画面
struct TypingPracticeView: View {
    @StateObject private var testManager = TypingTestManager()
    @State private var currentTask: IELTSTask?
    @State private var userInput: String = ""
    
    var body: some View {
        VStack(spacing: 16) {
            // ヘッダー
            HeaderView(
                taskTitle: currentTask?.topic ?? "IELTS Practice",
                remainingTime: testManager.remainingTime,
                isActive: testManager.isActive
            )
            
            // メインコンテンツエリア
            HStack(spacing: 16) {
                // 左側: 目標文表示
                TargetTextPanel(
                    text: currentTask?.modelAnswer ?? "",
                    wordCount: currentTask?.wordCount ?? 0,
                    bandScore: currentTask?.targetBandScore ?? 0
                )
                .frame(minWidth: 400)
                
                // 右側: 入力エリア
                InputArea(
                    input: $userInput,
                    isActive: testManager.isActive,
                    onInputChange: { input in
                        testManager.updateInput(input)
                    }
                )
                .frame(minWidth: 400)
            }
            
            // 下部: 統計パネル
            StatisticsPanel(
                wpm: testManager.currentWPM,
                accuracy: testManager.accuracy,
                qualityIndex: testManager.qualityIndex,
                completionPercentage: testManager.completionPercentage
            )
            
            // コントロールボタン
            ControlButtonsView(
                isActive: testManager.isActive,
                onStart: startTest,
                onPause: testManager.pauseTest,
                onEnd: endTest
            )
        }
        .padding()
        .onAppear(perform: loadTasks)
    }
    
    private func startTest() {
        guard let task = currentTask else { return }
        testManager.startTest(with: task)
    }
    
    private func endTest() {
        guard let result = testManager.endTest() else { return }
        // 結果保存処理
    }
}

// TargetTextPanel.swift - 目標文表示パネル
struct TargetTextPanel: View {
    let text: String
    let wordCount: Int
    let bandScore: Double
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            // ヘッダー
            HStack {
                Text("Target Text (\(wordCount) words)")
                    .font(.headline)
                    .foregroundColor(.primary)
                
                Spacer()
                
                Text("Band Score: \(String(format: "%.1f", bandScore))")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
                    .padding(.horizontal, 8)
                    .padding(.vertical, 4)
                    .background(Color.blue.opacity(0.1))
                    .cornerRadius(8)
            }
            
            Divider()
            
            // テキスト表示
            ScrollView {
                Text(text)
                    .font(.body)
                    .lineSpacing(4)
                    .multilineTextAlignment(.leading)
                    .frame(maxWidth: .infinity, alignment: .leading)
                    .padding()
                    .background(Color(NSColor.textBackgroundColor))
                    .cornerRadius(8)
            }
        }
        .padding()
        .background(Color(NSColor.controlBackgroundColor))
        .cornerRadius(12)
    }
}

// InputArea.swift - 入力エリア
struct InputArea: View {
    @Binding var input: String
    let isActive: Bool
    let onInputChange: (String) -> Void
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            // ヘッダー
            HStack {
                Text("Your Input")
                    .font(.headline)
                
                Spacer()
                
                Text("\(input.wordCount)/250 words")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
            }
            
            Divider()
            
            // テキストエディター
            TextEditor(text: $input)
                .font(.body)
                .lineSpacing(4)
                .disabled(!isActive)
                .background(Color(NSColor.textBackgroundColor))
                .cornerRadius(8)
                .onChange(of: input) { _, newValue in
                    onInputChange(newValue)
                }
            
            // 統計情報
            HStack {
                Text("Characters: \(input.count)")
                    .font(.caption)
                    .foregroundColor(.secondary)
                
                Spacer()
                
                Text("Sentences: \(input.sentenceCount)")
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
        }
        .padding()
        .background(Color(NSColor.controlBackgroundColor))
        .cornerRadius(12)
    }
}

// StatisticsPanel.swift - 統計パネル
struct StatisticsPanel: View {
    let wpm: Double
    let accuracy: Double
    let qualityIndex: Double
    let completionPercentage: Double
    
    var body: some View {
        HStack(spacing: 20) {
            StatCard(
                title: "WPM",
                value: String(format: "%.0f", wpm),
                color: .blue,
                icon: "speedometer"
            )
            
            StatCard(
                title: "Accuracy",
                value: String(format: "%.1f%%", accuracy),
                color: .green,
                icon: "target"
            )
            
            StatCard(
                title: "Quality",
                value: String(format: "%.2f", qualityIndex),
                color: .purple,
                icon: "trophy"
            )
            
            StatCard(
                title: "Progress",
                value: String(format: "%.0f%%", completionPercentage),
                color: .orange,
                icon: "chart.line.uptrend.xyaxis"
            )
        }
        .padding(.vertical, 12)
        .background(Color(NSColor.controlBackgroundColor))
        .cornerRadius(8)
    }
}

// StatCard.swift - 統計カード
struct StatCard: View {
    let title: String
    let value: String
    let color: Color
    let icon: String
    
    var body: some View {
        VStack(spacing: 8) {
            HStack(spacing: 4) {
                Image(systemName: icon)
                    .foregroundColor(color)
                    .font(.title3)
                
                Text(title)
                    .font(.caption)
                    .fontWeight(.medium)
                    .foregroundColor(.secondary)
            }
            
            Text(value)
                .font(.title2)
                .fontWeight(.semibold)
                .foregroundColor(.primary)
        }
        .frame(minWidth: 80)
        .padding(.vertical, 8)
    }
}
```

#### Week 6: UI改善・レスポンシブ対応
**目標**: ユーザビリティ向上
```swift
// 作業項目
✅ アニメーション実装
✅ キーボードショートカット
✅ アクセシビリティ対応
✅ レスポンシブレイアウト
✅ ダークモード対応
```

### Week 7-8: 基本統合・テスト

#### Week 7: エンドツーエンド統合
**目標**: 完全な練習フロー実現
```swift
// 作業項目
✅ データフロー統合テスト
✅ UI/UXフロー検証
✅ エラーケース処理
✅ パフォーマンス検証
✅ メモリリーク検証
```

#### Week 8: Phase 1 完成・検証
**目標**: MVP版の完成
```swift
// 作業項目
✅ 総合テスト実施
✅ パフォーマンス最終調整
✅ ドキュメント作成
✅ Phase 2 準備
✅ ユーザビリティテスト
```

**Phase 1 MVP版完成基準**:
- [ ] 2分タイピングテスト基本動作
- [ ] WPM・Accuracy基本計測（Quality Indexは削除）
- [ ] 実用的な入力応答性（数値は計測後改善）
- [ ] 20問以上のIELTS模範解答
- [ ] 基本データ永続化動作

---

## 📅 Phase 2: 基本TTS・簡易分析 (Week 5-8) - MVP版簡素化

### 🎯 Phase 2 目標
**基本学習支援機能の実装（簡素化版）**
- 基本TTS音声システム（全文再生のみ）
- 基本エラーカウント機能
- 3色基本差分表示
- 基本パフォーマンス計測

### Week 5-6: MVP版基本TTSシステム

#### Week 5: MVP版AVFoundation統合
**目標**: 基本音声機能実装（簡素化）
```swift
// MVP版作業項目
✅ BasicTTSManager実装
✅ AVSpeechSynthesizer統合
✅ 全文再生機能のみ（文・フレーズ再生は削除）
✅ 3段階速度調整（簡素化）
✅ US English固定（言語選択削除）
✅ 基本エラーハンドリング
// ~~詳細音声品質最適化~~ → v2.0
```

**技術実装詳細**:
```swift
// TTSManager.swift - 音声制御システム
@MainActor
@Observable
final class TTSManager: NSObject, AVSpeechSynthesizerDelegate {
    private let synthesizer = AVSpeechSynthesizer()
    
    // 状態管理
    private(set) var isPlaying: Bool = false
    private(set) var isPaused: Bool = false
    private(set) var currentPosition: TimeInterval = 0
    private(set) var totalDuration: TimeInterval = 0
    
    // 設定
    var playbackSpeed: Float = 1.0 {
        didSet { updateSettings() }
    }
    var voiceLanguage: VoiceLanguage = .usEnglish {
        didSet { updateVoice() }
    }
    var playbackMode: PlaybackMode = .full
    
    // 内部状態
    private var currentUtterance: AVSpeechUtterance?
    private var currentText: String = ""
    private var sentences: [String] = []
    private var currentSentenceIndex: Int = 0
    
    override init() {
        super.init()
        synthesizer.delegate = self
        updateVoice()
    }
    
    // MARK: - Public Methods
    
    func playFullText(_ text: String) {
        stop()
        currentText = text
        playbackMode = .full
        
        let utterance = createUtterance(from: text)
        speak(utterance)
        
        AppLogger.shared.logTTSEvent("play_full_text", duration: estimateDuration(text))
    }
    
    func playSentence(_ text: String, at index: Int = 0) {
        stop()
        currentText = text
        playbackMode = .sentence
        
        // 文に分割
        sentences = text.components(separatedBy: .sentenceTerminators)
            .map { $0.trimmingCharacters(in: .whitespacesAndNewlines) }
            .filter { !$0.isEmpty }
        
        guard index < sentences.count else { return }
        currentSentenceIndex = index
        
        let utterance = createUtterance(from: sentences[index])
        speak(utterance)
        
        AppLogger.shared.logTTSEvent("play_sentence", duration: estimateDuration(sentences[index]))
    }
    
    func playPhrase(_ phrase: String) {
        stop()
        currentText = phrase
        playbackMode = .phrase
        
        let utterance = createUtterance(from: phrase)
        speak(utterance)
        
        AppLogger.shared.logTTSEvent("play_phrase", duration: estimateDuration(phrase))
    }
    
    func pause() {
        guard isPlaying else { return }
        synthesizer.pauseSpeaking(at: .immediate)
        isPaused = true
        isPlaying = false
    }
    
    func resume() {
        guard isPaused else { return }
        synthesizer.continueSpeaking()
        isPaused = false
        isPlaying = true
    }
    
    func stop() {
        synthesizer.stopSpeaking(at: .immediate)
        isPlaying = false
        isPaused = false
        currentPosition = 0
        currentUtterance = nil
    }
    
    func setSpeed(_ speed: Float) {
        playbackSpeed = max(0.5, min(2.0, speed))
    }
    
    func setVoiceLanguage(_ language: VoiceLanguage) {
        voiceLanguage = language
        updateVoice()
    }
    
    // MARK: - Private Methods
    
    private func createUtterance(from text: String) -> AVSpeechUtterance {
        let utterance = AVSpeechUtterance(string: text)
        utterance.rate = AVSpeechUtteranceDefaultSpeechRate * playbackSpeed
        utterance.voice = getVoice(for: voiceLanguage)
        utterance.volume = 1.0
        utterance.pitchMultiplier = 1.0
        return utterance
    }
    
    private func speak(_ utterance: AVSpeechUtterance) {
        currentUtterance = utterance
        synthesizer.speak(utterance)
        isPlaying = true
        isPaused = false
    }
    
    private func updateSettings() {
        if isPlaying, let utterance = currentUtterance {
            utterance.rate = AVSpeechUtteranceDefaultSpeechRate * playbackSpeed
        }
    }
    
    private func updateVoice() {
        if isPlaying, let utterance = currentUtterance {
            utterance.voice = getVoice(for: voiceLanguage)
        }
    }
    
    private func getVoice(for language: VoiceLanguage) -> AVSpeechSynthesisVoice? {
        switch language {
        case .usEnglish:
            return AVSpeechSynthesisVoice(language: "en-US")
        case .ukEnglish:
            return AVSpeechSynthesisVoice(language: "en-GB")
        }
    }
    
    private func estimateDuration(_ text: String) -> TimeInterval {
        // 推定: 通常速度で150語/分
        let wordsPerMinute = 150.0 * Double(playbackSpeed)
        let wordCount = Double(text.wordCount)
        return (wordCount / wordsPerMinute) * 60.0
    }
    
    // MARK: - AVSpeechSynthesizerDelegate
    
    nonisolated func speechSynthesizer(_ synthesizer: AVSpeechSynthesizer, didStart utterance: AVSpeechUtterance) {
        Task { @MainActor in
            isPlaying = true
            isPaused = false
        }
    }
    
    nonisolated func speechSynthesizer(_ synthesizer: AVSpeechSynthesizer, didFinish utterance: AVSpeechUtterance) {
        Task { @MainActor in
            isPlaying = false
            isPaused = false
            currentPosition = 0
            
            // 文モードで次の文を自動再生
            if playbackMode == .sentence && currentSentenceIndex < sentences.count - 1 {
                playNextSentence()
            }
        }
    }
    
    private func playNextSentence() {
        currentSentenceIndex += 1
        let utterance = createUtterance(from: sentences[currentSentenceIndex])
        speak(utterance)
    }
}

// VoiceLanguage.swift - 音声言語定義
enum VoiceLanguage: String, CaseIterable {
    case usEnglish = "en-US"
    case ukEnglish = "en-GB"
    
    var displayName: String {
        switch self {
        case .usEnglish: return "US English"
        case .ukEnglish: return "UK English"
        }
    }
    
    var flag: String {
        switch self {
        case .usEnglish: return "🇺🇸"
        case .ukEnglish: return "🇬🇧"
        }
    }
}

enum PlaybackMode: String, CaseIterable {
    case full = "full"
    case sentence = "sentence"
    case phrase = "phrase"
    
    var displayName: String {
        switch self {
        case .full: return "Full Text"
        case .sentence: return "Sentence by Sentence"
        case .phrase: return "Selected Phrase"
        }
    }
}
```

#### Week 6: MVP版TTS制御UI実装
**目標**: 簡素化音声制御インターフェース
```swift
// MVP版作業項目
✅ BasicTTSControlView UI実装
✅ 基本制御ボタン(再生/停止/一時停止)
✅ 3段階速度選択ボタン（スライダー削除）
// ~~音声言語選択~~ → v2.0（US English固定）
// ~~再生モード切り替え~~ → v2.0（全文再生のみ）
```

**UI実装詳細**:
```swift
// TTSControlPanel.swift - 音声制御パネル
struct TTSControlPanel: View {
    @StateObject private var ttsManager = TTSManager()
    @State private var selectedText: String = ""
    
    var body: some View {
        VStack(spacing: 16) {
            // メイン制御ボタン
            HStack(spacing: 12) {
                Button(action: { ttsManager.playFullText(selectedText) }) {
                    Label("Play All", systemImage: "play.fill")
                        .foregroundColor(.white)
                        .padding(.horizontal, 16)
                        .padding(.vertical, 8)
                        .background(Color.blue)
                        .cornerRadius(8)
                }
                .disabled(ttsManager.isPlaying)
                
                Button(action: ttsManager.pause) {
                    Image(systemName: "pause.fill")
                        .foregroundColor(.orange)
                        .font(.title2)
                }
                .disabled(!ttsManager.isPlaying)
                
                Button(action: ttsManager.resume) {
                    Image(systemName: "play.fill")
                        .foregroundColor(.green)
                        .font(.title2)
                }
                .disabled(!ttsManager.isPaused)
                
                Button(action: ttsManager.stop) {
                    Image(systemName: "stop.fill")
                        .foregroundColor(.red)
                        .font(.title2)
                }
                .disabled(!ttsManager.isPlaying && !ttsManager.isPaused)
            }
            
            // 設定コントロール
            VStack(spacing: 12) {
                // 再生速度
                HStack {
                    Text("Speed:")
                        .font(.caption)
                        .foregroundColor(.secondary)
                    
                    Slider(
                        value: Binding(
                            get: { ttsManager.playbackSpeed },
                            set: { ttsManager.setSpeed($0) }
                        ),
                        in: 0.5...2.0,
                        step: 0.25
                    )
                    .frame(width: 120)
                    
                    Text("\(String(format: "%.1fx", ttsManager.playbackSpeed))")
                        .font(.caption)
                        .foregroundColor(.secondary)
                        .frame(width: 35)
                }
                
                // 音声言語選択
                HStack {
                    Text("Voice:")
                        .font(.caption)
                        .foregroundColor(.secondary)
                    
                    Picker("Voice Language", selection: Binding(
                        get: { ttsManager.voiceLanguage },
                        set: { ttsManager.setVoiceLanguage($0) }
                    )) {
                        ForEach(VoiceLanguage.allCases, id: \.self) { voice in
                            Text("\(voice.flag) \(voice.displayName)")
                                .tag(voice)
                        }
                    }
                    .pickerStyle(.menu)
                    .frame(width: 150)
                }
                
                // 再生モード
                HStack {
                    Text("Mode:")
                        .font(.caption)
                        .foregroundColor(.secondary)
                    
                    Picker("Playback Mode", selection: $ttsManager.playbackMode) {
                        ForEach(PlaybackMode.allCases, id: \.self) { mode in
                            Text(mode.displayName).tag(mode)
                        }
                    }
                    .pickerStyle(.segmented)
                    .frame(width: 200)
                }
            }
            
            // 再生状態表示
            if ttsManager.isPlaying || ttsManager.isPaused {
                HStack {
                    Text(ttsManager.isPlaying ? "Playing..." : "Paused")
                        .font(.caption)
                        .foregroundColor(.blue)
                    
                    Spacer()
                    
                    if ttsManager.isPlaying {
                        ProgressView()
                            .scaleEffect(0.7)
                    }
                }
            }
        }
        .padding(16)
        .background(Color(NSColor.controlBackgroundColor))
        .cornerRadius(12)
    }
}
```

### Week 7-8: MVP版基本エラーカウンター

#### Week 7: MVP版BasicErrorCounter実装
**目標**: 基本エラーカウント機能
```swift
// MVP版作業項目
✅ BasicErrorCounter構造実装
✅ 簡単な文字比較アルゴリズム
✅ 基本間違いカウント機能
✅ エラー率計算
// ~~8種類エャー分類~~ → v2.0
// ~~詳細スペルチェック~~ → v2.0
// ~~文法分析~~ → v2.0
```

**実装詳細**:
```swift
// ErrorAnalyzer.swift - エラー分析エンジン
struct ErrorAnalyzer {
    private let spellChecker = NSSpellChecker()
    
    func analyzeText(_ input: String, target: String) -> [ErrorResult] {
        var errors: [ErrorResult] = []
        
        // 1. 基本差分解析
        let basicErrors = findBasicDifferences(input: input, target: target)
        errors.append(contentsOf: basicErrors)
        
        // 2. スペルチェック
        let spellingErrors = analyzeSpelling(input)
        errors.append(contentsOf: spellingErrors)
        
        // 3. 句読点分析
        let punctuationErrors = analyzePunctuation(input: input, target: target)
        errors.append(contentsOf: punctuationErrors)
        
        // 4. 大文字化分析
        let capitalizationErrors = analyzeCapitalization(input: input, target: target)
        errors.append(contentsOf: capitalizationErrors)
        
        // 5. 文法分析(基本)
        let grammarErrors = analyzeBasicGrammar(input: input, target: target)
        errors.append(contentsOf: grammarErrors)
        
        // 重複除去・位置ソート
        return removeDuplicates(errors).sorted { $0.position < $1.position }
    }
    
    // MARK: - Basic Difference Analysis
    
    private func findBasicDifferences(input: String, target: String) -> [ErrorResult] {
        var errors: [ErrorResult] = []
        let inputChars = Array(input)
        let targetChars = Array(target)
        
        let maxLength = max(inputChars.count, targetChars.count)
        
        for i in 0..<maxLength {
            if i >= inputChars.count {
                // 未入力文字
                let error = ErrorResult(
                    type: .spelling,
                    position: i,
                    expected: String(targetChars[i]),
                    actual: "",
                    suggestion: "Add missing character: '\(targetChars[i])'"
                )
                errors.append(error)
            } else if i >= targetChars.count {
                // 余分な文字
                let error = ErrorResult(
                    type: .spelling,
                    position: i,
                    expected: "",
                    actual: String(inputChars[i]),
                    suggestion: "Remove extra character: '\(inputChars[i])'"
                )
                errors.append(error)
            } else if inputChars[i] != targetChars[i] {
                // 異なる文字
                let error = ErrorResult(
                    type: .spelling,
                    position: i,
                    expected: String(targetChars[i]),
                    actual: String(inputChars[i]),
                    suggestion: "Change '\(inputChars[i])' to '\(targetChars[i])'"
                )
                errors.append(error)
            }
        }
        
        return errors
    }
    
    // MARK: - Spelling Analysis
    
    private func analyzeSpelling(_ text: String) -> [ErrorResult] {
        var errors: [ErrorResult] = []
        let range = NSRange(location: 0, length: text.count)
        
        var currentRange = range
        while currentRange.length > 0 {
            let misspelledRange = spellChecker.rangeOfMisspelledWord(
                in: text,
                range: currentRange,
                startingAt: currentRange.location,
                wrap: false,
                language: "en"
            )
            
            if misspelledRange.location == NSNotFound {
                break
            }
            
            let misspelledWord = (text as NSString).substring(with: misspelledRange)
            let suggestions = spellChecker.guesses(
                forWordRange: misspelledRange,
                in: text,
                language: "en",
                inSpellDocumentWithTag: 0
            )
            
            let error = ErrorResult(
                type: .spelling,
                position: misspelledRange.location,
                expected: suggestions?.first ?? misspelledWord,
                actual: misspelledWord,
                suggestion: suggestions?.first != nil ? 
                    "Did you mean '\(suggestions!.first!)'?" : 
                    "Check spelling of '\(misspelledWord)'"
            )
            errors.append(error)
            
            currentRange = NSRange(
                location: misspelledRange.location + misspelledRange.length,
                length: range.length - (misspelledRange.location + misspelledRange.length)
            )
        }
        
        return errors
    }
    
    // MARK: - Punctuation Analysis
    
    private func analyzePunctuation(input: String, target: String) -> [ErrorResult] {
        var errors: [ErrorResult] = []
        
        let inputPunctuation = extractPunctuationWithPositions(from: input)
        let targetPunctuation = extractPunctuationWithPositions(from: target)
        
        let maxCount = max(inputPunctuation.count, targetPunctuation.count)
        
        for i in 0..<maxCount {
            if i >= inputPunctuation.count {
                // 句読点不足
                let targetPunct = targetPunctuation[i]
                let error = ErrorResult(
                    type: .punctuation,
                    position: targetPunct.position,
                    expected: String(targetPunct.character),
                    actual: "",
                    suggestion: "Add missing punctuation: '\(targetPunct.character)'"
                )
                errors.append(error)
            } else if i >= targetPunctuation.count {
                // 余分な句読点
                let inputPunct = inputPunctuation[i]
                let error = ErrorResult(
                    type: .punctuation,
                    position: inputPunct.position,
                    expected: "",
                    actual: String(inputPunct.character),
                    suggestion: "Remove extra punctuation: '\(inputPunct.character)'"
                )
                errors.append(error)
            } else if inputPunctuation[i].character != targetPunctuation[i].character {
                // 異なる句読点
                let inputPunct = inputPunctuation[i]
                let targetPunct = targetPunctuation[i]
                let error = ErrorResult(
                    type: .punctuation,
                    position: inputPunct.position,
                    expected: String(targetPunct.character),
                    actual: String(inputPunct.character),
                    suggestion: "Change '\(inputPunct.character)' to '\(targetPunct.character)'"
                )
                errors.append(error)
            }
        }
        
        return errors
    }
    
    // MARK: - Capitalization Analysis
    
    private func analyzeCapitalization(input: String, target: String) -> [ErrorResult] {
        var errors: [ErrorResult] = []
        
        let inputChars = Array(input)
        let targetChars = Array(target)
        
        for (index, targetChar) in targetChars.enumerated() {
            if index < inputChars.count {
                let inputChar = inputChars[index]
                
                if targetChar.isUppercase && inputChar.isLowercase {
                    // 大文字であるべき
                    let error = ErrorResult(
                        type: .capitalization,
                        position: index,
                        expected: String(targetChar),
                        actual: String(inputChar),
                        suggestion: "Capitalize '\(inputChar)' to '\(targetChar)'"
                    )
                    errors.append(error)
                } else if targetChar.isLowercase && inputChar.isUppercase {
                    // 小文字であるべき
                    let error = ErrorResult(
                        type: .capitalization,
                        position: index,
                        expected: String(targetChar),
                        actual: String(inputChar),
                        suggestion: "Make '\(inputChar)' lowercase: '\(targetChar)'"
                    )
                    errors.append(error)
                }
            }
        }
        
        return errors
    }
    
    // MARK: - Basic Grammar Analysis
    
    private func analyzeBasicGrammar(input: String, target: String) -> [ErrorResult] {
        var errors: [ErrorResult] = []
        
        // 冠詞分析
        errors.append(contentsOf: analyzeArticles(input: input, target: target))
        
        // 三単現分析
        errors.append(contentsOf: analyzeThirdPerson(input: input, target: target))
        
        // 複数形分析
        errors.append(contentsOf: analyzePlurals(input: input, target: target))
        
        return errors
    }
    
    private func analyzeArticles(input: String, target: String) -> [ErrorResult] {
        // 簡易的な冠詞分析実装
        // 実際には自然言語処理ライブラリを使用することが推奨
        var errors: [ErrorResult] = []
        
        let articles = ["a", "an", "the"]
        let inputWords = input.lowercased().components(separatedBy: .whitespacesAndNewlines)
        let targetWords = target.lowercased().components(separatedBy: .whitespacesAndNewlines)
        
        for (index, targetWord) in targetWords.enumerated() {
            if articles.contains(targetWord) {
                if index < inputWords.count {
                    let inputWord = inputWords[index]
                    if inputWord != targetWord {
                        let error = ErrorResult(
                            type: .article,
                            position: findWordPosition(targetWord, in: target, wordIndex: index),
                            expected: targetWord,
                            actual: inputWord,
                            suggestion: articles.contains(inputWord) ? 
                                "Use '\(targetWord)' instead of '\(inputWord)'" :
                                "Add article '\(targetWord)'"
                        )
                        errors.append(error)
                    }
                } else {
                    // 冠詞不足
                    let error = ErrorResult(
                        type: .article,
                        position: input.count,
                        expected: targetWord,
                        actual: "",
                        suggestion: "Add missing article '\(targetWord)'"
                    )
                    errors.append(error)
                }
            }
        }
        
        return errors
    }
    
    // MARK: - Helper Methods
    
    private func extractPunctuationWithPositions(from text: String) -> [(character: Character, position: Int)] {
        var punctuation: [(Character, Int)] = []
        
        for (index, char) in text.enumerated() {
            if char.isPunctuation {
                punctuation.append((char, index))
            }
        }
        
        return punctuation
    }
    
    private func findWordPosition(_ word: String, in text: String, wordIndex: Int) -> Int {
        let words = text.components(separatedBy: .whitespacesAndNewlines)
        var position = 0
        
        for i in 0..<min(wordIndex, words.count) {
            position += words[i].count + 1 // +1 for space
        }
        
        return position
    }
    
    private func removeDuplicates(_ errors: [ErrorResult]) -> [ErrorResult] {
        var seen: Set<String> = []
        return errors.filter { error in
            let key = "\(error.position)-\(error.type.rawValue)"
            return seen.insert(key).inserted
        }
    }
    
    private func analyzeThirdPerson(input: String, target: String) -> [ErrorResult] {
        // 三単現の's'分析実装
        // 簡易版実装 - 実際にはより高度な自然言語処理が必要
        return []
    }
    
    private func analyzePlurals(input: String, target: String) -> [ErrorResult] {
        // 複数形分析実装
        // 簡易版実装 - 実際にはより高度な自然言語処理が必要
        return []
    }
}
```

#### Week 8: MVP版基本差分表示
**目標**: 3色基本差分表示
```swift
// MVP版作業項目
✅ BasicDifferenceView UI実装
✅ 3色基本表示（正解・不正解・未入力）
✅ 文字単位差分表示
// ~~エラー種別別表示~~ → v2.0
// ~~設定画面~~ → v2.0
// ~~詳細エラー表示~~ → v2.0
// ~~学習提案~~ → v2.0
```


**Phase 2 MVP版完成基準**:
- [ ] 基本TTS音声動作（全文再生のみ）
- [ ] 基本エラーカウント機能動作
- [ ] 3色基本差分表示動作
- [ ] 実用的なTTS応答時間（数値は計測後改善）
- [ ] 実用的な差分表示速度（数値は計測後改善）

---

## 📅 Phase 3: 最小復習・データ出力 (Week 9-11) - MVP版簡素化

### 🎯 Phase 3 目標
**最小限の学習支援システム（簡素化版）**
- 基本間違い箱所リスト
- 簡単なデータ出力（CSV）
- 基本データ保存・読み込み
- 最小限の設定機能

### Week 9-10: MVP版基本復習システム

#### Week 9: MVP版BasicReviewSystem実装
**目標**: 最小限の間違いリスト機能
```swift
// MVP版作業項目
✅ BasicReviewList実装
✅ 間違い箱所の簡単な一覧表示
✅ 基本的な間違い情報表示
// ~~間隔反復アルゴリズム~~ → v2.0
// ~~習得度追跡~~ → v2.0
// ~~優先度計算~~ → v2.0
```

#### Week 10: MVP版基本復習UI
**目標**: 簡単な復習インターフェース
```swift
// MVP版作業項目
✅ BasicReviewListView実装
✅ 間違い箱所の基本一覧表示
// ~~フィルタリング~~ → v2.0
// ~~TTS統合~~ → v2.0
// ~~習得度可視化~~ → v2.0
```

### Week 11: MVP版基本データ出力

#### Week 11: BasicDataExport実装
**目標**: 最小限のデータ出力機能
```swift
// MVP版作業項目
✅ BasicDataExport実装
✅ 簡単なCSV出力機能
✅ 練習結果の基本保存
✅ データの基本読み込み
// ~~JSON出力~~ → v2.0
// ~~バックアップ・復元~~ → v2.0
// ~~複雑なデータ構造~~ → v2.0
// ~~統計ダッシュボード~~ → v2.0
```

**Phase 3 MVP版完成基準**:
- [ ] 基本間違いリスト表示動作
- [ ] 簡単なCSV出力機能動作
- [ ] 基本データ保存・読み込み動作
// ~~複雑な復習システム~~ → v2.0
// ~~統計分析~~ → v2.0
// ~~バックアップ~~ → v2.0

---

## 📅 Phase 4: MVP版最終調整・リリース (Week 12)

### Week 12: MVP版総合テスト・リリース
```swift
// MVP版作業項目
✅ 基本機能統合テスト
✅ 基本パフォーマンス検証
✅ 簡単なユーザビリティテスト
✅ 基本セキュリティチェック
✅ MVP版リリースビルド作成
✅ 基本ユーザーガイド作成
// ~~詳細なアクセシビリティ検証~~ → v2.0
// ~~App Store準備~~ → v2.0
// ~~マーケティング素材~~ → v2.0
```

---

## 🔧 開発環境・ツール設定

### 開発環境要件
```yaml
Xcode: 15.0+
macOS: 14.0+ (開発環境)
Target: macOS 14.0+ (配布)
Swift: 5.9+
SwiftUI: 5.0+
```

### プロジェクト構成
```
IELTSPractice/
├── IELTSPractice/
│   ├── App/
│   │   ├── IELTSPracticeApp.swift
│   │   └── ContentView.swift
│   ├── Models/
│   │   ├── IELTSTask.swift
│   │   ├── TypingResult.swift
│   │   └── ReviewItem.swift
│   ├── Views/
│   │   ├── Practice/
│   │   │   ├── TypingPracticeView.swift
│   │   │   ├── TargetTextPanel.swift
│   │   │   └── InputArea.swift
│   │   ├── TTS/
│   │   │   └── TTSControlPanel.swift
│   │   ├── Analysis/
│   │   │   └── ErrorAnalysisPanel.swift
│   │   └── Review/
│   │       └── ReviewListView.swift
│   ├── ViewModels/
│   │   ├── TypingTestManager.swift
│   │   ├── TTSManager.swift
│   │   └── ReviewManager.swift
│   ├── Services/
│   │   ├── ErrorAnalyzer.swift
│   │   ├── DifferenceCalculator.swift
│   │   └── ExportManager.swift
│   ├── Repositories/
│   │   ├── IELTSTaskRepository.swift
│   │   └── TypingResultRepository.swift
│   └── Utils/
│       ├── Extensions/
│       ├── Helpers/
│       └── Constants/
├── IELTSPracticeTests/
└── IELTSPracticeUITests/
```

### テスト戦略
```yaml
単体テスト:
  - カバレッジ: 85%以上
  - Manager/Service層重点
  - Repository層完全テスト

統合テスト:
  - UI Flow テスト
  - データフロー検証
  - パフォーマンステスト

UIテスト:
  - 主要ユーザーフロー
  - アクセシビリティテスト
  - エラーケーステスト
```

---

## 🎯 品質保証・パフォーマンス基準

### パフォーマンス目標
| 指標 | 目標値 | 測定方法 |
|------|--------|----------|
| 入力遅延 | <50ms | リアルタイム監視 |
| TTS応答 | <500ms | 音声再生開始まで |
| 差分計算 | <100ms | 250語テキスト処理 |
| アプリ起動 | <2秒 | コールドスタート |
| メモリ使用量 | <150MB | 通常使用時 |

### 品質基準
```yaml
機能品質:
  - 機能完全性: 100%
  - 機能正確性: 95%以上
  - 機能適合性: IELTS要件100%適合

信頼性:
  - クラッシュ率: <0.01%
  - データ損失率: 0%
  - 復旧時間: <3秒

使いやすさ:
  - 学習効率: 3クリック以内で練習開始
  - エラー回復: 明確なエラーメッセージ
  - アクセシビリティ: WCAG 2.1 AA準拠
```

---

## 📊 リスク管理・緩和策

### 技術リスク
| リスク | 影響度 | 発生確率 | 緩和策 |
|--------|--------|----------|--------|
| TTS音声品質 | High | Medium | 複数音声エンジン検討 |
| リアルタイム処理性能 | High | Medium | アルゴリズム最適化 |
| エラー分析精度 | Medium | High | 段階的精度向上 |
| SwiftData互換性 | Low | Low | Core Data併用 |

### プロジェクトリスク
```yaml
スケジュール遅延:
  - 緩和: 週次マイルストーン管理
  - 対応: 機能優先度調整

品質低下:
  - 緩和: 継続的テスト実行
  - 対応: 品質ゲート強化

要件変更:
  - 緩和: IELTS要件固定化
  - 対応: 変更影響分析
```

---

## 🚀 デプロイ・リリース計画

### リリース戦略
```yaml
MVP版 (Phase 1完了):
  - 内部テスト版
  - 基本機能検証
  - ユーザビリティテスト

Beta版 (Phase 2完了):
  - 限定ユーザーテスト
  - フィードバック収集
  - 機能改善

製品版 (Phase 4完了):
  - App Store申請
  - 一般リリース
  - マーケティング開始
```

### 配布計画
```yaml
配布方法:
  - Mac App Store (メイン)
  - 直接配布 (DMG)
  - 教育機関向け (ボリューム)

価格戦略:
  - 買い切り: ¥2,980
  - 教育割引: ¥1,980
  - 将来: サブスクリプション検討
```

---

## 📝 進捗管理・報告体制

### 週次進捗レポート
```yaml
報告内容:
  - 完了作業項目
  - 次週予定作業
  - 課題・ブロッカー
  - パフォーマンス指標
  - 品質指標

レビューポイント:
  - 機能完成度
  - 品質基準達成
  - スケジュール遵守
  - リスク状況
```

### マイルストーン管理
```yaml
Phase完了基準:
  - 全機能項目完了
  - 品質基準達成
  - パフォーマンス基準達成
  - テスト完了
  - ドキュメント完成

Go/No-Go判定:
  - 機能完全性チェック
  - 品質基準評価
  - パフォーマンステスト
  - ユーザビリティ検証
```

---

---

## 🎯 MVP版開発効果まとめ

### 開発効率向上
- **総期間短縮**: 20週間 → 12週間（40%短縮）
- **総工数削減**: 920時間 → 450時間（51%削減）
- **開発リスク軽減**: 複雑な機能を後回しにしてリスクを最小化

### MVP版の利点
1. **迅速なリリース**: 3ヶ月での市場投入
2. **早期フィードバック**: ユーザーからの実際の要求把握
3. **リスク軽減**: 小さく始めて段階的に改善
4. **開発効率**: コア機能に集中した効率的な開発

### v2.0への移行戦略
MVP版のユーザーフィードバックを基に、最も要求の高い機能から順次v2.0で実装:
- **高優先**: 詳細エラー分析、文・フレーズ単位TTS
- **中優先**: 復習システム、統計ダッシュボード  
- **低優先**: 高度なパフォーマンス最適化、複雑なデータ出力

---

*このMVP版実装計画書は、IELTS Writing Practice Appの迅速な開発とリリースを実現し、コア機能に集中した実用的なIELTSタイピング練習アプリケーションの完成を目指す実装ガイドです。*