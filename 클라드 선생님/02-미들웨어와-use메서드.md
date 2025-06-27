# 미들웨어와 use 메서드

## 미들웨어가 뭔가요?

미들웨어는 **"요청(Request)과 응답(Response) 사이에서 실행되는 함수들"**이에요.

**비유:**
- 요청 = 손님이 식당에 들어옴
- 미들웨어 = 웨이터, 요리사, 계산원 등
- 응답 = 음식이 손님에게 나감

```
요청 → 미들웨어1 → 미들웨어2 → 미들웨어3 → 응답
```

## 1. 미들웨어의 기본 구조

### 미들웨어 함수의 형태

```javascript
function myMiddleware(req, res, next) {
  // 요청 처리 로직
  console.log('미들웨어 실행됨');
  
  // 다음 미들웨어로 넘어가기
  next();
}
```

**3개의 매개변수:**
- `req`: 요청 객체 (Request)
- `res`: 응답 객체 (Response) 
- `next`: 다음 미들웨어로 넘어가는 함수

### 기본 미들웨어 예시

```javascript
const express = require('express');
const app = express();

// 미들웨어 1: 로그 남기기
app.use((req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.url}`);
  next(); // 중요! 이게 없으면 다음으로 안 넘어감
});

// 미들웨어 2: 사용자 정보 추가
app.use((req, res, next) => {
  req.user = { id: 1, name: '홍길동' };
  next();
});

// 라우트 핸들러
app.get('/', (req, res) => {
  res.json({
    message: '홈페이지',
    user: req.user // 미들웨어에서 추가한 정보 사용
  });
});

app.listen(3000);
```

## 2. app.use() 메서드 완전 분석

### app.use()의 다양한 사용법

```javascript
const express = require('express');
const app = express();

// 1. 모든 요청에 적용
app.use((req, res, next) => {
  console.log('모든 요청에서 실행');
  next();
});

// 2. 특정 경로에만 적용
app.use('/api', (req, res, next) => {
  console.log('API 경로에서만 실행');
  next();
});

// 3. 여러 경로에 적용
app.use(['/admin', '/dashboard'], (req, res, next) => {
  console.log('관리자 페이지에서 실행');
  next();
});

// 4. 내장 미들웨어 사용
app.use(express.json()); // JSON 파싱
app.use(express.static('public')); // 정적 파일 서빙

// 5. 외부 미들웨어 사용
const cors = require('cors');
app.use(cors()); // CORS 설정
```

## 3. 내장 미들웨어들

### A) express.json()

**역할:** JSON 형태의 요청 본문을 파싱

```javascript
const express = require('express');
const app = express();

// JSON 미들웨어 없이
app.post('/without-json', (req, res) => {
  console.log(req.body); // undefined
  res.send('JSON 파싱 안됨');
});

// JSON 미들웨어 사용
app.use(express.json());

app.post('/with-json', (req, res) => {
  console.log(req.body); // { name: '홍길동', age: 30 }
  res.json({
    message: '데이터 받음',
    received: req.body
  });
});

app.listen(3000);
```

**테스트:**
```bash
# JSON 미들웨어 없이
curl -X POST http://localhost:3000/without-json \
  -H "Content-Type: application/json" \
  -d '{"name": "홍길동", "age": 30}'

# JSON 미들웨어 사용
curl -X POST http://localhost:3000/with-json \
  -H "Content-Type: application/json" \
  -d '{"name": "홍길동", "age": 30}'
```

### B) express.urlencoded()

**역할:** HTML 폼 데이터 파싱

```javascript
// 폼 데이터 파싱 미들웨어
app.use(express.urlencoded({ extended: true }));

app.post('/form', (req, res) => {
  console.log(req.body); // { name: '홍길동', email: 'hong@example.com' }
  res.send('폼 데이터 받음');
});
```

### C) express.static()

**역할:** 정적 파일 서빙

```javascript
// public 폴더의 파일들을 웹에서 접근 가능하게
app.use(express.static('public'));

// 특정 경로에 정적 파일 서빙
app.use('/assets', express.static('public'));
```

## 4. 미들웨어 실행 순서

```javascript
const express = require('express');
const app = express();

// 1번째 실행
app.use((req, res, next) => {
  console.log('1번 미들웨어');
  next();
});

// 2번째 실행
app.use((req, res, next) => {
  console.log('2번 미들웨어');
  next();
});

// 3번째 실행 (특정 경로)
app.use('/api', (req, res, next) => {
  console.log('3번 미들웨어 (API 경로만)');
  next();
});

// 4번째 실행 (라우트 핸들러)
app.get('/api/users', (req, res) => {
  console.log('4번 라우트 핸들러');
  res.send('사용자 목록');
});

app.listen(3000);
```

**요청 결과:**
```
GET /api/users 요청시:
1번 미들웨어
2번 미들웨어
3번 미들웨어 (API 경로만)
4번 라우트 핸들러
```

## 5. next() 함수의 역할

### next() 없으면 멈춤

```javascript
// ❌ 잘못된 예시
app.use((req, res, next) => {
  console.log('미들웨어 실행');
  // next()를 호출하지 않음 → 여기서 멈춤!
});

app.get('/', (req, res) => {
  res.send('이 코드는 실행되지 않음');
});
```

### next() 사용법들

```javascript
// 1. 정상적으로 다음으로 넘어가기
app.use((req, res, next) => {
  console.log('로그');
  next(); // 다음 미들웨어로
});

// 2. 에러와 함께 넘어가기
app.use((req, res, next) => {
  if (!req.headers.authorization) {
    return next(new Error('인증 토큰이 없습니다'));
  }
  next();
});

// 3. 특정 라우트로 건너뛰기
app.use((req, res, next) => {
  if (req.path === '/skip') {
    return next('route'); // 다음 라우트로 건너뛰기
  }
  next();
});
```

## 6. 실전 미들웨어 예시들

### A) 인증 미들웨어

```javascript
const authenticate = (req, res, next) => {
  const token = req.headers.authorization;
  
  if (!token) {
    return res.status(401).json({
      error: '토큰이 필요합니다'
    });
  }
  
  if (token !== 'Bearer valid-token') {
    return res.status(401).json({
      error: '유효하지 않은 토큰입니다'
    });
  }
  
  // 토큰이 유효하면 사용자 정보 추가
  req.user = { id: 1, name: '홍길동' };
  next();
};

// 인증이 필요한 라우트에만 적용
app.get('/profile', authenticate, (req, res) => {
  res.json({
    message: '프로필 정보',
    user: req.user
  });
});
```

### B) 로깅 미들웨어

```javascript
const logger = (req, res, next) => {
  const start = Date.now();
  
  // 응답이 끝났을 때 로그 남기기
  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(`${req.method} ${req.url} - ${res.statusCode} - ${duration}ms`);
  });
  
  next();
};

app.use(logger);
```

### C) CORS 미들웨어

```javascript
const cors = (req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*');
  res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  
  if (req.method === 'OPTIONS') {
    return res.sendStatus(200);
  }
  
  next();
};

app.use(cors);
```

## 7. server.js에서 미들웨어 사용 예시

server.js 6줄에서 사용한 미들웨어:

```javascript
// JSON 파싱 미들웨어
app.use(express.json());
```

server.js 90-98줄의 인증 미들웨어:

```javascript
const authenticate = (req, res, next) => {
  const auth = req.headers.authorization;
  if (auth !== `Bearer ${DEPLOY_SECRET}`) {
    log(`❌ Unauthorized access attempt from ${req.ip}`);
    return res.status(401).json({ error: 'Unauthorized' });
  }
  next();
};
```

## 8. 에러 처리 미들웨어

에러 처리 미들웨어는 **4개의 매개변수**를 가져요:

```javascript
const errorHandler = (err, req, res, next) => {
  console.error('에러 발생:', err.message);
  
  // 에러 타입에 따른 다른 처리
  if (err.name === 'ValidationError') {
    return res.status(400).json({
      error: '입력 데이터가 올바르지 않습니다',
      details: err.message
    });
  }
  
  if (err.name === 'UnauthorizedError') {
    return res.status(401).json({
      error: '인증이 필요합니다'
    });
  }
  
  // 기본 에러 응답
  res.status(500).json({
    error: '서버 내부 오류가 발생했습니다'
  });
};

// 에러 처리 미들웨어는 맨 마지막에 등록
app.use(errorHandler);
```

## 9. 조건부 미들웨어

```javascript
const conditionalMiddleware = (condition) => {
  return (req, res, next) => {
    if (condition(req)) {
      console.log('조건 만족, 미들웨어 실행');
      // 실제 처리 로직
      req.customData = 'added by middleware';
    }
    next();
  };
};

// 개발 환경에서만 실행되는 미들웨어
app.use(conditionalMiddleware(req => process.env.NODE_ENV === 'development'));

// 특정 사용자만
app.use(conditionalMiddleware(req => req.headers['x-admin'] === 'true'));
```

## 10. 미들웨어 체이닝

```javascript
const middleware1 = (req, res, next) => {
  console.log('미들웨어 1');
  req.step = 1;
  next();
};

const middleware2 = (req, res, next) => {
  console.log('미들웨어 2');
  req.step = 2;
  next();
};

const middleware3 = (req, res, next) => {
  console.log('미들웨어 3');
  req.step = 3;
  next();
};

// 여러 미들웨어를 체이닝
app.get('/chain', middleware1, middleware2, middleware3, (req, res) => {
  res.json({
    message: '모든 미들웨어 통과',
    step: req.step
  });
});

// 또는 배열로 전달
app.get('/chain2', [middleware1, middleware2, middleware3], (req, res) => {
  res.json({ message: '배열로 미들웨어 전달' });
});
```

## 11. 실습 과제

다음 미들웨어들을 만들어보세요:

```javascript
// 1. 요청 시간 측정 미들웨어
const timeMiddleware = (req, res, next) => {
  // TODO: 시작 시간 기록
  // TODO: 응답 완료시 걸린 시간 계산
  next();
};

// 2. IP 차단 미들웨어
const blockIP = (blockedIPs) => {
  return (req, res, next) => {
    // TODO: req.ip가 차단 목록에 있는지 확인
    // TODO: 차단된 IP면 403 에러 반환
    next();
  };
};

// 3. 요청 횟수 제한 미들웨어 (간단한 rate limiting)
const rateLimiter = (maxRequests, windowMs) => {
  const requests = new Map();
  
  return (req, res, next) => {
    // TODO: IP별 요청 횟수 추적
    // TODO: 제한 초과시 429 에러 반환
    next();
  };
};
```

## 핵심 정리

**미들웨어의 핵심:**
1. **순서가 중요**: 등록한 순서대로 실행됨
2. **next() 필수**: 없으면 다음으로 넘어가지 않음
3. **req/res 수정 가능**: 다음 미들웨어나 라우터에서 사용
4. **에러 처리**: next(error)로 에러 전달 가능

**app.use()의 특징:**
- 모든 HTTP 메서드에 적용됨
- 경로 지정 가능
- 여러 미들웨어 동시 등록 가능
- 내장/외부 미들웨어 모두 사용 가능

**실무 팁:**
- 로깅은 가장 먼저, 에러 처리는 가장 마지막
- 인증 미들웨어는 필요한 라우트에만 적용
- 조건부 미들웨어로 성능 최적화
- 미들웨어 체이닝으로 코드 재사용성 향상