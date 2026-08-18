ERIS — Product Specification

AI-powered multilingual emergency response assistant integrating voice, maps, and hospitals.

1. Overview

ERIS is a voice-first multilingual emergency assistant designed to help people quickly identify appropriate nearby emergency facilities during stressful situations.

A user can describe an emergency naturally in a supported language. ERIS uses AI to understand the emergency, obtains the user's location, finds nearby emergency-capable hospitals, ranks the available options, and provides map-based navigation.

Core principle

AI understands the emergency. Deterministic application logic finds and ranks hospitals. Maps provide navigation.

ERIS is an emergency assistance and navigation tool, not a medical diagnosis system.

2. Problem

During an emergency, people may:

be too stressed to type
have difficulty communicating
speak a regional language
not know their exact location
not know which nearby hospital is appropriate
waste time searching through maps or hospital listings

Existing tools solve individual parts of the problem, but the user often has to combine multiple services themselves.

ERIS combines the critical steps into one workflow:

Voice
  ↓
Emergency Understanding
  ↓
Location
  ↓
Hospital Discovery
  ↓
Hospital Ranking
  ↓
Map
  ↓
Navigation
3. Target Users
Primary

People experiencing an emergency.

Secondary

Bystanders helping someone during an emergency.

Example scenarios
Accident / trauma
Severe bleeding
Breathing difficulty
Chest pain
Unconsciousness
Stroke-like symptoms
Other urgent situations
4. Product Goals

The MVP should:

Allow users to describe an emergency using voice.
Support multiple languages.
Understand and categorize the emergency.
Detect the user's location.
Find nearby hospitals/emergency facilities.
Rank suitable facilities.
Display the recommendation on a map.
Provide navigation to the selected facility.
Work reliably within a 24-hour hackathon environment.
5. Non-Goals

The MVP will not attempt to build:

Medical diagnosis
Patient medical records
User accounts
Hospital management systems
Ambulance fleet tracking
Real-time hospital bed management
Hospital booking
Wearable integration
Custom-trained medical AI
Native Android/iOS applications
Complex analytics
Payment systems
6. Core User Journey
Step 1 — Speak

The user taps Speak Emergency and describes the situation naturally.

Example:

"Mere bhai ka accident ho gaya hai aur bahut khoon nikal raha hai."

Step 2 — Understand

ERIS:

converts speech to text
detects/uses the selected language
identifies the emergency category
estimates urgency
generates a short response

Example:

{
  "language": "hi",
  "emergency_type": "severe_bleeding",
  "urgency": "high",
  "requires_emergency_facility": true
}
Step 3 — Find

ERIS obtains the user's location and searches for nearby hospitals.

Results are ranked using available verified information such as:

emergency capability
distance
estimated travel time
facility information
Step 4 — Navigate

ERIS displays the recommended facility on a map and allows the user to launch navigation.

7. MVP Features
P0 — Required
Voice input
Record emergency description.
Convert speech to text.
Display transcription.
Multilingual support
Support at least two languages for the demo.
Allow language selection where necessary.
Respond in the user's language.
Emergency understanding

AI extracts:

language
emergency type
urgency
whether emergency care is required
Location
Request browser/device location.
Obtain latitude and longitude.
Handle location permission failure.
Hospital search
Search nearby hospitals/emergency facilities.
Retrieve relevant facility information.
Hospital ranking

Rank results based on deterministic application logic.

Map

Display:

user location
recommended hospital
alternative hospitals
route where supported
Navigation

Provide a Navigate action that opens the selected map/navigation service.

Safety messaging

ERIS must not present a definitive medical diagnosis.

8. P1 Features

Implement only after the P0 flow works reliably.

Text-to-speech
Emergency instruction cards
One-tap emergency calling
Share location
Hospital phone number
Hospital detail view
More language options
9. P2 Features

Do not implement unless the complete MVP is finished.

Authentication
Patient accounts
Medical history
Medical records
Hospital dashboard
Hospital staff dashboard
Ambulance tracking
Real-time bed availability
Wearable integration
Native mobile applications
Hospital booking
Advanced analytics
Custom AI model
10. Application Pages
10.1 Emergency Home
Purpose

Provide an immediate, simple entry point into the emergency workflow.

UI
ERIS logo
Emergency status
Large Speak Emergency button
Language selector
Location status
Short safety disclaimer
Actions
Start emergency recording
Select language
Grant location permission
10.2 Emergency Understanding
Purpose

Show the user that ERIS understood their emergency.

UI
Recording indicator
Transcription
Detected/selected language
Emergency category
Urgency
Short response
Continue button
Actions
Start/stop recording
Retry
Continue to hospital search
10.3 Hospital Results
Purpose

Show nearby emergency facilities and identify the recommended option.

UI
Interactive map
User location
Hospital markers
Recommended hospital card
Alternative hospitals
Distance
Estimated travel time
Emergency capability
Navigate button
Actions
Select hospital
View details
Start navigation
10.4 Hospital Detail
Purpose

Provide final information before navigation.

UI
Hospital name
Address
Distance
Estimated travel time
Emergency capability
Map/route
Navigate button
Phone button if verified
11. System Architecture
                         USER
                           |
                           v
                  +----------------+
                  | React Frontend |
                  | Vite + TS      |
                  +-------+--------+
                          |
                         HTTPS
                          |
                          v
                 +-------------------+
                 | Node.js Backend   |
                 | Express           |
                 +---------+---------+
                           |
              +------------+------------+
              |                         |
              v                         v
       +-------------+          +---------------+
       | AI Provider |          | Maps/Places   |
       | Speech + LLM|          | + Routing     |
       +-------------+          +---------------+
              |                         |
              +------------+------------+
                           |
                           v
                    +-------------+
                    | Supabase    |
                    | PostgreSQL  |
                    +-------------+
12. Technology Stack
Component	Technology
Frontend	React + Vite + TypeScript
Styling	Tailwind CSS
UI components	shadcn/ui
Backend	Node.js + Express + TypeScript
Database	Supabase PostgreSQL
AI	One approved LLM provider
Speech-to-text	AI/speech API
Maps	Google Maps Platform or Mapbox
Authentication	None
File storage	None for MVP
Frontend hosting	Vercel
Backend hosting	Render
Source control	GitHub

The final AI and maps providers should depend on hackathon rules, available credits, and API access.

13. Backend Responsibilities

The backend acts as the main orchestration layer.

Required capabilities
Receive emergency audio.
Send audio to speech-to-text.
Send transcription to the LLM.
Validate structured AI output.
Search nearby hospitals.
Retrieve routing information.
Rank hospitals.
Return a normalized response to the frontend.
Store optional emergency session information.
Handle API failures safely.
14. API Endpoints

The MVP should expose as few endpoints as possible.

GET /api/health

Health check.

Response
{
  "status": "ok"
}
POST /api/emergency/analyze

Main ERIS endpoint.

Request

multipart/form-data

audio
language
latitude
longitude
Processing
Audio
  ↓
Speech-to-text
  ↓
Emergency classification
  ↓
Hospital search
  ↓
Routing
  ↓
Hospital ranking
Response
{
  "sessionId": "abc123",
  "transcription": "Mere bhai ka accident ho gaya hai...",
  "emergency": {
    "type": "severe_bleeding",
    "urgency": "high"
  },
  "message": "Yeh serious emergency ho sakti hai. Turant medical help lein.",
  "recommendedHospital": {
    "id": "hospital_123",
    "name": "Example Hospital",
    "latitude": 21.15,
    "longitude": 79.09,
    "distanceKm": 2.3,
    "durationMinutes": 6,
    "emergencyCapable": true
  },
  "alternatives": []
}

For the 24-hour MVP, this single business endpoint is preferred over creating many separate API endpoints.

15. AI Architecture

AI is used only where natural-language understanding is required.

Speech-to-text
User voice
    ↓
Speech API
    ↓
Transcription
Emergency classification
Transcription
    ↓
LLM
    ↓
Structured JSON

Example:

{
  "language": "hi",
  "emergency_type": "severe_bleeding",
  "urgency": "high",
  "requires_emergency_facility": true
}
Response generation

The LLM can generate a short response in the user's language.

AI must not
diagnose the patient
invent hospital information
claim real-time bed availability without a verified source
directly select a hospital
control navigation

Hospital selection remains deterministic application logic.

16. Hospital Ranking

Hospital ranking should be performed by normal backend code.

Conceptually:

Hospital suitability
        +
Emergency capability
        +
Distance
        +
Travel time
        ↓
Final ranking

The exact weighting can be adjusted based on available API data.

The LLM should not determine the final ranking.

17. Database

The database is intentionally minimal.

emergency_sessions
Field	Type
id	UUID
created_at	timestamp
language	string
transcription	text
emergency_type	string
urgency	string
latitude	decimal
longitude	decimal
selected_hospital_id	string

No unnecessary personal information should be stored.

hospitals

This table is optional.

Use it only if the hackathon provides a hospital dataset or if hospital information needs to be cached.

Field	Type
id	string
name	string
address	string
latitude	decimal
longitude	decimal
phone	string
emergency_capability	boolean
specialties	JSON
source	string
last_verified_at	timestamp

If the maps/places provider already supplies the required information, this table can be omitted.

18. Authentication

Authentication is not required for the MVP.

Users can access ERIS anonymously.

This reduces:

development time
UI complexity
database complexity
security requirements
19. File Storage

Permanent audio storage is not required.

Preferred flow:

Browser
   ↓
Audio
   ↓
Backend
   ↓
Speech-to-text
   ↓
Discard audio

No patient recordings need to be stored.

20. External Services
AI

Use one primary provider for:

speech-to-text
emergency classification
multilingual understanding
response generation
Maps

Use one provider for:

nearby hospitals
maps
routing
distance/travel time

Possible providers:

Google Maps Platform
Mapbox
approved OpenStreetMap-based services
Hospital data

Preferred source order:

Organizer-provided verified dataset/API
Official/approved hospital data
Maps/places provider
Other permitted public data
21. Error Handling

ERIS must handle common failures gracefully.

Speech failure

"We couldn't understand the audio. Please try again."

AI failure

"We couldn't understand the emergency. Please try again."

Location failure

"Location access is unavailable."

Hospital API failure

"We're unable to retrieve nearby facilities right now."

No suitable hospital

"No suitable nearby emergency facility was found."

The application should never silently fail or display fabricated information.

22. Demo Scenario
Scenario

A person witnesses a serious accident.

They open ERIS and say in Hindi:

"Mere bhai ka accident ho gaya hai aur bahut khoon nikal raha hai."

ERIS:

VOICE
  ↓
Speech-to-text
  ↓
Hindi understanding
  ↓
Severe bleeding detected
  ↓
High urgency
  ↓
Location detected
  ↓
Nearby hospitals searched
  ↓
Hospitals ranked
  ↓
Recommended hospital displayed
  ↓
Route displayed
  ↓
Navigate

The entire demonstration should take approximately 60–90 seconds, leaving the remainder of a 2-minute pitch for explanation.

23. Success Criteria

The MVP is complete when the following works end-to-end.

Voice
 User can record voice.
 Speech is converted to text.
 At least two demo languages work reliably.
AI
 Emergency category is correctly extracted.
 Urgency is correctly classified for prepared demo scenarios.
 Structured output is validated.
 AI does not present a definitive diagnosis.
Location
 User location can be obtained.
 Location failure is handled.
Hospitals
 Nearby hospitals can be retrieved.
 Hospital information comes from an actual data source.
 At least three usable options can normally be displayed.
Ranking
 One hospital is clearly recommended.
 Ranking uses deterministic application logic.
 Distance/travel time is displayed where available.
Map
 User location is displayed.
 Hospital location is displayed.
 Route can be displayed or launched.
Navigation
 Navigate button successfully opens the selected navigation service.
UX
 Core flow can be completed without presenter intervention.
 Loading states are clear.
 Errors are handled.
 Emergency action is immediately visible.
Reliability
 A fresh browser session can complete the prepared demo scenario.
 Backend/API failures do not crash the frontend.
 No fake medical or hospital information is presented.
24. 24-Hour Development Priority
P0 CORE FLOW
│
├── Emergency UI
├── Voice recording
├── Speech-to-text
├── AI emergency classification
├── Location
├── Hospital search
├── Ranking
├── Map
└── Navigation
        │
        ▼
   TEST END-TO-END
        │
        ▼
   FIX RELIABILITY
        │
        ▼
   POLISH UI
        │
        ▼
   P1 FEATURES

Rule: Do not start P1 features until the complete P0 flow works.

25. Product Principle

ERIS does not try to replace doctors, ambulances, or emergency services.

Its purpose is to reduce the friction between:

"Something is wrong."

and

"I know where to go for emergency care."

ERIS

Speak. Understand. Find. Navigate.
