# Teams Schema Documentation - Hockey League Simulation System

This comprehensive teams schema documentation defines the database structures, relationships, and constraints that support team management within the Hockey League Simulation System. The schema encompasses team identity, organizational structure, financial management, facility information, and operational configurations that enable realistic hockey team operations across diverse league environments.

## Core Team Entity Structure

### Primary Teams Table

The teams table serves as the central entity for all team-related operations within the hockey league simulation system. This table maintains team identity, organizational relationships, and operational configurations while supporting multi-tenant isolation and comprehensive team management capabilities.

```sql
CREATE TABLE teams (
    -- Primary identification
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Team identity and branding
    name VARCHAR(255) NOT NULL,
    city VARCHAR(255) NOT NULL,
    abbreviation VARCHAR(10) NOT NULL,
    nickname VARCHAR(255), -- Optional alternative name
    
    -- Visual identity
    logo_url VARCHAR(500),
    primary_color VARCHAR(7), -- Hex color code (#RRGGBB)
    secondary_color VARCHAR(7),
    accent_color VARCHAR(7),
    
    -- Organizational hierarchy
    league_id UUID NOT NULL REFERENCES leagues(id) ON DELETE CASCADE,
    division_id UUID REFERENCES divisions(id) ON DELETE SET NULL,
    conference_id UUID REFERENCES conferences(id) ON DELETE SET NULL,
    parent_team_id UUID REFERENCES teams(id), -- For farm team relationships
    
    -- Historical information
    founded_year INTEGER CHECK (founded_year >= 1850 AND founded_year <= EXTRACT(year FROM CURRENT_DATE)),
    relocated_from VARCHAR(255), -- Previous city if relocated
    previous_names TEXT[], -- Array of historical team names
    
    -- Operational status
    active BOOLEAN DEFAULT true,
    ai_controlled BOOLEAN DEFAULT false,
    
    -- Financial tracking (amounts in cents for precision)
    current_salary_used BIGINT DEFAULT 0 CHECK (current_salary_used >= 0),
    salary_cap_limit BIGINT, -- Can override league default
    luxury_tax_threshold BIGINT,
    
    -- Configuration and preferences
    settings JSONB DEFAULT '{}',
    strategic_preferences JSONB DEFAULT '{}',
    
    -- Audit and tracking
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    created_by UUID REFERENCES users(id),
    
    -- Constraints
    CONSTRAINT unique_team_abbreviation_per_league UNIQUE (league_id, abbreviation),
    CONSTRAINT unique_team_name_per_league UNIQUE (league_id, name),
    CONSTRAINT valid_hex_colors CHECK (
        (primary_color IS NULL OR primary_color ~ '^#[0-9A-Fa-f]{6}$') AND
        (secondary_color IS NULL OR secondary_color ~ '^#[0-9A-Fa-f]{6}$') AND
        (accent_color IS NULL OR accent_color ~ '^#[0-9A-Fa-f]{6}$')
    )
);
```

### Team Arena and Facilities

The team arena information supports comprehensive facility management including capacity, location details, and amenity tracking that influences revenue generation and operational capabilities throughout league operations.

```sql
CREATE TABLE team_arenas (
    -- Primary identification
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    team_id UUID NOT NULL REFERENCES teams(id) ON DELETE CASCADE,
    
    -- Arena identity
    name VARCHAR(255) NOT NULL,
    nickname VARCHAR(255), -- Common name or informal designation
    
    -- Location information
    address TEXT,
    city VARCHAR(255) NOT NULL,
    state_province VARCHAR(100),
    country VARCHAR(100) NOT NULL DEFAULT 'Canada',
    postal_code VARCHAR(20),
    coordinates POINT, -- Geographic coordinates for mapping
    
    -- Capacity and seating
    total_capacity INTEGER NOT NULL CHECK (total_capacity > 0),
    hockey_capacity INTEGER CHECK (hockey_capacity <= total_capacity),
    standing_room INTEGER DEFAULT 0,
    luxury_suites INTEGER DEFAULT 0,
    club_seats INTEGER DEFAULT 0,
    
    -- Seating breakdown
    seating_configuration JSONB DEFAULT '{}', -- Detailed seating layout
    accessibility_seats INTEGER DEFAULT 0,
    
    -- Facility details
    ice_surface_dimensions JSONB DEFAULT '{}', -- Length, width, corner radius
    built_year INTEGER CHECK (built_year >= 1850),
    last_renovation_year INTEGER,
    
    -- Amenities and features
    amenities TEXT[], -- Array of available amenities
    parking_spaces INTEGER DEFAULT 0,
    public_transportation BOOLEAN DEFAULT false,
    wifi_available BOOLEAN DEFAULT true,
    
    -- Operational information
    surface_type VARCHAR(50) DEFAULT 'ice', -- For multi-purpose venues
    primary_tenant BOOLEAN DEFAULT true,
    shared_venue BOOLEAN DEFAULT false,
    
    -- Financial information (rental costs, revenue sharing)
    lease_terms JSONB DEFAULT '{}',
    revenue_sharing_agreement JSONB DEFAULT '{}',
    
    -- Audit tracking
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    -- Constraints
    CONSTRAINT valid_capacity_relationship CHECK (hockey_capacity <= total_capacity),
    CONSTRAINT valid_construction_years CHECK (
        last_renovation_year IS NULL OR 
        last_renovation_year >= built_year
    )
);
```

## Financial Management Schema

### Team Financial Records

Team financial tracking maintains comprehensive budget management, revenue tracking, and expense monitoring while supporting salary cap compliance and long-term financial planning throughout multiple seasons and operational periods.

```sql
CREATE TABLE team_financial_records (
    -- Primary identification
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    team_id UUID NOT NULL REFERENCES teams(id) ON DELETE CASCADE,
    
    -- Temporal scope
    season INTEGER NOT NULL,
    fiscal_year INTEGER, -- May differ from season year
    reporting_period DATE NOT NULL, -- Monthly, quarterly, or annual
    
    -- Revenue streams (all amounts in cents)
    revenue_tickets BIGINT DEFAULT 0 CHECK (revenue_tickets >= 0),
    revenue_concessions BIGINT DEFAULT 0 CHECK (revenue_concessions >= 0),
    revenue_merchandise BIGINT DEFAULT 0 CHECK (revenue_merchandise >= 0),
    revenue_sponsorship BIGINT DEFAULT 0 CHECK (revenue_sponsorship >= 0),
    revenue_broadcasting BIGINT DEFAULT 0 CHECK (revenue_broadcasting >= 0),
    revenue_parking BIGINT DEFAULT 0 CHECK (revenue_parking >= 0),
    revenue_other BIGINT DEFAULT 0 CHECK (revenue_other >= 0),
    
    -- Expense categories
    expense_player_salaries BIGINT DEFAULT 0 CHECK (expense_player_salaries >= 0),
    expense_coaching_staff BIGINT DEFAULT 0 CHECK (expense_coaching_staff >= 0),
    expense_arena_operations BIGINT DEFAULT 0 CHECK (expense_arena_operations >= 0),
    expense_travel BIGINT DEFAULT 0 CHECK (expense_travel >= 0),
    expense_equipment BIGINT DEFAULT 0 CHECK (expense_equipment >= 0),
    expense_marketing BIGINT DEFAULT 0 CHECK (expense_marketing >= 0),
    expense_administration BIGINT DEFAULT 0 CHECK (expense_administration >= 0),
    expense_player_development BIGINT DEFAULT 0 CHECK (expense_player_development >= 0),
    expense_facilities BIGINT DEFAULT 0 CHECK (expense_facilities >= 0),
    expense_other BIGINT DEFAULT 0 CHECK (expense_other >= 0),
    
    -- Financial calculations
    total_revenue BIGINT GENERATED ALWAYS AS (
        revenue_tickets + revenue_concessions + revenue_merchandise + 
        revenue_sponsorship + revenue_broadcasting + revenue_parking + revenue_other
    ) STORED,
    
    total_expenses BIGINT GENERATED ALWAYS AS (
        expense_player_salaries + expense_coaching_staff + expense_arena_operations +
        expense_travel + expense_equipment + expense_marketing + expense_administration +
        expense_player_development + expense_facilities + expense_other
    ) STORED,
    
    net_income BIGINT GENERATED ALWAYS AS (
        (revenue_tickets + revenue_concessions + revenue_merchandise + 
         revenue_sponsorship + revenue_broadcasting + revenue_parking + revenue_other) -
        (expense_player_salaries + expense_coaching_staff + expense_arena_operations +
         expense_travel + expense_equipment + expense_marketing + expense_administration +
         expense_player_development + expense_facilities + expense_other)
    ) STORED,
    
    -- Budget planning
    budget_allocated BIGINT,
    budget_remaining BIGINT,
    budget_variance BIGINT GENERATED ALWAYS AS (
        budget_allocated - 
        (expense_player_salaries + expense_coaching_staff + expense_arena_operations +
         expense_travel + expense_equipment + expense_marketing + expense_administration +
         expense_player_development + expense_facilities + expense_other)
    ) STORED,
    
    -- Audit and approval
    approved_by UUID REFERENCES users(id),
    approved_at TIMESTAMP WITH TIME ZONE,
    financial_status VARCHAR(20) DEFAULT 'draft' CHECK (
        financial_status IN ('draft', 'submitted', 'approved', 'rejected')
    ),
    
    -- Record management
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    -- Constraints
    CONSTRAINT unique_team_season_period UNIQUE (team_id, season, reporting_period),
    CONSTRAINT valid_budget_allocation CHECK (budget_allocated IS NULL OR budget_allocated >= 0)
);
```

### Salary Cap Tracking

Salary cap management requires detailed tracking of contract obligations, luxury tax calculations, and compliance monitoring while supporting complex NHL-style salary cap rules and exceptions throughout multiple seasons.

```sql
CREATE TABLE team_salary_cap_tracking (
    -- Primary identification
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    team_id UUID NOT NULL REFERENCES teams(id) ON DELETE CASCADE,
    
    -- Temporal information
    season INTEGER NOT NULL,
    calculation_date DATE NOT NULL DEFAULT CURRENT_DATE,
    
    -- Base salary calculations (all amounts in cents)
    base_salary_total BIGINT DEFAULT 0 CHECK (base_salary_total >= 0),
    performance_bonuses_earned BIGINT DEFAULT 0 CHECK (performance_bonuses_earned >= 0),
    performance_bonuses_potential BIGINT DEFAULT 0 CHECK (performance_bonuses_potential >= 0),
    
    -- Salary cap relief and adjustments
    ltir_relief BIGINT DEFAULT 0 CHECK (ltir_relief >= 0),
    retained_salary_obligations BIGINT DEFAULT 0 CHECK (retained_salary_obligations >= 0),
    buyout_cap_charges BIGINT DEFAULT 0 CHECK (buyout_cap_charges >= 0),
    
    -- Cap calculations
    total_cap_hit BIGINT GENERATED ALWAYS AS (
        base_salary_total + performance_bonuses_earned - ltir_relief + 
        retained_salary_obligations + buyout_cap_charges
    ) STORED,
    
    effective_cap_space BIGINT, -- Calculated based on league cap and relief
    
    -- Compliance status
    cap_compliant BOOLEAN GENERATED ALWAYS AS (
        total_cap_hit <= (SELECT salary_cap_amount FROM leagues WHERE id = (SELECT league_id FROM teams WHERE teams.id = team_id))
    ) STORED,
    
    cap_overage BIGINT GENERATED ALWAYS AS (
        GREATEST(0, total_cap_hit - (SELECT salary_cap_amount FROM leagues WHERE id = (SELECT league_id FROM teams WHERE teams.id = team_id)))
    ) STORED,
    
    -- Luxury tax calculations (if applicable)
    luxury_tax_owed BIGINT DEFAULT 0 CHECK (luxury_tax_owed >= 0),
    luxury_tax_bracket VARCHAR(20),
    
    -- Projections and planning
    projected_cap_hit_end_season BIGINT,
    deadline_cap_space BIGINT, -- Available space at trade deadline
    
    -- Audit and validation
    calculated_by UUID REFERENCES users(id),
    verified_by UUID REFERENCES users(id),
    verification_status VARCHAR(20) DEFAULT 'pending' CHECK (
        verification_status IN ('pending', 'verified', 'disputed', 'corrected')
    ),
    
    -- Record management
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    -- Constraints
    CONSTRAINT unique_team_season_date UNIQUE (team_id, season, calculation_date)
);
```

## Team Statistics and Performance

### Team Season Statistics

Comprehensive team performance tracking maintains statistical records across multiple seasons while supporting comparative analysis and performance evaluation throughout regular season and playoff competition.

```sql
CREATE TABLE team_season_statistics (
    -- Primary identification
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    team_id UUID NOT NULL REFERENCES teams(id) ON DELETE CASCADE,
    season INTEGER NOT NULL,
    
    -- Basic record information
    games_played INTEGER DEFAULT 0 CHECK (games_played >= 0),
    wins INTEGER DEFAULT 0 CHECK (wins >= 0),
    losses INTEGER DEFAULT 0 CHECK (losses >= 0),
    overtime_losses INTEGER DEFAULT 0 CHECK (overtime_losses >= 0),
    points INTEGER DEFAULT 0 CHECK (points >= 0),
    
    -- Goal statistics
    goals_for INTEGER DEFAULT 0 CHECK (goals_for >= 0),
    goals_against INTEGER DEFAULT 0 CHECK (goals_against >= 0),
    goal_differential INTEGER GENERATED ALWAYS AS (goals_for - goals_against) STORED,
    
    -- Advanced statistics
    shots_for INTEGER DEFAULT 0 CHECK (shots_for >= 0),
    shots_against INTEGER DEFAULT 0 CHECK (shots_against >= 0),
    shot_percentage DECIMAL(5,3) GENERATED ALWAYS AS (
        CASE 
            WHEN shots_for > 0 THEN ROUND((goals_for::DECIMAL / shots_for) * 100, 3)
            ELSE 0
        END
    ) STORED,
    save_percentage DECIMAL(5,3) GENERATED ALWAYS AS (
        CASE 
            WHEN shots_against > 0 THEN ROUND(((shots_against - goals_against)::DECIMAL / shots_against) * 100, 3)
            ELSE 0
        END
    ) STORED,
    
    -- Special teams statistics
    power_play_opportunities INTEGER DEFAULT 0 CHECK (power_play_opportunities >= 0),
    power_play_goals INTEGER DEFAULT 0 CHECK (power_play_goals >= 0),
    power_play_percentage DECIMAL(5,2) GENERATED ALWAYS AS (
        CASE 
            WHEN power_play_opportunities > 0 THEN ROUND((power_play_goals::DECIMAL / power_play_opportunities) * 100, 2)
            ELSE 0
        END
    ) STORED,
    
    penalty_kill_opportunities INTEGER DEFAULT 0 CHECK (penalty_kill_opportunities >= 0),
    penalty_kill_goals_allowed INTEGER DEFAULT 0 CHECK (penalty_kill_goals_allowed >= 0),
    penalty_kill_percentage DECIMAL(5,2) GENERATED ALWAYS AS (
        CASE 
            WHEN penalty_kill_opportunities > 0 THEN ROUND(((penalty_kill_opportunities - penalty_kill_goals_allowed)::DECIMAL / penalty_kill_opportunities) * 100, 2)
            ELSE 0
        END
    ) STORED,
    
    -- Physical play statistics
    penalty_minutes INTEGER DEFAULT 0 CHECK (penalty_minutes >= 0),
    hits INTEGER DEFAULT 0 CHECK (hits >= 0),
    blocked_shots INTEGER DEFAULT 0 CHECK (blocked_shots >= 0),
    
    -- Goaltending statistics
    shutouts INTEGER DEFAULT 0 CHECK (shutouts >= 0),
    goals_against_average DECIMAL(4,2) GENERATED ALWAYS AS (
        CASE 
            WHEN games_played > 0 THEN ROUND((goals_against::DECIMAL / games_played), 2)
            ELSE 0
        END
    ) STORED,
    
    -- Home and away splits
    home_wins INTEGER DEFAULT 0 CHECK (home_wins >= 0),
    home_losses INTEGER DEFAULT 0 CHECK (home_losses >= 0),
    home_overtime_losses INTEGER DEFAULT 0 CHECK (home_overtime_losses >= 0),
    away_wins INTEGER DEFAULT 0 CHECK (away_wins >= 0),
    away_losses INTEGER DEFAULT 0 CHECK (away_losses >= 0),
    away_overtime_losses INTEGER DEFAULT 0 CHECK (away_overtime_losses >= 0),
    
    -- Divisional and conference records
    division_wins INTEGER DEFAULT 0 CHECK (division_wins >= 0),
    division_losses INTEGER DEFAULT 0 CHECK (division_losses >= 0),
    conference_wins INTEGER DEFAULT 0 CHECK (conference_wins >= 0),
    conference_losses INTEGER DEFAULT 0 CHECK (conference_losses >= 0),
    
    -- Streaks and trends
    current_streak VARCHAR(10), -- Format: "W5", "L3", "OT2"
    longest_win_streak INTEGER DEFAULT 0 CHECK (longest_win_streak >= 0),
    longest_losing_streak INTEGER DEFAULT 0 CHECK (longest_losing_streak >= 0),
    
    -- Advanced metrics (calculated periodically)
    strength_of_schedule DECIMAL(5,3),
    strength_of_victory DECIMAL(5,3),
    pythagorean_wins DECIMAL(5,2), -- Expected wins based on goal differential
    
    -- Record management
    last_game_date DATE,
    next_game_date DATE,
    statistics_current_as_of TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    -- Constraints
    CONSTRAINT unique_team_season_stats UNIQUE (team_id, season),
    CONSTRAINT valid_record_totals CHECK (games_played = wins + losses + overtime_losses),
    CONSTRAINT valid_home_away_totals CHECK (
        home_wins + home_losses + home_overtime_losses + 
        away_wins + away_losses + away_overtime_losses = games_played
    ),
    CONSTRAINT valid_special_teams CHECK (
        power_play_goals <= power_play_opportunities AND
        penalty_kill_goals_allowed <= penalty_kill_opportunities
    )
);
```

## Team Configuration and Settings

### Team Strategies and Preferences

Team strategy configuration enables comprehensive tactical setup and AI behavior customization while supporting different coaching philosophies and strategic approaches throughout competitive play and simulation activities.

```sql
CREATE TABLE team_strategies (
    -- Primary identification
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    team_id UUID NOT NULL REFERENCES teams(id) ON DELETE CASCADE,
    
    -- Strategy identification
    strategy_name VARCHAR(100) NOT NULL,
    strategy_type VARCHAR(50) NOT NULL CHECK (
        strategy_type IN ('even_strength', 'power_play', 'penalty_kill', 'overtime', 'general')
    ),
    
    -- Tactical preferences
    offensive_style VARCHAR(20) DEFAULT 'balanced' CHECK (
        offensive_style IN ('conservative', 'balanced', 'aggressive', 'high_tempo')
    ),
    defensive_style VARCHAR(20) DEFAULT 'balanced' CHECK (
        defensive_style IN ('defensive', 'balanced', 'offensive', 'run_and_gun')
    ),
    forechecking_intensity VARCHAR(20) DEFAULT 'medium' CHECK (
        forechecking_intensity IN ('passive', 'low', 'medium', 'high', 'aggressive')
    ),
    
    -- Line deployment preferences
    line_matching_priority VARCHAR(20) DEFAULT 'balanced' CHECK (
        line_matching_priority IN ('offense', 'defense', 'balanced', 'special_teams')
    ),
    power_play_emphasis VARCHAR(20) DEFAULT 'balanced' CHECK (
        power_play_emphasis IN ('movement', 'shots', 'traffic', 'balanced')
    ),
    penalty_kill_style VARCHAR(20) DEFAULT 'passive' CHECK (
        penalty_kill_style IN ('passive', 'aggressive', 'neutral_zone_trap', 'box')
    ),
    
    -- Goaltender management
    goaltender_usage VARCHAR(20) DEFAULT 'balanced' CHECK (
        goaltender_usage IN ('workhorse', 'balanced', 'platoon', 'matchup_based')
    ),
    backup_usage_percentage INTEGER DEFAULT 25 CHECK (
        backup_usage_percentage >= 0 AND backup_usage_percentage <= 50
    ),
    
    -- Risk tolerance and decision making
    risk_tolerance VARCHAR(20) DEFAULT 'medium' CHECK (
        risk_tolerance IN ('conservative', 'low', 'medium', 'high', 'aggressive')
    ),
    timeout_usage VARCHAR(20) DEFAULT 'strategic' CHECK (
        timeout_usage IN ('conservative', 'strategic', 'aggressive')
    ),
    challenge_usage VARCHAR(20) DEFAULT 'selective' CHECK (
        challenge_usage IN ('never', 'conservative', 'selective', 'aggressive')
    ),
    
    -- AI-specific configurations (for AI-controlled teams)
    ai_difficulty VARCHAR(20) CHECK (
        ai_difficulty IN ('easy', 'medium', 'hard', 'expert', 'adaptive')
    ),
    ai_trading_aggressiveness VARCHAR(20) DEFAULT 'medium' CHECK (
        ai_trading_aggressiveness IN ('passive', 'conservative', 'medium', 'aggressive', 'very_aggressive')
    ),
    ai_development_focus VARCHAR(20) DEFAULT 'balanced' CHECK (
        ai_development_focus IN ('prospects', 'veterans', 'balanced', 'win_now')
    ),
    
    -- Situational adjustments
    trailing_strategy JSONB DEFAULT '{}', -- Strategies when behind in games
    leading_strategy JSONB DEFAULT '{}', -- Strategies when ahead in games
    close_game_strategy JSONB DEFAULT '{}', -- Strategies for one-goal games
    
    -- Effectiveness tracking
    games_used INTEGER DEFAULT 0,
    wins_with_strategy INTEGER DEFAULT 0,
    losses_with_strategy INTEGER DEFAULT 0,
    goals_for_average DECIMAL(4,2),
    goals_against_average DECIMAL(4,2),
    
    -- Temporal information
    active BOOLEAN DEFAULT true,
    implemented_date DATE DEFAULT CURRENT_DATE,
    last_modified_date DATE DEFAULT CURRENT_DATE,
    created_by UUID REFERENCES users(id),
    
    -- Record management
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    -- Constraints
    CONSTRAINT unique_team_strategy_type UNIQUE (team_id, strategy_type, strategy_name)
);
```

## Indexes and Performance Optimization

### Strategic Indexing for Team Operations

Comprehensive indexing strategy optimizes common team-related queries while supporting real-time operations and analytical workloads throughout the hockey league simulation system.

```sql
-- Primary lookup indexes
CREATE INDEX CONCURRENTLY idx_teams_league_id ON teams(league_id);
CREATE INDEX CONCURRENTLY idx_teams_division_id ON teams(division_id) WHERE division_id IS NOT NULL;
CREATE INDEX CONCURRENTLY idx_teams_active ON teams(active) WHERE active = true;
CREATE INDEX CONCURRENTLY idx_teams_ai_controlled ON teams(ai_controlled);

-- Team identity and searching
CREATE INDEX CONCURRENTLY idx_teams_name_trgm ON teams USING gin(name gin_trgm_ops);
CREATE INDEX CONCURRENTLY idx_teams_city_trgm ON teams USING gin(city gin_trgm_ops);
CREATE INDEX CONCURRENTLY idx_teams_abbreviation ON teams(abbreviation);

-- Financial tracking indexes
CREATE INDEX CONCURRENTLY idx_team_financial_records_team_season 
ON team_financial_records(team_id, season);
CREATE INDEX CONCURRENTLY idx_team_financial_records_reporting_period 
ON team_financial_records(reporting_period);
CREATE INDEX CONCURRENTLY idx_team_financial_records_fiscal_year 
ON team_financial_records(fiscal_year) WHERE fiscal_year IS NOT NULL;

-- Salary cap tracking indexes
CREATE INDEX CONCURRENTLY idx_salary_cap_tracking_team_season 
ON team_salary_cap_tracking(team_id, season);
CREATE INDEX CONCURRENTLY idx_salary_cap_tracking_calculation_date 
ON team_salary_cap_tracking(calculation_date);
CREATE INDEX CONCURRENTLY idx_salary_cap_tracking_compliance 
ON team_salary_cap_tracking(cap_compliant) WHERE cap_compliant = false;

-- Statistics indexes
CREATE INDEX CONCURRENTLY idx_team_season_stats_season 
ON team_season_statistics(season);
CREATE INDEX CONCURRENTLY idx_team_season_stats_team_season 
ON team_season_statistics(team_id, season);
CREATE INDEX CONCURRENTLY idx_team_season_stats_points 
ON team_season_statistics(season, points DESC);

-- Arena information indexes
CREATE INDEX CONCURRENTLY idx_team_arenas_team_id ON team_arenas(team_id);
CREATE INDEX CONCURRENTLY idx_team_arenas_city ON team_arenas(city);
CREATE INDEX CONCURRENTLY idx_team_arenas_capacity ON team_arenas(total_capacity DESC);

-- Strategy and configuration indexes
CREATE INDEX CONCURRENTLY idx_team_strategies_team_type 
ON team_strategies(team_id, strategy_type);
CREATE INDEX CONCURRENTLY idx_team_strategies_active 
ON team_strategies(active) WHERE active = true;

-- Composite indexes for common query patterns
CREATE INDEX CONCURRENTLY idx_teams_league_division 
ON teams(league_id, division_id) WHERE active = true;
CREATE INDEX CONCURRENTLY idx_financial_records_team_season_period 
ON team_financial_records(team_id, season, reporting_period);
```

## Business Rules and Constraints

### Data Integrity and Validation Rules

Comprehensive business rule enforcement ensures data consistency and hockey domain accuracy while preventing invalid data combinations and maintaining referential integrity throughout all team-related operations.

```sql
-- Team name and abbreviation uniqueness within leagues
ALTER TABLE teams ADD CONSTRAINT enforce_unique_names_per_league 
CHECK (
    NOT EXISTS (
        SELECT 1 FROM teams t2 
        WHERE t2.league_id = teams.league_id 
        AND t2.id != teams.id 
        AND (LOWER(t2.name) = LOWER(teams.name) OR LOWER(t2.abbreviation) = LOWER(teams.abbreviation))
    )
);

-- Salary cap compliance validation
ALTER TABLE team_salary_cap_tracking ADD CONSTRAINT validate_cap_calculations
CHECK (
    base_salary_total >= 0 AND
    performance_bonuses_earned >= 0 AND
    ltir_relief >= 0 AND
    total_cap_hit >= 0
);

-- Financial record validation
ALTER TABLE team_financial_records ADD CONSTRAINT validate_financial_totals
CHECK (
    total_revenue >= 0 AND
    total_expenses >= 0 AND
    (budget_allocated IS NULL OR budget_allocated >= 0)
);

-- Arena capacity validation
ALTER TABLE team_arenas ADD CONSTRAINT validate_arena_capacity
CHECK (
    total_capacity > 0 AND
    (hockey_capacity IS NULL OR hockey_capacity <= total_capacity) AND
    (luxury_suites IS NULL OR luxury_suites >= 0) AND
    (club_seats IS NULL OR club_seats >= 0)
);

-- Statistics validation
ALTER TABLE team_season_statistics ADD CONSTRAINT validate_team_statistics
CHECK (
    games_played >= 0 AND
    wins >= 0 AND losses >= 0 AND overtime_losses >= 0 AND
    games_played = wins + losses + overtime_losses AND
    goals_for >= 0 AND goals_against >= 0 AND
    shots_for >= goals_for AND shots_against >= goals_against
);

-- Strategy configuration validation
ALTER TABLE team_strategies ADD CONSTRAINT validate_strategy_percentages
CHECK (
    backup_usage_percentage >= 0 AND backup_usage_percentage <= 50
);
```

## Sample Data and Usage Examples

### Realistic Team Data Examples

Example team configurations demonstrate proper schema utilization while providing realistic hockey scenarios for development and testing purposes throughout the system implementation.

```sql
-- Sample NHL-style team
INSERT INTO teams (
    name, city, abbreviation, league_id, division_id,
    primary_color, secondary_color, founded_year,
    salary_cap_limit, current_salary_used, ai_controlled,
    settings, strategic_preferences
) VALUES (
    'Maple Leafs', 'Toronto', 'TOR', 'nhl-league-id', 'atlantic-division-id',
    '#003E7E', '#FFFFFF', 1917,
    8500000000, -- $85M salary cap in cents
    7850000000, -- $78.5M currently used
    false,
    '{"timezone": "America/Toronto", "currency": "CAD"}',
    '{"style": "offensive", "development_focus": "balanced"}'
);

-- Sample arena information
INSERT INTO team_arenas (
    team_id, name, city, country,
    total_capacity, hockey_capacity, luxury_suites, club_seats,
    built_year, amenities, parking_spaces,
    seating_configuration
) VALUES (
    (SELECT id FROM teams WHERE abbreviation = 'TOR'),
    'Scotiabank Arena', 'Toronto', 'Canada',
    19800, 18800, 68, 1800,
    1999, 
    ARRAY['wifi', 'mobile_app', 'premium_dining', 'retail_shops'],
    500,
    '{
        "lower_bowl": 8000,
        "upper_bowl": 9000,
        "premium_seating": 1800
    }'
);

-- Sample financial record
INSERT INTO team_financial_records (
    team_id, season, reporting_period,
    revenue_tickets, revenue_concessions, revenue_merchandise,
    revenue_sponsorship, revenue_broadcasting,
    expense_player_salaries, expense_arena_operations,
    expense_travel, expense_equipment,
    budget_allocated
) VALUES (
    (SELECT id FROM teams WHERE abbreviation = 'TOR'),
    2024, '2024-06-30',
    12500000000, -- $125M ticket revenue
    2800000000,  -- $28M concessions
    1500000000,  -- $15M merchandise
    3500000000,  -- $35M sponsorship
    8000000000,  -- $80M broadcasting
    7850000000,  -- $78.5M player salaries
    1200000000,  -- $12M arena operations
    300000000,   -- $3M travel
    150000000,   -- $1.5M equipment
    9000000000   -- $90M total budget
);

-- Sample team statistics
INSERT INTO team_season_statistics (
    team_id, season, games_played, wins, losses, overtime_losses,
    goals_for, goals_against, shots_for, shots_against,
    power_play_opportunities, power_play_goals,
    penalty_kill_opportunities, penalty_kill_goals_allowed,
    home_wins, home_losses, away_wins, away_losses
) VALUES (
    (SELECT id FROM teams WHERE abbreviation = 'TOR'),
    2024, 82, 50, 25, 7,
    265, 220, 2650, 2380,
    245, 52, 198, 35,
    28, 10, 22, 15
);
```

### Common Query Patterns

Frequently used queries demonstrate optimal database access patterns while supporting typical application operations and administrative functions throughout team management workflows.

```sql
-- Get complete team information with arena details
SELECT 
    t.*,
    ta.name as arena_name,
    ta.total_capacity,
    ta.city as arena_city
FROM teams t
LEFT JOIN team_arenas ta ON t.id = ta.team_id
WHERE t.league_id = $1 AND t.active = true
ORDER BY t.name;

-- Calculate current salary cap status
SELECT 
    t.name,
    t.abbreviation,
    scr.total_cap_hit,
    scr.effective_cap_space,
    scr.cap_compliant,
    scr.cap_overage
FROM teams t
JOIN team_salary_cap_tracking scr ON t.id = scr.team_id
WHERE scr.season = $1 
  AND scr.calculation_date = (
    SELECT MAX(calculation_date) 
    FROM team_salary_cap_tracking 
    WHERE team_id = t.id AND season = $1
  );

-- Get league standings
SELECT 
    t.name,
    t.abbreviation,
    tss.games_played,
    tss.wins,
    tss.losses,
    tss.overtime_losses,
    tss.points,
    tss.goals_for,
    tss.goals_against,
    tss.goal_differential,
    RANK() OVER (ORDER BY tss.points DESC, tss.wins DESC, tss.goal_differential DESC) as standings_rank
FROM teams t
JOIN team_season_statistics tss ON t.id = tss.team_id
WHERE tss.season = $1 AND t.league_id = $2
ORDER BY tss.points DESC, tss.wins DESC, tss.goal_differential DESC;

-- Financial performance analysis
SELECT 
    t.name,
    tfr.total_revenue,
    tfr.total_expenses,
    tfr.net_income,
    tfr.budget_variance,
    ROUND((tfr.total_revenue::DECIMAL / tfr.budget_allocated * 100), 2) as budget_utilization_pct
FROM teams t
JOIN team_financial_records tfr ON t.id = tfr.team_id
WHERE tfr.season = $1
ORDER BY tfr.net_income DESC;
```

This comprehensive teams schema documentation provides the foundation for effective team management while ensuring data integrity, performance optimization, and hockey domain accuracy throughout all team-related operations within the Hockey League Simulation System.