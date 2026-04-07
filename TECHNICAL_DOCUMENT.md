# IGuardAI Technical Document

## 1. Vision
IGuardAI helps a user stay focused on a chosen learning goal (for example: "Learn topic X for 90 minutes").

When the user gets distracted (for example: watching unrelated videos, switching to non-goal apps, or leaving the study area), IGuardAI detects the distraction and issues a warning in real time.

## 2. Problem Statement
People often start learning sessions with intent, but lose focus due to:
- YouTube rabbit holes and unrelated browsing
- Phone notifications and app switching
- Context switching between productive and non-productive tasks

The project objective is to provide active focus protection using multimodal monitoring:
- Live webcam analysis
- Desktop activity monitoring
- Phone activity monitoring
- Optional home Wi-Fi level activity signals

## 3. Product Goals
- Define a clear focus goal and session duration.
- Detect distraction with low latency.
- Warn users quickly and non-intrusively.
- Provide session analytics (focus score, distraction timeline).
- Keep privacy and user consent as first-class requirements.

## 4. Non-Goals (Initial Versions)
- Hidden surveillance without user consent
- Enterprise-level parental control policies
- Full packet sniffing of all encrypted traffic
- Perfect distraction detection in every environment

## 5. Primary Use Cases
1. Student sets a goal: "Learn Operating Systems for 60 minutes".
2. System monitors webcam and desktop activity.
3. User opens unrelated entertainment content.
4. IGuardAI warns: "You are off-topic. Return to your goal?"
5. User confirms and resumes study.
6. Session summary shows distraction events and focus quality.

## 6. Functional Requirements
### 6.1 Goal and Session Management
- User can create a session:
  - Goal name/topic
  - Planned duration
  - Optional allowed websites/apps list
- Start, pause, resume, stop session

### 6.2 Webcam Monitoring
- Capture live webcam stream (local device).
- Run attention/activity inference (examples):
  - Face present / not present
  - Eye gaze direction proxy
  - Head pose and prolonged absence
- Generate distraction signal if user is repeatedly absent or disengaged.

### 6.3 Desktop Activity Monitoring
- Detect active window title/process name.
- Track website domains from browser tabs (via extension or local integration).
- Compare against allowlist/blocklist and current goal context.
- Trigger distraction signal if user spends threshold time off-task.

### 6.4 Phone Activity Monitoring
- Collect phone-level activity through an opt-in companion app.
- Signals (minimum):
  - Foreground app category
  - Screen unlock frequency
  - Session interruptions
- Send events securely to local backend.

### 6.5 Wi-Fi / Network Context Monitoring (Optional)
- Observe high-level network signals from user-owned devices under same Wi-Fi.
- Prefer metadata-based indicators (DNS categories, domain classes, usage bursts).
- Do not rely on decrypting encrypted traffic.
- Require explicit consent and local network ownership.

### 6.6 Warning and Intervention
- Warning channels:
  - Desktop notification
  - Sound alert
  - Full-screen focus prompt (optional)
  - Mobile push notification (optional)
- Warning should include reason and action:
  - "Off-topic tab open for 3+ minutes"
  - "Return to goal" / "Take a short break"

### 6.7 Session Analytics
- Focus score over time
- Count and type of distraction events
- Recovery time (time to return to focus)
- Daily/weekly trend summaries

## 7. Non-Functional Requirements
- Latency: warning decision in <= 5 seconds for local signals.
- Reliability: system should continue monitoring if one sensor fails.
- Privacy: default local processing for sensitive media (webcam frames).
- Security: encrypted communication between devices.
- Extensibility: pluggable signal sources and scoring models.

## 8. High-Level Architecture
### 8.1 Components
1. Desktop Agent
- Captures webcam frames
- Monitors active apps/windows and browsing context
- Runs local distraction inference
- Displays warnings

2. Mobile Companion App
- Collects allowed phone activity signals
- Sends events to backend/desktop agent

3. Local Orchestrator API
- Receives events from desktop/mobile/network collectors
- Normalizes event format
- Runs scoring policy engine

4. Policy & Scoring Engine
- Computes distraction risk from multiple signals
- Supports threshold rules and ML-based ranking

5. Notification Service
- Delivers warnings to desktop and phone

6. Data Store
- Session metadata
- Event timeline
- Aggregated analytics

7. Dashboard UI
- Goal setup
- Live session state
- Post-session analytics

### 8.2 Suggested Tech Stack
- Desktop app: Electron + TypeScript
- Desktop monitoring service: Node.js + native OS hooks
- Webcam inference: Python microservice (OpenCV + MediaPipe)
- Backend API: FastAPI or NestJS
- Mobile app: Flutter or React Native
- Data store: PostgreSQL (session/events) + Redis (real-time state)
- Messaging: WebSocket for live updates
- Deployment (later): Docker Compose/Kubernetes

## 9. Data Model (Draft)
### 9.1 Core Entities
- User
- FocusGoal
- Session
- SignalEvent
- DistractionEvent
- WarningEvent

### 9.2 Example Event Schema
```json
{
  "event_id": "uuid",
  "session_id": "uuid",
  "source": "desktop|webcam|mobile|network",
  "event_type": "active_window|domain_visit|face_absent|phone_unlock",
  "value": "string_or_number",
  "confidence": 0.0,
  "timestamp": "2026-04-07T12:34:56Z"
}
```

## 10. Distraction Detection Strategy
### 10.1 Rule-Based MVP
Weighted score per short window (for example, 30 seconds):
- Off-topic domain active > threshold: +40
- Entertainment app in foreground: +30
- Face absent from webcam: +20
- Frequent phone unlocks during session: +15

If score >= warning threshold (for example, 50), issue warning.

### 10.2 Phase 2 ML Enhancement
- Train personalized models from labeled sessions.
- Features:
  - Window/app dwell times
  - Tab switching frequency
  - Webcam engagement metrics
  - Phone interruption patterns
- Output probability of distraction in near real time.

## 11. Privacy, Consent, and Compliance
This project processes sensitive behavioral data. Requirements:
- Explicit opt-in for each data source (webcam, desktop, phone, network).
- Clear consent UI with per-source toggles.
- Data minimization: collect only required signals.
- Prefer local processing for webcam.
- Encryption in transit (TLS) and at rest.
- Data retention controls and delete-my-data flow.
- Compliance check for jurisdiction laws (GDPR/CCPA or local equivalents).

## 12. Security Design
- Device-to-device authentication using short-lived tokens.
- Signed API requests between mobile and orchestrator.
- Local network trust is not enough; always authenticate.
- Role-based access if multi-user support is added.
- Audit logs for settings changes and data access.

## 13. Risks and Mitigations
- False positives annoy users.
  - Mitigation: cooldown windows, explainable warnings, adjustable sensitivity.
- Webcam model bias/inaccuracy.
  - Mitigation: combine with desktop/mobile signals.
- Phone and Wi-Fi monitoring complexity.
  - Mitigation: start with companion app + desktop first.
- Privacy concerns.
  - Mitigation: transparent consent, local-first architecture.

## 14. MVP Scope (Recommended)
### In Scope
- Goal/session setup
- Desktop app/window/domain monitoring
- Webcam face presence signal
- Rule-based distraction scoring
- Desktop warning popups
- Session summary dashboard

### Out of Scope for MVP
- Full home network device monitoring
- Advanced ML personalization
- Multi-user and cloud sync

## 15. Implementation Roadmap
### Phase 1 (2-4 weeks)
- Project scaffold (desktop agent + backend + dashboard)
- Session lifecycle API
- Desktop activity collector

### Phase 2 (2-4 weeks)
- Webcam capture and basic engagement signals
- Rule engine + warning service
- Local storage and session analytics

### Phase 3 (3-6 weeks)
- Mobile companion app (basic activity signals)
- Real-time synchronization and push notifications
- Tuning and user testing

### Phase 4 (later)
- Optional network signal module
- ML-based personalization
- Cloud backup and cross-device continuity

## 16. Acceptance Criteria (MVP)
- User can start a focus session with a goal and duration.
- System detects at least 3 distraction types (off-topic domain, entertainment app, user absence).
- Warning is shown within 5 seconds of threshold breach.
- Session report shows timeline and focus score.
- User can disable any sensor and continue with remaining sensors.

## 17. Open Questions
- Which platforms are priority (Windows/macOS/Linux)?
- How strict should warning behavior be (soft nudges vs hard blocks)?
- Should YouTube be partially allowed for educational content?
- What level of phone data is acceptable to users?
- Is cloud sync required in the first public release?

## 18. Suggested Next Steps
1. Confirm MVP boundaries from Section 14.
2. Decide initial platform target (for fastest delivery).
3. Finalize event schema and scoring thresholds.
4. Build desktop-first prototype with local-only processing.
