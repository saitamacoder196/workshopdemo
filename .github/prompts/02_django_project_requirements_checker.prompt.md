---
description: Check if current Django project meets basic requirements - health endpoint, swagger, database config, and migrations
---

# Django Project Requirements Checker

This prompt checks if the current Django project meets the basic requirements including health endpoint, Swagger documentation, database configuration, and migration status.

## Requirements Checklist

This checker verifies:
1. ✅ **API Health Endpoint**: `/api/v1/health/` exists and responds correctly
2. ✅ **Swagger Support**: Swagger UI is configured and accessible
3. ✅ **Database Configuration**: Database is properly configured
4. ✅ **Migration Status**: No pending migrations

## Project Requirements Check

### Step 1: Check Project Structure

**Verify basic Django project structure:**

```bash
# Check current directory structure
echo "=== CHECKING PROJECT STRUCTURE ==="
echo "Current directory: $(pwd)"
echo "Project files:"
ls -la

# Check if it's a Django project
if [ -f "manage.py" ]; then
    echo "✅ Django project detected (manage.py found)"
else
    echo "❌ Not a Django project (manage.py not found)"
    exit 1
fi

# Check if apps directory exists
if [ -d "apps" ]; then
    echo "✅ Apps directory found"
    echo "Available apps:"
    ls -la apps/
else
    echo "⚠️ Apps directory not found - checking for standard Django apps"
fi

# Check settings configuration
if [ -f "config/settings/base.py" ] || [ -f "myproject/settings.py" ] || [ -f "*/settings.py" ]; then
    echo "✅ Django settings found"
else
    echo "❌ Django settings not found"
fi
```

### Step 2: Check Health Endpoint (Requirement 1)

**Verify `/api/v1/health/` endpoint exists and works:**

```python
# File: requirements_check/health_endpoint_checker.py

"""
Check if health endpoint exists and responds correctly
"""

import os
import sys
import django
from pathlib import Path
import requests
import subprocess
import time
from typing import Dict, Any

class HealthEndpointChecker:
    """Check health endpoint availability and functionality"""
    
    def __init__(self):
        self.health_url = "http://127.0.0.1:8000/api/v1/health/"
        self.server_process = None
        self.check_results = {
            'endpoint_configured': False,
            'url_pattern_exists': False,
            'view_implemented': False,
            'server_responds': False,
            'endpoint_accessible': False,
            'response_format_correct': False
        }
    
    def check_url_configuration(self) -> bool:
        """Check if health endpoint is configured in URLs"""
        url_files = [
            'config/urls.py',
            'myproject/urls.py',
            'apps/api/urls.py'
        ]
        
        health_endpoint_found = False
        
        for url_file in url_files:
            if Path(url_file).exists():
                with open(url_file, 'r') as f:
                    content = f.read()
                
                # Check for health endpoint patterns
                if 'health' in content.lower() or '/api/v1/' in content:
                    print(f"✅ Found API URL configuration in {url_file}")
                    health_endpoint_found = True
                    
                    # Check specific patterns
                    if 'health' in content.lower():
                        print("✅ Health endpoint pattern found")
                        self.check_results['url_pattern_exists'] = True
                    
                    break
        
        if not health_endpoint_found:
            print("❌ Health endpoint URL configuration not found")
            return False
        
        return True
    
    def check_view_implementation(self) -> bool:
        """Check if health view is implemented"""
        view_files = [
            'apps/api/views.py',
            'apps/*/views.py',
            'api/views.py'
        ]
        
        health_view_found = False
        
        # Use glob to find view files
        import glob
        for pattern in view_files:
            matching_files = glob.glob(pattern)
            for view_file in matching_files:
                if Path(view_file).exists():
                    with open(view_file, 'r') as f:
                        content = f.read()
                    
                    # Check for health view implementation
                    if 'health' in content.lower() and ('def ' in content or 'class ' in content):
                        print(f"✅ Health view implementation found in {view_file}")
                        health_view_found = True
                        self.check_results['view_implemented'] = True
                        break
        
        if not health_view_found:
            print("❌ Health view implementation not found")
        
        return health_view_found
    
    def start_dev_server(self) -> bool:
        """Start Django development server for testing"""
        try:
            print("Starting Django development server...")
            self.server_process = subprocess.Popen(
                ["python", "manage.py", "runserver", "127.0.0.1:8000"],
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE,
                text=True
            )
            
            # Wait for server to start
            time.sleep(3)
            
            # Check if server is running
            if self.server_process.poll() is None:
                print("✅ Django server started successfully")
                self.check_results['server_responds'] = True
                return True
            else:
                stdout, stderr = self.server_process.communicate()
                print(f"❌ Server failed to start: {stderr}")
                return False
                
        except Exception as e:
            print(f"❌ Failed to start server: {str(e)}")
            return False
    
    def test_health_endpoint(self) -> Dict[str, Any]:
        """Test the health endpoint response"""
        try:
            print(f"Testing health endpoint: {self.health_url}")
            
            response = requests.get(self.health_url, timeout=5)
            
            result = {
                'accessible': response.status_code == 200,
                'status_code': response.status_code,
                'response_time': response.elapsed.total_seconds(),
                'content_type': response.headers.get('content-type', ''),
                'response_data': None
            }
            
            if response.status_code == 200:
                print("✅ Health endpoint is accessible")
                self.check_results['endpoint_accessible'] = True
                
                try:
                    json_data = response.json()
                    result['response_data'] = json_data
                    
                    # Check response format
                    if 'status' in json_data:
                        print("✅ Health endpoint returns correct JSON format")
                        self.check_results['response_format_correct'] = True
                    else:
                        print("⚠️ Health endpoint response format may need improvement")
                        
                except ValueError:
                    print("⚠️ Health endpoint doesn't return JSON")
                    result['response_data'] = response.text
            else:
                print(f"❌ Health endpoint returned status code: {response.status_code}")
            
            return result
            
        except requests.exceptions.ConnectionError:
            print("❌ Cannot connect to health endpoint - server may not be running")
            return {'accessible': False, 'error': 'Connection failed'}
        except Exception as e:
            print(f"❌ Error testing health endpoint: {str(e)}")
            return {'accessible': False, 'error': str(e)}
    
    def stop_dev_server(self):
        """Stop the development server"""
        if self.server_process:
            self.server_process.terminate()
            self.server_process.wait()
            print("Django server stopped")
    
    def check_health_endpoint_complete(self) -> Dict:
        """Complete health endpoint check"""
        print("\n=== CHECKING HEALTH ENDPOINT (Requirement 1) ===")
        
        # Check URL configuration
        self.check_url_configuration()
        
        # Check view implementation
        self.check_view_implementation()
        
        # Start server and test endpoint
        if self.start_dev_server():
            time.sleep(2)  # Additional wait for server to be ready
            endpoint_result = self.test_health_endpoint()
            self.stop_dev_server()
        else:
            endpoint_result = {'accessible': False, 'error': 'Server failed to start'}
        
        # Update final results
        self.check_results['endpoint_configured'] = (
            self.check_results['url_pattern_exists'] and 
            self.check_results['view_implemented']
        )
        
        return {
            'requirement': 'Health Endpoint (/api/v1/health/)',
            'status': 'PASS' if self.check_results['endpoint_accessible'] else 'FAIL',
            'details': self.check_results,
            'endpoint_test': endpoint_result
        }


def check_health_endpoint():
    """Main function to check health endpoint"""
    checker = HealthEndpointChecker()
    return checker.check_health_endpoint_complete()


if __name__ == '__main__':
    result = check_health_endpoint()
    print(f"\nHealth Endpoint Check: {result['status']}")
```

### Step 3: Check Swagger Support (Requirement 2)

**Verify Swagger UI is configured and accessible:**

```python
# File: requirements_check/swagger_checker.py

"""
Check if Swagger documentation is properly configured
"""

import os
import subprocess
import time
import requests
from pathlib import Path
from typing import Dict, Any

class SwaggerChecker:
    """Check Swagger documentation configuration and accessibility"""
    
    def __init__(self):
        self.swagger_urls = [
            "http://127.0.0.1:8000/swagger/",
            "http://127.0.0.1:8000/api/swagger/",
            "http://127.0.0.1:8000/docs/"
        ]
        self.server_process = None
        self.check_results = {
            'drf_yasg_installed': False,
            'swagger_urls_configured': False,
            'swagger_accessible': False,
            'swagger_url': None
        }
    
    def check_swagger_installation(self) -> bool:
        """Check if drf-yasg is installed"""
        try:
            # Check requirements.txt
            if Path('requirements.txt').exists():
                with open('requirements.txt', 'r') as f:
                    requirements = f.read()
                
                if 'drf-yasg' in requirements:
                    print("✅ drf-yasg found in requirements.txt")
                    self.check_results['drf_yasg_installed'] = True
                    return True
            
            # Check if it's importable
            import drf_yasg
            print("✅ drf-yasg is installed and importable")
            self.check_results['drf_yasg_installed'] = True
            return True
            
        except ImportError:
            print("❌ drf-yasg is not installed")
            return False
    
    def check_swagger_settings(self) -> bool:
        """Check if Swagger is configured in Django settings"""
        settings_files = [
            'config/settings/base.py',
            'config/settings.py',
            'myproject/settings.py',
            'settings.py'
        ]
        
        swagger_in_settings = False
        
        for settings_file in settings_files:
            if Path(settings_file).exists():
                with open(settings_file, 'r') as f:
                    content = f.read()
                
                if 'drf_yasg' in content:
                    print(f"✅ drf_yasg configuration found in {settings_file}")
                    swagger_in_settings = True
                    break
        
        if not swagger_in_settings:
            print("❌ drf_yasg not found in Django settings")
        
        return swagger_in_settings
    
    def check_swagger_urls(self) -> bool:
        """Check if Swagger URLs are configured"""
        url_files = [
            'config/urls.py',
            'myproject/urls.py',
            'urls.py'
        ]
        
        swagger_urls_found = False
        
        for url_file in url_files:
            if Path(url_file).exists():
                with open(url_file, 'r') as f:
                    content = f.read()
                
                # Check for Swagger URL patterns
                if 'swagger' in content.lower() or 'schema_view' in content:
                    print(f"✅ Swagger URL configuration found in {url_file}")
                    swagger_urls_found = True
                    self.check_results['swagger_urls_configured'] = True
                    break
        
        if not swagger_urls_found:
            print("❌ Swagger URL configuration not found")
        
        return swagger_urls_found
    
    def start_server_for_testing(self) -> bool:
        """Start Django server for Swagger testing"""
        try:
            print("Starting Django server for Swagger testing...")
            self.server_process = subprocess.Popen(
                ["python", "manage.py", "runserver", "127.0.0.1:8000"],
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE,
                text=True
            )
            
            time.sleep(3)
            
            if self.server_process.poll() is None:
                print("✅ Django server started for Swagger testing")
                return True
            else:
                print("❌ Failed to start Django server")
                return False
                
        except Exception as e:
            print(f"❌ Error starting server: {str(e)}")
            return False
    
    def test_swagger_accessibility(self) -> Dict[str, Any]:
        """Test if Swagger UI is accessible"""
        for swagger_url in self.swagger_urls:
            try:
                print(f"Testing Swagger URL: {swagger_url}")
                
                response = requests.get(swagger_url, timeout=5)
                
                if response.status_code == 200:
                    print(f"✅ Swagger UI accessible at: {swagger_url}")
                    self.check_results['swagger_accessible'] = True
                    self.check_results['swagger_url'] = swagger_url
                    
                    # Check if it's actually Swagger content
                    if 'swagger' in response.text.lower() or 'api documentation' in response.text.lower():
                        return {
                            'accessible': True,
                            'url': swagger_url,
                            'status_code': response.status_code,
                            'content_type': response.headers.get('content-type', '')
                        }
                else:
                    print(f"❌ Swagger URL returned status: {response.status_code}")
                    
            except requests.exceptions.ConnectionError:
                print(f"❌ Cannot connect to {swagger_url}")
            except Exception as e:
                print(f"❌ Error testing {swagger_url}: {str(e)}")
        
        print("❌ Swagger UI is not accessible at any tested URLs")
        return {'accessible': False, 'error': 'No accessible Swagger URLs found'}
    
    def stop_server(self):
        """Stop the test server"""
        if self.server_process:
            self.server_process.terminate()
            self.server_process.wait()
            print("Test server stopped")
    
    def check_swagger_complete(self) -> Dict:
        """Complete Swagger check"""
        print("\n=== CHECKING SWAGGER SUPPORT (Requirement 2) ===")
        
        # Check installation
        installation_ok = self.check_swagger_installation()
        
        # Check settings configuration
        settings_ok = self.check_swagger_settings()
        
        # Check URL configuration
        urls_ok = self.check_swagger_urls()
        
        # Test accessibility
        accessibility_result = {'accessible': False}
        if installation_ok and settings_ok and urls_ok:
            if self.start_server_for_testing():
                time.sleep(2)
                accessibility_result = self.test_swagger_accessibility()
                self.stop_server()
        
        overall_status = (
            installation_ok and 
            settings_ok and 
            urls_ok and 
            accessibility_result.get('accessible', False)
        )
        
        return {
            'requirement': 'Swagger Documentation Support',
            'status': 'PASS' if overall_status else 'FAIL',
            'details': {
                'installation': installation_ok,
                'settings_configured': settings_ok,
                'urls_configured': urls_ok,
                'accessible': accessibility_result.get('accessible', False),
                'swagger_url': self.check_results.get('swagger_url')
            },
            'accessibility_test': accessibility_result
        }


def check_swagger_support():
    """Main function to check Swagger support"""
    checker = SwaggerChecker()
    return checker.check_swagger_complete()


if __name__ == '__main__':
    result = check_swagger_support()
    print(f"\nSwagger Support Check: {result['status']}")
```

### Step 4: Check Database Configuration (Requirement 3)

**Verify database is properly configured:**

```python
# File: requirements_check/database_checker.py

"""
Check if database is properly configured
"""

import os
import django
from pathlib import Path
from typing import Dict, Any

class DatabaseChecker:
    """Check database configuration"""
    
    def __init__(self):
        self.check_results = {
            'settings_configured': False,
            'database_accessible': False,
            'migrations_directory_exists': False,
            'initial_migration_exists': False
        }
    
    def setup_django(self):
        """Setup Django environment for database checks"""
        try:
            # Try to set up Django settings
            if not os.environ.get('DJANGO_SETTINGS_MODULE'):
                # Try common settings module patterns
                settings_patterns = [
                    'config.settings.base',
                    'config.settings',
                    'myproject.settings',
                    'settings'
                ]
                
                for pattern in settings_patterns:
                    try:
                        os.environ.setdefault('DJANGO_SETTINGS_MODULE', pattern)
                        django.setup()
                        print(f"✅ Django setup successful with {pattern}")
                        return True
                    except Exception:
                        continue
                
                print("❌ Could not setup Django environment")
                return False
            else:
                django.setup()
                return True
                
        except Exception as e:
            print(f"❌ Django setup failed: {str(e)}")
            return False
    
    def check_database_settings(self) -> bool:
        """Check if database settings are configured"""
        try:
            from django.conf import settings
            
            databases = getattr(settings, 'DATABASES', {})
            
            if not databases:
                print("❌ No database configuration found")
                return False
            
            default_db = databases.get('default', {})
            
            if not default_db:
                print("❌ Default database not configured")
                return False
            
            engine = default_db.get('ENGINE', '')
            name = default_db.get('NAME', '')
            
            print(f"✅ Database configured:")
            print(f"   Engine: {engine}")
            print(f"   Name: {name}")
            
            if engine and name:
                self.check_results['settings_configured'] = True
                return True
            else:
                print("❌ Database configuration incomplete")
                return False
                
        except Exception as e:
            print(f"❌ Error checking database settings: {str(e)}")
            return False
    
    def check_database_connection(self) -> bool:
        """Check if database connection works"""
        try:
            from django.db import connection
            
            # Test database connection
            with connection.cursor() as cursor:
                cursor.execute("SELECT 1")
                result = cursor.fetchone()
            
            if result:
                print("✅ Database connection successful")
                self.check_results['database_accessible'] = True
                return True
            else:
                print("❌ Database connection failed")
                return False
                
        except Exception as e:
            print(f"❌ Database connection error: {str(e)}")
            return False
    
    def check_migrations_structure(self) -> bool:
        """Check if migrations directory structure exists"""
        migrations_found = False
        
        # Check for migrations directories
        app_dirs = []
        
        # Check apps directory
        if Path('apps').exists():
            app_dirs.extend([d for d in Path('apps').iterdir() if d.is_dir()])
        
        # Check root level apps
        potential_apps = [d for d in Path('.').iterdir() if d.is_dir() and (d / 'models.py').exists()]
        app_dirs.extend(potential_apps)
        
        for app_dir in app_dirs:
            migrations_dir = app_dir / 'migrations'
            if migrations_dir.exists():
                print(f"✅ Migrations directory found: {migrations_dir}")
                migrations_found = True
                self.check_results['migrations_directory_exists'] = True
                
                # Check for initial migration
                migration_files = list(migrations_dir.glob('*.py'))
                if any('0001_initial.py' in str(f) for f in migration_files):
                    print(f"✅ Initial migration found in {app_dir.name}")
                    self.check_results['initial_migration_exists'] = True
        
        if not migrations_found:
            print("❌ No migrations directories found")
        
        return migrations_found
    
    def check_database_complete(self) -> Dict:
        """Complete database configuration check"""
        print("\n=== CHECKING DATABASE CONFIGURATION (Requirement 3) ===")
        
        # Setup Django
        django_setup_ok = self.setup_django()
        
        if not django_setup_ok:
            return {
                'requirement': 'Database Configuration',
                'status': 'FAIL',
                'details': {'error': 'Could not setup Django environment'},
                'recommendations': ['Check Django settings configuration']
            }
        
        # Check database settings
        settings_ok = self.check_database_settings()
        
        # Check database connection
        connection_ok = self.check_database_connection()
        
        # Check migrations structure
        migrations_ok = self.check_migrations_structure()
        
        overall_status = settings_ok and connection_ok
        
        recommendations = []
        if not settings_ok:
            recommendations.append("Configure database settings in Django settings")
        if not connection_ok:
            recommendations.append("Ensure database server is running and accessible")
        if not migrations_ok:
            recommendations.append("Run 'python manage.py makemigrations' to create initial migrations")
        
        return {
            'requirement': 'Database Configuration',
            'status': 'PASS' if overall_status else 'FAIL',
            'details': {
                'django_setup': django_setup_ok,
                'settings_configured': settings_ok,
                'connection_works': connection_ok,
                'migrations_exist': migrations_ok
            },
            'recommendations': recommendations
        }


def check_database_config():
    """Main function to check database configuration"""
    checker = DatabaseChecker()
    return checker.check_database_complete()


if __name__ == '__main__':
    result = check_database_config()
    print(f"\nDatabase Configuration Check: {result['status']}")
```

### Step 5: Check Migration Status (Requirement 4)

**Check if there are any pending migrations:**

```bash
# Check for pending migrations
echo "=== CHECKING MIGRATION STATUS (Requirement 4) ==="

# Function to check migrations
check_migrations() {
    echo "Checking for pending migrations..."
    
    # Run Django migration check
    migration_output=$(python manage.py showmigrations --plan 2>&1)
    migration_exit_code=$?
    
    if [ $migration_exit_code -eq 0 ]; then
        echo "✅ Migration check command successful"
        
        # Check for pending migrations (lines starting with [ ])
        pending_migrations=$(echo "$migration_output" | grep "^ \[ \]" | wc -l)
        applied_migrations=$(echo "$migration_output" | grep "^ \[X\]" | wc -l)
        
        echo "Migration status:"
        echo "  Applied migrations: $applied_migrations"
        echo "  Pending migrations: $pending_migrations"
        
        if [ $pending_migrations -eq 0 ]; then
            echo "✅ No pending migrations - database is up to date"
            return 0
        else
            echo "⚠️ Found $pending_migrations pending migrations"
            echo "Pending migrations:"
            echo "$migration_output" | grep "^ \[ \]"
            return 1
        fi
    else
        echo "❌ Migration check failed:"
        echo "$migration_output"
        return 1
    fi
}

# Run the migration check
if check_migrations; then
    migration_status="PASS"
else
    migration_status="FAIL"
fi

echo "Migration Status Check: $migration_status"
```

### Step 6: Generate Complete Requirements Report

**Generate comprehensive requirements check report:**

```python
# File: requirements_check/requirements_report.py

"""
Generate comprehensive requirements check report
"""

import json
import subprocess
from datetime import datetime
from pathlib import Path

def run_all_checks():
    """Run all requirement checks and generate report"""
    
    print("=== DJANGO PROJECT REQUIREMENTS CHECKER ===")
    print(f"Check started at: {datetime.now().isoformat()}")
    
    results = {
        'metadata': {
            'check_timestamp': datetime.now().isoformat(),
            'project_path': str(Path.cwd()),
            'checker_version': '1.0'
        },
        'requirements': {},
        'overall_status': 'UNKNOWN',
        'summary': {
            'total_requirements': 4,
            'passed': 0,
            'failed': 0,
            'pass_rate': 0.0
        },
        'recommendations': []
    }
    
    # Import and run individual checkers
    try:
        # Check 1: Health Endpoint
        exec(open('requirements_check/health_endpoint_checker.py').read())
        health_result = check_health_endpoint()
        results['requirements']['health_endpoint'] = health_result
        
        # Check 2: Swagger Support
        exec(open('requirements_check/swagger_checker.py').read())
        swagger_result = check_swagger_support()
        results['requirements']['swagger_support'] = swagger_result
        
        # Check 3: Database Configuration
        exec(open('requirements_check/database_checker.py').read())
        database_result = check_database_config()
        results['requirements']['database_config'] = database_result
        
        # Check 4: Migration Status
        migration_output = subprocess.run(
            ["python", "manage.py", "showmigrations", "--plan"],
            capture_output=True,
            text=True
        )
        
        pending_count = migration_output.stdout.count('[ ]')
        applied_count = migration_output.stdout.count('[X]')
        
        migration_result = {
            'requirement': 'Migration Status',
            'status': 'PASS' if pending_count == 0 else 'FAIL',
            'details': {
                'pending_migrations': pending_count,
                'applied_migrations': applied_count,
                'migration_output': migration_output.stdout if migration_output.returncode == 0 else migration_output.stderr
            }
        }
        results['requirements']['migration_status'] = migration_result
        
    except Exception as e:
        print(f"Error running checks: {str(e)}")
        results['error'] = str(e)
    
    # Calculate summary
    passed_count = sum(1 for req in results['requirements'].values() if req.get('status') == 'PASS')
    failed_count = len(results['requirements']) - passed_count
    
    results['summary']['passed'] = passed_count
    results['summary']['failed'] = failed_count
    results['summary']['pass_rate'] = (passed_count / len(results['requirements']) * 100) if results['requirements'] else 0
    
    # Determine overall status
    if passed_count == len(results['requirements']):
        results['overall_status'] = 'PASS'
    elif passed_count > 0:
        results['overall_status'] = 'PARTIAL'
    else:
        results['overall_status'] = 'FAIL'
    
    # Generate recommendations
    for req_name, req_result in results['requirements'].items():
        if req_result.get('status') == 'FAIL':
            if 'recommendations' in req_result:
                results['recommendations'].extend(req_result['recommendations'])
            else:
                results['recommendations'].append(f"Fix {req_result.get('requirement', req_name)}")
    
    # Save detailed report
    Path('requirements_check').mkdir(exist_ok=True)
    with open('requirements_check/requirements_report.json', 'w') as f:
        json.dump(results, f, indent=2)
    
    # Generate summary text report
    summary_text = f"""
=== DJANGO PROJECT REQUIREMENTS CHECK SUMMARY ===

Overall Status: {results['overall_status']}
Pass Rate: {results['summary']['pass_rate']:.1f}% ({results['summary']['passed']}/{results['summary']['total_requirements']})

REQUIREMENT RESULTS:
"""
    
    for req_name, req_result in results['requirements'].items():
        status_icon = "✅" if req_result.get('status') == 'PASS' else "❌"
        requirement_name = req_result.get('requirement', req_name)
        summary_text += f"{status_icon} {requirement_name}: {req_result.get('status', 'UNKNOWN')}\n"
    
    if results['recommendations']:
        summary_text += "\nRECOMMENDATIONS:\n"
        for i, rec in enumerate(results['recommendations'], 1):
            summary_text += f"{i}. {rec}\n"
    
    if results['overall_status'] == 'PASS':
        summary_text += "\n🎉 All requirements met! Project is ready for development.\n"
    elif results['overall_status'] == 'PARTIAL':
        summary_text += "\n⚠️ Some requirements need attention before proceeding.\n"
    else:
        summary_text += "\n❌ Multiple requirements failed. Project needs setup work.\n"
    
    with open('requirements_check/requirements_summary.txt', 'w') as f:
        f.write(summary_text)
    
    print(summary_text)
    
    return results


if __name__ == '__main__':
    run_all_checks()
```

## Main Execution Script

**Execute all requirement checks:**

```bash
#!/bin/bash

# Main requirements checker script
echo "=== DJANGO PROJECT REQUIREMENTS CHECKER ==="

# Create requirements check directory
mkdir -p requirements_check

# Copy checker scripts
cat > requirements_check/health_endpoint_checker.py << 'EOF'
# Health endpoint checker content would be here
EOF

cat > requirements_check/swagger_checker.py << 'EOF'
# Swagger checker content would be here  
EOF

cat > requirements_check/database_checker.py << 'EOF'
# Database checker content would be here
EOF

# Run the complete requirements check
python requirements_check/requirements_report.py

# Display final results
echo ""
echo "=== FINAL RESULTS ==="
if [ -f "requirements_check/requirements_summary.txt" ]; then
    cat requirements_check/requirements_summary.txt
else
    echo "❌ Requirements check failed to generate summary"
fi

echo ""
echo "📄 Detailed report: requirements_check/requirements_report.json"
echo "📝 Summary report: requirements_check/requirements_summary.txt"
```

## Usage Instructions

**To run the requirements checker:**

```bash
# Run the complete requirements check
bash requirements_check.sh

# Or run individual components:
python requirements_check/health_endpoint_checker.py
python requirements_check/swagger_checker.py  
python requirements_check/database_checker.py
python manage.py showmigrations --plan
```

## Success Criteria

**Project meets requirements when all checks pass:**

✅ **Health Endpoint**: `/api/v1/health/` responds with 200 OK  
✅ **Swagger Support**: Swagger UI accessible and functional  
✅ **Database Config**: Database connected and accessible  
✅ **Migration Status**: No pending migrations (all applied)

---

**Note**: This checker validates that your Django project has the essential components needed for API development with proper documentation and database setup.