---
description: Evaluate the comprehensive impact of implementing a Design Detail (DD) in Django REST Framework project
---

# Django Design Detail (DD) Implementation Impact Analysis

This prompt analyzes the comprehensive impact of implementing a new Design Detail (DD) in your Django REST Framework project, including API changes, service layer modifications, database changes, and configuration updates.

## Input Requirements

**Design Detail Folder Structure:**
```
design_details/
├── api_definition.md          # API endpoint specifications
├── service_definition.md      # Business logic and service layer specs  
├── dao_definition.md          # Data access object and model specs
└── additional_specs.md        # Additional configuration/infrastructure specs
```

**Project Information:**
- 📦 Project Name: `${input:project_name}`
- 📋 DD Name: `${input:dd_name}`
- 📁 DD Folder Path: `${input:dd_folder_path}`

## Analysis Workflow

### Step 1: Design Detail Documentation Analysis

**First, analyze all DD specification files:**

#### 1.1 API Definition Analysis
**Read and parse `api_definition.md`:**

```bash
# Analyze API endpoints defined in the DD
echo "=== API ENDPOINTS ANALYSIS ==="
cat ${input:dd_folder_path}/api_definition.md
```

**Extract key information:**
- List all API endpoints (GET, POST, PUT, DELETE)
- Identify request/response data structures
- Note authentication/permission requirements
- Document query parameters and filters
- Identify pagination requirements

#### 1.2 Service Definition Analysis
**Read and parse `service_definition.md`:**

```bash
# Analyze business logic requirements
echo "=== SERVICE LAYER ANALYSIS ==="
cat ${input:dd_folder_path}/service_definition.md
```

**Extract key information:**
- Business logic requirements
- Service layer functions needed
- External service integrations
- Data validation rules
- Transaction requirements

#### 1.3 DAO Definition Analysis
**Read and parse `dao_definition.md`:**

```bash
# Analyze data access requirements
echo "=== DATA ACCESS ANALYSIS ==="
cat ${input:dd_folder_path}/dao_definition.md
```

**Extract key information:**
- Database models required
- Model relationships (ForeignKey, ManyToMany, etc.)
- Database indexes needed
- Data constraints and validation
- Database-level functions or triggers

### Step 2: Existing Project Structure Analysis

**Analyze current Django project structure:**

```bash
# Get current project apps
echo "=== CURRENT PROJECT APPS ==="
ls -la apps/

# Get existing models
echo "=== EXISTING MODELS ==="
find apps/ -name "models.py" -exec echo "File: {}" \; -exec grep -n "class.*Model" {} \;

# Get existing API endpoints
echo "=== EXISTING API ENDPOINTS ==="
find apps/ -name "urls.py" -exec echo "File: {}" \; -exec cat {} \;

# Get existing views
echo "=== EXISTING VIEWS ==="
find apps/ -name "views.py" -exec echo "File: {}" \; -exec grep -n "class.*View\|def.*:" {} \;
```

### Step 3: Endpoint-to-App Mapping Analysis

**For each API endpoint in the DD, determine implementation location:**

#### 3.1 Endpoint Analysis Template
**For each endpoint, analyze:**
```
ENDPOINT: [METHOD] /api/v1/[endpoint_path]/
├── Domain Relevance Analysis
│   ├── Does this belong to existing app? (apps/[app_name]/)
│   ├── What domain does this serve? (user, product, order, etc.)
│   └── Is this a core/shared endpoint? (apps/core/, apps/api/)
├── Implementation Decision
│   ├── ✅ Use existing app: apps/[existing_app]/
│   ├── 🔄 Create new app: apps/[new_app]/
│   └── 📝 Reasoning: [explain why]
└── Impact Assessment
    ├── New files needed: [views.py, serializers.py, urls.py]
    ├── Existing files to modify: [list files]
    └── Dependencies: [other apps, external services]
```

#### 3.2 App Creation Analysis
**If new app is needed:**
```
NEW APP REQUIREMENTS:
├── App Name: apps/[new_app_name]/
├── Purpose: [brief description]
├── Dependencies: [other apps this will depend on]
├── Models Needed: [list from DAO definition]
├── Views Needed: [list from API definition]
└── Integration Points: [how it connects to existing apps]
```

### Step 4: Existing Endpoint Impact Analysis

**For endpoints that already exist:**

#### 4.1 Endpoint Modification Assessment
```
EXISTING ENDPOINT IMPACT:
├── Endpoint: [METHOD] /api/v1/[path]/
├── Current Implementation: apps/[app]/views.py:[line_number]
├── Required Changes:
│   ├── Request Structure Changes: [list changes]
│   ├── Response Structure Changes: [list changes] 
│   ├── Business Logic Changes: [list changes]
│   └── Permission/Auth Changes: [list changes]
├── Backward Compatibility:
│   ├── Breaking Changes: [Yes/No - list if yes]
│   ├── API Versioning Needed: [Yes/No]
│   └── Migration Strategy: [how to handle existing clients]
└── Testing Impact:
    ├── Unit Tests to Update: [list test files]
    ├── Integration Tests to Update: [list test files]
    └── New Tests Needed: [list new test scenarios]
```

### Step 5: Service Layer Impact Analysis

**Analyze impact on service layer (business logic):**

#### 5.1 New Service Requirements
```
NEW SERVICES NEEDED:
├── Service Module: apps/[app]/services.py
├── Service Classes/Functions:
│   ├── [ServiceClassName1] - [description]
│   ├── [ServiceClassName2] - [description]
│   └── [function_name] - [description]
├── External Integrations:
│   ├── Third-party APIs: [list]
│   ├── Message Queues: [Redis, Celery, etc.]
│   └── File Storage: [AWS S3, local, etc.]
└── Business Logic Complexity:
    ├── Simple CRUD: [endpoints that are basic CRUD]
    ├── Complex Logic: [endpoints with complex business rules]
    └── Transaction Management: [endpoints requiring DB transactions]
```

#### 5.2 Existing Service Modifications
```
EXISTING SERVICE CHANGES:
├── Service File: apps/[app]/services.py
├── Functions to Modify:
│   ├── [function_name] - [changes needed]
│   └── [function_name] - [changes needed]
├── New Dependencies:
│   ├── Python Packages: [list new packages needed]
│   ├── Django Apps: [list apps to add to INSTALLED_APPS]
│   └── External Services: [APIs, databases, etc.]
└── Impact Assessment:
    ├── Performance Impact: [analyze query complexity, caching needs]
    ├── Security Impact: [authentication, authorization changes]
    └── Scalability Impact: [will this create bottlenecks?]
```

### Step 6: Database and Model Impact Analysis

**Analyze database changes required:**

#### 6.1 New Model Requirements
```
NEW MODELS NEEDED:
├── App Location: apps/[app]/models.py
├── Model Definitions:
│   ├── Model: [ModelName1]
│   │   ├── Fields: [list all fields with types]
│   │   ├── Relationships: [ForeignKey, ManyToMany, etc.]
│   │   ├── Meta Options: [ordering, indexes, constraints]
│   │   └── Methods: [custom methods needed]
│   └── Model: [ModelName2]
│       ├── Fields: [list all fields with types]
│       └── [continue pattern]
├── Database Indexes:
│   ├── Performance Indexes: [fields that need indexing]
│   ├── Unique Constraints: [fields with unique requirements]
│   └── Composite Indexes: [multi-field indexes]
└── Migration Complexity:
    ├── Simple Migration: [straightforward field additions]
    ├── Complex Migration: [data migration, schema changes]
    └── Production Considerations: [downtime, rollback strategy]
```

#### 6.2 Existing Model Modifications
```
EXISTING MODEL CHANGES:
├── Model: apps/[app]/models.py - [ModelName]
├── Field Changes:
│   ├── New Fields: [field_name: field_type]
│   ├── Modified Fields: [field_name: old_type -> new_type]
│   └── Removed Fields: [field_name: impact on existing data]
├── Relationship Changes:
│   ├── New Relationships: [relationship_type to ModelName]
│   ├── Modified Relationships: [changes to existing relationships]
│   └── Removed Relationships: [impact assessment]
├── Migration Strategy:
│   ├── Data Migration Needed: [Yes/No - describe if yes]
│   ├── Backward Compatibility: [can old code still work?]
│   └── Rollback Plan: [how to undo changes if needed]
└── Production Impact:
    ├── Downtime Required: [Yes/No - duration estimate]
    ├── Performance Impact: [query performance changes]
    └── Storage Impact: [disk space requirements]
```

### Step 7: View and Serializer Impact Analysis

**Analyze Django views and serializers:**

#### 7.1 New Views Required
```
NEW VIEWS NEEDED:
├── App: apps/[app]/views.py
├── View Classes:
│   ├── [ViewClassName1] (ViewType: ListCreateAPIView, RetrieveUpdateDestroyAPIView, etc.)
│   │   ├── Model: [ModelName]
│   │   ├── Serializer: [SerializerName]
│   │   ├── Permissions: [permission classes needed]
│   │   ├── Filter/Search: [filtering capabilities]
│   │   └── Custom Logic: [any custom business logic]
│   └── [ViewClassName2]
│       └── [continue pattern]
├── Function-Based Views:
│   ├── [function_name] - [purpose]
│   └── [function_name] - [purpose]
└── View Mixins/Utils:
    ├── Custom Mixins Needed: [reusable view components]
    └── Utility Functions: [helper functions for views]
```

#### 7.2 New Serializers Required
```
NEW SERIALIZERS NEEDED:
├── App: apps/[app]/serializers.py
├── Serializer Classes:
│   ├── [SerializerName1]
│   │   ├── Model: [ModelName]
│   │   ├── Fields: [fields to serialize]
│   │   ├── Read-Only Fields: [fields that can't be modified]
│   │   ├── Validation: [custom validation methods]
│   │   └── Nested Serializers: [related model serialization]
│   └── [SerializerName2]
│       └── [continue pattern]
├── Custom Validation:
│   ├── Field-Level: [specific field validation rules]
│   ├── Object-Level: [cross-field validation]
│   └── Business Rules: [complex business logic validation]
└── Performance Considerations:
    ├── Prefetch Related: [optimize related model queries]
    ├── Select Related: [optimize foreign key queries]
    └── Field Selection: [only serialize needed fields]
```

#### 7.3 Existing View/Serializer Modifications
```
EXISTING MODIFICATIONS:
├── Views to Modify:
│   ├── File: apps/[app]/views.py - [ViewName]
│   │   ├── Changes: [describe modifications needed]
│   │   ├── New Methods: [methods to add]
│   │   └── Impact: [breaking changes, backward compatibility]
│   └── [continue for each view]
├── Serializers to Modify:
│   ├── File: apps/[app]/serializers.py - [SerializerName]
│   │   ├── Field Changes: [new/modified/removed fields]
│   │   ├── Validation Changes: [new validation rules]
│   │   └── Impact: [client-side impact, API contract changes]
│   └── [continue for each serializer]
└── URL Configuration:
    ├── New URL Patterns: [new routes to add]
    ├── Modified Patterns: [existing routes to change]
    └── URL Namespace: [impact on URL routing]
```

### Step 8: Configuration and Settings Impact

**Analyze Django settings and configuration changes:**

#### 8.1 Settings Changes Required
```
DJANGO SETTINGS IMPACT:
├── New Apps to Add:
│   ├── INSTALLED_APPS additions: [list new apps]
│   └── Third-Party Apps: [external packages needed]
├── Middleware Changes:
│   ├── New Middleware: [custom middleware needed]
│   ├── Middleware Order: [changes to existing order]
│   └── Conditional Middleware: [environment-specific middleware]
├── Database Settings:
│   ├── New Database Connections: [additional databases]
│   ├── Connection Pooling: [performance optimizations]
│   └── Database Routing: [if using multiple databases]
├── Cache Settings:
│   ├── New Cache Backends: [Redis, Memcached, etc.]
│   ├── Cache Keys: [caching strategy for new features]
│   └── Cache Invalidation: [when to clear caches]
├── Static/Media Files:
│   ├── New Static Files: [CSS, JS, images needed]
│   ├── Media Upload: [file upload requirements]
│   └── CDN Configuration: [content delivery network setup]
└── Environment Variables:
    ├── New Variables: [configuration values needed]
    ├── Secret Management: [sensitive data handling]
    └── Environment-Specific: [dev/staging/prod differences]
```

#### 8.2 Third-Party Integration Impact
```
EXTERNAL INTEGRATIONS:
├── New Python Packages:
│   ├── Package: [package_name] - [purpose]
│   ├── Version Compatibility: [Django/Python version requirements]
│   └── Installation: [pip install, system dependencies]
├── External APIs:
│   ├── API: [api_name] - [purpose]
│   ├── Authentication: [API key, OAuth, etc.]
│   ├── Rate Limits: [API usage limits to consider]
│   └── Error Handling: [how to handle API failures]
├── Infrastructure Changes:
│   ├── New Services: [databases, message queues, etc.]
│   ├── Security: [firewall rules, access controls]
│   └── Monitoring: [logging, metrics, alerting]
└── Development Environment:
    ├── Docker Changes: [container modifications]
    ├── Development Tools: [new tools needed]
    └── CI/CD Impact: [build/deployment changes]
```

### Step 9: Migration Planning and Risk Assessment

**Plan database migrations and assess risks:**

#### 9.1 Migration Strategy
```
DATABASE MIGRATION PLAN:
├── Migration Sequence:
│   ├── Step 1: [first migration - usually safe changes]
│   ├── Step 2: [subsequent migrations - build on previous]
│   └── Step N: [final migration - most complex changes]
├── Data Migration Required:
│   ├── Data Transformation: [how existing data will be converted]
│   ├── Default Values: [for new required fields]
│   ├── Data Cleanup: [removing invalid/obsolete data]
│   └── Validation: [ensuring data integrity after migration]
├── Performance Considerations:
│   ├── Large Table Impact: [migrations on tables with many rows]
│   ├── Index Creation: [time required for new indexes]
│   ├── Lock Duration: [how long tables will be locked]
│   └── Migration Time Estimate: [total time required]
└── Rollback Strategy:
    ├── Reversible Migrations: [migrations that can be undone]
    ├── Data Backup: [backup strategy before migration]
    ├── Rollback Testing: [testing rollback procedures]
    └── Emergency Procedures: [what to do if migration fails]
```

#### 9.2 Risk Assessment
```
IMPLEMENTATION RISKS:
├── High Risk Items:
│   ├── Breaking API Changes: [changes that break existing clients]
│   ├── Data Loss Potential: [operations that could lose data]
│   ├── Performance Degradation: [changes that could slow system]
│   └── Security Vulnerabilities: [new attack vectors introduced]
├── Medium Risk Items:
│   ├── Complex Business Logic: [intricate rules that might have bugs]
│   ├── External Dependencies: [reliance on third-party services]
│   ├── Migration Complexity: [complex database changes]
│   └── Integration Points: [changes affecting multiple systems]
├── Mitigation Strategies:
│   ├── Phased Rollout: [gradual deployment approach]
│   ├── Feature Flags: [ability to toggle features on/off]
│   ├── Monitoring: [extensive logging and alerting]
│   ├── Testing: [comprehensive test coverage]
│   └── Rollback Plan: [quick recovery procedures]
└── Testing Requirements:
    ├── Unit Tests: [test coverage for new/modified code]
    ├── Integration Tests: [test inter-system communication]
    ├── Performance Tests: [load testing for new features]
    ├── Security Tests: [penetration testing, vulnerability scans]
    └── User Acceptance Tests: [business stakeholder validation]
```

### Step 10: Implementation Timeline and Dependencies

**Create implementation plan with dependencies:**

#### 10.1 Implementation Phases
```
IMPLEMENTATION TIMELINE:
├── Phase 1: Foundation (Estimated: X days)
│   ├── Create new Django apps (if needed)
│   ├── Set up basic model structures
│   ├── Create initial migrations
│   ├── Set up basic URL routing
│   └── Dependencies: None
├── Phase 2: Core Models and Services (Estimated: X days)
│   ├── Implement all model classes
│   ├── Create model relationships
│   ├── Implement service layer logic
│   ├── Set up data access patterns
│   └── Dependencies: Phase 1 complete
├── Phase 3: API Implementation (Estimated: X days)
│   ├── Create serializers
│   ├── Implement view classes
│   ├── Set up URL patterns
│   ├── Implement authentication/permissions
│   └── Dependencies: Phase 2 complete
├── Phase 4: Integration and Testing (Estimated: X days)
│   ├── Integration testing
│   ├── Performance optimization
│   ├── Security testing
│   ├── Documentation updates
│   └── Dependencies: Phase 3 complete
└── Phase 5: Deployment and Monitoring (Estimated: X days)
    ├── Production configuration
    ├── Database migration in production
    ├── Monitoring setup
    ├── User training/documentation
    └── Dependencies: Phase 4 complete
```

#### 10.2 Critical Dependencies
```
DEPENDENCY ANALYSIS:
├── Technical Dependencies:
│   ├── External APIs: [must be available and tested]
│   ├── Database Changes: [migrations must complete successfully]
│   ├── Third-Party Packages: [must be installed and compatible]
│   └── Infrastructure: [servers, services must be provisioned]
├── Team Dependencies:
│   ├── Database Team: [for complex migrations]
│   ├── DevOps Team: [for infrastructure changes]
│   ├── Frontend Team: [for API contract coordination]
│   └── QA Team: [for testing planning and execution]
├── Business Dependencies:
│   ├── Stakeholder Approval: [for business logic decisions]
│   ├── User Training: [if UI/workflow changes significantly]
│   ├── Documentation: [user guides, API docs must be updated]
│   └── Compliance: [legal/regulatory requirements]
└── External Dependencies:
    ├── Third-Party Services: [external API availability]
    ├── Vendor Support: [for purchased software/services]
    └── Infrastructure Providers: [cloud services, hosting]
```

## Final Impact Summary Report

**Generate comprehensive impact assessment:**

```
=== DESIGN DETAIL IMPLEMENTATION IMPACT SUMMARY ===

DD Name: ${input:dd_name}
Analysis Date: [current_date]
Project: ${input:project_name}

1. SCOPE OVERVIEW:
   ├── New API Endpoints: X
   ├── Modified API Endpoints: X  
   ├── New Django Apps: X
   ├── Modified Django Apps: X
   ├── New Database Models: X
   ├── Modified Database Models: X
   └── Database Migrations Required: X

2. DEVELOPMENT EFFORT:
   ├── Estimated Development Time: X person-days
   ├── Testing Time: X person-days
   ├── Documentation Time: X person-days
   └── Total Effort: X person-days

3. RISK ASSESSMENT:
   ├── High Risk Items: X
   ├── Medium Risk Items: X
   ├── Low Risk Items: X
   └── Overall Risk Level: [High/Medium/Low]

4. INFRASTRUCTURE IMPACT:
   ├── New Services Required: [Yes/No]
   ├── Database Migration Downtime: [estimated duration]
   ├── Performance Impact: [High/Medium/Low/None]
   └── Security Changes: [Yes/No]

5. TEAM COORDINATION:
   ├── Teams Involved: [list teams]
   ├── External Dependencies: [list dependencies]
   ├── Stakeholder Approvals Needed: [Yes/No]
   └── User Impact: [High/Medium/Low/None]

6. DEPLOYMENT STRATEGY:
   ├── Deployment Phases: X
   ├── Feature Flags Needed: [Yes/No]
   ├── Rollback Complexity: [High/Medium/Low]
   └── Go-Live Timeline: [estimated date]

7. RECOMMENDATIONS:
   ├── Proceed with implementation: [Yes/No/Conditional]
   ├── Suggested modifications to DD: [list suggestions]
   ├── Additional research needed: [list items]
   └── Alternative approaches: [if any]
```

## Implementation Checklist

**Use this checklist to track implementation progress:**

### Pre-Implementation
- [ ] All DD documents analyzed and understood
- [ ] Impact assessment completed and reviewed
- [ ] Stakeholder approval obtained for high-impact changes
- [ ] Team dependencies identified and coordinated
- [ ] Development environment prepared
- [ ] External services/APIs tested and available

### Implementation Phase
- [ ] New Django apps created (if required)
- [ ] Database models implemented and tested
- [ ] Database migrations created and tested in development
- [ ] Service layer implemented and unit tested
- [ ] API views and serializers implemented
- [ ] URL routing configured
- [ ] Authentication/authorization implemented
- [ ] Integration tests written and passing
- [ ] API documentation updated
- [ ] Performance testing completed

### Pre-Deployment
- [ ] All tests passing (unit, integration, performance)
- [ ] Code review completed
- [ ] Database migration tested on staging environment
- [ ] Rollback procedures tested
- [ ] Monitoring and alerting configured
- [ ] Documentation updated (user guides, API docs)
- [ ] Deployment scripts/configurations prepared

### Post-Deployment
- [ ] Production deployment successful
- [ ] Database migrations completed without issues
- [ ] All endpoints responding correctly
- [ ] Performance metrics within acceptable ranges
- [ ] No errors in application logs
- [ ] User acceptance testing passed
- [ ] Stakeholder sign-off obtained

## Success Criteria

**Implementation is considered successful when:**

✅ **All API endpoints function as specified** in the DD  
✅ **Database migrations completed** without data loss or corruption  
✅ **Performance impact** is within acceptable ranges  
✅ **All tests pass** (unit, integration, performance, security)  
✅ **No breaking changes** to existing API clients (unless planned)  
✅ **Documentation is complete** and accurate  
✅ **Monitoring and alerting** are functioning correctly  
✅ **Stakeholders approve** the implemented functionality  

## Troubleshooting Guide

**Common issues and solutions during DD implementation:**

### Model/Migration Issues
- **Foreign key constraints**: Ensure related models exist before creating relationships
- **Data migration failures**: Test data transformations on copy of production data
- **Migration rollback**: Always test rollback procedures before production deployment

### API Implementation Issues  
- **Serializer validation errors**: Ensure all required fields have appropriate validation
- **Permission denied errors**: Verify authentication and authorization settings
- **Performance issues**: Use select_related/prefetch_related for optimal database queries

### Integration Issues
- **External API failures**: Implement proper error handling and retry logic
- **Service communication**: Ensure proper service layer abstraction
- **Configuration conflicts**: Verify settings across all environments

---

**Note**: This prompt provides a comprehensive framework for analyzing DD implementation impact. Adapt the analysis depth based on the complexity of your specific Design Detail and project requirements.