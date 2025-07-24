---
applyTo: "**"
---

# Django REST Framework Project Context & Guidelines

## Project Overview

This Django REST Framework backend provides robust API services with modern Python practices, comprehensive testing, and production-ready architecture.

### Core Technologies
- **Framework**: Django 4.2+ with Django REST Framework 3.14+
- **Database**: PostgreSQL (production), SQLite (development/testing)
- **Authentication**: JWT with djangorestframework-simplejwt
- **Documentation**: drf-yasg for OpenAPI/Swagger
- **Testing**: pytest-django with factory-boy
- **Code Quality**: black, isort, flake8, mypy

## Project Architecture

### Directory Structure
```
project_root/
├── apps/                     # Django applications
│   ├── core/                # Shared models and utilities
│   ├── authentication/      # User auth and profiles
│   ├── api/                 # API versioning and routing
│   └── [feature_apps]/      # Feature-specific apps
├── config/                  # Django settings
│   ├── settings/            # Environment-specific settings
│   ├── urls.py             # Root URL configuration
│   ├── wsgi.py             # WSGI configuration
│   └── asgi.py             # ASGI configuration
├── tests/                   # Test suite
├── utils/                   # Shared utilities
├── requirements/            # Dependency management
├── scripts/                 # Management scripts
└── docs/                   # Documentation
```

### Application Structure
Each Django app follows this structure:
```
app_name/
├── models.py               # Database models
├── serializers.py          # DRF serializers
├── views.py               # API views and viewsets
├── urls.py                # URL routing
├── permissions.py         # Custom permissions
├── filters.py             # Django-filter configurations
├── admin.py               # Django admin
├── apps.py                # App configuration
├── signals.py             # Django signals
├── managers.py            # Custom model managers
├── tasks.py               # Celery tasks
└── tests/                 # App-specific tests
```

## Coding Standards

### Python & Django Best Practices
- Follow PEP 8 with Black formatting (line length: 88)
- Use type hints for all function signatures
- Implement comprehensive docstrings (Google style)
- Apply Django naming conventions consistently
- Use Django's built-in features over custom solutions
- Implement proper error handling with custom exceptions

### API Design Standards
- **URL Structure**: `/api/v1/resource/` with proper versioning
- **HTTP Methods**: Standard RESTful conventions
- **Response Format**: Consistent JSON structure with success/error states
- **Status Codes**: Appropriate HTTP status codes for all scenarios
- **Pagination**: Use DRF's built-in pagination classes
- **Filtering**: Implement django-filter for complex filtering
- **Authentication**: JWT-based with proper token management

### Model Design Principles
- Use abstract base models for common fields (created_at, updated_at, is_active)
- Implement custom model managers for reusable queries
- Apply proper database constraints and indexes
- Use select_related() and prefetch_related() to avoid N+1 queries
- Implement soft deletion pattern where appropriate
- Add comprehensive model validation in clean() methods

### Serializer Patterns
- Use nested serializers for complex relationships
- Implement field-level and object-level validation
- Create dynamic serializers for different contexts
- Use SerializerMethodField for computed fields
- Mark sensitive fields as write_only
- Implement proper error messages for validation failures

### ViewSet & Permission Design
- Use ModelViewSets for standard CRUD operations
- Implement custom actions with @action decorator
- Apply permission classes at view level for fine-grained control
- Use throttling for rate limiting
- Implement proper filtering, searching, and ordering
- Handle bulk operations efficiently

## Testing Strategy

### Test Organization
- **Unit Tests**: Test individual models, serializers, utilities
- **Integration Tests**: Test API endpoints and workflows
- **Performance Tests**: Test response times and query efficiency
- **Security Tests**: Test authentication, permissions, validation

### Test Case Naming Convention
Format: `[AREA]-[TYPE]-[NUMBER]`
- **Areas**: MODEL, VIEW, API, SERIALIZER, AUTH, UTIL, DB
- **Types**: UNIT, INT (integration), E2E (end-to-end)
- **Examples**: `MODEL-UNIT-001`, `API-INT-001`, `AUTH-E2E-001`

### Testing Tools & Patterns
- **Framework**: pytest-django with fixtures
- **Data Generation**: factory-boy for test data
- **Mocking**: unittest.mock for external dependencies
- **API Testing**: DRF's APIClient and APITestCase
- **Coverage**: Maintain >80% test coverage
- **Database**: Use separate test database with transactions

## Security Requirements

### Authentication & Authorization
- Implement JWT token-based authentication
- Create role-based access control (RBAC)
- Apply object-level permissions where needed
- Implement secure password handling with proper hashing
- Use rate limiting to prevent abuse
- Configure proper CORS settings

### Input Validation & Security
- Validate all inputs through serializers
- Sanitize user inputs to prevent XSS
- Use Django ORM exclusively to prevent SQL injection
- Implement proper file upload validation
- Set reasonable limits on request payload sizes
- Never log or expose sensitive information

### Production Security
- Use HTTPS only in production
- Implement proper secret management with environment variables
- Configure secure database connections
- Set appropriate security headers
- Apply CSRF protection
- Use secure session configuration

## Performance Optimization

### Database Performance
- Use select_related() and prefetch_related() consistently
- Add database indexes for frequently queried fields
- Implement database connection pooling
- Monitor slow queries and optimize them
- Use bulk operations for large datasets

### API Performance
- Implement Redis caching for frequently accessed data
- Use pagination for large result sets
- Allow field selection in API responses
- Enable gzip compression
- Monitor API response times
- Use asynchronous tasks for long-running operations

### Memory & Resource Management
- Use generators for large datasets
- Implement proper cache invalidation strategies
- Monitor memory usage and optimize
- Use connection pooling for external services
- Apply proper logging levels

## Error Handling & Logging

### Exception Strategy
- Create domain-specific exception classes
- Implement global exception handler for DRF
- Provide meaningful error messages
- Return appropriate HTTP status codes
- Log errors with sufficient context

### Logging Configuration
- Use structured logging with JSON format
- Implement appropriate log levels (DEBUG, INFO, WARNING, ERROR)
- Log API requests and responses for debugging
- Integrate with monitoring services (Sentry)
- Avoid logging sensitive information

## Development Workflow

### Code Quality
- Use pre-commit hooks for code quality checks
- Run black, isort, flake8, mypy on every commit
- Implement comprehensive code reviews
- Maintain up-to-date documentation
- Keep dependencies updated and secure

### Git Workflow
- Use feature branches for development
- Write descriptive commit messages
- Require code review before merging
- Ensure all tests pass before merging
- Tag releases appropriately

### Environment Management
- Use separate settings for development, testing, production
- Manage secrets with environment variables
- Use Docker for consistent development environments
- Implement proper database migrations
- Automate deployment processes

## Documentation Standards

### Code Documentation
- Write comprehensive docstrings for all public APIs
- Document complex business logic
- Maintain API documentation with drf-yasg
- Keep README files up-to-date
- Document architectural decisions

### API Documentation
- Generate OpenAPI specifications automatically
- Provide interactive API documentation
- Include example requests and responses
- Document authentication requirements
- Explain error responses and status codes

## Deployment & DevOps

### Environment Configuration
- Use environment-specific Django settings
- Implement proper static file serving
- Configure database connections securely
- Set up proper logging and monitoring
- Use containerization for consistency

### CI/CD Pipeline
- Run full test suite on every commit
- Perform code quality checks
- Build and test Docker images
- Deploy to staging automatically
- Implement database migration automation

## Quality Standards

### Code Quality Metrics
- Maintain >80% test coverage
- Keep cyclomatic complexity low
- Monitor technical debt
- Perform regular security audits
- Update dependencies regularly

### Performance Benchmarks
- API response times <200ms for simple queries
- Database query optimization
- Memory usage monitoring
- Load testing for critical endpoints
- Monitor and optimize bottlenecks

When generating code or providing guidance:
- Follow these architectural patterns consistently
- Implement proper error handling and logging
- Write comprehensive tests for all functionality
- Apply security best practices throughout
- Optimize for performance and maintainability
- Document complex logic and decisions
- Use modern Python and Django features appropriately