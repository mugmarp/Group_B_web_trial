# Data Dictionary — Group B E-tivity 5

## Core Entities (Transcript Request System)

### Student
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID (cuid) | PK, unique | Internal identifier |
| `regNo` | String | Unique, not null | MUST registration number (format: `2025/BSE/001/PS`) |
| `name` | String | Not null | Full name |
| `email` | String | Unique, not null | Student email (`@student.must.ac.ug`) |
| `program` | String | Not null | Program code (BSE, BCS, BBA, BIT, BPCD, BSAF, ECO, PEM, MIE, DLT, HEAP) |
| `year` | Integer | Not null | Year of study (1-5) |
| `createdAt` | DateTime | Default now | Record creation timestamp |

### TranscriptRequest
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID (cuid) | PK, unique | Internal identifier |
| `studentId` | UUID | FK → Student.id, not null | Owning student |
| `type` | Enum | Not null | `FULL`, `PARTIAL`, `CERTIFIED` |
| `status` | Enum | Default `PENDING` | `PENDING`, `APPROVED`, `REJECTED`, `READY_FOR_PICKUP` |
| `purpose` | String | Nullable | Reason: employment, further studies, visa, etc. |
| `adminNotes` | String | Nullable | Internal notes from registrar staff |
| `createdAt` | DateTime | Default now | Request submission timestamp |
| `updatedAt` | DateTime | Auto-update | Last modification timestamp |
| `processedAt` | DateTime | Nullable | When admin took action |

### Enums

#### RequestType
| Value | Description |
|-------|-------------|
| `FULL` | Complete transcript all semesters |
| `PARTIAL` | Specific semesters/years only |
| `CERTIFIED` | Signed/stamped official copy |

#### RequestStatus
| Value | Description | Next Valid States |
|-------|-------------|-------------------|
| `PENDING` | Submitted, awaiting review | `APPROVED`, `REJECTED` |
| `APPROVED` | Approved, PDF generated | `READY_FOR_PICKUP` |
| `REJECTED` | Denied with reason | (terminal) |
| `READY_FOR_PICKUP` | PDF ready for collection | (terminal) |

---

## Stretch Entities (Attendance Dashboard — Read-Only)

### Course
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | PK | |
| `code` | String | Unique | e.g., `SWE2106` |
| `title` | String | | e.g., `Internet Technologies and Web Design` |
| `creditUnits` | Integer | | |
| `program` | String | | BSE, BCS, etc. |
| `year` | Integer | | 1-5 |
| `semester` | Integer | | 1 or 2 |

### Session (from timetable_export.json)
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | PK | |
| `naturalKey` | String | Unique | Composite: `program_group\|course_code\|day\|start_time` |
| `courseCode` | String | FK → Course.code | |
| `programGroup` | String | | e.g., `BSE II` |
| `day` | String | | Monday-Sunday |
| `startTime` | String | HH:mm | |
| `endTime` | String | HH:mm | |
| `sessionType` | String | | Lecture, Lab, Tutorial, Practical |
| `lecturer` | String | Nullable | Lecturer name |
| `room` | String | Nullable | Room name |
| `sharedWith` | String[] | | Other programs sharing session |
| `floor` | Integer | Nullable | Resolved floor (0-4) |

### Attendance (Synthesized for Demo)
| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | PK | |
| `studentId` | UUID | FK → Student.id | |
| `sessionId` | UUID | FK → Session.id | |
| `status` | Enum | | `PRESENT`, `ABSENT`, `LATE`, `EXCUSED` |
| `method` | Enum | | `QR`, `MANUAL`, `GEOFENCE` |
| `recordedAt` | DateTime | | |
| `syncedAt` | DateTime | Nullable | For offline-first demo |

---

## CSV Data Files (Committed to Repo)

### `data/students.csv`
- 50 records across 11 programs
- Realistic MUST registration number format
- `@student.must.ac.ug` emails
- Years 1-4 distribution

### `data/enrollments.csv` (for attendance stretch)
- Student ↔ CourseCode mappings
- ~3-5 courses per student

### `data/timetable_export.json`
- 1,304 real sessions from `timetable.must.ac.ug`
- Source: mylectures-2 export
- Semester I 2026/2027 final timetable

### `data/timetable_metadata.json`
- Version info: `Final Timetable`, `Official Final`, 2026/2027 Semester 1
- Total sessions: 1,304
- Generated: 2026-09-14

---

## Seed Scripts (To Implement)

```typescript
// prisma/seed.ts
// 1. Load students.csv → Student table
// 2. Load timetable_export.json → Session table (with floor resolution from mylectures-2 logic)
// 3. Load enrollments.csv → Enrollment table (stretch)
// 4. Create 2 admin users for transcript system
```

---

*Version 1.0 | 2026-11-10*