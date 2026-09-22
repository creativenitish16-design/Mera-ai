# MIRA AI - Android Voice Assistant

A premium, always-listening Android assistant: a floating Siri-style orb, offline
wake word, sub-second local commands, and cloud LLM conversation in Hindi,
English, Telugu and more.

## Open it

1. Install Android Studio Ladybug or newer.
2. File > Open and pick this folder.
3. Let Gradle sync, then press Run. minSdk 26, targetSdk 35.

## Add your keys

Open app/build.gradle.kts and fill in the three buildConfigField values:

- GEMINI_KEY  - free at aistudio.google.com (default provider)
- OPENAI_KEY  - optional; if set, GPT is used instead of Gemini
- PICOVOICE_KEY - free at console.picovoice.ai (required for the wake word)

## What works offline vs online

| Feature | Offline | Notes |
|---|---|---|
| Open app / call / SMS / torch / volume / timer | Yes | Instant, no network |
| Wake word "Hey Mira" | Yes | Porcupine, on-device |
| Speech to text | Yes on Android 13+ | createOnDeviceSpeechRecognizer |
| Text to speech | Yes | System TTS engine |
| General conversation, translation, summarising | No | Needs an API key |

## Architecture

- core/MiraEngine.kt - one shared brain used by the activity, service and orb
- ai/AiClient.kt - Gemini or OpenAI-compatible chat, memory-aware
- speech/ - SpeechManager (voice in) and TtsManager (voice out)
- wake/WakeWordEngine.kt - on-device hotword detection
- commands/ - CommandRouter plus App, Device and Comms controllers
- service/ - foreground mic service, floating orb overlay, boot receiver
- ui/ - Compose Material 3 theme, animated orb, waveform, chat and settings

## Android 14/15 rules already handled

- Foreground service declares foregroundServiceType="microphone|specialUse" and
  the matching FOREGROUND_SERVICE_MICROPHONE permission, or it crashes on API 34+.
- The orb runs as its own specialUse foreground service with SYSTEM_ALERT_WINDOW.
- Boot restart posts a notification instead of starting the mic service directly,
  because Android 14/15 block that from BOOT_COMPLETED.

## Performance notes

Local commands short-circuit before any network call, so they land well under one
second. The orb is a single Canvas draw with no per-frame allocations. Wake word
runs at roughly 1% of one CPU core, which is what makes always-listening viable.

## Extending

- OCR / PDF summaries / image understanding: add Google ML Kit text recognition
  and a Gemini vision call, then route the intent in CommandRouter.
- Custom wake word: train a .ppn model in the Picovoice console and swap
  setKeyword for setKeywordPath in WakeWordEngine.
