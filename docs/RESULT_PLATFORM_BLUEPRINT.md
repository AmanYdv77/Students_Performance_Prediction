# 🎓 College Result Management + Prediction Platform — Blueprint v2.0

> Status: **PRODUCTION READY — DUAL-MODEL ML & MONOLITH THEME ACTIVE**  
> Companion documents: `DUAL_ML_AND_HABIT_TELEMETRY.md`, `DATABASE_DESIGN.md`, `CODE_EXPLANATION_HANDBOOK.md`.

---

## 1. Vision & Architecture

EduPulse is an institutional academic telemetry and predictive modeling platform designed to replace legacy, static grade repositories. It merges relational university record-keeping with real-time behavioral machine learning to alert faculty and students before academic failure occurs.

### Key Pillars:
1. **Hierarchical Academic ERP**: Scoped access for Students, Teachers, HODs, Deans, and Admins.
2. **Dual-Model ML Engine**:
   - Model 1: Historical Academic Baseline (Ridge Regression).
   - Model 2: Dynamic Behavioral Multiplier (Study Volume, Sleep, Motivation, Tutoring).
3. **Flexible Habit Collection**: Cadence switcher (Daily 30-sec Check-in vs Weekly 2-min Summary).
4. **Faculty Early Warning Radar**: Automatic detection of students with subject scores $< 40\%$.
5. **Stitch-Inspired "Monolith Academic" UI**: Strict two-color palette (Dark Grey + Crisp White) with zero extraneous visual clutter.
6. **Automatic CSRF Token Synchronization**: Client-side synchronizer preventing bfcache and rotation mismatches.

---

## 2. System Status Matrix

| Component | Status | Implementation Details |
| :--- | :--- | :--- |
| **Authentication & RBAC** | ✅ Complete | Custom `User` model with roles: `student`, `teacher`, `hod`, `dean`, `admin`. |
| **Academic Ledger** | ✅ Complete | Multi-semester grade cards, credit calculations, SGPA aggregation. |
| **Habit Tracking Telemetry**| ✅ Complete | `StudentHabitPreference`, `HabitCheckInLog`, streak counter. |
| **Dual-Model ML Sync** | ✅ Complete | `sync_habits_to_semester_result`, real-time subject risk matrices. |
| **Faculty Risk Radar** | ✅ Complete | Department-scoped early warning table with intervention triggers. |
| **Monolith Frontend** | ✅ Complete | Dark grey (`#090a0c`) + crisp white (`#ffffff`) across all 8 views. |
| **CSRF & Session Security**| ✅ Complete | Auto-sync on form submit & `pageshow`, custom `403_csrf.html`. |

---

## 3. User Flows & Personas

### 3.1 Student Persona
1. **Authentication**: Enters student roll number (e.g. `25-engg-cse-ug-001`).
2. **Command Dashboard**: Displays forecasted semester performance %, attendance, weekly study volume, and telemetry streak.
3. **Habit Telemetry (`/habits/check-in/`)**:
   - Selects preferred cadence (Daily vs Weekly).
   - Adjusts study hours and sleep sliders with live readout.
   - Records motivation index and self-reflections.
4. **AI Diagnostics (`/my-predictions/`)**:
   - Inspects forecasted subject scores.
   - Reviews prescriptive study recommendations (e.g. "+4 hrs/wk study yields +6.4% score increase").
5. **Transcripts (`/my-results/`)**:
   - Views official semester grade sheets and SGPA breakdown.

### 3.2 Faculty Persona
1. **Authentication**: Enters faculty ID (e.g. `cse_fac01`).
2. **Executive Overview**: Shows cohort size, at-risk count, and departmental averages.
3. **Early Warning Radar (`/at-risk/`)**:
   - Scans students projected to score $< 40\%$ in assigned subjects.
   - Diagnoses root causes (study deficit vs attendance shortfall).
   - Logs interventions directly.
4. **Departmental Records (`/results-overview/`)**:
   - Master ledger of all students and course grades in assigned department.

---

## 4. Technology Stack

* **Backend Framework**: Django 6.0.6 (Python 3.12)
* **Database**: SQLite (Development / Analytical sandbox) $\rightarrow$ PostgreSQL (Production target)
* **ML Inference**: Scikit-Learn (Ridge Regression Pipeline + Behavioral Gradient Multiplier)
* **Styling**: Stitch MCP "Monolith Academic" Design System (Pure Vanilla CSS, no heavy dependencies)
* **Typography**: Geist / Inter (Body & Headers) + JetBrains Mono (Data & Telemetry)
