---
description: Create comprehensive pull request based on DD implementation plan and results from prompt 04
---

# Django Design Detail (DD) Pull Request Creator

This prompt creates a comprehensive pull request based on the implementation plan from prompt 04, including DD compliance verification, automated PR description generation, code review assignments, and deployment pipeline integration.

## Prerequisites

**Required Input:**
- 📋 **Implementation Plan**: Output from `04_django_dd_implementation_plan.prompt.md`
- 📦 **Project Name**: `${input:project_name}`
- 🏷️ **App Name**: `${input:app_name}`
- 📄 **Implementation Plan Path**: `${input:implementation_plan_path}`
- 🌿 **Target Branch**: `${input:target_branch:main}`
- 👤 **PR Author**: `${input:pr_author}`
- 📝 **DD Name**: `${input:dd_name}`

## Pull Request Creation Workflow

### Step 1: Load Implementation Plan and Analyze Changes

**Load and parse implementation plan from prompt 04:**

```bash
# Load implementation plan
echo "=== LOADING IMPLEMENTATION PLAN ==="
echo "Project: ${input:project_name}"
echo "DD: ${input:dd_name}"
echo "Target Branch: ${input:target_branch}"

# Validate implementation plan exists
if [ ! -f "${input:implementation_plan_path}/implementation_plan.json" ]; then
    echo "❌ Error: Implementation plan not found at ${input:implementation_plan_path}/implementation_plan.json"
    exit 1
fi

echo "✅ Implementation plan found"

# Load plan content
cat ${input:implementation_plan_path}/implementation_plan.json > pr_analysis/implementation_plan.json

# Get current git status and changes
echo "=== ANALYZING CURRENT CHANGES ==="
git status --porcelain > pr_analysis/git_status.txt
git diff --name-only ${input:target_branch}...HEAD > pr_analysis/changed_files.txt
git log --oneline ${input:target_branch}...HEAD > pr_analysis/commit_history.txt

echo "Changed files since ${input:target_branch}:"
cat pr_analysis/changed_files.txt
```

**Analyze implementation completeness:**

```python
# File: pr_analysis/implementation_analyzer.py

"""
Analyze implementation completeness against DD plan
"""

import json
import subprocess
import re
from datetime import datetime
from pathlib import Path
from typing import Dict, List, Any

class ImplementationAnalyzer:
    """Analyze implementation against DD plan"""
    
    def __init__(self, plan_path: str, app_name: str, dd_name: str):
        self.plan_path = plan_path
        self.app_name = app_name
        self.dd_name = dd_name
        
        # Load implementation plan
        with open(plan_path, 'r') as f:
            self.implementation_plan = json.load(f)
        
        self.analysis_results = {
            'dd_compliance': {},
            'implementation_coverage': {},
            'code_quality_metrics': {},
            'test_coverage': {},
            'documentation_status': {}
        }
    
    def analyze_dd_compliance(self) -> Dict:
        """Check implementation against DD requirements"""
        dd_requirements = self.implementation_plan.get('dd_requirements', {})
        compliance_results = {
            'api_endpoints': {'implemented': [], 'missing': [], 'compliance_rate': 0},
            'data_models': {'implemented': [], 'missing': [], 'compliance_rate': 0},
            'business_rules': {'implemented': [], 'missing': [], 'compliance_rate': 0},
            'authentication': {'implemented': [], 'missing': [], 'compliance_rate': 0},
            'overall_compliance': 0
        }
        
        # Check API endpoints implementation
        required_endpoints = dd_requirements.get('api_endpoints', [])
        for endpoint in required_endpoints:
            endpoint_path = endpoint.get('path', '')
            method = endpoint.get('method', 'GET')
            
            # Check if endpoint is implemented in URLs
            urls_file = f'apps/{self.app_name}/urls.py'
            if Path(urls_file).exists():
                with open(urls_file, 'r') as f:
                    urls_content = f.read()
                
                # Simple check for endpoint pattern
                endpoint_pattern = endpoint_path.replace('/', r'\/').replace('{', r'\w+').replace('}', '')
                if re.search(endpoint_pattern, urls_content):
                    compliance_results['api_endpoints']['implemented'].append(endpoint)
                else:
                    compliance_results['api_endpoints']['missing'].append(endpoint)
            else:
                compliance_results['api_endpoints']['missing'].append(endpoint)
        
        # Calculate API compliance rate
        total_endpoints = len(required_endpoints)
        implemented_endpoints = len(compliance_results['api_endpoints']['implemented'])
        compliance_results['api_endpoints']['compliance_rate'] = (
            (implemented_endpoints / total_endpoints * 100) if total_endpoints > 0 else 100
        )
        
        # Check data models implementation
        required_models = dd_requirements.get('data_models', [])
        models_file = f'apps/{self.app_name}/models.py'
        
        if Path(models_file).exists():
            with open(models_file, 'r') as f:
                models_content = f.read()
            
            for model in required_models:
                model_name = model.get('name', '')
                if f'class {model_name}' in models_content:
                    compliance_results['data_models']['implemented'].append(model)
                else:
                    compliance_results['data_models']['missing'].append(model)
        else:
            compliance_results['data_models']['missing'] = required_models
        
        # Calculate models compliance rate
        total_models = len(required_models)
        implemented_models = len(compliance_results['data_models']['implemented'])
        compliance_results['data_models']['compliance_rate'] = (
            (implemented_models / total_models * 100) if total_models > 0 else 100
        )
        
        # Check business rules implementation
        business_rules = dd_requirements.get('business_rules', [])
        for rule in business_rules:
            # This is a simplified check - in practice, would need more sophisticated analysis
            rule_description = rule.get('description', '').lower()
            
            # Check if rule might be implemented (simple keyword search)
            files_to_check = [
                f'apps/{self.app_name}/models.py',
                f'apps/{self.app_name}/serializers.py',
                f'apps/{self.app_name}/views.py'
            ]
            
            rule_found = False
            for file_path in files_to_check:
                if Path(file_path).exists():
                    with open(file_path, 'r') as f:
                        file_content = f.read().lower()
                    
                    # Simple keyword matching
                    key_words = rule_description.split()[:3]  # First 3 words
                    if any(word in file_content for word in key_words if len(word) > 3):
                        rule_found = True
                        break
            
            if rule_found:
                compliance_results['business_rules']['implemented'].append(rule)
            else:
                compliance_results['business_rules']['missing'].append(rule)
        
        # Calculate business rules compliance rate
        total_rules = len(business_rules)
        implemented_rules = len(compliance_results['business_rules']['implemented'])
        compliance_results['business_rules']['compliance_rate'] = (
            (implemented_rules / total_rules * 100) if total_rules > 0 else 100
        )
        
        # Check authentication implementation
        auth_requirements = dd_requirements.get('authentication', {})
        auth_method = auth_requirements.get('method', '')
        
        views_file = f'apps/{self.app_name}/views.py'
        settings_file = 'config/settings/base.py'
        
        auth_implemented = False
        if Path(views_file).exists():
            with open(views_file, 'r') as f:
                views_content = f.read()
            
            if 'authentication_classes' in views_content or 'permission_classes' in views_content:
                auth_implemented = True
                compliance_results['authentication']['implemented'].append({
                    'type': 'view_level_auth',
                    'method': auth_method
                })
        
        if Path(settings_file).exists():
            with open(settings_file, 'r') as f:
                settings_content = f.read()
            
            if 'DEFAULT_AUTHENTICATION_CLASSES' in settings_content:
                auth_implemented = True
                compliance_results['authentication']['implemented'].append({
                    'type': 'global_auth',
                    'method': auth_method
                })
        
        if not auth_implemented:
            compliance_results['authentication']['missing'].append({
                'type': 'authentication_setup',
                'method': auth_method
            })
        
        compliance_results['authentication']['compliance_rate'] = 100 if auth_implemented else 0
        
        # Calculate overall compliance
        compliance_rates = [
            compliance_results['api_endpoints']['compliance_rate'],
            compliance_results['data_models']['compliance_rate'],
            compliance_results['business_rules']['compliance_rate'],
            compliance_results['authentication']['compliance_rate']
        ]
        compliance_results['overall_compliance'] = sum(compliance_rates) / len(compliance_rates)
        
        return compliance_results
    
    def analyze_implementation_coverage(self) -> Dict:
        """Analyze what has been implemented vs planned"""
        planned_components = self.implementation_plan.get('implementation_components', {})
        coverage_results = {
            'models': {'planned': 0, 'implemented': 0, 'files': []},
            'serializers': {'planned': 0, 'implemented': 0, 'files': []},
            'views': {'planned': 0, 'implemented': 0, 'files': []},
            'urls': {'planned': 0, 'implemented': 0, 'files': []},
            'migrations': {'planned': 0, 'implemented': 0, 'files': []},
            'tests': {'planned': 0, 'implemented': 0, 'files': []},
            'overall_coverage': 0
        }
        
        # Check each component type
        for component_type in coverage_results.keys():
            if component_type == 'overall_coverage':
                continue
            
            planned_items = planned_components.get(component_type, [])
            coverage_results[component_type]['planned'] = len(planned_items)
            
            # Check if files exist
            file_patterns = {
                'models': [f'apps/{self.app_name}/models.py'],
                'serializers': [f'apps/{self.app_name}/serializers.py'],
                'views': [f'apps/{self.app_name}/views.py'],
                'urls': [f'apps/{self.app_name}/urls.py'],
                'migrations': [f'apps/{self.app_name}/migrations/*.py'],
                'tests': [f'tests/test_{self.app_name}/*.py']
            }
            
            implemented_files = []
            for pattern in file_patterns.get(component_type, []):
                if '*' in pattern:
                    # Handle glob patterns
                    from glob import glob
                    matching_files = glob(pattern)
                    implemented_files.extend(matching_files)
                else:
                    if Path(pattern).exists():
                        implemented_files.append(pattern)
            
            coverage_results[component_type]['files'] = implemented_files
            coverage_results[component_type]['implemented'] = len(implemented_files)
        
        # Calculate overall coverage
        total_planned = sum(result['planned'] for result in coverage_results.values() if isinstance(result, dict))
        total_implemented = sum(result['implemented'] for result in coverage_results.values() if isinstance(result, dict))
        
        coverage_results['overall_coverage'] = (
            (total_implemented / total_planned * 100) if total_planned > 0 else 100
        )
        
        return coverage_results
    
    def analyze_code_quality_metrics(self) -> Dict:
        """Analyze code quality metrics"""
        quality_metrics = {
            'syntax_errors': 0,
            'style_violations': 0,
            'complexity_score': 0,
            'documentation_coverage': 0,
            'type_hints_coverage': 0,
            'overall_quality_score': 0
        }
        
        app_files = list(Path(f'apps/{self.app_name}').glob('*.py'))
        
        for file_path in app_files:
            # Check syntax
            try:
                with open(file_path, 'r') as f:
                    content = f.read()
                
                compile(content, str(file_path), 'exec')
            except SyntaxError:
                quality_metrics['syntax_errors'] += 1
            
            # Simple documentation check
            if content:
                lines = content.split('\n')
                docstring_lines = sum(1 for line in lines if '"""' in line or "'''" in line)
                function_lines = sum(1 for line in lines if line.strip().startswith('def '))
                class_lines = sum(1 for line in lines if line.strip().startswith('class '))
                
                total_definitions = function_lines + class_lines
                if total_definitions > 0:
                    quality_metrics['documentation_coverage'] += (docstring_lines / total_definitions * 100) / len(app_files)
                
                # Simple type hints check
                type_hint_lines = sum(1 for line in lines if '->' in line or ': ' in line)
                if total_definitions > 0:
                    quality_metrics['type_hints_coverage'] += (type_hint_lines / total_definitions * 100) / len(app_files)
        
        # Calculate overall quality score
        quality_factors = [
            100 - (quality_metrics['syntax_errors'] * 10),  # Penalize syntax errors heavily
            100 - (quality_metrics['style_violations'] * 2),  # Penalize style violations lightly
            quality_metrics['documentation_coverage'],
            quality_metrics['type_hints_coverage']
        ]
        
        quality_metrics['overall_quality_score'] = max(0, sum(quality_factors) / len(quality_factors))
        
        return quality_metrics
    
    def analyze_test_coverage(self) -> Dict:
        """Analyze test coverage"""
        test_coverage = {
            'unit_tests_exist': False,
            'integration_tests_exist': False,
            'test_files_count': 0,
            'estimated_coverage': 0
        }
        
        # Check for test files
        test_patterns = [
            f'tests/test_{self.app_name}/*.py',
            f'apps/{self.app_name}/tests.py',
            f'apps/{self.app_name}/test_*.py'
        ]
        
        test_files = []
        for pattern in test_patterns:
            if '*' in pattern:
                from glob import glob
                test_files.extend(glob(pattern))
            else:
                if Path(pattern).exists():
                    test_files.append(pattern)
        
        test_coverage['test_files_count'] = len(test_files)
        test_coverage['unit_tests_exist'] = len(test_files) > 0
        
        # Simple estimation of coverage based on test files
        if test_files:
            total_test_functions = 0
            for test_file in test_files:
                with open(test_file, 'r') as f:
                    content = f.read()
                    total_test_functions += len(re.findall(r'def test_\w+', content))
            
            # Rough estimation: assume each test function covers ~10% of functionality
            test_coverage['estimated_coverage'] = min(100, total_test_functions * 10)
        
        return test_coverage
    
    def generate_comprehensive_analysis(self) -> Dict:
        """Generate comprehensive implementation analysis"""
        print("Analyzing DD compliance...")
        self.analysis_results['dd_compliance'] = self.analyze_dd_compliance()
        
        print("Analyzing implementation coverage...")
        self.analysis_results['implementation_coverage'] = self.analyze_implementation_coverage()
        
        print("Analyzing code quality...")
        self.analysis_results['code_quality_metrics'] = self.analyze_code_quality_metrics()
        
        print("Analyzing test coverage...")
        self.analysis_results['test_coverage'] = self.analyze_test_coverage()
        
        # Generate overall summary
        self.analysis_results['summary'] = {
            'dd_compliance_rate': self.analysis_results['dd_compliance']['overall_compliance'],
            'implementation_coverage_rate': self.analysis_results['implementation_coverage']['overall_coverage'],
            'code_quality_score': self.analysis_results['code_quality_metrics']['overall_quality_score'],
            'test_coverage_estimate': self.analysis_results['test_coverage']['estimated_coverage'],
            'overall_readiness_score': (
                self.analysis_results['dd_compliance']['overall_compliance'] * 0.4 +
                self.analysis_results['implementation_coverage']['overall_coverage'] * 0.3 +
                self.analysis_results['code_quality_metrics']['overall_quality_score'] * 0.2 +
                self.analysis_results['test_coverage']['estimated_coverage'] * 0.1
            )
        }
        
        return self.analysis_results


# Main analysis function
def analyze_implementation():
    """Main function to analyze implementation completeness"""
    analyzer = ImplementationAnalyzer(
        plan_path='pr_analysis/implementation_plan.json',
        app_name='${input:app_name}',
        dd_name='${input:dd_name}'
    )
    
    results = analyzer.generate_comprehensive_analysis()
    
    # Save analysis results
    with open('pr_analysis/implementation_analysis.json', 'w') as f:
        json.dump(results, f, indent=2)
    
    print(f"\n=== IMPLEMENTATION ANALYSIS COMPLETE ===")
    print(f"DD Compliance: {results['summary']['dd_compliance_rate']:.1f}%")
    print(f"Implementation Coverage: {results['summary']['implementation_coverage_rate']:.1f}%")
    print(f"Code Quality Score: {results['summary']['code_quality_score']:.1f}/100")
    print(f"Test Coverage: {results['summary']['test_coverage_estimate']:.1f}%")
    print(f"Overall Readiness: {results['summary']['overall_readiness_score']:.1f}%")
    
    return results


if __name__ == '__main__':
    analyze_implementation()
```

### Step 2: Generate PR Description and Metadata

**Create comprehensive PR description based on DD and implementation:**

```python
# File: pr_creation/pr_description_generator.py

"""
Generate comprehensive PR description based on DD implementation
"""

import json
from datetime import datetime
from typing import Dict, List

class PRDescriptionGenerator:
    """Generate comprehensive PR description"""
    
    def __init__(self, implementation_plan: Dict, analysis_results: Dict, dd_name: str):
        self.implementation_plan = implementation_plan
        self.analysis_results = analysis_results
        self.dd_name = dd_name
    
    def generate_pr_title(self) -> str:
        """Generate PR title based on DD"""
        dd_summary = self.implementation_plan.get('dd_summary', {})
        feature_type = dd_summary.get('type', 'feature')
        description = dd_summary.get('description', self.dd_name)
        
        # Create concise title
        if feature_type.lower() == 'feature':
            title = f"feat: {description}"
        elif feature_type.lower() == 'enhancement':
            title = f"enhance: {description}"
        elif feature_type.lower() == 'bugfix':
            title = f"fix: {description}"
        else:
            title = f"update: {description}"
        
        # Add DD reference
        title += f" (DD: {self.dd_name})"
        
        return title
    
    def generate_summary_section(self) -> str:
        """Generate PR summary section"""
        dd_summary = self.implementation_plan.get('dd_summary', {})
        
        summary = f"""## Summary

**Design Detail**: {self.dd_name}
**Type**: {dd_summary.get('type', 'Feature')}
**Priority**: {dd_summary.get('priority', 'Medium')}

### What this PR does
{dd_summary.get('description', 'Implements functionality as defined in the Design Detail.')}

### Business Value
{dd_summary.get('business_value', 'Delivers value to users by implementing requested functionality.')}
"""
        
        return summary
    
    def generate_implementation_details(self) -> str:
        """Generate implementation details section"""
        components = self.implementation_plan.get('implementation_components', {})
        
        details = """## Implementation Details

### Components Implemented
"""
        
        for component_type, items in components.items():
            if items:
                details += f"\n#### {component_type.title()}\n"
                for item in items:
                    if isinstance(item, dict):
                        item_name = item.get('name', str(item))
                        item_desc = item.get('description', '')
                    else:
                        item_name = str(item)
                        item_desc = ''
                    
                    details += f"- **{item_name}**"
                    if item_desc:
                        details += f": {item_desc}"
                    details += "\n"
        
        return details
    
    def generate_api_changes(self) -> str:
        """Generate API changes section"""
        dd_requirements = self.implementation_plan.get('dd_requirements', {})
        api_endpoints = dd_requirements.get('api_endpoints', [])
        
        if not api_endpoints:
            return ""
        
        api_section = """## API Changes

### New/Modified Endpoints
"""
        
        for endpoint in api_endpoints:
            method = endpoint.get('method', 'GET')
            path = endpoint.get('path', '')
            description = endpoint.get('description', '')
            
            api_section += f"""
#### `{method} {path}`
{description}

**Request**: {endpoint.get('request_format', 'JSON')}  
**Response**: {endpoint.get('response_format', 'JSON')}  
**Authentication**: {endpoint.get('authentication', 'Required')}
"""
        
        return api_section
    
    def generate_database_changes(self) -> str:
        """Generate database changes section"""
        migrations = self.implementation_plan.get('migrations', [])
        
        if not migrations:
            return ""
        
        db_section = """## Database Changes

### Migrations
"""
        
        for migration in migrations:
            migration_type = migration.get('type', 'schema')
            description = migration.get('description', '')
            
            db_section += f"- **{migration_type.title()}**: {description}\n"
            
            if migration.get('backward_compatible', True):
                db_section += "  - ✅ Backward compatible\n"
            else:
                db_section += "  - ⚠️ Breaking change - requires coordination\n"
        
        return db_section
    
    def generate_testing_information(self) -> str:
        """Generate testing information section"""
        test_coverage = self.analysis_results.get('test_coverage', {})
        
        testing_section = f"""## Testing

### Test Coverage
- **Unit Tests**: {'✅ Present' if test_coverage.get('unit_tests_exist') else '❌ Missing'}
- **Integration Tests**: {'✅ Present' if test_coverage.get('integration_tests_exist') else '❌ Missing'}
- **Test Files**: {test_coverage.get('test_files_count', 0)} files
- **Estimated Coverage**: {test_coverage.get('estimated_coverage', 0):.1f}%

### Manual Testing
- [ ] API endpoints respond correctly
- [ ] Authentication/authorization works as expected
- [ ] Business rules are enforced
- [ ] Error handling works properly
- [ ] Performance is acceptable

### Test Commands
```bash
# Run unit tests
python manage.py test apps.${input:app_name}

# Run integration tests
python manage.py test tests.integration

# Run all tests
python manage.py test
```
"""
        
        return testing_section
    
    def generate_deployment_notes(self) -> str:
        """Generate deployment notes section"""
        deployment_section = """## Deployment Notes

### Prerequisites
- [ ] Database migrations reviewed and approved
- [ ] Environment variables configured (if any)
- [ ] Dependencies installed/updated

### Deployment Steps
1. Run database migrations: `python manage.py migrate`
2. Collect static files: `python manage.py collectstatic --noinput`
3. Restart application server
4. Verify health check endpoints

### Rollback Plan
1. Revert to previous application version
2. Run migration rollback (if needed): `python manage.py migrate <app> <previous_migration>`
3. Restart application server

### Monitoring
- Monitor application logs for errors
- Check API response times
- Verify business metrics
"""
        
        return deployment_section
    
    def generate_compliance_checklist(self) -> str:
        """Generate DD compliance checklist"""
        dd_compliance = self.analysis_results.get('dd_compliance', {})
        
        checklist = """## DD Compliance Checklist

### API Requirements
"""
        
        api_compliance = dd_compliance.get('api_endpoints', {})
        checklist += f"- Compliance Rate: {api_compliance.get('compliance_rate', 0):.1f}%\n"
        
        implemented = api_compliance.get('implemented', [])
        missing = api_compliance.get('missing', [])
        
        for endpoint in implemented:
            path = endpoint.get('path', 'Unknown')
            method = endpoint.get('method', 'GET')
            checklist += f"- [x] `{method} {path}` implemented\n"
        
        for endpoint in missing:
            path = endpoint.get('path', 'Unknown')
            method = endpoint.get('method', 'GET')
            checklist += f"- [ ] `{method} {path}` **missing**\n"
        
        # Data Models
        models_compliance = dd_compliance.get('data_models', {})
        checklist += f"\n### Data Models\n- Compliance Rate: {models_compliance.get('compliance_rate', 0):.1f}%\n"
        
        for model in models_compliance.get('implemented', []):
            model_name = model.get('name', 'Unknown')
            checklist += f"- [x] {model_name} model implemented\n"
        
        for model in models_compliance.get('missing', []):
            model_name = model.get('name', 'Unknown')
            checklist += f"- [ ] {model_name} model **missing**\n"
        
        # Business Rules
        rules_compliance = dd_compliance.get('business_rules', {})
        checklist += f"\n### Business Rules\n- Compliance Rate: {rules_compliance.get('compliance_rate', 0):.1f}%\n"
        
        for rule in rules_compliance.get('implemented', []):
            rule_desc = rule.get('description', 'Unknown rule')
            checklist += f"- [x] {rule_desc}\n"
        
        for rule in rules_compliance.get('missing', []):
            rule_desc = rule.get('description', 'Unknown rule')
            checklist += f"- [ ] {rule_desc} **missing**\n"
        
        # Overall compliance
        overall_compliance = dd_compliance.get('overall_compliance', 0)
        if overall_compliance >= 90:
            checklist += f"\n### ✅ Overall Compliance: {overall_compliance:.1f}% - Ready for review"
        elif overall_compliance >= 70:
            checklist += f"\n### ⚠️ Overall Compliance: {overall_compliance:.1f}% - Minor issues need addressing"
        else:
            checklist += f"\n### ❌ Overall Compliance: {overall_compliance:.1f}% - Significant issues need addressing"
        
        return checklist
    
    def generate_code_quality_section(self) -> str:
        """Generate code quality section"""
        quality_metrics = self.analysis_results.get('code_quality_metrics', {})
        
        quality_section = f"""## Code Quality

### Metrics
- **Overall Quality Score**: {quality_metrics.get('overall_quality_score', 0):.1f}/100
- **Syntax Errors**: {quality_metrics.get('syntax_errors', 0)}
- **Style Violations**: {quality_metrics.get('style_violations', 0)}
- **Documentation Coverage**: {quality_metrics.get('documentation_coverage', 0):.1f}%
- **Type Hints Coverage**: {quality_metrics.get('type_hints_coverage', 0):.1f}%

### Quality Checks
- [ ] Code follows PEP 8 style guidelines
- [ ] All functions have docstrings
- [ ] Type hints are used where appropriate
- [ ] No TODO/FIXME comments remain
- [ ] Error handling is comprehensive
"""
        
        return quality_section
    
    def generate_review_assignments(self) -> str:
        """Generate review assignments section"""
        review_section = """## Review Assignment

### Required Reviewers
- **Technical Lead**: @tech-lead (for architecture review)
- **Domain Expert**: @domain-expert (for business logic review)
- **Security Review**: @security-team (for security implications)

### Review Checklist
- [ ] Code follows project standards
- [ ] Business requirements are met
- [ ] Security implications reviewed
- [ ] Performance impact assessed
- [ ] Documentation is complete
- [ ] Tests provide adequate coverage

### Approval Requirements
- [ ] At least 2 technical approvals
- [ ] Security approval (if applicable)
- [ ] QA sign-off for testing
"""
        
        return review_section
    
    def generate_complete_pr_description(self) -> str:
        """Generate complete PR description"""
        description = self.generate_summary_section()
        description += "\n" + self.generate_implementation_details()
        
        api_changes = self.generate_api_changes()
        if api_changes:
            description += "\n" + api_changes
        
        db_changes = self.generate_database_changes()
        if db_changes:
            description += "\n" + db_changes
        
        description += "\n" + self.generate_testing_information()
        description += "\n" + self.generate_deployment_notes()
        description += "\n" + self.generate_compliance_checklist()
        description += "\n" + self.generate_code_quality_section()
        description += "\n" + self.generate_review_assignments()
        
        # Add footer
        description += f"""

---

**Generated by**: Django DD Pull Request Creator  
**Timestamp**: {datetime.now().isoformat()}  
**Implementation Plan**: {self.implementation_plan.get('metadata', {}).get('created_at', 'Unknown')}
"""
        
        return description


# Generate PR description
def generate_pr_description():
    """Main function to generate PR description"""
    
    # Load implementation plan and analysis results
    with open('pr_analysis/implementation_plan.json', 'r') as f:
        implementation_plan = json.load(f)
    
    with open('pr_analysis/implementation_analysis.json', 'r') as f:
        analysis_results = json.load(f)
    
    generator = PRDescriptionGenerator(
        implementation_plan=implementation_plan,
        analysis_results=analysis_results,
        dd_name='${input:dd_name}'
    )
    
    # Generate PR components
    pr_title = generator.generate_pr_title()
    pr_description = generator.generate_complete_pr_description()
    
    # Save PR content
    pr_content = {
        'title': pr_title,
        'description': pr_description,
        'metadata': {
            'generated_at': datetime.now().isoformat(),
            'dd_name': '${input:dd_name}',
            'app_name': '${input:app_name}',
            'target_branch': '${input:target_branch}'
        }
    }
    
    with open('pr_creation/pr_content.json', 'w') as f:
        json.dump(pr_content, f, indent=2)
    
    # Also save as markdown for review
    with open('pr_creation/pr_description.md', 'w') as f:
        f.write(f"# {pr_title}\n\n{pr_description}")
    
    print(f"✅ PR content generated")
    print(f"Title: {pr_title}")
    print(f"Description length: {len(pr_description)} characters")
    
    return pr_content


if __name__ == '__main__':
    generate_pr_description()
```

### Step 3: Create Pull Request with GitHub CLI

**Create PR using GitHub CLI with all metadata:**

```bash
# Create pull request using GitHub CLI
echo "=== CREATING PULL REQUEST ==="

# Load PR content
pr_title=$(cat pr_creation/pr_content.json | jq -r '.title')
pr_body_file="pr_creation/pr_description.md"

# Ensure we're on the correct branch
current_branch=$(git branch --show-current)
echo "Current branch: $current_branch"

# Push current branch to remote
echo "Pushing branch to remote..."
git push origin $current_branch

# Create the pull request
echo "Creating pull request..."
gh pr create \
    --title "$pr_title" \
    --body-file "$pr_body_file" \
    --base "${input:target_branch}" \
    --head "$current_branch" \
    --label "enhancement,dd-implementation" \
    --label "${input:dd_name}" \
    --assignee "${input:pr_author}" \
    --reviewer "@tech-lead,@domain-expert"

# Get PR number and URL
pr_url=$(gh pr view --json url --jq '.url')
pr_number=$(gh pr view --json number --jq '.number')

echo "✅ Pull request created successfully!"
echo "PR Number: #$pr_number"
echo "PR URL: $pr_url"

# Save PR information
cat > pr_creation/pr_info.json << EOF
{
    "pr_number": $pr_number,
    "pr_url": "$pr_url",
    "branch": "$current_branch",
    "target_branch": "${input:target_branch}",
    "title": "$pr_title",
    "author": "${input:pr_author}",
    "created_at": "$(date -Iseconds)",
    "dd_name": "${input:dd_name}"
}
EOF
```

### Step 4: Configure PR Labels and Milestones

**Add appropriate labels and milestone based on DD:**

```python
# File: pr_creation/pr_metadata_manager.py

"""
Manage PR labels, milestones, and metadata
"""

import json
import subprocess
from typing import Dict, List

class PRMetadataManager:
    """Manage PR metadata like labels, milestones, etc."""
    
    def __init__(self, pr_number: str, implementation_plan: Dict, analysis_results: Dict):
        self.pr_number = pr_number
        self.implementation_plan = implementation_plan
        self.analysis_results = analysis_results
    
    def determine_labels(self) -> List[str]:
        """Determine appropriate labels for the PR"""
        labels = []
        
        # Base labels
        dd_summary = self.implementation_plan.get('dd_summary', {})
        dd_type = dd_summary.get('type', 'feature').lower()
        
        if dd_type == 'feature':
            labels.append('enhancement')
        elif dd_type == 'bugfix':
            labels.append('bug')
        elif dd_type == 'improvement':
            labels.append('improvement')
        else:
            labels.append('feature')
        
        # DD-specific label
        labels.append(f"dd:{self.implementation_plan.get('dd_name', 'unknown')}")
        
        # Priority-based labels
        priority = dd_summary.get('priority', 'medium').lower()
        labels.append(f"priority:{priority}")
        
        # Component-based labels
        components = self.implementation_plan.get('implementation_components', {})
        if components.get('migrations'):
            labels.append('database')
        if components.get('models'):
            labels.append('models')
        if components.get('api'):
            labels.append('api')
        
        # Quality-based labels
        quality_score = self.analysis_results.get('code_quality_metrics', {}).get('overall_quality_score', 0)
        if quality_score >= 90:
            labels.append('high-quality')
        elif quality_score < 70:
            labels.append('needs-improvement')
        
        # Test coverage labels
        test_coverage = self.analysis_results.get('test_coverage', {}).get('estimated_coverage', 0)
        if test_coverage >= 80:
            labels.append('well-tested')
        elif test_coverage < 50:
            labels.append('needs-tests')
        
        # Security labels
        auth_required = self.implementation_plan.get('dd_requirements', {}).get('authentication', {})
        if auth_required:
            labels.append('security-review-required')
        
        return labels
    
    def determine_milestone(self) -> str:
        """Determine appropriate milestone"""
        dd_summary = self.implementation_plan.get('dd_summary', {})
        priority = dd_summary.get('priority', 'medium').lower()
        
        # Map priority to milestone
        milestone_mapping = {
            'critical': 'Next Release',
            'high': 'Next Release',
            'medium': 'Upcoming',
            'low': 'Future'
        }
        
        return milestone_mapping.get(priority, 'Upcoming')
    
    def determine_reviewers(self) -> List[str]:
        """Determine appropriate reviewers"""
        reviewers = []
        
        # Always include tech lead
        reviewers.append('tech-lead')
        
        # Add domain expert for business logic
        business_rules = self.implementation_plan.get('dd_requirements', {}).get('business_rules', [])
        if business_rules:
            reviewers.append('domain-expert')
        
        # Add security reviewer if needed
        auth_required = self.implementation_plan.get('dd_requirements', {}).get('authentication', {})
        if auth_required:
            reviewers.append('security-team')
        
        # Add database reviewer for migrations
        migrations = self.implementation_plan.get('migrations', [])
        if migrations:
            reviewers.append('database-team')
        
        # Add API reviewer for API changes
        api_endpoints = self.implementation_plan.get('dd_requirements', {}).get('api_endpoints', [])
        if api_endpoints:
            reviewers.append('api-team')
        
        return reviewers
    
    def apply_labels(self, labels: List[str]) -> bool:
        """Apply labels to PR"""
        try:
            labels_str = ','.join(labels)
            result = subprocess.run(
                f"gh pr edit {self.pr_number} --add-label '{labels_str}'",
                shell=True,
                capture_output=True,
                text=True
            )
            return result.returncode == 0
        except Exception:
            return False
    
    def set_milestone(self, milestone: str) -> bool:
        """Set milestone for PR"""
        try:
            result = subprocess.run(
                f"gh pr edit {self.pr_number} --milestone '{milestone}'",
                shell=True,
                capture_output=True,
                text=True
            )
            return result.returncode == 0
        except Exception:
            return False
    
    def add_reviewers(self, reviewers: List[str]) -> bool:
        """Add reviewers to PR"""
        try:
            reviewers_str = ','.join(f"@{reviewer}" for reviewer in reviewers)
            result = subprocess.run(
                f"gh pr edit {self.pr_number} --add-reviewer '{reviewers_str}'",
                shell=True,
                capture_output=True,
                text=True
            )
            return result.returncode == 0
        except Exception:
            return False
    
    def configure_pr_metadata(self) -> Dict:
        """Configure all PR metadata"""
        labels = self.determine_labels()
        milestone = self.determine_milestone()
        reviewers = self.determine_reviewers()
        
        results = {
            'labels': {
                'determined': labels,
                'applied': self.apply_labels(labels)
            },
            'milestone': {
                'determined': milestone,
                'applied': self.set_milestone(milestone)
            },
            'reviewers': {
                'determined': reviewers,
                'applied': self.add_reviewers(reviewers)
            }
        }
        
        return results


# Configure PR metadata
def configure_pr_metadata():
    """Main function to configure PR metadata"""
    
    # Load PR info
    with open('pr_creation/pr_info.json', 'r') as f:
        pr_info = json.load(f)
    
    pr_number = pr_info['pr_number']
    
    # Load implementation plan and analysis
    with open('pr_analysis/implementation_plan.json', 'r') as f:
        implementation_plan = json.load(f)
    
    with open('pr_analysis/implementation_analysis.json', 'r') as f:
        analysis_results = json.load(f)
    
    # Configure metadata
    manager = PRMetadataManager(
        pr_number=str(pr_number),
        implementation_plan=implementation_plan,
        analysis_results=analysis_results
    )
    
    results = manager.configure_pr_metadata()
    
    # Save metadata configuration results
    with open('pr_creation/metadata_results.json', 'w') as f:
        json.dump(results, f, indent=2)
    
    print(f"✅ PR metadata configured for #{pr_number}")
    print(f"Labels: {results['labels']['determined']}")
    print(f"Milestone: {results['milestone']['determined']}")
    print(f"Reviewers: {results['reviewers']['determined']}")
    
    return results


if __name__ == '__main__':
    configure_pr_metadata()
```

### Step 5: Generate Final PR Report

**Create final report of PR creation process:**

```python
# File: pr_creation/pr_creation_report.py

"""
Generate final PR creation report
"""

import json
from datetime import datetime

def generate_pr_creation_report():
    """Generate comprehensive PR creation report"""
    
    # Load all data
    with open('pr_creation/pr_info.json', 'r') as f:
        pr_info = json.load(f)
    
    with open('pr_analysis/implementation_analysis.json', 'r') as f:
        analysis_results = json.load(f)
    
    with open('pr_creation/metadata_results.json', 'r') as f:
        metadata_results = json.load(f)
    
    # Generate comprehensive report
    report = {
        'pr_creation_summary': {
            'pr_number': pr_info['pr_number'],
            'pr_url': pr_info['pr_url'],
            'title': pr_info['title'],
            'author': pr_info['author'],
            'created_at': pr_info['created_at'],
            'dd_name': pr_info['dd_name'],
            'target_branch': pr_info['target_branch']
        },
        'implementation_quality': {
            'dd_compliance_rate': analysis_results['summary']['dd_compliance_rate'],
            'implementation_coverage': analysis_results['summary']['implementation_coverage_rate'],
            'code_quality_score': analysis_results['summary']['code_quality_score'],
            'test_coverage': analysis_results['summary']['test_coverage_estimate'],
            'overall_readiness': analysis_results['summary']['overall_readiness_score']
        },
        'pr_metadata': {
            'labels_applied': metadata_results['labels']['determined'],
            'milestone_set': metadata_results['milestone']['determined'],
            'reviewers_assigned': metadata_results['reviewers']['determined']
        },
        'next_steps': [
            'Wait for reviewer assignments',
            'Address any review feedback',
            'Ensure all CI/CD checks pass',
            'Coordinate with QA for testing',
            'Plan deployment after approval'
        ],
        'recommendations': []
    }
    
    # Add recommendations based on quality metrics
    readiness_score = report['implementation_quality']['overall_readiness']
    
    if readiness_score >= 90:
        report['recommendations'].append({
            'type': 'approval',
            'message': 'Implementation looks excellent. Ready for fast-track review.',
            'priority': 'low'
        })
    elif readiness_score >= 80:
        report['recommendations'].append({
            'type': 'review',
            'message': 'Implementation looks good. Standard review process recommended.',
            'priority': 'medium'
        })
    else:
        report['recommendations'].append({
            'type': 'improvement',
            'message': 'Implementation needs improvement before review. Address quality issues.',
            'priority': 'high'
        })
    
    # Check specific quality aspects
    if report['implementation_quality']['test_coverage'] < 60:
        report['recommendations'].append({
            'type': 'testing',
            'message': 'Test coverage is low. Add more unit and integration tests.',
            'priority': 'high'
        })
    
    if report['implementation_quality']['dd_compliance_rate'] < 80:
        report['recommendations'].append({
            'type': 'compliance',
            'message': 'DD compliance is below threshold. Ensure all requirements are met.',
            'priority': 'high'
        })
    
    # Save report
    with open('pr_creation/pr_creation_report.json', 'w') as f:
        json.dump(report, f, indent=2)
    
    # Generate text summary
    summary = f"""
=== PULL REQUEST CREATION REPORT ===

PR Details:
- Number: #{report['pr_creation_summary']['pr_number']}
- URL: {report['pr_creation_summary']['pr_url']}
- Title: {report['pr_creation_summary']['title']}
- DD: {report['pr_creation_summary']['dd_name']}

Quality Metrics:
- DD Compliance: {report['implementation_quality']['dd_compliance_rate']:.1f}%
- Implementation Coverage: {report['implementation_quality']['implementation_coverage']:.1f}%
- Code Quality: {report['implementation_quality']['code_quality_score']:.1f}/100
- Test Coverage: {report['implementation_quality']['test_coverage']:.1f}%
- Overall Readiness: {report['implementation_quality']['overall_readiness']:.1f}%

PR Metadata:
- Labels: {', '.join(report['pr_metadata']['labels_applied'])}
- Milestone: {report['pr_metadata']['milestone_set']}
- Reviewers: {', '.join(report['pr_metadata']['reviewers_assigned'])}

Recommendations:
"""
    
    for rec in report['recommendations']:
        summary += f"- [{rec['priority'].upper()}] {rec['message']}\n"
    
    summary += f"""
Next Steps:
"""
    for step in report['next_steps']:
        summary += f"- {step}\n"
    
    with open('pr_creation/pr_summary.txt', 'w') as f:
        f.write(summary)
    
    print(summary)
    
    return report


if __name__ == '__main__':
    generate_pr_creation_report()
```

## Final PR Creation Summary

**Complete the PR creation process:**

```bash
# Execute complete PR creation workflow
echo "=== EXECUTING COMPLETE PR CREATION WORKFLOW ==="

# Step 1: Initialize and analyze
mkdir -p pr_analysis pr_creation
python pr_analysis/implementation_analyzer.py

# Step 2: Generate PR description
python pr_creation/pr_description_generator.py

# Step 3: Create PR
gh pr create --title "$(cat pr_creation/pr_content.json | jq -r '.title')" \
             --body-file pr_creation/pr_description.md \
             --base "${input:target_branch}" \
             --assignee "${input:pr_author}"

# Step 4: Configure metadata
python pr_creation/pr_metadata_manager.py

# Step 5: Generate final report
python pr_creation/pr_creation_report.py

echo "✅ Pull request creation complete!"
echo "📊 Check pr_creation/ directory for all reports"
echo "🔗 PR URL: $(cat pr_creation/pr_info.json | jq -r '.pr_url')"
```

## Success Criteria

**PR creation is successful when:**

✅ **PR is created with comprehensive description**  
✅ **DD compliance is accurately documented**  
✅ **Appropriate labels and reviewers are assigned**  
✅ **Implementation quality metrics are included**  
✅ **Testing and deployment instructions are clear**  
✅ **Review checklist is complete**  
✅ **Milestone and priority are correctly set**  
✅ **All metadata is properly configured**

---

**Note**: This prompt creates a production-ready pull request with comprehensive documentation, quality metrics, and proper metadata based on the DD implementation plan from prompt 04.