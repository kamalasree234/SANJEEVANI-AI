# Design Document: Sanjeevani AI

## 1. System Overview

Sanjeevani AI is a smartphone-based emergency response system designed to solve the critical problem of delayed medical response in Indian road accidents. Every year, over 1.5 lakh people lose their lives in road accidents, often due to delayed emergency response when victims are unconscious or unable to call for help.

The system automatically detects accidents using smartphone sensors (accelerometer and gyroscope) and AI-powered motion analysis. Upon detection, it instantly triggers a coordinated emergency response involving ambulances, hospitals, and family members - all without requiring any manual intervention from the victim.

### Key Capabilities

- **Automatic Detection**: Uses on-device AI to detect accidents within 2 seconds
- **Zero Manual Intervention**: Works even when victim is unconscious
- **Offline Support**: Core functionality works without internet via SMS fallback
- **Smart Coordination**: Automatically matches nearest ambulance and available hospital
- **Real-Time Tracking**: Live location updates and ETA for all stakeholders
- **Multilingual**: Supports 10 Indian languages for inclusive access

### Problem Solved

Traditional emergency response relies on manual calls, which fail when:
- Victim is unconscious or severely injured
- Victim is alone with no witnesses
- Victim is in shock and unable to communicate
- Poor coordination between ambulance and hospital causes delays

Sanjeevani AI eliminates these delays by automating the entire emergency response chain from detection to hospital delivery.

## 2. System Architecture

### Layered Architecture

The system follows a three-tier architecture designed for reliability, scalability, and offline capability:

```
┌─────────────────────────────────────────────────────────────┐
│                    USER LAYER                                │
│  - Smartphone Sensors (Accelerometer, Gyroscope, GPS)       │
│  - SOS Triggers (Button, Voice, Shake)                      │
│  - User Interface (Flutter App)                             │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│              DATA COLLECTION LAYER                           │
│  - Sensor Monitor (50Hz sampling)                           │
│  - Location Service (GPS + Network)                         │
│  - Network Status Monitor                                   │
│  - Local Storage (SQLite)                                   │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│               PROCESSING LAYER                               │
│  - Accident Detection Engine (TensorFlow Lite)              │
│  - Rule-Based Fallback Logic                                │
│  - Confirmation Manager (30-second window)                  │
│  - Emergency Manager (Alert orchestration)                  │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│            BACKEND / CLOUD LAYER                             │
│  - Firebase Firestore (Database)                            │
│  - Cloud Functions (Emergency Orchestration)                │
│  - Firebase Authentication                                  │
│  - Firebase Cloud Messaging (Notifications)                 │
│  - Google Maps API (Routing, Traffic)                       │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│          RESPONSE & SERVICE LAYER                            │
│  - Ambulance Dispatch System                                │
│  - Hospital Alert System                                    │
│  - Family Notification System                               │
│  - SMS Gateway (Offline Fallback)                           │
│  - Real-Time Tracking Dashboard                             │
└─────────────────────────────────────────────────────────────┘
```

### Architecture Principles

1. **Offline-First**: Core accident detection runs on-device without internet dependency
2. **Multi-Channel Redundancy**: Alerts sent via FCM, SMS, and in-app simultaneously
3. **Progressive Enhancement**: Graceful degradation from real-time to SMS fallback
4. **Minimal Latency**: Sub-3-second response time from detection to alert dispatch
5. **Serverless Scalability**: Firebase auto-scales to millions of users
6. **Privacy-Preserving**: End-to-end encryption for sensitive medical data

## 3. Components Description

### Mobile App (Flutter)

The mobile application is the primary user interface built with Flutter for cross-platform support.

**Key Responsibilities**:
- Continuous sensor monitoring in background
- On-device accident detection using TensorFlow Lite
- User confirmation interface (30-second window)
- Location tracking and sharing
- Emergency alert dispatch
- Real-time tracking display
- Offline data caching

**Technology**: Flutter 3.x with Dart, Material Design UI

### Sensor Module

Monitors smartphone motion sensors to detect abnormal patterns indicating accidents.

**Sensors Used**:
- **Accelerometer**: Measures linear acceleration in 3 axes (x, y, z)
- **Gyroscope**: Measures rotational velocity in 3 axes

**Sampling Rate**: 50Hz (50 readings per second)

**Detection Triggers**:
- Sudden deceleration >4G (collision)
- High rotational velocity >3 rad/s (rollover)
- Combined acceleration + rotation (complex accidents)

### Location Module (GPS)

Captures and tracks user location for emergency response.

**Features**:
- High-accuracy GPS (<10m accuracy)
- Network-based fallback when GPS unavailable
- Reverse geocoding (coordinates to address)
- Continuous tracking during emergencies
- Heading, speed, altitude capture

**Technology**: Flutter geolocator package, Google Maps API

### Detection Engine (AI Model)

TensorFlow Lite model running on-device for real-time accident detection.

**Model Specifications**:
- Architecture: 1D Convolutional Neural Network (CNN)
- Input: 5-second sensor window (250 samples × 6 channels)
- Output: Accident probability and type (collision, rollover, sudden_stop)
- Model Size: ~5MB (INT8 quantized)
- Inference Time: <100ms
- Accuracy: 95%+

**Training Data**:
- Public datasets (UCI HAR, Kaggle accident data)
- Custom crash test data
- Normal driving data (10,000+ hours)
- Augmented data (time warping, rotation, noise)

**Rule-Based Fallback**:
- If acceleration >4G → trigger detection
- If gyroscope >3 rad/s → trigger detection
- Provides redundancy if ML model fails

### Backend (Firebase)

Cloud infrastructure providing serverless, auto-scaling backend services.

**Firebase Services**:

1. **Firestore Database**
   - NoSQL document database
   - Collections: users, emergencies, ambulances, hospitals
   - Real-time synchronization
   - Offline persistence
   - Geohash-based location queries

2. **Cloud Functions**
   - Serverless compute (Node.js/TypeScript)
   - Emergency orchestration logic
   - Ambulance matching algorithm
   - Hospital selection logic
   - Route optimization
   - Auto-scales based on load

3. **Authentication**
   - Phone number + OTP verification
   - Optional biometric authentication
   - Session management with JWT tokens
   - Role-based access control

4. **Cloud Storage**
   - AI model updates
   - User documents (medical records)
   - Emergency photos/videos

### Notification System

Multi-channel alert system ensuring message delivery even in poor network conditions.

**Channels**:

1. **Firebase Cloud Messaging (FCM)**
   - Push notifications to Android/iOS
   - High priority for emergency alerts
   - Delivery confirmation
   - Works when app in background

2. **In-App Notifications**
   - Real-time updates within app
   - Status changes, ETA updates
   - Interactive notifications

3. **SMS Fallback**
   - Automatic switch when internet unavailable
   - Pre-configured emergency numbers
   - Location coordinates in message
   - Works on any phone network

### SMS Gateway

External SMS service for offline alert delivery.

**Providers**: Twilio or MSG91

**Features**:
- Bulk SMS sending
- Delivery reports
- International support
- API integration

**Message Format**:
```
SANJEEVANI EMERGENCY
Name: [User Name]
Blood: [Blood Type]
Location: [Address]
GPS: [Lat, Lng]
Time: [Timestamp]
Call: [Phone]
```

### Dashboard (Flutter Web)

Real-time monitoring interface for emergency coordinators and family members.

**Features**:
- Live map showing all active emergencies
- Ambulance and victim location tracking
- Status timeline for each emergency
- Hospital capacity monitoring
- Analytics and reports
- Multilingual support

**Technology**: Flutter Web, Google Maps JavaScript API

## 4. Data Flow

### Complete Emergency Response Workflow

#### Step 1: Sensor Data Collection (Continuous)

- Sensor Monitor samples accelerometer and gyroscope at 50Hz
- Data buffered in 5-second sliding window (250 samples)
- When sudden motion detected (>4G), trigger analysis
- GPS captures current location continuously

#### Step 2: Accident Detection (0-2 seconds)

- Sensor data fed to TensorFlow Lite model
- Model analyzes motion patterns
- Output: isAccident (yes/no), confidence (0-1), type (collision/rollover/sudden_stop)
- If confidence >85%, proceed to confirmation
- Rule-based detector runs in parallel as backup

#### Step 3: User Confirmation (2-32 seconds)

- Display full-screen confirmation alert
- 30-second countdown with audio alerts
- User can tap "I'm Safe" to cancel
- If no response after 30 seconds, auto-confirm emergency
- Capture GPS location and address

#### Step 4: Alert Generation (32-35 seconds)

- Emergency Manager creates emergency record
- Store in local SQLite database
- Attempt sync to Firebase Firestore
- Capture user medical profile from cache
- Generate unique emergency ID

#### Step 5: Location Sharing (32-35 seconds)

- GPS service captures high-accuracy location
- Reverse geocoding converts to address
- Include nearby landmarks
- Capture heading, speed, altitude
- Start continuous location tracking (update every 10 seconds)

#### Step 6: Multi-Channel Alert Dispatch (35-40 seconds)

**Online Mode** (Internet Available):
- Send emergency data to Firebase Cloud Functions via FCM
- Cloud Function receives and validates data
- Creates Firestore emergency document
- Triggers parallel workflows

**Offline Mode** (No Internet):
- Detect no connectivity
- Format SMS with location and user details
- Send SMS to pre-configured emergency numbers
- Store emergency in local queue for later sync

#### Step 7: Ambulance Dispatch (40-100 seconds)

- Cloud Function queries Firestore for ambulances within 10km
- Filter by availability (isAvailable = true)
- Calculate distance from each to accident location
- Select nearest available ambulance
- Send FCM notification to ambulance driver
- Include: victim location, medical info, optimal route
- Driver has 60 seconds to accept
- If no response, escalate to next nearest ambulance

#### Step 8: Hospital Notification (60-150 seconds)

- Query Firestore for hospitals within 20km
- Filter by available beds and emergency capacity
- Rank by: distance (40%), specialization (30%), rating (20%), capacity (10%)
- Send preliminary patient info to top-ranked hospital
- Include: medical profile, accident type, ETA
- Hospital has 90 seconds to confirm
- If no confirmation, auto-select nearest government hospital
- Send hospital location and route to ambulance

#### Step 9: Family Notification (40-50 seconds)

- Retrieve emergency contacts from user profile
- Send multi-channel notifications (FCM, SMS, in-app)
- Include: accident location, ambulance details, hospital destination
- Provide tracking link for real-time monitoring
- Retry failed notifications every 5 minutes

#### Step 10: Real-Time Tracking (Ongoing)

- Ambulance app sends location updates every 10 seconds
- Updates stored in Firestore and broadcast via FCM
- Recalculate ETA based on current location and traffic
- Display on family tracking dashboard
- Notify stakeholders of significant ETA changes (>5 min)

#### Step 11: Status Updates (Ongoing)

- **Ambulance Arrives**: Status = AMBULANCE_ARRIVED, notify all
- **Patient Loaded**: Status = EN_ROUTE_HOSPITAL, update ETA
- **Ambulance at Hospital**: Status = DELIVERED, notify family
- **Hospital Confirms**: Status = COMPLETED, close emergency
- Each status change triggers notifications

#### Step 12: Emergency Completion

- Hospital confirms patient receipt
- Final notifications sent to family
- Ambulance marked as available
- Emergency record archived
- Analytics data collected

### Offline Mode Data Flow

When internet is unavailable:

1. **Local Detection**: Runs normally on-device (no internet needed)
2. **SMS Dispatch**: System sends SMS to emergency contacts with location
3. **Local Storage**: Emergency record stored in SQLite
4. **Background Sync**: Retry internet connection every 60 seconds
5. **Sync on Recovery**: Upload emergency data when connectivity restored

## 5. AI Model Design

### TensorFlow Lite Model Architecture

The accident detection model uses a 1D Convolutional Neural Network optimized for time-series sensor data:

```
Input: [250 samples, 6 channels]
  ↓
Conv1D: 64 filters, kernel=5, ReLU
  ↓
MaxPooling1D: pool_size=2
  ↓
Conv1D: 128 filters, kernel=5, ReLU
  ↓
MaxPooling1D: pool_size=2
  ↓
Conv1D: 256 filters, kernel=3, ReLU
  ↓
GlobalAveragePooling1D
  ↓
Dense: 128 units, ReLU, dropout=0.3
  ↓
Dense: 64 units, ReLU, dropout=0.2
  ↓
Output: 3 units (collision, rollover, sudden_stop), Softmax
```

### Model Training

**Training Data Sources**:
- UCI Human Activity Recognition Dataset (normal driving)
- Kaggle Accident Detection Dataset (simulated accidents)
- NHTSA Crash Test Data (real crash recordings)
- Custom crash tests with instrumented vehicles
- Volunteer driver data (10,000+ hours)

**Data Preprocessing**:
- Normalize sensor values to [-1, 1]
- Apply bandpass filter (0.5-20Hz) to remove noise
- Segment into 5-second windows with 50% overlap
- Balance classes (50% normal, 50% accidents)

**Training Process**:
- Framework: TensorFlow 2.x
- Optimizer: Adam (learning_rate=0.001)
- Loss: Categorical Cross-Entropy
- Batch Size: 64
- Epochs: 100 with early stopping
- Validation Split: 20%

**Model Optimization**:
- Post-training quantization (INT8)
- Pruning (remove 30% of weights)
- Convert to TensorFlow Lite format
- Final model size: ~5MB
- Inference time: <100ms on mid-range phones

### On-Device Inference

The model runs entirely on the smartphone without internet:

1. Sensor Monitor collects 5 seconds of data (250 samples)
2. Data preprocessed (normalization, filtering)
3. TensorFlow Lite interpreter runs inference
4. Output: accident probability and type
5. If probability >85%, trigger confirmation window

### Model Updates

- New models stored in Firebase Storage
- App checks for updates on launch
- Download and cache new model
- Fallback to bundled model if download fails
- A/B testing for new model versions

## 6. Offline Support

### Offline-First Design

The system is designed to work reliably even without internet connectivity:

**On-Device Processing**:
- Accident detection runs entirely on-device using TensorFlow Lite
- No internet required for sensor monitoring and analysis
- All critical logic embedded in mobile app

**Local Data Storage**:
- SQLite database stores user profile, emergency contacts, settings
- Emergency records cached locally before cloud sync
- Last known location stored for offline scenarios

**SMS Fallback**:
- When internet unavailable, automatically switch to SMS
- Pre-configured emergency numbers stored locally
- SMS sent via device's native SMS capability (no internet required)
- Message includes location coordinates and user details

### Offline Emergency Flow

1. **Detection**: Sensor monitoring and AI detection work normally (no internet needed)
2. **Confirmation**: 30-second confirmation window displayed locally
3. **SMS Alert**: System detects no internet, sends SMS to emergency contacts
4. **Local Queue**: Emergency record stored in SQLite queue
5. **Background Sync**: Monitor connectivity every 60 seconds
6. **Sync on Recovery**: Upload emergency data when internet restored

### SMS Message Format

```
SANJEEVANI EMERGENCY
Name: Rajesh Kumar
Blood: O+
Location: MG Road, Bangalore
GPS: 12.9716, 77.5946
Time: 15:30, 15 Feb 2026
Call: +91-9876543210
```

### Connectivity Recovery

When internet is restored:
- Upload queued emergency records to Firestore
- Sync user profile changes
- Enable real-time tracking
- Activate ambulance matching
- Resume FCM notifications

## 7. Scalability & Deployment

### Cloud Infrastructure

**Firebase Services** provide serverless, auto-scaling infrastructure:

1. **Firestore Database**
   - NoSQL document database
   - Automatic scaling to millions of users
   - 99.95% uptime SLA
   - Supports 1M+ concurrent connections

2. **Cloud Functions**
   - Serverless compute (Node.js/TypeScript)
   - Auto-scales from 0 to 10,000+ instances
   - Cold start <1 second
   - Pay-per-use pricing

3. **Firebase Authentication**
   - Supports 10M+ users
   - Phone + OTP verification
   - Session management with JWT

4. **Cloud Messaging (FCM)**
   - 1M+ messages per second
   - Push notifications to Android/iOS
   - Delivery confirmation

### Scalability Strategy

**Horizontal Scaling**:
- Firebase auto-scales based on load
- No manual server provisioning
- Handles traffic spikes automatically

**Database Optimization**:
- Composite indexes for common queries
- Geohash-based location queries (O(log n))
- Denormalization for read-heavy operations
- TTL policies for old records

**Caching**:
- User profiles cached on device
- Ambulance/hospital data cached (5-min TTL)
- CDN for static assets

**Geographic Distribution**:
- Firebase regions across India
- Nearest region routing for low latency
- Failover to backup regions

### Deployment Architecture

**Mobile App**:
- Android: Google Play Store (Android 8.0+)
- iOS: Apple App Store (iOS 13.0+)
- Over-the-air updates for non-critical changes
- Gradual rollout (10% → 50% → 100%)

**Backend**:
- Production: sanjeevani-prod
- Staging: sanjeevani-staging
- Development: sanjeevani-dev
- CI/CD with GitHub Actions
- Blue-green deployment for zero downtime

### Performance Targets

- Emergency Detection to Alert: <3 seconds
- Ambulance Matching: <10 seconds
- Hospital Selection: <15 seconds
- Location Update Latency: <2 seconds
- Database Query Time: <500ms
- API Response Time: <1 second (p95)

## 8. Security & Privacy

### Data Security

**Encryption**:
- Data at Rest: Firestore encrypts all data (AES-256)
- Data in Transit: TLS 1.3 for all network communications
- Local Storage: SQLite encrypted with SQLCipher
- Medical Records: Double-encrypted with user-specific keys

**Authentication**:
- Phone number verification with OTP
- Optional biometric (fingerprint, face)
- Session tokens expire after 30 days
- Multi-device support with token revocation

**Access Control**:
- Role-Based Access Control (RBAC)
- User: Access own profile and emergencies
- Ambulance Driver: Access assigned emergencies only
- Hospital Staff: Access incoming patients only
- Admin: Full system access with audit logging

### Privacy Protection

**Data Minimization**:
- Collect only essential information
- No location tracking when not in emergency
- Sensor data discarded after analysis
- Emergency records anonymized after 90 days

**User Consent**:
- Explicit consent for location access
- Opt-in for family notifications
- Control over medical information sharing
- Right to delete account and data

**Compliance**:
- GDPR compliance for data protection
- HIPAA considerations for medical data
- Indian IT Act compliance
- Regular security audits

**Access Logging**:
- All data access logged with timestamp
- Audit trail for emergency responder access
- Alerts for suspicious access patterns
- Logs retained for 1 year

## 9. Technology Stack

### Mobile Application
- **Framework**: Flutter 3.x (Dart)
- **State Management**: Riverpod
- **Local Database**: SQLite with sqflite
- **AI/ML**: TensorFlow Lite
- **Location**: geolocator package
- **Maps**: Google Maps Flutter plugin
- **Notifications**: firebase_messaging

### Backend Services
- **Platform**: Firebase (Google Cloud)
- **Database**: Firestore (NoSQL)
- **Compute**: Cloud Functions (Node.js/TypeScript)
- **Authentication**: Firebase Auth (Phone + OTP)
- **Messaging**: Firebase Cloud Messaging (FCM)
- **Storage**: Cloud Storage
- **Analytics**: Firebase Analytics

### External APIs
- **Maps & Routing**: Google Maps API
- **SMS Gateway**: Twilio or MSG91
- **Payment**: Razorpay (for premium features)

### Development Tools
- **Version Control**: Git with GitHub
- **CI/CD**: GitHub Actions
- **Testing**: Flutter test, flutter_driver, Appium
- **Monitoring**: Firebase Crashlytics, Cloud Monitoring
- **Project Management**: Jira, Confluence

## 10. Advantages of the Design

### 1. Fast Response Time
- Sub-3-second detection using on-device AI
- Automatic activation without manual intervention
- Multi-channel alerts ensure delivery
- Nearest ambulance dispatched within 10 seconds

### 2. Fully Automated System
- Works even when user unconscious
- End-to-end automation from detection to hospital
- Intelligent ambulance matching and hospital selection
- Automatic family notifications and updates

### 3. Works in Low-Network Areas
- Offline-first design with on-device AI
- SMS fallback when internet unavailable
- Local storage with background sync
- Graceful degradation based on network conditions

### 4. Scalable Across India
- Serverless Firebase architecture auto-scales
- No hardware required - works on standard smartphones
- Cost-effective pay-per-use pricing
- Geographic distribution for low latency

### 5. Privacy-Preserving
- End-to-end encryption for medical data
- Minimal data collection
- User control over data sharing
- GDPR and HIPAA compliant

### 6. Multilingual and Inclusive
- Supports 10 Indian languages
- Voice alerts in user's language
- Regional SMS formatting
- Accessible UI with Material Design

### 7. Real-Time Coordination
- Live ambulance tracking (10-second updates)
- Dynamic ETA calculation with traffic
- Status notifications to all stakeholders
- Dashboard for monitoring all emergencies

### 8. Reliable and Accurate
- 95%+ AI model accuracy
- <2% false positive rate
- <1% false negative rate for severe accidents
- Multi-sensor fusion for robust detection

### 9. Cost-Effective
- No hardware costs (uses existing smartphones)
- Serverless backend (pay only for usage)
- Free for users (ad-supported or government-funded)
- Linear cost scaling with users

### 10. Extensible and Future-Proof
- Modular architecture for easy feature additions
- Over-the-air AI model updates
- API-first design for integrations
- Support for wearables and vehicle systems in future

---

## Conclusion

The Sanjeevani AI system provides a comprehensive, scalable, and reliable solution for automatic accident detection and emergency response in India. The design prioritizes offline capability, low latency, and user privacy while ensuring seamless coordination between victims, ambulances, hospitals, and families.

The use of on-device AI, Firebase serverless architecture, and multi-channel communication ensures the system works reliably even in challenging network conditions common in rural India. The modular architecture allows for future enhancements such as integration with vehicle systems, wearable devices, and government emergency services.

The system is designed to scale to millions of users while maintaining sub-3-second response times and 99.9% reliability, ultimately saving lives by eliminating delays in emergency response.
