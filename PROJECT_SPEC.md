ERIS — Emergency Response Intelligence System
PRODUCT

Name: ERIS

One-line description:
A multilingual, voice-first emergency assistant that understands an emergency, detects the user's location, finds nearby suitable emergency hospitals, and guides the user to the best available option.

Target user:
A person experiencing an emergency or a bystander helping someone during an emergency, especially when they are stressed, unable to type, unfamiliar with the area, or more comfortable speaking a regional language.

MVP positioning:
ERIS is an emergency assistance and navigation tool, not a medical diagnosis system. AI interprets the user's description; verified application logic and external services handle location, hospital discovery, ranking, and navigation.

CORE USER JOURNEY
Step 1: Speak

The user taps "Speak Emergency" and describes the situation naturally in a supported language.

Example:

"Mere bhai ka accident ho gaya hai aur bahut khoon nikal raha hai."

Step 2: Understand

ERIS converts speech to text and uses AI to extract:

language
emergency category
urgency
relevant symptoms/context

ERIS then gives a short confirmation in the user's language.

Step 3: Find

ERIS obtains the user's location and searches for nearby emergency-capable hospitals.

The system ranks a small number of options using:

suitability for the emergency
distance
estimated travel time
available verified hospital information
Step 4: Guide

ERIS displays the recommended hospital on a map with:

hospital name
distance
estimated travel time
relevant emergency capability
navigation button

The user can open navigation and proceed to the facility.

FEATURES
P0 — Absolutely required for the demo
1. Emergency voice input
Large emergency voice button
Record user speech
Convert speech to text
Display transcription
2. Multilingual interaction
Support at least 2 languages for the demo
Detect or allow selection of language
Respond in the user's language

The exact languages should be finalized based on API quality and the hackathon audience.

3. Emergency intent extraction

AI extracts a structured emergency category from the user's description.

Example:

{
  "emergency_type": "severe_bleeding",
  "urgency": "high"
}
4. Location detection
Request browser/device location permission
Obtain latitude/longitude
Show detected location
Provide a manual fallback if location permission fails
5. Nearby hospital search

Find nearby hospitals/emergency facilities using an external map/places service or organizer-provided hospital dataset.

6. Hospital ranking

Return a small set of relevant options, preferably 3.

Ranking should consider:

emergency suitability
distance
travel time
available verified facility information
7. Map visualization

Show:

user's location
recommended hospital
nearby alternatives
route or route-launch option
8. Recommended hospital

Clearly identify one recommended option.

Example:

Recommended Emergency Facility

with a short explanation such as:

"Nearby facility listed with emergency services."

Do not claim real-time bed availability or specialist availability unless the data source actually provides it.

9. Navigation

A prominent:

Navigate to Hospital

button that opens the supported navigation service.

10. Emergency-safe messaging

ERIS should communicate that serious situations require immediate professional/emergency medical help.

It should not diagnose the user.

P1 — Useful if time permits
1. Text-to-speech

Read ERIS's important response aloud.

2. Emergency instruction card

A short, predefined instruction appropriate to the detected emergency category.

Keep this conservative and medically reviewed.

3. One-tap emergency call

Only if permitted by the hackathon and technically reliable in the demo environment.

The emergency number must be determined from the deployment region rather than hardcoded blindly.

4. Share location

Generate a shareable location/action for a trusted person.

5. Language selector

Allow the user to explicitly choose a supported language instead of relying entirely on automatic detection.

6. Hospital details

Show verified information such as:

address
phone number
emergency service indicator
distance
estimated travel time
P2 — Do not build unless everything else is finished
User accounts
Authentication
Patient profiles
Medical history
Medical records
Hospital administration dashboard
Hospital staff dashboard
Ambulance fleet tracking
Real-time ambulance dispatch
Real-time hospital bed management
Wearable integration
IoT sensors
Custom-trained medical AI model
Native Android/iOS applications
Full hospital booking system
Complex analytics dashboard
Payment system
Multi-hospital messaging
Continuous background location tracking
PAGES
PAGE 1 — Emergency Home
Purpose

Immediately communicate what ERIS does and get the user into the emergency flow with minimal interaction.

UI elements
ERIS logo/name
Emergency status/instruction
Large Speak Emergency button
Language indicator/selector
"Use my location" status
Optional small safety disclaimer
User actions
Tap Speak Emergency
Select language if necessary
Grant location permission
Data required
Supported languages
Location permission state
Basic application configuration
API calls required

Potentially none initially.

Location can be obtained directly from the browser/device.

PAGE 2 — Voice / Emergency Understanding
Purpose

Capture the emergency description and show that ERIS understood it.

UI elements
Recording animation
Microphone state
Transcribed speech
Detected language
Emergency category
Urgency indicator
Short AI response
Find Emergency Hospital button
User actions
Start recording
Stop recording
Review transcription
Retry recording
Continue
Data required
audio
transcription
language
emergency_type
urgency
API calls required
Speech-to-text API
AI structured extraction API
PAGE 3 — Hospital Results / Emergency Map
Purpose

Show the best nearby emergency facilities and make the recommended choice obvious.

UI elements
Map
User location marker
Recommended hospital marker
Alternative hospital markers
Recommended hospital card
Hospital name
Distance
Estimated travel time
Emergency capability indicator
Navigate button
Optional alternative hospitals
User actions
Select recommended hospital
Select alternative hospital
View hospital details
Start navigation
Data required
user_latitude
user_longitude
emergency_type
hospital_name
hospital_location
hospital_capabilities
distance
travel_time
navigation_destination
API calls required
Places/hospital search API
Routing/directions API
Optional geocoding API
PAGE 4 — Hospital Detail / Navigation
Purpose

Give the user enough information to confidently proceed to the selected facility.

UI elements
Hospital name
Address
Distance
Estimated travel time
Emergency capability
Map/route
Navigate Now button
Optional phone button if verified
User actions
Start navigation
Call hospital if available
Return to hospital list
Data required

Selected hospital record and route information.

API calls required
Routing API if route wasn't already obtained
External navigation service/deep link
BACKEND

The backend should remain deliberately small.

1. Emergency request processing

Accept:

user transcription
language
approximate/location coordinates
emergency context

Return structured emergency information.

2. AI orchestration

Send the user's text to the AI service and request structured output.

Example:

{
  "language": "Hindi",
  "emergency_type": "severe_bleeding",
  "urgency": "high",
  "requires_emergency_facility": true
}

The backend validates the returned structure before using it.

3. Hospital search

Accept:

latitude
longitude
emergency_type

Query the hospital/places data source.

4. Hospital normalization

Convert different external API results into a consistent ERIS format:

name
address
latitude
longitude
emergency_capability
phone
source
5. Hospital ranking

Rank results using deterministic application logic.

The LLM should not decide the final ranking by itself.

Example conceptual scoring:

hospital suitability
        +
distance
        +
travel time
        +
verified emergency capability

The exact scoring weights can be adjusted during testing.

6. Routing

Request travel distance/time from the map/routing provider.

7. Safety rules

The backend should:

prevent unsupported medical diagnoses
avoid inventing hospital capabilities
avoid claiming real-time availability without real-time data
provide emergency escalation messaging
handle missing location
handle failed APIs gracefully
8. API error handling

The system must gracefully handle:

microphone failure
speech-to-text failure
AI failure
location permission denial
hospital API failure
routing API failure
no nearby hospitals
DATABASE

For a 24-hour MVP, a database should be minimal.

If persistence is not required by the judging criteria, the application can operate largely without a database.

Entity 1 — Emergency Sessions

Fields:

id
created_at
language
transcription
emergency_type
urgency
latitude
longitude
selected_hospital_id

Purpose:

debugging
demo/session state
optional analytics

Do not store unnecessary personal medical information.

Entity 2 — Hospitals

Only create this table if the organizers provide a hospital dataset or if we need to cache/normalize hospital information.

Fields:

id
name
address
latitude
longitude
phone
emergency_capability
specialties
source
last_verified_at
Entity 3 — Emergency Categories

Optional but useful for deterministic application logic.

Fields:

id
name
display_name
urgency_level
instruction_template

Example:

severe_bleeding
breathing_difficulty
chest_pain
trauma
unconsciousness
stroke_like_symptoms

The category list should remain small for the MVP.

AI

AI should be used in exactly four core places.

1. Speech understanding

Speech is converted to text using a speech-to-text service.

Example:

"Mere bhai ka accident ho gaya hai..."

becomes:

"My brother had an accident..."

or remains in the original language for processing.

2. Multilingual understanding

AI understands supported languages and maps different ways of describing the same emergency into a common internal representation.

For example:

"bahut khoon nikal raha hai"
        ↓
severe bleeding
3. Emergency information extraction

AI converts natural language into structured data.

Example:

{
  "emergency_type": "severe_bleeding",
  "urgency": "high"
}

Use structured output/schema validation wherever the selected AI service supports it.

4. Natural-language response

AI generates a short response in the user's language.

The response should be:

concise
calm
action-oriented
non-diagnostic
AI should NOT:
determine actual hospital availability without data
invent hospital capabilities
independently choose a hospital without application logic
provide definitive medical diagnoses
control navigation
make unsupported medical claims
EXTERNAL SERVICES

The exact providers should be finalized after checking hackathon rules, credits, and API availability.

1. AI / LLM

Primary candidate:

OpenAI API

Use for:

multilingual understanding
structured emergency extraction
response generation

Alternative:

Gemini
Claude

Only one primary runtime LLM is needed.

2. Speech-to-text

Possible choices:

OpenAI speech capabilities
Google Cloud Speech-to-Text
another approved speech API
browser speech recognition for a rapid fallback

Choose based on actual language accuracy and hackathon availability, not brand preference.

3. Maps / Places

Primary candidates:

Google Maps Platform
Mapbox
OpenStreetMap-based services

Required capabilities:

nearby hospital/places search
maps
geolocation support
routing/directions
4. Hospital data

Preferred order:

Organizer-provided verified dataset/API
Official hospital data/API
Trusted map/places provider
Other permitted public data

Do not manufacture hospital capabilities for the demo.

5. Navigation

Use the selected maps provider or an external navigation deep link.

6. Hosting

Use the fastest reliable deployment platform available to the team.

Possible choices include:

Vercel
Netlify
Cloudflare
Render
similar approved service

The deployment provider is not a product feature; choose whichever gets the application online fastest.

RECOMMENDED TECHNICAL ARCHITECTURE
                    USER
                     │
                     ▼
              React Web App
                     │
          ┌──────────┴──────────┐
          │                     │
      Microphone               GPS
          │                     │
          ▼                     ▼
    Speech-to-Text          User Location
          │
          ▼
       Backend
          │
          ▼
    AI / LLM Layer
          │
          ▼
 Structured Emergency Data
          │
          ▼
 Deterministic Application Logic
          │
       ┌──┴─────────────┐
       │                │
       ▼                ▼
 Hospital/Places       Routing
       │                │
       └───────┬────────┘
               ▼
        Ranked Hospitals
               │
               ▼
          Map + Route
               │
               ▼
             USER
Recommended implementation

Frontend

React
Vite
Tailwind CSS
shadcn/ui or equivalent component library

Backend

Node.js
Express or lightweight API routes

Database

Supabase/PostgreSQL only if needed

AI

One primary LLM API
One speech-to-text service

Maps

One map/places/routing provider

Avoid building multiple backends or microservices.
