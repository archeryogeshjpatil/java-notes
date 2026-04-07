# Smart CV powered by Archer Infotech - Project Planning

> **Reference:** Full SRS document is available at  
> `Smart-CV-powered-by-Archer-Infotech/planning/smart cv planning srs.md`

---

## What is Smart CV?

**Smart CV** is an AI-driven web platform built by **Archer Infotech** that helps **students and fresh graduates** optimize their resumes for Applicant Tracking Systems (ATS). Users upload a resume (PDF or DOCX), and the platform uses **Google Gemini AI** to analyze, score, and provide actionable improvement suggestions. It also includes AI-powered resume regeneration and a CV Generator with multiple professional templates and PDF export.

**Everything runs 100% locally** — no cloud services, no third-party databases.

---

## Project Purpose

- Help job seekers get shortlisted by making their resumes ATS-friendly.
- Automatically extract key details (name, college, contact info, GitHub/social links) from uploaded resumes.
- Provide a composite ATS score (0-100) based on keyword relevance, formatting, and professional language.
- Detect defects such as missing sections, weak action verbs, and ATS-unfriendly formatting.
- Offer context-aware recommendations for improving resume content and structure.
- AI-powered resume regeneration that fixes all defects and applies recommendations automatically.
- Generate professional CVs using multiple templates with live preview and PDF download.
- Store resume analysis history locally for logged-in users.

---

## Tech Stack

| Layer       | Technology                                                        |
|-------------|-------------------------------------------------------------------|
| Frontend    | Next.js 16 (TypeScript), Tailwind CSS v4, Framer Motion          |
| Backend     | FastAPI (Python), Uvicorn                                         |
| AI Engine   | Google Gemini 2.5 Flash (via google-genai SDK) with model fallback|
| File Parsing| PyPDF (PDF), python-docx (DOCX)                                   |
| Database    | SQLite (local, zero config)                                       |
| Auth        | JWT tokens + PBKDF2 password hashing (all local)                 |
| PDF Export  | html2canvas + jsPDF                                               |

---

## Core Features (All Implemented)

1. **Resume Upload & Parsing** - Drag-and-drop upload supporting PDF and DOCX formats.
2. **AI-Powered ATS Analysis** - Gemini AI evaluates the resume and returns structured JSON with score, defects, and recommendations. Works without login.
3. **Detail Extraction** - Automatically pulls out name, college, email, phone, GitHub link, and other social links.
4. **ATS Score (0-100)** - Composite score with visual ring indicator, color-coded by performance (green/amber/red).
5. **Defect Detection** - Flags missing contact info, weak verbs, formatting issues, grammar errors.
6. **Smart Recommendations** - Contextual suggestions for bullet points, section headers, and keyword optimization.
7. **AI Resume Regeneration** (requires login) - After analysis, AI asks clarifying questions for missing info, then generates an improved, defect-free resume with all recommendations applied. Auto-fills the CV Generator form.
8. **CV Template Generator** - 3 professional templates (Minimal, Modern, Elegant) with live preview and instant PDF download. Supports auto-fill from AI regeneration.
9. **Authentication System** - Login/Signup with JWT tokens, passwords hashed with PBKDF2. All local.
10. **Dashboard** - View resume analysis history, stats (total analyzed, average score, best score), and delete old entries.
11. **Local SQLite Database** - Resume history stored locally with no cloud dependency.
12. **Responsive Navbar** - Navigation across all pages with mobile hamburger menu, auth-aware (login/logout).
13. **Gemini Model Fallback** - Tries gemini-2.5-flash, then gemini-2.0-flash, then gemini-2.0-flash-lite if quota is exceeded.
14. **Robust JSON Parsing** - Multi-strategy JSON extraction from AI responses (direct parse, code block extraction, brace matching, auto-fix for trailing commas and control characters).

---

## Access Control

| Feature              | Without Login | With Login |
|----------------------|:------------:|:----------:|
| Resume Analysis      | Yes          | Yes        |
| View Results         | Yes          | Yes        |
| Regenerate Resume    | No           | Yes        |
| CV Generator         | Yes          | Yes        |
| Dashboard / History  | No           | Yes        |

---

## Pages

| Route           | Description                                        |
|-----------------|----------------------------------------------------|
| `/`             | Landing page — upload resume, view analysis, regenerate |
| `/generate`     | CV Generator — fill form (or auto-fill), pick template, PDF |
| `/dashboard`    | Resume history and stats (requires login)          |
| `/auth/login`   | Sign in with email/password                        |
| `/auth/signup`  | Create new account                                 |

---

## Regeneration Flow

```
1. User uploads resume → gets ATS score + defects + recommendations
2. User clicks "Regenerate Resume" (must be logged in)
3. AI analyzes defects → generates clarifying questions
   e.g., "What is your GitHub URL?", "Quantify your TCS achievement?"
4. User answers questions in chat-like UI
5. AI generates improved resume (defect-free, recommendations applied)
6. Auto-redirects to CV Generator with all fields pre-filled
7. User picks template → downloads PDF
```

---

## Architecture Overview

```
User (Browser)
    |
    v
Next.js Frontend (localhost:3000)
    |
    ├── POST /analyze → Upload & analyze resume (no auth)
    │       ├── Extract text (PyPDF / python-docx)
    │       ├── Send to Gemini AI (2.5-flash → 2.0-flash → 2.0-flash-lite)
    │       └── Return structured JSON + raw text
    |
    ├── POST /regenerate/questions → Get clarifying questions (auth required)
    ├── POST /regenerate → Generate improved resume (auth required)
    |
    ├── POST /auth/signup, /auth/login → JWT token returned
    ├── GET  /auth/me → Verify token
    ├── POST /history/save → Save analysis (auth required)
    ├── GET  /history → Get user's history (auth required)
    └── DELETE /history/:id → Delete entry (auth required)

    All data stored in SQLite (backend/smartcv.db)
```

---

## API Endpoints

| Method | Endpoint               | Auth | Description                              |
|--------|------------------------|------|------------------------------------------|
| GET    | `/`                    | No   | Health check — API status                |
| POST   | `/auth/signup`         | No   | Create account, returns JWT              |
| POST   | `/auth/login`          | No   | Login, returns JWT                       |
| GET    | `/auth/me`             | Yes  | Verify token, get user info              |
| POST   | `/analyze`             | No   | Upload resume and get AI analysis        |
| POST   | `/regenerate/questions`| Yes  | Get clarifying questions for regeneration|
| POST   | `/regenerate`          | Yes  | Generate improved resume from answers    |
| POST   | `/history/save`        | Yes  | Save analysis result to history          |
| GET    | `/history`             | Yes  | Get user's resume history                |
| DELETE | `/history/:id`         | Yes  | Delete a history entry                   |

---

## How to Run

### Backend
```bash
cd backend
source venv/bin/activate
pip install -r requirements.txt
python main.py
# Runs on http://localhost:8000
```

### Frontend
```bash
cd frontend
npm install
npm run dev
# Runs on http://localhost:3000
```

### Required: Add Gemini API key
```bash
# In backend/.env
GEMINI_API_KEY=your-google-gemini-api-key
```

---

## Project Location

```
~/Documents/Smart-CV-powered-by-Archer-Infotech/
├── backend/
│   ├── main.py          # FastAPI app with all endpoints
│   ├── utils.py         # PDF/DOCX extraction + Gemini AI + regeneration
│   ├── database.py      # SQLite operations (users, history)
│   ├── auth.py          # JWT token + password hashing
│   ├── requirements.txt
│   └── .env             # Gemini API key
├── frontend/
│   ├── src/app/         # Pages (/, /generate, /dashboard, /auth/*)
│   ├── src/components/  # Navbar
│   └── src/lib/         # api.ts (local API client with auth, history, regeneration)
└── planning/            # SRS document
```

---

*Archer Infotech — Built for the next generation of engineers.*
