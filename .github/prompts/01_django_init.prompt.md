---
description: Initialize a simple Django REST Framework project with minimal structure
---

# Simple Django Project Initialization

This prompt creates a minimal Django REST Framework project with only essential components needed to run.

## Project Configuration

**Project Details:**
- 📦 Project Name: `${input:project_name}`
- 🗄️ Database: SQLite (default)

## Instructions

Follow these steps to create a simple, working Django project:

### 1. Create Project Directory Structure

Create the following minimal directory structure:

```
${input:project_name}/
├── manage.py
├── requirements.txt
├── .env
├── .gitignore
├── ${input:project_name}/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
└── apps/
    └── api/
        ├── __init__.py
        ├── apps.py
        ├── views.py
        ├── urls.py
        └── migrations/
            └── __init__.py
```

### 2. Create Essential Files

#### requirements.txt
```
Django>=4.2,<5.0
djangorestframework>=3.14.0
python-decouple>=3.8
drf-yasg>=1.21.0
```

#### .gitignore
```
# Python
__pycache__/
*.py[cod]
*$py.class
*.so

# Django
db.sqlite3
media/
staticfiles/

# Environment
.env
venv/
env/

# IDE
.vscode/
.idea/
*.swp
```

#### .env
```
SECRET_KEY=your-secret-key-here-change-this-in-production
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1
```

### 3. Initialize Django Project

```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# Linux/macOS:
source venv/bin/activate
# Windows:
venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Create Django project
django-admin startproject ${input:project_name} .

# Create api app in apps folder
mkdir -p apps/api
python manage.py startapp api apps/api
```

### 4. Configure Django Settings

#### ${input:project_name}/settings.py

**Replace the entire settings.py with this simplified version:**

```python
"""
Django settings for ${input:project_name} project.
"""
import os
from pathlib import Path
from decouple import config

# Build paths
BASE_DIR = Path(__file__).resolve().parent.parent

# Security
SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)
ALLOWED_HOSTS = config('ALLOWED_HOSTS', default='').split(',')

# Application definition
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Third party apps
    'rest_framework',
    'drf_yasg',
    # Local apps
    'apps.api',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = '${input:project_name}.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = '${input:project_name}.wsgi.application'

# Database - Using SQLite for simplicity
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

# Password validation
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]

# Internationalization
LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'UTC'
USE_I18N = True
USE_TZ = True

# Static files
STATIC_URL = 'static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'

# Default primary key field type
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

# REST Framework basic settings
REST_FRAMEWORK = {
    'DEFAULT_RENDERER_CLASSES': [
        'rest_framework.renderers.JSONRenderer',
    ],
}
```

### 5. Create API Views and URLs

#### apps/api/views.py
```python
"""
Simple API views with health check endpoint and Swagger documentation.
"""
from django.http import JsonResponse
from rest_framework.decorators import api_view
from rest_framework.response import Response
from drf_yasg.utils import swagger_auto_schema
from drf_yasg import openapi

def health_check(request):
    """
    Health check endpoint to verify the API is running.
    Returns the current status of the API service.
    """
    return JsonResponse({
        'status': 'healthy',
        'service': '${input:project_name}',
        'message': 'API is running successfully!'
    })

@swagger_auto_schema(
    method='get',
    operation_description="Get API information and available endpoints",
    responses={
        200: openapi.Response(
            description="API information",
            examples={
                "application/json": {
                    "message": "Welcome to ${input:project_name} API",
                    "endpoints": {
                        "health": "/api/health/",
                        "swagger": "/api/swagger/",
                        "admin": "/admin/"
                    }
                }
            }
        )
    }
)
@api_view(['GET'])
def api_root(request):
    """
    API root endpoint showing available endpoints and documentation.
    """
    return Response({
        'message': 'Welcome to ${input:project_name} API',
        'endpoints': {
            'health': '/api/health/',
            'swagger': '/api/swagger/',
            'admin': '/admin/'
        }
    })
```

#### apps/api/urls.py
```python
"""
API URL configuration with Swagger documentation.
"""
from django.urls import path
from rest_framework import permissions
from drf_yasg.views import get_schema_view
from drf_yasg import openapi
from . import views

# Swagger/OpenAPI schema configuration
schema_view = get_schema_view(
    openapi.Info(
        title="${input:project_name} API",
        default_version='v1',
        description="Simple API for ${input:project_name} project",
        contact=openapi.Contact(email="contact@example.com"),
        license=openapi.License(name="MIT License"),
    ),
    public=True,
    permission_classes=[permissions.AllowAny],
)

urlpatterns = [
    # API endpoints
    path('', views.api_root, name='api_root'),
    path('health/', views.health_check, name='health_check'),
    
    # Swagger documentation
    path('swagger/', schema_view.with_ui('swagger', cache_timeout=0), name='schema-swagger-ui'),
    path('swagger.json', schema_view.without_ui(cache_timeout=0), name='schema-json'),
]
```

#### ${input:project_name}/urls.py
```python
"""
Main URL configuration for ${input:project_name}.
"""
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('apps.api.urls')),
]
```

### 6. Final Setup Commands

```bash
# Run migrations
python manage.py migrate

# Create superuser (optional)
python manage.py createsuperuser

# Run the development server
python manage.py runserver
```

### 7. Verify Success

**Test these URLs in your browser:**

- ✅ **API Root**: http://127.0.0.1:8000/api/
- ✅ **Health Check**: http://127.0.0.1:8000/api/health/
- ✅ **Swagger UI**: http://127.0.0.1:8000/api/swagger/
- ✅ **Admin Panel**: http://127.0.0.1:8000/admin/

**Expected Health Check Response:**
```json
{
    "status": "healthy",
    "service": "${input:project_name}",
    "message": "API is running successfully!"
}
```

**Expected API Root Response:**
```json
{
    "message": "Welcome to ${input:project_name} API",
    "endpoints": {
        "health": "/api/health/",
        "swagger": "/api/swagger/",
        "admin": "/admin/"
    }
}
```

## Common Issues and Quick Fixes

### Issue: "No module named 'decouple'"
```bash
pip install python-decouple
```

### Issue: "No module named 'drf_yasg'" 
```bash
pip install drf-yasg
```

### Issue: "Secret key error"
```bash
# Make sure .env file exists with:
SECRET_KEY=any-random-string-here
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1
```

### Issue: Port already in use
```bash
# Use a different port
python manage.py runserver 8001
```

## Project Complete!

Your simple Django project is now running with:
- ✅ Minimal structure (only essential files)
- ✅ Working health check endpoint
- ✅ Swagger UI documentation at `/api/swagger/`
- ✅ Admin panel ready
- ✅ SQLite database (no external DB needed)
- ✅ Environment variables support

**Next steps:**
1. Add your own models in `apps/api/models.py`
2. Create more API endpoints in `apps/api/views.py`
3. Add authentication if needed
4. Deploy to production when ready

**Note**: This is a minimal setup focused on getting a Django project running quickly. Add more features as your project grows.