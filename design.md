# Design Specification
## Blind Assistance Smart Belt

### 1. System Architecture

**Architecture:** Multi-threaded Event-Driven System

```
Main Application (GUI Thread)
    ├── Hardware Monitor Loop
    │   ├── MPU6050 Reader
    │   ├── Ultrasonic Sensors (x3)
    │   └── Emergency Button
    ├── GPS Reader Thread
    ├── Camera Thread (DNN Processing)
    └── Alert System
        ├── Telegram API
        ├── Email (SMTP)
        └── Audio Playback
```

**Design Principles:**
- Non-blocking sensor operations
- Graceful hardware failure handling
- Alert cooldown mechanisms
- Thread-safe state management
- Modular component separation

---

### 2. Core Components

#### 2.1 GPS Reader

**Function:** Continuous GPS data acquisition and parsing

**Key Operations:**
- Attempts connection on multiple serial ports
- Reads and parses NMEA GPGGA sentences
- Converts coordinates to decimal degrees
- Validates GPS fix quality
- Automatic reconnection on failures
- Fallback to static coordinates

**Error Handling:**
- Reconnection attempts on serial failures
- Invalid data logged without crashing
- Timeout protection on reads

#### 2.2 Sensor Interfaces

**Hardware Abstraction:** Mock objects for testing without hardware

##### MPU6050 (Motion Sensor)
- Read acceleration data from I²C registers
- Calculate magnitude: √(x² + y² + z²)
- Return safe defaults on I²C failures

##### Ultrasonic Sensors
- Send trigger pulse, measure echo duration
- Calculate distance using speed of sound
- Timeout protection (30ms max)
- Return sentinel value on failure

##### Emergency Button
- GPIO with pull-up resistor (active LOW)
- Cooldown-based debouncing

#### 2.3 Fall Detection

**Algorithm:** Threshold-based acceleration monitoring

**Logic:**
- Detect when magnitude < 0.5g (free-fall) OR > 2.0g (impact)
- Check cooldown period (30s)
- Get GPS coordinates
- Send alerts via Telegram and Email

**Thresholds:**
- Low: 0.5g (sudden drop)
- High: 2.0g (impact)
- Cooldown: 30s (prevent spam)

#### 2.4 Object Detection

**Model:** MobileNet SSD with OpenCV DNN

**Pipeline:**
1. Capture frame (640×480)
2. Resize to 300×300
3. Normalize and create blob
4. DNN inference
5. Filter detections (confidence > 30%)
6. Draw bounding boxes
7. Trigger alerts for dangerous objects

**Dangerous Objects:** car, dog, train (30s cooldown per type)

**Performance:** ~20 FPS on Raspberry Pi 4

#### 2.5 Alert System

**Design:** Fire-and-forget with error isolation

**Channels:**

**Telegram:** POST to Bot API with 3s timeout

**Email:** SMTP with STARTTLS encryption (port 587)

**Audio:** Non-blocking WAV playback using aplay
- Directional: left.wav, right.wav, back.wav
- Object-specific: car.wav, dog.wav, etc.

**Error Handling:** Failures logged but don't block operations

#### 2.6 User Interface

**Framework:** Tkinter GUI with external OpenCV window

**Layout:**
- Hardware status panel (MPU6050, ultrasonic, GPS, emergency)
- Camera controls (start/stop)
- Scrollable event log
- Map view button

**Update:** 500ms polling loop for sensor readings

**Camera:** External OpenCV window for better performance

---

### 3. Data Flow

#### Obstacle Detection
Sensor → Distance Check → Cooldown → Audio + GPS + Alerts

#### Fall Detection
MPU6050 → Magnitude Check → Cooldown → GPS + Alerts

#### Emergency Alert
Button Press → Cooldown → GPS + Alerts

#### Object Detection
Camera → DNN → Filter → Bounding Boxes → Dangerous Check → Alerts

---

### 4. State Management

**Cooldown Tracking:** Timestamps for each alert type to prevent spam

**GPS State:** Current coordinates, availability status, reconnection attempts

**Camera State:** Running status, capture object

**State Transitions:**
- Camera: Stopped ↔ Running
- GPS: Initializing → Active/Unavailable → Reconnecting

---

### 5. Error Handling

**Hardware Failures:** Graceful degradation with safe defaults
- GPS unavailable → Use static coordinates
- Sensor errors → Log and return safe values
- Camera not found → Disable camera features

**Network Failures:** Log and continue (local audio still works)

**Software Errors:** Exception isolation prevents crashes

---

### 6. Performance

**Target Latency:**
- Sensor reads: < 30ms
- DNN inference: < 200ms
- Alert delivery: < 3s
- GUI updates: 500ms interval

**CPU Usage (Pi 4):** ~55-80% total
**Memory:** ~270MB (within Pi 3 limits)

---

### 7. Security

**Credentials:** Environment variables or config files (not hardcoded)

**Data Privacy:**
- GPS shared only via encrypted channels (HTTPS/TLS)
- Camera processed locally (no cloud upload)
- In-memory only (no file logging)

**Network:** STARTTLS for email, HTTPS for Telegram

---

### 8. Testing

**Unit Tests:** GPS parsing, distance calculation, cooldown logic

**Integration Tests:** Sensor failures, network outages, concurrent alerts

**Hardware Tests:** Obstacle accuracy, fall detection, GPS positioning, object recognition

---

### 9. Deployment

**File Structure:**
```
blind-assistance-belt/
├── main.py
├── requirements.txt
├── sounds/ (audio alerts)
└── models/ (DNN files)
```

**Installation:** System updates, dependencies, interface enablement, credential configuration

**Autostart:** systemd service for boot-time launch

---

### 10. Key Design Decisions

**Multi-Threading:** Separate threads for GPS and camera allow concurrent processing without blocking

**External OpenCV Window:** Better performance than Tkinter embedding for video display

**Cooldown Mechanisms:** Time-based delays prevent alert fatigue and notification spam

**Static Fallback GPS:** Provides location data even when GPS unavailable (indoor use)

**MobileNet SSD:** Optimized for embedded devices, good speed/accuracy balance for Pi

---

### 11. Future Enhancements

- Cloud integration with MQTT for remote monitoring
- Machine learning for personalized fall detection
- Offline map caching
- Voice interface (speech recognition and TTS)
- Mobile app for caregivers

---

**Document Version:** 1.0  
**Last Updated:** 2026-02-14
