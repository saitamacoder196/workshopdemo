---
description: Generate detailed Django coding implementation plan based on Design Detail (DD) impact assessment
---

# Django Design Detail (DD) Implementation Plan Generator

This prompt converts the impact assessment from prompt 03 into a detailed, step-by-step coding implementation plan, including all code files, migrations, configurations, and testing requirements.

## Prerequisites

**Required Input:**
- 📋 **DD Impact Assessment**: Output from `03_django_dd_impact_analysis.prompt.md`
- 📦 **Project Name**: `${input:project_name}`
- 📁 **DD Name**: `${input:dd_name}`
- 🔍 **Impact Assessment File**: `${input:impact_assessment_path}`

**Output Structure:**
This prompt follows the template defined in `.github/templates/output_templates.md` for Implementation Planning (Prompt 04).

**Output Folder:** `dd_implementation_plan/`
**Main Output File:** `implementation/implementation_plan.json`

## Implementation Plan Generation Workflow

### Step 1: Parse Impact Assessment Results

**First, analyze the completed impact assessment:**

```bash
# Load the impact assessment results
echo "=== LOADING DD IMPACT ASSESSMENT ==="
cat ${input:impact_assessment_path}

# Extract key implementation requirements
echo "=== EXTRACTING IMPLEMENTATION REQUIREMENTS ==="
```

**Parse and extract:**
- New Django apps to create
- New models to implement
- Existing models to modify
- New API endpoints to create
- Existing endpoints to modify
- Database migrations required
- Configuration changes needed

### Step 2: Database Implementation Plan

#### 2.1 Migration Plan Generation

**Create detailed migration sequence:**

```
=== DATABASE MIGRATION IMPLEMENTATION PLAN ===

Migration Sequence for ${input:dd_name}:

MIGRATION 1: Foundation Models
├── File: apps/[app_name]/migrations/000X_create_base_models.py
├── Purpose: Create core models without complex relationships
├── Models to Create:
│   ├── Model: [ModelName1]
│   │   ├── Fields Implementation:
│   │   │   ├── id = models.AutoField(primary_key=True)
│   │   │   ├── [field_name] = models.[FieldType]([parameters])
│   │   │   ├── created_at = models.DateTimeField(auto_now_add=True)
│   │   │   └── updated_at = models.DateTimeField(auto_now=True)
│   │   ├── Meta Class:
│   │   │   ├── db_table = '[table_name]'
│   │   │   ├── ordering = ['-created_at']
│   │   │   └── indexes = [models.Index(fields=['field_name'])]
│   │   └── Model Methods:
│   │       ├── def __str__(self): return [string_representation]
│   │       └── def [custom_method](self): [method_implementation]
│   └── Model: [ModelName2]
│       └── [similar structure]
├── Migration Code Template:
│   ```python
│   # Generated migration code structure
│   from django.db import migrations, models
│   
│   class Migration(migrations.Migration):
│       dependencies = [
│           ('[app_name]', '000X_previous_migration'),
│       ]
│       
│       operations = [
│           migrations.CreateModel(
│               name='[ModelName]',
│               fields=[
│                   # Field definitions here
│               ],
│               options={
│                   # Meta options here
│               },
│           ),
│       ]
│   ```
└── Validation Commands:
    ├── python manage.py makemigrations --dry-run
    ├── python manage.py sqlmigrate [app] 000X
    └── python manage.py migrate --plan

MIGRATION 2: Model Relationships
├── File: apps/[app_name]/migrations/000X_add_relationships.py  
├── Purpose: Add ForeignKey, ManyToMany, OneToOne relationships
├── Relationship Changes:
│   ├── Add ForeignKey: [model].[field] -> [related_model]
│   │   ├── Field Definition: models.ForeignKey([RelatedModel], on_delete=models.[CASCADE/PROTECT/SET_NULL])
│   │   ├── Related Names: related_name='[reverse_name]'
│   │   └── Database Index: db_index=True (if needed)
│   ├── Add ManyToMany: [model].[field] <-> [related_model]
│   │   ├── Field Definition: models.ManyToManyField([RelatedModel])
│   │   ├── Through Model: through='[ThroughModel]' (if custom)
│   │   └── Symmetrical: symmetrical=False (if needed)
│   └── Add OneToOne: [model].[field] -> [related_model]
│       ├── Field Definition: models.OneToOneField([RelatedModel], on_delete=models.[CASCADE/PROTECT])
│       └── Parent Link: parent_link=True (if inheritance)
├── Migration Code Template:
│   ```python
│   operations = [
│       migrations.AddField(
│           model_name='[model_name]',
│           name='[field_name]',
│           field=models.[FieldType]([parameters]),
│       ),
│   ]
│   ```
└── Data Migration (if needed):
    ├── Forward Migration: [describe data transformation]
    ├── Reverse Migration: [describe rollback logic]
    └── Code Template:
        ```python
        def migrate_data_forward(apps, schema_editor):
            # Data migration logic here
            pass
        
        def migrate_data_reverse(apps, schema_editor):
            # Rollback logic here
            pass
        
        operations = [
            migrations.RunPython(migrate_data_forward, migrate_data_reverse),
        ]
        ```

MIGRATION 3: Indexes and Constraints
├── File: apps/[app_name]/migrations/000X_add_indexes_constraints.py
├── Purpose: Add performance indexes and data constraints
├── Index Additions:
│   ├── Single Field Indexes:
│   │   ├── Field: [field_name] -> Index: idx_[table]_[field]
│   │   └── Code: models.Index(fields=['[field_name]'], name='idx_[table]_[field]')
│   ├── Composite Indexes:
│   │   ├── Fields: [field1, field2] -> Index: idx_[table]_[field1]_[field2]
│   │   └── Code: models.Index(fields=['[field1]', '[field2]'], name='idx_[table]_composite')
│   └── Unique Constraints:
│       ├── Fields: [field1, field2] -> Constraint: uniq_[table]_[fields]
│       └── Code: models.UniqueConstraint(fields=['[field1]', '[field2]'], name='uniq_[table]_[fields]')
├── Migration Code Template:
│   ```python
│   operations = [
│       migrations.AlterModelOptions(
│           name='[model_name]',
│           options={
│               'indexes': [
│                   models.Index(fields=['[field]'], name='[index_name]'),
│               ],
│           },
│       ),
│   ]
│   ```
└── Performance Testing:
    ├── Query Performance: Test queries that use new indexes
    ├── Index Usage: EXPLAIN ANALYZE for key queries
    └── Migration Time: Estimate time for large tables

MIGRATION 4: Model Modifications (if any)
├── File: apps/[app_name]/migrations/000X_modify_existing_models.py
├── Purpose: Modify existing models based on DD requirements
├── Field Modifications:
│   ├── Add Fields:
│   │   ├── Field: [new_field_name] = models.[FieldType]([parameters])
│   │   ├── Default Value: default=[value] (if required field)
│   │   └── Null Handling: null=True, blank=True (if optional)
│   ├── Modify Fields:
│   │   ├── From: [old_field_type] -> To: [new_field_type]
│   │   ├── Data Preservation: [describe how data is preserved]
│   │   └── Validation: [new validation rules]
│   └── Remove Fields:
│       ├── Field: [field_to_remove]
│       ├── Impact: [describe impact on existing data]
│       └── Backup: [data backup strategy before removal]
├── Migration Code Template:
│   ```python
│   operations = [
│       migrations.AddField(
│           model_name='[model]',
│           name='[field]',
│           field=models.[FieldType]([params]),
│       ),
│       migrations.AlterField(
│           model_name='[model]',
│           name='[field]',
│           field=models.[NewFieldType]([params]),
│       ),
│   ]
│   ```
└── Testing Strategy:
    ├── Backup Production Data: [backup commands]
    ├── Test on Staging: [staging migration test]
    └── Rollback Testing: [rollback validation]
```

#### 2.2 Models Implementation Plan

**Detailed model code implementation:**

```
=== MODELS IMPLEMENTATION PLAN ===

File: apps/[app_name]/models.py

MODEL IMPLEMENTATION STRUCTURE:

# Imports Section
from django.db import models
from django.contrib.auth.models import User
from django.core.validators import [validators_needed]
from apps.core.models import BaseModel  # If using base model
import uuid

# Model 1: [ModelName1] Implementation
class [ModelName1](BaseModel):  # or models.Model
    """
    [Model description and purpose]
    
    Business Rules:
    - [business_rule_1]
    - [business_rule_2]
    
    Related Models:
    - [related_model_1]: [relationship_description]
    - [related_model_2]: [relationship_description]
    """
    
    # Primary Key (if custom)
    id = models.UUIDField(
        primary_key=True, 
        default=uuid.uuid4, 
        editable=False,
        help_text="Unique identifier for [model_name]"
    )
    
    # Core Fields Implementation
    [field_name_1] = models.[FieldType](
        max_length=[value],  # if applicable
        null=[True/False],
        blank=[True/False],
        default=[default_value],
        choices=[CHOICES],  # if applicable
        validators=[validators_list],
        help_text="[field_description]",
        db_index=[True/False],  # for frequently queried fields
        unique=[True/False],  # if unique constraint needed
    )
    
    [field_name_2] = models.[FieldType](
        # Similar structure for each field
    )
    
    # Relationship Fields Implementation
    [foreign_key_field] = models.ForeignKey(
        [RelatedModel],
        on_delete=models.[CASCADE/PROTECT/SET_NULL],
        related_name='[reverse_relation_name]',
        null=[True/False],
        blank=[True/False],
        help_text="[relationship_description]",
        db_index=True,  # for performance
    )
    
    [many_to_many_field] = models.ManyToManyField(
        [RelatedModel],
        through='[ThroughModel]',  # if custom through model
        related_name='[reverse_relation_name]',
        blank=True,
        help_text="[relationship_description]",
    )
    
    # Timestamp Fields (if not using BaseModel)
    created_at = models.DateTimeField(
        auto_now_add=True,
        help_text="Timestamp when record was created"
    )
    updated_at = models.DateTimeField(
        auto_now=True,
        help_text="Timestamp when record was last updated"
    )
    
    # Status/State Fields
    is_active = models.BooleanField(
        default=True,
        help_text="Whether this record is active"
    )
    
    class Meta:
        """Model metadata and database configuration"""
        db_table = '[custom_table_name]'  # if custom table name needed
        ordering = ['-created_at', '[field_name]']
        verbose_name = '[Human Readable Name]'
        verbose_name_plural = '[Human Readable Plural Name]'
        
        # Database Indexes for Performance
        indexes = [
            models.Index(fields=['[field1]'], name='idx_[table]_[field1]'),
            models.Index(fields=['[field1]', '[field2]'], name='idx_[table]_composite'),
        ]
        
        # Unique Constraints
        constraints = [
            models.UniqueConstraint(
                fields=['[field1]', '[field2]'], 
                name='uniq_[table]_[field1]_[field2]'
            ),
            models.CheckConstraint(
                check=models.Q([field_name]__gte=0),
                name='chk_[table]_[field]_positive'
            ),
        ]
        
        # Permissions
        permissions = [
            ('[permission_code]', '[Permission Description]'),
        ]
    
    # String Representation
    def __str__(self):
        """Human-readable string representation"""
        return f"[ModelName1]: {self.[descriptive_field]}"
    
    def __repr__(self):
        """Developer-friendly representation"""
        return f"<{self.__class__.__name__}(id={self.pk}, [key_field]={self.[key_field]})>"
    
    # Custom Model Methods
    def [custom_method_name](self):
        """
        [Method description and business logic]
        
        Returns:
            [return_type]: [return_description]
        """
        # Implementation logic here
        pass
    
    # Property Methods
    @property
    def [property_name](self):
        """
        [Property description - computed field]
        
        Returns:
            [return_type]: [return_description]
        """
        # Property logic here
        return [computed_value]
    
    # Class Methods
    @classmethod
    def [class_method_name](cls, [parameters]):
        """
        [Class method description]
        
        Args:
            [param_name] ([param_type]): [param_description]
            
        Returns:
            [return_type]: [return_description]
        """
        # Class method logic here
        pass
    
    # Validation Methods
    def clean(self):
        """
        Custom model validation beyond field-level validation
        """
        from django.core.exceptions import ValidationError
        
        # Custom validation logic
        if [validation_condition]:
            raise ValidationError({
                '[field_name]': '[Error message]'
            })
    
    def save(self, *args, **kwargs):
        """
        Override save method for custom business logic
        """
        # Pre-save logic
        self.clean()  # Run validation
        
        # Custom business logic before save
        if [condition]:
            # Business logic here
            pass
        
        # Call parent save method
        super().save(*args, **kwargs)
        
        # Post-save logic (if needed)
        # Note: Be careful with post-save logic to avoid infinite loops

# Model 2: [ModelName2] Implementation  
class [ModelName2](BaseModel):
    """[Similar structure for each model]"""
    # Follow same pattern as Model 1
    pass

# Through Models (for ManyToMany relationships with additional fields)
class [ThroughModelName](models.Model):
    """
    Through model for [Model1] and [Model2] relationship
    
    Additional Fields:
    - [field_description_1]
    - [field_description_2]
    """
    
    [model1_field] = models.ForeignKey(
        [Model1],
        on_delete=models.CASCADE,
        related_name='[related_name]'
    )
    
    [model2_field] = models.ForeignKey(
        [Model2], 
        on_delete=models.CASCADE,
        related_name='[related_name]'
    )
    
    # Additional fields specific to the relationship
    [additional_field] = models.[FieldType]([parameters])
    
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        unique_together = [('[model1_field]', '[model2_field]')]
        db_table = '[through_table_name]'
        
    def __str__(self):
        return f"{self.[model1_field]} - {self.[model2_field]}"

# Constants and Choices (if needed)
class [ModelName]Status(models.TextChoices):
    """Status choices for [ModelName]"""
    ACTIVE = 'active', 'Active'
    INACTIVE = 'inactive', 'Inactive'
    PENDING = 'pending', 'Pending'
    ARCHIVED = 'archived', 'Archived'

# Model Manager (if custom queryset methods needed)
class [ModelName]Manager(models.Manager):
    """Custom manager for [ModelName] with business logic methods"""
    
    def active(self):
        """Return only active records"""
        return self.filter(is_active=True)
    
    def by_status(self, status):
        """Filter by status"""
        return self.filter(status=status)
    
    def [custom_queryset_method](self, [parameters]):
        """
        [Method description]
        
        Args:
            [param]: [description]
            
        Returns:
            QuerySet: [description]
        """
        return self.filter([filter_conditions])

# Apply custom manager to model (add this to the model class)
# objects = [ModelName]Manager()
```

### Step 3: Serializers Implementation Plan

**Detailed serializer code implementation:**

```
=== SERIALIZERS IMPLEMENTATION PLAN ===

File: apps/[app_name]/serializers.py

SERIALIZER IMPLEMENTATION STRUCTURE:

# Imports Section
from rest_framework import serializers
from rest_framework.validators import UniqueValidator, UniqueTogetherValidator
from django.contrib.auth.models import User
from django.core.exceptions import ValidationError
from .models import [ModelName1], [ModelName2], [ThroughModelName]
from apps.[related_app].serializers import [RelatedSerializer]
import re

# Base Serializer (if common functionality needed)
class BaseModelSerializer(serializers.ModelSerializer):
    """
    Base serializer with common fields and methods
    """
    created_at = serializers.DateTimeField(read_only=True)
    updated_at = serializers.DateTimeField(read_only=True)
    
    class Meta:
        abstract = True

# Primary Model Serializers

# 1. [ModelName1] Read Serializer (for GET requests)
class [ModelName1]ReadSerializer(BaseModelSerializer):
    """
    Serializer for reading [ModelName1] data (GET operations)
    
    Features:
    - Includes all fields for display
    - Nested related objects
    - Computed fields
    - Read-only optimization
    """
    
    # Nested Related Objects (read-only)
    [related_field] = serializers.SerializerMethodField()
    [foreign_key_field] = serializers.StringRelatedField(read_only=True)
    
    # Computed Fields
    [computed_field] = serializers.SerializerMethodField()
    
    # URL Field (if needed for API navigation)
    url = serializers.HyperlinkedIdentityField(
        view_name='[app_name]:[model_name]-detail',
        lookup_field='pk'
    )
    
    class Meta:
        model = [ModelName1]
        fields = [
            'id', 'url', '[field1]', '[field2]', '[foreign_key_field]',
            '[related_field]', '[computed_field]', 'created_at', 'updated_at'
        ]
        read_only_fields = [
            'id', 'created_at', 'updated_at'
        ]
    
    def get_[related_field](self, obj):
        """
        Get related objects data
        
        Args:
            obj ([ModelName1]): The model instance
            
        Returns:
            list: List of related objects data
        """
        related_objects = obj.[related_field].filter(is_active=True)
        return [RelatedSerializer](related_objects, many=True).data
    
    def get_[computed_field](self, obj):
        """
        Calculate computed field value
        
        Args:
            obj ([ModelName1]): The model instance
            
        Returns:
            [return_type]: Computed value
        """
        # Computation logic here
        return [computed_value]

# 2. [ModelName1] Write Serializer (for POST/PUT/PATCH requests)
class [ModelName1]WriteSerializer(serializers.ModelSerializer):
    """
    Serializer for writing [ModelName1] data (POST/PUT/PATCH operations)
    
    Features:
    - Field validation
    - Business logic validation
    - Nested object creation/updates
    - Custom save logic
    """
    
    # Field Overrides with Custom Validation
    [field_name] = serializers.[FieldType](
        required=[True/False],
        allow_null=[True/False],
        allow_blank=[True/False],
        max_length=[value],
        min_length=[value],
        validators=[
            [CustomValidator](),
            UniqueValidator(queryset=[ModelName1].objects.all())
        ],
        help_text="[Field description]",
        style={'input_type': 'text', 'placeholder': '[Placeholder text]'}
    )
    
    # Nested Write Fields
    [nested_field] = [NestedSerializer](required=False)
    
    # Many-to-Many Field Handling
    [many_to_many_field] = serializers.PrimaryKeyRelatedField(
        many=True,
        queryset=[RelatedModel].objects.filter(is_active=True),
        required=False
    )
    
    class Meta:
        model = [ModelName1]
        fields = [
            '[field1]', '[field2]', '[foreign_key_field]',
            '[nested_field]', '[many_to_many_field]'
        ]
        extra_kwargs = {
            '[field_name]': {
                'write_only': True,  # if sensitive field
                'style': {'input_type': 'password'}  # for password fields
            }
        }
        
        # Model-level validation
        validators = [
            UniqueTogetherValidator(
                queryset=[ModelName1].objects.all(),
                fields=['[field1]', '[field2]'],
                message="[Custom error message]"
            )
        ]
    
    def validate_[field_name](self, value):
        """
        Field-level validation for [field_name]
        
        Args:
            value: The field value to validate
            
        Returns:
            [field_type]: Validated value
            
        Raises:
            serializers.ValidationError: If validation fails
        """
        # Custom validation logic
        if [validation_condition]:
            raise serializers.ValidationError(
                "[Error message]"
            )
        
        # Additional business logic validation
        if [business_rule_condition]:
            raise serializers.ValidationError(
                "[Business rule error message]"
            )
        
        return value
    
    def validate(self, attrs):
        """
        Object-level validation (cross-field validation)
        
        Args:
            attrs (dict): Dictionary of field values
            
        Returns:
            dict: Validated attributes
            
        Raises:
            serializers.ValidationError: If validation fails
        """
        # Cross-field validation
        if attrs.get('[field1]') and attrs.get('[field2]'):
            if [cross_field_condition]:
                raise serializers.ValidationError({
                    '[field_name]': "[Cross-field validation error]"
                })
        
        # Business logic validation
        if [business_logic_condition]:
            raise serializers.ValidationError(
                "[Business logic error message]"
            )
        
        return attrs
    
    def create(self, validated_data):
        """
        Custom create method with business logic
        
        Args:
            validated_data (dict): Validated field data
            
        Returns:
            [ModelName1]: Created model instance
        """
        # Extract nested data
        [nested_data] = validated_data.pop('[nested_field]', None)
        [many_to_many_data] = validated_data.pop('[many_to_many_field]', [])
        
        # Create main object
        instance = [ModelName1].objects.create(**validated_data)
        
        # Handle nested object creation
        if [nested_data]:
            [NestedModel].objects.create(
                [parent_field]=instance,
                **[nested_data]
            )
        
        # Handle many-to-many relationships
        if [many_to_many_data]:
            instance.[many_to_many_field].set([many_to_many_data])
        
        # Custom business logic after creation
        [self._perform_post_create_logic(instance)]
        
        return instance
    
    def update(self, instance, validated_data):
        """
        Custom update method with business logic
        
        Args:
            instance ([ModelName1]): Existing model instance
            validated_data (dict): Validated field data
            
        Returns:
            [ModelName1]: Updated model instance
        """
        # Extract nested data
        [nested_data] = validated_data.pop('[nested_field]', None)
        [many_to_many_data] = validated_data.pop('[many_to_many_field]', None)
        
        # Update main object fields
        for attr, value in validated_data.items():
            setattr(instance, attr, value)
        
        # Handle nested object updates
        if [nested_data]:
            [nested_obj], created = [NestedModel].objects.get_or_create(
                [parent_field]=instance,
                defaults=[nested_data]
            )
            if not created:
                for attr, value in [nested_data].items():
                    setattr([nested_obj], attr, value)
                [nested_obj].save()
        
        # Handle many-to-many updates
        if [many_to_many_data] is not None:
            instance.[many_to_many_field].set([many_to_many_data])
        
        # Save main instance
        instance.save()
        
        # Custom business logic after update
        [self._perform_post_update_logic(instance)]
        
        return instance
    
    def _perform_post_create_logic(self, instance):
        """
        Custom business logic to execute after object creation
        
        Args:
            instance ([ModelName1]): Created model instance
        """
        # Post-creation business logic
        # Examples: send notifications, update related objects, etc.
        pass
    
    def _perform_post_update_logic(self, instance):
        """
        Custom business logic to execute after object update
        
        Args:
            instance ([ModelName1]): Updated model instance
        """
        # Post-update business logic
        pass

# 3. Nested Serializers for Related Objects
class [NestedModelName]Serializer(serializers.ModelSerializer):
    """
    Serializer for nested [NestedModelName] objects
    """
    
    class Meta:
        model = [NestedModelName]
        fields = ['id', '[field1]', '[field2]']
        read_only_fields = ['id']

# 4. List Serializers (for bulk operations)
class [ModelName1]ListSerializer(serializers.ListSerializer):
    """
    Custom list serializer for bulk operations on [ModelName1]
    """
    
    def create(self, validated_data):
        """
        Bulk create multiple instances
        
        Args:
            validated_data (list): List of validated data dictionaries
            
        Returns:
            list: List of created instances
        """
        instances = []
        for item_data in validated_data:
            instance = [ModelName1].objects.create(**item_data)
            instances.append(instance)
        
        return instances
    
    def update(self, instances, validated_data):
        """
        Bulk update multiple instances
        
        Args:
            instances (list): List of existing instances
            validated_data (list): List of validated data dictionaries
            
        Returns:
            list: List of updated instances
        """
        # Map instances by their ID for efficient lookup
        instance_mapping = {instance.id: instance for instance in instances}
        data_mapping = {item['id']: item for item in validated_data}
        
        # Update existing instances
        updated_instances = []
        for instance_id, data in data_mapping.items():
            instance = instance_mapping.get(instance_id)
            if instance:
                for attr, value in data.items():
                    setattr(instance, attr, value)
                instance.save()
                updated_instances.append(instance)
        
        return updated_instances

# Apply list serializer to main serializer
# Add this to [ModelName1]WriteSerializer Meta class:
# list_serializer_class = [ModelName1]ListSerializer

# 5. Custom Validation Classes
class [CustomValidator]:
    """
    Custom validator for [specific_validation_logic]
    """
    
    def __init__(self, [parameters]):
        self.[parameter] = [parameter]
    
    def __call__(self, value):
        """
        Validate the value
        
        Args:
            value: Value to validate
            
        Raises:
            serializers.ValidationError: If validation fails
        """
        if [validation_condition]:
            raise serializers.ValidationError(
                f"[Error message with {value}]"
            )

# 6. Pagination Serializers (if custom pagination needed)
class [ModelName1]PaginatedSerializer(serializers.Serializer):
    """
    Serializer for paginated [ModelName1] responses
    """
    count = serializers.IntegerField()
    next = serializers.URLField(allow_null=True)
    previous = serializers.URLField(allow_null=True)
    results = [ModelName1]ReadSerializer(many=True)

# 7. Filter/Search Serializers
class [ModelName1]FilterSerializer(serializers.Serializer):
    """
    Serializer for filtering/searching [ModelName1] objects
    """
    [search_field] = serializers.CharField(
        required=False,
        help_text="Search in [field_description]"
    )
    
    [filter_field] = serializers.ChoiceField(
        choices=[ModelName1].[CHOICE_FIELD],
        required=False,
        help_text="Filter by [field_description]"
    )
    
    [date_from] = serializers.DateTimeField(
        required=False,
        help_text="Filter records from this date"
    )
    
    [date_to] = serializers.DateTimeField(
        required=False,
        help_text="Filter records until this date"
    )
    
    def validate(self, attrs):
        """
        Validate filter parameters
        """
        date_from = attrs.get('[date_from]')
        date_to = attrs.get('[date_to]')
        
        if date_from and date_to and date_from > date_to:
            raise serializers.ValidationError(
                "Date from must be before date to"
            )
        
        return attrs
```

### Step 4: Views Implementation Plan

**Detailed views code implementation:**

```
=== VIEWS IMPLEMENTATION PLAN ===

File: apps/[app_name]/views.py

VIEWS IMPLEMENTATION STRUCTURE:

# Imports Section
from rest_framework import generics, status, filters
from rest_framework.decorators import api_view, permission_classes, action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated, IsAuthenticatedOrReadOnly
from rest_framework.viewsets import ModelViewSet, ReadOnlyModelViewSet
from rest_framework.pagination import PageNumberPagination
from django_filters.rest_framework import DjangoFilterBackend
from django.db import transaction
from django.shortcuts import get_object_or_404
from django.http import Http404
from drf_yasg.utils import swagger_auto_schema
from drf_yasg import openapi

from .models import [ModelName1], [ModelName2]
from .serializers import (
    [ModelName1]ReadSerializer, [ModelName1]WriteSerializer,
    [ModelName1]FilterSerializer, [ModelName2]ReadSerializer
)
from .permissions import [CustomPermission]
from .filters import [CustomFilter]
from apps.core.mixins import [CustomMixin]
from apps.core.utils import [utility_function]

# Custom Pagination Classes
class [ModelName1]Pagination(PageNumberPagination):
    """
    Custom pagination for [ModelName1] endpoints
    """
    page_size = 20
    page_size_query_param = 'page_size'
    max_page_size = 100

# ViewSets (for full CRUD operations)

class [ModelName1]ViewSet(ModelViewSet):
    """
    ViewSet for [ModelName1] providing full CRUD operations
    
    Endpoints:
    - GET /api/v1/[model-name]/ - List all objects (with filtering/search)
    - POST /api/v1/[model-name]/ - Create new object
    - GET /api/v1/[model-name]/{id}/ - Retrieve single object
    - PUT /api/v1/[model-name]/{id}/ - Update object (full update)
    - PATCH /api/v1/[model-name]/{id}/ - Partial update object
    - DELETE /api/v1/[model-name]/{id}/ - Delete object
    
    Custom Actions:
    - POST /api/v1/[model-name]/{id}/[custom-action]/ - Custom business action
    """
    
    queryset = [ModelName1].objects.filter(is_active=True)
    permission_classes = [IsAuthenticated, [CustomPermission]]
    pagination_class = [ModelName1]Pagination
    filter_backends = [DjangoFilterBackend, filters.SearchFilter, filters.OrderingFilter]
    filterset_class = [CustomFilter]
    search_fields = ['[field1]', '[field2]']
    ordering_fields = ['created_at', '[field1]']
    ordering = ['-created_at']
    
    def get_serializer_class(self):
        """
        Return appropriate serializer class based on action
        """
        if self.action in ['list', 'retrieve']:
            return [ModelName1]ReadSerializer
        return [ModelName1]WriteSerializer
    
    def get_queryset(self):
        """
        Customize queryset based on user permissions and filters
        """
        queryset = super().get_queryset()
        
        # Apply user-specific filtering
        if not self.request.user.is_staff:
            # Non-staff users can only see their own objects
            queryset = queryset.filter([user_field]=self.request.user)
        
        # Apply additional business logic filtering
        if [business_condition]:
            queryset = queryset.filter([business_filter])
        
        # Optimize queries with select_related/prefetch_related
        queryset = queryset.select_related('[foreign_key_field]')
        queryset = queryset.prefetch_related('[many_to_many_field]')
        
        return queryset
    
    def perform_create(self, serializer):
        """
        Custom logic during object creation
        """
        # Set user from request
        serializer.save([user_field]=self.request.user)
        
        # Custom business logic after creation
        instance = serializer.instance
        [self._perform_post_create_actions(instance)]
    
    def perform_update(self, serializer):
        """
        Custom logic during object update
        """
        # Store old instance for comparison
        old_instance = self.get_object()
        
        # Save updated instance
        serializer.save()
        new_instance = serializer.instance
        
        # Custom business logic after update
        [self._perform_post_update_actions(old_instance, new_instance)]
    
    def perform_destroy(self, instance):
        """
        Custom logic during object deletion (soft delete)
        """
        # Soft delete instead of hard delete
        instance.is_active = False
        instance.save()
        
        # Custom business logic after deletion
        [self._perform_post_delete_actions(instance)]
    
    @swagger_auto_schema(
        method='post',
        operation_description="Custom business action for [ModelName1]",
        request_body=openapi.Schema(
            type=openapi.TYPE_OBJECT,
            properties={
                '[parameter]': openapi.Schema(type=openapi.TYPE_STRING, description='[Description]'),
            }
        ),
        responses={
            200: openapi.Response(
                description="Action completed successfully",
                examples={
                    "application/json": {
                        "success": True,
                        "message": "Action completed",
                        "data": {}
                    }
                }
            )
        }
    )
    @action(detail=True, methods=['post'], permission_classes=[IsAuthenticated])
    def [custom_action](self, request, pk=None):
        """
        Custom business action for individual [ModelName1] objects
        
        Args:
            request: HTTP request object
            pk: Primary key of the object
            
        Returns:
            Response: JSON response with action result
        """
        instance = self.get_object()
        
        # Validate input parameters
        [parameter] = request.data.get('[parameter]')
        if not [parameter]:
            return Response(
                {'error': '[Parameter] is required'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        try:
            # Perform business logic
            result = [self._perform_custom_action(instance, parameter)]
            
            return Response({
                'success': True,
                'message': '[Action completed successfully]',
                'data': result
            }, status=status.HTTP_200_OK)
            
        except [CustomException] as e:
            return Response(
                {'error': str(e)},
                status=status.HTTP_400_BAD_REQUEST
            )
    
    @action(detail=False, methods=['get'])
    def [statistics_action](self, request):
        """
        Get statistics for [ModelName1] objects
        
        Returns:
            Response: Statistics data
        """
        queryset = self.get_queryset()
        
        statistics = {
            'total_count': queryset.count(),
            'active_count': queryset.filter(is_active=True).count(),
            '[custom_metric]': [self._calculate_custom_metric(queryset)],
        }
        
        return Response(statistics, status=status.HTTP_200_OK)
    
    def _perform_post_create_actions(self, instance):
        """
        Business logic to execute after object creation
        
        Args:
            instance ([ModelName1]): Created instance
        """
        # Example: Send notification, update related objects, etc.
        [utility_function](instance, 'created')
    
    def _perform_post_update_actions(self, old_instance, new_instance):
        """
        Business logic to execute after object update
        
        Args:
            old_instance ([ModelName1]): Instance before update
            new_instance ([ModelName1]): Instance after update
        """
        # Example: Track changes, send notifications, etc.
        if old_instance.[field] != new_instance.[field]:
            [utility_function](new_instance, 'field_changed')
    
    def _perform_post_delete_actions(self, instance):
        """
        Business logic to execute after object deletion
        
        Args:
            instance ([ModelName1]): Deleted instance
        """
        # Example: Cleanup related objects, send notifications, etc.
        [utility_function](instance, 'deleted')
    
    def _perform_custom_action(self, instance, parameter):
        """
        Business logic for custom action
        
        Args:
            instance ([ModelName1]): Target instance
            parameter (str): Action parameter
            
        Returns:
            dict: Action result data
        """
        # Custom business logic implementation
        result = {'processed': True, 'parameter': parameter}
        return result
    
    def _calculate_custom_metric(self, queryset):
        """
        Calculate custom metric for statistics
        
        Args:
            queryset: QuerySet to calculate metric for
            
        Returns:
            [return_type]: Calculated metric value
        """
        # Custom metric calculation
        return queryset.aggregate([aggregation_function])['[metric_name]']

# Generic API Views (for simpler endpoints)

class [ModelName1]ListView(generics.ListCreateAPIView):
    """
    List and create [ModelName1] objects
    
    GET: List all objects with filtering and pagination
    POST: Create new object
    """
    
    queryset = [ModelName1].objects.filter(is_active=True)
    permission_classes = [IsAuthenticatedOrReadOnly]
    pagination_class = [ModelName1]Pagination
    filter_backends = [DjangoFilterBackend, filters.SearchFilter]
    filterset_fields = ['[field1]', '[field2]']
    search_fields = ['[field1]', '[field2]']
    
    def get_serializer_class(self):
        """Return appropriate serializer based on method"""
        if self.request.method == 'GET':
            return [ModelName1]ReadSerializer
        return [ModelName1]WriteSerializer
    
    @swagger_auto_schema(
        operation_description="List [ModelName1] objects with filtering and search",
        manual_parameters=[
            openapi.Parameter(
                '[filter_param]',
                openapi.IN_QUERY,
                description="Filter by [field_description]",
                type=openapi.TYPE_STRING
            ),
            openapi.Parameter(
                'search',
                openapi.IN_QUERY,
                description="Search in [searchable_fields]",
                type=openapi.TYPE_STRING
            ),
        ]
    )
    def get(self, request, *args, **kwargs):
        """List objects with custom documentation"""
        return super().get(request, *args, **kwargs)
    
    def perform_create(self, serializer):
        """Custom create logic"""
        serializer.save([user_field]=self.request.user)

class [ModelName1]DetailView(generics.RetrieveUpdateDestroyAPIView):
    """
    Retrieve, update, or delete a [ModelName1] object
    
    GET: Retrieve single object
    PUT: Update object (full update)
    PATCH: Partial update object
    DELETE: Delete object
    """
    
    queryset = [ModelName1].objects.filter(is_active=True)
    permission_classes = [IsAuthenticated, [CustomPermission]]
    
    def get_serializer_class(self):
        """Return appropriate serializer based on method"""
        if self.request.method == 'GET':
            return [ModelName1]ReadSerializer
        return [ModelName1]WriteSerializer
    
    def perform_destroy(self, instance):
        """Soft delete instead of hard delete"""
        instance.is_active = False
        instance.save()

# Function-Based Views (for simple endpoints)

@swagger_auto_schema(
    method='get',
    operation_description="Get health status of [ModelName1] service",
    responses={
        200: openapi.Response(
            description="Service health status",
            examples={
                "application/json": {
                    "status": "healthy",
                    "service": "[ModelName1] Service",
                    "timestamp": "2024-01-01T00:00:00Z",
                    "total_objects": 100
                }
            }
        )
    }
)
@api_view(['GET'])
@permission_classes([IsAuthenticated])
def [model_name]_health_check(request):
    """
    Health check endpoint for [ModelName1] service
    
    Returns:
        Response: Service health status and basic statistics
    """
    try:
        total_objects = [ModelName1].objects.count()
        active_objects = [ModelName1].objects.filter(is_active=True).count()
        
        return Response({
            'status': 'healthy',
            'service': '[ModelName1] Service',
            'timestamp': timezone.now().isoformat(),
            'statistics': {
                'total_objects': total_objects,
                'active_objects': active_objects
            }
        }, status=status.HTTP_200_OK)
        
    except Exception as e:
        return Response({
            'status': 'unhealthy',
            'service': '[ModelName1] Service',
            'error': str(e),
            'timestamp': timezone.now().isoformat()
        }, status=status.HTTP_503_SERVICE_UNAVAILABLE)

@swagger_auto_schema(
    method='post',
    operation_description="Bulk operations on [ModelName1] objects",
    request_body=openapi.Schema(
        type=openapi.TYPE_OBJECT,
        properties={
            'action': openapi.Schema(
                type=openapi.TYPE_STRING,
                enum=['activate', 'deactivate', 'delete'],
                description='Bulk action to perform'
            ),
            'object_ids': openapi.Schema(
                type=openapi.TYPE_ARRAY,
                items=openapi.Schema(type=openapi.TYPE_INTEGER),
                description='List of object IDs to process'
            )
        },
        required=['action', 'object_ids']
    )
)
@api_view(['POST'])
@permission_classes([IsAuthenticated])
def [model_name]_bulk_operations(request):
    """
    Perform bulk operations on [ModelName1] objects
    
    Supported actions:
    - activate: Activate selected objects
    - deactivate: Deactivate selected objects
    - delete: Soft delete selected objects
    
    Returns:
        Response: Operation result with affected object count
    """
    action = request.data.get('action')
    object_ids = request.data.get('object_ids', [])
    
    if not action or not object_ids:
        return Response(
            {'error': 'Action and object_ids are required'},
            status=status.HTTP_400_BAD_REQUEST
        )
    
    if action not in ['activate', 'deactivate', 'delete']:
        return Response(
            {'error': 'Invalid action. Supported: activate, deactivate, delete'},
            status=status.HTTP_400_BAD_REQUEST
        )
    
    try:
        with transaction.atomic():
            queryset = [ModelName1].objects.filter(id__in=object_ids)
            
            if action == 'activate':
                affected = queryset.update(is_active=True)
            elif action == 'deactivate':
                affected = queryset.update(is_active=False)
            elif action == 'delete':
                affected = queryset.update(is_active=False)  # Soft delete
            
            return Response({
                'success': True,
                'action': action,
                'affected_count': affected,
                'message': f'{affected} objects {action}d successfully'
            }, status=status.HTTP_200_OK)
            
    except Exception as e:
        return Response(
            {'error': f'Bulk operation failed: {str(e)}'},
            status=status.HTTP_500_INTERNAL_SERVER_ERROR
        )

# Error Handling Views
def custom_404_view(request, exception):
    """Custom 404 error handler"""
    return Response(
        {'error': 'Resource not found'},
        status=status.HTTP_404_NOT_FOUND
    )

def custom_500_view(request):
    """Custom 500 error handler"""
    return Response(
        {'error': 'Internal server error'},
        status=status.HTTP_500_INTERNAL_SERVER_ERROR
    )
```

### Step 5: URL Configuration Implementation Plan

**Detailed URL routing implementation:**

```
=== URL CONFIGURATION IMPLEMENTATION PLAN ===

File: apps/[app_name]/urls.py

URL ROUTING STRUCTURE:

# Imports Section
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from rest_framework.urlpatterns import format_suffix_patterns
from .views import (
    [ModelName1]ViewSet, [ModelName2]ViewSet,
    [ModelName1]ListView, [ModelName1]DetailView,
    [model_name]_health_check, [model_name]_bulk_operations
)

# App namespace for URL reversing
app_name = '[app_name]'

# Router for ViewSets (provides automatic URL routing)
router = DefaultRouter()
router.register(
    r'[model-name-plural]',  # URL prefix: /api/v1/[model-name-plural]/
    [ModelName1]ViewSet,
    basename='[model-name]'  # For URL reversing: [app_name]:[model-name]-list
)
router.register(
    r'[model2-name-plural]',
    [ModelName2]ViewSet,
    basename='[model2-name]'
)

# Manual URL patterns for non-ViewSet views
urlpatterns = [
    # Health Check Endpoints
    path(
        'health/',
        [model_name]_health_check,
        name='health-check'
    ),
    
    # Bulk Operations
    path(
        '[model-name-plural]/bulk/',
        [model_name]_bulk_operations,
        name='[model-name]-bulk-operations'
    ),
    
    # Generic API Views (alternative to ViewSets)
    path(
        '[alternative-endpoint]/',
        [ModelName1]ListView.as_view(),
        name='[model-name]-list-alt'
    ),
    path(
        '[alternative-endpoint]/<int:pk>/',
        [ModelName1]DetailView.as_view(),
        name='[model-name]-detail-alt'
    ),
    
    # Custom Business Logic Endpoints
    path(
        '[model-name-plural]/<int:pk>/[custom-action]/',
        [custom_action_view],
        name='[model-name]-custom-action'
    ),
    
    # Nested Resource Endpoints
    path(
        '[parent-resource]/<int:parent_id>/[child-resource]/',
        [NestedResourceView].as_view(),
        name='nested-resource-list'
    ),
]

# Include router URLs
urlpatterns += router.urls

# Format suffix patterns (for .json, .xml endpoints)
urlpatterns = format_suffix_patterns(urlpatterns)

# Complete URL Structure Generated:
"""
Router-generated URLs:
- GET/POST     /api/v1/[app-name]/[model-name-plural]/
- GET/PUT/PATCH/DELETE /api/v1/[app-name]/[model-name-plural]/{id}/
- POST         /api/v1/[app-name]/[model-name-plural]/{id}/[custom-action]/
- GET          /api/v1/[app-name]/[model-name-plural]/[statistics-action]/

Manual URLs:
- GET          /api/v1/[app-name]/health/
- POST         /api/v1/[app-name]/[model-name-plural]/bulk/
- GET/POST     /api/v1/[app-name]/[alternative-endpoint]/
- GET/PUT/PATCH/DELETE /api/v1/[app-name]/[alternative-endpoint]/{id}/
"""
```

**Main Project URLs (config/urls.py):**

```
=== MAIN PROJECT URL CONFIGURATION ===

File: config/urls.py

MAIN URL ROUTING:

# Imports Section
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static
from rest_framework.documentation import include_docs_urls
from drf_yasg.views import get_schema_view
from drf_yasg import openapi
from rest_framework import permissions

# API Documentation Setup
schema_view = get_schema_view(
    openapi.Info(
        title="${input:project_name} API",
        default_version='v1',
        description="API documentation for ${input:dd_name} implementation",
        terms_of_service="https://www.example.com/terms/",
        contact=openapi.Contact(email="api@example.com"),
        license=openapi.License(name="MIT License"),
    ),
    public=True,
    permission_classes=[permissions.AllowAny],
)

urlpatterns = [
    # Django Admin
    path('admin/', admin.site.urls),
    
    # API Endpoints
    path('api/v1/', include([
        # Core API endpoints
        path('', include('apps.api.urls')),
        
        # App-specific endpoints
        path('[app-name]/', include('apps.[app_name].urls')),
        
        # Authentication endpoints
        path('auth/', include('apps.authentication.urls')),
        
        # Additional app endpoints (if any)
        # path('[other-app]/', include('apps.[other_app].urls')),
    ])),
    
    # API Documentation
    path('api/docs/', include_docs_urls(title='${input:project_name} API')),
    path('swagger/', schema_view.with_ui('swagger', cache_timeout=0), name='schema-swagger-ui'),
    path('swagger.json', schema_view.without_ui(cache_timeout=0), name='schema-json'),
    path('redoc/', schema_view.with_ui('redoc', cache_timeout=0), name='schema-redoc'),
]

# Debug Toolbar URLs (development only)
if settings.DEBUG and 'debug_toolbar' in settings.INSTALLED_APPS:
    import debug_toolbar
    urlpatterns = [
        path('__debug__/', include(debug_toolbar.urls)),
    ] + urlpatterns

# Static and Media Files (development only)
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
    urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```

### Step 6: Configuration Changes Implementation Plan

**Django settings and configuration updates:**

```
=== CONFIGURATION CHANGES IMPLEMENTATION PLAN ===

1. DJANGO SETTINGS UPDATES

File: config/settings/base.py

NEW INSTALLED APPS:
# Add to INSTALLED_APPS
INSTALLED_APPS = [
    # ... existing apps ...
    
    # New apps for DD implementation
    'apps.[app_name]',  # Main business logic app
    '[new_third_party_app]',  # If DD requires new packages
    
    # Additional third-party apps (if needed)
    'django_filters',  # For advanced filtering
    'drf_yasg',  # For API documentation
]

NEW MIDDLEWARE (if required):
MIDDLEWARE = [
    # ... existing middleware ...
    
    # Add new middleware if DD requires it
    '[new_middleware_class]',
]

DATABASE CONFIGURATION UPDATES:
# If DD requires additional database connections
DATABASES = {
    'default': {
        # ... existing default database ...
    },
    # Add new database connection if needed
    '[new_database_name]': {
        'ENGINE': 'django.db.backends.[database_engine]',
        'NAME': config('[NEW_DATABASE_NAME]'),
        'USER': config('[NEW_DATABASE_USER]'),
        'PASSWORD': config('[NEW_DATABASE_PASSWORD]'),
        'HOST': config('[NEW_DATABASE_HOST]', default='localhost'),
        'PORT': config('[NEW_DATABASE_PORT]', default='5432'),
    }
}

# Database routing (if multiple databases)
DATABASE_ROUTERS = ['apps.[app_name].routers.[CustomDatabaseRouter]']

REST FRAMEWORK UPDATES:
REST_FRAMEWORK = {
    # ... existing settings ...
    
    # Add new settings if DD requires them
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
        'rest_framework.filters.SearchFilter',
        'rest_framework.filters.OrderingFilter',
    ],
    
    # Custom pagination (if needed)
    'DEFAULT_PAGINATION_CLASS': 'apps.[app_name].pagination.CustomPagination',
    'PAGE_SIZE': 20,
    
    # Custom exception handler (if needed)
    'EXCEPTION_HANDLER': 'apps.core.exceptions.custom_exception_handler',
}

CACHE CONFIGURATION (if DD requires caching):
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': config('REDIS_URL', default='redis://localhost:6379/1'),
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        }
    }
}

CELERY CONFIGURATION (if DD requires async tasks):
# Celery settings
CELERY_BROKER_URL = config('CELERY_BROKER_URL', default='redis://localhost:6379/0')
CELERY_RESULT_BACKEND = config('CELERY_RESULT_BACKEND', default='redis://localhost:6379/0')
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'

LOGGING CONFIGURATION UPDATES:
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
        # ... existing handlers ...
        
        # Add new handler for DD-specific logging
        '[app_name]_file': {
            'level': 'INFO',
            'class': 'logging.FileHandler',
            'filename': 'logs/[app_name].log',
            'formatter': 'verbose',
        },
    },
    'loggers': {
        # ... existing loggers ...
        
        # Add logger for DD app
        'apps.[app_name]': {
            'handlers': ['[app_name]_file', 'console'],
            'level': 'INFO',
            'propagate': True,
        },
    },
}

2. ENVIRONMENT VARIABLES UPDATES

File: .env (and .env.example)

# Add new environment variables required by DD
[NEW_DATABASE_NAME]=[database_name]
[NEW_DATABASE_USER]=[database_user]
[NEW_DATABASE_PASSWORD]=[database_password]
[NEW_DATABASE_HOST]=localhost
[NEW_DATABASE_PORT]=5432

# External API credentials (if DD requires external services)
[EXTERNAL_API_KEY]=[api_key]
[EXTERNAL_API_SECRET]=[api_secret]
[EXTERNAL_API_BASE_URL]=https://api.example.com

# Cache settings
REDIS_URL=redis://localhost:6379/1

# Celery settings (if async tasks needed)
CELERY_BROKER_URL=redis://localhost:6379/0
CELERY_RESULT_BACKEND=redis://localhost:6379/0

# File storage settings (if DD handles file uploads)
AWS_ACCESS_KEY_ID=[aws_access_key]
AWS_SECRET_ACCESS_KEY=[aws_secret_key]
AWS_STORAGE_BUCKET_NAME=[bucket_name]
AWS_S3_REGION_NAME=[region_name]

3. REQUIREMENTS UPDATES

File: requirements/base.txt

# Add new packages required by DD
[new_package_name]==[version]  # Main package for DD functionality
django-filter>=23.0  # For advanced filtering
drf-yasg>=1.21.0  # For API documentation
celery>=5.3.0  # If async tasks needed
redis>=4.5.0  # For caching and Celery
boto3>=1.26.0  # If AWS integration needed

File: requirements/development.txt

# Add development-specific packages
[dev_package_name]==[version]  # Development tools for DD

4. DOCKER CONFIGURATION UPDATES (if using Docker)

File: docker-compose.yml

# Add new services if DD requires them
services:
  # ... existing services ...
  
  # Redis (if caching/Celery needed)
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
  
  # Celery worker (if async tasks needed)
  celery:
    build: .
    command: celery -A config worker -l info
    volumes:
      - .:/app
    depends_on:
      - db
      - redis
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - CELERY_BROKER_URL=${CELERY_BROKER_URL}
  
  # Additional database (if DD requires separate DB)
  [new_database]:
    image: postgres:15
    environment:
      POSTGRES_DB: ${NEW_DATABASE_NAME}
      POSTGRES_USER: ${NEW_DATABASE_USER}
      POSTGRES_PASSWORD: ${NEW_DATABASE_PASSWORD}
    ports:
      - "[port]:5432"
    volumes:
      - [new_database]_data:/var/lib/postgresql/data

volumes:
  # ... existing volumes ...
  redis_data:
  [new_database]_data:

5. NGINX CONFIGURATION UPDATES (if needed)

File: nginx/nginx.conf

# Add new location blocks if DD requires them
location /api/v1/[app-name]/ {
    proxy_pass http://web:8000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}

# File upload handling (if DD handles large files)
location /api/v1/[app-name]/upload/ {
    client_max_body_size 100M;
    proxy_pass http://web:8000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

### Step 7: Testing Implementation Plan

**Comprehensive testing strategy:**

```
=== TESTING IMPLEMENTATION PLAN ===

File: tests/test_[app_name]/test_models.py

MODEL TESTING:

# Imports
from django.test import TestCase
from django.core.exceptions import ValidationError
from django.db import IntegrityError
from apps.[app_name].models import [ModelName1], [ModelName2]
from django.contrib.auth.models import User

class [ModelName1]ModelTest(TestCase):
    """Test cases for [ModelName1] model"""
    
    def setUp(self):
        """Set up test data"""
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
        
        self.valid_data = {
            '[field1]': '[test_value1]',
            '[field2]': '[test_value2]',
            '[user_field]': self.user,
        }
    
    def test_model_creation_success(self):
        """Test successful model creation"""
        instance = [ModelName1].objects.create(**self.valid_data)
        
        self.assertIsNotNone(instance.id)
        self.assertEqual(instance.[field1], '[test_value1]')
        self.assertTrue(instance.is_active)
        self.assertIsNotNone(instance.created_at)
    
    def test_model_string_representation(self):
        """Test __str__ method"""
        instance = [ModelName1].objects.create(**self.valid_data)
        expected_str = f"[ModelName1]: {instance.[descriptive_field]}"
        
        self.assertEqual(str(instance), expected_str)
    
    def test_model_validation_failure(self):
        """Test model validation failures"""
        invalid_data = self.valid_data.copy()
        invalid_data['[field1]'] = ''  # Invalid value
        
        with self.assertRaises(ValidationError):
            instance = [ModelName1](**invalid_data)
            instance.full_clean()
    
    def test_unique_constraint(self):
        """Test unique constraints"""
        [ModelName1].objects.create(**self.valid_data)
        
        # Try to create duplicate
        with self.assertRaises(IntegrityError):
            [ModelName1].objects.create(**self.valid_data)
    
    def test_custom_model_methods(self):
        """Test custom model methods"""
        instance = [ModelName1].objects.create(**self.valid_data)
        
        result = instance.[custom_method]()
        self.assertEqual(result, [expected_value])

File: tests/test_[app_name]/test_serializers.py

SERIALIZER TESTING:

class [ModelName1]SerializerTest(TestCase):
    """Test cases for [ModelName1] serializers"""
    
    def setUp(self):
        """Set up test data"""
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com', 
            password='testpass123'
        )
        
        self.valid_data = {
            '[field1]': '[test_value1]',
            '[field2]': '[test_value2]',
        }
        
        self.instance = [ModelName1].objects.create(
            [user_field]=self.user,
            **self.valid_data
        )
    
    def test_read_serializer_valid_data(self):
        """Test read serializer with valid data"""
        serializer = [ModelName1]ReadSerializer(instance=self.instance)
        
        self.assertIn('[field1]', serializer.data)
        self.assertEqual(serializer.data['[field1]'], '[test_value1]')
    
    def test_write_serializer_valid_data(self):
        """Test write serializer with valid data"""
        serializer = [ModelName1]WriteSerializer(data=self.valid_data)
        
        self.assertTrue(serializer.is_valid())
        instance = serializer.save([user_field]=self.user)
        self.assertIsNotNone(instance.id)
    
    def test_write_serializer_invalid_data(self):
        """Test write serializer with invalid data"""
        invalid_data = self.valid_data.copy()
        invalid_data['[field1]'] = ''  # Invalid value
        
        serializer = [ModelName1]WriteSerializer(data=invalid_data)
        
        self.assertFalse(serializer.is_valid())
        self.assertIn('[field1]', serializer.errors)
    
    def test_custom_validation(self):
        """Test custom validation methods"""
        invalid_data = {
            '[field1]': '[invalid_value]',
            '[field2]': '[conflicting_value]',
        }
        
        serializer = [ModelName1]WriteSerializer(data=invalid_data)
        
        self.assertFalse(serializer.is_valid())
        self.assertIn('[validation_error_field]', serializer.errors)

File: tests/test_[app_name]/test_views.py

VIEW TESTING:

from rest_framework.test import APITestCase
from rest_framework import status
from django.urls import reverse

class [ModelName1]ViewSetTest(APITestCase):
    """Test cases for [ModelName1] ViewSet"""
    
    def setUp(self):
        """Set up test data"""
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
        
        self.instance = [ModelName1].objects.create(
            [field1]='[test_value1]',
            [field2]='[test_value2]',
            [user_field]=self.user
        )
        
        # Authenticate user
        self.client.force_authenticate(user=self.user)
    
    def test_list_endpoint(self):
        """Test GET /api/v1/[app-name]/[model-name-plural]/"""
        url = reverse('[app_name]:[model-name]-list')
        response = self.client.get(url)
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data['results']), 1)
    
    def test_create_endpoint(self):
        """Test POST /api/v1/[app-name]/[model-name-plural]/"""
        url = reverse('[app_name]:[model-name]-list')
        data = {
            '[field1]': '[new_test_value1]',
            '[field2]': '[new_test_value2]',
        }
        
        response = self.client.post(url, data, format='json')
        
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(response.data['[field1]'], '[new_test_value1]')
    
    def test_retrieve_endpoint(self):
        """Test GET /api/v1/[app-name]/[model-name-plural]/{id}/"""
        url = reverse('[app_name]:[model-name]-detail', kwargs={'pk': self.instance.pk})
        response = self.client.get(url)
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.data['id'], self.instance.pk)
    
    def test_update_endpoint(self):
        """Test PUT /api/v1/[app-name]/[model-name-plural]/{id}/"""
        url = reverse('[app_name]:[model-name]-detail', kwargs={'pk': self.instance.pk})
        data = {
            '[field1]': '[updated_value1]',
            '[field2]': '[updated_value2]',
        }
        
        response = self.client.put(url, data, format='json')
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.data['[field1]'], '[updated_value1]')
    
    def test_delete_endpoint(self):
        """Test DELETE /api/v1/[app-name]/[model-name-plural]/{id}/"""
        url = reverse('[app_name]:[model-name]-detail', kwargs={'pk': self.instance.pk})
        response = self.client.delete(url)
        
        self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)
        
        # Verify soft delete
        self.instance.refresh_from_db()
        self.assertFalse(self.instance.is_active)
    
    def test_custom_action(self):
        """Test POST /api/v1/[app-name]/[model-name-plural]/{id}/[custom-action]/"""
        url = reverse('[app_name]:[model-name]-[custom-action]', kwargs={'pk': self.instance.pk})
        data = {'[parameter]': '[test_value]'}
        
        response = self.client.post(url, data, format='json')
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertTrue(response.data['success'])
    
    def test_permission_required(self):
        """Test that authentication is required"""
        self.client.force_authenticate(user=None)  # Remove authentication
        
        url = reverse('[app_name]:[model-name]-list')
        response = self.client.get(url)
        
        self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)

File: tests/test_[app_name]/test_integration.py

INTEGRATION TESTING:

class [ModelName1]IntegrationTest(APITestCase):
    """Integration tests for complete [ModelName1] workflows"""
    
    def test_complete_crud_workflow(self):
        """Test complete CRUD workflow"""
        # Setup
        user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
        self.client.force_authenticate(user=user)
        
        # CREATE
        create_url = reverse('[app_name]:[model-name]-list')
        create_data = {
            '[field1]': '[test_value1]',
            '[field2]': '[test_value2]',
        }
        
        create_response = self.client.post(create_url, create_data, format='json')
        self.assertEqual(create_response.status_code, status.HTTP_201_CREATED)
        
        created_id = create_response.data['id']
        
        # READ
        detail_url = reverse('[app_name]:[model-name]-detail', kwargs={'pk': created_id})
        read_response = self.client.get(detail_url)
        self.assertEqual(read_response.status_code, status.HTTP_200_OK)
        
        # UPDATE
        update_data = {
            '[field1]': '[updated_value1]',
            '[field2]': '[updated_value2]',
        }
        update_response = self.client.put(detail_url, update_data, format='json')
        self.assertEqual(update_response.status_code, status.HTTP_200_OK)
        
        # DELETE
        delete_response = self.client.delete(detail_url)
        self.assertEqual(delete_response.status_code, status.HTTP_204_NO_CONTENT)
    
    def test_business_workflow(self):
        """Test specific business workflow"""
        # Test complete business process from start to finish
        # This would include multiple API calls, state changes, etc.
        pass
```

### Step 8: Implementation Execution Plan

**Step-by-step execution order:**

```
=== IMPLEMENTATION EXECUTION PLAN ===

PHASE 1: Foundation Setup (Estimated: 1-2 days)
├── Step 1.1: Create Django App
│   ├── Command: python manage.py startapp [app_name] apps/[app_name]
│   ├── Files Created: apps/[app_name]/ directory structure
│   └── Verification: App directory exists with standard Django files
├── Step 1.2: Update Settings
│   ├── File: config/settings/base.py
│   ├── Changes: Add app to INSTALLED_APPS, update configurations
│   └── Verification: python manage.py check passes
├── Step 1.3: Environment Setup
│   ├── File: .env and .env.example
│   ├── Changes: Add new environment variables
│   └── Verification: Settings load without errors
└── Step 1.4: Dependencies Installation
    ├── File: requirements/base.txt
    ├── Command: pip install -r requirements/base.txt
    └── Verification: All packages install successfully

PHASE 2: Database Implementation (Estimated: 2-3 days)
├── Step 2.1: Create Models
│   ├── File: apps/[app_name]/models.py
│   ├── Implementation: Follow models implementation plan
│   └── Verification: python manage.py check passes
├── Step 2.2: Create Initial Migration
│   ├── Command: python manage.py makemigrations [app_name]
│   ├── File Generated: migration file with model creation
│   └── Verification: python manage.py sqlmigrate [app_name] 000X
├── Step 2.3: Apply Migration
│   ├── Command: python manage.py migrate
│   ├── Database: Tables created in database
│   └── Verification: Models can be imported and used
└── Step 2.4: Test Models
    ├── File: tests/test_[app_name]/test_models.py
    ├── Command: python manage.py test tests.test_[app_name].test_models
    └── Verification: All model tests pass

PHASE 3: API Implementation (Estimated: 3-4 days)
├── Step 3.1: Create Serializers
│   ├── File: apps/[app_name]/serializers.py
│   ├── Implementation: Follow serializers implementation plan
│   └── Verification: Serializers can be imported
├── Step 3.2: Create Views
│   ├── File: apps/[app_name]/views.py
│   ├── Implementation: Follow views implementation plan
│   └── Verification: Views can be imported
├── Step 3.3: Configure URLs
│   ├── File: apps/[app_name]/urls.py
│   ├── Implementation: Follow URL configuration plan
│   └── Verification: python manage.py show_urls includes new endpoints
├── Step 3.4: Update Main URLs
│   ├── File: config/urls.py
│   ├── Changes: Include app URLs
│   └── Verification: All URLs accessible
└── Step 3.5: Test API Endpoints
    ├── File: tests/test_[app_name]/test_views.py
    ├── Command: python manage.py test tests.test_[app_name].test_views
    └── Verification: All API tests pass

PHASE 4: Business Logic Implementation (Estimated: 2-3 days)
├── Step 4.1: Custom Model Methods
│   ├── File: apps/[app_name]/models.py
│   ├── Implementation: Add custom business logic methods
│   └── Verification: Methods work as expected
├── Step 4.2: Service Layer (if needed)
│   ├── File: apps/[app_name]/services.py
│   ├── Implementation: Business logic functions
│   └── Verification: Services integrate with views
├── Step 4.3: Custom Permissions
│   ├── File: apps/[app_name]/permissions.py
│   ├── Implementation: Custom permission classes
│   └── Verification: Permissions enforce correctly
└── Step 4.4: Custom Filters
    ├── File: apps/[app_name]/filters.py
    ├── Implementation: Custom filtering logic
    └── Verification: Filters work in API endpoints

PHASE 5: Testing and Quality Assurance (Estimated: 2-3 days)
├── Step 5.1: Unit Tests
│   ├── Files: tests/test_[app_name]/*.py
│   ├── Command: python manage.py test tests.test_[app_name]
│   └── Target: 90%+ code coverage
├── Step 5.2: Integration Tests
│   ├── File: tests/test_[app_name]/test_integration.py
│   ├── Command: python manage.py test tests.test_[app_name].test_integration
│   └── Verification: End-to-end workflows work
├── Step 5.3: API Documentation
│   ├── Update: Swagger/OpenAPI documentation
│   ├── Verification: /swagger/ shows new endpoints
│   └── Manual Testing: Test all endpoints manually
└── Step 5.4: Performance Testing
    ├── Tool: Use load testing tools
    ├── Verification: Response times within acceptable limits
    └── Optimization: Query optimization if needed

PHASE 6: Deployment Preparation (Estimated: 1-2 days)
├── Step 6.1: Production Configuration
│   ├── File: config/settings/production.py
│   ├── Updates: Production-specific settings
│   └── Verification: Settings work in production environment
├── Step 6.2: Migration Scripts
│   ├── Files: Database migration files
│   ├── Testing: Test migrations on production-like data
│   └── Verification: Migrations run without data loss
├── Step 6.3: Deployment Documentation
│   ├── File: docs/deployment.md
│   ├── Content: Step-by-step deployment instructions
│   └── Verification: Documentation is complete and accurate
└── Step 6.4: Rollback Plan
    ├── Document: Rollback procedures
    ├── Testing: Test rollback scenarios
    └── Verification: Rollback procedures work correctly

TOTAL ESTIMATED TIME: 11-17 days
RECOMMENDED BUFFER: +20% (3-4 additional days)
FINAL ESTIMATE: 14-21 days
```

## Output Generation

**Create the standardized implementation plan using the template structure:**

```python
# Generate implementation plan following template structure
import json
from datetime import datetime
from pathlib import Path

def generate_implementation_plan():
    """Generate standardized implementation plan"""
    
    # Create output directory structure
    base_dir = Path('dd_implementation_plan')
    base_dir.mkdir(exist_ok=True)
    
    # Create subdirectories
    for subdir in ['metadata', 'analysis', 'implementation', 'timeline', 'documentation', 'validation']:
        (base_dir / subdir).mkdir(exist_ok=True)
    
    # Generate main implementation plan JSON
    implementation_plan = {
        "metadata": {
            "dd_name": "${input:dd_name}",
            "project_name": "${input:project_name}",
            "app_name": "[determined_app_name]",
            "generated_at": datetime.now().isoformat(),
            "generated_by": "prompt_04",
            "version": "1.0"
        },
        "dd_summary": {
            "description": "[DD description from impact assessment]",
            "business_value": "[business value statement]",
            "type": "[feature|enhancement|bugfix]",
            "priority": "[critical|high|medium|low]",
            "estimated_effort": "[estimated development time]",
            "target_completion": "[target completion date]"
        },
        "requirements_analysis": {
            "api_endpoints": "[list of endpoints to implement]",
            "data_models": "[list of models to create/modify]",
            "business_rules": "[list of business rules to implement]",
            "authentication_requirements": "[auth requirements]",
            "performance_requirements": "[performance criteria]",
            "integration_requirements": "[external integrations needed]"
        },
        "implementation_components": {
            "models": "[detailed model implementation plan]",
            "serializers": "[serializer implementation plan]",
            "views": "[view implementation plan]",
            "urls": "[URL configuration plan]",
            "migrations": "[migration sequence plan]",
            "tests": "[testing implementation plan]",
            "configurations": "[config changes needed]"
        },
        "implementation_plan": {
            "phases": "[implementation phases with timeline]",
            "dependencies": "[technical dependencies]",
            "risks": "[identified risks and mitigation]",
            "mitigation_strategies": "[risk mitigation approaches]"
        },
        "timeline": {
            "estimated_duration": "[total estimated time]",
            "milestones": "[key milestones and dates]",
            "critical_path": "[critical path items]",
            "buffer_time": "[recommended buffer time]"
        },
        "validation_criteria": {
            "acceptance_criteria": "[criteria for completion]",
            "test_scenarios": "[test scenarios to validate]",
            "success_metrics": "[metrics to measure success]",
            "rollback_criteria": "[when to rollback changes]"
        }
    }
    
    # Save main implementation plan
    with open(base_dir / 'implementation' / 'implementation_plan.json', 'w') as f:
        json.dump(implementation_plan, f, indent=2)
    
    # Generate supporting documentation
    generate_implementation_documentation(base_dir)
    
    print(f"✅ Implementation plan generated in {base_dir}/")
    print(f"📄 Main file: {base_dir}/implementation/implementation_plan.json")
    print(f"📚 Documentation: {base_dir}/documentation/")
    
    return implementation_plan

def generate_implementation_documentation(base_dir):
    """Generate markdown documentation files"""
    
    # Generate implementation plan markdown
    plan_md = f"""
# Implementation Plan: ${{input:dd_name}}

## Overview
[Implementation overview and summary]

## Technical Architecture
[Architecture decisions and design patterns]

## Implementation Phases
[Detailed breakdown of implementation phases]

## Risk Assessment
[Risk analysis and mitigation strategies]

## Success Criteria
[Criteria for successful implementation]
"""
    
    with open(base_dir / 'documentation' / 'implementation_plan.md', 'w') as f:
        f.write(plan_md)

# Execute plan generation
generate_implementation_plan()
```

## Template Validation

**Ensure output meets template requirements:**

✅ **Metadata Section**: Complete with DD name, project info, generation timestamp  
✅ **Folder Structure**: Follows standardized template with metadata/, analysis/, implementation/, etc.  
✅ **JSON Schema**: Matches template structure for implementation planning  
✅ **Documentation**: Includes both JSON and markdown formats  
✅ **Validation**: Contains acceptance criteria and success metrics  
✅ **Timeline**: Detailed timeline with milestones and dependencies  
✅ **Cross-Prompt Compatibility**: Output format compatible with prompt 05 input requirements

---

**Note**: This implementation plan provides the detailed technical roadmap for prompt 05 to execute the actual code generation and implementation work.
```

### Step 3: Serializers Implementation Plan

<function_calls>
<invoke name="TodoWrite">
<parameter name="todos">[{"content": "Create 04_django_dd_implementation_plan.prompt.md for detailed coding implementation plan", "status": "completed", "priority": "high", "id": "1"}, {"content": "Design migration code generation plan and database schema changes", "status": "completed", "priority": "high", "id": "2"}, {"content": "Create models implementation plan with relationships and validation", "status": "completed", "priority": "high", "id": "3"}, {"content": "Design serializers implementation plan with validation and nested objects", "status": "in_progress", "priority": "high", "id": "4"}, {"content": "Create views implementation plan with authentication and business logic", "status": "pending", "priority": "high", "id": "5"}, {"content": "Design URL routing and configuration changes implementation plan", "status": "pending", "priority": "high", "id": "6"}]