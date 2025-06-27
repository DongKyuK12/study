# Docker 네트워크

## Docker 네트워크가 뭔가요?

Docker 네트워크는 **"컨테이너들이 서로 통신할 수 있게 해주는 가상의 네트워크"**예요.

### 왜 필요한가요?

```bash
# 문제 상황
docker run -d --name db postgres:15
docker run -d --name api my-backend

# API에서 DB에 연결하려고 하면...
# "localhost:5432"로 접근? ❌ 안됨!
# 각각 독립된 컨테이너라서 서로 못 찾음
```

## 1. 네트워크 비유: 아파트 단지

### 네트워크 없이 (문제)

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   컨테이너1   │    │   컨테이너2   │    │   컨테이너3   │
│             │    │             │    │             │
│   데이터베이스 │    │  백엔드 API  │    │  프론트엔드   │
│             │    │             │    │             │
└─────────────┘    └─────────────┘    └─────────────┘
      🏝️              🏝️              🏝️
   (고립된 섬)        (고립된 섬)        (고립된 섬)

❌ 서로 통신 불가능!
```

### 네트워크 있음 (해결)

```
        Docker Network (가상 아파트 단지)
┌─────────────────────────────────────────────────────┐
│                                                     │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐ │
│  │ Container1  │    │ Container2  │    │ Container3  │ │
│  │   101호     │    │   102호     │    │   103호     │ │
│  │ 데이터베이스  │◄──►│ 백엔드 API   │◄──►│ 프론트엔드   │ │
│  │             │    │             │    │             │ │
│  └─────────────┘    └─────────────┘    └─────────────┘ │
│                                                     │
└─────────────────────────────────────────────────────┘

✅ 서로 통신 가능!
```

## 2. 기본 네트워크 명령어들

### 네트워크 관리

```bash
# 네트워크 목록 확인
docker network ls
# NETWORK ID     NAME      DRIVER    SCOPE
# abcd1234       bridge    bridge    local    ← 기본 네트워크
# efgh5678       host      host      local
# ijkl9012       none      null      local

# 새 네트워크 생성
docker network create my-network

# 네트워크 상세 정보
docker network inspect my-network

# 네트워크 삭제
docker network rm my-network
```

### 컨테이너를 네트워크에 연결

```bash
# 컨테이너 실행시 네트워크 지정
docker run -d --name db --network my-network postgres:15

# 실행 중인 컨테이너를 네트워크에 연결
docker network connect my-network existing-container

# 네트워크에서 분리
docker network disconnect my-network container-name
```

## 3. 네트워크 드라이버 종류

### 1. Bridge 네트워크 (기본)

**용도:** 같은 호스트의 컨테이너들끼리 통신

```bash
# 브리지 네트워크 생성
docker network create --driver bridge my-bridge

# 컨테이너들 연결
docker run -d --name web --network my-bridge nginx
docker run -d --name api --network my-bridge my-api

# web 컨테이너에서 api 컨테이너에 접근
docker exec web curl http://api:3000
# ↑ 'api'라는 컨테이너 이름으로 접근 가능!
```

### 2. Host 네트워크

**용도:** 컨테이너가 호스트의 네트워크를 직접 사용

```bash
# 호스트 네트워크 사용
docker run -d --network host nginx

# 포트 매핑 불필요 (호스트와 동일한 네트워크)
curl http://localhost:80  # 바로 접근 가능
```

### 3. None 네트워크

**용도:** 네트워크 없이 완전히 격리

```bash
# 네트워크 없이 실행
docker run -d --network none my-isolated-app
# 어디와도 통신 불가능
```

## 4. 실전 예시: 웹 애플리케이션 스택

### 문제 상황: 네트워크 없이

```bash
# 각각 따로 실행
docker run -d --name postgres \
  -e POSTGRES_PASSWORD=password \
  postgres:15

docker run -d --name backend \
  -e DB_HOST=localhost \    # ❌ 잘못된 설정
  -p 8000:8000 \
  my-backend

docker run -d --name frontend \
  -p 3000:3000 \
  my-frontend

# 결과: backend가 postgres에 연결 못함!
```

### 해결: 네트워크로 연결

```bash
# 1. 네트워크 생성
docker network create app-network

# 2. 데이터베이스 실행
docker run -d --name postgres \
  --network app-network \
  -e POSTGRES_PASSWORD=password \
  postgres:15

# 3. 백엔드 실행
docker run -d --name backend \
  --network app-network \
  -e DB_HOST=postgres \     # ✅ 컨테이너 이름으로 접근!
  -p 8000:8000 \
  my-backend

# 4. 프론트엔드 실행
docker run -d --name frontend \
  --network app-network \
  -p 3000:3000 \
  my-frontend

# 이제 backend에서 postgres에 연결 가능!
```

## 5. 네트워크 통신 테스트

### 실습: 간단한 네트워크 테스트

```bash
# 네트워크 생성
docker network create test-network

# 첫 번째 컨테이너 (서버 역할)
docker run -d --name server \
  --network test-network \
  nginx

# 두 번째 컨테이너 (클라이언트 역할)
docker run -it --name client \
  --network test-network \
  alpine sh

# 클라이언트에서 서버에 접근 테스트
# (alpine 컨테이너 안에서)
wget -qO- http://server
# nginx 기본 페이지가 출력되면 성공!

ping server
# 네트워크 연결 확인
```

### DNS 이름 해석 확인

```bash
# 컨테이너 안에서 DNS 확인
docker exec client nslookup server
# Name:      server
# Address 1: 172.18.0.2 server

# 실제 IP 확인
docker exec server hostname -i
# 172.18.0.2

# 같은 IP! DNS가 제대로 작동하고 있음
```

## 6. server.js에서 네트워크 사용 사례

### management_network 생성

server.js 113-122줄:

```javascript
// 1단계: 네트워크 생성 (존재하지 않는 경우)
try {
  await execAsync(`docker network create management_network`);
  log(`✅ Network 'management_network' created`);
} catch (networkError) {
  if (networkError.message.includes('already exists')) {
    log(`ℹ️ Network 'management_network' already exists`);
  } else {
    throw networkError;
  }
}
```

### 컨테이너들을 같은 네트워크에 연결

```javascript
// PostgreSQL 컨테이너
const runCommand = `docker run -d \
  --name management_postgres \
  --restart unless-stopped \
  --network management_network \     // 👈 같은 네트워크
  --network-alias postgres \         // 👈 'postgres'라는 별명
  postgres:15-alpine`;

// 백엔드 컨테이너  
const dockerArgs = [
  'run', '-d',
  '--name', 'management',
  '--restart', 'unless-stopped',
  '--network', 'management_network', // 👈 같은 네트워크
  '-e', `PG_HOST=postgres`,          // 👈 별명으로 접근!
  // ...
];
```

**결과:** 백엔드에서 `postgres:5432`로 데이터베이스에 연결 가능!

## 7. 포트 매핑 vs 네트워크

### 포트 매핑 (-p)

```bash
# 호스트 → 컨테이너 연결
docker run -d -p 8080:80 nginx

외부 세상 → 호스트:8080 → 컨테이너:80
```

### 네트워크 (--network)

```bash
# 컨테이너 ↔ 컨테이너 연결
docker run -d --network my-net nginx

컨테이너A ↔ my-net ↔ 컨테이너B
```

### 둘 다 사용하는 경우

```bash
# 프론트엔드: 외부 접근 + 내부 통신
docker run -d \
  --name frontend \
  --network app-network \    # 내부 통신용
  -p 3000:3000 \            # 외부 접근용
  my-frontend

# 백엔드: 내부 통신만
docker run -d \
  --name backend \
  --network app-network \    # 내부 통신만
  my-backend
```

## 8. Docker Compose로 네트워크 관리

### 수동 설정 (번거로움)

```bash
docker network create myapp-network
docker run -d --name db --network myapp-network postgres
docker run -d --name api --network myapp-network my-api
docker run -d --name web --network myapp-network my-web
```

### Compose로 자동화

```yaml
# docker-compose.yml
version: '3.8'

services:
  db:
    image: postgres:15
    networks:
      - app-network

  api:
    build: ./api
    networks:
      - app-network
    environment:
      - DB_HOST=db    # 서비스 이름으로 접근!

  web:
    build: ./web
    ports:
      - "3000:3000"
    networks:
      - app-network

networks:
  app-network:    # 자동으로 네트워크 생성!
```

```bash
# 한 번에 모든 서비스 + 네트워크 생성
docker-compose up -d

# 네트워크 확인
docker network ls
# myproject_app-network 자동 생성됨!
```

## 9. 네트워크 보안

### 네트워크 격리

```bash
# 프론트엔드 네트워크
docker network create frontend-network

# 백엔드 네트워크  
docker network create backend-network

# 프론트엔드는 백엔드 네트워크에만
docker run -d --name web --network frontend-network my-web

# API는 두 네트워크 모두 연결
docker run -d --name api my-api
docker network connect frontend-network api
docker network connect backend-network api

# 데이터베이스는 백엔드 네트워크에만
docker run -d --name db --network backend-network postgres

# 결과: web은 db에 직접 접근 불가능!
```

## 10. 실습 프로젝트: 채팅 앱

간단한 채팅 앱으로 네트워크 이해해보기:

### 1. Redis 서버 (메시지 저장)

```bash
docker network create chat-network

docker run -d \
  --name redis \
  --network chat-network \
  redis:alpine
```

### 2. 채팅 서버

```javascript
// chat-server.js
const express = require('express');
const redis = require('redis');

const app = express();
const client = redis.createClient({
  host: 'redis',  // 👈 컨테이너 이름으로 접근!
  port: 6379
});

app.use(express.json());

app.post('/message', async (req, res) => {
  const { user, message } = req.body;
  await client.lpush('messages', JSON.stringify({ user, message, time: new Date() }));
  res.json({ success: true });
});

app.get('/messages', async (req, res) => {
  const messages = await client.lrange('messages', 0, 9);
  res.json(messages.map(m => JSON.parse(m)));
});

app.listen(8000, () => {
  console.log('채팅 서버 실행 중: 8000');
});
```

```dockerfile
# Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 8000
CMD ["node", "chat-server.js"]
```

```bash
# 채팅 서버 실행
docker build -t chat-server .
docker run -d \
  --name chat-api \
  --network chat-network \
  -p 8000:8000 \
  chat-server
```

### 3. 테스트

```bash
# 메시지 전송
curl -X POST http://localhost:8000/message \
  -H "Content-Type: application/json" \
  -d '{"user": "홍길동", "message": "안녕하세요!"}'

# 메시지 조회
curl http://localhost:8000/messages
```

### 4. 네트워크 상태 확인

```bash
# 네트워크에 연결된 컨테이너들 확인
docker network inspect chat-network

# chat-api에서 redis로 ping 테스트
docker exec chat-api ping redis
```

## 11. 고급 네트워크 설정

### 사용자 정의 IP 범위

```bash
# 특정 IP 대역 사용
docker network create \
  --driver bridge \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  custom-network

# 컨테이너에 고정 IP 할당
docker run -d \
  --name web \
  --network custom-network \
  --ip 192.168.1.10 \
  nginx
```

### 외부 네트워크 연결

```bash
# 기존 네트워크를 Docker에 연결
docker network create \
  --driver bridge \
  --external \
  existing-network
```

## 12. 네트워크 디버깅

### 연결 상태 확인

```bash
# 네트워크 내 컨테이너 목록
docker network inspect my-network | grep Name

# 컨테이너 간 연결 테스트
docker exec container1 ping container2

# 포트 연결 테스트
docker exec container1 telnet container2 5432

# DNS 해석 확인
docker exec container1 nslookup container2
```

### 네트워크 트래픽 모니터링

```bash
# 특정 컨테이너의 네트워크 통계
docker stats container-name

# 네트워크 인터페이스 확인
docker exec container-name ip addr show

# 라우팅 테이블 확인
docker exec container-name ip route show
```

## 13. 실전 마이크로서비스 예제

```yaml
# docker-compose.yml
version: '3.8'

services:
  # 데이터베이스
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: microservice
      POSTGRES_PASSWORD: password
    networks:
      - backend-network
    volumes:
      - postgres_data:/var/lib/postgresql/data

  # 사용자 서비스
  user-service:
    build: ./user-service
    environment:
      DB_HOST: postgres
      DB_NAME: microservice
    networks:
      - backend-network
      - api-network
    depends_on:
      - postgres

  # 주문 서비스
  order-service:
    build: ./order-service
    environment:
      DB_HOST: postgres
      USER_SERVICE_URL: http://user-service:3000
    networks:
      - backend-network
      - api-network
    depends_on:
      - postgres
      - user-service

  # API 게이트웨이
  api-gateway:
    build: ./api-gateway
    environment:
      USER_SERVICE_URL: http://user-service:3000
      ORDER_SERVICE_URL: http://order-service:3000
    ports:
      - "8080:8080"
    networks:
      - api-network
    depends_on:
      - user-service
      - order-service

  # 프론트엔드
  frontend:
    build: ./frontend
    environment:
      API_URL: http://api-gateway:8080
    ports:
      - "3000:3000"
    networks:
      - frontend-network
    depends_on:
      - api-gateway

networks:
  frontend-network:    # 프론트엔드 전용
  api-network:        # API 서비스들
  backend-network:    # 백엔드 + DB

volumes:
  postgres_data:
```

**네트워크 구조:**
```
Frontend ←→ Frontend Network
              ↓
          API Gateway ←→ API Network ←→ Services
                                        ↓
                                   Backend Network ←→ Database
```

## 핵심 정리

**Docker 네트워크의 핵심:**
1. **격리**: 컨테이너들을 논리적으로 그룹화
2. **통신**: 같은 네트워크 내 컨테이너들은 이름으로 접근
3. **보안**: 네트워크별로 접근 제어 가능
4. **확장성**: 마이크로서비스 아키텍처 지원

**중요한 개념:**
- **Bridge**: 기본 네트워크, 같은 호스트 내 통신
- **Host**: 호스트 네트워크 직접 사용
- **None**: 네트워크 격리
- **Custom**: 사용자 정의 네트워크

**실무 팁:**
- 항상 커스텀 네트워크 사용 (기본 bridge 피하기)
- 서비스별로 네트워크 분리하여 보안 강화
- Docker Compose로 네트워크 자동 관리
- 컨테이너 이름을 호스트명으로 활용