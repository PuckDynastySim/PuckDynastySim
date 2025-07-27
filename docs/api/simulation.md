# Simulation API Documentation - Hockey League Simulation System

This comprehensive API documentation defines the game simulation endpoints, real-time communication protocols, and statistical output formats that enable realistic hockey game generation and live simulation experiences within the Hockey League Simulation System.

## Game Simulation Endpoints

### Start Game Simulation

Initiates game simulation processing with configurable parameters for realism, speed, and output detail. The simulation engine generates realistic hockey gameplay using player ratings, team strategies, and situational factors while maintaining statistical accuracy and authentic game flow.

**Endpoint:** `POST /api/games/{gameId}/simulate`

**Authentication:** Required (League Commissioner or authorized user)

**Request Parameters:**
```typescript
interface GameSimulationRequest {
  /** Simulation processing mode */
  mode: 'real_time' | 'accelerated' | 'instant';
  /** Enable detailed play-by-play generation */
  generatePlayByPlay: boolean;
  /** Enable comprehensive box score compilation */
  generateBoxScore: boolean;
  /** Enable real-time WebSocket updates */
  realTimeUpdates: boolean;
  /** Statistical realism level */
  realismLevel: 'arcade' | 'realistic' | 'simulation';
  /** Custom simulation parameters */
  parameters?: {
    injuryFrequency: number;        // 0.0 to 1.0 scale
    penaltyFrequency: number;       // 0.0 to 1.0 scale
    overtimeProbability: number;    // 0.0 to 1.0 scale
    fightingEnabled: boolean;
    weatherEffects: boolean;
  };
}
```

**Example Request:**
```bash
curl -X POST "https://api.hockeysim.com/api/games/game-12345/simulate" \
  -H "Authorization: Bearer jwt-token" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "real_time",
    "generatePlayByPlay": true,
    "generateBoxScore": true,
    "realTimeUpdates": true,
    "realismLevel": "simulation",
    "parameters": {
      "injuryFrequency": 0.02,
      "penaltyFrequency": 0.15,
      "overtimeProbability": 0.23,
      "fightingEnabled": true,
      "weatherEffects": false
    }
  }'
```

**Response Format:**
```typescript
interface GameSimulationResponse {
  success: boolean;
  simulation: {
    id: string;
    gameId: string;
    status: 'queued' | 'in_progress' | 'completed' | 'failed';
    mode: string;
    startedAt: string;
    estimatedCompletion?: string;
    progress?: {
      period: number;
      timeRemaining: string;
      homeScore: number;
      awayScore: number;
      eventsGenerated: number;
    };
    websocketChannel?: string;
  };
  game: {
    id: string;
    homeTeam: TeamSummary;
    awayTeam: TeamSummary;
    scheduledDate: string;
    venue: string;
  };
}
```

**Success Response (202 Accepted):**
```json
{
  "success": true,
  "simulation": {
    "id": "sim-uuid-12345",
    "gameId": "game-12345",
    "status": "in_progress",
    "mode": "real_time",
    "startedAt": "2024-01-15T19:00:00Z",
    "estimatedCompletion": "2024-01-15T22:30:00Z",
    "progress": {
      "period": 1,
      "timeRemaining": "15:23",
      "homeScore": 1,
      "awayScore": 0,
      "eventsGenerated": 45
    },
    "websocketChannel": "game-simulation-12345"
  },
  "game": {
    "id": "game-12345",
    "homeTeam": {
      "id": "team-456",
      "name": "Toronto Maple Leafs",
      "abbreviation": "TOR"
    },
    "awayTeam": {
      "id": "team-789",
      "name": "Montreal Canadiens", 
      "abbreviation": "MTL"
    },
    "scheduledDate": "2024-01-15T19:00:00Z",
    "venue": "Scotiabank Arena"
  }
}
```

**Error Responses:**

**400 Bad Request - Invalid Parameters:**
```json
{
  "error": "Invalid simulation parameters",
  "details": {
    "field": "realismLevel",
    "message": "Must be one of: arcade, realistic, simulation",
    "provided": "invalid_level"
  },
  "suggestions": [
    "Use 'realistic' for balanced gameplay",
    "Use 'simulation' for maximum realism",
    "Use 'arcade' for faster, simplified games"
  ]
}
```

**403 Forbidden - Insufficient Permissions:**
```json
{
  "error": "Insufficient permissions",
  "details": {
    "required": "league_commissioner or game_control",
    "current": "general_manager",
    "resource": "game-12345"
  }
}
```

**409 Conflict - Game Already in Progress:**
```json
{
  "error": "Game simulation already in progress",
  "details": {
    "currentSimulation": "sim-uuid-67890",
    "status": "in_progress",
    "progress": {
      "period": 2,
      "timeRemaining": "8:45"
    }
  },
  "actions": [
    "Wait for current simulation to complete",
    "Cancel current simulation if authorized",
    "Monitor progress via WebSocket channel"
  ]
}
```

### Simulation Control Operations

Game simulation control enables authorized users to pause, resume, accelerate, or terminate active simulations while maintaining game state consistency and providing appropriate user feedback throughout control operations.

**Pause Simulation**
`POST /api/games/{gameId}/simulate/pause`

**Resume Simulation** 
`POST /api/games/{gameId}/simulate/resume`

**Change Speed**
`POST /api/games/{gameId}/simulate/speed`
```json
{
  "speed": "real_time" | "accelerated" | "instant"
}
```

**Terminate Simulation**
`POST /api/games/{gameId}/simulate/terminate`
```json
{
  "reason": "user_request" | "system_error" | "league_decision",
  "preserveProgress": boolean
}
```

**Control Response Format:**
```typescript
interface SimulationControlResponse {
  success: boolean;
  action: string;
  simulation: {
    id: string;
    status: string;
    previousState?: string;
    updatedAt: string;
  };
  gameState: {
    period: number;
    timeRemaining: string;
    homeScore: number;
    awayScore: number;
    canResume: boolean;
  };
}
```

### Simulation Status and Progress

Retrieve current simulation status, progress information, and performance metrics for monitoring and troubleshooting simulation operations.

**Endpoint:** `GET /api/games/{gameId}/simulate/status`

**Authentication:** Required

**Response Format:**
```typescript
interface SimulationStatusResponse {
  simulation: {
    id: string;
    gameId: string;
    status: 'queued' | 'in_progress' | 'paused' | 'completed' | 'failed' | 'terminated';
    mode: string;
    startedAt: string;
    completedAt?: string;
    duration?: number; // seconds
    progress: {
      period: number;
      timeRemaining: string;
      homeScore: number;
      awayScore: number;
      overtime: boolean;
      shootout: boolean;
      eventsGenerated: number;
      lastEventAt: string;
    };
    performance: {
      eventsPerSecond: number;
      averageEventProcessingTime: number;
      memoryUsage: number;
      cpuUsage: number;
    };
    errors?: SimulationError[];
  };
  websocketChannel?: string;
}
```

**Example Response:**
```json
{
  "simulation": {
    "id": "sim-uuid-12345",
    "gameId": "game-12345", 
    "status": "in_progress",
    "mode": "real_time",
    "startedAt": "2024-01-15T19:00:00Z",
    "duration": 3600,
    "progress": {
      "period": 2,
      "timeRemaining": "12:34",
      "homeScore": 2,
      "awayScore": 1,
      "overtime": false,
      "shootout": false,
      "eventsGenerated": 156,
      "lastEventAt": "2024-01-15T20:47:26Z"
    },
    "performance": {
      "eventsPerSecond": 0.43,
      "averageEventProcessingTime": 45,
      "memoryUsage": 128.5,
      "cpuUsage": 15.2
    }
  },
  "websocketChannel": "game-simulation-12345"
}
```

## Real-Time WebSocket Communication

### WebSocket Connection and Authentication

Real-time game simulation updates utilize WebSocket connections with JWT authentication and subscription-based message filtering to provide live game events, statistical updates, and simulation control notifications.

**Connection URL:** `wss://api.hockeysim.com/ws`

**Authentication:**
```javascript
const socket = io('wss://api.hockeysim.com/ws', {
  auth: {
    token: 'jwt-access-token'
  },
  transports: ['websocket']
});

socket.on('connect', () => {
  console.log('Connected to simulation WebSocket');
  
  // Subscribe to specific game updates
  socket.emit('subscribe_game', { 
    gameId: 'game-12345',
    events: ['goals', 'penalties', 'period_end', 'game_end']
  });
});
```

### Game Event Broadcasting

Live game events broadcast through WebSocket connections provide real-time updates for subscribed clients with comprehensive event details and game state information.

**Event Types and Formats:**

**Goal Event:**
```typescript
interface GoalEvent {
  type: 'goal';
  gameId: string;
  timestamp: string;
  period: number;
  timeRemaining: string;
  team: {
    id: string;
    name: string;
    abbreviation: string;
  };
  scorer: {
    id: string;
    name: string;
    jerseyNumber: number;
    seasonGoals: number;
  };
  assists: Array<{
    id: string;
    name: string;
    jerseyNumber: number;
    seasonAssists: number;
  }>;
  goalType: 'even_strength' | 'power_play' | 'short_handed' | 'penalty_shot' | 'empty_net';
  shotType: 'wrist' | 'slap' | 'snap' | 'backhand' | 'tip' | 'deflection';
  location: {
    x: number; // -100 to 100 (defensive zone to offensive zone)
    y: number; // -42.5 to 42.5 (left boards to right boards)
  };
  description: string;
  gameState: {
    homeScore: number;
    awayScore: number;
    period: number;
    timeRemaining: string;
  };
}
```

**Penalty Event:**
```typescript
interface PenaltyEvent {
  type: 'penalty';
  gameId: string;
  timestamp: string;
  period: number;
  timeRemaining: string;
  player: {
    id: string;
    name: string;
    jerseyNumber: number;
    team: TeamSummary;
  };
  infraction: {
    type: 'minor' | 'major' | 'misconduct' | 'game_misconduct';
    name: string; // 'Tripping', 'High Sticking', 'Fighting', etc.
    duration: number; // minutes
    description: string;
  };
  victim?: {
    id: string;
    name: string;
    jerseyNumber: number;
  };
  powerPlayInfo: {
    teamOnPowerPlay: string;
    playersAdvantage: number; // +1, +2, etc.
    duration: number;
  };
}
```

**Period End Event:**
```typescript
interface PeriodEndEvent {
  type: 'period_end';
  gameId: string;
  timestamp: string;
  period: number;
  periodStats: {
    home: {
      goals: number;
      shots: number;
      faceoffWins: number;
      hits: number;
      blockedShots: number;
      penalties: number;
      penaltyMinutes: number;
    };
    away: {
      goals: number;
      shots: number;
      faceoffWins: number;
      hits: number;
      blockedShots: number;
      penalties: number;
      penaltyMinutes: number;
    };
  };
  gameState: {
    homeScore: number;
    awayScore: number;
    period: number;
    intermissionLength: number; // seconds
    nextPeriodStartsAt: string;
  };
}
```

**Game Completion Event:**
```typescript
interface GameEndEvent {
  type: 'game_end';
  gameId: string;
  timestamp: string;
  finalScore: {
    home: number;
    away: number;
  };
  gameLength: {
    regulation: boolean;
    overtime: boolean;
    shootout: boolean;
  };
  winningTeam: {
    id: string;
    name: string;
    abbreviation: string;
  };
  threeStars: Array<{
    star: 1 | 2 | 3;
    player: PlayerSummary;
    performance: string;
  }>;
  nextActions: {
    boxScoreAvailable: boolean;
    playByPlayAvailable: boolean;
    statisticsProcessed: boolean;
    standingsUpdated: boolean;
  };
}
```

### WebSocket Subscription Management

**Subscribe to Game Updates:**
```javascript
socket.emit('subscribe_game', {
  gameId: 'game-12345',
  events: ['all'] | ['goals', 'penalties', 'period_end'],
  detailLevel: 'full' | 'summary'
});
```

**Subscribe to League-Wide Events:**
```javascript
socket.emit('subscribe_league', {
  leagueId: 'league-789',
  events: ['game_starts', 'game_ends', 'trade_alerts']
});
```

**Unsubscribe from Updates:**
```javascript
socket.emit('unsubscribe_game', { gameId: 'game-12345' });
socket.emit('unsubscribe_league', { leagueId: 'league-789' });
```

**Connection Management Events:**
```javascript
socket.on('subscription_confirmed', (data) => {
  console.log('Subscribed to:', data.subscriptions);
});

socket.on('subscription_error', (error) => {
  console.error('Subscription failed:', error);
});

socket.on('connection_info', (info) => {
  console.log('Connection stats:', info);
});
```

## Statistical Output and Box Scores

### Comprehensive Box Score Generation

Game simulation generates detailed statistical output including individual player performance, team statistics, and advanced analytics that align with professional hockey statistical standards.

**Endpoint:** `GET /api/games/{gameId}/boxscore`

**Authentication:** Required

**Response Format:**
```typescript
interface GameBoxScore {
  game: {
    id: string;
    date: string;
    venue: string;
    attendance: number;
    gameLength: string; // "2:34:15" (hours:minutes:seconds)
    overtime: boolean;
    shootout: boolean;
  };
  teams: {
    home: TeamBoxScore;
    away: TeamBoxScore;
  };
  periods: Array<{
    period: number;
    home: {
      goals: number;
      shots: number;
    };
    away: {
      goals: number;
      shots: number;
    };
  }>;
  officials: {
    referees: string[];
    linesmen: string[];
  };
  threeStars: Array<{
    star: 1 | 2 | 3;
    player: PlayerSummary;
    performance: string;
  }>;
}

interface TeamBoxScore {
  team: TeamSummary;
  score: number;
  shots: number;
  statistics: {
    faceoffWins: number;
    faceoffAttempts: number;
    faceoffPercentage: number;
    powerPlays: number;
    powerPlayGoals: number;
    powerPlayPercentage: number;
    penaltyKillOpportunities: number;
    penaltyKillSuccesses: number;
    penaltyKillPercentage: number;
    shotsBlocked: number;
    hits: number;
    giveaways: number;
    takeaways: number;
    penaltyMinutes: number;
  };
  skaters: Array<PlayerBoxScore>;
  goalies: Array<GoalieBoxScore>;
}

interface PlayerBoxScore {
  player: PlayerSummary;
  position: string;
  timeOnIce: string; // "15:23"
  goals: number;
  assists: number;
  points: number;
  plusMinus: number;
  penaltyMinutes: number;
  shots: number;
  hits: number;
  faceoffWins: number;
  faceoffAttempts: number;
  blockedShots: number;
  giveaways: number;
  takeaways: number;
  powerPlayTOI: string;
  shortHandedTOI: string;
}

interface GoalieBoxScore {
  player: PlayerSummary;
  timeOnIce: string;
  decision: 'W' | 'L' | 'OTL' | 'T' | null;
  saves: number;
  shotsAgainst: number;
  goalsAgainst: number;
  savePercentage: number;
  shutout: boolean;
  evenStrengthSaves: number;
  powerPlaySaves: number;
  shortHandedSaves: number;
}
```

### Play-by-Play Generation

Detailed play-by-play narratives provide comprehensive game documentation with realistic descriptions and proper hockey terminology for authentic game recreation and analysis.

**Endpoint:** `GET /api/games/{gameId}/playbyplay`

**Query Parameters:**
- `period` (optional): Filter by specific period (1, 2, 3, OT, SO)
- `eventType` (optional): Filter by event type (goal, penalty, shot, etc.)
- `teamId` (optional): Filter by specific team events

**Response Format:**
```typescript
interface PlayByPlayResponse {
  game: GameSummary;
  events: Array<PlayByPlayEvent>;
  summary: {
    totalEvents: number;
    eventsByPeriod: Record<string, number>;
    eventsByType: Record<string, number>;
  };
}

interface PlayByPlayEvent {
  id: string;
  sequence: number;
  period: number;
  periodType: 'regulation' | 'overtime' | 'shootout';
  timeElapsed: string; // "4:37" (from start of period)
  timeRemaining: string; // "15:23" (remaining in period)
  eventType: string;
  team?: TeamSummary;
  players: Array<{
    player: PlayerSummary;
    role: 'primary' | 'assist1' | 'assist2' | 'victim' | 'witness';
  }>;
  description: string;
  location?: {
    zone: 'defensive' | 'neutral' | 'offensive';
    x: number;
    y: number;
  };
  situation: {
    homeSkaters: number;
    awaySkaters: number;
    strength: 'even' | 'power_play' | 'short_handed';
  };
  result?: {
    homeScore: number;
    awayScore: number;
    shotTotal?: number;
  };
}
```

**Example Play-by-Play Events:**
```json
[
  {
    "id": "event-001",
    "sequence": 1,
    "period": 1,
    "periodType": "regulation",
    "timeElapsed": "0:00",
    "timeRemaining": "20:00",
    "eventType": "faceoff",
    "description": "Opening faceoff at center ice. Connor McDavid wins the draw.",
    "players": [
      {
        "player": {
          "id": "player-123",
          "name": "Connor McDavid",
          "jerseyNumber": 97
        },
        "role": "primary"
      }
    ],
    "location": {
      "zone": "neutral",
      "x": 0,
      "y": 0
    },
    "situation": {
      "homeSkaters": 6,
      "awaySkaters": 6,
      "strength": "even"
    }
  },
  {
    "id": "event-045",
    "sequence": 45,
    "period": 1,
    "periodType": "regulation", 
    "timeElapsed": "8:23",
    "timeRemaining": "11:37",
    "eventType": "goal",
    "team": {
      "id": "team-456",
      "name": "Toronto Maple Leafs",
      "abbreviation": "TOR"
    },
    "description": "GOAL! Auston Matthews scores his 25th goal of the season! Wrist shot from the left circle beats the goaltender high glove side. Assists to William Nylander and Morgan Rielly.",
    "players": [
      {
        "player": {
          "id": "player-456",
          "name": "Auston Matthews",
          "jerseyNumber": 34
        },
        "role": "primary"
      },
      {
        "player": {
          "id": "player-789",
          "name": "William Nylander", 
          "jerseyNumber": 88
        },
        "role": "assist1"
      },
      {
        "player": {
          "id": "player-012",
          "name": "Morgan Rielly",
          "jerseyNumber": 44
        },
        "role": "assist2"
      }
    ],
    "location": {
      "zone": "offensive",
      "x": 75,
      "y": -25
    },
    "situation": {
      "homeSkaters": 6,
      "awaySkaters": 6,
      "strength": "even"
    },
    "result": {
      "homeScore": 1,
      "awayScore": 0
    }
  }
]
```

## Simulation Performance and Optimization

### Performance Monitoring

Game simulation performance metrics enable optimization and troubleshooting while ensuring efficient resource utilization and maintaining realistic simulation speeds for optimal user experience.

**Endpoint:** `GET /api/simulation/performance`

**Authentication:** Required (Administrator)

**Response Format:**
```typescript
interface SimulationPerformanceMetrics {
  overall: {
    activeSimulations: number;
    queuedSimulations: number;
    completedToday: number;
    averageSimulationTime: number; // seconds
    successRate: number; // 0.0 to 1.0
  };
  resources: {
    cpuUsage: number; // percentage
    memoryUsage: number; // MB
    networkBandwidth: number; // MB/s
    databaseConnections: number;
    redisConnections: number;
  };
  recentSimulations: Array<{
    gameId: string;
    duration: number;
    mode: string;
    eventsGenerated: number;
    performance: {
      eventsPerSecond: number;
      peakMemory: number;
      averageCpu: number;
    };
    status: 'completed' | 'failed' | 'terminated';
  }>;
  alerts: Array<{
    type: 'warning' | 'error' | 'info';
    message: string;
    timestamp: string;
    affectedSimulations?: string[];
  }>;
}
```

### Simulation Configuration

**Endpoint:** `GET /api/simulation/config`
**Endpoint:** `PUT /api/simulation/config` (Administrator only)

**Configuration Parameters:**
```typescript
interface SimulationConfiguration {
  engine: {
    maxConcurrentGames: number;
    defaultRealismLevel: 'arcade' | 'realistic' | 'simulation';
    enableAdvancedStatistics: boolean;
    enableInjuries: boolean;
    enableFighting: boolean;
  };
  performance: {
    maxEventsPerSecond: number;
    memoryLimitMB: number;
    timeoutSeconds: number;
    autoCleanupAfterHours: number;
  };
  statistics: {
    trackAdvancedMetrics: boolean;
    generateHeatMaps: boolean;
    calculateExpectedGoals: boolean;
    enableCorsiTracking: boolean;
  };
  realTime: {
    enableWebSocketBroadcast: boolean;
    maxWebSocketConnections: number;
    messageRateLimit: number; // per second
    compressionEnabled: boolean;
  };
}
```

### Batch Simulation Operations

**Endpoint:** `POST /api/simulation/batch`

Process multiple games simultaneously with efficient resource utilization and coordinated scheduling for league-wide simulation operations.

**Request Format:**
```typescript
interface BatchSimulationRequest {
  games: string[]; // Array of game IDs
  mode: 'sequential' | 'parallel';
  maxConcurrent?: number;
  parameters: {
    realismLevel: 'arcade' | 'realistic' | 'simulation';
    generatePlayByPlay: boolean;
    generateBoxScore: boolean;
    realTimeUpdates: boolean;
  };
  scheduling: {
    priority: 'low' | 'normal' | 'high';
    startTime?: string; // ISO timestamp
    completionDeadline?: string;
  };
}
```

**Response Format:**
```typescript
interface BatchSimulationResponse {
  batchId: string;
  status: 'queued' | 'in_progress' | 'completed' | 'failed';
  games: Array<{
    gameId: string;
    status: 'pending' | 'queued' | 'in_progress' | 'completed' | 'failed';
    simulationId?: string;
    startedAt?: string;
    completedAt?: string;
    error?: string;
  }>;
  progress: {
    completed: number;
    total: number;
    percentage: number;
    estimatedCompletion: string;
  };
  websocketChannel: string;
}
```

This comprehensive simulation API documentation provides complete integration guidance for realistic hockey game generation and live simulation experiences while maintaining performance efficiency and statistical accuracy throughout all simulation operations.