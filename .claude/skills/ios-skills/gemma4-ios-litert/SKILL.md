---
name: gemma4-ios-litert
description: Build iOS apps that run Google Gemma 4 (E2B / E4B) on-device using LiteRT-LM. Use this skill whenever the user mentions Gemma 4, LiteRT-LM, on-device LLM for iOS, edge AI on iPhone/iPad, Google AI Edge Gallery iOS, or wants to deploy a local language model on Apple hardware with no cloud dependency. Also trigger when the user asks about running open LLMs offline on iOS, private on-device chat, or integrating .litertlm model files into an Xcode project. Covers model selection (E2B vs E4B), Swift integration, streaming generation, function calling, memory management, and full Xcode project scaffolding.
---

# Gemma 4 on iOS with LiteRT-LM

This skill generates a production-ready iOS (Swift / SwiftUI) project that runs **Google Gemma 4 E2B or E4B** entirely on-device using the **LiteRT-LM** framework. No server, no API key, no internet required after the initial model download.

---

## Quick facts

| Detail | E2B | E4B |
|---|---|---|
| Effective active params | 2 B | 4 B |
| Architecture | MoE | MoE |
| Quantized file size | ~2.6 GB | ~4.5 GB |
| RAM (quantized) | ~1.5 GB | ~2.5 GB |
| Max context length | 32 768 tokens | 32 768 tokens |
| Modalities (text) | ✅ | ✅ |
| Function calling | ✅ | ✅ |
| Min iPhone chip | A14 (iPhone 12) | A15 (iPhone 13 Pro+) |
| Tokens/sec (A17 Pro) | ~25-35 | ~15-25 |

> **Rule of thumb:** Use E2B for speed-sensitive, simpler tasks. Use E4B when reasoning quality matters and the device has ≥ 8 GB RAM.

---

## Before you begin — read the reference

Before writing any code, **read the appropriate reference file**:

- For the full iOS project scaffold → `view /path/to/this-skill/references/ios-project-guide.md`

---

## Workflow

### 1. Choose the model variant

Ask the user (or infer from context):

- **E2B** — lighter, faster, works on more devices (iPhone 12+).
- **E4B** — smarter, better reasoning & function calling, needs flagship hardware (iPhone 13 Pro+ / any iPad with M-chip).

### 2. Download the model file

The `.litertlm` model files are hosted on Hugging Face:

| Model | Hugging Face repo |
|---|---|
| E2B | `litert-community/gemma-4-E2B-it-litert-lm` |
| E4B | `litert-community/gemma-4-E4B-it-litert-lm` |

The model file is **NOT** bundled inside the app binary. Instead the app downloads it on first launch (or the user sideloads it). The skill generates download-manager code that:
- Shows a progress bar during download.
- Stores the file in the app's `Application Support` directory.
- Validates the file hash before use.

### 3. Scaffold the Xcode project

Generate a SwiftUI project with:

```
GemmaChat/
├── GemmaChat.xcodeproj
├── Package.swift              # SPM dependency on LiteRT-LM
├── Sources/
│   ├── App/
│   │   └── GemmaChatApp.swift
│   ├── Models/
│   │   ├── LLMEngine.swift          # LiteRT-LM wrapper
│   │   ├── ConversationManager.swift # Multi-turn state
│   │   └── ModelDownloader.swift     # HF download + caching
│   ├── Views/
│   │   ├── ChatView.swift            # Main chat UI
│   │   ├── MessageBubble.swift
│   │   ├── ModelPickerView.swift     # E2B / E4B selector
│   │   └── SettingsView.swift        # Temperature, max tokens, context
│   └── Utilities/
│       ├── TokenCounter.swift
│       └── MemoryMonitor.swift
└── Resources/
    └── Assets.xcassets
```

### 4. Core integration code

#### Package.swift dependency

```swift
// swift-tools-version:5.9
import PackageDescription

let package = Package(
    name: "GemmaChat",
    platforms: [.iOS(.v16)],
    dependencies: [
        .package(url: "https://github.com/google-ai-edge/LiteRT-LM", from: "1.0.0")
    ],
    targets: [
        .target(
            name: "GemmaChat",
            dependencies: [
                .product(name: "LiteRTLM", package: "LiteRT-LM")
            ]
        )
    ]
)
```

#### LLMEngine.swift — Core wrapper

```swift
import Foundation
import LiteRTLM

@MainActor
final class LLMEngine: ObservableObject {
    @Published var isLoaded = false
    @Published var isGenerating = false
    @Published var loadError: String?
    
    private var model: LiteRtLmModel?
    private var session: LiteRtLmSession?
    
    // MARK: - Load model
    func loadModel(at path: String) async {
        do {
            model = try LiteRtLmModel(path: path)
            session = try model?.createSession()
            isLoaded = true
        } catch {
            loadError = "Failed to load model: \(error.localizedDescription)"
        }
    }
    
    // MARK: - Generate (streaming)
    func generate(prompt: String, maxTokens: Int = 1024, temperature: Float = 0.7) -> AsyncStream<String> {
        AsyncStream { continuation in
            Task {
                guard let session else {
                    continuation.finish()
                    return
                }
                isGenerating = true
                do {
                    let stream = try session.generateStream(
                        for: prompt,
                        config: .init(maxTokens: maxTokens, temperature: temperature)
                    )
                    for try await chunk in stream {
                        continuation.yield(chunk)
                    }
                } catch {
                    continuation.yield("[Error] \(error.localizedDescription)")
                }
                isGenerating = false
                continuation.finish()
            }
        }
    }
    
    // MARK: - Reset conversation
    func resetConversation() {
        session = try? model?.createSession()
    }
}
```

#### ModelDownloader.swift — Download from Hugging Face

```swift
import Foundation

final class ModelDownloader: ObservableObject {
    @Published var progress: Double = 0
    @Published var isDownloading = false
    @Published var downloadError: String?
    
    private let fileManager = FileManager.default
    
    var modelDirectory: URL {
        fileManager.urls(for: .applicationSupportDirectory, in: .userDomainMask)[0]
            .appendingPathComponent("GemmaModels", isDirectory: true)
    }
    
    func modelPath(variant: String) -> URL {
        modelDirectory.appendingPathComponent("gemma-4-\(variant)-it.litertlm")
    }
    
    func isModelDownloaded(variant: String) -> Bool {
        fileManager.fileExists(atPath: modelPath(variant: variant).path)
    }
    
    func downloadModel(variant: String) async throws -> URL {
        let destination = modelPath(variant: variant)
        if isModelDownloaded(variant: variant) { return destination }
        
        try fileManager.createDirectory(at: modelDirectory, withIntermediateDirectories: true)
        
        let repoBase = "https://huggingface.co/litert-community/gemma-4-\(variant)-it-litert-lm/resolve/main"
        let url = URL(string: "\(repoBase)/gemma-4-\(variant)-it.litertlm")!
        
        isDownloading = true
        defer { isDownloading = false }
        
        let (tempURL, _) = try await URLSession.shared.download(from: url, delegate: ProgressDelegate { [weak self] pct in
            Task { @MainActor in self?.progress = pct }
        })
        
        try fileManager.moveItem(at: tempURL, to: destination)
        return destination
    }
}
```

#### ChatView.swift — Minimal chat UI

```swift
import SwiftUI

struct ChatView: View {
    @StateObject private var engine = LLMEngine()
    @StateObject private var downloader = ModelDownloader()
    @State private var messages: [(role: String, text: String)] = []
    @State private var input = ""
    @State private var streamedResponse = ""
    @State private var selectedVariant = "E2B"
    
    var body: some View {
        NavigationStack {
            VStack {
                if !engine.isLoaded {
                    modelSetupView
                } else {
                    chatContentView
                }
            }
            .navigationTitle("Gemma 4 Chat")
        }
    }
    
    // Model setup / download progress
    private var modelSetupView: some View {
        VStack(spacing: 20) {
            Picker("Model", selection: $selectedVariant) {
                Text("E2B (faster)").tag("E2B")
                Text("E4B (smarter)").tag("E4B")
            }
            .pickerStyle(.segmented)
            
            if downloader.isDownloading {
                ProgressView(value: downloader.progress)
                Text("\(Int(downloader.progress * 100))%")
            }
            
            Button("Download & Load") {
                Task {
                    let path = try await downloader.downloadModel(variant: selectedVariant)
                    await engine.loadModel(at: path.path)
                }
            }
            .buttonStyle(.borderedProminent)
        }
        .padding()
    }
    
    // Chat messages + input
    private var chatContentView: some View {
        VStack {
            ScrollView {
                ForEach(Array(messages.enumerated()), id: \.offset) { _, msg in
                    MessageBubble(role: msg.role, text: msg.text)
                }
                if !streamedResponse.isEmpty {
                    MessageBubble(role: "assistant", text: streamedResponse)
                }
            }
            
            HStack {
                TextField("Message", text: $input)
                    .textFieldStyle(.roundedBorder)
                Button(action: sendMessage) {
                    Image(systemName: "arrow.up.circle.fill")
                        .font(.title2)
                }
                .disabled(input.isEmpty || engine.isGenerating)
            }
            .padding()
        }
    }
    
    private func sendMessage() {
        let userText = input
        input = ""
        messages.append((role: "user", text: userText))
        streamedResponse = ""
        
        Task {
            for await chunk in engine.generate(prompt: userText) {
                streamedResponse += chunk
            }
            messages.append((role: "assistant", text: streamedResponse))
            streamedResponse = ""
        }
    }
}
```

### 5. Memory management

On-device LLM inference is memory-intensive. The skill generates a `MemoryMonitor` utility:

```swift
import Foundation
import os

struct MemoryMonitor {
    /// Returns current physical memory footprint in MB
    static var currentUsageMB: Double {
        var info = task_vm_info_data_t()
        var count = mach_msg_type_number_t(MemoryLayout<task_vm_info_data_t>.size) / 4
        let result = withUnsafeMutablePointer(to: &info) {
            $0.withMemoryRebound(to: integer_t.self, capacity: Int(count)) {
                task_info(mach_task_self_, task_flavor_t(TASK_VM_INFO), $0, &count)
            }
        }
        guard result == KERN_SUCCESS else { return -1 }
        return Double(info.phys_footprint) / (1024 * 1024)
    }
    
    /// Warns if memory exceeds threshold (default 3 GB)
    static func shouldReduceContext(thresholdMB: Double = 3072) -> Bool {
        currentUsageMB > thresholdMB
    }
}
```

Use this to dynamically truncate conversation history or reduce context window when memory pressure is high.

### 6. Configuration & generation parameters

| Parameter | Default | Notes |
|---|---|---|
| `maxTokens` | 1024 | Max output tokens per turn |
| `temperature` | 0.7 | 0.0 = deterministic, 1.0 = creative |
| `contextLength` | 4096 | Start conservative; can go up to 32768 |
| `topK` | 40 | Top-K sampling |
| `topP` | 0.95 | Nucleus sampling |

> ⚠️ **Context length vs. memory:** 32k context is supported but will consume significantly more RAM. Start with 4096 and increase only on devices with ≥ 8 GB RAM. Monitor with `MemoryMonitor`.

### 7. Function calling (tool use)

Gemma 4 E2B/E4B support native function calling. Structure your system prompt with tool definitions:

```swift
let systemPrompt = """
You are a helpful assistant with access to the following tools:
[
  {
    "name": "get_weather",
    "description": "Get current weather for a city",
    "parameters": {
      "type": "object",
      "properties": {
        "city": { "type": "string" }
      },
      "required": ["city"]
    }
  }
]
When you need to call a tool, respond with a JSON function call block.
"""
```

Parse the model output for tool-call JSON, execute locally, and feed results back into the conversation.

### 8. Vision Integration (Ask Image)

Since Gemma 4 E2B/E4B are **text-only models**, image understanding requires a two-stage pipeline:

```
┌─────────┐     ┌─────────────┐     ┌───────────────────┐     ┌───────┐
│  Image  │ ──▶ │   Vision    │ ──▶ │ Text Description  │ ──▶ │ Gemma │
│ (Photo) │     │  Framework  │     │  "2 faces, text   │     │  LLM  │
└─────────┘     └─────────────┘     │   reading Hello"  │     └───────┘
                                    └───────────────────┘
```

#### ImageAnalyzer.swift — Vision framework wrapper

```swift
import Vision
import UIKit

@MainActor
final class ImageAnalyzer: ObservableObject {
    @Published var isAnalyzing = false

    // MARK: - Analyze image with multiple Vision requests
    func analyzeImage(_ image: UIImage) async -> ImageAnalysisResult {
        guard let cgImage = image.cgImage else {
            return ImageAnalysisResult(error: "Failed to get CGImage")
        }

        isAnalyzing = true
        defer { isAnalyzing = false }

        var result = ImageAnalysisResult()

        // Run multiple analyses in parallel
        async let textResult = recognizeText(in: cgImage)
        async let facesResult = detectFaces(in: cgImage)
        async let objectsResult = classifyScene(in: cgImage)
        async let barcodesResult = detectBarcodes(in: cgImage)

        result.recognizedText = await textResult
        result.detectedFaces = await facesResult
        result.sceneClassifications = await objectsResult
        result.detectedBarcodes = await barcodesResult

        return result
    }

    // MARK: - Text Recognition (OCR)
    private func recognizeText(in image: CGImage) async -> [String] {
        await withCheckedContinuation { continuation in
            let request = VNRecognizeTextRequest { request, error in
                guard let observations = request.results as? [VNRecognizedTextObservation] else {
                    continuation.resume(returning: [])
                    return
                }
                let texts = observations.compactMap { $0.topCandidates(1).first?.string }
                continuation.resume(returning: texts)
            }
            request.recognitionLevel = .accurate
            request.usesLanguageCorrection = true

            let handler = VNImageRequestHandler(cgImage: image, options: [:])
            try? handler.perform([request])
        }
    }

    // MARK: - Face Detection
    private func detectFaces(in image: CGImage) async -> Int {
        await withCheckedContinuation { continuation in
            let request = VNDetectFaceRectanglesRequest { request, error in
                let count = request.results?.count ?? 0
                continuation.resume(returning: count)
            }

            let handler = VNImageRequestHandler(cgImage: image, options: [:])
            try? handler.perform([request])
        }
    }

    // MARK: - Scene Classification
    private func classifyScene(in image: CGImage) async -> [String] {
        await withCheckedContinuation { continuation in
            let request = VNClassifyImageRequest { request, error in
                guard let observations = request.results as? [VNClassificationObservation] else {
                    continuation.resume(returning: [])
                    return
                }
                // Get top 5 classifications with confidence > 0.3
                let labels = observations
                    .filter { $0.confidence > 0.3 }
                    .prefix(5)
                    .map { "\($0.identifier) (\(Int($0.confidence * 100))%)" }
                continuation.resume(returning: Array(labels))
            }

            let handler = VNImageRequestHandler(cgImage: image, options: [:])
            try? handler.perform([request])
        }
    }

    // MARK: - Barcode/QR Detection
    private func detectBarcodes(in image: CGImage) async -> [String] {
        await withCheckedContinuation { continuation in
            let request = VNDetectBarcodesRequest { request, error in
                guard let observations = request.results as? [VNBarcodeObservation] else {
                    continuation.resume(returning: [])
                    return
                }
                let barcodes = observations.compactMap { observation -> String? in
                    guard let payload = observation.payloadStringValue else { return nil }
                    return "\(observation.symbology.rawValue): \(payload)"
                }
                continuation.resume(returning: barcodes)
            }

            let handler = VNImageRequestHandler(cgImage: image, options: [:])
            try? handler.perform([request])
        }
    }
}
```

#### ImageAnalysisResult.swift — Structured result

```swift
struct ImageAnalysisResult {
    var recognizedText: [String] = []
    var detectedFaces: Int = 0
    var sceneClassifications: [String] = []
    var detectedBarcodes: [String] = []
    var error: String?

    /// Convert to natural language description for LLM input
    func toTextDescription() -> String {
        var parts: [String] = []

        if !sceneClassifications.isEmpty {
            parts.append("Scene: \(sceneClassifications.joined(separator: ", "))")
        }

        if detectedFaces > 0 {
            let faceWord = detectedFaces == 1 ? "face" : "faces"
            parts.append("Detected \(detectedFaces) \(faceWord)")
        }

        if !recognizedText.isEmpty {
            let textPreview = recognizedText.prefix(10).joined(separator: " ")
            parts.append("Text in image: \"\(textPreview)\"")
        }

        if !detectedBarcodes.isEmpty {
            parts.append("Barcodes: \(detectedBarcodes.joined(separator: ", "))")
        }

        if parts.isEmpty {
            return "No significant content detected in the image."
        }

        return parts.joined(separator: ". ") + "."
    }
}
```

#### Integration with LLMEngine

```swift
extension LLMEngine {
    /// Generate response about an image using Vision analysis
    func askAboutImage(_ image: UIImage, question: String, analyzer: ImageAnalyzer) -> AsyncStream<String> {
        AsyncStream { continuation in
            Task {
                // Step 1: Analyze image with Vision
                let analysis = await analyzer.analyzeImage(image)
                let imageDescription = analysis.toTextDescription()

                // Step 2: Build prompt with image context
                let prompt = """
                [Image Analysis]
                \(imageDescription)

                [User Question]
                \(question)

                Based on the image analysis above, please answer the user's question.
                """

                // Step 3: Generate response with Gemma
                for await chunk in self.generate(prompt: prompt) {
                    continuation.yield(chunk)
                }
                continuation.finish()
            }
        }
    }
}
```

#### ChatView extension with image picker

```swift
import PhotosUI

extension ChatView {
    // Add to ChatView state:
    // @State private var selectedPhoto: PhotosPickerItem?
    // @State private var selectedImage: UIImage?
    // @StateObject private var imageAnalyzer = ImageAnalyzer()

    var imagePickerButton: some View {
        PhotosPicker(selection: $selectedPhoto, matching: .images) {
            Image(systemName: "photo.on.rectangle")
                .font(.title2)
        }
        .onChange(of: selectedPhoto) { newItem in
            Task {
                if let data = try? await newItem?.loadTransferable(type: Data.self),
                   let image = UIImage(data: data) {
                    selectedImage = image
                }
            }
        }
    }

    func sendImageMessage(question: String) {
        guard let image = selectedImage else { return }

        messages.append((role: "user", text: "[Image attached] \(question)"))
        streamedResponse = ""

        Task {
            for await chunk in engine.askAboutImage(image, question: question, analyzer: imageAnalyzer) {
                streamedResponse += chunk
            }
            messages.append((role: "assistant", text: streamedResponse))
            streamedResponse = ""
            selectedImage = nil
        }
    }
}
```

#### Memory considerations

Running Vision analysis and LLM inference simultaneously can strain device memory:

| Configuration | Peak RAM | Recommendation |
|---|---|---|
| Vision only | ~200 MB | Safe on all devices |
| E2B + Vision | ~1.7 GB | OK on iPhone 12+ |
| E4B + Vision | ~2.7 GB | Requires 6GB+ RAM |

**Best practices:**
1. Release Vision request handlers after use
2. Resize large images before analysis (max 2048px recommended)
3. Run Vision analysis sequentially with LLM, not in parallel
4. Monitor memory with `MemoryMonitor` and reduce context if needed

### 9. Audio Scribe (Speech-to-Text)

Voice input follows a similar pattern — transcribe speech to text, then send to Gemma:

```
┌─────────────┐     ┌─────────────┐     ┌──────────┐     ┌───────┐
│ Microphone  │ ──▶ │   Speech    │ ──▶ │   Text   │ ──▶ │ Gemma │
│   Audio     │     │  Framework  │     │ "Hello"  │     │  LLM  │
└─────────────┘     └─────────────┘     └──────────┘     └───────┘
```

#### Info.plist requirements

```xml
<!-- Required for microphone access -->
<key>NSMicrophoneUsageDescription</key>
<string>This app uses the microphone for voice input to the AI assistant.</string>

<!-- Required for speech recognition -->
<key>NSSpeechRecognitionUsageDescription</key>
<string>This app uses speech recognition to convert your voice to text.</string>
```

#### AudioTranscriber.swift — Speech framework wrapper

```swift
import Speech
import AVFoundation

@MainActor
final class AudioTranscriber: ObservableObject {
    @Published var isRecording = false
    @Published var isAuthorized = false
    @Published var transcribedText = ""
    @Published var error: String?

    private var recognizer: SFSpeechRecognizer?
    private var recognitionRequest: SFSpeechAudioBufferRecognitionRequest?
    private var recognitionTask: SFSpeechRecognitionTask?
    private let audioEngine = AVAudioEngine()

    init() {
        recognizer = SFSpeechRecognizer(locale: Locale(identifier: "en-US"))
    }

    // MARK: - Request authorization
    func requestAuthorization() async {
        let speechStatus = await withCheckedContinuation { continuation in
            SFSpeechRecognizer.requestAuthorization { status in
                continuation.resume(returning: status)
            }
        }

        let audioStatus = await AVAudioApplication.requestRecordPermission()

        isAuthorized = speechStatus == .authorized && audioStatus
    }

    // MARK: - Start live transcription
    func startRecording() throws {
        // Cancel any existing task
        recognitionTask?.cancel()
        recognitionTask = nil

        // Configure audio session
        let audioSession = AVAudioSession.sharedInstance()
        try audioSession.setCategory(.record, mode: .measurement, options: .duckOthers)
        try audioSession.setActive(true, options: .notifyOthersOnDeactivation)

        // Create recognition request
        recognitionRequest = SFSpeechAudioBufferRecognitionRequest()
        guard let recognitionRequest = recognitionRequest else {
            throw TranscriberError.requestCreationFailed
        }

        // Enable on-device recognition if available (iOS 13+)
        if recognizer?.supportsOnDeviceRecognition == true {
            recognitionRequest.requiresOnDeviceRecognition = true
        }

        recognitionRequest.shouldReportPartialResults = true

        // Set up recognition task
        recognitionTask = recognizer?.recognitionTask(with: recognitionRequest) { [weak self] result, error in
            Task { @MainActor in
                if let result = result {
                    self?.transcribedText = result.bestTranscription.formattedString
                }
                if error != nil || result?.isFinal == true {
                    self?.stopRecording()
                }
            }
        }

        // Configure audio input
        let inputNode = audioEngine.inputNode
        let recordingFormat = inputNode.outputFormat(forBus: 0)

        inputNode.installTap(onBus: 0, bufferSize: 1024, format: recordingFormat) { buffer, _ in
            self.recognitionRequest?.append(buffer)
        }

        audioEngine.prepare()
        try audioEngine.start()

        isRecording = true
    }

    // MARK: - Stop recording
    func stopRecording() {
        audioEngine.stop()
        audioEngine.inputNode.removeTap(onBus: 0)
        recognitionRequest?.endAudio()
        recognitionRequest = nil
        recognitionTask = nil
        isRecording = false
    }

    // MARK: - Transcribe audio file
    func transcribeFile(at url: URL) async throws -> String {
        guard let recognizer = recognizer, recognizer.isAvailable else {
            throw TranscriberError.recognizerUnavailable
        }

        let request = SFSpeechURLRecognitionRequest(url: url)
        request.shouldReportPartialResults = false

        // Use on-device if available
        if recognizer.supportsOnDeviceRecognition {
            request.requiresOnDeviceRecognition = true
        }

        return try await withCheckedThrowingContinuation { continuation in
            recognizer.recognitionTask(with: request) { result, error in
                if let error = error {
                    continuation.resume(throwing: error)
                } else if let result = result, result.isFinal {
                    continuation.resume(returning: result.bestTranscription.formattedString)
                }
            }
        }
    }
}

enum TranscriberError: Error {
    case requestCreationFailed
    case recognizerUnavailable
}
```

#### ChatView extension with microphone button

```swift
extension ChatView {
    // Add to ChatView state:
    // @StateObject private var transcriber = AudioTranscriber()

    var microphoneButton: some View {
        Button(action: toggleRecording) {
            Image(systemName: transcriber.isRecording ? "mic.fill" : "mic")
                .font(.title2)
                .foregroundColor(transcriber.isRecording ? .red : .primary)
        }
        .disabled(!transcriber.isAuthorized)
        .onAppear {
            Task { await transcriber.requestAuthorization() }
        }
    }

    private func toggleRecording() {
        if transcriber.isRecording {
            transcriber.stopRecording()
            // Use transcribed text as input
            if !transcriber.transcribedText.isEmpty {
                input = transcriber.transcribedText
                transcriber.transcribedText = ""
            }
        } else {
            try? transcriber.startRecording()
        }
    }
}
```

#### On-device vs server recognition

| Mode | Availability | Duration limit | Quality |
|---|---|---|---|
| On-device | iOS 13+ (select languages) | Unlimited | Good |
| Server | iOS 10+ (all languages) | 60 seconds | Excellent |

**Recommendation:** Use on-device recognition when available for:
- Privacy (audio stays on device)
- Offline capability
- No duration limits
- Lower latency

Check availability with:
```swift
if recognizer?.supportsOnDeviceRecognition == true {
    request.requiresOnDeviceRecognition = true
}
```

#### Continuous dictation with auto-send

```swift
extension AudioTranscriber {
    /// Start continuous dictation that auto-sends after silence
    func startContinuousDictation(
        silenceThreshold: TimeInterval = 2.0,
        onSentence: @escaping (String) -> Void
    ) throws {
        var lastSpeechTime = Date()
        var pendingText = ""

        // ... (similar setup as startRecording)

        recognitionTask = recognizer?.recognitionTask(with: recognitionRequest!) { [weak self] result, error in
            Task { @MainActor in
                guard let result = result else { return }

                let newText = result.bestTranscription.formattedString
                if newText != pendingText {
                    pendingText = newText
                    lastSpeechTime = Date()
                }

                // Check for silence timeout
                if Date().timeIntervalSince(lastSpeechTime) > silenceThreshold {
                    if !pendingText.isEmpty {
                        onSentence(pendingText)
                        pendingText = ""
                        self?.transcribedText = ""
                    }
                }
            }
        }
        // ... rest of setup
    }
}
```

### 10. Multimodal Chat Example

Combine text, image, and voice inputs in a unified chat interface:

#### Updated project structure

```
GemmaChat/
├── Sources/
│   ├── App/
│   │   └── GemmaChatApp.swift
│   ├── Models/
│   │   ├── LLMEngine.swift            # Core LLM wrapper
│   │   ├── ConversationManager.swift
│   │   ├── ModelDownloader.swift
│   │   ├── ImageAnalyzer.swift        # NEW: Vision wrapper
│   │   ├── ImageAnalysisResult.swift  # NEW: Vision results
│   │   └── AudioTranscriber.swift     # NEW: Speech wrapper
│   ├── Views/
│   │   ├── ChatView.swift
│   │   ├── MessageBubble.swift
│   │   ├── MultimodalChatView.swift   # NEW: Combined interface
│   │   ├── ModelPickerView.swift
│   │   └── SettingsView.swift
│   └── Utilities/
│       ├── TokenCounter.swift
│       └── MemoryMonitor.swift
└── Resources/
    └── Assets.xcassets
```

#### MultimodalChatView.swift — Complete example

```swift
import SwiftUI
import PhotosUI

struct MultimodalChatView: View {
    @StateObject private var engine = LLMEngine()
    @StateObject private var downloader = ModelDownloader()
    @StateObject private var imageAnalyzer = ImageAnalyzer()
    @StateObject private var transcriber = AudioTranscriber()

    @State private var messages: [ChatMessage] = []
    @State private var input = ""
    @State private var streamedResponse = ""
    @State private var selectedPhoto: PhotosPickerItem?
    @State private var selectedImage: UIImage?
    @State private var showImagePreview = false

    var body: some View {
        NavigationStack {
            VStack(spacing: 0) {
                // Chat messages
                ScrollViewReader { proxy in
                    ScrollView {
                        LazyVStack(spacing: 12) {
                            ForEach(messages) { message in
                                MessageRow(message: message)
                            }
                            if !streamedResponse.isEmpty {
                                MessageRow(message: ChatMessage(
                                    role: .assistant,
                                    content: streamedResponse
                                ))
                            }
                        }
                        .padding()
                    }
                    .onChange(of: messages.count) { _ in
                        if let lastId = messages.last?.id {
                            proxy.scrollTo(lastId, anchor: .bottom)
                        }
                    }
                }

                // Image preview (if selected)
                if let image = selectedImage {
                    imagePreviewBar(image: image)
                }

                // Input bar
                inputBar
            }
            .navigationTitle("Gemma 4 Chat")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Menu {
                        Button("New Chat") { resetConversation() }
                        Button("Settings") { /* show settings */ }
                    } label: {
                        Image(systemName: "ellipsis.circle")
                    }
                }
            }
        }
        .task {
            await transcriber.requestAuthorization()
        }
    }

    // MARK: - Image preview bar
    private func imagePreviewBar(image: UIImage) -> some View {
        HStack {
            Image(uiImage: image)
                .resizable()
                .scaledToFit()
                .frame(height: 60)
                .cornerRadius(8)

            VStack(alignment: .leading) {
                Text("Image attached")
                    .font(.caption.bold())
                Text("Ask a question about this image")
                    .font(.caption2)
                    .foregroundColor(.secondary)
            }

            Spacer()

            Button(action: { selectedImage = nil }) {
                Image(systemName: "xmark.circle.fill")
                    .foregroundColor(.secondary)
            }
        }
        .padding(.horizontal)
        .padding(.vertical, 8)
        .background(Color(.systemGray6))
    }

    // MARK: - Input bar
    private var inputBar: some View {
        VStack(spacing: 0) {
            Divider()

            HStack(spacing: 12) {
                // Photo picker
                PhotosPicker(selection: $selectedPhoto, matching: .images) {
                    Image(systemName: "photo")
                        .font(.title2)
                }
                .onChange(of: selectedPhoto) { newItem in
                    loadImage(from: newItem)
                }

                // Microphone button
                Button(action: toggleRecording) {
                    Image(systemName: transcriber.isRecording ? "mic.fill" : "mic")
                        .font(.title2)
                        .foregroundColor(transcriber.isRecording ? .red : .primary)
                }
                .disabled(!transcriber.isAuthorized)

                // Text field
                TextField("Message", text: $input, axis: .vertical)
                    .textFieldStyle(.plain)
                    .lineLimit(1...5)
                    .padding(.horizontal, 12)
                    .padding(.vertical, 8)
                    .background(Color(.systemGray6))
                    .cornerRadius(20)

                // Send button
                Button(action: sendMessage) {
                    Image(systemName: "arrow.up.circle.fill")
                        .font(.title)
                }
                .disabled(input.isEmpty && selectedImage == nil || engine.isGenerating)
            }
            .padding(.horizontal)
            .padding(.vertical, 8)

            // Recording indicator
            if transcriber.isRecording {
                HStack {
                    Circle()
                        .fill(Color.red)
                        .frame(width: 8, height: 8)
                    Text("Listening... \(transcriber.transcribedText)")
                        .font(.caption)
                        .lineLimit(1)
                    Spacer()
                }
                .padding(.horizontal)
                .padding(.bottom, 8)
            }
        }
    }

    // MARK: - Actions
    private func loadImage(from item: PhotosPickerItem?) {
        Task {
            if let data = try? await item?.loadTransferable(type: Data.self),
               let image = UIImage(data: data) {
                // Resize if needed (max 2048px for memory efficiency)
                selectedImage = image.resized(maxDimension: 2048)
            }
        }
    }

    private func toggleRecording() {
        if transcriber.isRecording {
            transcriber.stopRecording()
            if !transcriber.transcribedText.isEmpty {
                input = transcriber.transcribedText
                transcriber.transcribedText = ""
            }
        } else {
            try? transcriber.startRecording()
        }
    }

    private func sendMessage() {
        let userText = input
        let userImage = selectedImage
        input = ""
        selectedImage = nil
        selectedPhoto = nil

        // Create user message
        var content = userText
        if userImage != nil {
            content = "[Image attached] " + content
        }
        messages.append(ChatMessage(role: .user, content: content, image: userImage))
        streamedResponse = ""

        Task {
            if let image = userImage {
                // Image + text: Use Vision analysis
                for await chunk in engine.askAboutImage(image, question: userText, analyzer: imageAnalyzer) {
                    streamedResponse += chunk
                }
            } else {
                // Text only
                for await chunk in engine.generate(prompt: userText) {
                    streamedResponse += chunk
                }
            }

            messages.append(ChatMessage(role: .assistant, content: streamedResponse))
            streamedResponse = ""
        }
    }

    private func resetConversation() {
        messages.removeAll()
        engine.resetConversation()
    }
}

// MARK: - Supporting types

struct ChatMessage: Identifiable {
    let id = UUID()
    let role: Role
    let content: String
    var image: UIImage?

    enum Role {
        case user, assistant
    }
}

struct MessageRow: View {
    let message: ChatMessage

    var body: some View {
        HStack {
            if message.role == .user { Spacer() }

            VStack(alignment: message.role == .user ? .trailing : .leading, spacing: 4) {
                if let image = message.image {
                    Image(uiImage: image)
                        .resizable()
                        .scaledToFit()
                        .frame(maxWidth: 200, maxHeight: 200)
                        .cornerRadius(12)
                }

                Text(message.content)
                    .padding(.horizontal, 14)
                    .padding(.vertical, 10)
                    .background(message.role == .user ? Color.blue : Color(.systemGray5))
                    .foregroundColor(message.role == .user ? .white : .primary)
                    .cornerRadius(20)
            }
            .frame(maxWidth: 280, alignment: message.role == .user ? .trailing : .leading)

            if message.role == .assistant { Spacer() }
        }
    }
}

// MARK: - UIImage extension for resizing

extension UIImage {
    func resized(maxDimension: CGFloat) -> UIImage {
        let ratio = min(maxDimension / size.width, maxDimension / size.height)
        if ratio >= 1 { return self }

        let newSize = CGSize(width: size.width * ratio, height: size.height * ratio)
        let renderer = UIGraphicsImageRenderer(size: newSize)
        return renderer.image { _ in
            draw(in: CGRect(origin: .zero, size: newSize))
        }
    }
}
```

#### Memory budget for multimodal

Running all three input modalities requires careful memory management:

| Component | Typical RAM |
|---|---|
| Gemma E2B loaded | ~1.5 GB |
| Gemma E4B loaded | ~2.5 GB |
| Vision analysis (active) | ~200 MB |
| Speech recognition (active) | ~50 MB |
| **Total (E2B + Vision + Speech)** | **~1.75 GB** |
| **Total (E4B + Vision + Speech)** | **~2.75 GB** |

**Memory management tips:**
1. Only run Vision analysis when user sends an image (not continuously)
2. Stop audio engine when not actively recording
3. Release Vision handlers after each analysis completes
4. Monitor with `MemoryMonitor` and show warnings if RAM exceeds threshold

### 11. Performance optimization tips

1. **Use CPU with XNNPACK** — LiteRT-LM uses XNNPACK by default for CPU acceleration with 4 threads.
2. **Metal GPU** — On supported devices, LiteRT-LM can use Metal for GPU acceleration. This is automatic when available.
3. **Neural Engine** — On A12 Bionic+ chips, the Neural Engine can be leveraged for even faster inference.
4. **Prompt caching** — LiteRT-LM supports KV-cache reuse across turns in the same session; don't recreate sessions unnecessarily.
5. **Embedding memory-mapping** — LiteRT-LM memory-maps embedding parameters so only the fraction needed per inference is loaded into RAM.
6. **Thermal management** — Sustained inference heats mobile devices. Monitor thermals and consider throttling generation speed or adding cooldown pauses for long sessions.

### 12. Testing with AI Edge Gallery

For quick prototyping without writing code, users can install the **Google AI Edge Gallery** app on iOS from the App Store. It runs Gemma 4 models locally using the same LiteRT-LM runtime.

### 13. Deployment checklist

- [ ] Xcode 15+ with Swift 5.9+
- [ ] iOS 16+ deployment target
- [ ] LiteRT-LM Swift package added via SPM
- [ ] Model file download + caching logic
- [ ] Memory monitoring for context window management
- [ ] Background download support (for large model files)
- [ ] App Transport Security exception for huggingface.co (if downloading in-app)
- [ ] Info.plist: `UIBackgroundModes` → `processing` (for model download)
- [ ] Test on physical device (Simulator does not have Neural Engine)

---

## Common issues

| Issue | Fix |
|---|---|
| App crashes on launch after loading model | Device has insufficient RAM. Switch to E2B or reduce context length. |
| Very slow first token | First run initializes caches. Subsequent runs are faster. |
| Model download fails | Check ATS settings; ensure HTTPS to huggingface.co is allowed. |
| Garbled output | Ensure you're using the `-it` (instruction-tuned) variant, not the base model. |
| High memory warning | Use `MemoryMonitor.shouldReduceContext()` to trim history. |
                                                                                                                                                                                                                             gemma4-ios-litert/references/                                                                       0000755 0000000 0000000 00000000000 15166140413 015141  5                                                                                                    ustar   root                            root                                                                                                                                                                                                                   gemma4-ios-litert/references/ios-project-guide.md                                                   0000644 0000000 0000000 00000011124 15166140413 021013  0                                                                                                    ustar   root                            root                                                                                                                                                                                                                   # iOS Project Guide — Gemma 4 + LiteRT-LM

## Full Xcode Project Setup

### Prerequisites

- macOS 14+ (Sonoma or later)
- Xcode 15.2+
- Physical iOS device (A14 Bionic or later for E2B; A15+ for E4B)
- ~3-5 GB free storage on device for model file

### Step 1: Create Xcode project

1. Open Xcode → File → New → Project → iOS → App
2. Product Name: `GemmaChat`
3. Interface: SwiftUI
4. Language: Swift
5. Minimum Deployment: iOS 16.0

### Step 2: Add LiteRT-LM via Swift Package Manager

1. File → Add Package Dependencies
2. Enter URL: `https://github.com/google-ai-edge/LiteRT-LM`
3. Set version rule: "Up to Next Major" from `1.0.0`
4. Add `LiteRTLM` library to your target

### Step 3: Info.plist entries

```xml
<!-- Allow Hugging Face downloads -->
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSExceptionDomains</key>
    <dict>
        <key>huggingface.co</key>
        <dict>
            <key>NSIncludesSubdomains</key>
            <true/>
            <key>NSExceptionAllowsInsecureHTTPLoads</key>
            <false/>
        </dict>
    </dict>
</dict>

<!-- Background download -->
<key>UIBackgroundModes</key>
<array>
    <string>processing</string>
</array>
```

### Step 4: Model file strategy

**Option A — Download on first launch (recommended)**
- Use `ModelDownloader` from SKILL.md
- Show progress UI during download
- Cache in Application Support directory
- Survives app updates, cleared on app deletion

**Option B — Sideload via Files app**
- User downloads `.litertlm` from HF manually
- App uses UIDocumentPickerViewController to import
- Good for enterprise / MDM deployments

**Option C — On-Demand Resources (ODR)**
- Tag model as on-demand resource in Xcode
- Apple hosts it; downloaded when needed
- ⚠️ 4 GB ODR limit may be tight for E4B

### Step 5: Conversation history format

Gemma 4 uses standard role-based chat format:

```
<start_of_turn>user
Hello, how are you?<end_of_turn>
<start_of_turn>model
I'm doing well! How can I help you today?<end_of_turn>
<start_of_turn>user
What's the weather like?<end_of_turn>
<start_of_turn>model
```

When building multi-turn prompts, concatenate all previous turns using this template. LiteRT-LM's session API handles this automatically if you use `session.sendMessage()` or `session.generateResponse()`.

### Step 6: Context window management

The model supports up to 32k tokens, but practical limits depend on device RAM:

| Device RAM | Recommended max context |
|---|---|
| 4 GB (iPhone 12) | 2048 tokens |
| 6 GB (iPhone 14 Pro) | 8192 tokens |
| 8 GB (iPhone 15 Pro) | 16384 tokens |
| 16 GB (iPad M4) | 32768 tokens |

Implement a sliding window that drops oldest messages when approaching the limit. Always preserve the system prompt.

### Step 7: Async/await streaming pattern

```swift
// In your ViewModel
func chat(_ userMessage: String) async {
    messages.append(Message(role: .user, content: userMessage))
    
    var assistantText = ""
    messages.append(Message(role: .assistant, content: ""))
    
    for await token in engine.generate(prompt: buildPrompt()) {
        assistantText += token
        // Update last message in-place for streaming UI
        messages[messages.count - 1].content = assistantText
    }
}
```

### Step 8: Offline capability

Once the model file is downloaded, the app works fully offline:
- No network calls during inference
- No API keys or authentication
- No telemetry or data leaving the device
- Works in airplane mode

This is ideal for:
- Healthcare apps (HIPAA compliance — data never leaves device)
- Field work with no connectivity
- Privacy-sensitive personal assistants
- Regions with unreliable internet

### Step 9: App Store considerations

- **Binary size:** The model file is NOT in the app bundle, so the IPA stays small (~10 MB).
- **Review guidelines:** Apple allows on-device ML models. No special review needed.
- **Background downloads:** Use `BGProcessingTask` for large model downloads to avoid iOS killing the download.
- **Storage warnings:** Alert users about the 2.5-4.5 GB model file size before download.

### Step 10: Alternative iOS paths

If LiteRT-LM Swift APIs are not yet stable for your use case, consider:

1. **swift-llama.cpp** — Use GGUF-quantized Gemma 4 via llama.cpp's Swift bindings. More mature Swift API but requires GGUF conversion.
2. **MLX Swift** — Apple's own ML framework with native Swift support. Requires MLX-format model conversion.
3. **LM Studio iOS** — For prototyping only (no programmatic integration). Users can test Gemma 4 GGUF models directly.

Each path trades off integration effort vs. runtime maturity. LiteRT-LM is Google's recommended production path.
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            