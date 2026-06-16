# ueRFTest - Mobile RF Analyzer & Network Test Suite

![ueRFTest Banner](banner.png)

`ueRFTest` is a professional-grade Android application designed for RF engineering, network analysis, and automated field testing. It enables continuous background tracking of cellular signal metrics (LTE/5G), device throughput, and per-app traffic, alongside automated active testing modules for SMS, Voice, and Speed Tests.

---
## Developer Informaation

**Version:** 2.09 
**Developer:** xAPPS-Lab.ca  
**Website:** [android.xapps-lab.com](https://android.xapps-lab.com)  
**LinkedIn:** [jstaguan](https://www.linkedin.com/in/jstaguan)  
**Privacy Policy:** [androidapps.xapps-lab.com/privacy](https://androidapps.xapps-lab.com/privacy)

### APP Status: Under Internal Testing

**Google Play (Internal Testing): ** drop us your Gmail at android.xappslab@gmail.com & get the latest release.

---

## Key Features

### 1. Continuous RF & Location Logging
* **State Tracking**: Monitors GPS latitude, longitude, altitude, speed, and heading in real-time.
* **Serving Cell Parameters**:
  * **LTE**: RSRP, RSRQ, SINR, PCI, TAC, EARFCN, CA bands, total bandwidth, and PLMN ID.
  * **5G (NR SA/NSA)**: SS-RSRP, SS-SINR, NR-PCI, NR-TAC, NR-ARFCN, CA bands, and band info.
* **Persistent Engine**: Runs as an Android Foreground Service (`RFTestService`) to ensure uninterrupted background collection, even when the device is locked.

### 2. Live & Historical Charts
* **Custom Compose Charts**: Built entirely using Compose `Canvas` (avoiding heavy external libraries):
  * **TreeMap Chart**: Real-time traffic distribution per application.
  * **Stacked Bar Chart**: Hourly UL/DL traffic volume (moving 24-hour window) and serving technology sample ratios.
  * **Line Chart**: RSRP, SINR, and SS-RSRP/SS-SINR hourly averages, plus active throughput trends.
* **Granular Filtering**: Filter charts and analytics datasets by specific date ranges and resolutions.

### 3. Spatial Mapping (Maps Tab)
* **OpenStreetMap & Google Maps**: Uses `osmdroid` for offline/online spatial plotting, with optional Google Maps integration via API key configuration in settings.
* **Spatial Binning**: Implements dynamic binning ($10\text{m} \times 10\text{m}$ grid size) to group and average close coordinates, preventing rendering lag and coordinate noise.
* **Dynamic Legends**: Tap the legend container to switch plots between:
  * **Serving Technology** (LTE, 5G NSA, 5G SA)
  * **Device Speed** (km/h)
  * **Throughput** (Mbps)
  * **RSRP / SS-RSRP** (dBm)
  * **SINR / SS-SINR** (dB)
* **Visual Controls**: Real-time adjustable point size slider.

### 4. Automated Active Testing
* **Voice Calls**: Configurable loop frequency and intervals, validating ringing and active states.
* **SMS Test Suite**:
  * **SMS MoC**: Mobile Originating Call (validates delivery status within 10 seconds).
  * **SMS MoC & MTC**: Combined originator and receiver test (delivers and expects receiving SMS within 20 seconds).
* **Speed Test Servers**:
  * **Ookla Speed Test CLI**: Integrated with the official binary. Performs automated runtime ABI detection (supports `arm64-v8a`, `armeabi-v7a`, `x86_64`), downloads the binary dynamically from the official server, unpacks it using a pure-Java extractor, and streams real-time JSON metrics to a live Compose gauge.
  * **Cloudflare CDN**: Pure Kotlin implementation utilizing HTTP/1.1 chunked streams for RTT, 90MB Download, and POST Upload testing. Bypasses bot mitigations using realistic mobile User-Agents.
  * **Fast.com**: Netflix-based download throughput measurement.
  * **Hybrid Modes**: Combined execution chains (e.g. download via Fast.com, upload via Ookla/CDN) with 5-second silence stabilization buffers.
* **Dashboard Indicators**: Live test progression ring, Target KPI boundaries (e.g., target throughput threshold checks), and active report data tables.

### 5. Database & Maintenance
* **Room SQLite Database**: Storage configuration using WAL (Write-Ahead Logging) mode.
* **Auto-Pruning Engine**: Configurable maximum storage rules (prunes data older than 31 days or exceeding 2 GB on the first of the month).
* **Logs Management**: Manual and schedule-based database backups/restores, and CSV exporter for custom date ranges.

---

## Technical Stack & Architecture

The application follows the **MVVM** pattern combined with a singleton state manager:

```
┌───────────────────────────────────────────┐
│         RFTestService (Foreground)        │
│  - GPS updates (2s)                       │
│  - TelephonyCallback (RF metrics)         │
│  - TrafficStats (Throughput)              │
└─────────────┬───────────────────────┬─────┘
              │                       │
              ▼ (Flow Broadcasts)     ▼ (Writes every ~1s)
┌───────────────────────────┐   ┌───────────────────────────┐
│       RFDataManager       │   │       Room Database       │
│  (StateFlow Singleton)    │   │  - basic_data_samples     │
│  - Live Signal metrics    │   │  - traffic_samples        │
│  - Active test progress   │   │  - app_traffic_samples    │
│  - Logger status          │   │  - test_results           │
└─────────────┬─────────────┘   └─────────────┬─────────────┘
              │                               │
              ▼ (Exposes Live State)          ▼ (Historical Queries)
┌───────────────────────────────────────────────────────────┐
│                        AppViewModel                       │
│  - Central shared ViewModel across Compose tabs           │
│  - Triggers DB aggregations & CSV exports                 │
│  - Controls Service commands (Start, Stop, Schedule)      │
└─────────────────────────────┬─────────────────────────────┘
                              ▼
┌───────────────────────────────────────────────────────────┐
│                    Jetpack Compose UI                     │
│  - Home  |  Test  |  Map  |  Analytics  |  Settings       │
└───────────────────────────────────────────────────────────┘
```

* **SDK Version**: Min SDK 29 (Android 10) | Target SDK 36 (Android 15)
* **Jetpack Compose**: Navigation 3, Material 3, custom Compose `Canvas` drawing
* **Room ORM**: Managed with KSP (Kotlin Symbol Processing), uses destructive fallback on migrations
* **Coroutines & Flows**: Direct reactive data mapping from service to presentation layer

---

## How to Build & Install From Scratch

### Prerequisites
1. **Android SDK**: Install SDK 29 through 36 via Android Studio.
2. **JDK**: Java JDK 17 or later.
3. **Gradle**: Wrapper scripts are bundled in the repository.

### Required App Permissions
To execute properly, the application requires the user to manually grant the following runtime permissions on the device:
* **Location**: `ACCESS_FINE_LOCATION` & `ACCESS_COARSE_LOCATION` (Required to map cell coverage and track drive tests).
* **Phone State**: `READ_PHONE_STATE` & `READ_PRECISE_PHONE_STATE` (Required to read serving carrier details, PLMN, RSRP, and signal quality metrics).
* **SMS**: `SEND_SMS` & `RECEIVE_SMS` (Required for automated SMS MoC and MTC testing).
* **Phone Call**: `CALL_PHONE` (Required to automate voice call testing).
* **Notifications**: `POST_NOTIFICATIONS` (Required to display the active foreground service notification).

---

## UI Screenshots

### Home / Main Dashboard
| | | |
|:---:|:---:|:---:|
| ![Main Dashboard](screenshots/01_home_01_maindashboard.png) | ![Home Charts](screenshots/01_home_02.png) | ![Signal Metrics](screenshots/01_home_03.png) |
| Live RF metrics: serving cell, technology, and signal indicators | Hourly UL/DL traffic volume bar chart and technology sample ratio | RSRP/SINR trend chart and carrier aggregation status |
| ![Home Detail](screenshots/01_home_04.png) | | |
| Expanded network parameters and CA band detail | | |

### Test Screen
| | | |
|:---:|:---:|:---:|
| ![App Traffic](screenshots/02_Test_01_Applications.png) | ![Speed Test](screenshots/02_Test_02_01_SpeedTest.png) | ![Speed Options](screenshots/02_Test_02_02_SpeedTestOptions.png) |
| Per-application network traffic treemap and data usage breakdown | Speed test interface with live throughput gauge | Speed test server and mode selection |
| ![iPerf Config](screenshots/02_Test_02_03_iPerfTesting.png) | ![iPerf Running](screenshots/02_Test_02_04_iPerfTesting.png) | ![SMS Test](screenshots/02_Test_02_SMS.png) |
| iPerf3 test configuration (server, port, direction, volume) | iPerf3 test running with real-time RTT and throughput | Automated SMS MoC/MTC test setup and pass/fail results |
| ![Voice Call](screenshots/02_Test_03_VoiceCall.png) | | |
| Automated voice call test with ring/active state validation | | |

### Tasks / Scheduled Tests
| | | |
|:---:|:---:|:---:|
| ![Schedule 1](screenshots/03_Tasks_01_ScheduleTest.png) | ![Schedule 2](screenshots/03_Tasks_02_ScheduleTest.png) | ![Scheduled List](screenshots/03_Tasks_03_Scheduled.png) |
| Schedule a new automated test run | Test schedule time and repeat configuration | Active scheduled tests list |
| ![Scheduled Detail](screenshots/03_Tasks_04_Scheduled.png) | ![Running 1](screenshots/03_Tasks_05_ScheduledTestRunning.png) | ![Running 2](screenshots/03_Tasks_06_ScheduledTestRunning.png) |
| Scheduled test detail and status | Scheduled test executing in background | Live results during scheduled test execution |

### Map Screen
| | | |
|:---:|:---:|:---:|
| ![Map Overview](screenshots/03_Map_01.png) | ![Map Coverage](screenshots/04_Map_01.png) | ![Map Speed](screenshots/04_Map_02.png) |
| Map screen with initial OSM base layer | Color-coded signal coverage overlay (RSRP) | Map with device speed KPI coloring |
| ![Map Technology](screenshots/04_Map_03.png) | ![Map Legend](screenshots/04_Map_04.png) | |
| Serving technology color plot (LTE / 5G NSA / 5G SA) | Full KPI legend panel (tap to switch overlay) | |

### Analytics Screen
| | | |
|:---:|:---:|:---:|
| ![Analytics Filters](screenshots/05_Analytics_01.png) | ![Traffic Trend](screenshots/05_Analytics_02_Trend.png) | ![Signal Trend](screenshots/05_Analytics_03_Trend.png) |
| Date range picker and aggregation selector (Sec/Min/Hour/Day) | Traffic volume trend chart (DL/UL) over selected range | RSRP and SINR level trend over selected time window |
| ![Profile](screenshots/05_Analytics_04_Profile.png) | ![Session](screenshots/05_Analytics_05_UserSession.png) | ![CA Chart](screenshots/05_Analytics_06_CarrierAggregation.png) |
| RF profile tab: metrics rings and RSRP scatter plot | User session payload summary by interface | Carrier aggregation band distribution chart |
| ![Session Payload](screenshots/05_Analytics_06_SessionPayload.png) | | |
| Session-level data volume breakdown | | |

### Settings Screen
| | | |
|:---:|:---:|:---:|
| ![Settings 1](screenshots/06_Settins_01.png) | ![Settings 2](screenshots/06_Settins_02.png) | ![Settings 3](screenshots/06_Settins_03.png) |
| Map provider and Google Maps API key configuration | Schedule and data collection settings | Storage location selection (internal vs. external) |
| ![Settings 4](screenshots/06_Settins_04.png) | ![Settings 5](screenshots/06_Settins_05.png) | ![Settings 6](screenshots/06_Settins_06.png) |
| Database backup, restore, CSV export, and wipe | Map default KPI and legend color configuration | About application — version, developer info, and links |


---

## Disclaimer & Legal Notice

This application contains code that interfaces with third-party testing servers, including:
1. **Cloudflare Speedtest**: Performs API/CDN calls to Cloudflare's public test infrastructure.
2. **iPerfr Speedtest**: Performs TCP/UDP calls to Private test infrastructure.

**Terms of Use:** 
* These integrations are provided for network diagnostics, testing, and academic evaluation purposes only. 
* Excessive testing can consume substantial amounts of data. Users are solely responsible for carrier billing, data overage fees, and complying with the acceptable use policies and terms of service of each respective speed testing platform.
* The developers of `ueRFTest` are not associated, endorsed, or affiliated with Ookla, Netflix, or Cloudflare.
