# 📘 Student Performance Prediction — Code Line-by-Line Explanation Handbook

This handbook provides an exhaustive, line-by-line breakdown of every file, class, method, library, and tool used in this project. It is structured to help any developer understand exactly how the web platform and the machine learning system work under the hood.

---

## 🛠️ Third-Party Libraries & Inbuilt Tools Reference

Before diving into the code, here is an explanation of the key libraries and inbuilt tools utilized across the codebase:

1.  **Django ORM (Object-Relational Mapping)**: Allows us to interact with the database using Python classes (models) instead of writing raw SQL queries.
2.  **`django.db.transaction.atomic`**: A wrapper that ensures a block of database operations is treated as a single unit. If any operation fails, the database is rolled back. This speeds up bulk imports significantly in SQLite.
3.  **`bulk_create` / `bulk_update`**: Django ORM features that perform multi-row database insertions or updates in a single SQL query, avoiding thousands of round-trips to the database.
4.  **Scikit-Learn (sklearn)**:
    *   **`Pipeline`**: Combines preprocessing steps and model fitting into a single execution flow, ensuring consistency during both training and real-time dashboard predictions.
    *   **`ColumnTransformer`**: Allows different preprocessing steps to be applied to different columns (e.g. scaling numbers while one-hot encoding categories).
    *   **`StandardScaler`**: Scales numerical features to have a mean of 0 and a variance of 1, which standardizes numerical inputs.
    *   **`OneHotEncoder`**: Converts categorical text columns (e.g., 'OBC', 'General') into numerical binary arrays.
    *   **`RandomForestRegressor`**: An ensemble machine learning model that fits multiple decision trees on subsets of data to predict a continuous numerical value (percentage grade).
5.  **Pandas (`pd.DataFrame`)**: A data manipulation library used to format database outputs into structured grids (DataFrames) that the Scikit-learn pipeline expects.
6.  **Pickle (`pickle`)**: Built-in Python module used to serialize (save) and deserialize (load) the trained Scikit-learn model pipeline as a binary file (`.pkl`).

---

## 📂 Detailed File-by-File Code Breakdown

---

### 1. User Authentication System
#### File: [`app/accounts/models.py`](file:///e:/mini_projects/Student_Performance_Prediction/app/accounts/models.py)

This file overrides Django's default user model to introduce role-based access controls and scope variables.

*   **Lines 1-2**:
    ```python
    from django.contrib.auth.models import AbstractUser
    from django.db import models
    ```
    *   Imports `AbstractUser` to extend Django's built-in authentication system (inheriting password hashing, username validation, email, and admin logins).
    *   Imports Django's database `models` module to define schema fields.
*   **Lines 5-28**:
    ```python
    class User(AbstractUser):
        class Role(models.TextChoices):
            VC = "VC", "Vice Chancellor"
            REGISTRAR = "REGISTRAR", "Registrar"
            CONTROLLER = "CONTROLLER_OF_EXAMS", "Controller of Examinations"
            SYSADMIN = "SYSTEM_ADMIN", "System Administrator"
            DEAN = "DEAN", "Dean"
            HOD = "HOD", "Head of Department"
            TEACHER = "TEACHER", "Teacher"
            STUDENT = "STUDENT", "Student"
    ```
    *   Declares the custom `User` class.
    *   Uses `TextChoices` to declare an enumeration of roles. The left value is what is saved in the database (e.g. `"VC"`), and the right is the human-readable label (e.g. `"Vice Chancellor"`).
*   **Lines 30-47**:
    ```python
        role = models.CharField(
            max_length=30,
            choices=Role.choices,
            default=Role.STUDENT,
            help_text="Decides what this user is allowed to see.",
        )
        phone = models.CharField(max_length=15, blank=True)
        dob = models.CharField(max_length=20, blank=True)
        gender = models.CharField(max_length=10, blank=True)
        blood_group = models.CharField(max_length=5, blank=True)
        address_city = models.CharField(max_length=60, blank=True)
        address_state = models.CharField(max_length=60, blank=True)
        category = models.CharField(max_length=20, blank=True)
        status = models.CharField(max_length=20, default="active", blank=True)
    ```
    *   `role`: Sets the user's role, defaulting to Student.
    *   Adds demographic columns (gender, address_state, category) used directly as demographic features by our ML model.
*   **Lines 55-64**:
    ```python
        department = models.ForeignKey("academics.Department", on_delete=models.SET_NULL,
                                       null=True, blank=True, related_name="staff_members")
        school = models.ForeignKey("academics.School", on_delete=models.SET_NULL,
                                   null=True, blank=True, related_name="staff_members")

        def __str__(self):
            return f"{self.username} ({self.get_role_display()})"
    ```
    *   `department` and `school`: Foreign keys to academics models. This maps staff members to their corresponding academic departments to enforce scoped data query permissions.
    *   `__str__`: Returns a clean string formatting (e.g., `cse_fac01 (Teacher)`) in logs and admin screens.

---

### 2. Academic Infrastructure Models
#### File: [`app/academics/models.py`](file:///e:/mini_projects/Student_Performance_Prediction/app/academics/models.py)

Defines colleges, courses, batches, results, and features mapped from the Kaggle CSV.

*   **Lines 105-132: Student Profile & ML Static Features**:
    ```python
    class StudentProfile(models.Model):
        user = models.OneToOneField(settings.AUTH_USER_MODEL, on_delete=models.CASCADE,
                                    related_name="student_profile")
        roll_no = models.CharField(max_length=40, unique=True)
        batch = models.ForeignKey(Batch, on_delete=models.SET_NULL, null=True, blank=True)
        course = models.ForeignKey(Course, on_delete=models.SET_NULL, null=True, blank=True)
        current_semester = models.IntegerField(default=1)
        
        # Kaggle ML Static Features
        distance_from_home = models.CharField(max_length=20, blank=True, null=True)
        parental_education_level = models.CharField(max_length=50, blank=True, null=True)
        family_income = models.CharField(max_length=20, blank=True, null=True)
        internet_access = models.BooleanField(default=True)
        access_to_resources = models.CharField(max_length=20, blank=True, null=True)
        learning_disabilities = models.BooleanField(default=False)
    ```
    *   `user`: Links a user account directly to a single `StudentProfile` row in a 1-to-1 relationship.
    *   `distance_from_home` ... `learning_disabilities`: Captures the static demographics of each student, which are passed to the ML pipeline to gauge access constraints.
*   **Lines 184-210: Semester Cards & ML Dynamic Features**:
    ```python
    class SemesterResult(models.Model):
        student = models.ForeignKey(StudentProfile, on_delete=models.CASCADE,
                                    related_name="semester_results")
        semester = models.IntegerField()
        sgpa = models.FloatField(default=0)
        percentage = models.FloatField(default=0)

        # Kaggle ML Dynamic Features
        attendance_percentage = models.FloatField(default=0)
        hours_studied_per_week = models.FloatField(default=0)
        sleep_hours_per_night = models.FloatField(default=0)
        motivation_level = models.CharField(max_length=20, blank=True, null=True)
        tutoring_sessions = models.IntegerField(default=0)
        extracurricular_activities = models.BooleanField(default=False)
        physical_activity = models.IntegerField(default=0)
        parental_involvement = models.CharField(max_length=20, blank=True, null=True)
        peer_influence = models.CharField(max_length=20, blank=True, null=True)
    ```
    *   `student` & `semester`: Tracks performance indicators over time.
    *   `attendance_percentage` ... `peer_influence`: Stores the behavioral and lifestyle trends of the student during this specific semester block, allowing our ML model to predict grades dynamically.

---

### 3. Machine Learning Predictor Logic
#### File: [`app/academics/predictor.py`](file:///e:/mini_projects/Student_Performance_Prediction/app/academics/predictor.py)

Acts as the real-time inference layer, loading the trained model and predicting grades.

*   **Lines 5-8**:
    ```python
    MODEL_PATH = os.path.join(settings.BASE_DIR, '..', 'ml', 'student_predictor.pkl')
    _model = None
    ```
    *   Calculates the absolute path to the pickled model pipeline located outside the Django `app` folder.
*   **Lines 10-18**:
    ```python
    def get_model():
        global _model
        if _model is None:
            if os.path.exists(MODEL_PATH):
                with open(MODEL_PATH, 'rb') as f:
                    _model = pickle.load(f)
        return _model
    ```
    *   Implements a singleton caching pattern. It loads the model binary from the disk *only once* when the first prediction query is run, keeping memory footprints low.
*   **Lines 20-50: Predicting Current Semester Subjects**:
    ```python
    def predict_current_subjects(student):
        model = get_model()
        if not model:
            return []
            
        current_subjects = Subject.objects.filter(course=student.course, semester=student.current_semester)
        sem_res = SemesterResult.objects.filter(student=student, semester=student.current_semester).first()
        if not sem_res:
            sem_res = SemesterResult.objects.filter(student=student).order_by('-semester').first()
    ```
    *   Retrieves all classes the student is taking in their active semester.
    *   Attempts to fetch active semester behavioral records. If they don't exist yet, it falls back to the previous semester's metrics to make a realistic prediction.
*   **Lines 52-87: Formatting DataFrame & Executing Model**:
    ```python
        predictions = []
        user = student.user
        for subject in current_subjects:
            input_data = pd.DataFrame([{
                'distance_from_home': student.distance_from_home or 'Near',
                'parental_education_level': student.parental_education_level or 'College',
                'family_income': student.family_income or 'Medium',
                'internet_access': int(student.internet_access),
                'access_to_resources': student.access_to_resources or 'Medium',
                'learning_disabilities': int(student.learning_disabilities),
                'gender': user.gender or 'Male',
                'category': user.category or 'General',
                'address_state': user.address_state or 'Delhi',
                'subject_credits': subject.credits,
                'subject_type': subject.subject_type,
                'attendance_percentage': attendance,
                'hours_studied_per_week': hours_studied,
                'sleep_hours_per_night': sleep,
                'motivation_level': motivation,
                'tutoring_sessions': tutoring,
                'extracurricular_activities': int(extracurricular),
                'physical_activity': physical,
                'parental_involvement': parental,
                'peer_influence': peer
            }])
            
            predicted_pct = model.predict(input_data)[0]
            predicted_pct = max(0, min(100, predicted_pct))
    ```
    *   Constructs a single-row Pandas DataFrame containing exactly the features the Scikit-learn model was trained on.
    *   Executes `model.predict()` to calculate the score percentage, bounding it between `0` and `100` to prevent mathematical anomalies.
*   **Lines 88-120: Mapping Output to Letter Grades**:
    ```python
            if predicted_pct >= 90: letter_grade = 'O'
            elif predicted_pct >= 80: letter_grade = 'A+'
            ...
            status = 'PASS' if predicted_pct >= 50 else 'FAIL'
            predictions.append({
                'subject_code': subject.code,
                'predicted_percentage': round(predicted_pct, 1),
                'expected_grade': letter_grade,
                'is_at_risk': predicted_pct < 50
            })
        return predictions
    ```
    *   Maps the predicted percentage score into academic grade bands. If the predicted mark is less than 50%, it marks the student as **at-risk**.

---

### 4. Controller Views & Dashboard Permissions
#### File: [`app/accounts/views.py`](file:///e:/mini_projects/Student_Performance_Prediction/app/accounts/views.py)

Controls views and limits data queries to the active user's permissions scope.

*   **Lines 67-96: Hierarchical Access Scoping**:
    ```python
    def scoped_results_for(user):
        role = user.role
        if role in ("VC", "REGISTRAR", "CONTROLLER_OF_EXAMS", "SYSTEM_ADMIN"):
            return Result.objects.all(), "Entire University"
        if role == "DEAN":
            return Result.objects.filter(subject__course__department__school=user.school), f"School: {user.school}"
        if role == "HOD":
            return Result.objects.filter(subject__course__department=user.department), f"Department: {user.department}"
        if role == "TEACHER":
            return Result.objects.filter(teacher=user.teacher_profile), "Students you teach"
        return Result.objects.none(), "No access"
    ```
    *   **The Scoping Engine**: This single function filters result rows dynamically based on the user's role. A Teacher only queries results of subjects they teach, while the Vice Chancellor queries the entire database.
*   **Lines 171-197: Student Prediction Dashboard View**:
    ```python
    @login_required
    def my_predictions(request):
        student = getattr(request.user, "student_profile", None)
        predictions = predict_current_subjects(student)
        at_risk_count = sum(1 for p in predictions if p['is_at_risk'])
        avg_predicted = sum(p['predicted_percentage'] for p in predictions) / len(predictions) if predictions else 0
        
        behavior = SemesterResult.objects.filter(student=student, semester=student.current_semester).first()
        return render(request, "my_predictions.html", {
            "predictions": predictions,
            "at_risk_count": at_risk_count,
            "avg_predicted": round(avg_predicted, 1),
            "behavior": behavior
        })
    ```
    *   Uses Django's `login_required` decorator to deny access to anonymous users.
    *   Pulls predictions for the student's current classes and passes them to the template alongside the student's behavior variables to power lifestyle recommendations.
*   **Lines 200-230: Teacher Early Warning System View**:
    ```python
    @login_required
    def at_risk_students(request):
        results, scope_label = scoped_results_for(request.user)
        student_ids = results.values_list('student_id', flat=True).distinct()
        students = StudentProfile.objects.filter(id__in=student_ids).select_related('user', 'course')
        
        at_risk_list = []
        for student in students:
            predictions = predict_current_subjects(student)
            failing_subjects = [p for p in predictions if p['is_at_risk']]
            if failing_subjects:
                at_risk_list.append({
                    "student": student,
                    "failing_subjects": failing_subjects
                })
        return render(request, "at_risk_students.html", {
            "scope_label": scope_label,
            "at_risk_list": at_risk_list
        })
    ```
    *   Queries all students linked to results under the teacher's/administrator's scope.
    *   Iterates through them, running the ML model on each. If a student is predicted to fail any subject, they are appended to the warning list.

---

### 5. Seeding Behavior Metrics
#### File: [`scripts/seed_kaggle_features.py`](file:///e:/mini_projects/Student_Performance_Prediction/scripts/seed_kaggle_features.py)

*   **Lines 5-9: Django Setup outside of the App Framework**:
    ```python
    sys.path.append(os.path.join(os.path.dirname(os.path.dirname(os.path.abspath(__file__))), "app"))
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "resultplatform.settings")
    django.setup()
    ```
    *   Allows running the script directly from the terminal without executing `python manage.py`. It appends the Django application path to Python's system paths and initializes Django.
*   **Lines 38-52: Correlating Data using Gaussian Curves**:
    ```python
    for result in results:
        base_factor = result.sgpa / 10.0 if result.sgpa > 0 else 0.7
        attendance = max(40, min(100, int(random.gauss(50 + (base_factor * 40), 10))))
        result.attendance_percentage = attendance
    ```
    *   `base_factor`: Evaluates how well the student performed historically.
    *   `random.gauss(mean, standard_deviation)`: Creates a realistic bell-curve distribution. Higher SGPA values push the Gaussian mean higher, ensuring good students are seeded with logically higher attendance/study hours, preserving the statistical correlation needed for ML training.

---

### 6. Machine Learning Model Training
#### File: [`scripts/train_predictor.py`](file:///e:/mini_projects/Student_Performance_Prediction/scripts/train_predictor.py)

Queries SQLite, processes dataset features, and exports the Random Forest binary.

*   **Lines 19-24: ORM Query Performance Optimization**:
    ```python
    results = Result.objects.select_related('student', 'student__user', 'subject').all()
    semester_results = {
        (sr.student_id, sr.semester): sr 
        for sr in SemesterResult.objects.all()
    }
    ```
    *   `select_related`: Joins `Result` with `StudentProfile`, `User`, and `Subject` tables in a single SQL query, avoiding the N+1 database querying bottleneck.
    *   `semester_results` Dictionary: Maps all behavioral metrics in memory. Looking up a student's metrics by `(student_id, semester)` becomes an O(1) dictionary check instead of triggering thousands of slow database queries inside the loop.
*   **Lines 75-84: Defining Column Preprocessor**:
    ```python
    preprocessor = ColumnTransformer(
        transformers=[
            ('num', StandardScaler(), numeric_cols + boolean_cols),
            ('cat', OneHotEncoder(handle_unknown='ignore'), categorical_cols)
        ]
    )
    ```
    *   Uses `ColumnTransformer` to split features:
        *   `StandardScaler` standardizes numerical study inputs to have a mean of 0 and unit variance.
        *   `OneHotEncoder` converts text options into vectors, skipping unknown categories during prediction time (`handle_unknown='ignore'`).
*   **Lines 86-93: Model Fitting & Pipeline Export**:
    ```python
    model = Pipeline(steps=[
        ('preprocessor', preprocessor),
        ('regressor', RandomForestRegressor(n_estimators=100, random_state=42, n_jobs=-1))
    ])
    model.fit(X_train, y_train)
    ```
    *   Creates a unified model pipeline that applies the preprocessor before fitting the Random Forest Regressor.
    *   `n_jobs=-1`: Instructs the CPU to use all available cores to train the trees in parallel.
---

## ⚡ Behavioral Telemetry & Dual-Model Integration (New Code Breakdown)

### 1. `app/academics/models.py` — Habit Telemetry Models

#### `StudentHabitPreference`
```python
class StudentHabitPreference(models.Model):
    FREQUENCY_CHOICES = [
        ("DAILY", "Daily Quick-Check (30 sec)"),
        ("WEEKLY", "Weekly Summary (2 min)"),
    ]
    student = models.OneToOneField(StudentProfile, on_delete=models.CASCADE, related_name="habit_preference")
    frequency = models.CharField(max_length=10, choices=FREQUENCY_CHOICES, default="DAILY")
    streak_count = models.IntegerField(default=0)
    last_checkin_date = models.DateField(null=True, blank=True)
```
*   **`student`**: One-to-one relationship ensuring each student has exactly one configuration profile.
*   **`frequency`**: Stores whether the student prefers logging daily or weekly.
*   **`streak_count`**: Incremented by 1 when check-ins happen on consecutive days/weeks.

#### `HabitCheckInLog`
```python
class HabitCheckInLog(models.Model):
    student = models.ForeignKey(StudentProfile, on_delete=models.CASCADE, related_name="habit_logs")
    log_type = models.CharField(max_length=10, choices=LOG_TYPES, default="DAILY")
    log_date = models.DateField(default=timezone.now)
    hours_studied = models.FloatField(default=0.0)
    sleep_hours = models.FloatField(default=7.0)
    motivation_level = models.CharField(max_length=10, choices=MOTIVATION_CHOICES, default="Medium")
    tutoring_sessions = models.IntegerField(default=0)
    physical_activity = models.IntegerField(default=0)
    notes = models.TextField(blank=True, null=True)
```
*   **`log_date`**: Automatically stamped with current date.
*   **`hours_studied` & `sleep_hours`**: Captures raw student lifestyle data.

#### `sync_habits_to_semester_result(student)`
```python
def sync_habits_to_semester_result(student):
    current_sem = student.current_semester
    sem_result, _ = SemesterResult.objects.get_or_create(
        student=student, semester=current_sem,
        defaults={"percentage": 0, "sgpa": 0, "attendance_percentage": 85.0}
    )
    today = timezone.now().date()
    recent_logs = student.habit_logs.filter(log_date__gte=today - timedelta(days=7))
    ...
```
*   **Line-by-line**:
    1. Retrieves the active `SemesterResult` object for the student's current semester.
    2. Filters the student's logs for the past 7 days.
    3. If weekly logs are present, applies them directly to `hours_studied_per_week` and `sleep_hours_per_night`.
    4. If daily logs are present, averages daily study and multiplies by 7 to compute normalized weekly volume.
    5. Saves `sem_result`, making live telemetry immediately available to `predict_current_subjects()`.

---

## 🛡️ CSRF Synchronization & Session Security

### `app/templates/base.html` — Client-Side CSRF Auto-Synchronizer
```javascript
(function() {
    function getActiveCsrfCookie() {
        const name = 'csrftoken=';
        const cookies = document.cookie.split(';');
        for (let i = 0; i < cookies.length; i++) {
            let c = cookies[i].trim();
            if (c.indexOf(name) === 0) {
                return decodeURIComponent(c.substring(name.length, c.length));
            }
        }
        return null;
    }

    function syncCsrfInputs() {
        const activeToken = getActiveCsrfCookie();
        if (activeToken) {
            document.querySelectorAll('input[name="csrfmiddlewaretoken"]').forEach(function(input) {
                input.value = activeToken;
            });
        }
    }

    document.addEventListener('DOMContentLoaded', syncCsrfInputs);
    window.addEventListener('pageshow', syncCsrfInputs);
    document.addEventListener('submit', function(event) {
        const activeToken = getActiveCsrfCookie();
        if (activeToken && event.target && event.target.tagName === 'FORM') {
            let input = event.target.querySelector('input[name="csrfmiddlewaretoken"]');
            if (input) input.value = activeToken;
        }
    }, true);
})();
```
*   **`getActiveCsrfCookie`**: Reads the active `csrftoken` cookie directly from `document.cookie`.
*   **`pageshow` listener**: Automatically fires when the user clicks the browser Back or Forward button, preventing stale tokens restored from the browser's Back-Forward Cache (bfcache).
*   **`submit` listener (capture phase)**: Ensures that right before a form is dispatched to Django, the hidden `csrfmiddlewaretoken` matches the browser's active cookie. This completely eliminates `403 CSRF token from POST incorrect` errors.
