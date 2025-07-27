# Troubleshooting Guide - Hockey League Simulation System

This comprehensive troubleshooting guide provides systematic problem resolution procedures for common issues encountered during development, deployment, and operation of the Hockey League Simulation System. The guide emphasizes practical solutions while maintaining hockey domain accuracy and system reliability throughout problem diagnosis and resolution activities.

## Development Environment Issues

### Database Connection and Setup Problems

**Problem: PostgreSQL Connection Refused**
```
Error: connect ECONNREFUSED 127.0.0.1:5432
Error: password authentication failed for user "hockey_user"
```

**Diagnosis Steps:**
1. Verify PostgreSQL service is running
2. Check database credentials and connection string
3. Validate TimescaleDB extension installation
4. Confirm firewall and network accessibility

**Resolution:**
```bash
# Check PostgreSQL status
sudo service postgresql status
# OR for Docker
docker-compose ps postgres

# Restart PostgreSQL service
sudo service postgresql restart
# OR for Docker
docker-compose restart postgres

# Verify database exists and user has permissions
psql -h localhost -U postgres -d postgres
\l  # List databases
\du # List users

# Create missing database and user
CREATE DATABASE hockey_simulation_dev;
CREATE USER hockey_user WITH PASSWORD 'hockey_password';
GRANT ALL PRIVILEGES ON DATABASE hockey_simulation_dev TO hockey_user;

# Enable TimescaleDB extension
\c hockey_simulation_dev
CREATE EXTENSION IF NOT EXISTS timescaledb;
```

**Prevention:**
- Use Docker Compose for consistent database environments
- Implement health checks in database containers
- Document environment setup procedures clearly
- Maintain backup connection configurations

**Problem: Database Migration Failures**
```
Error: relation "players" already exists
Error: constraint "fk_player_team" does not exist
```

**Diagnosis Steps:**
1. Check migration status and history
2. Verify database schema state
3. Identify conflicting migrations or schema changes
4. Review migration file syntax and dependencies

**Resolution:**
```bash
# Check migration status
npm run db:migrate:status

# Rollback problematic migration
npm run db:migrate:down

# Reset database to clean state (development only)
npm run db:reset

# Run migrations individually for debugging
npx knex migrate:up 001_create_players_table.js
npx knex migrate:up 002_create_teams_table.js

# Force migration state if needed (use carefully)
npx knex migrate:unlock
```

**Prevention:**
- Always backup database before migrations
- Test migrations in development environment first
- Use transaction-wrapped migrations when possible
- Implement rollback procedures for all migrations

### Node.js and Dependency Issues

**Problem: Module Resolution Errors**
```
Error: Cannot resolve module '@/components/PlayerCard'
Error: Module not found: Can't resolve 'hockey-domain-types'
```

**Diagnosis Steps:**
1. Verify TypeScript path mapping configuration
2. Check package.json dependencies
3. Validate import statements and file paths
4. Confirm Node.js and npm versions

**Resolution:**
```bash
# Clear Node modules and reinstall
rm -rf node_modules package-lock.json
npm cache clean --force
npm install

# Check TypeScript configuration
npx tsc --showConfig

# Verify path mapping in tsconfig.json
{
  "compilerOptions": {
    "baseUrl": "./src",
    "paths": {
      "@/*": ["*"],
      "@components/*": ["components/*"],
      "@features/*": ["features/*"]
    }
  }
}

# Update import statements to use correct paths
// ❌ Incorrect
import { PlayerCard } from '../../../components/PlayerCard';

// ✅ Correct
import { PlayerCard } from '@/components/PlayerCard';
```

**Prevention:**
- Use consistent path mapping throughout project
- Document import conventions clearly
- Implement ESLint rules for import organization
- Regular dependency auditing and updates

**Problem: Version Compatibility Issues**
```
Error: Peer dependency warnings
Error: Node.js version 16.x not supported
```

**Diagnosis Steps:**
1. Check Node.js version requirements
2. Verify package compatibility matrix
3. Identify conflicting dependency versions
4. Review peer dependency warnings

**Resolution:**
```bash
# Check current versions
node --version
npm --version

# Install correct Node.js version (use nvm)
nvm install 18.19.0
nvm use 18.19.0

# Update package.json engines field
{
  "engines": {
    "node": ">=18.0.0",
    "npm": ">=9.0.0"
  }
}

# Resolve peer dependency issues
npm install --legacy-peer-deps
# OR
npm install --force

# Check for security vulnerabilities
npm audit
npm audit fix
```

**Prevention:**
- Pin Node.js versions in Docker and CI
- Regular dependency updates and security audits
- Use package-lock.json for consistent installations
- Document version requirements clearly

## Application Runtime Issues

### Authentication and Authorization Problems

**Problem: JWT Token Validation Failures**
```
Error: jwt malformed
Error: jwt expired
Error: invalid signature
```

**Diagnosis Steps:**
1. Verify JWT secret configuration
2. Check token expiration settings
3. Validate token format and structure
4. Confirm middleware configuration

**Resolution:**
```bash
# Check JWT configuration
echo $JWT_SECRET_KEY  # Should not be empty

# Verify token structure
node -e "
const jwt = require('jsonwebtoken');
const token = 'your_token_here';
try {
  const decoded = jwt.decode(token, { complete: true });
  console.log(JSON.stringify(decoded, null, 2));
} catch (err) {
  console.log('Token decode error:', err.message);
}
"

# Generate new JWT secret if needed
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"

# Update environment variable
export JWT_SECRET_KEY="new_secret_here"
```

**Token Refresh Issues:**
```javascript
// Implement proper token refresh logic
const refreshToken = async (expiredToken) => {
  try {
    const response = await fetch('/api/auth/refresh', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${expiredToken}`
      }
    });
    
    if (response.ok) {
      const { accessToken } = await response.json();
      localStorage.setItem('accessToken', accessToken);
      return accessToken;
    }
  } catch (error) {
    // Redirect to login
    window.location.href = '/login';
  }
};
```

**Prevention:**
- Use secure, randomly generated JWT secrets
- Implement proper token expiration handling
- Store tokens securely (httpOnly cookies for web)
- Regular token rotation and security audits

**Problem: Role-Based Access Control Failures**
```
Error: Insufficient permissions for operation
Error: User not associated with team
Error: League access denied
```

**Diagnosis Steps:**
1. Verify user role assignments in database
2. Check organizational hierarchy relationships
3. Validate permission inheritance logic
4. Confirm middleware authorization checks

**Resolution:**
```sql
-- Check user roles and assignments
SELECT u.email, ur.role_type, ur.league_id, ur.team_id 
FROM users u
JOIN user_roles ur ON u.id = ur.user_id
WHERE u.email = 'user@example.com';

-- Verify team assignment
SELECT t.name, t.league_id, ur.user_id
FROM teams t
JOIN user_roles ur ON t.id = ur.team_id
WHERE ur.user_id = 'user-uuid';

-- Check league membership
SELECT l.name, ur.role_type
FROM leagues l
JOIN user_roles ur ON l.id = ur.league_id
WHERE ur.user_id = 'user-uuid';
```

**Fix Permission Issues:**
```sql
-- Assign missing role
INSERT INTO user_roles (user_id, league_id, team_id, role_type)
VALUES ('user-uuid', 'league-uuid', 'team-uuid', 'general_manager');

-- Update existing role
UPDATE user_roles 
SET role_type = 'league_commissioner'
WHERE user_id = 'user-uuid' AND league_id = 'league-uuid';
```

**Prevention:**
- Implement comprehensive role validation
- Regular audit of user permissions
- Clear documentation of authorization hierarchy
- Automated testing of permission scenarios

### Hockey Domain Business Logic Issues

**Problem: Salary Cap Calculation Errors**
```
Error: Salary cap calculation mismatch
Error: Contract value exceeds individual limit
Error: LTIR relief calculation incorrect
```

**Diagnosis Steps:**
1. Validate individual contract calculations
2. Check LTIR and bonus calculations
3. Verify performance bonus handling
4. Compare with NHL salary cap rules

**Resolution:**
```javascript
// Debug salary cap calculation
const debugSalaryCap = (teamId) => {
  const team = getTeamById(teamId);
  const contracts = getActiveContracts(teamId);
  
  console.log('=== Salary Cap Debug ===');
  console.log('Team:', team.name);
  console.log('Salary Cap Limit:', team.salaryCap);
  
  let totalSalary = 0;
  let ltirRelief = 0;
  
  contracts.forEach(contract => {
    console.log(`Player: ${contract.playerName}`);
    console.log(`  Base Salary: $${contract.annualValue.toLocaleString()}`);
    console.log(`  LTIR Status: ${contract.ltirStatus}`);
    
    if (contract.ltirStatus) {
      ltirRelief += contract.annualValue;
    } else {
      totalSalary += contract.annualValue;
    }
  });
  
  console.log('Total Active Salary:', totalSalary);
  console.log('LTIR Relief:', ltirRelief);
  console.log('Effective Cap Usage:', totalSalary - ltirRelief);
  console.log('Remaining Cap Space:', team.salaryCap - (totalSalary - ltirRelief));
  console.log('===================');
};
```

**Common Calculation Fixes:**
```javascript
// Correct LTIR calculation
const calculateLTIRRelief = (contracts) => {
  return contracts
    .filter(contract => contract.ltirStatus && contract.isActive)
    .reduce((total, contract) => total + contract.annualValue, 0);
};

// Handle performance bonuses correctly
const calculateBonusImpact = (contracts) => {
  const currentYear = new Date().getFullYear();
  return contracts
    .filter(contract => contract.isActive)
    .reduce((total, contract) => {
      const applicableBonuses = contract.performanceBonuses
        .filter(bonus => bonus.season === currentYear && bonus.achieved);
      return total + applicableBonuses.reduce((sum, bonus) => sum + bonus.amount, 0);
    }, 0);
};
```

**Prevention:**
- Implement comprehensive salary cap test cases
- Regular validation against NHL rules
- Use precise decimal calculations (avoid floating point)
- Maintain audit trail for all financial calculations

**Problem: Game Simulation Statistical Anomalies**
```
Warning: Unrealistic goal scoring rate
Warning: Shot percentage outside normal range
Warning: Penalty frequency too high/low
```

**Diagnosis Steps:**
1. Compare statistics against NHL benchmarks
2. Check player rating distributions
3. Validate probability algorithms
4. Review game simulation parameters

**Resolution:**
```javascript
// Validate simulation statistics
const validateSimulationStats = (games) => {
  const totalGames = games.length;
  const totalGoals = games.reduce((sum, game) => sum + game.homeScore + game.awayScore, 0);
  const averageGoalsPerGame = totalGoals / totalGames;
  
  console.log('=== Simulation Validation ===');
  console.log(`Games Simulated: ${totalGames}`);
  console.log(`Average Goals/Game: ${averageGoalsPerGame.toFixed(2)} (NHL: ~6.2)`);
  
  // Check overtime frequency
  const overtimeGames = games.filter(game => game.overtime).length;
  const overtimeRate = (overtimeGames / totalGames) * 100;
  console.log(`Overtime Rate: ${overtimeRate.toFixed(1)}% (NHL: ~23%)`);
  
  // Check shutout frequency
  const shutouts = games.filter(game => game.homeScore === 0 || game.awayScore === 0).length;
  const shutoutRate = (shutouts / totalGames) * 100;
  console.log(`Shutout Rate: ${shutoutRate.toFixed(1)}% (NHL: ~8-12%)`);
  
  // Flag anomalies
  if (averageGoalsPerGame < 4.0 || averageGoalsPerGame > 8.0) {
    console.warn('⚠️ Goals per game outside normal range');
  }
  if (overtimeRate < 15 || overtimeRate > 35) {
    console.warn('⚠️ Overtime rate outside normal range');
  }
  if (shutoutRate < 5 || shutoutRate > 20) {
    console.warn('⚠️ Shutout rate outside normal range');
  }
};

// Adjust simulation parameters if needed
const adjustSimulationParameters = () => {
  return {
    baseScoringRate: 0.08,        // Goals per shot attempt
    overtimeProbability: 0.23,    // 23% of games go to overtime
    shootoutProbability: 0.33,    // 33% of OT games go to shootout
    penaltyFrequency: 0.15,       // Penalties per minute of game time
    injuryProbability: 0.002      // Injuries per player per game
  };
};
```

**Prevention:**
- Regular statistical validation against NHL data
- Implement configurable simulation parameters
- Comprehensive testing with large sample sizes
- Monitor and alert on statistical anomalies

## Performance and Scalability Issues

### Database Performance Problems

**Problem: Slow Query Performance**
```
Warning: Query execution time exceeded 5000ms
Error: Connection pool exhausted
Warning: High CPU usage on database
```

**Diagnosis Steps:**
1. Identify slow-running queries
2. Analyze query execution plans
3. Check index usage and effectiveness
4. Monitor connection pool status

**Resolution:**
```sql
-- Identify slow queries
SELECT query, calls, total_time, mean_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;

-- Analyze specific query performance
EXPLAIN ANALYZE SELECT p.*, t.name as team_name
FROM players p
LEFT JOIN teams t ON p.current_team_id = t.id
WHERE p.league_id = 'league-uuid'
AND p.position = 'center';

-- Add missing indexes
CREATE INDEX CONCURRENTLY idx_players_league_position 
ON players(league_id, position);

CREATE INDEX CONCURRENTLY idx_players_team_id 
ON players(current_team_id) 
WHERE current_team_id IS NOT NULL;

-- Optimize connection pooling
-- In application configuration
{
  "database": {
    "pool": {
      "min": 2,
      "max": 20,
      "acquireTimeoutMillis": 30000,
      "idleTimeoutMillis": 30000
    }
  }
}
```

**Query Optimization Examples:**
```sql
-- ❌ Inefficient query
SELECT * FROM players 
WHERE EXTRACT(year FROM birth_date) = 1995;

-- ✅ Optimized query with functional index
CREATE INDEX idx_players_birth_year 
ON players(EXTRACT(year FROM birth_date));

-- ❌ N+1 query problem
-- Multiple queries in application loop

-- ✅ Single optimized query with joins
SELECT 
  p.*,
  t.name as team_name,
  s.goals, s.assists, s.points
FROM players p
LEFT JOIN teams t ON p.current_team_id = t.id
LEFT JOIN player_statistics s ON p.id = s.player_id 
  AND s.season = 2024
WHERE p.league_id = $1;
```

**Prevention:**
- Regular query performance monitoring
- Proper indexing strategy for common queries
- Connection pool monitoring and tuning
- Database maintenance and statistics updates

**Problem: TimescaleDB Performance Issues**
```
Warning: Hypertable chunk size too large
Error: Compression policy not applied
Warning: Retention policy missing
```

**Diagnosis Steps:**
1. Check hypertable configuration
2. Verify compression policies
3. Monitor chunk sizes and distribution
4. Review retention policies

**Resolution:**
```sql
-- Check hypertable status
SELECT * FROM timescaledb_information.hypertables;

-- Verify chunk information
SELECT * FROM timescaledb_information.chunks 
WHERE hypertable_name = 'player_statistics';

-- Add compression policy
SELECT add_compression_policy('player_statistics', INTERVAL '30 days');

-- Add retention policy
SELECT add_retention_policy('player_statistics', INTERVAL '5 years');

-- Manually compress old chunks if needed
SELECT compress_chunk(chunk_name) 
FROM timescaledb_information.chunks 
WHERE hypertable_name = 'player_statistics' 
AND range_end < NOW() - INTERVAL '30 days';

-- Optimize chunk intervals
SELECT set_chunk_time_interval('player_statistics', INTERVAL '1 week');
```

**Prevention:**
- Proper TimescaleDB configuration from start
- Regular monitoring of chunk sizes
- Automated compression and retention policies
- Capacity planning for data growth

### Application Performance Issues

**Problem: High Memory Usage**
```
Warning: Memory usage exceeding 2GB
Error: JavaScript heap out of memory
Warning: React component re-render loops
```

**Diagnosis Steps:**
1. Profile memory usage and allocation
2. Identify memory leaks and retention issues
3. Check React component optimization
4. Monitor garbage collection patterns

**Resolution:**
```javascript
// Identify memory leaks with heap snapshots
const heapSnapshot = () => {
  if (process.env.NODE_ENV === 'development') {
    const used = process.memoryUsage();
    console.log('Memory Usage:');
    for (let key in used) {
      console.log(`${key}: ${Math.round(used[key] / 1024 / 1024 * 100) / 100} MB`);
    }
  }
};

// Optimize React components
const PlayerList = React.memo(({ players, onPlayerSelect }) => {
  // Use virtual scrolling for large lists
  const virtualizedPlayers = useMemo(() => {
    return players.slice(0, 100); // Limit visible items
  }, [players]);

  const handlePlayerSelect = useCallback((playerId) => {
    onPlayerSelect(playerId);
  }, [onPlayerSelect]);

  return (
    <div>
      {virtualizedPlayers.map(player => (
        <PlayerCard 
          key={player.id}
          player={player}
          onSelect={handlePlayerSelect}
        />
      ))}
    </div>
  );
});

// Implement proper cleanup
useEffect(() => {
  const subscription = websocketService.subscribe('game_updates', handleGameUpdate);
  
  return () => {
    subscription.unsubscribe(); // Prevent memory leaks
  };
}, []);

// Use WeakMap for temporary data storage
const playerCache = new WeakMap();
const getCachedPlayerData = (player) => {
  if (!playerCache.has(player)) {
    playerCache.set(player, calculatePlayerMetrics(player));
  }
  return playerCache.get(player);
};
```

**Memory Optimization:**
```javascript
// Implement data pagination
const usePlayersPagination = (leagueId, pageSize = 50) => {
  const [players, setPlayers] = useState([]);
  const [currentPage, setCurrentPage] = useState(0);
  const [hasMore, setHasMore] = useState(true);

  const loadMore = useCallback(async () => {
    const newPlayers = await api.getPlayers({
      leagueId,
      offset: currentPage * pageSize,
      limit: pageSize
    });

    if (newPlayers.length < pageSize) {
      setHasMore(false);
    }

    setPlayers(prev => [...prev, ...newPlayers]);
    setCurrentPage(prev => prev + 1);
  }, [leagueId, currentPage, pageSize]);

  return { players, loadMore, hasMore };
};

// Use proper data structure for large datasets
const usePlayerSearchIndex = (players) => {
  return useMemo(() => {
    const index = new Map();
    players.forEach(player => {
      // Index by multiple criteria for fast lookup
      index.set(player.id, player);
      index.set(`name-${player.lastName.toLowerCase()}`, player);
      index.set(`position-${player.position}`, player);
    });
    return index;
  }, [players]);
};
```

**Prevention:**
- Regular memory profiling and monitoring
- Implement virtual scrolling for large lists
- Use proper React optimization techniques
- Automated performance testing in CI/CD

### Real-Time Communication Issues

**Problem: WebSocket Connection Problems**
```
Error: WebSocket connection failed
Error: Message delivery timeout
Warning: High connection churn rate
```

**Diagnosis Steps:**
1. Check WebSocket server configuration
2. Verify client connection handling
3. Monitor message queue backlog
4. Test network connectivity and firewall rules

**Resolution:**
```javascript
// Implement robust WebSocket connection
class RobustWebSocketClient {
  constructor(url, options = {}) {
    this.url = url;
    this.options = {
      reconnectDelay: 1000,
      maxReconnectDelay: 30000,
      reconnectDelayGrowFactor: 1.3,
      maxReconnectAttempts: 10,
      ...options
    };
    this.reconnectAttempts = 0;
    this.reconnectDelay = this.options.reconnectDelay;
    this.shouldReconnect = true;
    this.messageQueue = [];
  }

  connect() {
    try {
      this.ws = new WebSocket(this.url);
      this.setupEventHandlers();
    } catch (error) {
      console.error('WebSocket connection failed:', error);
      this.scheduleReconnect();
    }
  }

  setupEventHandlers() {
    this.ws.onopen = () => {
      console.log('WebSocket connected');
      this.reconnectAttempts = 0;
      this.reconnectDelay = this.options.reconnectDelay;
      this.flushMessageQueue();
    };

    this.ws.onclose = (event) => {
      console.log('WebSocket disconnected:', event.code, event.reason);
      if (this.shouldReconnect && event.code !== 1000) {
        this.scheduleReconnect();
      }
    };

    this.ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };

    this.ws.onmessage = (event) => {
      try {
        const message = JSON.parse(event.data);
        this.handleMessage(message);
      } catch (error) {
        console.error('Failed to parse WebSocket message:', error);
      }
    };
  }

  scheduleReconnect() {
    if (this.reconnectAttempts >= this.options.maxReconnectAttempts) {
      console.error('Max reconnection attempts reached');
      return;
    }

    setTimeout(() => {
      console.log(`Reconnecting... Attempt ${this.reconnectAttempts + 1}`);
      this.reconnectAttempts++;
      this.connect();
    }, this.reconnectDelay);

    this.reconnectDelay = Math.min(
      this.reconnectDelay * this.options.reconnectDelayGrowFactor,
      this.options.maxReconnectDelay
    );
  }

  send(message) {
    if (this.ws && this.ws.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify(message));
    } else {
      this.messageQueue.push(message);
    }
  }

  flushMessageQueue() {
    while (this.messageQueue.length > 0) {
      const message = this.messageQueue.shift();
      this.send(message);
    }
  }

  disconnect() {
    this.shouldReconnect = false;
    if (this.ws) {
      this.ws.close(1000, 'Manual disconnect');
    }
  }
}
```

**Server-Side Connection Management:**
```javascript
// Implement connection pooling and cleanup
class WebSocketManager {
  constructor() {
    this.connections = new Map();
    this.gameSubscriptions = new Map();
    this.heartbeatInterval = 30000; // 30 seconds
    this.setupHeartbeat();
  }

  handleConnection(ws, userId) {
    this.connections.set(userId, ws);
    
    ws.on('close', () => {
      this.connections.delete(userId);
      this.cleanupSubscriptions(userId);
    });

    ws.on('pong', () => {
      ws.isAlive = true;
    });

    ws.on('message', (data) => {
      try {
        const message = JSON.parse(data);
        this.handleMessage(userId, message);
      } catch (error) {
        console.error(`Failed to parse message from user ${userId}:`, error);
      }
    });

    // Send initial connection confirmation
    ws.send(JSON.stringify({
      type: 'connection_established',
      userId,
      timestamp: new Date().toISOString()
    }));
  }

  setupHeartbeat() {
    setInterval(() => {
      this.connections.forEach((ws, userId) => {
        if (ws.isAlive === false) {
          console.log(`Terminating dead connection for user ${userId}`);
          ws.terminate();
          this.connections.delete(userId);
          this.cleanupSubscriptions(userId);
          return;
        }

        ws.isAlive = false;
        ws.ping();
      });
    }, this.heartbeatInterval);
  }

  cleanupSubscriptions(userId) {
    this.gameSubscriptions.forEach((subscribers, gameId) => {
      subscribers.delete(userId);
      if (subscribers.size === 0) {
        this.gameSubscriptions.delete(gameId);
      }
    });
  }
}
```

**Prevention:**
- Implement robust reconnection logic
- Use heartbeat/ping mechanisms
- Monitor connection metrics and alerts
- Proper connection cleanup and resource management

## Deployment and Infrastructure Issues

### Container and Orchestration Problems

**Problem: Docker Container Startup Failures**
```
Error: Container exits immediately
Error: Health check failing
Error: Port binding conflicts
```

**Diagnosis Steps:**
1. Check container logs and exit codes
2. Verify Dockerfile configuration
3. Test image build process
4. Confirm port availability and mapping

**Resolution:**
```bash
# Debug container startup
docker logs <container_id> --follow

# Check container health
docker inspect <container_id> | grep -A 5 "Health"

# Test image manually
docker run -it --rm hockey-sim:latest /bin/bash

# Check port conflicts
netstat -tulpn | grep :3000
lsof -i :3000

# Fix common Docker issues
# Dockerfile corrections
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
USER node
CMD ["npm", "start"]

# Environment variable issues
docker run -e NODE_ENV=production -e DATABASE_URL="..." hockey-sim:latest

# Volume mount problems
docker run -v $(pwd)/data:/app/data hockey-sim:latest
```

**Kubernetes Deployment Issues:**
```yaml
# Common Kubernetes fixes
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hockey-sim
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hockey-sim
  template:
    metadata:
      labels:
        app: hockey-sim
    spec:
      containers:
      - name: app
        image: hockey-sim:v1.0.0
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
```

**Prevention:**
- Implement proper health checks
- Use multi-stage Docker builds
- Regular security scanning of images
- Proper resource limits and monitoring

## Security and Compliance Issues

### Authentication and Session Security

**Problem: Security Vulnerabilities**
```
Warning: Weak JWT secret detected
Error: Session hijacking attempt
Warning: Brute force attack detected
```

**Diagnosis Steps:**
1. Audit authentication implementation
2. Check for security vulnerabilities
3. Review session management
4. Validate input sanitization

**Resolution:**
```javascript
// Implement secure JWT handling
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');
const rateLimit = require('express-rate-limit');

// Strong JWT secret generation
const generateSecureSecret = () => {
  return require('crypto').randomBytes(64).toString('hex');
};

// Secure password hashing
const hashPassword = async (password) => {
  const saltRounds = 12;
  return await bcrypt.hash(password, saltRounds);
};

// Rate limiting for auth endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 5, // 5 attempts per window
  message: 'Too many authentication attempts',
  standardHeaders: true,
  legacyHeaders: false,
});

// Secure session configuration
app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    maxAge: 24 * 60 * 60 * 1000, // 24 hours
    sameSite: 'strict'
  },
  store: new RedisStore({ client: redisClient })
}));

// Input validation and sanitization
const { body, validationResult } = require('express-validator');

const validatePlayerInput = [
  body('firstName').isLength({ min: 1, max: 50 }).trim().escape(),
  body('lastName').isLength({ min: 1, max: 50 }).trim().escape(),
  body('position').isIn(['left_wing', 'center', 'right_wing', 'left_defense', 'right_defense', 'goaltender']),
  body('overallRating').isInt({ min: 25, max: 99 }),
  (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    next();
  }
];
```

**Prevention:**
- Regular security audits and penetration testing
- Implement HTTPS everywhere
- Use secure session management
- Regular dependency security updates

## Monitoring and Alerting Setup

### Comprehensive System Monitoring

**Problem: Lack of Visibility into System Health**
```
Issue: No alerts when systems fail
Issue: Performance degradation goes unnoticed
Issue: Unable to troubleshoot user issues
```

**Monitoring Implementation:**
```javascript
// Application metrics collection
const prometheus = require('prom-client');

// Custom metrics for hockey simulation
const gameSimulationDuration = new prometheus.Histogram({
  name: 'game_simulation_duration_seconds',
  help: 'Duration of game simulation in seconds',
  labelNames: ['league_id', 'simulation_mode']
});

const activePlayers = new prometheus.Gauge({
  name: 'active_players_total',
  help: 'Total number of active players in system',
  labelNames: ['league_id']
});

const salaryCapViolations = new prometheus.Counter({
  name: 'salary_cap_violations_total',
  help: 'Total number of salary cap violations detected',
  labelNames: ['league_id', 'team_id']
});

// Health check endpoint
app.get('/health', async (req, res) => {
  try {
    // Check database connectivity
    await db.query('SELECT 1');
    
    // Check Redis connectivity
    await redis.ping();
    
    // Check external services
    const servicesHealthy = await checkExternalServices();
    
    res.status(200).json({
      status: 'healthy',
      timestamp: new Date().toISOString(),
      services: {
        database: 'healthy',
        redis: 'healthy',
        external: servicesHealthy ? 'healthy' : 'degraded'
      }
    });
  } catch (error) {
    res.status(503).json({
      status: 'unhealthy',
      error: error.message
    });
  }
});

// Error tracking and alerting
const winston = require('winston');
const { Loggly } = require('winston-loggly-bulk');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' }),
    new Loggly({
      token: process.env.LOGGLY_TOKEN,
      subdomain: process.env.LOGGLY_SUBDOMAIN,
      tags: ['hockey-sim', process.env.NODE_ENV],
      json: true
    })
  ]
});
```

**Alert Configuration:**
```yaml
# Prometheus alerting rules
groups:
- name: hockey-simulation-alerts
  rules:
  - alert: HighResponseTime
    expr: http_request_duration_seconds{quantile="0.95"} > 1
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High response time detected"
      description: "95th percentile response time is above 1 second"

  - alert: DatabaseConnectionFailure
    expr: up{job="postgres"} == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "Database connection failure"
      description: "PostgreSQL database is not responding"

  - alert: SalaryCapViolationSpike
    expr: increase(salary_cap_violations_total[5m]) > 10
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "High number of salary cap violations"
      description: "More than 10 salary cap violations in the last 5 minutes"
```

**Prevention:**
- Comprehensive monitoring from day one
- Proper alerting thresholds and escalation
- Regular review and tuning of monitoring
- Automated health checks and status pages

This comprehensive troubleshooting guide provides systematic problem resolution procedures while maintaining hockey domain accuracy and system reliability throughout all diagnostic and resolution activities.