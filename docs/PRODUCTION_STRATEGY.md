# Production Grade Application Strategy

This document outlines the transition of the Student Performance Prediction platform from a prototype (using generated mock data and a basic SQLite database) to a fully-fledged, production-ready application.

## 1. Bridging the Gap: Generated vs. Real Dataset

Currently, our database (`app/db.sqlite3`) is populated using a seeder script (`scripts/import_university_data.py`). While this is perfect for building out the UI and laying the groundwork for the ML pipeline, it is not a real-world scenario.

### How to shift to a real dataset:
*   **Data Ingestion Pipeline**: In a real scenario, data will be provided either via APIs from an existing university ERP system or via bulk CSV/Excel uploads from administrators.
*   **Django Management Commands**: We will transition our `import_university_data.py` into robust Django Management commands (e.g., `python manage.py load_students data.csv`) or asynchronous Celery tasks to handle massive data uploads without freezing the web server.
*   **Data Validation**: Real data is messy. We will implement robust validators in Django forms and serializers to clean missing fields, handle duplicates, and normalize text (e.g., converting "high school" and "High School" to standard ENUMs) before inserting them into our database.
*   **Database Migration**: When real data arrives, we will simply drop the mock database tables (or flush the data using `python manage.py flush`), connect our Django app to the production database (e.g., PostgreSQL or MySQL), and run the ingestion pipeline.

## 2. Production Grade Requirements vs. Current State

To classify this project as a "Production Grade App", we need to meet several industry standards. Below is an outline of the requirements, our current state, and the upgrade path.

### A. Database
*   **Requirement**: High concurrency, data integrity, robust backups.
*   **Current State**: SQLite3 (great for local development and prototyping, but locks on concurrent writes).
*   **Upgrade Path**: Migrate to **PostgreSQL**. Django handles this seamlessly; we just need to change the `DATABASES` setting in `settings.py` and run migrations.

### B. Security & Authentication
*   **Requirement**: Secure passwords, role-based access control, CSRF/XSS protection, rate limiting, and environment variable management.
*   **Current State**: Django provides built-in CSRF/XSS protection. We have basic role-based access (Student, Teacher, Admin). Passwords are hashed.
*   **Upgrade Path**: 
    *   Move hardcoded secret keys and debug variables to `.env` files using `python-dotenv`.
    *   Implement Two-Factor Authentication (2FA) or SSO (Single Sign-On).
    *   Set `DEBUG = False` and configure allowed hosts.

### C. Machine Learning Pipeline
*   **Requirement**: Scalable model training, model versioning, periodic retraining, and fast inference.
*   **Current State**: We are planning a local Scikit-Learn pipeline serialized as a `.pkl` file.
*   **Upgrade Path**:
    *   Shift training to an asynchronous background task (e.g., using Celery or Redis) so the main web server isn't blocked.
    *   Store models using a model registry (like MLflow) to track versions and performance metrics.
    *   Expose predictions via a REST API endpoint (Django REST Framework) to decouple the ML logic from the frontend views.

### D. Hosting and Deployment
*   **Requirement**: Load balancing, auto-scaling, static file management, CI/CD pipeline.
*   **Current State**: Running locally using `python manage.py runserver`.
*   **Upgrade Path**:
    *   **Server**: Use Gunicorn as the WSGI HTTP Server.
    *   **Proxy**: Put Nginx in front to serve static assets (CSS/JS) and proxy requests to Gunicorn.
    *   **Containerization**: Dockerize the application to ensure it runs consistently across any environment.
    *   **Cloud Hosting**: Deploy to platforms like AWS, Google Cloud (GCP), or Heroku.

## 3. The Future ML Features Plan (Archived)

As per user request, we have archived the plan to integrate behavioral metrics into our database for future implementation:
*   We will extend the `Result` model to include fields: `Attendance`, `Hours_Studied`, `Sleep_Hours`, `Motivation_Level`, `Tutoring_Sessions`, `Extracurricular_Activities`, `Physical_Activity`, `Parental_Involvement`, and `Peer_Influence`.
*   We will seed this data to match Kaggle distributions.
*   We will train the Scikit-learn model prioritizing these features and implement front-end forms for continuous real-world data collection.
