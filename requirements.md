# Requirements Specification
## Blind Assistance Smart Belt

### 1. Project Overview

**Purpose:** Real-time wearable assistive device for visually impaired individuals providing obstacle detection, fall monitoring, emergency alerts, and live location tracking.

**Target Platform:** Raspberry Pi 3/4  
**Primary Users:** Visually impaired individuals, senior citizens

---

### 2. Functional Requirements

#### 2.1 Obstacle Detection

- Detect obstacles using three ultrasonic sensors (left, right, back)
- Trigger alert when distance < 20cm
- Play directional audio alerts
- Send location-tagged notifications via Telegram and Email
- Implement cooldown to prevent alert spam

#### 2.2 Fall Detection

- Monitor acceleration data from MPU6050 motion sensor
- Detect falls based on acceleration magnitude thresholds
- Send immediate alerts with GPS coordinates
- Display real-time status in GUI

#### 2.3 GPS Location Tracking

- Continuously read GPS data from serial interface
- Parse NMEA sentences for coordinates
- Fallback to static coordinates when unavailable
- Automatic reconnection on failures
- Generate map links for all alerts

#### 2.4 Object Recognition

- Capture and process video frames using MobileNet SSD
- Detect multiple object classes (person, car, dog, etc.)
- Classify dangerous objects (car, dog, train)
- Play object-specific audio alerts
- Display bounding boxes on detected objects

#### 2.5 Emergency Alert System

- Monitor emergency button via GPIO
- Send immediate alerts when pressed
- Include GPS coordinates in messages
- Prevent accidental multiple triggers

#### 2.6 Multi-Channel Notifications

- Send alerts via Telegram and Email
- Include alert type, timestamp, GPS location, and sensor data
- Handle failures gracefully
- Log all notification attempts

#### 2.7 User Interface

- Display real-time sensor status (MPU6050, ultrasonic, GPS, emergency button)
- Camera controls and external video feed display
- Scrollable event log with timestamps
- Map view button for GPS location
- Regular status updates

---

### 3. Non-Functional Requirements

#### 3.1 Performance
- Low latency sensor processing
- Minimum 20 FPS camera processing
- Fast alert delivery (< 3 seconds)
- Multi-threaded concurrent operations

#### 3.2 Reliability
- Continuous operation without crashes
- Graceful handling of sensor failures
- Automatic reconnection on GPS failures
- Timeout protection for sensors

#### 3.3 Usability
- Clear and distinguishable audio alerts
- Easy emergency button access
- Clear visual status indicators
- Test mode for development

#### 3.4 Security
- Secure credential management
- Encrypted communication channels (HTTPS, TLS)

#### 3.5 Maintainability
- Modular code structure
- Configurable thresholds
- Detailed error logging

---

### 4. Hardware Requirements

| Component | Interface | Quantity |
|-----------|-----------|----------|
| Raspberry Pi 3/4 | - | 1 |
| Camera Module | CSI | 1 |
| MPU6050 (IMU) | I²C | 1 |
| GPS Module | UART Serial | 1 |
| HC-SR04 Ultrasonic | GPIO | 3 |
| Emergency Button | GPIO | 1 |
| Buzzer/Speaker | Audio Jack | 1 |

**Power:** 5V, 3A power supply or 10,000mAh power bank

**Note:** Use voltage divider for HC-SR04 ECHO pins (5V → 3.3V)

---

### 5. Software Requirements

**Operating System:** Raspberry Pi OS with Python 3.7+

**Key Libraries:**
- OpenCV (computer vision)
- NumPy (numerical processing)
- RPi.GPIO (hardware interface)
- smbus (I²C communication)
- pyserial (GPS communication)
- requests (API calls)

**AI Model:** MobileNet SSD (object detection)

**Audio Files:** Directional and object-specific WAV alerts

**System Setup:** Enable I²C, Serial, and Camera interfaces

---

### 6. Configuration

**Telegram:** Bot token and chat ID  
**Email:** SMTP server, credentials, recipient address  
**GPS:** Serial port and fallback coordinates  
**Thresholds:** Configurable sensor limits and cooldown periods

---

### 7. Success Criteria

- Accurate obstacle detection with appropriate alerts
- Reliable fall detection with GPS-tagged notifications
- Functional object recognition for dangerous items
- Immediate emergency button response
- Stable continuous operation (8+ hours)
- Real-time GUI updates
- Graceful handling of sensor failures

---

### 8. Future Enhancements

- Voice command integration
- Mobile app for remote monitoring
- Cloud-based analytics
- Battery monitoring
- Vibration feedback
- Multi-language support
- Personalized ML models

---

**Document Version:** 1.0  
**Last Updated:** 2026-02-14
