# Docker 기초

## Docker가 뭔가요?

Docker는 **"앱을 컨테이너라는 박스에 포장해서 어디서든 똑같이 실행할 수 있게 해주는 도구"**예요.

**비유:**
- Docker = 컨테이너 선박 🚢
- 앱 = 화물 📦
- 컨테이너 = 표준화된 화물 컨테이너

## 1. 왜 Docker가 필요한가요?

### 문제 상황

```bash
# 개발자 A 컴퓨터
Node.js 18, Ubuntu 20.04, MySQL 8.0
→ "내 컴퓨터에서는 잘 되는데?"

# 개발자 B 컴퓨터  
Node.js 16, Windows 11, MySQL 5.7
→ "에러가 나요! 😭"

# 서버
Node.js 20, CentOS 7, PostgreSQL 14
→ "배포했는데 안 돌아가요! 😱"
```

### Docker 해결책

```bash
# 모든 환경에서
docker run my-app
→ "어디서든 똑같이 동작! 🎉"
```

## 2. Docker 기본 개념들

### A) 이미지 (Image)

**이미지 = 앱을 실행하는데 필요한 모든 것을 포장한 템플릿**

```bash
# 공식 이미지들
docker pull node:18          # Node.js 18 환경
docker pull nginx:latest     # Nginx 웹서버
docker pull postgres:15      # PostgreSQL 15 데이터베이스
```

### B) 컨테이너 (Container)

**컨테이너 = 이미지로 만든 실행 중인 앱**

```bash
# 이미지로 컨테이너 실행
docker run -d --name my-app node:18
# 이미지 = 붕어빵 틀, 컨테이너 = 실제 붕어빵
```

### C) Dockerfile

**Dockerfile = 이미지를 만드는 설계도**

```dockerfile
FROM node:18              # Node.js 18 기반으로
COPY . /app              # 내 코드 복사
WORKDIR /app             # /app 폴더로 이동
RUN npm install          # 패키지 설치
CMD ["npm", "start"]     # 앱 실행
```

## 3. 기본 명령어들

### 이미지 관련

```bash
# 이미지 다운로드
docker pull nginx:latest

# 이미지 목록 보기
docker images

# 이미지 삭제
docker rmi nginx:latest
```

### 컨테이너 관련

```bash
# 컨테이너 실행
docker run -d --name my-nginx nginx:latest

# 실행 중인 컨테이너 보기
docker ps

# 모든 컨테이너 보기 (정지된 것 포함)
docker ps -a

# 컨테이너 정지
docker stop my-nginx

# 컨테이너 삭제
docker rm my-nginx

# 컨테이너 로그 보기
docker logs my-nginx
```

## 4. 실습: 간단한 Express 앱 Docker화

### Step 1: Express 앱 준비

```javascript
// app.js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.json({ 
    message: 'Docker에서 실행되는 Express 앱!',
    timestamp: new Date().toISOString(),
    hostname: require('os').hostname() // 컨테이너 ID
  });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`서버가 ${PORT}번 포트에서 실행중입니다.`);
});
```

```json
// package.json
{
  "name": "docker-express-app",
  "version": "1.0.0",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}
```

### Step 2: Dockerfile 작성

```dockerfile
# Dockerfile
FROM node:18-alpine          # 가벼운 Node.js 18 이미지

WORKDIR /app                 # 작업 디렉토리 설정

COPY package*.json ./        # package.json 먼저 복사
RUN npm install             # 패키지 설치

COPY . .                    # 나머지 파일들 복사

EXPOSE 3000                 # 3000번 포트 사용한다고 알림

CMD ["npm", "start"]        # 앱 실행 명령어
```

### Step 3: 이미지 빌드 및 실행

```bash
# 이미지 빌드
docker build -t my-express-app .

# 컨테이너 실행
docker run -d \
  --name express-container \
  -p 3000:3000 \
  my-express-app

# 테스트
curl http://localhost:3000
```

**결과:**
```json
{
  "message": "Docker에서 실행되는 Express 앱!",
  "timestamp": "2024-01-01T12:00:00.000Z",
  "hostname": "a1b2c3d4e5f6"
}
```

## 5. server.js에서 Docker 사용 예시

server.js에서 Docker 명령어를 많이 사용해요:

### A) PostgreSQL 배포 (101-215줄)

```javascript
// Docker로 PostgreSQL 컨테이너 실행
const runCommand = `docker run -d \
  --name management_postgres \
  --restart unless-stopped \
  --network management_network \
  -e POSTGRES_DB=management \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=gudqls93 \
  --expose 5432 \
  -v management_postgres_data:/var/lib/postgresql/data \
  postgres:15-alpine`;

await execAsync(runCommand);
```

### B) 백엔드 배포 (259줄)

```javascript
// 새 이미지 다운로드
await execAsync(`docker pull ${imageName}`);

// 기존 컨테이너 정리
await execAsync(`docker stop ${containerName}`);
await execAsync(`docker rm ${containerName}`);

// 새 컨테이너 실행
const result = await spawnAsync('docker', dockerArgs);
```

## 6. Docker 네트워크

### 네트워크가 왜 필요한가요?

```bash
# 문제: 컨테이너들이 서로 통신 못함
docker run -d --name db postgres:15
docker run -d --name app my-app        # app이 db에 연결 못함

# 해결: 같은 네트워크에 연결
docker network create my-network
docker run -d --name db --network my-network postgres:15
docker run -d --name app --network my-network my-app  # db에 연결 가능!
```

### server.js의 네트워크 사용

```javascript
// management_network 생성
await execAsync(`docker network create management_network`);

// PostgreSQL을 네트워크에 연결
docker run -d \
  --name management_postgres \
  --network management_network \  // 👈 네트워크 연결
  postgres:15-alpine

// 백엔드도 같은 네트워크에 연결
docker run -d \
  --name management \
  --network management_network \  // 👈 같은 네트워크
  my-backend-app

// 이제 백엔드에서 'postgres' 호스트명으로 DB 접근 가능!
```

## 7. Docker Compose 맛보기

여러 컨테이너를 한 번에 관리하는 도구:

```yaml
# docker-compose.yml
version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: myapp
      POSTGRES_PASSWORD: password
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - app_network

  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      DB_HOST: db  # 👈 서비스명으로 접근!
      DB_PASSWORD: password
    depends_on:
      - db
    networks:
      - app_network

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    depends_on:
      - backend
    networks:
      - app_network

networks:
  app_network:

volumes:
  db_data:
```

```bash
# 모든 서비스 시작
docker-compose up -d

# 특정 서비스만 재시작
docker-compose restart backend

# 모든 서비스 중지
docker-compose down
```

## 8. 실전 패턴들

### A) 개발환경 vs 실제환경

```dockerfile
# Dockerfile.dev (개발용)
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]  # nodemon 사용

# Dockerfile.prod (실제환경용)
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production  # 개발 의존성 제외
COPY . .
USER node  # 보안을 위해 root가 아닌 user로 실행
CMD ["npm", "start"]
```

### B) 멀티스테이지 빌드

```dockerfile
# 1단계: 빌드 환경
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# 2단계: 실행 환경 (더 작음)
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist  # 빌드 결과만 복사
CMD ["npm", "start"]
```

## 9. Docker 볼륨 (데이터 보관)

```bash
# 문제: 컨테이너 삭제시 데이터 사라짐
docker run -d --name db postgres:15
docker rm db  # 데이터베이스 내용 모두 사라짐! 😱

# 해결: 볼륨 사용
docker volume create my_db_data
docker run -d \
  --name db \
  -v my_db_data:/var/lib/postgresql/data \
  postgres:15
docker rm db  # 컨테이너는 사라져도 데이터는 볼륨에 보관됨! 🎉
```

## 10. 유용한 Docker 명령어들

```bash
# 컨테이너 안으로 들어가기
docker exec -it my-container /bin/bash

# 컨테이너 → 호스트 파일 복사
docker cp my-container:/app/logs ./logs

# 호스트 → 컨테이너 파일 복사
docker cp ./config.json my-container:/app/

# 컨테이너 리소스 사용량 확인
docker stats my-container

# 사용하지 않는 이미지/컨테이너 정리
docker system prune -f

# 모든 컨테이너 강제 삭제
docker rm -f $(docker ps -aq)
```

## 11. 실습 과제

간단한 TODO API를 Docker로 만들어보세요:

```javascript
// todo-app.js
const express = require('express');
const app = express();

app.use(express.json());

let todos = [
  { id: 1, text: 'Docker 배우기', done: false },
  { id: 2, text: 'Express 마스터하기', done: true }
];

app.get('/todos', (req, res) => {
  res.json(todos);
});

app.post('/todos', (req, res) => {
  const todo = {
    id: todos.length + 1,
    text: req.body.text,
    done: false
  };
  todos.push(todo);
  res.json(todo);
});

app.listen(4000, () => {
  console.log('TODO API가 4000번 포트에서 실행중');
});
```

```dockerfile
# Dockerfile 만들어보세요!
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 4000
CMD ["node", "todo-app.js"]
```

```bash
# 빌드하고 실행해보세요!
docker build -t todo-api .
docker run -d -p 4000:4000 --name todo todo-api

# 테스트
curl http://localhost:4000/todos
curl -X POST http://localhost:4000/todos \
  -H "Content-Type: application/json" \
  -d '{"text": "새로운 할일"}'
```

## 12. Docker에서 환경변수 사용

```dockerfile
# Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000

# 환경변수 기본값 설정
ENV NODE_ENV=production
ENV PORT=3000

CMD ["npm", "start"]
```

```bash
# 실행시 환경변수 오버라이드
docker run -d \
  --name my-app \
  -p 3000:3000 \
  -e NODE_ENV=development \
  -e DB_HOST=localhost \
  -e DB_PASSWORD=secret \
  my-app
```

## 13. 실전 예시: 풀스택 애플리케이션

```yaml
# docker-compose.yml
version: '3.8'

services:
  # 데이터베이스
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  # 백엔드 API
  backend:
    build: ./backend
    environment:
      DB_HOST: postgres
      DB_PASSWORD: password
      JWT_SECRET: mysecret
    ports:
      - "8000:8000"
    depends_on:
      - postgres
    volumes:
      - ./backend:/app
      - /app/node_modules

  # 프론트엔드
  frontend:
    build: ./frontend
    environment:
      REACT_APP_API_URL: http://localhost:8000
    ports:
      - "3000:3000"
    depends_on:
      - backend
    volumes:
      - ./frontend:/app
      - /app/node_modules

  # Redis 캐시
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

```bash
# 전체 스택 실행
docker-compose up -d

# 로그 확인
docker-compose logs -f backend

# 특정 서비스 재시작
docker-compose restart backend

# 전체 정리
docker-compose down -v
```

## 핵심 정리

**Docker의 핵심 개념:**
1. **이미지**: 앱을 실행할 수 있는 완전한 패키지
2. **컨테이너**: 이미지를 실행한 인스턴스
3. **Dockerfile**: 이미지를 만드는 설계도
4. **볼륨**: 데이터 영구 보관
5. **네트워크**: 컨테이너 간 통신
6. **Compose**: 여러 컨테이너 관리

**Docker의 장점:**
- 어디서든 동일한 실행 환경
- 빠른 배포와 확장
- 격리된 환경으로 안전성
- 쉬운 버전 관리
- 효율적인 리소스 사용