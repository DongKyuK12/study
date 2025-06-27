# Docker 이미지 완전이해

## 이미지가 정확히 뭔가요?

**Docker 이미지 = 프로그램 설치 파일 + 실행 환경이 모두 들어있는 완전한 패키지**

```
Windows 프로그램         →    Docker 이미지
─────────────────              ─────────────────
setup.exe (설치파일)          node:18 (Node.js 환경)
+ Windows OS 필요             + Linux 환경 포함
+ .NET Framework 필요         + 모든 의존성 포함
+ 각종 라이브러리 설치         + 앱 코드 포함
```

## 1. 이미지의 정체

### 이미지는 파일들의 모음집이에요

```bash
# 이미지를 풀어서 보면 이런 파일들이 들어있어요
docker run -it ubuntu:20.04 /bin/bash
root@container:/# ls -la /
# drwxr-xr-x   2 root root  4096 /bin     ← 실행 파일들
# drwxr-xr-x   2 root root  4096 /etc     ← 설정 파일들  
# drwxr-xr-x   2 root root  4096 /lib     ← 라이브러리들
# drwxr-xr-x   2 root root  4096 /usr     ← 프로그램들
# drwxr-xr-x   2 root root  4096 /var     ← 데이터 폴더들
```

### 이미지 = 압축된 파일시스템

```bash
# 이미지 크기 확인
docker images
# ubuntu        20.04    72.8MB  ← 압축된 Ubuntu OS!
# node          18       900MB   ← Ubuntu + Node.js + npm
# nginx         latest   142MB   ← Ubuntu + Nginx 웹서버
```

## 2. 이미지 vs 컨테이너 차이점

### 이미지 = 읽기 전용 템플릿

```bash
# 이미지는 변경할 수 없어요
docker run ubuntu:20.04 touch /test.txt
# 컨테이너 안에서 파일 생성

docker run ubuntu:20.04 ls /
# 새 컨테이너에는 test.txt 없음!
# 이미지 자체는 변하지 않았음
```

### 컨테이너 = 이미지 + 실행 레이어

```
┌─────────────────────────────────┐
│       Container                 │
├─────────────────────────────────┤ ← 여기서 파일 변경 가능
│    Writable Layer              │   (컨테이너만의 공간)
├─────────────────────────────────┤
│         Image                   │ ← 읽기 전용 (변경 불가)
│  ┌─────────────────────────────┐ │
│  │  App Layer                  │ │
│  ├─────────────────────────────┤ │
│  │  Node.js Layer              │ │
│  ├─────────────────────────────┤ │
│  │  Ubuntu Layer               │ │
│  └─────────────────────────────┘ │
└─────────────────────────────────┘
```

## 3. 이미지가 만들어지는 과정

### Dockerfile → 이미지 변환 과정

```dockerfile
# Dockerfile
FROM ubuntu:20.04        # ← 기본 이미지 (Layer 1)
RUN apt update           # ← 명령어 실행 (Layer 2)  
RUN apt install nodejs   # ← 또 다른 명령어 (Layer 3)
COPY app.js /app/        # ← 파일 복사 (Layer 4)
CMD ["node", "/app/app.js"] # ← 실행 명령어 (Layer 5)
```

### 빌드 과정 단계별

```bash
# 빌드 시작
docker build -t my-app .

# Step 1/5 : FROM ubuntu:20.04
# ubuntu:20.04 이미지를 기반으로 시작

# Step 2/5 : RUN apt update  
# 임시 컨테이너 생성 → apt update 실행 → 새 레이어 생성

# Step 3/5 : RUN apt install nodejs
# 이전 레이어 + nodejs 설치 → 새 레이어 생성

# Step 4/5 : COPY app.js /app/
# 이전 레이어 + app.js 파일 → 새 레이어 생성

# Step 5/5 : CMD ["node", "/app/app.js"]
# 실행 명령어 설정 → 최종 이미지 완성
```

## 4. 레이어 시스템 이해하기

### 이미지 = 레이어들의 쌓기

```
┌─────────────────────────────────┐
│  Layer 4: COPY app.js /app/     │ ← 5MB
├─────────────────────────────────┤
│  Layer 3: RUN apt install nodejs│ ← 200MB  
├─────────────────────────────────┤
│  Layer 2: RUN apt update        │ ← 50MB
├─────────────────────────────────┤
│  Layer 1: FROM ubuntu:20.04     │ ← 72MB
└─────────────────────────────────┘
총 이미지 크기: 327MB
```

### 레이어 재사용의 마법

```bash
# 첫 번째 앱
FROM ubuntu:20.04     # Layer A (캐시됨)
RUN apt update        # Layer B (캐시됨)  
RUN apt install nodejs# Layer C (캐시됨)
COPY app1.js /app/    # Layer D1 (새로 생성)

# 두 번째 앱  
FROM ubuntu:20.04     # Layer A (재사용! 다운로드 안함)
RUN apt update        # Layer B (재사용! 실행 안함)
RUN apt install nodejs# Layer C (재사용! 실행 안함)  
COPY app2.js /app/    # Layer D2 (새로 생성)
```

**장점:** 빌드 시간 단축 + 저장공간 절약!

## 5. 이미지 종류들

### A) 베이스 이미지 (Operating System)

```bash
# 순수 OS 이미지들
docker pull ubuntu:20.04      # Ubuntu Linux
docker pull alpine:latest     # Alpine Linux (5MB! 초경량)
docker pull centos:7          # CentOS Linux
```

### B) 런타임 이미지 (Programming Language)

```bash
# 프로그래밍 언어 환경
docker pull node:18           # Node.js 18 + npm + Ubuntu
docker pull python:3.9       # Python 3.9 + pip + Ubuntu  
docker pull openjdk:11        # Java 11 + Ubuntu
docker pull golang:1.19      # Go 1.19 + Ubuntu
```

### C) 애플리케이션 이미지 (Ready-to-use Apps)

```bash
# 바로 사용 가능한 앱들
docker pull nginx:latest      # Nginx 웹서버
docker pull postgres:15       # PostgreSQL 데이터베이스
docker pull redis:latest      # Redis 캐시 서버
docker pull mysql:8.0         # MySQL 데이터베이스
```

### D) 내가 만든 이미지

```bash
# 내 앱을 이미지로 만들기
docker build -t my-company/my-app:v1.0 .
docker build -t my-express-api:latest .
```

## 6. 실제 이미지 내부 들여다보기

### 이미지 해부하기

```bash
# Node.js 이미지 받기
docker pull node:18-alpine

# 이미지 상세 정보 확인
docker inspect node:18-alpine
# {
#   "Architecture": "amd64",
#   "Os": "linux", 
#   "Size": 166395097,  ← 166MB
#   "Layers": [         ← 레이어 목록
#     "sha256:...",
#     "sha256:...",
#     "sha256:..."
#   ]
# }

# 이미지 레이어 히스토리  
docker history node:18-alpine
# IMAGE          CREATED         SIZE
# abc123...      2 weeks ago     0B      CMD ["node"]
# def456...      2 weeks ago     45MB    RUN apk add --no-cache nodejs npm
# ghi789...      3 weeks ago     5MB     ADD alpine-minirootfs.tar.xz
```

### 이미지 내부 파일시스템 탐색

```bash
# 이미지를 컨테이너로 실행해서 내부 확인
docker run -it node:18-alpine /bin/sh

# Node.js가 어디에 설치되어 있나?
which node
# /usr/local/bin/node

# npm은?
which npm  
# /usr/local/bin/npm

# 어떤 파일들이 들어있나?
ls -la /usr/local/bin/
# node, npm, npx 등등...

# OS 버전 확인
cat /etc/os-release
# Alpine Linux v3.17
```

## 7. 이미지 태그 시스템

### 태그란?

```bash
# 이미지 이름의 구조
docker pull node:18-alpine
#          ├─┴─ 태그 (버전)
#          └── 이미지 이름

# 다양한 태그들
docker pull node:18           # Node.js 18 (Ubuntu 기반)
docker pull node:18-alpine    # Node.js 18 (Alpine 기반, 더 작음)
docker pull node:18-slim      # Node.js 18 (Ubuntu 경량화)
docker pull node:latest       # 최신 버전
```

### 태그의 의미

```bash
# 버전별 태그
node:18.19.0     # 정확한 버전
node:18          # 18.x의 최신
node:latest      # 전체 최신

# OS별 태그  
node:18-alpine   # Alpine Linux 기반 (작음)
node:18-slim     # 슬림 버전 (중간)
node:18          # 일반 버전 (큼)

# 크기 비교
node:18-alpine   # ~170MB
node:18-slim     # ~240MB  
node:18          # ~900MB
```

## 8. 이미지 만들기 실습

### 간단한 웹서버 이미지 만들기

```javascript
// server.js
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/html' });
  res.end(`
    <h1>나만의 Docker 이미지!</h1>
    <p>현재 시간: ${new Date()}</p>
    <p>호스트명: ${require('os').hostname()}</p>
  `);
});

server.listen(3000, () => {
  console.log('서버가 3000번 포트에서 실행중입니다.');
});
```

```dockerfile
# Dockerfile
FROM node:18-alpine

# 작업 디렉토리 설정
WORKDIR /app

# 앱 코드 복사
COPY server.js .

# 포트 노출
EXPOSE 3000

# 앱 실행
CMD ["node", "server.js"]
```

```bash
# 이미지 빌드
docker build -t my-web-server:1.0 .

# 빌드 과정 확인
# Step 1/4 : FROM node:18-alpine
# Step 2/4 : WORKDIR /app  
# Step 3/4 : COPY server.js .
# Step 4/4 : CMD ["node", "server.js"]
# Successfully built abc123456789
# Successfully tagged my-web-server:1.0

# 이미지 확인
docker images | grep my-web-server
# my-web-server  1.0   abc123...  1 minute ago  174MB

# 컨테이너 실행
docker run -d -p 3000:3000 --name web my-web-server:1.0

# 테스트
curl http://localhost:3000
```

## 9. 이미지 최적화

### ❌ 비효율적인 Dockerfile

```dockerfile
FROM ubuntu:20.04

# 매번 apt update 실행 (캐시 효율 낮음)
RUN apt update
RUN apt install -y curl
RUN apt install -y nodejs  
RUN apt install -y npm

# 전체 프로젝트 복사 (코드 변경시 캐시 무효화)
COPY . /app
WORKDIR /app  
RUN npm install

CMD ["node", "app.js"]
```

### ✅ 효율적인 Dockerfile

```dockerfile
FROM node:18-alpine

WORKDIR /app

# package.json만 먼저 복사 (의존성 캐시 최적화)
COPY package*.json ./
RUN npm ci --only=production

# 앱 코드는 나중에 복사 (코드 변경시 npm install 재실행 안함)
COPY . .

# 보안을 위해 root가 아닌 사용자로 실행
USER node

EXPOSE 3000
CMD ["node", "app.js"]
```

## 10. 이미지 공유하기

### Docker Hub에 올리기

```bash
# Docker Hub 로그인
docker login

# 이미지에 태그 추가 (username/imagename 형식)
docker tag my-web-server:1.0 myusername/my-web-server:1.0

# Docker Hub에 업로드
docker push myusername/my-web-server:1.0

# 다른 사람이 사용
docker pull myusername/my-web-server:1.0
docker run -p 3000:3000 myusername/my-web-server:1.0
```

### GitHub Container Registry (GHCR)

```bash
# GitHub에 로그인 (Personal Access Token 필요)
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# 이미지 태그
docker tag my-web-server:1.0 ghcr.io/username/my-web-server:1.0

# GHCR에 업로드
docker push ghcr.io/username/my-web-server:1.0
```

## 11. server.js에서 이미지 사용 사례

server.js에서 이런 이미지들을 사용해요:

```javascript
// PostgreSQL 이미지 사용 (176줄)
const imageName = 'postgres:15-alpine';
await execAsync(`docker run -d --name ${containerName} ${imageName}`);

// 백엔드 이미지 사용 (230줄)  
const imageName = `ghcr.io/hebububu/management:${tag}`;
await execAsync(`docker pull ${imageName}`);

// 프론트엔드 이미지 사용 (392줄)
const imageName = `ghcr.io/hebububu/management_web:${tag}`;
await execAsync(`docker pull ${imageName}`);
```

## 12. 멀티스테이지 빌드 심화

### 문제: 큰 이미지 크기

```dockerfile
# 문제가 있는 Dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install              # 개발 의존성도 포함 (큰 크기)
COPY . .
RUN npm run build           # 빌드 도구들도 포함
RUN npm test                # 테스트 도구들도 포함
CMD ["npm", "start"]
# 결과: 1.2GB 크기!
```

### 해결: 멀티스테이지 빌드

```dockerfile
# 1단계: 빌드 환경 (큰 이미지)
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install              # 모든 의존성 설치
COPY . .
RUN npm run build           # 빌드 실행
RUN npm test                # 테스트 실행

# 2단계: 실행 환경 (작은 이미지)
FROM node:18-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production  # 실행 의존성만 설치
COPY --from=builder /app/dist ./dist  # 빌드 결과만 복사
CMD ["npm", "start"]
# 결과: 180MB 크기!
```

## 13. 이미지 보안

### 보안 취약점 스캔

```bash
# Docker 내장 스캔
docker scan my-image:latest

# 외부 도구 사용
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image my-image:latest
```

### 안전한 이미지 만들기

```dockerfile
FROM node:18-alpine

# 보안 업데이트
RUN apk update && apk upgrade

# 보안을 위해 non-root 사용자 생성
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001

WORKDIR /app

# 소유자 변경
COPY --chown=nodejs:nodejs package*.json ./
RUN npm ci --only=production && npm cache clean --force

COPY --chown=nodejs:nodejs . .

# non-root 사용자로 전환
USER nodejs

EXPOSE 3000

CMD ["node", "server.js"]
```

## 14. 이미지 관리 베스트 프랙티스

### 태그 관리 전략

```bash
# 좋은 태그 전략
my-app:1.0.0        # 정확한 버전
my-app:1.0          # 마이너 버전
my-app:1            # 메이저 버전
my-app:latest       # 최신 (주의해서 사용)

# 환경별 태그
my-app:1.0.0-dev    # 개발용
my-app:1.0.0-staging # 스테이징용
my-app:1.0.0-prod   # 프로덕션용
```

### .dockerignore 사용

```
# .dockerignore
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
coverage/
.nyc_output
```

### 이미지 크기 최적화 체크리스트

1. **Alpine 베이스 이미지 사용**: `node:18-alpine`
2. **멀티스테이지 빌드**: 빌드와 실행 환경 분리
3. **의존성 최적화**: `npm ci --only=production`
4. **불필요한 파일 제외**: `.dockerignore` 활용
5. **레이어 최적화**: 명령어 그룹화
6. **캐시 최적화**: 자주 변경되는 파일은 나중에 복사

## 핵심 정리

**이미지란:**
- 앱을 실행하는데 필요한 모든 것이 들어있는 완전한 패키지
- 운영체제 + 런타임 + 라이브러리 + 앱 코드 + 설정
- 읽기 전용 템플릿
- 레이어로 구성되어 효율적
- 어디서든 동일하게 실행 가능

**중요한 특징:**
1. **불변성**: 한 번 만들면 변경되지 않음
2. **레이어 시스템**: 효율적인 저장과 전송
3. **포터빌리티**: 어디서든 동일하게 실행
4. **버전 관리**: 태그로 버전 관리
5. **재사용성**: 베이스 이미지로 활용