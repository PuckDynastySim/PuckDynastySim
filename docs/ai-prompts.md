# AI-Assisted Development Prompts - Hockey League Simulation System

This comprehensive collection of AI prompts provides structured templates and best practices for generating high-quality code using AI assistants throughout the Hockey League Simulation System development process. These prompts are specifically optimized for hockey domain requirements and system architecture patterns.

## Database and Schema Development Prompts

### Entity Creation and Relationship Modeling

When creating new database entities for hockey league management, utilize structured prompts that establish clear business context and technical requirements. Begin with comprehensive entity descriptions that explain the hockey domain purpose and organizational relationships within league hierarchies.

**Entity Creation Template:**
"Create a PostgreSQL table for [hockey entity] that supports [business function] within a multi-tenant hockey league simulation system. The entity should include [specific fields] with appropriate constraints for [hockey-specific rules]. Consider relationships to existing entities including teams, players, leagues, and users. Include proper indexing for common query patterns including [specific access patterns]. Ensure compliance with multi-tenant row-level security policies and include audit trail fields for change tracking."

**Relationship Modeling Template:**
"Design database relationships between [entity A] and [entity B] for hockey league management, considering [business rules] and [operational constraints]. Include foreign key constraints, cascade behaviors, and junction tables where appropriate. Account for hockey-specific scenarios such as player trades, team relocations, and league restructuring. Provide migration scripts that handle existing data and maintain referential integrity throughout the schema changes."

### Query Optimization and Performance Tuning

Database query generation requires specific attention to hockey league data patterns including seasonal data access, statistical aggregation, and real-time game state queries. Optimize for common access patterns while maintaining flexibility for complex analytical requirements.

**Statistical Query Template:**
"Create an optimized PostgreSQL query to calculate [hockey statistic] for [time period] that efficiently handles large datasets with proper indexing utilization. Include appropriate filtering for league isolation, seasonal boundaries, and player eligibility. Consider performance implications for real-time dashboard updates and provide query execution plan analysis. Include alternative query approaches for different data volumes and access patterns."

## Component Development and User Interface Prompts

### React Component Generation

React component development requires careful attention to hockey domain requirements, responsive design principles, and real-time data integration. Focus on reusable components that maintain consistency across different user roles and organizational contexts.

**Component Architecture Template:**
"Create a React functional component for [hockey management function] that handles [specific user interactions] within the hockey league simulation system. The component should accept props for [data requirements] and maintain local state for [interface elements]. Include proper TypeScript interfaces, error boundary implementation, and loading states for asynchronous operations. Ensure responsive design for mobile and desktop access patterns. Integrate with WebSocket connections for real-time updates when appropriate."

**Form and Validation Template:**
"Develop a comprehensive form component for [hockey data entry] that includes client-side validation for [business rules] and server-side integration for [API endpoints]. Implement proper error handling, user feedback mechanisms, and accessibility features. Include hockey-specific validation such as salary cap compliance, roster size limits, and player eligibility requirements. Provide clear user guidance and prevent invalid submissions through appropriate constraint checking."

### Dashboard and Analytics Interface Development

Dashboard components require sophisticated data visualization capabilities combined with real-time updates and interactive analysis tools. Emphasize performance optimization for large datasets and complex statistical calculations.

**Dashboard Component Template:**
"Build a comprehensive dashboard component for [hockey analytics function] that displays [specific metrics] with interactive filtering and drill-down capabilities. Include real-time data updates through WebSocket integration and efficient rendering for large datasets. Implement responsive chart components with appropriate hockey context and comparative analysis features. Provide export capabilities and customizable display options for different user preferences and organizational requirements."

## API Development and Integration Prompts

### RESTful Endpoint Creation

API endpoint development requires comprehensive consideration of authentication, authorization, data validation, and hockey-specific business logic. Focus on consistent patterns and comprehensive error handling throughout all endpoint implementations.

**API Endpoint Template:**
"Create a RESTful API endpoint for [hockey management operation] that handles [HTTP method] requests with proper authentication and role-based authorization. Include comprehensive input validation for [data fields] with hockey-specific business rule enforcement. Implement appropriate error responses, success confirmation, and audit logging. Consider rate limiting, caching strategies, and integration with existing service layer patterns. Provide comprehensive API documentation with request/response examples."

### Service Layer and Business Logic Implementation

Service layer development coordinates complex hockey operations while maintaining clear separation between API controllers and database access patterns. Emphasize transaction management and error handling for critical business operations.

**Service Implementation Template:**
"Implement a service class for [hockey business operation] that coordinates [multiple data operations] while maintaining transaction integrity and comprehensive error handling. Include validation for [hockey-specific rules] and integration with [related services]. Provide clear interfaces for controller integration and comprehensive logging for audit requirements. Consider performance optimization for high-frequency operations and appropriate caching strategies for frequently accessed data."

## Simulation Engine and Algorithm Development Prompts

### Game Simulation Algorithm Implementation

Game simulation development requires sophisticated probability calculations combined with hockey domain expertise and performance optimization for real-time processing. Focus on realistic statistical distributions and authentic gameplay progression.

**Simulation Algorithm Template:**
"Develop a game simulation algorithm for [hockey game aspect] that utilizes player ratings for [specific attributes] to generate realistic [game events]. Include probability calculations based on [situational factors] and maintain statistical accuracy over large sample sizes. Consider performance optimization for real-time simulation and provide configurable parameters for different realism levels. Include comprehensive testing procedures for statistical validation and performance benchmarking."

### Statistical Calculation and Analysis Systems

Statistical systems require accurate mathematical implementations combined with efficient data processing for real-time calculations and historical analysis. Emphasize accuracy verification and performance optimization.

**Statistical System Template:**
"Create a statistical calculation system for [hockey metric] that processes [data sources] to generate [analytical outputs] with appropriate accuracy validation and performance optimization. Include historical trend analysis and comparative evaluation capabilities. Consider real-time calculation requirements and provide caching strategies for frequently accessed statistics. Include comprehensive testing for mathematical accuracy and edge case handling."

## Testing and Quality Assurance Prompts

### Comprehensive Test Suite Development

Test development requires coverage of both technical functionality and hockey domain accuracy. Focus on realistic test scenarios that reflect actual league management operations and edge cases.

**Test Suite Template:**
"Generate a comprehensive test suite for [system component] that includes unit tests for [functionality areas], integration tests for [API endpoints], and end-to-end tests for [user workflows]. Include hockey-specific test scenarios such as [domain examples] and edge cases including [boundary conditions]. Provide realistic test data that reflects actual hockey statistics and organizational structures. Include performance testing for high-load scenarios and data accuracy validation."

### Data Validation and Business Rule Testing

Hockey domain testing requires validation of complex business rules and statistical accuracy across various scenarios. Emphasize comprehensive coverage of rule combinations and exception handling.

**Domain Testing Template:**
"Create validation tests for [hockey business rules] that verify compliance with [specific regulations] across various scenarios including [edge cases]. Include statistical accuracy testing for [calculation systems] and performance validation for [high-load operations]. Provide comprehensive test data that covers different league configurations and organizational structures. Include regression testing for rule changes and system modifications."

## Documentation and Knowledge Management Prompts

### Technical Documentation Generation

Documentation development requires clear explanation of complex technical concepts combined with hockey domain context and practical usage examples. Focus on maintainability and accessibility for different technical skill levels.

**Documentation Template:**
"Create comprehensive technical documentation for [system component] that explains [functionality] with clear examples and hockey domain context. Include architecture decisions, implementation details, and integration patterns. Provide troubleshooting guidance and common usage scenarios. Consider different audience levels from developers to system administrators and include appropriate detail levels for each audience type."

These AI prompt templates provide the foundation for consistent, high-quality code generation while maintaining hockey domain expertise and system architecture alignment throughout the development process. Utilize these patterns as starting points for specific development tasks while adapting the templates to match particular requirements and contexts within the hockey league simulation system.