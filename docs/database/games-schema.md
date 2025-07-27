# Games Schema Documentation - Hockey League Simulation System

This comprehensive games schema documentation defines the database structures that support game management, simulation, and statistical tracking within the Hockey League Simulation System. The schema encompasses game scheduling, real-time simulation data, event tracking, and comprehensive statistical compilation while maintaining hockey domain accuracy and supporting scalable multi-tenant operations.

## Core Game Management Tables

### Games Table Structure

The games table serves as the central entity for all hockey game management, containing scheduling information, game state, scoring details, and simulation metadata. This table supports both real-time game simulation and historical game tracking while maintaining referential integrity with league and team structures.

```sql
CREATE TABLE games (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    season_id UUID NOT NULL REFERENCES seasons(id) ON DELETE CASCADE,
    home_team_id UUID NOT NULL REFERENCES teams(id) ON DELETE CASCADE,
    away_team_id UUID NOT NULL REFERENCES teams(id) ON DELETE CASCADE,
    
    -- Scheduling Information
    scheduled_date TIMESTAMP WITH TIME ZONE NOT NULL,
    game_type VARCHAR(20) NOT NULL DEFAULT 'regular',
    game_number INTEGER, -- Sequential game number within season
    venue_id UUID REFERENCES venues(id),
    
    -- Game Status and Progress
    status VARCHAR(20) NOT NULL DEFAULT 'scheduled',
    current_period INTEGER DEFAULT 0,
    period_time_remaining VARCHAR(10), -- MM:SS format
    intermission_time_remaining VARCHAR(10),
    
    -- Scoring Information
    home_score INTEGER DEFAULT 0,
    away_score INTEGER DEFAULT 0,
    home_shots INTEGER DEFAULT 0,
    away_shots INTEGER DEFAULT 0,
    
    -- Game Outcome Details
    overtime BOOLEAN DEFAULT false,
    shootout BOOLEAN DEFAULT false,
    winning_team_id UUID REFERENCES teams(id),
    winning_goal_id UUID REFERENCES game_events(id),
    
    -- Attendance and Environment
    attendance INTEGER,
    capacity INTEGER,
    weather_conditions JSONB, -- For outdoor games
    
    -- Simulation Metadata
    simulation_speed VARCHAR(20) DEFAULT 'real_time',
    simulation_seed BIGINT, -- For reproducible simulations
    simulation_completed_at TIMESTAMP WITH TIME ZONE,
    
    -- Officials and Staff
    officials JSONB, -- Referee and linesmen assignments
    
    -- Broadcast and Media
    broadcast_info JSONB,
    game_recap TEXT,
    
    -- Timestamps
    started_at TIMESTAMP WITH TIME ZONE,
    completed_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    -- Constraints
    CONSTRAINT valid_game_type CHECK (game_type IN ('preseason', 'regular', 'playoff', 'exhibition')),
    CONSTRAINT valid_status CHECK (status IN ('scheduled', 'delayed', 'in_progress', 'intermission', 'completed', 'postponed', 'cancelled')),
    CONSTRAINT valid_period CHECK (current_period >= 0 AND current_period <= 10), -- Includes potential multiple overtimes
    CONSTRAINT valid_scores CHECK (home_score >= 0 AND away_score >= 0),
    CONSTRAINT different_teams CHECK (home_team_id != away_team_id),
    CONSTRAINT valid_attendance CHECK (attendance IS NULL OR (attendance >= 0 AND attendance <= capacity))
);

-- Performance Indexes
CREATE INDEX idx_games_season_id ON games(season_id);
CREATE INDEX idx_games_teams ON games(home_team_id, away_team_id);
CREATE INDEX idx_games_scheduled_date ON games(scheduled_date);
CREATE INDEX idx_games_status ON games(status);
CREATE INDEX idx_games_type_status ON games(game_type, status);
CREATE INDEX idx_games_completed_at ON games(completed_at) WHERE completed_at IS NOT NULL;

-- Multi-tenant security
ALTER TABLE games ENABLE ROW LEVEL SECURITY;
CREATE POLICY games_tenant_isolation ON games
    USING (season_id IN (
        SELECT s.id FROM seasons s 
        JOIN leagues l ON s.league_id = l.id 
        WHERE l.federation_id = current_setting('app.current_federation_id')::UUID
    ));
```

### Seasons and League Structure

The seasons table provides temporal organization for games while supporting multiple concurrent seasons across different leagues and competitive levels within the multi-tenant architecture.

```sql
CREATE TABLE seasons (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    league_id UUID NOT NULL REFERENCES leagues(id) ON DELETE CASCADE,
    
    -- Season Identification
    name VARCHAR(100) NOT NULL, -- "2023-24 Regular Season"
    season_type VARCHAR(20) NOT NULL DEFAULT 'regular',
    year_start INTEGER NOT NULL,
    year_end INTEGER NOT NULL,
    
    -- Season Schedule
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    playoff_start_date DATE,
    
    -- Season Configuration
    regular_season_games INTEGER DEFAULT 82,
    playoff_format JSONB,
    standings_format JSONB,
    
    -- Season Status
    status VARCHAR(20) NOT NULL DEFAULT 'upcoming',
    current_phase VARCHAR(20) DEFAULT 'preseason',
    
    -- Statistics and Records
    total_games_scheduled INTEGER DEFAULT 0,
    total_games_completed INTEGER DEFAULT 0,
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT valid_season_type CHECK (season_type IN ('preseason', 'regular', 'playoff', 'exhibition')),
    CONSTRAINT valid_season_status CHECK (status IN ('upcoming', 'active', 'completed', 'cancelled')),
    CONSTRAINT valid_season_phase CHECK (current_phase IN ('preseason', 'regular_season', 'playoff', 'offseason')),
    CONSTRAINT valid_year_range CHECK (year_end >= year_start AND year_end - year_start <= 1),
    CONSTRAINT valid_date_range CHECK (end_date >= start_date)
);

CREATE INDEX idx_seasons_league_id ON seasons(league_id);
CREATE INDEX idx_seasons_year ON seasons(year_start, year_end);
CREATE INDEX idx_seasons_status ON seasons(status);
CREATE UNIQUE INDEX idx_seasons_league_year ON seasons(league_id, year_start, season_type);
```

### Venues and Game Locations

The venues table manages hockey arena information including capacity, location details, and facility specifications that affect game atmosphere and attendance calculations.

```sql
CREATE TABLE venues (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    league_id UUID NOT NULL REFERENCES leagues(id) ON DELETE CASCADE,
    
    -- Venue Identification
    name VARCHAR(255) NOT NULL,
    nickname VARCHAR(100), -- "The Garden", "The Joe"
    
    -- Location Information
    city VARCHAR(100) NOT NULL,
    state_province VARCHAR(100),
    country VARCHAR(3) NOT NULL, -- ISO 3166-1 alpha-3
    address TEXT,
    postal_code VARCHAR(20),
    timezone VARCHAR(50) DEFAULT 'UTC',
    
    -- Facility Details
    capacity INTEGER NOT NULL,
    surface_type VARCHAR(20) DEFAULT 'ice',
    rink_dimensions JSONB, -- Length, width, corner radius
    
    -- Facility Features
    amenities TEXT[],
    home_team_id UUID REFERENCES teams(id),
    opened_year INTEGER,
    
    -- Environmental Factors
    altitude_meters INTEGER,
    climate_controlled BOOLEAN DEFAULT true,
    outdoor_venue BOOLEAN DEFAULT false,
    
    -- Operational Information
    active BOOLEAN DEFAULT true,
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT valid_capacity CHECK (capacity > 0 AND capacity <= 100000),
    CONSTRAINT valid_surface CHECK (surface_type IN ('ice', 'synthetic')),
    CONSTRAINT valid_opened_year CHECK (opened_year IS NULL OR opened_year >= 1875)
);

CREATE INDEX idx_venues_league_id ON venues(league_id);
CREATE INDEX idx_venues_home_team ON venues(home_team_id);
CREATE INDEX idx_venues_city_country ON venues(city, country);
```

## Game Events and Play-by-Play Tracking

### Game Events Table (TimescaleDB Hypertable)

The game_events table captures all significant events during hockey games using TimescaleDB for efficient time-series data management and real-time event streaming during live simulations.

```sql
CREATE TABLE game_events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    game_id UUID NOT NULL REFERENCES games(id) ON DELETE CASCADE,
    
    -- Event Timing
    period INTEGER NOT NULL,
    period_time VARCHAR(10) NOT NULL, -- MM:SS remaining in period
    game_time_elapsed INTEGER NOT NULL, -- Total seconds elapsed
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    -- Event Classification
    event_type VARCHAR(30) NOT NULL,
    event_subtype VARCHAR(30),
    significance_level VARCHAR(20) DEFAULT 'normal',
    
    -- Team and Player Involvement
    team_id UUID REFERENCES teams(id),
    primary_player_id UUID REFERENCES players(id),
    secondary_player_id UUID REFERENCES players(id), -- Assists, penalties drawn
    tertiary_player_id UUID REFERENCES players(id), -- Second assists
    goaltender_id UUID REFERENCES players(id),
    
    -- Location and Context
    zone VARCHAR(20), -- 'defensive', 'neutral', 'offensive'
    coordinates JSONB, -- {x: -42 to 42, y: -100 to 100}
    situation VARCHAR(30), -- 'even_strength', 'power_play', 'short_handed'
    strength_state VARCHAR(10), -- '5v5', '5v4', '4v3', etc.
    
    -- Event Details
    description TEXT NOT NULL,
    details JSONB, -- Event-specific additional information
    
    -- Statistical Impact
    plus_players UUID[], -- Players receiving +1
    minus_players UUID[], -- Players receiving -1
    
    -- Video and Media
    video_timestamp INTEGER, -- Seconds into broadcast
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT valid_period CHECK (period >= 1 AND period <= 10),
    CONSTRAINT valid_event_type CHECK (event_type IN (
        'goal', 'assist', 'shot', 'missed_shot', 'blocked_shot', 'save',
        'penalty', 'faceoff', 'hit', 'giveaway', 'takeaway',
        'period_start', 'period_end', 'game_start', 'game_end',
        'timeout', 'challenge', 'substitution', 'line_change'
    )),
    CONSTRAINT valid_zone CHECK (zone IN ('defensive', 'neutral', 'offensive')),
    CONSTRAINT valid_significance CHECK (significance_level IN ('low', 'normal', 'high', 'critical'))
);

-- Convert to TimescaleDB hypertable for time-series optimization
SELECT create_hypertable('game_events', 'timestamp', chunk_time_interval => INTERVAL '1 week');

-- Performance indexes
CREATE INDEX idx_game_events_game_id ON game_events(game_id, timestamp);
CREATE INDEX idx_game_events_type ON game_events(event_type, timestamp);
CREATE INDEX idx_game_events_player ON game_events(primary_player_id, timestamp);
CREATE INDEX idx_game_events_team ON game_events(team_id, timestamp);
CREATE INDEX idx_game_events_period ON game_events(game_id, period, game_time_elapsed);

-- Compression policy for older events
SELECT add_compression_policy('game_events', INTERVAL '30 days');

-- Retention policy for very old events
SELECT add_retention_policy('game_events', INTERVAL '10 years');
```

### Goal Scoring Details

Specialized table for detailed goal information that supports advanced hockey analytics and historical tracking of scoring plays with comprehensive context and attribution.

```sql
CREATE TABLE goals (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    game_event_id UUID NOT NULL REFERENCES game_events(id) ON DELETE CASCADE,
    game_id UUID NOT NULL REFERENCES games(id) ON DELETE CASCADE,
    
    -- Goal Details
    scorer_id UUID NOT NULL REFERENCES players(id),
    primary_assist_id UUID REFERENCES players(id),
    secondary_assist_id UUID REFERENCES players(id),
    goaltender_id UUID REFERENCES players(id),
    
    -- Goal Classification
    goal_type VARCHAR(30) NOT NULL,
    shot_type VARCHAR(30),
    strength VARCHAR(20) NOT NULL,
    
    -- Situation Context
    home_skaters INTEGER NOT NULL,
    away_skaters INTEGER NOT NULL,
    empty_net BOOLEAN DEFAULT false,
    penalty_shot BOOLEAN DEFAULT false,
    shootout_goal BOOLEAN DEFAULT false,
    
    -- Shot Details
    shot_distance_feet INTEGER,
    shot_angle INTEGER, -- Degrees from goal line
    rebound BOOLEAN DEFAULT false,
    deflection BOOLEAN DEFAULT false,
    screen BOOLEAN DEFAULT false,
    
    -- Game Impact
    game_winning_goal BOOLEAN DEFAULT false,
    go_ahead_goal BOOLEAN DEFAULT false,
    tying_goal BOOLEAN DEFAULT false,
    
    -- Video Review
    reviewed BOOLEAN DEFAULT false,
    review_result VARCHAR(20),
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT valid_goal_type CHECK (goal_type IN (
        'even_strength', 'power_play', 'short_handed', 'penalty_shot', 
        'empty_net', 'shootout'
    )),
    CONSTRAINT valid_shot_type CHECK (shot_type IN (
        'wrist', 'slap', 'snap', 'backhand', 'tip', 'deflection', 'wrap_around'
    )),
    CONSTRAINT valid_skater_counts CHECK (
        home_skaters >= 3 AND home_skaters <= 6 AND
        away_skaters >= 3 AND away_skaters <= 6
    ),
    CONSTRAINT valid_review_result CHECK (review_result IN ('confirmed', 'overturned', 'inconclusive'))
);

CREATE INDEX idx_goals_game_id ON goals(game_id);
CREATE INDEX idx_goals_scorer ON goals(scorer_id);
CREATE INDEX idx_goals_type ON goals(goal_type);
CREATE INDEX idx_goals_strength ON goals(strength);
```

### Penalties and Infractions

Comprehensive penalty tracking that supports hockey rule enforcement, player discipline analysis, and game situation management during power plays and penalty kills.

```sql
CREATE TABLE penalties (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    game_event_id UUID NOT NULL REFERENCES game_events(id) ON DELETE CASCADE,
    game_id UUID NOT NULL REFERENCES games(id) ON DELETE CASCADE,
    
    -- Penalty Details
    penalized_player_id UUID NOT NULL REFERENCES players(id),
    drawn_by_player_id UUID REFERENCES players(id),
    team_id UUID NOT NULL REFERENCES teams(id),
    
    -- Penalty Classification
    penalty_type VARCHAR(30) NOT NULL,
    penalty_severity VARCHAR(20) NOT NULL,
    duration_minutes INTEGER NOT NULL,
    
    -- Game Situation
    period INTEGER NOT NULL,
    time_assessed VARCHAR(10) NOT NULL,
    time_expires VARCHAR(10),
    served_by_player_id UUID REFERENCES players(id), -- For fighting majors, etc.
    
    -- Penalty Status
    status VARCHAR(20) DEFAULT 'active',
    power_play_created BOOLEAN DEFAULT true,
    coincidental BOOLEAN DEFAULT false,
    
    -- Discipline Tracking
    first_offense BOOLEAN DEFAULT true,
    supplemental_discipline BOOLEAN DEFAULT false,
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT valid_penalty_type CHECK (penalty_type IN (
        'minor', 'major', 'misconduct', 'game_misconduct', 'match', 'penalty_shot'
    )),
    CONSTRAINT valid_penalty_severity CHECK (penalty_severity IN (
        'minor', 'double_minor', 'major', 'misconduct', 'game_misconduct', 'match'
    )),
    CONSTRAINT valid_duration CHECK (duration_minutes >= 0 AND duration_minutes <= 60),
    CONSTRAINT valid_penalty_status CHECK (status IN ('active', 'expired', 'cancelled', 'served'))
);

CREATE INDEX idx_penalties_game_id ON penalties(game_id);
CREATE INDEX idx_penalties_player ON penalties(penalized_player_id);
CREATE INDEX idx_penalties_type ON penalties(penalty_type);
CREATE INDEX idx_penalties_team ON penalties(team_id);
```

## Game Statistics and Performance Tracking

### Team Game Statistics

Comprehensive team performance metrics for individual games that support advanced analytics and comparative analysis across different game situations and opponent matchups.

```sql
CREATE TABLE team_game_statistics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    game_id UUID NOT NULL REFERENCES games(id) ON DELETE CASCADE,
    team_id UUID NOT NULL REFERENCES teams(id) ON DELETE CASCADE,
    
    -- Basic Statistics
    goals INTEGER DEFAULT 0,
    assists INTEGER DEFAULT 0,
    points INTEGER DEFAULT 0,
    shots INTEGER DEFAULT 0,
    shots_against INTEGER DEFAULT 0,
    
    -- Advanced Metrics
    shots_blocked INTEGER DEFAULT 0,
    missed_shots INTEGER DEFAULT 0,
    hits INTEGER DEFAULT 0,
    giveaways INTEGER DEFAULT 0,
    takeaways INTEGER DEFAULT 0,
    
    -- Faceoff Performance
    faceoffs_won INTEGER DEFAULT 0,
    faceoffs_lost INTEGER DEFAULT 0,
    faceoff_percentage DECIMAL(5,2),
    
    -- Special Teams
    power_play_goals INTEGER DEFAULT 0,
    power_play_opportunities INTEGER DEFAULT 0,
    power_play_percentage DECIMAL(5,2),
    short_handed_goals INTEGER DEFAULT 0,
    penalty_kill_opportunities INTEGER DEFAULT 0,
    penalty_kill_percentage DECIMAL(5,2),
    
    -- Time-Based Metrics
    time_on_attack INTEGER DEFAULT 0, -- Seconds in offensive zone
    time_in_own_zone INTEGER DEFAULT 0, -- Seconds in defensive zone
    
    -- Penalties
    penalties INTEGER DEFAULT 0,
    penalty_minutes INTEGER DEFAULT 0,
    
    -- Goaltending (if applicable)
    saves INTEGER DEFAULT 0,
    goals_against INTEGER DEFAULT 0,
    save_percentage DECIMAL(5,3),
    
    -- Advanced Analytics
    corsi_for INTEGER DEFAULT 0, -- All shot attempts for
    corsi_against INTEGER DEFAULT 0, -- All shot attempts against
    fenwick_for INTEGER DEFAULT 0, -- Unblocked shot attempts for
    fenwick_against INTEGER DEFAULT 0, -- Unblocked shot attempts against
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT valid_percentages CHECK (
        (faceoff_percentage IS NULL OR (faceoff_percentage >= 0 AND faceoff_percentage <= 100)) AND
        (power_play_percentage IS NULL OR (power_play_percentage >= 0 AND power_play_percentage <= 100)) AND
        (penalty_kill_percentage IS NULL OR (penalty_kill_percentage >= 0 AND penalty_kill_percentage <= 100)) AND
        (save_percentage IS NULL OR (save_percentage >= 0 AND save_percentage <= 1))
    )
);

CREATE UNIQUE INDEX idx_team_game_stats_unique ON team_game_statistics(game_id, team_id);
CREATE INDEX idx_team_game_stats_team ON team_game_statistics(team_id);
```

### Player Game Statistics

Individual player performance tracking for each game that enables detailed player evaluation, line chemistry analysis, and performance trending over time.

```sql
CREATE TABLE player_game_statistics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    game_id UUID NOT NULL REFERENCES games(id) ON DELETE CASCADE,
    player_id UUID NOT NULL REFERENCES players(id) ON DELETE CASCADE,
    team_id UUID NOT NULL REFERENCES teams(id) ON DELETE CASCADE,
    
    -- Basic Scoring Statistics
    goals INTEGER DEFAULT 0,
    assists INTEGER DEFAULT 0,
    points INTEGER DEFAULT 0,
    plus_minus INTEGER DEFAULT 0,
    
    -- Shooting Statistics
    shots INTEGER DEFAULT 0,
    shooting_percentage DECIMAL(5,2),
    missed_shots INTEGER DEFAULT 0,
    blocked_shots INTEGER DEFAULT 0,
    
    -- Physical Play
    hits INTEGER DEFAULT 0,
    hits_received INTEGER DEFAULT 0,
    blocked_shots_against INTEGER DEFAULT 0,
    
    -- Puck Possession
    giveaways INTEGER DEFAULT 0,
    takeaways INTEGER DEFAULT 0,
    
    -- Faceoffs (Centers)
    faceoffs_won INTEGER DEFAULT 0,
    faceoffs_lost INTEGER DEFAULT 0,
    faceoff_percentage DECIMAL(5,2),
    
    -- Time on Ice
    time_on_ice_total INTEGER DEFAULT 0, -- Total seconds
    time_on_ice_even_strength INTEGER DEFAULT 0,
    time_on_ice_power_play INTEGER DEFAULT 0,
    time_on_ice_short_handed INTEGER DEFAULT 0,
    
    -- Special Teams Performance
    power_play_goals INTEGER DEFAULT 0,
    power_play_assists INTEGER DEFAULT 0,
    power_play_points INTEGER DEFAULT 0,
    short_handed_goals INTEGER DEFAULT 0,
    short_handed_assists INTEGER DEFAULT 0,
    short_handed_points INTEGER DEFAULT 0,
    
    -- Penalties
    penalties INTEGER DEFAULT 0,
    penalty_minutes INTEGER DEFAULT 0,
    penalties_drawn INTEGER DEFAULT 0,
    
    -- Goaltender-Specific Statistics
    wins INTEGER DEFAULT 0,
    losses INTEGER DEFAULT 0,
    overtime_losses INTEGER DEFAULT 0,
    saves INTEGER DEFAULT 0,
    shots_against INTEGER DEFAULT 0,
    goals_against INTEGER DEFAULT 0,
    save_percentage DECIMAL(5,3),
    goals_against_average DECIMAL(4,2),
    shutouts INTEGER DEFAULT 0,
    
    -- Advanced Metrics
    corsi_for INTEGER DEFAULT 0,
    corsi_against INTEGER DEFAULT 0,
    corsi_percentage DECIMAL(5,2),
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT valid_plus_minus CHECK (plus_minus >= -10 AND plus_minus <= 10),
    CONSTRAINT valid_percentages CHECK (
        (shooting_percentage IS NULL OR (shooting_percentage >= 0 AND shooting_percentage <= 100)) AND
        (faceoff_percentage IS NULL OR (faceoff_percentage >= 0 AND faceoff_percentage <= 100)) AND
        (save_percentage IS NULL OR (save_percentage >= 0 AND save_percentage <= 1)) AND
        (corsi_percentage IS NULL OR (corsi_percentage >= 0 AND corsi_percentage <= 100))
    ),
    CONSTRAINT valid_goalie_stats CHECK (
        (goals_against_average IS NULL OR goals_against_average >= 0)
    )
);

CREATE UNIQUE INDEX idx_player_game_stats_unique ON player_game_statistics(game_id, player_id);
CREATE INDEX idx_player_game_stats_player ON player_game_statistics(player_id);
CREATE INDEX idx_player_game_stats_team ON player_game_statistics(team_id);

-- Convert to TimescaleDB hypertable for efficient time-series queries
SELECT create_hypertable('player_game_statistics', 'created_at', chunk_time_interval => INTERVAL '1 month');
```

## Real-Time Simulation State Management

### Game Simulation State

Real-time game state tracking that supports live simulation with pause/resume functionality and detailed state preservation for complex game situations.

```sql
CREATE TABLE game_simulation_state (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    game_id UUID NOT NULL REFERENCES games(id) ON DELETE CASCADE,
    
    -- Simulation Control
    simulation_status VARCHAR(20) NOT NULL DEFAULT 'ready',
    simulation_speed DECIMAL(4,2) DEFAULT 1.0, -- 1.0 = real-time
    auto_advance BOOLEAN DEFAULT true,
    
    -- Current Game State
    current_period INTEGER DEFAULT 1,
    period_time_remaining INTEGER NOT NULL, -- Seconds
    intermission_time_remaining INTEGER DEFAULT 0,
    
    -- Score State
    home_score INTEGER DEFAULT 0,
    away_score INTEGER DEFAULT 0,
    
    -- Ice Conditions
    home_skaters INTEGER DEFAULT 6, -- Including goaltender
    away_skaters INTEGER DEFAULT 6,
    power_play_time_remaining INTEGER DEFAULT 0,
    penalty_kill_team_id UUID REFERENCES teams(id),
    
    -- Active Penalties
    active_penalties JSONB DEFAULT '[]',
    
    -- Line Combinations
    home_lines JSONB,
    away_lines JSONB,
    
    -- Simulation Metadata
    random_seed BIGINT,
    events_processed INTEGER DEFAULT 0,
    last_event_time TIMESTAMP WITH TIME ZONE,
    
    -- Pause/Resume State
    paused_at TIMESTAMP WITH TIME ZONE,
    paused_by_user_id UUID REFERENCES users(id),
    pause_reason TEXT,
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT valid_simulation_status CHECK (simulation_status IN (
        'ready', 'in_progress', 'paused', 'completed', 'error'
    )),
    CONSTRAINT valid_period CHECK (current_period >= 1 AND current_period <= 10),
    CONSTRAINT valid_time_remaining CHECK (period_time_remaining >= 0 AND period_time_remaining <= 1200),
    CONSTRAINT valid_skater_counts CHECK (
        home_skaters >= 3 AND home_skaters <= 6 AND
        away_skaters >= 3 AND away_skaters <= 6
    )
);

CREATE UNIQUE INDEX idx_game_simulation_state_game ON game_simulation_state(game_id);
CREATE INDEX idx_game_simulation_status ON game_simulation_state(simulation_status);
```

### Live Game Feed

Real-time event streaming table that supports WebSocket broadcasting and live game following with configurable update frequencies and event filtering.

```sql
CREATE TABLE live_game_feed (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    game_id UUID NOT NULL REFERENCES games(id) ON DELETE CASCADE,
    
    -- Feed Entry Details
    feed_type VARCHAR(30) NOT NULL,
    message TEXT NOT NULL,
    timestamp TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    -- Game Context
    period INTEGER,
    time_remaining VARCHAR(10),
    home_score INTEGER,
    away_score INTEGER,
    
    -- Event Reference
    game_event_id UUID REFERENCES game_events(id),
    
    -- Broadcast Control
    priority VARCHAR(20) DEFAULT 'normal',
    broadcast_delay INTEGER DEFAULT 0, -- Seconds
    
    -- Targeting
    target_audience VARCHAR(30) DEFAULT 'all', -- 'all', 'home_fans', 'away_fans'
    
    -- Media Attachments
    media_urls TEXT[],
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT valid_feed_type CHECK (feed_type IN (
        'goal', 'penalty', 'period_start', 'period_end', 'game_start', 'game_end',
        'save', 'hit', 'milestone', 'injury', 'review', 'timeout'
    )),
    CONSTRAINT valid_priority CHECK (priority IN ('low', 'normal', 'high', 'critical')),
    CONSTRAINT valid_audience CHECK (target_audience IN ('all', 'home_fans', 'away_fans', 'officials'))
);

-- Convert to TimescaleDB hypertable
SELECT create_hypertable('live_game_feed', 'timestamp', chunk_time_interval => INTERVAL '1 day');

CREATE INDEX idx_live_feed_game ON live_game_feed(game_id, timestamp);
CREATE INDEX idx_live_feed_type ON live_game_feed(feed_type, timestamp);
CREATE INDEX idx_live_feed_priority ON live_game_feed(priority, timestamp);

-- Retention policy for live feed (keep for 30 days)
SELECT add_retention_policy('live_game_feed', INTERVAL '30 days');
```

## Playoff and Tournament Management

### Playoff Series Structure

Comprehensive playoff management that supports various tournament formats including best-of series, single elimination, and round-robin competitions with automatic advancement logic.

```sql
CREATE TABLE playoff_series (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    season_id UUID NOT NULL REFERENCES seasons(id) ON DELETE CASCADE,
    
    -- Series Identification
    round_number INTEGER NOT NULL,
    series_number INTEGER NOT NULL,
    series_name VARCHAR(100), -- "Stanley Cup Final", "Conference Semifinal"
    
    -- Teams
    home_team_id UUID NOT NULL REFERENCES teams(id),
    away_team_id UUID NOT NULL REFERENCES teams(id),
    higher_seed_team_id UUID REFERENCES teams(id),
    
    -- Series Format
    series_format VARCHAR(20) NOT NULL DEFAULT 'best_of_7',
    games_required_to_win INTEGER NOT NULL,
    
    -- Series Status
    status VARCHAR(20) DEFAULT 'upcoming',
    home_wins INTEGER DEFAULT 0,
    away_wins INTEGER DEFAULT 0,
    winning_team_id UUID REFERENCES teams(id),
    
    -- Schedule
    start_date DATE,
    end_date DATE,
    
    -- Advancement
    advances_to_series_id UUID REFERENCES playoff_series(id),
    eliminated_team_id UUID REFERENCES teams(id),
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    CONSTRAINT valid_series_format CHECK (series_format IN (
        'best_of_3', 'best_of_5', 'best_of_7', 'single_game'
    )),
    CONSTRAINT valid_series_status CHECK (status IN (
        'upcoming', 'in_progress', 'completed'
    )),
    CONSTRAINT different_teams CHECK (home_team_id != away_team_id),
    CONSTRAINT valid_wins CHECK (home_wins >= 0 AND away_wins >= 0)
);

CREATE INDEX idx_playoff_series_season ON playoff_series(season_id);
CREATE INDEX idx_playoff_series_round ON playoff_series(round_number);
CREATE INDEX idx_playoff_series_teams ON playoff_series(home_team_id, away_team_id);
```

## Performance Optimization and Materialized Views

### Game Statistics Aggregation

Materialized views for efficient statistical queries and dashboard performance with automated refresh procedures that maintain data currency.

```sql
-- Season-level team statistics aggregation
CREATE MATERIALIZED VIEW team_season_statistics AS
SELECT 
    t.id as team_id,
    t.name as team_name,
    s.id as season_id,
    s.name as season_name,
    
    -- Games Played
    COUNT(g.id) as games_played,
    SUM(CASE WHEN g.winning_team_id = t.id THEN 1 ELSE 0 END) as wins,
    SUM(CASE WHEN g.winning_team_id != t.id AND g.status = 'completed' THEN 1 ELSE 0 END) as losses,
    SUM(CASE WHEN g.overtime = true AND g.winning_team_id != t.id THEN 1 ELSE 0 END) as overtime_losses,
    
    -- Points (2 for win, 1 for OT/SO loss)
    (SUM(CASE WHEN g.winning_team_id = t.id THEN 2 ELSE 0 END) + 
     SUM(CASE WHEN g.overtime = true AND g.winning_team_id != t.id THEN 1 ELSE 0 END)) as points,
    
    -- Goals
    SUM(CASE WHEN g.home_team_id = t.id THEN g.home_score ELSE g.away_score END) as goals_for,
    SUM(CASE WHEN g.home_team_id = t.id THEN g.away_score ELSE g.home_score END) as goals_against,
    
    -- Advanced Statistics
    AVG(tgs.shots) as avg_shots_for,
    AVG(tgs.shots_against) as avg_shots_against,
    AVG(tgs.power_play_percentage) as avg_power_play_percentage,
    AVG(tgs.penalty_kill_percentage) as avg_penalty_kill_percentage
    
FROM teams t
JOIN seasons s ON s.league_id = t.league_id
LEFT JOIN games g ON (g.home_team_id = t.id OR g.away_team_id = t.id) 
    AND g.season_id = s.id AND g.status = 'completed'
LEFT JOIN team_game_statistics tgs ON tgs.game_id = g.id AND tgs.team_id = t.id
GROUP BY t.id, t.name, s.id, s.name;

CREATE UNIQUE INDEX idx_team_season_stats_unique ON team_season_statistics(team_id, season_id);

-- Player season statistics aggregation
CREATE MATERIALIZED VIEW player_season_statistics AS
SELECT 
    p.id as player_id,
    p.first_name,
    p.last_name,
    p.position,
    t.id as team_id,
    t.name as team_name,
    s.id as season_id,
    
    -- Games and Basic Stats
    COUNT(pgs.game_id) as games_played,
    SUM(pgs.goals) as goals,
    SUM(pgs.assists) as assists,
    SUM(pgs.points) as points,
    SUM(pgs.plus_minus) as plus_minus,
    
    -- Shooting
    SUM(pgs.shots) as shots,
    CASE WHEN SUM(pgs.shots) > 0 
         THEN (SUM(pgs.goals)::DECIMAL / SUM(pgs.shots)) * 100 
         ELSE NULL END as shooting_percentage,
    
    -- Time on Ice
    SUM(pgs.time_on_ice_total) as total_time_on_ice,
    CASE WHEN COUNT(pgs.game_id) > 0 
         THEN SUM(pgs.time_on_ice_total) / COUNT(pgs.game_id) 
         ELSE 0 END as avg_time_on_ice,
    
    -- Special Teams
    SUM(pgs.power_play_goals) as power_play_goals,
    SUM(pgs.power_play_assists) as power_play_assists,
    SUM(pgs.short_handed_goals) as short_handed_goals,
    
    -- Goaltender Stats
    SUM(pgs.wins) as wins,
    SUM(pgs.losses) as losses,
    SUM(pgs.saves) as saves,
    SUM(pgs.shots_against) as shots_against,
    CASE WHEN SUM(pgs.shots_against) > 0 
         THEN SUM(pgs.saves)::DECIMAL / SUM(pgs.shots_against) 
         ELSE NULL END as save_percentage
    
FROM players p
JOIN teams t ON p.current_team_id = t.id
JOIN seasons s ON s.league_id = t.league_id
LEFT JOIN player_game_statistics pgs ON pgs.player_id = p.id 
    AND pgs.game_id IN (
        SELECT g.id FROM games g 
        WHERE g.season_id = s.id AND g.status = 'completed'
    )
GROUP BY p.id, p.first_name, p.last_name, p.position, t.id, t.name, s.id;

CREATE UNIQUE INDEX idx_player_season_stats_unique ON player_season_statistics(player_id, season_id);

-- Refresh policies for materialized views
CREATE OR REPLACE FUNCTION refresh_season_statistics()
RETURNS void AS $$
BEGIN
    REFRESH MATERIALIZED VIEW CONCURRENTLY team_season_statistics;
    REFRESH MATERIALIZED VIEW CONCURRENTLY player_season_statistics;
END;
$$ LANGUAGE plpgsql;

-- Schedule automatic refresh every hour
SELECT cron.schedule('refresh-season-stats', '0 * * * *', 'SELECT refresh_season_statistics();');
```

## Example Queries and Usage Patterns

### Common Game Data Queries

```sql
-- Get live game information with current state
SELECT 
    g.id,
    ht.name as home_team,
    at.name as away_team,
    g.home_score,
    g.away_score,
    g.current_period,
    g.period_time_remaining,
    g.status,
    g.overtime,
    g.shootout
FROM games g
JOIN teams ht ON g.home_team_id = ht.id
JOIN teams at ON g.away_team_id = at.id
WHERE g.status = 'in_progress'
ORDER BY g.scheduled_date;

-- Get recent goals with scorer information
SELECT 
    g.scheduled_date,
    ht.name as home_team,
    at.name as away_team,
    go.goal_type,
    p.first_name || ' ' || p.last_name as scorer,
    a1.first_name || ' ' || a1.last_name as primary_assist,
    a2.first_name || ' ' || a2.last_name as secondary_assist,
    ge.period,
    ge.period_time,
    ge.description
FROM goals go
JOIN game_events ge ON go.game_event_id = ge.id
JOIN games g ON go.game_id = g.id
JOIN teams ht ON g.home_team_id = ht.id
JOIN teams at ON g.away_team_id = at.id
JOIN players p ON go.scorer_id = p.id
LEFT JOIN players a1 ON go.primary_assist_id = a1.id
LEFT JOIN players a2 ON go.secondary_assist_id = a2.id
WHERE g.scheduled_date >= CURRENT_DATE - INTERVAL '7 days'
ORDER BY g.scheduled_date DESC, ge.game_time_elapsed DESC;

-- Get team standings for current season
SELECT 
    t.name as team_name,
    tss.games_played,
    tss.wins,
    tss.losses,
    tss.overtime_losses,
    tss.points,
    tss.goals_for,
    tss.goals_against,
    (tss.goals_for - tss.goals_against) as goal_differential,
    ROUND(tss.points::DECIMAL / (tss.games_played * 2) * 100, 1) as points_percentage
FROM team_season_statistics tss
JOIN teams t ON tss.team_id = t.id
JOIN seasons s ON tss.season_id = s.id
WHERE s.status = 'active'
ORDER BY tss.points DESC, goal_differential DESC;
```

This comprehensive games schema provides the foundation for sophisticated hockey game management, real-time simulation, and statistical tracking while maintaining performance and scalability for multi-tenant operations.