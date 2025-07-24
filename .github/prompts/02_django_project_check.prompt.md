---
description: Check if existing Django project can run successfully and fix any errors until health API is accessible
---

# Django Project Health Check and Fix

This prompt helps you verify if an existing Django REST Framework project can run properly and automatically fixes common issues until the health API endpoint is accessible.

## Project Verification Workflow

Welcome! This tool will help you check and fix an existing Django project to ensure it runs successfully.

## Step 1: System Environment Analysis

**Let's first analyze your system environment before proceeding with the Django project check.**

### 1.1 Current Directory and Operating System Check

**Check current working directory:**
```bash
# Show current directory
pwd

# List contents of current directory
ls -la  # Linux/macOS
dir     # Windows
```

**Detect operating system:**
```bash
# Linux/macOS
uname -s

# Windows (PowerShell)
$PSVersionTable.OS

# Windows (Command Prompt)
echo %OS%
```

**Expected Results:**
- **Linux**: `Linux`
- **macOS**: `Darwin` 
- **Windows**: `Windows_NT` or Windows version info

### 1.2 Python Installation Check

**Check if Python is installed:**

```bash
# Method 1: Check Python 3
python3 --version
python --version

# Method 2: Check Python installation path
which python3    # Linux/macOS
which python     # Linux/macOS
where python     # Windows
```

**Expected Results:**
- ✅ **Python Found**: Shows version (e.g., `Python 3.9.7`)
- ❌ **Python Not Found**: Command not found error

### 1.3 Conda Environment Check

**If Python is not found, check for Conda:**

```bash
# Check if conda is installed
conda --version
conda info

# Check current conda environment
conda info --envs
conda env list
```

**Conda Environment Analysis:**

**If Conda is available:**
```bash
# List all conda environments
conda env list

# Check if project-specific environment exists
# Look for environment named: ${input:project_name} or similar
```

**Expected Conda Results:**
- ✅ **Conda Available**: Shows conda version and environment list
- ❌ **Conda Not Available**: Command not found

### 1.4 System Requirements Resolution

**Based on the checks above, follow the appropriate path:**

#### Path A: Python Found ✅
```bash
echo "✅ Python is available"
python --version
# Proceed to Step 2
```

#### Path B: Conda Available, No Python ⚠️
```bash
echo "⚠️ Conda found, checking for project environment..."

# Check if project environment exists
conda env list | grep ${input:project_name}

# If project environment exists:
echo "✅ Found existing conda environment: ${input:project_name}"
conda activate ${input:project_name}

# If no project environment exists:
echo "Creating new conda environment for ${input:project_name}"
conda create -n ${input:project_name} python=3.9 -y
conda activate ${input:project_name}
```

#### Path C: Neither Python nor Conda Available ❌

**STOP - Installation Required**

```bash
echo "❌ Neither Python nor Conda is installed on this system"
echo "Please install one of the following before proceeding:"
```

**Installation Instructions:**

**For Linux (Ubuntu/Debian):**
```bash
# Install Python
sudo apt update
sudo apt install python3 python3-pip python3-venv

# Or install Miniconda
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```

**For macOS:**
```bash
# Install Python via Homebrew
brew install python3

# Or install Miniconda
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-x86_64.sh
bash Miniconda3-latest-MacOSX-x86_64.sh
```

**For Windows:**
```powershell
# Install Python from python.org
# Download from: https://www.python.org/downloads/windows/

# Or install Miniconda
# Download from: https://docs.conda.io/en/latest/miniconda.html
```

**⚠️ IMPORTANT: After installing Python or Conda, restart your terminal and run this check again.**

### 1.5 Environment Verification Summary

**System Check Results:**

| Component | Status | Action Required |
|-----------|--------|----------------|
| **Operating System** | `[Detected OS]` | None |
| **Current Directory** | `[Current Path]` | Navigate if needed |
| **Python Installation** | ✅/❌ | Install if missing |
| **Conda Installation** | ✅/❌ | Alternative to Python |
| **Project Environment** | ✅/❌ | Create/Activate |

**Proceed only when:**
- ✅ Python OR Conda is available
- ✅ Project environment is active (if using Conda)
- ✅ Current directory is confirmed

---

## Step 2: Project Location and Structure Assessment

**Now that the system environment is verified, let's locate and check your Django project.**

**First, let me ask: Do you need to check if your Django project folder exists and is properly structured?**

**User Response Options:**
- ✅ **Yes** - I want to verify the project folder structure first
- ❌ **No** - I know the project exists, just help me get it running

---

## Option A: If User Chooses "YES" - Full Project Structure Check

### 2.1 Verify Project Directory Structure

**Please provide your project name:**
- Project Name: `${input:project_name}`

**Checking project structure:**

```bash
# Navigate to project directory
cd ${input:project_name}

# Verify project structure exists
echo "Checking project structure..."
```

**Required directory structure verification:**

```
${input:project_name}/
├── apps/
│   ├── api/
│   ├── authentication/ 
│   ├── core/
│   └── <main_app>/
├── config/
│   ├── settings/
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── requirements/
├── manage.py
└── README.md (optional)
```

**Structure Check Results:**
- ✅ **Complete**: All required directories exist → Continue to Step 2
- ⚠️ **Incomplete**: Missing directories → Follow repair instructions below
- ❌ **Not Found**: Project directory doesn't exist → Use Project Init prompt first

### 2.2 Fix Missing Structure (if needed)

**If structure is incomplete, create missing directories:**

```bash
# Create missing directories
mkdir -p apps/api apps/authentication apps/core
mkdir -p config/settings
mkdir -p requirements tests static docs

# Create missing __init__.py files
touch apps/__init__.py
touch apps/api/__init__.py apps/authentication/__init__.py apps/core/__init__.py
```

---

## Step 3: Environment Setup and Dependency Check

### 3.1 Navigate to Project Directory

```bash
# Change to project directory
cd ${input:project_name}

# Verify we're in the right location
pwd
ls -la
```

### 3.2 Virtual Environment Setup (OS-Specific)

**Note: Skip this section if using Conda (environment already activated in Step 1)**

**For Linux/macOS:**
```bash
# Create virtual environment
python3 -m venv venv

# Activate virtual environment
source venv/bin/activate

# Verify activation
which python
```

**For Windows (Command Prompt):**
```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
venv\Scripts\activate.bat

# Verify activation
where python
```

**For Windows (PowerShell):**
```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment (handle execution policy if needed)
venv\Scripts\Activate.ps1

# If execution policy error occurs:
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
venv\Scripts\Activate.ps1
```

**Expected Result:** Virtual environment should be active (prompt shows `(venv)`)

### 3.3 Install Dependencies

```bash
# Upgrade pip first
pip install --upgrade pip

# Install dependencies (try multiple sources)
# Method 1: Try development requirements
pip install -r requirements/development.txt

# Method 2: If above fails, try base requirements
pip install -r requirements/base.txt

# Method 3: If requirements files missing, install essential packages
pip install Django djangorestframework python-decouple django-cors-headers
```

**Dependency Installation Check:**
```bash
# Verify Django installation
python -c "import django; print(f'Django version: {django.get_version()}')"

# Verify DRF installation  
python -c "import rest_framework; print('DRF installed successfully')"
```

## Step 4: Django Project Health Verification

### 4.1 Initial Django System Check

```bash
# Run Django system checks
python manage.py check

# Expected output: "System check identified no issues (0 silenced)."
```

**If errors occur, proceed to Error Fixing section below.**

### 4.2 Database Migration Check

```bash
# Check migration status
python manage.py showmigrations

# Create missing migrations
python manage.py makemigrations

# Apply migrations
python manage.py migrate
```

### 4.3 Development Server Launch Test

```bash
# Start development server
python manage.py runserver

# Expected output:
# "Starting development server at http://127.0.0.1:8000/"
```

**Keep server running and test in new terminal/browser:**

### 4.4 Health API Endpoint Verification

**Test the following URLs:**

1. **Health Check API**: http://127.0.0.1:8000/api/v1/health/
   - Expected: JSON response with `{"status": "healthy"}`

2. **Admin Panel**: http://127.0.0.1:8000/admin/
   - Expected: Django admin login page

3. **Main Site**: http://127.0.0.1:8000/
   - Expected: Django welcome page or custom homepage

**Success Criteria:** Health API returns valid JSON response

---

## Step 5: Common Error Fixing Workflow

**If any step fails, follow this systematic error fixing approach:**

### Error Type 1: "No module named 'apps.api.urls'"

**Fix:**
```bash
# Create missing apps/api/urls.py
cat > apps/api/urls.py << 'EOF'
"""API URL configuration"""
from django.urls import path
from django.http import JsonResponse

def health_check(request):
    return JsonResponse({
        'status': 'healthy',
        'service': '${input:project_name}',
        'timestamp': '2024-01-01T00:00:00Z'
    })

def api_root(request):
    return JsonResponse({
        'message': 'API Root',
        'endpoints': {
            'health': '/api/v1/health/',
            'admin': '/admin/'
        }
    })

urlpatterns = [
    path('health/', health_check, name='health_check'),
    path('', api_root, name='api_root'),
]
EOF
```

### Error Type 2: "No module named 'apps.core.middleware'"

**Fix:**
```bash
# Create missing apps/core/middleware.py
cat > apps/core/middleware.py << 'EOF'
"""Core middleware for request logging"""
import logging

logger = logging.getLogger(__name__)

class RequestLoggingMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        logger.info(f"Request: {request.method} {request.path}")
        response = self.get_response(request)
        logger.info(f"Response: {response.status_code}")
        return response
EOF
```

### Error Type 3: "djdt namespace not found" (Debug Toolbar)

**Fix:**
```bash
# Edit config/urls.py to properly handle debug toolbar
# Add conditional debug toolbar URLs only when DEBUG=True
```

### Error Type 4: "SessionMiddleware must be in MIDDLEWARE"

**Fix:**
```bash
# Edit config/settings/base.py
# Ensure correct middleware order:
# SessionMiddleware BEFORE AuthenticationMiddleware
```

### Error Type 5: "Directory 'static' does not exist"

**Fix:**
```bash
# Create missing directories
mkdir -p static media logs
touch static/.keep media/.keep logs/.keep
```

### Error Type 6: "ModuleNotFoundError: No module named 'decouple'"

**Fix:**
```bash
# Install missing dependencies
pip install python-decouple django-environ
```

### Error Type 7: Database Connection Errors

**Fix:**
```bash
# For SQLite (development)
rm -f db.sqlite3
python manage.py migrate

# For PostgreSQL/MySQL
# Check database connection settings in config/settings/base.py
```

## Step 6: Final Verification Loop

**Repeat until successful:**

```bash
# 1. Run system checks
python manage.py check

# 2. Start server
python manage.py runserver

# 3. Test health API
curl http://127.0.0.1:8000/api/v1/health/

# 4. Check browser access
# Open: http://127.0.0.1:8000/api/v1/health/
```

**Success Indicators:**
- ✅ Django server starts without errors
- ✅ Health API returns JSON response
- ✅ Admin panel is accessible
- ✅ No error messages in terminal

## Step 7: Project Status Report

**Once health API is working, provide status report:**

### ✅ **Project Status: HEALTHY**

**Verified Components:**
- ✅ Project structure complete
- ✅ Virtual environment active
- ✅ Dependencies installed
- ✅ Database migrations applied
- ✅ Django server running
- ✅ Health API responding

**Available Endpoints:**
- 🌐 **Main Site**: http://127.0.0.1:8000/
- 👤 **Admin Panel**: http://127.0.0.1:8000/admin/
- 💓 **Health Check**: http://127.0.0.1:8000/api/v1/health/

**Terminal Output Should Show:**
```
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
Django version 4.2.x, using settings 'config.settings.development'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

## Error Fixing Strategy

**If errors persist:**

1. **Identify Error Type**: Read error message carefully
2. **Check Common Issues**: Use the error fixes above
3. **Create Missing Files**: Ensure all referenced modules exist
4. **Fix Configuration**: Correct settings and middleware order
5. **Verify Dependencies**: Install missing packages
6. **Test Incrementally**: Fix one error at a time
7. **Repeat Until Success**: Continue until health API works

## Completion Criteria

**STOP when all these conditions are met:**

✅ **Django server starts** without errors  
✅ **Health API endpoint** returns valid JSON response  
✅ **No error messages** in terminal output  
✅ **Browser can access** http://127.0.0.1:8000/api/v1/health/  

**Project is now ready for development!**

---

## Option B: If User Chooses "NO" - Direct Health Check

**Skip structure verification and go directly to environment setup:**

**Prerequisites:** System environment analysis from Step 1 must be completed first.

1. Navigate to project: `cd ${input:project_name}`
2. Set up virtual environment (OS-specific commands from Step 3.2) or use activated Conda environment
3. Install packages: `pip install -r requirements/development.txt`
4. Run migrations: `python manage.py migrate`
5. Check health API: `python manage.py runserver` → Test http://127.0.0.1:8000/api/v1/health/
6. Fix errors using the error fixing workflow from Step 5
7. Stop when health API works successfully

---

**Note**: This prompt focuses on getting your Django project running successfully. It will systematically identify and fix common issues until your health API endpoint is accessible and working properly.