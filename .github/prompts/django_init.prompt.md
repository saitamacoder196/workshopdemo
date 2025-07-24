---
description: Initialize a Django REST Framework project with proper structure and avoid common setup errors
---

# Django REST Framework Project Initialization

Welcome! This prompt will guide you through creating a Django REST Framework project with proper structure and error prevention.

## Project Configuration

**Project Details:**
- 📦 Project Name: `${input:project_name}`
- 🏗️ Main App: `${input:app_name}`
- 🗄️ Database: `${input:database_type}` (postgresql/mysql/sqlite)
- 🐳 Docker Support: `${input:use_docker}`

## Instructions

Follow these steps carefully to avoid common Django setup errors:

### 1. Create Project Directory Structure

Create the following directory structure based on Django best practices:

```
${input:project_name}/
├── apps/
│   ├── api/
│   ├── authentication/
│   ├── core/
│   └── ${input:app_name}/
├── config/
│   └── settings/
├── docs/
├── requirements/
├── scripts/
├── static/
├── tests/
│   ├── factories/
│   ├── integration/
│   └── unit/
├── utils/
├── manage.py
├── pyproject.toml
└── README.md
```

### 2. Create Essential Configuration Files

#### requirements/base.txt
```
Django>=4.2,<5.0
djangorestframework>=3.14.0
django-cors-headers>=4.0.0
python-decouple>=3.8
```

Add database-specific requirements:
- postgresql: `psycopg2-binary>=2.9.0`
- mysql: `mysqlclient>=2.1.0`
- sqlite: (built-in, no additional requirement)

#### requirements/development.txt
```
-r base.txt
django-debug-toolbar>=4.0.0
```

#### pyproject.toml
```toml
[tool.black]
line-length = 88
target-version = ['py311']

[tool.isort]
profile = "black"
```

### 3. Initialize Django Project

Execute Django initialization commands:

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements/development.txt

# Initialize Django project in config directory
django-admin startproject config .

# Create Django apps with proper structure
python manage.py startapp core apps/core
python manage.py startapp authentication apps/authentication  
python manage.py startapp api apps/api
python manage.py startapp ${input:app_name} apps/${input:app_name}
```

### 4. Configure Django Settings

#### config/settings/base.py
**Structure the base settings file:**

```python
"""
Base settings for ${input:project_name} project.
"""

# Build paths and security
BASE_DIR = Path(__file__).resolve().parent.parent.parent
SECRET_KEY = config('SECRET_KEY')
DEBUG = False
ALLOWED_HOSTS = config('ALLOWED_HOSTS', cast=Csv())

# Application definition - Organize apps by category
DJANGO_APPS = [
    'django.contrib.admin',
    'django.contrib.auth', 
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

THIRD_PARTY_APPS = [
    'rest_framework',
    'corsheaders',
]

LOCAL_APPS = [
    'apps.core',
    'apps.authentication', 
    'apps.api',
    'apps.${input:app_name}',
]

INSTALLED_APPS = DJANGO_APPS + THIRD_PARTY_APPS + LOCAL_APPS

# CRITICAL: Correct middleware order to avoid admin.E410 error
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',  # Must be before Auth
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',  # After Sessions
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# Only add custom middleware if the module exists
# This prevents ModuleNotFoundError during initial setup
# 'apps.core.middleware.RequestLoggingMiddleware',  # Add after creating the module

# Database configuration based on selected type
# Configure according to ${input:database_type} selection

# Static files configuration  
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
# Create static directory first: mkdir -p static
STATICFILES_DIRS = [BASE_DIR / 'static']

# REST Framework basic configuration
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20,
}
```

#### config/settings/development.py
**Development-specific settings:**

```python
"""Development settings for ${input:project_name} project."""
from .base import *

DEBUG = True
ALLOWED_HOSTS = ['*']

# Add debug toolbar ONLY if you create the URL patterns
# This prevents djdt namespace error (Issue #002)
INSTALLED_APPS += ['django_debug_toolbar']

# Add debug toolbar middleware
MIDDLEWARE = ['debug_toolbar.middleware.DebugToolbarMiddleware'] + MIDDLEWARE

INTERNAL_IPS = ['127.0.0.1']
```

#### config/settings/production.py & config/settings/testing.py
**Create minimal production and testing configurations**

### 5. Configure URL Routing 

#### config/urls.py
**Main URL configuration - Prevent missing module errors:**

```python
"""
Main URL configuration for ${input:project_name} project.
"""
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    # Only include API URLs after creating the module
    path('api/v1/', include('apps.api.urls')),  # Create this module first!
]

# Debug toolbar URLs - only if module exists and DEBUG is True
if settings.DEBUG and 'debug_toolbar' in settings.INSTALLED_APPS:
    import debug_toolbar
    urlpatterns = [
        path('__debug__/', include(debug_toolbar.urls)),
    ] + urlpatterns

# Static/media files in development
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
    urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```

### 6. Create Required App Modules

**IMPORTANT: Create these modules BEFORE referencing them in settings**

#### apps/core/
**Purpose: Core functionality - models, middleware, exceptions**

Create these files with basic structure:

- `apps/core/__init__.py` - Package initialization
- `apps/core/apps.py` - App configuration  
- `apps/core/models.py` - Base models (implement BaseModel with created_at, updated_at, is_active)
- `apps/core/middleware.py` - Custom middleware (implement RequestLoggingMiddleware)
- `apps/core/exceptions.py` - Exception handlers (implement custom_exception_handler)

#### apps/api/
**Purpose: API routing and endpoints**

Create these essential files:
- `apps/api/__init__.py` - Package initialization
- `apps/api/apps.py` - App configuration
- `apps/api/urls.py` - **CRITICAL**: Must exist before including in main URLs
- `apps/api/views.py` - API views (implement health check endpoint)

#### apps/authentication/
**Purpose: User authentication and authorization**

Create basic authentication structure:
- `apps/authentication/__init__.py` 
- `apps/authentication/apps.py`
- `apps/authentication/models.py` - User model extensions
- `apps/authentication/serializers.py` - Authentication serializers
- `apps/authentication/views.py` - Auth views

#### apps/${input:app_name}/
**Purpose: Main business logic**

Create your main app structure:
- `apps/${input:app_name}/__init__.py`
- `apps/${input:app_name}/apps.py` 
- `apps/${input:app_name}/models.py` - Business models
- `apps/${input:app_name}/views.py` - Business logic views
- `apps/${input:app_name}/urls.py` - App-specific URLs

### 7. Implement Essential Modules

#### apps/api/urls.py Implementation Guide
**Create this file first to prevent ModuleNotFoundError:**

```python
"""
API URL configuration for ${input:project_name}.
Implement basic endpoints to establish API structure.
"""

# Import necessary modules
# Define URL patterns for:
# 1. Health check endpoint: /health/
# 2. API documentation: /docs/ 
# 3. Authentication routes: /auth/
# 4. Main business logic routes: /${input:app_name}/

# Example pattern:
# path('health/', views.health_check, name='health_check')
```

#### apps/core/middleware.py Implementation Guide 
**Implement basic request logging:**

```python
"""
Custom middleware for request logging and monitoring.
"""

# Create RequestLoggingMiddleware class
# - Log incoming requests (method, path, IP)
# - Log response status and timing
# - Include error logging for debugging

# Implementation should handle:
# - Request timing
# - Client IP detection  
# - Response status logging
# - Exception logging
```

#### apps/core/exceptions.py Implementation Guide
**Implement custom exception handling:**

```python
"""
Custom exception handlers for consistent API responses.
"""

# Create custom_exception_handler function
# - Handle Django exceptions
# - Return consistent JSON error format
# - Log exceptions for monitoring

# Response format should be:
# {
#   "success": false,
#   "error": {
#     "code": "error_code", 
#     "message": "Error description"
#   }
# }
```

### 8. Create Directory Structure

**Create all required directories to prevent warnings:**

```bash
# Create missing directories
mkdir -p static apps/core/migrations apps/api/migrations
mkdir -p apps/authentication/migrations apps/${input:app_name}/migrations  
mkdir -p tests/unit tests/integration tests/factories
mkdir -p docs scripts utils

# Create __init__.py files for Python packages
touch apps/__init__.py apps/core/__init__.py apps/api/__init__.py
touch apps/authentication/__init__.py apps/${input:app_name}/__init__.py
touch tests/__init__.py utils/__init__.py

# Create empty migration __init__.py files
touch apps/core/migrations/__init__.py apps/api/migrations/__init__.py
touch apps/authentication/migrations/__init__.py 
touch apps/${input:app_name}/migrations/__init__.py
```

### 9. Environment Configuration

#### .env.example
```
# Django settings
DEBUG=True
SECRET_KEY=your-secret-key-here
ALLOWED_HOSTS=localhost,127.0.0.1

# Database configuration for ${input:database_type}
# Add appropriate database settings based on selection

# Security
CORS_ALLOWED_ORIGINS=http://localhost:3000
```

#### .gitignore
**Include essential patterns for Django projects**

### 10. Initial Setup and Validation

**Run setup commands in correct order:**

```bash
# 1. Verify all referenced modules exist
python -c "import apps.core, apps.api, apps.authentication, apps.${input:app_name}"

# 2. Run Django system checks 
python manage.py check

# 3. Create initial migrations
python manage.py makemigrations

# 4. Apply migrations
python manage.py migrate

# 5. Create superuser
python manage.py createsuperuser

# 6. Test server startup
python manage.py runserver
```

### 11. Error Prevention Checklist

**Before starting the server, verify:**

✅ **Module Existence**: All modules referenced in settings exist
- `apps.core.middleware` exists before adding to MIDDLEWARE
- `apps.api.urls` exists before including in URL patterns
- All app modules are created with proper __init__.py files

✅ **Middleware Order**: Correct middleware sequence
- SessionMiddleware before AuthenticationMiddleware  
- Debug toolbar middleware only in development
- Custom middleware added only after module creation

✅ **Directory Structure**: Required directories exist
- `static/` directory exists (prevents staticfiles.W004)
- All migration directories have __init__.py files
- Test directories are properly structured

✅ **URL Configuration**: Proper URL routing
- Debug toolbar URLs only included when DEBUG=True
- All included URL modules exist
- Static file serving configured for development

## Expected Outcome

A properly structured Django REST Framework project that:

✅ **Follows Django best practices** for project organization
✅ **Prevents common setup errors** identified in issues #001-#004  
✅ **Has minimal but functional** core modules
✅ **Ready for development** with proper foundation
✅ **Includes health check endpoint** at `/api/v1/health/`
✅ **Configurable database support** for ${input:database_type}
✅ **Development-ready** with debug toolbar properly configured

## Common Issues Prevention

This setup specifically prevents:
- **ModuleNotFoundError**: All referenced modules are created
- **Namespace errors**: Debug toolbar URLs properly configured  
- **Middleware order errors**: Correct sequence established
- **Static files warnings**: Directories created upfront
- **Migration errors**: Proper app structure with __init__.py files

## Next Steps

1. **Implement business logic** in `apps/${input:app_name}/`
2. **Add authentication endpoints** in `apps/authentication/`
3. **Expand API endpoints** in `apps/api/`
4. **Add comprehensive testing** in `tests/`
5. **Configure production deployment** settings

**Note**: This prompt creates a minimal but complete Django structure. Add implementation details as needed for your specific requirements.