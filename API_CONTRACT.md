ERIS API Definition

Version: 1.0.0
Base URL: /api
Content Type: application/json unless otherwise specified.

ERIS exposes a minimal REST API for processing emergency voice requests and checking backend availability.

1. POST /api/emergency/analyze

Analyzes a user's emergency description, identifies the emergency category, obtains nearby emergency facilities, ranks them, and returns the recommended facility.

This is the primary ERIS API endpoint.

Request

Content-Type: multipart/form-data

Field	Type	Required	Description
audio	File	Yes	User's recorded emergency speech
language	String	No	User-selected language code, e.g. en, hi, mr
latitude	Number	Yes	User's current latitude
longitude	Number	Yes	User's current longitude
Example
POST /api/emergency/analyze
Content-Type: multipart/form-data

audio: emergency.webm
language: hi
latitude: 21.1458
longitude: 79.0882
Processing flow
Audio
  ↓
Speech-to-Text
  ↓
Emergency Classification
  ↓
Location + Emergency Type
  ↓
Nearby Hospital Search
  ↓
Hospital Ranking
  ↓
Route Calculation
  ↓
Response
Success Response

HTTP 200

{
  "success": true,
  "sessionId": "sess_01HXYZ123",
  "transcription": "Mere bhai ka accident ho gaya hai aur bahut khoon nikal raha hai.",
  "language": "hi",
  "emergency": {
    "type": "severe_bleeding",
    "urgency": "high",
    "requiresEmergencyFacility": true
  },
  "message": "Yeh serious emergency ho sakti hai. Kripya turant medical help lein.",
  "location": {
    "latitude": 21.1458,
    "longitude": 79.0882
  },
  "recommendedHospital": {
    "id": "hospital_123",
    "name": "Example Hospital",
    "address": "Example Road, Nagpur",
    "latitude": 21.1520,
    "longitude": 79.0900,
    "distanceKm": 2.3,
    "durationMinutes": 6,
    "emergencyCapable": true,
    "phone": "+91XXXXXXXXXX"
  },
  "alternatives": [
    {
      "id": "hospital_456",
      "name": "Example Hospital 2",
      "address": "Example Area, Nagpur",
      "latitude": 21.1600,
      "longitude": 79.1000,
      "distanceKm": 3.1,
      "durationMinutes": 9,
      "emergencyCapable": true
    }
  ]
}
Response fields
success
boolean

Indicates whether the request was successfully processed.

sessionId
string

Unique identifier for the emergency session.

transcription
string

Speech-to-text result.

language
string

Detected or requested language.

emergency.type

Possible MVP values:

severe_bleeding
breathing_difficulty
chest_pain
trauma
unconsciousness
stroke_like_symptoms
other
emergency.urgency

Possible values:

high
medium
unknown

The MVP should avoid presenting this as a medical diagnosis.

recommendedHospital

The highest-ranked nearby facility based on available verified information, distance, and travel time.

Error Responses
400 — Invalid request
{
  "success": false,
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Audio and location are required."
  }
}
413 — Audio too large
{
  "success": false,
  "error": {
    "code": "AUDIO_TOO_LARGE",
    "message": "Audio file exceeds the maximum allowed size."
  }
}
422 — Unable to understand speech
{
  "success": false,
  "error": {
    "code": "SPEECH_NOT_UNDERSTOOD",
    "message": "We could not understand the audio. Please try again."
  }
}
429 — Rate limit
{
  "success": false,
  "error": {
    "code": "RATE_LIMITED",
    "message": "Too many requests. Please try again shortly."
  }
}
502 — External service failure
{
  "success": false,
  "error": {
    "code": "EXTERNAL_SERVICE_ERROR",
    "message": "Emergency services are temporarily unavailable. Please try again."
  }
}
503 — No suitable facility found
{
  "success": false,
  "error": {
    "code": "NO_FACILITY_FOUND",
    "message": "No suitable nearby emergency facility was found."
  }
}
2. GET /api/health

Checks whether the ERIS backend is running.

This endpoint is used for:

deployment verification
debugging
monitoring
hackathon demo preparation
Request
GET /api/health
Success Response

HTTP 200

{
  "status": "ok",
  "service": "eris-api",
  "version": "1.0.0"
}
Navigation

Navigation does not require an ERIS backend endpoint.

The frontend can generate/open a navigation link using the selected hospital's coordinates.

Conceptually:

User Location
     +
Hospital Coordinates
     ↓
Maps Navigation

The frontend should not send the user through the ERIS backend just to open navigation.

API Architecture
Frontend
   │
   ├── POST /api/emergency/analyze
   │
   └── GET /api/health
             │
             ▼
       ERIS Backend
             │
       ┌─────┼─────────┐
       ▼     ▼         ▼
     Speech  AI      Maps/Places
       │     │         │
       └─────┼─────────┘
             ▼
       Ranking Logic
             │
             ▼
        API Response
             │
             ▼
         Frontend
Backend Internal Services

The external API surface remains small, but internally the backend should be separated into simple services.

src/
├── routes/
│   ├── emergency.routes.ts
│   └── health.routes.ts
│
├── controllers/
│   └── emergency.controller.ts
│
├── services/
│   ├── speech.service.ts
│   ├── ai.service.ts
│   ├── hospital.service.ts
│   ├── routing.service.ts
│   └── ranking.service.ts
│
├── schemas/
│   └── emergency.schema.ts
│
├── utils/
│   └── errors.ts
│
└── server.ts
Internal Emergency Processing Contract

The AI service should return a strict structured object to the backend.

{
  "language": "hi",
  "emergencyType": "severe_bleeding",
  "urgency": "high",
  "requiresEmergencyFacility": true
}

The backend validates this object before continuing.

The LLM should never directly return a hospital recommendation.

Hospital Ranking Contract

The ranking service receives:

{
  "emergencyType": "severe_bleeding",
  "userLocation": {
    "latitude": 21.1458,
    "longitude": 79.0882
  },
  "hospitals": []
}

It returns:

{
  "recommendedHospitalId": "hospital_123",
  "rankedHospitals": []
}

Ranking should use deterministic application logic based on available data.

Example priority:

1. Verified emergency capability
2. Suitability for emergency category, if verified
3. Travel time
4. Distance
5. Data completeness
Security
API keys

Never expose external API keys to the frontend.

Frontend
   ↓
ERIS Backend
   ↓
OPENAI_API_KEY
MAPS_API_KEY

Keys should exist only as backend environment variables.

Environment Variables
PORT=3000

OPENAI_API_KEY=

MAPS_API_KEY=

SUPABASE_URL=
SUPABASE_SERVICE_KEY=

ALLOWED_ORIGIN=

If Supabase is not required for the MVP, remove the Supabase variables entirely.

MVP API Scope

ERIS intentionally exposes only:

Method	Endpoint	Required
POST	/api/emergency/analyze	Yes
GET	/api/health	Yes

Do not add separate endpoints for:

authentication
users
hospitals
patients
medical records
analytics
ambulance tracking
hospital administration

unless the hackathon requirements specifically require them.

Important Safety Boundary

ERIS is an emergency assistance and navigation system.

It does not provide a medical diagnosis.

The AI is responsible for:

Natural language
      ↓
Understanding
      ↓
Structured emergency category

The application is responsible for:

Location
   ↓
Hospital search
   ↓
Hospital ranking
   ↓
Routing

The system must never claim:

real-time bed availability unless verified by an API
specialist availability unless verified
that a hospital is definitely able to treat a condition unless the data source supports that claim
that the AI has diagnosed the user

When a situation appears urgent, ERIS should encourage the user to seek professional emergency medical assistance immediately.

API Design Principle

The ERIS API intentionally follows one core principle:

One frontend request should accomplish the complete emergency workflow.

Instead of:

POST /speech
POST /classify
GET /hospitals
GET /route
POST /rank

the MVP uses:

POST /api/emergency/analyze

This minimizes frontend complexity, reduces network failure points, and makes the 24-hour hackathon implementation significantly easier to debug.
