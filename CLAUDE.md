# CLAUDE.md — Iris Android

## Project Overview

Iris is an offline Android chat application built on top of `llama.cpp`. It runs large language models (LLMs) entirely on-device for privacy-focused, internet-independent AI conversations. Published on the Google Play Store by Nerve Sparks.

Key capabilities: offline LLM inference, GGUF model management (download from Hugging Face), customizable generation parameters, text-to-speech, speech-to-text.

## Repository Structure

```
/
├── app/                        # Main Android application module
│   ├── build.gradle.kts        # App build config (Compose, dependencies)
│   └── src/main/java/com/nervesparks/iris/
│       ├── MainActivity.kt         # Entry point (ComponentActivity)
│       ├── MainViewModel.kt        # Central state management & business logic
│       ├── ChatScreen.kt           # Chat screen composable
│       ├── Downloadable.kt         # Download manager wrapper
│       ├── data/
│       │   └── UserPreferencesRepository.kt  # SharedPreferences persistence
│       └── ui/
│           ├── MainChatScreen.kt       # Primary chat UI (largest component)
│           ├── ModelsScreen.kt         # Model selection/management
│           ├── ParametersScreen.kt     # LLM parameter tuning
│           ├── SettingsScreen.kt       # App settings
│           ├── BenchMarkScreen.kt      # Performance benchmarking
│           ├── SearchResultScreen.kt   # Model search results
│           ├── AboutScreen.kt          # About page
│           ├── components/             # Reusable UI components
│           │   ├── ChatSection.kt
│           │   ├── ModelCard.kt
│           │   ├── DownloadModal.kt
│           │   ├── DownloadInfoModal.kt
│           │   └── LoadingModal.kt
│           └── theme/                  # Compose theming
│               ├── Theme.kt
│               ├── Color.kt
│               └── Type.kt
├── llama/                      # Native LLM library module
│   ├── build.gradle.kts        # Library build config (CMake, NDK)
│   └── src/main/
│       ├── java/android/llama/cpp/
│       │   └── LLamaAndroid.kt     # JNI wrapper (singleton)
│       └── cpp/
│           ├── llama-android.cpp    # JNI C++ implementation
│           └── CMakeLists.txt       # Native build config
├── build.gradle.kts            # Root Gradle config (plugin declarations)
├── settings.gradle.kts         # Multi-module settings (:app, :llama)
├── gradle.properties           # Gradle JVM args, Kotlin style
├── images/                     # Documentation screenshots
├── Readme.md                   # Project documentation
└── LICENSE                     # Apache 2.0
```

## Tech Stack

| Layer | Technology |
|-------|------------|
| Language | Kotlin (app), C++ (native inference) |
| UI | Jetpack Compose + Material 3 |
| Architecture | MVVM (ViewModel + Repository) |
| Async | Kotlin Coroutines + Flow |
| Native | llama.cpp via JNI |
| HTTP | OkHttp 4.10.0 |
| Storage | Android SharedPreferences (via DataStore wrapper) |
| Navigation | androidx.navigation-compose |
| Build | Gradle 8.7 + Android Gradle Plugin 8.5.2 |
| NDK | v26.1.10909125 with CMake 3.22.1 |

## Build & Run

Prerequisites: Android Studio with NDK 26.1.10909125 installed, CMake 3.22.1.

```bash
# Build debug APK
./gradlew assembleDebug

# Build release APK
./gradlew assembleRelease

# Run unit tests
./gradlew test

# Run instrumented tests (requires device/emulator)
./gradlew connectedAndroidTest

# Clean build
./gradlew clean
```

**Android SDK targets:**
- compileSdk: 35
- targetSdk: 34
- minSdk: 28

**Important:** The `llama` module depends on an external llama.cpp checkout. The CMakeLists.txt references it at a specific commit (`1f922254f0c984a8fb9fbaa0c390d7ffae49aedb`). Ensure the native sources are available before building.

## Architecture & Key Patterns

### MVVM Architecture
- **View layer:** Compose screens in `ui/` — declarative, stateless composables
- **ViewModel:** `MainViewModel.kt` — holds UI state, manages LLM lifecycle, TTS, download coordination
- **Repository:** `UserPreferencesRepository.kt` — abstracts SharedPreferences access

### Singleton Services
- `LLamaAndroid` — thread-safe singleton managing native llama.cpp lifecycle (load, context, inference)
- `UserPreferencesRepository` — synchronized singleton for preferences

### Native Integration (JNI)
- Kotlin calls → `LLamaAndroid.kt` (JNI wrapper) → `llama-android.cpp` (C++ implementation)
- Key native functions: `load_model`, `new_context`, `completion_loop`, `bench_model`
- Inference runs on background coroutine dispatcher, emits tokens via Flow

### Navigation
- Single-activity architecture with Compose Navigation
- Screens: Chat, Models, Parameters, Settings, Benchmark, Search, About

## Code Conventions

- **Kotlin style:** `kotlin.code.style=official` (follow official Kotlin conventions)
- **Package:** `com.nervesparks.iris.*`
- **JVM target:** Java 1.8
- **Compose:** All UI is declarative Compose — no legacy XML layouts
- **Naming:** ViewModel suffix for state classes, Repository suffix for data access, composable functions follow Compose naming conventions
- **Async:** Always use Coroutines/Flow, never blocking calls on main thread
- **No linter config:** No ktlint or detekt — follow Android Studio/IntelliJ defaults

## LLM Parameters

These runtime parameters are user-configurable:

| Parameter | Description |
|-----------|-------------|
| `n_threads` | CPU thread count for inference |
| `top_p` | Nucleus sampling (0.0–1.0) |
| `top_k` | Top-K sampling |
| `temp` | Temperature (randomness) |
| `nlen` | Max tokens per generation (hardcoded 1024) |
| `context_size` | Model context window (hardcoded 4096) |

## Permissions

The app declares these Android permissions:
- `INTERNET` — downloading models from Hugging Face
- `RECORD_AUDIO` — speech-to-text input
- `VIBRATE` — haptic feedback

## Testing

- **Unit tests:** JUnit 4 in `llama/src/test/`
- **Instrumented tests:** Espresso in `llama/src/androidTest/`
- Test coverage is minimal (example tests only)
- No CI/CD pipeline — builds and deploys are manual

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make changes following existing code conventions
4. Test on a physical Android device (LLM inference is resource-intensive)
5. Submit a pull request

## Common Tasks for AI Assistants

- **Adding a new screen:** Create composable in `ui/`, add navigation route in `MainActivity.kt`
- **Adding a new model:** Update `allModels` list in `MainViewModel.kt`
- **Modifying inference behavior:** Edit `LLamaAndroid.kt` (Kotlin side) and `llama-android.cpp` (native side)
- **Changing UI theme:** Edit files in `ui/theme/`
- **Adding a preference:** Add key/getter/setter in `UserPreferencesRepository.kt`, wire into `MainViewModel`
