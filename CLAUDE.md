# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Status

Planning-stage repository — Android source code has not yet been initialized. Per `myPlan.md`:
- Target version: `2.09`
- Remote repo: `https://github.com/tahgoi/DemoApp_ueRFTest.git`
- Keep a `TODO.md` and mark tasks complete as they are done
- Commits and pushes must not mention Claude, Gemini, or any other LLMs

## Build & Run (once project is initialized)

```bash
# Build debug APK
./gradlew assembleDebug

# Run unit tests
./gradlew test

# Run a single test class
./gradlew test --tests "com.example.ueRFTest.YourTestClass"

# Run instrumented tests on device/emulator
./gradlew connectedAndroidTest

# Install on connected device
./gradlew installDebug
```

- **Min SDK**: 29 (Android 10) | **Target SDK**: 36 (Android 15)
- **JDK**: 17+
- Room uses KSP; run `./gradlew kspDebugKotlin` if generated sources are missing after schema changes

## Architecture

MVVM with a singleton state manager. Unidirectional data flow:

```
RFTestService (Foreground)
  └─ GPS (2s) + TelephonyCallback + TrafficStats
       ├─ Flow broadcasts → RFDataManager (StateFlow singleton)
       └─ Writes (~1s)   → Room Database
                                ↓
                          AppViewModel  (single shared ViewModel across all tabs)
                                ↓
                     Jetpack Compose UI
                     Home | Test | Map | Analytics | Settings
```

**Key classes:**
- `RFTestService` — foreground service; sole producer of live RF + location data
- `RFDataManager` — singleton StateFlow hub; UI observes this, not the service directly
- `AppViewModel` — triggers DB aggregations, CSV exports, service start/stop/schedule commands
- Room tables: `basic_data_samples`, `traffic_samples`, `app_traffic_samples`, `test_results`

## Tech Stack

- **UI**: Jetpack Compose + Navigation 3 + Material 3; all charts use Compose `Canvas` only (no chart library)
- **Database**: Room with WAL mode, KSP code generation, destructive migration fallback
- **Mapping**: `osmdroid` (primary, offline-capable) + optional Google Maps (API key set in Settings)
- **Reactive layer**: Coroutines + Flows from service → ViewModel → UI
- **Speed test engines**: Ookla CLI (runtime ABI detection + pure-Java zip extractor), Cloudflare (chunked HTTP/1.1), Fast.com, iPerf3

## Key Implementation Details

- Spatial map binning: 10 m × 10 m grid to de-noise coordinates before rendering
- Auto-pruning fires on the 1st of each month: removes data older than 31 days or when DB exceeds 2 GB
- Ookla binary is downloaded and unpacked at runtime — do not bundle it in assets
- Hybrid speed test modes insert a 5-second silence stabilization buffer between phases
- SMS MoC timeout: 10 s; SMS MoC+MTC combined timeout: 20 s

## Runtime Permissions

Must be granted manually on-device: `ACCESS_FINE_LOCATION`, `ACCESS_COARSE_LOCATION`, `READ_PHONE_STATE`, `READ_PRECISE_PHONE_STATE`, `SEND_SMS`, `RECEIVE_SMS`, `CALL_PHONE`, `POST_NOTIFICATIONS`.
