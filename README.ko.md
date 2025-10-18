# PostgreSQL 17 Docker 컨테이너

Docker Compose를 사용한 PostgreSQL 17 데이터베이스 서버 구성 및 운영 가이드

> 한국어 | [English](./README.md)

## 📋 목차

- [프로젝트 개요](#프로젝트-개요)
- [주요 특징](#주요-특징)
- [사전 요구사항](#사전-요구사항)
- [프로젝트 구조](#프로젝트-구조)
- [설치 및 설정](#설치-및-설정)
- [환경 변수 설정](#환경-변수-설정)
- [시작하기](#시작하기)
- [데이터베이스 접근](#데이터베이스-접근)
- [CLI 사용법](#cli-사용법)
- [데이터 관리](#데이터-관리)
- [모니터링 및 로깅](#모니터링-및-로깅)
- [문제 해결](#문제-해결)
- [보안 고려사항](#보안-고려사항)
- [성능 튜닝](#성능-튜닝)
- [참고 자료](#참고-자료)

---

## 프로젝트 개요

이 프로젝트는 PostgreSQL 17을 Docker 컨테이너로 실행하기 위한 구성입니다. 개발 및 프로덕션 환경 모두에서 사용 가능하며, 간단한 설정으로 안정적인 PostgreSQL 데이터베이스 서버를 구축할 수 있습니다.

### 버전 정보
- **PostgreSQL**: 17 (최신 안정 버전)
- **Docker Compose**: 3.x 이상 (version 속성은 obsolete)
- **인코딩**: UTF-8
- **타임존**: Asia/Seoul (한국 표준시)

---

## 주요 특징

### ✨ 주요 기능
- **최신 PostgreSQL 17**: 향상된 성능과 새로운 기능
- **자동 재시작**: `unless-stopped` 정책으로 안정성 보장
- **로컬 데이터 저장**: Bind mount를 통한 직접적인 데이터 접근
- **한국 시간대**: Asia/Seoul 타임존 기본 설정
- **격리된 네트워크**: 전용 Bridge 네트워크 구성
- **환경 변수 기반 설정**: .env 파일을 통한 유연한 구성

### 🔧 기술 스택
- Docker & Docker Compose
- PostgreSQL 17
- Alpine Linux (컨테이너 베이스)

---

## 사전 요구사항

### 필수 소프트웨어
```bash
# Docker 설치 확인
docker --version
# Docker version 24.0.0 이상 권장

# Docker Compose 설치 확인
docker compose version
# Docker Compose version v2.0.0 이상 권장
```

### 시스템 요구사항
- **최소 메모리**: 512MB RAM
- **권장 메모리**: 2GB RAM 이상
- **디스크 공간**: 최소 1GB (데이터 크기에 따라 추가 필요)
- **운영체제**: Linux, macOS, Windows (WSL2)

### 권한 요구사항
- Docker 실행 권한 (일반적으로 docker 그룹에 속한 사용자)
- 데이터 디렉토리 읽기/쓰기 권한

---

## 프로젝트 구조

```
/opt/docker_containers/postgresql/
├── docker-compose.yml      # Docker Compose 설정 파일
├── .env                    # 환경 변수 설정 (생성 필요)
├── .env.example            # 환경 변수 템플릿
├── .gitignore              # Git 제외 파일 목록
├── README.md               # 영문 문서
├── README.ko.md            # 이 문서 (한글)
└── postgresql_data/        # 데이터베이스 파일 저장 (자동 생성)
    └── pgdata/            # PostgreSQL 데이터 디렉토리
        ├── base/          # 데이터베이스 파일
        ├── global/        # 클러스터 전역 데이터
        ├── pg_wal/        # Write-Ahead Log
        └── ...
```

### 파일 설명

#### `docker-compose.yml`
Docker Compose 구성 파일로, PostgreSQL 컨테이너의 모든 설정을 정의합니다.

#### `.env`
환경 변수를 저장하는 파일입니다. **보안상 중요한 정보를 포함하므로 Git에 커밋하면 안 됩니다.**

#### `.env.example`
환경 변수 템플릿 파일입니다. 이 파일을 복사하여 `.env` 파일을 생성합니다.

#### `postgresql_data/`
실제 데이터베이스 파일이 저장되는 디렉토리입니다. 컨테이너가 시작되면 자동으로 생성됩니다.

---

## 설치 및 설정

### 1. 프로젝트 디렉토리로 이동

```bash
cd /opt/docker_containers/postgresql
```

### 2. 환경 변수 파일 생성

```bash
# .env.example을 .env로 복사
cp .env.example .env

# .env 파일 편집
vi .env
# 또는
nano .env
```

### 3. 환경 변수 설정

`.env` 파일을 열어 다음 값들을 설정합니다:

```bash
# 데이터베이스 사용자명 설정
POSTGRES_USER=postgres

# 강력한 비밀번호 설정 (필수!)
POSTGRES_PASSWORD=your_secure_password_here

# 기본 데이터베이스 이름
POSTGRES_DB=default

# 포트 설정
POSTGRES_PORT=5432

# 타임존 설정
TZ=Asia/Seoul
PGTZ=Asia/Seoul
```

---

## 환경 변수 설정

### 📝 상세 환경 변수 설명

#### `POSTGRES_USER`
- **설명**: PostgreSQL 슈퍼유저의 사용자명
- **기본값**: `postgres`
- **권장사항**:
  - 보안을 위해 기본값 변경 권장
  - 영문자로 시작, 영문자/숫자/언더스코어만 사용
  - 예: `admin`, `dbuser`, `app_user`

```bash
# 예시
POSTGRES_USER=admin
```

#### `POSTGRES_PASSWORD`
- **설명**: PostgreSQL 슈퍼유저의 비밀번호
- **기본값**: 없음 (필수 설정)
- **보안 요구사항**:
  - **최소 20자 이상** 권장
  - 대소문자, 숫자, 특수문자 조합
  - 예측 가능한 패턴 사용 금지
  - 주기적 변경 권장 (3-6개월)

```bash
# 나쁜 예
POSTGRES_PASSWORD=password123
POSTGRES_PASSWORD=admin

# 좋은 예
POSTGRES_PASSWORD=Xk9$mN2#pQ7@wL4&vB8!zR5
```

**비밀번호 생성 도구**:
```bash
# 안전한 랜덤 비밀번호 생성 (Linux/macOS)
openssl rand -base64 32

# 또는
pwgen -s 32 1
```

#### `POSTGRES_DB`
- **설명**: 컨테이너 시작 시 자동 생성될 기본 데이터베이스 이름
- **기본값**: `default`
- **권장사항**:
  - 프로젝트명이나 애플리케이션명 사용
  - 소문자와 언더스코어만 사용
  - 예: `myapp`, `production_db`, `dev_database`

```bash
# 예시
POSTGRES_DB=my_application
```

#### `POSTGRES_PORT`
- **설명**: 호스트에서 접근할 포트 번호
- **기본값**: `5432`
- **사용 시나리오**:
  - 기본값 사용: 단일 PostgreSQL 인스턴스
  - 변경 필요: 포트 충돌 시, 다중 인스턴스 운영 시
  - 예: `5433`, `15432`

```bash
# 포트 변경 예시
POSTGRES_PORT=5433  # 호스트의 5433 포트로 접근
```

#### `TZ` (Timezone)
- **설명**: 컨테이너 시스템의 타임존
- **기본값**: `Asia/Seoul`
- **사용 가능한 값**:
  - `Asia/Seoul` - 한국 표준시 (KST, UTC+9)
  - `UTC` - 협정 세계시
  - `America/New_York` - 미국 동부 표준시
  - `Europe/London` - 영국 표준시
  - `Asia/Tokyo` - 일본 표준시

```bash
# 타임존 변경 예시
TZ=UTC                    # UTC 시간 사용
TZ=America/Los_Angeles    # 미국 서부 시간
```

#### `PGTZ` (PostgreSQL Timezone)
- **설명**: PostgreSQL 데이터베이스 내부 타임존
- **기본값**: `Asia/Seoul`
- **권장사항**:
  - `TZ`와 동일하게 설정 권장
  - 타임스탬프 데이터 일관성 보장
  - 글로벌 서비스의 경우 UTC 사용 고려

```bash
# PostgreSQL 타임존 설정
PGTZ=Asia/Seoul
```

### 🔐 환경 변수 보안

#### .gitignore 설정
`.gitignore` 파일에 `.env`가 포함되어 있는지 확인:

```bash
# 확인
cat .gitignore

# 출력 예시:
# .env
# postgresql_data/
```

#### 환경 변수 파일 권한 설정

```bash
# .env 파일 권한을 소유자만 읽기/쓰기로 제한
chmod 600 .env

# 확인
ls -la .env
# -rw------- 1 user user ... .env
```

#### 프로덕션 환경 권장사항

1. **Docker Secrets 사용** (Docker Swarm)
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

2. **환경 변수 암호화 도구**
   - AWS Secrets Manager
   - HashiCorp Vault
   - Azure Key Vault

### 📋 환경 변수 예시

#### 개발 환경
```bash
POSTGRES_USER=dev_user
POSTGRES_PASSWORD=dev_password_123
POSTGRES_DB=development_db
POSTGRES_PORT=5432
TZ=Asia/Seoul
PGTZ=Asia/Seoul
```

#### 프로덕션 환경
```bash
POSTGRES_USER=prod_admin
POSTGRES_PASSWORD=Xk9$mN2#pQ7@wL4&vB8!zR5yT3
POSTGRES_DB=production_db
POSTGRES_PORT=5432
TZ=UTC
PGTZ=UTC
```

#### 테스트 환경
```bash
POSTGRES_USER=test_user
POSTGRES_PASSWORD=test_password_456
POSTGRES_DB=test_db
POSTGRES_PORT=5433
TZ=Asia/Seoul
PGTZ=Asia/Seoul
```

---

## 시작하기

### 컨테이너 시작

```bash
# 백그라운드에서 시작
docker compose up -d

# 로그 확인하며 시작 (Ctrl+C로 종료하면 컨테이너도 종료됨)
docker compose up

# 시작 후 로그 확인
docker compose logs -f postgresql
```

**예상 출력**:
```
[+] Running 2/2
 ✔ Network postgresql_postgres_network  Created
 ✔ Container postgresql                 Started
```

### 컨테이너 상태 확인

```bash
# 실행 중인 컨테이너 확인
docker compose ps

# 상세 정보 확인
docker ps --filter name=postgresql
```

**예상 출력**:
```
NAME         IMAGE         STATUS        PORTS
postgresql   postgres:17   Up 2 minutes  0.0.0.0:5432->5432/tcp
```

### 컨테이너 중지

```bash
# 중지 (컨테이너 유지, 데이터 보존)
docker compose stop

# 중지 및 제거 (데이터는 보존)
docker compose down

# 중지 및 모든 리소스 제거 (경고: 데이터도 삭제!)
docker compose down -v
```

### 컨테이너 재시작

```bash
# 재시작
docker compose restart

# 또는
docker restart postgresql
```

---

## 데이터베이스 접근

### 1. Docker Exec를 통한 접근

```bash
# psql로 접속 (기본 데이터베이스)
docker exec -it postgresql psql -U postgres -d default

# 특정 데이터베이스로 접속
docker exec -it postgresql psql -U postgres -d mydb

# 슈퍼유저로 접속
docker exec -it postgresql psql -U postgres
```

### 2. 호스트에서 직접 접속

PostgreSQL 클라이언트가 호스트에 설치되어 있다면:

```bash
# psql 설치 (Ubuntu/Debian)
sudo apt-get install postgresql-client

# 접속
psql -h localhost -p 5432 -U postgres -d default
```

**비밀번호 입력**: `.env` 파일에 설정한 `POSTGRES_PASSWORD` 입력

### 3. 외부 도구를 통한 접근

#### DBeaver
1. 연결 설정
   - Host: `localhost`
   - Port: `5432`
   - Database: `default`
   - Username: `postgres`
   - Password: `.env`의 `POSTGRES_PASSWORD`

#### pgAdmin
1. 서버 추가
   - Name: `PostgreSQL Docker`
   - Host: `localhost`
   - Port: `5432`
   - Maintenance database: `default`
   - Username: `postgres`
   - Password: `.env`의 `POSTGRES_PASSWORD`

#### DataGrip / IntelliJ IDEA
1. Database → New → Data Source → PostgreSQL
2. 연결 정보 입력

---

## CLI 사용법

### 기본 psql 명령어

#### 데이터베이스 접속
```bash
# 컨테이너 내부에서 psql 실행
docker exec -it postgresql psql -U postgres -d default
```

### psql 내부 메타 명령어

#### 데이터베이스 관리

```sql
-- 모든 데이터베이스 목록 보기
\l
-- 또는
\list

-- 현재 데이터베이스 확인
SELECT current_database();

-- 데이터베이스 생성
CREATE DATABASE myapp;

-- 데이터베이스 삭제
DROP DATABASE myapp;

-- 데이터베이스 전환
\c myapp
-- 또는
\connect myapp

-- 데이터베이스 크기 확인
SELECT
    pg_database.datname AS database_name,
    pg_size_pretty(pg_database_size(pg_database.datname)) AS size
FROM pg_database
ORDER BY pg_database_size(pg_database.datname) DESC;
```

**출력 예시**:
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

#### 테이블 관리

```sql
-- 현재 데이터베이스의 모든 테이블 목록
\dt

-- 스키마 포함 모든 테이블
\dt *.*

-- 테이블 상세 정보
\d table_name

-- 테이블 구조만 보기
\d+ table_name

-- 테이블 생성
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 테이블 삭제
DROP TABLE users;

-- 테이블 이름 변경
ALTER TABLE old_name RENAME TO new_name;

-- 테이블의 행 수 확인
SELECT COUNT(*) FROM users;
```

#### 스키마 관리

```sql
-- 모든 스키마 목록
\dn

-- 스키마 생성
CREATE SCHEMA app_schema;

-- 스키마 삭제
DROP SCHEMA app_schema CASCADE;

-- 현재 스키마 확인
SELECT current_schema();

-- 스키마 내 테이블 목록
\dt app_schema.*
```

#### 사용자 및 권한 관리

```sql
-- 모든 사용자(Role) 목록
\du

-- 사용자 생성
CREATE USER myuser WITH PASSWORD 'mypassword';

-- 사용자에게 데이터베이스 권한 부여
GRANT ALL PRIVILEGES ON DATABASE myapp TO myuser;

-- 특정 테이블 권한 부여
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLE users TO myuser;

-- 스키마 권한 부여
GRANT USAGE ON SCHEMA public TO myuser;
GRANT CREATE ON SCHEMA public TO myuser;

-- 사용자 삭제
DROP USER myuser;

-- 비밀번호 변경
ALTER USER myuser WITH PASSWORD 'new_password';

-- 슈퍼유저 권한 부여
ALTER USER myuser WITH SUPERUSER;
```

#### 인덱스 관리

```sql
-- 테이블의 인덱스 확인
\di

-- 인덱스 생성
CREATE INDEX idx_users_email ON users(email);

-- 유니크 인덱스 생성
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- 인덱스 삭제
DROP INDEX idx_users_email;

-- 인덱스 재생성
REINDEX TABLE users;
```

#### 데이터 조회 및 조작

```sql
-- 데이터 조회
SELECT * FROM users;
SELECT username, email FROM users WHERE id = 1;
SELECT * FROM users ORDER BY created_at DESC LIMIT 10;

-- 데이터 삽입
INSERT INTO users (username, email)
VALUES ('john_doe', 'john@example.com');

-- 여러 행 삽입
INSERT INTO users (username, email) VALUES
    ('alice', 'alice@example.com'),
    ('bob', 'bob@example.com'),
    ('charlie', 'charlie@example.com');

-- 데이터 수정
UPDATE users SET email = 'newemail@example.com' WHERE id = 1;

-- 데이터 삭제
DELETE FROM users WHERE id = 1;

-- 모든 데이터 삭제 (주의!)
TRUNCATE TABLE users;
```

#### 시퀀스 관리

```sql
-- 시퀀스 목록
\ds

-- 시퀀스 현재 값 확인
SELECT currval('users_id_seq');

-- 시퀀스 다음 값 확인
SELECT nextval('users_id_seq');

-- 시퀀스 재설정
ALTER SEQUENCE users_id_seq RESTART WITH 1;
```

#### 뷰 관리

```sql
-- 뷰 목록
\dv

-- 뷰 생성
CREATE VIEW active_users AS
SELECT * FROM users WHERE status = 'active';

-- 뷰 삭제
DROP VIEW active_users;
```

#### 함수 및 프로시저

```sql
-- 함수 목록
\df

-- 함수 생성
CREATE OR REPLACE FUNCTION get_user_count()
RETURNS INTEGER AS $$
BEGIN
    RETURN (SELECT COUNT(*) FROM users);
END;
$$ LANGUAGE plpgsql;

-- 함수 실행
SELECT get_user_count();

-- 함수 삭제
DROP FUNCTION get_user_count;
```

### 시스템 정보 및 모니터링

```sql
-- PostgreSQL 버전 확인
SELECT version();

-- 현재 시간 (서버 시간)
SELECT NOW();
SELECT CURRENT_TIMESTAMP;

-- 현재 타임존 확인
SHOW timezone;

-- 활성 연결 확인
SELECT
    pid,
    usename,
    datname,
    client_addr,
    state,
    query
FROM pg_stat_activity
WHERE state = 'active';

-- 모든 연결 확인
SELECT
    COUNT(*) as total_connections,
    COUNT(*) FILTER (WHERE state = 'active') as active_connections,
    COUNT(*) FILTER (WHERE state = 'idle') as idle_connections
FROM pg_stat_activity;

-- 특정 연결 종료
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE datname = 'myapp' AND pid <> pg_backend_pid();

-- 테이블 통계
SELECT
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size,
    n_live_tup AS live_rows,
    n_dead_tup AS dead_rows
FROM pg_stat_user_tables
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- 데이터베이스 통계
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

### 유틸리티 명령어

```sql
-- SQL 파일 실행
\i /path/to/script.sql

-- 쿼리 결과를 파일로 저장
\o /path/to/output.txt
SELECT * FROM users;
\o  -- 파일 출력 종료

-- CSV로 내보내기
\copy users TO '/tmp/users.csv' WITH CSV HEADER;

-- CSV에서 가져오기
\copy users FROM '/tmp/users.csv' WITH CSV HEADER;

-- 쿼리 실행 시간 표시
\timing on

-- 확장된 출력 모드 (세로 형식)
\x
-- 또는
\x auto

-- 페이저 끄기/켜기
\pset pager off
\pset pager on

-- psql 종료
\q
-- 또는
exit
```

### 백업 및 복원 (CLI)

#### 데이터베이스 백업
```bash
# 전체 데이터베이스 백업 (컨테이너 외부에서)
docker exec postgresql pg_dump -U postgres default > backup.sql

# 압축 백업
docker exec postgresql pg_dump -U postgres default | gzip > backup.sql.gz

# 커스텀 포맷 백업 (복원 시 더 유연함)
docker exec postgresql pg_dump -U postgres -Fc default > backup.dump

# 특정 테이블만 백업
docker exec postgresql pg_dump -U postgres -t users default > users_backup.sql

# 스키마만 백업 (데이터 제외)
docker exec postgresql pg_dump -U postgres -s default > schema_only.sql

# 데이터만 백업 (스키마 제외)
docker exec postgresql pg_dump -U postgres -a default > data_only.sql
```

#### 데이터베이스 복원
```bash
# SQL 파일 복원
cat backup.sql | docker exec -i postgresql psql -U postgres -d default

# 압축 파일 복원
gunzip -c backup.sql.gz | docker exec -i postgresql psql -U postgres -d default

# 커스텀 포맷 복원
docker exec -i postgresql pg_restore -U postgres -d default < backup.dump

# 새 데이터베이스 생성 후 복원
docker exec postgresql psql -U postgres -c "CREATE DATABASE myapp_restored;"
cat backup.sql | docker exec -i postgresql psql -U postgres -d myapp_restored
```

#### 전체 클러스터 백업
```bash
# 모든 데이터베이스 + 글로벌 객체 백업
docker exec postgresql pg_dumpall -U postgres > full_backup.sql

# 복원
cat full_backup.sql | docker exec -i postgresql psql -U postgres
```

### 성능 분석

```sql
-- 쿼리 실행 계획 보기
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- 실행 계획 + 실제 실행 통계
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';

-- 느린 쿼리 찾기 (실행 시간 순)
SELECT
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    max_exec_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- 인덱스 사용률 확인
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan as index_scans,
    idx_tup_read as tuples_read,
    idx_tup_fetch as tuples_fetched
FROM pg_stat_user_indexes
ORDER BY idx_scan ASC;

-- 사용하지 않는 인덱스 찾기
SELECT
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) as index_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY pg_relation_size(indexrelid) DESC;
```

### 트랜잭션 관리

```sql
-- 트랜잭션 시작
BEGIN;

-- 쿼리 실행
INSERT INTO users (username, email) VALUES ('test', 'test@example.com');
UPDATE users SET status = 'active' WHERE username = 'test';

-- 커밋 (변경사항 저장)
COMMIT;

-- 또는 롤백 (변경사항 취소)
ROLLBACK;

-- 세이브포인트 사용
BEGIN;
INSERT INTO users (username, email) VALUES ('user1', 'user1@example.com');
SAVEPOINT sp1;
INSERT INTO users (username, email) VALUES ('user2', 'user2@example.com');
ROLLBACK TO sp1;  -- user2 삽입만 취소
COMMIT;  -- user1 삽입은 저장
```

### 확장 기능 관리

```sql
-- 설치된 확장 목록
\dx

-- 사용 가능한 확장 목록
SELECT * FROM pg_available_extensions ORDER BY name;

-- 확장 설치
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";  -- 유사 문자열 검색
CREATE EXTENSION IF NOT EXISTS "hstore";   -- Key-Value 저장

-- 확장 삭제
DROP EXTENSION "uuid-ossp";

-- UUID 생성 예시
SELECT uuid_generate_v4();
```

### 유용한 psql 설정

```bash
# ~/.psqlrc 파일 생성 (psql 시작 시 자동 실행)
cat > ~/.psqlrc << 'EOF'
-- 자동 완성 대소문자 구분 안 함
\set COMP_KEYWORD_CASE upper

-- NULL 값 표시
\pset null '(null)'

-- 쿼리 실행 시간 표시
\timing

-- 프롬프트 커스터마이징
\set PROMPT1 '%n@%M:%> %x%# '

-- 히스토리 파일 크기 증가
\set HISTSIZE 10000

-- 에러 발생 시 롤백
\set ON_ERROR_ROLLBACK interactive
EOF
```

### Docker 컨테이너 내부에서 파일 작업

```bash
# 컨테이너로 파일 복사
docker cp backup.sql postgresql:/tmp/

# 컨테이너에서 파일 가져오기
docker cp postgresql:/tmp/export.sql ./

# 컨테이너 내부에서 SQL 실행
docker exec postgresql psql -U postgres -d default -f /tmp/backup.sql
```

---

## 데이터 관리

### 백업 전략

#### 1. 자동 백업 스크립트

```bash
#!/bin/bash
# backup.sh - PostgreSQL 자동 백업 스크립트

BACKUP_DIR="/path/to/backups"
DATE=$(date +"%Y%m%d_%H%M%S")
CONTAINER="postgresql"
DB_NAME="default"
DB_USER="postgres"

# 백업 디렉토리 생성
mkdir -p $BACKUP_DIR

# 백업 실행
docker exec $CONTAINER pg_dump -U $DB_USER $DB_NAME | gzip > "$BACKUP_DIR/backup_${DATE}.sql.gz"

# 7일 이상 된 백업 삭제
find $BACKUP_DIR -name "backup_*.sql.gz" -mtime +7 -delete

echo "Backup completed: backup_${DATE}.sql.gz"
```

**실행 권한 부여 및 테스트**:
```bash
chmod +x backup.sh
./backup.sh
```

#### 2. Cron 자동화

```bash
# crontab 편집
crontab -e

# 매일 새벽 2시에 백업 실행
0 2 * * * /path/to/backup.sh >> /var/log/postgresql_backup.log 2>&1
```

### 복원 절차

#### 전체 복원
```bash
# 1. 기존 컨테이너 중지
docker compose down

# 2. 데이터 디렉토리 백업 (선택사항)
mv postgresql_data postgresql_data.old

# 3. 컨테이너 시작
docker compose up -d

# 4. 백업 파일 복원
gunzip -c backup_20240101_020000.sql.gz | docker exec -i postgresql psql -U postgres -d default

# 5. 복원 확인
docker exec -it postgresql psql -U postgres -d default -c "\dt"
```

### 데이터 마이그레이션

#### 다른 PostgreSQL 버전에서 마이그레이션

```bash
# 1. 구 버전에서 백업
docker exec old_postgresql pg_dumpall -U postgres > full_backup.sql

# 2. 새 컨테이너 시작
docker compose up -d

# 3. 복원
cat full_backup.sql | docker exec -i postgresql psql -U postgres
```

---

## 모니터링 및 로깅

### 로그 확인

```bash
# 실시간 로그 확인
docker compose logs -f postgresql

# 최근 100줄 로그
docker compose logs --tail=100 postgresql

# 특정 시간 이후 로그
docker compose logs --since="2024-01-01T00:00:00" postgresql

# 타임스탬프 포함
docker compose logs -t postgresql
```

### 컨테이너 리소스 사용량

```bash
# 실시간 리소스 모니터링
docker stats postgresql

# CPU, 메모리 사용량 확인
docker stats postgresql --no-stream
```

**출력 예시**:
```
CONTAINER ID   NAME         CPU %     MEM USAGE / LIMIT     MEM %
abc123def456   postgresql   2.45%     256.3MiB / 2GiB      12.52%
```

### 디스크 사용량

```bash
# PostgreSQL 데이터 디렉토리 크기
du -sh postgresql_data/

# 상세 분석
du -h --max-depth=2 postgresql_data/
```

### 성능 모니터링 쿼리

```sql
-- 데이터베이스 연결 수
SELECT count(*) FROM pg_stat_activity;

-- 가장 큰 테이블 Top 10
SELECT
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_stat_user_tables
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 10;

-- 캐시 히트율 (95% 이상이 이상적)
SELECT
    sum(heap_blks_read) as heap_read,
    sum(heap_blks_hit)  as heap_hit,
    sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) * 100 as cache_hit_ratio
FROM pg_statio_user_tables;
```

---

## 문제 해결

### 일반적인 문제

#### 1. 컨테이너가 시작되지 않음

**증상**:
```bash
docker compose up -d
# Error: ...
```

**해결 방법**:

```bash
# 로그 확인
docker compose logs postgresql

# 흔한 원인:
# - POSTGRES_PASSWORD 미설정
# - 포트 충돌 (5432)
# - 데이터 디렉토리 권한 문제

# 포트 충돌 확인
sudo netstat -tulpn | grep 5432
# 또는
sudo lsof -i :5432

# 다른 프로세스가 사용 중이면 포트 변경
# .env 파일에서 POSTGRES_PORT=5433으로 변경
```

#### 2. 비밀번호 오류

**증상**:
```
FATAL: password authentication failed for user "postgres"
```

**해결 방법**:

```bash
# .env 파일 확인
cat .env | grep POSTGRES_PASSWORD

# 비밀번호 재설정 (데이터 손실 주의!)
docker compose down
rm -rf postgresql_data/
docker compose up -d
```

#### 3. 권한 문제

**증상**:
```
initdb: could not change permissions of directory "/var/lib/postgresql/data/pgdata"
```

**해결 방법**:

```bash
# 데이터 디렉토리 권한 수정
# PostgreSQL 컨테이너는 UID 999로 실행됨
sudo chown -R 999:999 postgresql_data/

# 또는 모든 사용자 접근 허용 (개발 환경)
chmod -R 777 postgresql_data/
```

#### 4. 연결 거부

**증상**:
```
could not connect to server: Connection refused
```

**해결 방법**:

```bash
# 컨테이너 실행 확인
docker ps | grep postgresql

# 네트워크 연결 확인
docker exec postgresql pg_isready -U postgres

# 포트 매핑 확인
docker port postgresql

# 방화벽 확인 (Linux)
sudo ufw status
sudo ufw allow 5432/tcp
```

#### 5. 디스크 공간 부족

**증상**:
```
ERROR: could not extend file: No space left on device
```

**해결 방법**:

```bash
# 디스크 사용량 확인
df -h

# 오래된 로그 정리
docker system prune -a

# WAL 파일 정리 (컨테이너 내부)
docker exec postgresql psql -U postgres -c "CHECKPOINT;"

# 데이터베이스 VACUUM
docker exec postgresql psql -U postgres -d default -c "VACUUM FULL;"
```

### 디버깅 도구

```bash
# 컨테이너 내부 접속
docker exec -it postgresql /bin/bash

# PostgreSQL 설정 파일 확인
docker exec postgresql cat /var/lib/postgresql/data/pgdata/postgresql.conf

# 로그 파일 위치 확인
docker exec postgresql psql -U postgres -c "SHOW log_directory;"
docker exec postgresql psql -U postgres -c "SHOW log_filename;"
```

### 성능 문제

```sql
-- 긴 실행 중인 쿼리 찾기
SELECT
    pid,
    now() - query_start as duration,
    query,
    state
FROM pg_stat_activity
WHERE state != 'idle'
AND query_start < now() - interval '5 minutes'
ORDER BY duration DESC;

-- 느린 쿼리 강제 종료
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE pid = 12345;

-- 락 대기 확인
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

## 보안 고려사항

### 네트워크 보안

#### 1. 포트 노출 제한

프로덕션 환경에서는 외부 접근 차단:

```yaml
# docker-compose.yml
services:
  postgresql:
    ports:
      - "127.0.0.1:5432:5432"  # 로컬호스트만 접근 가능
```

#### 2. 애플리케이션 전용 사용자 생성

```sql
-- 읽기 전용 사용자
CREATE USER readonly WITH PASSWORD 'secure_password';
GRANT CONNECT ON DATABASE default TO readonly;
GRANT USAGE ON SCHEMA public TO readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO readonly;

-- 애플리케이션 사용자 (CRUD)
CREATE USER appuser WITH PASSWORD 'app_secure_password';
GRANT CONNECT ON DATABASE default TO appuser;
GRANT USAGE, CREATE ON SCHEMA public TO appuser;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO appuser;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO appuser;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO appuser;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT USAGE, SELECT ON SEQUENCES TO appuser;
```

### 데이터 암호화

#### 1. SSL/TLS 연결 (프로덕션 권장)

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

#### 2. 비밀번호 정책

```sql
-- 비밀번호 만료 설정 (90일)
ALTER USER appuser VALID UNTIL '2024-04-01';

-- 연결 제한
ALTER USER appuser CONNECTION LIMIT 10;
```

### 감사 로깅

```sql
-- pg_stat_statements 확장 설치
CREATE EXTENSION pg_stat_statements;

-- 모든 DDL 명령 로깅 (postgresql.conf)
-- log_statement = 'ddl'
```

### 정기 보안 점검 체크리스트

- [ ] 기본 postgres 사용자 비밀번호 변경
- [ ] 불필요한 데이터베이스 삭제
- [ ] 사용하지 않는 사용자 제거
- [ ] 권한 최소화 원칙 적용
- [ ] 정기적인 백업 확인
- [ ] PostgreSQL 버전 업데이트 확인
- [ ] 로그 정기 검토
- [ ] 비정상 접속 시도 모니터링

---

## 성능 튜닝

### PostgreSQL 설정 최적화

#### 1. 메모리 설정

컨테이너 내부에서 설정 조정:

```bash
# postgresql.conf 편집
docker exec -it postgresql bash
vi /var/lib/postgresql/data/pgdata/postgresql.conf
```

**권장 설정** (4GB RAM 시스템 기준):

```conf
# Memory Settings
shared_buffers = 1GB                    # 전체 메모리의 25%
effective_cache_size = 3GB              # 전체 메모리의 75%
maintenance_work_mem = 256MB            # RAM의 5-10%
work_mem = 16MB                         # 동시 연결 고려

# Checkpoint Settings
checkpoint_completion_target = 0.9
wal_buffers = 16MB
max_wal_size = 2GB
min_wal_size = 1GB

# Query Planner
random_page_cost = 1.1                  # SSD 사용 시
effective_io_concurrency = 200          # SSD 사용 시
```

**설정 후 재시작**:
```bash
docker compose restart
```

#### 2. 연결 풀링

**외부 PgBouncer 사용**:

```yaml
# docker-compose.yml에 추가
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

### 인덱스 최적화

```sql
-- 중복/사용하지 않는 인덱스 찾기
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

-- 테이블 스캔이 많은 테이블 찾기 (인덱스 필요)
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

### VACUUM 및 ANALYZE

```sql
-- 수동 VACUUM
VACUUM ANALYZE;

-- 특정 테이블 VACUUM
VACUUM ANALYZE users;

-- VACUUM FULL (테이블 잠금, 주의!)
VACUUM FULL users;

-- 자동 VACUUM 설정 확인
SHOW autovacuum;

-- 테이블별 VACUUM 통계
SELECT
    schemaname,
    tablename,
    last_vacuum,
    last_autovacuum,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables;
```

### 쿼리 최적화

```sql
-- 실행 계획 분석
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM users WHERE email = 'test@example.com';

-- 통계 업데이트
ANALYZE users;

-- 테이블 통계 확인
SELECT * FROM pg_stats WHERE tablename = 'users';
```

---

## 참고 자료

### 공식 문서
- [PostgreSQL 17 공식 문서](https://www.postgresql.org/docs/17/)
- [Docker Hub - PostgreSQL](https://hub.docker.com/_/postgres)
- [Docker Compose 문서](https://docs.docker.com/compose/)

### 유용한 도구
- [pgAdmin](https://www.pgadmin.org/) - 웹 기반 관리 도구
- [DBeaver](https://dbeaver.io/) - 크로스 플랫폼 DB 클라이언트
- [DataGrip](https://www.jetbrains.com/datagrip/) - JetBrains DB IDE
- [TablePlus](https://tableplus.com/) - 현대적인 DB 클라이언트
- [Adminer](https://www.adminer.org/) - 경량 웹 DB 관리

### 학습 자료
- [PostgreSQL Tutorial](https://www.postgresqltutorial.com/)
- [Use The Index, Luke](https://use-the-index-luke.com/) - SQL 인덱싱 가이드
- [PostgreSQL Performance](https://www.postgresql.org/docs/current/performance-tips.html)

### 커뮤니티
- [PostgreSQL Slack](https://postgres-slack.herokuapp.com/)
- [Stack Overflow - PostgreSQL](https://stackoverflow.com/questions/tagged/postgresql)
- [Reddit - r/PostgreSQL](https://www.reddit.com/r/PostgreSQL/)

### 모니터링 도구
- [pgMonitor](https://github.com/CrunchyData/pgmonitor)
- [pg_stat_statements](https://www.postgresql.org/docs/current/pgstatstatements.html)
- [Prometheus + Grafana](https://grafana.com/grafana/dashboards/9628)

---

## 라이선스

이 프로젝트는 PostgreSQL의 라이선스를 따릅니다.

- PostgreSQL: [PostgreSQL License](https://www.postgresql.org/about/licence/)

---

## 문의 및 지원

문제 발생 시:
1. 이 README의 [문제 해결](#문제-해결) 섹션 확인
2. [PostgreSQL 공식 문서](https://www.postgresql.org/docs/17/) 참조
3. 로그 확인: `docker compose logs postgresql`

**버전 정보**:
- 문서 버전: 1.0.0
- 최종 업데이트: 2025-10-18
- PostgreSQL 버전: 17

---

**🎉 PostgreSQL 17 Docker 컨테이너를 사용해주셔서 감사합니다!**
