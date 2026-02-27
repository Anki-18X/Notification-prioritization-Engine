🔔 Intelligent Notification Decision Engine

Cyepro Solutions – Assignment Submission
Drive Code: OC.36641.2026.57933
Built using Python + Flask
AI Assistance: Claude (Anthropic)

1️⃣ Overview

Modern applications generate excessive notifications — promotional pushes, reminders, alerts, system updates, and more. Without control, users experience:

Notification spam

Repeated duplicate messages

Alerts delivered at inconvenient times

Important notifications getting buried

This project implements a Notification Decision Engine that determines, for every incoming event:

Should this notification be delivered Now, deferred for Later, or suppressed Forever?

The system ensures that:

🚨 Critical alerts always reach the user

📢 Promotions and low-value content are controlled

😴 User fatigue is actively managed

2️⃣ High-Level Architecture
Request Flow
Upstream Service
(chat, marketing, monitoring, etc.)
        │
        ▼
POST /api/v1/notify/classify
        │
        ▼
Notification Decision Engine
        │
        ▼
Decision: now | later | never
Internal Components
Module	Responsibility
API Layer	Exposes REST endpoints
Decision Engine	Executes ordered validation checks
Rule Manager	Handles configurable suppression rules
Deduplication Store	Prevents repeated messages
User History Tracker	Tracks fatigue limits
Audit Logger	Logs every decision with reasoning

The engine is stateless and can scale horizontally when backed by Redis.

3️⃣ Decision Strategy

Each notification goes through a strict ordered pipeline.

The first rule that fails determines the outcome.

🧠 Decision Flow
Step 1 — Expiry Validation

If the notification has already expired →
➡ Decision: never

Step 2 — Duplicate Detection

Two detection mechanisms are applied:

Exact match via dedupe_key

Near-duplicate match via MD5 hash of content

If detected →
➡ Decision: never

Step 3 — Event Cooldown

Certain event types require spacing between messages.

Example:

Promotion → 1 hour gap

Reminder → 30 minutes

Alert → No cooldown

If cooldown active →
➡ Decision: later

Step 4 — Fatigue Limits

User-based suppression rules:

Max notifications per hour

Max notifications per day

If threshold exceeded:

Low priority → never

Medium priority → later

Step 5 — Quiet Hours

Configured window:
22:00 – 08:00 UTC

If notification is non-urgent during this period →
➡ Deferred to next valid time (8 AM)

⚡ Time-Sensitive Override

If:

event_type = alert OR system_event
OR
priority_hint = critical / urgent

The notification bypasses:

Cooldown

Fatigue limits

Quiet hours

And is immediately sent.

4️⃣ Input & Output Schemas
🔹 NotificationEvent (Input)
{
  "user_id": "u123",
  "event_type": "promotion",
  "message": "Flat 20% discount today!",
  "priority_hint": "low",
  "channel": "push",
  "timestamp": "2026-02-27T10:00:00Z",
  "expires_at": "2026-02-27T23:59:00Z"
}
🔹 DecisionResponse (Output)
{
  "notification_id": "uuid",
  "decision": "later",
  "reason": "Cooldown active for 'promotion'",
  "defer_seconds": 3600,
  "processed_at": "2026-02-27T10:00:01Z"
}
5️⃣ Rule Configuration (Dynamic)

Rules can be updated via API without restarting the server.

Example configuration:

{
  "quiet_hours": { "start": 22, "end": 8 },
  "max_per_hour": 10,
  "max_per_day": 30,
  "cooldown_seconds": {
    "promotion": 3600,
    "reminder": 1800,
    "update": 600,
    "alert": 0,
    "message": 0,
    "system_event": 0
  }
}

Rules are designed to be externally configurable in production.

6️⃣ REST API Endpoints
1️⃣ Classify Single Notification
POST /api/v1/notify/classify

Returns decision for one event.

2️⃣ Batch Classification
POST /api/v1/notify/batch

Supports up to 100 notifications per request.

Returns summary counts:

now

later

never

3️⃣ Audit Logs
GET /api/v1/audit/logs

Supports filters:

user_id

decision

limit

4️⃣ Rules Management
GET /api/v1/rules
PUT /api/v1/rules

Allows runtime rule updates.

5️⃣ User Notification History
GET /api/v1/users/{user_id}/history

Returns:

Recent notifications

Remaining hourly quota

Remaining daily quota

7️⃣ Duplicate Prevention Model

Two-layer defense:

Method	Purpose	Duration
dedupe_key match	Prevent exact retry storms	1 hour
Content hash match	Catch near-identical messages	5 minutes

This prevents:

Retry floods

Multiple services sending similar content

Accidental repeated pushes

8️⃣ Failure Handling Philosophy

The engine follows a fail-safe strategy.

If classification fails:

if critical_event:
    decision = "now"
else:
    decision = "later"
Behavior Summary
Scenario	Outcome
Engine crash	Safe fallback applied
Redis unavailable	In-memory fallback
Invalid payload	400 error
Unknown event_type	Treated as low priority

Principle:

Never silently drop critical alerts.

9️⃣ Metrics & Observability Plan

To support production readiness, the following metrics are proposed:

Decision distribution (now/later/never)

p99 classification latency

Fallback invocation rate

Duplicate suppression rate

User fatigue trends

Quiet hours deferrals

Alert Conditions

High fallback rate → Engine degradation alert

High latency → Scale horizontally

Excessive suppression rate → Investigate upstream services

🔟 Running the Application
Install Dependencies
pip install flask
Start Server
python app.py

Runs at:

http://localhost:5000
Run Demo Script
python demo.py

Simulates multiple decision scenarios.

Sample cURL
curl -X POST http://localhost:5000/api/v1/notify/classify \
  -H "Content-Type: application/json" \
  -d '{
    "user_id": "u001",
    "event_type": "alert",
    "message": "Database connection failed",
    "priority_hint": "critical",
    "channel": "sms"
  }'
🏭 Production Evolution Plan
Current Implementation	Production Upgrade
In-memory storage	Redis with TTL
Single instance	Horizontal scaling
No authentication	API key / JWT
Sync processing	Async queue for deferred jobs
Local rules	Centralized config service
🤖 AI Usage Disclosure

AI assistance (Claude – Anthropic) was used for:

Structuring initial Flask boilerplate

Drafting documentation outline

All core business logic including:

Decision order

Override behavior

Hashing strategy

Fatigue thresholds

were manually designed and implemented.

📂 Project Structure
notification-engine/
│
├── app.py      # Core API + decision logic
├── demo.py     # Test scenarios
└── README.md   # Documentation
