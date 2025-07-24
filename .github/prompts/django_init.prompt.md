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
├── .github/
│   ├── workflows/
│   │   ├── ci.yml
│   │   └── deploy.yml
│   └── instructions/
│       ├── copilot.instructions.md
│       ├── django.instructions.md
│       ├── python.instructions.md
│       └── testing.instructions.md
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
Reference: file:python.instructions.md for proper Python project configuration

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

Configure Django settings following file:django.instructions.md patterns:

#### config/settings/base.py
- Include all required Django and DRF settings
- Configure database for `${input:database_type}`
- Set up JWT authentication
- Configure CORS settings
- Add drf-yasg for API documentation
- Include Celery configuration if `${input:include_celery}` is true

#### config/settings/development.py
- Enable DEBUG mode
- Configure Django Debug Toolbar
- Set development-specific database settings

#### config/settings/production.py
- Security settings for production
- Configure static files handling
- Set production database settings
- Configure logging

### 5. Core Django Apps Setup

#### apps/core/
Create base models, utilities, and shared components:
- BaseModel with created_at, updated_at, is_active fields
- Custom model managers
- Utility functions
- Common exceptions

#### apps/authentication/
Set up user authentication:
- Custom User model (if needed)
- JWT authentication views
- User serializers
- Permission classes

#### apps/api/
Configure API routing and versioning:
- Root URL configuration
- API versioning setup
- OpenAPI documentation setup

#### apps/${input:app_name}/
Create main application:
- Models for business logic
- Serializers for API
- ViewSets and views
- URL routing
- Permissions

### 6. Testing Setup

Configure comprehensive testing following file:testing.instructions.md:

#### tests/conftest.py
- Global pytest fixtures
- Database configuration
- API client setup
- Authentication fixtures

#### tests/factories/
- Factory Boy factories for test data
- User factory
- Model factories for ${input:app_name}

#### tests/unit/
- Model tests
- Serializer tests
- View tests
- Utility tests

#### tests/integration/
- API integration tests
- Database integration tests
- Workflow tests

### 7. Docker Configuration (if ${input:use_docker} is true)

Create Docker configuration:

#### Dockerfile
```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements/ requirements/
RUN pip install -r requirements/production.txt

COPY . .

EXPOSE 8000

CMD ["gunicorn", "config.wsgi:application", "--bind", "0.0.0.0:8000"]
```

#### docker-compose.yml
Include services for:
- Django application
- Database (${input:database_type})
- Redis (if ${input:include_celery} is true)
- Celery worker (if ${input:include_celery} is true)

### 8. GitHub Integration

#### .github/workflows/ci.yml
Create CI pipeline with:
- Python version matrix testing
- Code quality checks (black, isort, flake8, mypy)
- Test execution with coverage
- Security scanning

#### .github/workflows/deploy.yml
Create deployment pipeline for:
- Docker image building
- Production deployment
- Database migrations

### 9. Documentation

#### README.md
Create comprehensive documentation including:
- Project overview
- Setup instructions
- Environment variables
- API documentation links
- Development workflow
- Deployment instructions

#### .env.example
Template for environment variables:
```
DEBUG=True
SECRET_KEY=your-secret-key-here
DATABASE_URL=${input:database_type}://user:pass@localhost/dbname
REDIS_URL=redis://localhost:6379/0
JWT_SECRET_KEY=your-jwt-secret-here
```

### 10. Code Quality Setup

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
```

#### .gitignore
Include Python, Django, and IDE-specific ignores

### 11. Final Setup Commands

Execute final setup:

```bash
# Create and run migrations
python manage.py makemigrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser

# Collect static files (production)
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

✅ Modern Python project structure
✅ Production-ready Django configuration  
✅ JWT authentication setup
✅ API documentation with drf-yasg
✅ Comprehensive testing suite
✅ Code quality tools configured
✅ Docker support (if requested)
✅ Celery integration (if requested)
✅ CI/CD pipelines
✅ Proper documentation

## Instructions Reference

This initialization follows the patterns and best practices defined in:
- file:.github/instructions/copilot.instructions.md
- file:.github/instructions/django.instructions.md  
- file:.github/instructions/python.instructions.md
- file:.github/instructions/testing.instructions.md

## Next Steps

1. Customize the `${input:app_name}` models for your specific business logic
2. Implement API endpoints following the established patterns
3. Add comprehensive tests for your business logic
4. Configure production deployment
5. Set up monitoring and logging
6. Implement additional security measures as needed

**Note**: Ensure all environment variables are properly configured before running the application. Review the .env.example file and create your local .env file with appropriate values for `${input:database_type}` and other services.