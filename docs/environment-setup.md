# Environment Setup Guide - Hockey League Simulation System

This comprehensive environment setup guide provides step-by-step instructions for establishing a complete local development environment for the Hockey League Simulation System. The setup procedures ensure consistent development experiences while providing all necessary tools and services for effective AI-assisted development and hockey domain functionality testing.

## Prerequisites and System Requirements

### Required Software Dependencies

Local development requires specific software versions and system configurations that support modern web development workflows, database operations, and real-time communication capabilities. The prerequisite software provides the foundation for effective development while ensuring compatibility with production deployment environments.

**Node.js and Package Management:**
- Node.js version 18.x or higher (LTS recommended)
- npm version 9.x or higher (included with Node.js)
- Optional: yarn version 1.22.x or higher as alternative package manager

**Database and Caching Services:**
- Docker Desktop version 4.x or higher for containerized services
- PostgreSQL 14.x or higher (can be installed via Docker)
- Redis 7.x or higher (can be installed via Docker)
- TimescaleDB extension for PostgreSQL (included in Docker setup)

**Development Tools:**
- Git version 2.30 or higher for version control
- VS Code with recommended extensions for optimal AI assistance
- Chrome or Firefox for debugging and testing
- Postman or similar API testing tool (optional but recommended)

### Operating System Compatibility

The development environment supports major operating systems with specific configuration considerations for each platform. Platform-specific instructions ensure optimal performance and compatibility while maintaining consistent development workflows across different development machines and team configurations.

**Windows 10/11:**
- Windows Subsystem for Linux (WSL2) recommended for optimal Docker performance
- PowerShell 7.x or Git Bash for command-line operations
- Windows Terminal for enhanced development experience

**macOS 10.15 or higher:**
- Homebrew package manager for dependency installation
- Xcode Command Line Tools for native compilation support
- Docker Desktop for Mac with appropriate resource allocation

**Linux (Ubuntu 20.04+ or equivalent):**
- Docker and Docker Compose for containerized services
- Build essentials for native module compilation
- Appropriate permissions for Docker usage without sudo

## Initial Project Setup

### Repository Cloning and Basic Configuration

Project initialization begins with repository cloning and basic configuration establishment that prepares the development environment for effective local development and AI-assisted coding workflows with appropriate tooling and service integration.

```bash
# Clone the repository
git clone https://github.com/your-organization/hockey-league-simulation.git
cd hockey-league-simulation

# Verify Node.js version
node --version  # Should be 18.x or higher
npm --version   # Should be 9.x or higher

# Install project dependencies
npm install

# Verify installation
npm list --depth=0
```

### Environment Variables Configuration

Environment configuration establishes the necessary parameters for local development including database connections, service integrations, and development-specific settings that enable effective local testing while maintaining security and operational flexibility.

```bash
# Copy environment template
cp .env.example .env.local

# Edit environment variables for local development
# Use your preferred text editor
code .env.local  # VS Code
# OR
nano .env.local  # Terminal editor
```

**Local Environment Variables (.env.local):**
```bash
# Application Configuration
NODE_ENV=development
PORT=3000
APP_NAME="Hockey League Simulation - Local"
APP_URL=http://localhost:3000

# Database Configuration
DATABASE_URL=postgresql://hockey_user:hockey_password@localhost:5432/hockey_simulation_dev
DATABASE_POOL_MIN=2
DATABASE_POOL_MAX=10
DATABASE_SSL=false

# Redis Configuration
REDIS_URL=redis://localhost:6379/0
REDIS_PASSWORD=

# Security Configuration (Development - Less Strict)
JWT_SECRET=your_local_jwt_secret_key_here
JWT_EXPIRES_IN=24h
BCRYPT_ROUNDS=10
SESSION_SECRET=your_local_session_secret_here

# External Services (Development/Mock)
STRIPE_PUBLISHABLE_KEY=pk_test_your_stripe_test_key
STRIPE_SECRET_KEY=sk_test_your_stripe_test_key
EMAIL_SERVICE=mock
EMAIL_FROM=dev@localhost

# Development Features
ENABLE_DEBUG_LOGGING=true
ENABLE_HOT_RELOAD=true
MOCK_EXTERNAL_SERVICES=true
AUTO_SEED_DATABASE=true
ENABLE_GRAPHQL_PLAYGROUND=true

# AI Development Tools
ENABLE_AI_SUGGESTIONS=true
LOG_AI_INTERACTIONS=true
```

## Database Setup and Configuration

### Docker-Based Database Services

Database services utilize Docker containers to provide consistent development environments with appropriate seeding and configuration that enables immediate development productivity while maintaining production compatibility and testing reliability.

```yaml
# docker-compose.yml for local development
version: '3.8'

services:
  postgres:
    image: timescale/timescaledb:latest-pg14
    container_name: hockey_postgres_dev
    environment:
      POSTGRES_DB: hockey_simulation_dev
      POSTGRES_USER: hockey_user
      POSTGRES_PASSWORD: hockey_password
      POSTGRES_INITDB_ARGS: "--auth-host=scram-sha-256"
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init-db.sql:/docker-entrypoint-initdb.d/01-init.sql
      - ./scripts/seed-dev-data.sql:/docker-entrypoint-initdb.d/02-seed.sql
    networks:
      - hockey_network
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    container_name: hockey_redis_dev
    command: redis-server --appendonly yes
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - hockey_network
    restart: unless-stopped

  # Optional: Redis Commander for GUI management
  redis-commander:
    image: rediscommander/redis-commander:latest
    container_name: hockey_redis_commander
    environment:
      REDIS_HOSTS: local:redis:6379
    ports:
      - "8081:8081"
    networks:
      - hockey_network
    depends_on:
      - redis

volumes:
  postgres_data:
  redis_data:

networks:
  hockey_network:
    driver: bridge
```

### Database Initialization and Seeding

Database initialization includes comprehensive schema creation, extension setup, and development data seeding that provides realistic hockey scenarios for testing and development while enabling immediate feature development and validation activities.

```bash
# Start database services
docker-compose up -d postgres redis

# Wait for services to be ready
echo "Waiting for PostgreSQL to be ready..."
until docker exec hockey_postgres_dev pg_isready -U hockey_user; do
  sleep 2
done

echo "Waiting for Redis to be ready..."
until docker exec hockey_redis_dev redis-cli ping; do
  sleep 2
done

# Run database migrations
npm run db:migrate

# Seed development data
npm run db:seed:dev

# Verify database setup
npm run db:status
```

**Database Initialization Script (scripts/init-db.sql):**
```sql
-- Enable required extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "timescaledb";
CREATE EXTENSION IF NOT EXISTS "pg_stat_statements";

-- Create additional development users
CREATE USER hockey_dev WITH PASSWORD 'dev_password';
GRANT ALL PRIVILEGES ON DATABASE hockey_simulation_dev TO hockey_dev;

-- Set up development-specific configurations
ALTER DATABASE hockey_simulation_dev SET timezone = 'UTC';
ALTER DATABASE hockey_simulation_dev SET log_statement = 'all';
```

## Application Development Server Setup

### Frontend Development Configuration

Frontend development server provides hot reloading, debugging capabilities, and proxy configuration that enables effective React development with real-time updates and comprehensive development tooling integration while maintaining optimal performance and debugging capabilities.

```bash
# Install frontend dependencies
cd frontend
npm install

# Start development server with hot reloading
npm run dev

# Alternative: Start with debugging enabled
npm run dev:debug

# Build for development testing
npm run build:dev
```

**Frontend Development Configuration (frontend/vite.config.ts):**
```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  
  // Development server configuration
  server: {
    port: 3000,
    host: true,
    proxy: {
      '/api': {
        target: 'http://localhost:3001',
        changeOrigin: true,
        secure: false
      },
      '/ws': {
        target: 'ws://localhost:3001',
        ws: true
      }
    }
  },

  // Path resolution for clean imports
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@features': path.resolve(__dirname, './src/features'),
      '@services': path.resolve(__dirname, './src/services'),
      '@utils': path.resolve(__dirname, './src/utils'),
      '@types': path.resolve(__dirname, './src/types')
    }
  },

  // Environment variables
  define: {
    __APP_VERSION__: JSON.stringify(process.env.npm_package_version),
    __API_URL__: JSON.stringify(process.env.VITE_API_URL || 'http://localhost:3001')
  },

  // Development optimizations
  optimizeDeps: {
    include: ['react', 'react-dom', 'react-router-dom']
  }
});
```

### Backend Development Server Configuration

Backend development server includes comprehensive API functionality, WebSocket support, and development middleware that enables effective server-side development with debugging capabilities and real-time development feedback while maintaining production compatibility.

```bash
# Start backend development server
cd backend
npm install
npm run dev

# Alternative: Start with debugger attached
npm run dev:debug

# Run with specific environment
npm run dev:local
```

**Backend Development Configuration (backend/src/server.ts):**
```typescript
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import compression from 'compression';
import { createServer } from 'http';
import { Server as SocketIOServer } from 'socket.io';

const app = express();
const server = createServer(app);

// Development middleware
if (process.env.NODE_ENV === 'development') {
  // Enhanced logging for development
  app.use((req, res, next) => {
    console.log(`${new Date().toISOString()} ${req.method} ${req.path}`);
    next();
  });

  // CORS configuration for development
  app.use(cors({
    origin: ['http://localhost:3000', 'http://localhost:3001'],
    credentials: true
  }));
}

// Security middleware (relaxed for development)
app.use(helmet({
  contentSecurityPolicy: process.env.NODE_ENV === 'production'
}));

app.use(compression());
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true }));

// WebSocket setup for real-time features
const io = new SocketIOServer(server, {
  cors: {
    origin: process.env.NODE_ENV === 'development' 
      ? ['http://localhost:3000'] 
      : process.env.ALLOWED_ORIGINS?.split(','),
    credentials: true
  }
});

// Development routes
if (process.env.NODE_ENV === 'development') {
  app.get('/dev/health', (req, res) => {
    res.json({
      status: 'ok',
      environment: 'development',
      timestamp: new Date().toISOString(),
      services: {
        database: 'connected',
        redis: 'connected'
      }
    });
  });
}

const PORT = process.env.PORT || 3001;
server.listen(PORT, () => {
  console.log(`🏒 Hockey League Simulation API running on port ${PORT}`);
  console.log(`📊 Environment: ${process.env.NODE_ENV}`);
  console.log(`🔗 API URL: http://localhost:${PORT}`);
});
```

## Development Tools and VS Code Configuration

### VS Code Extensions and Settings

VS Code configuration includes essential extensions and settings that optimize the development experience for AI-assisted development, TypeScript support, and hockey domain development while providing comprehensive debugging and productivity enhancement capabilities.

**Required VS Code Extensions (.vscode/extensions.json):**
```json
{
  "recommendations": [
    "ms-vscode.vscode-typescript-next",
    "bradlc.vscode-tailwindcss",
    "esbenp.prettier-vscode",
    "dbaeumer.vscode-eslint",
    "ms-vscode.vscode-json",
    "github.copilot",
    "github.copilot-chat",
    "ms-vscode.vscode-docker",
    "ms-vscode.vscode-postgresql",
    "humao.rest-client",
    "streetsidesoftware.code-spell-checker"
  ]
}
```

**VS Code Workspace Settings (.vscode/settings.json):**
```json
{
  "typescript.preferences.includePackageJsonAutoImports": "auto",
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.organizeImports": true
  },
  "files.exclude": {
    "**/node_modules": true,
    "**/.git": true,
    "**/dist": true,
    "**/build": true
  },
  "search.exclude": {
    "**/node_modules": true,
    "**/dist": true,
    "**/build": true
  },
  "github.copilot.enable": {
    "*": true,
    "yaml": true,
    "plaintext": false,
    "markdown": true
  },
  "github.copilot.inlineSuggest.enable": true
}
```

### Debugging Configuration

Debugging setup provides comprehensive debugging capabilities for both frontend and backend development with appropriate breakpoint support and inspection capabilities that enable effective troubleshooting and development workflow optimization throughout feature development activities.

**Debug Configuration (.vscode/launch.json):**
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Launch React App",
      "type": "node",
      "request": "launch",
      "cwd": "${workspaceFolder}/frontend",
      "runtimeExecutable": "npm",
      "runtimeArgs": ["run", "dev"]
    },
    {
      "name": "Launch Backend API",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/backend/src/server.ts",
      "outFiles": ["${workspaceFolder}/backend/dist/**/*.js"],
      "env": {
        "NODE_ENV": "development"
      },
      "envFile": "${workspaceFolder}/.env.local",
      "sourceMaps": true,
      "restart": true,
      "protocol": "inspector",
      "console": "integratedTerminal"
    },
    {
      "name": "Attach to Backend",
      "type": "node",
      "request": "attach",
      "port": 9229,
      "restart": true,
      "localRoot": "${workspaceFolder}/backend",
      "remoteRoot": "/app"
    }
  ],
  "compounds": [
    {
      "name": "Launch Full Stack",
      "configurations": ["Launch React App", "Launch Backend API"]
    }
  ]
}
```

## Testing Environment Setup

### Test Database and Service Configuration

Testing environment includes isolated database services and comprehensive test data management that enables effective unit testing, integration testing, and end-to-end testing while maintaining test isolation and reproducible test conditions throughout development activities.

```bash
# Create test database
createdb hockey_simulation_test

# Set up test environment variables
cp .env.local .env.test

# Update test-specific variables in .env.test
sed -i 's/hockey_simulation_dev/hockey_simulation_test/g' .env.test

# Run test setup
npm run test:setup

# Execute test suites
npm run test          # Unit tests
npm run test:integration  # Integration tests
npm run test:e2e     # End-to-end tests
```

**Test Configuration (jest.config.js):**
```javascript
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src', '<rootDir>/tests'],
  
  // Test file patterns
  testMatch: [
    '**/__tests__/**/*.ts',
    '**/?(*.)+(spec|test).ts'
  ],
  
  // Coverage configuration
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/types/**/*',
    '!src/**/index.ts'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 90,
      lines: 90,
      statements: 90
    }
  },
  
  // Test environment setup
  setupFilesAfterEnv: ['<rootDir>/tests/setup.ts'],
  
  // Module resolution
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
    '^@tests/(.*)$': '<rootDir>/tests/$1'
  },
  
  // Test timeout for hockey simulation tests
  testTimeout: 30000
};
```

## Verification and Troubleshooting

### Environment Verification Checklist

Environment verification ensures all components function correctly and provides systematic validation procedures that confirm proper setup while identifying potential issues before development activities commence with comprehensive service testing and integration validation.

```bash
#!/bin/bash
# Development Environment Verification Script

echo "🏒 Hockey League Simulation - Environment Verification"
echo "=================================================="

# Check Node.js version
echo "📦 Checking Node.js version..."
node_version=$(node --version)
echo "Node.js: $node_version"

# Check npm version
npm_version=$(npm --version)
echo "npm: $npm_version"

# Check Docker services
echo "🐳 Checking Docker services..."
if docker-compose ps | grep -q "Up"; then
    echo "✅ Docker services are running"
else
    echo "❌ Docker services are not running"
    echo "Run: docker-compose up -d"
fi

# Test database connection
echo "🗄️ Testing database connection..."
if npm run db:test-connection > /dev/null 2>&1; then
    echo "✅ Database connection successful"
else
    echo "❌ Database connection failed"
fi

# Test Redis connection
echo "📮 Testing Redis connection..."
if redis-cli -h localhost ping > /dev/null 2>&1; then
    echo "✅ Redis connection successful"
else
    echo "❌ Redis connection failed"
fi

# Check API health
echo "🔗 Testing API health..."
if curl -f http://localhost:3001/dev/health > /dev/null 2>&1; then
    echo "✅ API is responding"
else
    echo "❌ API is not responding"
    echo "Run: npm run dev in backend directory"
fi

# Check frontend
echo "🖥️ Testing frontend..."
if curl -f http://localhost:3000 > /dev/null 2>&1; then
    echo "✅ Frontend is running"
else
    echo "❌ Frontend is not running"
    echo "Run: npm run dev in frontend directory"
fi

echo "=================================================="
echo "Environment verification complete!"
```

### Common Issues and Solutions

Troubleshooting guidance addresses frequent setup issues and provides systematic resolution procedures that enable quick problem resolution while maintaining development productivity and ensuring consistent development environment operation across different systems and configurations.

**Database Connection Issues:**
```bash
# If PostgreSQL connection fails
docker-compose down
docker volume rm hockey-league-simulation_postgres_data
docker-compose up -d postgres

# Wait and test connection
sleep 10
npm run db:test-connection
```

**Port Conflicts:**
```bash
# Check what's using port 3000
lsof -i :3000
# Kill process if needed
kill -9 <PID>

# Or use different ports in environment variables
export PORT=3002
export FRONTEND_PORT=3001
```

**Node Modules Issues:**
```bash
# Clear npm cache and reinstall
rm -rf node_modules package-lock.json
npm cache clean --force
npm install
```

**Docker Permission Issues (Linux):**
```bash
# Add user to docker group
sudo usermod -aG docker $USER
# Logout and login again, or restart shell
newgrp docker
```

This comprehensive environment setup guide provides systematic procedures for establishing a complete local development environment that enables effective AI-assisted development and hockey domain functionality testing with all necessary services and tools properly configured.