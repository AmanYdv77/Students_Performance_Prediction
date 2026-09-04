# 🧩 Django Cheat-Sheet (for the Result Platform)

> Keep this open while doing the Django Girls / official tutorials.
> Companion to DJANGO_GUIDE.html. Plain, one-line explanations.

---

## 🧠 The 7 words to remember

| Word | One-line meaning | Restaurant analogy |
|---|---|---|
| **Project** | The whole website | The building |
| **App** | One feature-module inside the project | A room/department |
| **Model** | A database table written as a Python class | A filing cabinet |
| **Migration** | Turns your models into real database tables | Blueprint → construction |
| **View** | Python function that decides what to show | The chef |
| **URL** | Maps a web address to a view | Table of contents |
| **Template** | The HTML page the user sees | The plated dish |

**The flow:** `User → URL → View → Model → Template → User`

---

## ⌨️ Commands you'll use constantly

```bash
# --- Setup (once) ---
pip install django                       # install Django
django-admin startproject mysite         # create the project
cd mysite
python manage.py startapp results        # create an app

# --- Run the website (anytime) ---
python manage.py runserver               # then open http://127.0.0.1:8000

# --- After changing models (very often) ---
python manage.py makemigrations          # write the change plan
python manage.py migrate                 # apply it to the database

# --- Admin panel access (once) ---
python manage.py createsuperuser         # make an admin login
# then visit http://127.0.0.1:8000/admin

# --- Handy ---
python manage.py shell                   # play with data in Python
```

> 💡 99% of Django commands start with `python manage.py ...` — your remote control.

---

## 🗄️ Models — the table fields you'll need

```python
from django.db import models

class Result(models.Model):
    student      = models.ForeignKey("Student", on_delete=models.CASCADE)
    teacher      = models.ForeignKey("Teacher", on_delete=models.SET_NULL, null=True)
    semester     = models.IntegerField()
    total_marks  = models.IntegerField()
    letter_grade = models.CharField(max_length=2)
    exam_date    = models.DateField(null=True, blank=True)
```

### Common field types
| Field | Use for | Example |
|---|---|---|
| `CharField(max_length=…)` | short text | name, grade, code |
| `TextField()` | long text | remarks, address |
| `IntegerField()` | whole numbers | marks, semester |
| `FloatField()` | decimals | SGPA, percentage |
| `BooleanField()` | yes/no | is_active |
| `DateField()` / `DateTimeField()` | dates/times | exam_date, created_at |
| `EmailField()` | emails | email |
| `ForeignKey(X)` | **link to another table** | student, subject, teacher |

### Relationship rules (the important one)
| Code | Meaning |
|---|---|
| `ForeignKey` | "many of these belong to one X" (many results → one student) |
| `on_delete=models.CASCADE` | if the X is deleted, delete these too |
| `on_delete=models.SET_NULL, null=True` | if the X is deleted, leave this blank |
| `OneToOneField` | exactly one-to-one (e.g. user ↔ profile) |
| `ManyToManyField` | many-to-many (e.g. teacher ↔ many subjects) |

---

## 🔍 ORM — querying data (your analytics!)

```python
# Get things
Result.objects.all()                          # everything
Result.objects.get(id=5)                       # one specific row
Result.objects.filter(semester=3)              # all matching
Result.objects.filter(student=asha)            # Asha's results
Result.objects.exclude(letter_grade="F")       # all except F

# Sort & limit
Result.objects.order_by("total_marks")         # low → high
Result.objects.order_by("-total_marks")        # high → low (the "-")
Result.objects.order_by("-total_marks")[:3]    # top 3 (toppers!)

# Follow relationships (double underscore __)
Result.objects.filter(student__department=cse) # results in CSE dept
Result.objects.filter(teacher__user=prof)      # results taught by a teacher

# Math / aggregates (averages, counts)
from django.db.models import Avg, Count, Max, Min, Sum
Result.objects.aggregate(Avg("total_marks"))   # overall average
Result.objects.values("subject").annotate(avg=Avg("total_marks"))  # avg per subject
Result.objects.count()                          # how many rows
```

> 🔑 The `__` (double underscore) = "reach into a linked table." This powers
> your hierarchy filtering and teacher-performance queries.

---

## 🛂 Your hierarchy (scope) in a View

```python
def my_results(request):
    user = request.user
    if user.role == "STUDENT":
        data = Result.objects.filter(student__user=user)            # own only
    elif user.role == "TEACHER":
        data = Result.objects.filter(teacher__user=user)            # their classes
    elif user.role == "HOD":
        data = Result.objects.filter(student__department=user.department)
    elif user.role == "DEAN":
        data = Result.objects.filter(student__school=user.school)
    elif user.role in ("VC", "REGISTRAR", "CONTROLLER_OF_EXAMS"):
        data = Result.objects.all()                                  # everything
    return render(request, "results.html", {"data": data})
```

---

## 🗺️ URLs — connecting addresses to views

```python
# in urls.py
from django.urls import path
from . import views

urlpatterns = [
    path("dashboard/",  views.dashboard,  name="dashboard"),
    path("my-results/", views.my_results, name="my_results"),
]
```

---

## 🎨 Templates — showing data on a page

```django
<h1>My Results</h1>
<table>
  {% for r in data %}
    <tr>
      <td>{{ r.subject }}</td>
      <td>{{ r.total_marks }}</td>
      <td>{{ r.letter_grade }}</td>
    </tr>
  {% endfor %}
</table>

{% if data %}
  <p>Total subjects: {{ data.count }}</p>
{% else %}
  <p>No results yet.</p>
{% endif %}
```

| Tag | Does |
|---|---|
| `{{ variable }}` | print a value |
| `{% for x in list %} … {% endfor %}` | loop |
| `{% if … %} … {% else %} … {% endif %}` | condition |
| `{% url 'name' %}` | link to a URL by its name |
| `{% extends 'base.html' %}` | reuse a layout |

---

## 🔐 Auth — logins & protecting pages

```python
from django.contrib.auth.decorators import login_required

@login_required                      # must be logged in to see this page
def dashboard(request):
    user = request.user              # who is logged in right now
    ...
```

| Thing | Meaning |
|---|---|
| `request.user` | the currently logged-in user |
| `@login_required` | block page unless logged in |
| `user.is_authenticated` | True/False, are they logged in |
| Passwords | Django hashes them automatically — never store plain text |

---

## 🦸 Admin panel — free data-entry tool

```python
# in admin.py
from django.contrib import admin
from .models import Result, Student, Subject

admin.site.register(Result)
admin.site.register(Student)
admin.site.register(Subject)
```

> This gives the System Admin / Controller of Examinations a ready-made screen
> to add students and upload results — your "data feeder" tool, for free.

---

## 🧰 Typical file layout (so you know where things go)

```
mysite/                  ← the PROJECT
├── manage.py            ← your remote control
├── mysite/
│   ├── settings.py      ← configuration (database, apps list)
│   └── urls.py          ← main URL map
└── results/             ← an APP
    ├── models.py        ← your tables (Models)
    ├── views.py         ← your logic (Views)
    ├── admin.py         ← register models for the admin panel
    ├── urls.py          ← this app's URLs
    └── templates/       ← HTML pages
```

---

## ✅ "Am I ready to build?" checklist

You're ready when you can (even roughly):
- [ ] Create a project + app and run the server
- [ ] Write a Model with a `ForeignKey` and migrate it
- [ ] Add data through the admin panel
- [ ] Write a View that uses `.filter()` and renders a template
- [ ] Show that data in a template with a `{% for %}` loop
- [ ] Protect a page with `@login_required`

Don't aim for perfect — aim for *familiar*. We build the rest together. 🚀

---

## 🔗 Recommended learning order
1. **Django Girls Tutorial** — gentlest intro (build a blog)
2. **Official Django Tutorial** — the "polls app" (parts 1–5)
3. Come back here → we translate DATABASE_DESIGN.md into real Models together.
