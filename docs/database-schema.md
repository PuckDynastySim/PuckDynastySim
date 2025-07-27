# Database Schema - Hockey League Simulation System

This comprehensive database schema documentation defines the data structures, relationships, and constraints that support the Hockey League Simulation System. The schema design accommodates multi-tenant operations while optimizing performance for complex hockey league management and statistical tracking requirements.

## Schema Organization and Multi-Tenancy

### Organizational Hierarchy Structure

The database schema implements a hierarchical organizational structure that aligns with hockey league management patterns. Federations represent the highest organizational level and can contain multiple leagues, enabling management of complex hockey organizations such as national governing bodies that oversee multiple competitive divisions.

Leagues function as the primary operational unit within the system and contain teams, players, schedules, and all associated hockey operations. This design enables complete isolation between different hockey organizations while supporting shared infrastructure for cost-effective system operation.

The tenant isolation strategy employs row-level security policies that automatically filter data based on user authentication context and organizational assignments. Security policies operate transparently to application code while ensuring complete data separation between different hockey organizations.

### Core Entity Relationships

```sql
-- Primary organizational entities
CREATE TABLE federations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    country_code CHAR(2),
    founded_date DATE,
    contact_info JSONB,
    settings JSONB,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE leagues (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    federation_id UUID REFERENCES federations(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    league_type VARCHAR(50) NOT NULL, -- 'professional', 'farm', 'junior'
    salary_cap_enabled BOOLEAN DEFAULT true,
    salary_cap_amount BIGINT, -- stored in cents
    season_length INTEGER DEFAULT 82,
    playoff_teams INTEGER DEFAULT 16,
    current_season INTEGER,
    current_phase VARCHAR(50) DEFAULT 'offseason',
    settings JSONB,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE conferences (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    league_id UUID REFERENCES leagues(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    sort_order INTEGER,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE divisions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    conference_id UUID REFERENCES conferences(id) ON DELETE CASCADE,
    name VARCHAR(255) NOT NULL,
    sort_order INTEGER,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

The organizational structure provides flexible configuration capabilities while maintaining referential integrity through cascading delete operations that ensure data consistency when organizational restructuring occurs.

## User Management and Authentication

### User Account Structure

User accounts support multiple role assignments across different organizational contexts, enabling individuals to participate in multiple leagues with different responsibilities. The user profile design accommodates both personal information and professional hockey management credentials.

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    avatar_url VARCHAR(500),
    timezone VARCHAR(50) DEFAULT 'UTC',
    locale VARCHAR(10) DEFAULT 'en-US',
    preferences JSONB DEFAULT '{}',
    email_verified BOOLEAN DEFAULT false,
    email_verified_at TIMESTAMP WITH TIME ZONE,
    last_login_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE user_roles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    federation_id UUID REFERENCES federations(id) ON DELETE CASCADE,
    league_id UUID REFERENCES leagues(id) ON DELETE CASCADE,
    team_id UUID REFERENCES teams(id) ON DELETE CASCADE,
    role_type VARCHAR(50) NOT NULL, -- 'federation_admin', 'league_commissioner', 'general_manager', 'player'
    permissions JSONB DEFAULT '{}',
    active BOOLEAN DEFAULT true,
    assigned_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    assigned_by UUID REFERENCES users(id)
);
```

Role assignment table design enables granular permission management while maintaining clear audit trails for all authorization changes. Permission inheritance operates through the organizational hierarchy while allowing specific override capabilities for complex management scenarios.

### Session and Security Management

Session management utilizes secure token storage with appropriate expiration policies and refresh mechanisms that balance security requirements with user experience considerations. The session design supports concurrent access from multiple devices while maintaining comprehensive audit capabilities.

```sql
CREATE TABLE user_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    token_hash VARCHAR(255) NOT NULL,
    device_info JSONB,
    ip_address INET,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
    last_activity TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50),
    resource_id UUID,
    details JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

Audit logging captures comprehensive activity tracking for security monitoring and compliance reporting while maintaining efficient query performance through appropriate indexing strategies.

## Team and Player Management

### Team Organization Structure

Team entities support comprehensive organizational information including financial parameters, facility details, and strategic configurations that influence simulation outcomes and management capabilities.

```sql
CREATE TABLE teams (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    league_id UUID REFERENCES leagues(id) ON DELETE CASCADE,
    division_id UUID REFERENCES divisions(id),
    parent_team_id UUID REFERENCES teams(id), -- for farm team relationships
    name VARCHAR(255) NOT NULL,
    city VARCHAR(255) NOT NULL,
    abbreviation VARCHAR(10) NOT NULL,
    logo_url VARCHAR(500),
    primary_color VARCHAR(7), -- hex color code
    secondary_color VARCHAR(7),
    founded_year INTEGER,
    arena_name VARCHAR(255),
    arena_capacity INTEGER,
    current_salary_used BIGINT DEFAULT 0, -- stored in cents
    ai_controlled BOOLEAN DEFAULT false,
    settings JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE team_financial_records (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    team_id UUID REFERENCES teams(id) ON DELETE CASCADE,
    season INTEGER NOT NULL,
    revenue_tickets BIGINT DEFAULT 0,
    revenue_merchandise BIGINT DEFAULT 0,
    revenue_sponsorship BIGINT DEFAULT 0,
    expense_salaries BIGINT DEFAULT 0,
    expense_facilities BIGINT DEFAULT 0,
    expense_training BIGINT DEFAULT 0,
    budget_allocated BIGINT,
    budget_remaining BIGINT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

Financial tracking enables sophisticated budget management and revenue optimization while maintaining historical records for analytical purposes and financial planning activities.

### Player Attributes and Statistics

Player entities incorporate comprehensive attribute systems that support realistic simulation mechanics while accommodating different player types and positional requirements. The rating system design enables flexible algorithm implementations while maintaining consistent interfaces.

```sql
CREATE TABLE players (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    league_id UUID REFERENCES leagues(id) ON DELETE CASCADE,
    current_team_id UUID REFERENCES teams(id),
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    position VARCHAR(20) NOT NULL, -- 'left_wing', 'center', 'right_wing', 'left_defense', 'right_defense', 'goaltender'
    jersey_number INTEGER,
    birth_date DATE NOT NULL,
    nationality VARCHAR(3) NOT NULL, -- ISO 3166-1 alpha-3
    height_cm INTEGER,
    weight_kg INTEGER,
    handedness VARCHAR(10), -- 'left', 'right'
    draft_year INTEGER,
    draft_round INTEGER,
    draft_position INTEGER,
    draft_team_id UUID REFERENCES teams(id),
    overall_rating INTEGER CHECK (overall_rating >= 25 AND overall_rating <= 99),
    potential_rating INTEGER CHECK (potential_rating >= 25 AND potential_rating <= 99),
    ratings JSONB NOT NULL, -- position-specific ratings
    contract_details JSONB,
    injury_status JSONB,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Player ratings stored as JSONB for flexibility
-- Skater ratings: discipline, injury, fatigue, passing, shooting, defense, puck_control, checking, fighting, poise
-- Goaltender ratings: discipline, injury, fatigue, poise, movement, rebound_control, vision, aggressiveness, puck_control, flexibility

CREATE TABLE player_statistics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    player_id UUID REFERENCES players(id) ON DELETE CASCADE,
    team_id UUID REFERENCES teams(id) ON DELETE CASCADE,
    season INTEGER NOT NULL,
    league_id UUID REFERENCES leagues(id) ON DELETE CASCADE,
    games_played INTEGER DEFAULT 0,
    goals INTEGER DEFAULT 0,
    assists INTEGER DEFAULT 0,
    points INTEGER DEFAULT 0,
    plus_minus INTEGER DEFAULT 0,
    penalty_minutes INTEGER DEFAULT 0,
    shots INTEGER DEFAULT 0,
    shooting_percentage DECIMAL(5,2),
    time_on_ice_total INTEGER DEFAULT 0, -- total seconds
    -- Goaltender specific statistics
    wins INTEGER DEFAULT 0,
    losses INTEGER DEFAULT 0,
    overtime_losses INTEGER DEFAULT 0,
    saves INTEGER DEFAULT 0,
    shots_against INTEGER DEFAULT 0,
    save_percentage DECIMAL(5,3),
    goals_against_average DECIMAL(4,2),
    shutouts INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(player_id, season, league_id)
);
```

Statistical tracking utilizes TimescaleDB capabilities for efficient time-series data management with automated aggregation procedures that support real-time dashboard updates and comprehensive analytical queries.

## Game and Simulation Data

### Schedule and Game Management

Schedule generation and game management require sophisticated data structures that support complex tournament formats while maintaining efficient query performance for real-time simulation operations.

```sql
CREATE TABLE seasons (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    league_id UUID REFERENCES leagues(id) ON DELETE CASCADE,
    year INTEGER NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    regular_season_games INTEGER,
    playoff_format JSONB,
    status VARCHAR(50) DEFAULT 'upcoming', -- 'upcoming', 'active', 'completed'
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE games (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    season_id UUID REFERENCES seasons(id) ON DELETE CASCADE,
    home_team_id UUID REFERENCES teams(id) ON DELETE CASCADE,
    away_team_id UUID REFERENCES teams(id) ON DELETE CASCADE,
    scheduled_date TIMESTAMP WITH TIME ZONE NOT NULL,
    game_type VARCHAR(50) NOT NULL, -- 'regular', 'playoff', 'exhibition'
    status VARCHAR(50) DEFAULT 'scheduled', -- 'scheduled', 'in_progress', 'completed', 'postponed'
    period INTEGER DEFAULT 0,
    time_remaining VARCHAR(10), -- MM:SS format
    home_score INTEGER DEFAULT 0,
    away_score INTEGER DEFAULT 0,
    overtime BOOLEAN DEFAULT false,
    shootout BOOLEAN DEFAULT false,
    attendance INTEGER,
    venue VARCHAR(255),
    simulation_data JSONB, -- detailed simulation state
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE game_events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    game_id UUID REFERENCES games(id) ON DELETE CASCADE,
    period INTEGER NOT NULL,
    time_remaining VARCHAR(10) NOT NULL,
    event_type VARCHAR(50) NOT NULL, -- 'goal', 'penalty', 'shot', 'save', 'hit', 'faceoff'
    team_id UUID REFERENCES teams(id),
    primary_player_id UUID REFERENCES players(id),
    secondary_player_id UUID REFERENCES players(id), -- for assists
    description TEXT,
    coordinates JSONB, -- x, y coordinates on ice surface
    details JSONB, -- event-specific additional data
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

Game event tracking provides comprehensive play-by-play data that supports detailed statistical analysis while enabling realistic game recreation and highlight generation capabilities.

## Performance Optimization and Indexing

### Strategic Index Design

Database performance optimization utilizes comprehensive indexing strategies that balance query performance with write operation efficiency. Index design considers common query patterns including statistical lookups, roster management operations, and real-time game data access.

```sql
-- Core organizational indexes
CREATE INDEX idx_leagues_federation_id ON leagues(federation_id);
CREATE INDEX idx_teams_league_id ON teams(league_id);
CREATE INDEX idx_teams_division_id ON teams(division_id);

-- User and role management indexes
CREATE INDEX idx_user_roles_user_id ON user_roles(user_id);
CREATE INDEX idx_user_roles_league_id ON user_roles(league_id);
CREATE INDEX idx_user_roles_team_id ON user_roles(team_id);

-- Player and statistics indexes
CREATE INDEX idx_players_team_id ON players(current_team_id);
CREATE INDEX idx_players_league_id ON players(league_id);
CREATE INDEX idx_players_position ON players(position);
CREATE INDEX idx_player_statistics_season ON player_statistics(season);
CREATE INDEX idx_player_statistics_player_season ON player_statistics(player_id, season);

-- Game and simulation indexes
CREATE INDEX idx_games_season_id ON games(season_id);
CREATE INDEX idx_games_scheduled_date ON games(scheduled_date);
CREATE INDEX idx_games_teams ON games(home_team_id, away_team_id);
CREATE INDEX idx_game_events_game_id ON game_events(game_id);
CREATE INDEX idx_game_events_period_time ON game_events(period, time_remaining);

-- TimescaleDB hypertable for time-series data
SELECT create_hypertable('player_statistics', 'created_at');
SELECT create_hypertable('game_events', 'created_at');
```

Hypertable configurations optimize time-series data storage and query performance for historical statistics while maintaining real-time access capabilities for current season operations.

### Data Integrity and Constraints

Comprehensive constraint definitions ensure data consistency and enforce hockey-specific business rules at the database level. Constraint validation prevents invalid data combinations while maintaining referential integrity across complex entity relationships.

```sql
-- Hockey-specific constraints
ALTER TABLE players ADD CONSTRAINT valid_jersey_number 
    CHECK (jersey_number IS NULL OR (jersey_number >= 1 AND jersey_number <= 99));

ALTER TABLE players ADD CONSTRAINT valid_position 
    CHECK (position IN ('left_wing', 'center', 'right_wing', 'left_defense', 'right_defense', 'goaltender'));

ALTER TABLE teams ADD CONSTRAINT unique_team_jersey_numbers 
    EXCLUDE (league_id WITH =, jersey_number WITH =) WHERE (jersey_number IS NOT NULL);

-- Financial constraints
ALTER TABLE teams ADD CONSTRAINT non_negative_salary 
    CHECK (current_salary_used >= 0);

-- Game constraints
ALTER TABLE games ADD CONSTRAINT valid_scores 
    CHECK (home_score >= 0 AND away_score >= 0);

ALTER TABLE games ADD CONSTRAINT different_teams 
    CHECK (home_team_id != away_team_id);
```

This comprehensive database schema provides the foundation for reliable hockey league simulation while maintaining optimal performance characteristics and data integrity requirements across all system operations.