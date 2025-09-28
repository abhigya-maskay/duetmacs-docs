---
tags: [workflow, process, development, methodology]
aliases: [Workflow, Development Process, Feature Workflow]
---

# Adaptive Feature Development Workflow

## Phase 1: Feature Planning & Discovery
- **Epic/Feature Definition**: Define scope and business value
- **Feature Decomposition**: Break large features into smaller, manageable tasks with progressive refinement
- **User Flow Mapping** *(if user-facing feature)*: Map the user journey and identify all touchpoints for the feature
- **Requirements Validation**: Ensure feature requirements are clear, testable, and aligned with project goals

### Decision Point 1: Complexity & Scope Assessment
**Evaluate feature complexity to determine workflow path:**

- **Simple**: Single file, no DB/deps → Skip to Phase 4
- **Standard**: Multiple files, possible DB changes → Phase 2
- **Complex**: Multiple systems, major changes → Phase 2 + risk assessment

## Phase 2: Technical High-Level Planning
- **Codebase Analysis**: Analyze existing codebase patterns
- **Technical Decision Documentation**: Document key technical choices and their rationale
- **Non-functional Requirements**: Define security and maintainability requirements for the feature
- **Dependency Assessment**: Identify existing libraries, services, and APIs that will be used or affected
- **Risk & Impact Assessment**: Identify potential technical risks and impacts on existing functionality

### Decision Point 2: Risk Assessment
**If high-risk factors detected, complete ALL relevant sub-tasks before proceeding to Phase 3:**

- **External API Integrations**: Research APIs, build proof of concept, plan error handling
- **Security-Sensitive Areas**: Threat modeling, access control, data protection planning
- **Legacy System Modifications**: Analyze existing patterns, map dependencies, plan compatibility

## Phase 3: Code Unit Planning
- **Component Boundaries**: Identify functional units and define high-level responsibilities
- **Integration & Data Flow**: Plan system integration and data transformations
- **Testing Strategy**: Define unit, integration, and end-to-end testing approach

## Phase 4: Detailed Code Design
- **Technical Interface Design**: Define function signatures, data contracts, and detailed APIs
- **Code Organization**: Plan file structure, module organization, and naming conventions
- **Database Design**: Design schema changes, migrations, and data access patterns
- **Error Handling & Implementation Sequence**: Plan exception handling and implementation order

## Phase 5: Implementation & Delivery
- **Incremental Development with Testing**: Implement in small increments, writing tests alongside code following the planned sequence
- **Code Integration & Review**: Integrate with existing codebase following established patterns, self-review for quality and maintainability

## Cross-Cutting Practices (Throughout All Phases)
- **Code Quality & Documentation**: Follow project conventions, document decisions close to code
- **Security Awareness**: Consider security implications throughout

## Notes
This workflow is designed for individual developers working on features within a single repository. It emphasizes thorough planning and incremental implementation while maintaining code quality and integration with existing systems. The workflow scales from small feature additions to large feature implementations or modifications to existing functionality.

## Related Documentation
- [[Development-Workflow-Details]] - Detailed workflow implementation guide
- [[Decision Points]] - Decision framework and criteria
- [[Cross-Cutting Practices]] - Standards across all phases
