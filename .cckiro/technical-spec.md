# IELTS Writing Practice App - 技術仕様書

## 🏗️ システムアーキテクチャ

### MVP版アーキテクチャ（簡素化）
```
┌─────────────────────────────────────────────────────────┐
│              IELTS Practice App (MVP)                   │
├─────────────────────────────────────────────────────────┤
│ Presentation Layer (SwiftUI)                           │
│ ├─ TypingPracticeView    (Core)                        │
│ ├─ BasicTTSControlView   (Phase 2)                     │
│ └─ BasicSettingsView     (Phase 2)                     │
├─────────────────────────────────────────────────────────┤
│ Business Logic Layer (Simplified)                      │
│ ├─ TypingTestManager     (Core - Simplified)           │
│ ├─ BasicTTSManager       (Phase 2 - Basic only)       │
│ └─ BasicErrorCounter     (Phase 2 - Count only)       │
├─────────────────────────────────────────────────────────┤
│ Repository Layer (Essential only)                      │
│ ├─ IELTSTaskRepository   (Core)                        │
│ └─ TypingResultRepository (Core - Basic save/load)     │
├─────────────────────────────────────────────────────────┤
│ Data Layer (Simplified)                               │
│ ├─ SwiftData Models      (IELTSTask, TypingResult)     │
│ └─ User Defaults         (Basic settings)              │
├─────────────────────────────────────────────────────────┤
│ Infrastructure (Minimal)                              │
│ ├─ AVFoundation (TTS)    (Basic playback only)        │
│ └─ Foundation Utils      (Essential utilities)        │
└─────────────────────────────────────────────────────────┘
```

### 削除・後回しコンポーネント
- ~~ReviewManager~~ → v2.0
- ~~StatisticsManager~~ → v2.0
- ~~ExportManager~~ → v2.0
- ~~ErrorAnalyzer~~ → v2.0（BasicErrorCounterに簡素化）
- ~~ReviewItemRepository~~ → v2.0
- ~~複雑なUI Views~~ → v2.0

### 技術スタック詳細

#### コア技術
- **プラットフォーム**: macOS 14.0+ (Sonoma)
- **開発言語**: Swift 5.9+
- **UIフレームワーク**: SwiftUI 5.0+
- **データ管理**: SwiftData / Core Data
- **並行処理**: Swift Concurrency (async/await)
- **音声処理**: AVFoundation (AVSpeechSynthesizer)

#### 外部依存関係
```swift
// Package.swift dependencies
dependencies: [
    // No external dependencies - Native frameworks only
    // Reduces security risks and improves performance
]

// Native frameworks used
import SwiftUI
import SwiftData
import Foundation
import AVFoundation
import Combine
import AppKit
```

---

## 📊 データモデル設計

### Core Data Schema

#### IELTSTask Model（MVP版簡素化）
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
    
    init(taskType: TaskType, topic: String, modelAnswer: String, targetBandScore: Double) {
        self.id = UUID()
        self.taskType = taskType
        self.topic = topic
        self.modelAnswer = modelAnswer
        self.targetBandScore = targetBandScore
        self.wordCount = modelAnswer.count // 簡素化: 単純な文字数
        self.createdDate = Date()
    }
}

enum TaskType: String, CaseIterable, Codable {
    case task1 = "task1"  // 150 words target
    case task2 = "task2"  // 250 words target
    
    var targetWordCount: Int {
        switch self {
        case .task1: return 150
        case .task2: return 250
        }
    }
}

// MVP版で削除・後回し
// - ModuleType（Academic/General Training）→ v2.0
// - keyVocabulary, keyPhrases → v2.0
// - EssayStructure → v2.0
// - difficultyLevel → v2.0
// - prompt（問題文）→ v2.0（模範解答のみに集中）
```

#### TypingResult Model（MVP版簡素化）
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
    var basicErrorCount: Int // 簡素化: 基本エラーカウントのみ
    
    @Relationship(deleteRule: .nullify, inverse: \IELTSTask.typingResults)
    var task: IELTSTask?
    
    init(task: IELTSTask, userInput: String, testDuration: TimeInterval) {
        self.id = UUID()
        self.sessionDate = Date()
        self.task = task
        self.userInput = userInput
        self.testDuration = testDuration
        self.targetText = task.modelAnswer
        // 簡素化された計算
        self.calculateBasicMetrics()
    }
    
    private mutating func calculateBasicMetrics() {
        // 簡素化されたWPM計算（総文字数ベース）
        let totalChars = Double(userInput.count)
        let minutes = testDuration / 60.0
        self.finalWPM = minutes > 0 ? (totalChars / 5.0) / minutes : 0
        
        // 基本正確性計算
        let correctChars = countCorrectCharacters()
        self.accuracy = totalChars > 0 ? (correctChars / totalChars) * 100 : 100
        
        // 完了率
        let targetLength = Double(targetText.count)
        self.completionPercentage = targetLength > 0 ? (totalChars / targetLength) * 100 : 0
        
        // 基本エラーカウント
        self.basicErrorCount = Int(totalChars - correctChars)
    }
    
    private func countCorrectCharacters() -> Double {
        // 簡単な文字比較
        let target = Array(targetText)
        let input = Array(userInput)
        var correct = 0.0
        
        for (index, char) in input.enumerated() {
            if index < target.count && char == target[index] {
                correct += 1
            }
        }
        
        return correct
    }
}

// MVP版で削除・後回し
// - qualityIndex → v2.0
// - ErrorResult関係 → v2.0
// - averageWPM, peakWPM → v2.0
// - timeToFirstError → v2.0
// - 詳細なパフォーマンスメトリクス → v2.0
// MVP版で削除・後回しモデル
// ~~ErrorResult~~ → v2.0で実装
// ~~ReviewItem~~ → v2.0で実装  
// ~~ErrorType, ErrorSeverity~~ → v2.0で実装
// ~~MasteryLevel, ReviewPriority~~ → v2.0で実装

// 注記: MVP版はTypingResultのbasicErrorCountのみでエラー管理

---

## ⚙️ Core Manager Classes

### TypingTestManager（MVP版簡素化）
```swift
@MainActor
@Observable
final class TypingTestManager {
    // Test state（簡素化）
    private(set) var isActive: Bool = false
    private(set) var isPaused: Bool = false
    private(set) var remainingTime: TimeInterval = 120 // 2分
    private(set) var elapsedTime: TimeInterval = 0
    
    // Current metrics（MVP版: 基本メトリクスのみ）
    private(set) var currentWPM: Double = 0
    private(set) var accuracy: Double = 100
    private(set) var completionPercentage: Double = 0
    
    // Test data
    private var currentTask: IELTSTask?
    private var targetText: String = ""
    private var userInput: String = ""
    private var startTime: Date?
    private var timer: Timer?
    
    // Configuration（MVP版緩和）
    let timeLimit: TimeInterval = 120 // 2分
    let updateInterval: TimeInterval = 0.2 // 200ms更新（100ms→200msに緩和）
    
    // MVP版で削除・後回し
    // ~~instantWPM~~ → v2.0
    // ~~averageWPM~~ → v2.0  
    // ~~peakWPM~~ → v2.0
    // ~~qualityIndex~~ → v2.0
    // ~~wpmHistory~~ → v2.0
    // ~~InputDebouncer~~ → v2.0
    
    func startTest(with task: IELTSTask) {
        self.currentTask = task
        self.targetText = task.modelAnswer
        self.userInput = ""
        self.startTime = Date()
        self.remainingTime = timeLimit
        self.elapsedTime = 0
        self.isActive = true
        self.isPaused = false
        
        // Reset metrics
        resetMetrics()
        
        // Start timer
        startTimer()
    }
    
    func updateInput(_ input: String) {
        guard isActive && !isPaused else { return }
        
        self.userInput = input
        
        // MVP版: 直接計算（デバウンス削除で簡素化）
        calculateBasicMetrics()
    }
    
    func pauseTest() {
        guard isActive else { return }
        isPaused = true
        timer?.invalidate()
    }
    
    func resumeTest() {
        guard isActive && isPaused else { return }
        isPaused = false
        startTimer()
    }
    
    func endTest() -> TypingResult? {
        guard let task = currentTask else { return nil }
        
        isActive = false
        isPaused = false
        timer?.invalidate()
        
        // MVP版: 簡素化された結果作成
        let result = TypingResult(
            task: task,
            userInput: userInput,
            testDuration: elapsedTime
        )
        
        return result
    }
    
    private func startTimer() {
        timer = Timer.scheduledTimer(withTimeInterval: updateInterval, repeats: true) { [weak self] _ in
            self?.updateTimer()
        }
    }
    
    private func updateTimer() {
        guard let startTime = startTime else { return }
        
        elapsedTime = Date().timeIntervalSince(startTime)
        remainingTime = max(0, timeLimit - elapsedTime)
        
        // MVP版: 基本メトリクス計算
        calculateBasicMetrics()
        
        // End test if time is up
        if remainingTime <= 0 {
            _ = endTest()
        }
    }
    
    private func calculateBasicMetrics() {
        guard elapsedTime > 0 else { return }
        
        let minutes = elapsedTime / 60.0
        let totalChars = Double(userInput.count)
        let correctChars = Double(calculateCorrectCharacters())
        
        // MVP版: 基本WPM計算（総文字数ベース）
        currentWPM = minutes > 0 ? (totalChars / 5.0) / minutes : 0
        
        // MVP版: 基本正確性計算
        accuracy = totalChars > 0 ? (correctChars / totalChars) * 100 : 100
        
        // MVP版: 基本完了率
        let targetLength = Double(targetText.count)
        completionPercentage = targetLength > 0 ? min(100, (totalChars / targetLength) * 100) : 0
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
    
    private func resetMetrics() {
        currentWPM = 0
        accuracy = 100
        completionPercentage = 0
    }
}
```

### BasicTTSManager（MVP版大幅簡素化）
```swift
@MainActor
@Observable
final class BasicTTSManager: NSObject {
    private let synthesizer = AVSpeechSynthesizer()
    
    // MVP版: 基本状態のみ
    private(set) var isPlaying: Bool = false
    private(set) var isPaused: Bool = false
    
    // MVP版: 簡素化設定
    var playbackSpeed: TTSSpeed = .normal
    
    // Current playback
    private var currentUtterance: AVSpeechUtterance?
    
    // MVP版削除・後回し
    // ~~currentPosition, totalDuration~~ → v2.0
    // ~~VoiceLanguage選択~~ → v2.0（US Englishに固定）
    // ~~PlaybackMode~~ → v2.0（全文再生のみ）
    // ~~文単位・フレーズ再生~~ → v2.0
    // ~~sentences, currentSentenceIndex~~ → v2.0
    
    override init() {
        super.init()
        synthesizer.delegate = self
        updateVoice()
    }
    
    // MVP版: 全文再生のみ
    func playFullText(_ text: String) {
        stop()
        let utterance = createUtterance(from: text)
        speak(utterance)
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
    
    func setSpeed(_ speed: TTSSpeed) {
        playbackSpeed = speed
    }
    
    private func createUtterance(from text: String) -> AVSpeechUtterance {
        let utterance = AVSpeechUtterance(string: text)
        utterance.rate = AVSpeechUtteranceDefaultSpeechRate * playbackSpeed.multiplier
        utterance.voice = AVSpeechSynthesisVoice(language: "en-US") // MVP版: US English固定
        utterance.volume = 1.0
        return utterance
    }
    
    private func speak(_ utterance: AVSpeechUtterance) {
        currentUtterance = utterance
        synthesizer.speak(utterance)
        isPlaying = true
        isPaused = false
    }
}

// MARK: - AVSpeechSynthesizerDelegate
extension BasicTTSManager: AVSpeechSynthesizerDelegate {
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
        }
    }
    
    nonisolated func speechSynthesizer(_ synthesizer: AVSpeechSynthesizer, didPause utterance: AVSpeechUtterance) {
        Task { @MainActor in
            isPaused = true
            isPlaying = false
        }
    }
    
    nonisolated func speechSynthesizer(_ synthesizer: AVSpeechSynthesizer, didContinue utterance: AVSpeechUtterance) {
        Task { @MainActor in
            isPaused = false
            isPlaying = true
        }
    }
}

// MVP版: 3段階速度のみ
enum TTSSpeed: String, CaseIterable {
    case slow = "0.75x"
    case normal = "1.0x"
    case fast = "1.25x"
    
    var multiplier: Float {
        switch self {
        case .slow: return 0.75
        case .normal: return 1.0
        case .fast: return 1.25
        }
    }
    
    var displayName: String {
        return self.rawValue
    }
}

// MVP版削除・後回し
// ~~VoiceLanguage~~ → v2.0（US English固定）
// ~~PlaybackMode~~ → v2.0（全文再生のみ）
```

### BasicErrorCounter（MVP版大幅簡素化）
```swift
struct BasicErrorCounter {
    
    // MVP版: 基本間違いカウントのみ
    func countBasicErrors(input: String, target: String) -> BasicErrorInfo {
        let inputChars = Array(input)
        let targetChars = Array(target)
        var errorPositions: [Int] = []
        var totalErrors = 0
        
        // 簡単な文字比較
        let maxLength = max(inputChars.count, targetChars.count)
        
        for i in 0..<maxLength {
            if i >= inputChars.count || i >= targetChars.count {
                // 文字数の不一致
                errorPositions.append(i)
                totalErrors += 1
            } else if inputChars[i] != targetChars[i] {
                // 文字の不一致
                errorPositions.append(i)
                totalErrors += 1
            }
        }
        
        let errorRate = input.count > 0 ? Double(totalErrors) / Double(input.count) : 0
        
        return BasicErrorInfo(
            totalErrors: totalErrors,
            errorRate: errorRate,
            errorPositions: errorPositions
        )
    }
}

// MVP版で使用する基本エラー情報
struct BasicErrorInfo {
    let totalErrors: Int
    let errorRate: Double
    let errorPositions: [Int]
}

// MVP版削除・後回し機能
// ~~詳細エラー分析~~ → v2.0
// ~~8種類エラー分類~~ → v2.0
// ~~スペルチェック~~ → v2.0
// ~~文法チェック~~ → v2.0
// ~~句読点チェック~~ → v2.0
    
}
```

---

## 🛡️ セキュリティ・プライバシー実装（MVP版簡素化）

### App Sandbox設定
```xml
<!-- App.entitlements -->
<key>com.apple.security.app-sandbox</key>
<true/>
<key>com.apple.security.files.user-selected.read-write</key>
<true/>
<key>com.apple.security.network.client</key>
<false/>
```

### データ保護
- **ローカル処理**: 全データのローカル保存・処理
- **SwiftData暗号化**: 学習データの基本保護
- **プライバシー**: 個人情報の外部送信なし

### MVP版で削除・後回し
- ~~詳細データ暗号化~~ → v2.0
- ~~高度なセキュリティ機能~~ → v2.0

---

## 🚀 パフォーマンス最適化（MVP版現実化）

### 基本最適化方針

#### MVP版目標
- **入力遅延**: 実用的な応答性（計測後改善）
- **TTS応答**: 実用的な応答時間（計測後改善）
- **メモリ使用**: 適切なメモリ使用
- **起動時間**: 合理的な起動時間

### 計測システム実装
```swift
// MVP版: 基本的なパフォーマンス計測
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
```

### MVP版で削除・後回し
- ~~高度なキャッシュシステム~~ → v2.0
- ~~複雑なメモリ管理~~ → v2.0
- ~~デバウンス処理~~ → v2.0
- ~~バックグラウンド処理最適化~~ → v2.0

### 方針
**計測システム実装 > 数値達成**: まず計測の仕組みを作り、実際の数値を把握してから改善目標を設定

---

## 📊 監視・ログ・テレメトリ（MVP版簡素化）

### 基本ログシステム
```swift
import os.log

// MVP版: 基本的なログ機能のみ
class BasicAppLogger {
    static let shared = BasicAppLogger()
    
    private let subsystem = Bundle.main.bundleIdentifier!
    
    lazy var general = Logger(subsystem: subsystem, category: "general")
    lazy var error = Logger(subsystem: subsystem, category: "error")
    
    private init() {}
    
    func logInfo(_ message: String) {
        general.info("\(message)")
    }
    
    func logError(_ error: Error, context: String = "") {
        self.error.error("Error: \(error.localizedDescription) [\(context)]")
    }
}
```

### MVP版で削除・後回し
- ~~詳細パフォーマンス監視~~ → v2.0
- ~~複雑なテレメトリシステム~~ → v2.0
- ~~メトリクス収集・分析~~ → v2.0

### 基本品質保証
- **基本ログ**: エラーと基本情報のログ
- **クラッシュ対応**: 基本的なエラーハンドリング
- **データ保存**: 練習結果の確実な保存

---

## 🎯 MVP版開発効果まとめ

### アーキテクチャ簡素化効果
- **元の複雑度**: 15+のマネージャークラス、8種類エラー分析
- **MVP版複雑度**: 3つのコアマネージャー、基本機能のみ
- **簡素化率**: 約70%の機能削減

### 技術的利点
1. **迅速な開発**: 複雑な機能を後回しにして、コア機能に集中
2. **安定した動作**: シンプルな設計により、バグの発生確率を低減
3. **段階的改善**: MVP版でフィードバック収集後、v2.0で高度機能追加
4. **保守性向上**: コードの複雑度低下により、メンテナンスが容易

### v2.0への移行計画
MVP版のユーザーフィードバックを基に、最も要求の高い機能から順次v2.0で実装:
- **高優先**: 詳細エラー分析、文・フレーズ単位TTS
- **中優先**: 復習システム、統計ダッシュボード
- **低優先**: 高度なパフォーマンス最適化、複雑なデータ出力

---

*このMVP版技術仕様書は、IELTS Writing Practice Appの迅速な開発を実現するため、コア機能に集中した実用的な技術実装を定義します。v2.0での機能拡張を前提とした段階的開発アプローチを採用しています。*