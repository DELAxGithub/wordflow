# 📚 Wordflow Project Documentation Index

**Comprehensive Documentation Portal for IELTS Writing Practice App**

---

## 🎯 Project Overview

**Project Name**: Wordflow - IELTS Writing Practice App  
**Platform**: macOS (SwiftUI + SwiftData)  
**Architecture**: MVVM + Repository Pattern  
**Documentation Generated**: 2025-08-10  
**Current Version**: v1.1 - Enhanced Typing Metrics  

### Quick Stats
- **Total Files**: 16 Swift files + 4 Markdown documents
- **Lines of Code**: ~3,100 lines (+600 lines today)
- **Documentation Coverage**: Comprehensive
- **API Endpoints**: 30+ public interfaces
- **Data Models**: 4 primary models + enhanced metrics
- **New Features**: 30s timer mode, Net WPM, Personal Best tracking

### 🚀 **Latest Enhancements (v1.1)**
**Released**: 2025-08-10

#### **Enhanced Timer System**
- **Dual Timer Modes**: 30-second and 2-minute practice sessions
- **Segmented Control**: Easy mode switching with visual feedback
- **Time-up Popup**: Professional results modal with retry options

#### **Advanced Metrics System**
- **Net WPM**: Accurate typing speed (correct characters only)
- **Gross WPM**: Total typing speed including corrections
- **Word Accuracy**: Word-level precision alongside character accuracy
- **Consistency Tracking**: WPM variation analysis for stability metrics

#### **Personal Best System**
- **Mode-specific Records**: Separate best scores for 30s and 2-minute modes
- **Real-time Comparisons**: Live display of personal best achievements
- **Achievement Highlights**: 🏆 NEW BEST! celebrations for motivation
- **Persistent Storage**: UserDefaults-based record keeping

#### **Intelligent UI**
- **Color-coded Performance**: Green/Blue/Orange/Red based on skill levels
- **Enhanced Statistics**: Subtitle information and highlighting
- **Visual Feedback**: Dynamic timer colors and consistency indicators

---

## 📖 Documentation Hierarchy

### 🎯 **Level 1: Project Documentation**
| Document | Purpose | Lines | Status |
|----------|---------|-------|--------|
| [**spec.md**](./spec.md) | Complete technical specification | 435 | ✅ Complete |
| [**INDEX.md**](./INDEX.md) | Project structure overview | 184 | ✅ Complete |
| [**README_User_Manual.md**](./Wordflow/README_User_Manual.md) | User guide and manual | 471 | ✅ Complete |
| [**IELTS_Requirements_Gap_Analysis.md**](./IELTS_Requirements_Gap_Analysis.md) | Requirements analysis | - | ✅ Complete |
| [**要件比較表.md**](./要件比較表.md) | Requirements comparison (Japanese) | - | ✅ Complete |

### 🏗️ **Level 2: Architecture Documentation**
| Component | File | Lines | Purpose |
|-----------|------|-------|---------|
| **App Entry** | WordflowApp.swift | 75 | Application lifecycle & DI setup |
| **Main View** | ContentView.swift | 21 | Root view container |
| **Primary UI** | BasicTypingPracticeView.swift | 432 | Main typing practice interface |
| **Quality System** | DelaxQualityManager.swift | 301 | DELAX quality management |

### 🔧 **Level 3: Core Systems**
| System | Files | Purpose | Status |
|---------|-------|---------|---------|
| **Data Models** | IELTSTask.swift, TypingResult.swift | SwiftData models | ✅ Implemented |
| **Business Logic** | TypingTestManager.swift, BasicTTSManager.swift | Core managers | ✅ MVP Complete |
| **Data Access** | IELTSTaskRepository.swift, TypingResultRepository.swift | Repository pattern | ✅ Implemented |
| **Utilities** | BasicErrorCounter.swift, ExportManager.swift | Supporting services | ✅ Basic Version |

---

## 🗂️ **Project Structure Index**

```
wordflow/
├── 📄 Documentation Layer
│   ├── spec.md                    # Master specification
│   ├── INDEX.md                   # This file - comprehensive project overview
│   ├── IELTS_Requirements_Gap_Analysis.md
│   └── 要件比較表.md
│
├── 🏗️ Xcode Project
│   └── Wordflow/
│       ├── 📱 Application
│       │   ├── WordflowApp.swift         # @main App
│       │   ├── ContentView.swift         # Root view
│       │   └── Wordflow.entitlements     # App permissions
│       │
│       ├── 🎯 Core Features
│       │   ├── BasicTypingPracticeView.swift  # Main UI (590 lines)
│       │   ├── TypingTestManager.swift        # Enhanced test logic (370 lines)
│       │   ├── TestCompletionView.swift       # Time-up results modal (145 lines)
│       │   └── BasicTTSManager.swift          # Text-to-Speech
│       │
│       ├── 📊 Data Layer
│       │   ├── IELTSTask.swift               # Task model
│       │   ├── TypingResult.swift            # Result model
│       │   ├── IELTSTaskRepository.swift     # Task data access
│       │   └── TypingResultRepository.swift  # Result data access
│       │
│       ├── 🛠️ Utilities
│       │   ├── BasicErrorCounter.swift       # Error detection
│       │   ├── ExportManager.swift           # Data export
│       │   └── DelaxQualityManager.swift     # Quality system
│       │
│       └── 🎨 Resources
│           └── Assets.xcassets/              # App assets
```

---

## 🔌 **API Documentation**

### 📱 **Application Layer**

#### WordflowApp
```swift
@main struct WordflowApp: App
```
**Purpose**: Application entry point with SwiftData configuration  
**Key Features**:
- DELAX quality management integration
- SwiftData model container setup
- macOS menu command configuration

#### ContentView  
```swift
struct ContentView: View
```
**Purpose**: Root view container  
**Integration**: Hosts BasicTypingPracticeView

---

### 🎯 **Core Features API**

#### BasicTypingPracticeView
```swift
struct BasicTypingPracticeView: View
```
**Purpose**: Main typing practice interface (432 lines)  
**Key Components**:
- **3-Panel Layout**: Target text | Input area | Statistics
- **TTS Integration**: Play/pause/speed controls
- **Real-time Metrics**: WPM, accuracy, progress tracking
- **Error Highlighting**: Character-by-character comparison

**State Management**:
```swift
@State private var testManager = TypingTestManager()
@State private var ttsManager = BasicTTSManager()  
@State private var selectedTask: IELTSTask?
@State private var userInput = ""
```

#### TypingTestManager
```swift
@MainActor @Observable final class TypingTestManager
```
**Purpose**: Enhanced typing test logic and comprehensive metrics calculation

**Public API**:
```swift
func startTest(with task: IELTSTask)
func updateInput(_ input: String)
func pauseTest() / resumeTest()
func endTest() -> TypingResult?
func setTimerMode(_ mode: TimerMode)
func getPersonalBest(for mode: TimerMode) -> PersonalBest?
```

**Enhanced Metrics**:
- **Net WPM**: Accurate speed based on correct characters only
- **Gross WPM**: Total typing speed including corrections
- **Character Accuracy**: Character-level precision percentage
- **Word Accuracy**: Word-level precision percentage
- **Consistency**: WPM variation tracking for stability analysis
- **Personal Best**: Best scores per timer mode with persistence

**New Enumerations**:
```swift
enum TimerMode: CaseIterable {
    case thirtySeconds  // 30-second practice
    case twoMinutes     // 2-minute practice
}

struct PersonalBest: Codable {
    let netWPM: Double
    let accuracy: Double
    let date: Date
}
```

#### BasicTTSManager
```swift
@MainActor @Observable final class BasicTTSManager: NSObject
```
**Purpose**: Text-to-Speech functionality

**Public API**:
```swift
func playFullText(_ text: String)
func pause() / resume() / stop()
func setSpeed(_ speed: TTSSpeed)
```

**Features**:
- AVFoundation integration
- Speed control (0.75x, 1.0x, 1.25x)
- US English voice fixed
- State tracking (playing, paused)

#### TestCompletionView
```swift
struct TestCompletionView: View
```
**Purpose**: Time-up results modal with professional UI and user actions

**Properties**:
```swift
let result: TypingResult
let timerMode: TimerMode
let onRetry: () -> Void
let onNewTask: () -> Void
let onClose: () -> Void
```

**Features**:
- **Results Display**: WPM, accuracy, completion percentage with visual metrics
- **Achievement Recognition**: 🏆 indicator for new personal bests
- **Action Buttons**: Try Again, New Task, Close with prominent styling
- **Professional Design**: Glassmorphism background with shadow effects

**Components**:
```swift
struct ResultMetricView: View  // Individual metric display
struct EnhancedStatisticView: View  // Enhanced stats with highlighting
```

---

### 📊 **Data Models API**

#### IELTSTask
```swift
@Model final class IELTSTask
```
**Purpose**: IELTS practice task representation

**Properties**:
```swift
@Attribute(.unique) var id: UUID
var taskType: TaskType          // task1 (150 words) | task2 (250 words)
var topic: String
var modelAnswer: String
var targetBandScore: Double
var wordCount: Int
```

**Methods**:
```swift
func markAsUsed()               // Update last used date
var targetWordCount: Int        // Get target based on task type
var estimatedReadingTime: Int   // Calculate reading time
```

#### TypingResult
```swift
@Model final class TypingResult
```
**Purpose**: Practice session results storage

**Properties**:
```swift
@Attribute(.unique) var id: UUID
var sessionDate: Date
var testDuration: TimeInterval
var finalWPM: Double
var accuracy: Double
var completionPercentage: Double
var basicErrorCount: Int
```

**Export**:
```swift
func toCsvRow() -> String       // CSV export format
```

---

### 🗃️ **Repository Layer API**

#### IELTSTaskRepository
```swift
@MainActor final class IELTSTaskRepository: ObservableObject
```
**Purpose**: IELTS task data management

**CRUD Operations**:
```swift
func createTask(taskType: TaskType, topic: String, modelAnswer: String, targetBandScore: Double) -> IELTSTask
func fetchAllTasks() -> [IELTSTask]
func fetchTasksByType(_ type: TaskType) -> [IELTSTask]
func fetchTasksByBandScore(_ bandScore: Double) -> [IELTSTask]
func updateTask(_ task: IELTSTask)
func deleteTask(_ task: IELTSTask)
```

#### TypingResultRepository
```swift
@MainActor final class TypingResultRepository: ObservableObject
```
**Purpose**: Typing result data management

**Data Access**:
```swift
func saveResult(_ result: TypingResult)
func fetchAllResults() -> [TypingResult]
func fetchResultsForTask(_ task: IELTSTask) -> [TypingResult]
func fetchRecentResults(limit: Int = 10) -> [TypingResult]
func exportToCSV() -> String
```

---

### 🛠️ **Utilities API**

#### BasicErrorCounter
```swift
struct BasicErrorCounter
```
**Purpose**: Character-level error detection

**Core Method**:
```swift
func countBasicErrors(input: String, target: String) -> BasicErrorInfo
```

**Return Data**:
```swift
struct BasicErrorInfo {
    let totalErrors: Int
    let errorRate: Double        // Percentage
    let errorPositions: [Int]    // Character indices
}
```

#### ExportManager
```swift
@MainActor final class ExportManager: ObservableObject
```
**Purpose**: Data export functionality

**Export Methods**:
```swift
func exportResultsToCSV(results: [TypingResult]) -> String
func saveCSVToFile(csv: String, filename: String = "ielts_practice_results.csv")
func generateSummaryReport(results: [TypingResult]) -> String
```

**CSV Format**:
```
Date,Time,Task Type,Topic,WPM,Accuracy (%),Completion (%),Errors,Duration (min),Target Words
```

#### DelaxQualityManager
```swift
@Observable class DelaxQualityManager
```
**Purpose**: Quality monitoring and performance tracking

**Core API**:
```swift
func initialize()               // Setup quality system
func startMonitoring()          // Begin monitoring
func stopMonitoring()           // End monitoring
func performQualityCheck()      // Manual quality check
func reportIssue(_ issue: QualityIssue)
func resolveIssue(_ issue: QualityIssue)
```

**Quality Models**:
```swift
struct QualityIssue: Identifiable, Equatable {
    let type: QualityIssueType     // performance, memory, ui, data, security
    let severity: IssueSeverity    // low, medium, high, critical
    let message: String
    let location: String
}

struct PerformanceMetrics {
    var memoryUsage: Double
    var cpuUsage: Double
    var responseTime: Double
}
```

---

## 🔗 **Cross-Reference Navigation**

### 🎯 **Feature Cross-References**

#### Typing Practice Flow
1. **BasicTypingPracticeView** → UI Layer
2. **TypingTestManager** → Business Logic
3. **BasicErrorCounter** → Error Detection  
4. **TypingResult** → Data Model
5. **TypingResultRepository** → Data Persistence

#### TTS (Text-to-Speech) Flow
1. **BasicTypingPracticeView** → TTS Controls
2. **BasicTTSManager** → AVFoundation Integration
3. **IELTSTask** → Source Text

#### Data Export Flow
1. **BasicTypingPracticeView** → Export Trigger
2. **TypingResultRepository** → Data Retrieval
3. **ExportManager** → CSV Generation
4. **NSSavePanel** → File System

### 📊 **Data Flow Diagrams**

#### Main Data Flow
```
IELTSTask (SwiftData) 
    ↓
TypingTestManager (Business Logic)
    ↓
BasicTypingPracticeView (UI Updates)
    ↓
TypingResult (SwiftData)
    ↓
TypingResultRepository (Persistence)
```

#### Error Detection Flow
```
User Input → BasicErrorCounter → Error Positions → UI Highlighting
```

#### Quality Monitoring Flow
```
App Events → DelaxQualityManager → Performance Metrics → Quality Score
```

---

## 🧪 **Testing & Quality**

### 📋 **Test Coverage Map**

| Component | Unit Tests | Integration Tests | UI Tests | Coverage |
|-----------|------------|------------------|----------|----------|
| TypingTestManager | ❌ Needed | ❌ Needed | ❌ Needed | 0% |
| BasicErrorCounter | ❌ Needed | ❌ Needed | N/A | 0% |
| Repositories | ❌ Needed | ❌ Needed | ❌ Needed | 0% |
| Data Models | ❌ Needed | ❌ Needed | N/A | 0% |
| UI Components | ❌ Needed | ❌ Needed | ❌ Needed | 0% |

### 🔍 **Code Quality Metrics**

- **Complexity**: Medium (needs refactoring for large views)
- **Maintainability**: Good (clear separation of concerns)  
- **Testability**: Fair (needs protocol abstractions)
- **Documentation**: Excellent (comprehensive specs)
- **Error Handling**: Basic (needs improvement)

---

## 🚀 **Development Workflows**

### 📋 **Getting Started**

1. **Setup Environment**
   - macOS 14.0+ required
   - Xcode 15.0+ with SwiftUI/SwiftData support
   - Clone repository and open Wordflow.xcodeproj

2. **Run Application**
   - Select Wordflow target
   - Build and run (⌘+R)
   - Sample IELTS tasks auto-generated on first launch

3. **Development Patterns**
   - Follow MVVM architecture
   - Use Repository pattern for data access
   - Implement @Observable for state management
   - Apply @MainActor for UI-related classes

### 🔄 **Common Development Tasks**

#### Adding New IELTS Task
```swift
let task = taskRepository.createTask(
    taskType: .task2,
    topic: "Environmental Issues",
    modelAnswer: "...",
    targetBandScore: 7.5
)
```

#### Implementing New Manager
```swift
@MainActor @Observable
final class NewManager {
    // Follow established patterns
}
```

#### Adding New View Component
```swift
struct NewView: View {
    // Use environment for model context
    @Environment(\.modelContext) private var modelContext
    
    var body: some View {
        // SwiftUI implementation
    }
}
```

---

## 📈 **Future Roadmap**

### 🎯 **Next Development Phase**
1. **Performance Optimization** (Week 1-2)
   - Timer consolidation
   - Text highlighting optimization
   - Memory management improvements

2. **Enhanced Features** (Week 3-4)
   - Advanced error analysis
   - Grammar checking integration
   - Sentence-level TTS controls

3. **Quality & Testing** (Week 5-6)
   - Comprehensive test suite
   - Protocol abstractions
   - Error handling improvements

4. **Polish & Release** (Week 7-8)
   - UI/UX refinements
   - Performance benchmarking
   - Documentation updates

---

## 📞 **Support & Maintenance**

### 🔧 **Common Issues**
- **Performance**: Large text real-time highlighting
- **Memory**: Multiple timer instances
- **UI**: 432-line view needs decomposition
- **Testing**: No test coverage currently

> 🔄 **Issues Status**: INDEX記載の技術的課題は一旦保留中
> 実際の使用体験から生まれるフィードバックを優先してイシュー化します。
> GitHub Issues & Projectsを使用した使用感ベースの改善ワークフローに移行しました。

### 📋 **Maintenance Tasks**
- Regular dependency updates
- SwiftData migration support
- Performance monitoring
- Code refactoring for maintainability

### 🤝 **Contributing**
- Follow existing architecture patterns
- Maintain MVVM + Repository structure
- Add comprehensive documentation
- Include unit tests for new features

---

**📅 Last Updated**: 2025-08-10  
**🤖 Generated by**: Claude Code SuperClaude  
**📊 Documentation Version**: v1.1  

### 🏆 **Achievement Summary**
- ✅ **30-Second Timer Mode**: Fast practice sessions for quick improvement
- ✅ **Net WPM System**: Industry-standard accurate speed measurement  
- ✅ **Personal Best Tracking**: Motivation through achievement recognition
- ✅ **Professional UI**: Color-coded feedback and enhanced statistics
- ✅ **Time-up Modal**: Polished user experience with clear next steps

*This comprehensive index provides complete navigation through the enhanced Wordflow project with advanced typing metrics, dual timer modes, and professional user experience features.*