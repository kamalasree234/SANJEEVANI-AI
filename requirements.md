# Requirements Document: Sanjeevani AI

## Introduction

Sanjeevani AI is an emergency response system designed to automatically detect road accidents using smartphone sensors and instantly trigger emergency services without manual intervention. The system addresses the critical problem of delayed medical response in Indian road accidents, where over 1.5 lakh people die annually due to delayed treatment. The system must work reliably in low-connectivity environments, support multiple languages, and coordinate between victims, ambulances, hospitals, and family members.

## Glossary

- **Sanjeevani_System**: The complete emergency response platform including mobile app, backend services, and AI detection
- **Accident_Detector**: The AI-powered component that analyzes sensor data to detect accidents
- **Alert_Manager**: Component responsible for sending notifications to all stakeholders
- **Ambulance_Matcher**: Service that identifies and assigns the nearest available ambulance
- **Hospital_Selector**: Service that identifies hospitals with available capacity
- **Route_Optimizer**: Service that calculates optimal routes considering traffic conditions
- **User**: Person using the mobile application (driver, biker, pedestrian)
- **Emergency_Contact**: Pre-registered family member or friend to be notified
- **Ambulance_Service**: Emergency medical service provider with registered vehicles
- **Hospital**: Medical facility registered in the system with capacity information
- **Confirmation_Window**: 30-second period allowing user to cancel false positive detections
- **SOS_Trigger**: Manual emergency activation mechanism (button, voice, shake gesture)
- **Offline_Mode**: System operation using SMS when internet connectivity is unavailable
- **Emergency_Dashboard**: Real-time monitoring interface for emergency response coordination

## Requirements

### Requirement 1: Automatic Accident Detection

**User Story:** As a road user, I want the system to automatically detect when I'm in an accident, so that emergency services are alerted even if I'm unconscious or unable to call for help.

#### Acceptance Criteria

1. WHEN the smartphone experiences sudden deceleration exceeding 4G, THEN THE Accident_Detector SHALL analyze accelerometer and gyroscope data to determine if an accident occurred
2. WHEN sensor data patterns match accident signatures, THEN THE Accident_Detector SHALL trigger the confirmation window within 2 seconds
3. WHEN the Accident_Detector processes sensor data, THEN THE System SHALL use on-device TensorFlow Lite models without requiring internet connectivity
4. WHEN multiple sensor readings are analyzed, THEN THE Accident_Detector SHALL achieve minimum 95% accuracy in distinguishing accidents from normal driving events
5. WHEN an accident is detected, THEN THE System SHALL capture GPS coordinates, timestamp, and sensor data snapshot

### Requirement 2: False Alert Prevention

**User Story:** As a user, I want a confirmation window before alerts are sent, so that false detections from speed bumps or sudden braking don't trigger unnecessary emergency responses.

#### Acceptance Criteria

1. WHEN an accident is detected, THEN THE System SHALL display a confirmation screen for 30 seconds before sending alerts
2. WHEN the user cancels within the confirmation window, THEN THE System SHALL abort all emergency notifications and log the false positive
3. WHEN the 30-second confirmation window expires without user response, THEN THE System SHALL automatically proceed with emergency alert dispatch
4. WHEN the confirmation screen is displayed, THEN THE System SHALL provide audio countdown alerts in the user's selected language
5. WHEN the device is locked during confirmation, THEN THE System SHALL display the confirmation screen over the lock screen with emergency override

### Requirement 3: Manual Emergency Activation

**User Story:** As a user, I want multiple ways to manually trigger an SOS alert, so that I can call for help in emergencies that aren't automatically detected.

#### Acceptance Criteria

1. WHEN the user presses the dedicated SOS button for 3 seconds, THEN THE System SHALL immediately trigger emergency alert dispatch
2. WHEN the user speaks the voice command "Emergency Help" or regional language equivalent, THEN THE System SHALL activate SOS within 2 seconds
3. WHEN the user shakes the device with force exceeding 3G for 2 consecutive seconds, THEN THE System SHALL trigger the SOS confirmation prompt
4. WHEN any SOS trigger is activated, THEN THE System SHALL bypass the 30-second confirmation window and proceed immediately
5. WHEN SOS is triggered manually, THEN THE System SHALL capture current location, timestamp, and trigger method

### Requirement 4: Location Services

**User Story:** As an emergency responder, I want accurate real-time location of the accident victim, so that I can reach them quickly without delays.

#### Acceptance Criteria

1. WHEN an emergency is triggered, THEN THE System SHALL capture GPS coordinates with accuracy within 10 meters
2. WHEN GPS signal is weak or unavailable, THEN THE System SHALL use network-based location with fallback to last known location
3. WHEN location is captured, THEN THE System SHALL include address details, nearby landmarks, and road information
4. WHEN the victim's location changes, THEN THE System SHALL update coordinates every 10 seconds and broadcast to all stakeholders
5. WHEN location data is transmitted, THEN THE System SHALL include altitude, heading, and speed information when available

### Requirement 5: Multi-Channel Alert Dispatch

**User Story:** As a system administrator, I want alerts sent through multiple channels simultaneously, so that emergency services are notified even when some communication channels fail.

#### Acceptance Criteria

1. WHEN an emergency is confirmed, THEN THE Alert_Manager SHALL send notifications via Firebase Cloud Messaging, SMS, and in-app alerts simultaneously
2. WHEN internet connectivity is available, THEN THE Alert_Manager SHALL prioritize real-time data transmission through Firebase
3. WHEN internet connectivity is unavailable, THEN THE Alert_Manager SHALL automatically switch to SMS-based alerts within 5 seconds
4. WHEN SMS is sent, THEN THE Alert_Manager SHALL include victim location coordinates, emergency type, and callback number
5. WHEN any alert channel fails, THEN THE Alert_Manager SHALL retry through alternative channels and log the failure

### Requirement 6: Ambulance Matching and Dispatch

**User Story:** As an accident victim, I want the nearest available ambulance dispatched to my location, so that I receive medical help as quickly as possible.

#### Acceptance Criteria

1. WHEN an emergency is confirmed, THEN THE Ambulance_Matcher SHALL identify all ambulances within 10km radius of the accident location
2. WHEN multiple ambulances are available, THEN THE Ambulance_Matcher SHALL select the nearest ambulance that is currently available
3. WHEN the selected ambulance accepts the request, THEN THE System SHALL send victim location, accident details, and optimal route to the ambulance
4. WHEN no ambulance responds within 60 seconds, THEN THE Ambulance_Matcher SHALL automatically escalate to the next nearest ambulance
5. WHEN an ambulance is dispatched, THEN THE System SHALL notify the victim and emergency contacts with ambulance details and ETA

### Requirement 7: Hospital Selection and Notification

**User Story:** As an ambulance driver, I want to know which hospital has available capacity, so that I don't waste time going to hospitals that cannot accept patients.

#### Acceptance Criteria

1. WHEN an ambulance is dispatched, THEN THE Hospital_Selector SHALL identify hospitals within 20km with available emergency capacity
2. WHEN multiple hospitals are available, THEN THE Hospital_Selector SHALL rank them by distance, specialization, and current capacity
3. WHEN a hospital is selected, THEN THE System SHALL send patient preliminary information and estimated arrival time to the hospital
4. WHEN the hospital confirms readiness, THEN THE System SHALL provide the ambulance with hospital location and optimal route
5. WHEN no hospital confirms within 90 seconds, THEN THE Hospital_Selector SHALL automatically select the nearest government hospital

### Requirement 8: Traffic-Aware Route Optimization

**User Story:** As an ambulance driver, I want the fastest route considering current traffic conditions, so that I can reach the victim and hospital in minimum time.

#### Acceptance Criteria

1. WHEN an ambulance is dispatched, THEN THE Route_Optimizer SHALL calculate the optimal route using Google Maps API with real-time traffic data
2. WHEN traffic conditions change, THEN THE Route_Optimizer SHALL recalculate routes every 60 seconds and notify the driver of better alternatives
3. WHEN calculating routes, THEN THE Route_Optimizer SHALL prioritize highways and main roads over narrow streets
4. WHEN multiple route options exist, THEN THE Route_Optimizer SHALL display estimated time for each route with traffic indicators
5. WHEN the ambulance deviates from the suggested route, THEN THE Route_Optimizer SHALL automatically recalculate from the current position

### Requirement 9: Real-Time Tracking and ETA Updates

**User Story:** As a family member, I want to track the ambulance location and know when it will reach my loved one, so that I can coordinate and stay informed.

#### Acceptance Criteria

1. WHEN an ambulance is dispatched, THEN THE System SHALL broadcast ambulance location updates every 10 seconds to all stakeholders
2. WHEN location updates are received, THEN THE System SHALL calculate and display ETA to accident location and hospital
3. WHEN the ambulance reaches the victim, THEN THE System SHALL notify all stakeholders and update status to "Patient Picked Up"
4. WHEN the ambulance is en route to hospital, THEN THE System SHALL display live tracking with updated hospital ETA
5. WHEN the ambulance reaches the hospital, THEN THE System SHALL notify all stakeholders and mark the emergency as "Delivered to Hospital"

### Requirement 10: Family Notification System

**User Story:** As a user, I want my family members automatically notified in emergencies, so that they are aware of my situation and can provide support.

#### Acceptance Criteria

1. WHEN an emergency is confirmed, THEN THE Alert_Manager SHALL send notifications to all registered emergency contacts within 10 seconds
2. WHEN notifying emergency contacts, THEN THE Alert_Manager SHALL include victim location, accident details, ambulance information, and hospital destination
3. WHEN emergency contacts receive notifications, THEN THE System SHALL provide a tracking link to monitor the situation in real-time
4. WHEN the victim's status changes, THEN THE Alert_Manager SHALL send updates to emergency contacts automatically
5. WHEN emergency contacts are unreachable, THEN THE System SHALL retry notifications every 5 minutes for up to 30 minutes

### Requirement 11: Multilingual Support

**User Story:** As a user from a regional area, I want the app in my local language, so that I can understand and use all features without language barriers.

#### Acceptance Criteria

1. WHEN the user first launches the app, THEN THE System SHALL detect device language and offer to use that language or allow manual selection
2. WHEN displaying any user interface, THEN THE System SHALL support Hindi, English, Tamil, Telugu, Bengali, Marathi, Gujarati, Kannada, Malayalam, and Punjabi
3. WHEN sending voice alerts during confirmation window, THEN THE System SHALL use the user's selected language for audio prompts
4. WHEN sending SMS alerts, THEN THE System SHALL format messages in the recipient's preferred language when known
5. WHEN language is changed, THEN THE System SHALL update all interface elements immediately without requiring app restart

### Requirement 12: Offline Operation and SMS Fallback

**User Story:** As a user in a rural area with poor connectivity, I want emergency alerts to work even without internet, so that I can get help regardless of network conditions.

#### Acceptance Criteria

1. WHEN internet connectivity is unavailable, THEN THE System SHALL automatically switch to SMS-based emergency alerts within 5 seconds
2. WHEN operating in offline mode, THEN THE System SHALL send SMS to pre-configured emergency numbers with location coordinates
3. WHEN GPS is unavailable, THEN THE System SHALL include last known location and cell tower information in SMS alerts
4. WHEN connectivity is restored, THEN THE System SHALL synchronize all offline events with the backend within 30 seconds
5. WHEN in offline mode, THEN THE System SHALL store all emergency data locally and retry transmission every 60 seconds

### Requirement 13: Emergency Dashboard and Monitoring

**User Story:** As an emergency response coordinator, I want a real-time dashboard showing all active emergencies, so that I can monitor and coordinate responses effectively.

#### Acceptance Criteria

1. WHEN an emergency is triggered, THEN THE System SHALL display it on the emergency dashboard within 5 seconds
2. WHEN displaying emergencies, THEN THE Dashboard SHALL show victim location on map, ambulance status, hospital destination, and elapsed time
3. WHEN the coordinator selects an emergency, THEN THE Dashboard SHALL display detailed information including sensor data, photos, and communication logs
4. WHEN emergencies are resolved, THEN THE Dashboard SHALL move them to completed status and maintain records for 90 days
5. WHEN multiple emergencies occur simultaneously, THEN THE Dashboard SHALL prioritize display by severity and response time

### Requirement 14: User Registration and Profile Management

**User Story:** As a user, I want to register my medical information and emergency contacts, so that responders have critical information when helping me.

#### Acceptance Criteria

1. WHEN a user registers, THEN THE System SHALL collect name, phone number, blood type, medical conditions, allergies, and emergency contacts
2. WHEN the user updates profile information, THEN THE System SHALL synchronize changes to Firebase within 10 seconds
3. WHEN an emergency occurs, THEN THE System SHALL include user medical information in alerts sent to ambulances and hospitals
4. WHEN the user adds emergency contacts, THEN THE System SHALL verify phone numbers and allow up to 5 contacts
5. WHEN user data is stored, THEN THE System SHALL encrypt sensitive medical information using AES-256 encryption

### Requirement 15: Ambulance Service Registration and Management

**User Story:** As an ambulance service provider, I want to register my vehicles and update availability, so that I can receive emergency requests when available.

#### Acceptance Criteria

1. WHEN an ambulance service registers, THEN THE System SHALL collect service name, vehicle registration numbers, driver details, equipment list, and coverage area
2. WHEN an ambulance becomes available, THEN THE Driver SHALL update status through the app within 5 seconds
3. WHEN an emergency request is received, THEN THE System SHALL send notification to the ambulance with accept/reject options
4. WHEN an ambulance accepts a request, THEN THE System SHALL mark that ambulance as unavailable for other requests
5. WHEN an ambulance completes an emergency, THEN THE Driver SHALL update status and the System SHALL mark the ambulance as available

### Requirement 16: Hospital Registration and Capacity Management

**User Story:** As a hospital administrator, I want to register our facility and update emergency capacity, so that ambulances know when we can accept patients.

#### Acceptance Criteria

1. WHEN a hospital registers, THEN THE System SHALL collect hospital name, address, specializations, emergency capacity, and contact numbers
2. WHEN emergency capacity changes, THEN THE Hospital SHALL update available beds through the dashboard within real-time
3. WHEN an ambulance is bringing a patient, THEN THE System SHALL send preliminary patient information to the hospital 10 minutes before arrival
4. WHEN a hospital receives a patient, THEN THE Hospital SHALL confirm receipt through the system
5. WHEN a hospital is at full capacity, THEN THE System SHALL exclude it from hospital selection for new emergencies

### Requirement 17: System Scalability and Performance

**User Story:** As a system architect, I want the system to handle millions of users and thousands of concurrent emergencies, so that the service remains reliable as it grows.

#### Acceptance Criteria

1. WHEN the system is under normal load, THEN THE Backend SHALL process emergency triggers within 2 seconds
2. WHEN concurrent emergencies exceed 1000, THEN THE System SHALL maintain response times within 5 seconds
3. WHEN user base grows to 10 million users, THEN THE System SHALL handle authentication and profile operations without degradation
4. WHEN database queries are executed, THEN THE System SHALL use indexed queries and return results within 500 milliseconds
5. WHEN Firebase Cloud Functions are triggered, THEN THE System SHALL auto-scale to handle load spikes up to 10x normal traffic

### Requirement 18: Data Privacy and Security

**User Story:** As a user, I want my personal and medical information protected, so that my privacy is maintained while still enabling emergency response.

#### Acceptance Criteria

1. WHEN user data is stored, THEN THE System SHALL encrypt all personal and medical information using AES-256 encryption
2. WHEN data is transmitted, THEN THE System SHALL use TLS 1.3 or higher for all network communications
3. WHEN emergency responders access user information, THEN THE System SHALL log all access with timestamp and purpose
4. WHEN a user deletes their account, THEN THE System SHALL remove all personal data within 30 days while retaining anonymized emergency statistics
5. WHEN authentication is required, THEN THE System SHALL use Firebase Authentication with phone number verification and optional biometric authentication

### Requirement 19: Battery and Resource Optimization

**User Story:** As a user, I want the app to run efficiently without draining my battery, so that it can monitor for emergencies throughout the day.

#### Acceptance Criteria

1. WHEN the app is running in background, THEN THE System SHALL consume less than 5% battery per hour on average
2. WHEN monitoring sensors, THEN THE Accident_Detector SHALL use efficient sampling rates and sleep modes to minimize power consumption
3. WHEN GPS tracking is active, THEN THE System SHALL use adaptive location updates based on movement patterns
4. WHEN the device battery drops below 15%, THEN THE System SHALL reduce background monitoring frequency and notify the user
5. WHEN the app is idle, THEN THE System SHALL release unnecessary resources and minimize memory footprint to under 100MB

### Requirement 20: Testing and Quality Assurance

**User Story:** As a quality assurance engineer, I want comprehensive testing of accident detection, so that the system is reliable and accurate in real-world conditions.

#### Acceptance Criteria

1. WHEN the AI model is trained, THEN THE System SHALL validate against a test dataset of minimum 10,000 real accident scenarios
2. WHEN false positive rate is measured, THEN THE Accident_Detector SHALL maintain false positives below 2% across all driving conditions
3. WHEN false negative rate is measured, THEN THE Accident_Detector SHALL maintain false negatives below 1% for severe accidents
4. WHEN integration tests are executed, THEN THE System SHALL verify end-to-end emergency flow from detection to hospital delivery
5. WHEN load testing is performed, THEN THE System SHALL handle 10,000 concurrent users with 99.9% success rate
