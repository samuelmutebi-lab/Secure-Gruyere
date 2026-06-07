# Secure Gruyère

A secure-by-design Django 5.x implementation of the classic "Google Gruyère" vulnerable web application.

## Overview

This project serves as a modern, secure replacement for the original Gruyère application. It replicates the core functionality (user registration, profiles, snippets/posts, and file uploads) while eliminating all historical vulnerabilities through the use of Django's robust security features and secure coding practices.

## Security Design Summary

The application follows the "Secure by Design" philosophy:

1.  **Authentication & Session Management**:
    *   Uses Django's built-in authentication system.
    *   Passwords are salted and hashed using PBKDF2 (standard in Django).
    *   Session cookies are configured with `HttpOnly`, `SameSite=Lax`, and `Secure` (in production).
    *   `SECRET_KEY` is managed via environment variables.

2.  **Authorization (Prevention of IDOR)**:
    *   Strict object-level authorization is implemented in views.
    *   Users can only edit or delete their own posts (verified via `validate_ownership` service).
    *   Profile access is tied directly to the `request.user` instance.

3.  **Cross-Site Scripting (XSS) Protection**:
    *   Relies on Django's automatic HTML escaping in the template engine.
    *   User input is NEVER rendered using the `|safe` filter unless explicitly intended and sanitized.

4.  **Cross-Site Request Forgery (CSRF) Protection**:
    *   CSRF middleware is enabled globally.
    *   All state-changing operations (POST/PUT/DELETE) require a valid CSRF token.

5.  **SQL Injection Prevention**:
    *   All database interactions use the Django ORM, which employs parameterized queries.

6.  **Secure File Uploads**:
    *   Uploaded files are restricted by size (2MB) and extension.
    *   Files are stored in user-specific directories to prevent path traversal and overwrites.
    *   `SecurityMiddleware` ensures `X-Content-Type-Options: nosniff` is set.

7.  **Data Protection & Operational Security**:
    *   `DEBUG` is disabled for production environments.
    *   Security logging is implemented for login attempts and unauthorized access.
    *   Secure headers (HSTS, X-Frame-Options) are configured for production.

## Threat Model Overview

| Threat | Mitigation Strategy |
| --- | --- |
| **Account Takeover** | Strong password hashing, secure session management, CSRF protection. |
| **Data Leakage** | Explicit authorization checks, no sensitive data in templates, DEBUG=False. |
| **Malicious Uploads** | Strict extension and size validation, safe storage paths. |
| **Client-side Attacks** | Auto-escaping, secure cookies, security headers. |
| **Injection** | Use of ORM for database, no raw SQL, validated form inputs. |

## Setup Instructions

### 1. Prerequisites
*   Python 3.10+
*   pip

### 2. Local Installation

1.  **Clone the repository** (if applicable) or navigate to the `secure_gruyere` directory.
2.  **Create a virtual environment**:
    ```bash
    python -m venv venv
    venv\Scripts\activate
    source venv/bin/activate  # On Windows: venv\Scripts\activate
    ```
3.  **Install dependencies**:
    ```bash
    pip install -r requirements.txt
    ```
4.  **Configure Environment**:
    Create a `.env` file based on `.env.example`:
    ```bash
    cp .env.example .env
    ```
    Generate a secure `SECRET_KEY` and update the file. Set `DEBUG=True` for local development.

### 3. Run the Application

1.  **Run Migrations**:
    ```bash
    python manage.py migrate
    ```
2.  **Create a Superuser** (for admin access):
    ```bash
    python manage.py createsuperuser
    ```
3.  **Start the Server**:
    ```bash
    python manage.py runserver

    python manage.py runserver_plus  0.0.0.0:8000  --cert-file cert.crt   --key-file cert.key

    
    ```
    Access the app at `http://127.0.0.1:8000/`.

## Running Tests
(Recommended: Add unit tests in `accounts/tests.py` and `posts/tests.py`)
```bash
python manage.py test
```
<!-- for Production -->
pip install django-extensions
pip install Werkzeug
pip install pyOpenSSL
pip install django-csp

python manage.py runserver_plus  0.0.0.0:8000  --cert-file cert.crt   --key-file cert.key


venv\Scripts\activate
prompt $G