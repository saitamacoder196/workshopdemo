---
description: Initialize a production-ready Django REST Framework project with modern Python practices and comprehensive tooling
---

# Django REST Framework Project Initialization

Welcome, ${input:project_name} developer! This prompt will create a production-ready Django REST Framework backend with modern Python practices and comprehensive tooling.

## Project Configuration

**Project Details:**
- 📦 Project Name: `${input:project_name}`
- 🏗️ Main App: `${input:app_name}`
- 🗄️ Database: `${input:database_type}` (postgresql/mysql/sqlite)
- 🐳 Docker Support: `${input:use_docker}`
- 🔄 Celery Integration: `${input:include_celery}`

## Instructions

Please execute the following steps in order:

### 1. Project Structure Setup

Create the following directory structure for `${input:project_name}`:

```
${input:project_name}/
├── apps/
│   ├── core/
│   ├── authentication/
│   ├── api/
│   └── ${input:app_name}/
├── config/
│   ├── settings/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── development.py
│   │   ├── production.py
│   │   └── testing.py
│   ├── __init__.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── tests/
│   ├── factories/
│   ├── unit/
│   ├── integration/
│   └── conftest.py
├── utils/
├── requirements/
│   ├── base.txt
│   ├── development.txt
│   ├── production.txt
│   └── testing.txt
├── scripts/
├── docs/
├── .env.example
├── .gitignore
├── .editorconfig
├── pyproject.toml
├── manage.py
└── README.md
```

### 2. Core Configuration Files

Create the following essential configuration files:

#### pyproject.toml
```toml
[tool.poetry]
name = "${input:project_name}"
version = "0.1.0"
description = "A production-ready Django REST Framework project"
authors = ["Your Name <your.email@example.com>"]

[tool.black]
line-length = 88
target-version = ['py311']
include = '\.pyi?$'

[tool.isort]
profile = "black"
multi_line_output = 3
include_trailing_comma = true
force_grid_wrap = 0
use_parentheses = true
ensure_newline_before_comments = true
line_length = 88

[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
plugins = ["mypy_django_plugin.main"]

[tool.pytest.ini_options]
DJANGO_SETTINGS_MODULE = "config.settings.testing"
python_files = ["test_*.py", "*_test.py", "testing/python/*.py"]
addopts = "-v --tb=short --strict-markers --cov=apps --cov-report=term-missing"
```

#### requirements/base.txt
```
Django>=4.2,<5.0
djangorestframework>=3.14.0
django-cors-headers>=4.0.0
djangorestframework-simplejwt>=5.2.0
django-filter>=22.1
drf-yasg>=1.21.0
python-decouple>=3.8
redis>=4.5.0
gunicorn>=20.1.0
```

Add database-specific requirements based on `${input:database_type}`:
- postgresql: `psycopg2-binary>=2.9.0`
- mysql: `mysqlclient>=2.1.0`
- sqlite: (built-in, no additional requirement)

Add Celery if `${input:include_celery}` is true:
```
celery>=5.2.0
kombu>=5.2.0
```

#### requirements/development.txt
```
-r base.txt
pytest>=7.0.0
pytest-django>=4.5.0
pytest-cov>=4.0.0
factory-boy>=3.2.0
black>=22.0.0
isort>=5.10.0
flake8>=5.0.0
mypy>=0.991
django-debug-toolbar>=4.0.0
ipython>=8.0.0
django-extensions>=3.2.0
```

### 3. Django Project Initialization

Execute these Django commands:

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements/development.txt

# Initialize Django project
django-admin startproject config .

# Create Django apps
python manage.py startapp core apps/core
python manage.py startapp authentication apps/authentication
python manage.py startapp api apps/api
python manage.py startapp ${input:app_name} apps/${input:app_name}
```

### 4. Settings Configuration

#### config/settings/base.py
```python
"""
Base settings for ${input:project_name} project.
For more information on this file, see
https://docs.djangoproject.com/en/4.2/topics/settings/
"""
import os
from pathlib import Path
from datetime import timedelta
from decouple import config, Csv

# Build paths inside the project
BASE_DIR = Path(__file__).resolve().parent.parent.parent

# Security
SECRET_KEY = config('SECRET_KEY')
DEBUG = False
ALLOWED_HOSTS = config('ALLOWED_HOSTS', cast=Csv())

# Application definition
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
    'rest_framework_simplejwt',
    'django_filters',
    'corsheaders',
    'drf_yasg',
]

LOCAL_APPS = [
    'apps.core',
    'apps.authentication',
    'apps.api',
    'apps.${input:app_name}',
]

INSTALLED_APPS = DJANGO_APPS + THIRD_PARTY_APPS + LOCAL_APPS

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'apps.core.middleware.RequestLoggingMiddleware',
]

ROOT_URLCONF = 'config.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
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

WSGI_APPLICATION = 'config.wsgi.application'

# Database configuration based on ${input:database_type}
if '${input:database_type}' == 'postgresql':
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql',
            'NAME': config('DB_NAME'),
            'USER': config('DB_USER'),
            'PASSWORD': config('DB_PASSWORD'),
            'HOST': config('DB_HOST', default='localhost'),
            'PORT': config('DB_PORT', default='5432'),
        }
    }
elif '${input:database_type}' == 'mysql':
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',
            'NAME': config('DB_NAME'),
            'USER': config('DB_USER'),
            'PASSWORD': config('DB_PASSWORD'),
            'HOST': config('DB_HOST', default='localhost'),
            'PORT': config('DB_PORT', default='3306'),
        }
    }
else:  # sqlite
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
        'OPTIONS': {
            'min_length': 8,
        }
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
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
STATICFILES_DIRS = [BASE_DIR / 'static']

# Media files
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# Default primary key field type
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

# REST Framework configuration
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20,
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
        'rest_framework.filters.SearchFilter',
        'rest_framework.filters.OrderingFilter',
    ],
    'DEFAULT_RENDERER_CLASSES': [
        'rest_framework.renderers.JSONRenderer',
    ],
    'DEFAULT_PARSER_CLASSES': [
        'rest_framework.parsers.JSONParser',
        'rest_framework.parsers.MultiPartParser',
    ],
    'EXCEPTION_HANDLER': 'apps.core.exceptions.custom_exception_handler',
}

# JWT Configuration
SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=60),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
    'UPDATE_LAST_LOGIN': True,
    'ALGORITHM': 'HS256',
    'SIGNING_KEY': config('JWT_SECRET_KEY', default=SECRET_KEY),
    'AUTH_HEADER_TYPES': ('Bearer',),
    'AUTH_HEADER_NAME': 'HTTP_AUTHORIZATION',
    'USER_ID_FIELD': 'id',
    'USER_ID_CLAIM': 'user_id',
}

# CORS Configuration
CORS_ALLOWED_ORIGINS = config('CORS_ALLOWED_ORIGINS', cast=Csv(), default='')
CORS_ALLOW_ALL_ORIGINS = config('CORS_ALLOW_ALL_ORIGINS', cast=bool, default=False)
CORS_ALLOW_CREDENTIALS = True

# Celery Configuration (if ${input:include_celery} is true)
if ${input:include_celery}:
    CELERY_BROKER_URL = config('CELERY_BROKER_URL', default='redis://localhost:6379/0')
    CELERY_RESULT_BACKEND = config('CELERY_RESULT_BACKEND', default='redis://localhost:6379/0')
    CELERY_ACCEPT_CONTENT = ['json']
    CELERY_TASK_SERIALIZER = 'json'
    CELERY_RESULT_SERIALIZER = 'json'
    CELERY_TIMEZONE = TIME_ZONE

# Logging Configuration
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {process:d} {thread:d} {message}',
            'style': '{',
        },
    },
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'formatter': 'verbose',
        },
    },
    'root': {
        'handlers': ['console'],
        'level': config('LOG_LEVEL', default='INFO'),
    },
    'loggers': {
        'django': {
            'handlers': ['console'],
            'level': config('DJANGO_LOG_LEVEL', default='INFO'),
            'propagate': False,
        },
    },
}
```

#### config/settings/development.py
```python
"""Development settings for ${input:project_name} project."""
from .base import *

DEBUG = True

ALLOWED_HOSTS = ['*']

# Django Debug Toolbar
INSTALLED_APPS += ['debug_toolbar']
MIDDLEWARE = ['debug_toolbar.middleware.DebugToolbarMiddleware'] + MIDDLEWARE

INTERNAL_IPS = ['127.0.0.1']

# Email backend for development
EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'

# Disable password validators in development
AUTH_PASSWORD_VALIDATORS = []

# CORS - Allow all origins in development
CORS_ALLOW_ALL_ORIGINS = True
```

#### config/settings/production.py
```python
"""Production settings for ${input:project_name} project."""
from .base import *

DEBUG = False

# Security settings
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
X_FRAME_OPTIONS = 'DENY'

# HSTS settings
SECURE_HSTS_SECONDS = 31536000  # 1 year
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
```

#### config/urls.py
```python
"""
Main URL configuration for ${input:project_name} project.
"""
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    # Admin interface
    path('admin/', admin.site.urls),
    
    # API endpoints
    path('api/v1/', include('apps.api.urls')),
]

# Serve media files in development
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
    urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
    
    # Add Django Debug Toolbar URLs
    if 'debug_toolbar' in settings.INSTALLED_APPS:
        import debug_toolbar
        urlpatterns = [
            path('__debug__/', include(debug_toolbar.urls)),
        ] + urlpatterns

# Customize admin site headers
admin.site.site_header = '${input:project_name} Administration'
admin.site.site_title = '${input:project_name} Admin Portal'
admin.site.index_title = 'Welcome to ${input:project_name} Administration'
```

#### config/settings/testing.py
```python
"""Testing settings for ${input:project_name} project."""
from .base import *

# Test database
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': ':memory:',
    }
}

# Disable migrations for faster tests
class DisableMigrations:
    def __contains__(self, item):
        return True
    
    def __getitem__(self, item):
        return None

MIGRATION_MODULES = DisableMigrations()

# Faster password hashing for tests
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.MD5PasswordHasher',
]

# Disable cache for tests
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.dummy.DummyCache',
    }
}

# Disable logging during tests
LOGGING_CONFIG = None

# Celery settings for testing
CELERY_TASK_ALWAYS_EAGER = True
CELERY_TASK_EAGER_PROPAGATES = True

# Email backend for tests
EMAIL_BACKEND = 'django.core.mail.backends.locmem.EmailBackend'
```

### 5. Core Django Apps Setup

#### apps/core/__init__.py
```python
"""
Core app containing base models, utilities, exceptions, and middlewares.
This app provides common functionality used across the entire project.
"""
```

#### apps/core/apps.py
```python
"""
App configuration for the core app.
"""
from django.apps import AppConfig


class CoreConfig(AppConfig):
    """Configuration for the core app."""
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'apps.core'
    verbose_name = 'Core'
    
    def ready(self):
        """Import signals when the app is ready."""
        try:
            import apps.core.signals  # noqa F401
        except ImportError:
            pass
```

#### apps/core/models.py
```python
"""
Base models and model utilities for the project.
Provides abstract base models with common fields and behaviors.
"""
from django.db import models
from django.utils import timezone


class BaseModel(models.Model):
    """
    Abstract base model with common fields for all models.
    Provides created_at, updated_at timestamps and soft delete functionality.
    """
    created_at = models.DateTimeField(
        default=timezone.now,
        help_text="Timestamp when the record was created"
    )
    updated_at = models.DateTimeField(
        auto_now=True,
        help_text="Timestamp when the record was last updated"
    )
    is_active = models.BooleanField(
        default=True,
        help_text="Flag to indicate if the record is active (soft delete)"
    )
    
    class Meta:
        abstract = True
        ordering = ['-created_at']
    
    def soft_delete(self):
        """Soft delete the record by setting is_active to False."""
        self.is_active = False
        self.save(update_fields=['is_active', 'updated_at'])
    
    def restore(self):
        """Restore a soft deleted record."""
        self.is_active = True
        self.save(update_fields=['is_active', 'updated_at'])


class ActiveManager(models.Manager):
    """Manager that returns only active records."""
    
    def get_queryset(self):
        """Return only active records."""
        return super().get_queryset().filter(is_active=True)


class TimestampedModel(models.Model):
    """
    Abstract model with only timestamp fields.
    Use when you need timestamps but not soft delete functionality.
    """
    created_at = models.DateTimeField(default=timezone.now)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        abstract = True
        ordering = ['-created_at']
```

#### apps/core/exceptions.py
```python
"""
Custom exceptions and exception handlers for the project.
Provides consistent error response format across the API.
"""
from rest_framework.views import exception_handler
from rest_framework.response import Response
from rest_framework import status
from django.core.exceptions import ValidationError as DjangoValidationError
from django.http import Http404
import logging

logger = logging.getLogger(__name__)


class BaseAPIException(Exception):
    """Base exception class for API custom exceptions."""
    status_code = status.HTTP_500_INTERNAL_SERVER_ERROR
    default_detail = 'A server error occurred.'
    default_code = 'error'
    
    def __init__(self, detail=None, code=None):
        if detail is None:
            detail = self.default_detail
        if code is None:
            code = self.default_code
        
        self.detail = detail
        self.code = code


class NotFoundError(BaseAPIException):
    """Exception for resource not found errors."""
    status_code = status.HTTP_404_NOT_FOUND
    default_detail = 'Resource not found.'
    default_code = 'not_found'


class ValidationError(BaseAPIException):
    """Exception for validation errors."""
    status_code = status.HTTP_400_BAD_REQUEST
    default_detail = 'Invalid input.'
    default_code = 'validation_error'


class AuthenticationError(BaseAPIException):
    """Exception for authentication errors."""
    status_code = status.HTTP_401_UNAUTHORIZED
    default_detail = 'Authentication credentials were not provided or are invalid.'
    default_code = 'authentication_error'


class PermissionError(BaseAPIException):
    """Exception for permission errors."""
    status_code = status.HTTP_403_FORBIDDEN
    default_detail = 'You do not have permission to perform this action.'
    default_code = 'permission_denied'


class ConflictError(BaseAPIException):
    """Exception for conflict errors (e.g., duplicate resources)."""
    status_code = status.HTTP_409_CONFLICT
    default_detail = 'Request conflicts with current state.'
    default_code = 'conflict'


def custom_exception_handler(exc, context):
    """
    Custom exception handler that provides consistent error responses.
    Returns error responses in the format:
    {
        "success": false,
        "error": {
            "code": "error_code",
            "message": "Error message",
            "details": {...}  // Optional additional details
        }
    }
    """
    # Call REST framework's default exception handler first
    response = exception_handler(exc, context)
    
    # Handle Django 404
    if isinstance(exc, Http404):
        exc = NotFoundError()
    
    # Handle Django ValidationError
    elif isinstance(exc, DjangoValidationError):
        exc = ValidationError(detail=str(exc))
    
    # Handle custom API exceptions
    if isinstance(exc, BaseAPIException):
        logger.error(f"API Exception: {exc.code} - {exc.detail}")
        
        error_response = {
            'success': False,
            'error': {
                'code': exc.code,
                'message': exc.detail,
            }
        }
        
        return Response(error_response, status=exc.status_code)
    
    # Format DRF exceptions
    if response is not None:
        error_response = {
            'success': False,
            'error': {
                'code': getattr(exc, 'default_code', 'error'),
                'message': 'An error occurred',
                'details': response.data
            }
        }
        response.data = error_response
    
    return response
```

#### apps/core/middleware.py
```python
"""
Custom middleware for the project.
Provides request/response processing and logging functionality.
"""
import time
import json
import logging
from django.utils.deprecation import MiddlewareMixin
from django.http import JsonResponse

logger = logging.getLogger(__name__)


class RequestLoggingMiddleware(MiddlewareMixin):
    """
    Middleware to log all incoming requests and outgoing responses.
    Logs request method, path, response status, and response time.
    """
    
    def process_request(self, request):
        """Log incoming request and start timing."""
        request._start_time = time.time()
        
        logger.info(
            f"Request: {request.method} {request.path} "
            f"from {self.get_client_ip(request)}"
        )
        
        return None
    
    def process_response(self, request, response):
        """Log response details and timing."""
        if hasattr(request, '_start_time'):
            duration = time.time() - request._start_time
            
            logger.info(
                f"Response: {request.method} {request.path} "
                f"Status: {response.status_code} "
                f"Duration: {duration:.3f}s"
            )
        
        return response
    
    def get_client_ip(self, request):
        """Get the client's IP address from the request."""
        x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
        if x_forwarded_for:
            ip = x_forwarded_for.split(',')[0]
        else:
            ip = request.META.get('REMOTE_ADDR')
        return ip


class JSONParsingMiddleware(MiddlewareMixin):
    """
    Middleware to handle JSON parsing errors gracefully.
    Returns a properly formatted error response for invalid JSON.
    """
    
    def process_exception(self, request, exception):
        """Handle JSON parsing exceptions."""
        if isinstance(exception, json.JSONDecodeError):
            return JsonResponse({
                'success': False,
                'error': {
                    'code': 'invalid_json',
                    'message': 'Invalid JSON in request body',
                    'details': str(exception)
                }
            }, status=400)
        
        return None


class HealthCheckMiddleware(MiddlewareMixin):
    """
    Middleware to handle health check requests.
    Provides a simple health check endpoint that bypasses authentication.
    """
    
    def process_request(self, request):
        """Check if this is a health check request."""
        if request.path == '/api/v1/health/':
            return JsonResponse({
                'success': True,
                'status': 'healthy',
                'service': '${input:project_name}'
            })
        
        return None
```

#### apps/core/permissions.py
```python
"""
Custom permission classes for the project.
Provides fine-grained access control for API endpoints.
"""
from rest_framework import permissions


class IsOwnerOrReadOnly(permissions.BasePermission):
    """
    Custom permission to only allow owners of an object to edit it.
    Assumes the model has an 'owner' or 'user' field.
    """
    
    def has_object_permission(self, request, view, obj):
        # Read permissions are allowed to any request
        if request.method in permissions.SAFE_METHODS:
            return True
        
        # Write permissions are only allowed to the owner
        return obj.owner == request.user or obj.user == request.user


class IsOwner(permissions.BasePermission):
    """
    Custom permission to only allow owners of an object to view or edit it.
    """
    
    def has_object_permission(self, request, view, obj):
        return obj.owner == request.user or obj.user == request.user


class IsSuperUser(permissions.BasePermission):
    """
    Custom permission to only allow superusers.
    """
    
    def has_permission(self, request, view):
        return request.user and request.user.is_superuser


class IsStaffOrReadOnly(permissions.BasePermission):
    """
    Custom permission to allow staff users to edit, others read-only.
    """
    
    def has_permission(self, request, view):
        if request.method in permissions.SAFE_METHODS:
            return True
        return request.user and request.user.is_staff
```

#### apps/core/utils.py
```python
"""
Utility functions and helpers for the project.
Provides common functionality used across different apps.
"""
import string
import random
from django.utils.text import slugify


def generate_unique_code(length=6):
    """
    Generate a unique alphanumeric code.
    
    Args:
        length (int): Length of the code to generate
    
    Returns:
        str: Unique alphanumeric code
    """
    characters = string.ascii_uppercase + string.digits
    return ''.join(random.choice(characters) for _ in range(length))


def generate_unique_slug(text, model_class, slug_field='slug'):
    """
    Generate a unique slug for a model instance.
    
    Args:
        text (str): Text to create slug from
        model_class: Django model class to check uniqueness against
        slug_field (str): Name of the slug field in the model
    
    Returns:
        str: Unique slug
    """
    slug = slugify(text)
    unique_slug = slug
    num = 1
    
    while model_class.objects.filter(**{slug_field: unique_slug}).exists():
        unique_slug = f'{slug}-{num}'
        num += 1
    
    return unique_slug


def get_client_ip(request):
    """
    Get the client's IP address from the request.
    
    Args:
        request: Django request object
    
    Returns:
        str: Client's IP address
    """
    x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
    if x_forwarded_for:
        ip = x_forwarded_for.split(',')[0]
    else:
        ip = request.META.get('REMOTE_ADDR')
    return ip


class Choices:
    """
    Base class for creating choice constants.
    
    Example:
        class StatusChoices(Choices):
            PENDING = 'pending', 'Pending'
            APPROVED = 'approved', 'Approved'
            REJECTED = 'rejected', 'Rejected'
    """
    
    @classmethod
    def choices(cls):
        """Return choices for Django model field."""
        return [
            (value, label)
            for name, (value, label) in cls.__dict__.items()
            if not name.startswith('_') and isinstance(value, tuple)
        ]
    
    @classmethod
    def values(cls):
        """Return list of choice values."""
        return [value for value, label in cls.choices()]
    
    @classmethod
    def labels(cls):
        """Return list of choice labels."""
        return [label for value, label in cls.choices()]
```

#### apps/api/__init__.py
```python
"""
API app for handling API routing, versioning, and documentation.
This app serves as the main entry point for all API endpoints.
"""
```

#### apps/api/apps.py
```python
"""
App configuration for the API app.
"""
from django.apps import AppConfig


class ApiConfig(AppConfig):
    """Configuration for the API app."""
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'apps.api'
    verbose_name = 'API'
```

#### apps/api/urls.py
```python
"""
Main API URL configuration.
Includes versioning, documentation, and routing to other apps.
"""
from django.urls import path, include
from rest_framework import permissions
from drf_yasg.views import get_schema_view
from drf_yasg import openapi
from .views import HealthCheckView

# API documentation schema
schema_view = get_schema_view(
    openapi.Info(
        title="${input:project_name} API",
        default_version='v1',
        description="API documentation for ${input:project_name}",
        terms_of_service="https://www.example.com/terms/",
        contact=openapi.Contact(email="contact@example.com"),
        license=openapi.License(name="BSD License"),
    ),
    public=True,
    permission_classes=[permissions.AllowAny],
)

urlpatterns = [
    # Health check endpoint
    path('health/', HealthCheckView.as_view(), name='health-check'),
    
    # API documentation
    path('swagger/', schema_view.with_ui('swagger', cache_timeout=0), name='schema-swagger-ui'),
    path('redoc/', schema_view.with_ui('redoc', cache_timeout=0), name='schema-redoc'),
    
    # Authentication endpoints
    path('auth/', include('apps.authentication.urls')),
    
    # Main app endpoints
    path('${input:app_name}/', include('apps.${input:app_name}.urls')),
]
```

#### apps/api/views.py
```python
"""
API views for common endpoints.
Provides health check and other utility endpoints.
"""
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from rest_framework.permissions import AllowAny
from django.db import connection
from django.core.cache import cache
import redis
from django.conf import settings


class HealthCheckView(APIView):
    """
    Health check endpoint for monitoring service health.
    Returns the status of the service and its dependencies.
    """
    permission_classes = [AllowAny]
    
    def get(self, request):
        """
        Check the health of the service and its dependencies.
        
        Returns:
            Response: JSON response with health status
        """
        health_status = {
            'success': True,
            'status': 'healthy',
            'service': '${input:project_name}',
            'checks': {}
        }
        
        # Check database connection
        try:
            with connection.cursor() as cursor:
                cursor.execute("SELECT 1")
            health_status['checks']['database'] = 'healthy'
        except Exception as e:
            health_status['checks']['database'] = 'unhealthy'
            health_status['status'] = 'unhealthy'
            health_status['success'] = False
        
        # Check cache (Redis) connection
        try:
            cache.set('health_check', 'ok', 1)
            if cache.get('health_check') == 'ok':
                health_status['checks']['cache'] = 'healthy'
            else:
                raise Exception("Cache test failed")
        except Exception as e:
            health_status['checks']['cache'] = 'unhealthy'
            health_status['status'] = 'degraded'
        
        # Check Redis directly if Celery is enabled
        if hasattr(settings, 'CELERY_BROKER_URL') and settings.CELERY_BROKER_URL:
            try:
                redis_client = redis.from_url(settings.CELERY_BROKER_URL)
                redis_client.ping()
                health_status['checks']['redis'] = 'healthy'
            except Exception as e:
                health_status['checks']['redis'] = 'unhealthy'
                health_status['status'] = 'degraded'
        
        status_code = status.HTTP_200_OK if health_status['success'] else status.HTTP_503_SERVICE_UNAVAILABLE
        return Response(health_status, status=status_code)
```

#### apps/authentication/__init__.py
```python
"""
Authentication app for handling user authentication and authorization.
Provides JWT authentication, user management, and permission handling.
"""
```

#### apps/authentication/apps.py
```python
"""
App configuration for the authentication app.
"""
from django.apps import AppConfig


class AuthenticationConfig(AppConfig):
    """Configuration for the authentication app."""
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'apps.authentication'
    verbose_name = 'Authentication'
    
    def ready(self):
        """Import signals when the app is ready."""
        try:
            import apps.authentication.signals  # noqa F401
        except ImportError:
            pass
```

#### apps/authentication/models.py
```python
"""
Authentication models.
Extends Django's built-in User model if needed.
"""
from django.contrib.auth.models import AbstractUser
from django.db import models
from apps.core.models import BaseModel


# Uncomment and customize if you need a custom User model
# class User(AbstractUser, BaseModel):
#     """
#     Custom User model with additional fields.
#     """
#     email = models.EmailField(unique=True)
#     phone_number = models.CharField(max_length=20, blank=True)
#     
#     USERNAME_FIELD = 'email'
#     REQUIRED_FIELDS = ['username']
#     
#     class Meta:
#         db_table = 'users'
#         verbose_name = 'User'
#         verbose_name_plural = 'Users'
```

#### apps/authentication/serializers.py
```python
"""
Authentication serializers for user registration, login, and profile management.
"""
from rest_framework import serializers
from django.contrib.auth import get_user_model
from django.contrib.auth.password_validation import validate_password
from rest_framework_simplejwt.serializers import TokenObtainPairSerializer

User = get_user_model()


class UserSerializer(serializers.ModelSerializer):
    """
    Serializer for User model.
    Used for user profile and listing.
    """
    
    class Meta:
        model = User
        fields = ['id', 'username', 'email', 'first_name', 'last_name', 'date_joined']
        read_only_fields = ['id', 'date_joined']


class UserRegistrationSerializer(serializers.ModelSerializer):
    """
    Serializer for user registration.
    Validates password and creates new user.
    """
    password = serializers.CharField(
        write_only=True,
        required=True,
        validators=[validate_password],
        style={'input_type': 'password'}
    )
    password_confirm = serializers.CharField(
        write_only=True,
        required=True,
        style={'input_type': 'password'}
    )
    
    class Meta:
        model = User
        fields = ['username', 'email', 'password', 'password_confirm', 'first_name', 'last_name']
        extra_kwargs = {
            'email': {'required': True},
            'first_name': {'required': True},
            'last_name': {'required': True},
        }
    
    def validate(self, attrs):
        """Validate that passwords match."""
        if attrs['password'] != attrs['password_confirm']:
            raise serializers.ValidationError({"password": "Password fields didn't match."})
        return attrs
    
    def create(self, validated_data):
        """Create new user with validated data."""
        validated_data.pop('password_confirm')
        user = User.objects.create_user(**validated_data)
        return user


class CustomTokenObtainPairSerializer(TokenObtainPairSerializer):
    """
    Custom JWT token serializer that includes user info in response.
    """
    
    def validate(self, attrs):
        data = super().validate(attrs)
        
        # Add custom user data to token response
        data['user'] = {
            'id': self.user.id,
            'username': self.user.username,
            'email': self.user.email,
            'first_name': self.user.first_name,
            'last_name': self.user.last_name,
        }
        
        return data


class PasswordChangeSerializer(serializers.Serializer):
    """
    Serializer for password change.
    """
    old_password = serializers.CharField(required=True, style={'input_type': 'password'})
    new_password = serializers.CharField(
        required=True,
        validators=[validate_password],
        style={'input_type': 'password'}
    )
    
    def validate_old_password(self, value):
        """Validate that old password is correct."""
        user = self.context['request'].user
        if not user.check_password(value):
            raise serializers.ValidationError("Old password is incorrect.")
        return value
```

#### apps/authentication/views.py
```python
"""
Authentication views for user registration, login, and profile management.
"""
from rest_framework import generics, status
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.permissions import AllowAny, IsAuthenticated
from django.contrib.auth import get_user_model
from rest_framework_simplejwt.views import TokenObtainPairView
from .serializers import (
    UserSerializer,
    UserRegistrationSerializer,
    CustomTokenObtainPairSerializer,
    PasswordChangeSerializer
)

User = get_user_model()


class UserRegistrationView(generics.CreateAPIView):
    """
    User registration endpoint.
    Creates a new user account.
    """
    queryset = User.objects.all()
    serializer_class = UserRegistrationSerializer
    permission_classes = [AllowAny]
    
    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        user = serializer.save()
        
        return Response({
            'success': True,
            'message': 'User registered successfully',
            'data': UserSerializer(user).data
        }, status=status.HTTP_201_CREATED)


class CustomTokenObtainPairView(TokenObtainPairView):
    """
    Custom login endpoint that returns JWT tokens and user info.
    """
    serializer_class = CustomTokenObtainPairSerializer


class UserProfileView(generics.RetrieveUpdateAPIView):
    """
    User profile endpoint.
    Get or update the authenticated user's profile.
    """
    serializer_class = UserSerializer
    permission_classes = [IsAuthenticated]
    
    def get_object(self):
        return self.request.user


class PasswordChangeView(APIView):
    """
    Password change endpoint.
    Allows authenticated users to change their password.
    """
    permission_classes = [IsAuthenticated]
    
    def post(self, request):
        serializer = PasswordChangeSerializer(
            data=request.data,
            context={'request': request}
        )
        serializer.is_valid(raise_exception=True)
        
        # Change password
        user = request.user
        user.set_password(serializer.validated_data['new_password'])
        user.save()
        
        return Response({
            'success': True,
            'message': 'Password changed successfully'
        }, status=status.HTTP_200_OK)
```

#### apps/authentication/urls.py
```python
"""
Authentication URL configuration.
"""
from django.urls import path
from rest_framework_simplejwt.views import TokenRefreshView
from .views import (
    UserRegistrationView,
    CustomTokenObtainPairView,
    UserProfileView,
    PasswordChangeView
)

app_name = 'authentication'

urlpatterns = [
    # Registration
    path('register/', UserRegistrationView.as_view(), name='register'),
    
    # Login/Token
    path('login/', CustomTokenObtainPairView.as_view(), name='login'),
    path('token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
    
    # Profile
    path('profile/', UserProfileView.as_view(), name='profile'),
    path('password/change/', PasswordChangeView.as_view(), name='password_change'),
]
```

#### apps/${input:app_name}/__init__.py
```python
"""
${input:app_name} app - Main business logic for the application.
This app contains the core functionality specific to this project.
"""
```

#### apps/${input:app_name}/apps.py
```python
"""
App configuration for the ${input:app_name} app.
"""
from django.apps import AppConfig


class ${input:app_name|title}Config(AppConfig):
    """Configuration for the ${input:app_name} app."""
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'apps.${input:app_name}'
    verbose_name = '${input:app_name|title}'
    
    def ready(self):
        """Import signals when the app is ready."""
        try:
            import apps.${input:app_name}.signals  # noqa F401
        except ImportError:
            pass
```

#### apps/${input:app_name}/models.py
```python
"""
Models for the ${input:app_name} app.
Define your domain models here following Django best practices.
"""
from django.db import models
from django.contrib.auth import get_user_model
from apps.core.models import BaseModel, ActiveManager
from apps.core.utils import generate_unique_slug

User = get_user_model()


class ExampleModel(BaseModel):
    """
    Example model showing common patterns.
    Replace this with your actual domain models.
    """
    # Fields
    name = models.CharField(max_length=255, help_text="Name of the item")
    slug = models.SlugField(unique=True, blank=True, help_text="URL-friendly version of name")
    description = models.TextField(blank=True, help_text="Detailed description")
    user = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='${input:app_name}_items',
        help_text="User who created this item"
    )
    
    # Status choices
    class StatusChoices(models.TextChoices):
        DRAFT = 'draft', 'Draft'
        PUBLISHED = 'published', 'Published'
        ARCHIVED = 'archived', 'Archived'
    
    status = models.CharField(
        max_length=20,
        choices=StatusChoices.choices,
        default=StatusChoices.DRAFT,
        help_text="Current status of the item"
    )
    
    # Managers
    objects = models.Manager()  # Default manager
    active_objects = ActiveManager()  # Returns only active records
    
    class Meta:
        db_table = '${input:app_name}_examples'
        verbose_name = 'Example'
        verbose_name_plural = 'Examples'
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['slug']),
            models.Index(fields=['user', 'status']),
        ]
    
    def __str__(self):
        return self.name
    
    def save(self, *args, **kwargs):
        """Generate unique slug on save if not provided."""
        if not self.slug:
            self.slug = generate_unique_slug(self.name, ExampleModel)
        super().save(*args, **kwargs)
    
    @property
    def is_published(self):
        """Check if the item is published."""
        return self.status == self.StatusChoices.PUBLISHED
```

#### apps/${input:app_name}/serializers.py
```python
"""
Serializers for the ${input:app_name} app.
Define your API serializers here for data validation and transformation.
"""
from rest_framework import serializers
from .models import ExampleModel


class ExampleListSerializer(serializers.ModelSerializer):
    """
    Lightweight serializer for listing items.
    Used in list views for better performance.
    """
    user_name = serializers.CharField(source='user.get_full_name', read_only=True)
    
    class Meta:
        model = ExampleModel
        fields = ['id', 'name', 'slug', 'status', 'user_name', 'created_at']
        read_only_fields = ['id', 'slug', 'created_at']


class ExampleDetailSerializer(serializers.ModelSerializer):
    """
    Detailed serializer for single item views.
    Includes all fields and related data.
    """
    user = serializers.SerializerMethodField()
    is_published = serializers.ReadOnlyField()
    
    class Meta:
        model = ExampleModel
        fields = [
            'id', 'name', 'slug', 'description', 'status',
            'user', 'is_published', 'created_at', 'updated_at'
        ]
        read_only_fields = ['id', 'slug', 'user', 'created_at', 'updated_at']
    
    def get_user(self, obj):
        """Return user information."""
        return {
            'id': obj.user.id,
            'username': obj.user.username,
            'full_name': obj.user.get_full_name()
        }


class ExampleCreateUpdateSerializer(serializers.ModelSerializer):
    """
    Serializer for creating and updating items.
    Handles validation and data transformation.
    """
    
    class Meta:
        model = ExampleModel
        fields = ['name', 'description', 'status']
    
    def validate_name(self, value):
        """Validate name field."""
        if len(value) < 3:
            raise serializers.ValidationError("Name must be at least 3 characters long.")
        return value
    
    def create(self, validated_data):
        """Create new item with current user."""
        validated_data['user'] = self.context['request'].user
        return super().create(validated_data)
```

#### apps/${input:app_name}/views.py
```python
"""
Views for the ${input:app_name} app.
Define your API views and viewsets here.
"""
from rest_framework import viewsets, status, filters
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from django_filters.rest_framework import DjangoFilterBackend
from apps.core.permissions import IsOwnerOrReadOnly
from .models import ExampleModel
from .serializers import (
    ExampleListSerializer,
    ExampleDetailSerializer,
    ExampleCreateUpdateSerializer
)


class ExampleViewSet(viewsets.ModelViewSet):
    """
    ViewSet for ExampleModel with CRUD operations.
    Includes filtering, searching, and custom actions.
    """
    permission_classes = [IsAuthenticated, IsOwnerOrReadOnly]
    filter_backends = [DjangoFilterBackend, filters.SearchFilter, filters.OrderingFilter]
    filterset_fields = ['status', 'user']
    search_fields = ['name', 'description']
    ordering_fields = ['created_at', 'name']
    ordering = ['-created_at']
    
    def get_queryset(self):
        """
        Get queryset based on user permissions.
        Non-staff users only see their own items.
        """
        queryset = ExampleModel.active_objects.select_related('user')
        
        if not self.request.user.is_staff:
            queryset = queryset.filter(user=self.request.user)
        
        return queryset
    
    def get_serializer_class(self):
        """Return appropriate serializer based on action."""
        if self.action == 'list':
            return ExampleListSerializer
        elif self.action in ['create', 'update', 'partial_update']:
            return ExampleCreateUpdateSerializer
        return ExampleDetailSerializer
    
    def perform_create(self, serializer):
        """Set the user when creating new item."""
        serializer.save(user=self.request.user)
    
    @action(detail=True, methods=['post'])
    def publish(self, request, pk=None):
        """
        Custom action to publish an item.
        Changes status to published.
        """
        item = self.get_object()
        
        if item.status == ExampleModel.StatusChoices.PUBLISHED:
            return Response({
                'success': False,
                'error': {
                    'code': 'already_published',
                    'message': 'Item is already published'
                }
            }, status=status.HTTP_400_BAD_REQUEST)
        
        item.status = ExampleModel.StatusChoices.PUBLISHED
        item.save(update_fields=['status', 'updated_at'])
        
        return Response({
            'success': True,
            'message': 'Item published successfully',
            'data': self.get_serializer(item).data
        })
    
    @action(detail=False, methods=['get'])
    def my_items(self, request):
        """
        Get items belonging to the current user.
        """
        queryset = self.get_queryset().filter(user=request.user)
        serializer = self.get_serializer(queryset, many=True)
        
        return Response({
            'success': True,
            'data': serializer.data
        })
```

#### apps/${input:app_name}/urls.py
```python
"""
URL configuration for the ${input:app_name} app.
"""
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import ExampleViewSet

app_name = '${input:app_name}'

router = DefaultRouter()
router.register(r'examples', ExampleViewSet, basename='example')

urlpatterns = [
    path('', include(router.urls)),
]
```

#### apps/${input:app_name}/admin.py
```python
"""
Admin configuration for the ${input:app_name} app.
Customize Django admin interface for your models.
"""
from django.contrib import admin
from .models import ExampleModel


@admin.register(ExampleModel)
class ExampleModelAdmin(admin.ModelAdmin):
    """Admin configuration for ExampleModel."""
    list_display = ['name', 'user', 'status', 'created_at', 'is_active']
    list_filter = ['status', 'is_active', 'created_at']
    search_fields = ['name', 'description', 'user__username']
    prepopulated_fields = {'slug': ('name',)}
    date_hierarchy = 'created_at'
    ordering = ['-created_at']
    
    fieldsets = (
        ('Basic Information', {
            'fields': ('name', 'slug', 'description', 'user')
        }),
        ('Status', {
            'fields': ('status', 'is_active')
        }),
        ('Timestamps', {
            'fields': ('created_at', 'updated_at'),
            'classes': ('collapse',)
        }),
    )
    
    readonly_fields = ['created_at', 'updated_at']
    
    def get_queryset(self, request):
        """Optimize queryset with select_related."""
        return super().get_queryset(request).select_related('user')
```

#### utils/__init__.py
```python
"""
Utility modules for the project.
This package contains reusable utilities, validators, decorators, and mixins.
"""
```

#### utils/validators.py
```python
"""
Custom validators for form fields and model fields.
"""
import re
from django.core.exceptions import ValidationError
from django.utils.translation import gettext_lazy as _


def validate_phone_number(value):
    """
    Validate phone number format.
    Accepts formats like: +1234567890, (123) 456-7890, 123-456-7890
    """
    phone_regex = re.compile(r'^\+?1?\d{9,15}$')
    if not phone_regex.match(value.replace('(', '').replace(')', '').replace('-', '').replace(' ', '')):
        raise ValidationError(
            _('Enter a valid phone number.'),
            code='invalid_phone'
        )


def validate_alphanumeric_with_spaces(value):
    """Validate that the value contains only alphanumeric characters and spaces."""
    if not re.match(r'^[a-zA-Z0-9\s]+$', value):
        raise ValidationError(
            _('This field can only contain letters, numbers, and spaces.'),
            code='invalid_alphanumeric'
        )


def validate_no_special_characters(value):
    """Validate that the value contains no special characters."""
    if re.search(r'[!@#$%^&*(),.?":{}|<>]', value):
        raise ValidationError(
            _('This field cannot contain special characters.'),
            code='no_special_chars'
        )


def validate_file_size(value):
    """Validate file size (max 5MB)."""
    filesize = value.size
    if filesize > 5 * 1024 * 1024:  # 5MB
        raise ValidationError(
            _('File size cannot exceed 5MB.'),
            code='file_too_large'
        )


def validate_image_extension(value):
    """Validate image file extension."""
    valid_extensions = ['.jpg', '.jpeg', '.png', '.gif', '.webp']
    if not any(value.name.lower().endswith(ext) for ext in valid_extensions):
        raise ValidationError(
            _('File must be a valid image format (JPG, PNG, GIF, WebP).'),
            code='invalid_image_extension'
        )
```

#### utils/decorators.py
```python
"""
Custom decorators for views and functions.
"""
import functools
import time
from django.core.cache import cache
from django.http import JsonResponse
from django.views.decorators.cache import cache_page
from django.utils.decorators import method_decorator


def rate_limit(requests_per_minute=60):
    """
    Rate limiting decorator.
    Limits the number of requests per minute from the same IP.
    """
    def decorator(view_func):
        @functools.wraps(view_func)
        def wrapper(request, *args, **kwargs):
            # Get client IP
            ip = request.META.get('HTTP_X_FORWARDED_FOR', '').split(',')[0] or \
                 request.META.get('REMOTE_ADDR', '')
            
            # Create cache key
            cache_key = f'rate_limit_{ip}_{view_func.__name__}'
            
            # Get current request count
            current_requests = cache.get(cache_key, 0)
            
            if current_requests >= requests_per_minute:
                return JsonResponse({
                    'success': False,
                    'error': {
                        'code': 'rate_limit_exceeded',
                        'message': 'Too many requests. Please try again later.'
                    }
                }, status=429)
            
            # Increment counter
            cache.set(cache_key, current_requests + 1, 60)  # 60 seconds
            
            return view_func(request, *args, **kwargs)
        return wrapper
    return decorator


def log_execution_time(func):
    """
    Decorator to log function execution time.
    """
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        
        print(f"{func.__name__} executed in {end_time - start_time:.4f} seconds")
        return result
    return wrapper


def require_ajax(view_func):
    """
    Decorator that ensures the request is made via AJAX.
    """
    @functools.wraps(view_func)
    def wrapper(request, *args, **kwargs):
        if not request.headers.get('X-Requested-With') == 'XMLHttpRequest':
            return JsonResponse({
                'success': False,
                'error': {
                    'code': 'ajax_required',
                    'message': 'This endpoint requires an AJAX request.'
                }
            }, status=400)
        return view_func(request, *args, **kwargs)
    return wrapper


def cache_view(timeout=300):
    """
    Class decorator for caching class-based views.
    """
    def decorator(cls):
        cls.dispatch = method_decorator(cache_page(timeout))(cls.dispatch)
        return cls
    return decorator
```

#### utils/pagination.py
```python
"""
Custom pagination classes for API responses.
"""
from rest_framework.pagination import PageNumberPagination
from rest_framework.response import Response
from collections import OrderedDict


class CustomPageNumberPagination(PageNumberPagination):
    """
    Custom pagination class with enhanced response format.
    """
    page_size = 20
    page_size_query_param = 'page_size'
    max_page_size = 100
    
    def get_paginated_response(self, data):
        """
        Return a paginated style Response object with additional metadata.
        """
        return Response(OrderedDict([
            ('success', True),
            ('count', self.page.paginator.count),
            ('total_pages', self.page.paginator.num_pages),
            ('current_page', self.page.number),
            ('page_size', self.page_size),
            ('next', self.get_next_link()),
            ('previous', self.get_previous_link()),
            ('results', data)
        ]))


class LargeResultsSetPagination(PageNumberPagination):
    """
    Pagination class for large datasets.
    """
    page_size = 50
    page_size_query_param = 'page_size'
    max_page_size = 200


class SmallResultsSetPagination(PageNumberPagination):
    """
    Pagination class for small datasets.
    """
    page_size = 10
    page_size_query_param = 'page_size'
    max_page_size = 50
```

#### utils/mixins.py
```python
"""
Custom mixins for views and models.
"""
from django.contrib.auth.mixins import LoginRequiredMixin
from django.core.exceptions import PermissionDenied
from django.http import JsonResponse
from rest_framework import status
from rest_framework.response import Response


class AjaxRequiredMixin:
    """
    Mixin that requires AJAX requests.
    """
    def dispatch(self, request, *args, **kwargs):
        if not request.headers.get('X-Requested-With') == 'XMLHttpRequest':
            return JsonResponse({
                'success': False,
                'error': {
                    'code': 'ajax_required',
                    'message': 'This endpoint requires an AJAX request.'
                }
            }, status=400)
        return super().dispatch(request, *args, **kwargs)


class StaffRequiredMixin(LoginRequiredMixin):
    """
    Mixin that requires staff privileges.
    """
    def dispatch(self, request, *args, **kwargs):
        if not request.user.is_staff:
            raise PermissionDenied("Staff privileges required.")
        return super().dispatch(request, *args, **kwargs)


class SuperUserRequiredMixin(LoginRequiredMixin):
    """
    Mixin that requires superuser privileges.
    """
    def dispatch(self, request, *args, **kwargs):
        if not request.user.is_superuser:
            raise PermissionDenied("Superuser privileges required.")
        return super().dispatch(request, *args, **kwargs)


class JSONResponseMixin:
    """
    Mixin for consistent JSON responses.
    """
    def json_response(self, data=None, success=True, message=None, status_code=200):
        """
        Return a JSON response with consistent format.
        """
        response_data = {
            'success': success,
        }
        
        if message:
            response_data['message'] = message
        
        if data is not None:
            response_data['data'] = data
        
        return JsonResponse(response_data, status=status_code)
    
    def json_error_response(self, message, code=None, status_code=400):
        """
        Return a JSON error response.
        """
        error_data = {'message': message}
        if code:
            error_data['code'] = code
        
        return JsonResponse({
            'success': False,
            'error': error_data
        }, status=status_code)


class CacheControlMixin:
    """
    Mixin for adding cache control headers.
    """
    cache_timeout = 300  # 5 minutes
    
    def dispatch(self, request, *args, **kwargs):
        response = super().dispatch(request, *args, **kwargs)
        response['Cache-Control'] = f'max-age={self.cache_timeout}'
        return response


class TimestampMixin:
    """
    Mixin for models that need created_at and updated_at fields.
    This is now replaced by BaseModel, but kept for backward compatibility.
    """
    pass


class SoftDeleteMixin:
    """
    Mixin for implementing soft delete functionality.
    This is now replaced by BaseModel, but kept for backward compatibility.
    """
    pass


class APIResponseMixin:
    """
    Mixin for DRF views to provide consistent API responses.
    """
    def success_response(self, data=None, message=None, status_code=status.HTTP_200_OK):
        """Return a success response."""
        response_data = {'success': True}
        
        if message:
            response_data['message'] = message
        
        if data is not None:
            response_data['data'] = data
        
        return Response(response_data, status=status_code)
    
    def error_response(self, message, code=None, status_code=status.HTTP_400_BAD_REQUEST):
        """Return an error response."""
        error_data = {'message': message}
        if code:
            error_data['code'] = code
        
        return Response({
            'success': False,
            'error': error_data
        }, status=status_code)
```

### 6. Testing Setup

#### tests/conftest.py
```python
"""
Global pytest configuration and fixtures.
Provides common test utilities and fixtures for all tests.
"""
import pytest
from rest_framework.test import APIClient
from django.contrib.auth import get_user_model

User = get_user_model()


@pytest.fixture
def api_client():
    """Provide an API client for tests."""
    return APIClient()


@pytest.fixture
def user(db):
    """Create a test user."""
    return User.objects.create_user(
        username='testuser',
        email='test@example.com',
        password='testpass123',
        first_name='Test',
        last_name='User'
    )


@pytest.fixture
def authenticated_client(api_client, user):
    """Provide an authenticated API client."""
    api_client.force_authenticate(user=user)
    return api_client


@pytest.fixture
def admin_user(db):
    """Create an admin user."""
    return User.objects.create_superuser(
        username='admin',
        email='admin@example.com',
        password='adminpass123'
    )


@pytest.fixture
def admin_client(api_client, admin_user):
    """Provide an authenticated admin API client."""
    api_client.force_authenticate(user=admin_user)
    return api_client
```

#### tests/factories/__init__.py
```python
"""
Factory Boy factories for generating test data.
Provides realistic test data for models.
"""
import factory
from django.contrib.auth import get_user_model
from apps.${input:app_name}.models import ExampleModel

User = get_user_model()


class UserFactory(factory.django.DjangoModelFactory):
    """Factory for creating test users."""
    
    class Meta:
        model = User
    
    username = factory.Sequence(lambda n: f'user{n}')
    email = factory.LazyAttribute(lambda obj: f'{obj.username}@example.com')
    first_name = factory.Faker('first_name')
    last_name = factory.Faker('last_name')
    
    @factory.post_generation
    def password(obj, create, extracted, **kwargs):
        if not create:
            return
        
        password = extracted or 'defaultpass123'
        obj.set_password(password)
        obj.save()


class ExampleModelFactory(factory.django.DjangoModelFactory):
    """Factory for creating test ExampleModel instances."""
    
    class Meta:
        model = ExampleModel
    
    name = factory.Faker('sentence', nb_words=3)
    description = factory.Faker('text')
    user = factory.SubFactory(UserFactory)
    status = ExampleModel.StatusChoices.DRAFT
```

### 7. Docker Configuration (if ${input:use_docker} is true)

#### Dockerfile
```dockerfile
FROM python:3.11-slim

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Set work directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements/ requirements/
RUN pip install --upgrade pip
RUN pip install -r requirements/production.txt

# Copy project
COPY . .

# Collect static files
RUN python manage.py collectstatic --noinput

# Run gunicorn
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "config.wsgi:application"]
```

#### docker-compose.yml
```yaml
version: '3.8'

services:
  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/app
    ports:
      - "8000:8000"
    env_file:
      - .env
    depends_on:
      - db
      - redis

  db:
    image: postgres:14
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=${DB_NAME}
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    ports:
      - "5432:5432"

  redis:
    image: redis:7
    ports:
      - "6379:6379"

  celery:
    build: .
    command: celery -A config worker -l info
    volumes:
      - .:/app
    env_file:
      - .env
    depends_on:
      - db
      - redis

  celery-beat:
    build: .
    command: celery -A config beat -l info
    volumes:
      - .:/app
    env_file:
      - .env
    depends_on:
      - db
      - redis

volumes:
  postgres_data:
```

### 8. Documentation

#### README.md
```markdown
# ${input:project_name}

A production-ready Django REST Framework project with modern Python practices.

## Features

- Django 4.2+ with Django REST Framework
- JWT Authentication
- API Documentation with Swagger/ReDoc
- Comprehensive testing with pytest
- Docker support
- Celery integration (optional)
- Health check endpoint

## Quick Start

### Prerequisites

- Python 3.11+
- PostgreSQL/MySQL (or SQLite for development)
- Redis (for caching and Celery)

### Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd ${input:project_name}
```

2. Create virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install dependencies:
```bash
pip install -r requirements/development.txt
```

4. Set up environment variables:
```bash
cp .env.example .env
# Edit .env with your configuration
```

5. Run migrations:
```bash
python manage.py migrate
```

6. Create superuser:
```bash
python manage.py createsuperuser
```

7. Run development server:
```bash
python manage.py runserver
```

## API Documentation

API documentation is available at:
- Swagger UI: http://localhost:8000/api/v1/swagger/
- ReDoc: http://localhost:8000/api/v1/redoc/

## Health Check

Health check endpoint: http://localhost:8000/api/v1/health/

## Testing

Run tests with pytest:
```bash
pytest
```

Run with coverage:
```bash
pytest --cov=apps --cov-report=html
```

## Docker

Build and run with Docker Compose:
```bash
docker-compose up --build
```

## Project Structure

```
${input:project_name}/
├── apps/                   # Django applications
│   ├── core/              # Base models, utilities, middleware
│   ├── authentication/    # User authentication
│   ├── api/              # API routing and documentation
│   └── ${input:app_name}/         # Main business logic
├── config/                # Django configuration
├── tests/                 # Test files
├── requirements/          # Dependencies
└── docs/                  # Documentation
```

## Environment Variables

See `.env.example` for required environment variables.

## Contributing

1. Create a feature branch
2. Make your changes
3. Run tests and linting
4. Submit a pull request

## License

[Your License]
```

#### .env.example
```
# Django settings
DEBUG=True
SECRET_KEY=your-secret-key-here
ALLOWED_HOSTS=localhost,127.0.0.1

# Database
DB_NAME=${input:project_name}_db
DB_USER=${input:project_name}_user
DB_PASSWORD=your-db-password
DB_HOST=localhost
DB_PORT=5432

# Redis
REDIS_URL=redis://localhost:6379/0

# JWT
JWT_SECRET_KEY=your-jwt-secret-key

# CORS
CORS_ALLOWED_ORIGINS=http://localhost:3000,http://localhost:8080
CORS_ALLOW_ALL_ORIGINS=False

# Celery (if enabled)
CELERY_BROKER_URL=redis://localhost:6379/0
CELERY_RESULT_BACKEND=redis://localhost:6379/0

# Logging
LOG_LEVEL=INFO
DJANGO_LOG_LEVEL=INFO
```

### 9. Code Quality Setup

#### .editorconfig
```ini
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.{py,pyi}]
indent_style = space
indent_size = 4
max_line_length = 88

[*.{yml,yaml,json}]
indent_style = space
indent_size = 2

[*.md]
trim_trailing_whitespace = false
```

#### .gitignore
```
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
ENV/
.venv

# Django
*.log
*.pot
*.pyc
db.sqlite3
media/
staticfiles/

# IDE
.vscode/
.idea/
*.swp
*.swo

# Testing
.coverage
htmlcov/
.pytest_cache/

# Environment
.env
.env.local

# OS
.DS_Store
Thumbs.db

# Docker
docker-compose.override.yml
```

### 10. Verify and Complete Folder Structure

Before proceeding with setup commands, verify that all required directories exist to avoid errors:

```bash
# Create any missing directories in the structure
mkdir -p apps/core/migrations
mkdir -p apps/core/management/commands
mkdir -p apps/core/templatetags
mkdir -p apps/authentication/migrations
mkdir -p apps/authentication/management/commands
mkdir -p apps/api/migrations
mkdir -p apps/${input:app_name}/migrations
mkdir -p apps/${input:app_name}/management/commands
mkdir -p apps/${input:app_name}/templates/${input:app_name}
mkdir -p apps/${input:app_name}/static/${input:app_name}
mkdir -p config/settings
mkdir -p tests/unit/core
mkdir -p tests/unit/authentication
mkdir -p tests/unit/api
mkdir -p tests/unit/${input:app_name}
mkdir -p tests/integration/core
mkdir -p tests/integration/authentication
mkdir -p tests/integration/api
mkdir -p tests/integration/${input:app_name}
mkdir -p tests/fixtures
mkdir -p tests/mocks
mkdir -p utils
mkdir -p scripts
mkdir -p docs/api
mkdir -p docs/deployment
mkdir -p static/css
mkdir -p static/js
mkdir -p static/img
mkdir -p media/uploads
mkdir -p logs
mkdir -p locale

# Create __init__.py files for Python packages
touch apps/__init__.py
touch apps/core/__init__.py
touch apps/core/migrations/__init__.py
touch apps/core/management/__init__.py
touch apps/core/management/commands/__init__.py
touch apps/core/templatetags/__init__.py
touch apps/authentication/__init__.py
touch apps/authentication/migrations/__init__.py
touch apps/authentication/management/__init__.py
touch apps/authentication/management/commands/__init__.py
touch apps/api/__init__.py
touch apps/api/migrations/__init__.py
touch apps/${input:app_name}/__init__.py
touch apps/${input:app_name}/migrations/__init__.py
touch apps/${input:app_name}/management/__init__.py
touch apps/${input:app_name}/management/commands/__init__.py
touch config/__init__.py
touch config/settings/__init__.py
touch utils/__init__.py
touch tests/__init__.py
touch tests/unit/__init__.py
touch tests/unit/core/__init__.py
touch tests/unit/authentication/__init__.py
touch tests/unit/api/__init__.py
touch tests/unit/${input:app_name}/__init__.py
touch tests/integration/__init__.py
touch tests/integration/core/__init__.py
touch tests/integration/authentication/__init__.py
touch tests/integration/api/__init__.py
touch tests/integration/${input:app_name}/__init__.py
touch tests/factories/__init__.py
touch tests/fixtures/__init__.py
touch tests/mocks/__init__.py

# Create additional utility files
touch utils/validators.py
touch utils/decorators.py
touch utils/pagination.py
touch utils/mixins.py

# Create app configuration files
touch apps/core/apps.py
touch apps/authentication/apps.py
touch apps/api/apps.py
touch apps/${input:app_name}/apps.py

# Create additional test files
touch tests/test_settings.py
touch tests/test_urls.py

# Create deployment related files
touch scripts/deploy.sh
touch scripts/backup.sh
touch scripts/migrate.sh

# Create documentation files
touch docs/API.md
touch docs/DEPLOYMENT.md
touch docs/CONTRIBUTING.md
touch docs/ARCHITECTURE.md

# Set proper permissions for scripts
chmod +x scripts/*.sh

# Create .keep files for empty directories that need to be tracked by git
touch static/.keep
touch media/.keep
touch logs/.keep
touch locale/.keep
```

#### Verify Critical Files Exist

```bash
# Verify all critical configuration files exist
echo "Checking critical files..."
for file in manage.py .env.example .gitignore .editorconfig pyproject.toml README.md; do
    if [ ! -f "$file" ]; then
        echo "WARNING: Missing $file - please create it"
    fi
done

# Verify all requirements files exist
for file in requirements/base.txt requirements/development.txt requirements/production.txt requirements/testing.txt; do
    if [ ! -f "$file" ]; then
        echo "WARNING: Missing $file - please create it"
    fi
done

echo "Folder structure verification complete!"
```

### 11. Final Setup Commands

Execute final setup:

```bash
# Create and run migrations
python manage.py makemigrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser

# Collect static files (for production)
python manage.py collectstatic --noinput

# Run tests
pytest

# Check code quality
black --check .
isort --check-only .
flake8 .
mypy .
```

## Expected Output

A fully configured Django REST Framework project with:

✅ Modern Python project structure with meaningful code examples
✅ Production-ready Django configuration  
✅ JWT authentication setup
✅ Health check endpoint at /api/v1/health
✅ API documentation with drf-yasg
✅ Base models, middleware, exceptions, and utilities
✅ Comprehensive testing suite
✅ Code quality tools configured
✅ Docker support (if requested)
✅ Celery integration (if requested)
✅ Proper documentation

## Next Steps

1. Customize the `${input:app_name}` models for your specific business logic
2. Implement additional API endpoints following the established patterns
3. Add comprehensive tests for your business logic
4. Configure production deployment
5. Set up monitoring and logging
6. Implement additional security measures as needed

**Note**: All app folders now contain meaningful Python files with example code and comprehensive documentation. The project is ready for immediate development with a working health check endpoint at `/api/v1/health/`.