# Express 기초

## Express가 뭔가요?

Express는 **"Node.js에서 웹서버를 쉽게 만들 수 있게 해주는 프레임워크"**예요.

**비유:**
- Node.js = 재료들 (고기, 야채, 양념)
- Express = 요리 도구 (팬, 칼, 도마)
- 결과 = 맛있는 요리 (웹서버)

## 1. 기본 설치와 설정

### 프로젝트 시작

```bash
# 새 프로젝트 폴더
mkdir my-express-app
cd my-express-app

# package.json 생성
npm init -y

# Express 설치
npm install express
```

### 첫 번째 서버 만들기

```javascript
// server.js
const express = require('express');
const app = express();

// 기본 라우트
app.get('/', (req, res) => {
  res.send('안녕하세요! Express 서버입니다.');
});

// 서버 시작
const PORT = 3000;
app.listen(PORT, () => {
  console.log(`서버가 ${PORT}번 포트에서 실행중입니다.`);
});
```

```bash
# 실행
node server.js

# 브라우저에서 http://localhost:3000 접속
```

## 2. Express의 핵심 개념

### A) 라우트 (Route)

**라우트 = URL 경로에 따라 다른 응답을 보내는 것**

```javascript
const express = require('express');
const app = express();

// GET 요청 처리
app.get('/', (req, res) => {
  res.send('홈페이지입니다');
});

app.get('/about', (req, res) => {
  res.send('소개 페이지입니다');
});

app.get('/contact', (req, res) => {
  res.send('연락처 페이지입니다');
});

app.listen(3000);
```

### B) 미들웨어 (Middleware)

**미들웨어 = 요청과 응답 사이에서 실행되는 함수들**

```javascript
const express = require('express');
const app = express();

// 미들웨어 1: 로그 남기기
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url} - ${new Date()}`);
  next(); // 다음 미들웨어로 넘어가기
});

// 미들웨어 2: JSON 파싱
app.use(express.json());

// 라우트
app.get('/', (req, res) => {
  res.send('Hello World');
});

app.listen(3000);
```

## 3. 실제 API 만들어보기

### 사용자 관리 API

```javascript
const express = require('express');
const app = express();

// JSON 파싱 미들웨어
app.use(express.json());

// 가짜 사용자 데이터
let users = [
  { id: 1, name: '홍길동', email: 'hong@example.com' },
  { id: 2, name: '김철수', email: 'kim@example.com' }
];

// 모든 사용자 조회
app.get('/users', (req, res) => {
  res.json({
    success: true,
    data: users
  });
});

// 특정 사용자 조회
app.get('/users/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const user = users.find(u => u.id === id);
  
  if (!user) {
    return res.status(404).json({
      success: false,
      message: '사용자를 찾을 수 없습니다'
    });
  }
  
  res.json({
    success: true,
    data: user
  });
});

// 새 사용자 생성
app.post('/users', (req, res) => {
  const { name, email } = req.body;
  
  // 입력 검증
  if (!name || !email) {
    return res.status(400).json({
      success: false,
      message: '이름과 이메일은 필수입니다'
    });
  }
  
  const newUser = {
    id: users.length + 1,
    name,
    email
  };
  
  users.push(newUser);
  
  res.status(201).json({
    success: true,
    data: newUser
  });
});

// 사용자 수정
app.put('/users/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const userIndex = users.findIndex(u => u.id === id);
  
  if (userIndex === -1) {
    return res.status(404).json({
      success: false,
      message: '사용자를 찾을 수 없습니다'
    });
  }
  
  const { name, email } = req.body;
  users[userIndex] = { id, name, email };
  
  res.json({
    success: true,
    data: users[userIndex]
  });
});

// 사용자 삭제
app.delete('/users/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const userIndex = users.findIndex(u => u.id === id);
  
  if (userIndex === -1) {
    return res.status(404).json({
      success: false,
      message: '사용자를 찾을 수 없습니다'
    });
  }
  
  users.splice(userIndex, 1);
  
  res.json({
    success: true,
    message: '사용자가 삭제되었습니다'
  });
});

app.listen(3000, () => {
  console.log('사용자 API 서버가 3000번 포트에서 실행중입니다.');
});
```

### API 테스트 해보기

```bash
# 모든 사용자 조회
curl http://localhost:3000/users

# 특정 사용자 조회
curl http://localhost:3000/users/1

# 새 사용자 생성
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{"name": "이영희", "email": "lee@example.com"}'

# 사용자 수정
curl -X PUT http://localhost:3000/users/1 \
  -H "Content-Type: application/json" \
  -d '{"name": "홍길동 수정", "email": "newhong@example.com"}'

# 사용자 삭제
curl -X DELETE http://localhost:3000/users/1
```

## 4. 정적 파일 서빙

### HTML, CSS, JS 파일 제공

```javascript
const express = require('express');
const path = require('path');
const app = express();

// 정적 파일 서빙 (public 폴더)
app.use(express.static('public'));

// 또는 특정 경로에서 서빙
app.use('/assets', express.static('public'));

app.get('/', (req, res) => {
  res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

app.listen(3000);
```

### 폴더 구조

```
my-project/
├── server.js
├── public/
│   ├── index.html
│   ├── style.css
│   ├── script.js
│   └── images/
│       └── logo.png
└── package.json
```

## 5. server.js에서 Express 사용 예시

실제 server.js에서 Express가 어떻게 사용되는지 보세요:

```javascript
// server.js 1-4줄
const express = require('express');
const { exec, spawn } = require('child_process');
const app = express();

// server.js 6줄
app.use(express.json());

// server.js 90-98줄 - 인증 미들웨어
const authenticate = (req, res, next) => {
  const auth = req.headers.authorization;
  if (auth !== `Bearer ${DEPLOY_SECRET}`) {
    log(`❌ Unauthorized access attempt from ${req.ip}`);
    return res.status(401).json({ error: 'Unauthorized' });
  }
  next();
};
```

## 6. 에러 처리

### 기본 에러 처리

```javascript
const express = require('express');
const app = express();

app.use(express.json());

app.get('/error-test', (req, res) => {
  // 의도적 에러 발생
  throw new Error('테스트 에러입니다!');
});

// 에러 처리 미들웨어 (맨 마지막에 위치)
app.use((err, req, res, next) => {
  console.error('에러 발생:', err.message);
  
  res.status(500).json({
    success: false,
    message: '서버 내부 오류가 발생했습니다',
    error: process.env.NODE_ENV === 'development' ? err.message : undefined
  });
});

app.listen(3000);
```

## 7. 환경별 설정

### 개발/프로덕션 환경 분리

```javascript
const express = require('express');
const app = express();

// 환경별 설정
const isDevelopment = process.env.NODE_ENV === 'development';
const PORT = process.env.PORT || 3000;

// 개발 환경에서만 상세 로그
if (isDevelopment) {
  app.use((req, res, next) => {
    console.log(`${req.method} ${req.url}`);
    next();
  });
}

app.use(express.json());

app.get('/', (req, res) => {
  res.json({
    message: 'Express 서버입니다',
    environment: process.env.NODE_ENV || 'development',
    port: PORT
  });
});

// 프로덕션 환경에서는 에러 상세 정보 숨김
app.use((err, req, res, next) => {
  console.error(err.stack);
  
  res.status(500).json({
    success: false,
    message: '서버 오류',
    error: isDevelopment ? err.message : '내부 서버 오류'
  });
});

app.listen(PORT, () => {
  console.log(`서버가 ${PORT}번 포트에서 실행중입니다. (${process.env.NODE_ENV || 'development'} 모드)`);
});
```

## 8. CORS 설정

### 프론트엔드와 연결하기

```javascript
const express = require('express');
const app = express();

// CORS 미들웨어 (직접 구현)
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*');
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  
  if (req.method === 'OPTIONS') {
    res.sendStatus(200);
  } else {
    next();
  }
});

// 또는 cors 패키지 사용
// npm install cors
const cors = require('cors');
app.use(cors({
  origin: ['http://localhost:3000', 'https://mysite.com'],
  credentials: true
}));
```

## 9. 실습 과제

간단한 도서 관리 API를 만들어보세요:

```javascript
// books-api.js
const express = require('express');
const app = express();

app.use(express.json());

let books = [
  { id: 1, title: 'Node.js 완벽 가이드', author: '김개발', year: 2023 },
  { id: 2, title: 'Express 마스터하기', author: '이백엔드', year: 2024 }
];

// TODO: 다음 API들을 구현해보세요
// GET /books - 모든 책 조회
// GET /books/:id - 특정 책 조회
// POST /books - 새 책 추가
// PUT /books/:id - 책 정보 수정
// DELETE /books/:id - 책 삭제

app.listen(4000, () => {
  console.log('도서 API가 4000번 포트에서 실행중입니다.');
});
```

## 핵심 정리

**Express의 핵심 개념:**
1. **라우팅**: URL 경로별로 다른 처리
2. **미들웨어**: 요청-응답 사이의 처리 함수들
3. **요청/응답**: req/res 객체로 HTTP 처리
4. **에러 처리**: try-catch와 에러 미들웨어
5. **정적 파일**: HTML, CSS, JS 등 파일 서빙

**Express의 장점:**
- 간단하고 직관적인 API
- 풍부한 미들웨어 생태계
- 유연한 라우팅 시스템
- RESTful API 구축에 최적화