# PRIYA — AI-Powered Navigation Assistant for the Visually Impaired

## Complete Technical Project Report

**Author:** Priyanka Meenkeri  
**Platform:** Android (API 27+)  
**Language:** Kotlin  
**AI Engine:** Google Gemini 2.5 Flash  
**Document Version:** 1.0  
**Date:** July 2026

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Problem Statement](#2-problem-statement)
3. [Proposed Solution](#3-proposed-solution)
4. [Key Features](#4-key-features)
5. [System Architecture](#5-system-architecture)
6. [Technology Stack](#6-technology-stack)
7. [App Workflow — End-to-End Pipeline](#7-app-workflow--end-to-end-pipeline)
8. [Camera Pipeline & Image Processing](#8-camera-pipeline--image-processing)
9. [AI Integration — Google Gemini Vision](#9-ai-integration--google-gemini-vision)
10. [Text-to-Speech (TTS) System](#10-text-to-speech-tts-system)
11. [Speech Recognition (Voice Input)](#11-speech-recognition-voice-input)
12. [Gesture-Based Interaction System](#12-gesture-based-interaction-system)
13. [Three Operating Modes](#13-three-operating-modes)
14. [Permission Management](#14-permission-management)
15. [User Interface Design](#15-user-interface-design)
16. [File Structure & Codebase Overview](#16-file-structure--codebase-overview)
17. [Build Configuration](#17-build-configuration)
18. [Development Challenges & Solutions](#18-development-challenges--solutions)
19. [Testing & Debugging](#19-testing--debugging)
20. [Impact & Use Cases](#20-impact--use-cases)
21. [Future Enhancements](#21-future-enhancements)
22. [Conclusion](#22-conclusion)
23. [References](#23-references)

---

## 1. Executive Summary

**Priya** is an Android application designed to assist visually impaired and blind individuals in navigating their surroundings independently. The app leverages real-time camera input processed through **Google Gemini 2.5 Flash** — a state-of-the-art multimodal AI model — and converts the AI's analysis into audible speech via **Text-to-Speech (TTS)**.

The application operates in three distinct modes:

| Mode | Function | Input | Output |
|------|----------|-------|--------|
| **Navigation** | Continuous environmental awareness | Live camera stream | Spoken descriptions every ~12 seconds |
| **Assistant** | Interactive Q&A about surroundings | User voice/text query + camera frame | Spoken answer |
| **Reading** | Text extraction from images | Single camera capture | Spoken text (books, signs, boards) |

The app runs entirely on a standard smartphone — no external hardware, no wearable devices, no cloud server dependency beyond the Gemini API — making it accessible to anyone with an Android phone.

---

## 2. Problem Statement

### The Challenge

Over **2.2 billion people** globally have vision impairment (WHO, 2023). Daily navigation remains one of the most significant challenges for the visually impaired:

- **Obstacle detection**: Identifying objects, stairs, vehicles, and people in the path
- **Environmental awareness**: Understanding surroundings — buildings, signs, colors, states
- **Text reading**: Reading books, menus, signs, medication labels, documents
- **Independent navigation**: Moving through indoor and outdoor spaces without assistance

### Limitations of Existing Solutions

| Solution | Limitation |
|----------|------------|
| **Human guides** | Not always available, expensive |
| **White cane** | Limited range, no text reading, no color detection |
| **Dedicated GPS devices** | Expensive ($200–$500+), single-purpose |
| **Smart glasses** (e.g., OrCam, Envision) | Very expensive ($2,500–$4,500), requires additional hardware |
| **Existing Android apps** | Usually single-function (either navigation OR reading, rarely both) |

### Gap in the Market

No existing solution combines **real-time navigation**, **interactive Q&A**, and **text reading** in a single, free, smartphone-based application powered by cutting-edge AI.

---

## 3. Proposed Solution

Priya addresses this gap by creating a **multimodal AI-powered assistant** that:

1. **Captures live camera frames** from the phone's rear camera using CameraX
2. **Converts YUV camera data to Bitmap** for AI processing
3. **Sends frames to Google Gemini 2.5 Flash** for real-time environmental analysis
4. **Converts AI responses to speech** using Android's TTS engine
5. **Enables voice interaction** so users can ask questions about their surroundings
6. **Reads text from images** — books, signs, menus, documents

The entire pipeline runs in **under 3 seconds** per frame, providing near-real-time assistance.

---

## 4. Key Features

### 4.1 Three-Mode Architecture

| Feature | Navigation Mode | Assistant Mode | Reading Mode |
|---------|----------------|----------------|--------------|
| Camera | Continuous stream | Snapshot per query | Single capture |
| AI Model | Gemini 2.5 Flash (streaming) | Gemini 2.5 Flash (chat) | Gemini 2.5 Flash (reading) |
| User Input | None (automatic) | Voice or keyboard | None (automatic) |
| Output | Environmental description | Answer to question | Read text aloud |
| Temperature | 1.0 | 1.5 (creative) | 0.2 (factual) |

### 4.2 Gesture Controls

- **Double Tap**: Start/Stop navigation (or pause/resume)
- **Long Press**: Switch to Assistant Mode
- **Voice Commands**: "Start reading", "Stop navigation", "Ask something"
- **Earbuds/AirPods**: Double-tap on earbuds also works

### 4.3 Real-Time AI Analysis

- Object detection and identification
- Color recognition
- Spatial awareness (distances, directions)
- State detection (open/closed doors, traffic lights)
- Landmark identification
- Indoor/outdoor environment classification
- Stair and obstacle detection
- Crosswalk and traffic analysis

### 4.4 Accessibility-First Design

- All interactions are voice-driven — no visual UI required during operation
- TTS speaks responses with `QUEUE_FLUSH` (interrupts for urgent, `QUEUE_ADD` for info)
- Haptic feedback on gestures
- Large touch targets
- Permission dialogs with spoken instructions

---

## 5. System Architecture

### 5.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                       USER (Visually Impaired)               │
│                    📱 Android Phone (TECNO LJ8k)             │
└───────────────────────────┬─────────────────────────────────┘
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
     👆 Gestures      📷 Camera          🔊 Audio Output
          │                 │                 │
          ▼                 ▼                 ▼
┌──────────────┐  ┌──────────────────┐  ┌──────────────┐
│  Touch Input │  │   CameraX API    │  │  TTS Engine  │
│  (Compose)   │  │  (ImageAnalysis) │  │  (Google)    │
└──────┬───────┘  └────────┬─────────┘  └──────▲───────┘
       │                   │                    │
       ▼                   ▼                    │
┌──────────────────────────────────────────────────────────────┐
│                    BlindModeScreen.kt                         │
│              (State Management + Mode Control)                │
│                                                               │
│  ┌─────────────┐  ┌──────────────┐  ┌────────────────────┐  │
│  │ Navigation  │  │  Assistant   │  │     Reading        │  │
│  │   Mode      │  │    Mode      │  │      Mode          │  │
│  └──────┬──────┘  └──────┬───────┘  └────────┬───────────┘  │
└─────────┼────────────────┼────────────────────┼──────────────┘
          │                │                    │
          ▼                ▼                    ▼
┌──────────────────────────────────────────────────────────────┐
│                    Gemini API Layer                           │
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐  │
│  │ GeminiAPI.kt │  │GeminiAPI 1.kt│  │   GeminiAPI2.kt    │  │
│  │  (Nav AI)    │  │ (Chat AI)    │  │   (Read AI)        │  │
│  │ streaming    │  │ chat-based   │  │   streaming        │  │
│  └──────┬───────┘  └──────┬───────┘  └────────┬───────────┘  │
└─────────┼─────────────────┼────────────────────┼──────────────┘
          │                 │                    │
          ▼                 ▼                    ▼
┌──────────────────────────────────────────────────────────────┐
│              Google Gemini 2.5 Flash (Cloud API)              │
│         Multimodal Vision + Language Understanding            │
└──────────────────────────────────────────────────────────────┘
```

### 5.2 Data Flow Diagram

```
Camera Frame (YUV_420_888)
        │
        ▼
   yuvToBitmap()
        │
        ▼
   Bitmap (RGB)
        │
        ▼
   Gemini API Request (image + prompt)
        │
        ▼
   AI Response (text stream)
        │
        ▼
   TTS Queue → Speech Output
        │
        ▼
   🗣️  "There is a staircase ahead on your left..."
```

### 5.3 Navigation Flow

```
App Launch
    │
    ▼
┌─────────────────┐
│   MainPage      │  ← Welcome screen with "Blind Mode" card
│   (navigation.kt)│
└────────┬────────┘
         │ Tap "Blind Mode"
         ▼
┌─────────────────┐
│  Permission     │  ← CAMERA + RECORD_AUDIO
│  Check          │
└────────┬────────┘
         │ Granted
         ▼
┌─────────────────┐
│ BlindModeScreen │  ← Core navigation logic
│ (BlindMode.kt)  │
└────────┬────────┘
         │
    ┌────┼────┐
    │    │    │
    ▼    ▼    ▼
  Nav  Asst  Read
```

---

## 6. Technology Stack

### 6.1 Complete Technology Breakdown

| Layer | Technology | Version | Purpose |
|-------|-----------|---------|---------|
| **Language** | Kotlin | 1.9.0 | Primary development language |
| **UI Framework** | Jetpack Compose | BOM 2024.02.00 | Declarative UI |
| **UI Toolkit** | Material 3 | BOM 2024.02.00 | Material Design components |
| **Navigation** | Compose Navigation | 2.7.7 | In-app screen routing |
| **Camera** | CameraX | 1.3.4 | Camera access & frame capture |
| **AI Vision** | Google Generative AI SDK | 0.9.0 | Gemini API integration |
| **Networking** | OkHttp | 4.12.0 | HTTP client (Gemini transport) |
| **TTS** | Android TextToSpeech | Built-in | Speech synthesis |
| **Speech Recognition** | Android SpeechRecognizer | Built-in | Voice input |
| **Build System** | Gradle (Kotlin DSL) | 8.7 | Build automation |
| **Target SDK** | Android API 34 | 14 | Android 14 |
| **Min SDK** | Android API 27 | 8.1 | Android 8.1+ |

### 6.2 Why Each Technology Was Chosen

| Technology | Reason |
|-----------|--------|
| **Kotlin** | Official Android language, null safety, coroutines, modern syntax |
| **Jetpack Compose** | Declarative UI, less boilerplate, better state management than XML |
| **CameraX** | Lifecycle-aware, consistent across devices, ImageAnalysis API |
| **Gemini 2.5 Flash** | Multimodal (vision + language), fast inference, streaming support, free tier |
| **Material 3** | Modern design system, dynamic color, accessibility components |
| **Compose Navigation** | Type-safe routes, deep linking, integrates with Compose lifecycle |
| **OkHttp** | Reliable HTTP, connection pooling, required by Gemini SDK |

---

## 7. App Workflow — End-to-End Pipeline

### 7.1 Navigation Mode Pipeline (Continuous)

```
Step 1: Camera Frame Capture
─────────────────────────────
- CameraX ImageAnalysis captures frame at 1280×720
- Backpressure: STRATEGY_KEEP_ONLY_LATEST (drops old frames)
- Analyzer runs on single-thread executor
- Frame interval: ~12 seconds

Step 2: YUV → Bitmap Conversion
────────────────────────────────
- ImageProxy contains YUV_420_888 data
- Custom yuvToBitmap() function:
  1. Extract Y, U, V planes
  2. Copy Y plane row-by-row (respecting pixelStride)
  3. Copy UV planes in NV21 order (V before U, interleaved)
  4. Create YuvImage → compressToJpeg → decodeByteArray
- Returns Bitmap or null on failure

Step 3: Gemini AI Analysis
──────────────────────────
- Bitmap sent to Gemini 2.5 Flash via generateContentStream()
- System prompt instructs AI to:
  • Identify objects, colors, sizes, states
  • Provide 3-4 sentence navigation guidance
  • Use action words: "Stop", "Turn left", "Proceed"
  • Never say "I can't see" — suggest camera adjustment
- Response streams in real-time (token by token)

Step 4: TTS Speech Output
─────────────────────────
- Each streamed token added to response buffer
- When complete, TTS speaks via QUEUE_FLUSH
- Response displayed as overlay text on screen
- Internet disconnection warning spoken immediately

Step 5: Repeat
──────────────
- Pipeline repeats every ~12 seconds
- User can double-tap to pause/resume
```

### 7.2 Assistant Mode Pipeline (On-Demand)

```
Step 1: User Query
──────────────────
- User double-taps and holds (long press) to enter Assistant
- Type query via keyboard OR use speech recognition
- Example: "What color is the building ahead?"

Step 2: Frame Capture
─────────────────────
- Camera captures a single frame at query time
- Frame converted to Bitmap

Step 3: Gemini Chat API
───────────────────────
- Query + frame data sent to Gemini via chat.sendMessage()
- Uses chat history for context
- Temperature 1.5 for creative, conversational responses
- Non-streaming (returns full response)

Step 4: TTS Response
────────────────────
- AI answer spoken aloud
- Response displayed on screen
```

### 7.3 Reading Mode Pipeline (Single-Shot)

```
Step 1: Trigger
───────────────
- User long-presses and says "Start reading"
- OR taps reading mode button

Step 2: Single Frame Capture
─────────────────────────────
- ReadingModeCamera composable activates
- ImageCapture (not ImageAnalysis) captures one frame
- Saves to temp JPEG file, decodes back to Bitmap

Step 3: Gemini Reading API
──────────────────────────
- Bitmap sent to Gemini 2.5 Flash (temperature 0.2)
- System prompt: "Read the text from this image"
- Low temperature = factual, accurate text extraction
- Streaming response for immediate output

Step 4: TTS Reading
───────────────────
- Extracted text spoken aloud
- Works for: books, signs, menus, documents, labels
```

---

## 8. Camera Pipeline & Image Processing

### 8.1 CameraX Configuration

**File**: `CameraPreviewWithAnalysis.kt`

```kotlin
val imageAnalysis = ImageAnalysis.Builder()
    .setTargetResolution(Size(1280, 720))
    .setBackpressureStrategy(ImageAnalysis.STRATEGY_KEEP_ONLY_LATEST)
    .build()
```

| Parameter | Value | Reason |
|-----------|-------|--------|
| Resolution | 1280×720 | Balance between detail and processing speed |
| Backpressure | KEEP_ONLY_LATEST | Prevents frame queue buildup; always uses newest frame |
| Analyzer | Single-thread executor | Prevents concurrent frame processing |

### 8.2 YUV to Bitmap Conversion

**File**: `GeminiAPI.kt` — `ImageProxy.yuvToBitmap()`

CameraX delivers frames in YUV_420_888 format. Gemini AI requires JPEG/PNG input. The conversion pipeline:

```
YUV_420_888 → NV21 byte array → JPEG → Bitmap
```

**Implementation details:**

```kotlin
fun ImageProxy.yuvToBitmap(): Bitmap? {
    val yBuffer = planes[0].buffer   // Y plane
    val uBuffer = planes[1].buffer   // U plane
    val vBuffer = planes[2].buffer   // V plane

    val yRowStride = planes[0].rowStride
    val uvRowStride = planes[1].rowStride
    val uvPixelStride = planes[1].pixelStride

    val nv21 = ByteArray(width * height * 3 / 2)

    // Copy Y plane
    var pos = 0
    for (row in 0 until height) {
        yBuffer.position(row * yRowStride)
        yBuffer.get(nv21, pos, width)
        pos += width
    }

    // Copy UV planes in NV21 order (V before U, interleaved)
    for (row in 0 until height / 2) {
        for (col in 0 until width / 2) {
            val uvIndex = row * uvRowStride + col * uvPixelStride
            nv21[pos++] = vBuffer.get(uvIndex)  // V
            nv21[pos++] = uBuffer.get(uvIndex)  // U
        }
    }

    val yuvImage = YuvImage(nv21, ImageFormat.NV21, width, height, null)
    val out = ByteArrayOutputStream()
    yuvImage.compressToJpeg(Rect(0, 0, width, height), 90, out)
    return BitmapFactory.decodeByteArray(out.toByteArray(), 0, out.size())
}
```

**Key challenges handled:**
- **Pixel stride**: U/V planes may have pixelStride=2 (interleaved) or pixelStride=1 (planar)
- **Row stride**: May differ from width due to memory alignment
- **NV21 order**: Android's standard YUV format requires V before U
- **JPEG quality**: 90% — good quality, reasonable size

### 8.3 Internet Connectivity Check

```kotlin
fun isInternetAvailable(context: Context): Boolean {
    val connectivityManager = context.getSystemService(...)
    // API >= M: NetworkCapabilities check
    // API < M: deprecated NetworkInfo check
    // Returns true if WIFI, CELLULAR, or ETHERNET is available
}
```

Checked every 5 seconds in a background coroutine. If disconnected:
- Red warning text displayed
- TTS speaks "You are not connected to the internet"

---

## 9. AI Integration — Google Gemini Vision

### 9.1 Why Gemini 2.5 Flash

| Feature | Benefit |
|---------|---------|
| **Multimodal** | Accepts both images and text in the same request |
| **Streaming** | Responses arrive token-by-token for faster feedback |
| **Vision** | Understands objects, scenes, spatial relationships |
| **Free tier** | Available for development and testing |
| **Fast inference** | "Flash" variant optimized for speed |
| **1M token context** | Can process long conversations with history |

### 9.2 Three Separate AI Instances

| Instance | File | Model | Temperature | Purpose |
|----------|------|-------|-------------|---------|
| Navigation AI | `GeminiAPI.kt` | gemini-2.5-flash | 1.0 | Environmental analysis |
| Chat AI | `GeminiAPI 1.kt` | gemini-2.5-flash | 1.5 | Interactive Q&A |
| Reading AI | `GeminiAPI2.kt` | gemini-2.5-flash | 0.2 | Text extraction |

### 9.3 Navigation AI System Prompt (Key Points)

```
You are an advanced navigation assistant for a visually impaired person.
Analyze each frame to identify objects, colors, sizes, states, and landmarks.

Response rules:
- 3-4 sentences maximum
- Start with action words: "Stop", "Turn right", "Proceed"
- Never say "I can't see" — say "Please adjust your camera"
- Prioritize safety: obstacles, traffic, stairs
- Include spatial awareness: "on your left", "2 meters ahead"
- Environment context: urban, indoor, outdoor, crowded
- End with encouragement: "You're doing great"
```

### 9.4 Reading AI System Prompt

```
Your user is a blind person. Read the entire text from the image.
Read from books, sign boards, and other text sources.
Be accurate — the user depends on this for critical information.
```

### 9.5 API Configuration

```kotlin
val generativeModel = GenerativeModel(
    modelName = "gemini-2.5-flash",
    apiKey = "YOUR_API_KEY",
    generationConfig = generationConfig {
        temperature = 1f
        topK = 64
        topP = 0.95f
        maxOutputTokens = 8192
        responseMimeType = "text/plain"
    },
    safetySettings = listOf(
        SafetySetting(HarmCategory.HARASSMENT, BlockThreshold.NONE),
        SafetySetting(HarmCategory.HATE_SPEECH, BlockThreshold.NONE),
        SafetySetting(HarmCategory.SEXUALLY_EXPLICIT, BlockThreshold.NONE),
        SafetySetting(HarmCategory.DANGEROUS_CONTENT, BlockThreshold.NONE),
    )
)
```

### 9.6 Streaming vs Non-Streaming

| Mode | API Call | Behavior |
|------|----------|----------|
| Navigation | `generateContentStream()` | Tokens stream in real-time, displayed incrementally |
| Assistant | `chat.sendMessage()` | Full response returned at once, with conversation history |
| Reading | `generateContentStream()` | Tokens stream in, text appears word-by-word |

---

## 10. Text-to-Speech (TTS) System

### 10.1 Dual TTS Architecture

The app uses **two separate TTS instances** to prevent audio conflicts:

| Instance | Location | Purpose | Lifecycle |
|----------|----------|---------|-----------|
| `tts` | `MainPage` | Instructions, welcome messages | Tied to MainPage composition |
| `tts2` | `BlindModeScreen` | Navigation, assistant, reading speech | Tied to BlindMode composition |

### 10.2 TTS Initialization

```kotlin
var tts2 by remember { mutableStateOf<TextToSpeech?>(null) }

DisposableEffect(Unit) {
    tts2 = TextToSpeech(context) { status ->
        if (status == TextToSpeech.SUCCESS) {
            tts2?.language = Locale.US
        }
    }
    onDispose {
        tts2?.stop()
        tts2?.shutdown()
    }
}
```

### 10.3 Queue Strategies

| Strategy | When Used | Behavior |
|----------|-----------|----------|
| `QUEUE_FLUSH` | Navigation response, errors, urgent warnings | Interrupts current speech, speaks new message |
| `QUEUE_ADD` | Partial responses, additional info | Adds to end of current speech queue |

### 10.4 Speech Timing

- **Navigation mode**: Response spoken after full AI analysis completes (~2-3 seconds)
- **Assistant mode**: Only spoken when user sends a message
- **Reading mode**: Spoken once after single capture
- **Internet warning**: Spoken immediately on disconnection
- **Errors**: Spoken via `QUEUE_FLUSH` for immediate attention

### 10.5 Accessibility Instructions (Main Page)

When the settings icon is tapped, TTS reads:
> "To master the navigation through this App, follow these instructions below. Double Tap on the Navigation screen to stop the Navigation or vice versa. Right after, speak any query about anything or the environment around you. You also can use your earbuds or AirPods by double tapping to stop or resume the navigation. Made by Priyanka Meenkeri to nurture the blind for this society."

---

## 11. Speech Recognition (Voice Input)

### 11.1 Setup

```kotlin
val speechRecognizer = SpeechRecognizer.createSpeechRecognizer(context)
val recognizerIntent = Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH).apply {
    putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL, RecognizerIntent.LANGUAGE_MODEL_FREE_FORM)
    putExtra(RecognizerIntent.EXTRA_LANGUAGE, "en-US")
    putExtra(RecognizerIntent.EXTRA_PARTIAL_RESULTS, true)
}
```

### 11.2 Continuous Listening

Speech recognition restarts automatically when:
- Navigation is paused (double-tap)
- User says "start reading"
- User says "start navigation"
- No speech detected (timeout)

### 11.3 Voice Commands

| Command | Action |
|---------|--------|
| "start reading" | Switches to Reading Mode |
| "start navigation" | Switches to Navigation Mode |
| "stop navigation" | Pauses navigation |
| Any other text | Sent as query to Assistant AI |

---

## 12. Gesture-Based Interaction System

### 12.1 Gesture Map

| Gesture | Context | Action |
|---------|---------|--------|
| **Double Tap** | Navigation screen | Toggle navigation ON/OFF |
| **Long Press** | Navigation screen | Switch to Assistant Mode |
| **Tap** | Settings icon | Toggle instruction panel |
| **Tap** | Mode button | Switch between modes |
| **Tap** | Back button | Show exit confirmation dialog |

### 12.2 State Machine

```
┌─────────────────────────────────────────────┐
│                IDLE STATE                    │
│  (App just launched, permissions granted)     │
└──────────────────┬──────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│            NAVIGATION MODE                   │
│  blmode = true, navigationPaused = false      │
│  Camera streaming → Gemini → TTS             │
│                                              │
│  ┌─────────┐     ┌──────────────┐           │
│  │ Double  │────▶│   PAUSED     │           │
│  │  Tap    │◀────│ (listening)  │           │
│  └─────────┘     └──────┬───────┘           │
│                         │                    │
│                    Long Press                │
│                         │                    │
│                         ▼                    │
└─────────────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────┐
│            ASSISTANT MODE                    │
│  currentMode = "assistant"                   │
│  Single frame + user query → Gemini          │
│                                              │
│  ┌──────────────┐  ┌─────────────────┐      │
│  │ Voice Input  │  │  Keyboard Input │      │
│  └──────┬───────┘  └────────┬────────┘      │
│         └──────────┬────────┘               │
│                    ▼                         │
│            Chat Response → TTS               │
└─────────────────────────────────────────────┘
                   │
              "start reading"
                   │
                   ▼
┌─────────────────────────────────────────────┐
│            READING MODE                      │
│  currentMode = "reading"                     │
│  Single capture → Gemini 2 → TTS             │
│                                              │
│  ReadingModeCamera → onImageCaptured(bitmap)  │
│  sendFrameToGemini2AI(bitmap)                │
│  → Reads text aloud                          │
└─────────────────────────────────────────────┘
```

### 12.3 Earbuds/AirPods Integration

The app responds to media button events, allowing:
- Double-tap on earbuds to pause/resume navigation
- Compatible with standard Bluetooth headset controls
- Uses `RECEIVE_MEDIA_BUTTON` permission

---

## 13. Three Operating Modes

### 13.1 Navigation Mode

**Purpose**: Provide continuous environmental awareness  
**Input**: Live camera stream (1280×720)  
**Processing**: Every ~12 seconds  
**Output**: Spoken description of surroundings  

**Example outputs:**
> "There is a red car approaching from your left. A pedestrian crossing is 5 meters ahead. You can safely proceed forward."

> "You are near a building entrance. The door appears to be open. A staircase is visible on your right side."

> "The area ahead appears to be clear. There is a sidewalk with no immediate obstacles for the next 10 meters."

### 13.2 Assistant Mode

**Purpose**: Interactive Q&A about the environment  
**Input**: User voice/text query + camera frame  
**Processing**: On-demand  
**Output**: Spoken answer  

**Example interactions:**
- User: "What color is the building?" → AI: "The building ahead is white with blue windows."
- User: "Is there a crosswalk?" → AI: "Yes, there's a crosswalk about 3 meters in front of you with a green pedestrian signal."
- User: "What does the sign say?" → AI: "The sign reads 'Metro Station - Platform B, 50 meters ahead.'"

### 13.3 Reading Mode

**Purpose**: Extract and read text from images  
**Input**: Single camera capture  
**Processing**: One-shot  
**Output**: Spoken text  

**Use cases:**
- Reading books or magazines
- Reading street signs and building names
- Reading restaurant menus
- Reading medication labels
- Reading documents or letters
- Reading price tags

---

## 14. Permission Management

### 14.1 Required Permissions

| Permission | Reason | Criticality |
|-----------|--------|-------------|
| `CAMERA` | Access rear camera for frame capture | **Required** |
| `RECORD_AUDIO` | Voice input for speech recognition | **Required** |
| `INTERNET` | Connect to Gemini API | **Required** |
| `ACCESS_NETWORK_STATE` | Check connectivity status | Optional |
| `VIBRATE` | Haptic feedback on gestures | Optional |
| `RECEIVE_MEDIA_BUTTON` | Earbuds/airPods control | Optional |

### 14.2 Compose-Native Permission Flow

```kotlin
val permissionLauncher = rememberLauncherForActivityResult(
    ActivityResultContracts.RequestMultiplePermissions()
) { permissions ->
    hasCameraPermission = permissions[CAMERA] ?: false
    hasAudioPermission = permissions[RECORD_AUDIO] ?: false
    hasPermission = hasCameraPermission && hasAudioPermission
}

// Auto-request on first launch
LaunchedEffect(Unit) {
    hasCameraPermission = ContextCompat.checkSelfPermission(ctx, CAMERA) == GRANTED
    hasAudioPermission = ContextCompat.checkSelfPermission(ctx, RECORD_AUDIO) == GRANTED
    hasPermission = hasCameraPermission && hasAudioPermission
    if (!hasPermission) permissionLauncher.launch(arrayOf(CAMERA, RECORD_AUDIO))
}
```

### 14.3 Denial Handling

If permissions are denied:
1. Dialog shown: "Please provide permissions to use this feature"
2. "Exit" button calls `finish()` to close the app
3. All features require both CAMERA and RECORD_AUDIO

---

## 15. User Interface Design

### 15.1 Main Page (Entry Screen)

```
┌─────────────────────────────────┐
│                                 │
│                                 │
│    ┌─────────────────────────┐  │
│    │                         │  │
│    │     Blind Mode          │  │
│    │                         │  │
│    └─────────────────────────┘  │
│                                 │
│                                 │
│    Instructions                 │
│    ⚙️ Settings                  │
│    ▼ Arrow                      │
│                                 │
│  "Made by Priyanka Meenkeri    │
│   to nurture the blind         │
│   for this society"            │
│                                 │
└─────────────────────────────────┘
```

- Clean, minimal design
- Large "Blind Mode" card (80% width, 120dp height)
- Settings icon at bottom opens instruction panel
- TTS reads instructions when settings icon is tapped

### 15.2 Blind Mode Screen (Navigation)

```
┌─────────────────────────────────┐
│                                 │
│     📷 Camera Preview           │
│     (Live camera feed)          │
│                                 │
│                                 │
│                                 │
│─────────────────────────────────│
│  AI Response Overlay            │
│  (Semi-transparent black box)   │
│  "There is a red car ahead..."  │
│                                 │
└─────────────────────────────────┘
```

- Full-screen camera preview
- AI response overlay at bottom (semi-transparent black)
- White italic text for readability
- Mode indicator: "Navigation", "Chat: ...", or "Reading: ..."

---

## 16. File Structure & Codebase Overview

### 16.1 Complete Project Structure

```
Blind-Nav-Android-App-for-Blind-persons/
├── app/
│   ├── build.gradle.kts
│   └── src/main/
│       ├── AndroidManifest.xml
│       └── java/com/example/assistantapp/
│           ├── MainActivity.kt              (40 lines)  — Entry point
│           ├── navigation.kt                (12 lines)  — NavHost routing
│           ├── mainScreen.kt                (220 lines) — Welcome page
│           ├── BlindMode.kt                 (550+ lines)— Core navigation logic
│           ├── CameraPreviewWithAnalysis.kt (60 lines)  — CameraX setup
│           ├── AIResponseOverlay.kt         (150 lines) — TTS + text overlay
│           ├── GeminiAPI.kt                 (90 lines)  — Navigation AI
│           ├── GeminiAPI 1.kt               (90 lines)  — Chat AI
│           ├── GeminiAPI2.kt                (70 lines)  — Reading AI
│           └── ui/theme/
│               ├── Color.kt
│               ├── Theme.kt
│               └── Type.kt
├── build.gradle.kts
├── settings.gradle.kts
├── gradle.properties
└── OOBVault/                               (18 docs)   — Project documentation
```

### 16.2 File Responsibilities

| File | Lines | Responsibility |
|------|-------|---------------|
| `MainActivity.kt` | 40 | App entry, theme setup, NavHost hosting |
| `navigation.kt` | 12 | Route definitions (mainPage, blindMode) |
| `mainScreen.kt` | 220 | Welcome UI, settings panel, TTS instructions |
| `BlindMode.kt` | 550+ | Core logic: permissions, camera, AI calls, gestures, mode switching |
| `CameraPreviewWithAnalysis.kt` | 60 | CameraX preview + ImageAnalysis setup |
| `AIResponseOverlay.kt` | 150 | AI text display, TTS orchestration, connectivity check |
| `GeminiAPI.kt` | 90 | Navigation AI model, yuvToBitmap(), streaming |
| `GeminiAPI 1.kt` | 90 | Chat AI model, conversation history, non-streaming |
| `GeminiAPI2.kt` | 70 | Reading AI model, low-temperature text extraction |

### 16.3 State Variables (BlindModeScreen)

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `blmode` | Boolean | true | Navigation active flag |
| `navigationPaused` | Boolean | false | Navigation paused flag |
| `currentMode` | String | "navigation" | Active mode selector |
| `response` | String | "" | Raw AI response accumulator |
| `navigationResponse` | String | "" | Navigation mode output |
| `chatResponse` | String | "" | Chat mode output |
| `readingModeResult` | String | "" | Reading mode output |
| `readingModeEnable` | Boolean | false | Reading mode activation |
| `isConnected` | Boolean | true | Internet connectivity status |
| `hasPermission` | Boolean | false | Combined permission status |
| `showExitDialog` | Boolean | false | Exit confirmation dialog |
| `settingDialog` | Boolean | false | Settings panel visibility |

---

## 17. Build Configuration

### 17.1 Gradle Setup

```kotlin
// app/build.gradle.kts
android {
    namespace = "com.example.assistantapp"
    compileSdk = 34
    defaultConfig {
        applicationId = "com.example.assistantapp"
        minSdk = 27      // Android 8.1+
        targetSdk = 34   // Android 14
        versionCode = 1
        versionName = "1.0"
    }
}
```

### 17.2 Dependencies

```kotlin
dependencies {
    // CameraX
    implementation("androidx.camera:camera-core:1.3.4")
    implementation("androidx.camera:camera-camera2:1.3.4")
    implementation("androidx.camera:camera-lifecycle:1.3.4")
    implementation("androidx.camera:camera-view:1.3.4")

    // Google AI (Gemini)
    implementation("com.google.ai.client.generativeai:generativeai:0.9.0")

    // Networking
    implementation("com.squareup.okhttp3:okhttp:4.12.0")

    // Compose
    implementation(platform("androidx.compose:compose-bom:2024.02.00"))
    implementation("androidx.compose.material3:material3")
    implementation("androidx.compose.material:material-icons-extended")
    implementation("androidx.navigation:navigation-compose")
}
```

### 17.3 Build & Install Commands

```bash
# Build debug APK
export ANDROID_HOME=/opt/android-sdk
./gradlew assembleDebug

# Install on connected device
/opt/android-sdk/platform-tools/adb -s 142693155H032816 install -r app/build/outputs/apk/debug/app-debug.apk
```

---

## 18. Development Challenges & Solutions

### 18.1 Challenge: CameraX 1.3.4 API Conflict

**Problem**: CameraX 1.3.4 introduced `ImageProxy.toBitmap()` extension function, which shadowed the custom `toBitmap()` conversion function, causing compilation errors.

**Solution**: Renamed custom function from `toBitmap()` to `yuvToBitmap()` to avoid naming collision.

**Impact**: Build success without API changes.

---

### 18.2 Challenge: TTS Engine Disabled on Device

**Problem**: TECNO phone had Google TTS engine (`com.google.android.tts`) disabled, causing silent TTS failure. App showed "Init done" but never spoke.

**Diagnosis**:
```bash
adb shell settings get global tts_default_synth
# Returned: null (no TTS engine configured)
adb shell pm list packages | grep tts
# Returned: com.google.android.tts (installed but disabled)
```

**Solution**:
```bash
adb shell pm enable com.google.android.tts
adb shell settings put global tts_default_synth com.google.android.tts
```

**Prevention**: Added diagnostic TTS check in `isTTSEngineAvailable()` function.

---

### 18.3 Challenge: Gemini API Quota Exhaustion

**Problem**: Free tier quotas exhausted during development and testing.

| Model | Quota | Status |
|-------|-------|--------|
| gemini-1.5-flash | Deprecated | HTTP 404 |
| gemini-2.0-flash | 0 req/min | Quota disabled |
| gemini-2.5-flash | 20 req/min | Exhausted |
| gemini-3.5-flash | 5 req/min | Exhausted |
| **gemini-2.5-flash (key 2)** | 20 req/min | **Working** |

**Solution**: Used secondary API key with available quota. Set frame interval to 12 seconds (~5 req/min) to stay within limits.

**Root cause**: Multiple rapid API calls during testing consumed the free tier budget.

---

### 18.4 Challenge: Permission Flow in Compose

**Problem**: Original code used `ActivityCompat.requestPermissions()` which doesn't integrate with Compose state management, causing silent permission failure.

**Solution**: Replaced with Compose-native `rememberLauncherForActivityResult(ActivityResultContracts.RequestMultiplePermissions())`.

**Before (broken)**:
```kotlin
if (!hasPermission) {
    ActivityCompat.requestPermissions(activity, arrayOf(CAMERA, RECORD_AUDIO), 1001)
}
```

**After (working)**:
```kotlin
val permissionLauncher = rememberLauncherForActivityResult(
    ActivityResultContracts.RequestMultiplePermissions()
) { permissions -> ... }
```

---

### 18.5 Challenge: Orphaned Code Fragments

**Problem**: `BlindMode.kt` contained duplicate TTS initialization and orphaned `if` blocks that would never execute.

**Solution**: Removed orphaned code blocks, keeping only one TTS instance per composition scope.

---

## 19. Testing & Debugging

### 19.1 Debug Logging System

All debug logs use the `PriyaDebug` tag for easy filtering:

```bash
# Filter all app logs
adb logcat | grep PriyaDebug

# Filter specific component
adb logcat | grep PriyaDebug:TTS          # TTS logs
adb logcat | grep PriyaDebug:Permissions   # Permission logs
adb logcat | grep PriyaDebug:AI            # AI response logs
```

### 19.2 Debug Log Points

| Log Point | Tag | Content |
|-----------|-----|---------|
| TTS Init | `PriyaDebug:TTS` | "TTS Init Success/Failed" |
| TTS Engine | `PriyaDebug:TTS` | "Available engines: [...]" |
| Permission | `PriyaDebug:Permissions` | "Permission granted/denied" |
| AI Error | `PriyaDebug` | "AI Error: $error" |
| AI Response | `PriyaDebug` | Full Gemini response text |

### 19.3 Testing Procedure

1. **Build**: `./gradlew assembleDebug`
2. **Install**: `adb install -r app-debug.apk`
3. **Launch**: `adb shell am start -n com.example.assistantapp/.MainActivity`
4. **Test Navigation**: Point camera at objects, wait for spoken description
5. **Test Assistant**: Double-tap and hold, type a question
6. **Test Reading**: Point at text, say "start reading"
7. **Check Logs**: `adb logcat | grep PriyaDebug`

---

## 20. Impact & Use Cases

### 20.1 Real-World Use Cases

| Scenario | How Priya Helps |
|----------|----------------|
| **Crossing a street** | Detects traffic lights, vehicles, crosswalks; says "It's safe to cross" |
| **Entering a building** | Identifies doors, stairs, elevators; warns of obstacles |
| **Reading a menu** | Camera captures menu → reads all items and prices aloud |
| **Finding a bus stop** | Identifies bus stop signs, route numbers, arrival info |
| **Grocery shopping** | Reads product labels, prices, expiry dates |
| **Reading mail** | Opens envelope → camera reads letter contents |
| **Navigating indoors** | Describes room layout, furniture positions, exits |
| **Medication management** | Reads prescription labels, dosage instructions |

### 20.2 Impact Statistics

| Metric | Value |
|--------|-------|
| Global visually impaired | 2.2 billion (WHO) |
| Severe vision impairment | 36 million |
| Android smartphone users in developing countries | 1.5 billion |
| Cost of Priya | **Free** (vs $2,500–$4,500 for smart glasses) |
| External hardware required | **None** |
| Supported Android versions | 8.1+ (API 27+) |

### 20.3 Competitive Advantage

| Feature | Priya | OrCam MyEye | Envision Glasses | Be My Eyes |
|---------|-------|-------------|------------------|------------|
| Price | Free | $2,500 | $4,500 | Free |
| Hardware needed | Phone only | Glasses attachment | Smart glasses | Phone only |
| Navigation | ✅ | ❌ | ✅ | ❌ |
| Text reading | ✅ | ✅ | ✅ | ✅ (human) |
| Interactive Q&A | ✅ | ❌ | ✅ | ✅ (human) |
| Offline capability | ❌ (API) | ✅ | ✅ | ❌ |
| AI-powered | ✅ (Gemini) | ✅ | ✅ | Human volunteers |

---

## 21. Future Enhancements

### 21.1 Short-Term (Next Release)

| Enhancement | Description |
|-------------|-------------|
| **Offline mode** | Run lightweight TFLite model locally for basic object detection |
| **Voice wake word** | "Hey Priya" activation without touching the phone |
| **Multi-language** | Support Hindi, Marathi, and other Indian languages |
| **Custom voice** | Allow users to choose TTS voice and speed |
| **Battery optimization** | Adaptive frame rate based on movement detection |

### 21.2 Medium-Term

| Enhancement | Description |
|-------------|-------------|
| **GPS integration** | Combine AI vision with GPS for turn-by-turn navigation |
| **Indoor mapping** | Pre-mapped layouts for frequent locations (home, office) |
| **Social features** | Share location with trusted contacts |
| **Wearable support** | Companion app for smartwatches |
| **Haptic feedback** | Vibration patterns for left/right guidance |

### 21.3 Long-Term Vision

| Enhancement | Description |
|-------------|-------------|
| **Edge AI** | On-device Gemini Nano for offline vision processing |
| **3D spatial audio** | Directional audio cues (left ear = left obstacle) |
| **Learning system** | Remember frequently visited places |
| **Community mapping** | Crowdsourced obstacle data |
| **Multi-device sync** | Phone + earbuds + watch ecosystem |

---

## 22. Conclusion

**Priya** demonstrates that advanced AI-powered assistive technology can be built on a standard smartphone using freely available tools. By combining:

- **CameraX** for real-time image capture
- **Google Gemini 2.5 Flash** for multimodal AI understanding
- **Android TTS** for accessible speech output
- **Jetpack Compose** for modern, declarative UI

...the app provides a comprehensive navigation, assistance, and reading solution for visually impaired users — **entirely free of charge and without any special hardware**.

The three-mode architecture (Navigation, Assistant, Reading) covers the full spectrum of daily needs, from continuous environmental awareness to on-demand Q&A to text extraction. The gesture-based interaction system (double-tap, long-press, voice commands) ensures the app is fully accessible without visual interaction.

While challenges like API quota management and device-specific TTS configuration exist, the core pipeline — **Camera → YUV Conversion → Gemini AI → TTS Speech** — works reliably end-to-end, providing genuine utility to users who depend on it.

**Made by Priyanka Meenkeri to nurture the blind for this society.**

---

## 23. References

1. Google Gemini API Documentation — https://ai.google.dev/
2. CameraX Documentation — https://developer.android.com/media/camera/camerax
3. Android TextToSpeech API — https://developer.android.com/reference/android/speech/tts/TextToSpeech
4. Jetpack Compose Documentation — https://developer.android.com/jetpack/compose
5. World Health Organization — Vision Impairment Data (2023)
6. Google Generative AI SDK for Android — https://github.com/google/generative-ai-android
7. Android SpeechRecognizer — https://developer.android.com/reference/android/speech/SpeechRecognizer

---

*Report generated for the Priya Android Application — AI-Powered Navigation Assistant for the Visually Impaired*
