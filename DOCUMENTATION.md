# 🩺 MediEcho: AI Medical Voice Agent — Full Technical Documentation

[![Next.js](https://img.shields.io/badge/Next.js-15.3-black?style=flat&logo=next.js)](https://nextjs.org/)
[![React](https://img.shields.io/badge/React-19.0-blue?style=flat&logo=react)](https://react.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue?style=flat&logo=typescript)](https://www.typescriptlang.org/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-v4-38B2AC?style=flat&logo=tailwind-css)](https://tailwindcss.com/)
[![Drizzle ORM](https://img.shields.io/badge/Drizzle_ORM-0.44-C5F74F?style=flat)](https://orm.drizzle.team/)
[![Neon Database](https://img.shields.io/badge/Neon_DB-Serverless_PostgreSQL-00E599?style=flat&logo=postgresql)](https://neon.tech/)
[![Clerk](https://img.shields.io/badge/Clerk-Auth-6C47FF?style=flat&logo=clerk)](https://clerk.com/)
[![Vapi AI](https://img.shields.io/badge/Vapi-Voice_AI-FF6B6B?style=flat)](https://vapi.ai/)

---

## 📖 Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Key Features & Capabilities](#2-key-features--capabilities)
3. [System Architecture & Flow](#3-system-architecture--flow)
4. [Technology Stack](#4-technology-stack)
5. [Directory & File Structure](#5-directory--file-structure)
6. [Database Schema & Models](#6-database-schema--models)
7. [API Endpoints Reference](#7-api-endpoints-reference)
8. [Voice AI Consultation Pipeline](#8-voice-ai-consultation-pipeline)
9. [AI Doctor Specialists Directory](#9-ai-doctor-specialists-directory)
10. [Environment Variables & Configuration](#10-environment-variables--configuration)
11. [Installation & Setup Guide](#11-installation--setup-guide)
12. [Clinical Disclaimer & HIPAA Security](#12-clinical-disclaimer--hipaa-security)
13. [Troubleshooting & FAQ](#13-troubleshooting--faq)

---

## 1. Executive Summary

**MediEcho (AI Medical Voice Agent)** is a full-stack, voice-first telemedicine SaaS application designed to revolutionize patient pre-consultation and triage. Built with **Next.js 15 (App Router)**, **TypeScript**, **React 19**, and **Tailwind CSS**, MediEcho provides an ultra-low-latency bidirectional conversational experience where patients can describe their symptoms naturally via speech.

The platform couples real-time conversational intelligence with automated clinical documentation:
- Patients describe health concerns via text or voice.
- An intelligent triage model suggests matching medical specialties.
- A dedicated **AI Doctor Persona** engages in a natural voice consultation with customized clinical guidance and real-time transcription.
- At the conclusion of the call, an LLM pipeline transforms the dialogue into a structured, clinical-grade medical report containing chief complaints, symptoms, severity, and actionable recommendations.
- All session records and consultation histories are persisted in **Neon Serverless PostgreSQL** via **Drizzle ORM** and protected by **Clerk** authentication.

---

## 2. Key Features & Capabilities

### 🎙️ Real-Time Voice Consultation
- Powered by the **Vapi Web SDK**, enabling ultra-low-latency audio streaming directly in the browser.
- Integrates **Deepgram (Nova-2)** for speech-to-text (STT) and **PlayHT** for expressive, human-like text-to-speech (TTS).
- Real-time live transcript updates with active speaker role tracking (`user` vs `assistant`).
- Native turn-taking, barge-in detection (user interruption handling), and silence detection.

### 🧠 Intelligent Doctor Matching
- When a user submits symptoms, the `/api/suggest-doctors` endpoint evaluates the description using an LLM against 10 clinical specialties.
- Automatically recommends the top 3 best-suited doctor personas (e.g., Dermatologist for rashes, Cardiologist for chest flutter).

### 🩺 10 Specialized AI Doctor Personas
- Distinct medical personas including **General Physician**, **Pediatrician**, **Dermatologist**, **Psychologist**, **Nutritionist**, **Cardiologist**, **ENT Specialist**, **Orthopedic**, **Gynecologist**, and **Dentist**.
- Each persona features a tailored clinical system prompt and an assigned natural voice profile.

### 📋 Automated Clinical Report Generation
- Extracts structured clinical insights from conversational transcripts upon call termination.
- Generates standardized JSON reports covering:
  - **Chief Complaint**
  - **Clinical Summary**
  - **Identified Symptoms** (array)
  - **Duration & Severity Assessment** (`mild`, `moderate`, `severe`)
  - **Medications Mentioned**
  - **Lifestyle & Medical Recommendations**

### 🔐 User Dashboard & History
- Complete consultation history with timestamped records.
- Comprehensive modal report viewer with diagnostic tags and disclaimers.
- User credit quota system initialized upon sign-up.

---

## 3. System Architecture & Flow

### 3.1 High-Level Architecture Diagram

```mermaid
flowchart TD
    subgraph Client["Client Tier (Browser)"]
        UI["Next.js 15 UI / React 19"]
        ClerkAuth["Clerk Auth Provider"]
        VapiSDK["Vapi Web Audio SDK"]
    end

    subgraph VoiceLoop["Real-Time Voice Pipeline (Vapi)"]
        Deepgram["Deepgram Nova-2 (STT)"]
        LLM_Voice["OpenAI GPT-3.5-Turbo (Dialogue)"]
        PlayHT["PlayHT (TTS Voices)"]
    end

    subgraph BackendTier["Next.js Route Handlers (Edge / Node)"]
        AuthMiddleware["Clerk Middleware (Route Protection)"]
        APISuggest["/api/suggest-doctors"]
        APISession["/api/session-chat"]
        APIReport["/api/medical-report"]
        APIUsers["/api/users"]
    end

    subgraph Services["External AI & Cloud Database"]
        OpenRouter["OpenRouter / OpenAI API"]
        NeonDB[("Neon Serverless PostgreSQL")]
    end

    UI -->|1. Authenticate| ClerkAuth
    ClerkAuth -->|Protect Routes| AuthMiddleware
    UI -->|2. Submit Symptoms| APISuggest
    APISuggest -->|Analyze Symptoms| OpenRouter
    UI -->|3. Create Session| APISession
    APISession -->|Save Session| NeonDB
    UI -->|4. Connect Microphone| VapiSDK
    VapiSDK <-->|Live Audio Stream| VoiceLoop
    VoiceLoop <--> Deepgram
    VoiceLoop <--> LLM_Voice
    VoiceLoop <--> PlayHT
    VapiSDK -->|5. End Call & Transcript| APIReport
    APIReport -->|Structure Report Prompt| OpenRouter
    APIReport -->|Persist Report & Transcript| NeonDB
    UI -->|6. Fetch Past Reports| APISession
```

---

### 3.2 Consultation Lifecycle Sequence

```mermaid
sequenceDiagram
    autonumber
    actor Patient as User (Patient)
    participant Modal as Add Session Modal
    participant API as Next.js API Routes
    participant LLM as OpenAI / OpenRouter
    participant Vapi as Vapi Voice Service
    participant DB as Neon PostgreSQL (Drizzle)

    Patient->>Modal: Inputs symptoms/notes
    Modal->>API: POST /api/suggest-doctors { notes }
    API->>LLM: Evaluate symptoms against specialist list
    LLM-->>API: Returns recommended doctor IDs [1, 3, 6]
    API-->>Modal: Returns top 3 suggested doctors
    Patient->>Modal: Selects doctor & clicks "Start Consultation"
    Modal->>API: POST /api/session-chat { notes, selectedDoctor }
    API->>DB: Insert session with UUID into sessionChatTable
    API-->>Modal: Returns sessionId
    Modal->>Patient: Redirects to /dashboard/medical-agent/[sessionId]

    Patient->>Vapi: Clicks "Start Call" (Microphone stream)
    loop Conversational Turn Taking
        Patient->>Vapi: Speaks symptom detail
        Vapi->>LLM: Streaming context + persona prompt
        LLM-->>Vapi: Generates clinical answer
        Vapi-->>Patient: Plays synthesized voice response + emits live transcript
    end

    Patient->>Vapi: Clicks "End Call" (or hangup phrase)
    Vapi-->>Patient: Call stopped, returns complete message history
    Patient->>API: POST /api/medical-report { sessionId, messages, sessionDetail }
    API->>LLM: Generate structured clinical JSON report
    LLM-->>API: Standardized 11-field JSON
    API->>DB: UPDATE sessionChatTable SET report, conversation
    API-->>Patient: Returns completed report
    Patient->>Patient: Redirect to /dashboard with toast notification
```

---

## 4. Technology Stack

| Domain | Technology | Version | Purpose |
|---|---|---|---|
| **Framework** | Next.js (App Router) | `15.3.4` | Server-rendered React application & API routes |
| **Runtime / Library** | React & React DOM | `19.0.0` | Declarative UI component library |
| **Language** | TypeScript | `^5.0.0` | Static type safety across client and server |
| **Styling** | Tailwind CSS | `^4.0.0` | Utility-first CSS styling engine |
| **UI Components** | Radix UI Primitives | Latest | Accessible modal dialogs and slot elements |
| **Animations** | Motion (Framer) | `^12.19.1` | Fluid page and landing animations |
| **Icons** | Lucide React & Tabler Icons | `^0.522.0` | Modern, consistent iconography |
| **Authentication** | Clerk | `^6.23.0` | User authentication, sessions, and route guards |
| **Database** | Neon Serverless PostgreSQL | `^1.0.1` | Serverless SQL database via HTTP connection |
| **ORM** | Drizzle ORM | `^0.44.4` | Lightweight, performant TypeScript ORM |
| **Database Migration** | Drizzle Kit | `^0.31.2` | Schema migration tool |
| **Voice Orchestration** | Vapi Web SDK | `^2.3.8` | Real-time WebSocket audio client |
| **STT Engine** | Deepgram (Nova-2) | Integrated | Ultra-fast speech recognition via Vapi |
| **TTS Engine** | PlayHT | Integrated | Natural voice synthesis via Vapi |
| **LLM Provider** | OpenAI / OpenRouter | `^5.8.2` | GPT-3.5-Turbo for triage and report structuring |
| **Notifications** | Sonner | `^2.0.7` | Toast notification alerts |
| **Date Formatting** | Moment.js | `^2.30.1` | Human-readable date and time formatting |

---

## 5. Directory & File Structure

```
ai-medical-voice-agent/
├── app/
│   ├── (auth)/
│   │   ├── sign-in/[[...sign-in]]/page.tsx       # Clerk Sign-In Page
│   │   └── sign-up/[[...sign-up]]/page.tsx       # Clerk Sign-Up Page
│   ├── (routes)/
│   │   └── dashboard/
│   │       ├── _components/
│   │       │   ├── AddNewSessionDialog.tsx        # Consultation intake & doctor matcher modal
│   │       │   ├── AppHeader.tsx                  # Dashboard navigation & user profile
│   │       │   ├── DoctorAgentCard.tsx            # Specialist card component
│   │       │   ├── DoctorsAgentList.tsx           # Full catalog of 10 specialists
│   │       │   ├── HistoryList.tsx                # Consultation history wrapper
│   │       │   ├── HistoryTable.tsx               # Tabular view of past sessions
│   │       │   ├── SuggestedDoctorCard.tsx        # Recommended doctor selector card
│   │       │   └── ViewReportDialog.tsx           # Full clinical report modal viewer
│   │       ├── history/page.tsx                   # Dedicated session history view
│   │       ├── medical-agent/[sessionId]/page.tsx # Live voice consultation room
│   │       ├── layout.tsx                         # Dashboard authenticated layout
│   │       └── page.tsx                           # Main Dashboard overview
│   ├── _components/
│   │   └── FeatureBentoGrid.tsx                   # Landing page features grid
│   ├── api/
│   │   ├── medical-report/route.tsx               # POST: Generates & saves clinical report
│   │   ├── session-chat/route.tsx                 # GET/POST: Session management
│   │   ├── suggest-doctors/route.tsx              # POST: Evaluates symptoms & matches doctor
│   │   └── users/route.tsx                        # POST: User registration & credit init
│   ├── favicon.ico                                # App favicon
│   ├── globals.css                                # Global CSS & Tailwind imports
│   ├── layout.tsx                                 # Root layout with ClerkProvider & Toaster
│   ├── page.tsx                                   # Hero landing page ("MediEcho")
│   └── provider.tsx                               # User context & credits synchronization
├── components/
│   ├── ui/                                        # Radix + Tailwind base components
│   │   ├── bento-grid.tsx
│   │   ├── button.tsx
│   │   ├── dialog.tsx
│   │   ├── sonner.tsx
│   │   ├── table.tsx
│   │   └── textarea.tsx
│   └── hero-section-demo-1.tsx                    # Interactive landing preview
├── config/
│   ├── db.tsx                                     # Neon serverless HTTP client + Drizzle
│   ├── OpenAiModel.tsx                            # OpenRouter / OpenAI SDK client
│   └── schema.tsx                                 # PostgreSQL table definitions
├── context/
│   └── UserDetailContext.tsx                      # React Context for user profile & credits
├── lib/
│   └── utils.ts                                   # clsx + twMerge utility functions
├── public/                                        # Static image assets (Doctor avatars 1-10)
├── shared/
│   └── list.tsx                                   # 10 Doctor Agent definitions & prompts
├── drizzle.config.ts                              # Drizzle Kit configuration
├── middleware.ts                                  # Clerk route authorization guard
├── next.config.ts                                 # Next.js configuration
├── package.json                                   # Dependencies & project scripts
├── tsconfig.json                                  # TypeScript configuration
├── README.md                                      # Repository overview
└── DOCUMENTATION.md                               # Full technical documentation
```

---

## 6. Database Schema & Models

Database management is powered by **Neon Serverless PostgreSQL** through **Drizzle ORM** with HTTP connection pooling.

### 6.1 `users` Table
Stores authenticated user metadata and consultation credit balance.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY`, `GENERATED ALWAYS AS IDENTITY` | Unique internal user ID |
| `name` | `varchar(255)` | `NOT NULL` | Full name of the user from Clerk |
| `email` | `varchar(255)` | `NOT NULL`, `UNIQUE` | Primary email address |
| `credits` | `integer` | Default `10` | Available consultation credits |

### 6.2 `sessionChatTable`
Stores consultation sessions, selected doctor configuration, live transcripts, and the structured clinical report.

| Column | Type | Constraints | Description |
|---|---|---|---|
| `id` | `integer` | `PRIMARY KEY`, `GENERATED ALWAYS AS IDENTITY` | Auto-incrementing primary key |
| `sessionId` | `varchar` | `NOT NULL` | UUIDv4 session identifier |
| `notes` | `text` | Nullable | Preliminary symptoms entered by user |
| `selectedDoctor`| `json` | Nullable | Full doctor agent object (specialist, voice, prompt) |
| `conversation` | `json` | Nullable | Array of conversation turns `[{role, text}]` |
| `report` | `json` | Nullable | Standardized 11-field clinical report JSON |
| `createdBy` | `varchar` | `REFERENCES users(email)` | Email of the patient who created the session |
| `createdOn` | `varchar` | Nullable | Date string when session was created |

### 6.3 Structured Clinical Report Schema
The `report` column holds a strictly validated JSON structure:

```typescript
type MedicalReport = {
  sessionId: string;            // UUID session identifier
  agent: string;                // e.g., "General Physician AI"
  user: string;                 // Patient full name or "Anonymous"
  timestamp: string;            // ISO 8601 timestamp
  chiefComplaint: string;       // One-sentence primary concern
  summary: string;              // 2-3 sentence clinical narrative
  symptoms: string[];           // Array of recognized symptom strings
  duration: string;             // Duration description (e.g., "3 days")
  severity: "mild" | "moderate" | "severe" | "Not specified";
  medicationsMentioned: string[]; // Prescription/OTC meds discussed
  recommendations: string[];    // Actionable care suggestions
};
```

---

## 7. API Endpoints Reference

### 7.1 User Registration & Credits (`POST /api/users`)
Synchronizes Clerk authentication with the PostgreSQL database.
- **Access**: Protected (Clerk session required)
- **Request**: Empty payload (reads `currentUser()` from Clerk headers)
- **Behavior**:
  - Checks if a user record exists with `email`.
  - If missing, inserts a new user record with `credits: 10`.
  - Returns the user database record.

### 7.2 Doctor Suggestion (`POST /api/suggest-doctors`)
Analyzes symptom notes and returns the top 3 best-matching doctors.
- **Access**: Protected
- **Request Body**:
  ```json
  {
    "notes": "I have had a high fever and a red itchy rash on my arm for two days."
  }
  ```
- **Processing**:
  - Prompts `gpt-3.5-turbo` with the system catalog of `AIDoctorAgents`.
  - Parses returned JSON array of IDs (e.g., `[3, 1, 7]`).
- **Response**: Array of up to 3 `AIDoctorAgents` objects. Defaults to the first 3 agents as fallback if parsing fails.

### 7.3 Session Management (`/api/session-chat`)
#### Create Session (`POST /api/session-chat`)
- **Request Body**:
  ```json
  {
    "notes": "Persistent dry cough and fatigue",
    "selectedDoctor": { ...AIDoctorAgent }
  }
  ```
- **Response**: Newly created `sessionChatTable` record containing a newly minted `sessionId` (UUIDv4).

#### Retrieve Session (`GET /api/session-chat?sessionId={id}`)
- **Query Parameter**:
  - `sessionId=all`: Returns all sessions created by the authenticated user in descending order of ID.
  - `sessionId=<uuid>`: Returns the specific session record for the active call room or report view.

### 7.4 Medical Report Generation (`POST /api/medical-report`)
Compiles the conversation history into a structured medical report.
- **Request Body**:
  ```json
  {
    "sessionId": "b6a8d712-421b-4fc4-9a08-41e97682d385",
    "sessionDetail": { ...sessionDetail },
    "messages": [
      { "role": "user", "text": "I've had a sore throat for three days." },
      { "role": "assistant", "text": "Do you have any difficulty swallowing?" }
    ]
  }
  ```
- **Processing**:
  - Sends the formatted conversation to `gpt-3.5-turbo` with the 11-field clinical prompt.
  - Cleans and parses markdown code blocks (````json`).
  - Guarantees all 11 fields exist with fallback values.
  - Updates `sessionChatTable` with `report` and `conversation`.
- **Response**: The complete `MedicalReport` JSON object.

---

## 8. Voice AI Consultation Pipeline

The real-time voice loop is handled in `app/(routes)/dashboard/medical-agent/[sessionId]/page.tsx` through `@vapi-ai/web`:

```mermaid
sequenceDiagram
    participant Browser as Browser Client
    participant VapiSDK as Vapi Web Client
    participant VapiCloud as Vapi Orchestrator
    participant Deepgram as Deepgram (STT)
    participant Model as GPT-3.5-Turbo
    participant Voice as PlayHT (TTS)

    Browser->>VapiSDK: new Vapi(NEXT_PUBLIC_VAPI_API_KEY)
    Browser->>VapiSDK: vapi.start(VapiAgentConfig)
    VapiSDK->>VapiCloud: WebSocket Audio Session
    VapiCloud-->>Browser: Event: 'call-start'
    
    Browser->>VapiCloud: Stream Microphone Audio
    VapiCloud->>Deepgram: Real-time Audio Stream
    Deepgram-->>VapiCloud: Live Text Tokens
    VapiCloud-->>Browser: Event: 'message' (transcriptType: 'partial')
    Deepgram-->>VapiCloud: Final Sentence
    VapiCloud-->>Browser: Event: 'message' (transcriptType: 'final')
    
    VapiCloud->>Model: Send Message + System Prompt
    Model-->>VapiCloud: Response Stream
    VapiCloud->>Voice: Text-to-Speech Generation
    Voice-->>VapiCloud: Audio Stream
    VapiCloud-->>Browser: Playback Audio Stream + Event: 'speech-start'
    VapiCloud-->>Browser: Event: 'speech-end'
```

### 8.1 Configuration Parameters
- **Transcriber**: Deepgram (`nova-2`), English language model.
- **Voice**: PlayHT, dynamically assigned voice ID per doctor persona (e.g., `will`, `chris`, `susan`, `charlotte`).
- **Model**: OpenAI `gpt-3.5-turbo`, configured with a maximum token limit of 150 and temperature of 0.7 for concise, conversational responses.
- **Safety Limits**:
  - Silence timeout: 30 seconds
  - Maximum duration: 600 seconds (10 minutes)
  - End-call recognition phrases: `["goodbye", "end call", "hang up", "that's all"]`

---

## 9. AI Doctor Specialists Directory

| ID | Specialist | Voice ID | System Persona Prompt Summary |
|---|---|---|---|
| **1** | **General Physician** | `will` | Friendly GP AI. Quick symptom triage with concise, supportive general health advice. |
| **2** | **Pediatrician** | `chris` | Gentle Pediatrician AI. Focuses on children's health from infants to teens with safe recommendations. |
| **3** | **Dermatologist** | `sarge` | Knowledgeable Dermatologist AI. Analyzes rashes, acne, skin infections, and gives home care guidance. |
| **4** | **Psychologist** | `susan` | Empathetic Mental Health AI. Explores emotional state and offers calming, supportive coping techniques. |
| **5** | **Nutritionist** | `eileen` | Motivational Dietary AI. Inquires about current dietary intake, routines, and sustainable lifestyle goals. |
| **6** | **Cardiologist** | `charlotte`| Calm Cardiac Specialist AI. Inquires about chest discomfort, palpitations, and blood pressure history. |
| **7** | **ENT Specialist** | `ayla` | Friendly Ear, Nose, and Throat AI. Gathers symptoms related to sinus, throat, hearing, and allergies. |
| **8** | **Orthopedic** | `aaliyah` | Supportive Musculoskeletal AI. Evaluates joint, muscle, or spine pain and mobility limitations. |
| **9** | **Gynecologist** | `hudson` | Respectful Women's Health AI. Gentle, reassuring inquiries into reproductive and hormonal wellness. |
| **10**| **Dentist** | `atlas` | Cheerful Dental AI. Reviews toothaches, gum discomfort, dental hygiene, and emergency guidance. |

---

## 10. Environment Variables & Configuration

Create a `.env.local` file in the root directory with the following variables:

```bash
# ==============================================================================
# CLERK AUTHENTICATION CONFIGURATION
# ==============================================================================
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up

# ==============================================================================
# NEON SERVERLESS POSTGRESQL DATABASE
# ==============================================================================
# Format: postgresql://[user]:[password]@[endpoint].neon.tech/[dbname]?sslmode=require
DATABASE_URL=postgresql://neondb_owner:your_password@ep-cool-resonance-123456.us-east-2.aws.neon.tech/neondb?sslmode=require

# ==============================================================================
# VAPI VOICE AI SDK
# ==============================================================================
# Public API key for Vapi client-side Web SDK
NEXT_PUBLIC_VAPI_API_KEY=your_vapi_public_api_key

# ==============================================================================
# OPENAI / OPENROUTER API
# ==============================================================================
# Used for symptom triage and medical report generation via OpenRouter
OPEN_ROUTER_API_KEY=sk-or-v1-...
```

---

## 11. Installation & Setup Guide

### 11.1 Prerequisites
- **Node.js**: `v18.18.0` or higher (Node.js 20+ recommended)
- **Package Manager**: `npm`, `pnpm`, or `yarn`
- Cloud Accounts:
  - [Clerk Dashboard](https://dashboard.clerk.com) for authentication
  - [Neon Console](https://console.neon.tech) for serverless Postgres
  - [Vapi Dashboard](https://dashboard.vapi.ai) for voice API tokens
  - [OpenRouter](https://openrouter.ai) or [OpenAI](https://platform.openai.com) for LLM keys

### 11.2 Step-by-Step Installation

```bash
# 1. Clone the repository
git clone https://github.com/jayadityagandham/AI-Medical-Voice-Agent.git
cd AI-Medical-Voice-Agent

# 2. Install dependencies
npm install

# 3. Configure environment variables
cp .env.example .env.local
# (Edit .env.local with your API keys)

# 4. Synchronize database schema with Neon
npx drizzle-kit push

# 5. Start the development server
npm run dev
```

Visit `http://localhost:3000` to interact with the application.

### 11.3 Building for Production

```bash
# Build the production bundle
npm run build

# Start the production server
npm run start
```

---

## 12. Clinical Disclaimer & HIPAA Security

### ⚠️ Medical Disclaimer
> **IMPORTANT**: MediEcho is an artificial intelligence-assisted consultation demonstration and triage support tool. **It does not provide definitive medical diagnoses, prescriptions, or clinical treatment plans.** Patients experiencing severe symptoms, chest pain, difficulty breathing, or acute trauma must immediately contact emergency services (e.g., 911 or local emergency numbers).

### 🔒 Privacy & HIPAA Considerations
For production deployment handling Protected Health Information (PHI):
1. **Business Associate Agreements (BAA)**: Must be executed with Clerk, Neon DB, Vapi, and OpenAI/OpenRouter.
2. **Encryption**: Neon provides encryption in transit (TLS 1.3) and at rest (AES-256).
3. **Audit Logging**: Implement session access logs to track consultations and medical report retrieval.

---

## 13. Troubleshooting & FAQ

#### Q: Microphone permission is denied or no audio is transmitted.
- **Solution**: Ensure your browser has granted microphone access to `localhost:3000` or your deployed domain. Check browser permissions in the URL bar and test your input device in OS settings.

#### Q: Vapi fails with `401 Unauthorized` on call start.
- **Solution**: Verify that `NEXT_PUBLIC_VAPI_API_KEY` is present in `.env.local` and that the key starts with the public client prefix provided in the Vapi dashboard.

#### Q: Database query throws connection error or timeout.
- **Solution**: Check that `DATABASE_URL` contains `?sslmode=require`. Verify in the Neon console that the compute instance is active and not suspended.

#### Q: Report generation fails after the call ends.
- **Solution**: Ensure `OPEN_ROUTER_API_KEY` has active credit balance on OpenRouter. The report generation requires at least 2 messages in the call transcript to prevent generating empty reports.

---

*Authored by [Jayaditya Gandham](https://github.com/jayadityagandham). Licensed under the MIT License.*
