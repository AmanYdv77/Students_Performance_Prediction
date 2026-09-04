# 🗄️ Database Design — Result Platform & Behavioral Telemetry

> Status: **BUILT & PRODUCTION READY** (Active SQLite backend + PostgreSQL schema compatible).  
> Companion to: `DUAL_ML_AND_HABIT_TELEMETRY.md`, `RESULT_PLATFORM_BLUEPRINT.md`.

---

## 1. Architectural Decisions

### Decision A: Relational Database with Integrated Telemetry
We use a relational schema. The university $ightarrow$ school $ightarrow$ department $ightarrow$ teacher $ightarrow$ student hierarchy, as well as real-time habit check-in streams, are modeled using normalized relationships with foreign keys.

### Decision B: Dual Partitioning (Personal, Academic & Behavioral)
1. **Personal Identity**: `User`, `StudentProfile`, `StaffProfile`.
2. **Academic Records**: `Subject`, `TeachingAssignment`, `Result`, `SemesterResult`.
3. **Behavioral Telemetry**: `StudentHabitPreference`, `HabitCheckInLog`.

### Decision C: Continuous Normalization Sync
Rather than recalculating metrics on raw logs during every web request, daily and weekly logs are stored in `HabitCheckInLog` and automatically aggregated into `SemesterResult` via `sync_habits_to_semester_result()`. This provides sub-millisecond ML feature lookups.

---

## 2. Entity Relationship Overview

```
[University]
   └── [School]
         └── [Department]
               ├── [StaffProfile] (HOD / Faculty)
               │      └── [TeachingAssignment] ──> [Subject]
               └── [Course]
                     └── [Batch]
                           └── [StudentProfile]
                                 ├── [Result] (Per-subject marks)
                                 ├── [SemesterResult] (SGPA & ML Behavioral Features)
                                 ├── [StudentHabitPreference] (Daily vs Weekly Cadence)
                                 └── [HabitCheckInLog] (Continuous Telemetry Stream)
```

---

## 3. Table Specifications

### 🏛️ Institution & Hierarchy Tables
| Table | Model | Purpose | Key Relationships |
| :--- | :--- | :--- | :--- |
| `university` | `University` | Top-level institution | Root node |
| `schools` | `School` | Faculties/Schools | `university_id`, `dean_user_id` |
| `departments` | `Department` | Academic departments | `school_id`, `hod_user_id` |
| `courses` | `Course` | Degree programs (UG/PG) | `department_id` |
| `batches` | `Batch` | Admission cohorts | `course_id`, `admission_year`, `study_year` |

### 👤 Identity & People Tables
| Table | Model | Purpose | Key Relationships |
| :--- | :--- | :--- | :--- |
| `users` | `User` | Authentication & Roles | `role` (`student`, `teacher`, `hod`, `dean`, `admin`) |
| `student_profiles` | `StudentProfile` | Roll numbers, current sem | `user_id`, `batch_id`, `course_id` |
| `staff_profiles` | `TeacherProfile` | Designation, employee ID | `user_id`, `department_id` |

### 📚 Academic Performance Tables
| Table | Model | Purpose | Key Relationships |
| :--- | :--- | :--- | :--- |
| `subjects` | `Subject` | Curriculum definitions | `course_id`, `semester`, `credits` |
| `teaching_assignments`| `TeachingAssignment` | Faculty-subject assignments | `teacher_id`, `subject_id`, `batch_id` |
| `results` | `Result` | Per-subject internal/external marks | `student_id`, `subject_id`, `teacher_id` |
| `semester_results` | `SemesterResult` | Official semester summary + **ML Behavioral Features** | `student_id`, `semester`, `sgpa`, `percentage` |

### ⚡ Behavioral Telemetry Tables (New)
| Table | Model | Purpose | Key Relationships |
| :--- | :--- | :--- | :--- |
| `academics_studenthabitpreference` | `StudentHabitPreference` | Stores logging cadence (`DAILY`/`WEEKLY`) & streaks | `student_id` (OneToOne) |
| `academics_habitcheckinlog` | `HabitCheckInLog` | Immutable event log of study & sleep check-ins | `student_id` (ForeignKey) |

---

## 4. Behavioral Fields in `SemesterResult`

To power the Dual-Model Machine Learning engine in real time, `SemesterResult` stores the normalized lifestyle vectors:

```python
class SemesterResult(models.Model):
    student = models.ForeignKey(StudentProfile, on_delete=models.CASCADE)
    semester = models.IntegerField()
    percentage = models.FloatField(default=0)
    sgpa = models.FloatField(default=0)
    
    # ML Dynamic/Behavioral Features
    attendance_percentage = models.FloatField(default=85.0)
    hours_studied_per_week = models.FloatField(default=0.0)
    sleep_hours_per_night = models.FloatField(default=7.0)
    motivation_level = models.CharField(max_length=20, blank=True, null=True)
    tutoring_sessions = models.IntegerField(default=0)
    extracurricular_activities = models.BooleanField(default=False)
    physical_activity = models.IntegerField(default=0)
```

---

## 5. Synchronization Function (`sync_habits_to_semester_result`)

Whenever a student submits a habit check-in, `sync_habits_to_semester_result(student)` is triggered:
1. Queries the student's recent logs within a 7-day rolling window.
2. If `WEEKLY` logs exist: Uses the latest weekly summary values directly.
3. If `DAILY` logs exist: Averages daily study hours and scales by 7 ($	ext{Avg Daily} 	imes 7$), and averages sleep duration.
4. Updates the student's active `SemesterResult` record.
5. Live ML predictions instantly reflect the new values on the next page refresh.
