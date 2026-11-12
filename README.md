# 🩺 MediEcho: AI Medical Voice Agent

[![Next.js](https://img.shields.io/badge/Next.js-15.3-black?style=flat&logo=next.js)](https://nextjs.org/)
[![React](https://img.shields.io/badge/React-19.0-blue?style=flat&logo=react)](https://react.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue?style=flat&logo=typescript)](https://www.typescriptlang.org/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-v4-38B2AC?style=flat&logo=tailwind-css)](https://tailwindcss.com/)
[![Drizzle ORM](https://img.shields.io/badge/Drizzle_ORM-0.44-C5F74F?style=flat)](https://orm.drizzle.team/)
[![Neon Database](https://img.shields.io/badge/Neon_DB-Serverless_PostgreSQL-00E599?style=flat&logo=postgresql)](https://neon.tech/)
[![Clerk](https://img.shields.io/badge/Clerk-Auth-6C47FF?style=flat&logo=clerk)](https://clerk.com/)
[![Vapi AI](https://img.shields.io/badge/Vapi-Voice_AI-FF6B6B?style=flat)](https://vapi.ai/)

> **MediEcho** is a voice-first telemedicine SaaS application enabling real-time, bidirectional voice consultations with specialized AI doctor personas, automated symptom matching, and post-call clinical report generation.

👉 **Looking for comprehensive technical architecture, API references, and database schemas? Read the full [DOCUMENTATION.md](./DOCUMENTATION.md).**

---

## 🌟 Key Features

- **🎙️ Bidirectional Voice Consultations**: Talk naturally to an AI doctor with ultra-low latency, turn-taking, and barge-in interruption handling via **Vapi**, **Deepgram**, and **PlayHT**.
- **🧠 Intelligent Doctor Matching**: Enter symptoms to receive AI-powered recommendations for the most relevant medical specialists.
- **🩺 10 Specialized Doctor Personas**: General Physician, Pediatrician, Dermatologist, Psychologist, Nutritionist, Cardiologist, ENT, Orthopedic, Gynecologist, and Dentist.
- **📋 Automated Clinical Reports**: After every session, an LLM parses the conversation into a structured medical report (Chief Complaint, Symptoms, Severity, Duration, Recommendations).
- **📊 Consultation History & Dashboard**: Securely review past consultations and access clinical reports.
- **🔐 Protected Authentication**: User onboarding, session management, and route protection powered by **Clerk**.

---

## 🛠️ Tech Stack

- **Frontend**: Next.js 15 (App Router), React 19, TypeScript, Tailwind CSS v4, Motion (Framer Motion), Radix UI, Lucide Icons, Sonner.
- **Voice AI**: Vapi Web SDK (`@vapi-ai/web`), Deepgram Nova-2 (STT), PlayHT (TTS).
- **Intelligence**: OpenAI SDK with OpenRouter (`gpt-3.5-turbo`).
- **Database**: Neon Serverless PostgreSQL with Drizzle ORM (`drizzle-orm`, `drizzle-kit`).
- **Auth**: Clerk (`@clerk/nextjs`).

---

## 🚀 Quick Start

### 1. Clone & Install

```bash
git clone https://github.com/jayadityagandham/AI-Medical-Voice-Agent.git
cd AI-Medical-Voice-Agent
npm install
```

### 2. Configure Environment Variables

Create a `.env.local` file in the root directory:

```env
# Clerk Authentication
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up

# Neon Database URL
DATABASE_URL=postgresql://user:password@ep-xyz.us-east-2.aws.neon.tech/neondb?sslmode=require

# Vapi Web API Key
NEXT_PUBLIC_VAPI_API_KEY=your_vapi_web_token

# OpenRouter / OpenAI API Key
OPEN_ROUTER_API_KEY=sk-or-v1-...
```

### 3. Push Database Schema

```bash
npx drizzle-kit push
```

### 4. Run Development Server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) to view the application.

---

## 🏗️ Architecture Overview

```mermaid
flowchart LR
    A[Patient Voice] --> B[Vapi Web SDK]
    B --> C[Deepgram STT]
    C --> D[OpenAI GPT-3.5]
    D --> E[PlayHT TTS]
    E --> F[Audio Response]
    B --> G[Transcript to /api/medical-report]
    G --> H[OpenRouter LLM Structured Report]
    H --> I[(Neon PostgreSQL via Drizzle)]
```

For complete technical specifications, sequence diagrams, and API docs, refer to **[DOCUMENTATION.md](./DOCUMENTATION.md)**.

---

## ⚠️ Medical Disclaimer

MediEcho is an artificial intelligence-assisted consultation demonstration and triage support tool. It does not provide definitive medical diagnoses or clinical treatment plans. In case of emergency, contact local emergency services immediately.

---

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.
