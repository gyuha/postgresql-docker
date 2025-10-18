# PostgreSQL 17 Docker Container

Comprehensive guide for setting up and managing PostgreSQL 17 database server using Docker Compose

> English | [한국어](./README.ko.md)

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Key Features](#key-features)
- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Installation and Setup](#installation-and-setup)
- [Environment Variables](#environment-variables)
- [Getting Started](#getting-started)
- [Database Access](#database-access)
- [CLI Usage](#cli-usage)
- [Data Management](#data-management)
- [Monitoring and Logging](#monitoring-and-logging)
- [Troubleshooting](#troubleshooting)
- [Security Considerations](#security-considerations)
- [Performance Tuning](#performance-tuning)
- [References](#references)

---

## Project Overview

This project provides a Docker Compose configuration for running PostgreSQL 17 in a container. It's suitable for both development and production environments, enabling you to set up a stable PostgreSQL database server with simple configuration.

### Version Information
- **PostgreSQL**: 17 (Latest stable version)
- **Docker Compose**: 3.x or higher (version attribute is obsolete)
- **Encoding**: UTF-8
- **Timezone**: Asia/Seoul (Korean Standard Time)

---

## Key Features

### ✨ Main Features
- **Latest PostgreSQL 17**: Enhanced performance and new features
- **Automatic Restart**: Stability guaranteed with `unless-stopped` policy
- **Local Data Storage**: Direct data access through bind mount
- **Korean Timezone**: Asia/Seoul timezone by default
- **Isolated Network**: Dedicated bridge network configuration
- **Environment Variable Based**: Flexible configuration through .env file

### 🔧 Tech Stack
- Docker & Docker Compose
- PostgreSQL 17
- Alpine Linux (Container base)

---

## Prerequisites

### Required Software
```bash
# Check Docker installation
docker --version
# Docker version 24.0.0 or higher recommended

# Check Docker Compose installation
docker compose version
# Docker Compose version v2.0.0 or higher recommended
```

### System Requirements
- **Minimum Memory**: 512MB RAM
- **Recommended Memory**: 2GB RAM or more
- **Disk Space**: Minimum 1GB (additional space needed based on data size)
- **Operating System**: Linux, macOS, Windows (WSL2)

### Permission Requirements
- Docker execution permission (typically users in the docker group)
- Read/write permission for data directory

---

## Project Structure

```
/opt/docker_containers/postgresql/
├── docker-compose.yml      # Docker Compose configuration file
├── .env                    # Environment variables (needs to be created)
├── .env.example            # Environment variable template
├── .gitignore              # Git ignore file list
├── README.md               # This document (English)
├── README.ko.md            # Korean documentation
└── postgresql_data/        # Database files storage (auto-created)
    └── pgdata/            # PostgreSQL data directory
        ├── base/          # Database files
        ├── global/        # Cluster-wide data
        ├── pg_wal/        # Write-Ahead Log
        └── ...
```

### File Descriptions

#### `docker-compose.yml`
Docker Compose configuration file that defines all settings for the PostgreSQL container.

#### `.env`
File for storing environment variables. **Contains sensitive information and should not be committed to Git.**

#### `.env.example`
Environment variable template file. Copy this file to create `.env`.

#### `postgresql_data/`
Directory where actual database files are stored. Created automatically when the container starts.

---

## Installation and Setup

### 1. Navigate to Project Directory

```bash
cd /opt/docker_containers/postgresql
```

### 2. Create Environment Variable File

```bash
# Copy .env.example to .env
cp .env.example .env

# Edit .env file
vi .env
# or
nano .env
```

### 3. Configure Environment Variables

Open the `.env` file and set the following values:

```bash
# Database username
POSTGRES_USER=postgres

# Set strong password (Required!)
POSTGRES_PASSWORD=your_secure_password_here

# Default database name
POSTGRES_DB=default

# Port configuration
POSTGRES_PORT=5432

# Timezone settings
TZ=Asia/Seoul
PGTZ=Asia/Seoul
```

---

## Environment Variables

### 📝 Detailed Environment Variable Descriptions

#### `POSTGRES_USER`
- **Description**: PostgreSQL superuser username
- **Default**: `postgres`
- **Recommendations**:
  - Change default value for security
  - Start with letter, use only letters/numbers/underscores
  - Examples: `admin`, `dbuser`, `app_user`

```bash
# Example
POSTGRES_USER=admin
```

#### `POSTGRES_PASSWORD`
- **Description**: PostgreSQL superuser password
- **Default**: None (required setting)
- **Security Requirements**:
  - **Minimum 20 characters** recommended
  - Combination of uppercase, lowercase, numbers, special characters
  - Avoid predictable patterns
  - Regular changes recommended (3-6 months)

```bash
# Bad examples
POSTGRES_PASSWORD=password123
POSTGRES_PASSWORD=admin

# Good example
POSTGRES_PASSWORD=Xk9$mN2#pQ7@wL4&vB8!zR5
```

**Password Generation Tools**:
```bash
# Generate secure random password (Linux/macOS)
openssl rand -base64 32

# Or
pwgen -s 32 1
```

#### `POSTGRES_DB`
- **Description**: Default database name to be auto-created on container start
- **Default**: `default`
- **Recommendations**:
  - Use project name or application name
  - Use only lowercase and underscores
  - Examples: `myapp`, `production_db`, `dev_database`

```bash
# Example
POSTGRES_DB=my_application
```

#### `POSTGRES_PORT`
- **Description**: Port number to access from host
- **Default**: `5432`
- **Use Cases**:
  - Use default: Single PostgreSQL instance
  - Change needed: Port conflict, multiple instance operation
  - Examples: `5433`, `15432`

```bash
# Port change example
POSTGRES_PORT=5433  # Access via host's port 5433
```

#### `TZ` (Timezone)
- **Description**: Container system timezone
- **Default**: `Asia/Seoul`
- **Available Values**:
  - `Asia/Seoul` - Korean Standard Time (KST, UTC+9)
  - `UTC` - Coordinated Universal Time
  - `America/New_York` - US Eastern Time
  - `Europe/London` - UK Time
  - `Asia/Tokyo` - Japan Standard Time

```bash
# Timezone change examples
TZ=UTC                    # Use UTC time
TZ=America/Los_Angeles    # US Pacific Time
```

#### `PGTZ` (PostgreSQL Timezone)
- **Description**: PostgreSQL database internal timezone
- **Default**: `Asia/Seoul`
- **Recommendations**:
  - Set same as `TZ` recommended
  - Ensures timestamp data consistency
  - Consider UTC for global services

```bash
# PostgreSQL timezone setting
PGTZ=Asia/Seoul
```

### 🔐 Environment Variable Security

#### .gitignore Configuration
Verify `.env` is included in `.gitignore`:

```bash
# Check
cat .gitignore

# Expected output:
# .env
# postgresql_data/
```

#### Environment Variable File Permissions

```bash
# Restrict .env file permissions to owner read/write only
chmod 600 .env

# Verify
ls -la .env
# -rw------- 1 user user ... .env
```

#### Production Environment Recommendations

1. **Use Docker Secrets** (Docker Swarm)
```yaml
secrets:
  postgres_password:
    external: true

services:
  postgresql:
    secrets:
      - postgres_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/postgres_password
```

2. **Environment Variable Encryption Tools**
   - AWS Secrets Manager
   - HashiCorp Vault
   - Azure Key Vault

### 📋 Environment Variable Examples

#### Development Environment
```bash
POSTGRES_USER=dev_user
POSTGRES_PASSWORD=dev_password_123
POSTGRES_DB=development_db
POSTGRES_PORT=5432
TZ=Asia/Seoul
PGTZ=Asia/Seoul
```

#### Production Environment
```bash
POSTGRES_USER=prod_admin
POSTGRES_PASSWORD=Xk9$mN2#pQ7@wL4&vB8!zR5yT3
POSTGRES_DB=production_db
POSTGRES_PORT=5432
TZ=UTC
PGTZ=UTC
```

#### Test Environment
```bash
POSTGRES_USER=test_user
POSTGRES_PASSWORD=test_password_456
POSTGRES_DB=test_db
POSTGRES_PORT=5433
TZ=Asia/Seoul
PGTZ=Asia/Seoul
```

---

## Getting Started

### Start Container

```bash
# Start in background
docker compose up -d

# Start with logs (container stops when you Ctrl+C)
docker compose up

# View logs after starting
docker compose logs -f postgresql
```

**Expected Output**:
```
[+] Running 2/2
 ✔ Network postgresql_postgres_network  Created
 ✔ Container postgresql                 Started
```

### Check Container Status

```bash
# Check running containers
docker compose ps

# Check detailed information
docker ps --filter name=postgresql
```

**Expected Output**:
```
NAME         IMAGE         STATUS        PORTS
postgresql   postgres:17   Up 2 minutes  0.0.0.0:5432->5432/tcp
```

### Stop Container

```bash
# Stop (keep container, preserve data)
docker compose stop

# Stop and remove (preserve data)
docker compose down

# Stop and remove all resources (Warning: deletes data too!)
docker compose down -v
```

### Restart Container

```bash
# Restart
docker compose restart

# Or
docker restart postgresql
```

---

## Database Access

### 1. Access via Docker Exec

```bash
# Connect with psql (default database)
docker exec -it postgresql psql -U postgres -d default

# Connect to specific database
docker exec -it postgresql psql -U postgres -d mydb

# Connect as superuser
docker exec -it postgresql psql -U postgres
```

### 2. Direct Connection from Host

If PostgreSQL client is installed on host:

```bash
# Install psql (Ubuntu/Debian)
sudo apt-get install postgresql-client

# Connect
psql -h localhost -p 5432 -U postgres -d default
```

**Password Input**: Enter `POSTGRES_PASSWORD` set in `.env` file

### 3. Access via External Tools

#### DBeaver
1. Connection Settings
   - Host: `localhost`
   - Port: `5432`
   - Database: `default`
   - Username: `postgres`
   - Password: `POSTGRES_PASSWORD` from `.env`

#### pgAdmin
1. Add Server
   - Name: `PostgreSQL Docker`
   - Host: `localhost`
   - Port: `5432`
   - Maintenance database: `default`
   - Username: `postgres`
   - Password: `POSTGRES_PASSWORD` from `.env`

#### DataGrip / IntelliJ IDEA
1. Database → New → Data Source → PostgreSQL
2. Enter connection information

---

## CLI Usage

### Basic psql Commands

#### Database Connection
```bash
# Run psql inside container
docker exec -it postgresql psql -U postgres -d default
```

### psql Meta Commands

#### Database Management

```sql
-- List all databases
\l
-- or
\list

-- Check current database
SELECT current_database();

-- Create database
CREATE DATABASE myapp;

-- Drop database
DROP DATABASE myapp;

-- Switch database
\c myapp
-- or
\connect myapp

-- Check database size
SELECT
    pg_database.datname AS database_name,
    pg_size_pretty(pg_database_size(pg_database.datname)) AS size
FROM pg_database
ORDER BY pg_database_size(pg_database.datname) DESC;
```

**Example Output**:
```
                                  List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------+----------+----------+------------+------------+-----------------------
 default   | postgres | UTF8     | C          | C          |
 postgres  | postgres | UTF8     | C          | C          |
 template0 | postgres | UTF8     | C          | C          | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | C          | C          | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
```

#### Table Management

```sql
-- List all tables in current database
\dt

-- List all tables including schema
\dt *.*

-- Table detailed information
\d table_name

-- Table structure only
\d+ table_name

-- Create table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Drop table
DROP TABLE users;

-- Rename table
ALTER TABLE old_name RENAME TO new_name;

-- Count rows in table
SELECT COUNT(*) FROM users;
```

#### Schema Management

```sql
-- List all schemas
\dn

-- Create schema
CREATE SCHEMA app_schema;

-- Drop schema
DROP SCHEMA app_schema CASCADE;

-- Check current schema
SELECT current_schema();

-- List tables in schema
\dt app_schema.*
```

#### User and Permission Management

```sql
-- List all users (roles)
\du

-- Create user
CREATE USER myuser WITH PASSWORD 'mypassword';

-- Grant database privileges to user
GRANT ALL PRIVILEGES ON DATABASE myapp TO myuser;

-- Grant specific table privileges
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLE users TO myuser;

-- Grant schema privileges
GRANT USAGE ON SCHEMA public TO myuser;
GRANT CREATE ON SCHEMA public TO myuser;

-- Drop user
DROP USER myuser;

-- Change password
ALTER USER myuser WITH PASSWORD 'new_password';

-- Grant superuser privileges
ALTER USER myuser WITH SUPERUSER;
```

#### Index Management

```sql
-- Check table indexes
\di

-- Create index
CREATE INDEX idx_users_email ON users(email);

-- Create unique index
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- Drop index
DROP INDEX idx_users_email;

-- Rebuild index
REINDEX TABLE users;
```

#### Data Query and Manipulation

```sql
-- Query data
SELECT * FROM users;
SELECT username, email FROM users WHERE id = 1;
SELECT * FROM users ORDER BY created_at DESC LIMIT 10;

-- Insert data
INSERT INTO users (username, email)
VALUES ('john_doe', 'john@example.com');

-- Insert multiple rows
INSERT INTO users (username, email) VALUES
    ('alice', 'alice@example.com'),
    ('bob', 'bob@example.com'),
    ('charlie', 'charlie@example.com');

-- Update data
UPDATE users SET email = 'newemail@example.com' WHERE id = 1;

-- Delete data
DELETE FROM users WHERE id = 1;

-- Delete all data (Caution!)
TRUNCATE TABLE users;
```

#### Sequence Management

```sql
-- List sequences
\ds

-- Check current sequence value
SELECT currval('users_id_seq');

-- Check next sequence value
SELECT nextval('users_id_seq');

-- Reset sequence
ALTER SEQUENCE users_id_seq RESTART WITH 1;
```

#### View Management

```sql
-- List views
\dv

-- Create view
CREATE VIEW active_users AS
SELECT * FROM users WHERE status = 'active';

-- Drop view
DROP VIEW active_users;
```

#### Functions and Procedures

```sql
-- List functions
\df

-- Create function
CREATE OR REPLACE FUNCTION get_user_count()
RETURNS INTEGER AS $$
BEGIN
    RETURN (SELECT COUNT(*) FROM users);
END;
$$ LANGUAGE plpgsql;

-- Execute function
SELECT get_user_count();

-- Drop function
DROP FUNCTION get_user_count;
```

### System Information and Monitoring

```sql
-- Check PostgreSQL version
SELECT version();

-- Current time (server time)
SELECT NOW();
SELECT CURRENT_TIMESTAMP;

-- Check current timezone
SHOW timezone;

-- Check active connections
SELECT
    pid,
    usename,
    datname,
    client_addr,
    state,
    query
FROM pg_stat_activity
WHERE state = 'active';

-- Check all connections
SELECT
    COUNT(*) as total_connections,
    COUNT(*) FILTER (WHERE state = 'active') as active_connections,
    COUNT(*) FILTER (WHERE state = 'idle') as idle_connections
FROM pg_stat_activity;

-- Terminate specific connection
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE datname = 'myapp' AND pid <> pg_backend_pid();

-- Table statistics
SELECT
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size,
    n_live_tup AS live_rows,
    n_dead_tup AS dead_rows
FROM pg_stat_user_tables
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Database statistics
SELECT
    datname,
    numbackends as connections,
    xact_commit as commits,
    xact_rollback as rollbacks,
    blks_read,
    blks_hit,
    tup_returned,
    tup_fetched,
    tup_inserted,
    tup_updated,
    tup_deleted
FROM pg_stat_database
WHERE datname = current_database();
```

### Utility Commands

```sql
-- Execute SQL file
\i /path/to/script.sql

-- Save query results to file
\o /path/to/output.txt
SELECT * FROM users;
\o  -- Stop file output

-- Export to CSV
\copy users TO '/tmp/users.csv' WITH CSV HEADER;

-- Import from CSV
\copy users FROM '/tmp/users.csv' WITH CSV HEADER;

-- Show query execution time
\timing on

-- Extended output mode (vertical format)
\x
-- or
\x auto

-- Turn pager off/on
\pset pager off
\pset pager on

-- Exit psql
\q
-- or
exit
```

### Backup and Restore (CLI)

#### Database Backup
```bash
# Full database backup (from outside container)
docker exec postgresql pg_dump -U postgres default > backup.sql

# Compressed backup
docker exec postgresql pg_dump -U postgres default | gzip > backup.sql.gz

# Custom format backup (more flexible for restore)
docker exec postgresql pg_dump -U postgres -Fc default > backup.dump

# Backup specific table only
docker exec postgresql pg_dump -U postgres -t users default > users_backup.sql

# Schema only backup (exclude data)
docker exec postgresql pg_dump -U postgres -s default > schema_only.sql

# Data only backup (exclude schema)
docker exec postgresql pg_dump -U postgres -a default > data_only.sql
```

#### Database Restore
```bash
# Restore SQL file
cat backup.sql | docker exec -i postgresql psql -U postgres -d default

# Restore compressed file
gunzip -c backup.sql.gz | docker exec -i postgresql psql -U postgres -d default

# Restore custom format
docker exec -i postgresql pg_restore -U postgres -d default < backup.dump

# Create new database and restore
docker exec postgresql psql -U postgres -c "CREATE DATABASE myapp_restored;"
cat backup.sql | docker exec -i postgresql psql -U postgres -d myapp_restored
```

#### Full Cluster Backup
```bash
# Backup all databases + global objects
docker exec postgresql pg_dumpall -U postgres > full_backup.sql

# Restore
cat full_backup.sql | docker exec -i postgresql psql -U postgres
```

### Performance Analysis

```sql
-- View query execution plan
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- Execution plan + actual execution statistics
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';

-- Find slow queries (by execution time)
SELECT
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    max_exec_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- Check index usage rate
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan as index_scans,
    idx_tup_read as tuples_read,
    idx_tup_fetch as tuples_fetched
FROM pg_stat_user_indexes
ORDER BY idx_scan ASC;

-- Find unused indexes
SELECT
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) as index_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY pg_relation_size(indexrelid) DESC;
```

### Transaction Management

```sql
-- Start transaction
BEGIN;

-- Execute queries
INSERT INTO users (username, email) VALUES ('test', 'test@example.com');
UPDATE users SET status = 'active' WHERE username = 'test';

-- Commit (save changes)
COMMIT;

-- Or rollback (cancel changes)
ROLLBACK;

-- Use savepoints
BEGIN;
INSERT INTO users (username, email) VALUES ('user1', 'user1@example.com');
SAVEPOINT sp1;
INSERT INTO users (username, email) VALUES ('user2', 'user2@example.com');
ROLLBACK TO sp1;  -- Cancel only user2 insert
COMMIT;  -- Save user1 insert
```

### Extension Management

```sql
-- List installed extensions
\dx

-- List available extensions
SELECT * FROM pg_available_extensions ORDER BY name;

-- Install extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";  -- Similar string search
CREATE EXTENSION IF NOT EXISTS "hstore";   -- Key-Value storage

-- Drop extension
DROP EXTENSION "uuid-ossp";

-- UUID generation example
SELECT uuid_generate_v4();
```

### Useful psql Configuration

```bash
# Create ~/.psqlrc file (auto-executed on psql start)
cat > ~/.psqlrc << 'EOF'
-- Case-insensitive auto-completion
\set COMP_KEYWORD_CASE upper

-- Display NULL values
\pset null '(null)'

-- Show query execution time
\timing

-- Customize prompt
\set PROMPT1 '%n@%M:%> %x%# '

-- Increase history file size
\set HISTSIZE 10000

-- Rollback on error
\set ON_ERROR_ROLLBACK interactive
EOF
```

### Docker Container File Operations

```bash
# Copy file to container
docker cp backup.sql postgresql:/tmp/

# Copy file from container
docker cp postgresql:/tmp/export.sql ./

# Execute SQL inside container
docker exec postgresql psql -U postgres -d default -f /tmp/backup.sql
```

---

## Data Management

### Backup Strategy

#### 1. Automated Backup Script

```bash
#!/bin/bash
# backup.sh - PostgreSQL automated backup script

BACKUP_DIR="/path/to/backups"
DATE=$(date +"%Y%m%d_%H%M%S")
CONTAINER="postgresql"
DB_NAME="default"
DB_USER="postgres"

# Create backup directory
mkdir -p $BACKUP_DIR

# Execute backup
docker exec $CONTAINER pg_dump -U $DB_USER $DB_NAME | gzip > "$BACKUP_DIR/backup_${DATE}.sql.gz"

# Delete backups older than 7 days
find $BACKUP_DIR -name "backup_*.sql.gz" -mtime +7 -delete

echo "Backup completed: backup_${DATE}.sql.gz"
```

**Grant execution permission and test**:
```bash
chmod +x backup.sh
./backup.sh
```

#### 2. Cron Automation

```bash
# Edit crontab
crontab -e

# Execute backup daily at 2 AM
0 2 * * * /path/to/backup.sh >> /var/log/postgresql_backup.log 2>&1
```

### Restore Procedure

#### Full Restore
```bash
# 1. Stop existing container
docker compose down

# 2. Backup data directory (optional)
mv postgresql_data postgresql_data.old

# 3. Start container
docker compose up -d

# 4. Restore backup file
gunzip -c backup_20240101_020000.sql.gz | docker exec -i postgresql psql -U postgres -d default

# 5. Verify restore
docker exec -it postgresql psql -U postgres -d default -c "\dt"
```

### Data Migration

#### Migration from Different PostgreSQL Version

```bash
# 1. Backup from old version
docker exec old_postgresql pg_dumpall -U postgres > full_backup.sql

# 2. Start new container
docker compose up -d

# 3. Restore
cat full_backup.sql | docker exec -i postgresql psql -U postgres
```

---

## Monitoring and Logging

### View Logs

```bash
# Real-time log monitoring
docker compose logs -f postgresql

# Recent 100 lines of logs
docker compose logs --tail=100 postgresql

# Logs after specific time
docker compose logs --since="2024-01-01T00:00:00" postgresql

# Include timestamps
docker compose logs -t postgresql
```

### Container Resource Usage

```bash
# Real-time resource monitoring
docker stats postgresql

# CPU, memory usage check
docker stats postgresql --no-stream
```

**Example Output**:
```
CONTAINER ID   NAME         CPU %     MEM USAGE / LIMIT     MEM %
abc123def456   postgresql   2.45%     256.3MiB / 2GiB      12.52%
```

### Disk Usage

```bash
# PostgreSQL data directory size
du -sh postgresql_data/

# Detailed analysis
du -h --max-depth=2 postgresql_data/
```

### Performance Monitoring Queries

```sql
-- Database connection count
SELECT count(*) FROM pg_stat_activity;

-- Largest tables Top 10
SELECT
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_stat_user_tables
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 10;

-- Cache hit ratio (95%+ is ideal)
SELECT
    sum(heap_blks_read) as heap_read,
    sum(heap_blks_hit)  as heap_hit,
    sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) * 100 as cache_hit_ratio
FROM pg_statio_user_tables;
```

---

## Troubleshooting

### Common Issues

#### 1. Container Won't Start

**Symptoms**:
```bash
docker compose up -d
# Error: ...
```

**Solutions**:

```bash
# Check logs
docker compose logs postgresql

# Common causes:
# - POSTGRES_PASSWORD not set
# - Port conflict (5432)
# - Data directory permission issues

# Check port conflict
sudo netstat -tulpn | grep 5432
# or
sudo lsof -i :5432

# If another process is using it, change port
# Change POSTGRES_PORT=5433 in .env file
```

#### 2. Password Error

**Symptoms**:
```
FATAL: password authentication failed for user "postgres"
```

**Solutions**:

```bash
# Check .env file
cat .env | grep POSTGRES_PASSWORD

# Reset password (Warning: data loss!)
docker compose down
rm -rf postgresql_data/
docker compose up -d
```

#### 3. Permission Issues

**Symptoms**:
```
initdb: could not change permissions of directory "/var/lib/postgresql/data/pgdata"
```

**Solutions**:

```bash
# Fix data directory permissions
# PostgreSQL container runs as UID 999
sudo chown -R 999:999 postgresql_data/

# Or allow all users access (development environment)
chmod -R 777 postgresql_data/
```

#### 4. Connection Refused

**Symptoms**:
```
could not connect to server: Connection refused
```

**Solutions**:

```bash
# Check container is running
docker ps | grep postgresql

# Check network connection
docker exec postgresql pg_isready -U postgres

# Check port mapping
docker port postgresql

# Check firewall (Linux)
sudo ufw status
sudo ufw allow 5432/tcp
```

#### 5. Disk Space Full

**Symptoms**:
```
ERROR: could not extend file: No space left on device
```

**Solutions**:

```bash
# Check disk usage
df -h

# Clean old logs
docker system prune -a

# Clean WAL files (inside container)
docker exec postgresql psql -U postgres -c "CHECKPOINT;"

# Database VACUUM
docker exec postgresql psql -U postgres -d default -c "VACUUM FULL;"
```

### Debugging Tools

```bash
# Access container shell
docker exec -it postgresql /bin/bash

# Check PostgreSQL configuration file
docker exec postgresql cat /var/lib/postgresql/data/pgdata/postgresql.conf

# Check log file location
docker exec postgresql psql -U postgres -c "SHOW log_directory;"
docker exec postgresql psql -U postgres -c "SHOW log_filename;"
```

### Performance Issues

```sql
-- Find long-running queries
SELECT
    pid,
    now() - query_start as duration,
    query,
    state
FROM pg_stat_activity
WHERE state != 'idle'
AND query_start < now() - interval '5 minutes'
ORDER BY duration DESC;

-- Force kill slow query
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE pid = 12345;

-- Check lock waits
SELECT
    locktype,
    relation::regclass,
    mode,
    granted,
    pid
FROM pg_locks
WHERE NOT granted;
```

---

## Security Considerations

### Network Security

#### 1. Restrict Port Exposure

Block external access in production:

```yaml
# docker-compose.yml
services:
  postgresql:
    ports:
      - "127.0.0.1:5432:5432"  # Localhost only
```

#### 2. Create Application-Specific Users

```sql
-- Read-only user
CREATE USER readonly WITH PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE default TO readonly;
GRANT USAGE ON SCHEMA public TO readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO readonly;

-- Application user (CRUD)
CREATE USER appuser WITH PASSWORD 'app_secure_password';
GRANT CONNECT ON DATABASE default TO appuser;
GRANT USAGE, CREATE ON SCHEMA public TO appuser;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO appuser;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO appuser;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO appuser;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT USAGE, SELECT ON SEQUENCES TO appuser;
```

### Data Encryption

#### 1. SSL/TLS Connection (Production Recommended)

```yaml
# docker-compose.yml
services:
  postgresql:
    environment:
      POSTGRES_HOST_AUTH_METHOD: scram-sha-256
    volumes:
      - ./ssl/server.crt:/var/lib/postgresql/server.crt
      - ./ssl/server.key:/var/lib/postgresql/server.key
    command: >
      -c ssl=on
      -c ssl_cert_file=/var/lib/postgresql/server.crt
      -c ssl_key_file=/var/lib/postgresql/server.key
```

#### 2. Password Policy

```sql
-- Set password expiration (90 days)
ALTER USER appuser VALID UNTIL '2024-04-01';

-- Connection limit
ALTER USER appuser CONNECTION LIMIT 10;
```

### Audit Logging

```sql
-- Install pg_stat_statements extension
CREATE EXTENSION pg_stat_statements;

-- Log all DDL commands (postgresql.conf)
-- log_statement = 'ddl'
```

### Regular Security Checklist

- [ ] Change default postgres user password
- [ ] Delete unnecessary databases
- [ ] Remove unused users
- [ ] Apply principle of least privilege
- [ ] Verify regular backups
- [ ] Check PostgreSQL version updates
- [ ] Regular log review
- [ ] Monitor abnormal connection attempts

---

## Performance Tuning

### PostgreSQL Configuration Optimization

#### 1. Memory Settings

Adjust settings inside container:

```bash
# Edit postgresql.conf
docker exec -it postgresql bash
vi /var/lib/postgresql/data/pgdata/postgresql.conf
```

**Recommended Settings** (4GB RAM system):

```conf
# Memory Settings
shared_buffers = 1GB                    # 25% of total memory
effective_cache_size = 3GB              # 75% of total memory
maintenance_work_mem = 256MB            # 5-10% of RAM
work_mem = 16MB                         # Consider concurrent connections

# Checkpoint Settings
checkpoint_completion_target = 0.9
wal_buffers = 16MB
max_wal_size = 2GB
min_wal_size = 1GB

# Query Planner
random_page_cost = 1.1                  # For SSD
effective_io_concurrency = 200          # For SSD
```

**Restart after configuration**:
```bash
docker compose restart
```

#### 2. Connection Pooling

**Using External PgBouncer**:

```yaml
# Add to docker-compose.yml
services:
  pgbouncer:
    image: pgbouncer/pgbouncer:latest
    environment:
      - DATABASES_HOST=postgresql
      - DATABASES_PORT=5432
      - DATABASES_USER=postgres
      - DATABASES_PASSWORD=${POSTGRES_PASSWORD}
      - DATABASES_DBNAME=default
      - PGBOUNCER_POOL_MODE=transaction
      - PGBOUNCER_MAX_CLIENT_CONN=1000
      - PGBOUNCER_DEFAULT_POOL_SIZE=25
    ports:
      - "6432:6432"
    depends_on:
      - postgresql
    networks:
      - postgres_network
```

### Index Optimization

```sql
-- Find duplicate/unused indexes
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan,
    pg_size_pretty(pg_relation_size(indexrelid)) as size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
AND indexrelid NOT IN (
    SELECT indexrelid FROM pg_index WHERE indisunique
)
ORDER BY pg_relation_size(indexrelid) DESC;

-- Find tables with many sequential scans (need indexes)
SELECT
    schemaname,
    tablename,
    seq_scan,
    seq_tup_read,
    idx_scan,
    seq_tup_read / seq_scan as avg_seq_tup
FROM pg_stat_user_tables
WHERE seq_scan > 0
ORDER BY seq_tup_read DESC
LIMIT 10;
```

### VACUUM and ANALYZE

```sql
-- Manual VACUUM
VACUUM ANALYZE;

-- Specific table VACUUM
VACUUM ANALYZE users;

-- VACUUM FULL (locks table, caution!)
VACUUM FULL users;

-- Check autovacuum settings
SHOW autovacuum;

-- Per-table VACUUM statistics
SELECT
    schemaname,
    tablename,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables;
```

### Query Optimization

```sql
-- Analyze execution plan
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM users WHERE email = 'test@example.com';

-- Update statistics
ANALYZE users;

-- Check table statistics
SELECT * FROM pg_stats WHERE tablename = 'users';
```

---

## References

### Official Documentation
- [PostgreSQL 17 Official Documentation](https://www.postgresql.org/docs/17/)
- [Docker Hub - PostgreSQL](https://hub.docker.com/_/postgres)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

### Useful Tools
- [pgAdmin](https://www.pgadmin.org/) - Web-based administration tool
- [DBeaver](https://dbeaver.io/) - Cross-platform DB client
- [DataGrip](https://www.jetbrains.com/datagrip/) - JetBrains DB IDE
- [TablePlus](https://tableplus.com/) - Modern DB client
- [Adminer](https://www.adminer.org/) - Lightweight web DB management

### Learning Resources
- [PostgreSQL Tutorial](https://www.postgresqltutorial.com/)
- [Use The Index, Luke](https://use-the-index-luke.com/) - SQL indexing guide
- [PostgreSQL Performance](https://www.postgresql.org/docs/current/performance-tips.html)

### Community
- [PostgreSQL Slack](https://postgres-slack.herokuapp.com/)
- [Stack Overflow - PostgreSQL](https://stackoverflow.com/questions/tagged/postgresql)
- [Reddit - r/PostgreSQL](https://www.reddit.com/r/PostgreSQL/)

### Monitoring Tools
- [pgMonitor](https://github.com/CrunchyData/pgmonitor)
- [pg_stat_statements](https://www.postgresql.org/docs/current/pgstatstatements.html)
- [Prometheus + Grafana](https://grafana.com/grafana/dashboards/9628)

---

## License

This project follows the PostgreSQL license.

- PostgreSQL: [PostgreSQL License](https://www.postgresql.org/about/licence/)

---

## Contact and Support

If you encounter issues:
1. Check the [Troubleshooting](#troubleshooting) section in this README
2. Refer to [PostgreSQL Official Documentation](https://www.postgresql.org/docs/17/)
3. Check logs: `docker compose logs postgresql`

**Version Information**:
- Documentation Version: 1.0.0
- Last Updated: 2025-10-18
- PostgreSQL Version: 17

---

**🎉 Thank you for using PostgreSQL 17 Docker Container!**
