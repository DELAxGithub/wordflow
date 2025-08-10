# IELTS Writing Practice App - API仕様書

## 📋 概要

この文書では、IELTS Writing Practice Appの内部API設計、データ構造、サービス間通信プロトコルを定義します。現在は完全ローカル処理ですが、将来のクラウド同期・外部連携に対応できる設計となっています。

---

## 🏗️ アーキテクチャ概要

### サービス層構成
```
┌─────────────────────────────────────────────────────────┐
│                Presentation Layer                       │
│                   (SwiftUI Views)                       │
├─────────────────────────────────────────────────────────┤
│                 Service Layer                           │
│ ┌─────────────────┬─────────────────┬─────────────────┐ │
│ │ TypingService   │   TTSService    │ AnalysisService │ │
│ ├─────────────────┼─────────────────┼─────────────────┤ │
│ │ ReviewService   │ ExportService   │ SettingsService │ │
│ └─────────────────┴─────────────────┴─────────────────┘ │
├─────────────────────────────────────────────────────────┤
│                Repository Layer                         │
│ ┌─────────────────┬─────────────────┬─────────────────┐ │
│ │ TaskRepository  │ResultRepository │ReviewRepository │ │
│ └─────────────────┴─────────────────┴─────────────────┘ │
├─────────────────────────────────────────────────────────┤
│                 Data Access Layer                       │
│         (SwiftData / Core Data / FileSystem)            │
└─────────────────────────────────────────────────────────┘
```

---

## 🔧 Service API 仕様

### TypingService

#### インターフェース定義
```swift
protocol TypingServiceProtocol {
    func startTypingTest(_ request: StartTypingTestRequest) async throws -> TypingTestResponse
    func updateInput(_ request: UpdateInputRequest) async -> InputUpdateResponse
    func pauseTest(_ request: PauseTestRequest) async -> TestStateResponse
    func resumeTest(_ request: ResumeTestRequest) async -> TestStateResponse
    func endTest(_ request: EndTestRequest) async throws -> TypingResultResponse
    func getTestMetrics(_ request: TestMetricsRequest) async -> TestMetricsResponse
}

@MainActor
class TypingService: TypingServiceProtocol {
    private let repository: TypingResultRepositoryProtocol
    private let testManager: TypingTestManager
    private let performanceMonitor: PerformanceMonitor
    
    init(repository: TypingResultRepositoryProtocol) {
        self.repository = repository
        self.testManager = TypingTestManager()
        self.performanceMonitor = PerformanceMonitor.shared
    }
    
    func startTypingTest(_ request: StartTypingTestRequest) async throws -> TypingTestResponse {
        let startTime = CFAbsoluteTimeGetCurrent()
        
        // Validate request
        try validateStartRequest(request)
        
        // Fetch IELTS task
        guard let task = try await getTask(id: request.taskID) else {
            throw TypingServiceError.taskNotFound
        }
        
        // Initialize test session
        let sessionID = UUID()
        testManager.startTest(with: task)
        
        let responseTime = CFAbsoluteTimeGetCurrent() - startTime
        performanceMonitor.recordMetric("test_start_time", value: responseTime * 1000)
        
        return TypingTestResponse(
            sessionID: sessionID,
            task: task.toDTO(),
            timeLimit: testManager.timeLimit,
            status: .active
        )
    }
    
    func updateInput(_ request: UpdateInputRequest) async -> InputUpdateResponse {
        let startTime = CFAbsoluteTimeGetCurrent()
        
        // Update input with debouncing
        testManager.updateInput(request.inputText)
        
        // Get current metrics
        let metrics = getCurrentMetrics()
        
        let responseTime = CFAbsoluteTimeGetCurrent() - startTime
        performanceMonitor.recordMetric("input_update_time", value: responseTime * 1000)
        
        return InputUpdateResponse(
            sessionID: request.sessionID,
            metrics: metrics,
            remainingTime: testManager.remainingTime,
            differenceHighlights: calculateDifferences(request.inputText, testManager.targetText)
        )
    }
    
    func endTest(_ request: EndTestRequest) async throws -> TypingResultResponse {
        guard let result = testManager.endTest() else {
            throw TypingServiceError.testNotActive
        }
        
        // Save result
        try await repository.save(result)
        
        // Generate review items
        let reviewItems = await generateReviewItems(from: result)
        
        return TypingResultResponse(
            sessionID: request.sessionID,
            result: result.toDTO(),
            reviewItems: reviewItems.map { $0.toDTO() }
        )
    }
    
    private func validateStartRequest(_ request: StartTypingTestRequest) throws {
        guard !request.taskID.uuidString.isEmpty else {
            throw TypingServiceError.invalidTaskID
        }
    }
    
    private func getCurrentMetrics() -> TypingMetricsDTO {
        return TypingMetricsDTO(
            wpm: testManager.currentWPM,
            accuracy: testManager.accuracy,
            qualityIndex: testManager.qualityIndex,
            completionPercentage: testManager.completionPercentage,
            elapsedTime: testManager.elapsedTime
        )
    }
}
```

#### Request/Response データ型

```swift
// MARK: - Requests
struct StartTypingTestRequest {
    let taskID: UUID
    let userSettings: TypingTestSettings
}

struct UpdateInputRequest {
    let sessionID: UUID
    let inputText: String
    let timestamp: Date
}

struct EndTestRequest {
    let sessionID: UUID
    let finalInput: String
    let userTerminated: Bool
}

struct TestMetricsRequest {
    let sessionID: UUID
}

// MARK: - Responses  
struct TypingTestResponse {
    let sessionID: UUID
    let task: IELTSTaskDTO
    let timeLimit: TimeInterval
    let status: TestStatus
}

struct InputUpdateResponse {
    let sessionID: UUID
    let metrics: TypingMetricsDTO
    let remainingTime: TimeInterval
    let differenceHighlights: [DifferenceHighlightDTO]
}

struct TypingResultResponse {
    let sessionID: UUID
    let result: TypingResultDTO
    let reviewItems: [ReviewItemDTO]
}

struct TestMetricsResponse {
    let sessionID: UUID
    let metrics: TypingMetricsDTO
    let isActive: Bool
}

// MARK: - Supporting Types
enum TestStatus: String, Codable {
    case active = "active"
    case paused = "paused"
    case completed = "completed"
    case cancelled = "cancelled"
}

struct TypingTestSettings {
    let enableRealTimeAnalysis: Bool
    let errorAnalysisCategories: Set<ErrorType>
    let showDifferenceHighlights: Bool
    let autoPauseOnInactivity: Bool
}
```

---

### TTSService

#### インターフェース定義
```swift
protocol TTSServiceProtocol {
    func playText(_ request: PlayTextRequest) async throws -> PlaybackResponse
    func pausePlayback(_ request: PausePlaybackRequest) async -> PlaybackStateResponse
    func resumePlayback(_ request: ResumePlaybackRequest) async -> PlaybackStateResponse
    func stopPlayback(_ request: StopPlaybackRequest) async -> PlaybackStateResponse
    func setPlaybackSettings(_ request: PlaybackSettingsRequest) async -> SettingsResponse
    func getPlaybackStatus() async -> PlaybackStatusResponse
}

@MainActor
class TTSService: TTSServiceProtocol {
    private let ttsManager: TTSManager
    private let repository: SettingsRepositoryProtocol
    
    init(repository: SettingsRepositoryProtocol) {
        self.ttsManager = TTSManager()
        self.repository = repository
    }
    
    func playText(_ request: PlayTextRequest) async throws -> PlaybackResponse {
        let startTime = CFAbsoluteTimeGetCurrent()
        
        // Validate request
        try validatePlayRequest(request)
        
        // Apply settings
        ttsManager.setVoiceLanguage(request.settings.voiceLanguage)
        ttsManager.setSpeed(request.settings.playbackSpeed)
        
        // Start playback based on mode
        switch request.playbackMode {
        case .full:
            ttsManager.playFullText(request.text)
        case .sentence:
            ttsManager.playSentence(request.text, at: request.sentenceIndex ?? 0)
        case .phrase:
            ttsManager.playPhrase(request.text)
        }
        
        let responseTime = CFAbsoluteTimeGetCurrent() - startTime
        AppLogger.shared.logTTSEvent("playback_started", duration: responseTime)
        
        return PlaybackResponse(
            sessionID: UUID(),
            status: .playing,
            estimatedDuration: estimateDuration(request.text, speed: request.settings.playbackSpeed)
        )
    }
    
    func setPlaybackSettings(_ request: PlaybackSettingsRequest) async -> SettingsResponse {
        // Apply settings to TTS manager
        ttsManager.setVoiceLanguage(request.settings.voiceLanguage)
        ttsManager.setSpeed(request.settings.playbackSpeed)
        
        // Save settings
        try? await repository.savePlaybackSettings(request.settings)
        
        return SettingsResponse(
            success: true,
            appliedSettings: request.settings
        )
    }
    
    private func validatePlayRequest(_ request: PlayTextRequest) throws {
        guard !request.text.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty else {
            throw TTSServiceError.emptyText
        }
        
        guard request.settings.playbackSpeed >= 0.5 && request.settings.playbackSpeed <= 2.0 else {
            throw TTSServiceError.invalidPlaybackSpeed
        }
    }
    
    private func estimateDuration(_ text: String, speed: Float) -> TimeInterval {
        // Rough estimation: 150 words per minute at normal speed
        let wordsPerMinute = 150.0 * Double(speed)
        let wordCount = Double(text.components(separatedBy: .whitespacesAndNewlines).count)
        return (wordCount / wordsPerMinute) * 60.0
    }
}
```

#### Request/Response データ型
```swift
// MARK: - Requests
struct PlayTextRequest {
    let text: String
    let playbackMode: PlaybackMode
    let sentenceIndex: Int?
    let settings: PlaybackSettings
}

struct PausePlaybackRequest {
    let sessionID: UUID
}

struct StopPlaybackRequest {
    let sessionID: UUID
}

struct PlaybackSettingsRequest {
    let settings: PlaybackSettings
}

// MARK: - Responses
struct PlaybackResponse {
    let sessionID: UUID
    let status: PlaybackStatus
    let estimatedDuration: TimeInterval
}

struct PlaybackStateResponse {
    let sessionID: UUID
    let status: PlaybackStatus
    let currentPosition: TimeInterval
}

struct PlaybackStatusResponse {
    let isPlaying: Bool
    let currentText: String?
    let remainingDuration: TimeInterval
}

// MARK: - Settings
struct PlaybackSettings: Codable {
    let voiceLanguage: VoiceLanguage
    let playbackSpeed: Float
    let volume: Float
    let pitch: Float
    
    static let defaultSettings = PlaybackSettings(
        voiceLanguage: .usEnglish,
        playbackSpeed: 1.0,
        volume: 1.0,
        pitch: 1.0
    )
}

enum PlaybackStatus: String, Codable {
    case playing = "playing"
    case paused = "paused"
    case stopped = "stopped"
    case error = "error"
}
```

---

### AnalysisService

#### インターフェース定義
```swift
protocol AnalysisServiceProtocol {
    func analyzeText(_ request: TextAnalysisRequest) async throws -> AnalysisResponse
    func generateDifferenceHighlights(_ request: DifferenceRequest) async -> DifferenceResponse
    func updateAnalysisSettings(_ request: AnalysisSettingsRequest) async -> SettingsResponse
    func getErrorStatistics(_ request: ErrorStatisticsRequest) async -> ErrorStatisticsResponse
}

class AnalysisService: AnalysisServiceProtocol {
    private let errorAnalyzer: ErrorAnalyzer
    private let differenceCalculator: DifferenceCalculator
    private let repository: TypingResultRepositoryProtocol
    
    init(repository: TypingResultRepositoryProtocol) {
        self.errorAnalyzer = ErrorAnalyzer()
        self.differenceCalculator = DifferenceCalculator()
        self.repository = repository
    }
    
    func analyzeText(_ request: TextAnalysisRequest) async throws -> AnalysisResponse {
        let startTime = CFAbsoluteTimeGetCurrent()
        
        // Perform error analysis
        let errors = errorAnalyzer.analyzeText(request.inputText, target: request.targetText)
        
        // Filter based on enabled categories
        let filteredErrors = errors.filter { request.enabledCategories.contains($0.type) }
        
        // Generate suggestions
        let suggestions = generateSuggestions(for: filteredErrors)
        
        let processingTime = CFAbsoluteTimeGetCurrent() - startTime
        AppLogger.shared.logPerformance("text_analysis_time", value: processingTime * 1000, unit: "ms")
        
        return AnalysisResponse(
            sessionID: request.sessionID,
            errors: filteredErrors.map { $0.toDTO() },
            suggestions: suggestions,
            processingTime: processingTime,
            totalErrors: errors.count,
            enabledCategoryErrors: filteredErrors.count
        )
    }
    
    func generateDifferenceHighlights(_ request: DifferenceRequest) async -> DifferenceResponse {
        let differences = differenceCalculator.calculateDifferences(
            input: request.inputText,
            target: request.targetText
        )
        
        return DifferenceResponse(
            sessionID: request.sessionID,
            highlights: differences.map { $0.toDTO() },
            accuracy: calculateAccuracy(from: differences),
            completionPercentage: calculateCompletion(request.inputText, request.targetText)
        )
    }
    
    func getErrorStatistics(_ request: ErrorStatisticsRequest) async -> ErrorStatisticsResponse {
        let results = try? await repository.fetchResults(for: request.dateRange, taskType: request.taskType)
        
        let statistics = generateErrorStatistics(from: results ?? [])
        
        return ErrorStatisticsResponse(
            statistics: statistics,
            dateRange: request.dateRange,
            totalSessions: results?.count ?? 0
        )
    }
    
    private func generateSuggestions(for errors: [ErrorResult]) -> [SuggestionDTO] {
        return errors.compactMap { error in
            guard !error.suggestion.isEmpty else { return nil }
            
            return SuggestionDTO(
                errorType: error.type,
                suggestion: error.suggestion,
                explanation: getExplanation(for: error.type),
                examples: getExamples(for: error.type),
                priority: error.severity.rawValue
            )
        }
    }
    
    private func calculateAccuracy(from differences: [TextDifference]) -> Double {
        let totalCharacters = differences.reduce(0) { $0 + $1.length }
        let correctCharacters = differences.filter { $0.type == .match }.reduce(0) { $0 + $1.length }
        return totalCharacters > 0 ? Double(correctCharacters) / Double(totalCharacters) * 100 : 100
    }
}
```

#### Request/Response データ型
```swift
// MARK: - Requests
struct TextAnalysisRequest {
    let sessionID: UUID
    let inputText: String
    let targetText: String
    let enabledCategories: Set<ErrorType>
}

struct DifferenceRequest {
    let sessionID: UUID
    let inputText: String
    let targetText: String
}

struct AnalysisSettingsRequest {
    let enabledErrorTypes: Set<ErrorType>
    let strictnessLevel: StrictnessLevel
    let realTimeAnalysis: Bool
}

struct ErrorStatisticsRequest {
    let dateRange: DateRange
    let taskType: TaskType?
    let errorTypes: Set<ErrorType>?
}

// MARK: - Responses
struct AnalysisResponse {
    let sessionID: UUID
    let errors: [ErrorResultDTO]
    let suggestions: [SuggestionDTO]
    let processingTime: TimeInterval
    let totalErrors: Int
    let enabledCategoryErrors: Int
}

struct DifferenceResponse {
    let sessionID: UUID
    let highlights: [DifferenceHighlightDTO]
    let accuracy: Double
    let completionPercentage: Double
}

struct ErrorStatisticsResponse {
    let statistics: ErrorStatisticsDTO
    let dateRange: DateRange
    let totalSessions: Int
}

// MARK: - Supporting Types
struct ErrorStatisticsDTO {
    let errorsByType: [ErrorType: Int]
    let improvementTrends: [ErrorType: TrendData]
    let mostCommonErrors: [CommonErrorDTO]
    let overallAccuracyTrend: TrendData
}

struct TrendData {
    let current: Double
    let previous: Double
    let changePercentage: Double
    let trend: TrendDirection
}

enum TrendDirection: String, Codable {
    case improving = "improving"
    case declining = "declining"
    case stable = "stable"
}

struct CommonErrorDTO {
    let errorType: ErrorType
    let frequency: Int
    let exampleText: String
    let suggestion: String
}

struct SuggestionDTO {
    let errorType: ErrorType
    let suggestion: String
    let explanation: String
    let examples: [String]
    let priority: String
}

enum StrictnessLevel: String, CaseIterable, Codable {
    case lenient = "lenient"
    case moderate = "moderate"
    case strict = "strict"
}
```

---

## 📊 データ転送オブジェクト (DTO)

### Core DTO定義

```swift
// MARK: - IELTS Task DTO
struct IELTSTaskDTO: Codable, Identifiable {
    let id: UUID
    let taskType: TaskType
    let moduleType: ModuleType
    let topic: String
    let prompt: String
    let modelAnswer: String
    let targetBandScore: Double
    let keyVocabulary: [String]
    let keyPhrases: [String]
    let wordCount: Int
    let structure: EssayStructureDTO
    let createdDate: Date
    let lastUsedDate: Date?
    let difficultyLevel: DifficultyLevel
}

extension IELTSTask {
    func toDTO() -> IELTSTaskDTO {
        return IELTSTaskDTO(
            id: id,
            taskType: taskType,
            moduleType: moduleType,
            topic: topic,
            prompt: prompt,
            modelAnswer: modelAnswer,
            targetBandScore: targetBandScore,
            keyVocabulary: keyVocabulary,
            keyPhrases: keyPhrases,
            wordCount: wordCount,
            structure: structure.toDTO(),
            createdDate: createdDate,
            lastUsedDate: lastUsedDate,
            difficultyLevel: difficultyLevel
        )
    }
}

// MARK: - Typing Result DTO
struct TypingResultDTO: Codable, Identifiable {
    let id: UUID
    let sessionDate: Date
    let testDuration: TimeInterval
    let targetText: String
    let userInput: String
    let finalWPM: Double
    let accuracy: Double
    let qualityIndex: Double
    let completionPercentage: Double
    let errorCount: Int
    let errorResults: [ErrorResultDTO]
    let taskID: UUID?
    
    // Performance metrics
    let averageWPM: Double
    let peakWPM: Double
    let characterCount: Int
    let correctCharacterCount: Int
    let timeToFirstError: TimeInterval?
}

extension TypingResult {
    func toDTO() -> TypingResultDTO {
        return TypingResultDTO(
            id: id,
            sessionDate: sessionDate,
            testDuration: testDuration,
            targetText: targetText,
            userInput: userInput,
            finalWPM: finalWPM,
            accuracy: accuracy,
            qualityIndex: qualityIndex,
            completionPercentage: completionPercentage,
            errorCount: errorCount,
            errorResults: errorResults.map { $0.toDTO() },
            taskID: task?.id,
            averageWPM: averageWPM,
            peakWPM: peakWPM,
            characterCount: characterCount,
            correctCharacterCount: correctCharacterCount,
            timeToFirstError: timeToFirstError
        )
    }
}

// MARK: - Error Result DTO
struct ErrorResultDTO: Codable, Identifiable {
    let id: UUID
    let errorType: ErrorType
    let position: Int
    let length: Int
    let expected: String
    let actual: String
    let suggestion: String
    let confidence: Double
    let severity: ErrorSeverity
    let context: String
}

extension ErrorResult {
    func toDTO() -> ErrorResultDTO {
        return ErrorResultDTO(
            id: id,
            errorType: errorType,
            position: position,
            length: length,
            expected: expected,
            actual: actual,
            suggestion: suggestion,
            confidence: confidence,
            severity: severity,
            context: context
        )
    }
}

// MARK: - Review Item DTO
struct ReviewItemDTO: Codable, Identifiable {
    let id: UUID
    let originalText: String
    let userInput: String
    let errorType: ErrorType
    let errorCount: Int
    let firstOccurrence: Date
    let lastReview: Date?
    let masteryLevel: MasteryLevel
    let priority: ReviewPriority
    let reviewCount: Int
    let successfulReviews: Int
    let context: String
    let nextReviewDate: Date
    let successRate: Double
}

extension ReviewItem {
    func toDTO() -> ReviewItemDTO {
        let successRate = reviewCount > 0 ? Double(successfulReviews) / Double(reviewCount) : 0
        
        return ReviewItemDTO(
            id: id,
            originalText: originalText,
            userInput: userInput,
            errorType: errorType,
            errorCount: errorCount,
            firstOccurrence: firstOccurrence,
            lastReview: lastReview,
            masteryLevel: masteryLevel,
            priority: priority,
            reviewCount: reviewCount,
            successfulReviews: successfulReviews,
            context: context,
            nextReviewDate: nextReviewDate,
            successRate: successRate
        )
    }
}

// MARK: - Typing Metrics DTO
struct TypingMetricsDTO: Codable {
    let wpm: Double
    let accuracy: Double
    let qualityIndex: Double
    let completionPercentage: Double
    let elapsedTime: TimeInterval
    let remainingTime: TimeInterval?
    let characterCount: Int
    let correctCharacterCount: Int
    let errorCount: Int
}

// MARK: - Difference Highlight DTO
struct DifferenceHighlightDTO: Codable {
    let type: DifferenceType
    let range: NSRange
    let text: String
    let errorType: ErrorType?
    let severity: ErrorSeverity?
}

enum DifferenceType: String, Codable {
    case match = "match"
    case insertion = "insertion"
    case deletion = "deletion" 
    case substitution = "substitution"
}

// MARK: - Supporting DTOs
struct EssayStructureDTO: Codable {
    let introduction: RangeDTO
    let body: [RangeDTO]
    let conclusion: RangeDTO
}

struct RangeDTO: Codable {
    let location: Int
    let length: Int
}

extension Range where Bound == String.Index {
    func toDTO() -> RangeDTO {
        // Convert String.Index range to Int range
        // This would need the original string for proper conversion
        // For now, returning placeholder
        return RangeDTO(location: 0, length: 0)
    }
}

struct DateRange: Codable {
    let startDate: Date
    let endDate: Date
    
    static func lastWeek() -> DateRange {
        let endDate = Date()
        let startDate = Calendar.current.date(byAdding: .day, value: -7, to: endDate)!
        return DateRange(startDate: startDate, endDate: endDate)
    }
    
    static func lastMonth() -> DateRange {
        let endDate = Date()
        let startDate = Calendar.current.date(byAdding: .month, value: -1, to: endDate)!
        return DateRange(startDate: startDate, endDate: endDate)
    }
}
```

---

## ❌ エラーハンドリング

### Service Errors
```swift
// MARK: - Service Errors
enum TypingServiceError: LocalizedError, CaseIterable {
    case taskNotFound
    case testNotActive
    case invalidTaskID
    case testAlreadyActive
    case invalidInput
    
    var errorDescription: String? {
        switch self {
        case .taskNotFound: return "IELTS task not found"
        case .testNotActive: return "No active typing test"
        case .invalidTaskID: return "Invalid task identifier"
        case .testAlreadyActive: return "A typing test is already active"
        case .invalidInput: return "Invalid input data"
        }
    }
}

enum TTSServiceError: LocalizedError, CaseIterable {
    case emptyText
    case invalidPlaybackSpeed
    case voiceNotAvailable
    case playbackFailed
    case invalidSettings
    
    var errorDescription: String? {
        switch self {
        case .emptyText: return "Text is empty or invalid"
        case .invalidPlaybackSpeed: return "Playback speed must be between 0.5 and 2.0"
        case .voiceNotAvailable: return "Selected voice is not available"
        case .playbackFailed: return "Failed to start playback"
        case .invalidSettings: return "Invalid playback settings"
        }
    }
}

enum AnalysisServiceError: LocalizedError, CaseIterable {
    case textTooLong
    case analysisTimeout
    case unsupportedLanguage
    case configurationError
    
    var errorDescription: String? {
        switch self {
        case .textTooLong: return "Text is too long for analysis"
        case .analysisTimeout: return "Analysis operation timed out"
        case .unsupportedLanguage: return "Language not supported for analysis"
        case .configurationError: return "Analysis configuration error"
        }
    }
}

enum ReviewServiceError: LocalizedError, CaseIterable {
    case reviewItemNotFound
    case invalidMasteryLevel
    case reviewGenerationFailed
    
    var errorDescription: String? {
        switch self {
        case .reviewItemNotFound: return "Review item not found"
        case .invalidMasteryLevel: return "Invalid mastery level"
        case .reviewGenerationFailed: return "Failed to generate review items"
        }
    }
}
```

### API Response Wrapper
```swift
struct APIResponse<T: Codable>: Codable {
    let success: Bool
    let data: T?
    let error: APIError?
    let timestamp: Date
    let processingTime: TimeInterval?
    
    init(success: Bool, data: T? = nil, error: APIError? = nil, processingTime: TimeInterval? = nil) {
        self.success = success
        self.data = data
        self.error = error
        self.timestamp = Date()
        self.processingTime = processingTime
    }
}

struct APIError: Codable, LocalizedError {
    let code: String
    let message: String
    let details: [String: String]?
    
    var errorDescription: String? {
        return message
    }
}

// MARK: - Result Extensions
extension Result where Success: Codable, Failure: LocalizedError {
    func toAPIResponse(processingTime: TimeInterval? = nil) -> APIResponse<Success> {
        switch self {
        case .success(let data):
            return APIResponse(success: true, data: data, processingTime: processingTime)
        case .failure(let error):
            let apiError = APIError(
                code: String(describing: type(of: error)),
                message: error.localizedDescription,
                details: nil
            )
            return APIResponse(success: false, error: apiError, processingTime: processingTime)
        }
    }
}
```

---

## 🔄 Repository Pattern

### Repository Protocol定義
```swift
protocol RepositoryProtocol {
    associatedtype Entity
    associatedtype EntityID
    
    func save(_ entity: Entity) async throws
    func fetch(id: EntityID) async throws -> Entity?
    func fetchAll() async throws -> [Entity]
    func delete(id: EntityID) async throws
    func update(_ entity: Entity) async throws
}

// MARK: - IELTS Task Repository
protocol IELTSTaskRepositoryProtocol: RepositoryProtocol where Entity == IELTSTask, EntityID == UUID {
    func fetchByTaskType(_ taskType: TaskType) async throws -> [IELTSTask]
    func fetchByBandScore(_ bandScore: Double) async throws -> [IELTSTask]
    func fetchByTopic(_ topic: String) async throws -> [IELTSTask]
    func searchTasks(_ query: String) async throws -> [IELTSTask]
}

// MARK: - Typing Result Repository
protocol TypingResultRepositoryProtocol: RepositoryProtocol where Entity == TypingResult, EntityID == UUID {
    func fetchResults(for dateRange: DateRange, taskType: TaskType?) async throws -> [TypingResult]
    func fetchRecentResults(limit: Int) async throws -> [TypingResult]
    func getStatistics(for dateRange: DateRange) async throws -> TypingStatistics
    func deleteOldResults(olderThan date: Date) async throws
}

// MARK: - Review Item Repository
protocol ReviewItemRepositoryProtocol: RepositoryProtocol where Entity == ReviewItem, EntityID == UUID {
    func fetchDueForReview() async throws -> [ReviewItem]
    func fetchByErrorType(_ errorType: ErrorType) async throws -> [ReviewItem]
    func fetchByMasteryLevel(_ level: MasteryLevel) async throws -> [ReviewItem]
    func updateMasteryLevel(id: UUID, level: MasteryLevel, success: Bool) async throws
}
```

### Repository 実装例
```swift
class SwiftDataIELTSTaskRepository: IELTSTaskRepositoryProtocol {
    private let modelContext: ModelContext
    
    init(modelContext: ModelContext) {
        self.modelContext = modelContext
    }
    
    func save(_ entity: IELTSTask) async throws {
        modelContext.insert(entity)
        try modelContext.save()
    }
    
    func fetch(id: UUID) async throws -> IELTSTask? {
        let descriptor = FetchDescriptor<IELTSTask>(
            predicate: #Predicate { $0.id == id }
        )
        let results = try modelContext.fetch(descriptor)
        return results.first
    }
    
    func fetchAll() async throws -> [IELTSTask] {
        let descriptor = FetchDescriptor<IELTSTask>()
        return try modelContext.fetch(descriptor)
    }
    
    func fetchByTaskType(_ taskType: TaskType) async throws -> [IELTSTask] {
        let descriptor = FetchDescriptor<IELTSTask>(
            predicate: #Predicate { $0.taskType == taskType },
            sortBy: [SortDescriptor(\.lastUsedDate, order: .reverse)]
        )
        return try modelContext.fetch(descriptor)
    }
    
    func fetchByBandScore(_ bandScore: Double) async throws -> [IELTSTask] {
        let descriptor = FetchDescriptor<IELTSTask>(
            predicate: #Predicate { $0.targetBandScore == bandScore },
            sortBy: [SortDescriptor(\.topic)]
        )
        return try modelContext.fetch(descriptor)
    }
    
    func searchTasks(_ query: String) async throws -> [IELTSTask] {
        let descriptor = FetchDescriptor<IELTSTask>(
            predicate: #Predicate { task in
                task.topic.contains(query) || 
                task.modelAnswer.contains(query) ||
                task.keyVocabulary.contains { $0.contains(query) }
            }
        )
        return try modelContext.fetch(descriptor)
    }
    
    func delete(id: UUID) async throws {
        guard let task = try await fetch(id: id) else { return }
        modelContext.delete(task)
        try modelContext.save()
    }
    
    func update(_ entity: IELTSTask) async throws {
        try modelContext.save()
    }
}
```

---

## 📱 Export/Import API

### Export Service
```swift
protocol ExportServiceProtocol {
    func exportToCSV(_ request: ExportRequest) async throws -> ExportResponse
    func exportToJSON(_ request: ExportRequest) async throws -> ExportResponse
    func exportToBackup(_ request: BackupRequest) async throws -> BackupResponse
}

class ExportService: ExportServiceProtocol {
    private let resultRepository: TypingResultRepositoryProtocol
    private let taskRepository: IELTSTaskRepositoryProtocol
    private let reviewRepository: ReviewItemRepositoryProtocol
    
    func exportToCSV(_ request: ExportRequest) async throws -> ExportResponse {
        let results = try await resultRepository.fetchResults(
            for: request.dateRange,
            taskType: request.taskType
        )
        
        let csvData = try generateCSV(from: results, format: request.format)
        let fileURL = try saveToFile(csvData, extension: "csv")
        
        return ExportResponse(
            fileURL: fileURL,
            format: .csv,
            recordCount: results.count,
            fileSize: csvData.count
        )
    }
    
    func exportToJSON(_ request: ExportRequest) async throws -> ExportResponse {
        let results = try await resultRepository.fetchResults(
            for: request.dateRange,
            taskType: request.taskType
        )
        
        let jsonData = try generateJSON(from: results, format: request.format)
        let fileURL = try saveToFile(jsonData, extension: "json")
        
        return ExportResponse(
            fileURL: fileURL,
            format: .json,
            recordCount: results.count,
            fileSize: jsonData.count
        )
    }
    
    private func generateCSV(from results: [TypingResult], format: ExportFormat) throws -> Data {
        var csvContent = generateCSVHeader(for: format)
        
        for result in results {
            csvContent += generateCSVRow(for: result, format: format)
        }
        
        guard let data = csvContent.data(using: .utf8) else {
            throw ExportServiceError.encodingFailed
        }
        
        return data
    }
    
    private func generateCSVHeader(for format: ExportFormat) -> String {
        switch format {
        case .summary:
            return "Date,Task Type,WPM,Accuracy,Quality Index,Duration\n"
        case .detailed:
            return "Date,Task Type,WPM,Accuracy,Quality Index,Duration,Error Count,Completion %\n"
        case .full:
            return "Date,Task Type,WPM,Accuracy,Quality Index,Duration,Error Count,Completion %,Target Text,User Input\n"
        }
    }
}

// MARK: - Export Types
struct ExportRequest {
    let dateRange: DateRange
    let taskType: TaskType?
    let format: ExportFormat
    let includeErrorDetails: Bool
}

struct ExportResponse {
    let fileURL: URL
    let format: ExportFormat
    let recordCount: Int
    let fileSize: Int
}

enum ExportFormat: String, CaseIterable {
    case summary = "summary"
    case detailed = "detailed"
    case full = "full"
}

enum ExportServiceError: LocalizedError {
    case noDataToExport
    case encodingFailed
    case fileCreationFailed
    
    var errorDescription: String? {
        switch self {
        case .noDataToExport: return "No data available for export"
        case .encodingFailed: return "Failed to encode data"
        case .fileCreationFailed: return "Failed to create export file"
        }
    }
}
```

---

## 🔧 Future Extensions

### Cloud Sync API (Future)
```swift
protocol CloudSyncServiceProtocol {
    func syncUserData() async throws -> SyncResponse
    func uploadResults(_ results: [TypingResult]) async throws -> UploadResponse
    func downloadTasks() async throws -> DownloadResponse
    func resolveConflicts(_ conflicts: [SyncConflict]) async throws -> ConflictResolutionResponse
}

struct SyncResponse {
    let syncID: UUID
    let status: SyncStatus
    let uploadCount: Int
    let downloadCount: Int
    let conflictCount: Int
}

enum SyncStatus: String, Codable {
    case success = "success"
    case partial = "partial"
    case failed = "failed"
    case conflict = "conflict"
}
```

### Plugin API (Future)
```swift
protocol PluginServiceProtocol {
    func loadPlugin(_ plugin: PluginDefinition) async throws -> PluginResponse
    func executePlugin(_ pluginID: UUID, parameters: [String: Any]) async throws -> PluginExecutionResponse
    func listAvailablePlugins() async -> [PluginDefinition]
}

struct PluginDefinition: Codable {
    let id: UUID
    let name: String
    let version: String
    let capabilities: [PluginCapability]
    let entryPoint: String
}

enum PluginCapability: String, Codable {
    case errorAnalysis = "error_analysis"
    case textSuggestion = "text_suggestion"
    case customReporting = "custom_reporting"
}
```

---

*この API仕様書は、IELTS Writing Practice Appの内部サービス構成と将来の外部連携に対応する包括的な API設計を定義しています。*