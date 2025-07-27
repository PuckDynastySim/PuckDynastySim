# Copilot Instructions - Hockey League Simulation System

## Project Overview

This is a comprehensive web-based hockey league simulation system that enables users to create and manage virtual hockey leagues with complete season management capabilities. The system supports multiple organizational levels and provides sophisticated simulation mechanics for realistic hockey gameplay and statistics tracking.

### Business Domain Context

The application serves three primary user roles within a hierarchical structure. League commissioners function as administrators with full system access, including league setup, team creation, user management, and global configuration. General managers operate individual teams with responsibilities for roster management, strategy configuration, player transactions, and financial operations. The system also supports AI-controlled teams for leagues without sufficient human participants.

The simulation encompasses three interconnected league tiers. The professional league represents the top competitive level where experienced players compete. The farm league serves as a developmental tier connected to professional teams, allowing player movement between levels. The junior league functions as the entry point for young players aged 16-21, with an 18-year minimum age requirement for advancement to higher tiers.

### Core System Features

Player management includes comprehensive generation systems that create fictional players with realistic names based on nationality demographics. Each player possesses detailed rating systems covering discipline, injury resistance, fatigue, passing, shooting, defense, puck control, checking, fighting, and poise. Goaltenders utilize specialized ratings for movement, rebound control, vision, aggressiveness, puck control, and flexibility. All ratings operate on a 25-99 scale with overall ratings calculated as statistical averages.

The simulation engine processes games using player and coach ratings to generate realistic outcomes. Each game produces detailed boxscores with period-by-period breakdowns, comprehensive play-by-play narratives, team statistics, and individual player performance metrics. The engine considers fatigue, injury probability, strategic configurations, and coaching effectiveness to create authentic hockey simulation experiences.

Financial systems incorporate salary cap management, revenue generation through ticket pricing, arena upgrade investments, and training facility expenditures. Teams operate within league-defined financial constraints while pursuing competitive advantages through strategic spending decisions.

## Technical Architecture

### Technology Stack

The application utilizes Next.js with React for the frontend framework, providing server-side rendering capabilities and dynamic user interface components. The backend employs Node.js with Express for API development and business logic implementation. PostgreSQL serves as the primary database with TimescaleDB extensions for efficient time-series data management of player statistics and game history.

Real-time functionality leverages WebSocket connections through Socket.IO for live game simulation updates, instant messaging between users, and notification delivery. The system implements multi-tenant architecture supporting both database-per-tenant isolation for large organizations and shared database configurations with row-level security for smaller leagues.

### Database Design Principles

The database schema follows normalized design patterns with specific accommodations for hockey domain requirements. Player statistics utilize JSONB columns for flexible attribute storage while maintaining query performance through GIN indexing. Historical data employs partitioning strategies by season to optimize query performance and data management.

Multi-tenant isolation operates at the league level, ensuring complete data separation between different hockey organizations. User authentication employs JWT tokens with role-based access control implementing hierarchical permission structures that align with hockey league organizational patterns.

## Development Standards

### Code Organization

The codebase follows feature-based modular architecture where each business domain contains its own components, services, types, and documentation. The admin feature encompasses league setup, team creation, player generation, schedule management, and system configuration. Team management includes roster operations, line configurations, strategy settings, financial management, and player transactions.

Simulation functionality covers game engine mechanics, statistical calculations, play-by-play generation, and result processing. Financial systems manage salary cap enforcement, revenue tracking, expense management, and financial reporting. Messaging features handle instant communication, trade negotiations, and system notifications.

### TypeScript Implementation

All code utilizes TypeScript for type safety and development efficiency. Interface definitions clearly specify data structures for players, teams, games, financial transactions, and user interactions. Generic types accommodate different league configurations while maintaining type safety across the application.

Enum definitions standardize values for player positions, team roles, game events, transaction types, and user permissions. Union types handle complex state management scenarios such as game phases, player statuses, and league progression states.

### API Design Patterns

REST API endpoints follow conventional HTTP methods with consistent response structures. Authentication middleware enforces role-based access control on all protected routes. Error handling provides detailed information for development while maintaining security in production environments.

WebSocket events utilize typed message structures for real-time communication. Event naming conventions clearly indicate the affected domain and action type, such as game simulation updates, roster changes, and trade notifications.

## Hockey-Specific Business Rules

### Player Generation Logic

Player generation algorithms consider nationality demographics to create realistic name distributions matching real-world hockey participation patterns. Age generation respects league tier requirements with appropriate skill distributions based on player development curves. Rating generation employs weighted random distributions that reflect natural talent variation while maintaining competitive balance.

Position assignment considers league roster requirements and ensures appropriate distribution across forward lines, defensive pairings, and goaltending positions. Handedness distribution follows hockey norms with approximately 70% left-handed shooters and 30% right-handed shooters for forwards and defensemen.

### Simulation Engine Mechanics

Game simulation processes utilize Monte Carlo methods to generate realistic event probabilities based on player ratings, team strategies, and situational factors. Shot generation considers offensive and defensive ratings along with goaltender effectiveness. Goal scoring probability calculations factor shooting skill, defensive pressure, and goaltender positioning.

Penalty assessment incorporates discipline ratings and game situation contexts. Injury calculations utilize injury resistance ratings combined with checking intensity and random event generation. Fatigue accumulation affects player performance throughout games and across multiple game periods.

### Financial System Rules

Salary cap enforcement operates as a hard constraint preventing roster moves that exceed league limits. Revenue calculation considers attendance levels affected by team performance, ticket pricing strategies, and market size factors. Training facility investments provide long-term player development benefits with costs distributed across multiple seasons.

Trade logic validates salary cap compliance, roster size requirements, and position balance before processing transactions. Contract negotiation considers player performance, market conditions, and team financial constraints.

## AI Assistant Guidelines

### Code Generation Preferences

When generating database queries, prioritize readability and performance with appropriate indexing considerations. Use parameterized queries to prevent SQL injection and implement connection pooling for scalability. For complex statistical calculations, provide clear variable naming and inline comments explaining hockey-specific logic.

React component generation should emphasize responsive design principles with mobile-first considerations. Implement loading states for all asynchronous operations and error boundaries for robust user experience. Use TypeScript interfaces for all prop definitions and state management.

### Testing Approach

Generate comprehensive test suites covering unit tests for business logic, integration tests for API endpoints, and end-to-end tests for critical user workflows. Hockey simulation accuracy tests should validate statistical distributions and game outcome realism. Financial calculation tests must verify salary cap enforcement and revenue calculation accuracy.

Mock external dependencies appropriately while maintaining realistic test data that reflects actual hockey scenarios. Include edge case testing for scenarios such as overtime games, playoff formats, and complex trade situations.

### Documentation Requirements

Generate inline documentation for complex algorithms, particularly simulation engine calculations and financial system logic. API documentation should include request/response examples with realistic hockey data. Component documentation must explain props, state management, and user interaction patterns.

Database schema documentation should clarify relationships between entities and explain constraints that enforce hockey business rules. Include migration scripts with clear explanations of schema changes and data transformation requirements.

### Performance Considerations

Optimize database queries for large datasets typical in hockey league management, including historical statistics and multi-season data. Implement appropriate caching strategies for frequently accessed data such as current standings and player statistics. Consider pagination for large result sets such as player lists and game histories.

Real-time features should minimize bandwidth usage while maintaining responsive user experience. WebSocket message optimization reduces network overhead during active game simulation periods.

This context enables AI-assisted development that aligns with both technical requirements and hockey domain expertise, ensuring generated code meets the specific needs of sports simulation while maintaining high quality standards.