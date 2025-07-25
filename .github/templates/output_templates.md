# Django DD Workflow Output Templates

This document defines standardized output templates and folder structures for all Django DD workflow prompts that generate documents.

## Template Categories

### 1. Planning Documents (Prompt 04)
### 2. Test Generation Documents (Prompt 06.1)
### 3. Review Checklists (Prompt 07.1)
### 4. Bug Fix Planning Documents (Prompt 08)

---

## 1. Implementation Planning Template (Prompt 04)

### Output Folder Structure
```
dd_implementation_plan/
├── metadata/
│   ├── dd_summary.json
│   ├── project_info.json
│   └── generation_metadata.json
├── analysis/
│   ├── dd_requirements_analysis.json
│   ├── impact_assessment.json
│   └── component_mapping.json
├── implementation/
│   ├── implementation_plan.json
│   ├── migration_plan.json
│   ├── component_specifications.json
│   └── api_specifications.json
├── timeline/
│   ├── implementation_timeline.json
│   ├── milestone_definitions.json
│   └── dependency_graph.json
├── documentation/
│   ├── implementation_plan.md
│   ├── api_documentation.md
│   ├── deployment_guide.md
│   └── developer_manual.md
└── validation/
    ├── acceptance_criteria.json
    ├── test_scenarios.json
    └── success_metrics.json
```

### Main Template: `implementation_plan.json`
```json
{
  "metadata": {
    "dd_name": "string",
    "project_name": "string",
    "app_name": "string",
    "generated_at": "ISO datetime",
    "generated_by": "prompt_04",
    "version": "1.0"
  },
  "dd_summary": {
    "description": "string",
    "business_value": "string",
    "type": "feature|enhancement|bugfix",
    "priority": "critical|high|medium|low",
    "estimated_effort": "string",
    "target_completion": "ISO date"
  },
  "requirements_analysis": {
    "api_endpoints": [],
    "data_models": [],
    "business_rules": [],
    "authentication_requirements": {},
    "performance_requirements": {},
    "integration_requirements": []
  },
  "implementation_components": {
    "models": [],
    "serializers": [],
    "views": [],
    "urls": [],
    "migrations": [],
    "tests": [],
    "configurations": []
  },
  "implementation_plan": {
    "phases": [],
    "dependencies": [],
    "risks": [],
    "mitigation_strategies": []
  },
  "timeline": {
    "estimated_duration": "string",
    "milestones": [],
    "critical_path": [],
    "buffer_time": "string"
  },
  "validation_criteria": {
    "acceptance_criteria": [],
    "test_scenarios": [],
    "success_metrics": [],
    "rollback_criteria": []
  }
}
```

---

## 2. Test Generation Template (Prompt 06.1)

### Output Folder Structure
```
test_generation/
├── metadata/
│   ├── test_generation_summary.json
│   ├── test_matrix.json
│   └── coverage_targets.json
├── analysis/
│   ├── dd_test_requirements.json
│   ├── component_test_mapping.json
│   └── test_scenarios.json
├── templates/
│   ├── model_test_templates.py
│   ├── serializer_test_templates.py
│   ├── view_test_templates.py
│   ├── api_test_templates.py
│   └── business_logic_test_templates.py
├── generated_tests/
│   ├── test_models.py
│   ├── test_serializers.py
│   ├── test_views.py
│   ├── test_api.py
│   ├── test_business_logic.py
│   └── test_integration.py
├── fixtures/
│   ├── test_data.json
│   ├── mock_responses.json
│   └── sample_fixtures.py
└── documentation/
    ├── test_plan.md
    ├── test_coverage_report.md
    └── testing_guidelines.md
```

### Main Template: `test_generation_summary.json`
```json
{
  "metadata": {
    "app_name": "string",
    "dd_name": "string",
    "generated_at": "ISO datetime",
    "generated_by": "prompt_06.1",
    "version": "1.0"
  },
  "test_summary": {
    "total_test_cases": "number",
    "test_categories": {
      "model_tests": "number",
      "serializer_tests": "number",
      "view_tests": "number",
      "api_tests": "number",
      "business_logic_tests": "number",
      "integration_tests": "number"
    },
    "coverage_targets": {
      "overall_target": "percentage",
      "model_coverage": "percentage",
      "view_coverage": "percentage",
      "api_coverage": "percentage"
    }
  },
  "test_matrix": {
    "model_tests": [],
    "serializer_tests": [],
    "view_tests": [],
    "api_tests": [],
    "business_logic_tests": [],
    "integration_tests": []
  },
  "test_files_generated": [
    {
      "file_name": "string",
      "file_path": "string",
      "test_count": "number",
      "test_types": []
    }
  ],
  "test_scenarios": {
    "positive_scenarios": [],
    "negative_scenarios": [],
    "edge_cases": [],
    "performance_tests": []
  },
  "next_steps": [
    "Run prompt 06.2 to execute tests",
    "Review generated test files",
    "Customize test data if needed"
  ]
}
```

---

## 3. Review Checklists Template (Prompt 07.1)

### Output Folder Structure
```
review_checklists/
├── metadata/
│   ├── checklist_summary.json
│   ├── generation_info.json
│   └── compliance_mapping.json
├── quality/
│   ├── quality_checklist.json
│   ├── quality_checklist.md
│   ├── django_best_practices.json
│   ├── code_quality_standards.json
│   └── performance_criteria.json
├── security/
│   ├── security_checklist.json
│   ├── security_checklist.md
│   ├── owasp_top10_checks.json
│   ├── django_security_checks.json
│   └── api_security_checks.json
├── dd_compliance/
│   ├── dd_compliance_checklist.json
│   ├── api_compliance_checks.json
│   ├── model_compliance_checks.json
│   └── business_rule_checks.json
├── templates/
│   ├── manual_review_template.md
│   ├── code_review_template.md
│   └── security_review_template.md
└── documentation/
    ├── review_guidelines.md
    ├── checklist_usage_guide.md
    └── scoring_methodology.md
```

### Main Template: `checklist_summary.json`
```json
{
  "metadata": {
    "project_name": "string",
    "app_name": "string",
    "dd_name": "string",
    "generated_at": "ISO datetime",
    "generated_by": "prompt_07.1",
    "version": "1.0",
    "security_frameworks": ["OWASP Top 10 2021", "Django Security Guidelines"]
  },
  "checklist_summary": {
    "total_checks": "number",
    "quality_checks": "number",
    "security_checks": "number",
    "dd_compliance_checks": "number",
    "automated_checks": "number",
    "manual_checks": "number"
  },
  "quality_checklist": {
    "categories": {
      "django_best_practices": {
        "weight": "percentage",
        "check_count": "number"
      },
      "code_quality_standards": {
        "weight": "percentage", 
        "check_count": "number"
      },
      "performance_optimization": {
        "weight": "percentage",
        "check_count": "number"
      },
      "dd_compliance": {
        "weight": "percentage",
        "check_count": "number"
      }
    }
  },
  "security_checklist": {
    "categories": {
      "owasp_top10": {
        "weight": "percentage",
        "check_count": "number"
      },
      "django_security": {
        "weight": "percentage",
        "check_count": "number"
      },
      "api_security": {
        "weight": "percentage",
        "check_count": "number"
      }
    }
  },
  "severity_breakdown": {
    "critical": "number",
    "high": "number", 
    "medium": "number",
    "low": "number"
  },
  "review_workflow": {
    "automated_tools": ["bandit", "safety", "flake8", "custom_checks"],
    "manual_review_areas": [],
    "estimated_review_time": "string"
  },
  "next_steps": [
    "Run prompt 07.2 to execute automated reviews",
    "Use templates for manual reviews",
    "Address findings and re-review"
  ]
}
```

---

## 4. Bug Fix Planning Template (Prompt 08)

### Output Folder Structure
```
bug_fix_planning/
├── metadata/
│   ├── bug_analysis_summary.json
│   ├── planning_metadata.json
│   └── source_reports_info.json
├── analysis/
│   ├── test_failure_analysis.json
│   ├── code_review_issues.json
│   ├── bug_categorization.json
│   └── priority_assessment.json
├── planning/
│   ├── bug_fix_plan.json
│   ├── priority_phases.json
│   ├── implementation_timeline.json
│   └── resource_allocation.json
├── prioritization/
│   ├── critical_issues.json
│   ├── high_priority_issues.json
│   ├── medium_priority_issues.json
│   └── low_priority_issues.json
├── risk_assessment/
│   ├── risk_analysis.json
│   ├── impact_assessment.json
│   ├── mitigation_strategies.json
│   └── rollback_plans.json
└── documentation/
    ├── bug_fix_plan.md
    ├── implementation_guide.md
    ├── testing_strategy.md
    └── deployment_notes.md
```

### Main Template: `bug_fix_plan.json`
```json
{
  "metadata": {
    "app_name": "string",
    "dd_name": "string", 
    "generated_at": "ISO datetime",
    "generated_by": "prompt_08",
    "version": "1.0",
    "source_reports": {
      "test_report_path": "string",
      "review_report_path": "string"
    }
  },
  "bug_analysis_summary": {
    "total_issues": "number",
    "test_failures": "number",
    "code_review_issues": "number",
    "security_issues": "number",
    "severity_breakdown": {
      "critical": "number",
      "high": "number",
      "medium": "number", 
      "low": "number"
    }
  },
  "bug_categorization": {
    "test_failure_categories": {
      "model_validation": [],
      "serializer_validation": [],
      "view_logic": [],
      "authentication": [],
      "permissions": [],
      "business_logic": [],
      "database_integrity": [],
      "api_contract": []
    },
    "code_review_categories": {
      "security_vulnerabilities": [],
      "code_quality": [],
      "performance": [],
      "django_best_practices": [],
      "dd_compliance": []
    }
  },
  "priority_phases": {
    "phase_1_critical": {
      "issues": [],
      "estimated_duration": "string",
      "priority_score_range": "70-100"
    },
    "phase_2_high": {
      "issues": [],
      "estimated_duration": "string", 
      "priority_score_range": "50-69"
    },
    "phase_3_medium": {
      "issues": [],
      "estimated_duration": "string",
      "priority_score_range": "30-49"
    },
    "phase_4_low": {
      "issues": [],
      "estimated_duration": "string",
      "priority_score_range": "0-29"
    }
  },
  "implementation_timeline": {
    "total_estimated_time": "string",
    "phase_schedule": [],
    "dependencies": [],
    "critical_path": []
  },
  "risk_assessment": {
    "high_risk_changes": [],
    "breaking_changes": [],
    "data_migration_needed": [],
    "rollback_complexity": "low|medium|high"
  },
  "recommendations": [],
  "next_steps": [
    "Run prompt 09 to execute bug fixes",
    "Review priority phases",
    "Prepare rollback procedures"
  ]
}
```

---

## Template Usage Guidelines

### 1. Input Template Specifications

Each prompt should specify required input templates:

```yaml
input_templates:
  required:
    - template_name: "dd_requirements"
      path: "design_details/"
      format: "markdown|json"
      description: "Original DD specification files"
  
  optional:
    - template_name: "previous_results"
      path: "outputs/prompt_XX/"
      format: "json"
      description: "Results from previous prompt"
```

### 2. Output Validation

Each generated output should include:
- Metadata validation
- Schema compliance check
- Required fields verification
- File structure validation

### 3. Template Versioning

Templates follow semantic versioning:
- Major: Breaking schema changes
- Minor: New optional fields
- Patch: Documentation updates

### 4. Cross-Prompt Compatibility

Templates ensure compatibility between prompts:
- Standardized field names
- Consistent data types
- Predictable file structures
- Clear dependency chains

---

## Implementation Notes

1. **Template Validation**: Each prompt should validate output against template schema
2. **Error Handling**: Clear error messages when template validation fails
3. **Documentation**: Each template includes usage examples and field descriptions
4. **Extensibility**: Templates allow for custom fields while maintaining core structure
5. **Tooling**: Consider JSON Schema validation for automated template checking