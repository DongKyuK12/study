# 13. Docker 심화 - Dockerfile, Docker-Compose, pgAdmin

## 📚 학습 목표
- Dockerfile 기본 작성법 완전 이해
- docker-compose 환경변수 주입 방법
- pgAdmin 웹 인터페이스 설정 및 PostgreSQL 연결

---

## 1️⃣ Dockerfile 기본 작성법 (상세)

### 기본 구조와 명령어들

```dockerfile
# 1. 베이스 이미지 지정 (필수)
FROM node:18-alpine
# alpine = 경량화된 리눅스 배포판 (용량 작음)

# 2. 메타데이터 (선택사항)
LABEL maintainer="your-email@example.com"
LABEL version="1.0"
LABEL description="My Node.js App"

# 3. 작업 디렉토리 설정
WORKDIR /app
# 컨테이너 내부에서 /app 폴더를 기본 작업 위치로 설정

# 4. 파일 복사 (의존성 먼저!)
COPY package*.json ./
# package.json, package-lock.json을 현재 디렉토리(.)에 복사

# 5. 의존성 설치
RUN npm ci --only=production
# npm ci = npm install보다 빠르고 안정적

# 6. 소스코드 복사
COPY . .
# 현재 폴더의 모든 파일을 컨테이너 /app에 복사

# 7. 불필요한 파일 제거 (선택사항)
RUN rm -rf node_modules/.cache

# 8. 사용자 권한 설정 (보안)
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001
USER nextjs

# 9. 포트 노출 (문서화 목적)
EXPOSE 3000

# 10. 헬스체크 (선택사항)
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# 11. 실행 명령어
CMD ["npm", "start"]
```

### 최적화 팁들

```dockerfile
# ❌ 비효율적 (매번 의존성 재설치)
COPY . .
RUN npm install

# ✅ 효율적 (Docker 레이어 캐싱 활용)
COPY package*.json ./
RUN npm install
COPY . .
```

**핵심 원리**: 자주 변경되지 않는 부분(의존성)을 먼저 처리하여 캐싱 효과를 극대화!

---

## 2️⃣ Docker-Compose 환경변수 주입

### 방법 1: .env 파일 사용 (가장 추천!)

**프로젝트 루트에 .env 파일 생성:**
```bash
# .env
NODE_ENV=development
DATABASE_URL=postgresql://postgres:password@database:5432/myapp
API_PORT=3000
DB_PORT=5432
POSTGRES_DB=myapp
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password
PGADMIN_EMAIL=admin@example.com
PGADMIN_PASSWORD=admin123
PGADMIN_PORT=8080
```

**docker-compose.yml에서 사용:**
```yaml
version: '3.8'

services:
  backend:
    build: .
    environment:
      - NODE_ENV=${NODE_ENV}
      - DATABASE_URL=${DATABASE_URL}
    ports:
      - "${API_PORT}:3000"  # .env에서 가져옴
    depends_on:
      - database

  database:
    image: postgres:15
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    ports:
      - "${DB_PORT}:5432"
```

### 방법 2: 환경별 .env 파일

```bash
# .env.development
NODE_ENV=development
API_PORT=3000

# .env.production  
NODE_ENV=production
API_PORT=8080
```

```bash
# 실행 시 환경 지정
docker-compose --env-file .env.development up
```

### 방법 3: docker-compose 내부에서 직접 설정

```yaml
services:
  backend:
    environment:
      NODE_ENV: development
      DATABASE_URL: postgresql://postgres:password@database:5432/myapp
      # 또는 리스트 형태
      - NODE_ENV=development
      - DATABASE_URL=postgresql://postgres:password@database:5432/myapp
```

---

## 3️⃣ pgAdmin 웹 인터페이스 설정

### 완성된 docker-compose.yml 예시

```yaml
version: '3.8'

services:
  # Node.js 백엔드
  backend:
    build: .
    environment:
      - NODE_ENV=${NODE_ENV:-development}
      - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@database:5432/${POSTGRES_DB}
    ports:
      - "${API_PORT:-3000}:3000"
    depends_on:
      - database
    volumes:
      - .:/app
      - /app/node_modules

  # PostgreSQL 데이터베이스
  database:
    image: postgres:15
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-myapp}
      POSTGRES_USER: ${POSTGRES_USER:-postgres}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-password}
    ports:
      - "${DB_PORT:-5432}:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql

  # pgAdmin 웹 인터페이스
  pgadmin:
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_EMAIL:-admin@example.com}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_PASSWORD:-admin123}
      PGADMIN_CONFIG_SERVER_MODE: 'False'  # 개발용 설정
    ports:
      - "${PGADMIN_PORT:-8080}:80"
    volumes:
      - pgadmin_data:/var/lib/pgadmin
    depends_on:
      - database

volumes:
  postgres_data:
  pgadmin_data:

networks:
  default:
    name: myapp_network
```

---

## 4️⃣ pgAdmin에서 PostgreSQL 컨테이너 연결

### 단계별 연결 방법

#### 1단계: 컨테이너 실행
```bash
docker-compose up -d
```

#### 2단계: pgAdmin 웹 접속
- 브라우저에서 `http://localhost:8080` 접속
- 로그인: 
  - Email: `admin@example.com`
  - Password: `admin123`

#### 3단계: 서버 등록

**General 탭:**
- Name: `My PostgreSQL Server` (아무 이름)

**Connection 탭:** ⭐ **중요한 부분!**
- **Host name/address**: `database` ← (컨테이너 이름!)
- **Port**: `5432`
- **Username**: `postgres`
- **Password**: `password`

> **⚠️ 주의**: Host는 `localhost`가 아니라 `database`입니다! 
> Docker 내부에서는 서비스명으로 통신합니다!

#### 4단계: 연결 테스트
- "Save" 버튼 클릭
- 연결 성공하면 왼쪽 트리에 서버가 나타남
- Database → myapp → Schemas → Tables 확인

### 연결 안될 때 문제해결

```bash
# 1. 컨테이너 상태 확인
docker-compose ps

# 2. PostgreSQL 로그 확인
docker-compose logs database

# 3. pgAdmin 로그 확인  
docker-compose logs pgadmin

# 4. 네트워크 확인
docker network ls
docker network inspect [네트워크명]
```

---

## 🎯 실전 실행 순서

```bash
# 1. 프로젝트 폴더 이동
cd /path/to/your/project

# 2. .env 파일 확인
cat .env

# 3. 컨테이너 시작
docker-compose up -d

# 4. 상태 확인
docker-compose ps

# 5. pgAdmin 접속: http://localhost:8080
# 6. PostgreSQL 연결: Host=database, Port=5432
```

---

## 📝 핵심 포인트 정리

1. **Dockerfile**: 의존성 먼저 복사해서 캐싱 최적화!
2. **환경변수**: .env 파일 사용해서 깔끔하게 관리!  
3. **pgAdmin**: Host명은 `database` (컨테이너 이름)!
4. **포트**: .env에서 관리해서 유연하게!
5. **네트워킹**: Docker Compose는 자동으로 네트워크를 생성하여 서비스명으로 통신 가능

---

## 🔍 추가 학습 자료

- [Docker 공식 문서](https://docs.docker.com/)
- [Docker Compose 문서](https://docs.docker.com/compose/)
- [pgAdmin 공식 문서](https://www.pgadmin.org/docs/)

> **클라드 선생님의 한마디**: Docker는 개발환경을 표준화하는 최고의 도구입니다. 한번 익히면 어디서든 동일한 환경에서 개발할 수 있어요! 화이팅! ♡