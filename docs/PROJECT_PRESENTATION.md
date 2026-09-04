# 🎓 Project Presentation: Student Performance Prediction Platform

> **Comprehensive Academic Presentation Deck**  
> Compatible with Markdown slide renderers (Marp, Pandoc, reveal.js)

---

## 💻 Slide 1: Title & Overview

### **EduPulse: Dual-Model Academic Telemetry & Performance Prediction**
*   **Sub-title**: Bridging Relational University ERPs with Dynamic Lifestyle Telemetry for Early Intervention
*   **Architecture**: Django 6.0 + Scikit-Learn Dual ML Pipeline + Stitch Monolith Dark UI
*   **Key Innovation**: Continuous Behavioral Tracking (Daily vs. Weekly Cadence) with Real-Time Risk Radar

---

## 💡 Slide 2: The Core Problem & Innovation

### **The Academic Dilemma**
*   **Traditional SIS Systems**: Pure post-mortem tools. Grades are recorded after final exams when student failure cannot be reversed.
*   **Academic ML Research**: Typically trained on isolated static CSVs in notebooks without active connection to living university databases.

### **Our Solution: The Dual-Model Architecture**
*   **Model 1 (Academic Prior)**: Establishes a baseline capability trajectory using historical semester exams and internal test scores.
*   **Model 2 (Behavioral Multiplier)**: Dynamically scales expected outcomes using real-time study volume, sleep duration, and motivation indices.

---

## ⚡ Slide 3: Telemetry Collection & Normalization

### **Solving Student Logging Fatigue**
*   **Daily Quick-Check (30 sec)**: Captures day-to-day study hours and sleep duration.
*   **Weekly Summary (2 min)**: Captures aggregated weekly study volume for busy schedules.

### **The Mathematical Bridge**
$$	ext{Normalized Weekly Study} = \left(rac{1}{N}\sum_{i=1}^N 	ext{Daily Study}_iight) 	imes 7$$
*   Automatically aggregated by `sync_habits_to_semester_result()`.
*   Maintains consistent feature inputs for Scikit-Learn without user friction.

---

## 🎯 Slide 4: Faculty Early Warning Radar

### **Closing the Feedback Loop**
*   **Real-time Risk Classification**: Identifies students with forecasted subject marks $< 40\%$.
*   **Deficit Attribution**: Flags whether failure is caused by academic fundamentals or behavioral deficits (e.g. study volume $< 8$ hrs/week).
*   **Intervention Workflow**: Faculty can log mentoring sessions and assign targeted revision sets before semester exams occur.

---

## 🎨 Slide 5: Stitch Monolith Design System

### **Visual Excellence: Strict Two-Color Palette**
*   **Canvas & Surfaces**: Deep Charcoal & Dark Grey (`#090a0c`, `#14161c`, `#20242e`).
*   **Text & Accents**: Crisp High-Contrast White (`#ffffff`).
*   **Typography**: Geist / Inter + JetBrains Mono for data readouts.
*   **Monochromatic Risk Badges**: High-contrast white cards for critical risk warnings.
*   **Zero Session Glitches**: Built-in client-side CSRF auto-synchronizer for seamless back/forward browser navigation.

---

## 📊 Slide 6: Verified Demo Credentials

*   **Student Account**: `25-engg-cse-ug-001` / `password123` (Interactive Habit Logging & AI Forecast)
*   **Faculty Account**: `cse_fac01` / `password123` (At-Risk Early Warning Radar & Dept Ledger)
*   **Admin Account**: `Aman_Yadav` / `password123` (University-wide administration)
