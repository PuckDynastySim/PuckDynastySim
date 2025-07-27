# Team Management API - Hockey League Simulation System

This comprehensive API documentation defines all team management endpoints that enable general managers to operate their hockey organizations effectively. The API provides complete roster management, strategic configuration, financial oversight, and player development capabilities while maintaining hockey domain accuracy and league rule compliance.

## Team Information and Configuration

### Get Team Details

Retrieves comprehensive team information including roster composition, financial status, strategic configurations, and organizational settings with appropriate access control and data filtering based on user permissions.

**Endpoint:** `GET /api/teams/{teamId}`

**Parameters:**
- `teamId` (path, required): Unique team identifier
- `include` (query, optional): Comma-separated list of additional data to include
  - `roster` - Include current roster with player details
  - `finances` - Include salary cap and financial information
  - `stats` - Include team performance statistics
  - `strategy` - Include tactical configurations

**Request Example:**
```http
GET /api/teams/team-uuid-123?include=roster,finances,stats
Authorization: Bearer jwt-token-here
```

**Response Example:**
```json
{
  "team": {
    "id": "team-uuid-123",
    "name": "Toronto Maple Leafs",
    "city": "Toronto",
    "abbreviation": "TOR",
    "leagueId": "league-uuid-456",
    "divisionId": "division-uuid-789",
    "primaryColor": "#003E7E",
    "secondaryColor": "#FFFFFF",
    "foundedYear": 1917,
    "arena": {
      "name": "Scotiabank Arena",
      "capacity": 18819,
      "location": "Toronto, ON"
    },
    "aiControlled": false,
    "roster": {
      "forwards": [
        {
          "id": "player-uuid-001",
          "firstName": "Auston",
          "lastName": "Matthews",
          "position": "center",
          "jerseyNumber": 34,
          "overallRating": 94,
          "contract": {
            "salary": 11640250,
            "length": 4,
            "expiryYear": 2027
          }
        }
      ],
      "defensemen": [...],
      "goaltenders": [...]
    },
    "finances": {
      "salaryCap": {
        "limit": 85000000,
        "used": 78450000,
        "available": 6550000,
        "utilizationPercentage": 92.3,
        "isCompliant": true
      },
      "budget": {
        "totalBudget": 150000000,
        "salaryBudget": 85000000,
        "facilitiesBudget": 25000000,
        "trainingBudget": 15000000,
        "marketingBudget": 25000000
      }
    },
    "stats": {
      "currentSeason": {
        "wins": 45,
        "losses": 25,
        "overtimeLosses": 12,
        "points": 102,
        "goalsFor": 265,
        "goalsAgainst": 198,
        "divisionRank": 2,
        "conferenceRank": 4,
        "leagueRank": 8
      }
    }
  }
}
```

**Error Responses:**
- `404 Not Found`: Team not found or user lacks access
- `403 Forbidden`: Insufficient permissions to view team details
- `400 Bad Request`: Invalid parameters or team ID format

### Update Team Configuration

Updates team settings including strategic preferences, facility configurations, and operational parameters with appropriate validation and authorization checking for team management permissions.

**Endpoint:** `PUT /api/teams/{teamId}/configuration`

**Request Body:**
```json
{
  "strategicPreferences": {
    "offensiveStyle": "aggressive",
    "defensiveStyle": "balanced",
    "specialTeamsEmphasis": "power_play",
    "fightingTolerance": "moderate"
  },
  "facilities": {
    "trainingFacilityLevel": 8,
    "medicalFacilityLevel": 7,
    "scoutingDepartmentLevel": 9
  },
  "ticketPricing": {
    "lowerBowl": 150,
    "upperBowl": 85,
    "club": 300,
    "suites": 2500,
    "seasonTicketDiscount": 15
  },
  "aiAssistance": {
    "lineOptimization": true,
    "injuryManagement": false,
    "contractNegotiation": false
  }
}
```

**Response Example:**
```json
{
  "success": true,
  "message": "Team configuration updated successfully",
  "updatedConfiguration": {
    "strategicPreferences": { ... },
    "facilities": { ... },
    "ticketPricing": { ... },
    "aiAssistance": { ... }
  },
  "timestamp": "2024-01-15T14:30:00Z"
}
```

## Roster Management Operations

### Get Team Roster

Retrieves complete team roster with player details, contract information, and strategic deployments including line assignments and depth chart positioning with filtering and sorting capabilities.

**Endpoint:** `GET /api/teams/{teamId}/roster`

**Parameters:**
- `position` (query, optional): Filter by player position
- `status` (query, optional): Filter by player status (`active`, `injured`, `suspended`)
- `sortBy` (query, optional): Sort field (`name`, `position`, `rating`, `salary`)
- `sortOrder` (query, optional): Sort direction (`asc`, `desc`)

**Response Example:**
```json
{
  "roster": {
    "summary": {
      "totalPlayers": 23,
      "forwards": 12,
      "defensemen": 8,
      "goaltenders": 3,
      "healthy": 21,
      "injured": 2,
      "suspended": 0
    },
    "players": [
      {
        "id": "player-uuid-001",
        "firstName": "Connor",
        "lastName": "McDavid",
        "position": "center",
        "jerseyNumber": 97,
        "age": 27,
        "overallRating": 97,
        "status": "active",
        "contract": {
          "salary": 12500000,
          "length": 8,
          "yearsRemaining": 5,
          "noTradeClause": true,
          "noMovementClause": false
        },
        "lineAssignment": {
          "line": 1,
          "position": "center",
          "powerPlay": true,
          "penaltyKill": false
        },
        "statistics": {
          "currentSeason": {
            "gamesPlayed": 67,
            "goals": 52,
            "assists": 78,
            "points": 130,
            "plusMinus": 25
          }
        },
        "injury": null
      }
    ],
    "depthChart": {
      "forwards": {
        "line1": {
          "leftWing": "player-uuid-002",
          "center": "player-uuid-001",
          "rightWing": "player-uuid-003"
        },
        "line2": { ... },
        "line3": { ... },
        "line4": { ... }
      },
      "defensemen": {
        "pair1": {
          "leftDefense": "player-uuid-010",
          "rightDefense": "player-uuid-011"
        },
        "pair2": { ... },
        "pair3": { ... }
      },
      "goaltenders": {
        "starter": "player-uuid-020",
        "backup": "player-uuid-021",
        "third": "player-uuid-022"
      }
    }
  }
}
```

### Sign Free Agent Player

Signs an available free agent player with contract negotiation and salary cap validation while ensuring roster size compliance and position requirements.

**Endpoint:** `POST /api/teams/{teamId}/players/sign`

**Request Body:**
```json
{
  "playerId": "player-uuid-789",
  "contract": {
    "salary": 5500000,
    "length": 3,
    "signingBonus": 2000000,
    "performanceBonuses": [
      {
        "type": "goals",
        "threshold": 30,
        "amount": 500000
      },
      {
        "type": "team_playoffs",
        "amount": 1000000
      }
    ],
    "noTradeClause": false,
    "noMovementClause": false
  },
  "lineAssignment": {
    "line": 2,
    "position": "left_wing",
    "powerPlay": false,
    "penaltyKill": true
  }
}
```

**Response Example:**
```json
{
  "success": true,
  "transaction": {
    "id": "transaction-uuid-456",
    "type": "player_signing",
    "timestamp": "2024-01-15T16:45:00Z",
    "player": {
      "id": "player-uuid-789",
      "name": "Jake Virtanen",
      "position": "left_wing"
    },
    "contract": {
      "salary": 5500000,
      "length": 3,
      "capHit": 5500000,
      "totalValue": 16500000
    },
    "salaryCap": {
      "previousUsed": 72500000,
      "newUsed": 78000000,
      "remaining": 7000000,
      "isCompliant": true
    }
  },
  "message": "Player signed successfully"
}
```

**Error Responses:**
- `422 Unprocessable Entity`: Salary cap violation or roster size exceeded
- `400 Bad Request`: Player not eligible for signing or invalid contract terms
- `409 Conflict`: Player already under contract with another team

### Release Player

Releases a player from the team with appropriate buyout calculations, salary cap implications, and waiver procedures if applicable.

**Endpoint:** `POST /api/teams/{teamId}/players/{playerId}/release`

**Request Body:**
```json
{
  "releaseType": "unconditional_release",
  "buyoutTerms": {
    "buyoutPortion": 0.67,
    "spreadOverYears": 6
  },
  "reason": "Performance and salary cap management"
}
```

**Response Example:**
```json
{
  "success": true,
  "transaction": {
    "id": "transaction-uuid-789",
    "type": "player_release",
    "timestamp": "2024-01-15T18:20:00Z",
    "player": {
      "id": "player-uuid-456",
      "name": "Expensive Veteran",
      "position": "right_wing"
    },
    "financialImpact": {
      "immediateSavings": 3500000,
      "buyoutCost": 4200000,
      "capRecapture": 700000,
      "deadMoney": [
        { "year": 2024, "amount": 700000 },
        { "year": 2025, "amount": 700000 },
        { "year": 2026, "amount": 700000 }
      ]
    },
    "waiverStatus": "exempt"
  }
}
```

### Propose Player Trade

Initiates a trade proposal involving players, draft picks, or cash considerations with comprehensive validation and notification to involved parties.

**Endpoint:** `POST /api/teams/{teamId}/trades/propose`

**Request Body:**
```json
{
  "targetTeamId": "team-uuid-987",
  "tradeItems": {
    "outgoing": {
      "players": [
        {
          "id": "player-uuid-123",
          "salaryRetention": 25
        }
      ],
      "draftPicks": [
        {
          "year": 2025,
          "round": 2,
          "conditions": "If player plays 60+ games"
        }
      ],
      "cash": 1000000
    },
    "incoming": {
      "players": [
        {
          "id": "player-uuid-456"
        }
      ],
      "draftPicks": [
        {
          "year": 2024,
          "round": 1
        }
      ]
    }
  },
  "conditions": [
    "Trade void if either player fails physical",
    "Additional 3rd round pick if team makes playoffs"
  ],
  "expirationDate": "2024-01-20T23:59:59Z",
  "message": "Looking to add depth at center position"
}
```

**Response Example:**
```json
{
  "success": true,
  "tradeProposal": {
    "id": "trade-proposal-uuid-321",
    "status": "pending",
    "proposingTeam": "team-uuid-123",
    "targetTeam": "team-uuid-987",
    "items": { ... },
    "validation": {
      "salaryCap": {
        "proposingTeam": {
          "currentUsed": 82000000,
          "afterTrade": 79500000,
          "compliant": true
        },
        "targetTeam": {
          "currentUsed": 75000000,
          "afterTrade": 77500000,
          "compliant": true
        }
      },
      "rosterRequirements": {
        "proposingTeam": "compliant",
        "targetTeam": "compliant"
      }
    },
    "expirationDate": "2024-01-20T23:59:59Z",
    "createdAt": "2024-01-15T20:15:00Z"
  }
}
```

## Line Configuration and Strategy Management

### Get Line Configurations

Retrieves current line combinations, special team units, and strategic deployments with effectiveness statistics and usage patterns.

**Endpoint:** `GET /api/teams/{teamId}/lines`

**Response Example:**
```json
{
  "lineConfigurations": {
    "forwards": {
      "line1": {
        "leftWing": {
          "playerId": "player-uuid-001",
          "name": "Mitch Marner",
          "chemistry": 85
        },
        "center": {
          "playerId": "player-uuid-002",
          "name": "Auston Matthews",
          "chemistry": 92
        },
        "rightWing": {
          "playerId": "player-uuid-003",
          "name": "William Nylander",
          "chemistry": 88
        },
        "overallChemistry": 88,
        "iceMimePercentage": 22.5,
        "effectiveness": {
          "goalsFor": 45,
          "goalsAgainst": 12,
          "shotDifferential": "+125",
          "faceoffWinPercentage": 58.2
        }
      },
      "line2": { ... },
      "line3": { ... },
      "line4": { ... }
    },
    "defensemen": {
      "pair1": {
        "leftDefense": {
          "playerId": "player-uuid-010",
          "name": "Morgan Rielly",
          "chemistry": 78
        },
        "rightDefense": {
          "playerId": "player-uuid-011",
          "name": "T.J. Brodie",
          "chemistry": 82
        },
        "overallChemistry": 80,
        "iceTimePercentage": 28.3,
        "effectiveness": {
          "goalsFor": 15,
          "goalsAgainst": 18,
          "shotDifferential": "+45",
          "penaltyKillTime": "125:30"
        }
      },
      "pair2": { ... },
      "pair3": { ... }
    },
    "specialTeams": {
      "powerPlay": {
        "unit1": {
          "forwards": ["player-uuid-001", "player-uuid-002", "player-uuid-003"],
          "defensemen": ["player-uuid-010", "player-uuid-011"],
          "effectiveness": {
            "powerPlayPercentage": 24.5,
            "goalsFor": 45,
            "opportunities": 184
          }
        },
        "unit2": { ... }
      },
      "penaltyKill": {
        "unit1": {
          "forwards": ["player-uuid-015", "player-uuid-016"],
          "defensemen": ["player-uuid-012", "player-uuid-013"],
          "effectiveness": {
            "penaltyKillPercentage": 82.1,
            "goalsAgainst": 35,
            "timesShorthanded": 196
          }
        },
        "unit2": { ... }
      }
    }
  }
}
```

### Update Line Configurations

Updates line combinations and strategic deployments with chemistry calculations and effectiveness validation.

**Endpoint:** `PUT /api/teams/{teamId}/lines`

**Request Body:**
```json
{
  "forwards": {
    "line1": {
      "leftWing": "player-uuid-001",
      "center": "player-uuid-002",
      "rightWing": "player-uuid-003"
    },
    "line2": {
      "leftWing": "player-uuid-004",
      "center": "player-uuid-005",
      "rightWing": "player-uuid-006"
    }
  },
  "defensemen": {
    "pair1": {
      "leftDefense": "player-uuid-010",
      "rightDefense": "player-uuid-011"
    }
  },
  "goaltenders": {
    "starter": "player-uuid-020",
    "backup": "player-uuid-021"
  },
  "specialTeams": {
    "powerPlay": {
      "unit1": {
        "forwards": ["player-uuid-001", "player-uuid-002", "player-uuid-003"],
        "defensemen": ["player-uuid-010", "player-uuid-011"]
      }
    },
    "penaltyKill": {
      "unit1": {
        "forwards": ["player-uuid-015", "player-uuid-016"],
        "defensemen": ["player-uuid-012", "player-uuid-013"]
      }
    }
  }
}
```

**Response Example:**
```json
{
  "success": true,
  "lineConfigurations": {
    "updated": true,
    "chemistryCalculations": {
      "line1": {
        "previousChemistry": 85,
        "newChemistry": 88,
        "improvement": "+3"
      }
    },
    "validation": {
      "rosterCompliance": true,
      "positionRequirements": true,
      "warnings": [
        "Defensive pair 3 has low chemistry (62)"
      ]
    }
  },
  "timestamp": "2024-01-15T21:30:00Z"
}
```

## Financial Management and Salary Cap

### Get Salary Cap Status

Retrieves comprehensive salary cap information including current usage, projections, and compliance status with detailed breakdowns and analysis.

**Endpoint:** `GET /api/teams/{teamId}/salary-cap`

**Response Example:**
```json
{
  "salaryCap": {
    "limit": 85000000,
    "currentUsage": 78450000,
    "available": 6550000,
    "utilizationPercentage": 92.3,
    "isCompliant": true,
    "breakdown": {
      "activeSalaries": 76450000,
      "bonusOverhang": 0,
      "ltirRelief": 0,
      "retainedSalary": 2000000,
      "deadMoney": 0
    },
    "projections": {
      "nextSeason": {
        "committedSalary": 65000000,
        "contractsExpiring": 13450000,
        "restrictedFreeAgents": 3,
        "unrestrictedFreeAgents": 8
      }
    },
    "contracts": [
      {
        "playerId": "player-uuid-001",
        "playerName": "Connor McDavid",
        "salary": 12500000,
        "capHit": 12500000,
        "yearsRemaining": 5,
        "isLtir": false,
        "retainedPercentage": 0
      }
    ],
    "ltirPool": {
      "available": 0,
      "players": []
    },
    "emergencyRecalls": {
      "active": 0,
      "capReliefUsed": 0
    }
  }
}
```

### Salary Cap Simulation

Simulates potential salary cap scenarios including player signings, trades, and contract modifications to analyze financial impact and compliance.

**Endpoint:** `POST /api/teams/{teamId}/salary-cap/simulate`

**Request Body:**
```json
{
  "scenarios": [
    {
      "name": "Sign Free Agent Center",
      "transactions": [
        {
          "type": "sign_player",
          "playerId": "player-uuid-new",
          "salary": 6000000,
          "length": 4
        }
      ]
    },
    {
      "name": "Trade for Veteran Defenseman",
      "transactions": [
        {
          "type": "trade_incoming",
          "playerId": "player-uuid-trade",
          "salary": 7500000,
          "retainedPercentage": 20
        },
        {
          "type": "trade_outgoing",
          "playerId": "player-uuid-current",
          "salary": 3500000
        }
      ]
    }
  ]
}
```

**Response Example:**
```json
{
  "simulations": [
    {
      "scenario": "Sign Free Agent Center",
      "result": {
        "feasible": false,
        "capUsageAfter": 84450000,
        "capOverage": 450000,
        "recommendations": [
          "Release player with salary > $1M",
          "Negotiate lower salary ($5.5M max)",
          "Consider short-term contract with performance bonuses"
        ]
      }
    },
    {
      "scenario": "Trade for Veteran Defenseman",
      "result": {
        "feasible": true,
        "capUsageAfter": 80950000,
        "capSavings": 2500000,
        "rosterImpact": {
          "defenseRatingImprovement": "+5",
          "overallTeamRatingChange": "+2"
        }
      }
    }
  ]
}
```

## Player Development and Training

### Get Training Programs

Retrieves available training programs and player development initiatives with effectiveness tracking and resource requirements.

**Endpoint:** `GET /api/teams/{teamId}/training`

**Response Example:**
```json
{
  "trainingPrograms": {
    "individual": [
      {
        "id": "shooting-accuracy-program",
        "name": "Shooting Accuracy Development",
        "targetSkills": ["shooting"],
        "duration": "3 months",
        "cost": 150000,
        "maxParticipants": 8,
        "currentEnrollment": 5,
        "effectiveness": {
          "averageImprovement": 3.2,
          "successRate": 78
        }
      }
    ],
    "team": [
      {
        "id": "power-play-systems",
        "name": "Power Play Systems Training",
        "targetAreas": ["power_play_effectiveness"],
        "duration": "6 weeks",
        "cost": 500000,
        "teamwideImpact": true,
        "effectiveness": {
          "powerPlayImprovementPercentage": 4.8
        }
      }
    ],
    "facilities": {
      "currentLevel": 7,
      "upgradeOptions": [
        {
          "level": 8,
          "cost": 5000000,
          "benefits": {
            "trainingEffectiveness": "+15%",
            "injuryReduction": "+8%",
            "playerSatisfaction": "+12%"
          }
        }
      ]
    }
  }
}
```

### Enroll Player in Training

Enrolls players in development programs with cost calculation and schedule coordination.

**Endpoint:** `POST /api/teams/{teamId}/training/enroll`

**Request Body:**
```json
{
  "programId": "shooting-accuracy-program",
  "players": [
    {
      "playerId": "player-uuid-123",
      "focusAreas": ["wrist_shot", "one_timer"],
      "intensity": "high"
    },
    {
      "playerId": "player-uuid-456", 
      "focusAreas": ["backhand", "tip_ins"],
      "intensity": "medium"
    }
  ],
  "startDate": "2024-02-01",
  "budget": 300000
}
```

**Response Example:**
```json
{
  "success": true,
  "enrollment": {
    "programId": "shooting-accuracy-program",
    "playersEnrolled": 2,
    "totalCost": 300000,
    "scheduleConflicts": [],
    "projectedOutcomes": [
      {
        "playerId": "player-uuid-123",
        "currentShootingRating": 82,
        "projectedImprovement": "+3 to +5",
        "completionDate": "2024-05-01"
      }
    ],
    "budgetImpact": {
      "trainingBudgetUsed": 1200000,
      "trainingBudgetRemaining": 3800000
    }
  }
}
```

## Team Statistics and Analytics

### Get Team Performance Analytics

Retrieves comprehensive team performance data including advanced statistics, trend analysis, and comparative metrics.

**Endpoint:** `GET /api/teams/{teamId}/analytics`

**Parameters:**
- `season` (query, optional): Specific season year
- `gameType` (query, optional): Regular season or playoffs
- `dateRange` (query, optional): Custom date range for analysis

**Response Example:**
```json
{
  "analytics": {
    "summary": {
      "record": "45-25-12",
      "points": 102,
      "pointsPercentage": 62.2,
      "divisionRank": 2,
      "conferenceRank": 4,
      "leagueRank": 8
    },
    "offense": {
      "goalsFor": 265,
      "goalsPerGame": 3.23,
      "powerPlayGoals": 45,
      "powerPlayPercentage": 24.5,
      "shotsFor": 2580,
      "shootingPercentage": 10.3,
      "faceoffWinPercentage": 52.1
    },
    "defense": {
      "goalsAgainst": 198,
      "goalsAgainstPerGame": 2.41,
      "penaltyKillPercentage": 82.1,
      "shotsAgainst": 2342,
      "savePercentage": 91.5,
      "penaltyMinutes": 856
    },
    "advancedStats": {
      "corsiFor": 54.2,
      "fenwickFor": 53.8,
      "expectedGoalsFor": 242.8,
      "expectedGoalsAgainst": 201.3,
      "pdo": 101.8,
      "zoneStartPercentage": 51.2
    },
    "trends": {
      "last10Games": {
        "record": "7-2-1",
        "goalsFor": 35,
        "goalsAgainst": 22,
        "trend": "improving"
      },
      "homeVsAway": {
        "home": { "record": "25-10-6", "pointsPercentage": 68.3 },
        "away": { "record": "20-15-6", "pointsPercentage": 56.1 }
      }
    },
    "playerContributions": {
      "topScorers": [
        {
          "playerId": "player-uuid-001",
          "name": "Connor McDavid",
          "points": 130,
          "contributionPercentage": 49.1
        }
      ],
      "goalieStats": [
        {
          "playerId": "player-uuid-020",
          "name": "Stuart Skinner", 
          "wins": 32,
          "savePercentage": 91.8,
          "goalsAgainstAverage": 2.15
        }
      ]
    }
  }
}
```

## Error Handling and Status Codes

### Common Error Responses

**Validation Errors (400 Bad Request):**
```json
{
  "error": "Validation failed",
  "details": [
    {
      "field": "contract.salary",
      "message": "Salary must be between league minimum and maximum",
      "value": 500000,
      "constraints": {
        "minimum": 700000,
        "maximum": 15000000
      }
    }
  ],
  "suggestions": [
    "Adjust salary to meet league minimum of $700,000",
    "Consider entry-level contract for players under 25"
  ]
}
```

**Authorization Errors (403 Forbidden):**
```json
{
  "error": "Insufficient permissions",
  "message": "General manager role required for roster modifications",
  "requiredRole": "general_manager",
  "userRole": "player",
  "supportedActions": [
    "View roster (requires team association)",
    "View public statistics"
  ]
}
```

**Business Rule Violations (422 Unprocessable Entity):**
```json
{
  "error": "Salary cap violation",
  "message": "Transaction would exceed salary cap limit",
  "details": {
    "currentUsage": 78450000,
    "transactionAmount": 8000000,
    "capLimit": 85000000,
    "overage": 1450000
  },
  "recommendations": [
    "Release players worth $1.5M+ in salary",
    "Negotiate salary retention in trade",
    "Consider players on LTIR for cap relief"
  ]
}
```

**Hockey Domain Errors (422 Unprocessable Entity):**
```json
{
  "error": "Roster composition invalid",
  "message": "Team requires minimum 20 players with valid position distribution",
  "details": {
    "currentRoster": 19,
    "minimumRequired": 20,
    "positionDeficit": {
      "goaltenders": "Need at least 2 goaltenders"
    }
  },
  "solutions": [
    "Sign additional goaltender from free agency",
    "Call up goaltender from farm team",
    "Claim goaltender from waivers"
  ]
}
```

This comprehensive team management API documentation provides complete functionality for hockey team operations while maintaining domain accuracy and regulatory compliance throughout all management activities.