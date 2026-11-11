# Group B — E-tivity 5: Practical Feasibility Analysis

**Course:** SWE2106 — Internet Technologies and Web Design  
**Team:** Group B (11 members)  
**Deadline:** 12 November 2026  
**Repo:** https://github.com/mugmarp/Group_B_web_trial

---

## 1. The Assignment (Recap)

> Choose a registrar process → propose digital solution → build Express API → Postman tests → connect DB + frontend → team presentation

**Registrar processes suggested:** examination results, applications, student records, class attendance, transcripts, course registration, graduation clearance.

---

## 2. Our Constraints (No Assumptions)

| Constraint | Reality |
|------------|---------|
| **API access to MUST systems** | ❌ None. SIMS, AIMS, Timetable system — no public APIs |
| **Student data access** | ❌ Registrar won't give us live data. We synthesize. |
| **Lecturer/examiner buy-in** | ❓ Unknown. Need to ask. |
| **Server hosting** | ❓ MUST ICT may give test server. Fallback: local/ngrok. |
| **Team size** | 11 members. Only ~4-5 can code backend/frontend. |
| **Time** | ~5 weeks (Nov 10 → Dec 12 presentation). |
| **Skill level** | Mixed. Most new to Express, Git, PRs, Postman. |

---

## 3. Three Options — Ranked by Feasibility

### Option A: Student Attendance Management System (SAMS)

| Aspect | Assessment |
|--------|------------|
| **Public data available** | ✅ Timetable: 1,304 sessions from `timetable.must.ac.ug` + mylectures-2 JSON export |
| **Student data** | ❌ Must synthesize (CSV with regNo, name, program, year) |
| **Core rule** | ✅ Clear: 80% attendance = exam eligibility (MUST Regulations §2.3) |
| **Daily users** | Examiners (must use every class) — **high adoption risk** |
| **Technical complexity** | High: offline-first, QR check-in, sync, eligibility calc |
| **Demo value** | High: live QR scan → instant eligibility list |

**Verdict:** **Doable ONLY if** 3+ examiners commit to pilot + Registrar gives student CSV.

---

### Option B: Transcript Request System

| Aspect | Assessment |
|--------|------------|
| **Public data available** | ❌ No public transcript data. But: simple domain model. |
| **Student data** | ❌ Synthesize same as SAMS. |
| **Core rule** | ✅ Simple: student requests → admin approves → PDF generated |
| **Daily users** | Admin staff only (low volume) — **low adoption risk** |
| **Technical complexity** | Low: CRUD + PDF generation + status tracking |
| **Demo value** | Medium: request → approve → download PDF |

**Verdict:** **Most feasible.** Simple domain, no external dependencies, clear workflow.

---

### Option C: Course Registration Portal

| Aspect | Assessment |
|--------|------------|
| **Public data available** | ❌ SIMS has it, no API. Timetable shows courses but not capacity/enrollment. |
| **Student data** | ❌ Synthesize. |
| **Core rule** | ⚠️ Complex: prerequisites, capacity, conflicts, fees, approval workflows. |
| **Daily users** | Students (self-serve) — **medium adoption risk** |
| **Technical complexity** | High: conflict detection, waitlists, prerequisites, transactions. |
| **Demo value** | High: student logs in → picks courses → sees conflicts. |

**Verdict:** **Too complex** for 5 weeks with no real data.

---

## 4. Recommended Choice: **Transcript Request System** (with Attendance as Stretch)

### Why Transcript Request?

| Reason | Detail |
|--------|--------|
| **Simplest domain** | Student → Request → Admin → PDF. No daily users. |
| **No external deps** | No timetable sync, no QR, no offline sync. |
| **Clear MVP** | 3 endpoints, 1 PDF template, 2 roles. |
| **Fits team skills** | 4-5 coders can split: API, DB, PDF, frontend, tests. |
| **Regulatory fit** | Registrar process listed in assignment. |
| **Data synthesis trivial** | 50 students, 5 programs, 3 request types. |

### Stretch: Attendance Dashboard (Read-Only)
- Import mylectures-2 timetable JSON
- Show "what attendance tracking would look like"
- No check-in, no examiners, no eligibility calc
- Pure visualization — zero adoption risk

---

## 5. Transcript Request — MVP Scope

### 5.1 Core Entities

```prisma
model Student {
  id        String   @id @default(cuid())
  regNo     String   @unique  // 2025/BSE/108/PS
  name      String
  email     String   @unique
  program   String   // BSE, BCS, BBA, etc.
  year      Int
  createdAt DateTime @default(now())
}

model TranscriptRequest {
  id          String   @id @default(cuid())
  studentId   String
  student     Student  @relation(fields: [studentId], references: [id])
  type        RequestType  // FULL, PARTIAL, CERTIFIED
  status      RequestStatus @default(PENDING)
  purpose     String?      // employment, further studies, etc.
  adminNotes  String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  processedAt DateTime?
}

enum RequestType {
  FULL        // Complete transcript
  PARTIAL     // Specific semesters
  CERTIFIED   // Signed/stamped copy
}

enum RequestStatus {
  PENDING
  APPROVED
  REJECTED
  READY_FOR_PICKUP
}
```

### 5.2 API Endpoints (7 Total)

| Method | Endpoint | Role | Description |
|--------|----------|------|-------------|
| POST | `/api/auth/login` | All | JWT login (student/admin) |
| GET | `/api/student/requests` | Student | List own requests |
| POST | `/api/student/requests` | Student | Create new request |
| GET | `/api/admin/requests` | Admin | List all (filter by status) |
| PATCH | `/api/admin/requests/:id` | Admin | Update status + notes |
| GET | `/api/admin/requests/:id/pdf` | Admin | Generate transcript PDF |
| GET | `/api/health` | Public | Health check |

### 5.3 Postman Collection

- Auth flow (login → token)
- Student: create request, list requests
- Admin: list all, approve/reject, generate PDF
- Negative tests: invalid token, wrong role, missing fields

---

## 6. Data Strategy — No APIs, No Problem

### 6.1 Timetable (Real Public Data) — For Stretch Dashboard Only
```bash
# Source: https://github.com/mugmarp/mylectures-2/raw/main/app/src/main/assets/timetable_export.json
# 1,304 sessions, all programs, rooms, lecturers
# Use: seed script → PostgreSQL → read-only dashboard
```

### 6.2 Students (Synthesized) — For Both Systems
```csv
# students.csv — 50 records across programs
regNo,name,email,program,year
2025/BSE/001/PS,Alice Nakato,alice@student.must.ac.ug,BSE,2
2025/BCS/002/PS,Bob Kato,bob@student.must.ac.ug,BCS,3
2025/BBA/003/PS,Carol Amayo,carol@student.must.ac.ug,BBA,1
...
```

### 6.3 Enrollments (Synthesized) — For Stretch Only
```csv
# enrollments.csv — student ↔ course
regNo,courseCode
2025/BSE/001/PS,SWE2106
2025/BSE/001/PS,BCS2101
...
```

**All CSV files committed to repo** — no external dependency at runtime.

---

## 7. Technical Stack (Minimal, Standard)

| Layer | Choice | Why |
|-------|--------|-----|
| **Runtime** | Node 20 + Express | Team knows JS; assignment requires Express |
| **Language** | TypeScript | Type safety for API contracts |
| **Database** | PostgreSQL + Prisma | Relational; Prisma = type-safe migrations |
| **Auth** | JWT (access + refresh) | Simple, stateless, standard |
| **PDF** | `pdfkit` or `@react-pdf/renderer` | Generate transcript PDFs |
| **Frontend** | React + Vite + Tailwind | Fast dev; PWA optional |
| **Testing** | Jest (unit) + Postman (integration) | Assignment requires Postman |
| **CI** | GitHub Actions | Lint + test on every PR |
| **Deploy** | Docker → MUST test server OR Render free tier | Portable |

---

## 8. Team Workstreams (5 Weeks)

| Week | Backend (2-3) | Frontend (2) | QA/Docs (1-2) | DevOps (1) |
|------|---------------|--------------|---------------|------------|
| **1** | Prisma schema, auth, `/health` | Login page, repo setup | Postman collection skeleton | GitHub Actions CI |
| **2** | Student CRUD, request CRUD | Student dashboard, create request | Unit tests, API docs | Dockerfile, staging deploy |
| **3** | Admin endpoints, PDF generation | Admin panel, request list | Integration tests, negative cases | DB migrations, secrets |
| **4** | Polish, edge cases, seed data | Polish UI, responsive | Full Postman collection, test report | Production deploy |
| **5** | **Buffer** | **Buffer** | **Demo rehearsal** | **Demo rehearsal** |

---

## 9. Risk Register — Honest Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| **Team members can't code** | High | High | Pair programming; non-coders do docs, tests, CSV data, Postman |
| **Git/PR workflow fails** | Medium | High | Week 1: mandatory PR practice session (see `git-pr-guide.md`) |
| **PDF generation bugs** | Medium | Medium | Use `@react-pdf/renderer` (React-based, easier debugging) |
| **No test server from MUST ICT** | Medium | Low | Deploy to Render free tier (public HTTPS) |
| **Scope creep** | High | High | **Freeze scope Nov 17** — only MVP endpoints |
| **Data synthesis looks fake** | Low | Low | Use realistic MUST regNo format, program codes |

---

## 10. Decision Matrix — Final

| Criterion | Transcript Request | SAMS | Course Registration |
|-----------|-------------------|------|-------------------|
| **Feasibility (1-5)** | 5 | 2 | 2 |
| **Meets assignment** | ✅ | ✅ | ✅ |
| **Team can deliver** | ✅ | ⚠️ | ❌ |
| **No external deps** | ✅ | ❌ | ❌ |
| **Demo in 5 weeks** | ✅ | ⚠️ | ❌ |
| **Stretch potential** | Attendance dashboard | — | — |

**Decision: Transcript Request System + Attendance Dashboard (stretch)**

---

## 11. Next Actions (This Week)

1. **Team vote** — confirm Transcript Request as primary
2. **Create repo structure** — `backend/`, `frontend/`, `docs/`, `postman/`
3. **Synthesize `students.csv`** — 50 realistic records
4. **Week 1 sprint planning** — assign workstreams
5. **PR practice session** — 30 min, everyone creates dummy PR

---

## Appendix: Files in This Repo

```
Group_B_web_trial/
├── README.md                    # This file
├── docs/
│   ├── feasibility-analysis.md  # This analysis
│   ├── api-spec.md              # OpenAPI spec (to create)
│   └── data-dictionary.md       # Entity definitions
├── backend/                     # Express + Prisma (to create)
├── frontend/                    # React + Vite (to create)
├── postman/                     # Collection + environments (to create)
├── data/
│   ├── students.csv             # 50 synthesized students
│   ├── enrollments.csv          # For attendance stretch
│   └── timetable_export.json    # Real data from mylectures-2
└── docker-compose.yml           # Local dev stack
```

---

*Version 1.0 | 2026-11-10 | Group B — no assumptions, real constraints only*