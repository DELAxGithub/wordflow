# IELTS Writing Practice App - UI/UX設計書

## 🎨 UI設計概要

### デザインフィロソフィー
**「学習に集中できるミニマルで機能的なインターフェース」**

#### 核心原則
1. **Focus-First**: 練習に集中できる環境の提供
2. **Immediate Feedback**: リアルタイムなフィードバック表示
3. **Cognitive Load Reduction**: 認知負荷を最小化したシンプル設計  
4. **IELTS-Optimized**: IELTS受験者の学習フローに最適化

#### デザイン指針
- **情報階層**: 重要度に応じた情報の視覚的優先順位
- **一貫性**: 全画面での統一されたデザインパターン
- **応答性**: 即座なフィードバックによるユーザー安心感
- **アクセシビリティ**: 全ユーザーが利用可能なインクルーシブデザイン

---

## 📱 画面構成・情報アーキテクチャ

### メイン画面構成
```
┌─────────────────────────────────────────────────────────┐
│                    App Header                           │
│  IELTS Practice  [Settings⚙️] [Progress📊] [Help❓]     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────────┬─────────────────────────────────┐   │
│  │   Target Text   │        Input Area               │   │
│  │   (模範解答)      │      (入力エリア)                │   │
│  │                 │                                 │   │
│  │   250 words     │   + Real-time typing            │   │
│  │   + Structure   │   + Auto-scroll                 │   │
│  │   + TTS Control │   + Difference highlight        │   │
│  │   + Highlight   │   + Word count display          │   │
│  │                 │                                 │   │
│  └─────────────────┴─────────────────────────────────┘   │
│                                                         │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Live Statistics Panel                  │   │
│  │  ⏱️ 1:23  📊 45WPM  🎯 92%  🏆 0.85  📈 85% Done    │   │
│  │  Timer   Speed   Accuracy  Quality   Progress      │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                         │
│  ┌─────────────────────────────────────────────────────┐   │
│  │               Error Analysis Panel                  │   │
│  │  🔴 Spelling: 3  🟠 Grammar: 2  🔵 Punct: 1       │   │
│  │  🟣 Capital: 0   🟤 Article: 1   🟢 Other: 0      │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 画面比率・レイアウト
- **Target Text Area**: 40% (左側)
- **Input Area**: 40% (右側)  
- **Statistics Panel**: 15% (下部)
- **Error Analysis**: 5% (最下部)

---

## 🖼️ 詳細画面設計

### 1. メイン練習画面 (TypingPracticeView)

#### Target Text Area (左側パネル)
```swift
struct TargetTextPanel: View {
    @State private var highlightedSentence: Int? = nil
    @State private var showStructure: Bool = true
    
    var body: some View {
        VStack(spacing: 0) {
            // Header with controls
            HStack {
                Text("Target Text (250 words)")
                    .font(.headline)
                    .foregroundColor(.primary)
                
                Spacer()
                
                // Structure toggle
                Button(action: { showStructure.toggle() }) {
                    Image(systemName: showStructure ? "list.bullet" : "text.alignleft")
                }
                .help("Toggle structure view")
            }
            .padding(.bottom, 8)
            
            Divider()
            
            // Text with highlighting
            ScrollView {
                if showStructure {
                    StructuredTextView(text: targetText, 
                                     highlightedSentence: highlightedSentence)
                } else {
                    ContinuousTextView(text: targetText,
                                     highlightedSentence: highlightedSentence)
                }
            }
            
            Divider()
            
            // TTS Controls
            TTSControlPanel()
        }
        .padding()
        .background(Color(NSColor.textBackgroundColor))
        .cornerRadius(12)
    }
}
```

#### Input Area (右側パネル)
```swift
struct InputArea: View {
    @Binding var inputText: String
    @State private var textEditor: NSTextView?
    
    var body: some View {
        VStack(spacing: 0) {
            // Header with word count
            HStack {
                Text("Your Input")
                    .font(.headline)
                
                Spacer()
                
                Text("\(inputText.wordCount)/250 words")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
            }
            .padding(.bottom, 8)
            
            Divider()
            
            // Real-time difference highlighting text editor
            DifferenceHighlightTextEditor(
                text: $inputText,
                targetText: targetText,
                fontSize: 16,
                fontFamily: .systemFont
            )
            
            Divider()
            
            // Input statistics
            HStack {
                Text("Characters: \(inputText.count)")
                Spacer()
                Text("Sentences: \(inputText.sentenceCount)")
                Spacer()  
                Text("Paragraphs: \(inputText.paragraphCount)")
            }
            .font(.caption)
            .foregroundColor(.secondary)
            .padding(.top, 8)
        }
        .padding()
        .background(Color(NSColor.textBackgroundColor))
        .cornerRadius(12)
    }
}
```

#### Live Statistics Panel
```swift
struct LiveStatisticsPanel: View {
    @ObservedObject var testManager: TypingTestManager
    
    var body: some View {
        HStack(spacing: 20) {
            StatisticCard(
                icon: "⏱️",
                title: "Timer",
                value: testManager.formattedRemainingTime,
                color: timeColor
            )
            
            StatisticCard(
                icon: "📊", 
                title: "WPM",
                value: String(format: "%.0f", testManager.currentWPM),
                color: .blue
            )
            
            StatisticCard(
                icon: "🎯",
                title: "Accuracy", 
                value: String(format: "%.0f%%", testManager.accuracy),
                color: .green
            )
            
            StatisticCard(
                icon: "🏆",
                title: "Quality",
                value: String(format: "%.2f", testManager.qualityIndex),
                color: .purple
            )
            
            StatisticCard(
                icon: "📈",
                title: "Progress",
                value: String(format: "%.0f%%", testManager.completionPercentage),
                color: .orange
            )
        }
        .padding(.vertical, 12)
        .background(Color(NSColor.controlBackgroundColor))
        .cornerRadius(8)
    }
}
```

---

### 2. TTS制御パネル

```swift
struct TTSControlPanel: View {
    @StateObject private var ttsManager = TTSManager()
    @State private var playbackSpeed: Float = 1.0
    @State private var selectedVoice: VoiceLanguage = .usEnglish
    
    var body: some View {
        VStack(spacing: 12) {
            // Main controls
            HStack(spacing: 16) {
                // Play/Pause/Stop buttons  
                HStack(spacing: 8) {
                    Button(action: ttsManager.playFullText) {
                        Image(systemName: "play.fill")
                            .font(.title2)
                    }
                    .disabled(ttsManager.isPlaying)
                    
                    Button(action: ttsManager.pauseSpeech) {
                        Image(systemName: "pause.fill")
                            .font(.title2)
                    }
                    .disabled(!ttsManager.isPlaying)
                    
                    Button(action: ttsManager.stopSpeech) {
                        Image(systemName: "stop.fill")
                            .font(.title2)
                    }
                }
                
                Divider()
                    .frame(height: 20)
                
                // Playback modes
                VStack {
                    Text("Mode")
                        .font(.caption)
                        .foregroundColor(.secondary)
                    
                    Picker("Mode", selection: $ttsManager.playbackMode) {
                        Text("Full").tag(PlaybackMode.full)
                        Text("Sentence").tag(PlaybackMode.sentence)
                        Text("Phrase").tag(PlaybackMode.phrase)
                    }
                    .pickerStyle(.segmented)
                    .frame(width: 150)
                }
                
                Divider()
                    .frame(height: 20)
                
                // Speed control
                VStack {
                    Text("Speed: \(String(format: "%.1fx", playbackSpeed))")
                        .font(.caption)
                        .foregroundColor(.secondary)
                    
                    Slider(value: $playbackSpeed, in: 0.5...2.0, step: 0.25)
                        .frame(width: 100)
                        .onChange(of: playbackSpeed) { _, newValue in
                            ttsManager.setSpeed(newValue)
                        }
                }
            }
            
            // Voice selection
            HStack {
                Text("Voice:")
                    .font(.caption)
                    .foregroundColor(.secondary)
                
                Picker("Voice", selection: $selectedVoice) {
                    Text("🇺🇸 US English").tag(VoiceLanguage.usEnglish)
                    Text("🇬🇧 UK English").tag(VoiceLanguage.ukEnglish)
                }
                .pickerStyle(.menu)
                .onChange(of: selectedVoice) { _, newVoice in
                    ttsManager.setVoiceLanguage(newVoice)
                }
                
                Spacer()
                
                // Progress indicator
                if ttsManager.isPlaying {
                    HStack(spacing: 4) {
                        Text("Playing...")
                            .font(.caption)
                            .foregroundColor(.blue)
                        
                        ProgressView()
                            .scaleEffect(0.7)
                    }
                }
            }
        }
        .padding(12)
        .background(Color(NSColor.controlBackgroundColor))
        .cornerRadius(8)
    }
}
```

---

### 3. エラー分析パネル

```swift
struct ErrorAnalysisPanel: View {
    let errorResults: [ErrorResult]
    @State private var selectedErrorType: ErrorType?
    
    private var errorCounts: [ErrorType: Int] {
        Dictionary(grouping: errorResults, by: { $0.type })
            .mapValues { $0.count }
    }
    
    var body: some View {
        VStack(spacing: 8) {
            HStack {
                Text("Error Analysis")
                    .font(.headline)
                    .foregroundColor(.primary)
                
                Spacer()
                
                Text("Total: \(errorResults.count) errors")
                    .font(.subheadline)
                    .foregroundColor(.secondary)
            }
            
            // Error category chips
            LazyVGrid(columns: Array(repeating: GridItem(.flexible()), count: 4), 
                      spacing: 8) {
                ForEach(ErrorType.allCases, id: \.self) { errorType in
                    ErrorCategoryChip(
                        errorType: errorType,
                        count: errorCounts[errorType] ?? 0,
                        isSelected: selectedErrorType == errorType
                    ) {
                        selectedErrorType = selectedErrorType == errorType ? nil : errorType
                    }
                }
            }
            
            // Error details (if category selected)
            if let selectedType = selectedErrorType,
               let errors = errorCounts[selectedType], errors > 0 {
                
                Divider()
                
                VStack(alignment: .leading, spacing: 4) {
                    Text("\(selectedType.displayName) Errors")
                        .font(.subheadline)
                        .fontWeight(.medium)
                    
                    ForEach(errorResults.filter { $0.type == selectedType }, 
                           id: \.position) { error in
                        ErrorDetailRow(error: error)
                    }
                }
            }
        }
        .padding(12)
        .background(Color(NSColor.controlBackgroundColor))
        .cornerRadius(8)
    }
}

struct ErrorCategoryChip: View {
    let errorType: ErrorType
    let count: Int
    let isSelected: Bool
    let action: () -> Void
    
    var body: some View {
        Button(action: action) {
            HStack(spacing: 4) {
                Text(errorType.icon)
                    .font(.caption)
                
                Text(errorType.shortName)
                    .font(.caption)
                    .fontWeight(.medium)
                
                Text("\(count)")
                    .font(.caption)
                    .fontWeight(.bold)
                    .padding(.horizontal, 6)
                    .padding(.vertical, 2)
                    .background(Color.primary.opacity(0.1))
                    .cornerRadius(8)
            }
            .padding(.horizontal, 8)
            .padding(.vertical, 4)
            .background(isSelected ? errorType.color.opacity(0.2) : Color.clear)
            .overlay(
                RoundedRectangle(cornerRadius: 6)
                    .stroke(errorType.color, lineWidth: isSelected ? 2 : 1)
            )
        }
        .buttonStyle(.plain)
    }
}
```

---

### 4. 差分ハイライト表示システム

```swift
struct DifferenceHighlightTextEditor: NSViewRepresentable {
    @Binding var text: String
    let targetText: String
    let fontSize: CGFloat
    let fontFamily: NSFont
    
    func makeNSView(context: Context) -> NSScrollView {
        let scrollView = NSScrollView()
        let textView = NSTextView()
        
        // Text view configuration
        textView.delegate = context.coordinator
        textView.isRichText = true
        textView.allowsUndo = true
        textView.font = fontFamily
        textView.textColor = NSColor.labelColor
        textView.backgroundColor = NSColor.textBackgroundColor
        
        // Real-time highlighting
        textView.textStorage?.delegate = context.coordinator
        
        scrollView.documentView = textView
        scrollView.hasVerticalScroller = true
        
        return scrollView
    }
    
    func updateNSView(_ nsView: NSScrollView, context: Context) {
        guard let textView = nsView.documentView as? NSTextView else { return }
        
        if textView.string != text {
            textView.string = text
            applyDifferenceHighlighting(to: textView)
        }
    }
    
    private func applyDifferenceHighlighting(to textView: NSTextView) {
        let attributedString = NSMutableAttributedString(string: text)
        let targetWords = targetText.components(separatedBy: .whitespacesAndNewlines)
        let inputWords = text.components(separatedBy: .whitespacesAndNewlines)
        
        // Apply character-by-character comparison
        for (index, character) in text.enumerated() {
            let range = NSRange(location: index, length: 1)
            
            if index < targetText.count {
                let targetChar = targetText[targetText.index(targetText.startIndex, offsetBy: index)]
                
                if character == targetChar {
                    // Correct character - green background
                    attributedString.addAttribute(.backgroundColor, 
                                                value: NSColor.systemGreen.withAlphaComponent(0.3), 
                                                range: range)
                } else {
                    // Incorrect character - red background
                    attributedString.addAttribute(.backgroundColor, 
                                                value: NSColor.systemRed.withAlphaComponent(0.3), 
                                                range: range)
                    
                    // Add underline for emphasis
                    attributedString.addAttribute(.underlineStyle, 
                                                value: NSUnderlineStyle.single.rawValue, 
                                                range: range)
                }
            } else {
                // Extra characters - pink background
                attributedString.addAttribute(.backgroundColor, 
                                            value: NSColor.systemPink.withAlphaComponent(0.3), 
                                            range: range)
            }
        }
        
        textView.textStorage?.setAttributedString(attributedString)
    }
}
```

---

### 5. 復習リスト画面

```swift
struct ReviewListView: View {
    @StateObject private var reviewManager = ReviewManager()
    @State private var selectedErrorType: ErrorType?
    @State private var showingMasteredItems = false
    
    var filteredReviewItems: [ReviewItem] {
        var items = reviewManager.reviewItems
        
        if let errorType = selectedErrorType {
            items = items.filter { $0.errorType == errorType }
        }
        
        if !showingMasteredItems {
            items = items.filter { $0.masteryLevel != .mastered }
        }
        
        return items.sorted { $0.priority > $1.priority }
    }
    
    var body: some View {
        VStack(spacing: 16) {
            // Header with filters
            HStack {
                Text("Review List")
                    .font(.title2)
                    .fontWeight(.semibold)
                
                Spacer()
                
                // Error type filter
                Picker("Filter", selection: $selectedErrorType) {
                    Text("All Errors").tag(nil as ErrorType?)
                    ForEach(ErrorType.allCases, id: \.self) { type in
                        Text(type.displayName).tag(type as ErrorType?)
                    }
                }
                .pickerStyle(.menu)
                .frame(width: 150)
                
                // Show mastered toggle
                Toggle("Show Mastered", isOn: $showingMasteredItems)
            }
            
            // Statistics bar
            HStack(spacing: 20) {
                StatCard(title: "Total Items", 
                        value: "\(reviewManager.reviewItems.count)")
                StatCard(title: "Need Practice", 
                        value: "\(reviewManager.needsPracticeCount)")
                StatCard(title: "Mastered", 
                        value: "\(reviewManager.masteredCount)")
                StatCard(title: "Success Rate", 
                        value: "\(Int(reviewManager.overallSuccessRate * 100))%")
            }
            
            Divider()
            
            // Review items list
            ScrollView {
                LazyVStack(spacing: 12) {
                    ForEach(filteredReviewItems, id: \.id) { item in
                        ReviewItemCard(item: item) {
                            reviewManager.markAsReviewed(item)
                        }
                    }
                    
                    if filteredReviewItems.isEmpty {
                        EmptyReviewListView()
                    }
                }
                .padding()
            }
        }
        .padding()
        .onAppear {
            reviewManager.loadReviewItems()
        }
    }
}

struct ReviewItemCard: View {
    let item: ReviewItem  
    let onReviewed: () -> Void
    @StateObject private var ttsManager = TTSManager()
    
    var body: some View {
        HStack(spacing: 12) {
            // Error type indicator
            VStack {
                Text(item.errorType.icon)
                    .font(.title2)
                
                Text(item.errorType.shortName)
                    .font(.caption)
                    .multilineTextAlignment(.center)
            }
            .frame(width: 60)
            .padding(8)
            .background(item.errorType.color.opacity(0.1))
            .cornerRadius(8)
            
            // Error details
            VStack(alignment: .leading, spacing: 4) {
                HStack {
                    Text("Expected:")
                        .font(.caption)
                        .foregroundColor(.secondary)
                    
                    Text(item.expectedText)
                        .font(.body)
                        .fontWeight(.medium)
                        .foregroundColor(.green)
                }
                
                HStack {
                    Text("Your input:")
                        .font(.caption)
                        .foregroundColor(.secondary)
                    
                    Text(item.actualText)
                        .font(.body)
                        .fontWeight(.medium)
                        .foregroundColor(.red)
                }
                
                if !item.suggestion.isEmpty {
                    HStack {
                        Text("Tip:")
                            .font(.caption)
                            .foregroundColor(.secondary)
                        
                        Text(item.suggestion)
                            .font(.caption)
                            .foregroundColor(.blue)
                    }
                }
            }
            
            Spacer()
            
            // Mastery level and controls
            VStack(spacing: 8) {
                MasteryLevelIndicator(level: item.masteryLevel)
                
                HStack(spacing: 4) {
                    // TTS button
                    Button(action: { 
                        ttsManager.playPhrase(item.expectedText) 
                    }) {
                        Image(systemName: "speaker.2")
                            .font(.caption)
                    }
                    .buttonStyle(.borderless)
                    .help("Play pronunciation")
                    
                    // Mark as reviewed
                    Button(action: onReviewed) {
                        Image(systemName: "checkmark.circle")
                            .font(.caption)
                    }
                    .buttonStyle(.borderless)
                    .help("Mark as reviewed")
                }
            }
        }
        .padding(12)
        .background(Color(NSColor.controlBackgroundColor))
        .cornerRadius(12)
        .overlay(
            RoundedRectangle(cornerRadius: 12)
                .stroke(item.errorType.color.opacity(0.3), lineWidth: 1)
        )
    }
}
```

---

## 🎨 ビジュアルデザインシステム

### カラーパレット

#### プライマリカラー
```swift
enum AppColors {
    // Brand colors
    static let primary = Color(#colorLiteral(red: 0.0, green: 0.48, blue: 1.0, alpha: 1.0))      // #007AFF
    static let secondary = Color(#colorLiteral(red: 0.56, green: 0.56, blue: 0.58, alpha: 1.0))  // #8E8E93
    
    // Semantic colors
    static let success = Color(#colorLiteral(red: 0.20, green: 0.78, blue: 0.35, alpha: 1.0))    // #34C759
    static let warning = Color(#colorLiteral(red: 1.0, green: 0.58, blue: 0.0, alpha: 1.0))     // #FF9500
    static let error = Color(#colorLiteral(red: 1.0, green: 0.23, blue: 0.19, alpha: 1.0))      // #FF3B30
    
    // Error type colors
    static let spelling = error                      // #FF3B30 (赤)
    static let grammar = warning                     // #FF9500 (オレンジ)
    static let punctuation = primary                 // #007AFF (青)
    static let capitalization = Color(.systemPurple) // #AF52DE (紫)
}
```

#### ダークモード対応
```swift
extension AppColors {
    static let adaptiveBackground = Color(NSColor.textBackgroundColor)
    static let adaptiveText = Color(NSColor.labelColor)
    static let adaptiveSecondaryText = Color(NSColor.secondaryLabelColor)
    static let adaptiveCardBackground = Color(NSColor.controlBackgroundColor)
}
```

### タイポグラフィ

```swift
enum AppFonts {
    // Text editor fonts (monospace for typing)
    static let editorFont = NSFont.monospacedSystemFont(ofSize: 16, weight: .regular)
    static let targetTextFont = NSFont.systemFont(ofSize: 16, weight: .regular)
    
    // UI fonts
    static let titleFont = Font.title2.weight(.semibold)
    static let headlineFont = Font.headline
    static let bodyFont = Font.body
    static let captionFont = Font.caption
    static let statisticValueFont = Font.title3.weight(.semibold)
    static let statisticLabelFont = Font.caption.weight(.medium)
}
```

### アイコンシステム

```swift
enum AppIcons {
    // Navigation
    static let settings = "gearshape"
    static let progress = "chart.bar"
    static let help = "questionmark.circle"
    
    // TTS controls
    static let play = "play.fill"
    static let pause = "pause.fill"
    static let stop = "stop.fill"
    static let speaker = "speaker.2"
    
    // Statistics
    static let timer = "timer"
    static let speed = "speedometer"  
    static let accuracy = "target"
    static let quality = "trophy"
    static let progress = "chart.line.uptrend.xyaxis"
    
    // Error types
    static let spelling = "textformat.abc"
    static let grammar = "text.badge.checkmark"
    static let punctuation = "exclamationmark.triangle"
    static let capitalization = "textformat"
}
```

---

## 📐 レイアウト・レスポンシブデザイン

### 画面サイズ対応

#### 最小サイズ: 1280x720
```swift
struct ResponsiveLayout {
    static let minWindowWidth: CGFloat = 1280
    static let minWindowHeight: CGFloat = 720
    
    // Component sizing
    static let targetTextMinWidth: CGFloat = 500
    static let inputAreaMinWidth: CGFloat = 500
    static let statisticsPanelHeight: CGFloat = 80
    static let errorPanelHeight: CGFloat = 120
}
```

#### レスポンシブ対応
```swift
struct AdaptiveLayout: View {
    let screenSize: CGSize
    
    private var useCompactLayout: Bool {
        screenSize.width < 1400
    }
    
    var body: some View {
        if useCompactLayout {
            // Vertical stack for smaller screens
            VStack(spacing: 12) {
                HStack(spacing: 12) {
                    TargetTextPanel()
                    InputArea()
                }
                
                VStack(spacing: 8) {
                    StatisticsPanel()
                    ErrorAnalysisPanel()
                }
            }
        } else {
            // Standard horizontal layout
            VStack(spacing: 12) {
                HStack(spacing: 12) {
                    TargetTextPanel()
                        .frame(minWidth: 600)
                    InputArea()
                        .frame(minWidth: 600)
                }
                
                HStack(spacing: 12) {
                    StatisticsPanel()
                    ErrorAnalysisPanel()
                }
            }
        }
    }
}
```

---

## ♿ アクセシビリティ設計

### VoiceOver対応
```swift
extension View {
    func accessibilityConfiguration() -> some View {
        self
            .accessibilityElement(children: .combine)
            .accessibilityLabel("IELTS typing practice")
            .accessibilityHint("Practice typing IELTS essay with real-time feedback")
    }
}

struct AccessibleStatisticCard: View {
    let title: String
    let value: String
    
    var body: some View {
        VStack {
            Text(value)
                .font(.title3)
                .fontWeight(.semibold)
            
            Text(title)
                .font(.caption)
        }
        .accessibilityElement(children: .combine)
        .accessibilityLabel("\(title): \(value)")
        .accessibilityHint("Current \(title.lowercased()) statistic")
    }
}
```

### キーボードナビゲーション
```swift
struct KeyboardNavigationView: View {
    @FocusState private var focusedField: FocusField?
    
    enum FocusField {
        case targetText, inputArea, ttsControls, statistics
    }
    
    var body: some View {
        VStack {
            TargetTextPanel()
                .focused($focusedField, equals: .targetText)
                .onKeyPress(.tab) { focusedField = .inputArea; return .handled }
            
            InputArea()
                .focused($focusedField, equals: .inputArea)
                .onKeyPress(.tab) { focusedField = .ttsControls; return .handled }
            
            TTSControlPanel()
                .focused($focusedField, equals: .ttsControls)
                .onKeyPress(.tab) { focusedField = .statistics; return .handled }
                
            StatisticsPanel()
                .focused($focusedField, equals: .statistics)
                .onKeyPress(.tab) { focusedField = .targetText; return .handled }
        }
    }
}
```

### 高コントラストモード
```swift
@Environment(\.accessibilityDifferentiateWithoutColor) var differentiateWithoutColor
@Environment(\.accessibilityReduceTransparency) var reduceTransparency

struct AccessibleErrorHighlight: View {
    let errorType: ErrorType
    
    var body: some View {
        if differentiateWithoutColor {
            // Use patterns instead of just colors
            HStack {
                errorType.patternIcon
                Text(errorType.displayName)
            }
            .padding(4)
            .overlay(
                RoundedRectangle(cornerRadius: 4)
                    .stroke(Color.primary, lineWidth: 2)
                    .background(errorType.pattern)
            )
        } else {
            // Standard color-based highlight
            Text(errorType.displayName)
                .padding(4)
                .background(errorType.color.opacity(0.3))
                .cornerRadius(4)
        }
    }
}
```

---

## 🎭 アニメーション・インタラクション設計

### マイクロアニメーション
```swift
struct AnimatedStatisticCard: View {
    let value: Double
    @State private var animatedValue: Double = 0
    
    var body: some View {
        Text(String(format: "%.0f", animatedValue))
            .font(.title3)
            .fontWeight(.semibold)
            .contentTransition(.numericText())
            .onAppear {
                withAnimation(.easeInOut(duration: 0.5)) {
                    animatedValue = value
                }
            }
            .onChange(of: value) { _, newValue in
                withAnimation(.easeInOut(duration: 0.3)) {
                    animatedValue = newValue
                }
            }
    }
}
```

### フィードバックアニメーション
```swift
struct TypingFeedbackView: View {
    @State private var lastCorrectInput: String = ""
    @State private var showSuccess: Bool = false
    
    var body: some View {
        ZStack {
            InputArea()
            
            // Success feedback
            if showSuccess {
                HStack {
                    Image(systemName: "checkmark.circle.fill")
                        .foregroundColor(.green)
                        .font(.title)
                    
                    Text("Great!")
                        .fontWeight(.semibold)
                        .foregroundColor(.green)
                }
                .scaleEffect(showSuccess ? 1.2 : 0.8)
                .opacity(showSuccess ? 1 : 0)
                .animation(.spring(duration: 0.6), value: showSuccess)
            }
        }
        .onChange(of: inputText) { _, newText in
            if isCorrectSequence(newText) {
                triggerSuccessFeedback()
            }
        }
    }
    
    private func triggerSuccessFeedback() {
        showSuccess = true
        
        DispatchQueue.main.asyncAfter(deadline: .now() + 1.0) {
            showSuccess = false
        }
    }
}
```

---

## 🎬 ユーザーフロー・ナビゲーション

### 初回起動フロー
```
App Launch → Welcome Screen → Tutorial → First Practice → Results → Review Setup
     ↓              ↓             ↓            ↓           ↓           ↓
   1 sec         5 sec         30 sec       2 min      10 sec      20 sec
```

### 日常利用フロー
```
App Launch → Dashboard → Practice Selection → Typing Test → Results → Review
     ↓           ↓              ↓               ↓          ↓         ↓
   1 sec       5 sec         10 sec          2 min     10 sec    Optional
```

### ナビゲーション構造
```swift
enum AppDestination: Hashable {
    case dashboard
    case practice(IELTSTask)
    case results(TypingResult)
    case reviewList
    case settings
    case statistics
    case help
}

@main
struct IELTSPracticeApp: App {
    @StateObject private var navigationManager = NavigationManager()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(navigationManager)
                .onOpenURL { url in
                    navigationManager.handle(url)
                }
        }
        .windowResizability(.contentSize)
        .windowStyle(.titleBar)
    }
}
```

---

*この UI/UX設計書は、IELTS受験者の学習フローと認知負荷を最小化し、効果的な練習環境を提供する包括的なデザインシステムを定義しています。全てのデザイン決定は、ユーザビリティテストとアクセシビリティガイドラインに基づいて最適化されています。*