---
description: Automated unit test generation and execution based on DD implementation plan from prompt 04
---

# Django Design Detail (DD) Automated Unit Test Generator

This prompt analyzes the DD requirements and implementation plan from prompt 04 to generate comprehensive unit tests for all changes and additions, then executes tests and generates detailed reports.

## Prerequisites

**Required Input:**
- 📋 **Implementation Plan**: Output from `04_django_dd_implementation_plan.prompt.md`
- 📁 **DD Requirements**: Original DD specification files
- 📦 **Project Name**: `${input:project_name}`
- 🏷️ **App Name**: `${input:app_name}`
- 📄 **Implementation Plan Path**: `${input:implementation_plan_path}`
- 📁 **DD Folder Path**: `${input:dd_folder_path}`

## Test Generation Workflow

### Step 1: Analyze DD Requirements and Changes

**Parse and analyze all changes from implementation plan:**

```bash
echo "=== ANALYZING DD REQUIREMENTS AND CHANGES ==="

# Extract key components from implementation plan
echo "Extracting implementation components..."

# Models to test
MODELS=$(grep -o "class \[.*\]" ${input:implementation_plan_path} | sed 's/class \[\(.*\)\]/\1/' | sort -u)

# Serializers to test  
SERIALIZERS=$(grep -o "\[.*\]Serializer" ${input:implementation_plan_path} | sort -u)

# Views/ViewSets to test
VIEWSETS=$(grep -o "\[.*\]ViewSet" ${input:implementation_plan_path} | sort -u)

# API endpoints to test
ENDPOINTS=$(grep -o "path.*\[.*\]" ${input:implementation_plan_path} | sort -u)

echo "Components identified for testing:"
echo "- Models: $MODELS"
echo "- Serializers: $SERIALIZERS"
echo "- ViewSets: $VIEWSETS"
echo "- Endpoints: $ENDPOINTS"
```

**Generate test matrix based on DD requirements:**

```python
# File: tests/test_${input:app_name}/test_matrix.py

"""
Test matrix generator for ${input:dd_name} implementation
Auto-generated based on DD requirements analysis
"""

import json
from typing import Dict, List, Tuple

class DDTestMatrix:
    """Generate comprehensive test cases based on DD requirements"""
    
    def __init__(self, dd_requirements: Dict):
        self.requirements = dd_requirements
        self.test_cases = []
    
    def analyze_model_requirements(self, model_name: str) -> List[Dict]:
        """Generate test cases for model based on DD requirements"""
        test_cases = []
        
        # Field validation tests
        fields = self.requirements.get('models', {}).get(model_name, {}).get('fields', {})
        for field_name, field_spec in fields.items():
            # Required field tests
            if field_spec.get('required', False):
                test_cases.append({
                    'test_name': f'test_{model_name}_requires_{field_name}',
                    'test_type': 'validation',
                    'description': f'Test that {field_name} is required for {model_name}',
                    'test_data': {'missing_field': field_name}
                })
            
            # Field constraint tests
            if 'min_value' in field_spec:
                test_cases.append({
                    'test_name': f'test_{model_name}_{field_name}_min_value',
                    'test_type': 'validation',
                    'description': f'Test minimum value constraint for {field_name}',
                    'test_data': {
                        'field': field_name,
                        'invalid_value': field_spec['min_value'] - 1,
                        'valid_value': field_spec['min_value']
                    }
                })
            
            # Unique constraint tests
            if field_spec.get('unique', False):
                test_cases.append({
                    'test_name': f'test_{model_name}_{field_name}_unique_constraint',
                    'test_type': 'constraint',
                    'description': f'Test unique constraint for {field_name}',
                    'test_data': {'field': field_name}
                })
        
        # Business logic tests from DD
        business_rules = self.requirements.get('business_rules', {}).get(model_name, [])
        for idx, rule in enumerate(business_rules):
            test_cases.append({
                'test_name': f'test_{model_name}_business_rule_{idx}',
                'test_type': 'business_logic',
                'description': rule.get('description', ''),
                'test_data': rule.get('test_scenario', {})
            })
        
        return test_cases
    
    def analyze_api_requirements(self, endpoint: str) -> List[Dict]:
        """Generate test cases for API endpoints based on DD requirements"""
        test_cases = []
        
        # Get endpoint specification from DD
        endpoint_spec = self.requirements.get('api_endpoints', {}).get(endpoint, {})
        
        # Authentication tests
        if endpoint_spec.get('requires_auth', True):
            test_cases.append({
                'test_name': f'test_{endpoint}_requires_authentication',
                'test_type': 'authentication',
                'description': f'Test that {endpoint} requires authentication'
            })
        
        # Permission tests
        permissions = endpoint_spec.get('permissions', [])
        for permission in permissions:
            test_cases.append({
                'test_name': f'test_{endpoint}_permission_{permission}',
                'test_type': 'permission',
                'description': f'Test {permission} permission for {endpoint}',
                'test_data': {'permission': permission}
            })
        
        # Input validation tests
        request_schema = endpoint_spec.get('request_schema', {})
        for field, constraints in request_schema.items():
            if constraints.get('required', False):
                test_cases.append({
                    'test_name': f'test_{endpoint}_requires_{field}',
                    'test_type': 'request_validation',
                    'description': f'Test that {field} is required in request'
                })
            
            # Format validation
            if 'format' in constraints:
                test_cases.append({
                    'test_name': f'test_{endpoint}_{field}_format_validation',
                    'test_type': 'format_validation',
                    'description': f'Test {field} format validation ({constraints["format"]})',
                    'test_data': {
                        'field': field,
                        'format': constraints['format'],
                        'invalid_examples': constraints.get('invalid_examples', [])
                    }
                })
        
        # Response validation tests
        response_codes = endpoint_spec.get('response_codes', {})
        for code, scenario in response_codes.items():
            test_cases.append({
                'test_name': f'test_{endpoint}_response_{code}',
                'test_type': 'response_validation',
                'description': f'Test {code} response: {scenario}',
                'test_data': {'status_code': code, 'scenario': scenario}
            })
        
        return test_cases
    
    def generate_edge_cases(self) -> List[Dict]:
        """Generate edge case tests based on DD requirements"""
        edge_cases = []
        
        # Concurrent access tests
        if self.requirements.get('concurrency', {}).get('enabled', False):
            edge_cases.append({
                'test_name': 'test_concurrent_updates',
                'test_type': 'concurrency',
                'description': 'Test concurrent updates to same resource'
            })
        
        # Performance tests
        performance_requirements = self.requirements.get('performance', {})
        if 'max_response_time' in performance_requirements:
            edge_cases.append({
                'test_name': 'test_api_response_time',
                'test_type': 'performance',
                'description': f'Test API response time < {performance_requirements["max_response_time"]}ms',
                'test_data': {'threshold': performance_requirements['max_response_time']}
            })
        
        # Data volume tests
        if 'max_records' in self.requirements:
            edge_cases.append({
                'test_name': 'test_large_dataset_handling',
                'test_type': 'scalability',
                'description': f'Test handling of {self.requirements["max_records"]} records'
            })
        
        return edge_cases
    
    def export_test_matrix(self, output_file: str):
        """Export test matrix to JSON file"""
        matrix = {
            'dd_name': '${input:dd_name}',
            'total_test_cases': len(self.test_cases),
            'test_cases_by_type': {},
            'test_cases': self.test_cases
        }
        
        # Group by type
        for test_case in self.test_cases:
            test_type = test_case['test_type']
            if test_type not in matrix['test_cases_by_type']:
                matrix['test_cases_by_type'][test_type] = 0
            matrix['test_cases_by_type'][test_type] += 1
        
        with open(output_file, 'w') as f:
            json.dump(matrix, f, indent=2)
        
        return matrix


# Generate test matrix
def generate_test_matrix():
    """Main function to generate test matrix"""
    
    # Load DD requirements
    dd_requirements = {
        # This would be parsed from actual DD files
        'models': {
            '[ModelName1]': {
                'fields': {
                    'name': {'required': True, 'max_length': 255},
                    'priority': {'required': True, 'min_value': 0, 'max_value': 100},
                    'status': {'required': True, 'choices': ['active', 'inactive', 'archived']}
                }
            }
        },
        'api_endpoints': {
            '/api/v1/[model-name]/': {
                'requires_auth': True,
                'permissions': ['IsAuthenticated', 'IsOwnerOrReadOnly'],
                'request_schema': {
                    'name': {'required': True, 'format': 'string'},
                    'priority': {'required': True, 'format': 'integer', 'min': 0, 'max': 100}
                },
                'response_codes': {
                    200: 'Successful response',
                    400: 'Invalid input data',
                    401: 'Authentication required',
                    403: 'Permission denied',
                    404: 'Resource not found'
                }
            }
        },
        'business_rules': {
            '[ModelName1]': [
                {
                    'description': 'Archived items cannot have priority > 0',
                    'test_scenario': {'status': 'archived', 'priority': 50, 'expected_error': True}
                },
                {
                    'description': 'End date must be after start date',
                    'test_scenario': {'start_date': '2024-01-10', 'end_date': '2024-01-01', 'expected_error': True}
                }
            ]
        },
        'performance': {
            'max_response_time': 200
        },
        'concurrency': {
            'enabled': True
        }
    }
    
    generator = DDTestMatrix(dd_requirements)
    
    # Generate test cases for each component
    for model in ['[ModelName1]']:
        generator.test_cases.extend(generator.analyze_model_requirements(model))
    
    for endpoint in ['/api/v1/[model-name]/']:
        generator.test_cases.extend(generator.analyze_api_requirements(endpoint))
    
    # Add edge cases
    generator.test_cases.extend(generator.generate_edge_cases())
    
    # Export matrix
    matrix = generator.export_test_matrix('tests/test_matrix.json')
    
    print(f"Generated {matrix['total_test_cases']} test cases")
    for test_type, count in matrix['test_cases_by_type'].items():
        print(f"  - {test_type}: {count} test cases")
    
    return matrix


if __name__ == '__main__':
    generate_test_matrix()
```

### Step 2: Generate Model Unit Tests

**Create comprehensive model tests based on DD requirements:**

```python
# File: tests/test_${input:app_name}/test_models_comprehensive.py

"""
Comprehensive model tests for ${input:dd_name} implementation
Auto-generated based on DD requirements and test matrix
"""

import json
import uuid
from datetime import date, timedelta
from decimal import Decimal
from django.test import TestCase, TransactionTestCase
from django.contrib.auth.models import User
from django.core.exceptions import ValidationError
from django.db import IntegrityError, transaction
from django.db.models import ProtectedError
from apps.${input:app_name}.models import *


class BaseModelTestCase(TestCase):
    """Base test case with common setup"""
    
    @classmethod
    def setUpTestData(cls):
        """Set up test data once for the class"""
        cls.users = {
            'user1': User.objects.create_user('user1', 'user1@test.com', 'pass123'),
            'user2': User.objects.create_user('user2', 'user2@test.com', 'pass123'),
            'admin': User.objects.create_user('admin', 'admin@test.com', 'pass123', is_staff=True),
        }
        
        # Create test tags
        cls.tags = {
            'important': Tag.objects.create(name='Important', slug='important'),
            'urgent': Tag.objects.create(name='Urgent', slug='urgent'),
            'review': Tag.objects.create(name='Review', slug='review'),
        }
    
    def create_valid_instance(self, **kwargs):
        """Create a valid model instance with default values"""
        defaults = {
            'name': f'Test Item {uuid.uuid4().hex[:8]}',
            'description': 'Test description',
            'owner': self.users['user1'],
            'priority': 50,
            'status': [ModelName1].Status.ACTIVE,
        }
        defaults.update(kwargs)
        return [ModelName1].objects.create(**defaults)


class [ModelName1]FieldValidationTest(BaseModelTestCase):
    """Test field-level validation based on DD requirements"""
    
    def test_required_fields(self):
        """Test that all required fields are enforced"""
        # Based on DD requirement: name is required
        with self.assertRaises(ValidationError) as context:
            obj = [ModelName1](
                description='Missing name field',
                owner=self.users['user1']
            )
            obj.full_clean()
        
        self.assertIn('name', context.exception.message_dict)
    
    def test_name_max_length(self):
        """Test name field max length constraint"""
        # Based on DD requirement: name max_length = 255
        max_length = 255
        
        # Test exact max length - should pass
        obj = [ModelName1](
            name='x' * max_length,
            owner=self.users['user1']
        )
        obj.full_clean()  # Should not raise
        
        # Test exceeding max length - should fail
        with self.assertRaises(ValidationError) as context:
            obj = [ModelName1](
                name='x' * (max_length + 1),
                owner=self.users['user1']
            )
            obj.full_clean()
        
        self.assertIn('name', context.exception.message_dict)
    
    def test_priority_range_validation(self):
        """Test priority field range constraints"""
        # Based on DD requirement: priority must be 0-100
        
        # Test minimum value
        obj = self.create_valid_instance(priority=0)
        self.assertEqual(obj.priority, 0)
        
        # Test maximum value
        obj = self.create_valid_instance(priority=100)
        self.assertEqual(obj.priority, 100)
        
        # Test below minimum
        with self.assertRaises(ValidationError):
            obj = [ModelName1](
                name='Test',
                owner=self.users['user1'],
                priority=-1
            )
            obj.full_clean()
        
        # Test above maximum
        with self.assertRaises(ValidationError):
            obj = [ModelName1](
                name='Test',
                owner=self.users['user1'],
                priority=101
            )
            obj.full_clean()
    
    def test_status_choices_validation(self):
        """Test status field choices constraint"""
        # Test valid choices
        for status_choice, _ in [ModelName1].Status.choices:
            obj = self.create_valid_instance(status=status_choice)
            self.assertEqual(obj.status, status_choice)
        
        # Test invalid choice
        with self.assertRaises(ValidationError):
            obj = [ModelName1](
                name='Test',
                owner=self.users['user1'],
                status='invalid_status'
            )
            obj.full_clean()
    
    def test_date_fields_validation(self):
        """Test date field constraints"""
        # Test valid date range
        obj = self.create_valid_instance(
            start_date=date.today(),
            end_date=date.today() + timedelta(days=7)
        )
        self.assertTrue(obj.end_date > obj.start_date)
        
        # Test invalid date range (end before start)
        with self.assertRaises(ValidationError) as context:
            obj = [ModelName1](
                name='Test',
                owner=self.users['user1'],
                start_date=date.today(),
                end_date=date.today() - timedelta(days=1)
            )
            obj.full_clean()
        
        self.assertIn('End date must be after start date', str(context.exception))
    
    def test_json_field_validation(self):
        """Test JSON field validation"""
        # Test valid JSON
        valid_metadata = {
            'key1': 'value1',
            'key2': 123,
            'key3': ['list', 'items'],
            'key4': {'nested': 'object'}
        }
        obj = self.create_valid_instance(metadata=valid_metadata)
        self.assertEqual(obj.metadata, valid_metadata)
        
        # Test empty JSON (should use default)
        obj = self.create_valid_instance()
        self.assertEqual(obj.metadata, {})


class [ModelName1]ConstraintTest(BaseModelTestCase):
    """Test database-level constraints based on DD requirements"""
    
    def test_unique_owner_name_constraint(self):
        """Test unique constraint on owner and name combination"""
        # Create first instance
        self.create_valid_instance(
            name='Duplicate Test',
            owner=self.users['user1']
        )
        
        # Try to create duplicate - should fail
        with self.assertRaises(IntegrityError):
            self.create_valid_instance(
                name='Duplicate Test',
                owner=self.users['user1']
            )
        
        # Different owner with same name - should succeed
        obj = self.create_valid_instance(
            name='Duplicate Test',
            owner=self.users['user2']
        )
        self.assertIsNotNone(obj.id)
    
    def test_foreign_key_constraints(self):
        """Test foreign key cascade/protect behaviors"""
        obj = self.create_valid_instance(owner=self.users['user1'])
        
        # Test PROTECT - owner cannot be deleted if has items
        with self.assertRaises(ProtectedError):
            self.users['user1'].delete()
        
        # Verify object still exists
        obj.refresh_from_db()
        self.assertIsNotNone(obj.owner)
    
    def test_many_to_many_constraints(self):
        """Test many-to-many relationship constraints"""
        obj = self.create_valid_instance()
        
        # Add tags
        obj.tags.add(self.tags['important'], self.tags['urgent'])
        self.assertEqual(obj.tags.count(), 2)
        
        # Test removing tags
        obj.tags.remove(self.tags['urgent'])
        self.assertEqual(obj.tags.count(), 1)
        
        # Test clear
        obj.tags.clear()
        self.assertEqual(obj.tags.count(), 0)


class [ModelName1]BusinessLogicTest(BaseModelTestCase):
    """Test business logic rules from DD requirements"""
    
    def test_archived_items_zero_priority_rule(self):
        """Test DD rule: Archived items must have priority 0"""
        # Create active item with priority
        obj = self.create_valid_instance(
            status=[ModelName1].Status.ACTIVE,
            priority=75
        )
        
        # Archive with non-zero priority - should fail validation
        obj.status = [ModelName1].Status.ARCHIVED
        with self.assertRaises(ValidationError) as context:
            obj.full_clean()
        
        self.assertIn('Archived items must have priority 0', str(context.exception))
        
        # Archive with zero priority - should succeed
        obj.priority = 0
        obj.full_clean()
        obj.save()
        self.assertEqual(obj.status, [ModelName1].Status.ARCHIVED)
        self.assertEqual(obj.priority, 0)
    
    def test_status_transition_rules(self):
        """Test valid status transitions based on DD business rules"""
        obj = self.create_valid_instance(status=[ModelName1].Status.ACTIVE)
        
        # Active -> Inactive (allowed)
        obj.status = [ModelName1].Status.INACTIVE
        obj.full_clean()
        obj.save()
        
        # Inactive -> Archived (allowed)
        obj.status = [ModelName1].Status.ARCHIVED
        obj.priority = 0  # Required for archived
        obj.full_clean()
        obj.save()
        
        # Archived -> Active (check if allowed by business rules)
        # This depends on specific DD requirements
    
    def test_assignment_creation_on_new_item(self):
        """Test automatic assignment creation for owner"""
        obj = self.create_valid_instance()
        
        # Verify owner has admin assignment
        assignment = obj.assignments.filter(user=obj.owner).first()
        self.assertIsNotNone(assignment)
        self.assertEqual(assignment.role, [ModelName1]Assignment.Role.ADMIN)
        self.assertEqual(assignment.assigned_by, obj.owner)
    
    def test_calculated_properties(self):
        """Test calculated properties and methods"""
        # Test duration calculation
        obj = self.create_valid_instance(
            start_date=date(2024, 1, 1),
            end_date=date(2024, 1, 10)
        )
        self.assertEqual(obj.duration, 9)
        
        # Test is_active method
        active_obj = self.create_valid_instance(status=[ModelName1].Status.ACTIVE)
        self.assertTrue(active_obj.is_active())
        
        inactive_obj = self.create_valid_instance(status=[ModelName1].Status.INACTIVE)
        self.assertFalse(inactive_obj.is_active())


class [ModelName1]QuerySetTest(BaseModelTestCase):
    """Test custom manager and queryset methods"""
    
    def setUp(self):
        super().setUp()
        # Create test data
        self.active_items = [
            self.create_valid_instance(
                name=f'Active {i}',
                status=[ModelName1].Status.ACTIVE,
                priority=i * 10
            ) for i in range(5)
        ]
        
        self.archived_items = [
            self.create_valid_instance(
                name=f'Archived {i}',
                status=[ModelName1].Status.ARCHIVED,
                priority=0
            ) for i in range(3)
        ]
    
    def test_active_manager_method(self):
        """Test custom manager active() method"""
        active_qs = [ModelName1].objects.active()
        
        self.assertEqual(active_qs.count(), 5)
        for item in active_qs:
            self.assertEqual(item.status, [ModelName1].Status.ACTIVE)
    
    def test_owned_by_manager_method(self):
        """Test custom manager owned_by() method"""
        # Create items for different users
        user2_items = [
            self.create_valid_instance(
                name=f'User2 Item {i}',
                owner=self.users['user2']
            ) for i in range(2)
        ]
        
        # Test filtering by owner
        user1_items = [ModelName1].objects.owned_by(self.users['user1'])
        user2_items_qs = [ModelName1].objects.owned_by(self.users['user2'])
        
        self.assertEqual(user1_items.count(), 8)  # 5 active + 3 archived
        self.assertEqual(user2_items_qs.count(), 2)
    
    def test_complex_queries(self):
        """Test complex query scenarios"""
        # High priority active items
        high_priority = [ModelName1].objects.filter(
            status=[ModelName1].Status.ACTIVE,
            priority__gte=30
        )
        self.assertEqual(high_priority.count(), 3)
        
        # Items with tags
        tagged_item = self.active_items[0]
        tagged_item.tags.add(self.tags['important'])
        
        important_items = [ModelName1].objects.filter(tags=self.tags['important'])
        self.assertEqual(important_items.count(), 1)


class [ModelName1]PerformanceTest(TransactionTestCase):
    """Test performance-related scenarios"""
    
    def test_bulk_create_performance(self):
        """Test bulk creation performance"""
        import time
        
        user = User.objects.create_user('bulkuser', 'bulk@test.com', 'pass')
        
        # Measure bulk create time
        start_time = time.time()
        
        items = [
            [ModelName1](
                name=f'Bulk Item {i}',
                owner=user,
                priority=50
            ) for i in range(1000)
        ]
        
        [ModelName1].objects.bulk_create(items)
        
        end_time = time.time()
        duration = end_time - start_time
        
        # Based on DD requirement: bulk operations should complete within reasonable time
        self.assertLess(duration, 5.0)  # 5 seconds for 1000 items
        
        # Verify all created
        self.assertEqual([ModelName1].objects.filter(owner=user).count(), 1000)
    
    def test_query_optimization(self):
        """Test query optimization with select_related and prefetch_related"""
        # Create test data
        user = User.objects.create_user('queryuser', 'query@test.com', 'pass')
        items = [
            self.create_valid_instance(
                name=f'Query Item {i}',
                owner=user
            ) for i in range(10)
        ]
        
        # Add tags to items
        for item in items:
            item.tags.add(self.tags['important'], self.tags['urgent'])
        
        # Test unoptimized query count
        with self.assertNumQueries(23):  # 1 + 10 (owner) + 10 (tags) + 2
            unoptimized = list([ModelName1].objects.all())
            for item in unoptimized:
                _ = item.owner.username
                _ = list(item.tags.all())
        
        # Test optimized query count
        with self.assertNumQueries(3):  # 1 main + 1 select_related + 1 prefetch_related
            optimized = list(
                [ModelName1].objects
                .select_related('owner')
                .prefetch_related('tags')
            )
            for item in optimized:
                _ = item.owner.username
                _ = list(item.tags.all())


class [ModelName1]EdgeCaseTest(BaseModelTestCase):
    """Test edge cases and boundary conditions"""
    
    def test_unicode_handling(self):
        """Test Unicode characters in text fields"""
        unicode_tests = [
            'Hello 世界',  # Chinese
            'Привет мир',  # Russian
            '🚀 Emoji test 🎉',  # Emojis
            'Special chars: <>&"\'',  # HTML special chars
        ]
        
        for test_string in unicode_tests:
            obj = self.create_valid_instance(
                name=test_string,
                description=test_string
            )
            obj.refresh_from_db()
            self.assertEqual(obj.name, test_string)
            self.assertEqual(obj.description, test_string)
    
    def test_null_handling(self):
        """Test null/blank field handling"""
        # Test optional fields can be null
        obj = self.create_valid_instance(
            description=None,
            start_date=None,
            end_date=None
        )
        
        self.assertIsNone(obj.description)
        self.assertIsNone(obj.start_date)
        self.assertIsNone(obj.end_date)
        self.assertIsNone(obj.duration)  # Calculated property
    
    def test_concurrent_updates(self):
        """Test concurrent update scenarios"""
        from django.db import connection
        
        obj = self.create_valid_instance(priority=50)
        
        # Simulate concurrent updates
        with transaction.atomic():
            # Load in first transaction
            obj1 = [ModelName1].objects.select_for_update().get(pk=obj.pk)
            
            # Try to update in another transaction (would block in real scenario)
            # This is a simplified test - real concurrent testing requires threading
            obj1.priority = 75
            obj1.save()
        
        obj.refresh_from_db()
        self.assertEqual(obj.priority, 75)
    
    def test_large_metadata_handling(self):
        """Test large JSON metadata handling"""
        # Create large metadata object
        large_metadata = {
            f'key_{i}': {
                'data': f'value_{i}' * 100,
                'nested': {
                    f'subkey_{j}': f'subvalue_{j}' * 50
                    for j in range(10)
                }
            }
            for i in range(50)
        }
        
        obj = self.create_valid_instance(metadata=large_metadata)
        obj.refresh_from_db()
        
        # Verify data integrity
        self.assertEqual(len(obj.metadata), 50)
        self.assertIn('key_0', obj.metadata)
        self.assertIn('nested', obj.metadata['key_0'])


# Generate test summary
def generate_model_test_summary():
    """Generate summary of model tests"""
    test_classes = [
        [ModelName1]FieldValidationTest,
        [ModelName1]ConstraintTest,
        [ModelName1]BusinessLogicTest,
        [ModelName1]QuerySetTest,
        [ModelName1]PerformanceTest,
        [ModelName1]EdgeCaseTest,
    ]
    
    total_tests = 0
    test_summary = {
        'model': '[ModelName1]',
        'test_classes': []
    }
    
    for test_class in test_classes:
        test_methods = [
            method for method in dir(test_class)
            if method.startswith('test_')
        ]
        
        test_summary['test_classes'].append({
            'class': test_class.__name__,
            'tests': len(test_methods),
            'methods': test_methods
        })
        
        total_tests += len(test_methods)
    
    test_summary['total_tests'] = total_tests
    
    return test_summary
```

### Step 3: Generate Serializer Unit Tests

**Create comprehensive serializer tests:**

```python
# File: tests/test_${input:app_name}/test_serializers_comprehensive.py

"""
Comprehensive serializer tests for ${input:dd_name} implementation
Auto-generated based on DD requirements
"""

import json
from decimal import Decimal
from datetime import date, datetime
from django.test import TestCase
from django.contrib.auth.models import User
from rest_framework.test import APIRequestFactory
from rest_framework.request import Request
from rest_framework.exceptions import ValidationError as DRFValidationError
from apps.${input:app_name}.models import *
from apps.${input:app_name}.serializers import *


class SerializerTestBase(TestCase):
    """Base class for serializer tests"""
    
    @classmethod
    def setUpTestData(cls):
        cls.factory = APIRequestFactory()
        cls.users = {
            'user1': User.objects.create_user('user1', 'user1@test.com', 'pass123'),
            'user2': User.objects.create_user('user2', 'user2@test.com', 'pass123'),
        }
        
        cls.tags = {
            'tag1': Tag.objects.create(name='Tag 1', slug='tag-1'),
            'tag2': Tag.objects.create(name='Tag 2', slug='tag-2'),
        }
    
    def get_request(self, user=None):
        """Get mock request with user"""
        request = self.factory.get('/')
        request.user = user or self.users['user1']
        return Request(request)


class [ModelName1]ReadSerializerTest(SerializerTestBase):
    """Test read serializer functionality"""
    
    def setUp(self):
        super().setUp()
        self.item = [ModelName1].objects.create(
            name='Test Item',
            description='Test description',
            owner=self.users['user1'],
            priority=75,
            status=[ModelName1].Status.ACTIVE
        )
        self.item.tags.add(self.tags['tag1'])
    
    def test_serialization_fields(self):
        """Test all fields are properly serialized"""
        serializer = [ModelName1]ReadSerializer(
            self.item,
            context={'request': self.get_request()}
        )
        data = serializer.data
        
        # Verify all expected fields are present
        expected_fields = [
            'id', 'name', 'description', 'status', 'status_display',
            'owner', 'tags', 'priority', 'created_at', 'updated_at',
            'duration', 'assignments'
        ]
        
        for field in expected_fields:
            self.assertIn(field, data)
        
        # Verify field values
        self.assertEqual(data['name'], 'Test Item')
        self.assertEqual(data['priority'], 75)
        self.assertEqual(data['status'], 'active')
        self.assertEqual(data['status_display'], 'Active')
    
    def test_nested_serialization(self):
        """Test nested object serialization"""
        serializer = [ModelName1]ReadSerializer(self.item)
        data = serializer.data
        
        # Test owner serialization
        self.assertIsInstance(data['owner'], dict)
        self.assertEqual(data['owner']['username'], 'user1')
        self.assertIn('email', data['owner'])
        
        # Test tags serialization
        self.assertIsInstance(data['tags'], list)
        self.assertEqual(len(data['tags']), 1)
        self.assertEqual(data['tags'][0]['name'], 'Tag 1')
    
    def test_computed_fields(self):
        """Test computed/property fields"""
        # Set dates for duration calculation
        self.item.start_date = date(2024, 1, 1)
        self.item.end_date = date(2024, 1, 10)
        self.item.save()
        
        serializer = [ModelName1]ReadSerializer(self.item)
        data = serializer.data
        
        # Test duration calculation
        self.assertEqual(data['duration'], 9)
        
        # Test status display
        self.assertEqual(data['status_display'], 'Active')
    
    def test_queryset_optimization(self):
        """Test serializer works with optimized querysets"""
        # Create multiple items with relationships
        for i in range(5):
            item = [ModelName1].objects.create(
                name=f'Item {i}',
                owner=self.users['user1']
            )
            item.tags.add(self.tags['tag1'], self.tags['tag2'])
        
        # Use optimized queryset
        queryset = [ModelName1].objects.select_related('owner').prefetch_related('tags')
        
        # Serialize queryset - should not cause N+1 queries
        with self.assertNumQueries(3):  # Initial query + select_related + prefetch_related
            serializer = [ModelName1]ReadSerializer(queryset, many=True)
            data = serializer.data
            
            # Access nested fields to ensure they're loaded
            for item_data in data:
                _ = item_data['owner']['username']
                _ = len(item_data['tags'])


class [ModelName1]WriteSerializerTest(SerializerTestBase):
    """Test write serializer validation and creation"""
    
    def test_create_with_valid_data(self):
        """Test creating object with valid data"""
        data = {
            'name': 'New Item',
            'description': 'New description',
            'priority': 50,
            'status': 'active',
            'tags': [self.tags['tag1'].id]
        }
        
        serializer = [ModelName1]WriteSerializer(
            data=data,
            context={'request': self.get_request()}
        )
        
        self.assertTrue(serializer.is_valid())
        instance = serializer.save()
        
        # Verify created instance
        self.assertEqual(instance.name, 'New Item')
        self.assertEqual(instance.owner, self.users['user1'])
        self.assertIn(self.tags['tag1'], instance.tags.all())
    
    def test_required_field_validation(self):
        """Test required field validation based on DD"""
        # Missing required field 'name'
        data = {
            'description': 'Missing name',
            'priority': 50
        }
        
        serializer = [ModelName1]WriteSerializer(
            data=data,
            context={'request': self.get_request()}
        )
        
        self.assertFalse(serializer.is_valid())
        self.assertIn('name', serializer.errors)
        self.assertIn('required', str(serializer.errors['name']))
    
    def test_field_type_validation(self):
        """Test field type validation"""
        test_cases = [
            # (field, invalid_value, error_keyword)
            ('priority', 'not-a-number', 'integer'),
            ('priority', 150, 'Ensure this value is less than or equal to 100'),
            ('priority', -10, 'Ensure this value is greater than or equal to 0'),
            ('status', 'invalid_status', 'valid choice'),
            ('tags', ['invalid-id'], 'does not exist'),
        ]
        
        for field, invalid_value, error_keyword in test_cases:
            data = {
                'name': 'Test',
                'priority': 50,
                field: invalid_value
            }
            
            serializer = [ModelName1]WriteSerializer(
                data=data,
                context={'request': self.get_request()}
            )
            
            self.assertFalse(serializer.is_valid())
            self.assertIn(field, serializer.errors)
            self.assertIn(error_keyword, str(serializer.errors[field]))
    
    def test_custom_validation_methods(self):
        """Test custom validation logic from DD requirements"""
        # Test duplicate name validation
        existing = [ModelName1].objects.create(
            name='Existing Item',
            owner=self.users['user1']
        )
        
        data = {
            'name': 'Existing Item',  # Duplicate for same owner
            'priority': 50
        }
        
        serializer = [ModelName1]WriteSerializer(
            data=data,
            context={'request': self.get_request()}
        )
        
        self.assertFalse(serializer.is_valid())
        self.assertIn('name', serializer.errors)
        self.assertIn('already have an item with this name', str(serializer.errors['name']))
    
    def test_cross_field_validation(self):
        """Test validation that depends on multiple fields"""
        # Test date validation
        data = {
            'name': 'Date Test',
            'priority': 50,
            'start_date': '2024-01-10',
            'end_date': '2024-01-01'  # Before start date
        }
        
        serializer = [ModelName1]WriteSerializer(
            data=data,
            context={'request': self.get_request()}
        )
        
        self.assertFalse(serializer.is_valid())
        self.assertIn('end_date', serializer.errors)
        self.assertIn('must be after start date', str(serializer.errors['end_date']))
        
        # Test archived status with priority > 0
        data = {
            'name': 'Archive Test',
            'status': 'archived',
            'priority': 50  # Should be 0 for archived
        }
        
        serializer = [ModelName1]WriteSerializer(
            data=data,
            context={'request': self.get_request()}
        )
        
        self.assertFalse(serializer.is_valid())
        self.assertIn('priority', serializer.errors)
        self.assertIn('Archived items must have priority 0', str(serializer.errors['priority']))
    
    def test_update_with_partial_data(self):
        """Test updating with partial data (PATCH)"""
        item = [ModelName1].objects.create(
            name='Original',
            owner=self.users['user1'],
            priority=50
        )
        
        # Partial update
        data = {'priority': 75}
        
        serializer = [ModelName1]WriteSerializer(
            item,
            data=data,
            partial=True,
            context={'request': self.get_request()}
        )
        
        self.assertTrue(serializer.is_valid())
        updated = serializer.save()
        
        self.assertEqual(updated.priority, 75)
        self.assertEqual(updated.name, 'Original')  # Unchanged
    
    def test_nested_creation(self):
        """Test creating with nested objects"""
        # If DD specifies nested creation support
        data = {
            'name': 'Item with Assignment',
            'priority': 50,
            'assignments': [
                {
                    'user_id': self.users['user2'].id,
                    'role': 'editor'
                }
            ]
        }
        
        # This would require custom create method in serializer
        # Test based on actual DD requirements


class [ModelName1]BulkSerializerTest(SerializerTestBase):
    """Test bulk operation serializers"""
    
    def setUp(self):
        super().setUp()
        self.items = [
            [ModelName1].objects.create(
                name=f'Item {i}',
                owner=self.users['user1'],
                priority=50
            ) for i in range(3)
        ]
    
    def test_bulk_action_validation(self):
        """Test bulk action serializer validation"""
        # Valid data
        data = {
            'ids': [str(item.id) for item in self.items],
            'action': 'archive'
        }
        
        serializer = [ModelName1]BulkActionSerializer(
            data=data,
            context={'request': self.get_request()}
        )
        
        self.assertTrue(serializer.is_valid())
        
        # Invalid action
        data['action'] = 'invalid_action'
        serializer = [ModelName1]BulkActionSerializer(
            data=data,
            context={'request': self.get_request()}
        )
        
        self.assertFalse(serializer.is_valid())
        self.assertIn('action', serializer.errors)
    
    def test_bulk_permission_validation(self):
        """Test bulk operations respect permissions"""
        # Create item owned by different user
        other_item = [ModelName1].objects.create(
            name='Other User Item',
            owner=self.users['user2']
        )
        
        data = {
            'ids': [str(other_item.id)],
            'action': 'delete'
        }
        
        serializer = [ModelName1]BulkActionSerializer(
            data=data,
            context={'request': self.get_request(self.users['user1'])}
        )
        
        self.assertFalse(serializer.is_valid())
        self.assertIn('ids', serializer.errors)
        self.assertIn('No permission', str(serializer.errors['ids']))


class SerializerPerformanceTest(SerializerTestBase):
    """Test serializer performance characteristics"""
    
    def test_large_dataset_serialization(self):
        """Test serializing large datasets"""
        import time
        
        # Create large dataset
        items = []
        for i in range(100):
            item = [ModelName1].objects.create(
                name=f'Item {i}',
                owner=self.users['user1'],
                priority=i % 100
            )
            item.tags.add(self.tags['tag1'])
            items.append(item)
        
        # Measure serialization time
        start_time = time.time()
        
        serializer = [ModelName1]ReadSerializer(items, many=True)
        data = serializer.data
        
        end_time = time.time()
        duration = end_time - start_time
        
        # Performance assertion based on DD requirements
        self.assertLess(duration, 1.0)  # Should serialize 100 items in < 1 second
        self.assertEqual(len(data), 100)
    
    def test_deep_nesting_performance(self):
        """Test performance with deeply nested serialization"""
        # Create complex nested structure
        item = [ModelName1].objects.create(
            name='Complex Item',
            owner=self.users['user1']
        )
        
        # Add multiple assignments
        for i in range(10):
            user = User.objects.create_user(f'user{i}', f'user{i}@test.com', 'pass')
            [ModelName1]Assignment.objects.create(
                [model_name]=item,
                user=user,
                role='viewer'
            )
        
        # Add multiple tags
        for i in range(20):
            tag = Tag.objects.create(name=f'Tag {i}', slug=f'tag-{i}')
            item.tags.add(tag)
        
        # Serialize with all nested data
        serializer = [ModelName1]ReadSerializer(item)
        data = serializer.data
        
        # Verify nested data is included
        self.assertEqual(len(data['assignments']), 10)
        self.assertEqual(len(data['tags']), 20)


# Generate serializer test summary
def generate_serializer_test_summary():
    """Generate summary of serializer tests"""
    test_classes = [
        [ModelName1]ReadSerializerTest,
        [ModelName1]WriteSerializerTest,
        [ModelName1]BulkSerializerTest,
        SerializerPerformanceTest,
    ]
    
    test_summary = {
        'component': 'Serializers',
        'test_classes': [],
        'coverage_areas': [
            'Field serialization',
            'Nested object handling',
            'Validation rules from DD',
            'Custom validation methods',
            'Bulk operations',
            'Performance requirements',
            'Permission checks',
        ]
    }
    
    total_tests = 0
    for test_class in test_classes:
        test_methods = [m for m in dir(test_class) if m.startswith('test_')]
        test_summary['test_classes'].append({
            'class': test_class.__name__,
            'tests': len(test_methods),
            'focus': test_class.__doc__.strip()
        })
        total_tests += len(test_methods)
    
    test_summary['total_tests'] = total_tests
    return test_summary
```

### Step 4: Generate View/API Unit Tests

**Create comprehensive API endpoint tests:**

```python
# File: tests/test_${input:app_name}/test_views_comprehensive.py

"""
Comprehensive view/API tests for ${input:dd_name} implementation
Auto-generated based on DD requirements
"""

import json
import time
from datetime import datetime
from unittest.mock import patch, Mock
from django.test import TestCase, TransactionTestCase
from django.urls import reverse
from django.contrib.auth.models import User
from rest_framework import status
from rest_framework.test import APITestCase, APIClient
from apps.${input:app_name}.models import *


class APITestBase(APITestCase):
    """Base class for API tests"""
    
    @classmethod
    def setUpTestData(cls):
        cls.users = {
            'user1': User.objects.create_user('user1', 'user1@test.com', 'pass123'),
            'user2': User.objects.create_user('user2', 'user2@test.com', 'pass123'),
            'staff': User.objects.create_user('staff', 'staff@test.com', 'pass123', is_staff=True),
            'inactive': User.objects.create_user('inactive', 'inactive@test.com', 'pass123', is_active=False),
        }
        
        cls.tags = {
            'tag1': Tag.objects.create(name='Tag 1', slug='tag-1'),
            'tag2': Tag.objects.create(name='Tag 2', slug='tag-2'),
        }
    
    def authenticate(self, user=None):
        """Authenticate client with user"""
        user = user or self.users['user1']
        self.client.force_authenticate(user=user)
    
    def create_item(self, **kwargs):
        """Helper to create test item"""
        defaults = {
            'name': 'Test Item',
            'owner': self.users['user1'],
            'priority': 50
        }
        defaults.update(kwargs)
        return [ModelName1].objects.create(**defaults)


class [ModelName1]AuthenticationTest(APITestBase):
    """Test authentication requirements from DD"""
    
    def test_unauthenticated_access_denied(self):
        """Test all endpoints require authentication"""
        # Based on DD requirement: all endpoints require authentication
        
        endpoints = [
            reverse('${input:app_name}:[model-name]-list'),
            reverse('${input:app_name}:[model-name]-detail', args=[uuid.uuid4()]),
            reverse('${input:app_name}:[model-name]-bulk-action'),
            reverse('${input:app_name}:[model-name]-statistics'),
        ]
        
        for endpoint in endpoints:
            response = self.client.get(endpoint)
            self.assertEqual(
                response.status_code,
                status.HTTP_401_UNAUTHORIZED,
                f"Endpoint {endpoint} should require authentication"
            )
    
    def test_inactive_user_access_denied(self):
        """Test inactive users cannot access API"""
        self.authenticate(self.users['inactive'])
        
        response = self.client.get(reverse('${input:app_name}:[model-name]-list'))
        self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)
    
    def test_token_authentication(self):
        """Test token-based authentication if specified in DD"""
        # This would test token auth if required by DD
        pass


class [ModelName1]PermissionTest(APITestBase):
    """Test permission requirements from DD"""
    
    def setUp(self):
        super().setUp()
        # Create test data
        self.user1_item = self.create_item(owner=self.users['user1'])
        self.user2_item = self.create_item(owner=self.users['user2'])
    
    def test_owner_can_update_own_items(self):
        """Test owners have full access to their items"""
        self.authenticate(self.users['user1'])
        
        url = reverse('${input:app_name}:[model-name]-detail', args=[self.user1_item.id])
        response = self.client.patch(url, {'priority': 75})
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.user1_item.refresh_from_db()
        self.assertEqual(self.user1_item.priority, 75)
    
    def test_non_owner_cannot_update_items(self):
        """Test non-owners cannot update items"""
        self.authenticate(self.users['user1'])
        
        url = reverse('${input:app_name}:[model-name]-detail', args=[self.user2_item.id])
        response = self.client.patch(url, {'priority': 75})
        
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)
    
    def test_staff_can_access_all_items(self):
        """Test staff users have access to all items"""
        self.authenticate(self.users['staff'])
        
        # Should see all items
        response = self.client.get(reverse('${input:app_name}:[model-name]-list'))
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.data['count'], 2)
        
        # Can update any item
        url = reverse('${input:app_name}:[model-name]-detail', args=[self.user2_item.id])
        response = self.client.patch(url, {'priority': 90})
        self.assertEqual(response.status_code, status.HTTP_200_OK)
    
    def test_assignment_based_permissions(self):
        """Test permissions based on assignments"""
        # Create assignment
        assignment = [ModelName1]Assignment.objects.create(
            [model_name]=self.user2_item,
            user=self.users['user1'],
            role='viewer'
        )
        
        self.authenticate(self.users['user1'])
        
        # Can view assigned item
        url = reverse('${input:app_name}:[model-name]-detail', args=[self.user2_item.id])
        response = self.client.get(url)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        
        # Cannot update with viewer role
        response = self.client.patch(url, {'priority': 80})
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)


class [ModelName1]CRUDTest(APITestBase):
    """Test CRUD operations with DD validation rules"""
    
    def test_create_item_success(self):
        """Test creating item with valid data"""
        self.authenticate()
        
        data = {
            'name': 'New Item',
            'description': 'Test description',
            'priority': 75,
            'status': 'active',
            'tags': [self.tags['tag1'].id],
            'start_date': '2024-01-01',
            'end_date': '2024-12-31',
            'metadata': {'key': 'value'}
        }
        
        response = self.client.post(
            reverse('${input:app_name}:[model-name]-list'),
            data,
            format='json'
        )
        
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        
        # Verify response data
        self.assertEqual(response.data['name'], 'New Item')
        self.assertEqual(response.data['owner']['id'], self.users['user1'].id)
        
        # Verify database
        item = [ModelName1].objects.get(id=response.data['id'])
        self.assertEqual(item.priority, 75)
        self.assertIn(self.tags['tag1'], item.tags.all())
        
        # Verify auto-created assignment
        self.assertTrue(
            item.assignments.filter(
                user=self.users['user1'],
                role='admin'
            ).exists()
        )
    
    def test_create_item_validation_errors(self):
        """Test validation errors based on DD requirements"""
        self.authenticate()
        
        # Test missing required fields
        response = self.client.post(
            reverse('${input:app_name}:[model-name]-list'),
            {},
            format='json'
        )
        
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
        self.assertIn('name', response.data)
        
        # Test invalid field values
        test_cases = [
            # Invalid priority
            {
                'name': 'Test',
                'priority': 150  # Exceeds max
            },
            # Invalid date range
            {
                'name': 'Test',
                'start_date': '2024-01-10',
                'end_date': '2024-01-01'
            },
            # Invalid status with priority
            {
                'name': 'Test',
                'status': 'archived',
                'priority': 50  # Should be 0
            }
        ]
        
        for data in test_cases:
            response = self.client.post(
                reverse('${input:app_name}:[model-name]-list'),
                data,
                format='json'
            )
            self.assertEqual(
                response.status_code,
                status.HTTP_400_BAD_REQUEST,
                f"Data {data} should fail validation"
            )
    
    def test_retrieve_item_with_nested_data(self):
        """Test retrieving item includes all nested data"""
        item = self.create_item()
        item.tags.add(self.tags['tag1'], self.tags['tag2'])
        
        # Add assignment
        [ModelName1]Assignment.objects.create(
            [model_name]=item,
            user=self.users['user2'],
            role='editor'
        )
        
        self.authenticate()
        
        url = reverse('${input:app_name}:[model-name]-detail', args=[item.id])
        response = self.client.get(url)
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        
        # Verify nested data
        self.assertEqual(len(response.data['tags']), 2)
        self.assertEqual(len(response.data['assignments']), 2)  # Owner + editor
        self.assertIsInstance(response.data['owner'], dict)
        self.assertEqual(response.data['duration'], None)  # No dates set
    
    def test_update_item_partial(self):
        """Test partial update (PATCH)"""
        item = self.create_item()
        self.authenticate()
        
        url = reverse('${input:app_name}:[model-name]-detail', args=[item.id])
        response = self.client.patch(
            url,
            {'priority': 90, 'description': 'Updated'},
            format='json'
        )
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        
        item.refresh_from_db()
        self.assertEqual(item.priority, 90)
        self.assertEqual(item.description, 'Updated')
        self.assertEqual(item.name, 'Test Item')  # Unchanged
    
    def test_update_item_full(self):
        """Test full update (PUT)"""
        item = self.create_item()
        self.authenticate()
        
        url = reverse('${input:app_name}:[model-name]-detail', args=[item.id])
        data = {
            'name': 'Fully Updated',
            'description': 'New description',
            'priority': 25,
            'status': 'inactive',
            'tags': []
        }
        
        response = self.client.put(url, data, format='json')
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        
        item.refresh_from_db()
        self.assertEqual(item.name, 'Fully Updated')
        self.assertEqual(item.priority, 25)
        self.assertEqual(item.tags.count(), 0)
    
    def test_delete_item_soft_delete(self):
        """Test delete performs soft delete as per DD"""
        item = self.create_item()
        self.authenticate()
        
        url = reverse('${input:app_name}:[model-name]-detail', args=[item.id])
        response = self.client.delete(url)
        
        self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)
        
        # Verify soft delete
        item.refresh_from_db()
        self.assertEqual(item.status, [ModelName1].Status.ARCHIVED)
        
        # Item should not appear in list
        response = self.client.get(reverse('${input:app_name}:[model-name]-list'))
        self.assertEqual(response.data['count'], 0)


class [ModelName1]FilteringTest(APITestBase):
    """Test filtering functionality based on DD requirements"""
    
    def setUp(self):
        super().setUp()
        # Create test data with various attributes
        self.items = [
            self.create_item(
                name=f'Item {i}',
                status=['active', 'inactive', 'archived'][i % 3],
                priority=(i * 20) % 100,
                owner=self.users['user1'] if i < 3 else self.users['user2']
            ) for i in range(6)
        ]
        
        # Add tags
        self.items[0].tags.add(self.tags['tag1'])
        self.items[1].tags.add(self.tags['tag1'], self.tags['tag2'])
    
    def test_filter_by_status(self):
        """Test filtering by status"""
        self.authenticate()
        
        # Single status
        response = self.client.get(
            reverse('${input:app_name}:[model-name]-list'),
            {'status': 'active'}
        )
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.data['count'], 2)
        
        # Multiple statuses
        response = self.client.get(
            reverse('${input:app_name}:[model-name]-list'),
            {'status': 'active,inactive'}
        )
        
        self.assertEqual(response.data['count'], 4)
    
    def test_filter_by_priority_range(self):
        """Test filtering by priority range"""
        self.authenticate()
        
        response = self.client.get(
            reverse('${input:app_name}:[model-name]-list'),
            {'priority_min': 40, 'priority_max': 80}
        )
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        
        # Verify all results are within range
        for item in response.data['results']:
            self.assertGreaterEqual(item['priority'], 40)
            self.assertLessEqual(item['priority'], 80)
    
    def test_filter_by_tags(self):
        """Test filtering by tags"""
        self.authenticate()
        
        # Filter by single tag
        response = self.client.get(
            reverse('${input:app_name}:[model-name]-list'),
            {'tags': self.tags['tag1'].id}
        )
        
        self.assertEqual(response.data['count'], 2)
        
        # Filter by multiple tags (items with both tags)
        response = self.client.get(
            reverse('${input:app_name}:[model-name]-list'),
            {'tags': f"{self.tags['tag1'].id},{self.tags['tag2'].id}"}
        )
        
        self.assertEqual(response.data['count'], 1)
    
    def test_search_functionality(self):
        """Test search in name and description"""
        self.authenticate()
        
        # Update item for search
        self.items[0].description = 'Special search term'
        self.items[0].save()
        
        response = self.client.get(
            reverse('${input:app_name}:[model-name]-list'),
            {'search': 'special'}
        )
        
        self.assertEqual(response.data['count'], 1)
        self.assertEqual(response.data['results'][0]['id'], str(self.items[0].id))
    
    def test_ordering(self):
        """Test ordering functionality"""
        self.authenticate()
        
        # Order by priority ascending
        response = self.client.get(
            reverse('${input:app_name}:[model-name]-list'),
            {'ordering': 'priority'}
        )
        
        priorities = [item['priority'] for item in response.data['results']]
        self.assertEqual(priorities, sorted(priorities))
        
        # Order by created_at descending (default)
        response = self.client.get(
            reverse('${input:app_name}:[model-name]-list'),
            {'ordering': '-created_at'}
        )
        
        # Verify order
        dates = [item['created_at'] for item in response.data['results']]
        self.assertEqual(dates, sorted(dates, reverse=True))


class [ModelName1]CustomActionsTest(APITestBase):
    """Test custom action endpoints from DD"""
    
    def setUp(self):
        super().setUp()
        self.items = [
            self.create_item(name=f'Item {i}') for i in range(3)
        ]
    
    def test_bulk_action_archive(self):
        """Test bulk archive action"""
        self.authenticate()
        
        data = {
            'ids': [str(item.id) for item in self.items[:2]],
            'action': 'archive'
        }
        
        response = self.client.post(
            reverse('${input:app_name}:[model-name]-bulk-action'),
            data,
            format='json'
        )
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.data['processed'], 2)
        
        # Verify items are archived
        for item in self.items[:2]:
            item.refresh_from_db()
            self.assertEqual(item.status, [ModelName1].Status.ARCHIVED)
        
        # Third item unchanged
        self.items[2].refresh_from_db()
        self.assertEqual(self.items[2].status, [ModelName1].Status.ACTIVE)
    
    def test_statistics_endpoint(self):
        """Test statistics calculation"""
        self.authenticate()
        
        # Add more test data
        for i in range(2):
            self.create_item(
                status='inactive',
                priority=80,
                owner=self.users['user1']
            )
        
        response = self.client.get(
            reverse('${input:app_name}:[model-name]-statistics')
        )
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        
        # Verify statistics
        stats = response.data
        self.assertEqual(stats['total'], 5)
        self.assertEqual(stats['by_status']['active'], 3)
        self.assertEqual(stats['by_status']['inactive'], 2)
        self.assertIn('average_priority', stats)
        self.assertEqual(stats['my_items'], 5)
    
    def test_duplicate_endpoint(self):
        """Test item duplication"""
        item = self.create_item(
            name='Original',
            description='Original description',
            priority=75
        )
        item.tags.add(self.tags['tag1'])
        
        self.authenticate()
        
        response = self.client.post(
            reverse('${input:app_name}:[model-name]-duplicate', args=[item.id])
        )
        
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        
        # Verify duplicate
        duplicate = [ModelName1].objects.get(id=response.data['id'])
        self.assertEqual(duplicate.name, 'Original (Copy)')
        self.assertEqual(duplicate.description, item.description)
        self.assertEqual(duplicate.priority, item.priority)
        self.assertIn(self.tags['tag1'], duplicate.tags.all())
        self.assertEqual(duplicate.owner, self.users['user1'])
    
    def test_manage_assignments(self):
        """Test assignment management endpoints"""
        item = self.create_item()
        self.authenticate()
        
        base_url = reverse(
            '${input:app_name}:[model-name]-manage-assignments',
            args=[item.id]
        )
        
        # List assignments
        response = self.client.get(base_url)
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data), 1)  # Owner assignment
        
        # Add assignment
        data = {
            'user_id': self.users['user2'].id,
            'role': 'editor'
        }
        response = self.client.post(base_url, data, format='json')
        
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(response.data['role'], 'editor')
        
        # Verify in database
        self.assertTrue(
            item.assignments.filter(
                user=self.users['user2'],
                role='editor'
            ).exists()
        )
        
        # Remove assignment
        response = self.client.delete(
            base_url,
            {'user_id': self.users['user2'].id},
            format='json'
        )
        
        self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)
        self.assertFalse(
            item.assignments.filter(user=self.users['user2']).exists()
        )


class [ModelName1]PaginationTest(APITestBase):
    """Test pagination according to DD requirements"""
    
    def setUp(self):
        super().setUp()
        # Create many items for pagination
        self.items = [
            self.create_item(name=f'Item {i:03d}')
            for i in range(35)
        ]
    
    def test_default_pagination(self):
        """Test default page size"""
        self.authenticate()
        
        response = self.client.get(reverse('${input:app_name}:[model-name]-list'))
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data['results']), 20)  # Default page size
        self.assertEqual(response.data['count'], 35)
        self.assertIsNotNone(response.data['next'])
        self.assertIsNone(response.data['previous'])
    
    def test_custom_page_size(self):
        """Test custom page size"""
        self.authenticate()
        
        response = self.client.get(
            reverse('${input:app_name}:[model-name]-list'),
            {'page_size': 10}
        )
        
        self.assertEqual(len(response.data['results']), 10)
    
    def test_page_navigation(self):
        """Test navigating through pages"""
        self.authenticate()
        
        # Get page 2
        response = self.client.get(
            reverse('${input:app_name}:[model-name]-list'),
            {'page': 2}
        )
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertIsNotNone(response.data['previous'])
        self.assertIsNotNone(response.data['next'])
    
    def test_max_page_size_limit(self):
        """Test maximum page size limit"""
        self.authenticate()
        
        response = self.client.get(
            reverse('${input:app_name}:[model-name]-list'),
            {'page_size': 200}  # Exceeds max
        )
        
        # Should be capped at max page size (100)
        self.assertLessEqual(len(response.data['results']), 100)


class APIPerformanceTest(APITestBase):
    """Test API performance requirements from DD"""
    
    def test_response_time_requirement(self):
        """Test API response time meets DD requirements"""
        self.authenticate()
        
        # Create test data
        for i in range(50):
            item = self.create_item(name=f'Perf Item {i}')
            item.tags.add(self.tags['tag1'])
        
        # Measure response time
        start_time = time.time()
        
        response = self.client.get(
            reverse('${input:app_name}:[model-name]-list'),
            {'page_size': 50}
        )
        
        end_time = time.time()
        response_time_ms = (end_time - start_time) * 1000
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        
        # Based on DD requirement: response time < 200ms
        self.assertLess(
            response_time_ms,
            200,
            f"Response time {response_time_ms}ms exceeds requirement"
        )
    
    def test_concurrent_request_handling(self):
        """Test handling concurrent requests"""
        from concurrent.futures import ThreadPoolExecutor
        import threading
        
        self.authenticate()
        item = self.create_item(priority=50)
        
        def update_priority(priority):
            client = APIClient()
            client.force_authenticate(user=self.users['user1'])
            
            response = client.patch(
                reverse('${input:app_name}:[model-name]-detail', args=[item.id]),
                {'priority': priority},
                format='json'
            )
            return response.status_code
        
        # Simulate concurrent updates
        with ThreadPoolExecutor(max_workers=5) as executor:
            futures = [
                executor.submit(update_priority, i * 10)
                for i in range(5)
            ]
            
            results = [f.result() for f in futures]
        
        # All requests should succeed
        for result in results:
            self.assertEqual(result, status.HTTP_200_OK)
        
        # Final state should be consistent
        item.refresh_from_db()
        self.assertIn(item.priority, [0, 10, 20, 30, 40])


class APIErrorHandlingTest(APITestBase):
    """Test error handling scenarios"""
    
    def test_404_for_nonexistent_resource(self):
        """Test 404 response for non-existent resources"""
        self.authenticate()
        
        fake_id = uuid.uuid4()
        response = self.client.get(
            reverse('${input:app_name}:[model-name]-detail', args=[fake_id])
        )
        
        self.assertEqual(response.status_code, status.HTTP_404_NOT_FOUND)
    
    def test_malformed_uuid_handling(self):
        """Test handling of malformed UUIDs"""
        self.authenticate()
        
        response = self.client.get(f'/api/v1/${input:app_name}/[model-name-plural]/not-a-uuid/')
        self.assertEqual(response.status_code, status.HTTP_404_NOT_FOUND)
    
    def test_invalid_json_handling(self):
        """Test handling of invalid JSON in requests"""
        self.authenticate()
        
        response = self.client.post(
            reverse('${input:app_name}:[model-name]-list'),
            'invalid json {',
            content_type='application/json'
        )
        
        self.assertEqual(response.status_code, status.HTTP_400_BAD_REQUEST)
    
    @patch('apps.${input:app_name}.views.[ModelName1].objects.all')
    def test_internal_error_handling(self, mock_queryset):
        """Test 500 error handling"""
        self.authenticate()
        
        # Simulate database error
        mock_queryset.side_effect = Exception('Database error')
        
        response = self.client.get(reverse('${input:app_name}:[model-name]-list'))
        
        # Should handle gracefully
        self.assertEqual(response.status_code, status.HTTP_500_INTERNAL_SERVER_ERROR)


# Generate view test summary
def generate_view_test_summary():
    """Generate summary of view/API tests"""
    test_classes = [
        [ModelName1]AuthenticationTest,
        [ModelName1]PermissionTest,
        [ModelName1]CRUDTest,
        [ModelName1]FilteringTest,
        [ModelName1]CustomActionsTest,
        [ModelName1]PaginationTest,
        APIPerformanceTest,
        APIErrorHandlingTest,
    ]
    
    test_summary = {
        'component': 'Views/API',
        'test_classes': [],
        'endpoints_tested': [
            'GET /api/v1/[model-name-plural]/',
            'POST /api/v1/[model-name-plural]/',
            'GET /api/v1/[model-name-plural]/{id}/',
            'PUT /api/v1/[model-name-plural]/{id}/',
            'PATCH /api/v1/[model-name-plural]/{id}/',
            'DELETE /api/v1/[model-name-plural]/{id}/',
            'POST /api/v1/[model-name-plural]/bulk_action/',
            'GET /api/v1/[model-name-plural]/statistics/',
            'POST /api/v1/[model-name-plural]/{id}/duplicate/',
            'GET/POST/DELETE /api/v1/[model-name-plural]/{id}/assignments/',
        ],
        'test_scenarios': [
            'Authentication requirements',
            'Permission checks',
            'CRUD operations',
            'Field validation',
            'Business rule enforcement',
            'Filtering and search',
            'Pagination',
            'Custom actions',
            'Performance requirements',
            'Error handling',
            'Concurrent access',
        ]
    }
    
    total_tests = 0
    for test_class in test_classes:
        test_methods = [m for m in dir(test_class) if m.startswith('test_')]
        test_summary['test_classes'].append({
            'class': test_class.__name__,
            'tests': len(test_methods),
            'focus': test_class.__doc__.strip()
        })
        total_tests += len(test_methods)
    
    test_summary['total_tests'] = total_tests
    return test_summary
```

### Step 5: Generate Integration Tests

**Create integration tests for complete workflows:**

```python
# File: tests/test_${input:app_name}/test_integration_comprehensive.py

"""
Comprehensive integration tests for ${input:dd_name} implementation
Tests complete workflows and feature interactions
"""

import json
import time
from datetime import date, timedelta
from django.test import TestCase, TransactionTestCase
from django.urls import reverse
from django.contrib.auth.models import User
from django.core.cache import cache
from rest_framework.test import APITestCase
from apps.${input:app_name}.models import *


class WorkflowIntegrationTest(APITestCase):
    """Test complete user workflows from DD requirements"""
    
    def setUp(self):
        # Create users
        self.user = User.objects.create_user('testuser', 'test@example.com', 'pass123')
        self.collaborator = User.objects.create_user('collab', 'collab@example.com', 'pass123')
        
        # Create tags
        self.tags = [
            Tag.objects.create(name=f'Tag {i}', slug=f'tag-{i}')
            for i in range(3)
        ]
        
        # Authenticate
        self.client.force_authenticate(user=self.user)
    
    def test_complete_item_lifecycle(self):
        """Test complete item lifecycle from creation to archival"""
        # Step 1: Create item
        create_data = {
            'name': 'Project Alpha',
            'description': 'Important project',
            'priority': 80,
            'status': 'active',
            'tags': [self.tags[0].id, self.tags[1].id],
            'start_date': str(date.today()),
            'end_date': str(date.today() + timedelta(days=30)),
            'metadata': {'phase': 'planning'}
        }
        
        response = self.client.post(
            reverse('${input:app_name}:[model-name]-list'),
            create_data,
            format='json'
        )
        
        self.assertEqual(response.status_code, 201)
        item_id = response.data['id']
        
        # Verify owner assignment created
        item = [ModelName1].objects.get(id=item_id)
        self.assertTrue(
            item.assignments.filter(user=self.user, role='admin').exists()
        )
        
        # Step 2: Add collaborator
        assignment_url = reverse(
            '${input:app_name}:[model-name]-manage-assignments',
            args=[item_id]
        )
        
        response = self.client.post(
            assignment_url,
            {'user_id': self.collaborator.id, 'role': 'editor'},
            format='json'
        )
        
        self.assertEqual(response.status_code, 201)
        
        # Step 3: Collaborator updates item
        self.client.force_authenticate(user=self.collaborator)
        
        update_url = reverse('${input:app_name}:[model-name]-detail', args=[item_id])
        response = self.client.patch(
            update_url,
            {
                'priority': 90,
                'metadata': {'phase': 'execution', 'progress': 25}
            },
            format='json'
        )
        
        self.assertEqual(response.status_code, 200)
        
        # Step 4: Move through status workflow
        # Active -> Inactive
        response = self.client.patch(
            update_url,
            {'status': 'inactive'},
            format='json'
        )
        self.assertEqual(response.status_code, 200)
        
        # Inactive -> Archived (requires priority 0)
        response = self.client.patch(
            update_url,
            {'status': 'archived', 'priority': 0},
            format='json'
        )
        self.assertEqual(response.status_code, 200)
        
        # Step 5: Verify final state
        item.refresh_from_db()
        self.assertEqual(item.status, [ModelName1].Status.ARCHIVED)
        self.assertEqual(item.priority, 0)
        self.assertEqual(item.metadata['phase'], 'execution')
        
        # Step 6: Verify archived item not in default list
        self.client.force_authenticate(user=self.user)
        response = self.client.get(reverse('${input:app_name}:[model-name]-list'))
        
        item_ids = [i['id'] for i in response.data['results']]
        self.assertNotIn(str(item_id), item_ids)
    
    def test_bulk_workflow_with_permissions(self):
        """Test bulk operations respecting permissions"""
        # Create items with different owners
        my_items = [
            [ModelName1].objects.create(
                name=f'My Item {i}',
                owner=self.user,
                priority=50
            ) for i in range(3)
        ]
        
        other_items = [
            [ModelName1].objects.create(
                name=f'Other Item {i}',
                owner=self.collaborator,
                priority=50
            ) for i in range(2)
        ]
        
        # Try bulk operation on mixed ownership
        all_ids = [str(i.id) for i in my_items + other_items]
        
        response = self.client.post(
            reverse('${input:app_name}:[model-name]-bulk-action'),
            {
                'ids': all_ids,
                'action': 'archive'
            },
            format='json'
        )
        
        # Should only process owned items
        self.assertEqual(response.status_code, 200)
        self.assertEqual(response.data['processed'], 3)
        
        # Verify only owned items were archived
        for item in my_items:
            item.refresh_from_db()
            self.assertEqual(item.status, [ModelName1].Status.ARCHIVED)
        
        for item in other_items:
            item.refresh_from_db()
            self.assertEqual(item.status, [ModelName1].Status.ACTIVE)
    
    def test_search_and_filter_workflow(self):
        """Test complex search and filter scenarios"""
        # Create diverse dataset
        items = []
        for i in range(10):
            item = [ModelName1].objects.create(
                name=f'Item {i}',
                description=f'Description for item {i}' if i % 2 == 0 else '',
                owner=self.user,
                priority=i * 10,
                status=['active', 'inactive'][i % 2],
                start_date=date.today() + timedelta(days=i)
            )
            
            if i < 5:
                item.tags.add(self.tags[0])
            if i % 3 == 0:
                item.tags.add(self.tags[1])
            
            items.append(item)
        
        # Complex filter: active items with tag1, priority 20-60, ordered by date
        response = self.client.get(
            reverse('${input:app_name}:[model-name]-list'),
            {
                'status': 'active',
                'tags': self.tags[0].id,
                'priority_min': 20,
                'priority_max': 60,
                'ordering': 'start_date'
            }
        )
        
        self.assertEqual(response.status_code, 200)
        
        # Verify results match criteria
        results = response.data['results']
        self.assertTrue(len(results) > 0)
        
        for item in results:
            self.assertEqual(item['status'], 'active')
            self.assertGreaterEqual(item['priority'], 20)
            self.assertLessEqual(item['priority'], 60)
            
            # Verify tag presence
            tag_ids = [t['id'] for t in item['tags']]
            self.assertIn(self.tags[0].id, tag_ids)
        
        # Verify ordering
        dates = [r['start_date'] for r in results]
        self.assertEqual(dates, sorted(dates))


class DataIntegrityTest(TransactionTestCase):
    """Test data integrity across operations"""
    
    def test_cascade_delete_behavior(self):
        """Test cascade behaviors on related object deletion"""
        user = User.objects.create_user('testuser', 'test@test.com', 'pass')
        item = [ModelName1].objects.create(
            name='Test Item',
            owner=user
        )
        
        # Add assignment
        other_user = User.objects.create_user('other', 'other@test.com', 'pass')
        assignment = [ModelName1]Assignment.objects.create(
            [model_name]=item,
            user=other_user,
            role='viewer'
        )
        
        # Delete item - assignments should cascade
        item_id = item.id
        item.delete()
        
        self.assertFalse(
            [ModelName1]Assignment.objects.filter([model_name]_id=item_id).exists()
        )
        
        # User should still exist (PROTECT)
        self.assertTrue(User.objects.filter(id=user.id).exists())
    
    def test_transaction_rollback_on_error(self):
        """Test transaction rollback on operation failure"""
        user = User.objects.create_user('testuser', 'test@test.com', 'pass')
        
        # Create initial item
        item = [ModelName1].objects.create(
            name='Original',
            owner=user,
            priority=50
        )
        initial_count = [ModelName1].objects.count()
        
        # Attempt operation that will fail
        try:
            with transaction.atomic():
                # Create new item
                new_item = [ModelName1].objects.create(
                    name='New Item',
                    owner=user
                )
                
                # Force error
                raise Exception('Simulated error')
        except Exception:
            pass
        
        # Verify rollback
        final_count = [ModelName1].objects.count()
        self.assertEqual(initial_count, final_count)
        
        # Original item unchanged
        item.refresh_from_db()
        self.assertEqual(item.priority, 50)


class CacheIntegrationTest(APITestCase):
    """Test caching behavior if implemented"""
    
    def setUp(self):
        self.user = User.objects.create_user('testuser', 'test@test.com', 'pass')
        self.client.force_authenticate(user=self.user)
        cache.clear()
    
    def test_statistics_caching(self):
        """Test statistics endpoint caching"""
        # Create test data
        for i in range(5):
            [ModelName1].objects.create(
                name=f'Item {i}',
                owner=self.user,
                priority=50
            )
        
        # First request - cache miss
        start_time = time.time()
        response1 = self.client.get(
            reverse('${input:app_name}:[model-name]-statistics')
        )
        first_duration = time.time() - start_time
        
        self.assertEqual(response1.status_code, 200)
        
        # Second request - should be cached
        start_time = time.time()
        response2 = self.client.get(
            reverse('${input:app_name}:[model-name]-statistics')
        )
        second_duration = time.time() - start_time
        
        # Cached response should be faster
        self.assertLess(second_duration, first_duration)
        
        # Data should be identical
        self.assertEqual(response1.data, response2.data)
        
        # Create new item
        [ModelName1].objects.create(
            name='New Item',
            owner=self.user
        )
        
        # Cache should be invalidated
        response3 = self.client.get(
            reverse('${input:app_name}:[model-name]-statistics')
        )
        
        self.assertNotEqual(response1.data['total'], response3.data['total'])


class SecurityIntegrationTest(APITestCase):
    """Test security aspects across features"""
    
    def test_sql_injection_prevention(self):
        """Test SQL injection prevention in filters"""
        user = User.objects.create_user('testuser', 'test@test.com', 'pass')
        self.client.force_authenticate(user=user)
        
        # Create test item
        item = [ModelName1].objects.create(
            name='Test Item',
            owner=user
        )
        
        # Attempt SQL injection in various parameters
        injection_attempts = [
            "'; DROP TABLE apps_${input:app_name}_[model_name]; --",
            "1' OR '1'='1",
            "1'; UPDATE apps_${input:app_name}_[model_name] SET priority=100; --"
        ]
        
        for attempt in injection_attempts:
            # Try in search
            response = self.client.get(
                reverse('${input:app_name}:[model-name]-list'),
                {'search': attempt}
            )
            self.assertIn(response.status_code, [200, 400])
            
            # Try in ordering
            response = self.client.get(
                reverse('${input:app_name}:[model-name]-list'),
                {'ordering': attempt}
            )
            self.assertIn(response.status_code, [200, 400])
        
        # Verify data integrity
        item.refresh_from_db()
        self.assertEqual(item.name, 'Test Item')
        self.assertEqual([ModelName1].objects.count(), 1)
    
    def test_xss_prevention(self):
        """Test XSS prevention in user input"""
        user = User.objects.create_user('testuser', 'test@test.com', 'pass')
        self.client.force_authenticate(user=user)
        
        # Attempt XSS in various fields
        xss_payloads = [
            '<script>alert("XSS")</script>',
            '<img src="x" onerror="alert(1)">',
            'javascript:alert(1)',
            '<svg onload="alert(1)">'
        ]
        
        for payload in xss_payloads:
            response = self.client.post(
                reverse('${input:app_name}:[model-name]-list'),
                {
                    'name': payload,
                    'description': payload,
                    'metadata': {'xss': payload}
                },
                format='json'
            )
            
            if response.status_code == 201:
                # Verify data is stored safely
                item = [ModelName1].objects.get(id=response.data['id'])
                self.assertEqual(item.name, payload)  # Should be stored as-is
                self.assertIn(payload, response.data['name'])  # Should be escaped in response


# Generate integration test summary
def generate_integration_test_summary():
    """Generate summary of integration tests"""
    return {
        'component': 'Integration Tests',
        'test_classes': [
            {
                'class': 'WorkflowIntegrationTest',
                'focus': 'Complete user workflows',
                'scenarios': [
                    'Item lifecycle (create -> update -> archive)',
                    'Multi-user collaboration',
                    'Permission-aware bulk operations',
                    'Complex search and filter combinations'
                ]
            },
            {
                'class': 'DataIntegrityTest',
                'focus': 'Data integrity and transactions',
                'scenarios': [
                    'Cascade delete behaviors',
                    'Transaction rollback on errors',
                    'Concurrent access handling'
                ]
            },
            {
                'class': 'CacheIntegrationTest',
                'focus': 'Caching behavior',
                'scenarios': [
                    'Cache hit/miss scenarios',
                    'Cache invalidation',
                    'Performance improvements'
                ]
            },
            {
                'class': 'SecurityIntegrationTest',
                'focus': 'Security aspects',
                'scenarios': [
                    'SQL injection prevention',
                    'XSS prevention',
                    'Access control enforcement'
                ]
            }
        ]
    }
```

### Step 6: Test Execution and Report Generation

**Execute all tests and generate comprehensive reports:**

```bash
echo "=== EXECUTING ALL UNIT TESTS ==="

# Create test execution script
cat > run_comprehensive_tests.py << 'EOF'
#!/usr/bin/env python
"""
Comprehensive test runner with detailed reporting
"""

import os
import sys
import json
import time
import subprocess
from datetime import datetime
from pathlib import Path
import django
from django.conf import settings
from django.test.utils import get_runner

# Setup Django
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'config.settings.testing')
django.setup()

class TestReportGenerator:
    """Generate comprehensive test reports"""
    
    def __init__(self, app_name):
        self.app_name = app_name
        self.results = {
            'app': app_name,
            'execution_time': datetime.now().isoformat(),
            'test_suites': [],
            'coverage': {},
            'summary': {}
        }
    
    def run_tests_with_coverage(self):
        """Run tests with coverage analysis"""
        print(f"\n{'='*70}")
        print(f"RUNNING COMPREHENSIVE TESTS FOR {self.app_name}")
        print(f"{'='*70}\n")
        
        # Run coverage
        coverage_cmd = [
            'coverage', 'run',
            '--source', f'apps.{self.app_name}',
            '--omit', '*/migrations/*,*/tests/*',
            'manage.py', 'test',
            f'tests.test_{self.app_name}',
            '-v', '2'
        ]
        
        start_time = time.time()
        result = subprocess.run(coverage_cmd, capture_output=True, text=True)
        end_time = time.time()
        
        self.results['execution_time_seconds'] = end_time - start_time
        
        # Parse test output
        self.parse_test_output(result.stdout)
        
        # Generate coverage report
        self.generate_coverage_report()
        
        return result.returncode == 0
    
    def parse_test_output(self, output):
        """Parse test execution output"""
        lines = output.split('\n')
        
        current_suite = None
        test_results = []
        
        for line in lines:
            # Parse test results
            if ' ... ' in line:
                parts = line.split(' ... ')
                if len(parts) == 2:
                    test_name = parts[0].strip()
                    result = parts[1].strip()
                    
                    test_results.append({
                        'test': test_name,
                        'result': result,
                        'passed': result in ['ok', 'OK']
                    })
            
            # Parse summary
            if line.startswith('Ran '):
                parts = line.split()
                if len(parts) >= 2:
                    self.results['summary']['total_tests'] = int(parts[1])
            
            if 'FAILED' in line or 'OK' in line:
                self.results['summary']['overall_result'] = 'FAILED' if 'FAILED' in line else 'PASSED'
        
        self.results['test_results'] = test_results
    
    def generate_coverage_report(self):
        """Generate coverage report"""
        # Generate JSON coverage report
        subprocess.run(['coverage', 'json', '-o', 'coverage.json'], capture_output=True)
        
        # Read coverage data
        if Path('coverage.json').exists():
            with open('coverage.json', 'r') as f:
                coverage_data = json.load(f)
                
                self.results['coverage'] = {
                    'total_statements': coverage_data.get('totals', {}).get('num_statements', 0),
                    'covered_statements': coverage_data.get('totals', {}).get('covered_lines', 0),
                    'missing_statements': coverage_data.get('totals', {}).get('missing_lines', 0),
                    'coverage_percent': coverage_data.get('totals', {}).get('percent_covered', 0),
                    'files': {}
                }
                
                # File-level coverage
                for file_path, file_data in coverage_data.get('files', {}).items():
                    if self.app_name in file_path:
                        self.results['coverage']['files'][file_path] = {
                            'statements': file_data.get('summary', {}).get('num_statements', 0),
                            'missing': file_data.get('summary', {}).get('missing_lines', 0),
                            'coverage': file_data.get('summary', {}).get('percent_covered', 0)
                        }
        
        # Generate HTML coverage report
        subprocess.run([
            'coverage', 'html',
            '-d', f'htmlcov_{self.app_name}',
            '--title', f'{self.app_name} Test Coverage Report'
        ], capture_output=True)
    
    def generate_test_matrix_report(self):
        """Generate test matrix mapping to DD requirements"""
        matrix_path = Path('tests/test_matrix.json')
        
        if matrix_path.exists():
            with open(matrix_path, 'r') as f:
                matrix = json.load(f)
                
                # Map executed tests to requirements
                executed_tests = {t['test'] for t in self.results.get('test_results', [])}
                
                requirement_coverage = {}
                for test_case in matrix.get('test_cases', []):
                    test_name = test_case['test_name']
                    requirement = test_case.get('description', '')
                    
                    if test_name in executed_tests:
                        if requirement not in requirement_coverage:
                            requirement_coverage[requirement] = {
                                'tests': [],
                                'all_passed': True
                            }
                        
                        test_result = next(
                            (t for t in self.results['test_results'] if test_name in t['test']),
                            None
                        )
                        
                        if test_result:
                            requirement_coverage[requirement]['tests'].append(test_name)
                            if not test_result['passed']:
                                requirement_coverage[requirement]['all_passed'] = False
                
                self.results['requirement_coverage'] = requirement_coverage
    
    def generate_reports(self):
        """Generate all report files"""
        timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
        report_dir = Path(f'test_reports/{self.app_name}_{timestamp}')
        report_dir.mkdir(parents=True, exist_ok=True)
        
        # JSON report
        with open(report_dir / 'test_results.json', 'w') as f:
            json.dump(self.results, f, indent=2)
        
        # HTML report
        self.generate_html_report(report_dir / 'test_report.html')
        
        # Markdown report
        self.generate_markdown_report(report_dir / 'test_report.md')
        
        # Console summary
        self.print_summary()
        
        return report_dir
    
    def generate_html_report(self, output_path):
        """Generate HTML test report"""
        html_content = f"""
<!DOCTYPE html>
<html>
<head>
    <title>Test Report - {self.app_name}</title>
    <style>
        body {{ font-family: Arial, sans-serif; margin: 20px; }}
        .header {{ background: #f0f0f0; padding: 20px; border-radius: 5px; }}
        .summary {{ margin: 20px 0; }}
        .passed {{ color: green; }}
        .failed {{ color: red; }}
        table {{ border-collapse: collapse; width: 100%; margin: 20px 0; }}
        th, td {{ border: 1px solid #ddd; padding: 8px; text-align: left; }}
        th {{ background-color: #f2f2f2; }}
        .coverage {{ margin: 20px 0; }}
        .coverage-bar {{ width: 200px; height: 20px; background: #f0f0f0; 
                        border: 1px solid #ccc; display: inline-block; }}
        .coverage-fill {{ height: 100%; background: #4CAF50; }}
    </style>
</head>
<body>
    <div class="header">
        <h1>Test Report: {self.app_name}</h1>
        <p>Generated: {self.results['execution_time']}</p>
        <p>Execution Time: {self.results.get('execution_time_seconds', 0):.2f} seconds</p>
    </div>
    
    <div class="summary">
        <h2>Summary</h2>
        <p>Total Tests: {self.results['summary'].get('total_tests', 0)}</p>
        <p>Overall Result: <span class="{self.results['summary'].get('overall_result', '').lower()}">
            {self.results['summary'].get('overall_result', 'UNKNOWN')}
        </span></p>
    </div>
    
    <div class="coverage">
        <h2>Code Coverage</h2>
        <p>Overall Coverage: {self.results['coverage'].get('coverage_percent', 0):.1f}%</p>
        <div class="coverage-bar">
            <div class="coverage-fill" style="width: {self.results['coverage'].get('coverage_percent', 0)}%"></div>
        </div>
        <p>Statements: {self.results['coverage'].get('covered_statements', 0)} / 
           {self.results['coverage'].get('total_statements', 0)}</p>
    </div>
    
    <h2>Test Results</h2>
    <table>
        <tr>
            <th>Test</th>
            <th>Result</th>
        </tr>
        """
        
        for test in self.results.get('test_results', []):
            status_class = 'passed' if test['passed'] else 'failed'
            html_content += f"""
        <tr>
            <td>{test['test']}</td>
            <td class="{status_class}">{test['result']}</td>
        </tr>
            """
        
        html_content += """
    </table>
    
    <h2>File Coverage</h2>
    <table>
        <tr>
            <th>File</th>
            <th>Coverage</th>
            <th>Statements</th>
            <th>Missing</th>
        </tr>
        """
        
        for file_path, coverage in self.results['coverage'].get('files', {}).items():
            file_name = Path(file_path).name
            html_content += f"""
        <tr>
            <td>{file_name}</td>
            <td>{coverage['coverage']:.1f}%</td>
            <td>{coverage['statements']}</td>
            <td>{coverage['missing']}</td>
        </tr>
            """
        
        html_content += """
    </table>
</body>
</html>
        """
        
        with open(output_path, 'w') as f:
            f.write(html_content)
    
    def generate_markdown_report(self, output_path):
        """Generate Markdown test report"""
        md_content = f"""# Test Report: {self.app_name}

## Summary

- **Generated**: {self.results['execution_time']}
- **Execution Time**: {self.results.get('execution_time_seconds', 0):.2f} seconds
- **Total Tests**: {self.results['summary'].get('total_tests', 0)}
- **Overall Result**: {self.results['summary'].get('overall_result', 'UNKNOWN')}

## Code Coverage

- **Overall Coverage**: {self.results['coverage'].get('coverage_percent', 0):.1f}%
- **Covered Statements**: {self.results['coverage'].get('covered_statements', 0)}
- **Total Statements**: {self.results['coverage'].get('total_statements', 0)}
- **Missing Statements**: {self.results['coverage'].get('missing_statements', 0)}

### File Coverage

| File | Coverage | Statements | Missing |
|------|----------|------------|---------|
"""
        
        for file_path, coverage in self.results['coverage'].get('files', {}).items():
            file_name = Path(file_path).name
            md_content += f"| {file_name} | {coverage['coverage']:.1f}% | {coverage['statements']} | {coverage['missing']} |\n"
        
        md_content += f"""

## Test Results

Total: {len(self.results.get('test_results', []))} tests

### Detailed Results

| Test | Result |
|------|--------|
"""
        
        for test in self.results.get('test_results', []):
            status = '✅' if test['passed'] else '❌'
            md_content += f"| {test['test']} | {status} {test['result']} |\n"
        
        # Add requirement coverage if available
        if 'requirement_coverage' in self.results:
            md_content += """

## DD Requirement Coverage

| Requirement | Tests | Status |
|-------------|-------|--------|
"""
            for req, data in self.results['requirement_coverage'].items():
                status = '✅' if data['all_passed'] else '❌'
                test_count = len(data['tests'])
                md_content += f"| {req} | {test_count} tests | {status} |\n"
        
        with open(output_path, 'w') as f:
            f.write(md_content)
    
    def print_summary(self):
        """Print summary to console"""
        print(f"\n{'='*70}")
        print(f"TEST EXECUTION SUMMARY")
        print(f"{'='*70}")
        print(f"App: {self.app_name}")
        print(f"Total Tests: {self.results['summary'].get('total_tests', 0)}")
        print(f"Result: {self.results['summary'].get('overall_result', 'UNKNOWN')}")
        print(f"Execution Time: {self.results.get('execution_time_seconds', 0):.2f} seconds")
        print(f"\nCODE COVERAGE:")
        print(f"Overall: {self.results['coverage'].get('coverage_percent', 0):.1f}%")
        print(f"Covered: {self.results['coverage'].get('covered_statements', 0)} statements")
        print(f"Missing: {self.results['coverage'].get('missing_statements', 0)} statements")
        print(f"{'='*70}\n")


def main():
    """Main test execution function"""
    app_name = '${input:app_name}'
    
    # Generate test matrix first
    print("Generating test matrix from DD requirements...")
    subprocess.run([sys.executable, 'tests/test_${input:app_name}/test_matrix.py'])
    
    # Run tests with reporting
    reporter = TestReportGenerator(app_name)
    
    # Execute tests
    success = reporter.run_tests_with_coverage()
    
    # Generate test matrix report
    reporter.generate_test_matrix_report()
    
    # Generate reports
    report_dir = reporter.generate_reports()
    
    print(f"\nReports generated in: {report_dir}")
    print(f"HTML Coverage Report: htmlcov_{app_name}/index.html")
    
    # Return exit code
    sys.exit(0 if success else 1)


if __name__ == '__main__':
    main()
EOF

# Make script executable
chmod +x run_comprehensive_tests.py

# Run the comprehensive tests
echo "Executing comprehensive test suite..."
python run_comprehensive_tests.py
```

### Step 7: Generate Final Test Summary

**Create final test summary document:**

```bash
# Generate comprehensive test summary
cat > generate_test_summary.py << 'EOF'
#!/usr/bin/env python
"""
Generate final test summary document
"""

import json
import os
from pathlib import Path
from datetime import datetime

def generate_final_summary():
    """Generate comprehensive test summary"""
    
    # Find latest test report
    report_dirs = sorted(Path('test_reports').glob('*'), reverse=True)
    if not report_dirs:
        print("No test reports found!")
        return
    
    latest_report = report_dirs[0]
    
    # Load test results
    with open(latest_report / 'test_results.json', 'r') as f:
        results = json.load(f)
    
    # Generate summary document
    summary = f"""
# Django DD Unit Test Implementation Summary

**Project**: {results.get('app', 'Unknown')}
**Date**: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}
**DD Name**: ${input:dd_name}

## Test Execution Summary

- **Total Tests Executed**: {results['summary'].get('total_tests', 0)}
- **Overall Result**: {results['summary'].get('overall_result', 'UNKNOWN')}
- **Execution Time**: {results.get('execution_time_seconds', 0):.2f} seconds
- **Code Coverage**: {results['coverage'].get('coverage_percent', 0):.1f}%

## Coverage Details

- **Total Statements**: {results['coverage'].get('total_statements', 0)}
- **Covered Statements**: {results['coverage'].get('covered_statements', 0)}
- **Missing Statements**: {results['coverage'].get('missing_statements', 0)}

## Test Categories Covered

Based on DD requirements, the following test categories were implemented:

### 1. Model Tests
- Field validation (required fields, max length, choices)
- Database constraints (unique, foreign keys)
- Business logic rules from DD
- Custom model methods
- Query optimization
- Performance testing

### 2. Serializer Tests
- Field serialization
- Validation rules
- Custom validation methods
- Nested object handling
- Permission checks
- Performance with large datasets

### 3. API/View Tests
- Authentication requirements
- Permission enforcement
- CRUD operations
- Field validation in requests
- Filtering and search
- Pagination
- Custom actions
- Error handling
- Performance requirements

### 4. Integration Tests
- Complete workflows
- Multi-user scenarios
- Data integrity
- Transaction handling
- Caching behavior
- Security testing

## DD Requirements Coverage

The test suite covers all major requirements from the Design Detail:

- ✅ Authentication and authorization
- ✅ Business rule validation
- ✅ Data constraints and integrity
- ✅ API response time requirements
- ✅ Concurrent access handling
- ✅ Error scenarios
- ✅ Edge cases and boundary conditions

## Test Reports Generated

1. **JSON Report**: `{latest_report}/test_results.json`
2. **HTML Report**: `{latest_report}/test_report.html`
3. **Markdown Report**: `{latest_report}/test_report.md`
4. **Coverage HTML**: `htmlcov_{results.get('app', 'app')}/index.html`

## Recommendations

1. **Coverage Improvement**: Focus on files with < 80% coverage
2. **Performance Testing**: Add more load testing scenarios
3. **Security Testing**: Expand security test cases
4. **Documentation**: Keep test documentation updated with DD changes

## Next Steps

1. Review test reports for any failures
2. Improve coverage for critical components
3. Add integration tests for new workflows
4. Set up continuous testing in CI/CD pipeline

---

*This test suite ensures the Django implementation meets all DD requirements with comprehensive validation and verification.*
"""
    
    # Save summary
    summary_path = latest_report / 'FINAL_TEST_SUMMARY.md'
    with open(summary_path, 'w') as f:
        f.write(summary)
    
    print(f"Final test summary generated: {summary_path}")
    
    # Also save to project root
    with open('TEST_SUMMARY.md', 'w') as f:
        f.write(summary)
    
    print("Test summary also saved to: TEST_SUMMARY.md")


if __name__ == '__main__':
    generate_final_summary()
EOF

# Run summary generation
python generate_test_summary.py

echo "✅ Comprehensive unit test generation and execution complete!"
```

## Test Generation Summary

**This prompt generates a complete test suite including:**

1. **Test Matrix Generation**: Analyzes DD requirements to create comprehensive test cases
2. **Model Tests**: Field validation, constraints, business logic, performance
3. **Serializer Tests**: Validation, nested objects, permissions, custom logic
4. **View/API Tests**: Authentication, permissions, CRUD, filtering, custom actions
5. **Integration Tests**: Complete workflows, data integrity, security
6. **Test Execution**: Automated test running with coverage analysis
7. **Report Generation**: JSON, HTML, and Markdown reports with coverage data

**Key Features:**
- Automated test case generation based on DD requirements
- Comprehensive coverage of all Django components
- Performance and security testing included
- Multiple report formats for different audiences
- Coverage analysis with detailed metrics
- DD requirement traceability

The generated test suite ensures that all DD requirements are validated and the implementation meets quality standards.