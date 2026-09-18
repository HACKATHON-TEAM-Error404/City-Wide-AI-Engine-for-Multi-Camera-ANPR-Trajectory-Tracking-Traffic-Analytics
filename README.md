# 🚦 City-Wide AI Engine for Multi-Camera ANPR Trajectory Tracking & Urban Traffic Analytics

> **Smart India Hackathon 2026 --- Problem Statement SIH26127**\
> **Theme:** Smart Automation\
> **Category:** Software\
> **Team ID:** 133312\
> **Team:** Error 404!!

------------------------------------------------------------------------

## 📌 Overview

The **City-Wide AI Engine for Multi-Camera ANPR Trajectory Tracking and
Urban Traffic Analytics** is a centralized AI-powered software platform
designed to connect geographically distributed CCTV/ANPR camera feeds
into a unified spatial-temporal intelligence system.

Modern cities deploy large networks of CCTV and Automatic Number Plate
Recognition (ANPR) cameras. However, many existing deployments operate
as isolated systems, making it difficult to correlate observations from
different locations and extract city-wide traffic movement insights.

This project addresses that gap by combining:

-   🎥 Multi-camera RTSP video ingestion
-   🚗 Vehicle detection and localization
-   🔎 License-plate region extraction
-   🧠 AI-based OCR
-   🗺️ Cross-camera trajectory reconstruction
-   ⏱️ Timestamp-based spatial-temporal correlation
-   📍 Geospatial validation
-   📊 City-wide traffic analytics
-   🚨 Real-time vehicle alerts
-   🗺️ Interactive GIS visualization

The proposed system targets **greater than 90% OCR accuracy** and is
designed to work with existing CCTV/ANPR infrastructure with minimal
hardware changes.

------------------------------------------------------------------------

## 🎯 Problem Statement

**Problem Statement ID:** `SIH26127`

**Title:**\
`City-Wide AI Engine for Multi-Camera ANPR Trajectory Tracking and Urban Traffic Analytics`

The problem statement identifies three primary requirements:

### 1. 🔤 High-Accuracy ANPR & OCR

The system should recognize vehicle license plates with greater than 90%
accuracy across challenging real-world conditions, including:

-   Variable lighting
-   Poor weather
-   Angled camera views
-   Motion blur
-   Dirty plates
-   Damaged plates

### 2. 🛣️ Single-Plate Trajectory Tracking

The system should reconstruct the movement history of a specific vehicle
across multiple geographically distributed ANPR cameras.

The reconstructed trajectory should contain:

-   Vehicle/license-plate observation
-   Timestamp
-   Camera location
-   Direction
-   Route history

### 3. 📈 Macro Traffic Flow & Movement Analytics

Aggregated camera data should provide city-wide traffic intelligence
such as:

-   Traffic density
-   Origin-destination patterns
-   Route densities
-   Average vehicle speeds
-   Congestion bottlenecks
-   Traffic movement trends
-   Real-time heatmaps

------------------------------------------------------------------------

## 💡 Proposed Solution

The proposed platform creates a centralized pipeline that transforms
independent camera feeds into a connected city-wide traffic intelligence
layer.

``` text
🎥 CCTV / ANPR Cameras
          │
          ▼
📡 Multi-RTSP Stream Capture
          │
          ▼
🚗 Vehicle Detection
          │
          ▼
🔎 Plate Region Extraction
          │
          ▼
🖼️ Image Preprocessing
          │
          ▼
🧠 OCR / Plate Recognition
          │
          ▼
🗄️ PostgreSQL + PostGIS
          │
          ▼
⏱️ Spatial-Temporal Matching
          │
          ├───────────────┐
          ▼               ▼
🛣️ Trajectory        📊 Traffic Analytics
 Reconstruction
          │               │
          └───────┬───────┘
                  ▼
          🗺️ GIS Web Dashboard
                  │
                  ▼
             🚨 Alerts
```

------------------------------------------------------------------------

# 🏗️ System Architecture

## 🎥 1. Multi-RTSP Stream Capture

The platform receives video streams from geographically distributed CCTV
cameras through RTSP pipelines.

The proposed implementation uses **OpenCV/RTSP streaming pipelines** for
distributed video ingestion.

### Responsibilities

-   Connect to multiple camera streams
-   Capture frames
-   Maintain camera identifiers
-   Associate observations with timestamps
-   Feed frames into the computer-vision pipeline

------------------------------------------------------------------------

## 🚗 2. Vehicle Detection

**YOLOv8** is proposed for vehicle detection and localization.

The detection layer is intended to identify vehicle classes such as:

-   🚙 Cars
-   🚚 Trucks
-   🏍️ Motorcycles

The detected vehicle regions are then passed to the license-plate
processing pipeline.

------------------------------------------------------------------------

## 🔎 3. License Plate Extraction & OCR

After vehicle detection, the system extracts the license-plate region.

The proposed preprocessing pipeline includes **CLAHE** to improve image
quality before OCR.

**PaddleOCR** is proposed for character recognition.

### OCR Pipeline

``` text
Vehicle Frame
     ↓
Vehicle Detection
     ↓
Plate Region Extraction
     ↓
CLAHE Preprocessing
     ↓
PaddleOCR
     ↓
Recognized Plate
     ↓
Confidence / Validation
```

### 🎯 Target

> **Greater than 90% OCR accuracy**

The target is intended to cover diverse real-world conditions including
lighting variation, weather, angled shots, motion blur, and dirty or
damaged plates.

------------------------------------------------------------------------

# 🧭 4. Spatial-Temporal Trajectory Engine

The trajectory engine is the core component that connects observations
from different cameras.

Instead of treating every ANPR detection independently, the system uses
available spatial and temporal information to determine whether
observations can represent a plausible sequential movement.

### 🔗 Matching Signals

The proposed approach considers:

-   🔤 Plate matching
-   ⏱️ Timestamp ordering
-   📍 Camera geolocation
-   📏 Geospatial distance
-   🚘 Estimated speed
-   🧭 Direction
-   🛣️ Plausible sequential movement

### Example

``` text
Camera A
09:10:15
Vehicle: MHXX1234
      │
      │ Spatial + Temporal Validation
      ▼
Camera B
09:14:42
Vehicle: MHXX1234
      │
      │ Spatial + Temporal Validation
      ▼
Camera C
09:19:08
Vehicle: MHXX1234
```

The resulting observations form a chronological vehicle trajectory.

------------------------------------------------------------------------

# 🗺️ 5. GIS Web Dashboard

The platform includes a centralized GIS-integrated web dashboard.

The proposed frontend technology is:

-   ⚛️ React
-   🗺️ Mapbox GL

### Dashboard Capabilities

#### 🚘 Vehicle Trajectory View

Authorized operators can query a vehicle plate and visualize its
historical movement across the camera network.

#### 🔥 Traffic Heatmaps

Visualize aggregated traffic activity across the city.

#### 📊 Traffic Analytics

Display:

-   Traffic density
-   Average vehicle speeds
-   Route density
-   Origin-destination patterns
-   Traffic flow trends
-   Congestion areas

#### 📍 Camera Network

Display camera nodes and associated geographic information.

------------------------------------------------------------------------

# 🚨 6. Alert System

The platform includes an alert mechanism for monitored vehicles.

The proposed system can generate alerts when:

-   A blacklisted/monitored plate is detected
-   A relevant vehicle observation occurs
-   A suspicious route anomaly is identified

The alert layer is intended to support faster operational monitoring and
response.

> **Note:** Any deployment involving vehicle identification and
> monitoring should be operated by authorized personnel and according to
> applicable laws, policies, access controls, and data-governance
> requirements.

------------------------------------------------------------------------

# 🗄️ Data Layer

## PostgreSQL + PostGIS

Vehicle observations are proposed to be stored in **PostgreSQL with
PostGIS**.

Each observation can associate relevant information such as:

  Data                      Purpose
  ------------------------- ------------------------------
  🔤 Plate                  Vehicle identification value
  ⏱️ Timestamp              Temporal ordering
  📍 Latitude / Longitude   Geographic position
  📷 Camera ID              Observation source
  🚗 Vehicle information    Detection context
  🧠 OCR result             Recognized plate information

PostGIS enables spatial operations needed for geographic validation and
mapping.

------------------------------------------------------------------------

# 🧠 Core Processing Pipeline

``` text
┌───────────────────────────┐
│ 🎥 Multiple CCTV Streams  │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│ 📡 RTSP / OpenCV Ingestion │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│ 🚗 YOLOv8 Vehicle Detect. │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│ 🔎 Plate Region Extraction │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│ 🖼️ CLAHE Preprocessing     │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│ 🧠 PaddleOCR Recognition   │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│ 🗄️ PostgreSQL + PostGIS    │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│ 🧭 Spatial-Temporal Engine │
└─────────────┬─────────────┘
              │
       ┌──────┴───────┐
       ▼              ▼
┌─────────────┐ ┌──────────────┐
│ 🛣️ Trajectory│ │ 📊 Analytics │
└──────┬──────┘ └──────┬───────┘
       │               │
       └───────┬───────┘
               ▼
      ┌──────────────────┐
      │ 🗺️ React + Mapbox │
      └────────┬─────────┘
               │
               ▼
          🚨 Alerts
```

------------------------------------------------------------------------

# 🧰 Technology Stack

  Layer                    Proposed Technology
  ------------------------ ------------------------------------------
  🎥 Video Ingestion       OpenCV + RTSP
  🚗 Vehicle Detection     YOLOv8
  🔎 Image Preprocessing   CLAHE
  🔤 OCR                   PaddleOCR
  🗄️ Database              PostgreSQL
  📍 Spatial Database      PostGIS
  ⚛️ Frontend              React
  🗺️ GIS Visualization     Mapbox GL
  🧭 Trajectory Logic      Spatial-temporal + geospatial validation

------------------------------------------------------------------------

# ✨ Key Features

### 🚘 Vehicle Intelligence

-   Multi-camera vehicle detection
-   License plate recognition
-   Plate-based vehicle observation records
-   Cross-camera movement correlation

### 🛣️ Trajectory Tracking

-   Chronological vehicle history
-   Camera-to-camera movement
-   Timestamp linking
-   Geographic validation
-   Route visualization

### 📊 Urban Traffic Analytics

-   City-wide traffic density
-   Origin-destination insights
-   Route density
-   Average vehicle speed
-   Congestion bottleneck analysis
-   Traffic heatmaps

### 🚨 Operational Alerts

-   Blacklisted/monitored plate alerts
-   Suspicious route anomaly alerts
-   Real-time monitoring support

### 🏙️ Infrastructure Integration

-   Multi-camera support
-   Distributed processing
-   Reuse of existing CCTV/ANPR infrastructure
-   Centralized monitoring

------------------------------------------------------------------------

# 📈 Expected Impact

The proposed platform is intended to address several limitations of
isolated CCTV/ANPR systems.

  -----------------------------------------------------------------------
  Existing Challenge                  Proposed Capability
  ----------------------------------- -----------------------------------
  🔒 Isolated camera feeds            🔗 Centralized camera integration

  🔍 Manual cross-camera search       🧭 Automated trajectory
                                      reconstruction

  📍 Limited movement context         🗺️ Spatial-temporal vehicle history

  📊 Fragmented traffic data          📈 City-wide traffic analytics

  ⏳ Slow monitoring                  🚨 Automated alert generation

  🏗️ Expensive infrastructure changes ♻️ Reuse of existing CCTV
                                      infrastructure
  -----------------------------------------------------------------------

The submitted concept identifies faster cross-camera tracking, connected
CCTV feeds, traffic-flow insights, timestamped verification, alert
generation, and reduced upgrade costs as key benefits.

------------------------------------------------------------------------

# 🚀 Scalability & Feasibility

The architecture is designed with distributed camera processing in mind.

### 📡 Distributed Processing

Multiple CCTV streams can be processed across distributed nodes.

### ⚡ Fast Processing

AI-based detection and OCR are intended to support rapid vehicle and
license-plate processing.

### ♻️ Infrastructure Reusability

The system is designed to integrate with existing CCTV/ANPR
infrastructure without major hardware changes.

### 🏢 Enterprise-Oriented Architecture

The centralized database, processing pipeline, GIS dashboard, and
alerting layer provide a foundation for city-wide deployment.

------------------------------------------------------------------------

# 🔐 Data Integrity & Responsible Deployment

The submitted concept emphasizes timestamped and geospatially validated
records.

For a real-world deployment, the platform should additionally enforce:

-   🔑 Role-based access control
-   🛡️ Secure authentication
-   📝 Audit logging
-   🔒 Encryption in transit and at rest
-   📜 Appropriate data-retention policies
-   👮 Authorized operational access
-   ⚖️ Applicable legal and regulatory requirements

These controls are important because ANPR systems process
vehicle-related information and can affect individuals when used for
enforcement or monitoring.

------------------------------------------------------------------------

# 🎯 Project Objectives

1.  🎥 Integrate multiple CCTV/ANPR streams into one platform.
2.  🚗 Detect vehicles across camera feeds.
3.  🔤 Achieve the proposed \>90% OCR accuracy target.
4.  🧭 Reconstruct vehicle trajectories across cameras.
5.  📍 Validate movements using time and geospatial information.
6.  📊 Generate city-wide traffic analytics.
7.  🗺️ Provide GIS-based visualization.
8.  🚨 Generate relevant real-time alerts.
9.  ♻️ Reuse existing CCTV infrastructure.
10. 📈 Provide a scalable foundation for city-wide deployment.

------------------------------------------------------------------------

# 🧪 Prototype Demonstration Flow

A practical demonstration can follow this sequence:

``` text
1️⃣ Start multiple simulated/live CCTV streams
        ↓
2️⃣ Detect vehicles using YOLOv8
        ↓
3️⃣ Extract license plates
        ↓
4️⃣ Apply CLAHE preprocessing
        ↓
5️⃣ Recognize plates using PaddleOCR
        ↓
6️⃣ Store observations in PostgreSQL/PostGIS
        ↓
7️⃣ Correlate observations across cameras
        ↓
8️⃣ Build the vehicle trajectory
        ↓
9️⃣ Display route on the GIS dashboard
        ↓
🔟 Generate traffic analytics / alerts
```

------------------------------------------------------------------------

# 🏆 Smart India Hackathon Context

**Problem Statement:** `SIH26127`

**Problem:**\
City-Wide AI Engine for Multi-Camera ANPR Trajectory Tracking and Urban
Traffic Analytics

**Theme:**\
Smart Automation

**Category:**\
Software

**Team ID:**\
`133312`

**Registered Team Name:**\
`Error 404!!`

------------------------------------------------------------------------

# 📚 Research & References

The supplied SIH submission contains a dedicated **Research and
References** section, but the uploaded document does not include
readable reference entries in that section.

Therefore, this README does **not invent or add external references**
that were not present in the supplied materials.

For the implementation, the technologies explicitly identified in the
supplied proposal are:

-   YOLOv8
-   PaddleOCR
-   OpenCV / RTSP
-   PostgreSQL
-   PostGIS
-   React
-   Mapbox GL

------------------------------------------------------------------------

# 🛣️ Future Scope

The supplied proposal establishes the core direction of the platform.
Potential implementation extensions can be developed around the same
architecture, including:

-   📡 Larger camera-network deployments
-   ⚡ More efficient distributed processing
-   🧠 Improved OCR robustness
-   🧭 More sophisticated trajectory validation
-   📊 Expanded traffic analytics
-   🗺️ Advanced GIS visualization
-   🔔 More configurable alert workflows
-   🔐 Stronger enterprise security and governance

------------------------------------------------------------------------

# 👥 Team

### Team ID: `133312`

### Team Name: `Error 404!!`

> 🚀 **Building smarter systems with technology.**

------------------------------------------------------------------------

# 📄 Source Basis

This README has been prepared from the supplied Smart India Hackathon
2026 problem-statement material and the team's submitted idea document.

**Official problem:** `SIH26127`\
**Solution focus:** Multi-camera ANPR + OCR + trajectory tracking +
urban traffic analytics

------------------------------------------------------------------------

## ⭐ Project Vision

> **Connect isolated city camera feeds, reconstruct vehicle movement
> across space and time, and transform camera observations into
> actionable urban traffic intelligence.**

------------------------------------------------------------------------
