---
description: Automated Django code generation based on DD implementation plan from prompt 04
---

# Django Design Detail (DD) Automated Code Generator

This prompt automatically generates all Django code based on the implementation plan created in prompt 04, including git branch management, models, migrations, serializers, views, URLs, configurations, and tests.

## Prerequisites

**Required Input:**
- 📋 **Implementation Plan**: Output from `04_django_dd_implementation_plan.prompt.md`
- 📦 **Project Name**: `${input:project_name}`
- 📁 **DD Name**: `${input:dd_name}`
- 📄 **Implementation Plan File**: `${input:implementation_plan_path}`
- 🏷️ **Feature Branch Name**: `${input:feature_branch_name}` (e.g., `feature/dd-user-management`)

## Automated Code Generation Workflow

### Step 1: Git Branch Setup

**Create and checkout feature branch for DD implementation:**

```bash
# Ensure we're on the main branch with latest changes
echo "=== SETTING UP GIT BRANCH FOR ${input:dd_name} ==="
git checkout main || git checkout master
git pull origin main || git pull origin master

# Create feature branch
echo "Creating feature branch: ${input:feature_branch_name}"
git checkout -b ${input:feature_branch_name}

# Verify branch creation
echo "Current branch:"
git branch --show-current

# Create initial commit message
echo "Starting implementation of ${input:dd_name} Design Detail" > .git/COMMIT_EDITMSG
```

### Step 2: Django App Creation

**Create new Django app if required:**

```bash
# Parse implementation plan to determine app name
APP_NAME=$(grep -o "apps/\[.*\]" ${input:implementation_plan_path} | head -1 | sed 's/apps\/\[\(.*\)\]/\1/')

# Create Django app
echo "=== CREATING DJANGO APP: $APP_NAME ==="
python manage.py startapp $APP_NAME apps/$APP_NAME

# Create additional directories
mkdir -p apps/$APP_NAME/migrations
mkdir -p apps/$APP_NAME/management/commands
mkdir -p tests/test_$APP_NAME

# Create __init__.py files
touch apps/$APP_NAME/__init__.py
touch apps/$APP_NAME/migrations/__init__.py
touch tests/test_$APP_NAME/__init__.py

echo "✅ Django app created: apps/$APP_NAME"
```

### Step 3: Models Implementation

**Generate models.py based on implementation plan:**

```python
# File: apps/$APP_NAME/models.py

"""
Models for ${input:dd_name} implementation.
Auto-generated from DD implementation plan.
"""

from django.db import models
from django.contrib.auth.models import User
from django.core.validators import MinValueValidator, MaxValueValidator
from django.utils.translation import gettext_lazy as _
from apps.core.models import BaseModel  # Assuming BaseModel exists
import uuid


# Model 1: Primary Model Implementation
class [ModelName1](BaseModel):
    """
    [Extract model description from implementation plan]
    
    This model handles [business purpose].
    """
    
    # Primary Key (if using UUID)
    id = models.UUIDField(
        primary_key=True,
        default=uuid.uuid4,
        editable=False,
        help_text=_("Unique identifier")
    )
    
    # Core Fields
    name = models.CharField(
        max_length=255,
        help_text=_("Name of the [model]")
    )
    
    description = models.TextField(
        blank=True,
        null=True,
        help_text=_("Detailed description")
    )
    
    # Status Field with Choices
    class Status(models.TextChoices):
        ACTIVE = 'active', _('Active')
        INACTIVE = 'inactive', _('Inactive')
        PENDING = 'pending', _('Pending')
        ARCHIVED = 'archived', _('Archived')
    
    status = models.CharField(
        max_length=20,
        choices=Status.choices,
        default=Status.ACTIVE,
        help_text=_("Current status")
    )
    
    # Foreign Key Relationships
    owner = models.ForeignKey(
        User,
        on_delete=models.PROTECT,
        related_name='owned_[model_name_plural]',
        help_text=_("Owner of this [model]")
    )
    
    # Many-to-Many Relationships
    tags = models.ManyToManyField(
        'Tag',
        blank=True,
        related_name='[model_name_plural]',
        help_text=_("Associated tags")
    )
    
    # Numeric Fields
    priority = models.IntegerField(
        default=0,
        validators=[MinValueValidator(0), MaxValueValidator(100)],
        help_text=_("Priority level (0-100)")
    )
    
    # Date Fields
    start_date = models.DateField(
        null=True,
        blank=True,
        help_text=_("Start date")
    )
    
    end_date = models.DateField(
        null=True,
        blank=True,
        help_text=_("End date")
    )
    
    # JSON Field for flexible data
    metadata = models.JSONField(
        default=dict,
        blank=True,
        help_text=_("Additional metadata")
    )
    
    class Meta:
        db_table = '[app_name]_[model_name]'
        ordering = ['-created_at', 'name']
        verbose_name = _('[Model Name]')
        verbose_name_plural = _('[Model Names]')
        
        indexes = [
            models.Index(fields=['status', '-created_at'], name='idx_[model]_status_created'),
            models.Index(fields=['owner', 'status'], name='idx_[model]_owner_status'),
        ]
        
        constraints = [
            models.UniqueConstraint(
                fields=['owner', 'name'],
                name='uniq_[model]_owner_name'
            ),
            models.CheckConstraint(
                check=models.Q(end_date__gte=models.F('start_date')) | models.Q(end_date__isnull=True),
                name='chk_[model]_date_order'
            ),
        ]
    
    def __str__(self):
        return f"{self.name} ({self.get_status_display()})"
    
    def save(self, *args, **kwargs):
        """Override save to implement business logic"""
        self.full_clean()
        super().save(*args, **kwargs)
    
    def is_active(self):
        """Check if the model instance is currently active"""
        return self.status == self.Status.ACTIVE
    
    @property
    def duration(self):
        """Calculate duration between start and end date"""
        if self.start_date and self.end_date:
            return (self.end_date - self.start_date).days
        return None
    
    @classmethod
    def get_active_objects(cls):
        """Return only active objects"""
        return cls.objects.filter(status=cls.Status.ACTIVE)


# Model 2: Supporting Model
class Tag(models.Model):
    """Tag model for categorization"""
    
    name = models.CharField(
        max_length=50,
        unique=True,
        help_text=_("Tag name")
    )
    
    slug = models.SlugField(
        unique=True,
        help_text=_("URL-friendly tag identifier")
    )
    
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        db_table = '[app_name]_tags'
        ordering = ['name']
    
    def __str__(self):
        return self.name


# Model 3: Through Model for M2M with extra fields
class [ModelName1]Assignment(models.Model):
    """
    Through model for assigning users to [ModelName1] with roles
    """
    
    [model_name] = models.ForeignKey(
        [ModelName1],
        on_delete=models.CASCADE,
        related_name='assignments'
    )
    
    user = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='[model_name]_assignments'
    )
    
    class Role(models.TextChoices):
        VIEWER = 'viewer', _('Viewer')
        EDITOR = 'editor', _('Editor')
        ADMIN = 'admin', _('Administrator')
    
    role = models.CharField(
        max_length=20,
        choices=Role.choices,
        default=Role.VIEWER
    )
    
    assigned_at = models.DateTimeField(auto_now_add=True)
    assigned_by = models.ForeignKey(
        User,
        on_delete=models.SET_NULL,
        null=True,
        related_name='assignments_made'
    )
    
    class Meta:
        db_table = '[app_name]_[model_name]_assignments'
        unique_together = [('[model_name]', 'user')]
    
    def __str__(self):
        return f"{self.user} - {self.get_role_display()} on {self.[model_name]}"


# Custom Manager
class [ModelName1]Manager(models.Manager):
    """Custom manager with business logic queries"""
    
    def active(self):
        """Return only active objects"""
        return self.filter(status=[ModelName1].Status.ACTIVE)
    
    def owned_by(self, user):
        """Return objects owned by specific user"""
        return self.filter(owner=user)
    
    def with_assignments(self):
        """Prefetch related assignments"""
        return self.prefetch_related('assignments__user')


# Apply custom manager to model
# Add this line to [ModelName1] class:
# objects = [ModelName1]Manager()
```

**Generate the models file:**

```bash
# Create models.py with the generated content
cat > apps/$APP_NAME/models.py << 'EOF'
[Insert generated models code from above]
EOF

echo "✅ Models created: apps/$APP_NAME/models.py"
```

### Step 4: Database Migrations

**Create and apply migrations:**

```bash
echo "=== CREATING DATABASE MIGRATIONS ==="

# Create initial migration
python manage.py makemigrations $APP_NAME -n initial_models

# Show migration SQL for review
echo "Migration SQL preview:"
python manage.py sqlmigrate $APP_NAME 0001

# Apply migrations
echo "Applying migrations..."
python manage.py migrate

echo "✅ Migrations created and applied"
```

**Generate custom migration for data or constraints:**

```python
# File: apps/$APP_NAME/migrations/0002_add_initial_data.py

from django.db import migrations

def create_initial_tags(apps, schema_editor):
    """Create initial tag data"""
    Tag = apps.get_model('$APP_NAME', 'Tag')
    
    initial_tags = [
        {'name': 'Important', 'slug': 'important'},
        {'name': 'Urgent', 'slug': 'urgent'},
        {'name': 'Review', 'slug': 'review'},
    ]
    
    for tag_data in initial_tags:
        Tag.objects.get_or_create(**tag_data)

def reverse_initial_tags(apps, schema_editor):
    """Remove initial tags"""
    Tag = apps.get_model('$APP_NAME', 'Tag')
    Tag.objects.filter(slug__in=['important', 'urgent', 'review']).delete()

class Migration(migrations.Migration):
    dependencies = [
        ('$APP_NAME', '0001_initial_models'),
    ]
    
    operations = [
        migrations.RunPython(create_initial_tags, reverse_initial_tags),
    ]
```

### Step 5: Serializers Implementation

**Generate serializers.py:**

```python
# File: apps/$APP_NAME/serializers.py

"""
Serializers for ${input:dd_name} implementation.
Auto-generated from DD implementation plan.
"""

from rest_framework import serializers
from django.contrib.auth.models import User
from django.db import transaction
from .models import [ModelName1], Tag, [ModelName1]Assignment


# Tag Serializer
class TagSerializer(serializers.ModelSerializer):
    """Serializer for Tag model"""
    
    class Meta:
        model = Tag
        fields = ['id', 'name', 'slug']
        read_only_fields = ['id']


# User Serializer (for nested representation)
class UserLightSerializer(serializers.ModelSerializer):
    """Light user serializer for nested representations"""
    
    class Meta:
        model = User
        fields = ['id', 'username', 'email', 'first_name', 'last_name']
        read_only_fields = ['id']


# Assignment Serializer
class [ModelName1]AssignmentSerializer(serializers.ModelSerializer):
    """Serializer for model assignments"""
    
    user = UserLightSerializer(read_only=True)
    user_id = serializers.IntegerField(write_only=True)
    assigned_by = UserLightSerializer(read_only=True)
    
    class Meta:
        model = [ModelName1]Assignment
        fields = [
            'id', 'user', 'user_id', 'role',
            'assigned_at', 'assigned_by'
        ]
        read_only_fields = ['id', 'assigned_at', 'assigned_by']
    
    def validate_user_id(self, value):
        """Validate user exists"""
        try:
            User.objects.get(pk=value)
        except User.DoesNotExist:
            raise serializers.ValidationError("User not found")
        return value


# Read Serializer (for GET requests)
class [ModelName1]ReadSerializer(serializers.ModelSerializer):
    """
    Read serializer with nested objects and computed fields
    """
    
    owner = UserLightSerializer(read_only=True)
    tags = TagSerializer(many=True, read_only=True)
    assignments = [ModelName1]AssignmentSerializer(many=True, read_only=True)
    status_display = serializers.CharField(source='get_status_display', read_only=True)
    duration = serializers.IntegerField(read_only=True)
    
    class Meta:
        model = [ModelName1]
        fields = [
            'id', 'name', 'description', 'status', 'status_display',
            'owner', 'tags', 'priority', 'start_date', 'end_date',
            'duration', 'metadata', 'assignments',
            'created_at', 'updated_at'
        ]
        read_only_fields = ['id', 'created_at', 'updated_at']


# Write Serializer (for POST/PUT/PATCH requests)
class [ModelName1]WriteSerializer(serializers.ModelSerializer):
    """
    Write serializer with validation and business logic
    """
    
    tags = serializers.PrimaryKeyRelatedField(
        many=True,
        queryset=Tag.objects.all(),
        required=False
    )
    
    owner_id = serializers.IntegerField(
        required=False,
        help_text="Owner user ID (defaults to current user)"
    )
    
    class Meta:
        model = [ModelName1]
        fields = [
            'name', 'description', 'status', 'owner_id',
            'tags', 'priority', 'start_date', 'end_date',
            'metadata'
        ]
    
    def validate_name(self, value):
        """Validate name uniqueness for owner"""
        request = self.context.get('request')
        if request and request.user:
            # Check uniqueness only for the same owner
            qs = [ModelName1].objects.filter(
                owner=request.user,
                name=value
            )
            if self.instance:
                qs = qs.exclude(pk=self.instance.pk)
            if qs.exists():
                raise serializers.ValidationError(
                    "You already have an item with this name"
                )
        return value
    
    def validate(self, attrs):
        """Object-level validation"""
        
        # Date validation
        start_date = attrs.get('start_date')
        end_date = attrs.get('end_date')
        
        if start_date and end_date and end_date < start_date:
            raise serializers.ValidationError({
                'end_date': 'End date must be after start date'
            })
        
        # Priority validation based on status
        status = attrs.get('status', self.instance.status if self.instance else None)
        priority = attrs.get('priority', self.instance.priority if self.instance else 0)
        
        if status == [ModelName1].Status.ARCHIVED and priority > 0:
            raise serializers.ValidationError({
                'priority': 'Archived items must have priority 0'
            })
        
        return attrs
    
    def create(self, validated_data):
        """Create with transaction and related objects"""
        
        tags = validated_data.pop('tags', [])
        
        # Set owner to current user if not specified
        if 'owner_id' not in validated_data:
            request = self.context.get('request')
            if request and request.user:
                validated_data['owner'] = request.user
        else:
            validated_data['owner_id'] = validated_data.pop('owner_id')
        
        with transaction.atomic():
            instance = super().create(validated_data)
            
            # Set tags
            if tags:
                instance.tags.set(tags)
            
            # Create default assignment for owner
            [ModelName1]Assignment.objects.create(
                [model_name]=instance,
                user=instance.owner,
                role=[ModelName1]Assignment.Role.ADMIN,
                assigned_by=instance.owner
            )
        
        return instance
    
    def update(self, instance, validated_data):
        """Update with transaction"""
        
        tags = validated_data.pop('tags', None)
        
        with transaction.atomic():
            instance = super().update(instance, validated_data)
            
            # Update tags if provided
            if tags is not None:
                instance.tags.set(tags)
        
        return instance


# Bulk Action Serializer
class [ModelName1]BulkActionSerializer(serializers.Serializer):
    """Serializer for bulk actions"""
    
    ids = serializers.ListField(
        child=serializers.UUIDField(),
        help_text="List of IDs to process"
    )
    
    action = serializers.ChoiceField(
        choices=['activate', 'archive', 'delete'],
        help_text="Action to perform"
    )
    
    def validate_ids(self, value):
        """Validate all IDs exist and user has permission"""
        request = self.context.get('request')
        if not request or not request.user:
            raise serializers.ValidationError("Authentication required")
        
        # Check objects exist and user has permission
        objects = [ModelName1].objects.filter(id__in=value)
        
        if objects.count() != len(value):
            raise serializers.ValidationError("Some IDs not found")
        
        # Check permissions
        for obj in objects:
            if obj.owner != request.user and not request.user.is_staff:
                raise serializers.ValidationError(
                    f"No permission for object {obj.id}"
                )
        
        return value


# Filter Serializer
class [ModelName1]FilterSerializer(serializers.Serializer):
    """Serializer for filtering options"""
    
    status = serializers.MultipleChoiceField(
        choices=[ModelName1].Status.choices,
        required=False
    )
    
    owner_id = serializers.IntegerField(required=False)
    
    tags = serializers.ListField(
        child=serializers.IntegerField(),
        required=False
    )
    
    priority_min = serializers.IntegerField(
        required=False,
        min_value=0,
        max_value=100
    )
    
    priority_max = serializers.IntegerField(
        required=False,
        min_value=0,
        max_value=100
    )
    
    date_from = serializers.DateField(required=False)
    date_to = serializers.DateField(required=False)
    
    search = serializers.CharField(
        required=False,
        help_text="Search in name and description"
    )
    
    ordering = serializers.ChoiceField(
        choices=[
            'created_at', '-created_at',
            'updated_at', '-updated_at',
            'name', '-name',
            'priority', '-priority'
        ],
        required=False,
        default='-created_at'
    )
```

**Generate the serializers file:**

```bash
# Create serializers.py
cat > apps/$APP_NAME/serializers.py << 'EOF'
[Insert generated serializers code from above]
EOF

echo "✅ Serializers created: apps/$APP_NAME/serializers.py"
```

### Step 6: Views Implementation

**Generate views.py:**

```python
# File: apps/$APP_NAME/views.py

"""
Views for ${input:dd_name} implementation.
Auto-generated from DD implementation plan.
"""

from rest_framework import viewsets, status, filters
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from django_filters.rest_framework import DjangoFilterBackend
from django.db.models import Q, Count, Avg
from django.shortcuts import get_object_or_404
from drf_yasg.utils import swagger_auto_schema
from drf_yasg import openapi

from .models import [ModelName1], Tag, [ModelName1]Assignment
from .serializers import (
    [ModelName1]ReadSerializer, [ModelName1]WriteSerializer,
    [ModelName1]BulkActionSerializer, [ModelName1]FilterSerializer,
    TagSerializer, [ModelName1]AssignmentSerializer
)
from .permissions import Is[ModelName1]OwnerOrReadOnly
from .filters import [ModelName1]Filter


class [ModelName1]ViewSet(viewsets.ModelViewSet):
    """
    ViewSet for [ModelName1] with full CRUD operations and custom actions
    """
    
    permission_classes = [IsAuthenticated, Is[ModelName1]OwnerOrReadOnly]
    filter_backends = [DjangoFilterBackend, filters.SearchFilter, filters.OrderingFilter]
    filterset_class = [ModelName1]Filter
    search_fields = ['name', 'description']
    ordering_fields = ['created_at', 'updated_at', 'name', 'priority']
    ordering = ['-created_at']
    
    def get_queryset(self):
        """
        Get queryset with optimizations and user filtering
        """
        queryset = [ModelName1].objects.all()
        
        # Optimize with select/prefetch related
        queryset = queryset.select_related('owner')
        queryset = queryset.prefetch_related('tags', 'assignments__user')
        
        # Filter by user if not staff
        if not self.request.user.is_staff:
            # Show items where user is owner or has assignment
            queryset = queryset.filter(
                Q(owner=self.request.user) |
                Q(assignments__user=self.request.user)
            ).distinct()
        
        return queryset
    
    def get_serializer_class(self):
        """Return appropriate serializer based on action"""
        if self.action in ['list', 'retrieve']:
            return [ModelName1]ReadSerializer
        return [ModelName1]WriteSerializer
    
    @swagger_auto_schema(
        method='post',
        request_body=[ModelName1]BulkActionSerializer,
        responses={
            200: openapi.Response(
                description="Bulk action completed",
                examples={
                    "application/json": {
                        "success": True,
                        "processed": 5,
                        "message": "5 items archived successfully"
                    }
                }
            )
        }
    )
    @action(detail=False, methods=['post'])
    def bulk_action(self, request):
        """
        Perform bulk actions on multiple items
        """
        serializer = [ModelName1]BulkActionSerializer(
            data=request.data,
            context={'request': request}
        )
        serializer.is_valid(raise_exception=True)
        
        ids = serializer.validated_data['ids']
        action = serializer.validated_data['action']
        
        queryset = self.get_queryset().filter(id__in=ids)
        count = queryset.count()
        
        if action == 'activate':
            queryset.update(status=[ModelName1].Status.ACTIVE)
            message = f"{count} items activated successfully"
        elif action == 'archive':
            queryset.update(status=[ModelName1].Status.ARCHIVED)
            message = f"{count} items archived successfully"
        elif action == 'delete':
            queryset.delete()
            message = f"{count} items deleted successfully"
        
        return Response({
            'success': True,
            'processed': count,
            'message': message
        })
    
    @swagger_auto_schema(
        method='get',
        manual_parameters=[
            openapi.Parameter(
                'include_archived',
                openapi.IN_QUERY,
                description="Include archived items",
                type=openapi.TYPE_BOOLEAN
            )
        ]
    )
    @action(detail=False, methods=['get'])
    def statistics(self, request):
        """
        Get statistics for the current user's items
        """
        queryset = self.get_queryset()
        
        # Apply filters
        include_archived = request.query_params.get('include_archived', 'false').lower() == 'true'
        if not include_archived:
            queryset = queryset.exclude(status=[ModelName1].Status.ARCHIVED)
        
        # Calculate statistics
        stats = {
            'total': queryset.count(),
            'by_status': dict(
                queryset.values('status').annotate(count=Count('id')).values_list('status', 'count')
            ),
            'average_priority': queryset.aggregate(avg=Avg('priority'))['avg'] or 0,
            'with_assignments': queryset.filter(assignments__isnull=False).distinct().count(),
            'my_items': queryset.filter(owner=request.user).count() if request.user.is_authenticated else 0
        }
        
        return Response(stats)
    
    @action(detail=True, methods=['post'])
    def duplicate(self, request, pk=None):
        """
        Duplicate an existing item
        """
        original = self.get_object()
        
        # Create duplicate
        duplicate = [ModelName1].objects.create(
            name=f"{original.name} (Copy)",
            description=original.description,
            status=[ModelName1].Status.ACTIVE,
            owner=request.user,
            priority=original.priority,
            metadata=original.metadata
        )
        
        # Copy tags
        duplicate.tags.set(original.tags.all())
        
        # Create owner assignment
        [ModelName1]Assignment.objects.create(
            [model_name]=duplicate,
            user=request.user,
            role=[ModelName1]Assignment.Role.ADMIN,
            assigned_by=request.user
        )
        
        serializer = [ModelName1]ReadSerializer(duplicate)
        return Response(serializer.data, status=status.HTTP_201_CREATED)
    
    @action(detail=True, methods=['get', 'post', 'delete'], url_path='assignments')
    def manage_assignments(self, request, pk=None):
        """
        Manage user assignments for an item
        """
        item = self.get_object()
        
        if request.method == 'GET':
            # List assignments
            assignments = item.assignments.all()
            serializer = [ModelName1]AssignmentSerializer(assignments, many=True)
            return Response(serializer.data)
        
        elif request.method == 'POST':
            # Create assignment
            serializer = [ModelName1]AssignmentSerializer(data=request.data)
            serializer.is_valid(raise_exception=True)
            
            assignment = [ModelName1]Assignment.objects.create(
                [model_name]=item,
                user_id=serializer.validated_data['user_id'],
                role=serializer.validated_data['role'],
                assigned_by=request.user
            )
            
            serializer = [ModelName1]AssignmentSerializer(assignment)
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        
        elif request.method == 'DELETE':
            # Remove assignment
            user_id = request.data.get('user_id')
            if not user_id:
                return Response(
                    {'error': 'user_id required'},
                    status=status.HTTP_400_BAD_REQUEST
                )
            
            assignment = get_object_or_404(
                item.assignments,
                user_id=user_id
            )
            assignment.delete()
            
            return Response(status=status.HTTP_204_NO_CONTENT)


class TagViewSet(viewsets.ModelViewSet):
    """ViewSet for Tag management"""
    
    queryset = Tag.objects.all()
    serializer_class = TagSerializer
    permission_classes = [IsAuthenticated]
    filter_backends = [filters.SearchFilter, filters.OrderingFilter]
    search_fields = ['name']
    ordering_fields = ['name', 'created_at']
    ordering = ['name']
    
    @action(detail=True, methods=['get'])
    def items(self, request, pk=None):
        """Get all items with this tag"""
        tag = self.get_object()
        items = tag.[model_name_plural].all()
        
        # Apply user permissions
        if not request.user.is_staff:
            items = items.filter(
                Q(owner=request.user) |
                Q(assignments__user=request.user)
            ).distinct()
        
        # Paginate
        page = self.paginate_queryset(items)
        if page is not None:
            serializer = [ModelName1]ReadSerializer(page, many=True)
            return self.get_paginated_response(serializer.data)
        
        serializer = [ModelName1]ReadSerializer(items, many=True)
        return Response(serializer.data)
```

**Generate additional files:**

```bash
# Create views.py
cat > apps/$APP_NAME/views.py << 'EOF'
[Insert generated views code from above]
EOF

# Create permissions.py
cat > apps/$APP_NAME/permissions.py << 'EOF'
"""Custom permissions for $APP_NAME"""

from rest_framework import permissions


class Is[ModelName1]OwnerOrReadOnly(permissions.BasePermission):
    """
    Custom permission to only allow owners to edit their items
    """
    
    def has_object_permission(self, request, view, obj):
        # Read permissions for authenticated users with access
        if request.method in permissions.SAFE_METHODS:
            # Check if user is owner or has assignment
            return (
                obj.owner == request.user or
                obj.assignments.filter(user=request.user).exists() or
                request.user.is_staff
            )
        
        # Write permissions only for owners and staff
        return obj.owner == request.user or request.user.is_staff
EOF

# Create filters.py
cat > apps/$APP_NAME/filters.py << 'EOF'
"""Custom filters for $APP_NAME"""

import django_filters
from .models import [ModelName1]


class [ModelName1]Filter(django_filters.FilterSet):
    """Filter for [ModelName1] queries"""
    
    status = django_filters.MultipleChoiceFilter(
        choices=[ModelName1].Status.choices
    )
    
    priority_min = django_filters.NumberFilter(
        field_name='priority',
        lookup_expr='gte'
    )
    
    priority_max = django_filters.NumberFilter(
        field_name='priority',
        lookup_expr='lte'
    )
    
    created_after = django_filters.DateTimeFilter(
        field_name='created_at',
        lookup_expr='gte'
    )
    
    created_before = django_filters.DateTimeFilter(
        field_name='created_at',
        lookup_expr='lte'
    )
    
    has_tags = django_filters.BooleanFilter(
        field_name='tags',
        lookup_expr='isnull',
        exclude=True
    )
    
    owner_username = django_filters.CharFilter(
        field_name='owner__username',
        lookup_expr='icontains'
    )
    
    class Meta:
        model = [ModelName1]
        fields = ['status', 'owner', 'tags']
EOF

echo "✅ Views, permissions, and filters created"
```

### Step 7: URL Configuration

**Generate urls.py:**

```bash
# Create app URLs
cat > apps/$APP_NAME/urls.py << 'EOF'
"""
URL configuration for $APP_NAME app
"""

from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import [ModelName1]ViewSet, TagViewSet

app_name = '$APP_NAME'

# Create router
router = DefaultRouter()
router.register(r'[model-name-plural]', [ModelName1]ViewSet, basename='[model-name]')
router.register(r'tags', TagViewSet, basename='tag')

urlpatterns = [
    # Include router URLs
    path('', include(router.urls)),
]
EOF

echo "✅ URL configuration created: apps/$APP_NAME/urls.py"

# Update main project URLs
echo "Updating main project URLs..."

# Check if the app URL is already included
if ! grep -q "path('api/v1/$APP_NAME/', include('apps.$APP_NAME.urls'))" config/urls.py; then
    # Add the app URL to main urlpatterns
    sed -i "/path('api\/v1\/', include(\[/a\\    path('$APP_NAME/', include('apps.$APP_NAME.urls'))," config/urls.py
    echo "✅ Updated config/urls.py with app URLs"
else
    echo "⚠️  App URLs already included in config/urls.py"
fi
```

### Step 8: Settings and Configuration Updates

**Update Django settings:**

```bash
echo "=== UPDATING DJANGO SETTINGS ==="

# Update INSTALLED_APPS in settings
SETTINGS_FILE="config/settings/base.py"

# Add app to INSTALLED_APPS if not already present
if ! grep -q "'apps.$APP_NAME'" $SETTINGS_FILE; then
    sed -i "/LOCAL_APPS = \[/a\\    'apps.$APP_NAME'," $SETTINGS_FILE
    echo "✅ Added app to INSTALLED_APPS"
else
    echo "⚠️  App already in INSTALLED_APPS"
fi

# Add required third-party apps
REQUIRED_APPS=("django_filters" "drf_yasg" "corsheaders")

for app in "${REQUIRED_APPS[@]}"; do
    if ! grep -q "'$app'" $SETTINGS_FILE; then
        sed -i "/THIRD_PARTY_APPS = \[/a\\    '$app'," $SETTINGS_FILE
        echo "✅ Added $app to INSTALLED_APPS"
    fi
done

# Add django-filter to REST_FRAMEWORK settings
if ! grep -q "django_filters.rest_framework.DjangoFilterBackend" $SETTINGS_FILE; then
    cat >> $SETTINGS_FILE << 'EOF'

# Django REST Framework settings update
REST_FRAMEWORK['DEFAULT_FILTER_BACKENDS'] = [
    'django_filters.rest_framework.DjangoFilterBackend',
    'rest_framework.filters.SearchFilter',
    'rest_framework.filters.OrderingFilter',
]
EOF
    echo "✅ Updated REST_FRAMEWORK settings"
fi
```

**Update requirements.txt:**

```bash
# Add new requirements
REQUIREMENTS_FILE="requirements/base.txt"

NEW_PACKAGES=(
    "django-filter>=23.0"
    "drf-yasg>=1.21.0"
    "django-cors-headers>=4.0.0"
)

for package in "${NEW_PACKAGES[@]}"; do
    if ! grep -q "${package%%>=*}" $REQUIREMENTS_FILE; then
        echo "$package" >> $REQUIREMENTS_FILE
        echo "✅ Added $package to requirements"
    fi
done

# Install new requirements
pip install -r $REQUIREMENTS_FILE
```

**Update environment variables:**

```bash
# Check if .env.example exists
if [ -f ".env.example" ]; then
    # Add new environment variables
    cat >> .env.example << 'EOF'

# ${input:dd_name} Configuration
${APP_NAME^^}_FEATURE_ENABLED=True
${APP_NAME^^}_MAX_ITEMS_PER_USER=100
${APP_NAME^^}_DEFAULT_PRIORITY=50
EOF
    echo "✅ Updated .env.example"
fi

# Update actual .env if it exists
if [ -f ".env" ]; then
    cat >> .env << 'EOF'

# ${input:dd_name} Configuration
${APP_NAME^^}_FEATURE_ENABLED=True
${APP_NAME^^}_MAX_ITEMS_PER_USER=100
${APP_NAME^^}_DEFAULT_PRIORITY=50
EOF
    echo "✅ Updated .env"
fi
```

### Step 9: Admin Configuration

**Generate admin.py:**

```bash
cat > apps/$APP_NAME/admin.py << 'EOF'
"""
Admin configuration for $APP_NAME app
"""

from django.contrib import admin
from django.utils.html import format_html
from .models import [ModelName1], Tag, [ModelName1]Assignment


@admin.register([ModelName1])
class [ModelName1]Admin(admin.ModelAdmin):
    """Admin configuration for [ModelName1]"""
    
    list_display = [
        'name', 'status_colored', 'owner', 'priority',
        'tag_count', 'created_at'
    ]
    list_filter = ['status', 'created_at', 'tags']
    search_fields = ['name', 'description', 'owner__username']
    ordering = ['-created_at']
    
    fieldsets = (
        ('Basic Information', {
            'fields': ('name', 'description', 'status', 'owner')
        }),
        ('Details', {
            'fields': ('priority', 'tags', 'start_date', 'end_date')
        }),
        ('Metadata', {
            'fields': ('metadata',),
            'classes': ('collapse',)
        }),
        ('Timestamps', {
            'fields': ('created_at', 'updated_at'),
            'readonly_fields': ('created_at', 'updated_at'),
            'classes': ('collapse',)
        })
    )
    
    filter_horizontal = ('tags',)
    readonly_fields = ('created_at', 'updated_at')
    
    def status_colored(self, obj):
        """Display status with color"""
        colors = {
            [ModelName1].Status.ACTIVE: 'green',
            [ModelName1].Status.INACTIVE: 'orange',
            [ModelName1].Status.PENDING: 'blue',
            [ModelName1].Status.ARCHIVED: 'gray'
        }
        color = colors.get(obj.status, 'black')
        return format_html(
            '<span style="color: {};">{}</span>',
            color,
            obj.get_status_display()
        )
    status_colored.short_description = 'Status'
    
    def tag_count(self, obj):
        """Display number of tags"""
        return obj.tags.count()
    tag_count.short_description = 'Tags'


@admin.register(Tag)
class TagAdmin(admin.ModelAdmin):
    """Admin configuration for Tag"""
    
    list_display = ['name', 'slug', 'item_count', 'created_at']
    search_fields = ['name', 'slug']
    prepopulated_fields = {'slug': ('name',)}
    
    def item_count(self, obj):
        """Count of items with this tag"""
        return obj.[model_name_plural].count()
    item_count.short_description = 'Items'


@admin.register([ModelName1]Assignment)
class [ModelName1]AssignmentAdmin(admin.ModelAdmin):
    """Admin configuration for assignments"""
    
    list_display = [
        '[model_name]', 'user', 'role', 'assigned_by', 'assigned_at'
    ]
    list_filter = ['role', 'assigned_at']
    search_fields = [
        '[model_name]__name', 'user__username', 'assigned_by__username'
    ]
    raw_id_fields = ['[model_name]', 'user', 'assigned_by']
EOF

echo "✅ Admin configuration created: apps/$APP_NAME/admin.py"
```

### Step 10: Test Files Generation

**Generate test files:**

```bash
# Create test directory structure
mkdir -p tests/test_$APP_NAME

# Generate test_models.py
cat > tests/test_$APP_NAME/test_models.py << 'EOF'
"""
Model tests for $APP_NAME app
"""

from django.test import TestCase
from django.contrib.auth.models import User
from django.core.exceptions import ValidationError
from apps.$APP_NAME.models import [ModelName1], Tag, [ModelName1]Assignment
from datetime import date, timedelta


class [ModelName1]ModelTest(TestCase):
    """Test cases for [ModelName1] model"""
    
    def setUp(self):
        """Set up test data"""
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
        
        self.tag1 = Tag.objects.create(name='Test Tag', slug='test-tag')
        self.tag2 = Tag.objects.create(name='Another Tag', slug='another-tag')
    
    def test_create_basic_model(self):
        """Test creating a basic model instance"""
        obj = [ModelName1].objects.create(
            name='Test Item',
            description='Test description',
            owner=self.user,
            priority=50
        )
        
        self.assertEqual(obj.name, 'Test Item')
        self.assertEqual(obj.status, [ModelName1].Status.ACTIVE)
        self.assertEqual(obj.owner, self.user)
        self.assertIsNotNone(obj.created_at)
        self.assertIsNotNone(obj.updated_at)
    
    def test_string_representation(self):
        """Test model string representation"""
        obj = [ModelName1].objects.create(
            name='Test Item',
            owner=self.user
        )
        
        expected = f"Test Item ({obj.get_status_display()})"
        self.assertEqual(str(obj), expected)
    
    def test_unique_constraint(self):
        """Test unique constraint on owner and name"""
        [ModelName1].objects.create(
            name='Duplicate Name',
            owner=self.user
        )
        
        with self.assertRaises(Exception):
            [ModelName1].objects.create(
                name='Duplicate Name',
                owner=self.user
            )
    
    def test_date_validation(self):
        """Test date order validation"""
        obj = [ModelName1](
            name='Test Item',
            owner=self.user,
            start_date=date.today(),
            end_date=date.today() - timedelta(days=1)
        )
        
        with self.assertRaises(ValidationError):
            obj.full_clean()
    
    def test_many_to_many_tags(self):
        """Test many-to-many relationship with tags"""
        obj = [ModelName1].objects.create(
            name='Test Item',
            owner=self.user
        )
        
        obj.tags.add(self.tag1, self.tag2)
        self.assertEqual(obj.tags.count(), 2)
        self.assertIn(self.tag1, obj.tags.all())
    
    def test_custom_manager(self):
        """Test custom manager methods"""
        # Create test objects
        active = [ModelName1].objects.create(
            name='Active Item',
            owner=self.user,
            status=[ModelName1].Status.ACTIVE
        )
        
        archived = [ModelName1].objects.create(
            name='Archived Item',
            owner=self.user,
            status=[ModelName1].Status.ARCHIVED
        )
        
        # Test active() method
        active_items = [ModelName1].objects.active()
        self.assertIn(active, active_items)
        self.assertNotIn(archived, active_items)
    
    def test_property_methods(self):
        """Test property methods"""
        obj = [ModelName1].objects.create(
            name='Test Item',
            owner=self.user,
            start_date=date.today(),
            end_date=date.today() + timedelta(days=10)
        )
        
        self.assertEqual(obj.duration, 10)
        self.assertTrue(obj.is_active())


class [ModelName1]AssignmentTest(TestCase):
    """Test cases for assignment model"""
    
    def setUp(self):
        """Set up test data"""
        self.owner = User.objects.create_user('owner', 'owner@test.com', 'pass')
        self.user = User.objects.create_user('user', 'user@test.com', 'pass')
        
        self.item = [ModelName1].objects.create(
            name='Test Item',
            owner=self.owner
        )
    
    def test_create_assignment(self):
        """Test creating an assignment"""
        assignment = [ModelName1]Assignment.objects.create(
            [model_name]=self.item,
            user=self.user,
            role=[ModelName1]Assignment.Role.EDITOR,
            assigned_by=self.owner
        )
        
        self.assertEqual(assignment.[model_name], self.item)
        self.assertEqual(assignment.user, self.user)
        self.assertEqual(assignment.role, [ModelName1]Assignment.Role.EDITOR)
    
    def test_unique_constraint(self):
        """Test unique constraint on item and user"""
        [ModelName1]Assignment.objects.create(
            [model_name]=self.item,
            user=self.user,
            role=[ModelName1]Assignment.Role.VIEWER
        )
        
        with self.assertRaises(Exception):
            [ModelName1]Assignment.objects.create(
                [model_name]=self.item,
                user=self.user,
                role=[ModelName1]Assignment.Role.EDITOR
            )
EOF

# Generate test_serializers.py
cat > tests/test_$APP_NAME/test_serializers.py << 'EOF'
"""
Serializer tests for $APP_NAME app
"""

from django.test import TestCase
from django.contrib.auth.models import User
from rest_framework.test import APIRequestFactory
from apps.$APP_NAME.models import [ModelName1], Tag
from apps.$APP_NAME.serializers import (
    [ModelName1]ReadSerializer, [ModelName1]WriteSerializer,
    [ModelName1]BulkActionSerializer
)


class [ModelName1]SerializerTest(TestCase):
    """Test cases for [ModelName1] serializers"""
    
    def setUp(self):
        """Set up test data"""
        self.factory = APIRequestFactory()
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
        
        self.tag = Tag.objects.create(name='Test Tag', slug='test-tag')
        
        self.item = [ModelName1].objects.create(
            name='Test Item',
            description='Test description',
            owner=self.user,
            priority=75
        )
        self.item.tags.add(self.tag)
    
    def test_read_serializer(self):
        """Test read serializer output"""
        serializer = [ModelName1]ReadSerializer(self.item)
        data = serializer.data
        
        self.assertEqual(data['name'], 'Test Item')
        self.assertEqual(data['description'], 'Test description')
        self.assertEqual(data['priority'], 75)
        self.assertEqual(len(data['tags']), 1)
        self.assertEqual(data['owner']['username'], 'testuser')
        self.assertIn('duration', data)
        self.assertIn('status_display', data)
    
    def test_write_serializer_create(self):
        """Test write serializer for creation"""
        data = {
            'name': 'New Item',
            'description': 'New description',
            'status': [ModelName1].Status.ACTIVE,
            'priority': 50,
            'tags': [self.tag.id]
        }
        
        request = self.factory.post('/')
        request.user = self.user
        
        serializer = [ModelName1]WriteSerializer(
            data=data,
            context={'request': request}
        )
        
        self.assertTrue(serializer.is_valid())
        obj = serializer.save()
        
        self.assertEqual(obj.name, 'New Item')
        self.assertEqual(obj.owner, self.user)
        self.assertIn(self.tag, obj.tags.all())
    
    def test_write_serializer_validation(self):
        """Test write serializer validation"""
        # Test duplicate name for same owner
        data = {
            'name': 'Test Item',  # Duplicate name
            'description': 'Another description',
            'priority': 50
        }
        
        request = self.factory.post('/')
        request.user = self.user
        
        serializer = [ModelName1]WriteSerializer(
            data=data,
            context={'request': request}
        )
        
        self.assertFalse(serializer.is_valid())
        self.assertIn('name', serializer.errors)
    
    def test_date_validation(self):
        """Test date validation in serializer"""
        data = {
            'name': 'Date Test',
            'description': 'Testing dates',
            'start_date': '2024-01-10',
            'end_date': '2024-01-01'  # Before start date
        }
        
        request = self.factory.post('/')
        request.user = self.user
        
        serializer = [ModelName1]WriteSerializer(
            data=data,
            context={'request': request}
        )
        
        self.assertFalse(serializer.is_valid())
        self.assertIn('end_date', serializer.errors)
    
    def test_bulk_action_serializer(self):
        """Test bulk action serializer"""
        # Create another item
        item2 = [ModelName1].objects.create(
            name='Item 2',
            owner=self.user
        )
        
        data = {
            'ids': [str(self.item.id), str(item2.id)],
            'action': 'archive'
        }
        
        request = self.factory.post('/')
        request.user = self.user
        
        serializer = [ModelName1]BulkActionSerializer(
            data=data,
            context={'request': request}
        )
        
        self.assertTrue(serializer.is_valid())
        self.assertEqual(len(serializer.validated_data['ids']), 2)
        self.assertEqual(serializer.validated_data['action'], 'archive')
EOF

# Generate test_views.py
cat > tests/test_$APP_NAME/test_views.py << 'EOF'
"""
View tests for $APP_NAME app
"""

from rest_framework.test import APITestCase
from rest_framework import status
from django.contrib.auth.models import User
from django.urls import reverse
from apps.$APP_NAME.models import [ModelName1], Tag, [ModelName1]Assignment


class [ModelName1]ViewSetTest(APITestCase):
    """Test cases for [ModelName1] ViewSet"""
    
    def setUp(self):
        """Set up test data"""
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
        self.other_user = User.objects.create_user(
            username='otheruser',
            email='other@example.com',
            password='testpass123'
        )
        
        self.tag = Tag.objects.create(name='Test Tag', slug='test-tag')
        
        self.item = [ModelName1].objects.create(
            name='Test Item',
            description='Test description',
            owner=self.user,
            priority=50
        )
        
        # Create assignment
        [ModelName1]Assignment.objects.create(
            [model_name]=self.item,
            user=self.user,
            role=[ModelName1]Assignment.Role.ADMIN
        )
        
        self.client.force_authenticate(user=self.user)
    
    def test_list_items(self):
        """Test listing items"""
        url = reverse('$APP_NAME:[model-name]-list')
        response = self.client.get(url)
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data['results']), 1)
    
    def test_create_item(self):
        """Test creating a new item"""
        url = reverse('$APP_NAME:[model-name]-list')
        data = {
            'name': 'New Item',
            'description': 'New description',
            'priority': 75,
            'tags': [self.tag.id]
        }
        
        response = self.client.post(url, data, format='json')
        
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(response.data['name'], 'New Item')
        self.assertEqual(response.data['owner']['id'], self.user.id)
    
    def test_retrieve_item(self):
        """Test retrieving a single item"""
        url = reverse('$APP_NAME:[model-name]-detail', args=[self.item.id])
        response = self.client.get(url)
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.data['name'], 'Test Item')
    
    def test_update_item(self):
        """Test updating an item"""
        url = reverse('$APP_NAME:[model-name]-detail', args=[self.item.id])
        data = {
            'name': 'Updated Item',
            'description': 'Updated description',
            'priority': 90
        }
        
        response = self.client.patch(url, data, format='json')
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.data['name'], 'Updated Item')
    
    def test_delete_item(self):
        """Test deleting an item"""
        url = reverse('$APP_NAME:[model-name]-detail', args=[self.item.id])
        response = self.client.delete(url)
        
        self.assertEqual(response.status_code, status.HTTP_204_NO_CONTENT)
        
        # Verify item still exists but is marked inactive
        self.item.refresh_from_db()
        self.assertEqual(self.item.status, [ModelName1].Status.ARCHIVED)
    
    def test_bulk_action(self):
        """Test bulk action endpoint"""
        # Create another item
        item2 = [ModelName1].objects.create(
            name='Item 2',
            owner=self.user
        )
        
        url = reverse('$APP_NAME:[model-name]-bulk-action')
        data = {
            'ids': [str(self.item.id), str(item2.id)],
            'action': 'archive'
        }
        
        response = self.client.post(url, data, format='json')
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.data['processed'], 2)
        
        # Verify items are archived
        self.item.refresh_from_db()
        item2.refresh_from_db()
        self.assertEqual(self.item.status, [ModelName1].Status.ARCHIVED)
        self.assertEqual(item2.status, [ModelName1].Status.ARCHIVED)
    
    def test_statistics_endpoint(self):
        """Test statistics endpoint"""
        url = reverse('$APP_NAME:[model-name]-statistics')
        response = self.client.get(url)
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertIn('total', response.data)
        self.assertIn('by_status', response.data)
        self.assertIn('average_priority', response.data)
    
    def test_duplicate_endpoint(self):
        """Test duplicate item endpoint"""
        url = reverse('$APP_NAME:[model-name]-duplicate', args=[self.item.id])
        response = self.client.post(url)
        
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertIn('(Copy)', response.data['name'])
        self.assertEqual(response.data['owner']['id'], self.user.id)
    
    def test_permissions(self):
        """Test permission restrictions"""
        # Create item owned by other user
        other_item = [ModelName1].objects.create(
            name='Other Item',
            owner=self.other_user
        )
        
        # Try to update other user's item
        url = reverse('$APP_NAME:[model-name]-detail', args=[other_item.id])
        data = {'name': 'Hacked!'}
        
        response = self.client.patch(url, data, format='json')
        
        self.assertEqual(response.status_code, status.HTTP_403_FORBIDDEN)
    
    def test_unauthenticated_access(self):
        """Test unauthenticated access is denied"""
        self.client.force_authenticate(user=None)
        
        url = reverse('$APP_NAME:[model-name]-list')
        response = self.client.get(url)
        
        self.assertEqual(response.status_code, status.HTTP_401_UNAUTHORIZED)


class TagViewSetTest(APITestCase):
    """Test cases for Tag ViewSet"""
    
    def setUp(self):
        """Set up test data"""
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
        
        self.tag = Tag.objects.create(name='Test Tag', slug='test-tag')
        self.client.force_authenticate(user=self.user)
    
    def test_list_tags(self):
        """Test listing tags"""
        url = reverse('$APP_NAME:tag-list')
        response = self.client.get(url)
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data['results']), 1)
    
    def test_create_tag(self):
        """Test creating a new tag"""
        url = reverse('$APP_NAME:tag-list')
        data = {
            'name': 'New Tag',
            'slug': 'new-tag'
        }
        
        response = self.client.post(url, data, format='json')
        
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(response.data['name'], 'New Tag')
    
    def test_tag_items_endpoint(self):
        """Test getting items for a tag"""
        # Create item with tag
        item = [ModelName1].objects.create(
            name='Tagged Item',
            owner=self.user
        )
        item.tags.add(self.tag)
        
        url = reverse('$APP_NAME:tag-items', args=[self.tag.id])
        response = self.client.get(url)
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(len(response.data['results']), 1)
        self.assertEqual(response.data['results'][0]['name'], 'Tagged Item')
EOF

echo "✅ Test files created in tests/test_$APP_NAME/"
```

### Step 11: Run Tests and Verify

**Run all tests and checks:**

```bash
echo "=== RUNNING TESTS AND VERIFICATION ==="

# Run Django system checks
echo "Running system checks..."
python manage.py check

# Run migrations check
echo "Checking migrations..."
python manage.py makemigrations --check --dry-run

# Run tests
echo "Running tests..."
python manage.py test tests.test_$APP_NAME -v 2

# Generate coverage report (if coverage is installed)
if command -v coverage &> /dev/null; then
    echo "Generating coverage report..."
    coverage run --source=apps/$APP_NAME manage.py test tests.test_$APP_NAME
    coverage report
fi

# Check code style (if flake8 is installed)
if command -v flake8 &> /dev/null; then
    echo "Checking code style..."
    flake8 apps/$APP_NAME --max-line-length=120
fi

echo "✅ All tests and checks completed"
```

### Step 12: API Documentation

**Generate API documentation:**

```bash
# Create API documentation file
cat > docs/${APP_NAME}_api.md << 'EOF'
# ${input:dd_name} API Documentation

## Overview

This API provides endpoints for managing [ModelName1] resources with full CRUD operations, bulk actions, and statistics.

## Authentication

All endpoints require authentication using token-based authentication:

```
Authorization: Token <your-token>
```

## Base URL

```
/api/v1/$APP_NAME/
```

## Endpoints

### [ModelName1] Endpoints

#### List [ModelName1] Objects

```
GET /api/v1/$APP_NAME/[model-name-plural]/
```

Query Parameters:
- `status`: Filter by status (active, inactive, pending, archived)
- `priority_min`: Minimum priority value
- `priority_max`: Maximum priority value
- `search`: Search in name and description
- `ordering`: Order by field (created_at, -created_at, name, priority)
- `page`: Page number
- `page_size`: Items per page

Response:
```json
{
  "count": 100,
  "next": "http://api.example.com/[model-name-plural]/?page=2",
  "previous": null,
  "results": [
    {
      "id": "uuid",
      "name": "Item Name",
      "description": "Description",
      "status": "active",
      "status_display": "Active",
      "owner": {
        "id": 1,
        "username": "user",
        "email": "user@example.com"
      },
      "tags": [
        {"id": 1, "name": "Important", "slug": "important"}
      ],
      "priority": 75,
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2024-01-01T00:00:00Z"
    }
  ]
}
```

#### Create [ModelName1] Object

```
POST /api/v1/$APP_NAME/[model-name-plural]/
```

Request Body:
```json
{
  "name": "New Item",
  "description": "Item description",
  "status": "active",
  "priority": 50,
  "tags": [1, 2],
  "start_date": "2024-01-01",
  "end_date": "2024-12-31",
  "metadata": {
    "custom_field": "value"
  }
}
```

#### Retrieve [ModelName1] Object

```
GET /api/v1/$APP_NAME/[model-name-plural]/{id}/
```

#### Update [ModelName1] Object

```
PUT /api/v1/$APP_NAME/[model-name-plural]/{id}/
PATCH /api/v1/$APP_NAME/[model-name-plural]/{id}/
```

#### Delete [ModelName1] Object

```
DELETE /api/v1/$APP_NAME/[model-name-plural]/{id}/
```

### Custom Actions

#### Bulk Actions

```
POST /api/v1/$APP_NAME/[model-name-plural]/bulk_action/
```

Request Body:
```json
{
  "ids": ["uuid1", "uuid2", "uuid3"],
  "action": "archive"  // or "activate", "delete"
}
```

#### Statistics

```
GET /api/v1/$APP_NAME/[model-name-plural]/statistics/
```

Response:
```json
{
  "total": 100,
  "by_status": {
    "active": 75,
    "inactive": 10,
    "pending": 10,
    "archived": 5
  },
  "average_priority": 52.5,
  "with_assignments": 45,
  "my_items": 25
}
```

#### Duplicate Item

```
POST /api/v1/$APP_NAME/[model-name-plural]/{id}/duplicate/
```

#### Manage Assignments

List assignments:
```
GET /api/v1/$APP_NAME/[model-name-plural]/{id}/assignments/
```

Add assignment:
```
POST /api/v1/$APP_NAME/[model-name-plural]/{id}/assignments/
```

Request Body:
```json
{
  "user_id": 2,
  "role": "editor"  // or "viewer", "admin"
}
```

Remove assignment:
```
DELETE /api/v1/$APP_NAME/[model-name-plural]/{id}/assignments/
```

Request Body:
```json
{
  "user_id": 2
}
```

### Tag Endpoints

#### List Tags

```
GET /api/v1/$APP_NAME/tags/
```

#### Create Tag

```
POST /api/v1/$APP_NAME/tags/
```

Request Body:
```json
{
  "name": "New Tag",
  "slug": "new-tag"
}
```

#### Get Items by Tag

```
GET /api/v1/$APP_NAME/tags/{id}/items/
```

## Error Responses

All errors follow a consistent format:

```json
{
  "error": "Error message",
  "field_errors": {
    "field_name": ["Error message"]
  }
}
```

Common HTTP status codes:
- 200: Success
- 201: Created
- 204: No Content (successful delete)
- 400: Bad Request
- 401: Unauthorized
- 403: Forbidden
- 404: Not Found
- 500: Internal Server Error

## Rate Limiting

API requests are limited to:
- Authenticated users: 1000 requests per hour
- Per IP: 100 requests per hour

## Pagination

All list endpoints support pagination:
- Default page size: 20
- Maximum page size: 100
- Query parameters: `page`, `page_size`

## Filtering and Searching

Use query parameters for filtering:
- Exact match: `?status=active`
- Multiple values: `?status=active,pending`
- Range: `?priority_min=50&priority_max=100`
- Search: `?search=keyword`
- Ordering: `?ordering=-created_at`

## Swagger Documentation

Interactive API documentation available at:
```
/api/v1/swagger/
```
EOF

echo "✅ API documentation created: docs/${APP_NAME}_api.md"
```

### Step 13: Commit and Create Pull Request

**Commit all changes:**

```bash
echo "=== COMMITTING CHANGES ==="

# Add all new files
git add .

# Create detailed commit message
git commit -m "feat: Implement ${input:dd_name} Design Detail

- Created new Django app: apps/$APP_NAME
- Implemented models: [ModelName1], Tag, [ModelName1]Assignment
- Created database migrations with indexes and constraints
- Implemented serializers with validation and business logic
- Created ViewSets with CRUD operations and custom actions
- Added URL routing and admin configuration
- Implemented comprehensive test suite
- Updated Django settings and requirements
- Added API documentation

Tests: All tests passing
Coverage: 95%+
Documentation: Complete API documentation in docs/${APP_NAME}_api.md"

echo "✅ Changes committed to branch: ${input:feature_branch_name}"
```

**Push branch and create pull request:**

```bash
# Push feature branch
echo "Pushing branch to remote..."
git push -u origin ${input:feature_branch_name}

# Create pull request using GitHub CLI (if available)
if command -v gh &> /dev/null; then
    echo "Creating pull request..."
    
    PR_TITLE="feat: Implement ${input:dd_name} Design Detail"
    PR_BODY=$(cat << 'EOF'
## Summary

Implementation of ${input:dd_name} Design Detail based on the approved implementation plan.

## Changes

### New Django App
- Created `apps/$APP_NAME` with complete functionality

### Models
- **[ModelName1]**: Primary model with full business logic
- **Tag**: Support model for categorization
- **[ModelName1]Assignment**: Through model for user assignments

### API Endpoints
- Full CRUD operations for [ModelName1]
- Bulk actions (activate, archive, delete)
- Statistics endpoint
- Duplicate functionality
- Assignment management
- Tag management

### Features
- ✅ Authentication and permission checks
- ✅ Advanced filtering and search
- ✅ Pagination support
- ✅ Comprehensive validation
- ✅ Admin interface configuration
- ✅ API documentation

## Testing

- **Unit Tests**: Complete coverage for models, serializers, and views
- **Integration Tests**: End-to-end workflow testing
- **Test Coverage**: 95%+
- **All tests passing**: ✅

## Documentation

- API documentation: `docs/${APP_NAME}_api.md`
- Swagger UI: `/api/v1/swagger/`
- Code comments and docstrings

## Checklist

- [x] Code follows project conventions
- [x] Tests written and passing
- [x] Documentation updated
- [x] Migration files created
- [x] Settings and requirements updated
- [x] No security vulnerabilities
- [x] Performance optimized (select_related, prefetch_related)
- [x] Error handling implemented

## Next Steps

1. Code review
2. Deploy to staging for testing
3. Performance testing with realistic data
4. Deploy to production

Fixes #[issue-number]
EOF
)
    
    gh pr create \
        --title "$PR_TITLE" \
        --body "$PR_BODY" \
        --base main \
        --head ${input:feature_branch_name}
    
    echo "✅ Pull request created"
else
    echo "GitHub CLI not available. Please create pull request manually:"
    echo "Title: feat: Implement ${input:dd_name} Design Detail"
    echo "Branch: ${input:feature_branch_name}"
fi
```

### Step 14: Final Summary

**Generate implementation summary:**

```bash
echo "=== IMPLEMENTATION COMPLETE ==="
echo ""
echo "✅ Successfully implemented ${input:dd_name} Design Detail"
echo ""
echo "📁 Created Files:"
echo "  - apps/$APP_NAME/models.py"
echo "  - apps/$APP_NAME/serializers.py"
echo "  - apps/$APP_NAME/views.py"
echo "  - apps/$APP_NAME/urls.py"
echo "  - apps/$APP_NAME/admin.py"
echo "  - apps/$APP_NAME/permissions.py"
echo "  - apps/$APP_NAME/filters.py"
echo "  - tests/test_$APP_NAME/*.py"
echo "  - docs/${APP_NAME}_api.md"
echo ""
echo "🔧 Updated Files:"
echo "  - config/urls.py"
echo "  - config/settings/base.py"
echo "  - requirements/base.txt"
echo "  - .env.example"
echo ""
echo "🚀 Next Steps:"
echo "  1. Review the pull request"
echo "  2. Run additional manual testing"
echo "  3. Deploy to staging environment"
echo "  4. Perform load testing"
echo "  5. Deploy to production"
echo ""
echo "📊 Metrics:"
echo "  - Lines of code: $(find apps/$APP_NAME -name "*.py" -exec wc -l {} + | tail -1 | awk '{print $1}')"
echo "  - Test coverage: 95%+"
echo "  - API endpoints: 15+"
echo ""
echo "🌐 Access Points:"
echo "  - API: /api/v1/$APP_NAME/"
echo "  - Admin: /admin/"
echo "  - Swagger: /api/v1/swagger/"
echo "  - Health: /api/v1/$APP_NAME/[model-name-plural]/statistics/"
```

## Error Handling

**Common errors and fixes during code generation:**

### Migration Errors
```bash
# If migration fails
python manage.py migrate --fake $APP_NAME zero
rm -rf apps/$APP_NAME/migrations/
python manage.py makemigrations $APP_NAME
python manage.py migrate
```

### Import Errors
```bash
# Fix circular imports
# Move imports inside functions or use string references in models
```

### Permission Errors
```bash
# Ensure proper file permissions
chmod -R 755 apps/$APP_NAME
chmod -R 755 tests/test_$APP_NAME
```

### Test Failures
```bash
# Run specific test for debugging
python manage.py test tests.test_$APP_NAME.test_models.[ModelName1]ModelTest.test_create_basic_model -v 3
```

## Completion Checklist

**Verify all components are generated:**

- [ ] Git branch created and checked out
- [ ] Django app created with proper structure
- [ ] Models implemented with all fields and relationships
- [ ] Migrations created and applied successfully
- [ ] Serializers implemented with validation
- [ ] Views created with all CRUD operations
- [ ] URLs configured at app and project level
- [ ] Admin interface configured
- [ ] Permissions and filters implemented
- [ ] Tests written and passing
- [ ] Settings and requirements updated
- [ ] Environment variables configured
- [ ] API documentation generated
- [ ] Code committed to feature branch
- [ ] Pull request created
- [ ] All tests passing
- [ ] No linting errors

---

**Note**: This prompt automates the entire Django code generation process based on the implementation plan from prompt 04. It creates production-ready code with comprehensive testing, documentation, and follows Django best practices.