# Developer Run Manual

This manual provides step-by-step instructions for developers to set up and run Django REST Framework projects created with our initialization prompt.

## Prerequisites

Before starting, ensure you have:
- Python 3.8+ installed
- Git installed
- Access to the project repository
- Basic familiarity with command line operations

## Quick Start Guide

### 1. Clone and Navigate to Project

```bash
# Clone the repository
git clone <repository-url>

# Navigate to project directory
cd <project-name>
```

### 2. Virtual Environment Setup

**Choose the appropriate command for your operating system:**

#### Linux/macOS
```bash
# Create virtual environment
python3 -m venv venv

# Activate virtual environment
source venv/bin/activate

# Verify activation (should show project path)
which python
```

#### Windows Command Prompt
```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
venv\Scripts\activate.bat

# Verify activation
where python
```

#### Windows PowerShell
```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
venv\Scripts\Activate.ps1

# If execution policy error occurs:
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Then retry activation
venv\Scripts\Activate.ps1
```

### 3. Install Dependencies

```bash
# Upgrade pip first
pip install --upgrade pip

# Install development dependencies
pip install -r requirements/development.txt

# Verify installation
pip list
```

### 4. Environment Configuration

```bash
# Copy environment template
cp .env.example .env

# Edit .env file with your settings
# On Linux/macOS: nano .env
# On Windows: notepad .env
```

**Required environment variables:**
```env
DEBUG=True
SECRET_KEY=your-secret-key-here-minimum-50-characters-long
ALLOWED_HOSTS=localhost,127.0.0.1
DATABASE_URL=sqlite:///db.sqlite3
CORS_ALLOWED_ORIGINS=http://localhost:3000,http://localhost:8080
```

### 5. Database Setup

```bash
# Run system checks
python manage.py check

# Create migrations
python manage.py makemigrations

# Apply migrations
python manage.py migrate

# Create superuser (optional)
python manage.py createsuperuser
```

### 6. Launch Development Server

```bash
# Start the server
python manage.py runserver

# Server should start at: http://127.0.0.1:8000/
```

### 7. Verify Installation

Test these endpoints in your browser:

| Endpoint | URL | Expected Result |
|----------|-----|-----------------|
| **Main Site** | http://127.0.0.1:8000/ | Django welcome page |
| **Admin Panel** | http://127.0.0.1:8000/admin/ | Django admin login |
| **Health Check** | http://127.0.0.1:8000/api/v1/health/ | JSON health status |
| **API Docs** | http://127.0.0.1:8000/api/v1/docs/ | API endpoint list |
| **Debug Toolbar** | Any page (development only) | Debug toolbar on right |

## Troubleshooting Guide

### Common Errors and Solutions

#### Error: "No module named 'apps.api.urls'"

**Cause**: Missing URL configuration file
**Solution**:
```bash
# Create the missing file
touch apps/api/urls.py

# Add basic content:
cat > apps/api/urls.py << 'EOF'
from django.urls import path
from django.http import JsonResponse

def health_check(request):
    return JsonResponse({'status': 'healthy'})

urlpatterns = [
    path('health/', health_check, name='health_check'),
]
EOF
```

#### Error: "djdt namespace not found"

**Cause**: Debug toolbar URLs not properly configured
**Solution**:
```bash
# Edit config/urls.py and ensure debug toolbar URLs are conditional:
# if settings.DEBUG and 'debug_toolbar' in settings.INSTALLED_APPS:
#     import debug_toolbar
#     urlpatterns = [path('__debug__/', include(debug_toolbar.urls))] + urlpatterns
```

#### Error: "SessionMiddleware must be in MIDDLEWARE"

**Cause**: Incorrect middleware order
**Solution**:
```bash
# Edit config/settings/base.py
# Ensure SessionMiddleware comes BEFORE AuthenticationMiddleware
```

#### Error: "Directory 'static' does not exist"

**Cause**: Missing static files directory
**Solution**:
```bash
mkdir -p static
mkdir -p media
```

#### Error: "ModuleNotFoundError: No module named 'decouple'"

**Cause**: Missing dependencies
**Solution**:
```bash
pip install python-decouple
# or
pip install -r requirements/development.txt
```

#### Error: Virtual environment activation fails on Windows

**Cause**: PowerShell execution policy
**Solution**:
```powershell
# Run as Administrator
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope LocalMachine

# Or for current user only
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### Database Issues

#### Error: "no such table" errors

**Solution**:
```bash
# Reset migrations
python manage.py migrate --run-syncdb

# Or reset database (development only)
rm db.sqlite3
python manage.py migrate
```

#### Error: Migration conflicts

**Solution**:
```bash
# Reset migrations for specific app
python manage.py migrate <app_name> zero
python manage.py makemigrations <app_name>
python manage.py migrate
```

## Development Workflow

### Daily Development Routine

```bash
# 1. Activate virtual environment
source venv/bin/activate  # Linux/macOS
# venv\Scripts\activate.bat  # Windows

# 2. Pull latest changes
git pull origin main

# 3. Install any new dependencies
pip install -r requirements/development.txt

# 4. Run migrations
python manage.py migrate

# 5. Start development server
python manage.py runserver
```

### Code Quality Checks

```bash
# Run Django system checks
python manage.py check

# Run tests
python manage.py test

# Code formatting (if configured)
black .
isort .

# Linting (if configured)
flake8 .
```

### Adding New Features

1. **Create new app** (if needed):
   ```bash
   python manage.py startapp <app_name> apps/<app_name>
   ```

2. **Add to INSTALLED_APPS** in `config/settings/base.py`

3. **Create models, views, URLs** following existing patterns

4. **Create and run migrations**:
   ```bash
   python manage.py makemigrations
   python manage.py migrate
   ```

5. **Test thoroughly** before committing

## Project Structure Reference

```
project_name/
├── apps/                    # Django applications
│   ├── api/                 # API routing and endpoints
│   ├── authentication/     # User authentication
│   ├── core/               # Core functionality
│   └── <app_name>/         # Main business logic
├── config/                 # Django configuration
│   ├── settings/           # Environment-specific settings
│   ├── urls.py            # Main URL configuration
│   ├── wsgi.py            # WSGI configuration
│   └── asgi.py            # ASGI configuration
├── tests/                  # Test files
├── requirements/           # Dependency files
├── static/                 # Static files
├── docs/                   # Documentation
├── utils/                  # Utility functions
├── manage.py              # Django management script
└── .env                   # Environment variables
```

## Production Deployment Notes

### Environment Variables for Production

```env
DEBUG=False
SECRET_KEY=<strong-production-secret-key>
ALLOWED_HOSTS=yourdomain.com,www.yourdomain.com
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
CORS_ALLOWED_ORIGINS=https://yourdomain.com
```

### Production Checklist

- [ ] Set `DEBUG=False`
- [ ] Use strong `SECRET_KEY`
- [ ] Configure proper database (PostgreSQL/MySQL)
- [ ] Set up static file serving
- [ ] Configure logging
- [ ] Set up SSL/HTTPS
- [ ] Configure allowed hosts
- [ ] Set up monitoring and error tracking

## Getting Help

### Resources

- **Django Documentation**: https://docs.djangoproject.com/
- **Django REST Framework**: https://www.django-rest-framework.org/
- **Project Issues**: Check `.github/issues/` directory for known issues

### Support Contacts

- **Technical Issues**: Create issue in project repository
- **Development Questions**: Consult team lead or senior developer
- **Infrastructure**: Contact DevOps team

## Frequently Asked Questions

**Q: Why do I get permission errors on Windows?**
A: Run Command Prompt or PowerShell as Administrator, or adjust execution policy.

**Q: Can I use different Python versions?**
A: Project supports Python 3.8+. Use `python --version` to check your version.

**Q: How do I reset my local database?**
A: Delete `db.sqlite3` file and run migrations again (development only).

**Q: Where are the log files?**
A: Check `logs/` directory or console output in development.

**Q: How do I add new API endpoints?**
A: Add views to relevant app, create URL patterns, and update API documentation.

---

**Last Updated**: 2025-01-24  
**Version**: 1.0  
**Maintainer**: Development Team