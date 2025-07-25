---
description: Automated bug fixing planner based on unittest failures and code review findings from prompts 06 and 07
---

# Django Design Detail (DD) Bug Fix Planner & Implementation

This prompt analyzes bugs and issues found in unit tests (prompt 06) and code review/security scan (prompt 07), creates a comprehensive fix plan, and implements the necessary corrections to ensure code quality and functionality.

## Prerequisites

**Required Input:**
- 📋 **Unit Test Report**: Output from `06_django_dd_unittest_generator.prompt.md`
- 🔍 **Code Review Report**: Output from `07_django_dd_code_review_scanner.prompt.md`
- 📦 **Project Name**: `${input:project_name}`
- 🏷️ **App Name**: `${input:app_name}`
- 📄 **Test Report Path**: `${input:test_report_path}`
- 📄 **Review Report Path**: `${input:review_report_path}`

## Bug Fix Workflow

### Step 1: Analyze Test Failures and Review Issues

**Load and analyze reports from previous prompts:**

```bash
# Load unit test report from prompt 06
echo "=== LOADING UNIT TEST REPORT ==="
cat ${input:test_report_path}/test_report.json

# Load code review report from prompt 07  
echo "=== LOADING CODE REVIEW REPORT ==="
cat ${input:review_report_path}/review_report.json
```

**Analyze test failures:**

```python
# File: bug_analysis/test_failure_analyzer.py

"""
Analyze unit test failures and categorize issues
"""

import json
from datetime import datetime
from typing import Dict, List, Any
from pathlib import Path

class TestFailureAnalyzer:
    """Analyze unit test failures and categorize bugs"""
    
    def __init__(self, test_report_path: str, app_name: str):
        self.test_report_path = test_report_path
        self.app_name = app_name
        self.test_report = self.load_test_report()
        self.bug_categories = {
            'model_validation': [],
            'serializer_validation': [],
            'view_logic': [],
            'authentication': [],
            'permissions': [],
            'business_logic': [],
            'database_integrity': [],
            'api_contract': []
        }
    
    def load_test_report(self) -> Dict:
        """Load test report from prompt 06"""
        try:
            with open(f"{self.test_report_path}/test_report.json", 'r') as f:
                return json.load(f)
        except FileNotFoundError:
            print(f"Test report not found at {self.test_report_path}")
            return {}
    
    def categorize_test_failures(self) -> Dict:
        """Categorize test failures by type"""
        failures = self.test_report.get('failures', [])
        errors = self.test_report.get('errors', [])
        
        all_issues = failures + errors
        
        for issue in all_issues:
            test_name = issue.get('test_name', '')
            error_message = issue.get('error_message', '')
            traceback = issue.get('traceback', '')
            
            # Categorize based on test name and error patterns
            if 'model' in test_name.lower():
                if 'validation' in error_message.lower():
                    self.bug_categories['model_validation'].append({
                        'test': test_name,
                        'error': error_message,
                        'traceback': traceback,
                        'severity': self.determine_severity(error_message),
                        'fix_type': 'model_field_validation'
                    })
                elif 'integrity' in error_message.lower():
                    self.bug_categories['database_integrity'].append({
                        'test': test_name,
                        'error': error_message,
                        'traceback': traceback,
                        'severity': 'high',
                        'fix_type': 'database_constraint'
                    })
            
            elif 'serializer' in test_name.lower():
                self.bug_categories['serializer_validation'].append({
                    'test': test_name,
                    'error': error_message,
                    'traceback': traceback,
                    'severity': self.determine_severity(error_message),
                    'fix_type': 'serializer_logic'
                })
            
            elif 'view' in test_name.lower() or 'api' in test_name.lower():
                if 'permission' in error_message.lower() or 'forbidden' in error_message.lower():
                    self.bug_categories['permissions'].append({
                        'test': test_name,
                        'error': error_message,
                        'traceback': traceback,
                        'severity': 'high',
                        'fix_type': 'permission_logic'
                    })
                elif 'authentication' in error_message.lower() or 'unauthorized' in error_message.lower():
                    self.bug_categories['authentication'].append({
                        'test': test_name,
                        'error': error_message,
                        'traceback': traceback,
                        'severity': 'high',
                        'fix_type': 'authentication_logic'
                    })
                elif 'status' in error_message.lower() or 'response' in error_message.lower():
                    self.bug_categories['api_contract'].append({
                        'test': test_name,
                        'error': error_message,
                        'traceback': traceback,
                        'severity': 'medium',
                        'fix_type': 'api_response'
                    })
                else:
                    self.bug_categories['view_logic'].append({
                        'test': test_name,
                        'error': error_message,
                        'traceback': traceback,
                        'severity': self.determine_severity(error_message),
                        'fix_type': 'view_implementation'
                    })
            
            else:
                # General business logic issues
                self.bug_categories['business_logic'].append({
                    'test': test_name,
                    'error': error_message,
                    'traceback': traceback,
                    'severity': self.determine_severity(error_message),
                    'fix_type': 'business_rule'
                })
        
        return self.bug_categories
    
    def determine_severity(self, error_message: str) -> str:
        """Determine bug severity based on error message"""
        error_lower = error_message.lower()
        
        if any(keyword in error_lower for keyword in ['critical', 'security', 'data loss', 'corruption']):
            return 'critical'
        elif any(keyword in error_lower for keyword in ['authentication', 'permission', 'unauthorized']):
            return 'high'
        elif any(keyword in error_lower for keyword in ['validation', 'constraint', 'integrity']):
            return 'high'
        elif any(keyword in error_lower for keyword in ['assertion', 'expected', 'actual']):
            return 'medium'
        else:
            return 'low'
    
    def generate_test_fix_summary(self) -> Dict:
        """Generate summary of test failures to fix"""
        total_failures = sum(len(issues) for issues in self.bug_categories.values())
        
        severity_count = {'critical': 0, 'high': 0, 'medium': 0, 'low': 0}
        for issues in self.bug_categories.values():
            for issue in issues:
                severity_count[issue['severity']] += 1
        
        return {
            'total_failures': total_failures,
            'severity_breakdown': severity_count,
            'categories': {k: len(v) for k, v in self.bug_categories.items()},
            'detailed_issues': self.bug_categories
        }


class CodeReviewIssueAnalyzer:
    """Analyze code review issues and categorize problems"""
    
    def __init__(self, review_report_path: str, app_name: str):
        self.review_report_path = review_report_path
        self.app_name = app_name
        self.review_report = self.load_review_report()
        self.issue_categories = {
            'security_vulnerabilities': [],
            'code_quality': [],
            'performance': [],
            'django_best_practices': [],
            'dd_compliance': [],
            'documentation': []
        }
    
    def load_review_report(self) -> Dict:
        """Load review report from prompt 07"""
        try:
            with open(f"{self.review_report_path}/review_report.json", 'r') as f:
                return json.load(f)
        except FileNotFoundError:
            print(f"Review report not found at {self.review_report_path}")
            return {}
    
    def categorize_review_issues(self) -> Dict:
        """Categorize code review issues"""
        quality_issues = self.review_report.get('quality_review', {}).get('failed_checks', [])
        security_issues = self.review_report.get('security_review', {}).get('failed_checks', [])
        
        # Process quality issues
        for issue in quality_issues:
            check_id = issue.get('check_id', '')
            category = issue.get('category', '')
            severity = issue.get('severity', 'medium')
            description = issue.get('description', '')
            file_path = issue.get('file_path', '')
            line_number = issue.get('line_number', 0)
            
            if 'DBP' in check_id:  # Django Best Practices
                self.issue_categories['django_best_practices'].append({
                    'check_id': check_id,
                    'description': description,
                    'file_path': file_path,
                    'line_number': line_number,
                    'severity': severity,
                    'fix_type': 'django_pattern'
                })
            elif 'DDR' in check_id:  # DD Requirements
                self.issue_categories['dd_compliance'].append({
                    'check_id': check_id,
                    'description': description,
                    'file_path': file_path,
                    'line_number': line_number,
                    'severity': severity,
                    'fix_type': 'dd_requirement'
                })
            elif 'CQS' in check_id:  # Code Quality Standards
                self.issue_categories['code_quality'].append({
                    'check_id': check_id,
                    'description': description,
                    'file_path': file_path,
                    'line_number': line_number,
                    'severity': severity,
                    'fix_type': 'code_style'
                })
            elif 'PER' in check_id:  # Performance
                self.issue_categories['performance'].append({
                    'check_id': check_id,
                    'description': description,
                    'file_path': file_path,
                    'line_number': line_number,
                    'severity': severity,
                    'fix_type': 'performance_optimization'
                })
        
        # Process security issues
        for issue in security_issues:
            check_id = issue.get('check_id', '')
            severity = issue.get('severity', 'high')
            description = issue.get('description', '')
            file_path = issue.get('file_path', '')
            line_number = issue.get('line_number', 0)
            
            self.issue_categories['security_vulnerabilities'].append({
                'check_id': check_id,
                'description': description,
                'file_path': file_path,
                'line_number': line_number,
                'severity': severity,
                'fix_type': 'security_patch'
            })
        
        return self.issue_categories
    
    def generate_review_fix_summary(self) -> Dict:
        """Generate summary of review issues to fix"""
        total_issues = sum(len(issues) for issues in self.issue_categories.values())
        
        severity_count = {'critical': 0, 'high': 0, 'medium': 0, 'low': 0}
        for issues in self.issue_categories.values():
            for issue in issues:
                severity_count[issue['severity']] += 1
        
        return {
            'total_issues': total_issues,
            'severity_breakdown': severity_count,
            'categories': {k: len(v) for k, v in self.issue_categories.items()},
            'detailed_issues': self.issue_categories
        }


# Main analysis function
def analyze_bugs_and_issues():
    """Main function to analyze all bugs and issues"""
    
    print("=== ANALYZING TEST FAILURES ===")
    test_analyzer = TestFailureAnalyzer(
        test_report_path='${input:test_report_path}',
        app_name='${input:app_name}'
    )
    
    test_categories = test_analyzer.categorize_test_failures()
    test_summary = test_analyzer.generate_test_fix_summary()
    
    print(f"Found {test_summary['total_failures']} test failures")
    print(f"Severity breakdown: {test_summary['severity_breakdown']}")
    
    print("\n=== ANALYZING CODE REVIEW ISSUES ===")
    review_analyzer = CodeReviewIssueAnalyzer(
        review_report_path='${input:review_report_path}',
        app_name='${input:app_name}'
    )
    
    review_categories = review_analyzer.categorize_review_issues()
    review_summary = review_analyzer.generate_review_fix_summary()
    
    print(f"Found {review_summary['total_issues']} code review issues")
    print(f"Severity breakdown: {review_summary['severity_breakdown']}")
    
    return {
        'test_analysis': {
            'categories': test_categories,
            'summary': test_summary
        },
        'review_analysis': {
            'categories': review_categories,
            'summary': review_summary
        }
    }


if __name__ == '__main__':
    analyze_bugs_and_issues()
```

### Step 2: Prioritize and Plan Bug Fixes

**Create comprehensive bug fix plan with prioritization:**

```python
# File: bug_analysis/bug_fix_planner.py

"""
Create comprehensive bug fix plan with prioritization
"""

import json
from datetime import datetime, timedelta
from typing import Dict, List, Any

class BugFixPlanner:
    """Plan and prioritize bug fixes"""
    
    def __init__(self, analysis_results: Dict, app_name: str):
        self.analysis_results = analysis_results
        self.app_name = app_name
        self.fix_plan = {
            'metadata': {
                'app_name': app_name,
                'created_at': datetime.now().isoformat(),
                'total_issues': 0
            },
            'priority_phases': {},
            'implementation_timeline': {},
            'risk_assessment': {}
        }
    
    def calculate_priority_score(self, issue: Dict) -> int:
        """Calculate priority score for an issue (0-100, higher = more urgent)"""
        score = 0
        
        # Severity scoring
        severity_scores = {
            'critical': 40,
            'high': 30,
            'medium': 20,
            'low': 10
        }
        score += severity_scores.get(issue.get('severity', 'low'), 10)
        
        # Issue type scoring
        type_scores = {
            'security_patch': 30,
            'authentication_logic': 25,
            'permission_logic': 25,
            'database_constraint': 20,
            'model_field_validation': 15,
            'dd_requirement': 15,
            'business_rule': 10,
            'django_pattern': 10,
            'performance_optimization': 10,
            'code_style': 5
        }
        score += type_scores.get(issue.get('fix_type', 'code_style'), 5)
        
        # Impact scoring
        if 'authentication' in issue.get('description', '').lower():
            score += 15
        if 'data' in issue.get('description', '').lower():
            score += 10
        if 'api' in issue.get('description', '').lower():
            score += 10
        
        return min(score, 100)  # Cap at 100
    
    def group_issues_by_priority(self) -> Dict:
        """Group all issues by priority phases"""
        all_issues = []
        
        # Collect test failures
        test_analysis = self.analysis_results.get('test_analysis', {})
        for category, issues in test_analysis.get('categories', {}).items():
            for issue in issues:
                issue['source'] = 'unit_test'
                issue['category'] = category
                issue['priority_score'] = self.calculate_priority_score(issue)
                all_issues.append(issue)
        
        # Collect review issues  
        review_analysis = self.analysis_results.get('review_analysis', {})
        for category, issues in review_analysis.get('categories', {}).items():
            for issue in issues:
                issue['source'] = 'code_review'
                issue['category'] = category
                issue['priority_score'] = self.calculate_priority_score(issue)
                all_issues.append(issue)
        
        # Sort by priority score (highest first)
        all_issues.sort(key=lambda x: x['priority_score'], reverse=True)
        
        # Group into priority phases
        phases = {
            'phase_1_critical': [],  # Score 70-100
            'phase_2_high': [],      # Score 50-69
            'phase_3_medium': [],    # Score 30-49
            'phase_4_low': []        # Score 0-29
        }
        
        for issue in all_issues:
            score = issue['priority_score']
            if score >= 70:
                phases['phase_1_critical'].append(issue)
            elif score >= 50:
                phases['phase_2_high'].append(issue)
            elif score >= 30:
                phases['phase_3_medium'].append(issue)
            else:
                phases['phase_4_low'].append(issue)
        
        return phases
    
    def create_implementation_timeline(self, phases: Dict) -> Dict:
        """Create implementation timeline for bug fixes"""
        timeline = {}
        start_date = datetime.now()
        
        # Phase 1: Critical issues (immediate - 1 day)
        phase1_issues = phases['phase_1_critical']
        if phase1_issues:
            timeline['phase_1_critical'] = {
                'start_date': start_date.isoformat(),
                'end_date': (start_date + timedelta(days=1)).isoformat(),
                'duration_days': 1,
                'issues_count': len(phase1_issues),
                'description': 'Critical security and functionality issues',
                'parallel_execution': True
            }
        
        # Phase 2: High priority (1-3 days)
        phase2_start = start_date + timedelta(days=1)
        phase2_issues = phases['phase_2_high']
        if phase2_issues:
            timeline['phase_2_high'] = {
                'start_date': phase2_start.isoformat(),
                'end_date': (phase2_start + timedelta(days=2)).isoformat(),
                'duration_days': 2,
                'issues_count': len(phase2_issues),
                'description': 'High priority functionality and compliance issues',
                'parallel_execution': True
            }
        
        # Phase 3: Medium priority (3-5 days)
        phase3_start = phase2_start + timedelta(days=2)
        phase3_issues = phases['phase_3_medium']
        if phase3_issues:
            timeline['phase_3_medium'] = {
                'start_date': phase3_start.isoformat(),
                'end_date': (phase3_start + timedelta(days=2)).isoformat(),
                'duration_days': 2,
                'issues_count': len(phase3_issues),
                'description': 'Code quality and performance improvements',
                'parallel_execution': False
            }
        
        # Phase 4: Low priority (5-7 days)
        phase4_start = phase3_start + timedelta(days=2)
        phase4_issues = phases['phase_4_low']
        if phase4_issues:
            timeline['phase_4_low'] = {
                'start_date': phase4_start.isoformat(),
                'end_date': (phase4_start + timedelta(days=2)).isoformat(),
                'duration_days': 2,
                'issues_count': len(phase4_issues),
                'description': 'Code style and documentation improvements',
                'parallel_execution': False
            }
        
        return timeline
    
    def assess_risks(self, phases: Dict) -> Dict:
        """Assess risks for bug fix implementation"""
        risks = {
            'high_risk_changes': [],
            'breaking_changes': [],
            'data_migration_needed': [],
            'external_dependencies': [],
            'rollback_complexity': 'low'
        }
        
        all_issues = []
        for phase_issues in phases.values():
            all_issues.extend(phase_issues)
        
        for issue in all_issues:
            # Check for high-risk changes
            if issue.get('severity') == 'critical':
                risks['high_risk_changes'].append({
                    'issue': issue.get('description', ''),
                    'file': issue.get('file_path', ''),
                    'risk_level': 'high'
                })
            
            # Check for potential breaking changes
            if 'api' in issue.get('description', '').lower() or 'serializer' in issue.get('category', ''):
                risks['breaking_changes'].append({
                    'issue': issue.get('description', ''),
                    'file': issue.get('file_path', ''),
                    'impact': 'API contract change'
                })
            
            # Check for data migration needs
            if 'model' in issue.get('category', '') or 'database' in issue.get('fix_type', ''):
                risks['data_migration_needed'].append({
                    'issue': issue.get('description', ''),
                    'file': issue.get('file_path', ''),
                    'migration_type': 'schema_change'
                })
        
        # Determine overall rollback complexity
        if risks['high_risk_changes'] or risks['data_migration_needed']:
            risks['rollback_complexity'] = 'high'
        elif risks['breaking_changes']:
            risks['rollback_complexity'] = 'medium'
        
        return risks
    
    def generate_fix_plan(self) -> Dict:
        """Generate complete bug fix plan"""
        phases = self.group_issues_by_priority()
        timeline = self.create_implementation_timeline(phases)
        risks = self.assess_risks(phases)
        
        total_issues = sum(len(issues) for issues in phases.values())
        
        self.fix_plan.update({
            'metadata': {
                'app_name': self.app_name,
                'created_at': datetime.now().isoformat(),
                'total_issues': total_issues
            },
            'priority_phases': phases,
            'implementation_timeline': timeline,
            'risk_assessment': risks
        })
        
        return self.fix_plan
    
    def save_plan(self, output_path: str):
        """Save bug fix plan to file"""
        plan = self.generate_fix_plan()
        
        Path(output_path).mkdir(parents=True, exist_ok=True)
        
        with open(f"{output_path}/bug_fix_plan.json", 'w') as f:
            json.dump(plan, f, indent=2)
        
        return plan


# Generate bug fix plan
def create_bug_fix_plan(analysis_results: Dict):
    """Main function to create bug fix plan"""
    
    planner = BugFixPlanner(
        analysis_results=analysis_results,
        app_name='${input:app_name}'
    )
    
    plan = planner.save_plan('bug_fix_reports')
    
    print(f"Bug fix plan created with {plan['metadata']['total_issues']} total issues")
    print("Priority phases:")
    for phase, issues in plan['priority_phases'].items():
        print(f"  {phase}: {len(issues)} issues")
    
    return plan


if __name__ == '__main__':
    # This would be called with results from previous analysis
    analysis_results = analyze_bugs_and_issues()
    create_bug_fix_plan(analysis_results)
```

### Step 3: Implement Bug Fixes by Priority

**Execute bug fixes in prioritized order:**

```python
# File: bug_fixing/bug_fix_implementer.py

"""
Implement bug fixes based on the prioritized plan
"""

import json
import re
import ast
from pathlib import Path
from typing import Dict, List, Any

class BugFixImplementer:
    """Implement bug fixes based on prioritized plan"""
    
    def __init__(self, fix_plan_path: str, app_name: str):
        self.fix_plan_path = fix_plan_path
        self.app_name = app_name
        self.fix_plan = self.load_fix_plan()
        self.fix_results = {
            'implemented_fixes': [],
            'failed_fixes': [],
            'skipped_fixes': []
        }
    
    def load_fix_plan(self) -> Dict:
        """Load bug fix plan"""
        try:
            with open(f"{self.fix_plan_path}/bug_fix_plan.json", 'r') as f:
                return json.load(f)
        except FileNotFoundError:
            print(f"Fix plan not found at {self.fix_plan_path}")
            return {}
    
    def fix_model_validation_issue(self, issue: Dict) -> Dict:
        """Fix model validation issues"""
        file_path = issue.get('file_path', f'apps/{self.app_name}/models.py')
        error_message = issue.get('error', '')
        
        result = {'status': 'success', 'changes': []}
        
        try:
            with open(file_path, 'r') as f:
                content = f.read()
            
            # Common model validation fixes
            if 'required field' in error_message.lower():
                # Add null=False, blank=False to required fields
                pattern = r'(\w+)\s*=\s*models\.(\w+Field)\((.*?)\)'
                
                def fix_required_field(match):
                    field_name = match.group(1)
                    field_type = match.group(2)
                    field_args = match.group(3)
                    
                    if 'null=' not in field_args and 'blank=' not in field_args:
                        if field_args:
                            field_args += ', null=False, blank=False'
                        else:
                            field_args = 'null=False, blank=False'
                    
                    return f"{field_name} = models.{field_type}({field_args})"
                
                new_content = re.sub(pattern, fix_required_field, content)
                result['changes'].append('Added null=False, blank=False to required fields')
            
            elif 'max_length' in error_message.lower():
                # Add max_length to CharField/TextField
                pattern = r'(\w+)\s*=\s*models\.(Char|Text)Field\(([^)]*)\)'
                
                def fix_max_length(match):
                    field_name = match.group(1)
                    field_type = match.group(2)
                    field_args = match.group(3)
                    
                    if 'max_length=' not in field_args:
                        if field_args:
                            field_args += ', max_length=255'
                        else:
                            field_args = 'max_length=255'
                    
                    return f"{field_name} = models.{field_type}Field({field_args})"
                
                new_content = re.sub(pattern, fix_max_length, content)
                result['changes'].append('Added max_length to CharField/TextField')
            
            else:
                new_content = content
            
            # Write back if changes were made
            if new_content != content:
                with open(file_path, 'w') as f:
                    f.write(new_content)
                result['changes'].append(f"Updated {file_path}")
            else:
                result['status'] = 'no_changes_needed'
        
        except Exception as e:
            result = {'status': 'failed', 'error': str(e)}
        
        return result
    
    def fix_serializer_validation_issue(self, issue: Dict) -> Dict:
        """Fix serializer validation issues"""
        file_path = issue.get('file_path', f'apps/{self.app_name}/serializers.py')
        error_message = issue.get('error', '')
        
        result = {'status': 'success', 'changes': []}
        
        try:
            with open(file_path, 'r') as f:
                content = f.read()
            
            new_content = content
            
            # Add missing validation methods
            if 'validation' in error_message.lower():
                # Add validate method template
                validation_template = '''
    def validate(self, data):
        """Custom validation for the serializer"""
        # Add cross-field validation here
        return data
    
    def validate_field_name(self, value):
        """Validate specific field"""
        # Add field-specific validation here
        return value
'''
                
                # Find the class definition and add validation methods
                class_pattern = r'(class \w+Serializer\([^)]+\):.*?)(class|\Z)'
                
                def add_validation(match):
                    class_content = match.group(1)
                    next_class = match.group(2) if match.group(2) else ''
                    
                    if 'def validate(' not in class_content:
                        class_content += validation_template
                        result['changes'].append('Added validation methods to serializer')
                    
                    return class_content + next_class
                
                new_content = re.sub(class_pattern, add_validation, content, flags=re.DOTALL)
            
            # Write back if changes were made
            if new_content != content:
                with open(file_path, 'w') as f:
                    f.write(new_content)
                result['changes'].append(f"Updated {file_path}")
            else:
                result['status'] = 'no_changes_needed'
        
        except Exception as e:
            result = {'status': 'failed', 'error': str(e)}
        
        return result
    
    def fix_view_logic_issue(self, issue: Dict) -> Dict:
        """Fix view logic issues"""
        file_path = issue.get('file_path', f'apps/{self.app_name}/views.py')
        error_message = issue.get('error', '')
        
        result = {'status': 'success', 'changes': []}
        
        try:
            with open(file_path, 'r') as f:
                content = f.read()
            
            new_content = content
            
            # Fix permission issues
            if 'permission' in error_message.lower():
                # Add permission classes
                if 'permission_classes' not in content:
                    permission_template = '''
from rest_framework.permissions import IsAuthenticated, IsAuthenticatedOrReadOnly

class YourViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthenticated]
'''
                    # Add import at the top
                    if 'from rest_framework.permissions import' not in content:
                        import_line = 'from rest_framework.permissions import IsAuthenticated, IsAuthenticatedOrReadOnly\n'
                        new_content = import_line + content
                        result['changes'].append('Added permission imports')
                    
                    # Add permission_classes to ViewSets
                    class_pattern = r'(class \w+ViewSet\([^)]+\):)'
                    replacement = r'\1\n    permission_classes = [IsAuthenticated]'
                    new_content = re.sub(class_pattern, replacement, new_content)
                    result['changes'].append('Added permission_classes to ViewSets')
            
            # Fix authentication issues
            if 'authentication' in error_message.lower():
                # Add authentication classes
                if 'authentication_classes' not in content:
                    if 'from rest_framework.authentication import' not in content:
                        import_line = 'from rest_framework.authentication import TokenAuthentication\n'
                        new_content = import_line + content
                        result['changes'].append('Added authentication imports')
                    
                    # Add authentication_classes to ViewSets
                    class_pattern = r'(class \w+ViewSet\([^)]+\):)'
                    replacement = r'\1\n    authentication_classes = [TokenAuthentication]'
                    new_content = re.sub(class_pattern, replacement, new_content)
                    result['changes'].append('Added authentication_classes to ViewSets')
            
            # Write back if changes were made
            if new_content != content:
                with open(file_path, 'w') as f:
                    f.write(new_content)
                result['changes'].append(f"Updated {file_path}")
            else:
                result['status'] = 'no_changes_needed'
        
        except Exception as e:
            result = {'status': 'failed', 'error': str(e)}
        
        return result
    
    def fix_security_vulnerability(self, issue: Dict) -> Dict:
        """Fix security vulnerabilities"""
        file_path = issue.get('file_path', '')
        check_id = issue.get('check_id', '')
        
        result = {'status': 'success', 'changes': []}
        
        try:
            with open(file_path, 'r') as f:
                content = f.read()
            
            new_content = content
            
            # Fix specific security issues based on check_id
            if check_id.startswith('OWASP001'):  # Injection prevention
                # Replace raw SQL with ORM
                raw_sql_pattern = r'cursor\.execute\([^)]+\)'
                if re.search(raw_sql_pattern, content):
                    # This is a manual fix that needs review
                    result['status'] = 'manual_review_needed'
                    result['recommendation'] = 'Replace raw SQL queries with Django ORM'
            
            elif check_id.startswith('OWASP002'):  # Authentication
                # Add secure authentication settings
                if 'settings' in file_path:
                    if 'SESSION_COOKIE_SECURE' not in content:
                        new_content += '\nSESSION_COOKIE_SECURE = True\n'
                        result['changes'].append('Added SESSION_COOKIE_SECURE = True')
                    
                    if 'CSRF_COOKIE_SECURE' not in content:
                        new_content += 'CSRF_COOKIE_SECURE = True\n'
                        result['changes'].append('Added CSRF_COOKIE_SECURE = True')
            
            elif check_id.startswith('OWASP003'):  # Sensitive data exposure
                # Remove hardcoded secrets
                secret_pattern = r'(password|secret|key)\s*=\s*[\'"][^\'"]+[\'"]'
                if re.search(secret_pattern, content, re.IGNORECASE):
                    result['status'] = 'manual_review_needed'
                    result['recommendation'] = 'Replace hardcoded secrets with environment variables'
            
            # Write back if changes were made
            if new_content != content:
                with open(file_path, 'w') as f:
                    f.write(new_content)
                result['changes'].append(f"Updated {file_path}")
            
        except Exception as e:
            result = {'status': 'failed', 'error': str(e)}
        
        return result
    
    def fix_performance_issue(self, issue: Dict) -> Dict:
        """Fix performance issues"""
        file_path = issue.get('file_path', f'apps/{self.app_name}/views.py')
        description = issue.get('description', '')
        
        result = {'status': 'success', 'changes': []}
        
        try:
            with open(file_path, 'r') as f:
                content = f.read()
            
            new_content = content
            
            # Add query optimization
            if 'n+1' in description.lower() or 'query' in description.lower():
                # Add select_related and prefetch_related
                viewset_pattern = r'(class \w+ViewSet\([^)]+\):)(.*?)(def get_queryset|\Z)'
                
                def add_optimization(match):
                    class_def = match.group(1)
                    class_body = match.group(2)
                    next_method = match.group(3)
                    
                    if 'get_queryset' not in class_body:
                        optimization = '''
    
    def get_queryset(self):
        """Optimized queryset with select_related and prefetch_related"""
        queryset = super().get_queryset()
        return queryset.select_related().prefetch_related()
'''
                        class_body += optimization
                        result['changes'].append('Added optimized get_queryset method')
                    
                    return class_def + class_body + next_method
                
                new_content = re.sub(viewset_pattern, add_optimization, content, flags=re.DOTALL)
            
            # Write back if changes were made
            if new_content != content:
                with open(file_path, 'w') as f:
                    f.write(new_content)
                result['changes'].append(f"Updated {file_path}")
            else:
                result['status'] = 'no_changes_needed'
        
        except Exception as e:
            result = {'status': 'failed', 'error': str(e)}
        
        return result
    
    def implement_fix(self, issue: Dict) -> Dict:
        """Route issue to appropriate fix method"""
        fix_type = issue.get('fix_type', '')
        
        if fix_type == 'model_field_validation':
            return self.fix_model_validation_issue(issue)
        elif fix_type == 'serializer_logic':
            return self.fix_serializer_validation_issue(issue)
        elif fix_type in ['view_implementation', 'permission_logic', 'authentication_logic']:
            return self.fix_view_logic_issue(issue)
        elif fix_type == 'security_patch':
            return self.fix_security_vulnerability(issue)
        elif fix_type == 'performance_optimization':
            return self.fix_performance_issue(issue)
        else:
            return {
                'status': 'skipped',
                'reason': f'No automated fix available for {fix_type}'
            }
    
    def execute_phase_fixes(self, phase_name: str) -> Dict:
        """Execute all fixes for a specific phase"""
        phases = self.fix_plan.get('priority_phases', {})
        phase_issues = phases.get(phase_name, [])
        
        phase_results = {
            'phase': phase_name,
            'total_issues': len(phase_issues),
            'successful_fixes': 0,
            'failed_fixes': 0,
            'skipped_fixes': 0,
            'manual_review_needed': 0,
            'detailed_results': []
        }
        
        print(f"\n=== EXECUTING {phase_name.upper()} FIXES ===")
        print(f"Total issues to fix: {len(phase_issues)}")
        
        for issue in phase_issues:
            print(f"\nFixing: {issue.get('description', 'Unknown issue')}")
            
            fix_result = self.implement_fix(issue)
            
            # Update counters
            if fix_result['status'] == 'success':
                phase_results['successful_fixes'] += 1
                print("✅ Fixed successfully")
            elif fix_result['status'] == 'failed':
                phase_results['failed_fixes'] += 1
                print(f"❌ Fix failed: {fix_result.get('error', 'Unknown error')}")
            elif fix_result['status'] == 'manual_review_needed':
                phase_results['manual_review_needed'] += 1
                print(f"⚠️ Manual review needed: {fix_result.get('recommendation', '')}")
            else:
                phase_results['skipped_fixes'] += 1
                print(f"⏭️ Skipped: {fix_result.get('reason', 'Unknown reason')}")
            
            # Store detailed result
            phase_results['detailed_results'].append({
                'issue': issue,
                'fix_result': fix_result
            })
        
        return phase_results
    
    def execute_all_fixes(self) -> Dict:
        """Execute all bug fixes according to the plan"""
        all_results = {
            'execution_start': datetime.now().isoformat(),
            'phases': {},
            'summary': {
                'total_issues': 0,
                'successful_fixes': 0,
                'failed_fixes': 0,
                'skipped_fixes': 0,
                'manual_review_needed': 0
            }
        }
        
        phases = self.fix_plan.get('priority_phases', {})
        timeline = self.fix_plan.get('implementation_timeline', {})
        
        for phase_name in ['phase_1_critical', 'phase_2_high', 'phase_3_medium', 'phase_4_low']:
            if phase_name in phases and phases[phase_name]:
                phase_result = self.execute_phase_fixes(phase_name)
                all_results['phases'][phase_name] = phase_result
                
                # Update summary
                all_results['summary']['total_issues'] += phase_result['total_issues']
                all_results['summary']['successful_fixes'] += phase_result['successful_fixes']
                all_results['summary']['failed_fixes'] += phase_result['failed_fixes']
                all_results['summary']['skipped_fixes'] += phase_result['skipped_fixes']
                all_results['summary']['manual_review_needed'] += phase_result['manual_review_needed']
        
        all_results['execution_end'] = datetime.now().isoformat()
        
        # Save results
        with open('bug_fix_reports/fix_execution_results.json', 'w') as f:
            json.dump(all_results, f, indent=2)
        
        return all_results


# Main implementation function
def implement_bug_fixes():
    """Main function to implement bug fixes"""
    
    implementer = BugFixImplementer(
        fix_plan_path='bug_fix_reports',
        app_name='${input:app_name}'
    )
    
    results = implementer.execute_all_fixes()
    
    print(f"\n=== BUG FIX EXECUTION COMPLETE ===")
    print(f"Total issues processed: {results['summary']['total_issues']}")
    print(f"Successful fixes: {results['summary']['successful_fixes']}")
    print(f"Failed fixes: {results['summary']['failed_fixes']}")
    print(f"Skipped fixes: {results['summary']['skipped_fixes']}")
    print(f"Manual review needed: {results['summary']['manual_review_needed']}")
    
    return results


if __name__ == '__main__':
    implement_bug_fixes()
```

### Step 4: Verify Fixes and Re-run Tests

**Verify that bug fixes work correctly:**

```bash
# Verify fixes by re-running tests
echo "=== VERIFYING BUG FIXES ==="

# Re-run unit tests to check if failures are resolved
echo "Running unit tests to verify fixes..."
python manage.py test apps.${input:app_name} --verbosity=2

# Re-run specific failed tests
echo "Re-running previously failed tests..."
# This would run specific tests that previously failed

# Check code quality after fixes
echo "Running code quality checks..."
flake8 apps/${input:app_name}/ --max-line-length=88
black --check apps/${input:app_name}/
isort --check-only apps/${input:app_name}/

# Re-run security scan
echo "Running security scan..."
bandit -r apps/${input:app_name}/ -f json -o security_scan_after_fixes.json

# Check for any new issues introduced
echo "Checking for new issues..."
python manage.py check --deploy
```

### Step 5: Generate Fix Implementation Report

**Create comprehensive report of bug fixes:**

```python
# File: bug_fix_reports/fix_report_generator.py

"""
Generate comprehensive report of bug fix implementation
"""

import json
from datetime import datetime
from pathlib import Path

def generate_fix_implementation_report():
    """Generate comprehensive fix implementation report"""
    
    # Load execution results
    with open('bug_fix_reports/fix_execution_results.json', 'r') as f:
        execution_results = json.load(f)
    
    # Load original fix plan
    with open('bug_fix_reports/bug_fix_plan.json', 'r') as f:
        fix_plan = json.load(f)
    
    report = {
        'metadata': {
            'app_name': '${input:app_name}',
            'project_name': '${input:project_name}',
            'report_generated': datetime.now().isoformat(),
            'execution_start': execution_results.get('execution_start'),
            'execution_end': execution_results.get('execution_end')
        },
        'summary': execution_results.get('summary', {}),
        'phase_results': execution_results.get('phases', {}),
        'recommendations': [],
        'next_steps': []
    }
    
    # Generate recommendations based on results
    summary = report['summary']
    
    if summary['failed_fixes'] > 0:
        report['recommendations'].append({
            'type': 'failed_fixes',
            'priority': 'high',
            'description': f"Address {summary['failed_fixes']} failed fixes",
            'action': 'Review error logs and fix manually'
        })
    
    if summary['manual_review_needed'] > 0:
        report['recommendations'].append({
            'type': 'manual_review',
            'priority': 'medium',
            'description': f"Manual review needed for {summary['manual_review_needed']} issues",
            'action': 'Schedule code review session'
        })
    
    if summary['skipped_fixes'] > 0:
        report['recommendations'].append({
            'type': 'skipped_fixes',
            'priority': 'low',
            'description': f"{summary['skipped_fixes']} fixes were skipped",
            'action': 'Evaluate if custom fix implementations are needed'
        })
    
    # Generate next steps
    if summary['successful_fixes'] > 0:
        report['next_steps'].append({
            'step': 'regression_testing',
            'description': 'Run comprehensive regression tests',
            'estimated_time': '2 hours'
        })
    
    report['next_steps'].extend([
        {
            'step': 'performance_testing',
            'description': 'Verify performance impact of fixes',
            'estimated_time': '1 hour'
        },
        {
            'step': 'deployment_preparation',
            'description': 'Prepare for deployment to staging',
            'estimated_time': '30 minutes'
        }
    ])
    
    # Save report
    with open('bug_fix_reports/implementation_report.json', 'w') as f:
        json.dump(report, f, indent=2)
    
    # Generate human-readable summary
    summary_text = f"""
=== BUG FIX IMPLEMENTATION REPORT ===

Project: {report['metadata']['project_name']}
App: {report['metadata']['app_name']}
Execution Time: {report['metadata']['execution_start']} to {report['metadata']['execution_end']}

SUMMARY:
- Total Issues: {summary['total_issues']}
- ✅ Successfully Fixed: {summary['successful_fixes']}
- ❌ Failed to Fix: {summary['failed_fixes']}
- ⏭️ Skipped: {summary['skipped_fixes']}
- ⚠️ Manual Review Needed: {summary['manual_review_needed']}

Success Rate: {(summary['successful_fixes'] / summary['total_issues'] * 100):.1f}%

RECOMMENDATIONS:
"""
    
    for rec in report['recommendations']:
        summary_text += f"- [{rec['priority'].upper()}] {rec['description']}: {rec['action']}\n"
    
    summary_text += "\nNEXT STEPS:\n"
    for step in report['next_steps']:
        summary_text += f"- {step['description']} (Est. {step['estimated_time']})\n"
    
    with open('bug_fix_reports/implementation_summary.txt', 'w') as f:
        f.write(summary_text)
    
    print(summary_text)
    
    return report

if __name__ == '__main__':
    generate_fix_implementation_report()
```

## Final Implementation Checklist

**Track implementation progress:**

### Phase 1: Critical Issues (Immediate)
- [ ] Security vulnerabilities fixed
- [ ] Authentication/authorization issues resolved  
- [ ] Critical functionality bugs fixed
- [ ] Database integrity issues addressed

### Phase 2: High Priority Issues (1-3 days)
- [ ] DD compliance issues resolved
- [ ] API contract violations fixed
- [ ] Permission logic corrected
- [ ] Data validation enhanced

### Phase 3: Medium Priority Issues (3-5 days) 
- [ ] Code quality improvements implemented
- [ ] Performance optimizations applied
- [ ] Django best practices adopted
- [ ] Documentation updated

### Phase 4: Low Priority Issues (5-7 days)
- [ ] Code style improvements
- [ ] Minor refactoring completed
- [ ] Additional documentation added
- [ ] Optional enhancements implemented

### Verification Steps
- [ ] All unit tests passing
- [ ] Code quality checks passing
- [ ] Security scan shows no critical issues
- [ ] Performance benchmarks met
- [ ] Manual testing completed
- [ ] Regression testing passed

## Success Criteria

**Implementation is successful when:**

✅ **All critical and high priority bugs are fixed**  
✅ **Unit tests pass with >95% success rate**  
✅ **Code quality score improves significantly**  
✅ **Security vulnerabilities are resolved**  
✅ **Performance requirements are met**  
✅ **DD compliance is achieved**  
✅ **No new bugs are introduced**  
✅ **Documentation is updated**

## Troubleshooting

**Common issues during bug fixing:**

### Fix Implementation Issues
- **Syntax errors**: Run Python syntax check before testing
- **Import errors**: Verify all required imports are present
- **Migration conflicts**: Resolve database migration dependencies

### Test Failures After Fixes
- **New test failures**: Check if fixes introduced new issues
- **Flaky tests**: Identify and fix non-deterministic test behavior
- **Performance regressions**: Monitor response times after fixes

### Integration Issues
- **API compatibility**: Ensure fixes don't break API contracts
- **Third-party dependencies**: Verify external service integration
- **Environment differences**: Test fixes across all environments

---

**Note**: This prompt provides a comprehensive framework for fixing bugs found in unit tests and code reviews. The automated fixes handle common patterns, while complex issues are flagged for manual review.