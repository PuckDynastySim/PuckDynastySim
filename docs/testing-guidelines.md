# Testing Guidelines - Hockey League Simulation System

This comprehensive testing guidelines document establishes systematic testing methodologies and quality assurance procedures for the Hockey League Simulation System. The guidelines emphasize hockey domain accuracy, AI-assisted development validation, and comprehensive coverage across all system components while ensuring reliable, maintainable, and performant testing practices.

## Testing Framework and Architecture

### Testing Strategy Overview

The testing strategy implements a multi-layered approach that encompasses unit testing for individual components, integration testing for system interactions, and end-to-end testing for complete user workflows. This comprehensive approach ensures thorough validation while maintaining development efficiency and providing reliable feedback throughout the development lifecycle.

Testing architecture follows the testing pyramid principle with extensive unit testing as the foundation, focused integration testing for critical system interactions, and targeted end-to-end testing for essential user journeys. This distribution ensures fast feedback cycles while providing comprehensive coverage of hockey domain functionality and system reliability.

Test isolation maintains independence between test cases while enabling realistic data scenarios that reflect actual hockey league operations. Isolation strategies include database transaction rollback, service mocking, and containerized testing environments that prevent test interference while maintaining testing authenticity and reproducibility.

### Testing Technology Stack

**Unit Testing Framework:**
- Jest as the primary testing framework for JavaScript/TypeScript code
- React Testing Library for component testing with realistic user interaction simulation
- Supertest for API endpoint testing with request/response validation
- ts-jest for TypeScript compilation and testing integration

**Integration Testing Tools:**
- Docker Compose for test environment containerization
- TestContainers for database testing with isolated PostgreSQL instances
- Redis testing with embedded Redis instances for session and caching validation
- WebSocket testing libraries for real-time communication validation

**End-to-End Testing Platform:**
- Playwright for cross-browser testing with realistic user workflow simulation
- Kubernetes testing environments for production-like integration validation
- Performance testing with k6 for load testing and scalability validation
- Visual regression testing for user interface consistency verification

## Unit Testing Standards and Practices

### Component Testing Guidelines

React component testing emphasizes user interaction simulation and realistic data scenarios that reflect hockey league management workflows. Component tests validate both functional behavior and accessibility requirements while ensuring proper integration with hockey domain business logic and state management systems.

```typescript
// Example: Player Card Component Testing
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { PlayerCard } from '@/components/PlayerCard';
import { createMockPlayer } from '@/test-utils/hockey-fixtures';

describe('PlayerCard Component', () => {
  const mockPlayer = createMockPlayer({
    position: 'center',
    overallRating: 85,
    contract: {
      salary: 5000000, // $5M in cents
      length: 3
    }
  });

  beforeEach(() => {
    jest.clearAllMocks();
  });

  it('displays player information correctly', () => {
    render(<PlayerCard player={mockPlayer} />);
    
    expect(screen.getByText(mockPlayer.firstName)).toBeInTheDocument();
    expect(screen.getByText(mockPlayer.lastName)).toBeInTheDocument();
    expect(screen.getByText('C')).toBeInTheDocument(); // Position abbreviation
    expect(screen.getByText('85')).toBeInTheDocument(); // Overall rating
  });

  it('handles player action callbacks correctly', async () => {
    const mockOnAction = jest.fn();
    render(
      <PlayerCard 
        player={mockPlayer} 
        onPlayerAction={mockOnAction}
        canEdit={true}
      />
    );

    const actionButton = screen.getByRole('button', { name: /trade player/i });
    fireEvent.click(actionButton);

    await waitFor(() => {
      expect(mockOnAction).toHaveBeenCalledWith(mockPlayer.id, {
        type: 'trade',
        playerId: mockPlayer.id
      });
    });
  });

  it('validates salary cap display formatting', () => {
    render(<PlayerCard player={mockPlayer} showFinancials={true} />);
    
    // Verify salary is displayed in millions with proper formatting
    expect(screen.getByText('$5.00M')).toBeInTheDocument();
  });

  it('meets accessibility requirements', () => {
    render(<PlayerCard player={mockPlayer} />);
    
    // Verify ARIA labels and roles
    expect(screen.getByRole('article')).toHaveAttribute(
      'aria-label', 
      `Player ${mockPlayer.firstName} ${mockPlayer.lastName}`
    );
  });
});
```

### Business Logic Testing

Hockey business logic testing requires comprehensive validation of domain-specific calculations, rule enforcement, and statistical accuracy. Business logic tests include mathematical verification, edge case handling, and integration with hockey industry standards while ensuring computational accuracy and regulatory compliance.

```typescript
// Example: Salary Cap Calculator Testing
import { SalaryCapCalculator } from '@/services/finance/SalaryCapCalculator';
import { createMockTeam, createMockContract } from '@/test-utils/hockey-fixtures';

describe('SalaryCapCalculator', () => {
  let calculator: SalaryCapCalculator;
  
  beforeEach(() => {
    calculator = new SalaryCapCalculator();
  });

  describe('calculateTeamSalaryCap', () => {
    it('calculates basic salary cap usage correctly', () => {
      const team = createMockTeam({
        contracts: [
          createMockContract({ annualValue: 8000000, isActive: true }),
          createMockContract({ annualValue: 6500000, isActive: true }),
          createMockContract({ annualValue: 2000000, isActive: false }) // Inactive
        ]
      });

      const result = calculator.calculateTeamSalaryCap(team.contracts);

      expect(result.used).toBe(14500000); // Only active contracts
      expect(result.available).toBe(70500000); // 85M cap - 14.5M used
      expect(result.isCompliant).toBe(true);
      expect(result.utilizationPercentage).toBeCloseTo(0.1706, 4);
    });

    it('handles LTIR calculations correctly', () => {
      const team = createMockTeam({
        contracts: [
          createMockContract({ 
            annualValue: 10000000, 
            isActive: true, 
            ltirStatus: true 
          }),
          createMockContract({ annualValue: 75000000, isActive: true })
        ]
      });

      const result = calculator.calculateTeamSalaryCap(team.contracts);

      // LTIR should provide relief allowing team to exceed cap
      expect(result.isCompliant).toBe(true);
      expect(result.ltirRelief).toBe(10000000);
      expect(result.effectiveCapSpace).toBeGreaterThan(0);
    });

    it('validates salary cap violations accurately', () => {
      const team = createMockTeam({
        contracts: Array(23).fill(null).map(() => 
          createMockContract({ annualValue: 4000000, isActive: true })
        )
      });

      const result = calculator.calculateTeamSalaryCap(team.contracts);

      expect(result.isCompliant).toBe(false);
      expect(result.overage).toBe(7000000); // 92M - 85M cap
      expect(result.violationType).toBe('salary_cap_exceeded');
    });

    it('handles performance bonus edge cases', () => {
      const team = createMockTeam({
        contracts: [
          createMockContract({ 
            annualValue: 2000000,
            performanceBonuses: [
              { type: 'goals', threshold: 30, amount: 1000000 },
              { type: 'assists', threshold: 50, amount: 500000 }
            ],
            isActive: true
          })
        ]
      });

      const result = calculator.calculateTeamSalaryCap(team.contracts);

      // Performance bonuses should be included in cap calculations
      expect(result.potentialBonusOverage).toBe(1500000);
      expect(result.worstCaseScenario).toBe(3500000);
    });
  });

  describe('validatePlayerSigning', () => {
    it('approves valid player signings', () => {
      const team = createMockTeam({
        currentCapUsage: 75000000 // 75M used
      });
      
      const newContract = createMockContract({ annualValue: 5000000 });

      const result = calculator.validatePlayerSigning(team, newContract);

      expect(result.isValid).toBe(true);
      expect(result.projectedCapUsage).toBe(80000000);
      expect(result.remainingSpace).toBe(5000000);
    });

    it('rejects signings that exceed salary cap', () => {
      const team = createMockTeam({
        currentCapUsage: 82000000 // 82M used
      });
      
      const expensiveContract = createMockContract({ annualValue: 8000000 });

      const result = calculator.validatePlayerSigning(team, expensiveContract);

      expect(result.isValid).toBe(false);
      expect(result.violations).toContain('salary_cap_exceeded');
      expect(result.overage).toBe(5000000);
      expect(result.suggestions).toContain('Consider releasing players to create cap space');
    });
  });
});
```

### Game Simulation Testing

Game simulation testing validates statistical accuracy, probability distributions, and realistic hockey gameplay patterns. Simulation tests include large-sample statistical validation, edge case scenario testing, and performance verification while ensuring computational efficiency and domain authenticity.

```typescript
// Example: Game Simulation Engine Testing
import { GameSimulationEngine } from '@/services/simulation/GameSimulationEngine';
import { createMockPlayer, createMockTeam, createMockGame } from '@/test-utils/hockey-fixtures';

describe('GameSimulationEngine', () => {
  let engine: GameSimulationEngine;
  let homeTeam: Team;
  let awayTeam: Team;

  beforeEach(() => {
    engine = new GameSimulationEngine();
    homeTeam = createMockTeam({ name: 'Home Team' });
    awayTeam = createMockTeam({ name: 'Away Team' });
  });

  describe('simulateGame', () => {
    it('produces realistic game results', async () => {
      const gameResults = [];
      
      // Simulate multiple games for statistical validation
      for (let i = 0; i < 1000; i++) {
        const game = await engine.simulateGame(homeTeam, awayTeam);
        gameResults.push(game);
      }

      // Validate statistical distributions
      const totalGoals = gameResults.reduce((sum, game) => 
        sum + game.homeScore + game.awayScore, 0
      );
      const averageGoalsPerGame = totalGoals / gameResults.length;

      // NHL average is approximately 6.2 goals per game
      expect(averageGoalsPerGame).toBeGreaterThan(5.0);
      expect(averageGoalsPerGame).toBeLessThan(7.5);

      // Validate score distributions are realistic
      const shutouts = gameResults.filter(game => 
        game.homeScore === 0 || game.awayScore === 0
      ).length;
      const shutoutPercentage = shutouts / gameResults.length;

      // NHL shutout rate is approximately 8-12%
      expect(shutoutPercentage).toBeGreaterThan(0.05);
      expect(shutoutPercentage).toBeLessThan(0.15);
    });

    it('handles overtime and shootout scenarios correctly', async () => {
      // Create evenly matched teams to increase overtime probability
      const evenTeam1 = createMockTeam({
        averageSkaterRating: 75,
        goaltenderRating: 85
      });
      const evenTeam2 = createMockTeam({
        averageSkaterRating: 75,
        goaltenderRating: 85
      });

      const overtimeGames = [];
      
      for (let i = 0; i < 500; i++) {
        const game = await engine.simulateGame(evenTeam1, evenTeam2);
        if (game.overtime || game.shootout) {
          overtimeGames.push(game);
        }
      }

      // Validate overtime frequency (NHL ~23% of games go to overtime)
      const overtimeRate = overtimeGames.length / 500;
      expect(overtimeRate).toBeGreaterThan(0.15);
      expect(overtimeRate).toBeLessThan(0.35);

      // Validate shootout subset of overtime games
      const shootoutGames = overtimeGames.filter(game => game.shootout);
      const shootoutRate = shootoutGames.length / overtimeGames.length;
      expect(shootoutRate).toBeGreaterThan(0.25); // ~25-40% of overtime games go to shootout
    });

    it('applies player ratings realistically', async () => {
      const superstarTeam = createMockTeam({
        players: Array(20).fill(null).map(() => createMockPlayer({
          ratings: { shooting: 95, passing: 95, defense: 90 }
        }))
      });

      const weakTeam = createMockTeam({
        players: Array(20).fill(null).map(() => createMockPlayer({
          ratings: { shooting: 55, passing: 55, defense: 55 }
        }))
      });

      const results = [];
      for (let i = 0; i < 100; i++) {
        const game = await engine.simulateGame(superstarTeam, weakTeam);
        results.push(game);
      }

      // Superstar team should win significantly more often
      const superstarWins = results.filter(game => 
        game.homeScore > game.awayScore
      ).length;
      const winPercentage = superstarWins / results.length;

      expect(winPercentage).toBeGreaterThan(0.70); // Should win >70% of games
    });
  });

  describe('generateGameEvents', () => {
    it('creates realistic event sequences', () => {
      const game = createMockGame(homeTeam, awayTeam);
      const events = engine.generateGameEvents(game);

      // Validate event types are realistic
      const goals = events.filter(e => e.type === 'goal');
      const shots = events.filter(e => e.type === 'shot');
      const penalties = events.filter(e => e.type === 'penalty');

      // Basic ratio validations
      expect(shots.length).toBeGreaterThan(goals.length * 5); // Shot-to-goal ratio
      expect(penalties.length).toBeGreaterThan(2); // Minimum penalties per game
      expect(penalties.length).toBeLessThan(20); // Maximum realistic penalties

      // Validate timing constraints
      events.forEach(event => {
        expect(event.period).toBeGreaterThanOrEqual(1);
        expect(event.period).toBeLessThanOrEqual(3);
        expect(event.timeRemaining).toMatch(/^\d{1,2}:\d{2}$/);
      });
    });
  });
});
```

## Integration Testing Methodologies

### API Endpoint Testing

API integration testing validates complete request-response cycles, authentication flows, and database interactions while ensuring proper error handling and hockey domain business rule enforcement. API tests include realistic data scenarios and comprehensive edge case validation.

```typescript
// Example: API Integration Testing
import request from 'supertest';
import { app } from '@/server';
import { setupTestDatabase, cleanupTestDatabase } from '@/test-utils/database';
import { createTestUser, createTestLeague, createTestTeam } from '@/test-utils/fixtures';

describe('Player Management API', () => {
  let testUser: User;
  let testLeague: League;
  let testTeam: Team;
  let authToken: string;

  beforeAll(async () => {
    await setupTestDatabase();
    testUser = await createTestUser({ role: 'general_manager' });
    testLeague = await createTestLeague();
    testTeam = await createTestTeam({ 
      leagueId: testLeague.id,
      managerId: testUser.id 
    });
    
    // Authenticate user for protected endpoints
    const authResponse = await request(app)
      .post('/api/auth/login')
      .send({
        email: testUser.email,
        password: 'test_password'
      });
    
    authToken = authResponse.body.accessToken;
  });

  afterAll(async () => {
    await cleanupTestDatabase();
  });

  describe('POST /api/teams/:teamId/players/sign', () => {
    it('successfully signs a player within salary cap', async () => {
      const player = await createTestPlayer({ 
        leagueId: testLeague.id,
        currentTeamId: null // Free agent
      });

      const contractData = {
        playerId: player.id,
        salary: 2000000, // $2M
        length: 3,
        position: 'left_wing'
      };

      const response = await request(app)
        .post(`/api/teams/${testTeam.id}/players/sign`)
        .set('Authorization', `Bearer ${authToken}`)
        .send(contractData)
        .expect(201);

      expect(response.body.success).toBe(true);
      expect(response.body.transaction.type).toBe('player_signing');
      expect(response.body.salaryCap.used).toBeGreaterThan(0);
      expect(response.body.salaryCap.isCompliant).toBe(true);
    });

    it('rejects signing that exceeds salary cap', async () => {
      // Set team near salary cap limit
      await setTeamSalaryUsage(testTeam.id, 83000000); // $83M of $85M cap

      const player = await createTestPlayer({ 
        leagueId: testLeague.id,
        currentTeamId: null 
      });

      const expensiveContract = {
        playerId: player.id,
        salary: 5000000, // $5M - would exceed cap
        length: 2,
        position: 'center'
      };

      const response = await request(app)
        .post(`/api/teams/${testTeam.id}/players/sign`)
        .set('Authorization', `Bearer ${authToken}`)
        .send(expensiveContract)
        .expect(422);

      expect(response.body.error).toBe('Salary cap violation');
      expect(response.body.details.overage).toBe(3000000);
      expect(response.body.suggestions).toContain(
        'Consider releasing players to create cap space'
      );
    });

    it('validates player eligibility correctly', async () => {
      const signedPlayer = await createTestPlayer({ 
        leagueId: testLeague.id,
        currentTeamId: 'other-team-id' // Already signed
      });

      const contractData = {
        playerId: signedPlayer.id,
        salary: 1000000,
        length: 1,
        position: 'right_wing'
      };

      const response = await request(app)
        .post(`/api/teams/${testTeam.id}/players/sign`)
        .set('Authorization', `Bearer ${authToken}`)
        .send(contractData)
        .expect(400);

      expect(response.body.error).toBe('Player not eligible for signing');
      expect(response.body.details.reason).toBe('Player already under contract');
    });

    it('enforces proper authorization', async () => {
      const unauthorizedUser = await createTestUser({ role: 'player' });
      const unauthorizedToken = await getAuthToken(unauthorizedUser);

      const player = await createTestPlayer({ 
        leagueId: testLeague.id,
        currentTeamId: null 
      });

      const response = await request(app)
        .post(`/api/teams/${testTeam.id}/players/sign`)
        .set('Authorization', `Bearer ${unauthorizedToken}`)
        .send({
          playerId: player.id,
          salary: 1000000,
          length: 1,
          position: 'center'
        })
        .expect(403);

      expect(response.body.error).toBe('Insufficient permissions');
    });
  });

  describe('GET /api/teams/:teamId/salary-cap', () => {
    it('returns accurate salary cap information', async () => {
      const response = await request(app)
        .get(`/api/teams/${testTeam.id}/salary-cap`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);

      expect(response.body).toHaveProperty('limit');
      expect(response.body).toHaveProperty('used');
      expect(response.body).toHaveProperty('available');
      expect(response.body).toHaveProperty('isCompliant');
      expect(response.body.limit).toBe(85000000); // Default NHL cap
      expect(response.body.used + response.body.available).toBe(response.body.limit);
    });
  });
});
```

### Database Integration Testing

Database integration testing validates data persistence, transaction management, and constraint enforcement while ensuring referential integrity and hockey domain business rule compliance at the database level.

```typescript
// Example: Database Integration Testing
import { DatabaseService } from '@/services/database/DatabaseService';
import { PlayerRepository } from '@/repositories/PlayerRepository';
import { TeamRepository } from '@/repositories/TeamRepository';
import { setupTestDatabase, getTestConnection } from '@/test-utils/database';

describe('Player-Team Database Integration', () => {
  let db: DatabaseService;
  let playerRepo: PlayerRepository;
  let teamRepo: TeamRepository;

  beforeAll(async () => {
    db = await setupTestDatabase();
    playerRepo = new PlayerRepository(db);
    teamRepo = new TeamRepository(db);
  });

  afterEach(async () => {
    await db.query('TRUNCATE players, teams, contracts CASCADE');
  });

  afterAll(async () => {
    await db.close();
  });

  describe('Player Assignment and Constraints', () => {
    it('enforces roster size limits through database constraints', async () => {
      const team = await teamRepo.create({
        name: 'Test Team',
        leagueId: 'test-league-id',
        maxRosterSize: 23
      });

      // Create and assign 23 players (maximum)
      const players = [];
      for (let i = 0; i < 23; i++) {
        const player = await playerRepo.create({
          firstName: `Player${i}`,
          lastName: 'Test',
          position: 'center',
          leagueId: 'test-league-id'
        });
        
        await playerRepo.assignToTeam(player.id, team.id);
        players.push(player);
      }

      // Attempt to assign 24th player should fail
      const extraPlayer = await playerRepo.create({
        firstName: 'Extra',
        lastName: 'Player',
        position: 'left_wing',
        leagueId: 'test-league-id'
      });

      await expect(
        playerRepo.assignToTeam(extraPlayer.id, team.id)
      ).rejects.toThrow('Roster size limit exceeded');
    });

    it('maintains referential integrity during team operations', async () => {
      const team = await teamRepo.create({
        name: 'Test Team',
        leagueId: 'test-league-id'
      });

      const player = await playerRepo.create({
        firstName: 'Test',
        lastName: 'Player',
        position: 'goaltender',
        leagueId: 'test-league-id'
      });

      await playerRepo.assignToTeam(player.id, team.id);

      // Verify foreign key relationship
      const assignedPlayer = await playerRepo.findById(player.id);
      expect(assignedPlayer.currentTeamId).toBe(team.id);

      // Attempt to delete team with assigned players should fail
      await expect(
        teamRepo.delete(team.id)
      ).rejects.toThrow('Cannot delete team with assigned players');
    });

    it('handles concurrent player assignment transactions', async () => {
      const team = await teamRepo.create({
        name: 'Test Team',
        leagueId: 'test-league-id'
      });

      const player = await playerRepo.create({
        firstName: 'Contested',
        lastName: 'Player',
        position: 'right_defense',
        leagueId: 'test-league-id'
      });

      const team2 = await teamRepo.create({
        name: 'Test Team 2',
        leagueId: 'test-league-id'
      });

      // Attempt concurrent assignment to different teams
      const assignments = await Promise.allSettled([
        playerRepo.assignToTeam(player.id, team.id),
        playerRepo.assignToTeam(player.id, team2.id)
      ]);

      // One should succeed, one should fail
      const successful = assignments.filter(result => result.status === 'fulfilled');
      const failed = assignments.filter(result => result.status === 'rejected');

      expect(successful).toHaveLength(1);
      expect(failed).toHaveLength(1);
    });
  });

  describe('Contract and Salary Cap Data Integrity', () => {
    it('maintains salary cap consistency across transactions', async () => {
      const team = await teamRepo.create({
        name: 'Financial Test Team',
        leagueId: 'test-league-id',
        salaryCap: 85000000
      });

      await db.transaction(async (trx) => {
        // Create multiple contracts within single transaction
        for (let i = 0; i < 5; i++) {
          const player = await playerRepo.create({
            firstName: `Player${i}`,
            lastName: 'Contract',
            position: 'center',
            leagueId: 'test-league-id'
          }, trx);

          await playerRepo.createContract({
            playerId: player.id,
            teamId: team.id,
            salary: 10000000, // $10M each
            length: 3
          }, trx);
        }

        // Verify total doesn't exceed cap within transaction
        const capUsage = await teamRepo.calculateSalaryCapUsage(team.id, trx);
        expect(capUsage.total).toBe(50000000);
        expect(capUsage.isCompliant).toBe(true);
      });

      // Verify persistence after transaction commit
      const finalCapUsage = await teamRepo.calculateSalaryCapUsage(team.id);
      expect(finalCapUsage.total).toBe(50000000);
    });
  });
});
```

## Real-Time Feature Testing

### WebSocket Communication Testing

Real-time feature testing validates WebSocket connections, message broadcasting, and live synchronization while ensuring proper connection management and error handling throughout concurrent user sessions and system interactions.

```typescript
// Example: WebSocket Integration Testing
import { io as Client, Socket } from 'socket.io-client';
import { createTestServer } from '@/test-utils/server';
import { createTestUser, authenticateTestUser } from '@/test-utils/auth';

describe('Real-Time Game Simulation WebSocket', () => {
  let server: any;
  let clientSocket: Socket;
  let adminSocket: Socket;
  let port: number;

  beforeAll(async () => {
    const testServer = await createTestServer();
    server = testServer.server;
    port = testServer.port;
  });

  afterAll(async () => {
    await server.close();
  });

  beforeEach(async () => {
    // Create authenticated WebSocket connections
    const user = await createTestUser({ role: 'general_manager' });
    const admin = await createTestUser({ role: 'league_commissioner' });
    
    const userToken = await authenticateTestUser(user);
    const adminToken = await authenticateTestUser(admin);

    clientSocket = Client(`http://localhost:${port}`, {
      auth: { token: userToken }
    });

    adminSocket = Client(`http://localhost:${port}`, {
      auth: { token: adminToken }
    });

    await Promise.all([
      new Promise(resolve => clientSocket.on('connect', resolve)),
      new Promise(resolve => adminSocket.on('connect', resolve))
    ]);
  });

  afterEach(() => {
    clientSocket.disconnect();
    adminSocket.disconnect();
  });

  describe('Game Simulation Broadcasting', () => {
    it('broadcasts game events to subscribed clients', async () => {
      const gameId = 'test-game-id';
      const receivedEvents: any[] = [];

      // Subscribe to game updates
      clientSocket.emit('subscribe_game', { gameId });
      
      clientSocket.on('game_event', (event) => {
        receivedEvents.push(event);
      });

      // Admin initiates game simulation
      adminSocket.emit('start_game_simulation', { gameId });

      // Wait for simulation events
      await new Promise(resolve => setTimeout(resolve, 2000));

      expect(receivedEvents.length).toBeGreaterThan(0);
      expect(receivedEvents[0]).toHaveProperty('type');
      expect(receivedEvents[0]).toHaveProperty('gameId', gameId);
      expect(receivedEvents[0]).toHaveProperty('timestamp');
    });

    it('handles connection drops and reconnection gracefully', async () => {
      const gameId = 'reconnection-test-game';
      let eventCount = 0;

      clientSocket.emit('subscribe_game', { gameId });
      clientSocket.on('game_event', () => eventCount++);

      // Start simulation
      adminSocket.emit('start_game_simulation', { gameId });
      
      // Wait for some events
      await new Promise(resolve => setTimeout(resolve, 1000));
      const eventsBeforeDisconnect = eventCount;

      // Simulate connection drop
      clientSocket.disconnect();
      
      // Wait during disconnection
      await new Promise(resolve => setTimeout(resolve, 1000));

      // Reconnect
      const userToken = await authenticateTestUser(await createTestUser());
      clientSocket = Client(`http://localhost:${port}`, {
        auth: { token: userToken }
      });

      await new Promise(resolve => clientSocket.on('connect', resolve));

      // Resubscribe and verify catch-up events
      clientSocket.emit('subscribe_game', { gameId });
      clientSocket.on('game_event', () => eventCount++);
      clientSocket.on('game_state_sync', (state) => {
        expect(state.gameId).toBe(gameId);
        expect(state.eventsSinceDisconnect).toBeGreaterThan(0);
      });

      await new Promise(resolve => setTimeout(resolve, 1000));
      expect(eventCount).toBeGreaterThan(eventsBeforeDisconnect);
    });

    it('enforces proper authorization for game control', async () => {
      const gameId = 'auth-test-game';
      let unauthorizedAttempt = false;

      clientSocket.on('error', (error) => {
        if (error.message.includes('Insufficient permissions')) {
          unauthorizedAttempt = true;
        }
      });

      // Regular user attempts to control simulation (should fail)
      clientSocket.emit('pause_game_simulation', { gameId });

      await new Promise(resolve => setTimeout(resolve, 500));
      expect(unauthorizedAttempt).toBe(true);

      // Admin should succeed
      let adminControlSuccessful = false;
      adminSocket.on('simulation_paused', () => {
        adminControlSuccessful = true;
      });

      adminSocket.emit('pause_game_simulation', { gameId });
      await new Promise(resolve => setTimeout(resolve, 500));
      expect(adminControlSuccessful).toBe(true);
    });
  });

  describe('Real-Time Notifications', () => {
    it('delivers trade notifications to relevant parties', async () => {
      const tradeProposal = {
        fromTeamId: 'team-1',
        toTeamId: 'team-2',
        players: ['player-1-id'],
        offerDetails: 'Test trade proposal'
      };

      const notifications: any[] = [];
      clientSocket.on('trade_notification', (notification) => {
        notifications.push(notification);
      });

      // Simulate trade proposal from admin
      adminSocket.emit('propose_trade', tradeProposal);

      await new Promise(resolve => setTimeout(resolve, 1000));

      expect(notifications).toHaveLength(1);
      expect(notifications[0].type).toBe('trade_proposal');
      expect(notifications[0].fromTeamId).toBe(tradeProposal.fromTeamId);
    });
  });
});
```

## Performance Testing Standards

### Load Testing for Game Simulation

Performance testing validates system behavior under realistic load conditions including concurrent game simulations, multiple user sessions, and peak usage scenarios while ensuring acceptable response times and resource utilization throughout operational scaling.

```typescript
// Example: Performance Testing with k6
import { check, sleep } from 'k6';
import http from 'k6/http';
import ws from 'k6/ws';

export let options = {
  scenarios: {
    game_simulation_load: {
      executor: 'ramping-vus',
      startVUs: 1,
      stages: [
        { duration: '2m', target: 10 },   // Ramp up to 10 users
        { duration: '5m', target: 50 },   // Ramp up to 50 users
        { duration: '10m', target: 100 }, // Ramp up to 100 users
        { duration: '5m', target: 50 },   // Ramp down to 50 users
        { duration: '2m', target: 0 },    // Ramp down to 0 users
      ],
    },
    websocket_connections: {
      executor: 'constant-vus',
      vus: 200,
      duration: '15m',
    }
  },
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% of requests under 500ms
    http_req_failed: ['rate<0.1'],    // Error rate under 10%
    ws_connecting: ['p(95)<1000'],    // WebSocket connection under 1s
  }
};

export default function() {
  // Test API endpoints under load
  const authResponse = http.post('http://localhost:3001/api/auth/login', {
    email: 'testuser@example.com',
    password: 'testpassword'
  });

  check(authResponse, {
    'authentication successful': (r) => r.status === 200,
    'auth response time OK': (r) => r.timings.duration < 200,
  });

  const token = authResponse.json('accessToken');

  // Test game simulation API
  const gameResponse = http.post(
    'http://localhost:3001/api/games/simulate',
    JSON.stringify({
      gameId: 'load-test-game',
      mode: 'accelerated'
    }),
    {
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      }
    }
  );

  check(gameResponse, {
    'simulation started': (r) => r.status === 202,
    'simulation response time OK': (r) => r.timings.duration < 1000,
  });

  // Test WebSocket connections
  const wsUrl = 'ws://localhost:3001/ws';
  const response = ws.connect(wsUrl, {
    headers: { Authorization: `Bearer ${token}` }
  }, function(socket) {
    socket.on('open', () => {
      socket.send(JSON.stringify({
        type: 'subscribe_game_updates',
        gameId: 'load-test-game'
      }));
    });

    socket.on('message', (data) => {
      const message = JSON.parse(data);
      check(message, {
        'valid game update': (msg) => msg.type === 'game_event',
        'has required fields': (msg) => msg.gameId && msg.timestamp,
      });
    });

    socket.setTimeout(30000); // 30 second timeout
  });

  sleep(1);
}

// Stress test specifically for salary cap calculations
export function salaryCap() {
  const token = getAuthToken();
  
  for (let i = 0; i < 100; i++) {
    const response = http.get(
      `http://localhost:3001/api/teams/team-${i % 10}/salary-cap`,
      { headers: { Authorization: `Bearer ${token}` } }
    );

    check(response, {
      'salary cap calculation successful': (r) => r.status === 200,
      'calculation time acceptable': (r) => r.timings.duration < 100,
    });
  }
}
```

## Test Data Management and Fixtures

### Hockey Domain Test Data Generation

Test data management provides realistic hockey scenarios and comprehensive edge case coverage while ensuring reproducible test conditions and maintaining data privacy throughout testing activities and development workflows.

```typescript
// Example: Hockey Domain Test Fixtures
import { faker } from '@faker-js/faker';

export interface TestFixtureOptions {
  realistic?: boolean;
  includeEdgeCases?: boolean;
  seasonYear?: number;
}

export class HockeyTestFixtures {
  private readonly NHL_NATIONALITIES = {
    'CAN': 0.45, 'USA': 0.25, 'SWE': 0.08, 'FIN': 0.05,
    'RUS': 0.04, 'CZE': 0.03, 'SVK': 0.02, 'OTHER': 0.08
  };

  private readonly POSITION_DISTRIBUTION = {
    'left_wing': 0.15, 'center': 0.15, 'right_wing': 0.15,
    'left_defense': 0.15, 'right_defense': 0.15, 'goaltender': 0.25
  };

  createMockPlayer(overrides: Partial<Player> = {}): Player {
    const nationality = this.selectNationality();
    const position = overrides.position || this.selectPosition();
    const age = faker.number.int({ min: 18, max: 35 });

    return {
      id: faker.string.uuid(),
      firstName: this.generateNameByNationality(nationality, 'first'),
      lastName: this.generateNameByNationality(nationality, 'last'),
      position,
      nationality,
      age,
      height: faker.number.int({ min: 165, max: 205 }), // cm
      weight: faker.number.int({ min: 70, max: 115 }), // kg
      birthDate: faker.date.birthdate({ min: 18, max: 35, mode: 'age' }),
      handedness: faker.helpers.weightedArrayElement([
        { weight: 0.7, value: 'left' },
        { weight: 0.3, value: 'right' }
      ]),
      draftYear: age >= 18 ? faker.date.past({ years: age - 18 }).getFullYear() : null,
      overallRating: this.generateRatingByAge(age),
      ratings: this.generatePositionRatings(position, age),
      ...overrides
    };
  }

  createMockTeam(overrides: Partial<Team> = {}): Team {
    const cities = ['Toronto', 'Montreal', 'Vancouver', 'Calgary', 'Edmonton', 'Ottawa'];
    const teamNames = ['Maple Leafs', 'Canadiens', 'Canucks', 'Flames', 'Oilers', 'Senators'];
    
    const cityIndex = faker.number.int({ min: 0, max: cities.length - 1 });
    
    return {
      id: faker.string.uuid(),
      name: `${cities[cityIndex]} ${teamNames[cityIndex]}`,
      city: cities[cityIndex],
      abbreviation: this.generateAbbreviation(cities[cityIndex]),
      leagueId: faker.string.uuid(),
      divisionId: faker.string.uuid(),
      primaryColor: faker.color.rgb(),
      secondaryColor: faker.color.rgb(),
      foundedYear: faker.number.int({ min: 1920, max: 2020 }),
      arena: {
        name: `${cities[cityIndex]} Arena`,
        capacity: faker.number.int({ min: 15000, max: 22000 }),
        location: cities[cityIndex]
      },
      currentSalaryUsed: faker.number.int({ min: 40000000, max: 80000000 }),
      aiControlled: faker.datatype.boolean(),
      ...overrides
    };
  }

  createMockLeague(overrides: Partial<League> = {}): League {
    return {
      id: faker.string.uuid(),
      name: faker.helpers.arrayElement([
        'National Hockey League',
        'American Hockey League', 
        'Canadian Hockey League',
        'International Hockey Federation'
      ]),
      federationId: faker.string.uuid(),
      salaryCap: 85000000, // NHL salary cap in cents
      seasonLength: 82,
      playoffTeams: 16,
      currentSeason: new Date().getFullYear(),
      currentPhase: 'regular_season',
      settings: {
        tradeDeadline: faker.date.future(),
        draftDate: faker.date.future(),
        salaryCapEnabled: true,
        luxuryTaxEnabled: false
      },
      ...overrides
    };
  }

  createMockContract(overrides: Partial<PlayerContract> = {}): PlayerContract {
    const salary = faker.number.int({ min: 700000, max: 15000000 }); // NHL min to max
    
    return {
      id: faker.string.uuid(),
      playerId: faker.string.uuid(),
      teamId: faker.string.uuid(),
      annualValue: salary,
      length: faker.number.int({ min: 1, max: 8 }),
      signingDate: faker.date.past(),
      expiryDate: faker.date.future(),
      noTradeClause: faker.datatype.boolean(0.2), // 20% chance
      noMovementClause: faker.datatype.boolean(0.1), // 10% chance
      performanceBonuses: this.generatePerformanceBonuses(),
      isActive: true,
      ...overrides
    };
  }

  createMockGame(homeTeam: Team, awayTeam: Team, overrides: Partial<Game> = {}): Game {
    const homeScore = faker.number.int({ min: 0, max: 8 });
    const awayScore = faker.number.int({ min: 0, max: 8 });
    const isOvertime = homeScore === awayScore ? faker.datatype.boolean(0.23) : false;
    
    return {
      id: faker.string.uuid(),
      seasonId: faker.string.uuid(),
      homeTeamId: homeTeam.id,
      awayTeamId: awayTeam.id,
      scheduledDate: faker.date.future(),
      gameType: 'regular',
      status: faker.helpers.arrayElement(['scheduled', 'in_progress', 'completed']),
      period: faker.number.int({ min: 1, max: 3 }),
      timeRemaining: '00:00',
      homeScore: isOvertime ? homeScore + 1 : homeScore,
      awayScore: isOvertime ? awayScore : awayScore,
      overtime: isOvertime,
      shootout: isOvertime ? faker.datatype.boolean(0.33) : false,
      attendance: faker.number.int({ 
        min: Math.floor(homeTeam.arena.capacity * 0.7),
        max: homeTeam.arena.capacity 
      }),
      ...overrides
    };
  }

  // Generate statistically accurate game events
  createMockGameEvents(game: Game, eventCount: number = 100): GameEvent[] {
    const events: GameEvent[] = [];
    const periods = [1, 2, 3];
    
    for (let i = 0; i < eventCount; i++) {
      const period = faker.helpers.arrayElement(periods);
      const timeRemaining = this.generateGameTime();
      
      events.push({
        id: faker.string.uuid(),
        gameId: game.id,
        period,
        timeRemaining,
        eventType: this.selectEventType(),
        teamId: faker.helpers.arrayElement([game.homeTeamId, game.awayTeamId]),
        primaryPlayerId: faker.string.uuid(),
        secondaryPlayerId: faker.datatype.boolean(0.4) ? faker.string.uuid() : null,
        description: this.generateEventDescription(),
        coordinates: {
          x: faker.number.int({ min: -100, max: 100 }),
          y: faker.number.int({ min: -42, max: 42 })
        },
        timestamp: faker.date.recent()
      });
    }

    return events.sort((a, b) => {
      if (a.period !== b.period) return a.period - b.period;
      return this.timeStringToSeconds(b.timeRemaining) - this.timeStringToSeconds(a.timeRemaining);
    });
  }

  private selectNationality(): string {
    return faker.helpers.weightedArrayElement(
      Object.entries(this.NHL_NATIONALITIES).map(([nat, weight]) => ({
        weight,
        value: nat
      }))
    );
  }

  private selectPosition(): PlayerPosition {
    return faker.helpers.weightedArrayElement(
      Object.entries(this.POSITION_DISTRIBUTION).map(([pos, weight]) => ({
        weight,
        value: pos as PlayerPosition
      }))
    );
  }

  private generateRatingByAge(age: number): number {
    // Peak performance around 26-28
    const peakAge = 27;
    const ageFactor = 1 - Math.abs(age - peakAge) * 0.02;
    const baseRating = faker.number.int({ min: 50, max: 95 });
    
    return Math.min(99, Math.max(25, Math.floor(baseRating * ageFactor)));
  }

  private generatePositionRatings(position: PlayerPosition, age: number): PlayerRatings {
    const overall = this.generateRatingByAge(age);
    const variance = faker.number.int({ min: -10, max: 10 });

    if (position === 'goaltender') {
      return {
        discipline: this.boundRating(overall + faker.number.int({ min: -5, max: 5 })),
        injury: this.boundRating(overall + faker.number.int({ min: -5, max: 5 })),
        fatigue: this.boundRating(overall + faker.number.int({ min: -5, max: 5 })),
        poise: this.boundRating(overall + faker.number.int({ min: -5, max: 5 })),
        movement: this.boundRating(overall + faker.number.int({ min: -5, max: 5 })),
        reboundControl: this.boundRating(overall + faker.number.int({ min: -5, max: 5 })),
        vision: this.boundRating(overall + faker.number.int({ min: -5, max: 5 })),
        aggressiveness: this.boundRating(overall + faker.number.int({ min: -5, max: 5 })),
        puckControl: this.boundRating(overall + faker.number.int({ min: -5, max: 5 })),
        flexibility: this.boundRating(overall + faker.number.int({ min: -5, max: 5 }))
      };
    }

    // Skater ratings with position-specific adjustments
    const defensiveBonus = position.includes('defense') ? 5 : 0;
    const offensiveBonus = position.includes('wing') || position === 'center' ? 5 : 0;

    return {
      discipline: this.boundRating(overall + faker.number.int({ min: -5, max: 5 })),
      injury: this.boundRating(overall + faker.number.int({ min: -5, max: 5 })),
      fatigue: this.boundRating(overall + faker.number.int({ min: -5, max: 5 })),
      passing: this.boundRating(overall + offensiveBonus + faker.number.int({ min: -5, max: 5 })),
      shooting: this.boundRating(overall + offensiveBonus + faker.number.int({ min: -5, max: 5 })),
      defense: this.boundRating(overall + defensiveBonus + faker.number.int({ min: -5, max: 5 })),
      puckControl: this.boundRating(overall + faker.number.int({ min: -5, max: 5 })),
      checking: this.boundRating(overall + defensiveBonus + faker.number.int({ min: -5, max: 5 })),
      fighting: this.boundRating(faker.number.int({ min: 25, max: 85 })), // Independent of overall
      poise: this.boundRating(overall + faker.number.int({ min: -5, max: 5 }))
    };
  }

  private boundRating(rating: number): number {
    return Math.min(99, Math.max(25, rating));
  }

  private generateGameTime(): string {
    const minutes = faker.number.int({ min: 0, max: 19 });
    const seconds = faker.number.int({ min: 0, max: 59 });
    return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
  }

  private timeStringToSeconds(timeString: string): number {
    const [minutes, seconds] = timeString.split(':').map(Number);
    return minutes * 60 + seconds;
  }
}

// Export singleton instance
export const hockeyFixtures = new HockeyTestFixtures();
```

## Automated Testing Pipeline

### Continuous Integration Testing

CI testing integration provides comprehensive validation throughout the development pipeline while ensuring code quality, hockey domain accuracy, and performance standards are maintained across all environment deployments and code changes.

```yaml
# GitHub Actions Testing Pipeline
name: Hockey League Simulation - Test Suite

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18.x, 20.x]

    services:
      postgres:
        image: timescale/timescaledb:latest-pg14
        env:
          POSTGRES_PASSWORD: test_password
          POSTGRES_DB: hockey_test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:7-alpine
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run linting
      run: npm run lint

    - name: Run type checking
      run: npm run type-check

    - name: Setup test database
      run: |
        npm run db:migrate:test
        npm run db:seed:test
      env:
        DATABASE_URL: postgresql://postgres:test_password@localhost:5432/hockey_test

    - name: Run unit tests
      run: npm run test:unit -- --coverage --watchAll=false
      env:
        DATABASE_URL: postgresql://postgres:test_password@localhost:5432/hockey_test
        REDIS_URL: redis://localhost:6379

    - name: Upload coverage reports
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info
        flags: unittests
        name: codecov-umbrella

  integration-tests:
    runs-on: ubuntu-latest
    needs: unit-tests

    services:
      postgres:
        image: timescale/timescaledb:latest-pg14
        env:
          POSTGRES_PASSWORD: integration_password
          POSTGRES_DB: hockey_integration
        ports:
          - 5432:5432

      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: 18.x
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Setup integration test environment
      run: |
        npm run db:migrate:integration
        npm run db:seed:integration
      env:
        DATABASE_URL: postgresql://postgres:integration_password@localhost:5432/hockey_integration

    - name: Run integration tests
      run: npm run test:integration
      env:
        DATABASE_URL: postgresql://postgres:integration_password@localhost:5432/hockey_integration
        REDIS_URL: redis://localhost:6379

  e2e-tests:
    runs-on: ubuntu-latest
    needs: integration-tests

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: 18.x
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Install Playwright browsers
      run: npx playwright install --with-deps

    - name: Start application in test mode
      run: |
        npm run build:test
        npm run start:test &
        sleep 30
      env:
        NODE_ENV: test
        DATABASE_URL: postgresql://test_user:test_password@localhost:5432/hockey_e2e

    - name: Run end-to-end tests
      run: npm run test:e2e

    - name: Upload test results
      uses: actions/upload-artifact@v3
      if: always()
      with:
        name: playwright-report
        path: playwright-report/
        retention-days: 30

  performance-tests:
    runs-on: ubuntu-latest
    needs: integration-tests
    if: github.ref == 'refs/heads/main'

    steps:
    - uses: actions/checkout@v3

    - name: Setup k6
      run: |
        sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
        echo "deb https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
        sudo apt-get update
        sudo apt-get install k6

    - name: Setup performance test environment
      run: |
        docker-compose -f docker-compose.perf.yml up -d
        sleep 60

    - name: Run performance tests
      run: k6 run tests/performance/load-test.js

    - name: Run simulation performance tests
      run: k6 run tests/performance/simulation-load.js

    - name: Cleanup performance environment
      run: docker-compose -f docker-compose.perf.yml down
```

This comprehensive testing guidelines document establishes systematic testing methodologies that ensure high-quality, reliable, and performant hockey league simulation functionality while supporting effective AI-assisted development workflows and maintaining hockey domain accuracy throughout the development lifecycle.