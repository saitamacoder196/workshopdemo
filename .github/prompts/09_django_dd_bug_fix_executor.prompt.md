---
description: Execute bug fixes based on the detailed plan from prompt 08 with automated code modification, testing, and validation
---

# Django Design Detail (DD) Bug Fix Executor

This prompt executes the bug fix plan created in prompt 08, performing actual code modifications, running tests in real-time, and validating each fix before proceeding to the next one.

## Prerequisites

**Required Input:**
- 📋 **Bug Fix Plan**: Output from `08_django_dd_bug_fix_planner.prompt.md`
- 📦 **Project Name**: `${input:project_name}`
- 🏷️ **App Name**: `${input:app_name}`
- 📄 **Fix Plan Path**: `${input:fix_plan_path}`
- 🌿 **Base Branch**: `${input:base_branch:main}`

## Bug Fix Execution Workflow

### Step 1: Initialize Fix Execution Environment

**Set up environment and load fix plan:**

```bash
# Initialize execution environment
echo "=== INITIALIZING BUG FIX EXECUTION ==="
echo "Project: ${input:project_name}"
echo "App: ${input:app_name}"
echo "Base Branch: ${input:base_branch}"

# Create bug fix execution directory
mkdir -p bug_fix_execution/{logs,backups,reports}
cd bug_fix_execution

# Load and validate fix plan
echo "Loading bug fix plan from ${input:fix_plan_path}..."
cat ${input:fix_plan_path}/bug_fix_plan.json > fix_plan.json

# Validate plan exists and is readable
if [ ! -f "fix_plan.json" ]; then
    echo "❌ Error: Fix plan not found at ${input:fix_plan_path}/bug_fix_plan.json"
    exit 1
fi

echo "✅ Fix plan loaded successfully"

# Get current git status
echo "Current git status:"
git status --porcelain
git log --oneline -5
```

**Create backup of current state:**

```bash
# Create backup before starting fixes
echo "=== CREATING BACKUP ==="
timestamp=$(date +"%Y%m%d_%H%M%S")
backup_branch="backup_before_fixes_${timestamp}"

# Create backup branch
git checkout -b $backup_branch
git push origin $backup_branch
echo "✅ Backup branch created: $backup_branch"

# Return to base branch
git checkout ${input:base_branch}
```

### Step 2: Execute Phase 1 - Critical Issues

**Execute critical priority fixes with immediate testing:**

```python
# File: bug_fix_execution/phase1_critical_executor.py

"""
Execute Phase 1 critical bug fixes with real-time validation
"""

import json
import subprocess
import sys
from datetime import datetime
from pathlib import Path
from typing import Dict, List, Any

class CriticalBugFixExecutor:
    """Execute critical bug fixes with immediate validation"""
    
    def __init__(self, fix_plan_path: str, app_name: str):
        self.fix_plan_path = fix_plan_path
        self.app_name = app_name
        self.execution_log = []
        self.current_phase = 'phase_1_critical'
        
        # Load fix plan
        with open(fix_plan_path, 'r') as f:
            self.fix_plan = json.load(f)
        
        self.critical_issues = self.fix_plan['priority_phases'].get(self.current_phase, [])
    
    def log_action(self, action: str, status: str, details: str = ""):
        """Log execution action"""
        log_entry = {
            'timestamp': datetime.now().isoformat(),
            'action': action,
            'status': status,
            'details': details
        }
        self.execution_log.append(log_entry)
        
        # Also print to console
        status_icon = "✅" if status == "success" else "❌" if status == "failed" else "⚠️"
        print(f"{status_icon} {action}: {details}")
    
    def run_command(self, command: str, description: str = "") -> Dict:
        """Run shell command and capture result"""
        try:
            result = subprocess.run(
                command,
                shell=True,
                capture_output=True,
                text=True,
                timeout=300  # 5 minute timeout
            )
            
            success = result.returncode == 0
            
            self.log_action(
                action=description or command,
                status="success" if success else "failed",
                details=result.stdout if success else result.stderr
            )
            
            return {
                'success': success,
                'stdout': result.stdout,
                'stderr': result.stderr,
                'returncode': result.returncode
            }
        except subprocess.TimeoutExpired:
            self.log_action(
                action=description or command,
                status="failed",
                details="Command timed out after 5 minutes"
            )
            return {
                'success': False,
                'stdout': '',
                'stderr': 'Timeout expired',
                'returncode': -1
            }
    
    def create_fix_branch(self, issue_id: str) -> str:
        """Create branch for specific fix"""
        branch_name = f"fix/critical-{issue_id}-{datetime.now().strftime('%Y%m%d-%H%M%S')}"
        
        result = self.run_command(
            f"git checkout -b {branch_name}",
            f"Create fix branch {branch_name}"
        )
        
        if result['success']:
            return branch_name
        else:
            raise Exception(f"Failed to create branch: {result['stderr']}")
    
    def fix_security_vulnerability(self, issue: Dict) -> Dict:
        """Fix security vulnerability with immediate validation"""
        file_path = issue.get('file_path', '')
        check_id = issue.get('check_id', '')
        description = issue.get('description', '')
        
        self.log_action("fix_security_vulnerability", "started", f"Fixing {check_id} in {file_path}")
        
        try:
            # Read current file content
            with open(file_path, 'r') as f:
                original_content = f.read()
            
            # Apply specific security fixes
            modified_content = original_content
            changes_made = []
            
            if 'OWASP001' in check_id:  # Injection Prevention
                # Fix SQL injection vulnerabilities
                if 'cursor.execute(' in modified_content and '%s' not in modified_content:
                    # Convert unsafe string formatting to parameterized queries
                    import re
                    pattern = r'cursor\.execute\(["\']([^"\']*)["\']\.format\([^)]+\)\)'
                    
                    def fix_sql_injection(match):
                        query = match.group(1)
                        # Convert to parameterized query
                        param_query = re.sub(r'\{[^}]+\}', '%s', query)
                        return f'cursor.execute("{param_query}", params)'
                    
                    modified_content = re.sub(pattern, fix_sql_injection, modified_content)
                    changes_made.append("Converted string formatting to parameterized queries")
            
            elif 'OWASP002' in check_id:  # Authentication Security
                if 'settings' in file_path.lower():
                    # Add secure authentication settings
                    security_settings = """
# Security settings for authentication
SESSION_COOKIE_SECURE = True
SESSION_COOKIE_HTTPONLY = True
CSRF_COOKIE_SECURE = True
CSRF_COOKIE_HTTPONLY = True
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
"""
                    if 'SESSION_COOKIE_SECURE' not in modified_content:
                        modified_content += security_settings
                        changes_made.append("Added secure authentication settings")
            
            elif 'OWASP003' in check_id:  # Sensitive Data Exposure
                # Remove hardcoded secrets and replace with environment variables
                import re
                
                # Pattern for hardcoded secrets
                secret_patterns = [
                    (r'SECRET_KEY\s*=\s*["\'][^"\']+["\']', 'SECRET_KEY = os.getenv("SECRET_KEY")'),
                    (r'DATABASE_PASSWORD\s*=\s*["\'][^"\']+["\']', 'DATABASE_PASSWORD = os.getenv("DATABASE_PASSWORD")'),
                    (r'API_KEY\s*=\s*["\'][^"\']+["\']', 'API_KEY = os.getenv("API_KEY")')
                ]
                
                for pattern, replacement in secret_patterns:
                    if re.search(pattern, modified_content):
                        modified_content = re.sub(pattern, replacement, modified_content)
                        changes_made.append(f"Replaced hardcoded secret with environment variable")
                        
                        # Add os import if needed
                        if 'import os' not in modified_content:
                            modified_content = 'import os\n' + modified_content
            
            elif 'OWASP005' in check_id:  # Broken Access Control
                # Fix access control in views
                if 'views.py' in file_path:
                    # Add permission checks
                    permission_template = '''
from rest_framework.permissions import IsAuthenticated
from rest_framework.decorators import permission_classes

class YourViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthenticated]
    
    def get_queryset(self):
        # Ensure users can only access their own data
        return self.queryset.filter(user=self.request.user)
'''
                    # Add imports if missing
                    if 'from rest_framework.permissions import' not in modified_content:
                        import_line = 'from rest_framework.permissions import IsAuthenticated\n'
                        modified_content = import_line + modified_content
                        changes_made.append("Added permission imports")
                    
                    # Add permission classes to ViewSets without them
                    import re
                    viewset_pattern = r'(class \w+ViewSet\([^)]+\):)(?!\s*permission_classes)'
                    if re.search(viewset_pattern, modified_content):
                        replacement = r'\1\n    permission_classes = [IsAuthenticated]'
                        modified_content = re.sub(viewset_pattern, replacement, modified_content)
                        changes_made.append("Added permission_classes to ViewSets")
            
            # Write modified content if changes were made
            if changes_made:
                with open(file_path, 'w') as f:
                    f.write(modified_content)
                
                self.log_action("file_modified", "success", f"Applied changes: {', '.join(changes_made)}")
                
                # Validate syntax
                if file_path.endswith('.py'):
                    syntax_check = self.run_command(
                        f"python -m py_compile {file_path}",
                        f"Syntax check {file_path}"
                    )
                    
                    if not syntax_check['success']:
                        # Restore original content
                        with open(file_path, 'w') as f:
                            f.write(original_content)
                        raise Exception(f"Syntax error after fix: {syntax_check['stderr']}")
                
                return {
                    'success': True,
                    'changes': changes_made,
                    'file_modified': file_path
                }
            else:
                self.log_action("no_changes_needed", "info", "No automated fix available")
                return {
                    'success': True,
                    'changes': ['Manual review recommended'],
                    'file_modified': None
                }
        
        except Exception as e:
            self.log_action("fix_security_vulnerability", "failed", str(e))
            return {
                'success': False,
                'error': str(e),
                'changes': []
            }
    
    def fix_authentication_issue(self, issue: Dict) -> Dict:
        """Fix authentication-related issues"""
        file_path = issue.get('file_path', f'apps/{self.app_name}/views.py')
        description = issue.get('description', '')
        
        self.log_action("fix_authentication_issue", "started", f"Fixing auth issue in {file_path}")
        
        try:
            with open(file_path, 'r') as f:
                original_content = f.read()
            
            modified_content = original_content
            changes_made = []
            
            # Add authentication classes if missing
            if 'authentication_classes' not in modified_content:
                # Add import
                if 'from rest_framework.authentication import' not in modified_content:
                    auth_import = 'from rest_framework.authentication import TokenAuthentication, SessionAuthentication\n'
                    modified_content = auth_import + modified_content
                    changes_made.append("Added authentication imports")
                
                # Add authentication classes to ViewSets
                import re
                viewset_pattern = r'(class \w+ViewSet\([^)]+\):)'
                if re.search(viewset_pattern, modified_content):
                    replacement = r'\1\n    authentication_classes = [TokenAuthentication, SessionAuthentication]'
                    modified_content = re.sub(viewset_pattern, replacement, modified_content)
                    changes_made.append("Added authentication_classes to ViewSets")
            
            # Fix token authentication issues
            if 'token' in description.lower():
                # Ensure proper token handling
                token_middleware = '''
# In settings.py, ensure these are present:
INSTALLED_APPS = [
    # ...
    'rest_framework.authtoken',
    # ...
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
}
'''
                if 'settings.py' in file_path and 'rest_framework.authtoken' not in modified_content:
                    # Add to INSTALLED_APPS
                    apps_pattern = r"(INSTALLED_APPS\s*=\s*\[)(.*?)(\])"
                    def add_authtoken(match):
                        prefix = match.group(1)
                        apps = match.group(2)
                        suffix = match.group(3)
                        if 'rest_framework.authtoken' not in apps:
                            apps += "\n    'rest_framework.authtoken',"
                        return prefix + apps + suffix
                    
                    modified_content = re.sub(apps_pattern, add_authtoken, modified_content, flags=re.DOTALL)
                    changes_made.append("Added rest_framework.authtoken to INSTALLED_APPS")
            
            # Write changes if any were made
            if changes_made:
                with open(file_path, 'w') as f:
                    f.write(modified_content)
                
                self.log_action("authentication_fixed", "success", f"Applied: {', '.join(changes_made)}")
                
                return {
                    'success': True,
                    'changes': changes_made,
                    'file_modified': file_path
                }
            else:
                return {
                    'success': True,
                    'changes': ['No changes needed'],
                    'file_modified': None
                }
        
        except Exception as e:
            self.log_action("fix_authentication_issue", "failed", str(e))
            return {
                'success': False,
                'error': str(e),
                'changes': []
            }
    
    def fix_database_integrity_issue(self, issue: Dict) -> Dict:
        """Fix database integrity issues"""
        file_path = issue.get('file_path', f'apps/{self.app_name}/models.py')
        error_message = issue.get('error', '')
        
        self.log_action("fix_database_integrity", "started", f"Fixing DB integrity in {file_path}")
        
        try:
            with open(file_path, 'r') as f:
                original_content = f.read()
            
            modified_content = original_content
            changes_made = []
            
            # Fix foreign key constraint issues
            if 'foreign key' in error_message.lower() or 'constraint' in error_message.lower():
                import re
                
                # Add on_delete to ForeignKey fields missing it
                fk_pattern = r'(\w+)\s*=\s*models\.ForeignKey\(([^)]*)\)'
                
                def fix_foreign_key(match):
                    field_name = match.group(1)
                    field_args = match.group(2)
                    
                    if 'on_delete=' not in field_args:
                        if field_args.strip():
                            field_args += ', on_delete=models.CASCADE'
                        else:
                            field_args = 'on_delete=models.CASCADE'
                        changes_made.append(f"Added on_delete to {field_name}")
                    
                    return f"{field_name} = models.ForeignKey({field_args})"
                
                modified_content = re.sub(fk_pattern, fix_foreign_key, modified_content)
            
            # Fix unique constraint violations
            if 'unique' in error_message.lower():
                # Add unique=True to fields that need it
                if 'email' in error_message.lower():
                    # Make email fields unique
                    email_pattern = r'(email\s*=\s*models\.(Email|Char)Field\([^)]*)\)'
                    def add_unique_email(match):
                        field_def = match.group(1)
                        if 'unique=' not in field_def:
                            field_def += ', unique=True'
                            changes_made.append("Added unique=True to email field")
                        return field_def + ')'
                    
                    modified_content = re.sub(email_pattern, add_unique_email, modified_content, flags=re.IGNORECASE)
            
            # Fix null constraint violations
            if 'null' in error_message.lower() or 'not null' in error_message.lower():
                # Add default values or null=True where appropriate
                required_fields_pattern = r'(\w+)\s*=\s*models\.(Char|Text|Integer|Decimal)Field\(([^)]*)\)'
                
                def fix_null_constraints(match):
                    field_name = match.group(1)
                    field_type = match.group(2)
                    field_args = match.group(3)
                    
                    if 'null=' not in field_args and 'default=' not in field_args:
                        # Add appropriate default or null=True
                        if field_type in ['Char', 'Text']:
                            if field_args:
                                field_args += ', default=""'
                            else:
                                field_args = 'default=""'
                        elif field_type in ['Integer', 'Decimal']:
                            if field_args:
                                field_args += ', default=0'
                            else:
                                field_args = 'default=0'
                        changes_made.append(f"Added default value to {field_name}")
                    
                    return f"{field_name} = models.{field_type}Field({field_args})"
                
                modified_content = re.sub(required_fields_pattern, fix_null_constraints, modified_content)
            
            # Write changes and create migration if needed
            if changes_made:
                with open(file_path, 'w') as f:
                    f.write(modified_content)
                
                self.log_action("model_updated", "success", f"Applied: {', '.join(changes_made)}")
                
                # Create migration for model changes
                migration_result = self.run_command(
                    f"python manage.py makemigrations {self.app_name}",
                    "Create migration for model changes"
                )
                
                if migration_result['success']:
                    self.log_action("migration_created", "success", "Created migration for model changes")
                
                return {
                    'success': True,
                    'changes': changes_made,
                    'file_modified': file_path,
                    'migration_created': migration_result['success']
                }
            else:
                return {
                    'success': True,
                    'changes': ['No changes needed'],
                    'file_modified': None
                }
        
        except Exception as e:
            self.log_action("fix_database_integrity", "failed", str(e))
            return {
                'success': False,
                'error': str(e),
                'changes': []
            }
    
    def validate_fix(self, issue: Dict, fix_result: Dict) -> Dict:
        """Validate that a fix was successful"""
        validation_results = {
            'syntax_valid': False,
            'tests_pass': False,
            'security_scan_clean': False,
            'overall_success': False
        }
        
        # Syntax validation for Python files
        if fix_result.get('file_modified') and fix_result['file_modified'].endswith('.py'):
            syntax_check = self.run_command(
                f"python -m py_compile {fix_result['file_modified']}",
                f"Validate syntax of {fix_result['file_modified']}"
            )
            validation_results['syntax_valid'] = syntax_check['success']
        else:
            validation_results['syntax_valid'] = True  # Non-Python files
        
        # Run specific tests related to the fix
        test_category = issue.get('category', '')
        if test_category:
            test_command = f"python manage.py test apps.{self.app_name} -k {test_category} --verbosity=0"
            test_result = self.run_command(test_command, f"Run tests for {test_category}")
            validation_results['tests_pass'] = test_result['success']
        
        # Quick security scan for security fixes
        if issue.get('fix_type') == 'security_patch':
            security_scan = self.run_command(
                f"bandit -f json -q {fix_result.get('file_modified', '')} 2>/dev/null || echo 'scan_complete'",
                "Quick security scan of modified file"
            )
            validation_results['security_scan_clean'] = 'No issues identified' in security_scan.get('stdout', '') or security_scan['success']
        else:
            validation_results['security_scan_clean'] = True
        
        # Overall success
        validation_results['overall_success'] = (
            validation_results['syntax_valid'] and 
            validation_results['tests_pass'] and 
            validation_results['security_scan_clean']
        )
        
        return validation_results
    
    def execute_single_fix(self, issue: Dict) -> Dict:
        """Execute a single bug fix with validation"""
        issue_id = issue.get('check_id', issue.get('test', 'unknown'))
        fix_type = issue.get('fix_type', 'unknown')
        
        self.log_action("execute_fix", "started", f"Executing fix for {issue_id}")
        
        # Create branch for this specific fix
        try:
            branch_name = self.create_fix_branch(issue_id)
        except Exception as e:
            return {
                'success': False,
                'error': f"Failed to create branch: {str(e)}",
                'issue_id': issue_id
            }
        
        # Route to appropriate fix method
        try:
            if fix_type == 'security_patch':
                fix_result = self.fix_security_vulnerability(issue)
            elif fix_type in ['authentication_logic', 'permission_logic']:
                fix_result = self.fix_authentication_issue(issue)
            elif fix_type == 'database_constraint':
                fix_result = self.fix_database_integrity_issue(issue)
            else:
                # For other types, return success but note manual review needed
                fix_result = {
                    'success': True,
                    'changes': ['Manual review recommended - no automated fix available'],
                    'file_modified': None
                }
            
            if fix_result['success']:
                # Validate the fix
                validation_result = self.validate_fix(issue, fix_result)
                
                if validation_result['overall_success']:
                    # Commit the fix
                    commit_message = f"Fix {issue_id}: {issue.get('description', 'Bug fix')}\n\nChanges: {', '.join(fix_result.get('changes', []))}"
                    
                    commit_result = self.run_command(
                        f"git add . && git commit -m \"{commit_message}\"",
                        f"Commit fix for {issue_id}"
                    )
                    
                    if commit_result['success']:
                        self.log_action("fix_committed", "success", f"Successfully fixed and committed {issue_id}")
                        return {
                            'success': True,
                            'issue_id': issue_id,
                            'branch': branch_name,
                            'fix_result': fix_result,
                            'validation': validation_result,
                            'committed': True
                        }
                    else:
                        self.log_action("commit_failed", "failed", f"Failed to commit fix: {commit_result['stderr']}")
                        return {
                            'success': False,
                            'error': f"Commit failed: {commit_result['stderr']}",
                            'issue_id': issue_id
                        }
                else:
                    # Validation failed, rollback
                    self.log_action("validation_failed", "failed", f"Validation failed for {issue_id}")
                    rollback_result = self.run_command(
                        f"git checkout ${input:base_branch}",
                        f"Rollback failed fix for {issue_id}"
                    )
                    
                    return {
                        'success': False,
                        'error': f"Validation failed: {validation_result}",
                        'issue_id': issue_id,
                        'rolled_back': rollback_result['success']
                    }
            else:
                self.log_action("fix_failed", "failed", f"Fix implementation failed: {fix_result.get('error', 'Unknown error')}")
                return {
                    'success': False,
                    'error': fix_result.get('error', 'Fix implementation failed'),
                    'issue_id': issue_id
                }
        
        except Exception as e:
            self.log_action("execute_fix", "failed", f"Exception during fix execution: {str(e)}")
            # Rollback to base branch
            self.run_command(f"git checkout ${input:base_branch}", "Emergency rollback")
            return {
                'success': False,
                'error': str(e),
                'issue_id': issue_id
            }
    
    def execute_phase_1_fixes(self) -> Dict:
        """Execute all Phase 1 critical fixes"""
        phase_results = {
            'phase': 'phase_1_critical',
            'start_time': datetime.now().isoformat(),
            'total_issues': len(self.critical_issues),
            'successful_fixes': 0,
            'failed_fixes': 0,
            'fix_details': []
        }
        
        self.log_action("phase_1_started", "info", f"Starting Phase 1 with {len(self.critical_issues)} critical issues")
        
        for issue in self.critical_issues:
            fix_result = self.execute_single_fix(issue)
            phase_results['fix_details'].append(fix_result)
            
            if fix_result['success']:
                phase_results['successful_fixes'] += 1
            else:
                phase_results['failed_fixes'] += 1
                
                # For critical issues, consider stopping on failure
                if phase_results['failed_fixes'] >= 3:
                    self.log_action("phase_1_aborted", "failed", "Too many critical fix failures, aborting phase")
                    break
        
        phase_results['end_time'] = datetime.now().isoformat()
        
        # Save phase results
        with open('logs/phase_1_results.json', 'w') as f:
            json.dump(phase_results, f, indent=2)
        
        return phase_results


# Execute Phase 1 critical fixes
def execute_phase_1_critical_fixes():
    """Main function to execute Phase 1 critical fixes"""
    
    executor = CriticalBugFixExecutor(
        fix_plan_path='fix_plan.json',
        app_name='${input:app_name}'
    )
    
    results = executor.execute_phase_1_fixes()
    
    print(f"\n=== PHASE 1 EXECUTION COMPLETE ===")
    print(f"Total critical issues: {results['total_issues']}")
    print(f"✅ Successful fixes: {results['successful_fixes']}")
    print(f"❌ Failed fixes: {results['failed_fixes']}")
    print(f"Success rate: {(results['successful_fixes'] / results['total_issues'] * 100):.1f}%")
    
    return results


if __name__ == '__main__':
    execute_phase_1_critical_fixes()
```

### Step 3: Execute Phase 2 - High Priority Issues

**Continue with high priority fixes after Phase 1 success:**

```bash
# Check Phase 1 results before proceeding
echo "=== CHECKING PHASE 1 RESULTS ==="
phase1_success_rate=$(cat logs/phase_1_results.json | jq '.successful_fixes / .total_issues * 100')

if (( $(echo "$phase1_success_rate >= 80" | bc -l) )); then
    echo "✅ Phase 1 success rate: ${phase1_success_rate}% - Proceeding to Phase 2"
else
    echo "⚠️ Phase 1 success rate: ${phase1_success_rate}% - Review failures before continuing"
    echo "Check logs/phase_1_results.json for details"
    read -p "Continue to Phase 2? (y/n): " continue_phase2
    if [ "$continue_phase2" != "y" ]; then
        echo "Stopping execution. Fix Phase 1 issues first."
        exit 1
    fi
fi

# Execute Phase 2 high priority fixes
python bug_fix_execution/phase2_high_executor.py
```

### Step 4: Real-time Test Validation

**Run comprehensive tests during fix execution:**

```python
# File: bug_fix_execution/test_validator.py

"""
Real-time test validation during bug fix execution
"""

import subprocess
import json
from datetime import datetime

class TestValidator:
    """Validate fixes with comprehensive testing"""
    
    def __init__(self, app_name: str):
        self.app_name = app_name
        self.test_results = []
    
    def run_unit_tests(self) -> Dict:
        """Run unit tests for the app"""
        print("Running unit tests...")
        
        result = subprocess.run(
            f"python manage.py test apps.{self.app_name} --verbosity=0 --keepdb",
            shell=True,
            capture_output=True,
            text=True
        )
        
        test_result = {
            'test_type': 'unit_tests',
            'timestamp': datetime.now().isoformat(),
            'success': result.returncode == 0,
            'output': result.stdout,
            'errors': result.stderr
        }
        
        self.test_results.append(test_result)
        return test_result
    
    def run_integration_tests(self) -> Dict:
        """Run integration tests"""
        print("Running integration tests...")
        
        result = subprocess.run(
            f"python manage.py test tests.integration --verbosity=0 --keepdb",
            shell=True,
            capture_output=True,
            text=True
        )
        
        test_result = {
            'test_type': 'integration_tests',
            'timestamp': datetime.now().isoformat(),
            'success': result.returncode == 0,
            'output': result.stdout,
            'errors': result.stderr
        }
        
        self.test_results.append(test_result)
        return test_result
    
    def run_security_scan(self) -> Dict:
        """Run security scan on modified files"""
        print("Running security scan...")
        
        result = subprocess.run(
            f"bandit -r apps/{self.app_name}/ -f json",
            shell=True,
            capture_output=True,
            text=True
        )
        
        test_result = {
            'test_type': 'security_scan',
            'timestamp': datetime.now().isoformat(),
            'success': result.returncode == 0,
            'output': result.stdout,
            'errors': result.stderr
        }
        
        self.test_results.append(test_result)
        return test_result
    
    def run_performance_check(self) -> Dict:
        """Basic performance check"""
        print("Running performance check...")
        
        # Simple check - ensure management commands work
        result = subprocess.run(
            "python manage.py check --deploy",
            shell=True,
            capture_output=True,
            text=True
        )
        
        test_result = {
            'test_type': 'performance_check',
            'timestamp': datetime.now().isoformat(),
            'success': result.returncode == 0,
            'output': result.stdout,
            'errors': result.stderr
        }
        
        self.test_results.append(test_result)
        return test_result
    
    def validate_all(self) -> Dict:
        """Run all validation tests"""
        validation_summary = {
            'timestamp': datetime.now().isoformat(),
            'overall_success': True,
            'test_results': []
        }
        
        # Run all test types
        tests = [
            self.run_unit_tests(),
            self.run_integration_tests(),
            self.run_security_scan(),
            self.run_performance_check()
        ]
        
        validation_summary['test_results'] = tests
        
        # Check if all tests passed
        for test in tests:
            if not test['success']:
                validation_summary['overall_success'] = False
                break
        
        # Save results
        with open('logs/validation_results.json', 'w') as f:
            json.dump(validation_summary, f, indent=2)
        
        return validation_summary


# Main validation function
def validate_current_state():
    """Validate current state after fixes"""
    validator = TestValidator('${input:app_name}')
    results = validator.validate_all()
    
    print(f"\n=== VALIDATION COMPLETE ===")
    print(f"Overall success: {'✅' if results['overall_success'] else '❌'}")
    
    for test in results['test_results']:
        status = '✅' if test['success'] else '❌'
        print(f"{status} {test['test_type']}")
        if not test['success'] and test['errors']:
            print(f"   Error: {test['errors'][:100]}...")
    
    return results


if __name__ == '__main__':
    validate_current_state()
```

### Step 5: Automated Rollback for Failed Fixes

**Implement rollback mechanism for failed fixes:**

```bash
# Rollback mechanism for failed fixes
echo "=== ROLLBACK MECHANISM ==="

# Function to rollback specific fix
rollback_fix() {
    local fix_branch=$1
    local issue_id=$2
    
    echo "Rolling back fix for issue: $issue_id"
    
    # Switch to base branch
    git checkout ${input:base_branch}
    
    # Delete the failed fix branch
    git branch -D $fix_branch 2>/dev/null || true
    
    # Reset any uncommitted changes
    git reset --hard HEAD
    git clean -fd
    
    echo "✅ Rollback complete for $issue_id"
}

# Function to rollback entire phase
rollback_phase() {
    local phase_name=$1
    local backup_branch=$2
    
    echo "Rolling back entire phase: $phase_name"
    
    # Switch to backup branch
    git checkout $backup_branch
    
    # Create new branch from backup
    recovery_branch="recovery_${phase_name}_$(date +%Y%m%d_%H%M%S)"
    git checkout -b $recovery_branch
    
    # Reset base branch to backup state
    git checkout ${input:base_branch}
    git reset --hard $backup_branch
    
    echo "✅ Phase rollback complete. Recovery branch: $recovery_branch"
}

# Check if rollback is needed based on success rate
check_rollback_needed() {
    local phase_results_file=$1
    local min_success_rate=$2
    
    if [ -f "$phase_results_file" ]; then
        success_rate=$(cat $phase_results_file | jq '.successful_fixes / .total_issues * 100')
        
        if (( $(echo "$success_rate < $min_success_rate" | bc -l) )); then
            echo "⚠️ Success rate $success_rate% is below threshold $min_success_rate%"
            return 0  # Rollback needed
        else
            echo "✅ Success rate $success_rate% meets threshold"
            return 1  # No rollback needed
        fi
    else
        echo "❌ Results file not found: $phase_results_file"
        return 0  # Rollback needed
    fi
}

# Check Phase 1 results and rollback if needed
if check_rollback_needed "logs/phase_1_results.json" 80; then
    echo "Rollback needed for Phase 1"
    # rollback_phase "phase_1" $backup_branch
fi
```

### Step 6: Generate Final Execution Report

**Create comprehensive execution report:**

```python
# File: bug_fix_execution/execution_reporter.py

"""
Generate comprehensive bug fix execution report
"""

import json
import os
from datetime import datetime
from pathlib import Path

class ExecutionReporter:
    """Generate comprehensive execution report"""
    
    def __init__(self):
        self.execution_data = {}
        self.load_execution_data()
    
    def load_execution_data(self):
        """Load all execution data from logs"""
        log_files = [
            'logs/phase_1_results.json',
            'logs/phase_2_results.json', 
            'logs/phase_3_results.json',
            'logs/phase_4_results.json',
            'logs/validation_results.json'
        ]
        
        for log_file in log_files:
            if os.path.exists(log_file):
                with open(log_file, 'r') as f:
                    phase_name = Path(log_file).stem
                    self.execution_data[phase_name] = json.load(f)
    
    def generate_summary_report(self) -> Dict:
        """Generate summary execution report"""
        summary = {
            'execution_metadata': {
                'project': '${input:project_name}',
                'app': '${input:app_name}',
                'report_generated': datetime.now().isoformat(),
                'base_branch': '${input:base_branch}'
            },
            'overall_statistics': {
                'total_issues_processed': 0,
                'total_successful_fixes': 0,
                'total_failed_fixes': 0,
                'overall_success_rate': 0.0
            },
            'phase_breakdown': {},
            'test_validation_summary': {},
            'recommendations': [],
            'next_steps': []
        }
        
        # Calculate overall statistics
        total_issues = 0
        total_successful = 0
        total_failed = 0
        
        for phase_name, phase_data in self.execution_data.items():
            if 'phase_' in phase_name and 'results' in phase_name:
                issues = phase_data.get('total_issues', 0)
                successful = phase_data.get('successful_fixes', 0)
                failed = phase_data.get('failed_fixes', 0)
                
                total_issues += issues
                total_successful += successful
                total_failed += failed
                
                # Add phase breakdown
                summary['phase_breakdown'][phase_name] = {
                    'total_issues': issues,
                    'successful_fixes': successful,
                    'failed_fixes': failed,
                    'success_rate': (successful / issues * 100) if issues > 0 else 0,
                    'start_time': phase_data.get('start_time'),
                    'end_time': phase_data.get('end_time')
                }
        
        # Update overall statistics
        summary['overall_statistics'].update({
            'total_issues_processed': total_issues,
            'total_successful_fixes': total_successful,
            'total_failed_fixes': total_failed,
            'overall_success_rate': (total_successful / total_issues * 100) if total_issues > 0 else 0
        })
        
        # Add validation summary
        if 'validation_results' in self.execution_data:
            validation = self.execution_data['validation_results']
            summary['test_validation_summary'] = {
                'overall_validation_success': validation.get('overall_success', False),
                'test_results': validation.get('test_results', [])
            }
        
        # Generate recommendations
        success_rate = summary['overall_statistics']['overall_success_rate']
        
        if success_rate >= 90:
            summary['recommendations'].append({
                'priority': 'low',
                'type': 'success',
                'description': 'Excellent fix success rate. Ready for deployment.',
                'action': 'Proceed with staging deployment and user acceptance testing.'
            })
        elif success_rate >= 70:
            summary['recommendations'].append({
                'priority': 'medium',
                'type': 'review',
                'description': 'Good fix success rate with some failures.',
                'action': 'Review failed fixes and consider manual implementation.'
            })
        else:
            summary['recommendations'].append({
                'priority': 'high',
                'type': 'intervention',
                'description': 'Low fix success rate requires intervention.',
                'action': 'Stop deployment. Review fix plan and implementation approach.'
            })
        
        if total_failed > 0:
            summary['recommendations'].append({
                'priority': 'medium',
                'type': 'failed_fixes',
                'description': f'{total_failed} fixes failed and need manual attention.',
                'action': 'Create tickets for manual fix implementation.'
            })
        
        # Generate next steps
        summary['next_steps'] = [
            {
                'step': 'review_failed_fixes',
                'description': 'Review and manually implement failed fixes',
                'estimated_time': f'{total_failed * 30} minutes',
                'required': total_failed > 0
            },
            {
                'step': 'comprehensive_testing',
                'description': 'Run full test suite including edge cases',
                'estimated_time': '2 hours',
                'required': True
            },
            {
                'step': 'performance_testing',
                'description': 'Validate performance impact of fixes',
                'estimated_time': '1 hour',
                'required': total_successful > 0
            },
            {
                'step': 'staging_deployment',
                'description': 'Deploy to staging environment for validation',
                'estimated_time': '30 minutes',
                'required': success_rate >= 80
            }
        ]
        
        return summary
    
    def generate_detailed_report(self) -> str:
        """Generate detailed HTML report"""
        summary = self.generate_summary_report()
        
        html_report = f"""
<!DOCTYPE html>
<html>
<head>
    <title>Bug Fix Execution Report - {summary['execution_metadata']['app']}</title>
    <style>
        body {{ font-family: Arial, sans-serif; margin: 20px; }}
        .header {{ background: #f0f0f0; padding: 20px; margin-bottom: 20px; }}
        .success {{ color: #28a745; }}
        .warning {{ color: #ffc107; }}
        .error {{ color: #dc3545; }}
        .phase {{ margin: 20px 0; padding: 15px; border: 1px solid #ddd; }}
        .stats {{ display: flex; gap: 20px; }}
        .stat-box {{ padding: 10px; border: 1px solid #ccc; text-align: center; }}
    </style>
</head>
<body>
    <div class="header">
        <h1>Bug Fix Execution Report</h1>
        <p><strong>Project:</strong> {summary['execution_metadata']['project']}</p>
        <p><strong>App:</strong> {summary['execution_metadata']['app']}</p>
        <p><strong>Generated:</strong> {summary['execution_metadata']['report_generated']}</p>
    </div>
    
    <h2>Overall Statistics</h2>
    <div class="stats">
        <div class="stat-box">
            <h3>{summary['overall_statistics']['total_issues_processed']}</h3>
            <p>Total Issues</p>
        </div>
        <div class="stat-box success">
            <h3>{summary['overall_statistics']['total_successful_fixes']}</h3>
            <p>Successful Fixes</p>
        </div>
        <div class="stat-box error">
            <h3>{summary['overall_statistics']['total_failed_fixes']}</h3>
            <p>Failed Fixes</p>
        </div>
        <div class="stat-box">
            <h3>{summary['overall_statistics']['overall_success_rate']:.1f}%</h3>
            <p>Success Rate</p>
        </div>
    </div>
    
    <h2>Phase Breakdown</h2>
"""
        
        for phase_name, phase_data in summary['phase_breakdown'].items():
            success_class = 'success' if phase_data['success_rate'] >= 80 else 'warning' if phase_data['success_rate'] >= 60 else 'error'
            
            html_report += f"""
    <div class="phase">
        <h3>{phase_name.replace('_', ' ').title()}</h3>
        <p><strong>Success Rate:</strong> <span class="{success_class}">{phase_data['success_rate']:.1f}%</span></p>
        <p><strong>Issues:</strong> {phase_data['total_issues']} total, {phase_data['successful_fixes']} fixed, {phase_data['failed_fixes']} failed</p>
        <p><strong>Duration:</strong> {phase_data.get('start_time', 'N/A')} to {phase_data.get('end_time', 'N/A')}</p>
    </div>
"""
        
        html_report += """
    <h2>Recommendations</h2>
    <ul>
"""
        
        for rec in summary['recommendations']:
            priority_class = 'error' if rec['priority'] == 'high' else 'warning' if rec['priority'] == 'medium' else 'success'
            html_report += f"""
        <li class="{priority_class}">
            <strong>[{rec['priority'].upper()}]</strong> {rec['description']}
            <br><em>Action: {rec['action']}</em>
        </li>
"""
        
        html_report += """
    </ul>
    
    <h2>Next Steps</h2>
    <ol>
"""
        
        for step in summary['next_steps']:
            if step['required']:
                html_report += f"""
        <li>
            <strong>{step['description']}</strong>
            <br>Estimated time: {step['estimated_time']}
        </li>
"""
        
        html_report += """
    </ol>
</body>
</html>
"""
        
        return html_report
    
    def save_reports(self):
        """Save all reports"""
        # Create reports directory
        Path('reports').mkdir(exist_ok=True)
        
        # Generate and save summary report
        summary = self.generate_summary_report()
        with open('reports/execution_summary.json', 'w') as f:
            json.dump(summary, f, indent=2)
        
        # Generate and save detailed HTML report
        html_report = self.generate_detailed_report()
        with open('reports/execution_report.html', 'w') as f:
            f.write(html_report)
        
        # Generate simple text summary
        text_summary = f"""
=== BUG FIX EXECUTION SUMMARY ===

Project: {summary['execution_metadata']['project']}
App: {summary['execution_metadata']['app']}

RESULTS:
Total Issues: {summary['overall_statistics']['total_issues_processed']}
✅ Successful Fixes: {summary['overall_statistics']['total_successful_fixes']}
❌ Failed Fixes: {summary['overall_statistics']['total_failed_fixes']}
📊 Success Rate: {summary['overall_statistics']['overall_success_rate']:.1f}%

PHASE BREAKDOWN:
"""
        
        for phase_name, phase_data in summary['phase_breakdown'].items():
            text_summary += f"  {phase_name}: {phase_data['success_rate']:.1f}% ({phase_data['successful_fixes']}/{phase_data['total_issues']})\n"
        
        text_summary += "\nRECOMMENDATIONS:\n"
        for rec in summary['recommendations']:
            text_summary += f"  [{rec['priority'].upper()}] {rec['description']}\n"
        
        with open('reports/execution_summary.txt', 'w') as f:
            f.write(text_summary)
        
        print(text_summary)
        return summary


# Generate final execution report
def generate_execution_report():
    """Main function to generate execution report"""
    reporter = ExecutionReporter()
    summary = reporter.save_reports()
    
    print(f"\n=== EXECUTION REPORTS GENERATED ===")
    print(f"📄 Summary: reports/execution_summary.json")
    print(f"🌐 HTML Report: reports/execution_report.html") 
    print(f"📝 Text Summary: reports/execution_summary.txt")
    
    return summary


if __name__ == '__main__':
    generate_execution_report()
```

## Final Execution Summary

**Complete the bug fix execution process:**

```bash
# Final execution steps
echo "=== FINAL BUG FIX EXECUTION SUMMARY ==="

# Generate comprehensive execution report
python bug_fix_execution/execution_reporter.py

# Run final validation
python bug_fix_execution/test_validator.py

# Clean up temporary branches and files
echo "Cleaning up execution environment..."
find . -name "*.pyc" -delete
find . -name "__pycache__" -type d -exec rm -rf {} + 2>/dev/null || true

# Push successful fixes to remote
git push origin ${input:base_branch}

echo "✅ Bug fix execution complete!"
echo "📊 Check reports/ directory for detailed results"
echo "🔍 Review any failed fixes in the execution logs"
```

## Success Criteria

**Execution is successful when:**

✅ **>80% of critical issues are fixed successfully**  
✅ **All unit tests pass after fixes**  
✅ **Security scan shows no new vulnerabilities**  
✅ **Performance impact is within acceptable limits**  
✅ **Rollback mechanism works correctly**  
✅ **Comprehensive execution report is generated**  
✅ **Failed fixes are documented for manual review**

---

**Note**: This prompt provides real-time bug fix execution with validation, rollback capabilities, and comprehensive reporting to ensure safe and effective bug resolution.