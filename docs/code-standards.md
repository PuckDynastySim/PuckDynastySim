# Code Standards - Hockey League Simulation System

This comprehensive code standards document establishes consistent development practices and quality guidelines for the Hockey League Simulation System. These standards are specifically optimized for AI-assisted development while ensuring maintainable, secure, and performant code that accurately represents hockey domain requirements.

## TypeScript and JavaScript Standards

### Type Safety and Interface Design

All code must utilize TypeScript with strict type checking enabled to ensure compile-time error detection and enhanced development experience. Interface definitions require comprehensive documentation explaining business context and usage patterns with clear indication of required versus optional properties and their hockey domain significance.

```typescript
// ✅ Good: Comprehensive interface with hockey context
interface Player {
  /** Unique player identifier */
  id: string;
  /** Player's first name */
  firstName: string;
  /** Player's last name */
  lastName: string;
  /** Playing position: 'left_wing' | 'center' | 'right_wing' | 'left_defense' | 'right_defense' | 'goaltender' */
  position: PlayerPosition;
  /** Overall skill rating (25-99 scale) */
  overallRating: number;
  /** Detailed skill ratings for simulation calculations */
  ratings: PlayerRatings;
  /** Current team assignment, null if unassigned */
  currentTeamId: string | null;
  /** Contract details including salary and terms */
  contract?: PlayerContract;
}

// ❌ Bad: Vague interface without context
interface Player {
  id: string;
  name: string;
  position: string;
  rating: number;
}
```

Union types and enums should be used for hockey-specific values including player positions, game events, and transaction types with clear documentation of all possible values and their business significance within hockey operations.

```typescript
// ✅ Good: Clear enum with hockey context
enum PlayerPosition {
  LEFT_WING = 'left_wing',
  CENTER = 'center',
  RIGHT_WING = 'right_wing',
  LEFT_DEFENSE = 'left_defense',
  RIGHT_DEFENSE = 'right_defense',
  GOALTENDER = 'goaltender'
}

enum GameEventType {
  GOAL = 'goal',
  ASSIST = 'assist',
  PENALTY = 'penalty',
  SAVE = 'save',
  SHOT = 'shot',
  HIT = 'hit',
  FACEOFF = 'faceoff'
}
```

### Function and Variable Naming Conventions

Function naming must use descriptive camelCase that clearly indicates the operation and context within hockey domain operations. Boolean variables should begin with appropriate prefixes such as "is", "has", "can", or "should" to indicate their nature and usage context.

```typescript
// ✅ Good: Descriptive function names with hockey context
function calculatePlayerOverallRating(ratings: PlayerRatings): number {
  // Implementation
}

function validateSalaryCapCompliance(team: Team, newContract: PlayerContract): boolean {
  // Implementation
}

function generateGameSimulationEvents(homeTeam: Team, awayTeam: Team): GameEvent[] {
  // Implementation
}

// Boolean variables with clear prefixes
const isPlayerEligibleForDraft = player.age >= 18;
const hasActivePenalty = player.penalties.some(p => p.isActive);
const canExecuteTrade = validateTradeCompliance(tradeProposal);
```

Constants should utilize UPPER_SNAKE_CASE formatting with clear indication of their purpose and scope within hockey operations. Magic numbers must be replaced with named constants that explain their hockey domain significance.

```typescript
// ✅ Good: Named constants with hockey context
const SALARY_CAP_LIMIT = 85_000_000; // in cents
const MIN_ROSTER_SIZE = 20;
const MAX_ROSTER_SIZE = 23;
const REGULATION_PERIOD_LENGTH_MINUTES = 20;
const OVERTIME_LENGTH_MINUTES = 5;

// ❌ Bad: Magic numbers without context
if (team.players.length < 20) {
  // What does 20 represent?
}
```

### Error Handling and Validation Patterns

Error handling must be comprehensive with specific error types for different failure scenarios including hockey business rule violations, system errors, and user input validation failures. Custom error classes should provide detailed context and recovery guidance.

```typescript
// ✅ Good: Custom error classes with hockey context
class SalaryCapViolationError extends Error {
  constructor(
    public readonly teamId: string,
    public readonly currentUsage: number,
    public readonly attemptedAddition: number,
    public readonly capLimit: number
  ) {
    super(`Salary cap violation: Team ${teamId} would exceed cap limit`);
    this.name = 'SalaryCapViolationError';
  }
}

class InvalidTradeError extends Error {
  constructor(
    public readonly reason: string,
    public readonly tradeDetails: TradeProposal
  ) {
    super(`Invalid trade: ${reason}`);
    this.name = 'InvalidTradeError';
  }
}

// Function with comprehensive error handling
async function executePlayerTrade(tradeProposal: TradeProposal): Promise<TradeResult> {
  try {
    // Validate salary cap compliance
    if (!validateSalaryCap(tradeProposal)) {
      throw new SalaryCapViolationError(/* parameters */);
    }
    
    // Validate roster requirements
    if (!validateRosterRequirements(tradeProposal)) {
      throw new InvalidTradeError('Roster requirements not met', tradeProposal);
    }
    
    // Execute trade
    return await processTradeTransaction(tradeProposal);
  } catch (error) {
    logger.error('Trade execution failed', { tradeProposal, error });
    throw error;
  }
}
```

## React Component Standards

### Component Architecture and Prop Design

React components must follow functional component patterns with TypeScript interfaces for all props including comprehensive documentation of usage patterns and integration requirements within hockey management workflows.

```typescript
// ✅ Good: Well-documented component interface
interface PlayerCardProps {
  /** Player data to display */
  player: Player;
  /** Whether to show detailed statistics */
  showDetailedStats?: boolean;
  /** Whether the current user can edit this player */
  canEdit?: boolean;
  /** Callback when player is selected for action */
  onPlayerAction?: (playerId: string, action: PlayerAction) => void;
  /** Additional CSS classes for styling */
  className?: string;
}

const PlayerCard: React.FC<PlayerCardProps> = ({
  player,
  showDetailedStats = false,
  canEdit = false,
  onPlayerAction,
  className
}) => {
  // Component implementation with proper error boundaries and loading states
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  // Comprehensive event handling with validation
  const handlePlayerAction = useCallback((action: PlayerAction) => {
    if (!canEdit && action.requiresEditPermission) {
      setError('Insufficient permissions for this action');
      return;
    }
    
    onPlayerAction?.(player.id, action);
  }, [player.id, canEdit, onPlayerAction]);

  // Render with proper error handling and accessibility
  return (
    <div className={`player-card ${className || ''}`} role="article" aria-label={`Player ${player.firstName} ${player.lastName}`}>
      {/* Component JSX with proper accessibility and error handling */}
    </div>
  );
};

export default PlayerCard;
```

Custom hooks must encapsulate complex business logic and provide consistent interfaces for data manipulation across different components. Hook naming should follow the "use" prefix convention with clear indication of functionality and usage context.

```typescript
// ✅ Good: Custom hook with hockey business logic
interface UsePlayerManagementOptions {
  teamId: string;
  includeInjured?: boolean;
}

interface PlayerManagementState {
  players: Player[];
  isLoading: boolean;
  error: string | null;
  salaryCap: SalaryCapInfo;
}

const usePlayerManagement = (options: UsePlayerManagementOptions): PlayerManagementState & {
  signPlayer: (playerId: string, contract: PlayerContract) => Promise<void>;
  releasePlayer: (playerId: string) => Promise<void>;
  proposeTradePlayer: (playerId: string, targetTeamId: string) => Promise<void>;
} => {
  // Hook implementation with comprehensive state management
  const [state, setState] = useState<PlayerManagementState>({
    players: [],
    isLoading: true,
    error: null,
    salaryCap: { used: 0, limit: 0, available: 0 }
  });

  // Business logic functions with validation
  const signPlayer = useCallback(async (playerId: string, contract: PlayerContract) => {
    try {
      setState(prev => ({ ...prev, isLoading: true, error: null }));
      
      // Validate salary cap before proceeding
      const salaryCapValid = await validateSalaryCapForSigning(options.teamId, contract);
      if (!salaryCapValid) {
        throw new SalaryCapViolationError(/* parameters */);
      }
      
      await playerService.signPlayer(playerId, contract);
      // Update state appropriately
    } catch (error) {
      setState(prev => ({ ...prev, error: error.message, isLoading: false }));
    }
  }, [options.teamId]);

  return { ...state, signPlayer, releasePlayer, proposeTradePlayer };
};
```

### State Management and Performance Optimization

State management should utilize React hooks with immutable update patterns and appropriate optimization through React.memo, useMemo, and useCallback where performance benefits are measurable. Component re-renders should be minimized through proper dependency arrays and state structure optimization.

```typescript
// ✅ Good: Optimized component with proper memoization
const TeamRoster = React.memo<TeamRosterProps>(({ teamId, onPlayerUpdate }) => {
  const { players, isLoading, error } = usePlayerManagement({ teamId });
  
  // Memoize expensive calculations
  const playersByPosition = useMemo(() => {
    return groupPlayersByPosition(players);
  }, [players]);
  
  const salaryCapUtilization = useMemo(() => {
    return calculateSalaryCapUtilization(players);
  }, [players]);
  
  // Memoize callback to prevent unnecessary re-renders
  const handlePlayerAction = useCallback((playerId: string, action: PlayerAction) => {
    onPlayerUpdate?.(playerId, action);
  }, [onPlayerUpdate]);

  // Component render logic with loading and error states
});
```

## Database and API Standards

### Query Optimization and Data Access Patterns

Database queries must utilize appropriate indexing strategies and avoid N+1 query patterns that degrade performance under realistic data volumes. Parameterized queries are required to prevent SQL injection while maintaining query readability and maintainability.

```typescript
// ✅ Good: Optimized query with proper indexing consideration
class PlayerRepository {
  async getPlayersByTeam(teamId: string, options: PlayerQueryOptions = {}): Promise<Player[]> {
    const query = `
      SELECT p.*, c.salary, c.contract_length
      FROM players p
      LEFT JOIN contracts c ON p.id = c.player_id AND c.is_active = true
      WHERE p.current_team_id = $1
      ${options.includeInjured ? '' : 'AND p.injury_status IS NULL'}
      ORDER BY p.jersey_number ASC
    `;
    
    const result = await this.database.query(query, [teamId]);
    return result.rows.map(row => this.mapRowToPlayer(row));
  }

  // Batch operations to avoid N+1 queries
  async getPlayersWithStatistics(playerIds: string[]): Promise<PlayerWithStats[]> {
    const query = `
      SELECT 
        p.*,
        s.goals, s.assists, s.points, s.games_played
      FROM players p
      LEFT JOIN player_statistics s ON p.id = s.player_id AND s.season = $1
      WHERE p.id = ANY($2)
    `;
    
    const result = await this.database.query(query, [currentSeason, playerIds]);
    return result.rows.map(row => this.mapRowToPlayerWithStats(row));
  }
}
```

API endpoint implementation must include comprehensive input validation, proper HTTP status codes, and consistent error response formats with appropriate security considerations and rate limiting.

```typescript
// ✅ Good: Well-structured API endpoint with validation
export async function POST(request: NextRequest) {
  try {
    // Authentication and authorization
    const user = await authenticateRequest(request);
    if (!user) {
      return NextResponse.json(
        { error: 'Authentication required' },
        { status: 401 }
      );
    }

    // Input validation with detailed error messages
    const body = await request.json();
    const validationResult = validateTradeProposal(body);
    if (!validationResult.isValid) {
      return NextResponse.json(
        { 
          error: 'Validation failed',
          details: validationResult.errors,
          suggestions: validationResult.suggestions
        },
        { status: 400 }
      );
    }

    // Authorization check for specific resource
    const hasPermission = await checkTradePermission(user, body.teamId);
    if (!hasPermission) {
      return NextResponse.json(
        { error: 'Insufficient permissions for trade execution' },
        { status: 403 }
      );
    }

    // Business logic execution with error handling
    const tradeResult = await executeTradeProposal(body);
    
    // Success response with comprehensive data
    return NextResponse.json({
      success: true,
      trade: tradeResult,
      timestamp: new Date().toISOString()
    }, { status: 201 });

  } catch (error) {
    logger.error('Trade execution endpoint error', { error, requestId: request.headers.get('x-request-id') });
    
    if (error instanceof SalaryCapViolationError) {
      return NextResponse.json(
        {
          error: 'Salary cap violation',
          details: {
            currentUsage: error.currentUsage,
            attemptedAddition: error.attemptedAddition,
            capLimit: error.capLimit
          }
        },
        { status: 422 }
      );
    }

    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

## Hockey Domain-Specific Standards

### Business Logic Implementation

Hockey business logic must maintain accuracy and authenticity through proper implementation of industry-standard rules and calculations. Salary cap calculations, player ratings, and game simulation algorithms require comprehensive validation and testing against real-world hockey scenarios.

```typescript
// ✅ Good: Accurate hockey business logic with validation
class SalaryCapCalculator {
  private readonly SALARY_CAP_LIMIT = 85_000_000; // Current NHL cap in cents
  
  calculateTeamSalaryCap(contracts: PlayerContract[]): SalaryCapInfo {
    const activeContracts = contracts.filter(contract => contract.isActive);
    
    // Calculate base salary usage
    const baseSalaryTotal = activeContracts.reduce((total, contract) => {
      return total + contract.annualValue;
    }, 0);
    
    // Account for performance bonuses (cap hit vs actual)
    const bonusOverages = this.calculateBonusOverages(activeContracts);
    
    // Calculate LTIR relief if applicable
    const ltirRelief = this.calculateLTIRRelief(activeContracts);
    
    const effectiveCapUsage = baseSalaryTotal + bonusOverages - ltirRelief;
    
    return {
      limit: this.SALARY_CAP_LIMIT,
      used: effectiveCapUsage,
      available: this.SALARY_CAP_LIMIT - effectiveCapUsage,
      isCompliant: effectiveCapUsage <= this.SALARY_CAP_LIMIT,
      projectedOverage: Math.max(0, effectiveCapUsage - this.SALARY_CAP_LIMIT)
    };
  }

  validatePlayerSigning(team: Team, contract: PlayerContract): ValidationResult {
    const currentCapInfo = this.calculateTeamSalaryCap(team.contracts);
    const projectedUsage = currentCapInfo.used + contract.annualValue;
    
    if (projectedUsage > this.SALARY_CAP_LIMIT) {
      return {
        isValid: false,
        errors: ['Signing would exceed salary cap limit'],
        details: {
          currentUsage: currentCapInfo.used,
          contractValue: contract.annualValue,
          projectedUsage,
          overage: projectedUsage - this.SALARY_CAP_LIMIT
        }
      };
    }
    
    return { isValid: true, errors: [] };
  }
}
```

Game simulation algorithms must implement realistic probability distributions and statistical accuracy while maintaining computational efficiency for real-time processing.

```typescript
// ✅ Good: Realistic game simulation with proper probability calculations
class GameSimulationEngine {
  simulateShot(shooter: Player, goaltender: Player, gameContext: GameContext): ShotResult {
    // Base shooting probability from player rating (25-99 scale)
    const baseShootingSkill = shooter.ratings.shooting / 100;
    const goalieSkill = goaltender.ratings.save / 100;
    
    // Situational modifiers based on game context
    const situationModifier = this.calculateSituationModifier(gameContext);
    const fatigueModifier = this.calculateFatigueModifier(shooter, gameContext);
    
    // Final goal probability calculation
    const goalProbability = baseShootingSkill * situationModifier * fatigueModifier * (1 - goalieSkill);
    
    // Generate random result with weighted probability
    const randomValue = Math.random();
    
    if (randomValue < goalProbability) {
      return {
        result: 'goal',
        shooter: shooter.id,
        goaltender: goaltender.id,
        gameTime: gameContext.timeRemaining,
        period: gameContext.period
      };
    } else {
      return {
        result: 'save',
        shooter: shooter.id,
        goaltender: goaltender.id,
        reboundGenerated: randomValue < (goalProbability + 0.3),
        gameTime: gameContext.timeRemaining,
        period: gameContext.period
      };
    }
  }
}
```

## Testing and Quality Assurance Standards

### Test Coverage and Organization

All code must include comprehensive test coverage with minimum 90% coverage for business logic functions and 80% coverage for user interface components. Tests should focus on hockey domain accuracy and edge case handling with realistic test data and scenarios.

```typescript
// ✅ Good: Comprehensive test suite with hockey domain focus
describe('SalaryCapCalculator', () => {
  let calculator: SalaryCapCalculator;
  let mockTeam: Team;

  beforeEach(() => {
    calculator = new SalaryCapCalculator();
    mockTeam = createMockTeam({
      contracts: [
        createMockContract({ annualValue: 10_000_000, isActive: true }),
        createMockContract({ annualValue: 8_500_000, isActive: true }),
        createMockContract({ annualValue: 2_000_000, isActive: false }) // injured reserve
      ]
    });
  });

  describe('calculateTeamSalaryCap', () => {
    it('should calculate basic salary cap usage correctly', () => {
      const result = calculator.calculateTeamSalaryCap(mockTeam.contracts);
      
      expect(result.used).toBe(18_500_000); // Only active contracts
      expect(result.available).toBe(66_500_000);
      expect(result.isCompliant).toBe(true);
    });

    it('should handle salary cap violations correctly', () => {
      const overCapTeam = createMockTeam({
        contracts: Array(23).fill(null).map(() => 
          createMockContract({ annualValue: 4_000_000, isActive: true })
        )
      });

      const result = calculator.calculateTeamSalaryCap(overCapTeam.contracts);
      
      expect(result.isCompliant).toBe(false);
      expect(result.projectedOverage).toBeGreaterThan(0);
    });

    it('should account for LTIR relief calculations', () => {
      const ltirTeam = createMockTeam({
        contracts: [
          createMockContract({ annualValue: 10_000_000, isActive: true, isLTIR: true }),
          createMockContract({ annualValue: 75_000_000, isActive: true })
        ]
      });

      const result = calculator.calculateTeamSalaryCap(ltirTeam.contracts);
      
      expect(result.isCompliant).toBe(true); // LTIR should provide relief
    });
  });

  describe('validatePlayerSigning', () => {
    it('should validate successful player signing', () => {
      const newContract = createMockContract({ annualValue: 5_000_000 });
      
      const result = calculator.validatePlayerSigning(mockTeam, newContract);
      
      expect(result.isValid).toBe(true);
      expect(result.errors).toHaveLength(0);
    });

    it('should reject signings that exceed salary cap', () => {
      const expensiveContract = createMockContract({ annualValue: 70_000_000 });
      
      const result = calculator.validatePlayerSigning(mockTeam, expensiveContract);
      
      expect(result.isValid).toBe(false);
      expect(result.errors).toContain('Signing would exceed salary cap limit');
      expect(result.details.overage).toBeGreaterThan(0);
    });
  });
});
```

## Performance and Security Standards

### Code Performance Guidelines

Performance-critical code must include appropriate optimization techniques including memoization, lazy loading, and efficient algorithm selection with measurable performance targets and validation procedures.

```typescript
// ✅ Good: Performance-optimized code with caching
class PlayerStatisticsService {
  private readonly cache = new Map<string, PlayerStatistics>();
  private readonly CACHE_TTL = 5 * 60 * 1000; // 5 minutes

  async getPlayerStatistics(playerId: string, season: number): Promise<PlayerStatistics> {
    const cacheKey = `${playerId}-${season}`;
    const cached = this.cache.get(cacheKey);
    
    // Return cached result if still valid
    if (cached && Date.now() - cached.timestamp < this.CACHE_TTL) {
      return cached.data;
    }

    // Fetch from database with optimized query
    const statistics = await this.repository.getPlayerStatistics(playerId, season);
    
    // Cache the result
    this.cache.set(cacheKey, {
      data: statistics,
      timestamp: Date.now()
    });

    return statistics;
  }

  // Batch processing for improved performance
  async getMultiplePlayerStatistics(playerIds: string[], season: number): Promise<Map<string, PlayerStatistics>> {
    const results = new Map<string, PlayerStatistics>();
    const uncachedIds: string[] = [];

    // Check cache first
    for (const playerId of playerIds) {
      const cached = this.cache.get(`${playerId}-${season}`);
      if (cached && Date.now() - cached.timestamp < this.CACHE_TTL) {
        results.set(playerId, cached.data);
      } else {
        uncachedIds.push(playerId);
      }
    }

    // Batch fetch uncached data
    if (uncachedIds.length > 0) {
      const batchResults = await this.repository.getBatchPlayerStatistics(uncachedIds, season);
      
      for (const [playerId, stats] of batchResults) {
        results.set(playerId, stats);
        this.cache.set(`${playerId}-${season}`, {
          data: stats,
          timestamp: Date.now()
        });
      }
    }

    return results;
  }
}
```

Security implementation must follow established patterns with comprehensive input validation, authorization checking, and audit logging throughout all system operations.

```typescript
// ✅ Good: Secure implementation with comprehensive validation
class TradeService {
  async proposePlayerTrade(
    userId: string,
    tradeProposal: TradeProposal
  ): Promise<TradeResult> {
    // Input validation
    const validation = await this.validateTradeProposal(tradeProposal);
    if (!validation.isValid) {
      throw new ValidationError('Invalid trade proposal', validation.errors);
    }

    // Authorization check
    const hasPermission = await this.authService.canUserExecuteTrade(userId, tradeProposal);
    if (!hasPermission) {
      throw new AuthorizationError('Insufficient permissions for trade execution');
    }

    // Audit logging
    await this.auditService.logTradeProposal({
      userId,
      tradeProposal,
      timestamp: new Date(),
      ipAddress: this.getClientIP(),
      userAgent: this.getUserAgent()
    });

    try {
      // Execute trade with transaction safety
      const result = await this.database.transaction(async (tx) => {
        return await this.executeTradeTransaction(tradeProposal, tx);
      });

      // Success audit
      await this.auditService.logTradeExecution({
        userId,
        tradeId: result.tradeId,
        status: 'completed',
        timestamp: new Date()
      });

      return result;
    } catch (error) {
      // Failure audit
      await this.auditService.logTradeExecution({
        userId,
        tradeProposal,
        status: 'failed',
        error: error.message,
        timestamp: new Date()
      });
      
      throw error;
    }
  }
}
```

These code standards ensure consistent, maintainable, and secure development practices while optimizing for AI-assisted development and hockey domain accuracy throughout the Hockey League Simulation System.