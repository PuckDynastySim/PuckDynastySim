# API Documentation - Hockey League Simulation System

This comprehensive API documentation defines the REST endpoints, WebSocket events, and authentication patterns that enable client-server communication within the Hockey League Simulation System. The API design follows RESTful principles with consistent response structures and comprehensive error handling across all operations.

## Authentication and Authorization

### JWT Token Management

The authentication system utilizes JSON Web Tokens with role-based access control that implements hierarchical permission structures aligned with hockey league organizational patterns. All protected endpoints require valid JWT tokens provided through the Authorization header using Bearer token format.

Token expiration employs a dual-token approach with short-lived access tokens valid for 15 minutes and longer-lived refresh tokens valid for 30 days. Token refresh procedures enable seamless user experience while maintaining security through regular token rotation. Invalid or expired tokens receive consistent 401 Unauthorized responses with clear error messaging.

The token payload includes user identification, assigned roles, league associations, and team assignments that enable granular authorization decisions at the endpoint level. Role hierarchy encompasses Federation Administrators with system-wide access, League Commissioners with league-specific control, General Managers with team-specific permissions, and individual users with limited access to personal information.

### Role-Based Access Control

Permission validation occurs at multiple levels including endpoint access control, resource ownership verification, and action authorization based on user roles and organizational relationships. League isolation ensures complete data separation between different hockey organizations while enabling appropriate cross-league administrative functions.

Administrative endpoints require Federation Administrator or League Commissioner roles with verification of appropriate league association. Team management endpoints validate General Manager assignment to specific teams with additional verification for actions affecting other teams such as trade proposals and player acquisitions.

## Core API Endpoints

### User Management Operations

User authentication endpoints provide comprehensive account management capabilities including registration, login, password reset, and profile management operations. The registration process includes email verification and role assignment procedures that integrate with league invitation workflows.

```
POST /api/auth/register
Content-Type: application/json

{
  "email": "gm@example.com",
  "password": "securePassword123",
  "firstName": "John",
  "lastName": "Smith",
  "invitationCode": "league-invite-token"
}

Response: 201 Created
{
  "user": {
    "id": "user-uuid",
    "email": "gm@example.com",
    "firstName": "John",
    "lastName": "Smith",
    "roles": ["general_manager"],
    "leagues": ["league-uuid"],
    "teams": ["team-uuid"]
  },
  "tokens": {
    "accessToken": "jwt-access-token",
    "refreshToken": "jwt-refresh-token"
  }
}
```

Login operations validate credentials and return appropriate tokens with user profile information including role assignments and organizational associations. Password reset procedures utilize secure token generation with time-limited validity and email-based verification.

User profile management enables updates to personal information while maintaining audit trails for security monitoring. Role modification requires appropriate administrative permissions with comprehensive logging of all authorization changes.

### League Administration Endpoints

League creation and management endpoints provide comprehensive configuration capabilities for establishing new hockey leagues with customized rules, schedules, and organizational structures. League setup includes salary cap configuration, season parameters, playoff formats, and draft procedures that align with specific organizational requirements.

```
POST /api/admin/leagues
Authorization: Bearer jwt-token
Content-Type: application/json

{
  "name": "Professional Hockey League",
  "salaryCap": 85000000,
  "seasonLength": 82,
  "playoffTeams": 16,
  "draftRounds": 7,
  "conferences": [
    {
      "name": "Eastern Conference",
      "divisions": ["Atlantic", "Metropolitan"]
    },
    {
      "name": "Western Conference", 
      "divisions": ["Central", "Pacific"]
    }
  ]
}

Response: 201 Created
{
  "league": {
    "id": "league-uuid",
    "name": "Professional Hockey League",
    "settings": {
      "salaryCap": 85000000,
      "seasonLength": 82,
      "playoffTeams": 16,
      "draftRounds": 7
    },
    "structure": {
      "conferences": [...],
      "divisions": [...]
    },
    "status": "setup",
    "createdAt": "2024-01-15T10:00:00Z"
  }
}
```

Team creation endpoints enable systematic establishment of league participants with geographic assignments, conference alignments, and initial roster configurations. Team management includes logo uploads, arena information, and financial parameter establishment.

User invitation procedures enable league commissioners to add general managers with appropriate role assignments and team associations. Invitation management includes expiration handling and revocation capabilities for security control.

### Team Management Operations

Roster management endpoints provide comprehensive player transaction capabilities including signings, releases, trades, and position assignments. All roster operations include salary cap validation and league rule compliance verification before transaction completion.

```
PUT /api/teams/{teamId}/roster/{playerId}
Authorization: Bearer jwt-token
Content-Type: application/json

{
  "action": "sign",
  "contract": {
    "salary": 2500000,
    "length": 3,
    "position": "center"
  },
  "lineAssignment": {
    "line": 1,
    "position": "center"
  }
}

Response: 200 OK
{
  "transaction": {
    "id": "transaction-uuid",
    "type": "player_signing",
    "player": {
      "id": "player-uuid",
      "name": "Connor Example",
      "position": "center",
      "ratings": {...}
    },
    "contract": {...},
    "salaryCap": {
      "used": 78500000,
      "remaining": 6500000,
      "projected": 85000000
    },
    "effectiveDate": "2024-01-15T15:30:00Z"
  }
}
```

Strategy configuration endpoints enable tactical setup including line combinations, power play units, penalty kill formations, and goaltender rotation schedules. Strategy validation ensures proper position assignments and identifies potential conflicts or rule violations.

Financial management operations include budget tracking, revenue monitoring, and expense management with integration to league financial systems. Training facility and arena upgrade endpoints enable strategic investments with long-term impact calculations.

### Game Simulation Interface

Game simulation endpoints provide comprehensive control over league scheduling, game execution, and result processing. Simulation operations support both individual game processing and bulk season advancement with appropriate progress tracking and error handling.

```
POST /api/simulation/games/{gameId}/simulate
Authorization: Bearer jwt-token
Content-Type: application/json

{
  "mode": "detailed",
  "generatePlayByPlay": true,
  "generateBoxScore": true,
  "realTimeUpdates": true
}

Response: 202 Accepted
{
  "simulation": {
    "id": "simulation-uuid",
    "gameId": "game-uuid",
    "status": "in_progress",
    "progress": {
      "period": 1,
      "timeRemaining": "15:23",
      "homeScore": 1,
      "awayScore": 0
    },
    "websocketChannel": "game-simulation-uuid"
  }
}
```

Statistical tracking endpoints provide access to comprehensive player performance data, team analytics, and league-wide statistics with appropriate filtering and aggregation capabilities. Historical data access includes season comparisons and trend analysis.

Schedule management enables modification of game dates, venue assignments, and broadcast scheduling with appropriate conflict detection and resolution procedures. Playoff bracket management provides tournament progression tracking with automatic advancement procedures.

## WebSocket Event Specifications

### Real-Time Game Updates

WebSocket connections provide live game simulation updates including score changes, penalty assessments, player substitutions, and period transitions. Event payloads include comprehensive game state information with timestamp accuracy for client synchronization.

```
Event: game_update
Payload: {
  "gameId": "game-uuid",
  "timestamp": "2024-01-15T20:45:23Z",
  "event": {
    "type": "goal",
    "period": 2,
    "timeRemaining": "12:34",
    "player": {
      "id": "player-uuid",
      "name": "Connor Example",
      "team": "team-uuid"
    },
    "assists": [...],
    "description": "Wrist shot from the slot"
  },
  "gameState": {
    "homeScore": 2,
    "awayScore": 1,
    "period": 2,
    "timeRemaining": "12:34"
  }
}
```

User notification events include trade proposals, roster alerts, financial warnings, and system announcements with appropriate priority levels and delivery preferences. Notification management enables user customization of event types and delivery methods.

Chat message distribution supports both league-wide announcements and private communications between general managers with appropriate moderation capabilities and message history retention.

### Connection Management

WebSocket authentication utilizes the same JWT token system as REST endpoints with additional connection validation and periodic token refresh requirements. Connection pooling enables efficient resource management for multiple simultaneous users and game simulations.

Error handling provides clear indication of connection issues, authentication failures, and message delivery problems with appropriate reconnection procedures and message queuing for reliability.

## Error Handling and Response Formats

### Standardized Error Responses

All API endpoints utilize consistent error response formats that provide clear indication of failure causes and appropriate resolution guidance. Error codes align with HTTP standards while providing additional context through custom error types specific to hockey league management scenarios.

```
Response: 400 Bad Request
{
  "error": {
    "type": "salary_cap_violation",
    "message": "Transaction would exceed salary cap limit",
    "details": {
      "currentCap": 78500000,
      "transactionAmount": 8000000,
      "availableSpace": 6500000,
      "salaryCap": 85000000
    },
    "suggestions": [
      "Release other players to create cap space",
      "Negotiate lower salary amount",
      "Wait for next season cap increase"
    ]
  }
}
```

Validation error responses provide field-specific feedback for form submissions and data updates with clear indication of correction requirements. Business rule violations include detailed explanations of hockey-specific constraints and alternative resolution approaches.

Rate limiting responses indicate usage threshold violations with retry timing information and upgrade options for higher usage tiers. Authentication errors provide security-appropriate messaging while maintaining audit trail capabilities.

This API documentation provides the foundation for effective client application development while ensuring consistent integration patterns across all system components and user interfaces.