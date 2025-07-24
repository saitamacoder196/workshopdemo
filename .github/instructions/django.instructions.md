---
applyTo: "**/*.py"
---

# Django REST Framework Development Instructions

## Framework Standards

### Django Version & Setup
- Use Django 4.2+ LTS with Django REST Framework 3.14+
- Follow Django best practices and conventions
- Implement proper settings configuration with environment separation
- Use Django's app registry and proper URL routing
- Apply Django coding style with Black formatter

### Project Architecture
- Structure projects with feature-based Django apps
- Separate concerns with proper service layers
- Implement repository pattern for complex data operations
- Use Django's built-in features before creating custom solutions
- Apply MTV (Model-Template-View) pattern correctly

## Models & Database

### Model Design
```python
from django.db import models
from django.utils import timezone
from django.core.validators import MinValueValidator

class BaseModel(models.Model):
    """Abstract base model with common fields."""
    created_at = models.DateTimeField(default=timezone.now)
    updated_at = models.DateTimeField(auto_now=True)
    is_active = models.BooleanField(default=True)
    
    class Meta:
        abstract = True
        ordering = ['-created_at']
    
    def soft_delete(self):
        """Implement soft deletion."""
        self.is_active = False
        self.save(update_fields=['is_active'])

class User(BaseModel):
    """User model with proper validation."""
    email = models.EmailField(unique=True)
    name = models.CharField(max_length=100)
    age = models.PositiveIntegerField(validators=[MinValueValidator(13)])
    
    class Meta:
        db_table = 'users'
        indexes = [
            models.Index(fields=['email']),
            models.Index(fields=['is_active']),
        ]
    
    def __str__(self):
        return f"{self.name} ({self.email})"
```

### Database Best Practices
- Use proper field types and constraints
- Implement database indexes for frequently queried fields
- Apply select_related() and prefetch_related() to avoid N+1 queries
- Handle migrations properly with proper data migration scripts
- Implement custom model managers for reusable queries
- Use F() and Q() objects for complex database operations

## Serializers

### Serializer Patterns
```python
from rest_framework import serializers
from rest_framework.validators import UniqueValidator
from .models import User

class UserSerializer(serializers.ModelSerializer):
    """User serializer with validation."""
    email = serializers.EmailField(
        validators=[UniqueValidator(queryset=User.objects.all())]
    )
    password = serializers.CharField(write_only=True, min_length=8)
    
    class Meta:
        model = User
        fields = ['id', 'email', 'name', 'age', 'password', 'created_at']
        read_only_fields = ['id', 'created_at']
    
    def validate_age(self, value):
        """Validate age constraints."""
        if value < 13:
            raise serializers.ValidationError("Age must be at least 13")
        return value
    
    def validate(self, attrs):
        """Object-level validation."""
        # Add cross-field validation here
        return attrs
    
    def create(self, validated_data):
        """Create user with password hashing."""
        password = validated_data.pop('password')
        user = User.objects.create(**validated_data)
        user.set_password(password)
        user.save()
        return user

class UserListSerializer(serializers.ModelSerializer):
    """Lightweight serializer for listing users."""
    class Meta:
        model = User
        fields = ['id', 'email', 'name', 'is_active']
```

### Nested Serializers
```python
class UserDetailSerializer(serializers.ModelSerializer):
    """Detailed user serializer with nested data."""
    profile = ProfileSerializer(read_only=True)
    total_orders = serializers.SerializerMethodField()
    
    class Meta:
        model = User
        fields = ['id', 'email', 'name', 'profile', 'total_orders']
    
    def get_total_orders(self, obj):
        """Calculate total orders for user."""
        return obj.orders.filter(is_active=True).count()
```

## Views & ViewSets

### ViewSet Implementation
```python
from rest_framework import viewsets, status, filters
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from django_filters.rest_framework import DjangoFilterBackend
from .models import User
from .serializers import UserSerializer, UserListSerializer
from .permissions import IsOwnerOrReadOnly

class UserViewSet(viewsets.ModelViewSet):
    """User ViewSet with custom actions."""
    queryset = User.objects.select_related('profile').filter(is_active=True)
    permission_classes = [IsAuthenticated, IsOwnerOrReadOnly]
    filter_backends = [DjangoFilterBackend, filters.SearchFilter, filters.OrderingFilter]
    filterset_fields = ['age', 'is_active']
    search_fields = ['name', 'email']
    ordering_fields = ['created_at', 'name']
    ordering = ['-created_at']
    
    def get_serializer_class(self):
        """Return appropriate serializer class."""
        if self.action == 'list':
            return UserListSerializer
        return UserSerializer
    
    def get_queryset(self):
        """Filter queryset based on user permissions."""
        queryset = super().get_queryset()
        if not self.request.user.is_staff:
            queryset = queryset.filter(id=self.request.user.id)
        return queryset
    
    @action(detail=True, methods=['post'])
    def set_password(self, request, pk=None):
        """Custom action to change user password."""
        user = self.get_object()
        password = request.data.get('password')
        
        if not password:
            return Response(
                {'error': 'Password is required'}, 
                status=status.HTTP_400_BAD_REQUEST
            )
        
        user.set_password(password)
        user.save()
        
        return Response({'message': 'Password updated successfully'})
    
    @action(detail=False, methods=['get'])
    def me(self, request):
        """Get current user profile."""
        serializer = self.get_serializer(request.user)
        return Response(serializer.data)
```

### Generic Views
```python
from rest_framework.generics import ListCreateAPIView, RetrieveUpdateDestroyAPIView

class UserListCreateView(ListCreateAPIView):
    """List and create users."""
    queryset = User.objects.filter(is_active=True)
    serializer_class = UserSerializer
    permission_classes = [IsAuthenticated]

class UserDetailView(RetrieveUpdateDestroyAPIView):
    """Retrieve, update, and delete user."""
    queryset = User.objects.filter(is_active=True)
    serializer_class = UserSerializer
    permission_classes = [IsAuthenticated, IsOwnerOrReadOnly]
```

## Permissions & Authentication

### Custom Permissions
```python
from rest_framework import permissions

class IsOwnerOrReadOnly(permissions.BasePermission):
    """Custom permission to only allow owners to edit objects."""
    
    def has_object_permission(self, request, view, obj):
        """Check object-level permissions."""
        # Read permissions for any request
        if request.method in permissions.SAFE_METHODS:
            return True
        
        # Write permissions only to owner
        return obj.owner == request.user

class IsAdminOrReadOnly(permissions.BasePermission):
    """Admin can edit, others read-only."""
    
    def has_permission(self, request, view):
        """Check view-level permissions."""
        if request.method in permissions.SAFE_METHODS:
            return True
        return request.user.is_staff
```

### JWT Authentication Setup
```python
# settings.py
from datetime import timedelta

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=60),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
}

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
}
```

## URL Routing

### URL Configuration
```python
# urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView
from .views import UserViewSet

router = DefaultRouter()
router.register(r'users', UserViewSet)

urlpatterns = [
    path('api/v1/', include(router.urls)),
    path('api/v1/auth/login/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/v1/auth/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
]
```

## Testing

### Model Tests
```python
import pytest
from django.test import TestCase
from django.core.exceptions import ValidationError
from .models import User

@pytest.mark.django_db
class TestUserModel:
    """Test User model functionality."""
    
    def test_user_creation(self):
        """Test successful user creation."""
        user = User.objects.create(
            email='test@example.com',
            name='Test User',
            age=25
        )
        assert user.email == 'test@example.com'
        assert user.is_active is True
    
    def test_user_str_representation(self):
        """Test string representation."""
        user = User(email='test@example.com', name='Test User')
        assert str(user) == 'Test User (test@example.com)'
    
    def test_age_validation(self):
        """Test age validation."""
        with pytest.raises(ValidationError):
            user = User(email='test@example.com', name='Test', age=10)
            user.full_clean()
```

### API Tests
```python
import pytest
from rest_framework.test import APIClient
from rest_framework import status
from django.contrib.auth.models import User

@pytest.mark.django_db
class TestUserAPI:
    """Test User API endpoints."""
    
    def setup_method(self):
        """Set up test data."""
        self.client = APIClient()
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
    
    def test_user_list_requires_auth(self):
        """Test that user list requires authentication."""
        response = self.client.get('/api/v1/users/')
        assert response.status_code == status.HTTP_401_UNAUTHORIZED
    
    def test_user_list_with_auth(self):
        """Test authenticated user list."""
        self.client.force_authenticate(user=self.user)
        response = self.client.get('/api/v1/users/')
        assert response.status_code == status.HTTP_200_OK
    
    def test_user_creation(self):
        """Test user creation via API."""
        data = {
            'email': 'new@example.com',
            'name': 'New User',
            'age': 30,
            'password': 'newpass123'
        }
        self.client.force_authenticate(user=self.user)
        response = self.client.post('/api/v1/users/', data)
        assert response.status_code == status.HTTP_201_CREATED
```

## Error Handling

### Custom Exceptions
```python
from rest_framework.views import exception_handler
from rest_framework.response import Response
from rest_framework import status

class CustomAPIException(Exception):
    """Custom API exception."""
    default_message = 'An error occurred'
    default_code = 'error'
    
    def __init__(self, message=None, code=None):
        self.message = message or self.default_message
        self.code = code or self.default_code

def custom_exception_handler(exc, context):
    """Custom exception handler."""
    response = exception_handler(exc, context)
    
    if response is not None:
        custom_response_data = {
            'success': False,
            'error': {
                'code': getattr(exc, 'code', 'error'),
                'message': getattr(exc, 'message', str(exc)),
                'details': response.data
            }
        }
        response.data = custom_response_data
    
    return response
```

## Configuration

### Settings Structure
```python
# settings/base.py
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent.parent

DJANGO_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

THIRD_PARTY_APPS = [
    'rest_framework',
    'rest_framework_simplejwt',
    'django_filters',
    'corsheaders',
    'drf_yasg',
]

LOCAL_APPS = [
    'apps.core',
    'apps.authentication',
    'apps.api',
]

INSTALLED_APPS = DJANGO_APPS + THIRD_PARTY_APPS + LOCAL_APPS

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20,
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
        'rest_framework.filters.SearchFilter',
        'rest_framework.filters.OrderingFilter',
    ],
}
```

## Management Commands

### Custom Management Command
```python
# management/commands/create_sample_data.py
from django.core.management.base import BaseCommand
from apps.authentication.models import User

class Command(BaseCommand):
    """Create sample data for development."""
    help = 'Create sample users for development'
    
    def add_arguments(self, parser):
        parser.add_argument(
            '--count',
            type=int,
            default=10,
            help='Number of users to create'
        )
    
    def handle(self, *args, **options):
        count = options['count']
        
        for i in range(count):
            User.objects.get_or_create(
                email=f'user{i}@example.com',
                defaults={
                    'name': f'User {i}',
                    'age': 20 + (i % 50)
                }
            )
        
        self.stdout.write(
            self.style.SUCCESS(f'Successfully created {count} users')
        )
```

When working with Django REST Framework:
- Use Django's built-in features and conventions
- Implement proper validation at both serializer and model levels
- Apply appropriate permissions and authentication
- Write comprehensive tests for all functionality
- Follow RESTful API design principles
- Optimize database queries with select_related/prefetch_related
- Handle errors gracefully with proper HTTP status codes
- Document APIs with drf-yasg
- Use environment variables for configuration
- Implement proper logging and monitoring