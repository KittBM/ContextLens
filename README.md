# ContextLens Specification

## 1. Product Overview

ContextLens คือแอปแปลข้อความบนหน้าจอด้วย AI โดยผู้ใช้สามารถกำหนดบริบทการแปลผ่าน Prompt และเลือก AI Provider ได้เอง เช่น Ollama, OpenRouter, OpenAI-compatible API

รองรับแพลตฟอร์มหลัก:

* Android
* Windows

---

## 2. Core Concept

```text
Screen Capture
→ OCR
→ Prompt Builder
→ AI Provider
→ Translation Result
→ Overlay / Floating Window
```

ผู้ใช้สามารถแปลข้อความจากหน้าจอเกม แอป เว็บ เอกสาร หรือโปรแกรมต่าง ๆ โดยไม่ต้อง copy text เอง

---

# 3. Target Platforms

## 3.1 Android

ใช้สำหรับ:

* แปลข้อความจากแอปอื่น
* แปลเกม
* แปลมังงะ / นิยาย / เว็บ
* แสดงผลแบบ overlay ทับหน้าจอ

Android components:

* MediaProjection สำหรับจับภาพหน้าจอ
* Foreground Service สำหรับทำงานต่อเนื่อง
* Accessibility Service สำหรับ overlay / interaction เพิ่มเติม
* ML Kit Text Recognition สำหรับ OCR
* Flutter UI สำหรับ setting และ prompt preset

---

## 3.2 Windows

ใช้สำหรับ:

* แปลข้อความจากหน้าจอ PC
* แปลเกม PC
* แปลเว็บ / PDF / โปรแกรม
* แสดง floating translation window

Windows components:

* Screen Capture API
* OCR Engine
* Floating overlay window
* System tray app
* Local AI / Remote AI connection

---

# 4. Main Features

## 4.1 Screen Capture

### Android

ผู้ใช้สามารถเลือกได้:

* Capture ทั้งหน้าจอ
* Capture เฉพาะพื้นที่
* Capture แบบ manual
* Capture แบบ interval เช่น ทุก 2 วินาที

Requirement:

* ขอ permission ผ่าน MediaProjection
* ต้องมี foreground notification ขณะจับหน้าจอ
* ต้องหยุด capture ได้ทันทีจาก notification หรือในแอป

---

### Windows

ผู้ใช้สามารถเลือกได้:

* Capture ทั้งหน้าจอ
* Capture เฉพาะ window
* Capture เฉพาะ region
* Capture จาก hotkey

Requirement:

* รองรับหลายจอ
* เลือก monitor ได้
* มี hotkey เช่น Ctrl + Shift + T เพื่อแปลหน้าจอ

---

## 4.2 OCR

OCR ต้องคืนค่าเป็น block พร้อมตำแหน่ง

```json
{
  "text": "Attack power +20%",
  "boundingBox": {
    "x": 120,
    "y": 250,
    "width": 180,
    "height": 40
  }
}
```

OCR requirements:

* ตรวจจับข้อความหลาย block
* รองรับภาษาอังกฤษ ญี่ปุ่น จีน ไทย
* รวมข้อความที่อยู่ใกล้กันได้
* เก็บตำแหน่งเพื่อใช้แสดง overlay

---

## 4.3 Prompt Context

ผู้ใช้สามารถสร้าง Prompt Preset ได้

ตัวอย่าง preset:

```text
This is a fantasy RPG game.
Translate into Thai.
Keep skill names, item names, character names in English.
Use natural Thai game localization.
```

Preset fields:

```json
{
  "name": "RPG Game",
  "targetLanguage": "Thai",
  "contextPrompt": "This is a fantasy RPG game.",
  "rules": [
    "Keep item names in English",
    "Keep skill names in English",
    "Use natural Thai localization"
  ]
}
```

---

## 4.4 AI Provider

ระบบต้องรองรับ AI Provider แบบเปลี่ยนได้

Provider types:

```dart
enum AiProviderType {
  ollama,
  openAiCompatible,
  openRouter,
  openAi,
  gemini,
  customHttp
}
```

---

## 4.5 Ollama Provider

ใช้สำหรับ local AI

Example config:

```json
{
  "name": "Local Ollama",
  "type": "ollama",
  "baseUrl": "http://192.168.1.10:11434",
  "model": "qwen2.5:7b",
  "apiKey": null
}
```

Endpoint:

```text
POST /api/chat
```

Use case:

* Android เรียก Ollama ที่รันบน PC
* Windows เรียก Ollama local ที่เครื่องตัวเอง
* ใช้งาน offline ภายในบ้านได้

---

## 4.6 OpenRouter / OpenAI-Compatible Provider

Example config:

```json
{
  "name": "OpenRouter",
  "type": "openAiCompatible",
  "baseUrl": "https://openrouter.ai/api/v1",
  "model": "openai/gpt-4o-mini",
  "apiKey": "xxx"
}
```

Endpoint:

```text
POST /chat/completions
```

ใช้ร่วมกับ:

* OpenRouter
* LM Studio
* LocalAI
* vLLM
* OpenAI-compatible server อื่น ๆ

---

# 5. AI Translation Flow

## Input

```json
{
  "sourceText": "Attack power +20%",
  "targetLanguage": "Thai",
  "context": "Fantasy RPG game",
  "rules": [
    "Keep skill names in English",
    "Use natural Thai localization"
  ]
}
```

## System Prompt

```text
You are a professional translator.
Translate text based on the user's context.
Return only translated text.
Do not explain unless requested.
```

## User Prompt

```text
Target language: Thai

Context:
Fantasy RPG game

Rules:
- Keep skill names in English
- Use natural Thai localization

Text:
Attack power +20%
```

## Output

```text
พลังโจมตี +20%
```

---

# 6. Overlay Display

## Android Overlay

Overlay mode:

* แสดงข้อความแปลทับตำแหน่งเดิม
* แสดงเป็น floating bubble
* แตะข้อความเพื่อดูต้นฉบับ
* แตะค้างเพื่อถาม AI เพิ่มเติม

Android permissions:

* Draw over other apps
* MediaProjection
* Foreground service
* Internet

---

## Windows Overlay

Overlay mode:

* Floating window
* Always on top
* Click-through mode
* Region highlight
* Hotkey toggle

Windows features:

* System tray
* Global hotkey
* Multi-monitor support
* Transparent overlay

---

# 7. Translation Modes

## 7.1 Manual Mode

ผู้ใช้กดปุ่มแปลเอง

เหมาะกับ:

* เอกสาร
* เว็บ
* รูปภาพ
* หน้าจอที่ไม่เปลี่ยนบ่อย

---

## 7.2 Real-Time Mode

ระบบ capture หน้าจอตาม interval

ตัวอย่าง:

```text
Every 2 seconds
→ Capture
→ OCR
→ Translate
→ Update Overlay
```

เหมาะกับ:

* เกม
* subtitle
* visual novel

---

## 7.3 Region Mode

ผู้ใช้เลือกพื้นที่แปลเฉพาะจุด

เหมาะกับ:

* subtitle
* กล่องข้อความเกม
* manga panel

---

# 8. App Screens

## 8.1 Home

* Start Translation
* Select Mode
* Select Prompt Preset
* Select AI Provider

---

## 8.2 Prompt Preset

* Create preset
* Edit preset
* Delete preset
* Duplicate preset
* Set as default

---

## 8.3 AI Provider Settings

Fields:

* Provider name
* Provider type
* Base URL
* API key
* Model
* Temperature
* Max tokens
* Timeout
* Test connection

---

## 8.4 OCR Settings

* OCR language
* Minimum text size
* Merge nearby text
* Ignore repeated text
* Capture quality

---

## 8.5 Overlay Settings

* Font size
* Background opacity
* Position mode
* Click-through mode
* Show original text
* Auto-hide

---

# 9. Data Model

## AiProviderConfig

```dart
class AiProviderConfig {
  final String id;
  final String name;
  final AiProviderType type;
  final String baseUrl;
  final String model;
  final String? apiKey;
  final double temperature;
  final int maxTokens;
  final int timeoutSeconds;
}
```

## PromptPreset

```dart
class PromptPreset {
  final String id;
  final String name;
  final String targetLanguage;
  final String contextPrompt;
  final List<String> rules;
  final bool isDefault;
}
```

## OcrTextBlock

```dart
class OcrTextBlock {
  final String text;
  final Rect boundingBox;
  final double confidence;
}
```

## TranslationResult

```dart
class TranslationResult {
  final String sourceText;
  final String translatedText;
  final Rect boundingBox;
  final String providerId;
  final String model;
}
```

---

# 10. Technical Architecture

## Flutter Layer

ใช้ร่วมกันทั้ง Android และ Windows

Responsibilities:

* UI
* Settings
* Prompt preset
* Provider config
* Translation history
* State management
* API calling

Recommended stack:

* Flutter
* Riverpod
* Dio
* Hive / Isar
* go_router

---

## Android Native Layer

Responsibilities:

* MediaProjection
* Foreground Service
* Overlay Window
* Accessibility integration
* OCR bridge

Communication:

```text
Flutter
↔ MethodChannel
↔ Android Native Service
```

---

## Windows Native Layer

Responsibilities:

* Screen capture
* Floating overlay
* Global hotkey
* System tray
* Window selection

Communication:

```text
Flutter
↔ MethodChannel / FFI
↔ Windows Native Module
```

---

# 11. Provider Interface

```dart
abstract interface class AiTranslatorClient {
  Future<String> translate({
    required String systemPrompt,
    required String userPrompt,
    required AiProviderConfig config,
  });
}
```

Implementations:

```text
OllamaTranslatorClient
OpenAiCompatibleTranslatorClient
GeminiTranslatorClient
CustomHttpTranslatorClient
```

---

# 12. MVP Scope

## V1 Android

Required:

* Manual screen capture
* OCR
* Prompt preset
* Ollama provider
* OpenAI-compatible provider
* Overlay result
* Provider test connection

Not required:

* Real-time mode
* Translation history sync
* AI glossary
* Auto context detection

---

## V1 Windows

Required:

* Hotkey capture
* Region capture
* OCR
* Prompt preset
* Ollama provider
* OpenAI-compatible provider
* Floating result window
* System tray

Not required:

* Full transparent overlay
* Click-through overlay
* Multi-monitor advanced mode
* Auto translate interval

---

# 13. V2 Scope

## V2 Features

* Real-time translation
* Region auto-refresh
* AI glossary
* Translation history
* Auto context detection
* Multi-monitor support
* Click-through overlay
* AI explain selected word
* Export prompt preset
* Import prompt preset
* Cloud sync settings

---

# 14. Security

## API Key Storage

Android:

* flutter_secure_storage
* Android Keystore

Windows:

* Windows Credential Manager
* Encrypted local storage

---

## Privacy

Requirements:

* ไม่ส่ง screenshot ทั้งภาพไป AI ถ้าไม่จำเป็น
* ส่งเฉพาะ OCR text เป็น default
* มี option ให้ส่ง image เฉพาะกรณีใช้ vision model
* แจ้งผู้ใช้เมื่อใช้ remote AI provider
* Local Ollama mode ต้องสามารถทำงานใน LAN ได้

---

# 15. Performance Target

## Android

* Manual translation: ไม่เกิน 3 วินาที
* OCR: ไม่เกิน 800 ms
* Overlay update: ไม่เกิน 300 ms หลังได้ผลแปล
* Real-time mode V2: 1–2 วินาทีต่อรอบ

---

## Windows

* Region capture: ไม่เกิน 300 ms
* OCR: ไม่เกิน 800 ms
* Translation latency ขึ้นกับ provider
* Overlay update: ไม่เกิน 300 ms หลังได้ผลแปล

---

# 16. Error Handling

Cases:

* OCR ไม่เจอข้อความ
* AI provider connection failed
* API key invalid
* Model not found
* Timeout
* Overlay permission denied
* Screen capture permission denied

Example error:

```text
ไม่สามารถเชื่อมต่อ AI Provider ได้
กรุณาตรวจสอบ Base URL, Model หรือ API Key
```

---

# 17. Suggested App Name

Recommended:

```text
ContextLens
```

Alternatives:

* PromptLens
* ScreenContext
* TranslateLens
* AI Screen Translator
* PromptTranslate

---

# 18. Development Priority

## Phase 1

* Flutter UI
* Provider settings
* Prompt preset
* OpenAI-compatible client
* Ollama client

## Phase 2

* Android screen capture
* Android OCR
* Android overlay

## Phase 3

* Windows screen capture
* Windows OCR
* Windows floating window

## Phase 4

* Real-time mode
* Region mode
* Glossary
* AI explain

---
