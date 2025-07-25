---
description: Automated code quality review and security scanning based on DD implementation from prompt 04
---

# Django Design Detail (DD) Code Review & Security Scanner

This prompt creates comprehensive checklists for code quality and security reviews based on DD requirements, then performs automated reviews and security scans on the implemented code, generating detailed reports with findings and recommendations.

## Prerequisites

**Required Input:**
- 📋 **Implementation Plan**: Output from `04_django_dd_implementation_plan.prompt.md`
- 📁 **DD Requirements**: Original DD specification files
- 📦 **Project Name**: `${input:project_name}`
- 🏷️ **App Name**: `${input:app_name}`
- 📂 **Code Base Path**: `${input:code_base_path}`
- 📄 **Implementation Plan Path**: `${input:implementation_plan_path}`

## Review Process Workflow

### Step 1: Generate Quality Review Checklist

**Create comprehensive quality checklist based on DD requirements:**

```python
# File: review_checklists/quality_checklist.py

"""
Quality review checklist generator based on DD requirements
"""

import json
from datetime import datetime
from pathlib import Path
from typing import Dict, List, Any

class QualityChecklistGenerator:
    """Generate quality review checklist based on DD requirements"""
    
    def __init__(self, dd_requirements: Dict, project_name: str, app_name: str):
        self.dd_requirements = dd_requirements
        self.project_name = project_name
        self.app_name = app_name
        self.checklist = {
            'metadata': {
                'project': project_name,
                'app': app_name,
                'generated_at': datetime.now().isoformat(),
                'dd_name': '${input:dd_name}'
            },
            'categories': {}
        }
    
    def generate_django_best_practices_checklist(self) -> Dict:
        """Generate Django best practices checklist"""
        return {
            'name': 'Django Best Practices',
            'description': 'Standard Django development best practices',
            'weight': 25,
            'checks': [
                {
                    'id': 'DBP001',
                    'title': 'Model Field Definitions',
                    'description': 'All model fields have appropriate types and constraints',
                    'severity': 'high',
                    'criteria': [
                        'Fields use appropriate Django field types',
                        'Required fields are marked as required',
                        'String fields have appropriate max_length',
                        'Numeric fields have validators where needed',
                        'Choice fields use proper choices definition'
                    ],
                    'files_to_check': [f'apps/{self.app_name}/models.py'],
                    'automated_check': True
                },
                {
                    'id': 'DBP002',
                    'title': 'Model Meta Configuration',
                    'description': 'Models have proper Meta configuration',
                    'severity': 'medium',
                    'criteria': [
                        'Models have appropriate db_table names',
                        'Ordering is specified for display models',
                        'Indexes are defined for frequently queried fields',
                        'Unique constraints are properly defined',
                        'Verbose names are provided'
                    ],
                    'files_to_check': [f'apps/{self.app_name}/models.py'],
                    'automated_check': True
                },
                {
                    'id': 'DBP003',
                    'title': 'Serializer Implementation',
                    'description': 'Serializers follow DRF best practices',
                    'severity': 'high',
                    'criteria': [
                        'Read and write serializers are separated',
                        'Validation methods are implemented',
                        'Nested serialization is handled properly',
                        'Field-level validation is comprehensive',
                        'Custom validation follows DRF patterns'
                    ],
                    'files_to_check': [f'apps/{self.app_name}/serializers.py'],
                    'automated_check': True
                },
                {
                    'id': 'DBP004',
                    'title': 'View Implementation',
                    'description': 'Views follow REST API best practices',
                    'severity': 'high',
                    'criteria': [
                        'ViewSets use appropriate base classes',
                        'Permissions are properly configured',
                        'Queryset optimization (select_related, prefetch_related)',
                        'Custom actions are well-documented',
                        'Error handling is implemented'
                    ],
                    'files_to_check': [f'apps/{self.app_name}/views.py'],
                    'automated_check': True
                },
                {
                    'id': 'DBP005',
                    'title': 'URL Configuration',
                    'description': 'URL patterns are properly configured',
                    'severity': 'medium',
                    'criteria': [
                        'Router is used for ViewSets',
                        'URL names follow naming conventions',
                        'App namespace is defined',
                        'Custom URL patterns are documented',
                        'URL patterns are RESTful'
                    ],
                    'files_to_check': [f'apps/{self.app_name}/urls.py'],
                    'automated_check': True
                },
                {
                    'id': 'DBP006',
                    'title': 'Settings Configuration',
                    'description': 'Django settings follow best practices',
                    'severity': 'medium',
                    'criteria': [
                        'App is added to INSTALLED_APPS',
                        'Database settings are environment-specific',
                        'Static files configuration is correct',
                        'Security settings are properly configured',
                        'Environment variables are used for secrets'
                    ],
                    'files_to_check': ['config/settings/base.py', 'config/settings/development.py'],
                    'automated_check': True
                }
            ]
        }
    
    def generate_dd_requirements_checklist(self) -> Dict:
        """Generate checklist based on DD requirements"""
        return {
            'name': 'DD Requirements Compliance',
            'description': 'Compliance with Design Detail requirements',
            'weight': 35,
            'checks': [
                {
                    'id': 'DDR001',
                    'title': 'Business Logic Implementation',
                    'description': 'All business rules from DD are implemented',
                    'severity': 'critical',
                    'criteria': [
                        'All business rules are implemented in models or services',
                        'Validation rules match DD specifications',
                        'State transitions follow DD workflow',
                        'Calculated fields are implemented correctly',
                        'Business constraints are enforced'
                    ],
                    'dd_reference': 'business_rules',
                    'files_to_check': [f'apps/{self.app_name}/models.py', f'apps/{self.app_name}/serializers.py'],
                    'automated_check': True
                },
                {
                    'id': 'DDR002',
                    'title': 'API Endpoint Compliance',
                    'description': 'API endpoints match DD specifications',
                    'severity': 'critical',
                    'criteria': [
                        'All required endpoints are implemented',
                        'Request/response schemas match DD',
                        'HTTP methods are correctly used',
                        'Status codes follow DD specifications',
                        'Error responses are standardized'
                    ],
                    'dd_reference': 'api_endpoints',
                    'files_to_check': [f'apps/{self.app_name}/views.py', f'apps/{self.app_name}/serializers.py'],
                    'automated_check': True
                },
                {
                    'id': 'DDR003',
                    'title': 'Data Model Compliance',
                    'description': 'Data models match DD specifications',
                    'severity': 'critical',
                    'criteria': [
                        'All required fields are implemented',
                        'Field types match DD specifications',
                        'Relationships are correctly implemented',
                        'Constraints match DD requirements',
                        'Default values are set as specified'
                    ],
                    'dd_reference': 'data_models',
                    'files_to_check': [f'apps/{self.app_name}/models.py'],
                    'automated_check': True
                },
                {
                    'id': 'DDR004',
                    'title': 'Authentication & Authorization',
                    'description': 'Auth requirements from DD are implemented',
                    'severity': 'critical',
                    'criteria': [
                        'Authentication methods match DD requirements',
                        'Permission classes are correctly implemented',
                        'User roles and permissions are enforced',
                        'Protected endpoints require authentication',
                        'Authorization logic follows DD specifications'
                    ],
                    'dd_reference': 'authentication',
                    'files_to_check': [f'apps/{self.app_name}/views.py', f'apps/{self.app_name}/permissions.py'],
                    'automated_check': True
                },
                {
                    'id': 'DDR005',
                    'title': 'Performance Requirements',
                    'description': 'Performance requirements from DD are met',
                    'severity': 'high',
                    'criteria': [
                        'Database queries are optimized',
                        'Response time requirements are met',
                        'Pagination is implemented where required',
                        'Caching is used where specified',
                        'Bulk operations are efficient'
                    ],
                    'dd_reference': 'performance',
                    'files_to_check': [f'apps/{self.app_name}/views.py'],
                    'automated_check': True
                }
            ]
        }
    
    def generate_code_quality_checklist(self) -> Dict:
        """Generate code quality checklist"""
        return {
            'name': 'Code Quality Standards',
            'description': 'Code quality, maintainability and documentation',
            'weight': 20,
            'checks': [
                {
                    'id': 'CQS001',
                    'title': 'Code Documentation',
                    'description': 'Code is properly documented',
                    'severity': 'medium',
                    'criteria': [
                        'All classes have docstrings',
                        'All public methods have docstrings',
                        'Complex business logic is commented',
                        'API endpoints are documented',
                        'Model fields have help_text'
                    ],
                    'files_to_check': [f'apps/{self.app_name}/*.py'],
                    'automated_check': True
                },
                {
                    'id': 'CQS002',
                    'title': 'Code Style Compliance',
                    'description': 'Code follows PEP 8 and project style guidelines',
                    'severity': 'low',
                    'criteria': [
                        'Code follows PEP 8 style guidelines',
                        'Imports are properly organized',
                        'Line length is within limits',
                        'Naming conventions are consistent',
                        'Code is formatted consistently'
                    ],
                    'files_to_check': [f'apps/{self.app_name}/*.py'],
                    'automated_check': True
                },
                {
                    'id': 'CQS003',
                    'title': 'Error Handling',
                    'description': 'Proper error handling is implemented',
                    'severity': 'high',
                    'criteria': [
                        'Exceptions are properly handled',
                        'Error messages are user-friendly',
                        'Logging is implemented for errors',
                        'Graceful degradation is implemented',
                        'Validation errors are comprehensive'
                    ],
                    'files_to_check': [f'apps/{self.app_name}/*.py'],
                    'automated_check': True
                },
                {
                    'id': 'CQS004',
                    'title': 'Test Coverage',
                    'description': 'Adequate test coverage is provided',
                    'severity': 'high',
                    'criteria': [
                        'Unit tests cover critical functionality',
                        'Test coverage is above 80%',
                        'Edge cases are tested',
                        'Integration tests are provided',
                        'Tests are maintainable and readable'
                    ],
                    'files_to_check': [f'tests/test_{self.app_name}/*.py'],
                    'automated_check': True
                }
            ]
        }
    
    def generate_performance_checklist(self) -> Dict:
        """Generate performance review checklist"""
        return {
            'name': 'Performance Optimization',
            'description': 'Performance and scalability considerations',
            'weight': 20,
            'checks': [
                {
                    'id': 'PER001',
                    'title': 'Database Query Optimization',
                    'description': 'Database queries are optimized',
                    'severity': 'high',
                    'criteria': [
                        'N+1 query problems are avoided',
                        'select_related is used for ForeignKey fields',
                        'prefetch_related is used for ManyToMany fields',
                        'Database indexes are properly utilized',
                        'Bulk operations are used where appropriate'
                    ],
                    'files_to_check': [f'apps/{self.app_name}/views.py', f'apps/{self.app_name}/models.py'],
                    'automated_check': True
                },
                {
                    'id': 'PER002',
                    'title': 'API Response Time',
                    'description': 'API endpoints meet response time requirements',
                    'severity': 'high',
                    'criteria': [
                        'Endpoints respond within required time limits',
                        'Pagination is implemented for large datasets',
                        'Filtering reduces response payload',
                        'Unnecessary data is not included in responses',
                        'Caching is used where appropriate'
                    ],
                    'files_to_check': [f'apps/{self.app_name}/views.py'],
                    'automated_check': True
                },
                {
                    'id': 'PER003',
                    'title': 'Memory Usage',
                    'description': 'Memory usage is optimized',
                    'severity': 'medium',
                    'criteria': [
                        'Large querysets are paginated',
                        'Bulk operations use bulk_create/bulk_update',
                        'Temporary objects are properly cleaned up',
                        'Iterator is used for large datasets',
                        'Memory leaks are avoided'
                    ],
                    'files_to_check': [f'apps/{self.app_name}/*.py'],
                    'automated_check': True
                }
            ]
        }
    
    def generate_complete_checklist(self) -> Dict:
        """Generate complete quality checklist"""
        self.checklist['categories'] = {
            'django_best_practices': self.generate_django_best_practices_checklist(),
            'dd_requirements': self.generate_dd_requirements_checklist(),
            'code_quality': self.generate_code_quality_checklist(),
            'performance': self.generate_performance_checklist()
        }
        
        # Calculate total checks
        total_checks = sum(
            len(category['checks'])
            for category in self.checklist['categories'].values()
        )
        
        self.checklist['metadata']['total_checks'] = total_checks
        
        return self.checklist
    
    def save_checklist(self, output_path: str):
        """Save checklist to file"""
        checklist = self.generate_complete_checklist()
        
        with open(output_path, 'w') as f:
            json.dump(checklist, f, indent=2)
        
        return checklist


# Generate quality checklist
def generate_quality_checklist():
    """Main function to generate quality checklist"""
    
    # This would be loaded from actual DD requirements
    dd_requirements = {
        'business_rules': [
            'Archived items must have priority 0',
            'End date must be after start date',
            'Owner automatically gets admin role'
        ],
        'api_endpoints': {
            '/api/v1/items/': {
                'methods': ['GET', 'POST'],
                'authentication': 'required',
                'response_time_ms': 200
            }
        },
        'data_models': {
            'Item': {
                'fields': {
                    'name': {'type': 'CharField', 'max_length': 255, 'required': True},
                    'priority': {'type': 'IntegerField', 'min': 0, 'max': 100}
                }
            }
        },
        'authentication': {
            'method': 'token',
            'permissions': ['IsAuthenticated', 'IsOwnerOrReadOnly']
        },
        'performance': {
            'response_time_ms': 200,
            'pagination_size': 20
        }
    }
    
    generator = QualityChecklistGenerator(
        dd_requirements=dd_requirements,
        project_name='${input:project_name}',
        app_name='${input:app_name}'
    )
    
    checklist = generator.save_checklist('review_checklists/quality_checklist.json')
    
    print(f"Quality checklist generated with {checklist['metadata']['total_checks']} checks")
    return checklist


if __name__ == '__main__':
    generate_quality_checklist()
```

### Step 2: Generate Security Review Checklist

**Create comprehensive security checklist based on OWASP and Django security guidelines:**

```python
# File: review_checklists/security_checklist.py

"""
Security review checklist generator based on OWASP and Django security guidelines
"""

import json
from datetime import datetime
from typing import Dict, List

class SecurityChecklistGenerator:
    """Generate security review checklist"""
    
    def __init__(self, project_name: str, app_name: str):
        self.project_name = project_name
        self.app_name = app_name
        self.checklist = {
            'metadata': {
                'project': project_name,
                'app': app_name,
                'generated_at': datetime.now().isoformat(),
                'security_frameworks': ['OWASP Top 10', 'Django Security Guidelines']
            },
            'categories': {}
        }
    
    def generate_owasp_top10_checklist(self) -> Dict:
        """Generate OWASP Top 10 security checklist"""
        return {
            'name': 'OWASP Top 10 Security Risks',
            'description': 'Protection against OWASP Top 10 security vulnerabilities',
            'weight': 40,
            'checks': [
                {
                    'id': 'OWASP001',
                    'title': 'Injection Prevention',
                    'description': 'Protection against SQL, NoSQL, OS, and LDAP injection',
                    'severity': 'critical',
                    'owasp_category': 'A03:2021 – Injection',
                    'criteria': [
                        'Django ORM is used instead of raw SQL queries',
                        'User input is properly validated and sanitized',
                        'Parameterized queries are used when raw SQL is necessary',
                        'Input validation is performed server-side',
                        'Command injection is prevented in system calls'
                    ],
                    'files_to_check': [
                        f'apps/{self.app_name}/models.py',
                        f'apps/{self.app_name}/views.py',
                        f'apps/{self.app_name}/serializers.py'
                    ],
                    'automated_check': True,
                    'manual_check': True
                },
                {
                    'id': 'OWASP002',
                    'title': 'Authentication Security',
                    'description': 'Broken authentication and session management',
                    'severity': 'critical',
                    'owasp_category': 'A07:2021 – Identification and Authentication Failures',
                    'criteria': [
                        'Strong authentication mechanisms are implemented',
                        'Session tokens are properly managed',
                        'Password policies are enforced',
                        'Multi-factor authentication is considered',
                        'Account lockout mechanisms are in place'
                    ],
                    'files_to_check': [
                        f'apps/{self.app_name}/views.py',
                        'config/settings/*.py'
                    ],
                    'automated_check': True,
                    'manual_check': True
                },
                {
                    'id': 'OWASP003',
                    'title': 'Sensitive Data Exposure',
                    'description': 'Protection of sensitive data',
                    'severity': 'critical',
                    'owasp_category': 'A02:2021 – Cryptographic Failures',
                    'criteria': [
                        'Sensitive data is encrypted at rest and in transit',
                        'Secrets are not hardcoded in source code',
                        'Environment variables are used for configuration',
                        'Data is classified and protected accordingly',
                        'Unnecessary sensitive data is not stored'
                    ],
                    'files_to_check': [
                        f'apps/{self.app_name}/*.py',
                        'config/settings/*.py',
                        '.env*'
                    ],
                    'automated_check': True,
                    'manual_check': True
                },
                {
                    'id': 'OWASP004',
                    'title': 'XML External Entities (XXE)',
                    'description': 'Protection against XXE attacks',
                    'severity': 'high',
                    'owasp_category': 'A05:2021 – Security Misconfiguration',
                    'criteria': [
                        'XML parsers are configured securely',
                        'External entity processing is disabled',
                        'Input validation for XML data',
                        'Safe XML parsing libraries are used',
                        'File upload validation prevents XXE'
                    ],
                    'files_to_check': [f'apps/{self.app_name}/*.py'],
                    'automated_check': True,
                    'manual_check': False
                },
                {
                    'id': 'OWASP005',
                    'title': 'Broken Access Control',
                    'description': 'Proper access control implementation',
                    'severity': 'critical',
                    'owasp_category': 'A01:2021 – Broken Access Control',
                    'criteria': [
                        'Access control is enforced server-side',
                        'Users can only access their authorized resources',
                        'Administrative functions are properly protected',
                        'Direct object references are validated',
                        'Default access is deny'
                    ],
                    'files_to_check': [
                        f'apps/{self.app_name}/views.py',
                        f'apps/{self.app_name}/permissions.py'
                    ],
                    'automated_check': True,
                    'manual_check': True
                },
                {
                    'id': 'OWASP006',
                    'title': 'Security Misconfiguration',
                    'description': 'Secure configuration management',
                    'severity': 'high',
                    'owasp_category': 'A05:2021 – Security Misconfiguration',
                    'criteria': [
                        'Security headers are properly configured',
                        'Debug mode is disabled in production',
                        'Error messages do not leak sensitive information',
                        'Default credentials are changed',
                        'Unnecessary features are disabled'
                    ],
                    'files_to_check': [
                        'config/settings/*.py',
                        'config/urls.py'
                    ],
                    'automated_check': True,
                    'manual_check': True
                },
                {
                    'id': 'OWASP007',
                    'title': 'Cross-Site Scripting (XSS)',
                    'description': 'Protection against XSS attacks',
                    'severity': 'high',
                    'owasp_category': 'A03:2021 – Injection',
                    'criteria': [
                        'Output encoding is properly implemented',
                        'Input validation prevents script injection',
                        'Content Security Policy is configured',
                        'Safe rendering practices are used',
                        'User input is sanitized before storage'
                    ],
                    'files_to_check': [
                        f'apps/{self.app_name}/serializers.py',
                        f'apps/{self.app_name}/views.py'
                    ],
                    'automated_check': True,
                    'manual_check': True
                },
                {
                    'id': 'OWASP008',
                    'title': 'Insecure Deserialization',
                    'description': 'Safe deserialization practices',
                    'severity': 'high',
                    'owasp_category': 'A08:2021 – Software and Data Integrity Failures',
                    'criteria': [
                        'Serialization is performed safely',
                        'Untrusted data is not deserialized',
                        'Integrity checks are performed',
                        'Safe serialization libraries are used',
                        'Input validation for serialized data'
                    ],
                    'files_to_check': [
                        f'apps/{self.app_name}/serializers.py',
                        f'apps/{self.app_name}/views.py'
                    ],
                    'automated_check': True,
                    'manual_check': False
                },
                {
                    'id': 'OWASP009',
                    'title': 'Components with Known Vulnerabilities',
                    'description': 'Secure dependency management',
                    'severity': 'high',
                    'owasp_category': 'A06:2021 – Vulnerable and Outdated Components',
                    'criteria': [
                        'Dependencies are regularly updated',
                        'Vulnerability scanning is performed',
                        'Only necessary components are included',
                        'Component inventory is maintained',
                        'Security patches are applied promptly'
                    ],
                    'files_to_check': [
                        'requirements/*.txt',
                        'setup.py',
                        'pyproject.toml'
                    ],
                    'automated_check': True,
                    'manual_check': False
                },
                {
                    'id': 'OWASP010',
                    'title': 'Insufficient Logging & Monitoring',
                    'description': 'Adequate logging and monitoring',
                    'severity': 'medium',
                    'owasp_category': 'A09:2021 – Security Logging and Monitoring Failures',
                    'criteria': [
                        'Security events are logged',
                        'Logs are protected from tampering',
                        'Log format is consistent and parseable',
                        'Monitoring alerts are configured',
                        'Incident response procedures are in place'
                    ],
                    'files_to_check': [
                        f'apps/{self.app_name}/*.py',
                        'config/settings/*.py'
                    ],
                    'automated_check': True,
                    'manual_check': True
                }
            ]
        }
    
    def generate_django_security_checklist(self) -> Dict:
        """Generate Django-specific security checklist"""
        return {
            'name': 'Django Security Guidelines',
            'description': 'Django framework-specific security best practices',
            'weight': 35,
            'checks': [
                {
                    'id': 'DJS001',
                    'title': 'CSRF Protection',
                    'description': 'Cross-Site Request Forgery protection',
                    'severity': 'critical',
                    'criteria': [
                        'CSRF middleware is enabled',
                        'CSRF tokens are used in forms',
                        'CSRF exemptions are minimal and justified',
                        'SameSite cookie attribute is configured',
                        'CSRF_COOKIE_SECURE is True in production'
                    ],
                    'files_to_check': [
                        'config/settings/*.py',
                        f'apps/{self.app_name}/views.py'
                    ],
                    'automated_check': True
                },
                {
                    'id': 'DJS002',
                    'title': 'Clickjacking Protection',
                    'description': 'X-Frame-Options header configuration',
                    'severity': 'medium',
                    'criteria': [
                        'X-Frame-Options middleware is enabled',
                        'X_FRAME_OPTIONS is set appropriately',
                        'Frame embedding is controlled',
                        'Content Security Policy frame-ancestors is configured',
                        'Clickjacking decorators are used where needed'
                    ],
                    'files_to_check': [
                        'config/settings/*.py',
                        f'apps/{self.app_name}/views.py'
                    ],
                    'automated_check': True
                },
                {
                    'id': 'DJS003',
                    'title': 'SQL Injection Prevention',
                    'description': 'Django ORM and raw query security',
                    'severity': 'critical',
                    'criteria': [
                        'Django ORM is used for database queries',
                        'Raw SQL uses parameterized queries',
                        'User input is not concatenated into SQL',
                        'Quote characters are properly escaped',
                        'Database user has minimal privileges'
                    ],
                    'files_to_check': [
                        f'apps/{self.app_name}/models.py',
                        f'apps/{self.app_name}/views.py'
                    ],
                    'automated_check': True
                },
                {
                    'id': 'DJS004',
                    'title': 'Secret Key Security',
                    'description': 'Django SECRET_KEY protection',
                    'severity': 'critical',
                    'criteria': [
                        'SECRET_KEY is not hardcoded',
                        'SECRET_KEY is loaded from environment',
                        'SECRET_KEY is sufficiently random',
                        'SECRET_KEY is different per environment',
                        'SECRET_KEY is properly secured in production'
                    ],
                    'files_to_check': [
                        'config/settings/*.py',
                        '.env*'
                    ],
                    'automated_check': True
                },
                {
                    'id': 'DJS005',
                    'title': 'Debug Mode Security',
                    'description': 'Debug mode configuration',
                    'severity': 'critical',
                    'criteria': [
                        'DEBUG is False in production',
                        'Debug toolbar is not installed in production',
                        'Error pages do not leak sensitive information',
                        'ALLOWED_HOSTS is properly configured',
                        'Development-only middleware is removed'
                    ],
                    'files_to_check': [
                        'config/settings/*.py'
                    ],
                    'automated_check': True
                },
                {
                    'id': 'DJS006',
                    'title': 'HTTPS Configuration',
                    'description': 'Secure transport configuration',
                    'severity': 'critical',
                    'criteria': [
                        'SECURE_SSL_REDIRECT is True in production',
                        'SECURE_HSTS_SECONDS is configured',
                        'SECURE_HSTS_INCLUDE_SUBDOMAINS is True',
                        'SECURE_HSTS_PRELOAD is considered',
                        'SESSION_COOKIE_SECURE is True'
                    ],
                    'files_to_check': [
                        'config/settings/production.py'
                    ],
                    'automated_check': True
                },
                {
                    'id': 'DJS007',
                    'title': 'File Upload Security',
                    'description': 'Secure file upload handling',
                    'severity': 'high',
                    'criteria': [
                        'File types are validated',
                        'File size limits are enforced',
                        'Uploaded files are not executed',
                        'File names are sanitized',
                        'Files are stored outside web root'
                    ],
                    'files_to_check': [
                        f'apps/{self.app_name}/views.py',
                        f'apps/{self.app_name}/serializers.py'
                    ],
                    'automated_check': True
                },
                {
                    'id': 'DJS008',
                    'title': 'User Input Validation',
                    'description': 'Comprehensive input validation',
                    'severity': 'high',
                    'criteria': [
                        'All user input is validated',
                        'Input validation is performed server-side',
                        'Serializers have proper validation',
                        'Form validation is comprehensive',
                        'Input length limits are enforced'
                    ],
                    'files_to_check': [
                        f'apps/{self.app_name}/serializers.py',
                        f'apps/{self.app_name}/forms.py'
                    ],
                    'automated_check': True
                }
            ]
        }
    
    def generate_authentication_security_checklist(self) -> Dict:
        """Generate authentication and authorization security checklist"""
        return {
            'name': 'Authentication & Authorization Security',
            'description': 'Security of authentication and authorization mechanisms',
            'weight': 25,
            'checks': [
                {
                    'id': 'AUTH001',
                    'title': 'Password Security',
                    'description': 'Secure password handling',
                    'severity': 'critical',
                    'criteria': [
                        'Passwords are properly hashed',
                        'Strong password policies are enforced',
                        'Password reset is secure',
                        'Account lockout is implemented',
                        'Password history is maintained'
                    ],
                    'files_to_check': [
                        'config/settings/*.py',
                        f'apps/{self.app_name}/views.py'
                    ],
                    'automated_check': True
                },
                {
                    'id': 'AUTH002',
                    'title': 'Session Security',
                    'description': 'Secure session management',
                    'severity': 'high',
                    'criteria': [
                        'Session timeout is configured',
                        'Session cookies are secure',
                        'Session fixation is prevented',
                        'Sessions are invalidated on logout',
                        'Concurrent sessions are managed'
                    ],
                    'files_to_check': [
                        'config/settings/*.py',
                        f'apps/{self.app_name}/views.py'
                    ],
                    'automated_check': True
                },
                {
                    'id': 'AUTH003',
                    'title': 'Permission Enforcement',
                    'description': 'Proper permission checks',
                    'severity': 'critical',
                    'criteria': [
                        'Permissions are checked on all protected resources',
                        'Object-level permissions are enforced',
                        'Default access is deny',
                        'Administrative functions are protected',
                        'Permission inheritance is secure'
                    ],
                    'files_to_check': [
                        f'apps/{self.app_name}/views.py',
                        f'apps/{self.app_name}/permissions.py'
                    ],
                    'automated_check': True
                },
                {
                    'id': 'AUTH004',
                    'title': 'Token Security',
                    'description': 'API token security',
                    'severity': 'high',
                    'criteria': [
                        'Tokens are properly generated',
                        'Token expiration is implemented',
                        'Token transmission is secure',
                        'Token storage is secure',
                        'Token revocation is supported'
                    ],
                    'files_to_check': [
                        'config/settings/*.py',
                        f'apps/{self.app_name}/views.py'
                    ],
                    'automated_check': True
                }
            ]
        }
    
    def generate_complete_checklist(self) -> Dict:
        """Generate complete security checklist"""
        self.checklist['categories'] = {
            'owasp_top10': self.generate_owasp_top10_checklist(),
            'django_security': self.generate_django_security_checklist(),
            'auth_security': self.generate_authentication_security_checklist()
        }
        
        # Calculate total checks
        total_checks = sum(
            len(category['checks'])
            for category in self.checklist['categories'].values()
        )
        
        self.checklist['metadata']['total_checks'] = total_checks
        
        return self.checklist
    
    def save_checklist(self, output_path: str):
        """Save security checklist to file"""
        checklist = self.generate_complete_checklist()
        
        with open(output_path, 'w') as f:
            json.dump(checklist, f, indent=2)
        
        return checklist


# Generate security checklist
def generate_security_checklist():
    """Main function to generate security checklist"""
    
    generator = SecurityChecklistGenerator(
        project_name='${input:project_name}',
        app_name='${input:app_name}'
    )
    
    checklist = generator.save_checklist('review_checklists/security_checklist.json')
    
    print(f"Security checklist generated with {checklist['metadata']['total_checks']} checks")
    return checklist


if __name__ == '__main__':
    generate_security_checklist()
```

### Step 3: Automated Code Review Engine

**Create automated code review engine:**

```python
# File: code_review_engine.py

"""
Automated code review engine based on generated checklists
"""

import os
import re
import ast
import json
import subprocess
from pathlib import Path
from typing import Dict, List, Any, Tuple
from datetime import datetime

class CodeReviewEngine:
    """Automated code review engine"""
    
    def __init__(self, code_base_path: str, app_name: str):
        self.code_base_path = Path(code_base_path)
        self.app_name = app_name
        self.results = {
            'metadata': {
                'review_date': datetime.now().isoformat(),
                'app_name': app_name,
                'code_base_path': str(code_base_path)
            },
            'quality_review': {},
            'security_review': {},
            'summary': {}
        }
    
    def load_checklists(self):
        """Load quality and security checklists"""
        with open('review_checklists/quality_checklist.json', 'r') as f:
            self.quality_checklist = json.load(f)
        
        with open('review_checklists/security_checklist.json', 'r') as f:
            self.security_checklist = json.load(f)
    
    def analyze_python_file(self, file_path: Path) -> Dict:
        """Analyze Python file for code quality and security issues"""
        try:
            with open(file_path, 'r', encoding='utf-8') as f:
                content = f.read()
            
            # Parse AST
            tree = ast.parse(content)
            
            analysis = {
                'file_path': str(file_path),
                'lines_of_code': len(content.splitlines()),
                'classes': [],
                'functions': [],
                'imports': [],
                'docstrings': {
                    'class_docstrings': 0,
                    'function_docstrings': 0,
                    'total_classes': 0,
                    'total_functions': 0
                },
                'security_patterns': [],
                'code_quality_issues': []
            }
            
            # Analyze AST nodes
            for node in ast.walk(tree):
                if isinstance(node, ast.ClassDef):
                    analysis['classes'].append(node.name)
                    analysis['docstrings']['total_classes'] += 1
                    if ast.get_docstring(node):
                        analysis['docstrings']['class_docstrings'] += 1
                
                elif isinstance(node, ast.FunctionDef):
                    analysis['functions'].append(node.name)
                    analysis['docstrings']['total_functions'] += 1
                    if ast.get_docstring(node):
                        analysis['docstrings']['function_docstrings'] += 1
                
                elif isinstance(node, ast.Import):
                    for alias in node.names:
                        analysis['imports'].append(alias.name)
                
                elif isinstance(node, ast.ImportFrom):
                    if node.module:
                        analysis['imports'].append(node.module)
            
            # Security pattern analysis
            analysis['security_patterns'] = self.check_security_patterns(content)
            
            # Code quality analysis
            analysis['code_quality_issues'] = self.check_code_quality(content, tree)
            
            return analysis
            
        except Exception as e:
            return {
                'file_path': str(file_path),
                'error': str(e),
                'analysis_failed': True
            }
    
    def check_security_patterns(self, content: str) -> List[Dict]:
        """Check for security-related patterns in code"""
        patterns = []
        
        # SQL injection patterns
        sql_patterns = [
            r'\.raw\s*\(',  # Raw SQL usage
            r'\.extra\s*\(',  # Extra SQL usage
            r'format\s*\(\s*["\'].*%.*["\']',  # String formatting in SQL
            r'%\s*\w+.*["\'].*SELECT.*["\']',  # SQL concatenation
        ]
        
        for pattern in sql_patterns:
            matches = re.finditer(pattern, content, re.IGNORECASE)
            for match in matches:
                patterns.append({
                    'type': 'potential_sql_injection',
                    'pattern': pattern,
                    'line': content[:match.start()].count('\n') + 1,
                    'severity': 'high'
                })
        
        # Hardcoded secrets patterns
        secret_patterns = [
            r'SECRET_KEY\s*=\s*["\'][^"\']+["\']',  # Hardcoded SECRET_KEY
            r'PASSWORD\s*=\s*["\'][^"\']+["\']',  # Hardcoded passwords
            r'API_KEY\s*=\s*["\'][^"\']+["\']',  # Hardcoded API keys
            r'TOKEN\s*=\s*["\'][^"\']+["\']',  # Hardcoded tokens
        ]
        
        for pattern in secret_patterns:
            matches = re.finditer(pattern, content, re.IGNORECASE)
            for match in matches:
                patterns.append({
                    'type': 'hardcoded_secret',
                    'pattern': pattern,
                    'line': content[:match.start()].count('\n') + 1,
                    'severity': 'critical'
                })
        
        # Debug patterns
        debug_patterns = [
            r'DEBUG\s*=\s*True',  # Debug mode
            r'print\s*\(',  # Print statements
            r'pdb\.set_trace\(\)',  # Debugger
        ]
        
        for pattern in debug_patterns:
            matches = re.finditer(pattern, content, re.IGNORECASE)
            for match in matches:
                patterns.append({
                    'type': 'debug_code',
                    'pattern': pattern,
                    'line': content[:match.start()].count('\n') + 1,
                    'severity': 'medium'
                })
        
        return patterns
    
    def check_code_quality(self, content: str, tree: ast.AST) -> List[Dict]:
        """Check code quality issues"""
        issues = []
        
        # Check for TODO/FIXME comments
        todo_pattern = r'#.*(?:TODO|FIXME|HACK|XXX)'
        matches = re.finditer(todo_pattern, content, re.IGNORECASE)
        for match in matches:
            issues.append({
                'type': 'todo_comment',
                'line': content[:match.start()].count('\n') + 1,
                'severity': 'low',
                'message': 'TODO/FIXME comment found'
            })
        
        # Check for long lines
        lines = content.splitlines()
        for i, line in enumerate(lines, 1):
            if len(line) > 120:  # PEP 8 recommends 79, but 120 is common
                issues.append({
                    'type': 'line_too_long',
                    'line': i,
                    'severity': 'low',
                    'message': f'Line too long ({len(line)} characters)'
                })
        
        # Check for complex functions (high cyclomatic complexity)
        for node in ast.walk(tree):
            if isinstance(node, ast.FunctionDef):
                complexity = self.calculate_complexity(node)
                if complexity > 10:
                    issues.append({
                        'type': 'high_complexity',
                        'line': node.lineno,
                        'severity': 'medium',
                        'message': f'Function {node.name} has high complexity ({complexity})'
                    })
        
        return issues
    
    def calculate_complexity(self, node: ast.FunctionDef) -> int:
        """Calculate cyclomatic complexity of a function"""
        complexity = 1  # Base complexity
        
        for child in ast.walk(node):
            if isinstance(child, (ast.If, ast.While, ast.For, ast.AsyncFor)):
                complexity += 1
            elif isinstance(child, ast.ExceptHandler):
                complexity += 1
            elif isinstance(child, ast.With, ast.AsyncWith):
                complexity += 1
        
        return complexity
    
    def run_external_tools(self) -> Dict:
        """Run external code analysis tools"""
        tools_results = {}
        
        # Run flake8 for style checking
        try:
            result = subprocess.run(
                ['flake8', f'apps/{self.app_name}', '--format=json'],
                capture_output=True,
                text=True,
                cwd=self.code_base_path
            )
            
            if result.stdout:
                tools_results['flake8'] = json.loads(result.stdout)
            else:
                tools_results['flake8'] = []
                
        except Exception as e:
            tools_results['flake8'] = {'error': str(e)}
        
        # Run bandit for security checking
        try:
            result = subprocess.run(
                ['bandit', '-r', f'apps/{self.app_name}', '-f', 'json'],
                capture_output=True,
                text=True,
                cwd=self.code_base_path
            )
            
            if result.stdout:
                tools_results['bandit'] = json.loads(result.stdout)
            else:
                tools_results['bandit'] = {}
                
        except Exception as e:
            tools_results['bandit'] = {'error': str(e)}
        
        # Run safety for dependency checking
        try:
            result = subprocess.run(
                ['safety', 'check', '--json'],
                capture_output=True,
                text=True,
                cwd=self.code_base_path
            )
            
            if result.stdout:
                tools_results['safety'] = json.loads(result.stdout)
            else:
                tools_results['safety'] = []
                
        except Exception as e:
            tools_results['safety'] = {'error': str(e)}
        
        return tools_results
    
    def review_quality_checklist(self) -> Dict:
        """Review code against quality checklist"""
        quality_results = {
            'categories': {},
            'total_score': 0,
            'max_score': 0,
            'issues': []
        }
        
        # Analyze Python files
        python_files = list(self.code_base_path.glob(f'apps/{self.app_name}/**/*.py'))
        file_analyses = []
        
        for file_path in python_files:
            analysis = self.analyze_python_file(file_path)
            file_analyses.append(analysis)
        
        # Review each category
        for category_name, category in self.quality_checklist['categories'].items():
            category_result = {
                'name': category['name'],
                'weight': category['weight'],
                'checks': [],
                'score': 0,
                'max_score': 0,
                'issues': []
            }
            
            for check in category['checks']:
                check_result = self.evaluate_quality_check(check, file_analyses)
                category_result['checks'].append(check_result)
                category_result['score'] += check_result['score']
                category_result['max_score'] += check_result['max_score']
                category_result['issues'].extend(check_result['issues'])
            
            quality_results['categories'][category_name] = category_result
            quality_results['total_score'] += category_result['score']
            quality_results['max_score'] += category_result['max_score']
            quality_results['issues'].extend(category_result['issues'])
        
        return quality_results
    
    def evaluate_quality_check(self, check: Dict, file_analyses: List[Dict]) -> Dict:
        """Evaluate a single quality check"""
        check_result = {
            'id': check['id'],
            'title': check['title'],
            'severity': check['severity'],
            'score': 0,
            'max_score': 100,
            'status': 'pending',
            'issues': [],
            'files_checked': []
        }
        
        # Quality check evaluation logic
        if check['id'] == 'DBP001':  # Model field definitions
            check_result = self.check_model_fields(check, file_analyses)
        elif check['id'] == 'DBP002':  # Model Meta configuration
            check_result = self.check_model_meta(check, file_analyses)
        elif check['id'] == 'DBP003':  # Serializer implementation
            check_result = self.check_serializers(check, file_analyses)
        elif check['id'] == 'DBP004':  # View implementation
            check_result = self.check_views(check, file_analyses)
        elif check['id'] == 'CQS001':  # Code documentation
            check_result = self.check_documentation(check, file_analyses)
        elif check['id'] == 'CQS002':  # Code style
            check_result = self.check_code_style(check, file_analyses)
        else:
            # Default evaluation
            check_result['status'] = 'manual_review_required'
            check_result['score'] = 50  # Neutral score for manual review
        
        return check_result
    
    def check_model_fields(self, check: Dict, file_analyses: List[Dict]) -> Dict:
        """Check model field definitions"""
        result = {
            'id': check['id'],
            'title': check['title'],
            'severity': check['severity'],
            'score': 0,
            'max_score': 100,
            'status': 'completed',
            'issues': [],
            'files_checked': []
        }
        
        model_files = [f for f in file_analyses if 'models.py' in f.get('file_path', '')]
        
        if not model_files:
            result['issues'].append({
                'type': 'missing_file',
                'message': 'No models.py file found',
                'severity': 'high'
            })
            result['score'] = 0
            return result
        
        # Check for proper field definitions
        total_checks = 0
        passed_checks = 0
        
        for model_file in model_files:
            result['files_checked'].append(model_file['file_path'])
            
            # Check if models are defined
            if model_file.get('classes'):
                total_checks += 1
                passed_checks += 1
            else:
                result['issues'].append({
                    'type': 'no_models',
                    'file': model_file['file_path'],
                    'message': 'No model classes found',
                    'severity': 'medium'
                })
        
        if total_checks > 0:
            result['score'] = (passed_checks / total_checks) * 100
        
        return result
    
    def check_documentation(self, check: Dict, file_analyses: List[Dict]) -> Dict:
        """Check code documentation"""
        result = {
            'id': check['id'],
            'title': check['title'],
            'severity': check['severity'],
            'score': 0,
            'max_score': 100,
            'status': 'completed',
            'issues': [],
            'files_checked': []
        }
        
        total_classes = 0
        documented_classes = 0
        total_functions = 0
        documented_functions = 0
        
        for analysis in file_analyses:
            if analysis.get('analysis_failed'):
                continue
                
            result['files_checked'].append(analysis['file_path'])
            
            docstrings = analysis.get('docstrings', {})
            total_classes += docstrings.get('total_classes', 0)
            documented_classes += docstrings.get('class_docstrings', 0)
            total_functions += docstrings.get('total_functions', 0)
            documented_functions += docstrings.get('function_docstrings', 0)
        
        # Calculate documentation coverage
        class_coverage = (documented_classes / total_classes * 100) if total_classes > 0 else 100
        function_coverage = (documented_functions / total_functions * 100) if total_functions > 0 else 100
        
        overall_coverage = (class_coverage + function_coverage) / 2
        result['score'] = overall_coverage
        
        # Add issues for poor documentation
        if class_coverage < 80:
            result['issues'].append({
                'type': 'poor_class_documentation',
                'message': f'Class documentation coverage: {class_coverage:.1f}%',
                'severity': 'medium'
            })
        
        if function_coverage < 80:
            result['issues'].append({
                'type': 'poor_function_documentation',
                'message': f'Function documentation coverage: {function_coverage:.1f}%',
                'severity': 'medium'
            })
        
        return result
    
    def review_security_checklist(self) -> Dict:
        """Review code against security checklist"""
        security_results = {
            'categories': {},
            'total_score': 0,
            'max_score': 0,
            'vulnerabilities': [],
            'critical_issues': 0,
            'high_issues': 0,
            'medium_issues': 0,
            'low_issues': 0
        }
        
        # Run external security tools
        external_tools = self.run_external_tools()
        
        # Analyze Python files for security issues
        python_files = list(self.code_base_path.glob(f'apps/{self.app_name}/**/*.py'))
        file_analyses = []
        
        for file_path in python_files:
            analysis = self.analyze_python_file(file_path)
            file_analyses.append(analysis)
        
        # Review each security category
        for category_name, category in self.security_checklist['categories'].items():
            category_result = {
                'name': category['name'],
                'weight': category['weight'],
                'checks': [],
                'score': 0,
                'max_score': 0,
                'vulnerabilities': []
            }
            
            for check in category['checks']:
                check_result = self.evaluate_security_check(check, file_analyses, external_tools)
                category_result['checks'].append(check_result)
                category_result['score'] += check_result['score']
                category_result['max_score'] += check_result['max_score']
                category_result['vulnerabilities'].extend(check_result['vulnerabilities'])
            
            security_results['categories'][category_name] = category_result
            security_results['total_score'] += category_result['score']
            security_results['max_score'] += category_result['max_score']
            security_results['vulnerabilities'].extend(category_result['vulnerabilities'])
        
        # Count vulnerability severities
        for vuln in security_results['vulnerabilities']:
            severity = vuln.get('severity', 'low')
            if severity == 'critical':
                security_results['critical_issues'] += 1
            elif severity == 'high':
                security_results['high_issues'] += 1
            elif severity == 'medium':
                security_results['medium_issues'] += 1
            else:
                security_results['low_issues'] += 1
        
        return security_results
    
    def evaluate_security_check(self, check: Dict, file_analyses: List[Dict], external_tools: Dict) -> Dict:
        """Evaluate a single security check"""
        check_result = {
            'id': check['id'],
            'title': check['title'],
            'severity': check['severity'],
            'score': 0,
            'max_score': 100,
            'status': 'completed',
            'vulnerabilities': [],
            'files_checked': []
        }
        
        # Security check evaluation logic
        if check['id'] == 'OWASP001':  # Injection prevention
            check_result = self.check_injection_prevention(check, file_analyses)
        elif check['id'] == 'OWASP003':  # Sensitive data exposure
            check_result = self.check_sensitive_data_exposure(check, file_analyses)
        elif check['id'] == 'DJS003':  # SQL injection prevention
            check_result = self.check_sql_injection_prevention(check, file_analyses)
        elif check['id'] == 'DJS004':  # Secret key security
            check_result = self.check_secret_key_security(check, file_analyses)
        elif check['id'] == 'DJS005':  # Debug mode security
            check_result = self.check_debug_mode_security(check, file_analyses)
        else:
            # Use external tool results for other checks
            check_result = self.evaluate_with_external_tools(check, external_tools)
        
        return check_result
    
    def check_injection_prevention(self, check: Dict, file_analyses: List[Dict]) -> Dict:
        """Check for injection prevention measures"""
        result = {
            'id': check['id'],
            'title': check['title'],
            'severity': check['severity'],
            'score': 100,  # Start with perfect score
            'max_score': 100,
            'status': 'completed',
            'vulnerabilities': [],
            'files_checked': []
        }
        
        for analysis in file_analyses:
            if analysis.get('analysis_failed'):
                continue
                
            result['files_checked'].append(analysis['file_path'])
            
            # Check for SQL injection patterns
            for pattern in analysis.get('security_patterns', []):
                if pattern['type'] == 'potential_sql_injection':
                    result['vulnerabilities'].append({
                        'type': 'potential_sql_injection',
                        'file': analysis['file_path'],
                        'line': pattern['line'],
                        'severity': pattern['severity'],
                        'message': 'Potential SQL injection vulnerability detected'
                    })
                    result['score'] -= 20  # Deduct points for each issue
        
        result['score'] = max(0, result['score'])  # Ensure score doesn't go negative
        return result
    
    def check_sensitive_data_exposure(self, check: Dict, file_analyses: List[Dict]) -> Dict:
        """Check for sensitive data exposure"""
        result = {
            'id': check['id'],
            'title': check['title'],
            'severity': check['severity'],
            'score': 100,
            'max_score': 100,
            'status': 'completed',
            'vulnerabilities': [],
            'files_checked': []
        }
        
        for analysis in file_analyses:
            if analysis.get('analysis_failed'):
                continue
                
            result['files_checked'].append(analysis['file_path'])
            
            # Check for hardcoded secrets
            for pattern in analysis.get('security_patterns', []):
                if pattern['type'] == 'hardcoded_secret':
                    result['vulnerabilities'].append({
                        'type': 'hardcoded_secret',
                        'file': analysis['file_path'],
                        'line': pattern['line'],
                        'severity': 'critical',
                        'message': 'Hardcoded secret detected'
                    })
                    result['score'] -= 30  # Major deduction for hardcoded secrets
        
        result['score'] = max(0, result['score'])
        return result
    
    def generate_review_report(self) -> Dict:
        """Generate comprehensive review report"""
        # Perform reviews
        quality_results = self.review_quality_checklist()
        security_results = self.review_security_checklist()
        
        # Calculate overall scores
        quality_percentage = (quality_results['total_score'] / quality_results['max_score'] * 100) if quality_results['max_score'] > 0 else 0
        security_percentage = (security_results['total_score'] / security_results['max_score'] * 100) if security_results['max_score'] > 0 else 0
        
        overall_score = (quality_percentage + security_percentage) / 2
        
        # Determine grade
        if overall_score >= 90:
            grade = 'A'
        elif overall_score >= 80:
            grade = 'B'
        elif overall_score >= 70:
            grade = 'C'
        elif overall_score >= 60:
            grade = 'D'
        else:
            grade = 'F'
        
        self.results['quality_review'] = quality_results
        self.results['security_review'] = security_results
        self.results['summary'] = {
            'overall_score': overall_score,
            'grade': grade,
            'quality_score': quality_percentage,
            'security_score': security_percentage,
            'total_issues': len(quality_results['issues']) + len(security_results['vulnerabilities']),
            'critical_security_issues': security_results['critical_issues'],
            'recommendations': self.generate_recommendations(quality_results, security_results)
        }
        
        return self.results
    
    def generate_recommendations(self, quality_results: Dict, security_results: Dict) -> List[str]:
        """Generate improvement recommendations"""
        recommendations = []
        
        # Quality recommendations
        if quality_results['total_score'] / quality_results['max_score'] < 0.8:
            recommendations.append("Improve code quality by addressing identified issues")
        
        # Security recommendations
        if security_results['critical_issues'] > 0:
            recommendations.append("Address critical security vulnerabilities immediately")
        
        if security_results['high_issues'] > 0:
            recommendations.append("Review and fix high-severity security issues")
        
        # Documentation recommendations
        for category in quality_results['categories'].values():
            for check in category['checks']:
                if check['id'] == 'CQS001' and check['score'] < 80:
                    recommendations.append("Improve code documentation coverage")
        
        # General recommendations
        recommendations.extend([
            "Run automated security scans regularly",
            "Implement code review process for all changes",
            "Set up continuous integration with quality gates"
        ])
        
        return recommendations


def main():
    """Main review execution function"""
    print("=== DJANGO DD CODE REVIEW & SECURITY SCAN ===\n")
    
    # Initialize review engine
    engine = CodeReviewEngine('${input:code_base_path}', '${input:app_name}')
    
    # Load checklists
    print("Loading review checklists...")
    engine.load_checklists()
    
    # Generate review report
    print("Performing code review and security scan...")
    results = engine.generate_review_report()
    
    # Save results
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    report_file = f'review_reports/review_report_{timestamp}.json'
    
    os.makedirs('review_reports', exist_ok=True)
    with open(report_file, 'w') as f:
        json.dump(results, f, indent=2)
    
    # Print summary
    summary = results['summary']
    print(f"\n=== REVIEW SUMMARY ===")
    print(f"Overall Score: {summary['overall_score']:.1f}%")
    print(f"Grade: {summary['grade']}")
    print(f"Quality Score: {summary['quality_score']:.1f}%")
    print(f"Security Score: {summary['security_score']:.1f}%")
    print(f"Total Issues: {summary['total_issues']}")
    print(f"Critical Security Issues: {summary['critical_security_issues']}")
    
    print(f"\nDetailed report saved to: {report_file}")
    
    return results


if __name__ == '__main__':
    main()
```

### Step 4: Setup and Execution Script

**Create setup and execution script:**

```bash
echo "=== SETTING UP CODE REVIEW & SECURITY SCANNER ==="

# Create directory structure
mkdir -p review_checklists
mkdir -p review_reports
mkdir -p security_scans

# Install required tools
echo "Installing security and quality analysis tools..."

# Install Python security tools
pip install bandit safety flake8 pylint

# Install additional security tools if available
if command -v npm &> /dev/null; then
    npm install -g audit-ci
fi

# Generate checklists
echo "Generating review checklists..."
python review_checklists/quality_checklist.py
python review_checklists/security_checklist.py

# Run comprehensive review
echo "Running comprehensive code review and security scan..."
python code_review_engine.py

echo "✅ Code review and security scan completed!"
```

### Step 5: Report Generation

**Generate comprehensive reports:**

```python
# File: generate_review_reports.py

"""
Generate comprehensive review reports in multiple formats
"""

import json
import os
from pathlib import Path
from datetime import datetime
from typing import Dict

def generate_html_report(results: Dict, output_path: str):
    """Generate HTML review report"""
    
    summary = results['summary']
    quality = results['quality_review']
    security = results['security_review']
    
    # Determine color based on grade
    grade_colors = {
        'A': '#4CAF50',
        'B': '#8BC34A', 
        'C': '#FFC107',
        'D': '#FF9800',
        'F': '#F44336'
    }
    
    grade_color = grade_colors.get(summary['grade'], '#666')
    
    html_content = f"""
<!DOCTYPE html>
<html>
<head>
    <title>Code Review Report - {results['metadata']['app_name']}</title>
    <style>
        body {{ font-family: Arial, sans-serif; margin: 20px; }}
        .header {{ background: #f5f5f5; padding: 20px; border-radius: 5px; }}
        .grade {{ font-size: 48px; font-weight: bold; color: {grade_color}; }}
        .score {{ font-size: 24px; margin: 10px 0; }}
        .category {{ margin: 20px 0; border: 1px solid #ddd; padding: 15px; border-radius: 5px; }}
        .critical {{ color: #F44336; }}
        .high {{ color: #FF9800; }}
        .medium {{ color: #FFC107; }}
        .low {{ color: #4CAF50; }}
        table {{ border-collapse: collapse; width: 100%; margin: 10px 0; }}
        th, td {{ border: 1px solid #ddd; padding: 8px; text-align: left; }}
        th {{ background-color: #f2f2f2; }}
        .progress-bar {{ width: 100%; height: 20px; background: #f0f0f0; border-radius: 10px; }}
        .progress-fill {{ height: 100%; background: #4CAF50; border-radius: 10px; }}
        .recommendations {{ background: #e3f2fd; padding: 15px; border-radius: 5px; }}
    </style>
</head>
<body>
    <div class="header">
        <h1>Code Review Report: {results['metadata']['app_name']}</h1>
        <p>Generated: {results['metadata']['review_date']}</p>
        
        <div style="display: flex; align-items: center; gap: 30px;">
            <div class="grade">{summary['grade']}</div>
            <div>
                <div class="score">Overall Score: {summary['overall_score']:.1f}%</div>
                <div>Quality: {summary['quality_score']:.1f}% | Security: {summary['security_score']:.1f}%</div>
            </div>
        </div>
    </div>
    
    <div class="category">
        <h2>Summary</h2>
        <table>
            <tr><th>Metric</th><th>Value</th></tr>
            <tr><td>Overall Score</td><td>{summary['overall_score']:.1f}%</td></tr>
            <tr><td>Grade</td><td>{summary['grade']}</td></tr>
            <tr><td>Total Issues</td><td>{summary['total_issues']}</td></tr>
            <tr><td>Critical Security Issues</td><td class="critical">{summary['critical_security_issues']}</td></tr>
        </table>
    </div>
    
    <div class="category">
        <h2>Quality Review Results</h2>
        <div class="progress-bar">
            <div class="progress-fill" style="width: {summary['quality_score']}%"></div>
        </div>
        <p>Score: {summary['quality_score']:.1f}% ({quality['total_score']}/{quality['max_score']} points)</p>
        
        <h3>Category Breakdown</h3>
        <table>
            <tr><th>Category</th><th>Score</th><th>Issues</th></tr>
    """
    
    for category_name, category in quality['categories'].items():
        category_score = (category['score'] / category['max_score'] * 100) if category['max_score'] > 0 else 0
        html_content += f"""
            <tr>
                <td>{category['name']}</td>
                <td>{category_score:.1f}%</td>
                <td>{len(category['issues'])}</td>
            </tr>
        """
    
    html_content += """
        </table>
    </div>
    
    <div class="category">
        <h2>Security Review Results</h2>
        <div class="progress-bar">
            <div class="progress-fill" style="width: """ + str(summary['security_score']) + """%"></div>
        </div>
        <p>Score: """ + f"{summary['security_score']:.1f}% ({security['total_score']}/{security['max_score']} points)" + """</p>
        
        <h3>Vulnerability Summary</h3>
        <table>
            <tr><th>Severity</th><th>Count</th></tr>
            <tr><td class="critical">Critical</td><td>""" + str(security['critical_issues']) + """</td></tr>
            <tr><td class="high">High</td><td>""" + str(security['high_issues']) + """</td></tr>
            <tr><td class="medium">Medium</td><td>""" + str(security['medium_issues']) + """</td></tr>
            <tr><td class="low">Low</td><td>""" + str(security['low_issues']) + """</td></tr>
        </table>
    </div>
    """
    
    # Add recommendations
    html_content += f"""
    <div class="recommendations">
        <h2>Recommendations</h2>
        <ul>
    """
    
    for recommendation in summary['recommendations']:
        html_content += f"<li>{recommendation}</li>"
    
    html_content += """
        </ul>
    </div>
    
    <div class="category">
        <h2>Detailed Findings</h2>
        <h3>Quality Issues</h3>
        <table>
            <tr><th>Check ID</th><th>Title</th><th>Severity</th><th>Score</th><th>Issues</th></tr>
    """
    
    for category in quality['categories'].values():
        for check in category['checks']:
            html_content += f"""
            <tr>
                <td>{check['id']}</td>
                <td>{check['title']}</td>
                <td class="{check['severity']}">{check['severity']}</td>
                <td>{check['score']:.1f}%</td>
                <td>{len(check.get('issues', []))}</td>
            </tr>
            """
    
    html_content += """
        </table>
        
        <h3>Security Vulnerabilities</h3>
        <table>
            <tr><th>Check ID</th><th>Title</th><th>Severity</th><th>Vulnerabilities</th></tr>
    """
    
    for category in security['categories'].values():
        for check in category['checks']:
            html_content += f"""
            <tr>
                <td>{check['id']}</td>
                <td>{check['title']}</td>
                <td class="{check['severity']}">{check['severity']}</td>
                <td>{len(check.get('vulnerabilities', []))}</td>
            </tr>
            """
    
    html_content += """
        </table>
    </div>
</body>
</html>
    """
    
    with open(output_path, 'w') as f:
        f.write(html_content)


def generate_markdown_report(results: Dict, output_path: str):
    """Generate Markdown review report"""
    
    summary = results['summary']
    quality = results['quality_review']
    security = results['security_review']
    
    md_content = f"""# Code Review Report: {results['metadata']['app_name']}

**Generated**: {results['metadata']['review_date']}

## Summary

| Metric | Value |
|--------|-------|
| **Overall Score** | {summary['overall_score']:.1f}% |
| **Grade** | **{summary['grade']}** |
| **Quality Score** | {summary['quality_score']:.1f}% |
| **Security Score** | {summary['security_score']:.1f}% |
| **Total Issues** | {summary['total_issues']} |
| **Critical Security Issues** | {summary['critical_security_issues']} |

## Quality Review Results

**Score**: {summary['quality_score']:.1f}% ({quality['total_score']}/{quality['max_score']} points)

### Category Breakdown

| Category | Score | Issues |
|----------|-------|--------|
"""
    
    for category_name, category in quality['categories'].items():
        category_score = (category['score'] / category['max_score'] * 100) if category['max_score'] > 0 else 0
        md_content += f"| {category['name']} | {category_score:.1f}% | {len(category['issues'])} |\n"
    
    md_content += f"""

## Security Review Results

**Score**: {summary['security_score']:.1f}% ({security['total_score']}/{security['max_score']} points)

### Vulnerability Summary

| Severity | Count |
|----------|-------|
| Critical | {security['critical_issues']} |
| High | {security['high_issues']} |
| Medium | {security['medium_issues']} |
| Low | {security['low_issues']} |

## Recommendations

"""
    
    for recommendation in summary['recommendations']:
        md_content += f"- {recommendation}\n"
    
    md_content += """

## Detailed Findings

### Quality Issues

| Check ID | Title | Severity | Score | Issues |
|----------|-------|----------|-------|--------|
"""
    
    for category in quality['categories'].values():
        for check in category['checks']:
            md_content += f"| {check['id']} | {check['title']} | {check['severity']} | {check['score']:.1f}% | {len(check.get('issues', []))} |\n"
    
    md_content += """

### Security Vulnerabilities

| Check ID | Title | Severity | Vulnerabilities |
|----------|-------|----------|----------------|
"""
    
    for category in security['categories'].values():
        for check in category['checks']:
            md_content += f"| {check['id']} | {check['title']} | {check['severity']} | {len(check.get('vulnerabilities', []))} |\n"
    
    with open(output_path, 'w') as f:
        f.write(md_content)


def main():
    """Generate all review reports"""
    
    # Find latest review report
    report_files = sorted(Path('review_reports').glob('review_report_*.json'), reverse=True)
    
    if not report_files:
        print("No review reports found!")
        return
    
    latest_report = report_files[0]
    
    # Load results
    with open(latest_report, 'r') as f:
        results = json.load(f)
    
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    
    # Generate HTML report
    html_path = f'review_reports/review_report_{timestamp}.html'
    generate_html_report(results, html_path)
    print(f"HTML report generated: {html_path}")
    
    # Generate Markdown report
    md_path = f'review_reports/review_report_{timestamp}.md'
    generate_markdown_report(results, md_path)
    print(f"Markdown report generated: {md_path}")
    
    # Create summary file
    summary_path = 'REVIEW_SUMMARY.md'
    generate_markdown_report(results, summary_path)
    print(f"Review summary saved to: {summary_path}")


if __name__ == '__main__':
    main()
```

### Step 6: Complete Execution Workflow

**Create complete execution script:**

```bash
echo "=== DJANGO DD CODE REVIEW & SECURITY SCANNER ==="
echo "Starting comprehensive code review and security scan..."

# Step 1: Generate checklists
echo "1. Generating review checklists..."
python review_checklists/quality_checklist.py
python review_checklists/security_checklist.py

# Step 2: Run code review
echo "2. Running automated code review..."
python code_review_engine.py

# Step 3: Generate reports
echo "3. Generating review reports..."
python generate_review_reports.py

# Step 4: Display summary
echo "4. Review completed!"
echo ""
echo "Generated files:"
echo "- Quality checklist: review_checklists/quality_checklist.json"
echo "- Security checklist: review_checklists/security_checklist.json"
echo "- Review reports: review_reports/"
echo "- Summary: REVIEW_SUMMARY.md"

echo ""
echo "✅ Code review and security scan completed successfully!"
```

## Summary

This comprehensive prompt creates a complete code review and security scanning system that:

1. **Generates Quality Checklists**: Based on Django best practices and DD requirements
2. **Creates Security Checklists**: Following OWASP Top 10 and Django security guidelines  
3. **Performs Automated Reviews**: Uses static analysis and pattern matching
4. **Integrates External Tools**: Bandit, Safety, Flake8 for comprehensive analysis
5. **Generates Multiple Reports**: JSON, HTML, and Markdown formats
6. **Provides Actionable Recommendations**: Specific guidance for improvements

The system evaluates code against established standards and provides scored results with detailed findings and recommendations for improvement.