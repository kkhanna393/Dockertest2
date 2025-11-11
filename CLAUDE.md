# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Django 5.2.8 web application named `helloworld_project` with a single app called `hello`. The project uses SQLite as the database and includes a basic "Hello World" view.

## Project Structure

```
helloworld_project/     # Main project configuration
├── settings.py         # Django settings (SECRET_KEY, INSTALLED_APPS, middleware, etc.)
├── urls.py             # Root URL configuration (includes hello.urls)
├── wsgi.py             # WSGI application entry point
└── asgi.py             # ASGI application entry point

hello/                  # Main Django app
├── views.py            # View functions (currently has index view)
├── urls.py             # App-specific URL patterns
├── models.py           # Database models (currently empty)
├── admin.py            # Django admin configuration
├── tests.py            # Test cases
└── migrations/         # Database migrations

manage.py               # Django management script
db.sqlite3              # SQLite database file
venv/                   # Python virtual environment
```

## Common Commands

### Development Server
```bash
python manage.py runserver
```
Start the development server at http://127.0.0.1:8000/

### Database Operations
```bash
python manage.py makemigrations    # Create new migrations based on model changes
python manage.py migrate           # Apply database migrations
python manage.py dbshell           # Open database shell
python manage.py showmigrations    # Show migration status
```

### Testing
```bash
python manage.py test              # Run all tests
python manage.py test hello        # Run tests for the hello app
python manage.py test hello.tests.TestClassName  # Run specific test class
```

### Admin and User Management
```bash
python manage.py createsuperuser   # Create admin user for Django admin
python manage.py changepassword    # Change user password
```

### Shell and Inspection
```bash
python manage.py shell             # Open Django shell with project context
python manage.py check             # Check for common issues
python manage.py inspectdb         # Inspect existing database and output models
```

### Static Files
```bash
python manage.py collectstatic     # Collect static files for production
python manage.py findstatic        # Find static file locations
```

## Architecture Notes

### URL Routing
The project uses a two-level URL routing system:
- Root URLconf in `helloworld_project/urls.py` includes the admin interface at `/admin/`
- App-level URLs in `hello/urls.py` are included at the root path (`''`)
- The `hello` app handles the root URL with its index view

### Settings Configuration
- Project name: `helloworld_project`
- Default settings module: `helloworld_project.settings` (set in manage.py)
- Database: SQLite (db.sqlite3)
- Debug mode is enabled (DEBUG = True)
- Installed apps include all default Django apps plus `hello`

### Application Layer
The `hello` app is registered in INSTALLED_APPS and currently provides:
- A simple HttpResponse-based view at the root URL
- No models defined yet
- Standard Django app structure ready for expansion

## Development Notes

- Always activate the virtual environment (`venv`) before running commands
- The SECRET_KEY in settings.py is insecure and should be changed for production
- DEBUG should be set to False in production environments
- ALLOWED_HOSTS must be configured before deploying to production
