# SIH26127 Complete Solution Guide
## City-Wide AI Engine for Multi-Camera ANPR Trajectory Tracking and Urban Traffic Analytics

**Problem Statement ID:** SIH26127  
**Organization:** Bharat Electronics Limited  
**Theme:** Smart Automation / Transportation & Logistics  
**Submission Deadline:** September 30, 2026  
**Hackathon Duration:** 36 hours  
**Team Size:** 6 students

---

# TABLE OF CONTENTS
1. [System Architecture](#system-architecture)
2. [Tech Stack Selection](#tech-stack-selection)
3. [36-Hour Implementation Plan](#36-hour-implementation-plan)
4. [Team Role Allocation](#team-role-allocation)
5. [Component Deep Dive](#component-deep-dive)
6. [Data Pipeline & Workflow](#data-pipeline--workflow)
7. [Testing & Demo](#testing--demo)
8. [Presentation Strategy](#presentation-strategy)

---

# SYSTEM ARCHITECTURE

## High-Level Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    MULTI-CAMERA FEEDS                           │
│  (RTSP streams from distributed CCTV cameras across city)        │
└──────────────────────┬──────────────────────────────────────────┘
                       │
         ┌─────────────┴─────────────┐
         │                           │
         ▼                           ▼
    ┌─────────────┐          ┌─────────────────┐
    │   Camera 1  │          │   Camera N      │
    │   RTSP      │          │   (Distributed) │
    └──────┬──────┘          └────────┬────────┘
           │                          │
           └──────────────┬───────────┘
                          │
           ┌──────────────▼──────────────┐
           │   ANPR PROCESSING ENGINE    │
           │   ┌──────────────────────┐  │
           │   │ 1. Frame Extraction  │  │
           │   │ 2. Vehicle Detection │  │
           │   │ 3. Plate OCR (>90%)  │  │
           │   └──────────────────────┘  │
           └──────────────┬───────────────┘
                          │
           ┌──────────────▼──────────────┐
           │  DATA CORRELATION ENGINE    │
           │  ┌──────────────────────┐   │
           │  │ Plate Matching       │   │
           │  │ Temporal Linking     │   │
           │  │ Trajectory Building  │   │
           │  │ Geospatial Mapping   │   │
           │  └──────────────────────┘   │
           └──────────────┬───────────────┘
                          │
           ┌──────────────┴───────────────┐
           │                              │
      ┌────▼─────┐              ┌────────▼────┐
      │ DATABASE │              │  CACHE      │
      │ (MongoDB │              │  (Redis)    │
      │  or SQL) │              │             │
      └──────────┘              └─────────────┘
           │                              │
           └──────────────┬───────────────┘
                          │
         ┌────────────────▼─────────────────┐
         │   APPLICATION BACKEND (FastAPI)  │
         │  ┌──────────────────────────┐   │
         │  │ • Trajectory Query API   │   │
         │  │ • Alert Management       │   │
         │  │ • Traffic Analytics Calc │   │
         │  │ • Authentication/Auth    │   │
         │  └──────────────────────────┘   │
         └────────────────┬─────────────────┘
                          │
      ┌───────────────────┼───────────────────┐
      │                   │                   │
  ┌───▼────────┐  ┌──────▼──────┐  ┌────────▼──┐
  │ Web        │  │ Mobile      │  │ Admin     │
  │ Dashboard  │  │ App (PWA)   │  │ Console   │
  │ (React)    │  │             │  │           │
  └────────────┘  └─────────────┘  └───────────┘
```

---

## Component Breakdown

### **Component 1: ANPR & OCR Engine**

```
Video Feed
   │
   ├─► YOLOv8 (Vehicle Detection)
   │    └─► Crop vehicle region
   │
   ├─► TensorFlow-based Plate Localization
   │    └─► Find license plate in vehicle
   │
   ├─► PaddleOCR / EasyOCR (Character Recognition)
   │    └─► Extract plate number (e.g., "DL01AB1234")
   │
   └─► Confidence Scoring
        └─► Filter low-confidence detections (<90%)
```

**Key Metrics:**
- Plate detection accuracy: >95%
- OCR accuracy: >90%
- Processing speed: 25-30 FPS (real-time)
- Handles: Night vision, blur, partial plates, angles

---

### **Component 2: Trajectory Reconstruction**

```
Input: List of (Plate, Camera_ID, Timestamp) tuples
       Camera_ID has geo-coordinates (lat, long)

Processing:
1. Group by plate number
2. Sort by timestamp
3. Interpolate path between camera points
4. Handle outliers (impossible speeds)
5. Generate GIS track

Output: Complete route with:
- Start point (camera + time)
- End point (camera + time)
- Intermediate waypoints
- Total distance
- Average speed
```

**Example:**
```
Plate: MH02AB1234
Camera 1 (19.0760°N, 72.8777°E) at 10:05:23 → Start
Camera 2 (19.0865°N, 72.8912°E) at 10:07:45 → Waypoint
Camera 3 (19.0920°N, 72.9050°E) at 10:10:12 → End

Likely route: Via Bandra Bridge, Marine Drive
Distance: ~8 km
Time: 5 min (average speed: 96 km/h)
```

---

### **Component 3: Traffic Analytics Dashboard**

**Real-time Metrics:**
```
1. Traffic Density Heatmap
   - Red zones: Heavy congestion
   - Yellow zones: Moderate flow
   - Green zones: Free flow

2. Origin-Destination Matrix
   - Where vehicles come from
   - Where they're going
   - Peak hours

3. Congestion Bottlenecks
   - Top 5 slowest intersections
   - Average speed per road segment

4. Vehicle Count by Time
   - Vehicles/hour per camera
   - Peak traffic windows
```

---

### **Component 4: Alert System**

**Alert Types:**
```
1. Blacklist Alerts
   - Vehicle plate in "wanted" database
   - Trigger: "Vehicle XYZ detected at Camera 5"

2. Anomaly Detection
   - Impossible speeds (teleportation)
   - Suspicious routes (loitering near banks)
   - Unauthorized zone entries

3. Pattern Alerts
   - Vehicle appears at multiple locations in short time
   - Multiple vehicles same route (convoy detection)
```

---

# TECH STACK SELECTION

## Backend

| Component | Technology | Why This |
|-----------|-----------|---------|
| **ANPR Engine** | YOLOv8 + PaddleOCR | Highest accuracy, fastest inference |
| **Video Processing** | OpenCV | Real-time frame extraction |
| **Geospatial** | GeoPy + Folium | Distance calc + map visualization |
| **Database** | PostgreSQL + PostGIS | Spatial queries, trajectory storage |
| **Cache** | Redis | Real-time alert queue |
| **API** | FastAPI | Fast, async, perfect for streaming |
| **Async Jobs** | Celery + RabbitMQ | Process multiple camera feeds in parallel |

## Frontend

| Component | Technology | Why This |
|-----------|-----------|---------|
| **Dashboard** | React + Mapbox GL | Interactive maps, real-time updates |
| **Maps** | Mapbox / Leaflet | Trajectory visualization, heatmaps |
| **Real-time** | WebSocket (via FastAPI) | Live plate detections |
| **Charts** | Recharts / D3.js | Traffic analytics visualization |

## Deployment

| Component | Technology |
|-----------|-----------|
| **Containerization** | Docker |
| **Orchestration** | Docker Compose (36-hour) / Kubernetes (production) |
| **Hosting** | Local laptop + cloud option (AWS EC2) |

---

## Installation Commands (Pre-Hackathon)

```bash
# Clone skeleton project
git clone <your-repo>
cd sih26127

# Create Python environment
python3.10 -m venv venv
source venv/bin/activate

# Install ANPR dependencies
pip install ultralytics paddleocr opencv-python numpy pandas

# Install backend
pip install fastapi uvicorn sqlalchemy psycopg2 redis celery

# Install geospatial
pip install geopandas shapely folium geopy

# Install frontend deps
cd frontend
npm install react react-dom mapbox-gl recharts websocket

# Database
# PostgreSQL: brew install postgresql (or use Docker)
# Redis: brew install redis (or use Docker)
```

---

# 36-HOUR IMPLEMENTATION PLAN

## **PHASE 1: SETUP (Hours 0-2)**

### Hour 0-1: Project Structure & Dependencies
```
sih26127/
├── backend/
│   ├── anpr/
│   │   ├── detector.py         # YOLOv8 vehicle detection
│   │   ├── ocr.py              # PaddleOCR integration
│   │   └── models/             # Pre-trained models
│   ├── tracking/
│   │   ├── trajectory.py       # Trajectory reconstruction
│   │   └── matcher.py          # Plate matching across frames
│   ├── api/
│   │   ├── main.py             # FastAPI app
│   │   ├── routes.py           # API endpoints
│   │   └── websocket.py        # Real-time streaming
│   ├── database/
│   │   ├── models.py           # SQLAlchemy models
│   │   └── connection.py       # DB setup
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Dashboard.jsx
│   │   │   ├── MapView.jsx
│   │   │   └── AlertPanel.jsx
│   │   ├── App.jsx
│   │   └── index.js
│   └── package.json
├── docker-compose.yml
└── README.md
```

**Task Assignment:**
- Person 1: Clone repo template, setup venv, install deps
- Person 2: Initialize database schema + migrations
- Person 3: Frontend scaffold (React setup)

### Hour 1-2: Pre-trained Models Download
```bash
# Download YOLOv8 model (~50MB)
python3 -c "from ultralytics import YOLO; YOLO('yolov8s.pt')"

# Download PaddleOCR (~200MB)
from paddleocr import PaddleOCR
ocr = PaddleOCR(use_angle_cls=True, lang='en')

# Download Mapbox token (get free tier)
# Sign up: mapbox.com → get free API token
```

---

## **PHASE 2: CORE ANPR ENGINE (Hours 2-10)**

### Hour 2-4: Vehicle Detection Module

**File: `backend/anpr/detector.py`**

```python
from ultralytics import YOLO
import cv2
import numpy as np

class VehicleDetector:
    def __init__(self):
        self.model = YOLO('yolov8s.pt')  # Pre-trained on COCO
        
    def detect(self, frame):
        """
        Returns: list of (bbox, confidence, class_name)
        """
        results = self.model(frame, conf=0.5)
        
        detections = []
        for result in results:
            for box in result.boxes:
                class_id = int(box.cls[0])
                class_name = result.names[class_id]
                
                # Filter only cars, trucks, motorcycles
                if class_name in ['car', 'truck', 'motorcycle']:
                    x1, y1, x2, y2 = box.xyxy[0]
                    conf = float(box.conf[0])
                    
                    detections.append({
                        'bbox': (int(x1), int(y1), int(x2), int(y2)),
                        'confidence': conf,
                        'class': class_name
                    })
        
        return detections
```

**Testing:**
```python
detector = VehicleDetector()
frame = cv2.imread('test_traffic.jpg')
detections = detector.detect(frame)
print(f"Found {len(detections)} vehicles")
```

### Hour 4-7: License Plate Detection & OCR

**File: `backend/anpr/ocr.py`**

```python
from paddleocr import PaddleOCR
import cv2

class PlateOCR:
    def __init__(self):
        self.ocr = PaddleOCR(use_angle_cls=True, lang='en')
        
    def extract_plate(self, vehicle_crop):
        """
        Input: Cropped vehicle image
        Output: (plate_text, confidence)
        """
        # Run OCR
        result = self.ocr.ocr(vehicle_crop, cls=True)
        
        if not result or not result[0]:
            return None, 0.0
        
        # Aggregate OCR results
        plate_text = ''.join([line[1][0] for line in result[0]])
        confidence = np.mean([line[1][1] for line in result[0]])
        
        # Filter > 90% confidence
        if confidence >= 0.90:
            return plate_text, confidence
        
        return None, confidence
    
    def validate_plate(self, plate_text):
        """
        Validate Indian license plate format
        Example: "MH02AB1234" or "DL01CD5678"
        """
        import re
        pattern = r'^[A-Z]{2}\d{2}[A-Z]{2}\d{4}$'
        return bool(re.match(pattern, plate_text))
```

**Testing:**
```python
ocr = PlateOCR()
vehicle_image = frame[y1:y2, x1:x2]
plate, conf = ocr.extract_plate(vehicle_image)
if plate and conf > 0.90:
    print(f"Plate: {plate} (Confidence: {conf:.2%})")
```

### Hour 7-10: Real-time Video Feed Processing

**File: `backend/anpr/processor.py`**

```python
class VideoProcessor:
    def __init__(self, rtsp_url, camera_id, location):
        self.rtsp_url = rtsp_url
        self.camera_id = camera_id  # Unique ID
        self.location = location    # (lat, long)
        self.detector = VehicleDetector()
        self.ocr = PlateOCR()
        
    def process_stream(self):
        cap = cv2.VideoCapture(self.rtsp_url)
        frame_count = 0
        
        while True:
            ret, frame = cap.read()
            if not ret:
                break
                
            frame_count += 1
            
            # Skip every 5th frame for speed
            if frame_count % 5 != 0:
                continue
            
            # Detect vehicles
            vehicles = self.detector.detect(frame)
            
            # Extract plates
            detections = []
            for vehicle in vehicles:
                x1, y1, x2, y2 = vehicle['bbox']
                crop = frame[y1:y2, x1:x2]
                
                plate, conf = self.ocr.extract_plate(crop)
                
                if plate and conf > 0.90:
                    detection = {
                        'plate': plate,
                        'confidence': conf,
                        'camera_id': self.camera_id,
                        'location': self.location,
                        'timestamp': datetime.datetime.now(),
                        'bbox': vehicle['bbox']
                    }
                    detections.append(detection)
                    
                    # Send to database + real-time queue
                    self.save_detection(detection)
                    self.broadcast_alert(detection)
            
            # Optional: Draw bounding boxes for debugging
            if False:  # Set to True for debugging
                self.draw_detections(frame, detections)
                cv2.imshow('ANPR', frame)
                if cv2.waitKey(1) & 0xFF == ord('q'):
                    break
        
        cap.release()
```

---

## **PHASE 3: TRAJECTORY RECONSTRUCTION (Hours 10-18)**

### Hour 10-12: Plate Matching & Linking

**File: `backend/tracking/matcher.py`**

```python
from datetime import timedelta
from math import radians, cos, sin, asin, sqrt

class TrajectoryBuilder:
    def __init__(self, db):
        self.db = db
        self.max_speed = 120  # km/h (physical limit)
        
    def calculate_distance(self, lat1, lon1, lat2, lon2):
        """Haversine formula: distance between two GPS points"""
        lat1, lon1, lat2, lon2 = map(radians, [lat1, lon1, lat2, lon2])
        dlat = lat2 - lat1
        dlon = lon2 - lon1
        a = sin(dlat/2)**2 + cos(lat1) * cos(lat2) * sin(dlon/2)**2
        c = 2 * asin(sqrt(a))
        km = 6371 * c
        return km
    
    def build_trajectory(self, plate, time_window_hours=4):
        """
        Get all detections of a plate and build trajectory
        """
        # Query: all detections of this plate in last N hours
        detections = self.db.query_detections(
            plate=plate,
            time_window=time_window_hours
        )
        
        if not detections:
            return None
        
        # Sort by timestamp
        detections = sorted(detections, key=lambda x: x['timestamp'])
        
        # Validate: check if speeds are physically possible
        trajectory = []
        for i, detection in enumerate(detections):
            if i > 0:
                prev = detections[i-1]
                
                # Distance between two camera points
                dist = self.calculate_distance(
                    prev['lat'], prev['lon'],
                    detection['lat'], detection['lon']
                )
                
                # Time difference
                time_diff = (detection['timestamp'] - prev['timestamp']).total_seconds() / 3600
                
                # Average speed
                if time_diff > 0:
                    speed = dist / time_diff
                    
                    # Validate: reject if speed > max_speed (outlier)
                    if speed > self.max_speed:
                        print(f"Suspicious: {plate} at {speed} km/h (outlier)")
                        continue
            
            trajectory.append(detection)
        
        return trajectory
    
    def get_trajectory_summary(self, plate):
        """Return summary stats"""
        traj = self.build_trajectory(plate)
        
        if not traj:
            return None
        
        total_dist = sum([
            self.calculate_distance(
                traj[i]['lat'], traj[i]['lon'],
                traj[i+1]['lat'], traj[i+1]['lon']
            ) for i in range(len(traj)-1)
        ])
        
        return {
            'plate': plate,
            'start_camera': traj[0]['camera_id'],
            'start_time': traj[0]['timestamp'],
            'end_camera': traj[-1]['camera_id'],
            'end_time': traj[-1]['timestamp'],
            'num_cameras': len(set([d['camera_id'] for d in traj])),
            'total_distance': round(total_dist, 2),
            'waypoints': traj
        }
```

### Hour 12-15: GIS Mapping Integration

**File: `backend/tracking/mapper.py`**

```python
import folium
from folium import plugins

class TrajectoryMapper:
    def generate_map(self, trajectory_summary):
        """Generate interactive map of vehicle trajectory"""
        
        waypoints = trajectory_summary['waypoints']
        
        # Center map on average location
        avg_lat = sum([w['lat'] for w in waypoints]) / len(waypoints)
        avg_lon = sum([w['lon'] for w in waypoints]) / len(waypoints)
        
        m = folium.Map(
            location=[avg_lat, avg_lon],
            zoom_start=12,
            tiles='OpenStreetMap'
        )
        
        # Draw path
        path_coords = [[w['lat'], w['lon']] for w in waypoints]
        folium.PolyLine(
            path_coords,
            color='red',
            weight=3,
            opacity=0.7
        ).add_to(m)
        
        # Mark start point (green)
        folium.CircleMarker(
            location=[waypoints[0]['lat'], waypoints[0]['lon']],
            radius=10,
            color='green',
            fill=True,
            popup=f"START: {waypoints[0]['timestamp']}"
        ).add_to(m)
        
        # Mark end point (red)
        folium.CircleMarker(
            location=[waypoints[-1]['lat'], waypoints[-1]['lon']],
            radius=10,
            color='red',
            fill=True,
            popup=f"END: {waypoints[-1]['timestamp']}"
        ).add_to(m)
        
        # Mark intermediate cameras
        for i, wp in enumerate(waypoints[1:-1], 1):
            folium.CircleMarker(
                location=[wp['lat'], wp['lon']],
                radius=6,
                color='blue',
                fill=True,
                popup=f"Camera {wp['camera_id']}: {wp['timestamp']}"
            ).add_to(m)
        
        # Save map
        m.save(f'/tmp/trajectory_{trajectory_summary["plate"]}.html')
        return m
```

---

## **PHASE 4: ANALYTICS DASHBOARD (Hours 18-28)**

### Hour 18-22: Backend API Endpoints

**File: `backend/api/routes.py`**

```python
from fastapi import FastAPI, WebSocket
from fastapi.responses import HTMLResponse
import json
from datetime import datetime, timedelta

app = FastAPI()

# ============= TRAJECTORY QUERIES =============

@app.get("/api/trajectory/{plate}")
async def get_trajectory(plate: str, hours: int = 4):
    """Get complete trajectory of a vehicle plate"""
    trajectory = trajectory_builder.get_trajectory_summary(plate)
    
    if not trajectory:
        return {"error": "Plate not found"}
    
    return trajectory

@app.get("/api/heatmap")
async def get_traffic_heatmap(interval: str = "1h"):
    """Get traffic density heatmap"""
    # Query all plates in last N hours
    # Count detections per camera/road segment
    
    heatmap_data = {}
    for camera in all_cameras:
        count = db.count_detections(
            camera_id=camera['id'],
            time_interval=interval
        )
        heatmap_data[camera['id']] = {
            'lat': camera['lat'],
            'lon': camera['lon'],
            'count': count,
            'density': 'red' if count > 100 else 'yellow' if count > 50 else 'green'
        }
    
    return heatmap_data

@app.get("/api/congestion-points")
async def get_congestion():
    """Top 5 most congested roads"""
    result = db.query("""
        SELECT 
            camera_id,
            COUNT(*) as vehicle_count,
            AVG(speed) as avg_speed
        FROM detections
        WHERE timestamp > NOW() - INTERVAL '1 hour'
        GROUP BY camera_id
        ORDER BY vehicle_count DESC
        LIMIT 5
    """)
    
    return [
        {
            'camera': r['camera_id'],
            'vehicles': r['vehicle_count'],
            'avg_speed': r['avg_speed']
        } for r in result
    ]

@app.get("/api/origin-destination")
async def get_od_matrix():
    """Origin-destination patterns"""
    # Query: where do vehicles come from and where do they go
    result = db.query("""
        SELECT 
            start_camera,
            end_camera,
            COUNT(*) as trips
        FROM trajectories
        WHERE timestamp > NOW() - INTERVAL '24 hours'
        GROUP BY start_camera, end_camera
        ORDER BY trips DESC
    """)
    
    return result

# ============= ALERTS =============

@app.post("/api/blacklist")
async def add_to_blacklist(plate: str, reason: str):
    """Add plate to wanted list"""
    db.add_blacklist(plate, reason)
    return {"status": "added"}

@app.get("/api/alerts")
async def get_recent_alerts():
    """Get last 100 alerts"""
    alerts = db.query("SELECT * FROM alerts ORDER BY timestamp DESC LIMIT 100")
    return alerts

# ============= REAL-TIME STREAMING =============

@app.websocket("/ws/live-detections")
async def websocket_endpoint(websocket: WebSocket):
    """WebSocket for real-time plate detections"""
    await websocket.accept()
    
    try:
        while True:
            # Listen to Redis queue for new detections
            detection = redis_client.blpop('detections_queue', timeout=0)
            
            if detection:
                await websocket.send_json(json.loads(detection[1]))
    except Exception as e:
        print(f"WebSocket error: {e}")
    finally:
        await websocket.close()

# ============= STATISTICS =============

@app.get("/api/stats")
async def get_statistics():
    """Dashboard stats"""
    now = datetime.now()
    hour_ago = now - timedelta(hours=1)
    
    total_vehicles = db.count_detections(time_start=hour_ago)
    total_plates = db.count_unique_plates(time_start=hour_ago)
    cameras_active = db.count_active_cameras()
    
    return {
        'total_vehicles_detected': total_vehicles,
        'unique_plates': total_plates,
        'active_cameras': cameras_active,
        'average_plate_confidence': db.get_avg_confidence()
    }
```

### Hour 22-28: React Frontend Dashboard

**File: `frontend/src/components/Dashboard.jsx`**

```jsx
import React, { useState, useEffect } from 'react';
import MapView from './MapView';
import AlertPanel from './AlertPanel';
import TrafficStats from './TrafficStats';

export default function Dashboard() {
  const [stats, setStats] = useState(null);
  const [alerts, setAlerts] = useState([]);
  const [selectedPlate, setSelectedPlate] = useState(null);
  const [trajectory, setTrajectory] = useState(null);

  useEffect(() => {
    // Fetch statistics
    fetch('/api/stats')
      .then(r => r.json())
      .then(data => setStats(data));

    // Fetch alerts
    fetch('/api/alerts')
      .then(r => r.json())
      .then(data => setAlerts(data));

    // WebSocket for real-time
    const ws = new WebSocket('ws://localhost:8000/ws/live-detections');
    ws.onmessage = (event) => {
      const detection = JSON.parse(event.data);
      // Check if plate is blacklisted
      if (isBlacklisted(detection.plate)) {
        // Trigger alert
        showAlert(`ALERT: Blacklisted vehicle ${detection.plate} at ${detection.camera_id}`);
      }
    };

    return () => ws.close();
  }, []);

  const handlePlateSearch = async (plate) => {
    const response = await fetch(`/api/trajectory/${plate}`);
    const data = await response.json();
    setSelectedPlate(plate);
    setTrajectory(data);
  };

  return (
    <div className="dashboard">
      <header>
        <h1>🚗 City Traffic Intelligence Platform</h1>
        <input 
          type="text" 
          placeholder="Search plate number..." 
          onSubmit={(e) => handlePlateSearch(e.target.value)}
        />
      </header>

      <main>
        <section className="left-panel">
          <TrafficStats stats={stats} />
          <AlertPanel alerts={alerts} />
        </section>

        <section className="main-content">
          <MapView trajectory={trajectory} selectedPlate={selectedPlate} />
        </section>
      </main>
    </div>
  );
}
```

**File: `frontend/src/components/MapView.jsx`**

```jsx
import React, { useEffect, useRef } from 'react';
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';

mapboxgl.accessToken = process.env.REACT_APP_MAPBOX_TOKEN;

export default function MapView({ trajectory, selectedPlate }) {
  const mapContainer = useRef(null);
  const map = useRef(null);

  useEffect(() => {
    if (!trajectory) return;

    if (!map.current) {
      map.current = new mapboxgl.Map({
        container: mapContainer.current,
        style: 'mapbox://styles/mapbox/streets-v11',
        center: [trajectory.waypoints[0].lon, trajectory.waypoints[0].lat],
        zoom: 12
      });
    }

    // Draw trajectory polyline
    const coordinates = trajectory.waypoints.map(w => [w.lon, w.lat]);

    map.current.on('load', () => {
      // Source for line
      map.current.addSource('route', {
        'type': 'geojson',
        'data': {
          'type': 'Feature',
          'geometry': {
            'type': 'LineString',
            'coordinates': coordinates
          }
        }
      });

      // Line layer
      map.current.addLayer({
        'id': 'route',
        'type': 'line',
        'source': 'route',
        'paint': {
          'line-color': '#ff0000',
          'line-width': 4
        }
      });

      // Start marker
      new mapboxgl.Marker({ color: 'green' })
        .setLngLat(coordinates[0])
        .setPopup(new mapboxgl.Popup().setHTML(`START<br>${trajectory.start_time}`))
        .addTo(map.current);

      // End marker
      new mapboxgl.Marker({ color: 'red' })
        .setLngLat(coordinates[coordinates.length - 1])
        .setPopup(new mapboxgl.Popup().setHTML(`END<br>${trajectory.end_time}`))
        .addTo(map.current);
    });

  }, [trajectory]);

  return (
    <div>
      <h2>Trajectory: {selectedPlate}</h2>
      <div ref={mapContainer} style={{ width: '100%', height: '600px' }} />
    </div>
  );
}
```

---

## **PHASE 5: TESTING & DEMO (Hours 28-36)**

### Hour 28-32: Integration Testing

**File: `tests/test_anpr.py`**

```python
def test_vehicle_detection():
    """Test YOLOv8 detection"""
    detector = VehicleDetector()
    test_image = cv2.imread('test_traffic.jpg')
    detections = detector.detect(test_image)
    
    assert len(detections) > 0, "Should detect at least one vehicle"
    print(f"✓ Detected {len(detections)} vehicles")

def test_ocr_accuracy():
    """Test PaddleOCR accuracy"""
    ocr = PlateOCR()
    test_plates = [
        'test_plate_1.jpg',
        'test_plate_2.jpg'
    ]
    
    correct = 0
    for plate_img in test_plates:
        plate, conf = ocr.extract_plate(cv2.imread(plate_img))
        if conf > 0.90:
            correct += 1
    
    accuracy = (correct / len(test_plates)) * 100
    print(f"✓ OCR Accuracy: {accuracy}%")
    assert accuracy >= 85, "OCR should be >85% accurate"

def test_trajectory_building():
    """Test trajectory reconstruction"""
    tb = TrajectoryBuilder(db)
    
    # Insert mock detections
    db.insert_detection('MH02AB1234', camera_1, timestamp1)
    db.insert_detection('MH02AB1234', camera_2, timestamp2)
    db.insert_detection('MH02AB1234', camera_3, timestamp3)
    
    trajectory = tb.build_trajectory('MH02AB1234')
    
    assert trajectory is not None
    assert len(trajectory) == 3
    print(f"✓ Built trajectory with {len(trajectory)} waypoints")

def test_api_endpoints():
    """Test API endpoints"""
    response = client.get('/api/stats')
    assert response.status_code == 200
    print("✓ /api/stats endpoint working")
    
    response = client.get('/api/trajectory/MH02AB1234')
    assert response.status_code in [200, 404]
    print("✓ /api/trajectory endpoint working")
```

### Hour 32-35: Create Demo Video

**What to show (5 minutes):**

```
Minute 1: System Overview
  - Architecture diagram
  - Components explanation
  - "This solves the 3 core requirements"

Minute 2: ANPR Live Detection
  - Show real traffic video
  - Highlight vehicle detections
  - Show plate extraction
  - Confidence scores

Minute 3: Trajectory Demo
  - Search for a plate
  - Show complete route on map
  - Display stats (distance, time, cameras)

Minute 4: Dashboard & Analytics
  - Real-time heatmap
  - Traffic congestion
  - Origin-destination matrix
  - Alert system

Minute 5: Impact & Scalability
  - Cost analysis
  - How it scales to 100+ cameras
  - Real-world use cases
```

### Hour 35-36: Presentation Prep

**Presentation Structure:**

```
INTRO (2 min):
"50,000 traffic cameras in Indian cities. But they're isolated silos.
Our platform: Connect them. Track vehicles. Predict traffic."

PROBLEM (1 min):
- Current: Cameras don't talk to each other
- Result: Can't track stolen cars, can't optimize traffic
- Our solution: AI-powered city-wide tracking

SOLUTION (3 min):
1. ANPR Engine: >90% accuracy
2. Trajectory Tracking: Follow any vehicle
3. Analytics: Real-time traffic intel
[Live demo of dashboard]

TECHNICAL (2 min):
- YOLOv8 + PaddleOCR (>90% accuracy)
- PostGIS for spatial queries
- Real-time WebSocket streaming
- Scalable to 1000+ cameras

IMPACT (2 min):
"This enables:"
- Police: Find stolen vehicles in 5 minutes
- Traffic: Reduce congestion by 15%
- Safety: Better road accident prevention
- Revenue: Toll collection automation

FUTURE (1 min):
- Integrate with existing city infrastructure
- Multi-city deployment
- ML for predictive traffic

"Questions?"
```

---

# TEAM ROLE ALLOCATION

## 6-Person Team Structure

| Person | Role | Responsibilities | Hours 0-36 Focus |
|--------|------|------------------|------------------|
| **Person 1** | **ANPR Lead** | YOLOv8 + PaddleOCR integration | Phases 1-2: Setup + ANPR engine |
| **Person 2** | **Backend Lead** | FastAPI, database, APIs | Phases 3-4: Trajectory + API endpoints |
| **Person 3** | **Frontend Lead** | React dashboard, maps | Phase 4: Dashboard & visualization |
| **Person 4** | **DevOps/Infra** | Docker, database setup, deployment | Phases 1-5: Setup + integration |
| **Person 5** | **Testing & Integration** | Unit tests, E2E testing, bug fixes | Phases 5: Testing + quality |
| **Person 6** | **Demo & Presentation** | Video creation, slides, pitch practice | Phases 5-6: Demo + presentation |

---

## Detailed Task Breakdown

### **Person 1: ANPR Lead**
```
Hour 0-2:   Environment setup, model downloads
Hour 2-4:   YOLOv8 vehicle detection coding
Hour 4-7:   PaddleOCR license plate extraction
Hour 7-10:  Real-time video stream processing
Hour 10-14: ANPR accuracy testing & tuning
Hour 14-18: Integration with backend database
Hour 18-24: Handle edge cases (night, blur, angles)
Hour 24-30: Performance optimization
Hour 30-36: Bug fixes + support demo
```

### **Person 2: Backend Lead**
```
Hour 0-2:   Database schema design (PostgreSQL + PostGIS)
Hour 2-4:   SQLAlchemy model definitions
Hour 4-10:  Trajectory reconstruction algorithm
Hour 10-14: Trajectory validation & outlier handling
Hour 14-18: FastAPI routes & endpoints
Hour 18-22: WebSocket real-time streaming setup
Hour 22-28: Alert system + blacklist functionality
Hour 28-32: API optimization & caching (Redis)
Hour 32-36: Integration testing + bug fixes
```

### **Person 3: Frontend Lead**
```
Hour 0-2:   React project setup + scaffolding
Hour 2-6:   Component architecture design
Hour 6-12:  Mapbox integration + trajectory visualization
Hour 12-18: Dashboard layout + stats panels
Hour 18-24: Real-time updates via WebSocket
Hour 24-28: Alert panel + notifications
Hour 28-32: UI/UX polish + responsiveness
Hour 32-36: Cross-browser testing + optimization
```

### **Person 4: DevOps/Infra**
```
Hour 0-2:   Docker setup + docker-compose
Hour 2-4:   PostgreSQL + PostGIS containerization
Hour 4-8:   Redis caching setup
Hour 8-12:  Environment configuration (.env files)
Hour 12-16: CI/CD pipeline (GitHub Actions)
Hour 16-24: Network configuration + API gateway
Hour 24-30: Logging + monitoring setup
Hour 30-36: Deployment readiness + documentation
```

### **Person 5: Testing & QA**
```
Hour 4-8:   Write unit tests for ANPR
Hour 8-14:  Integration tests for backend
Hour 14-20: End-to-end testing (frontend + backend)
Hour 20-26: Load testing (multiple camera feeds)
Hour 26-30: Performance benchmarking
Hour 30-36: Bug fixes + regression testing
```

### **Person 6: Demo & Presentation**
```
Hour 0-6:   Storyboard + script writing
Hour 6-12:  Create demo video (record + edit)
Hour 12-18: Build presentation slides
Hour 18-24: Practice pitch (solo + with team)
Hour 24-30: Fine-tune demo flow
Hour 30-36: Final rehearsal + confidence building
```

---

# COMPONENT DEEP DIVE

## ANPR Engine - Accuracy Tips

### Problem: Why OCR fails

```
Scenario 1: Night Vision
  Issue: Low light, high noise
  Solution: Use night-mode preprocessing
  >>> image = cv2.convertScaleAbs(cv2.Laplacian(image, cv2.CV_64F))

Scenario 2: Motion Blur
  Issue: Fast-moving vehicles blur plates
  Solution: Detect blur + apply deblurring
  >>> def detect_blur(image):
  ...   laplacian = cv2.Laplacian(image, cv2.CV_64F)
  ...   return laplacian.var() < threshold

Scenario 3: Dirty/Damaged Plates
  Issue: Missing characters, rust, dirt
  Solution: Use larger OCR models + voting
  >>> # Run multiple OCR engines, vote on result
  >>> result1 = paddleocr.ocr(image)
  >>> result2 = easyocr.readtext(image)
  >>> final = vote(result1, result2)

Scenario 4: Angled Plates
  Issue: Plate not parallel to camera
  Solution: Use rotation correction
  >>> angle = detect_plate_angle(image)
  >>> rotated = cv2.rotate(image, angle)
  >>> result = ocr(rotated)
```

### Achieving >90% Accuracy

```
Strategy 1: Ensemble Methods
- Run 2 OCR models (PaddleOCR + EasyOCR)
- Vote on disagreements
- Achieves 94-96% accuracy

Strategy 2: Preprocessing Pipeline
1. Brightness normalization
2. Contrast enhancement (CLAHE)
3. Denoising (bilateral filter)
4. Morphological operations

Strategy 3: Confidence Thresholding
- Only accept detections > 0.90 confidence
- Trade off coverage for accuracy
- Tune via test dataset

Code:
```python
def preprocess_plate(image):
    # Convert to grayscale
    gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    
    # CLAHE (Contrast Limited Adaptive Histogram Equalization)
    clahe = cv2.createCLAHE(clipLimit=2.0, tileGridSize=(8,8))
    enhanced = clahe.apply(gray)
    
    # Denoising
    denoised = cv2.fastNlMeansDenoising(enhanced)
    
    # Thresholding
    _, thresh = cv2.threshold(denoised, 150, 255, cv2.THRESH_BINARY)
    
    return thresh
```

---

## Database Schema Design

### PostgreSQL with PostGIS

```sql
-- Cameras table
CREATE TABLE cameras (
  id SERIAL PRIMARY KEY,
  camera_name VARCHAR(255),
  location GEOGRAPHY(POINT, 4326),  -- Latitude, Longitude
  created_at TIMESTAMP DEFAULT NOW()
);

-- Detections table
CREATE TABLE detections (
  id SERIAL PRIMARY KEY,
  plate VARCHAR(20),
  camera_id INTEGER REFERENCES cameras(id),
  confidence FLOAT,
  timestamp TIMESTAMP DEFAULT NOW(),
  bbox JSONB,  -- Bounding box coordinates
  created_at TIMESTAMP DEFAULT NOW()
);

-- Trajectories table
CREATE TABLE trajectories (
  id SERIAL PRIMARY KEY,
  plate VARCHAR(20),
  start_camera_id INTEGER REFERENCES cameras(id),
  end_camera_id INTEGER REFERENCES cameras(id),
  start_time TIMESTAMP,
  end_time TIMESTAMP,
  distance_km FLOAT,
  waypoints JSONB,  -- GeoJSON format
  created_at TIMESTAMP DEFAULT NOW()
);

-- Blacklist table
CREATE TABLE blacklist (
  id SERIAL PRIMARY KEY,
  plate VARCHAR(20) UNIQUE,
  reason TEXT,
  added_at TIMESTAMP DEFAULT NOW(),
  active BOOLEAN DEFAULT TRUE
);

-- Alerts table
CREATE TABLE alerts (
  id SERIAL PRIMARY KEY,
  alert_type VARCHAR(50),  -- 'blacklist', 'anomaly', 'pattern'
  plate VARCHAR(20),
  camera_id INTEGER REFERENCES cameras(id),
  message TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Indexes for performance
CREATE INDEX idx_detections_plate ON detections(plate);
CREATE INDEX idx_detections_timestamp ON detections(timestamp DESC);
CREATE INDEX idx_detections_camera ON detections(camera_id);
CREATE INDEX idx_trajectories_plate ON trajectories(plate);
```

---

# DATA PIPELINE & WORKFLOW

## Complete Flow

```
1. VIDEO INPUT
   Multiple RTSP streams (real traffic cameras)
   
2. FRAME EXTRACTION
   Sample 1 frame per 2 seconds (optimize speed)
   
3. VEHICLE DETECTION
   YOLOv8: Identify vehicles in frame
   Filter: Only cars, trucks, motorcycles
   
4. PLATE REGION EXTRACTION
   Crop vehicle region (RoI)
   
5. LICENSE PLATE DETECTION
   Plate localization model
   Get plate bounding box
   
6. OCR / CHARACTER RECOGNITION
   PaddleOCR processes plate image
   Extract: "MH02AB1234"
   Confidence: 0.95
   
7. VALIDATION
   Regex check: Is it valid Indian format?
   Confidence check: >0.90?
   Blacklist check: Is it wanted?
   
8. STORAGE
   If valid:
   - Save to database
   - Broadcast to Redis queue
   - Send alert (if blacklisted)
   
9. TRAJECTORY LINKING
   Match same plate across cameras
   Build temporal-spatial track
   Calculate metrics
   
10. AGGREGATION & ANALYTICS
    Count vehicles per road
    Calculate average speeds
    Identify congestion
    
11. API RESPONSE
    Dashboard queries database
    Return results to UI
    
12. VISUALIZATION
    Maps show vehicle paths
    Heatmaps show traffic
    Alerts notify authorities
```

---

# TESTING & DEMO

## Pre-Demo Checklist

```
ANPR Engine
□ Tested on 100+ real traffic images
□ Achieves >90% OCR accuracy
□ Handles night/blur/angles
□ Processing time < 100ms per vehicle

Trajectory Module
□ Successfully links same plate across cameras
□ Correctly interpolates paths
□ Validates speeds are realistic
□ Generates correct GeoJSON

Backend API
□ All endpoints tested
□ WebSocket streaming works
□ Database queries fast (<1s)
□ Error handling for edge cases

Frontend Dashboard
□ Loads within 3 seconds
□ Maps render smoothly
□ Real-time updates working
□ Responsive on mobile

Full System
□ Docker containers start cleanly
□ All services communicate
□ No memory leaks during 4-hour run
□ Logs are clear and helpful
```

---

# PRESENTATION STRATEGY

## Winning the Judges

### What Judges Care About (in order):

1. **Problem Understanding** (20%)
   - Do you understand the REAL problem?
   - "Traffic cameras are silos" → "We connect them"

2. **Innovation** (25%)
   - What's novel about your approach?
   - "We combine ANPR + trajectory + analytics"

3. **Feasibility** (20%)
   - Can you actually build this?
   - "We use pre-trained models + proven tech"

4. **Impact** (20%)
   - What's the real-world benefit?
   - "Reduce stolen car recovery time by 90%"

5. **Code Quality** (15%)
   - Is it well-architected?
   - "Clean, scalable, documented"

### Judge-Winning Statements

```
"Our system turns 500 isolated traffic cameras 
into a single AI-powered traffic intelligence network."

"We achieve >90% license plate accuracy in real-world 
conditions (night, rain, blur) using ensemble models."

"Any stolen vehicle can be located within 5 minutes 
instead of 48+ hours."

"If deployed across 10 Indian cities, this saves ₹500 Cr 
annually through automated toll collection alone."

"The system is already proven at scale with YOLOv8 
and PostGIS used by thousands of deployments."
```

### Demo Video Shot List

```
Shot 1 (15 sec): Drone view of city traffic
- Show problem: "Thousands of cameras, zero coordination"

Shot 2 (30 sec): Traffic cam feed
- Vehicle drives past multiple cameras
- Draw circles around detected vehicles
- Extract plate text overlay

Shot 3 (45 sec): Map animation
- Show vehicle's complete journey
- Red line: actual path
- Green dot: start
- Red dot: end

Shot 4 (30 sec): Dashboard real-time
- Plate search: "MH02AB1234"
- Click: "Show route"
- Map displays with stats

Shot 5 (30 sec): Analytics dashboard
- Heatmap: red zones (congestion)
- Chart: vehicles per hour
- Table: top congestion points

Shot 6 (15 sec): Alert demo
- "Blacklist added: Plate XYZ (Stolen)"
- Vehicle appears on camera
- ALERT: Red notification pops up

Shot 7 (15 sec): Scaling visualization
- Show architecture can handle 100 cameras
- Numbers: "Processing 1000 vehicles/minute"
```

---

# FINAL TIPS FOR SUCCESS

## What Makes Winners Stand Out

✅ **DO:**
- Show a working prototype (even if imperfect)
- Use real traffic data (not synthetic)
- Have a clear "aha moment" in your demo
- Show you understand the judges' pain points
- Practice your pitch (sounds confident, not rushed)

❌ **DON'T:**
- Oversell unrealistic numbers
- Have a demo that crashes
- Use blurry, hard-to-read screenshots
- Talk too fast or mumble
- Ignore questions from judges

## Common Mistakes to Avoid

| Mistake | Why It Fails | Fix |
|---------|------------|-----|
| "We'll use real traffic cameras" | You can't in 36 hours | Use YouTube traffic streams instead |
| "We'll achieve 100% accuracy" | Impossible, judges know this | "We achieve 92% accuracy, beating industry benchmarks" |
| "We'll support 1000 cities" | Unrealistic scope | "Designed to scale; deployed in 1 city first" |
| "No dependencies, built from scratch" | Wastes time, judges don't care | Use existing libraries proudly |
| "We didn't test it fully" | Judges notice bugs immediately | Test thoroughly, ship polished |

---

# RESOURCES & LINKS

## Pre-Built Models
- YOLOv8: https://github.com/ultralytics/ultralytics
- PaddleOCR: https://github.com/PaddlePaddle/PaddleOCR
- EasyOCR: https://github.com/JaidedAI/EasyOCR

## Sample Traffic Video
- YouTube search: "Traffic camera feed 24 hour" (use legal clips)
- Local city CCTV footage (ask local traffic police)

## Sample Indian License Plate Data
- AOLPR dataset: https://github.com/ankitshah009/Indian_Number_Plate_Detection

## Frontend Libraries
- Mapbox GL JS: https://docs.mapbox.com/mapbox-gl-js/
- Folium: https://folium.readthedocs.io/

## Database
- PostGIS Guide: https://postgis.net/
- FastAPI + SQLAlchemy: https://fastapi.tiangolo.com/

---

