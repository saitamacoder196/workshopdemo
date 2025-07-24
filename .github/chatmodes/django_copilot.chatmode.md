---
description: 'GitHub Copilot Custom Instructions for Django REST Framework Backend'
tools: ['changes', 'codebase', 'editFiles', 'extensions', 'fetch', 'findTestFiles', 'githubRepo', 'new', 'openSimpleBrowser', 'problems', 'runCommands', 'runNotebooks', 'runTasks', 'runTests', 'search', 'searchResults', 'terminalLastCommand', 'terminalSelection', 'testFailure', 'usages', 'vscodeAPI']
---
# GitHub Copilot Custom Instructions for Django REST Framework Backend

## Project Overview

I'm developing a Django REST Framework (DRF) backend application that provides robust API endpoints for web and mobile applications. The backend focuses on scalable architecture, security, performance optimization, and comprehensive testing.

---

## Tech Stack & Framework

### Core Framework
- **Framework**: Django 4.2+ with Django REST Framework 3.14+
- **Database**: PostgreSQL (production), SQLite (development/testing)
- **Authentication**: JWT authentication (djangorestframework-simplejwt)
- **API Documentation**: drf-yasg for Swagger/OpenAPI documentation
- **Task Queue**: Celery with Redis/RabbitMQ
- **Caching**: Redis for session and data caching
- **File Storage**: AWS S3 (production), local storage (development)

### Development Tools
- **Testing**: pytest, pytest-django, factory-boy
- **Code Quality**: pylint, black, isort, pre-commit hooks
- **Logging**: Python logging with structured JSON format
- **Monitoring**: Django Debug Toolbar (development)
- **Environment**: python-decouple for environment variables
- **CORS**: django-cors-headers for frontend integration

---

## Project Structure

```
django_backend/
├── apps/                          # Django applications
│   ├── authentication/            # User auth and profiles
│   ├── core/                     # Shared models and utilities
│   ├── api/                      # API versioning and routing
│   └── [feature_apps]/           # Feature-specific apps
├── config/                       # Django settings and configuration
│   ├── settings/                 # Environment-specific settings
│   │   ├── base.py              # Base settings
│   │   ├── development.py       # Development settings
│   │   ├── production.py        # Production settings
│   │   └── testing.py           # Test settings
│   ├── urls.py                  # Root URL configuration
│   ├── wsgi.py                  # WSGI configuration
│   └── asgi.py                  # ASGI configuration
├── utils/                        # Shared utility functions
│   ├── exceptions.py            # Custom exception classes
│   ├── permissions.py           # Custom permission classes
│   ├── pagination.py            # Custom pagination classes
│   ├── mixins.py               # Reusable model/view mixins
│   └── validators.py           # Custom field validators
├── tests/                        # Test suite
│   ├── factories/               # Factory Boy factories
│   ├── fixtures/                # Test data fixtures
│   └── [app_tests]/            # App-specific tests
├── logs/                         # Log files
├── media/                        # Uploaded media files
├── static/                       # Static files
├── requirements/                 # Dependency management
│   ├── base.txt                # Base requirements
│   ├── development.txt         # Development requirements
│   ├── production.txt          # Production requirements
│   └── testing.txt             # Testing requirements
├── scripts/                      # Management and deployment scripts
├── docker/                       # Docker configuration
├── .env.example                 # Environment variables template
├── manage.py                    # Django management script
└── README.md                    # Project documentation
```

---

## Django Applications Architecture

### Core Application Structure
Each Django app should follow this structure:
```
app_name/
├── __init__.py
├── admin.py                      # Django admin configuration
├── apps.py                       # App configuration
├── models.py                     # Database models
├── serializers.py                # DRF serializers
├── views.py                      # API views and viewsets  
├── urls.py                       # URL routing
├── permissions.py                # Custom permissions
├── filters.py                    # Django-filter configurations
├── tasks.py                      # Celery tasks
├── signals.py                    # Django signals
├── managers.py                   # Custom model managers
├── migrations/                   # Database migrations
└── tests/                        # App-specific tests
    ├── test_models.py
    ├── test_views.py
    ├── test_serializers.py
    └── test_permissions.py
```

### Application Separation Principles
- **Authentication App**: User management, JWT tokens, permissions
- **Core App**: Base models, mixins, shared utilities
- **API App**: API versioning, root routing, global configurations
- **Feature Apps**: Business logic separated by domain (e.g., orders, products, notifications)

---

## API Design Standards

### URL Structure & Versioning
- **Base URL Pattern**: `/api/v1/`
- **Resource Endpoints**: `/api/v1/users/`, `/api/v1/orders/`
- **Nested Resources**: `/api/v1/users/{id}/orders/`
- **Custom Actions**: `/api/v1/users/{id}/change-password/`
- **API Versioning**: URL-based versioning (`v1`, `v2`)

### RESTful Conventions
- **CRUD Operations**: Use standard HTTP methods (GET, POST, PUT, PATCH, DELETE)
- **Status Codes**: Return appropriate HTTP status codes
- **Response Format**: Consistent JSON response structure
- **Error Handling**: Standardized error responses with meaningful messages
- **Pagination**: Use DRF's built-in pagination classes
- **Filtering**: Implement django-filter for complex filtering
- **Searching**: Use DRF's SearchFilter for text-based search
- **Ordering**: Implement OrderingFilter for result sorting

### API Response Standards
```python
# Success Response Format
{
    "success": true,
    "data": {
        "results": [...],
        "count": 100,
        "next": "url",
        "previous": "url"
    },
    "message": "Success message"
}

# Error Response Format  
{
    "success": false,
    "errors": {
        "field_name": ["Error message"],
        "non_field_errors": ["General error message"]
    },
    "message": "Error occurred"
}
```

---

## Django Model Design Principles

### Model Architecture
- **Abstract Base Models**: Create base models with common fields (created_at, updated_at, is_active)
- **Model Inheritance**: Use abstract base classes for shared behavior
- **Custom Managers**: Implement custom model managers for reusable queries
- **Model Methods**: Add business logic methods to models
- **Meta Options**: Define proper ordering, unique constraints, and indexes

### Database Optimization
- **Indexes**: Add database indexes for frequently queried fields
- **Relationships**: Use appropriate relationship types (ForeignKey, ManyToMany, OneToOne)
- **Query Optimization**: Use select_related() and prefetch_related() to avoid N+1 queries
- **Database Constraints**: Implement database-level constraints for data integrity
- **Soft Deletion**: Implement soft delete pattern for data preservation

### Example Base Model
```python
from django.db import models
from django.utils import timezone

class BaseModel(models.Model):
    """Abstract base model with common fields"""
    created_at = models.DateTimeField(default=timezone.now)
    updated_at = models.DateTimeField(auto_now=True)
    is_active = models.BooleanField(default=True)
    
    class Meta:
        abstract = True
        ordering = ['-created_at']
    
    def soft_delete(self):
        """Soft delete implementation"""
        self.is_active = False
        self.save(update_fields=['is_active'])
```

---

## Django REST Framework Best Practices

### ViewSets and Views
- **ModelViewSets**: Use for standard CRUD operations with additional custom actions
- **Generic Views**: Use for simple operations (ListAPIView, CreateAPIView, etc.)
- **Custom Actions**: Implement using @action decorator
- **Permission Classes**: Apply at view level for fine-grained control
- **Throttling**: Implement rate limiting for API endpoints
- **Filtering & Search**: Use DRF's built-in filtering capabilities

### Serializers
- **Nested Serializers**: Handle complex relationships
- **Validation**: Implement field-level and object-level validation
- **Dynamic Fields**: Create flexible serializers for different contexts
- **SerializerMethodField**: Add computed fields to responses
- **Write-Only Fields**: Use for sensitive data like passwords
- **Read-Only Fields**: Mark computed or system-generated fields

### Authentication & Permissions
- **JWT Authentication**: Implement token-based authentication
- **Custom Permissions**: Create domain-specific permission classes
- **Role-Based Access**: Implement user roles and permissions
- **Object-Level Permissions**: Control access to specific objects
- **API Key Authentication**: For third-party integrations

---

## Testing Architecture

### Testing Strategy
- **Unit Tests**: Test individual models, serializers, and utility functions
- **Integration Tests**: Test API endpoints and business workflows
- **Performance Tests**: Test API response times and database query efficiency
- **Security Tests**: Test authentication, permissions, and data validation

### Testing Tools & Patterns
- **pytest-django**: Primary testing framework
- **factory-boy**: Generate test data with factories
- **django.test.Client**: Test API endpoints
- **Mock & Patch**: Mock external services and dependencies
- **Database Transactions**: Use TestCase for database tests
- **API Client Testing**: Test authentication and permissions

### Test Organization
```python
# tests/test_models.py
import pytest
from django.core.exceptions import ValidationError
from apps.users.models import User

@pytest.mark.django_db
class TestUserModel:
    def test_user_creation(self):
        user = User.objects.create_user(
            email='test@example.com',
            password='testpass123'
        )
        assert user.email == 'test@example.com'
        assert user.is_active == True

# tests/test_views.py  
import pytest
from rest_framework.test import APIClient
from rest_framework import status

@pytest.mark.django_db
class TestUserAPI:
    def test_user_list_requires_auth(self):
        client = APIClient()
        response = client.get('/api/v1/users/')
        assert response.status_code == status.HTTP_401_UNAUTHORIZED
```

---

## Security Best Practices

### Authentication & Authorization
- **JWT Tokens**: Implement secure token generation and validation
- **Token Refresh**: Handle token expiration and refresh mechanisms
- **Password Security**: Use Django's built-in password hashing
- **Rate Limiting**: Implement API rate limiting with DRF throttling
- **CORS Configuration**: Properly configure CORS for frontend integration

### Input Validation & Sanitization
- **Serializer Validation**: Validate all incoming data through serializers
- **SQL Injection Prevention**: Use Django ORM exclusively, avoid raw SQL
- **XSS Prevention**: Sanitize user inputs and use Django's built-in protection
- **File Upload Security**: Validate file types and sizes for uploads
- **API Input Limits**: Set reasonable limits on request payload sizes

### Data Protection
- **Sensitive Data**: Never log or expose sensitive information
- **Database Encryption**: Encrypt sensitive fields at database level
- **HTTPS Only**: Enforce HTTPS in production environments
- **Environment Variables**: Store secrets in environment variables
- **Database Connections**: Use secure database connection strings

---

## Performance Optimization

### Database Performance
- **Query Optimization**: Use select_related() and prefetch_related()
- **Database Indexes**: Add indexes for frequently queried fields
- **Connection Pooling**: Configure database connection pooling
- **Query Monitoring**: Monitor slow queries and optimize them
- **Bulk Operations**: Use bulk_create() and bulk_update() for large datasets

### Caching Strategies
- **Redis Caching**: Implement Redis for data and session caching
- **QuerySet Caching**: Cache expensive database queries
- **API Response Caching**: Cache API responses for read-heavy endpoints
- **Cache Invalidation**: Implement proper cache invalidation strategies
- **Cache Warming**: Pre-populate cache for frequently accessed data

### API Performance
- **Pagination**: Implement pagination for large datasets
- **Field Selection**: Allow clients to specify required fields
- **Compression**: Enable gzip compression for API responses
- **Asynchronous Tasks**: Use Celery for long-running operations
- **API Monitoring**: Monitor API response times and error rates

---

## Error Handling & Logging

### Exception Handling
- **Custom Exceptions**: Create domain-specific exception classes
- **Global Exception Handler**: Implement DRF's custom exception handler
- **Validation Errors**: Provide meaningful validation error messages
- **HTTP Status Codes**: Return appropriate status codes for different scenarios
- **Error Response Format**: Maintain consistent error response structure

### Logging Configuration
- **Structured Logging**: Use JSON format for log entries
- **Log Levels**: Implement appropriate log levels (DEBUG, INFO, WARNING, ERROR, CRITICAL)
- **Request Logging**: Log API requests and responses for debugging
- **Error Tracking**: Integrate with error tracking services (Sentry)
- **Performance Logging**: Log slow database queries and API calls

### Example Logging Setup
```python
# config/settings/base.py
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {process:d} {thread:d} {message}',
            'style': '{',
        },
        'json': {
            'format': '{"level": "%(levelname)s", "time": "%(asctime)s", "module": "%(module)s", "message": "%(message)s"}',
        },
    },
    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.FileHandler',
            'filename': 'logs/django.log',
            'formatter': 'json',
        },
        'console': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',
            'formatter': 'verbose',
        },
    },
    'root': {
        'handlers': ['console', 'file'],
        'level': 'INFO',
    },
}
```

---

## Deployment & DevOps

### Environment Configuration
- **Settings Management**: Separate settings for different environments
- **Environment Variables**: Use environment variables for sensitive configuration
- **Docker Support**: Containerize the application for consistent deployments
- **Database Migrations**: Automate database migrations in deployment pipeline
- **Static Files**: Configure static file serving for production

### CI/CD Pipeline
- **Automated Testing**: Run test suite on every commit
- **Code Quality Checks**: Run linting and code quality tools
- **Security Scanning**: Scan for security vulnerabilities
- **Automated Deployment**: Deploy to staging and production environments
- **Database Backups**: Implement automated database backup strategies

### Monitoring & Maintenance
- **Health Checks**: Implement health check endpoints
- **Performance Monitoring**: Monitor API performance and database queries
- **Error Tracking**: Track and alert on application errors
- **Log Aggregation**: Centralize logs for analysis and debugging
- **Dependency Updates**: Regularly update dependencies for security patches

---

## Development Workflow

### Code Standards
- **PEP 8 Compliance**: Follow Python coding standards with Black formatting
- **Import Organization**: Use isort for consistent import ordering
- **Code Documentation**: Write comprehensive docstrings for classes and methods
- **Type Hints**: Use Python type hints for better code clarity
- **Pre-commit Hooks**: Implement pre-commit hooks for code quality

### Git Workflow
- **Branch Strategy**: Use feature branches for development
- **Commit Messages**: Write descriptive commit messages
- **Code Review**: Require code review before merging
- **Testing**: Ensure all tests pass before merging
- **Documentation**: Update documentation with code changes

### Development Environment
- **Virtual Environment**: Use virtual environments for dependency isolation
- **Environment Variables**: Use .env files for local development configuration
- **Database Setup**: Provide scripts for local database setup and seeding
- **Development Server**: Configure development server with debug tools
- **API Documentation**: Generate and maintain API documentation

---

## Common Development Tasks

### API Development
- **Creating ViewSets**: Implement CRUD operations with ModelViewSets
- **Custom Actions**: Add custom endpoints using @action decorator
- **Serializer Creation**: Design serializers for data validation and transformation
- **Permission Implementation**: Create custom permission classes
- **Filtering & Search**: Add filtering and search capabilities to endpoints

### Database Operations
- **Model Creation**: Design models with proper relationships and constraints
- **Migration Management**: Create and apply database migrations
- **Data Seeding**: Create management commands for populating test data
- **Query Optimization**: Optimize database queries for performance
- **Data Validation**: Implement model-level and serializer-level validation

### Testing Implementation
- **Test Setup**: Configure test environment and database
- **Factory Creation**: Build Factory Boy factories for test data generation
- **API Testing**: Test API endpoints with different scenarios
- **Permission Testing**: Verify permission and authentication logic
- **Performance Testing**: Test API performance under load

---

## Quality Standards

### Code Quality
- **Clean Code**: Write readable, maintainable code with meaningful names
- **SOLID Principles**: Apply SOLID principles where appropriate
- **DRY Principle**: Avoid code duplication through proper abstraction
- **Error Handling**: Implement comprehensive error handling
- **Security**: Follow security best practices for web applications

### Documentation
- **API Documentation**: Maintain up-to-date API documentation with Swagger
- **Code Comments**: Document complex business logic and algorithms
- **README Files**: Provide clear setup and usage instructions
- **Architecture Documentation**: Document system architecture and design decisions
- **Changelog**: Maintain changelog for version tracking

### Performance
- **Response Times**: Ensure API responses are within acceptable limits
- **Database Efficiency**: Optimize database queries and relationships
- **Caching**: Implement caching where appropriate for performance
- **Scalability**: Design system to handle increased load
- **Resource Usage**: Monitor and optimize memory and CPU usage

---

## Troubleshooting Guide

### Common Issues
- **Database Connection**: Troubleshoot database connection problems
- **Migration Conflicts**: Resolve Django migration conflicts
- **Permission Errors**: Debug authentication and permission issues
- **Performance Problems**: Identify and resolve performance bottlenecks
- **CORS Issues**: Fix cross-origin resource sharing problems

### Debugging Tools
- **Django Debug Toolbar**: Use for development debugging
- **Logging**: Implement comprehensive logging for troubleshooting
- **Error Tracking**: Use error tracking services for production issues
- **Database Queries**: Monitor and optimize database query performance
- **API Testing**: Use tools like Postman or curl for API testing

---

## Best Practices Summary

### Development
- Use Django's built-in features and conventions
- Implement proper error handling and logging
- Write comprehensive tests for all functionality
- Follow RESTful API design principles
- Optimize database queries and implement caching

### Security
- Validate all inputs through serializers
- Implement proper authentication and authorization
- Use HTTPS and secure configuration in production
- Keep dependencies updated and scan for vulnerabilities
- Never expose sensitive information in logs or responses

### Performance
- Optimize database queries with select_related and prefetch_related
- Implement pagination for large datasets
- Use caching for frequently accessed data
- Monitor API performance and optimize bottlenecks
- Use asynchronous tasks for long-running operations

### Maintenance
- Keep dependencies updated
- Monitor application performance and errors
- Implement proper backup and recovery procedures
- Document changes and maintain API documentation
- Use automated testing and deployment pipelines