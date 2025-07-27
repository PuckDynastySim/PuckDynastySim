# Configuration Guide - Hockey League Simulation System

This comprehensive configuration guide establishes the systematic setup and management procedures for all configurable parameters within the Hockey League Simulation System. The configuration framework enables flexible system behavior adaptation while maintaining security, performance, and operational reliability across diverse deployment environments and organizational requirements.

## Environment Configuration Framework

### Core Environment Variables

The system utilizes environment-based configuration management that enables deployment-specific customization while maintaining security through encrypted credential storage and appropriate access controls. Environment variables follow standardized naming conventions with clear indication of purpose and scope throughout system operations.

```bash
# Database Configuration
DATABASE_URL="postgresql://username:password@localhost:5432/hockey_simulation"
DATABASE_POOL_MIN=2
DATABASE_POOL_MAX=20
DATABASE_SSL_MODE="require"
DATABASE_TIMEOUT_MS=30000

# Redis Configuration  
REDIS_URL="redis://localhost:6379"
REDIS_PASSWORD="secure_redis_password"
REDIS_DB_INDEX=0
REDIS_KEY_PREFIX="hockey_sim:"
REDIS_SESSION_TTL=1800

# Application Server Configuration
PORT=3000
NODE_ENV="production"
APP_SECRET_KEY="highly_secure_application_secret_key"
JWT_SECRET_KEY="jwt_signing_secret_key"
JWT_ACCESS_TOKEN_TTL="15m"
JWT_REFRESH_TOKEN_TTL="30d"

# External Service Integration
STRIPE_SECRET_KEY="sk_live_stripe_secret_key"
STRIPE_WEBHOOK_SECRET="whsec_stripe_webhook_secret"
EMAIL_SERVICE_API_KEY="email_service_api_key"
EMAIL_FROM_ADDRESS="noreply@hockeysim.com"

# Security Configuration
CORS_ALLOWED_ORIGINS="https://app.hockeysim.com,https://admin.hockeysim.com"
RATE_LIMIT_REQUESTS_PER_MINUTE=100
SESSION_COOKIE_SECURE=true
SESSION_COOKIE_HTTP_ONLY=true
BCRYPT_SALT_ROUNDS=12

# Feature Flags
ENABLE_REAL_TIME_SIMULATION=true
ENABLE_ADVANCED_ANALYTICS=true
ENABLE_MULTI_FACTOR_AUTH=true
ENABLE_AUDIT_LOGGING=true

# Performance and Monitoring
LOG_LEVEL="info"
METRICS_ENABLED=true
PERFORMANCE_MONITORING_SAMPLE_RATE=0.1
ERROR_REPORTING_DSN="monitoring_service_dsn"
```

Environment variable validation ensures all required configuration parameters are present and properly formatted before application startup with comprehensive error reporting and configuration verification procedures that prevent deployment with incomplete or invalid settings.

### Development Environment Configuration

Development environment configuration provides comprehensive local development support including database seeding, hot reloading, and debugging capabilities with appropriate security relaxation and enhanced logging for effective development workflows and rapid iteration cycles.

```bash
# Development Environment (.env.development)
NODE_ENV="development"
DATABASE_URL="postgresql://dev_user:dev_password@localhost:5432/hockey_sim_dev"
REDIS_URL="redis://localhost:6379/1"

# Development Security (Relaxed for local development)
CORS_ALLOWED_ORIGINS="http://localhost:3000,http://localhost:3001"
SESSION_COOKIE_SECURE=false
RATE_LIMIT_REQUESTS_PER_MINUTE=1000

# Development Features
ENABLE_DATABASE_SEEDING=true
ENABLE_HOT_RELOAD=true
ENABLE_DEBUG_LOGGING=true
MOCK_EXTERNAL_SERVICES=true

# Development Tools
WEBPACK_ANALYZE_BUNDLE=false
SOURCE_MAPS_ENABLED=true
AUTO_GENERATE_TEST_DATA=true
```

Development configuration includes comprehensive test data generation and mock service integration that enables effective local development without external service dependencies while maintaining realistic development conditions and workflow efficiency.

### Production Environment Configuration

Production configuration emphasizes security, performance, and reliability through comprehensive security enforcement, performance optimization, and monitoring integration with appropriate backup procedures and disaster recovery preparation throughout operational deployment.

```bash
# Production Environment (.env.production)
NODE_ENV="production"
DATABASE_SSL_MODE="require"
SESSION_COOKIE_SECURE=true
FORCE_HTTPS=true

# Production Performance Optimization
DATABASE_POOL_MAX=50
REDIS_CONNECTION_POOL_SIZE=20
ENABLE_RESPONSE_COMPRESSION=true
STATIC_ASSET_CDN_URL="https://cdn.hockeysim.com"

# Production Security
ENABLE_SECURITY_HEADERS=true
ENABLE_RATE_LIMITING=true
ENABLE_IP_WHITELISTING=false
SECURITY_AUDIT_LOGGING=true

# Production Monitoring
METRICS_ENABLED=true
PERFORMANCE_MONITORING_ENABLED=true
ERROR_REPORTING_ENABLED=true
HEALTH_CHECK_ENDPOINT_ENABLED=true

# Production Backup and Recovery
DATABASE_BACKUP_ENABLED=true
DATABASE_BACKUP_SCHEDULE="0 2 * * *"
BACKUP_RETENTION_DAYS=30
DISASTER_RECOVERY_ENABLED=true
```

## League Configuration Parameters

### Core League Settings

League configuration enables comprehensive customization of hockey league operations including competitive parameters, financial constraints, and operational procedures with validation mechanisms that ensure configuration consistency and rule compliance throughout league establishment and management activities.

```typescript
interface LeagueConfiguration {
  // Financial Configuration
  salaryCapEnabled: boolean;
  salaryCapAmount: number; // in cents
  luxuryTaxEnabled: boolean;
  luxuryTaxThreshold: number;
  luxuryTaxRate: number;
  revenueSharing: {
    enabled: boolean;
    percentage: number;
    distributionMethod: 'equal' | 'performance_based' | 'market_size';
  };

  // Roster Configuration
  rosterSize: {
    minimum: number;
    maximum: number;
    dressed: number;
    healthy_scratches: number;
  };
  
  // Season Configuration
  seasonLength: number;
  playoffTeams: number;
  playoffFormat: 'single_elimination' | 'best_of_series' | 'round_robin';
  tradeDeadline: {
    enabled: boolean;
    daysBeforePlayoffs: number;
  };

  // Draft Configuration
  draftRounds: number;
  draftLottery: {
    enabled: boolean;
    numberOfDraws: number;
    eligibilityBasedOnRecord: boolean;
  };

  // Simulation Configuration
  simulationSpeed: 'real_time' | 'accelerated' | 'instant';
  statisticalRealism: 'arcade' | 'realistic' | 'simulation';
  injuryFrequency: number; // 0.0 to 1.0 scale
  
  // League Structure
  conferences: ConferenceConfiguration[];
  divisions: DivisionConfiguration[];
  interleaguePlay: boolean;
}
```

League configuration validation ensures all parameters maintain hockey authenticity and competitive balance through comprehensive rule checking and constraint verification that prevents invalid configuration combinations and maintains operational integrity throughout league establishment.

### Team Configuration Options

Team configuration provides comprehensive customization for individual team operations including financial parameters, facility specifications, and strategic preferences with appropriate validation and constraint enforcement that ensures realistic team operations within league parameters.

```typescript
interface TeamConfiguration {
  // Team Identity
  teamName: string;
  city: string;
  abbreviation: string;
  primaryColor: string;
  secondaryColor: string;
  logoUrl?: string;

  // Arena Configuration
  arena: {
    name: string;
    capacity: number;
    location: {
      city: string;
      state: string;
      country: string;
    };
    amenities: ArenaAmenity[];
  };

  // Financial Configuration
  budget: {
    totalBudget: number;
    salaryBudget: number;
    facilitiesBudget: number;
    trainingBudget: number;
    marketingBudget: number;
  };

  // Pricing Configuration
  ticketPricing: {
    lowerBowl: number;
    upperBowl: number;
    club: number;
    suites: number;
    seasonTicketDiscount: number;
  };

  // Operational Configuration
  aiControlled: boolean;
  aiDifficulty?: 'easy' | 'medium' | 'hard' | 'expert';
  strategicPreferences: {
    offensiveStyle: 'conservative' | 'balanced' | 'aggressive';
    defensiveStyle: 'defensive' | 'balanced' | 'offensive';
    specialTeamsEmphasis: 'power_play' | 'penalty_kill' | 'balanced';
  };
}
```

### Player Generation Configuration

Player generation configuration controls the systematic creation of fictional players with realistic attributes and demographic distributions that reflect hockey industry participation patterns while maintaining competitive balance and authentic career development trajectories.

```typescript
interface PlayerGenerationConfiguration {
  // Generation Parameters
  totalPlayersToGenerate: number;
  ageDistribution: {
    junior: { min: 16, max: 21, weight: 0.3 };
    professional: { min: 18, max: 35, weight: 0.6 };
    veteran: { min: 30, max: 42, weight: 0.1 };
  };

  // Position Distribution
  positionDistribution: {
    leftWing: number;
    center: number;
    rightWing: number;
    leftDefense: number;
    rightDefense: number;
    goaltender: number;
  };

  // Skill Distribution
  skillDistribution: {
    elite: { ratingRange: [90, 99], percentage: 0.05 };
    star: { ratingRange: [80, 89], percentage: 0.15 };
    topSix: { ratingRange: [70, 79], percentage: 0.25 };
    middle: { ratingRange: [60, 69], percentage: 0.35 };
    depth: { ratingRange: [50, 59], percentage: 0.15 };
    fringe: { ratingRange: [25, 49], percentage: 0.05 };
  };

  // Nationality Distribution (based on NHL demographics)
  nationalityDistribution: {
    canada: 0.45;
    usa: 0.25;
    sweden: 0.08;
    finland: 0.05;
    russia: 0.04;
    czech: 0.03;
    slovakia: 0.02;
    other: 0.08;
  };

  // Career Development
  developmentCurves: {
    earlyPeak: { peakAge: 24, declineRate: 0.02 };
    normalPeak: { peakAge: 27, declineRate: 0.015 };
    latePeak: { peakAge: 30, declineRate: 0.025 };
  };
}
```

## Database Configuration Management

### Connection and Performance Configuration

Database configuration encompasses connection management, performance optimization, and reliability enhancement through comprehensive parameter tuning and monitoring integration that ensures optimal database operation under varying load conditions and operational requirements.

```typescript
interface DatabaseConfiguration {
  // Connection Configuration
  connection: {
    host: string;
    port: number;
    database: string;
    username: string;
    password: string;
    ssl: {
      enabled: boolean;
      rejectUnauthorized: boolean;
      certificatePath?: string;
    };
  };

  // Connection Pooling
  pool: {
    min: number;
    max: number;
    idleTimeoutMillis: number;
    connectionTimeoutMillis: number;
    acquireTimeoutMillis: number;
  };

  // Performance Configuration
  performance: {
    statementTimeout: number;
    queryTimeout: number;
    lockTimeout: number;
    maxConnections: number;
    sharedBuffers: string;
    effectiveCacheSize: string;
    workMem: string;
  };

  // TimescaleDB Configuration
  timescaledb: {
    enabled: boolean;
    chunkTimeInterval: string;
    compressionEnabled: boolean;
    retentionPolicy: {
      statisticalData: string;
      gameEvents: string;
      auditLogs: string;
    };
  };

  // Backup Configuration
  backup: {
    enabled: boolean;
    schedule: string;
    retentionDays: number;
    compressionEnabled: boolean;
    encryptionEnabled: boolean;
    remoteStorage: {
      enabled: boolean;
      provider: 'aws_s3' | 'google_cloud' | 'azure_blob';
      bucket: string;
      region: string;
    };
  };
}
```

### Index and Query Optimization Configuration

Database optimization configuration includes comprehensive index management, query performance tuning, and analytical query optimization with appropriate monitoring and maintenance automation that ensures sustained database performance throughout system growth and operational scaling.

```sql
-- Performance Configuration Parameters
-- Shared Memory Configuration
shared_buffers = '1GB'                    -- 25% of system RAM
effective_cache_size = '3GB'              -- 75% of system RAM
work_mem = '50MB'                         -- Per-query working memory

-- Query Performance
random_page_cost = 1.1                    -- SSD optimization
seq_page_cost = 1.0                       -- Sequential scan cost
cpu_tuple_cost = 0.01                     -- CPU processing cost
cpu_index_tuple_cost = 0.005              -- Index scan cost

-- Write Performance
checkpoint_completion_target = 0.9        -- Checkpoint distribution
wal_buffers = '32MB'                      -- WAL buffer size
max_wal_size = '2GB'                      -- Maximum WAL size

-- TimescaleDB Specific Configuration
timescaledb.max_background_workers = 8    -- Background worker processes
timescaledb.last_updated_optimization = 'on'  -- Query optimization
```

## Security Configuration Framework

### Authentication and Authorization Configuration

Security configuration implements comprehensive protection mechanisms including authentication policies, authorization frameworks, and audit logging with appropriate encryption and access control measures that ensure system security while maintaining operational efficiency and user experience quality.

```typescript
interface SecurityConfiguration {
  // Authentication Configuration
  authentication: {
    jwtSecret: string;
    accessTokenTTL: string;
    refreshTokenTTL: string;
    passwordPolicy: {
      minLength: number;
      requireUppercase: boolean;
      requireLowercase: boolean;
      requireNumbers: boolean;
      requireSpecialChars: boolean;
      preventReuse: number;
    };
    accountLockout: {
      enabled: boolean;
      maxAttempts: number;
      lockoutDuration: string;
      progressiveLockout: boolean;
    };
  };

  // Multi-Factor Authentication
  mfa: {
    enabled: boolean;
    required: boolean;
    backupCodes: {
      enabled: boolean;
      count: number;
      length: number;
    };
    totpSettings: {
      issuer: string;
      algorithm: 'SHA1' | 'SHA256' | 'SHA512';
      digits: 6 | 8;
      window: number;
    };
  };

  // Session Management
  session: {
    cookieName: string;
    secure: boolean;
    httpOnly: boolean;
    sameSite: 'strict' | 'lax' | 'none';
    maxAge: number;
    rolling: boolean;
  };

  // Rate Limiting
  rateLimiting: {
    enabled: boolean;
    windowMs: number;
    maxRequests: number;
    skipSuccessfulRequests: boolean;
    skipFailedRequests: boolean;
    keyGenerator: (req: Request) => string;
  };

  // CORS Configuration
  cors: {
    origin: string[] | boolean;
    credentials: boolean;
    optionsSuccessStatus: number;
    allowedHeaders: string[];
    exposedHeaders: string[];
  };
}
```

### Encryption and Data Protection Configuration

Data protection configuration includes comprehensive encryption implementation, privacy controls, and compliance measures with appropriate key management and access controls that ensure sensitive information protection throughout storage, transmission, and processing operations.

```typescript
interface EncryptionConfiguration {
  // Encryption at Rest
  databaseEncryption: {
    enabled: boolean;
    algorithm: 'AES-256-GCM';
    keyRotationInterval: string;
    keyDerivationFunction: 'PBKDF2' | 'Argon2';
  };

  // Encryption in Transit
  tlsConfiguration: {
    minVersion: '1.2' | '1.3';
    cipherSuites: string[];
    certificateValidation: boolean;
    hstsEnabled: boolean;
    hstsMaxAge: number;
  };

  // Field-Level Encryption
  fieldEncryption: {
    personalData: {
      enabled: boolean;
      fields: string[];
      algorithm: 'AES-256-GCM';
    };
    financialData: {
      enabled: boolean;
      fields: string[];
      algorithm: 'AES-256-GCM';
    };
  };

  // Key Management
  keyManagement: {
    provider: 'local' | 'aws_kms' | 'azure_keyvault' | 'hashicorp_vault';
    rotationSchedule: string;
    backupEnabled: boolean;
    accessLogging: boolean;
  };
}
```

## Performance Configuration and Optimization

### Caching Strategy Configuration

Performance configuration includes comprehensive caching strategies, resource optimization, and scalability parameters with appropriate monitoring and adjustment mechanisms that ensure optimal system responsiveness throughout varying load conditions and operational requirements.

```typescript
interface CachingConfiguration {
  // Redis Caching
  redis: {
    enabled: boolean;
    cluster: boolean;
    maxMemoryPolicy: 'allkeys-lru' | 'volatile-lru' | 'allkeys-lfu';
    keyExpiration: {
      sessions: number;
      playerStatistics: number;
      gameResults: number;
      leagueStandings: number;
    };
  };

  // Application-Level Caching
  applicationCache: {
    enabled: boolean;
    maxSize: number;
    ttl: number;
    strategy: 'lru' | 'lfu' | 'fifo';
  };

  // CDN Configuration
  cdn: {
    enabled: boolean;
    provider: 'cloudflare' | 'aws_cloudfront' | 'azure_cdn';
    cacheHeaders: {
      staticAssets: string;
      apiResponses: string;
      images: string;
    };
  };

  // Browser Caching
  browserCache: {
    enabled: boolean;
    maxAge: {
      staticAssets: number;
      apiResponses: number;
      images: number;
    };
  };
}
```

### Monitoring and Alerting Configuration

Monitoring configuration provides comprehensive system visibility including performance tracking, error monitoring, and capacity planning with appropriate alerting and escalation procedures that enable proactive system management and issue resolution throughout operational deployment.

```typescript
interface MonitoringConfiguration {
  // Application Performance Monitoring
  apm: {
    enabled: boolean;
    provider: 'newrelic' | 'datadog' | 'dynatrace';
    sampleRate: number;
    transactionTracing: boolean;
    errorTracking: boolean;
  };

  // System Metrics
  metrics: {
    enabled: boolean;
    interval: number;
    retention: string;
    customMetrics: {
      gameSimulationPerformance: boolean;
      userActivityTracking: boolean;
      financialTransactionMonitoring: boolean;
    };
  };

  // Alerting Configuration
  alerting: {
    enabled: boolean;
    channels: {
      email: boolean;
      slack: boolean;
      pagerduty: boolean;
    };
    thresholds: {
      responseTime: number;
      errorRate: number;
      memoryUsage: number;
      cpuUsage: number;
    };
  };

  // Log Management
  logging: {
    level: 'debug' | 'info' | 'warn' | 'error';
    format: 'json' | 'text';
    retention: string;
    aggregation: {
      enabled: boolean;
      provider: 'elasticsearch' | 'splunk' | 'datadog';
    };
  };
}
```

This comprehensive configuration framework provides systematic control over all system parameters while maintaining security, performance, and operational reliability throughout diverse deployment environments and organizational requirements.