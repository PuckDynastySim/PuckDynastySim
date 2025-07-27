# Admin API Documentation - Hockey League Simulation System

This comprehensive API documentation defines all administrative endpoints for the Hockey League Simulation System. These endpoints provide federation administrators and league commissioners with complete control over league creation, user management, system configuration, and operational oversight while maintaining appropriate security and audit trail requirements.

## Authentication and Authorization

### Administrative Access Requirements

All administrative endpoints require authenticated users with appropriate administrative roles within the organizational hierarchy. Authentication utilizes JWT tokens with role-based access control that validates both user identity and administrative permissions before granting access to protected administrative functions.

**Required Headers:**
```http
Authorization: Bearer <jwt_access_token>
Content-Type: application/json
Accept: application/json
```

**Role-Based Access Levels:**
- **Federation Administrator**: System-wide access across all leagues and organizations
- **League Commissioner**: Full control within assigned leagues
- **Assistant Commissioner**: Limited administrative functions within specific leagues

**Authentication Example:**
```bash
curl -X GET "https://api.hockeysim.com/api/admin/users" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json"
```

## League Management Operations

### Create New League

Creates a new hockey league with comprehensive configuration including competitive parameters, financial constraints, and organizational structure while ensuring proper validation and initialization of all required league components.

**Endpoint:** `POST /api/admin/leagues`

**Required Permissions:** Federation Administrator

**Request Body:**
```json
{
  "name": "Professional Hockey League",
  "federationId": "federation-uuid",
  "leagueType": "professional",
  "salaryCap": {
    "enabled": true,
    "amount": 85000000,
    "luxuryTax": {
      "enabled": false,
      "threshold": 85000000,
      "rate": 0.50
    }
  },
  "seasonConfiguration": {
    "length": 82,
    "startDate": "2024-10-01",
    "endDate": "2025-04-15",
    "playoffTeams": 16,
    "playoffFormat": "best_of_series"
  },
  "rosterRules": {
    "minimumSize": 20,
    "maximumSize": 23,
    "maxContracts": 50,
    "dressedPlayers": 18
  },
  "draftConfiguration": {
    "rounds": 7,
    "reverseOrder": true,
    "lotteryEnabled": true,
    "eligibilityAge": 18
  },
  "structure": {
    "conferences": [
      {
        "name": "Eastern Conference",
        "divisions": [
          { "name": "Atlantic Division", "teams": 8 },
          { "name": "Metropolitan Division", "teams": 8 }
        ]
      },
      {
        "name": "Western Conference", 
        "divisions": [
          { "name": "Central Division", "teams": 8 },
          { "name": "Pacific Division", "teams": 8 }
        ]
      }
    ]
  },
  "financialSettings": {
    "revenueSharing": {
      "enabled": true,
      "percentage": 0.15,
      "method": "performance_based"
    },
    "minimumPayroll": 60000000,
    "contractRules": {
      "maxLength": 8,
      "maxAnnualValue": 15000000,
      "buyoutRules": "standard"
    }
  }
}
```

**Response:**
```json
{
  "success": true,
  "league": {
    "id": "league-uuid",
    "name": "Professional Hockey League",
    "federationId": "federation-uuid",
    "status": "setup",
    "currentSeason": null,
    "settings": {
      "salaryCap": {
        "enabled": true,
        "amount": 85000000
      },
      "seasonLength": 82,
      "playoffTeams": 16
    },
    "structure": {
      "totalTeams": 32,
      "conferences": 2,
      "divisions": 4
    },
    "createdAt": "2024-01-15T10:00:00Z",
    "createdBy": "admin-user-uuid"
  },
  "nextSteps": [
    "Create teams and assign to divisions",
    "Generate or import player database",
    "Assign general managers to teams",
    "Configure season schedule"
  ]
}
```

### Update League Configuration

Modifies existing league settings including competitive parameters, financial rules, and organizational structure while maintaining data integrity and providing migration support for configuration changes that affect ongoing operations.

**Endpoint:** `PUT /api/admin/leagues/{leagueId}`

**Required Permissions:** Federation Administrator or League Commissioner

**Request Body:**
```json
{
  "salaryCap": {
    "amount": 87000000,
    "effectiveDate": "2024-07-01"
  },
  "rosterRules": {
    "maximumSize": 25
  },
  "seasonConfiguration": {
    "playoffTeams": 20
  }
}
```

**Response:**
```json
{
  "success": true,
  "league": {
    "id": "league-uuid",
    "updatedSettings": {
      "salaryCap": 87000000,
      "maxRosterSize": 25,
      "playoffTeams": 20
    },
    "changeLog": [
      {
        "field": "salaryCap",
        "oldValue": 85000000,
        "newValue": 87000000,
        "effectiveDate": "2024-07-01"
      }
    ]
  },
  "impactAnalysis": {
    "teamsAffected": 3,
    "salaryCapViolations": 0,
    "rosterAdjustmentsRequired": 5
  }
}
```

### Delete League

Permanently removes a league and all associated data including teams, players, contracts, and historical records while ensuring proper data cleanup and maintaining referential integrity throughout the deletion process.

**Endpoint:** `DELETE /api/admin/leagues/{leagueId}`

**Required Permissions:** Federation Administrator

**Query Parameters:**
- `confirm`: Boolean (required) - Must be true to proceed with deletion
- `transferData`: Boolean (optional) - Whether to offer data export before deletion

**Response:**
```json
{
  "success": true,
  "message": "League permanently deleted",
  "deletionSummary": {
    "leagueId": "league-uuid",
    "teamsDeleted": 32,
    "playersDeleted": 960,
    "contractsDeleted": 736,
    "gamesDeleted": 1312,
    "backupCreated": "backup-file-uuid"
  }
}
```

## User Management and Role Assignment

### Create User Accounts

Creates new user accounts with appropriate role assignments and organizational associations while ensuring proper authentication setup and security validation throughout the account creation process.

**Endpoint:** `POST /api/admin/users`

**Required Permissions:** Federation Administrator or League Commissioner (within assigned leagues)

**Request Body:**
```json
{
  "personalInfo": {
    "email": "gm@example.com",
    "firstName": "John",
    "lastName": "Smith",
    "timezone": "America/New_York",
    "locale": "en-US"
  },
  "authentication": {
    "temporaryPassword": "TempPass123!",
    "requirePasswordChange": true,
    "mfaRequired": false
  },
  "roles": [
    {
      "type": "general_manager",
      "federationId": "federation-uuid",
      "leagueId": "league-uuid",
      "teamId": "team-uuid",
      "permissions": {
        "roster_management": true,
        "financial_management": true,
        "trade_negotiation": true,
        "draft_participation": true
      }
    }
  ],
  "notifications": {
    "email": true,
    "browser": true,
    "mobile": false,
    "preferences": {
      "trade_offers": "immediate",
      "roster_alerts": "daily_digest",
      "league_announcements": "immediate"
    }
  }
}
```

**Response:**
```json
{
  "success": true,
  "user": {
    "id": "user-uuid",
    "email": "gm@example.com",
    "firstName": "John",
    "lastName": "Smith",
    "status": "pending_activation",
    "roles": [
      {
        "type": "general_manager",
        "leagueId": "league-uuid",
        "teamId": "team-uuid",
        "assignedAt": "2024-01-15T10:30:00Z"
      }
    ],
    "createdAt": "2024-01-15T10:30:00Z"
  },
  "activation": {
    "method": "email_link",
    "expiresAt": "2024-01-16T10:30:00Z",
    "emailSent": true
  }
}
```

### Modify User Roles

Updates user role assignments and permissions while maintaining audit trails and ensuring proper authorization validation throughout role modification procedures and organizational boundary enforcement.

**Endpoint:** `PUT /api/admin/users/{userId}/roles`

**Required Permissions:** Federation Administrator or League Commissioner (for roles within their jurisdiction)

**Request Body:**
```json
{
  "roleChanges": [
    {
      "action": "add",
      "role": {
        "type": "league_commissioner",
        "leagueId": "league-uuid",
        "permissions": {
          "user_management": true,
          "team_creation": true,
          "schedule_management": true,
          "rule_enforcement": true
        },
        "effectiveDate": "2024-02-01"
      }
    },
    {
      "action": "modify",
      "roleId": "existing-role-uuid",
      "permissions": {
        "financial_management": false
      }
    }
  ],
  "reason": "Promotion to league commissioner role"
}
```

**Response:**
```json
{
  "success": true,
  "user": {
    "id": "user-uuid",
    "email": "user@example.com",
    "roles": [
      {
        "id": "role-uuid",
        "type": "league_commissioner",
        "leagueId": "league-uuid",
        "permissions": {
          "user_management": true,
          "team_creation": true,
          "schedule_management": true,
          "rule_enforcement": true
        },
        "assignedAt": "2024-01-15T10:30:00Z",
        "assignedBy": "admin-user-uuid"
      }
    ]
  },
  "auditTrail": [
    {
      "action": "role_added",
      "roleType": "league_commissioner",
      "timestamp": "2024-01-15T10:30:00Z",
      "performedBy": "admin-user-uuid",
      "reason": "Promotion to league commissioner role"
    }
  ]
}
```

### User Activity and Audit Logs

Retrieves comprehensive user activity logs and audit trails for security monitoring, compliance reporting, and administrative oversight of user actions throughout system operations and league management activities.

**Endpoint:** `GET /api/admin/users/{userId}/activity`

**Required Permissions:** Federation Administrator or League Commissioner

**Query Parameters:**
- `startDate`: ISO date string (optional)
- `endDate`: ISO date string (optional)
- `actionType`: String (optional) - Filter by action type
- `limit`: Number (optional, default: 50)
- `offset`: Number (optional, default: 0)

**Response:**
```json
{
  "success": true,
  "user": {
    "id": "user-uuid",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Smith"
  },
  "activityLog": [
    {
      "id": "log-uuid",
      "timestamp": "2024-01-15T14:30:00Z",
      "action": "player_trade_proposed",
      "details": {
        "tradeId": "trade-uuid",
        "playersInvolved": ["player1-uuid", "player2-uuid"],
        "teamsInvolved": ["team1-uuid", "team2-uuid"]
      },
      "ipAddress": "192.168.1.100",
      "userAgent": "Mozilla/5.0...",
      "result": "success"
    },
    {
      "id": "log-uuid-2",
      "timestamp": "2024-01-15T13:15:00Z",
      "action": "roster_lineup_changed",
      "details": {
        "teamId": "team-uuid",
        "changesCount": 3,
        "positionsAffected": ["left_wing", "center"]
      },
      "result": "success"
    }
  ],
  "summary": {
    "totalActions": 247,
    "dateRange": {
      "start": "2024-01-01T00:00:00Z",
      "end": "2024-01-15T23:59:59Z"
    },
    "actionsByType": {
      "roster_changes": 45,
      "trade_activities": 12,
      "financial_transactions": 8,
      "administrative_actions": 182
    }
  }
}
```

## Team Creation and Management

### Create New Teams

Establishes new teams within leagues including organizational information, facility details, financial configurations, and initial personnel assignments while ensuring proper league integration and competitive balance maintenance.

**Endpoint:** `POST /api/admin/teams`

**Required Permissions:** Federation Administrator or League Commissioner

**Request Body:**
```json
{
  "basicInfo": {
    "name": "Metropolitan Thunderbirds",
    "city": "New York",
    "abbreviation": "NYT",
    "leagueId": "league-uuid",
    "divisionId": "division-uuid"
  },
  "branding": {
    "primaryColor": "#FF6B00",
    "secondaryColor": "#003366",
    "logoUrl": "https://cdn.example.com/logos/nyt-logo.png",
    "foundedYear": 2024
  },
  "arena": {
    "name": "Thunder Arena",
    "capacity": 18500,
    "location": {
      "address": "123 Hockey Blvd",
      "city": "New York",
      "state": "NY",
      "zipCode": "10001"
    },
    "amenities": [
      "luxury_suites",
      "club_level",
      "family_zone",
      "concourse_dining"
    ]
  },
  "financial": {
    "initialBudget": 100000000,
    "ticketPricing": {
      "lowerBowl": 85.00,
      "upperBowl": 45.00,
      "club": 150.00,
      "suites": 2500.00
    },
    "seasonTicketDiscount": 0.15
  },
  "management": {
    "generalManagerId": "user-uuid",
    "aiControlled": false,
    "difficultyLevel": null
  }
}
```

**Response:**
```json
{
  "success": true,
  "team": {
    "id": "team-uuid",
    "name": "Metropolitan Thunderbirds",
    "city": "New York",
    "abbreviation": "NYT",
    "leagueId": "league-uuid",
    "divisionId": "division-uuid",
    "status": "active",
    "arena": {
      "name": "Thunder Arena",
      "capacity": 18500,
      "revenueProjection": {
        "annual": 15750000,
        "perGame": 192073
      }
    },
    "financial": {
      "currentBudget": 100000000,
      "salaryCapUsage": 0,
      "projectedRevenue": 15750000
    },
    "roster": {
      "currentSize": 0,
      "maxSize": 23,
      "openPositions": "all"
    },
    "createdAt": "2024-01-15T11:00:00Z"
  },
  "nextSteps": [
    "Assign general manager if not specified",
    "Begin player acquisition through draft or free agency",
    "Configure team strategies and preferences",
    "Set up training facilities and staff"
  ]
}
```

### Bulk Team Operations

Performs batch operations on multiple teams including mass updates, bulk configuration changes, and coordinated organizational modifications while maintaining data consistency and operational efficiency throughout multi-team management procedures.

**Endpoint:** `POST /api/admin/teams/bulk`

**Required Permissions:** Federation Administrator or League Commissioner

**Request Body:**
```json
{
  "operation": "update_salary_cap",
  "teams": ["team1-uuid", "team2-uuid", "team3-uuid"],
  "parameters": {
    "newSalaryCap": 87000000,
    "effectiveDate": "2024-07-01",
    "adjustmentReason": "League-wide salary cap increase"
  },
  "options": {
    "validateCompliance": true,
    "generateReport": true,
    "notifyGeneralManagers": true
  }
}
```

**Response:**
```json
{
  "success": true,
  "operation": {
    "id": "bulk-operation-uuid",
    "type": "update_salary_cap",
    "status": "completed",
    "summary": {
      "teamsProcessed": 3,
      "successful": 3,
      "failed": 0,
      "warnings": 1
    }
  },
  "results": [
    {
      "teamId": "team1-uuid",
      "teamName": "Team Alpha",
      "status": "success",
      "oldSalaryCap": 85000000,
      "newSalaryCap": 87000000,
      "complianceStatus": "compliant"
    },
    {
      "teamId": "team2-uuid", 
      "teamName": "Team Beta",
      "status": "success",
      "oldSalaryCap": 85000000,
      "newSalaryCap": 87000000,
      "complianceStatus": "compliant"
    },
    {
      "teamId": "team3-uuid",
      "teamName": "Team Gamma", 
      "status": "success",
      "oldSalaryCap": 85000000,
      "newSalaryCap": 87000000,
      "complianceStatus": "warning",
      "warnings": ["Currently using 86.5M, close to new limit"]
    }
  ],
  "notifications": {
    "emailsSent": 3,
    "systemNotifications": 3
  }
}
```

## Player Generation and Management

### Generate Player Database

Creates comprehensive player databases with realistic attributes, demographic distributions, and career development profiles while ensuring statistical accuracy and competitive balance throughout player generation and league integration processes.

**Endpoint:** `POST /api/admin/players/generate`

**Required Permissions:** Federation Administrator or League Commissioner

**Request Body:**
```json
{
  "leagueId": "league-uuid",
  "generationParameters": {
    "totalPlayers": 1500,
    "leagueTier": "professional",
    "ageDistribution": {
      "young": { "min": 18, "max": 23, "percentage": 0.25 },
      "prime": { "min": 24, "max": 29, "percentage": 0.45 },
      "veteran": { "min": 30, "max": 38, "percentage": 0.30 }
    },
    "positionDistribution": {
      "left_wing": 0.15,
      "center": 0.15,
      "right_wing": 0.15,
      "left_defense": 0.15,
      "right_defense": 0.15,
      "goaltender": 0.25
    },
    "skillDistribution": {
      "elite": { "ratingRange": [90, 99], "percentage": 0.05 },
      "star": { "ratingRange": [80, 89], "percentage": 0.15 },
      "topSix": { "ratingRange": [70, 79], "percentage": 0.25 },
      "nhLevel": { "ratingRange": [60, 69], "percentage": 0.35 },
      "depth": { "ratingRange": [50, 59], "percentage": 0.15 },
      "fringe": { "ratingRange": [25, 49], "percentage": 0.05 }
    },
    "nationalityDistribution": {
      "CAN": 0.45, "USA": 0.25, "SWE": 0.08, "FIN": 0.05,
      "RUS": 0.04, "CZE": 0.03, "SVK": 0.02, "OTHER": 0.08
    }
  },
  "options": {
    "generateProspects": true,
    "createDraftEligible": true,
    "includeInjuredReserve": false,
    "validateRatings": true
  }
}
```

**Response:**
```json
{
  "success": true,
  "generation": {
    "id": "generation-uuid",
    "status": "completed",
    "summary": {
      "totalPlayersGenerated": 1500,
      "processingTime": "45.2 seconds",
      "validationPassed": true
    },
    "distribution": {
      "byPosition": {
        "left_wing": 225,
        "center": 225,
        "right_wing": 225,
        "left_defense": 225,
        "right_defense": 225,
        "goaltender": 375
      },
      "byAge": {
        "18-23": 375,
        "24-29": 675,
        "30-38": 450
      },
      "byNationality": {
        "CAN": 675,
        "USA": 375,
        "SWE": 120,
        "FIN": 75,
        "RUS": 60,
        "CZE": 45,
        "SVK": 30,
        "OTHER": 120
      },
      "bySkillLevel": {
        "elite": 75,
        "star": 225,
        "topSix": 375,
        "nhLevel": 525,
        "depth": 225,
        "fringe": 75
      }
    },
    "validation": {
      "ratingDistribution": "passed",
      "nameGeneration": "passed",
      "positionBalance": "passed",
      "ageRealism": "passed"
    }
  },
  "nextSteps": [
    "Review generated player database",
    "Conduct dispersal draft if needed",
    "Assign free agents to available player pool",
    "Configure draft eligible players"
  ]
}
```

### Player Database Management

Manages existing player databases including attribute modifications, career progression updates, and database maintenance operations while ensuring data integrity and competitive balance throughout player management activities.

**Endpoint:** `PUT /api/admin/players/database`

**Required Permissions:** Federation Administrator or League Commissioner

**Request Body:**
```json
{
  "operation": "update_ratings",
  "filters": {
    "leagueId": "league-uuid",
    "ageRange": { "min": 30, "max": 38 },
    "ratingThreshold": { "min": 70 }
  },
  "modifications": {
    "agingCurve": {
      "enabled": true,
      "declineRate": 0.02,
      "affectedRatings": ["speed", "acceleration", "endurance"]
    },
    "injuryProneness": {
      "increase": 0.01,
      "basedOnAge": true
    }
  },
  "validation": {
    "preserveElitePlayers": true,
    "maintainPositionBalance": true,
    "validateStatisticalDistribution": true
  }
}
```

**Response:**
```json
{
  "success": true,
  "operation": {
    "id": "update-operation-uuid",
    "type": "update_ratings",
    "status": "completed",
    "playersAffected": 156,
    "summary": {
      "ratingChanges": {
        "decreased": 143,
        "unchanged": 13,
        "increased": 0
      },
      "averageDecline": 1.8,
      "significantChanges": 23
    }
  },
  "validation": {
    "distributionMaintained": true,
    "balancePreserved": true,
    "elitePlayersProtected": true
  },
  "affectedPlayers": [
    {
      "playerId": "player-uuid",
      "name": "John Veteran",
      "age": 35,
      "position": "center",
      "ratingChanges": {
        "overall": { "old": 78, "new": 76 },
        "speed": { "old": 75, "new": 73 },
        "endurance": { "old": 80, "new": 78 }
      }
    }
  ]
}
```

## Schedule Generation and Management

### Generate Season Schedule

Creates comprehensive season schedules including regular season games, playoff tournaments, and special events while optimizing for competitive balance, travel efficiency, and venue availability throughout the scheduling process.

**Endpoint:** `POST /api/admin/schedule/generate`

**Required Permissions:** Federation Administrator or League Commissioner

**Request Body:**
```json
{
  "leagueId": "league-uuid",
  "season": 2024,
  "parameters": {
    "seasonStart": "2024-10-01",
    "seasonEnd": "2025-04-15",
    "regularSeasonGames": 82,
    "gameFrequency": {
      "gamesPerWeek": 3,
      "preferredDays": ["Tuesday", "Thursday", "Saturday"],
      "avoidDays": ["Monday"]
    },
    "travelOptimization": {
      "enabled": true,
      "maxConsecutiveRoadGames": 6,
      "homestandPreference": 3,
      "restDaysRequired": 1
    },
    "competitiveBalance": {
      "intraDivisionGames": 4,
      "intraConferenceGames": 3,
      "interConferenceGames": 2,
      "randomizationSeed": 12345
    },
    "specialEvents": [
      {
        "name": "Winter Classic",
        "date": "2025-01-01",
        "participants": ["team1-uuid", "team2-uuid"],
        "venue": "outdoor"
      }
    ]
  },
  "constraints": {
    "arenaAvailability": {
      "checkConflicts": true,
      "alternativeVenues": false
    },
    "televisionWindows": {
      "nationalGames": 15,
      "preferredTimeSlots": ["19:00", "20:00"]
    },
    "holidayConsiderations": {
      "avoidChristmas": true,
      "reducedNewYear": true,
      "thanksgivingBreak": 2
    }
  }
}
```

**Response:**
```json
{
  "success": true,
  "schedule": {
    "id": "schedule-uuid",
    "leagueId": "league-uuid",
    "season": 2024,
    "status": "generated",
    "summary": {
      "totalGames": 1312,
      "teamsIncluded": 32,
      "seasonSpan": {
        "start": "2024-10-01",
        "end": "2025-04-15",
        "totalDays": 196
      },
      "optimization": {
        "travelEfficiency": 87.3,
        "competitiveBalance": 94.1,
        "venueUtilization": 91.8
      }
    },
    "statistics": {
      "gamesPerTeam": 82,
      "homeGamesPerTeam": 41,
      "awayGamesPerTeam": 41,
      "averageRestDays": 2.1,
      "backToBackGames": 156,
      "weekendGames": 524
    },
    "specialEvents": [
      {
        "name": "Winter Classic",
        "date": "2025-01-01",
        "gameId": "special-game-uuid",
        "venue": "Outdoor Stadium"
      }
    ]
  },
  "validation": {
    "constraintsSatisfied": true,
    "balanceAchieved": true,
    "conflictsResolved": true,
    "qualityScore": 93.7
  },
  "nextSteps": [
    "Review and approve schedule",
    "Publish to teams and media",
    "Configure television broadcasting",
    "Set up ticketing systems"
  ]
}
```

### Schedule Modifications

Modifies existing schedules including game rescheduling, venue changes, and special event integration while maintaining competitive balance and minimizing disruption to league operations and team preparations.

**Endpoint:** `PUT /api/admin/schedule/{scheduleId}`

**Required Permissions:** Federation Administrator or League Commissioner

**Request Body:**
```json
{
  "modifications": [
    {
      "type": "reschedule_game",
      "gameId": "game-uuid",
      "newDate": "2024-12-20T19:00:00Z",
      "reason": "Arena conflict resolution",
      "affectedTeams": ["team1-uuid", "team2-uuid"]
    },
    {
      "type": "venue_change",
      "gameId": "game-uuid-2",
      "newVenue": "Alternate Arena",
      "reason": "Emergency venue change"
    },
    {
      "type": "add_special_event",
      "event": {
        "name": "All-Star Game",
        "date": "2025-02-15",
        "participants": "selected_players",
        "venue": "All-Star Arena"
      }
    }
  ],
  "options": {
    "notifyTeams": true,
    "updateBroadcastSchedule": true,
    "recalculateBalance": true
  }
}
```

**Response:**
```json
{
  "success": true,
  "schedule": {
    "id": "schedule-uuid",
    "modificationsApplied": 3,
    "summary": {
      "gamesRescheduled": 1,
      "venuesChanged": 1,
      "eventsAdded": 1
    },
    "impact": {
      "teamsAffected": 4,
      "newRestDaysAnalysis": {
        "improved": 2,
        "unchanged": 28,
        "degraded": 2
      },
      "competitiveBalanceChange": 0.2
    }
  },
  "notifications": {
    "teamsSent": 4,
    "mediaSent": true,
    "broadcastersNotified": true
  },
  "conflicts": {
    "resolved": 2,
    "remaining": 0
  }
}
```

## System Configuration and Settings

### League Rule Configuration

Configures comprehensive league rules including game regulations, financial constraints, roster requirements, and operational procedures while ensuring rule consistency and competitive integrity throughout league operations.

**Endpoint:** `PUT /api/admin/leagues/{leagueId}/rules`

**Required Permissions:** Federation Administrator or League Commissioner

**Request Body:**
```json
{
  "gameRules": {
    "periods": 3,
    "periodLength": 20,
    "overtimeLength": 5,
    "overtimeFormat": "3v3",
    "shootoutRounds": 3,
    "penaltyDurations": {
      "minor": 2,
      "major": 5,
      "misconduct": 10,
      "gameConduct": "ejection"
    }
  },
  "rosterRules": {
    "activeRosterSize": 23,
    "dressedPlayers": 18,
    "minimumGoaltenders": 2,
    "callUpRules": {
      "emergencyOnly": false,
      "paperTransactions": true,
      "salaryCapImpact": true
    }
  },
  "financialRules": {
    "salaryCap": {
      "enabled": true,
      "amount": 85000000,
      "overageAllowed": 0,
      "offseasonExemption": 1.1
    },
    "contractRules": {
      "minimumSalary": 700000,
      "maximumLength": 8,
      "buyoutPermitted": true,
      "frontLoadingLimit": 0.35,
      "backDivingLimit": 0.50
    },
    "luxuryTax": {
      "enabled": false,
      "threshold": 85000000,
      "rates": [0.20, 0.30, 0.50, 1.00]
    }
  },
  "tradeRules": {
    "deadline": "2025-03-01T15:00:00Z",
    "maxPlayersPerTrade": 10,
    "retainedSalarySlots": 3,
    "retainedSalaryPercentage": 0.50,
    "noTradeClauseRespected": true,
    "draftPickTradingYears": 7
  },
  "draftRules": {
    "eligibilityAge": 18,
    "rounds": 7,
    "timePerPick": 300,
    "tradingAllowed": true,
    "lotteryTeams": 15,
    "lotteryDraws": 3
  }
}
```

**Response:**
```json
{
  "success": true,
  "rules": {
    "leagueId": "league-uuid",
    "version": "2024.1",
    "effectiveDate": "2024-07-01T00:00:00Z",
    "updateSummary": {
      "sectionsModified": 5,
      "rulesChanged": 12,
      "newRulesAdded": 3
    },
    "validation": {
      "ruleConsistency": "passed",
      "competitiveBalance": "maintained",
      "financialIntegrity": "validated"
    }
  },
  "impact": {
    "teamsRequiringAdjustment": 2,
    "contractsAffected": 15,
    "immediateActions": [
      "Update salary cap calculations",
      "Notify teams of rule changes",
      "Adjust contract validation systems"
    ]
  },
  "changeLog": [
    {
      "section": "financialRules.salaryCap.amount",
      "oldValue": 82500000,
      "newValue": 85000000,
      "reason": "Annual salary cap adjustment"
    }
  ]
}
```

### System Feature Configuration

Manages system-wide feature toggles, performance settings, and operational parameters while ensuring proper deployment coordination and maintaining system stability throughout configuration changes.

**Endpoint:** `PUT /api/admin/system/configuration`

**Required Permissions:** Federation Administrator

**Request Body:**
```json
{
  "features": {
    "realTimeSimulation": {
      "enabled": true,
      "maxConcurrentGames": 50,
      "updateFrequency": 1000
    },
    "advancedAnalytics": {
      "enabled": true,
      "historicalDataAccess": true,
      "predictionModels": true
    },
    "socialFeatures": {
      "messaging": true,
      "tradeNegotiation": true,
      "leagueForums": false
    },
    "mobileApi": {
      "enabled": true,
      "pushNotifications": true,
      "offlineMode": false
    }
  },
  "performance": {
    "databaseConnections": {
      "maxPoolSize": 50,
      "connectionTimeout": 30000
    },
    "caching": {
      "redisTtl": 3600,
      "applicationCache": true,
      "cdnCaching": true
    },
    "simulation": {
      "parallelProcessing": true,
      "maxWorkerThreads": 8,
      "memoryLimit": "4GB"
    }
  },
  "security": {
    "rateLimiting": {
      "enabled": true,
      "requestsPerMinute": 100,
      "burstAllowance": 20
    },
    "sessionSecurity": {
      "timeoutMinutes": 120,
      "concurrentSessions": 3,
      "ipValidation": true
    }
  }
}
```

**Response:**
```json
{
  "success": true,
  "configuration": {
    "version": "system-config-v2024.3",
    "appliedAt": "2024-01-15T12:00:00Z",
    "summary": {
      "featuresEnabled": 8,
      "featuresDisabled": 2,
      "performanceSettingsUpdated": 7,
      "securityPoliciesModified": 3
    }
  },
  "deploymentStatus": {
    "rolloutStrategy": "gradual",
    "serversUpdated": 5,
    "pendingDeployments": 0,
    "estimatedCompletion": "2024-01-15T12:15:00Z"
  },
  "monitoring": {
    "healthChecksPassing": true,
    "performanceMetrics": "stable",
    "errorRatesNormal": true
  }
}
```

## Data Export and Backup Operations

### League Data Export

Generates comprehensive data exports including all league information, statistical records, and operational data while ensuring data integrity and providing flexible export formats for various use cases and integration requirements.

**Endpoint:** `POST /api/admin/export/{leagueId}`

**Required Permissions:** Federation Administrator or League Commissioner

**Request Body:**
```json
{
  "exportType": "comprehensive",
  "format": "json",
  "includeData": {
    "teams": true,
    "players": true,
    "contracts": true,
    "games": true,
    "statistics": true,
    "financial": true,
    "historical": true
  },
  "filters": {
    "seasons": ["2023", "2024"],
    "dateRange": {
      "start": "2023-10-01",
      "end": "2024-04-30"
    },
    "dataTypes": ["active_records", "archived_records"]
  },
  "options": {
    "compression": true,
    "encryption": true,
    "anonymizePersonalData": false,
    "includeSystemMetadata": true
  }
}
```

**Response:**
```json
{
  "success": true,
  "export": {
    "id": "export-uuid",
    "status": "processing",
    "estimatedCompletion": "2024-01-15T12:30:00Z",
    "summary": {
      "leagueId": "league-uuid",
      "recordsIncluded": {
        "teams": 32,
        "players": 960,
        "contracts": 736,
        "games": 1312,
        "statistics": 45680,
        "financialRecords": 2847
      },
      "dataSize": "2.3 GB",
      "compressionRatio": 0.31
    },
    "downloadInfo": {
      "availableAt": "2024-01-15T12:30:00Z",
      "expiresAt": "2024-01-22T12:30:00Z",
      "downloadUrl": "https://exports.hockeysim.com/download/export-uuid",
      "accessToken": "temp-access-token"
    }
  },
  "security": {
    "encrypted": true,
    "accessRestricted": true,
    "auditLogged": true
  }
}
```

### System Backup Management

Manages comprehensive system backups including database snapshots, configuration backups, and disaster recovery procedures while ensuring data protection and maintaining operational continuity throughout backup and recovery operations.

**Endpoint:** `POST /api/admin/backup/create`

**Required Permissions:** Federation Administrator

**Request Body:**
```json
{
  "backupType": "full",
  "scope": {
    "databases": ["primary", "analytics", "audit"],
    "configurations": ["application", "league_rules", "user_permissions"],
    "files": ["uploads", "logs", "static_assets"]
  },
  "options": {
    "compression": true,
    "encryption": true,
    "verification": true,
    "remoteStorage": true
  },
  "retention": {
    "localCopies": 7,
    "remoteCopies": 30,
    "archiveAfterDays": 90
  },
  "scheduling": {
    "immediate": true,
    "recurringSchedule": null
  }
}
```

**Response:**
```json
{
  "success": true,
  "backup": {
    "id": "backup-uuid",
    "type": "full",
    "status": "in_progress",
    "startedAt": "2024-01-15T13:00:00Z",
    "estimatedCompletion": "2024-01-15T14:30:00Z",
    "progress": {
      "percentage": 15,
      "currentPhase": "database_snapshot",
      "itemsProcessed": 1500000,
      "totalItems": 10000000
    }
  },
  "storage": {
    "localPath": "/backups/2024-01-15/backup-uuid",
    "remoteLocation": "s3://backups/full/backup-uuid",
    "encryptionKey": "key-reference-uuid",
    "estimatedSize": "25 GB"
  },
  "validation": {
    "checksumGeneration": "enabled",
    "integrityVerification": "enabled",
    "restoreTestScheduled": "2024-01-16T02:00:00Z"
  }
}
```

This comprehensive admin API documentation provides complete coverage of all administrative functions while maintaining security, audit trails, and operational efficiency throughout hockey league management and system administration activities.