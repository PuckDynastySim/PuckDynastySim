# Players Schema Documentation - Hockey League Simulation System

This comprehensive players schema documentation defines the complete database structure for player entities within the Hockey League Simulation System. The schema encompasses player attributes, career statistics, contract relationships, and hockey-specific business rules that ensure realistic player management and accurate simulation mechanics.

## Core Player Entity Structure

### Primary Player Table

The players table serves as the central entity for all individual player information including biographical data, physical attributes, and current organizational assignments. The schema design accommodates comprehensive player tracking while maintaining performance optimization for frequent queries and updates.

```sql
CREATE TABLE players (
    -- Primary identification
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Organizational relationships
    league_id UUID NOT NULL REFERENCES leagues(id) ON DELETE CASCADE,
    current_team_id UUID REFERENCES teams(id) ON DELETE SET NULL,
    draft_team_id UUID REFERENCES teams(id) ON DELETE SET NULL,
    
    -- Personal information
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    birth_date DATE NOT NULL,
    nationality VARCHAR(3) NOT NULL, -- ISO 3166-1 alpha-3 country code
    
    -- Physical attributes
    height_cm INTEGER CHECK (height_cm >= 150 AND height_cm <= 220),
    weight_kg INTEGER CHECK (weight_kg >= 60 AND weight_kg <= 140),
    handedness VARCHAR(10) CHECK (handedness IN ('left', 'right')),
    
    -- Hockey-specific attributes
    position VARCHAR(20) NOT NULL CHECK (position IN (
        'left_wing', 'center', 'right_wing', 
        'left_defense', 'right_defense', 'goaltender'
    )),
    jersey_number INTEGER CHECK (jersey_number >= 1 AND jersey_number <= 99),
    
    -- Career information
    draft_year INTEGER,
    draft_round INTEGER CHECK (draft_round >= 1 AND draft_round <= 15),
    draft_position INTEGER CHECK (draft_position >= 1 AND draft_position <= 300),
    professional_debut DATE,
    
    -- Performance ratings
    overall_rating INTEGER NOT NULL CHECK (overall_rating >= 25 AND overall_rating <= 99),
    potential_rating INTEGER NOT NULL CHECK (potential_rating >= 25 AND potential_rating <= 99),
    
    -- Detailed skill ratings (JSONB for flexibility)
    ratings JSONB NOT NULL,
    
    -- Contract and status information
    contract_status VARCHAR(20) DEFAULT 'unsigned' CHECK (contract_status IN (
        'signed', 'unsigned', 'restricted_free_agent', 'unrestricted_free_agent',
        'waivers', 'retired', 'suspended'
    )),
    
    -- Injury and availability
    injury_status JSONB DEFAULT '{}',
    availability_status VARCHAR(20) DEFAULT 'active' CHECK (availability_status IN (
        'active', 'injured', 'suspended', 'personal_leave', 'retired'
    )),
    
    -- Development and progression
    development_curve VARCHAR(20) DEFAULT 'normal' CHECK (development_curve IN (
        'early_peak', 'normal', 'late_bloomer', 'steady_decline'
    )),
    morale INTEGER DEFAULT 75 CHECK (morale >= 0 AND morale <= 100),
    
    -- Metadata
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES users(id),
    
    -- Constraints
    CONSTRAINT players_age_check CHECK (
        EXTRACT(year FROM age(birth_date)) >= 16 AND 
        EXTRACT(year FROM age(birth_date)) <= 50
    ),
    CONSTRAINT players_potential_rating_check CHECK (potential_rating >= overall_rating),
    CONSTRAINT players_unique_jersey_per_team EXCLUDE (
        current_team_id WITH =, jersey_number WITH =
    ) WHERE (current_team_id IS NOT NULL AND jersey_number IS NOT NULL)
);
```

### Player Ratings Schema

Player ratings utilize JSONB storage for flexibility while maintaining validation through application logic and database constraints. The ratings structure differs between skaters and goaltenders to accommodate position-specific skills and simulation requirements.

```sql
-- Validation function for player ratings
CREATE OR REPLACE FUNCTION validate_player_ratings(ratings JSONB, position VARCHAR)
RETURNS BOOLEAN AS $$
BEGIN
    -- All ratings must be between 25 and 99
    IF NOT (
        SELECT bool_and(value::int >= 25 AND value::int <= 99)
        FROM jsonb_each_text(ratings)
        WHERE key != 'notes'
    ) THEN
        RETURN FALSE;
    END IF;
    
    -- Position-specific validation
    IF position = 'goaltender' THEN
        RETURN ratings ? 'movement' AND ratings ? 'rebound_control' AND
               ratings ? 'vision' AND ratings ? 'aggressiveness' AND
               ratings ? 'puck_control' AND ratings ? 'flexibility' AND
               ratings ? 'discipline' AND ratings ? 'injury' AND
               ratings ? 'fatigue' AND ratings ? 'poise';
    ELSE
        RETURN ratings ? 'passing' AND ratings ? 'shooting' AND
               ratings ? 'defense' AND ratings ? 'puck_control' AND
               ratings ? 'checking' AND ratings ? 'fighting' AND
               ratings ? 'discipline' AND ratings ? 'injury' AND
               ratings ? 'fatigue' AND ratings ? 'poise';
    END IF;
END;
$$ LANGUAGE plpgsql;

-- Add constraint to validate ratings
ALTER TABLE players ADD CONSTRAINT players_valid_ratings 
CHECK (validate_player_ratings(ratings, position));
```

**Skater Ratings Structure:**
```json
{
    "discipline": 75,      // Penalty avoidance and composure
    "injury": 82,          // Injury resistance and durability
    "fatigue": 78,         // Stamina and endurance
    "passing": 85,         // Passing accuracy and vision
    "shooting": 79,        // Shot power and accuracy
    "defense": 88,         // Defensive positioning and stick work
    "puck_control": 83,    // Stickhandling and puck possession
    "checking": 77,        // Body checking and physical play
    "fighting": 45,        // Fighting ability (optional)
    "poise": 81,           // Performance under pressure
    "notes": "Strong two-way forward with excellent defensive awareness"
}
```

**Goaltender Ratings Structure:**
```json
{
    "discipline": 88,      // Penalty avoidance and composure
    "injury": 79,          // Injury resistance and durability
    "fatigue": 85,         // Stamina and endurance
    "poise": 91,           // Performance under pressure
    "movement": 87,        // Skating and positioning
    "rebound_control": 83, // Rebound direction and control
    "vision": 89,          // Reading plays and anticipation
    "aggressiveness": 76,  // Challenge timing and style
    "puck_control": 72,    // Puck handling behind net
    "flexibility": 84,     // Save reach and recovery
    "notes": "Aggressive butterfly style with excellent glove hand"
}
```

## Player Statistics and Performance Tracking

### Career Statistics Table

Player statistics utilize TimescaleDB hypertables for efficient time-series data management while maintaining comprehensive tracking of individual and team performance across multiple seasons and league contexts.

```sql
CREATE TABLE player_statistics (
    -- Primary identification
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    player_id UUID NOT NULL REFERENCES players(id) ON DELETE CASCADE,
    team_id UUID NOT NULL REFERENCES teams(id) ON DELETE CASCADE,
    league_id UUID NOT NULL REFERENCES leagues(id) ON DELETE CASCADE,
    season INTEGER NOT NULL,
    
    -- Game participation
    games_played INTEGER DEFAULT 0 CHECK (games_played >= 0),
    games_started INTEGER DEFAULT 0 CHECK (games_started >= 0 AND games_started <= games_played),
    
    -- Offensive statistics (skaters)
    goals INTEGER DEFAULT 0 CHECK (goals >= 0),
    assists INTEGER DEFAULT 0 CHECK (assists >= 0),
    points INTEGER GENERATED ALWAYS AS (goals + assists) STORED,
    shots INTEGER DEFAULT 0 CHECK (shots >= 0),
    shooting_percentage DECIMAL(5,2) GENERATED ALWAYS AS (
        CASE WHEN shots > 0 THEN (goals::decimal / shots * 100) ELSE 0 END
    ) STORED,
    
    -- Power play statistics
    power_play_goals INTEGER DEFAULT 0 CHECK (power_play_goals >= 0 AND power_play_goals <= goals),
    power_play_assists INTEGER DEFAULT 0 CHECK (power_play_assists >= 0 AND power_play_assists <= assists),
    power_play_points INTEGER GENERATED ALWAYS AS (power_play_goals + power_play_assists) STORED,
    
    -- Short-handed statistics
    short_handed_goals INTEGER DEFAULT 0 CHECK (short_handed_goals >= 0 AND short_handed_goals <= goals),
    short_handed_assists INTEGER DEFAULT 0 CHECK (short_handed_assists >= 0 AND short_handed_assists <= assists),
    
    -- Game-winning and overtime statistics
    game_winning_goals INTEGER DEFAULT 0 CHECK (game_winning_goals >= 0 AND game_winning_goals <= goals),
    overtime_goals INTEGER DEFAULT 0 CHECK (overtime_goals >= 0 AND overtime_goals <= goals),
    shootout_goals INTEGER DEFAULT 0 CHECK (shootout_goals >= 0),
    shootout_attempts INTEGER DEFAULT 0 CHECK (shootout_attempts >= shootout_goals),
    
    -- Defensive and physical statistics
    plus_minus INTEGER DEFAULT 0,
    penalty_minutes INTEGER DEFAULT 0 CHECK (penalty_minutes >= 0),
    hits INTEGER DEFAULT 0 CHECK (hits >= 0),
    blocked_shots INTEGER DEFAULT 0 CHECK (blocked_shots >= 0),
    takeaways INTEGER DEFAULT 0 CHECK (takeaways >= 0),
    giveaways INTEGER DEFAULT 0 CHECK (giveaways >= 0),
    
    -- Faceoff statistics (centers primarily)
    faceoff_wins INTEGER DEFAULT 0 CHECK (faceoff_wins >= 0),
    faceoff_losses INTEGER DEFAULT 0 CHECK (faceoff_losses >= 0),
    faceoff_percentage DECIMAL(5,2) GENERATED ALWAYS AS (
        CASE WHEN (faceoff_wins + faceoff_losses) > 0 
        THEN (faceoff_wins::decimal / (faceoff_wins + faceoff_losses) * 100) 
        ELSE 0 END
    ) STORED,
    
    -- Time on ice (in seconds)
    time_on_ice_total INTEGER DEFAULT 0 CHECK (time_on_ice_total >= 0),
    time_on_ice_even_strength INTEGER DEFAULT 0 CHECK (time_on_ice_even_strength >= 0),
    time_on_ice_power_play INTEGER DEFAULT 0 CHECK (time_on_ice_power_play >= 0),
    time_on_ice_short_handed INTEGER DEFAULT 0 CHECK (time_on_ice_short_handed >= 0),
    average_time_on_ice DECIMAL(5,2) GENERATED ALWAYS AS (
        CASE WHEN games_played > 0 
        THEN (time_on_ice_total::decimal / games_played / 60) 
        ELSE 0 END
    ) STORED,
    
    -- Goaltender-specific statistics
    wins INTEGER DEFAULT 0 CHECK (wins >= 0),
    losses INTEGER DEFAULT 0 CHECK (losses >= 0),
    overtime_losses INTEGER DEFAULT 0 CHECK (overtime_losses >= 0),
    ties INTEGER DEFAULT 0 CHECK (ties >= 0),
    saves INTEGER DEFAULT 0 CHECK (saves >= 0),
    shots_against INTEGER DEFAULT 0 CHECK (shots_against >= 0 AND shots_against >= saves),
    goals_against INTEGER DEFAULT 0 CHECK (goals_against >= 0),
    save_percentage DECIMAL(5,3) GENERATED ALWAYS AS (
        CASE WHEN shots_against > 0 
        THEN (saves::decimal / shots_against) 
        ELSE 0 END
    ) STORED,
    goals_against_average DECIMAL(4,2) GENERATED ALWAYS AS (
        CASE WHEN time_on_ice_total > 0 
        THEN (goals_against::decimal * 3600 / time_on_ice_total) 
        ELSE 0 END
    ) STORED,
    shutouts INTEGER DEFAULT 0 CHECK (shutouts >= 0),
    
    -- Quality of competition metrics
    quality_of_teammates DECIMAL(4,2),
    quality_of_competition DECIMAL(4,2),
    zone_start_percentage DECIMAL(5,2),
    
    -- Advanced analytics
    corsi_for INTEGER DEFAULT 0,
    corsi_against INTEGER DEFAULT 0,
    fenwick_for INTEGER DEFAULT 0,
    fenwick_against INTEGER DEFAULT 0,
    
    -- Metadata
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    -- Constraints
    CONSTRAINT player_statistics_unique_season UNIQUE (player_id, season, league_id),
    CONSTRAINT player_statistics_goalie_games CHECK (
        (wins + losses + overtime_losses + ties) <= games_played
    )
);

-- Convert to TimescaleDB hypertable for time-series optimization
SELECT create_hypertable('player_statistics', 'created_at');

-- Add compression policy for older statistics
SELECT add_compression_policy('player_statistics', INTERVAL '1 year');
```

### Game-by-Game Performance Tracking

Individual game performance tracking provides detailed event-level statistics that support advanced analytics and performance evaluation while maintaining efficient storage and query capabilities.

```sql
CREATE TABLE player_game_statistics (
    -- Primary identification
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    player_id UUID NOT NULL REFERENCES players(id) ON DELETE CASCADE,
    game_id UUID NOT NULL REFERENCES games(id) ON DELETE CASCADE,
    team_id UUID NOT NULL REFERENCES teams(id) ON DELETE CASCADE,
    
    -- Game participation
    played BOOLEAN DEFAULT true,
    started BOOLEAN DEFAULT false,
    position_played VARCHAR(20),
    jersey_number INTEGER,
    
    -- Basic statistics
    goals INTEGER DEFAULT 0 CHECK (goals >= 0),
    assists INTEGER DEFAULT 0 CHECK (assists >= 0),
    shots INTEGER DEFAULT 0 CHECK (shots >= 0),
    hits INTEGER DEFAULT 0 CHECK (hits >= 0),
    blocked_shots INTEGER DEFAULT 0 CHECK (blocked_shots >= 0),
    penalty_minutes INTEGER DEFAULT 0 CHECK (penalty_minutes >= 0),
    plus_minus INTEGER DEFAULT 0,
    
    -- Time on ice by situation (in seconds)
    toi_even_strength INTEGER DEFAULT 0,
    toi_power_play INTEGER DEFAULT 0,
    toi_short_handed INTEGER DEFAULT 0,
    toi_total INTEGER GENERATED ALWAYS AS (
        toi_even_strength + toi_power_play + toi_short_handed
    ) STORED,
    
    -- Shift tracking
    shifts INTEGER DEFAULT 0 CHECK (shifts >= 0),
    average_shift_length DECIMAL(4,2) GENERATED ALWAYS AS (
        CASE WHEN shifts > 0 THEN (toi_total::decimal / shifts) ELSE 0 END
    ) STORED,
    
    -- Goaltender game statistics
    decision VARCHAR(10) CHECK (decision IN ('W', 'L', 'OTL', 'T', 'ND')),
    saves INTEGER DEFAULT 0,
    shots_against INTEGER DEFAULT 0,
    goals_against INTEGER DEFAULT 0,
    
    -- Performance ratings (post-game evaluation)
    performance_rating INTEGER CHECK (performance_rating >= 1 AND performance_rating <= 10),
    
    -- Special achievements
    star_of_game INTEGER CHECK (star_of_game IN (1, 2, 3)),
    hat_trick BOOLEAN DEFAULT false,
    natural_hat_trick BOOLEAN DEFAULT false,
    gordie_howe_hat_trick BOOLEAN DEFAULT false,
    
    -- Metadata
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    -- Constraints
    CONSTRAINT player_game_stats_unique UNIQUE (player_id, game_id),
    CONSTRAINT goalie_decision_logic CHECK (
        (position_played = 'goaltender' AND decision IS NOT NULL) OR
        (position_played != 'goaltender' AND decision IS NULL)
    )
);

-- Convert to TimescaleDB hypertable
SELECT create_hypertable('player_game_statistics', 'created_at');
```

## Contract and Financial Relationships

### Player Contracts Table

Player contracts maintain comprehensive financial tracking including salary structures, performance bonuses, and contract clauses that affect player movement and salary cap calculations.

```sql
CREATE TABLE player_contracts (
    -- Primary identification
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    player_id UUID NOT NULL REFERENCES players(id) ON DELETE CASCADE,
    team_id UUID NOT NULL REFERENCES teams(id) ON DELETE CASCADE,
    
    -- Contract terms
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    contract_length INTEGER GENERATED ALWAYS AS (
        EXTRACT(year FROM end_date) - EXTRACT(year FROM start_date) + 1
    ) STORED,
    
    -- Financial terms (stored in cents for precision)
    annual_salary BIGINT NOT NULL CHECK (annual_salary >= 70000000), -- NHL minimum
    signing_bonus BIGINT DEFAULT 0 CHECK (signing_bonus >= 0),
    total_value BIGINT GENERATED ALWAYS AS (
        annual_salary * contract_length + signing_bonus
    ) STORED,
    
    -- Salary cap impact
    cap_hit BIGINT GENERATED ALWAYS AS (
        (annual_salary * contract_length + signing_bonus) / contract_length
    ) STORED,
    
    -- Contract clauses
    no_trade_clause BOOLEAN DEFAULT false,
    no_movement_clause BOOLEAN DEFAULT false,
    one_way_contract BOOLEAN DEFAULT true,
    two_way_salary BIGINT CHECK (
        (two_way_salary IS NULL AND one_way_contract = true) OR
        (two_way_salary IS NOT NULL AND one_way_contract = false)
    ),
    
    -- Performance bonuses
    performance_bonuses JSONB DEFAULT '[]',
    bonus_cushion BIGINT DEFAULT 0,
    
    -- Contract status
    status VARCHAR(20) DEFAULT 'active' CHECK (status IN (
        'active', 'expired', 'bought_out', 'terminated', 'suspended'
    )),
    is_entry_level BOOLEAN DEFAULT false,
    is_bridge_deal BOOLEAN DEFAULT false,
    
    -- Trade and movement restrictions
    trade_protection JSONB DEFAULT '{}',
    modified_no_trade_clause BOOLEAN DEFAULT false,
    
    -- Metadata
    signed_date DATE NOT NULL,
    signed_by UUID REFERENCES users(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    -- Constraints
    CONSTRAINT contract_date_logic CHECK (end_date > start_date),
    CONSTRAINT contract_length_limit CHECK (contract_length <= 8),
    CONSTRAINT entry_level_restrictions CHECK (
        (is_entry_level = false) OR 
        (is_entry_level = true AND contract_length <= 3 AND annual_salary <= 92500000)
    )
);
```

### Performance Bonuses Schema

Performance bonuses utilize detailed tracking for salary cap calculations and achievement validation while supporting complex bonus structures and payout scenarios.

```sql
CREATE TABLE player_contract_bonuses (
    -- Primary identification
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    contract_id UUID NOT NULL REFERENCES player_contracts(id) ON DELETE CASCADE,
    player_id UUID NOT NULL REFERENCES players(id) ON DELETE CASCADE,
    
    -- Bonus details
    bonus_type VARCHAR(30) NOT NULL CHECK (bonus_type IN (
        'games_played', 'goals', 'assists', 'points', 'plus_minus',
        'save_percentage', 'goals_against_average', 'wins',
        'all_star', 'awards', 'playoff_performance', 'team_performance'
    )),
    
    -- Achievement criteria
    threshold_value DECIMAL(10,3) NOT NULL,
    threshold_operator VARCHAR(5) DEFAULT '>=' CHECK (threshold_operator IN ('>=', '>', '=', '<', '<=')),
    measurement_period VARCHAR(20) DEFAULT 'season' CHECK (measurement_period IN (
        'season', 'calendar_year', 'playoff', 'specific_games'
    )),
    
    -- Financial impact
    bonus_amount BIGINT NOT NULL CHECK (bonus_amount > 0),
    cap_impact_year INTEGER, -- Which contract year the bonus affects cap
    
    -- Achievement tracking
    achieved BOOLEAN DEFAULT false,
    achievement_date DATE,
    actual_value DECIMAL(10,3),
    
    -- Validation and metadata
    season_applicable INTEGER NOT NULL,
    notes TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    -- Constraints
    CONSTRAINT bonus_reasonable_amount CHECK (bonus_amount <= 500000000), -- $5M max
    CONSTRAINT achievement_logic CHECK (
        (achieved = false AND achievement_date IS NULL) OR
        (achieved = true AND achievement_date IS NOT NULL)
    )
);
```

## Performance Optimization and Indexing

### Strategic Index Design

Database indexes optimize common query patterns including player searches, statistical analysis, and performance tracking while maintaining efficient write operations and reasonable storage overhead.

```sql
-- Primary lookup indexes
CREATE INDEX CONCURRENTLY idx_players_league_id ON players(league_id);
CREATE INDEX CONCURRENTLY idx_players_team_id ON players(current_team_id) 
WHERE current_team_id IS NOT NULL;
CREATE INDEX CONCURRENTLY idx_players_position ON players(position);

-- Search and filtering indexes
CREATE INDEX CONCURRENTLY idx_players_name ON players(last_name, first_name);
CREATE INDEX CONCURRENTLY idx_players_nationality ON players(nationality);
CREATE INDEX CONCURRENTLY idx_players_draft_year ON players(draft_year) 
WHERE draft_year IS NOT NULL;

-- Performance-based indexes
CREATE INDEX CONCURRENTLY idx_players_overall_rating ON players(overall_rating DESC);
CREATE INDEX CONCURRENTLY idx_players_age ON players(birth_date);

-- Contract status indexes
CREATE INDEX CONCURRENTLY idx_players_contract_status ON players(contract_status);
CREATE INDEX CONCURRENTLY idx_players_availability ON players(availability_status);

-- Composite indexes for common queries
CREATE INDEX CONCURRENTLY idx_players_league_position_rating 
ON players(league_id, position, overall_rating DESC);

CREATE INDEX CONCURRENTLY idx_players_team_position 
ON players(current_team_id, position) 
WHERE current_team_id IS NOT NULL;

-- Statistics table indexes
CREATE INDEX CONCURRENTLY idx_player_stats_player_season 
ON player_statistics(player_id, season DESC);

CREATE INDEX CONCURRENTLY idx_player_stats_league_season 
ON player_statistics(league_id, season DESC);

CREATE INDEX CONCURRENTLY idx_player_stats_team_season 
ON player_statistics(team_id, season DESC);

-- Performance leaderboard indexes
CREATE INDEX CONCURRENTLY idx_player_stats_goals 
ON player_statistics(league_id, season, goals DESC) 
WHERE games_played >= 10;

CREATE INDEX CONCURRENTLY idx_player_stats_points 
ON player_statistics(league_id, season, points DESC) 
WHERE games_played >= 10;

CREATE INDEX CONCURRENTLY idx_player_stats_gaa 
ON player_statistics(league_id, season, goals_against_average ASC) 
WHERE games_played >= 10 AND position = 'goaltender';

-- Game statistics indexes
CREATE INDEX CONCURRENTLY idx_player_game_stats_player 
ON player_game_statistics(player_id, created_at DESC);

CREATE INDEX CONCURRENTLY idx_player_game_stats_game 
ON player_game_statistics(game_id);

-- Contract indexes
CREATE INDEX CONCURRENTLY idx_player_contracts_player 
ON player_contracts(player_id, start_date DESC);

CREATE INDEX CONCURRENTLY idx_player_contracts_team_active 
ON player_contracts(team_id, status) 
WHERE status = 'active';

CREATE INDEX CONCURRENTLY idx_player_contracts_expiry 
ON player_contracts(end_date) 
WHERE status = 'active';

-- Bonus tracking indexes
CREATE INDEX CONCURRENTLY idx_contract_bonuses_contract 
ON player_contract_bonuses(contract_id, season_applicable);

CREATE INDEX CONCURRENTLY idx_contract_bonuses_achieved 
ON player_contract_bonuses(achieved, season_applicable) 
WHERE achieved = true;

-- JSONB indexes for flexible queries
CREATE INDEX CONCURRENTLY idx_players_ratings_gin ON players USING gin(ratings);
CREATE INDEX CONCURRENTLY idx_players_injury_gin ON players USING gin(injury_status);
CREATE INDEX CONCURRENTLY idx_contract_bonuses_gin ON player_contract_bonuses USING gin(performance_bonuses);
```

### Query Optimization Examples

Common query patterns demonstrate optimal database access methods while maintaining performance standards for typical hockey league management operations.

```sql
-- Efficient roster query with statistics
SELECT 
    p.id,
    p.first_name,
    p.last_name,
    p.position,
    p.jersey_number,
    p.overall_rating,
    ps.goals,
    ps.assists,
    ps.points,
    ps.games_played,
    pc.annual_salary,
    pc.cap_hit
FROM players p
LEFT JOIN player_statistics ps ON p.id = ps.player_id 
    AND ps.season = EXTRACT(year FROM CURRENT_DATE)
LEFT JOIN player_contracts pc ON p.id = pc.player_id 
    AND pc.status = 'active'
WHERE p.current_team_id = $1
ORDER BY p.position, p.jersey_number;

-- Efficient free agent search
SELECT 
    p.id,
    p.first_name,
    p.last_name,
    p.position,
    p.overall_rating,
    p.nationality,
    EXTRACT(year FROM age(p.birth_date)) as age,
    COALESCE(ps.points, 0) as last_season_points
FROM players p
LEFT JOIN player_statistics ps ON p.id = ps.player_id 
    AND ps.season = EXTRACT(year FROM CURRENT_DATE) - 1
WHERE p.contract_status IN ('unsigned', 'unrestricted_free_agent')
    AND p.league_id = $1
    AND p.overall_rating >= $2
    AND ($3 IS NULL OR p.position = $3)
ORDER BY p.overall_rating DESC, ps.points DESC NULLS LAST
LIMIT 50;

-- Contract expiration analysis
SELECT 
    t.name as team_name,
    p.first_name,
    p.last_name,
    p.position,
    pc.end_date,
    pc.annual_salary,
    pc.cap_hit,
    CASE 
        WHEN pc.no_trade_clause THEN 'NTC'
        WHEN pc.no_movement_clause THEN 'NMC'
        ELSE 'Tradeable'
    END as trade_status
FROM player_contracts pc
JOIN players p ON pc.player_id = p.id
JOIN teams t ON pc.team_id = t.id
WHERE pc.status = 'active'
    AND pc.end_date <= CURRENT_DATE + INTERVAL '1 year'
    AND t.league_id = $1
ORDER BY pc.end_date, pc.cap_hit DESC;

-- Statistical leaders query with minimum games threshold
WITH qualified_players AS (
    SELECT player_id
    FROM player_statistics
    WHERE season = $1 
        AND league_id = $2 
        AND games_played >= $3
)
SELECT 
    p.first_name,
    p.last_name,
    p.position,
    t.name as team_name,
    ps.goals,
    ps.assists,
    ps.points,
    ps.games_played,
    ROUND(ps.points::decimal / ps.games_played, 2) as points_per_game
FROM player_statistics ps
JOIN players p ON ps.player_id = p.id
JOIN teams t ON p.current_team_id = t.id
WHERE ps.season = $1 
    AND ps.league_id = $2
    AND ps.player_id IN (SELECT player_id FROM qualified_players)
ORDER BY ps.points DESC, ps.goals DESC
LIMIT 20;
```

## Business Rules and Constraints

### Hockey-Specific Validation Rules

Database constraints enforce hockey domain business rules including eligibility requirements, roster limitations, and competitive balance measures while maintaining data integrity and preventing invalid configurations.

```sql
-- Age-based eligibility constraints
CREATE OR REPLACE FUNCTION validate_player_eligibility()
RETURNS TRIGGER AS $$
BEGIN
    -- Junior league age restrictions
    IF NEW.league_id IN (SELECT id FROM leagues WHERE league_type = 'junior') THEN
        IF EXTRACT(year FROM age(NEW.birth_date)) > 21 THEN
            RAISE EXCEPTION 'Player too old for junior league (max age 21)';
        END IF;
    END IF;
    
    -- Professional league minimum age
    IF NEW.league_id IN (SELECT id FROM leagues WHERE league_type IN ('professional', 'farm')) THEN
        IF EXTRACT(year FROM age(NEW.birth_date)) < 18 THEN
            RAISE EXCEPTION 'Player too young for professional league (min age 18)';
        END IF;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER player_eligibility_check
    BEFORE INSERT OR UPDATE ON players
    FOR EACH ROW EXECUTE FUNCTION validate_player_eligibility();

-- Roster size enforcement
CREATE OR REPLACE FUNCTION enforce_roster_limits()
RETURNS TRIGGER AS $$
DECLARE
    current_roster_size INTEGER;
    max_roster_size INTEGER;
    position_count INTEGER;
    max_position_count INTEGER;
BEGIN
    -- Get team's maximum roster size
    SELECT COALESCE(max_roster_size, 23) INTO max_roster_size
    FROM teams 
    WHERE id = NEW.current_team_id;
    
    -- Count current roster size
    SELECT COUNT(*) INTO current_roster_size
    FROM players 
    WHERE current_team_id = NEW.current_team_id;
    
    -- Check roster size limit
    IF current_roster_size > max_roster_size THEN
        RAISE EXCEPTION 'Roster size limit exceeded (max %, current %)', 
            max_roster_size, current_roster_size;
    END IF;
    
    -- Position-specific limits (example: max 3 goaltenders)
    IF NEW.position = 'goaltender' THEN
        SELECT COUNT(*) INTO position_count
        FROM players 
        WHERE current_team_id = NEW.current_team_id 
            AND position = 'goaltender';
        
        IF position_count > 3 THEN
            RAISE EXCEPTION 'Too many goaltenders on roster (max 3)';
        END IF;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER roster_limits_check
    BEFORE INSERT OR UPDATE OF current_team_id ON players
    FOR EACH ROW 
    WHEN (NEW.current_team_id IS NOT NULL)
    EXECUTE FUNCTION enforce_roster_limits();

-- Jersey number uniqueness within team
CREATE OR REPLACE FUNCTION validate_jersey_number()
RETURNS TRIGGER AS $$
BEGIN
    -- Check for jersey number conflicts within team
    IF NEW.jersey_number IS NOT NULL AND NEW.current_team_id IS NOT NULL THEN
        IF EXISTS (
            SELECT 1 FROM players 
            WHERE current_team_id = NEW.current_team_id 
                AND jersey_number = NEW.jersey_number 
                AND id != COALESCE(NEW.id, '00000000-0000-0000-0000-000000000000'::uuid)
        ) THEN
            RAISE EXCEPTION 'Jersey number % already in use by team', NEW.jersey_number;
        END IF;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER jersey_number_validation
    BEFORE INSERT OR UPDATE ON players
    FOR EACH ROW EXECUTE FUNCTION validate_jersey_number();

-- Contract validation
CREATE OR REPLACE FUNCTION validate_player_contract()
RETURNS TRIGGER AS $$
DECLARE
    player_age INTEGER;
    team_salary_total BIGINT;
    team_salary_cap BIGINT;
BEGIN
    -- Get player age
    SELECT EXTRACT(year FROM age(birth_date)) INTO player_age
    FROM players WHERE id = NEW.player_id;
    
    -- Entry-level contract validation
    IF NEW.is_entry_level THEN
        IF player_age > 25 THEN
            RAISE EXCEPTION 'Player too old for entry-level contract (age %, max 25)', player_age;
        END IF;
        
        IF NEW.annual_salary > 92500000 THEN -- $925k
            RAISE EXCEPTION 'Entry-level contract salary too high (max $925,000)';
        END IF;
    END IF;
    
    -- Salary cap validation
    SELECT 
        COALESCE(SUM(pc.cap_hit), 0) + NEW.cap_hit,
        t.salary_cap
    INTO team_salary_total, team_salary_cap
    FROM teams t
    LEFT JOIN player_contracts pc ON t.id = pc.team_id AND pc.status = 'active'
    WHERE t.id = NEW.team_id
    GROUP BY t.salary_cap;
    
    IF team_salary_total > team_salary_cap THEN
        RAISE EXCEPTION 'Contract would exceed salary cap (total: %, cap: %)', 
            team_salary_total, team_salary_cap;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER contract_validation
    BEFORE INSERT OR UPDATE ON player_contracts
    FOR EACH ROW EXECUTE FUNCTION validate_player_contract();
```

### Data Integrity and Audit Trails

Comprehensive audit trails track all player-related changes while maintaining data integrity through systematic logging and validation procedures that support accountability and operational transparency.

```sql
-- Player audit trail
CREATE TABLE player_audit_log (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    player_id UUID NOT NULL,
    action VARCHAR(20) NOT NULL, -- INSERT, UPDATE, DELETE
    old_values JSONB,
    new_values JSONB,
    changed_by UUID REFERENCES users(id),
    changed_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    ip_address INET,
    user_agent TEXT
);

-- Audit trigger function
CREATE OR REPLACE FUNCTION player_audit_trigger()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'DELETE' THEN
        INSERT INTO player_audit_log (player_id, action, old_values)
        VALUES (OLD.id, 'DELETE', to_jsonb(OLD));
        RETURN OLD;
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO player_audit_log (player_id, action, old_values, new_values)
        VALUES (NEW.id, 'UPDATE', to_jsonb(OLD), to_jsonb(NEW));
        RETURN NEW;
    ELSIF TG_OP = 'INSERT' THEN
        INSERT INTO player_audit_log (player_id, action, new_values)
        VALUES (NEW.id, 'INSERT', to_jsonb(NEW));
        RETURN NEW;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- Create audit triggers
CREATE TRIGGER player_audit_trigger
    AFTER INSERT OR UPDATE OR DELETE ON players
    FOR EACH ROW EXECUTE FUNCTION player_audit_trigger();

CREATE TRIGGER contract_audit_trigger
    AFTER INSERT OR UPDATE OR DELETE ON player_contracts
    FOR EACH ROW EXECUTE FUNCTION player_audit_trigger();
```

This comprehensive players schema documentation provides the complete database foundation for realistic player management and accurate hockey simulation mechanics while maintaining optimal performance and data integrity throughout all system operations.