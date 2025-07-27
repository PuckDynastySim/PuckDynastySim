# Contributing to Hockey League Simulation System

This document establishes the development standards, workflows, and quality assurance procedures for contributing to the Hockey League Simulation System. These guidelines ensure consistent code quality, maintainable architecture, and effective collaboration across all development activities.

## Development Standards

### Code Organization Principles

The codebase follows feature-based modular architecture where each business domain maintains complete separation of concerns. Administrative functionality, team management capabilities, simulation engine components, financial systems, and messaging features operate as independent modules with clearly defined interfaces and minimal coupling between domains.

All code must utilize TypeScript with strict type checking enabled to ensure compile-time error detection and enhanced development experience. Interface definitions require comprehensive documentation explaining business context and usage patterns. Generic types should accommodate different league configurations while maintaining type safety across all application components.

Component architecture follows established React patterns with functional components utilizing hooks for state management. Custom hooks encapsulate complex business logic and provide reusable functionality across different user interface components. All components require default export with named interface definitions for props and clear documentation explaining intended usage scenarios.

### Naming Conventions and Structure

File naming follows kebab-case conventions for all directories and files to ensure cross-platform compatibility and consistent organization. Component files utilize PascalCase names that directly correspond to the exported component name. Service files employ descriptive names that clearly indicate their business domain and primary responsibility.

Function and variable naming employs camelCase with descriptive names that explain the purpose and context. Boolean variables begin with appropriate prefixes such as "is", "has", or "should" to indicate their nature. Constants utilize UPPER_SNAKE_CASE formatting with clear indication of their purpose and scope.

Database table and column names follow snake_case conventions that align with PostgreSQL best practices. Foreign key relationships employ descriptive names that clearly indicate the referenced table and purpose of the relationship. Index names include the table name and columns covered to facilitate maintenance and performance optimization.

### API Design Standards

REST API endpoints follow conventional HTTP methods with consistent response structures across all operations. Resource naming utilizes plural nouns with hierarchical relationships clearly expressed through URL structure. Version numbering employs semantic versioning principles with clear migration paths between different API versions.

Authentication middleware enforces role-based access control on all protected routes with consistent error messaging for unauthorized access attempts. Input validation operates at multiple levels including client-side validation for user experience and server-side validation for security and data integrity.

WebSocket events utilize typed message structures with clear naming conventions that indicate the affected domain and action type. Event payload structures maintain consistency with REST API response formats to ensure predictable data handling across different communication mechanisms.

## AI-Assisted Development Workflow

### Context Management Strategy

All development activities begin with comprehensive context establishment through review of relevant documentation files including feature-specific README files and architectural decision records. Developers must familiarize themselves with the .github/copilot-instructions.md file to understand global project context and AI assistant guidelines.

Feature development requires creation or updating of context documentation before code generation activities commence. This includes business requirement clarification, technical specification documentation, and expected outcome definition that enables effective AI assistance throughout the development process.

Code generation sessions should utilize structured prompts that reference established patterns and business rules documented within the project. Developers must review and validate all AI-generated code to ensure alignment with project standards and hockey domain requirements before committing changes to version control.

### Quality Assurance for AI-Generated Code

All AI-generated code requires human review and validation before integration into the main codebase. Review activities include verification of business logic correctness, adherence to established coding standards, and proper error handling implementation. Complex business rules related to hockey simulation mechanics receive additional scrutiny to ensure accuracy and realistic behavior.

Testing requirements for AI-generated code include comprehensive unit tests covering both expected functionality and edge cases relevant to hockey league management scenarios. Integration tests must validate proper interaction with existing system components and database operations.

Performance considerations require evaluation of AI-generated database queries, algorithm efficiency, and resource utilization patterns. Code that processes large datasets or operates in real-time contexts receives additional performance testing and optimization review.

## Testing Requirements

### Comprehensive Test Coverage

All new functionality requires complete test coverage including unit tests for individual functions and components, integration tests for API endpoints and database operations, and end-to-end tests for critical user workflows. Test coverage must maintain minimum thresholds of 90% for unit tests and 80% for integration tests.

Hockey simulation accuracy tests validate statistical distributions, game outcome realism, and player performance calculations against established hockey analytics benchmarks. Financial calculation tests verify salary cap enforcement, revenue calculation accuracy, and transaction processing correctness under various scenarios.

Multi-user interaction tests confirm proper isolation between different leagues and teams while validating real-time communication functionality during concurrent usage scenarios. Performance tests ensure system responsiveness under realistic load conditions including simultaneous game simulation and user interaction.

### Test Data Management

Test data utilizes realistic hockey scenarios including player statistics, team configurations, and league structures that reflect actual hockey league operations. Test databases employ consistent seeding procedures that create reproducible test environments for reliable continuous integration execution.

Mock data generation follows established patterns for player names, team identifications, and statistical distributions that maintain consistency with production data structures. External service mocking ensures test reliability while maintaining realistic response patterns for third-party integrations.

### Continuous Integration Standards

All pull requests trigger automated test execution including linting, type checking, unit tests, integration tests, and security vulnerability scanning. Build processes verify successful compilation and deployment preparation across different environment configurations.

Test execution must complete successfully before code review activities commence. Failed tests require investigation and resolution before additional review activities proceed. Test results documentation provides clear failure analysis and resolution guidance for common issues.

## Documentation Standards

### Code Documentation Requirements

All functions and methods require comprehensive documentation explaining purpose, parameters, return values, and potential exceptions. Complex algorithms, particularly those related to hockey simulation mechanics, require detailed explanations of mathematical formulas and business logic implementation.

Component documentation includes props interface definitions, state management patterns, and usage examples that demonstrate proper integration within larger application contexts. Custom hooks documentation explains intended usage scenarios and any dependencies or limitations.

Database schema documentation clarifies relationships between entities, explains constraints that enforce hockey business rules, and provides examples of common query patterns. Migration scripts include detailed explanations of schema changes and any required data transformation procedures.

### Feature Documentation Standards

Each new feature requires comprehensive documentation explaining business context, technical implementation approach, and integration patterns with existing system components. User workflow documentation includes step-by-step procedures for common operations and troubleshooting guidance for typical issues.

API documentation includes request and response examples with realistic hockey data that demonstrates proper usage patterns. Error handling documentation explains common failure scenarios and appropriate client-side response strategies.

Architecture decision records document significant technical choices including rationale, alternatives considered, and implications for future development activities. These records provide essential context for long-term system maintenance and enhancement planning.

## Pull Request Procedures

### Submission Requirements

Pull requests require detailed descriptions explaining the business context, technical approach, and testing methodology employed. All submissions must reference relevant issue numbers and include verification that automated testing completes successfully.

Code changes require appropriate documentation updates including README files, API documentation, and architectural decision records when applicable. Breaking changes require migration guides and clear communication regarding impact on existing functionality.

Large feature implementations require incremental submission through multiple pull requests that enable effective review and minimize integration risks. Each pull request should represent a logical unit of functionality that can be independently tested and validated.

### Review Process Standards

All pull requests require review by at least two team members with one reviewer possessing domain expertise in the affected system components. Review activities include verification of business logic correctness, adherence to coding standards, test coverage adequacy, and documentation completeness.

Security review focuses on authentication and authorization implementation, input validation procedures, and potential vulnerability identification. Performance review evaluates database query efficiency, algorithm complexity, and resource utilization patterns.

Hockey domain expertise review ensures realistic simulation behavior, accurate statistical calculations, and proper implementation of hockey-specific business rules. This specialized review validates that technical implementation aligns with hockey industry standards and user expectations.

## Quality Assurance Guidelines

### Performance Standards

All database queries must utilize appropriate indexing strategies and avoid N+1 query patterns that degrade performance under realistic data volumes. Real-time features require response time validation to ensure acceptable user experience during concurrent usage scenarios.

Memory usage patterns receive evaluation to prevent resource leaks and ensure stable operation during extended simulation periods. Client-side performance testing validates responsive user interface behavior across different device types and network conditions.

Scalability considerations require evaluation of system behavior under increasing user loads and data volumes typical of growing hockey league organizations. Load testing validates system stability and identifies potential bottlenecks before production deployment.

### Security Requirements

All user input requires validation and sanitization to prevent injection attacks and data corruption. Authentication and authorization mechanisms receive regular review to ensure proper access control enforcement across all system components.

Sensitive data handling follows established security protocols including encryption for data at rest and in transit. Audit logging captures all significant user actions and system events to enable security monitoring and compliance reporting.

Regular security dependency updates maintain protection against known vulnerabilities in third-party packages and frameworks. Security testing includes penetration testing and vulnerability scanning as part of regular quality assurance activities.

These contributing guidelines ensure consistent development practices that maintain high code quality while leveraging AI assistance effectively throughout the development lifecycle. Adherence to these standards enables sustainable development practices that support long-term system maintenance and enhancement activities.