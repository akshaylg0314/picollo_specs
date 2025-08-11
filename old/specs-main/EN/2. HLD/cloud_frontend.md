# PICCOLO Cloud CI/CD Platform - Frontend UI Specification

**Document Number**: PICCOLO-CLOUD-CICD-UI-2025-001  
**Date**: 2025-07-18  
**Author**: joshua-jung_LGESDV  
**Classification**: UI Specification  
**Current Time**: 2025-07-18 01:21:52 UTC

## 1. Development Stage UI

### 1.1 Project Creation Screen
```
+-----------------------------------------------------------+
|  PICCOLO Cloud CI/CD                                [user▼]|
+-----------------------------------------------------------+
|                                                           |
| [Home] [Projects] [Resources] [Activities] [Settings]      |
|                                                           |
+-----------------------------------------------------------+
|                                                           |
|  ◆ Create New Project                                     |
|                                                           |
|  Basic Information                                        |
|  +---------------------------------------------------+    |
|  | Project Name*        | [Vehicle Navigation System]  |    |
|  | Project Code*        | [VEH-NAV-2025              ] |    |
|  | Description          | [Navigation system for vehicles] |    |
|  | Access Scope         | (●) Team  ( ) Organization  ( ) Public |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Target Environment                                       |
|  +---------------------------------------------------+    |
|  | Region          | [✓] North America [✓] Europe [✓] Asia |    |
|  | Vehicle Models  | [✓] Model A   [✓] Model B         |    |
|  |                 | [ ] Model C   [ ] Model D         |    |
|  | Hardware        | [ Add + ]                         |    |
|  |  ├─ NVIDIA Xavier   | Main Processing Unit  | [Delete] |    |
|  |  └─ Qualcomm 8155  | Sensor Integration Unit | [Delete] |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Repository Information                                   |
|  +---------------------------------------------------+    |
|  | [ Add + ]                                         |    |
|  |  ├─ https://github.com/lgesdv/nav-system          |    |
|  |  |    Branch: main   Purpose: Main Repo  | [Delete] |    |
|  |  └─ https://github.com/lgesdv/ui-components       |    |
|  |       Branch: stable Purpose: UI Library | [Delete] |    |
|  +---------------------------------------------------+    |
|                                                           |
|                                                           |
|  [  Cancel  ]                           [  Create Project  ] |
|                                                           |
+-----------------------------------------------------------+
```

### 1.2 Development Environment (Cloud IDE) Screen
```
+-----------------------------------------------------------+
|  PICCOLO Cloud IDE - Vehicle Navigation System      [user▼]|
+-----------------------------------------------------------+
|                                                           |
| [File] [Edit] [View] [Run] [Debug] [Terminal] [Tools] [Help] |
|                                                           |
+-----------------------------------------------------------+
|                    |                                      |
| EXPLORER           |  // src/navigation/MapRenderer.js    |
| ├─ 📂 src          |                                      |
| │  ├─ 📂 navigation|  import React, { useEffect } from 'r'|
| │  │  ├─ 📄 MapRen |                                      |
| │  │  ├─ 📄 RouteC |  /**                                 |
| │  │  └─ 📄 MapSty |   * 3D Map Rendering Component       |
| │  ├─ 📂 ui        |   * @param {Object} props           |
| │  │  ├─ 📄 Button |   */                                |
| │  │  └─ 📄 Panel. |  function MapRenderer(props) {       |
| │  └─ 📂 utils     |    useEffect(() => {                 |
| ├─ 📂 tests        |      // Map initialization code      |
| ├─ 📄 package.json |      initializeMap();                |
| └─ 📄 README.md    |    }, []);                           |
|                    |                                      |
| SIGNAL SIMULATOR   |    return (                          |
| ├─ 📡 VehicleSpeed |      <div className="map-container"> |
| │   ├─ Current: 65km/h|     <canvas id="map-canvas"/>     |
| │   └─ [▶] [⏸] [⏹]|        {props.children}              |
| ├─ 📡 GpsLocation  |      </div>                          |
| │   └─ [▶] [⏸] [⏹]|    );                                |
| └─ [Open Scenario] |  }                                   |
|                    |                                      |
| PROBLEMS     CONSOLE  OUTPUT  DEBUG  TERMINAL             |
| > Building... Done                                        |
| > Running tests... 15 passed, 0 failed                    |
+-----------------------------------------------------------+
```

### 1.3 Signal Simulation Screen
```
+-----------------------------------------------------------+
|  PICCOLO Signal Simulator                           [user▼]|
+-----------------------------------------------------------+
|                                                           |
| [Dashboard] [Signal Definition] [Scenarios] [Settings]     |
|                                                           |
+-----------------------------------------------------------+
|                                                           |
|  ◆ Active Signal Simulation                               |
|                                                           |
|  Vehicle Signals                                         |
|  +---------------------------------------------------+    |
|  | Signal Name        | Value       | Control         |    |
|  |-------------------|-------------|----------------|    |
|  | Vehicle.Speed     | 65 km/h     | [▶] [⏸] [⏹]    |    |
|  | Vehicle.Gear      | DRIVE       | [▶] [⏸] [⏹]    |    |
|  | Vehicle.EngineRPM | 1800        | [▶] [⏸] [⏹]    |    |
|  | Vehicle.FuelLevel | 75%         | [▶] [⏸] [⏹]    |    |
|  | Add Signal...     |             |                |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Environment Signals                                      |
|  +---------------------------------------------------+    |
|  | Signal Name        | Value           | Control     |    |
|  |-------------------|-----------------|-------------|    |
|  | GPS.Latitude      | 37.4219983      | [▶] [⏸] [⏹]|    |
|  | GPS.Longitude     | -122.0840012    | [▶] [⏸] [⏹]|    |
|  | Environment.Temp  | 22°C            | [▶] [⏸] [⏹]|    |
|  | Environment.Light | DAYLIGHT        | [▶] [⏸] [⏹]|    |
|  +---------------------------------------------------+    |
|                                                           |
|  Loaded Scenario: Urban Driving Test                      |
|  +---------------------------------------------------+    |
|  | Progress: [=======================------] 70%      |    |
|  | Duration: 00:07:35 / 00:10:00                     |    |
|  +---------------------------------------------------+    |
|                                                           |
|  [  Stop Scenario  ]  [  Pause  ]  [  Export Results  ]   |
|                                                           |
+-----------------------------------------------------------+
```

### 1.4 CI/CD Pipeline Configuration Screen
```
+-----------------------------------------------------------+
|  PICCOLO Cloud CI/CD - Pipeline Configuration       [user▼]|
+-----------------------------------------------------------+
|                                                           |
| [Project Overview] [Code] [Pipelines] [Artifacts] [Tests] |
|                                                           |
+-----------------------------------------------------------+
|                                                           |
|  ◆ Pipeline: VEH-NAV-2025-main                           |
|                                                           |
|  Pipeline Stages                                          |
|  +---------------------------------------------------+    |
|  | [ Add Stage + ]                                   |    |
|  |                                                   |    |
|  | ┌──────────┐  ┌──────────┐  ┌──────────┐          |    |
|  | │   Build  │──│   Test   │──│ Security │          |    |
|  | └──────────┘  └──────────┘  └──────────┘          |    |
|  |        │                          │               |    |
|  |        │     ┌──────────┐  ┌──────────┐          |    |
|  |        └─────│ Package  │──│  Deploy  │          |    |
|  |              └──────────┘  └──────────┘          |    |
|  |                                                   |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Stage Configuration: Build                               |
|  +---------------------------------------------------+    |
|  | Runtime Environment:  [ Node.js 20.x       ▼ ]    |    |
|  | Commands:                                        |    |
|  | ┌─────────────────────────────────────────────┐  |    |
|  | │ npm install                                  │  |    |
|  | │ npm run build                                │  |    |
|  | └─────────────────────────────────────────────┘  |    |
|  | Artifacts to Cache: [ node_modules, dist      ]  |    |
|  | Timeout: [ 10 ] minutes                          |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Dependencies and Variables                               |
|  +---------------------------------------------------+    |
|  | Environment Variables:                             |    |
|  | [ Add Variable + ]                                |    |
|  |  ├─ NODE_ENV   | production        | [Delete]     |    |
|  |  └─ API_KEY    | ********          | [Delete]     |    |
|  |                                                   |    |
|  | Required Resources:                               |    |
|  |  CPU: [ 2 ] cores    Memory: [ 4 ] GB             |    |
|  +---------------------------------------------------+    |
|                                                           |
|  [  Cancel  ]  [  Save as Draft  ]  [  Save & Validate  ] |
|                                                           |
+-----------------------------------------------------------+
```

### 1.5 Test Report Dashboard
```
+-----------------------------------------------------------+
|  PICCOLO Cloud CI/CD - Test Reports                 [user▼]|
+-----------------------------------------------------------+
|                                                           |
| [Summary] [Unit Tests] [Integration Tests] [E2E Tests]    |
|                                                           |
+-----------------------------------------------------------+
|                                                           |
|  ◆ Test Summary: Build #127 (July 18, 2025)              |
|                                                           |
|  Overall Status: ✅ PASSED                                |
|  +---------------------------------------------------+    |
|  | Test Type      | Passed | Failed | Skipped | Total |    |
|  |----------------|--------|--------|---------|-------|    |
|  | Unit Tests     |  215   |   0    |   5     |  220  |    |
|  | Integration    |   42   |   0    |   2     |   44  |    |
|  | End-to-End     |   12   |   0    |   1     |   13  |    |
|  | Security Scans |   5    |   0    |   0     |   5   |    |
|  | Total          |  274   |   0    |   8     |  282  |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Code Coverage                                            |
|  +---------------------------------------------------+    |
|  | Module              | Coverage | Trend             |    |
|  |--------------------|---------|-------------------|    |
|  | Navigation Engine   | 92.5%   | ↑ 1.2% from last |    |
|  | UI Components       | 87.8%   | ↑ 0.5% from last |    |
|  | Data Services       | 94.3%   | ↓ 0.3% from last |    |
|  | Utilities           | 78.6%   | ↔ No change      |    |
|  | Overall             | 89.7%   | ↑ 0.6% from last |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Performance Tests                                        |
|  +---------------------------------------------------+    |
|  | Test Name         | Result  | Threshold | Status   |    |
|  |-------------------|---------|----------|----------|    |
|  | Map Rendering     | 62ms    | <100ms   | ✅ PASS  |    |
|  | Route Calculation | 245ms   | <300ms   | ✅ PASS  |    |
|  | App Startup       | 1.2s    | <1.5s    | ✅ PASS  |    |
|  | Memory Usage      | 128MB   | <150MB   | ✅ PASS  |    |
|  +---------------------------------------------------+    |
|                                                           |
|  [  Export Report  ]    [  Rerun Failed Tests  ]          |
|                                                           |
+-----------------------------------------------------------+
```

## 2. Deployment Stage UI

### 2.1 Vehicle OTA Update Management Screen
```
+-----------------------------------------------------------+
|  PICCOLO Cloud CI/CD - OTA Management              [user▼]|
+-----------------------------------------------------------+
|                                                           |
| [Updates] [Deployments] [Monitoring] [Configuration]      |
|                                                           |
+-----------------------------------------------------------+
|                                                           |
|  ◆ OTA Update Package Management                          |
|                                                           |
|  Update Packages                                          |
|  +---------------------------------------------------+    |
|  | [ Create New Update Package + ]                   |    |
|  |                                                   |    |
|  | Package ID         | Version  | Status     | Action |    |
|  |-------------------|----------|------------|--------|    |
|  | VEH-NAV-UPDATE-NA | 2.4.3    | Ready      | [Deploy]|    |
|  | VEH-NAV-UPDATE-EU | 2.4.3    | Ready      | [Deploy]|    |
|  | VEH-NAV-UPDATE-AS | 2.4.3    | Ready      | [Deploy]|    |
|  | VEH-NAV-HOTFIX    | 2.3.5-p2 | Deployed   | [View] |    |
|  | VEH-NAV-EMERGENCY | 2.3.5-e1 | Archived   | [View] |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Package Details: VEH-NAV-UPDATE-NA v2.4.3                |
|  +---------------------------------------------------+    |
|  | Target Vehicles:  4,238 vehicles                  |    |
|  | Update Size:      24.7 MB (Compressed: 18.2 MB)   |    |
|  | Components:                                       |    |
|  |  ├─ Navigation Engine   | v2.4.3  | Required     |    |
|  |  ├─ UI Components       | v1.8.1  | Required     |    |
|  |  ├─ Map Data North America | July 2025 | Optional |    |
|  |  └─ Voice Package English  | v3.2.0    | Optional |    |
|  |                                                   |    |
|  | Validation Status:                                |    |
|  |  ├─ Security Scan       | ✅ Passed              |    |
|  |  ├─ Certification       | ✅ Passed              |    |
|  |  ├─ Vehicle Compatibility | ✅ Compatible        |    |
|  |  └─ Rollback Support    | ✅ Available           |    |
|  +---------------------------------------------------+    |
|                                                           |
|  [  Edit Package  ]  [  Clone  ]  [  Deploy to Vehicles  ]|
|                                                           |
+-----------------------------------------------------------+
```

### 2.2 Vehicle Fleet Deployment Screen
```
+-----------------------------------------------------------+
|  PICCOLO Cloud CI/CD - Fleet Deployment            [user▼]|
+-----------------------------------------------------------+
|                                                           |
| [Campaign Overview] [Vehicles] [Progress] [Troubleshooting] |
|                                                           |
+-----------------------------------------------------------+
|                                                           |
|  ◆ Deployment Campaign: NAV-UPDATE-2025-Q3                |
|                                                           |
|  Campaign Status: ACTIVE                                  |
|  +---------------------------------------------------+    |
|  | Package:       VEH-NAV-UPDATE-NA v2.4.3           |    |
|  | Start Date:    2025-07-15 00:00 UTC               |    |
|  | End Date:      2025-08-15 00:00 UTC               |    |
|  | Deployment Strategy: Phased Rollout (10% daily)   |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Deployment Progress                                      |
|  +---------------------------------------------------+    |
|  | [======================---------------] 51%        |    |
|  | Total Vehicles:    4,238                          |    |
|  | Updated:           2,162 (51%)                    |    |
|  | Pending:           1,985 (47%)                    |    |
|  | Failed:            91 (2%)                        |    |
|  |                                                   |    |
|  | Regions:                                          |    |
|  |  ├─ Northeast  | 87% Complete | 523/602 vehicles |    |
|  |  ├─ Southeast  | 62% Complete | 489/789 vehicles |    |
|  |  ├─ Midwest    | 43% Complete | 431/1002 vehicles|    |
|  |  ├─ Southwest  | 38% Complete | 304/800 vehicles |    |
|  |  └─ West       | 39% Complete | 415/1045 vehicles|    |
|  +---------------------------------------------------+    |
|                                                           |
|  Analytics                                                |
|  +---------------------------------------------------+    |
|  | Average Download Time: 3.2 minutes                |    |
|  | Average Install Time:  5.8 minutes                |    |
|  | Success Rate:          98%                        |    |
|  | Rollback Rate:         0.4%                       |    |
|  | Connection Issues:     1.2%                       |    |
|  +---------------------------------------------------+    |
|                                                           |
|  [  Pause Campaign  ]  [  Increase Rate  ]  [  Reports  ] |
|                                                           |
+-----------------------------------------------------------+
```

### 2.3 Vehicle Monitoring Dashboard
```
+-----------------------------------------------------------+
|  PICCOLO Cloud CI/CD - Vehicle Monitoring          [user▼]|
+-----------------------------------------------------------+
|                                                           |
| [Fleet Overview] [Health Metrics] [Updates] [Diagnostics] |
|                                                           |
+-----------------------------------------------------------+
|                                                           |
|  ◆ Fleet Health Dashboard                                 |
|                                                           |
|  Fleet Overview                                           |
|  +---------------------------------------------------+    |
|  | Total Vehicles: 27,542                            |    |
|  | Online: 22,863 (83%)   Offline: 4,679 (17%)       |    |
|  |                                                   |    |
|  | Health Status:                                    |    |
|  | [========================] 91% Healthy            |    |
|  | [=====] 5% Warning       [==] 3% Critical         |    |
|  | [=] 1% Update Required                            |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Active Alerts                                            |
|  +---------------------------------------------------+    |
|  | Priority | Count | Type                  | Action  |    |
|  |----------|-------|----------------------|---------|    |
|  | 🔴 HIGH  |  23   | Memory Leak Detected | [View]  |    |
|  | 🔴 HIGH  |  12   | Storage Full         | [View]  |    |
|  | 🟠 MED   |  58   | CPU Throttling       | [View]  |    |
|  | 🟠 MED   |  72   | Update Failed        | [View]  |    |
|  | 🟡 LOW   | 127   | Signal Disruption    | [View]  |    |
|  +---------------------------------------------------+    |
|                                                           |
|  System Health by Component                               |
|  +---------------------------------------------------+    |
|  | Component       | Health | Affected Vehicles      |    |
|  |----------------|--------|------------------------|    |
|  | Navigation      | 97.8%  | 607 vehicles affected  |    |
|  | Connectivity    | 96.3%  | 1,019 vehicles affected|    |
|  | Media System    | 99.1%  | 248 vehicles affected  |    |
|  | Instrument Panel| 98.7%  | 358 vehicles affected  |    |
|  | Voice Control   | 94.2%  | 1,597 vehicles affected|    |
|  +---------------------------------------------------+    |
|                                                           |
|  [  Export Report  ]  [  Schedule Diagnostics  ]          |
|                                                           |
+-----------------------------------------------------------+
```

## 3. Integration UI

### 3.1 API Explorer Screen
```
+-----------------------------------------------------------+
|  PICCOLO Cloud API Explorer                         [user▼]|
+-----------------------------------------------------------+
|                                                           |
| [Endpoints] [Documentation] [Test Console] [My Apps]      |
|                                                           |
+-----------------------------------------------------------+
|                                                           |
|  ◆ API Endpoints                                          |
|                                                           |
|  Select API Category                                      |
|  [Vehicle APIs                                      ▼]    |
|                                                           |
|  Endpoints                                                |
|  +---------------------------------------------------+    |
|  | Method | Endpoint                 | Description    |    |
|  |--------|--------------------------|----------------|    |
|  | GET    | /api/v1/vehicles         | List vehicles  |    |
|  | GET    | /api/v1/vehicles/{id}    | Vehicle details|    |
|  | POST   | /api/v1/vehicles/command | Send command   |    |
|  | GET    | /api/v1/vehicles/{id}/status | Get status |    |
|  | GET    | /api/v1/vehicles/{id}/telemetry | Telemetry|    |
|  +---------------------------------------------------+    |
|                                                           |
|  Endpoint Details: GET /api/v1/vehicles/{id}              |
|  +---------------------------------------------------+    |
|  | Description: Retrieve details for a specific vehicle|    |
|  |                                                   |    |
|  | Path Parameters:                                  |    |
|  | - id: Vehicle identifier (string, required)       |    |
|  |                                                   |    |
|  | Query Parameters:                                 |    |
|  | - include: Fields to include (string, optional)   |    |
|  |   Example: "status,location,telemetry"            |    |
|  |                                                   |    |
|  | Headers:                                          |    |
|  | - Authorization: Bearer token (required)          |    |
|  |                                                   |    |
|  | Response Format:                                  |    |
|  | Content-Type: application/json                    |    |
|  |                                                   |    |
|  | Rate Limiting:                                    |    |
|  | - 100 requests per minute                         |    |
|  |                                                   |    |
|  | Required Permissions:                             |    |
|  | - vehicles:read                                   |    |
|  +---------------------------------------------------+    |
|                                                           |
|  [  Documentation  ]  [  Try It  ]  [  SDK Example  ]     |
|                                                           |
+-----------------------------------------------------------+
```

### 3.2 Developer Portal - Application Registration
```
+-----------------------------------------------------------+
|  PICCOLO Developer Portal                           [user▼]|
+-----------------------------------------------------------+
|                                                           |
| [Dashboard] [Applications] [API Keys] [Documentation]     |
|                                                           |
+-----------------------------------------------------------+
|                                                           |
|  ◆ Register New Application                               |
|                                                           |
|  Application Information                                  |
|  +---------------------------------------------------+    |
|  | Application Name* | [Fleet Management Dashboard  ] |    |
|  | Description*     | [Real-time fleet monitoring and] |    |
|  |                  | [management application       ] |    |
|  | Website URL      | [https://fleet.example.com    ] |    |
|  | Callback URLs*   | [https://fleet.example.com/auth] |    |
|  |                  | [https://localhost:3000/auth  ] |    |
|  +---------------------------------------------------+    |
|                                                           |
|  API Access Requirements                                  |
|  +---------------------------------------------------+    |
|  | Select API Categories:                             |    |
|  | [✓] Vehicle Status APIs                           |    |
|  | [✓] Vehicle Command APIs                          |    |
|  | [✓] Telemetry APIs                                |    |
|  | [ ] Diagnostic APIs                               |    |
|  | [ ] Firmware Management APIs                      |    |
|  |                                                   |    |
|  | Explain why your app needs these permissions:     |    |
|  | [Our fleet management application requires     ] |    |
|  | [these permissions to monitor vehicle status,  ] |    |
|  | [collect telemetry data for analysis, and send ] |    |
|  | [commands for remote operations.              ] |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Security Configuration                                   |
|  +---------------------------------------------------+    |
|  | Authentication Type:                               |    |
|  | ( ) API Key Only                                  |    |
|  | (●) OAuth 2.0                                     |    |
|  | ( ) Client Certificate                            |    |
|  |                                                   |    |
|  | Request Rate Limit:                               |    |
|  | [Standard (100 req/min)                     ▼]    |    |
|  +---------------------------------------------------+    |
|                                                           |
|  [  Cancel  ]                         [  Register App  ]  |
|                                                           |
+-----------------------------------------------------------+
```

### 3.3 SDK Integration Examples Screen
```
+-----------------------------------------------------------+
|  PICCOLO Developer Portal - SDK Examples            [user▼]|
+-----------------------------------------------------------+
|                                                           |
| [SDKs] [Examples] [Tutorials] [Reference]                 |
|                                                           |
+-----------------------------------------------------------+
|                                                           |
|  ◆ SDK Code Examples                                      |
|                                                           |
|  Select Language/Platform:                                |
|  [ JavaScript  ▼]                                         |
|                                                           |
|  Example: Vehicle Status Monitoring                       |
|  +---------------------------------------------------+    |
|  | // Initialize PICCOLO SDK with your credentials   |    |
|  | const piccolo = new PICCOLO.Client({              |    |
|  |   clientId: 'YOUR_CLIENT_ID',                     |    |
|  |   clientSecret: 'YOUR_CLIENT_SECRET',             |    |
|  |   region: 'na' // North America region            |    |
|  | });                                               |    |
|  |                                                   |    |
|  | // Authenticate your application                  |    |
|  | await piccolo.authenticate();                     |    |
|  |                                                   |    |
|  | // Get list of vehicles                           |    |
|  | const vehicles = await piccolo.vehicles.list({    |    |
|  |   limit: 10,                                      |    |
|  |   status: 'online'                                |    |
|  | });                                               |    |
|  |                                                   |    |
|  | // Subscribe to real-time vehicle status updates  |    |
|  | const subscription = piccolo.vehicles.subscribe(  |    |
|  |   'VIN123456789',                                 |    |
|  |   ['location', 'speed', 'batteryLevel'],          |    |
|  |   (data) => {                                     |    |
|  |     console.log('Vehicle update:', data);         |    |
|  |     // Update your UI with the new data           |    |
|  |     updateDashboard(data);                        |    |
|  |   }                                               |    |
|  | );                                                |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Available Examples:                                      |
|  +---------------------------------------------------+    |
|  | ● Vehicle Status Monitoring                        |    |
|  | ○ Send Commands to Vehicle                        |    |
|  | ○ Historical Telemetry Analysis                   |    |
|  | ○ Vehicle Geofencing                              |    |
|  | ○ Battery Management                              |    |
|  | ○ User Authentication and Profiles                |    |
|  +---------------------------------------------------+    |
|                                                           |
|  [  Copy Code  ]  [  Download Full Example  ]            |
|                                                           |
+-----------------------------------------------------------+
```

### 3.4 Integration Sandbox Environment
```
+-----------------------------------------------------------+
|  PICCOLO Integration Sandbox                        [user▼]|
+-----------------------------------------------------------+
|                                                           |
| [Dashboard] [Virtual Vehicles] [API Console] [Logs]       |
|                                                           |
+-----------------------------------------------------------+
|                                                           |
|  ◆ Virtual Vehicle Fleet                                  |
|                                                           |
|  Active Virtual Vehicles                                  |
|  +---------------------------------------------------+    |
|  | [ Create Virtual Vehicle + ]                      |    |
|  |                                                   |    |
|  | ID           | Model    | Status  | Action         |    |
|  |--------------|----------|---------|---------------|    |
|  | VIRT-VEH-001 | Model A  | Online  | [Control] [Log]|    |
|  | VIRT-VEH-002 | Model A  | Online  | [Control] [Log]|    |
|  | VIRT-VEH-003 | Model B  | Offline | [Control] [Log]|    |
|  | VIRT-VEH-004 | Model C  | Online  | [Control] [Log]|    |
|  | VIRT-VEH-005 | Model D  | Sleep   | [Control] [Log]|    |
|  +---------------------------------------------------+    |
|                                                           |
|  Virtual Vehicle Control: VIRT-VEH-001                    |
|  +---------------------------------------------------+    |
|  | State Control:                                    |    |
|  | Status: [Online  ▼]    Location: [Edit Coordinates]|    |
|  |                                                   |    |
|  | Signal Values:                                    |    |
|  | Speed:    [65    ] km/h                           |    |
|  | Battery:  [78    ] %                              |    |
|  | Charging: ( ) Yes (●) No                          |    |
|  | Doors:    ( ) Locked (●) Unlocked                 |    |
|  | Engine:   (●) Running ( ) Off                     |    |
|  |                                                   |    |
|  | Scenario Simulation:                              |    |
|  | [  Normal Operation  ▼]  [  Apply  ]              |    |
|  |                                                   |    |
|  | Response Configuration:                           |    |
|  | Command Delay: [ 300 ] ms                         |    |
|  | Error Rate:    [ 5   ] %                          |    |
|  +---------------------------------------------------+    |
|                                                           |
|  API Test Result                                          |
|  +---------------------------------------------------+    |
|  | > GET /api/v1/vehicles/VIRT-VEH-001               |    |
|  | < Status: 200 OK                                  |    |
|  | < Response Time: 312ms                            |    |
|  | < Body:                                           |    |
|  | {                                                 |    |
|  |   "id": "VIRT-VEH-001",                           |    |
|  |   "model": "Model A",                             |    |
|  |   "status": "online",                             |    |
|  |   "location": {                                   |    |
|  |     "latitude": 37.4219983,                       |    |
|  |     "longitude": -122.0840012                     |    |
|  |   },                                              |    |
|  |   "telemetry": {                                  |    |
|  |     "speed": 65,                                  |    |
|  |     "batteryLevel": 78,                           |    |
|  |     "charging": false                             |    |
|  |   }                                               |    |
|  | }                                                 |    |
|  +---------------------------------------------------+    |
|                                                           |
|  [  Reset Vehicle  ]  [  Export Logs  ]                  |
|                                                           |
+-----------------------------------------------------------+
```

## 4. Administration UI

### 4.1 User and Access Management Screen
```
+-----------------------------------------------------------+
|  PICCOLO Cloud Administration                       [admin▼]|
+-----------------------------------------------------------+
|                                                           |
| [Users] [Teams] [API Access] [Settings] [Audit]          |
|                                                           |
+-----------------------------------------------------------+
|                                                           |
|  ◆ User Management                                        |
|                                                           |
|  Users                                                    |
|  +---------------------------------------------------+    |
|  | [ Add User + ]  [ Import Users ]  [ Export ]      |    |
|  |                                                   |    |
|  | Username | Email          | Role      | Status    |    |
|  |----------|----------------|-----------|-----------|    |
|  | jdoe     | jdoe@lgesdv.com | Admin    | Active    |    |
|  | msmith   | msmith@lgesdv.com| Developer | Active   |    |
|  | rjohnson | rjohnson@lgesdv.com| Operator | Active |    |
|  | alee     | alee@lgesdv.com | Developer | Inactive |    |
|  | bwilson  | bwilson@lgesdv.com| Viewer  | Active   |    |
|  +---------------------------------------------------+    |
|                                                           |
|  User Details: msmith                                     |
|  +---------------------------------------------------+    |
|  | Name:    Maria Smith                              |    |
|  | Email:   msmith@lgesdv.com                         |    |
|  | Role:    Developer                                |    |
|  | Status:  Active                                   |    |
|  | Created: 2025-05-12                               |    |
|  | Last Login: 2025-07-17 14:23:45 UTC               |    |
|  | Two-Factor Auth: Enabled                          |    |
|  |                                                   |    |
|  | Team Memberships:                                 |    |
|  |  ├─ Navigation Team (Member)                      |    |
|  |  └─ Frontend Team (Leader)                        |    |
|  |                                                   |    |
|  | Permissions:                                      |    |
|  |  ├─ Projects: Create, Read, Update                |    |
|  |  ├─ Pipelines: Create, Read, Update, Execute      |    |
|  |  ├─ Code: Read, Write                             |    |
|  |  ├─ Deployments: Read                             |    |
|  |  └─ API Access: Read, Execute                     |    |
|  +---------------------------------------------------+    |
|                                                           |
|  [  Edit User  ]  [  Reset Password  ]  [  Deactivate  ]  |
|                                                           |
+-----------------------------------------------------------+
```

### 4.2 Audit Log and Compliance Screen
```
+-----------------------------------------------------------+
|  PICCOLO Cloud Administration - Audit Log           [admin▼]|
+-----------------------------------------------------------+
|                                                           |
| [System Audit] [Security Events] [Compliance] [Reports]   |
|                                                           |
+-----------------------------------------------------------+
|                                                           |
|  ◆ Audit Log                                              |
|                                                           |
|  Filters                                                  |
|  +---------------------------------------------------+    |
|  | Date Range: [Last 7 Days        ▼]                |    |
|  | Event Type: [All                ▼]                |    |
|  | User:       [All Users          ▼]                |    |
|  | Severity:   [All                ▼]                |    |
|  |                                                   |    |
|  | [  Apply Filters  ]  [  Reset  ]  [  Save Filter  ] |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Audit Events                                             |
|  +---------------------------------------------------+    |
|  | Time              | User   | Event          | Details |    |
|  |-------------------|--------|----------------|---------|    |
|  | 2025-07-18 00:12:32| jdoe   | Login          | Success |    |
|  | 2025-07-18 00:15:47| jdoe   | Project Create | VEH-NAV-2025 |    |
|  | 2025-07-18 00:23:09| jdoe   | API Key Create | api-key-nav-123 |    |
|  | 2025-07-18 00:42:15| msmith | Code Commit    | feat: add map renderer |    |
|  | 2025-07-18 01:03:22| system | Pipeline Run   | Build #127 |    |
|  | 2025-07-18 01:15:33| rjohnson | Deployment   | NAV-UPDATE-2025-Q3 |    |
|  | 2025-07-18 01:18:02| alee   | Login Failed   | Invalid credentials |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Event Details: Pipeline Run - Build #127                 |
|  +---------------------------------------------------+    |
|  | Timestamp: 2025-07-18 01:03:22 UTC                |    |
|  | Event Type: Pipeline Run                          |    |
|  | Initiated By: system (Scheduled Trigger)          |    |
|  | Project: VEH-NAV-2025                             |    |
|  | Pipeline: main                                    |    |
|  | Build Number: 127                                 |    |
|  | Status: Success                                   |    |
|  | Duration: 12m 37s                                 |    |
|  |                                                   |    |
|  | Event Context:                                    |    |
|  | - Commit: 7a8d9f3 (feat: add map renderer)        |    |
|  | - Branch: main                                    |    |
|  | - Repository: github.com/lgesdv/nav-system        |    |
|  | - Runner: cloud-runner-03                         |    |
|  | - Environment: production                         |    |
|  +---------------------------------------------------+    |
|                                                           |
|  [  Export Log  ]  [  View Related Events  ]              |
|                                                           |
+-----------------------------------------------------------+
```

### 4.3 System Monitoring and Health Screen
```
+-----------------------------------------------------------+
|  PICCOLO Cloud Administration - System Health       [admin▼]|
+-----------------------------------------------------------+
|                                                           |
| [Overview] [Services] [Resources] [Alerts] [Maintenance]  |
|                                                           |
+-----------------------------------------------------------+
|                                                           |
|  ◆ System Health Overview                                 |
|                                                           |
|  Global Status: ✅ Operational                            |
|  +---------------------------------------------------+    |
|  | Service           | Status       | Uptime          |    |
|  |-------------------|--------------|----------------|    |
|  | API Services      | ✅ Operational| 99.998% (30d)  |    |
|  | CI/CD Pipelines   | ✅ Operational| 99.987% (30d)  |    |
|  | Deployment Services| ✅ Operational| 99.992% (30d) |    |
|  | Data Storage      | ✅ Operational| 100.000% (30d) |    |
|  | User Management   | ✅ Operational| 99.999% (30d)  |    |
|  | Vehicle Communication| ✅ Operational| 99.978% (30d)|    |
|  +---------------------------------------------------+    |
|                                                           |
|  Resource Utilization                                     |
|  +---------------------------------------------------+    |
|  | Resource          | Current    | Peak (24h)        |    |
|  |-------------------|------------|------------------|    |
|  | CPU Utilization   | 42%        | 78%              |    |
|  | Memory Usage      | 58%        | 86%              |    |
|  | Storage Utilization| 62%       | 63%              |    |
|  | Network Throughput| 4.7 Gbps   | 8.2 Gbps         |    |
|  | Database Connections| 1,247    | 2,835            |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Recent Incidents                                         |
|  +---------------------------------------------------+    |
|  | ID      | Time        | Description        | Status |    |
|  |---------|-------------|-------------------|--------|    |
|  | INC-427 | 2025-07-16  | API Rate Limiting | Resolved|    |
|  | INC-412 | 2025-07-10  | Database Latency  | Resolved|    |
|  | INC-398 | 2025-07-02  | Storage I/O       | Resolved|    |
|  +---------------------------------------------------+    |
|                                                           |
|  Scheduled Maintenance                                    |
|  +---------------------------------------------------+    |
|  | ID      | Planned     | Description        | Impact |    |
|  |---------|-------------|-------------------|--------|    |
|  | MNT-089 | 2025-07-21  | Database Upgrade  | Minimal|    |
|  | MNT-092 | 2025-07-28  | Security Patches  | None   |    |
|  +---------------------------------------------------+    |
|                                                           |
|  [  System Report  ]  [  Schedule Maintenance  ]          |
|                                                           |
+-----------------------------------------------------------+
```

## 5. Vehicle-Cloud Configuration UI

### 5.1 Vehicle Connectivity Setup Screen
```
+-----------------------------------------------------------+
|  PICCOLO Cloud - Vehicle Connectivity Configuration [admin▼]|
+-----------------------------------------------------------+
|                                                           |
| [Connectivity] [Security] [Policies] [Registration]       |
|                                                           |
+-----------------------------------------------------------+
|                                                           |
|  ◆ Vehicle Connectivity Configuration                     |
|                                                           |
|  Connection Settings                                      |
|  +---------------------------------------------------+    |
|  | Primary Connection Protocol: [ MQTT          ▼]    |    |
|  | Secondary Connection Protocol:[ WebSocket     ▼]    |    |
|  | Connection Retry Strategy:   [ Exponential   ▼]    |    |
|  | Max Retry Interval:          [ 60           ] sec  |    |
|  | Keep-Alive Interval:         [ 30           ] sec  |    |
|  | Data Compression:            [✓] Enabled           |    |
|  | Compression Algorithm:       [ ZLIB         ▼]     |    |
|  | Offline Data Storage:        [ Up to 7 days ▼]     |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Endpoint Configuration                                   |
|  +---------------------------------------------------+    |
|  | Region       | Primary Endpoint            | Status |    |
|  |--------------|----------------------------|--------|    |
|  | North America| mqtt.na.piccolo.sdv.org:8883| ✅ Active|    |
|  | Europe       | mqtt.eu.piccolo.sdv.org:8883| ✅ Active|    |
|  | Asia Pacific | mqtt.ap.piccolo.sdv.org:8883| ✅ Active|    |
|  | [ Add Endpoint + ]                                |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Data Transfer Policies                                   |
|  +---------------------------------------------------+    |
|  | [ Add Policy + ]                                  |    |
|  |                                                   |    |
|  | Policy Name   | Description             | Status   |    |
|  |--------------|-------------------------|----------|    |
|  | Telemetry    | Vehicle telemetry data  | Enabled  |    |
|  | Diagnostics  | System diagnostic data  | Enabled  |    |
|  | Logs         | Application logs        | Enabled  |    |
|  | Updates      | OTA update status       | Enabled  |    |
|  | Commands     | Vehicle commands        | Enabled  |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Policy Details: Telemetry                                |
|  +---------------------------------------------------+    |
|  | Upload Frequency:    [ 60        ] seconds        |    |
|  | Batch Size:          [ 100       ] records        |    |
|  | Priority:            [ High      ▼]               |    |
|  | Network Constraints: [✓] Cellular [✓] Wi-Fi       |    |
|  | Data Fields:                                      |    |
|  | [✓] Location      [✓] Speed       [✓] Battery     |    |
|  | [✓] Engine Status [✓] Fuel/Energy [✓] Temperature |    |
|  | [ ] Interior      [ ] Entertainment [✓] ADAS      |    |
|  +---------------------------------------------------+    |
|                                                           |
|  [  Cancel  ]  [  Test Configuration  ]  [  Save  ]       |
|                                                           |
+-----------------------------------------------------------+
```

### 5.2 Security and Certificate Management Screen
```
+-----------------------------------------------------------+
|  PICCOLO Cloud - Security Configuration            [admin▼]|
+-----------------------------------------------------------+
|                                                           |
| [Authentication] [Certificates] [Encryption] [Policies]   |
|                                                           |
+-----------------------------------------------------------+
|                                                           |
|  ◆ Certificate Management                                 |
|                                                           |
|  Root Certificates                                        |
|  +---------------------------------------------------+    |
|  | [ Import Certificate + ]  [ Generate New + ]      |    |
|  |                                                   |    |
|  | Certificate Name | Issuer     | Expiration  | Status |    |
|  |----------------|------------|------------|---------|    |
|  | PICCOLO Root CA | Self-signed | 2030-07-18 | Active |    |
|  | PICCOLO Dev CA  | PICCOLO Root| 2027-07-18 | Active |    |
|  | PICCOLO Prod CA | PICCOLO Root| 2027-07-18 | Active |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Vehicle Certificates                                     |
|  +---------------------------------------------------+    |
|  | [ Generate Batch + ]  [ Revoke Selected ]         |    |
|  |                                                   |    |
|  | Status    | Count   | Expiration Range           |    |
|  |-----------|---------|---------------------------|    |
|  | Active    | 27,542  | 2025-10-15 to 2026-07-18  |    |
|  | Reserved  | 5,000   | 2025-12-01 to 2026-12-01  |    |
|  | Revoked   | 83      | N/A                       |    |
|  | Expired   | 0       | N/A                       |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Certificate Template Configuration                       |
|  +---------------------------------------------------+    |
|  | Subject Format: CN={vin},O=PICCOLO SDV,C=US       |    |
|  | Key Type:       [ ECC P256      ▼]                |    |
|  | Validity Period:[ 365           ] days            |    |
|  | Key Usage:      [✓] Digital Signature             |    |
|  |                 [✓] Key Encipherment              |    |
|  |                 [ ] Data Encipherment             |    |
|  | Extended Usage: [✓] Client Authentication         |    |
|  |                 [ ] Server Authentication         |    |
|  |                 [ ] Code Signing                  |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Certificate Rotation Policy                             |
|  +---------------------------------------------------+    |
|  | Auto-Rotation:       [✓] Enabled                  |    |
|  | Rotation Trigger:    [ 30           ] days before |    |
|  |                      expiration                   |    |
|  | Overlap Period:      [ 7            ] days        |    |
|  | Max Attempts:        [ 3            ]             |    |
|  | Failure Notification:[ Email, Dashboard▼]          |    |
|  +---------------------------------------------------+    |
|                                                           |
|  [  Cancel  ]  [  Apply Changes  ]                       |
|                                                           |
+-----------------------------------------------------------+
```

### 5.3 Vehicle Registration and Provisioning Screen
```
+-----------------------------------------------------------+
|  PICCOLO Cloud - Vehicle Onboarding                 [admin▼]|
+-----------------------------------------------------------+
|                                                           |
| [Registration] [Provisioning] [Fleet Transfer] [Templates]|
|                                                           |
+-----------------------------------------------------------+
|                                                           |
|  ◆ Vehicle Registration                                   |
|                                                           |
|  Register New Vehicles                                    |
|  +---------------------------------------------------+    |
|  | Registration Method: [ Batch Upload        ▼]     |    |
|  |                                                   |    |
|  | Upload CSV File:     [ Choose File... ] vehicle_list.csv |    |
|  |                                                   |    |
|  | File Format: VIN,Model,Region,Hardware Version    |    |
|  |                                                   |    |
|  | Preview (first 5 rows):                           |    |
|  | 5YJSA1E21FF000001,Model A,NA,HW2.0                |    |
|  | 5YJSA1E25FF000002,Model A,NA,HW2.0                |    |
|  | WVWZZZ1KZAM000123,Model B,EU,HW3.0                |    |
|  | JH4NA21674S000098,Model C,AP,HW2.5                |    |
|  | WAUAF78E45A000456,Model B,NA,HW3.0                |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Registration Options                                     |
|  +---------------------------------------------------+    |
|  | Default Region:     [ Based on file      ▼]       |    |
|  | Auto-Provisioning: [✓] Enabled                    |    |
|  | Apply Template:    [ Standard Vehicle    ▼]        |    |
|  | Generate Certificates: [✓] Enabled                |    |
|  | Notification:      [✓] Email [✓] Dashboard        |    |
|  +---------------------------------------------------+    |
|                                                           |
|  Recent Registration Activities                           |
|  +---------------------------------------------------+    |
|  | Batch ID   | Date       | Count | Status           |    |
|  |------------|------------|-------|-----------------|    |
|  | REG-35782  | 2025-07-15 | 1,245 | Completed       |    |
|  | REG-35720  | 2025-07-10 | 2,890 | Completed       |    |
|  | REG-35645  | 2025-07-02 | 583   | Completed       |    |
|  +---------------------------------------------------+    |
|                                                           |
|  [  Cancel  ]  [  Validate  ]  [  Register Vehicles  ]    |
|                                                           |
+-----------------------------------------------------------+
```

## 6. Responsive Design Considerations

This UI is designed with responsiveness in mind, considering various device sizes:

### 6.1 Desktop (1920x1080 and above)
- Full feature visibility
- Multi-panel layouts
- Advanced visualization tools
- Comprehensive data tables

### 6.2 Laptop (1366x768)
- Slightly condensed layouts
- Some advanced visualizations simplified
- Tab-based navigation for secondary features
- Full functionality preserved

### 6.3 Tablet (768x1024)
- Simplified layouts with collapsible sections
- Primary features prioritized
- Modal dialogs for detailed views
- Touch-optimized controls

### 6.4 Mobile (360x640)
- Single column layout
- Hamburger menu for navigation
- Essential monitoring features only
- Limited editing capabilities
- Focus on status checking and alerts

## 7. Accessibility Considerations

The UI follows WCAG 2.1 AA guidelines with:

- High contrast mode support
- Screen reader compatibility
- Keyboard navigation
- Focus indicators
- Text scaling support (up to 200%)
- Alternative text for all interface elements
- Appropriate use of ARIA landmarks and roles

---

Document End
