# Deployment Environments - Hockey League Simulation System

This comprehensive deployment environments guide establishes the systematic configuration and management procedures for multiple deployment environments within the Hockey League Simulation System. The multi-environment strategy ensures reliable development workflows, thorough testing validation, and stable production operations across diverse organizational requirements and operational scales.

## Environment Architecture Overview

### Multi-Environment Strategy Framework

The deployment architecture implements a three-tier environment strategy that provides systematic progression from development through staging to production deployment with appropriate isolation, testing validation, and configuration management at each tier. This strategy ensures comprehensive quality assurance while maintaining development efficiency and operational reliability throughout the deployment lifecycle.

Environment isolation maintains complete separation of data, infrastructure, and configuration between different deployment tiers while enabling appropriate data synchronization and testing coordination procedures. Isolation strategies include network segregation, database separation, and access control enforcement that prevents cross-environment contamination and ensures testing accuracy.

Configuration management coordination ensures consistent application behavior across environments while accommodating environment-specific requirements including security policies, performance optimization, and integration configurations. Management procedures include automated configuration validation and deployment verification that maintains system reliability throughout environment transitions.

Promotion workflows establish systematic procedures for advancing code and configuration changes through environment tiers with appropriate validation gates and rollback capabilities at each stage. Workflow automation includes testing coordination and deployment verification that ensures quality maintenance throughout the promotion process.

## Development Environment Configuration

### Local Development Environment Setup

The development environment provides comprehensive local development capabilities including database seeding, hot reloading, and debugging tools with relaxed security policies and enhanced logging that facilitate rapid development cycles and effective troubleshooting procedures throughout feature development and testing activities.

```yaml
# Development Environment Configuration (docker-compose.dev.yml)
version: '3.8'
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
      - "9229:9229"  # Debug port
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://dev_user:dev_password@postgres:5432/hockey_sim_dev
      - REDIS_URL=redis://redis:6379/1
      - LOG_LEVEL=debug
      - ENABLE_HOT_RELOAD=true
      - MOCK_EXTERNAL_SERVICES=true
    volumes:
      - .:/app
      - /app/node_modules
    depends_on:
      - postgres
      - redis

  postgres:
    image: timescale/timescaledb:latest-pg14
    environment:
      - POSTGRES_DB=hockey_sim_dev
      - POSTGRES_USER=dev_user
      - POSTGRES_PASSWORD=dev_password
    ports:
      - "5432:5432"
    volumes:
      - dev_postgres_data:/var/lib/postgresql/data
      - ./scripts/dev-seed.sql:/docker-entrypoint-initdb.d/seed.sql

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - dev_redis_data:/data

volumes:
  dev_postgres_data:
  dev_redis_data:
```

Development environment features include comprehensive test data generation with realistic hockey scenarios, automated database seeding with sample leagues and players, and mock service integration that eliminates external dependencies while maintaining realistic development conditions and workflow efficiency.

### Development Tools and Debugging Configuration

Development tooling includes comprehensive debugging capabilities, performance profiling, and code analysis tools with appropriate integration and automation that enhances development productivity while maintaining code quality and system reliability throughout feature development activities.

```javascript
// Development Webpack Configuration
const path = require('path');

module.exports = {
  mode: 'development',
  devtool: 'eval-source-map',
  entry: './src/index.tsx',
  
  devServer: {
    hot: true,
    port: 3000,
    historyApiFallback: true,
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

  module: {
    rules: [
      {
        test: /\.(ts|tsx)$/,
        use: [
          {
            loader: 'ts-loader',
            options: {
              transpileOnly: true,
              compilerOptions: {
                noEmit: false,
                incremental: true
              }
            }
          }
        ]
      }
    ]
  },

  plugins: [
    new webpack.HotModuleReplacementPlugin(),
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify('development'),
      'process.env.API_BASE_URL': JSON.stringify('http://localhost:3001'),
      'process.env.WS_URL': JSON.stringify('ws://localhost:3001')
    })
  ]
};
```

## Staging Environment Configuration

### Pre-Production Testing Environment

The staging environment replicates production infrastructure and configuration while providing isolated testing capabilities for integration validation, performance evaluation, and user acceptance testing with production-equivalent data volumes and realistic operational conditions.

```yaml
# Staging Environment Infrastructure (staging.yml)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hockey-sim-staging
  namespace: staging
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hockey-sim-staging
  template:
    metadata:
      labels:
        app: hockey-sim-staging
    spec:
      containers:
      - name: app
        image: hockeysim/app:staging-latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "staging"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: staging-db-secret
              key: url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: staging-redis-secret
              key: url
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
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

---
apiVersion: v1
kind: Service
metadata:
  name: hockey-sim-staging-service
  namespace: staging
spec:
  selector:
    app: hockey-sim-staging
  ports:
  - port: 80
    targetPort: 3000
  type: LoadBalancer
```

Staging environment data management includes production data sanitization procedures, realistic test data generation, and comprehensive backup coordination that enables effective testing while maintaining data privacy and security requirements throughout validation activities.

### Integration Testing and Validation Procedures

Staging environment testing includes comprehensive integration validation, performance benchmarking, and user acceptance testing with automated test execution and manual validation procedures that ensure system reliability and user experience quality before production deployment.

```bash
#!/bin/bash
# Staging Environment Validation Script

set -e

echo "Starting staging environment validation..."

# Health Check Validation
echo "Checking application health..."
curl -f http://staging-api.hockeysim.com/health || exit 1

# Database Connectivity
echo "Validating database connectivity..."
curl -f http://staging-api.hockeysim.com/api/health/database || exit 1

# Redis Connectivity
echo "Validating Redis connectivity..."
curl -f http://staging-api.hockeysim.com/api/health/redis || exit 1

# External Service Integration
echo "Testing external service integration..."
curl -f http://staging-api.hockeysim.com/api/health/external-services || exit 1

# Performance Baseline Testing
echo "Running performance baseline tests..."
npm run test:performance:staging

# Critical Path Testing
echo "Executing critical path tests..."
npm run test:e2e:staging

# Security Validation
echo "Running security validation..."
npm run test:security:staging

# Data Migration Validation
echo "Validating data migration procedures..."
npm run validate:migration:staging

echo "Staging validation completed successfully!"
```

## Production Environment Configuration

### High-Availability Production Infrastructure

Production environment configuration implements high-availability architecture with redundancy, load balancing, and disaster recovery capabilities that ensure system reliability and performance under realistic operational loads and usage patterns while maintaining security and compliance requirements.

```yaml
# Production Environment Infrastructure (production.yml)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hockey-sim-production
  namespace: production
spec:
  replicas: 6
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
  selector:
    matchLabels:
      app: hockey-sim-production
  template:
    metadata:
      labels:
        app: hockey-sim-production
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - hockey-sim-production
              topologyKey: kubernetes.io/hostname
      containers:
      - name: app
        image: hockeysim/app:v1.0.0
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: production-db-secret
              key: url
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 60
          periodSeconds: 30
          timeoutSeconds: 10
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hockey-sim-production-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hockey-sim-production
  minReplicas: 6
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### Production Security and Monitoring Configuration

Production security implementation includes comprehensive protection mechanisms, monitoring integration, and compliance procedures with appropriate audit logging and incident response capabilities that ensure system security while maintaining operational visibility and accountability.

```yaml
# Production Security Configuration
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: hockey-sim-production-network-policy
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: hockey-sim-production
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 3000
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: database
    ports:
    - protocol: TCP
      port: 5432
  - to:
    - namespaceSelector:
        matchLabels:
          name: redis
    ports:
    - protocol: TCP
      port: 6379

---
apiVersion: v1
kind: Secret
metadata:
  name: production-tls-secret
  namespace: production
type: kubernetes.io/tls
data:
  tls.crt: LS0tLS1CRUdJTi... # Base64 encoded certificate
  tls.key: LS0tLS1CRUdJTi... # Base64 encoded private key
```

## Environment-Specific Database Configuration

### Database Environment Isolation and Management

Database environment management implements complete isolation between development, staging, and production databases with appropriate data synchronization, backup coordination, and migration procedures that ensure data integrity while enabling effective testing and development workflows.

```sql
-- Production Database Configuration
-- postgresql.conf settings for production

# Connection Settings
max_connections = 200
shared_buffers = 4GB
effective_cache_size = 12GB
work_mem = 256MB
maintenance_work_mem = 1GB

# Write Ahead Logging
wal_level = replica
max_wal_size = 4GB
min_wal_size = 1GB
checkpoint_completion_target = 0.9

# Replication Settings
hot_standby = on
max_replication_slots = 3
max_wal_senders = 3

# Performance Settings
random_page_cost = 1.1
effective_io_concurrency = 200
max_worker_processes = 8
max_parallel_workers_per_gather = 4
max_parallel_workers = 8
```

### Data Migration and Synchronization Procedures

Data migration coordination ensures consistent schema evolution across all environments with appropriate validation and rollback procedures while maintaining data integrity and system availability throughout migration execution and environment synchronization activities.

```bash
#!/bin/bash
# Database Migration Script for All Environments

ENVIRONMENT=$1
if [ -z "$ENVIRONMENT" ]; then
    echo "Usage: $0 <development|staging|production>"
    exit 1
fi

echo "Running database migrations for $ENVIRONMENT environment..."

# Set environment-specific variables
case $ENVIRONMENT in
    "development")
        DB_URL=$DEV_DATABASE_URL
        BACKUP_REQUIRED=false
        ;;
    "staging")
        DB_URL=$STAGING_DATABASE_URL
        BACKUP_REQUIRED=true
        ;;
    "production")
        DB_URL=$PRODUCTION_DATABASE_URL
        BACKUP_REQUIRED=true
        ;;
    *)
        echo "Invalid environment: $ENVIRONMENT"
        exit 1
        ;;
esac

# Create backup if required
if [ "$BACKUP_REQUIRED" = true ]; then
    echo "Creating pre-migration backup..."
    timestamp=$(date +%Y%m%d_%H%M%S)
    backup_file="backup_${ENVIRONMENT}_${timestamp}.sql"
    
    pg_dump $DB_URL > $backup_file
    if [ $? -ne 0 ]; then
        echo "Backup failed, aborting migration"
        exit 1
    fi
    echo "Backup created: $backup_file"
fi

# Run pending migrations
echo "Executing database migrations..."
npx knex migrate:latest --env $ENVIRONMENT

if [ $? -eq 0 ]; then
    echo "Migrations completed successfully"
else
    echo "Migration failed"
    if [ "$BACKUP_REQUIRED" = true ]; then
        echo "Restoring from backup: $backup_file"
        psql $DB_URL < $backup_file
    fi
    exit 1
fi

# Validate migration success
echo "Validating migration results..."
npx knex migrate:status --env $ENVIRONMENT

echo "Database migration completed for $ENVIRONMENT environment"
```

## Deployment Pipeline and Automation

### Continuous Integration and Deployment Configuration

Deployment pipeline automation implements comprehensive testing, building, and deployment procedures with appropriate validation gates and rollback capabilities that ensure code quality and system reliability throughout the deployment process across all environment tiers.

```yaml
# GitHub Actions Deployment Pipeline
name: Deploy Hockey League Simulation

on:
  push:
    branches:
      - main
      - develop
      - staging/*
  pull_request:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18.x, 20.x]
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
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
    
    - name: Run unit tests
      run: npm run test:unit
    
    - name: Run integration tests
      run: npm run test:integration
      env:
        DATABASE_URL: postgresql://test_user:test_password@localhost:5432/test_db
        REDIS_URL: redis://localhost:6379/0

  deploy-development:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Deploy to development
      run: |
        echo "Deploying to development environment..."
        # Development deployment commands
        
  deploy-staging:
    needs: test
    runs-on: ubuntu-latest
    if: startsWith(github.ref, 'refs/heads/staging/')
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Deploy to staging
      run: |
        echo "Deploying to staging environment..."
        # Staging deployment commands
        
    - name: Run staging validation
      run: |
        echo "Running staging validation tests..."
        npm run test:staging-validation

  deploy-production:
    needs: [test, deploy-staging]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Deploy to production
      run: |
        echo "Deploying to production environment..."
        # Production deployment commands
        
    - name: Run production smoke tests
      run: |
        echo "Running production smoke tests..."
        npm run test:production-smoke
```

### Environment Promotion and Rollback Procedures

Environment promotion workflows establish systematic procedures for advancing validated changes through environment tiers with appropriate testing validation and rollback capabilities that ensure system reliability and minimize deployment risks throughout the promotion process.

```bash
#!/bin/bash
# Environment Promotion Script

SOURCE_ENV=$1
TARGET_ENV=$2
VERSION=$3

if [ $# -lt 3 ]; then
    echo "Usage: $0 <source_env> <target_env> <version>"
    echo "Example: $0 staging production v1.2.0"
    exit 1
fi

echo "Promoting $VERSION from $SOURCE_ENV to $TARGET_ENV..."

# Validation checks
echo "Running pre-promotion validation..."

# Check source environment health
curl -f "http://${SOURCE_ENV}-api.hockeysim.com/health" || {
    echo "Source environment health check failed"
    exit 1
}

# Validate version exists in source
echo "Validating version $VERSION in $SOURCE_ENV..."

# Create deployment backup
if [ "$TARGET_ENV" = "production" ]; then
    echo "Creating production backup before deployment..."
    kubectl create backup production-backup-$(date +%Y%m%d_%H%M%S) --namespace=production
fi

# Deploy to target environment
echo "Deploying $VERSION to $TARGET_ENV..."
kubectl set image deployment/hockey-sim-$TARGET_ENV app=hockeysim/app:$VERSION --namespace=$TARGET_ENV

# Wait for rollout completion
kubectl rollout status deployment/hockey-sim-$TARGET_ENV --namespace=$TARGET_ENV --timeout=600s

# Run post-deployment validation
echo "Running post-deployment validation..."
./scripts/validate-environment.sh $TARGET_ENV

if [ $? -eq 0 ]; then
    echo "Promotion to $TARGET_ENV completed successfully"
else
    echo "Promotion validation failed, initiating rollback..."
    kubectl rollout undo deployment/hockey-sim-$TARGET_ENV --namespace=$TARGET_ENV
    exit 1
fi
```

## Monitoring and Maintenance Across Environments

### Environment-Specific Monitoring Configuration

Monitoring implementation provides comprehensive visibility across all environments with appropriate alerting thresholds and dashboard customization that enables effective operational oversight while maintaining system performance and reliability throughout diverse operational conditions.

```yaml
# Monitoring Configuration for All Environments
apiVersion: v1
kind: ConfigMap
metadata:
  name: monitoring-config
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s
    
    rule_files:
      - "hockey_sim_rules.yml"
    
    scrape_configs:
      - job_name: 'hockey-sim-development'
        static_configs:
          - targets: ['dev-api.hockeysim.com:3000']
        scrape_interval: 30s
        metrics_path: '/metrics'
        
      - job_name: 'hockey-sim-staging'
        static_configs:
          - targets: ['staging-api.hockeysim.com:3000']
        scrape_interval: 15s
        metrics_path: '/metrics'
        
      - job_name: 'hockey-sim-production'
        static_configs:
          - targets: ['api.hockeysim.com:3000']
        scrape_interval: 10s
        metrics_path: '/metrics'
        
    alerting:
      alertmanagers:
        - static_configs:
            - targets:
              - alertmanager:9093

  hockey_sim_rules.yml: |
    groups:
    - name: hockey_sim_alerts
      rules:
      - alert: HighResponseTime
        expr: http_request_duration_seconds{quantile="0.95"} > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High response time detected"
          
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
```

This comprehensive deployment environments framework provides systematic management across development, staging, and production environments while ensuring reliability, security, and operational effectiveness throughout the Hockey League Simulation System deployment lifecycle.