# HireSphere

### An AI-Powered Career Development and Recruitment Management System

HireSphere is a web application that connects **students** with **companies** through a single platform. It combines career preparation — CV analysis, skill-gap reports, practice tests, mock interviews, and an AI career chatbot — with recruitment management — job posting, applications, online assessments, shortlisting, and analytics. Trust between the two sides is built through company verification, fake-job protection, and transparent platform-verified hiring history.

The system has three portals: **Student**, **Company**, and **Admin**.

> HireSphere does not guarantee ATS results, job eligibility, selection, or placement. AI outputs are decision-support recommendations, and final hiring decisions remain with human recruiters.

---

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [Portals](#portals)
- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [Available Scripts](#available-scripts)
- [Environment Configuration](#environment-configuration)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

| | |
|---|---|
| **Document basis** | Software Requirements Specification v1.1 (IEEE 830 / ISO 29148) |
| **Status** | Draft for review |
| **Architecture** | Decoupled three-tier — React SPA, Spring Boot REST API, MySQL |
| **AI** | External AI service (no custom model trained) |
| **Release model** | MVP (first release) and Later (deferred features) |

HireSphere uses a modular monolith in its first release. The React single-page frontend communicates with the Spring Boot backend only through secured REST APIs, with Server-Sent Events (SSE) for long AI responses. The backend stores data in MySQL, keeps CV files and company verification documents in file storage (metadata in MySQL), and calls an external AI provider for language-based features.

---

## Key Features

### Career Preparation (Student-side)

- **Smart CV Analyzer** — Compare a CV with a specific job description (Case 1) or with all published jobs (Case 2). Produces an ATS compatibility result, a job-match percentage, an eligibility checklist, and a skill-gap report. Rule-based results appear first; AI suggestions load asynchronously afterward.
- **Practice Tests** — Aptitude and technical MCQs by topic, company, and difficulty with correct answers and explanations.
- **AI Mock Interview** — Text-based mock interviews (HR or technical) for a target role, with streamed AI feedback on relevance, completeness, and clarity.
- **AI Career Chatbot** — Conversational guidance for career, CV, interview, skill, and job-search questions, with streamed responses.
- **Job Recommendations & Alerts** — Notifications when newly published jobs match preferred roles or skills.

### Recruitment Management (Company-side)

- **Job Posting** — Verified companies publish and manage vacancies with required skills, eligibility criteria, salary, and deadlines.
- **Applications & Tracking** — Students apply once to an open job and track status through a defined lifecycle.
- **Timed Assessments** — Companies create MCQ tests from the question bank, assign them to applicants, and auto-evaluate.
- **Applicant Management & Shortlisting** — Review, filter, sort, and shortlist candidates; set final status with role and package.
- **Company Profile & Hiring History** — Platform-verified hiring history (students hired, average and highest package) per academic year, with privacy protection (aggregates only, hidden when fewer than 3 hires).

### Trust & Safety (Platform-wide)

- **Company Verification** — Registration requires a registration number (CIN/GSTIN), website, official work email, and proof documents. Administrators complete a verification checklist before approval.
- **Fake Job Protection** — First 3 jobs of a new company are held for admin review; scam indicator screening (payment requests, unrealistic promises, unofficial contact); student reports with auto-hide threshold.
- **Verified Badges** — Verified companies display a badge; job pages show a "Never pay money to apply" warning.

### Administration

- User management, company verification, job moderation, report handling, append-only audit log, and question bank management.

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Frontend** | React (SPA), Vite, responsive design (360 px – 1920 px) |
| **Backend** | Java 17+, Spring Boot, Spring Security, Spring Data JPA / Hibernate |
| **Database** | MySQL 8.x |
| **AI** | External AI provider API over HTTPS (accessed via Spring AI or provider SDK) |
| **Auth** | JWT (stateless), BCrypt password hashing, Role-Based Access Control (RBAC) |
| **Streaming** | Server-Sent Events (SSE) for long AI operations |
| **Document parsing** | Apache PDFBox or Apache Tika (CV text extraction, PDF in MVP) |
| **Email** | SMTP service for company email verification (MVP); other email notifications deferred |
| **File storage** | Local or cloud storage outside the public web root |

---

## Portals

### Student Portal
Dashboard, CV Analyzer, Job list and details, Company profile and hiring history, Application Tracker, Practice tests and timed tests, AI mock interview, Career chatbot, Followed companies, Report job, Profile and settings. *(Activity and points — Later)*

### Company Portal
Dashboard, Manage jobs, Job form, Applicants and shortlisting, Create tests and results, Company profile and gallery, Hiring history and career paths, Verification documents.

### Admin Portal
Dashboard, Users, Company verification (with checklist), Job moderation and review queue, Reports queue, Question bank, Announcements, Audit log.

---

## Getting Started

### Prerequisites

- **Java** 17 or later
- **Node.js** 18+ and npm
- **MySQL** 8.x
- An **AI provider API key** (TBD-01 — provider to be chosen)
- An **SMTP email service** for company email verification (TBD-03)

### 1. Clone the repository

```bash
git clone <repository-url>
cd HireSphere
```

### 2. Backend setup (Spring Boot)

```bash
cd backend
./mvnw clean install
# Configure your database and secrets (see Environment Configuration)
./mvnw spring-boot:run
```

The backend starts on `http://localhost:8080`. API docs are available at `/swagger-ui.html` (OpenAPI / Swagger UI).

### 3. Frontend setup (React + Vite)

```bash
cd frontend
npm install
npm run dev
```

The frontend starts on `http://localhost:5173`.

### 4. Database setup

Create a MySQL database and run the versioned migration scripts (see `backend/src/main/resources/db/migration`). The schema is applied through migration scripts (DR-06).

---

## Project Structure

```
HireSphere/
├── backend/                    # Spring Boot REST API
│   └── src/main/java/com/hiresphere/
│       ├── auth/                # Authentication & user management
│       ├── student/             # Student profiles
│       ├── job/                 # Job management & browsing
│       ├── cv/                  # Smart CV Analyzer
│       ├── application/         # Application tracker
│       ├── assessment/          # Tests & practice
│       ├── ai/                  # AI interview, chatbot, async tasks
│       ├── company/             # Company profiles & hiring history
│       ├── analytics/           # Dashboards & analytics
│       ├── notification/        # Notifications & follow priority
│       ├── admin/               # Administration & trust-safety
│       └── trustsafety/         # Company verification, job moderation
│   └── src/main/resources/
│       ├── application.yml      # Externalised configuration
│       └── db/migration/        # Versioned schema migrations
├── frontend/                    # React SPA (Vite)
│   └── src/
│       ├── pages/               # Screen components per portal
│       ├── components/          # Shared UI components
│       ├── services/            # API clients (REST + fetch-based SSE)
│       └── context/            # Auth, theme, notification context
└── docs/
    └── HireSphereSRS.pdf        # Software Requirements Specification v1.1
```

> Each backend module follows the layered pattern: **controller → service → repository → entity → DTO** (NFR-MNT-01).

---

## Available Scripts

### Frontend (`frontend/`)

| Command | Description |
|---|---|
| `npm run dev` | Start the Vite dev server |
| `npm run build` | Production build |
| `npm run preview` | Preview the production build locally |
| `npm run lint` | Run the linter |

### Backend (`backend/`)

| Command | Description |
|---|---|
| `./mvnw spring-boot:run` | Run the backend |
| `./mvnw clean install` | Build the backend |
| `./mvnw test` | Run automated tests |

---

## Environment Configuration

Secrets are supplied through environment configuration, never stored in source code (NFR-SEC-08). Create a `.env` file or set the following environment variables:

| Variable | Description |
|---|---|
| `DB_URL` | MySQL JDBC connection URL |
| `DB_USERNAME` | Database username |
| `DB_PASSWORD` | Database password |
| `JWT_SECRET` | Secret used to sign JWTs |
| `JWT_EXPIRATION` | JWT expiration time (default 24 hours) |
| `AI_API_KEY` | External AI provider API key |
| `AI_API_URL` | External AI provider endpoint |
| `SMTP_HOST` | Email service host (company email verification) |
| `SMTP_PORT` | Email service port |
| `SMTP_USERNAME` | Email service username |
| `SMTP_PASSWORD` | Email service password |
| `FILE_STORAGE_PATH` | Base path for CV and document file storage |
| `CORS_ALLOWED_ORIGINS` | Allowed frontend origin(s) for CORS |

> The same build runs in development and production through externalised configuration (NFR-MNT-04).

---

## Contributing

1. Create a branch from `main` for your feature or fix.
2. Keep backend modules in the controller → service → repository → entity → DTO layering.
3. Ensure core services (matching, scoring, authentication) have automated tests — target at least 60% line coverage (NFR-MNT-03).
4. Document all REST endpoints with OpenAPI annotations (NFR-MNT-02).
5. Run the build and tests before committing.

---

## License

This project is a college final-year project. See the project team for licensing details.
