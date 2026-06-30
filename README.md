# AI-Powered Applicant Tracking System

A full-stack recruitment management platform built with Django REST Framework. Synergy streamlines the end-to-end hiring pipeline — from job requisition approval to interview scheduling — with AI-assisted resume screening powered by the Groq LLM API and a RAG-based semantic matching engine.

---

## Demo / Screenshots

> Live deployment isn't available since the project uses company-confidential data. Below are sanitized screenshots of the core workflows.

### Dashboard
HR overview — requirement stats, interview email status, and a live snapshot of the Kanban pipeline.
![Dashboard](./docs/screenshots/dashboard.png)

### My Profile
Account details and role/department visibility for the logged-in user.
![My Profile](./docs/screenshots/my-profile.png)

### Internal Job Board
Employees browse approved openings across departments, with filters by department, work mode, and employment type.
![Browse Jobs](./docs/screenshots/browse-jobs.png)

### New Requirement Form
Departments raise hiring requests with role details, required skills, salary range, and a job description.
![New Requirement Modal](./docs/screenshots/new-requirement-modal.png)

### Requirements Management
HR view of all raised requirements with status, skills, and approve/reject actions.
![Requirements List](./docs/screenshots/requirements-list.png)

### Resume Manager
Bulk/single resume upload, AI-powered JD screening, and per-candidate scheduling actions.
![Resume Manager](./docs/screenshots/resume-manager.png)

### Interview Scheduling
Shortlisted candidates with screening scores, scheduled date/venue, and email status.
![Interviews](./docs/screenshots/interviews.png)

### Kanban Pipeline
Drag-and-drop candidate tracking across Applied → Screening → Shortlisted → Interview Scheduled → Selected/Rejected.
![Kanban Pipeline](./docs/screenshots/kanban-pipeline.png)

### Departments / Roles / Skills (Admin)
Master data management for organization structure, access roles, and the skill list used in screening.

![Departments](./docs/screenshots/departments.png)
![Roles](./docs/screenshots/roles.png)
![Skills](./docs/screenshots/skills.png)

### Automated Interview Emails
Confirmation email sent automatically when a candidate is scheduled, via SMTP (Mailtrap in dev).
![Interview Email](./docs/screenshots/interview-email.png)

### Database Schema
![ER Diagram](./docs/screenshots/er-diagram.png)

---

## Features

- **Role-based access** — HR managers, recruiters, and interviewers each have scoped permissions via custom DRF permission classes
- **Job requisition workflow** — departments raise hiring requirements; HR approves or rejects with comments before recruiting begins
- **Resume management** — single and bulk PDF resume uploads with OCR fallback for scanned documents (Tesseract + pdf2image)
- **AI resume screening** — resumes are scored against job requirements using the Groq LLM; results are cached on the model to avoid repeated API calls
- **RAG-based semantic matching** — resumes are chunked using section-based splitting, embedded, and stored in a single ChromaDB collection (`resumes`); incoming job descriptions are embedded and compared against stored resume chunks using cosine similarity to surface the best-matching candidates
- **Kanban pipeline** — candidates move through stages: Applied → Screening → Shortlisted → Interview Scheduled → Selected / Rejected
- **SharePoint integration** — fetches resumes directly from a SharePoint document library via the Microsoft Graph API; when a candidate's status changes to *Shortlisted*, their profile/resume is automatically pushed back to SharePoint
- **Interview scheduling** — schedule interviews, record interviewer notes, and send confirmation emails via Mailtrap (dev) or any SMTP provider
- **JWT authentication** — access + refresh tokens with rotation and blacklisting on logout
- **Auto-generated API docs** — Swagger UI available at `/api/docs/`

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Django 6.0.5, Django REST Framework |
| Auth | djangorestframework-simplejwt |
| Database | SQLite (dev) |
| AI Screening | Groq API (LLaMA) |
| RAG / Embeddings | ChromaDB (single `resumes` collection), section-based chunking, cosine similarity |
| Document Source | SharePoint (Microsoft Graph API) — resume fetch + shortlisted-status sync |
| PDF Parsing | pypdf, pdf2image, pytesseract |
| Email | Mailtrap SMTP (dev) |
| API Docs | drf-spectacular (Swagger) |
| Frontend | Vanilla JS + HTML templates (served by Django) |

---

## Architecture Overview

### Database Schema

The ER diagram below shows the core entities — Employee, Department, Role, Skill, Requirement, Resume, and Interview — and how they relate.

![ER Diagram](./docs/screenshots/er-diagram.png)

### RAG Resume Matching Pipeline

1. Resume PDFs are parsed (with OCR fallback for scanned documents)
2. Text is split using **section-based chunking** (e.g. Experience, Education, Skills sections kept intact rather than arbitrary token windows)
3. Each chunk is embedded and stored in a single ChromaDB collection (`resumes`), tagged with candidate/resume metadata
4. When a new job description (JD) comes in, it's embedded the same way
5. **Cosine similarity** is computed between the JD embedding and stored resume chunk embeddings to rank candidates by relevance

### SharePoint Sync Flow

- **Inbound:** resumes are fetched from a configured SharePoint document library via Microsoft Graph API and ingested into the pipeline
- **Outbound:** when a candidate's pipeline status is updated to `Shortlisted`, their record is automatically pushed/synced back to SharePoint

---

## Project Structure

```
project/
├── api/
│   ├── models.py          # Employee, Department, Role, Skill, Requirement, Resume, Interview
│   ├── serializers.py
│   ├── permissions.py
│   ├── screening.py        # Groq LLM screening logic + OCR fallback
│   ├── rag/
│   │   ├── chunking.py      # Section-based resume chunking
│   │   ├── embeddings.py    # Embedding generation
│   │   └── matching.py      # ChromaDB queries + cosine similarity ranking
│   ├── sharepoint/
│   │   ├── client.py        # Microsoft Graph API client
│   │   ├── fetch.py         # Resume fetch from SharePoint
│   │   └── sync.py          # Push shortlisted candidates back to SharePoint
│   ├── email_utils.py
│   ├── views/
│   │   ├── auth_views.py
│   │   ├── employee_views.py
│   │   ├── department_views.py
│   │   ├── requirement_views.py
│   │   ├── resume_views.py
│   │   ├── interview_views.py
│   │   └── role_skill_views.py
│   └── migrations/
├── project/
│   ├── settings.py
│   └── urls.py
├── docs/
│   └── screenshots/        # README images live here
└── templates/               # Dashboard, Jobs, Resumes, Login pages
    └── static/
        ├── css/
        └── js/
```

---

## API Endpoints

> Replace this section with your actual endpoint list/table — paste it here and I'll format it consistently with the rest of the doc.

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/auth/login/` | Obtain JWT access/refresh tokens |
| ... | ... | ... |

---

## Getting Started

### Prerequisites
- Python 3.11+
- pip
- (Optional) Tesseract OCR installed locally for scanned PDF support

### Installation
```bash
git clone <repo-url>
cd project
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### Environment Variables

Create a `.env` file in the project root:
```env
GROQ_API_KEY=your_groq_api_key
EMAIL_HOST_USER=your_mailtrap_username
EMAIL_HOST_PASSWORD=your_mailtrap_password

# SharePoint / Microsoft Graph
SHAREPOINT_CLIENT_ID=your_client_id
SHAREPOINT_CLIENT_SECRET=your_client_secret
SHAREPOINT_TENANT_ID=your_tenant_id
SHAREPOINT_SITE_ID=your_site_id
SHAREPOINT_DRIVE_ID=your_drive_id

# ChromaDB
CHROMA_PERSIST_DIR=./chroma_data
```

### Running Locally
```bash
python manage.py migrate
python manage.py runserver
```

API docs available at `http://localhost:8000/api/docs/`

---

## Note on Data Privacy

This project handles real company resume and candidate data, so it is **not publicly deployed**. All screenshots above use sanitized/dummy data. Reach out if you'd like a walkthrough or a sandboxed demo with seed data.
