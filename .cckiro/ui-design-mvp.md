# IELTS Writing Practice App - MVP版UI設計書

## 🎨 MVP版UI設計概要

### MVP版デザインフィロソフィー
**「必要最小限で使いやすいシンプルなインターフェース」**

#### MVP版核心原則
1. **シンプル第一**: 複雑な機能は削除し、基本機能のみに集中
2. **すぐに使える**: チュートリアル不要の直感的な操作
3. **基本フィードバック**: 必要最小限のリアルタイム情報表示
4. **段階的改善**: MVP版でフィードバック収集後、v2.0で機能拡張

#### MVP版削除・後回し機能
- ~~詳細なエラー分析表示~~ → v2.0
- ~~復習リスト画面~~ → v2.0
- ~~統計ダッシュボード~~ → v2.0  
- ~~設定画面~~ → v2.0
- ~~高度なTTS制御~~ → v2.0
- ~~アニメーション効果~~ → v2.0

---

## 📱 MVP版画面構成

### メイン練習画面（1画面のみ）
```
┌─────────────────────────────────────────────────────────┐
│                 IELTS Typing Practice                   │
│                     Task 1: 150 words                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────────┬─────────────────────────────────┐   │
│  │   Target Text   │        Your Input               │   │
│  │   (模範解答)      │      (あなたの入力)              │   │
│  │                 │                                 │   │
│  │   [Play TTS]    │   Here you type...              │   │
│  │                 │                                 │   │
│  │   文章がここに      │   入力がここに                   │   │
│  │   表示されます      │   表示されます                   │   │
│  │                 │                                 │   │
│  └─────────────────┴─────────────────────────────────┘   │
│                                                         │
│  ┌─────────────────────────────────────────────────────┐   │
│  │          [Start] [Pause] [Stop]                     │   │
│  │   ⏱️ 1:23   📊 45 WPM   🎯 92%   📈 85%           │   │
│  │   Time     Speed      Accuracy  Progress           │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### MVP版画面構成詳細
- **Target Text Area**: 45% (左側) - 模範解答表示
- **Input Area**: 45% (右側) - ユーザー入力エリア  
- **Control & Stats**: 10% (下部) - 基本制御と統計

---

## 🖼️ MVP版画面詳細設計

### 1. メイン練習画面 (BasicTypingPracticeView)

#### Target Text Panel（左側：簡素化）
```swift
struct BasicTargetTextPanel: View {
    let text: String
    let wordCount: Int
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            // 簡単なヘッダー
            HStack {
                Text("Target Text (\(wordCount) words)")
                    .font(.headline)
                    .foregroundColor(.primary)
                
                Spacer()
                
                // MVP版: 基本TTS再生ボタンのみ
                Button("Play") {
                    // TTS再生
                }
                .buttonStyle(.borderedProminent)
            }
            
            Divider()
            
            // MVP版: シンプルなテキスト表示のみ
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
```

#### Input Area（右側：簡素化）
```swift
struct BasicInputArea: View {
    @Binding var input: String
    let isActive: Bool
    let onInputChange: (String) -> Void
    
    var body: some View {
        VStack(alignment: .leading, spacing: 12) {
            // 簡単なヘッダー
            HStack {
                Text("Your Input")
                    .font(.headline)
                
                Spacer()
                
                // MVP版: 基本文字カウントのみ
                Text("\(input.count) characters")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
            }
            
            Divider()
            
            // MVP版: シンプルなテキストエディター
            TextEditor(text: $input)
                .font(.body)
                .lineSpacing(4)
                .disabled(!isActive)
                .background(Color(NSColor.textBackgroundColor))
                .cornerRadius(8)
                .onChange(of: input) { _, newValue in
                    onInputChange(newValue)
                }
        }
        .padding()
        .background(Color(NSColor.controlBackgroundColor))
        .cornerRadius(12)
    }
}
```

#### Statistics Panel（下部：大幅簡素化）
```swift
struct BasicStatisticsPanel: View {
    let wpm: Double
    let accuracy: Double
    let completionPercentage: Double
    let remainingTime: TimeInterval
    let isActive: Bool
    let onStart: () -> Void
    let onPause: () -> Void
    let onStop: () -> Void
    
    var body: some View {
        VStack(spacing: 12) {
            // MVP版: 基本制御ボタン
            HStack(spacing: 16) {
                Button("Start") { onStart() }
                    .disabled(isActive)
                    .buttonStyle(.borderedProminent)
                
                Button("Pause") { onPause() }
                    .disabled(!isActive)
                    .buttonStyle(.bordered)
                
                Button("Stop") { onStop() }
                    .disabled(!isActive)
                    .buttonStyle(.bordered)
            }
            
            // MVP版: 基本統計表示
            HStack(spacing: 20) {
                VStack {
                    Text("Time")
                        .font(.caption)
                        .foregroundColor(.secondary)
                    Text(String(format: "%.0f", remainingTime))
                        .font(.title3)
                        .fontWeight(.semibold)
                }
                
                VStack {
                    Text("WPM")
                        .font(.caption)
                        .foregroundColor(.secondary)
                    Text(String(format: "%.0f", wpm))
                        .font(.title3)
                        .fontWeight(.semibold)
                }
                
                VStack {
                    Text("Accuracy")
                        .font(.caption)
                        .foregroundColor(.secondary)
                    Text(String(format: "%.0f%%", accuracy))
                        .font(.title3)
                        .fontWeight(.semibold)
                }
                
                VStack {
                    Text("Progress")
                        .font(.caption)
                        .foregroundColor(.secondary)
                    Text(String(format: "%.0f%%", completionPercentage))
                        .font(.title3)
                        .fontWeight(.semibold)
                }
            }
        }
        .padding(.vertical, 12)
        .background(Color(NSColor.controlBackgroundColor))
        .cornerRadius(8)
    }
}
```

### 2. MVP版メイン画面統合
```swift
struct BasicTypingPracticeView: View {
    @StateObject private var testManager = TypingTestManager()
    @State private var currentTask: IELTSTask?
    @State private var userInput: String = ""
    
    var body: some View {
        VStack(spacing: 16) {
            // MVP版: シンプルなヘッダー
            Text("IELTS Typing Practice")
                .font(.title)
                .fontWeight(.bold)
            
            if let task = currentTask {
                Text("Task \(task.taskType.rawValue): \(task.targetBandScore) Band Score")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
            }
            
            // メインコンテンツエリア
            HStack(spacing: 16) {
                // 左側: 目標文表示
                BasicTargetTextPanel(
                    text: currentTask?.modelAnswer ?? "",
                    wordCount: currentTask?.wordCount ?? 0
                )
                .frame(minWidth: 400)
                
                // 右側: 入力エリア
                BasicInputArea(
                    input: $userInput,
                    isActive: testManager.isActive,
                    onInputChange: { input in
                        testManager.updateInput(input)
                    }
                )
                .frame(minWidth: 400)
            }
            
            // 下部: 統計・制御パネル
            BasicStatisticsPanel(
                wmp: testManager.currentWPM,
                accuracy: testManager.accuracy,
                completionPercentage: testManager.completionPercentage,
                remainingTime: testManager.remainingTime,
                isActive: testManager.isActive,
                onStart: startTest,
                onPause: testManager.pauseTest,
                onStop: endTest
            )
        }
        .padding()
        .onAppear(perform: loadTasks)
    }
    
    // MVP版: 基本的な制御メソッド
    private func startTest() {
        guard let task = currentTask else { return }
        testManager.startTest(with: task)
    }
    
    private func endTest() {
        guard let result = testManager.endTest() else { return }
        // MVP版: 基本的な結果保存のみ
        saveResult(result)
    }
    
    private func loadTasks() {
        // MVP版: サンプルタスクの読み込み
    }
    
    private func saveResult(_ result: TypingResult) {
        // MVP版: 基本的なデータ保存
    }
}
```

---

## 🎨 MVP版デザインシステム

### 色彩設計（簡素化）
```swift
extension Color {
    // MVP版: 基本色のみ
    static let mvpPrimary = Color.blue
    static let mvpSecondary = Color.gray
    static let mvpSuccess = Color.green
    static let mvpError = Color.red
    static let mvpWarning = Color.orange
}
```

### タイポグラフィ（簡素化）
```swift
extension Font {
    // MVP版: システムフォントのみ使用
    static let mvpTitle = Font.title
    static let mvpHeadline = Font.headline
    static let mvpBody = Font.body
    static let mvpCaption = Font.caption
}
```

### MVP版で削除・後回し要素
- ~~カスタムカラーパレット~~ → v2.0
- ~~詳細なタイポグラフィ階層~~ → v2.0
- ~~アニメーション定義~~ → v2.0
- ~~アイコンシステム~~ → v2.0（システムアイコンのみ使用）
- ~~スタイルガイド詳細~~ → v2.0

---

## 📱 MVP版レスポンシブ対応

### 最小要件（現実化）
- **最小ウィンドウサイズ**: 1024×768（実用的な最小サイズ）
- **推奨サイズ**: 1280×800以上
- **対象解像度**: 一般的なmacOS環境のみ

### MVP版で削除・後回し
- ~~複数解像度対応~~ → v2.0
- ~~詳細なレスポンシブブレークポイント~~ → v2.0
- ~~アクセシビリティ詳細対応~~ → v2.0（基本対応のみ）

---

## 🎯 MVP版UI開発効果

### 開発効率向上
- **UI実装工数**: 約60%削減（複雑なコンポーネントを削除）
- **デザイン工数**: 約70%削減（基本設計のみに集中）
- **テスト工数**: 約50%削減（シンプルなUIによる）

### MVP版の利点
1. **迅速な実装**: シンプルなコンポーネントで開発速度向上
2. **安定性向上**: 複雑な機能がないため、バグの発生確率低減
3. **ユーザビリティテスト容易**: 基本機能のみでテストが集中
4. **フィードバック収集**: 本当に必要な機能を特定可能

### v2.0でのUI機能追加予定
MVP版のフィードバックを基に、以下の機能を段階的に追加：
- **高優先**: リアルタイム差分ハイライト、詳細エラー表示
- **中優先**: 復習リスト画面、統計ダッシュボード
- **低優先**: アニメーション効果、高度なカスタマイズ機能

---

*このMVP版UI設計書は、IELTS Writing Practice Appの迅速な開発を実現し、ユーザーが即座に使い始められるシンプルで実用的なインターフェースの実装を目指します。v2.0での機能拡張を前提とした段階的UI改善アプローチを採用しています。*