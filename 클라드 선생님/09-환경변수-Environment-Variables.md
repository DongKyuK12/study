# 환경변수 (Environment Variables)

## 환경변수가 뭔가요?

환경변수는 **"프로그램 실행 환경에 따라 다르게 설정할 수 있는 값들"**이에요.

**왜 필요한가요?**
- 개발환경 vs 실제환경에서 다른 설정 사용
- 비밀번호, API 키 등을 코드에 직접 쓰지 않기 위해
- 서버별로 다른 설정 적용

## 1. Next.js에서 써본 경험과 비교

Next.js에서 이런 거 써보셨죠?

```javascript
// Next.js에서
const apiUrl = process.env.NEXT_PUBLIC_API_URL;
const secretKey = process.env.SECRET_KEY;
```

Node.js 서버에서도 똑같이 사용해요:

```javascript
// Express 서버에서
const PORT = process.env.PORT || 3000;
const DB_PASSWORD = process.env.DB_PASSWORD;
const JWT_SECRET = process.env.JWT_SECRET;
```

## 2. 기본 사용법

### .env 파일 만들기

프로젝트 루트에 `.env` 파일 생성:

```bash
# .env 파일
PORT=9000
DB_HOST=localhost
DB_PASSWORD=mypassword123
JWT_SECRET=supersecretkey
API_URL=https://api.example.com
DEBUG=true
```

### 서버에서 사용하기

```javascript
// server.js
const express = require('express');
const app = express();

// 환경변수 읽기
const PORT = process.env.PORT || 3000;        // 기본값: 3000
const DB_HOST = process.env.DB_HOST;
const DB_PASSWORD = process.env.DB_PASSWORD;
const JWT_SECRET = process.env.JWT_SECRET;
const DEBUG = process.env.DEBUG === 'true';   // 문자열 → 불린 변환

console.log('설정 정보:');
console.log('포트:', PORT);
console.log('DB 호스트:', DB_HOST);
console.log('JWT 시크릿:', JWT_SECRET ? 'SET' : 'NOT SET');
console.log('디버그 모드:', DEBUG);

app.listen(PORT, () => {
  console.log(`서버가 ${PORT}번 포트에서 실행중입니다.`);
});
```

## 3. dotenv 패키지 사용하기

Node.js는 기본적으로 `.env` 파일을 자동으로 읽지 않아요. `dotenv` 패키지가 필요해요!

### 설치

```bash
npm install dotenv
```

### 사용법

```javascript
// server.js 맨 위에
require('dotenv').config();  // .env 파일 로드

const express = require('express');
const app = express();

// 이제 process.env로 접근 가능
const PORT = process.env.PORT || 3000;
const SECRET = process.env.SECRET;

console.log('환경변수 로드됨:', {
  PORT,
  SECRET: SECRET ? '***' : 'NOT SET'
});
```

## 4. server.js에서 실제 사용 예시

server.js 8-12줄을 보면:

```javascript
// 환경변수 설정
const DEPLOY_SECRET = process.env.DEPLOY_SECRET;
const PORT = process.env.PORT || 9000;
const GITHUB_USERNAME = process.env.GITHUBUSERNAME;
const GITHUB_TOKEN = process.env.GITHUBTOKEN;
```

server.js 768-780줄에서 확인:

```javascript
log(`🔧 Environment check:`);
log(`   GITHUB_USERNAME: ${GITHUB_USERNAME}`);
log(`   GITHUB_TOKEN: ${GITHUB_TOKEN ? 'SET' : 'NOT SET'}`);
log(`   JWT_SECRET_KEY: ${process.env.JWT_SECRET_KEY ? 'SET' : 'NOT SET'}`);
log(`   PG_PASSWORD: ${process.env.PG_PASSWORD ? 'SET' : 'NOT SET'}`);
```

## 5. 환경별 설정 관리

### .env 파일들

```bash
# 프로젝트 구조
my-project/
├── .env                 # 공통 설정
├── .env.development     # 개발환경
├── .env.production      # 실제환경
├── .env.test           # 테스트환경
└── server.js
```

### .env.development

```bash
# 개발환경 설정
NODE_ENV=development
PORT=3000
DB_HOST=localhost
DB_PASSWORD=dev123
API_URL=http://localhost:8000
DEBUG=true
LOG_LEVEL=debug
```

### .env.production

```bash
# 실제환경 설정
NODE_ENV=production
PORT=80
DB_HOST=prod-db.example.com
DB_PASSWORD=super_secure_password_here
API_URL=https://api.production.com
DEBUG=false
LOG_LEVEL=error
```

### 환경에 따라 다른 파일 로드

```javascript
const path = require('path');

// 환경에 따라 다른 .env 파일 로드
const envFile = process.env.NODE_ENV === 'production' 
  ? '.env.production' 
  : '.env.development';

require('dotenv').config({ path: path.resolve(envFile) });

const config = {
  port: process.env.PORT,
  dbHost: process.env.DB_HOST,
  dbPassword: process.env.DB_PASSWORD,
  apiUrl: process.env.API_URL,
  debug: process.env.DEBUG === 'true'
};

console.log('현재 환경:', process.env.NODE_ENV);
console.log('설정:', config);
```

## 6. 실전 예시 - 데이터베이스 연결

```javascript
require('dotenv').config();

const express = require('express');
const app = express();

// 환경변수로 DB 설정
const dbConfig = {
  host: process.env.DB_HOST || 'localhost',
  port: parseInt(process.env.DB_PORT) || 5432,
  database: process.env.DB_NAME || 'myapp',
  username: process.env.DB_USERNAME || 'postgres',
  password: process.env.DB_PASSWORD,  // 필수값
  ssl: process.env.DB_SSL === 'true'
};

// 환경변수 검증
const requiredEnvVars = ['DB_PASSWORD', 'JWT_SECRET'];
const missingVars = requiredEnvVars.filter(varName => !process.env[varName]);

if (missingVars.length > 0) {
  console.error('❌ 필수 환경변수가 설정되지 않았습니다:');
  missingVars.forEach(varName => {
    console.error(`   - ${varName}`);
  });
  process.exit(1);  // 서버 종료
}

// 가짜 DB 연결 함수
async function connectToDatabase() {
  try {
    console.log('🔌 데이터베이스에 연결 중...');
    console.log(`   호스트: ${dbConfig.host}:${dbConfig.port}`);
    console.log(`   데이터베이스: ${dbConfig.database}`);
    console.log(`   사용자: ${dbConfig.username}`);
    console.log(`   SSL: ${dbConfig.ssl ? 'Enabled' : 'Disabled'}`);
    
    // 실제로는 여기서 DB 연결
    // await pg.connect(dbConfig);
    
    console.log('✅ 데이터베이스 연결 성공');
  } catch (error) {
    console.error('❌ 데이터베이스 연결 실패:', error.message);
    process.exit(1);
  }
}

app.get('/config', (req, res) => {
  // 민감한 정보는 숨김
  const safeConfig = {
    environment: process.env.NODE_ENV,
    port: process.env.PORT,
    dbHost: dbConfig.host,
    dbPort: dbConfig.port,
    dbName: dbConfig.database,
    dbPassword: dbConfig.password ? '***' : 'NOT SET',
    debug: process.env.DEBUG === 'true'
  };
  
  res.json(safeConfig);
});

// 서버 시작
const PORT = process.env.PORT || 3000;
app.listen(PORT, async () => {
  await connectToDatabase();
  console.log(`🚀 서버가 ${PORT}번 포트에서 실행중입니다.`);
});
```

## 7. 환경변수 타입 변환

환경변수는 항상 **문자열**이에요! 다른 타입으로 변환해야 해요.

```javascript
// .env 파일
// PORT=3000
// DEBUG=true
// MAX_CONNECTIONS=100
// RATE_LIMIT=1.5

// 문자열 → 숫자
const PORT = parseInt(process.env.PORT) || 3000;
const MAX_CONNECTIONS = parseInt(process.env.MAX_CONNECTIONS) || 50;
const RATE_LIMIT = parseFloat(process.env.RATE_LIMIT) || 1.0;

// 문자열 → 불린
const DEBUG = process.env.DEBUG === 'true';
const ENABLE_LOGGING = process.env.ENABLE_LOGGING !== 'false'; // 기본 true

// 문자열 → 배열
const ALLOWED_ORIGINS = process.env.ALLOWED_ORIGINS?.split(',') || [];
// ALLOWED_ORIGINS=http://localhost:3000,https://mysite.com
// → ['http://localhost:3000', 'https://mysite.com']

// 문자열 → 객체
const CORS_CONFIG = process.env.CORS_CONFIG ? JSON.parse(process.env.CORS_CONFIG) : {};
// CORS_CONFIG={"origin": true, "credentials": true}

console.log('변환된 설정:');
console.log('PORT (number):', PORT, typeof PORT);
console.log('DEBUG (boolean):', DEBUG, typeof DEBUG);
console.log('ALLOWED_ORIGINS (array):', ALLOWED_ORIGINS);
```

## 8. 보안 주의사항

### ❌ 하지 말아야 할 것들

```javascript
// ❌ 코드에 직접 비밀번호 쓰기
const password = 'mypassword123';  // 위험!

// ❌ .env 파일을 git에 커밋
// .env 파일은 반드시 .gitignore에 추가!
```

### ✅ 올바른 방법

```bash
# .gitignore 파일에 추가
.env
.env.local
.env.*.local
```

```javascript
// ✅ 환경변수 사용
const password = process.env.DB_PASSWORD;

// ✅ 민감한 정보 로깅시 숨김
console.log('DB Password:', password ? '***' : 'NOT SET');
```

## 9. 환경변수 헬퍼 함수

```javascript
// config.js
class Config {
  static getString(key, defaultValue = '') {
    return process.env[key] || defaultValue;
  }
  
  static getNumber(key, defaultValue = 0) {
    const value = process.env[key];
    return value ? parseInt(value) : defaultValue;
  }
  
  static getBoolean(key, defaultValue = false) {
    const value = process.env[key];
    if (value === undefined) return defaultValue;
    return value.toLowerCase() === 'true';
  }
  
  static getArray(key, separator = ',', defaultValue = []) {
    const value = process.env[key];
    return value ? value.split(separator).map(s => s.trim()) : defaultValue;
  }
  
  static required(key) {
    const value = process.env[key];
    if (!value) {
      throw new Error(`환경변수 ${key}는 필수입니다!`);
    }
    return value;
  }
}

// 사용법
const config = {
  port: Config.getNumber('PORT', 3000),
  debug: Config.getBoolean('DEBUG', false),
  dbPassword: Config.required('DB_PASSWORD'),
  allowedOrigins: Config.getArray('ALLOWED_ORIGINS'),
  jwtSecret: Config.required('JWT_SECRET')
};
```

## 10. 실습해보기

`.env` 파일 만들고 테스트:

```bash
# .env
APP_NAME=My Express App
PORT=9000
DEBUG=true
MAX_USERS=100
ALLOWED_IPS=127.0.0.1,192.168.1.1,10.0.0.1
DATABASE_URL=postgres://user:pass@localhost:5432/mydb
SECRET_KEY=my-super-secret-key-here
```

```javascript
require('dotenv').config();

const express = require('express');
const app = express();

// 환경변수 읽기 및 변환
const config = {
  appName: process.env.APP_NAME || 'Express App',
  port: parseInt(process.env.PORT) || 3000,
  debug: process.env.DEBUG === 'true',
  maxUsers: parseInt(process.env.MAX_USERS) || 50,
  allowedIPs: process.env.ALLOWED_IPS?.split(',') || [],
  databaseUrl: process.env.DATABASE_URL,
  secretKey: process.env.SECRET_KEY
};

// 설정 확인 API
app.get('/config', (req, res) => {
  res.json({
    appName: config.appName,
    port: config.port,
    debug: config.debug,
    maxUsers: config.maxUsers,
    allowedIPs: config.allowedIPs,
    databaseUrl: config.databaseUrl ? '***' : 'NOT SET',
    secretKey: config.secretKey ? '***' : 'NOT SET'
  });
});

app.listen(config.port, () => {
  console.log(`${config.appName}이 ${config.port}번 포트에서 실행중입니다.`);
  console.log('디버그 모드:', config.debug);
});
```

**테스트:**
```bash
curl http://localhost:9000/config
```

## 11. 실전 프로젝트에서 환경변수 구조

```javascript
// config/index.js
require('dotenv').config();

const config = {
  // 서버 설정
  server: {
    port: parseInt(process.env.PORT) || 3000,
    host: process.env.HOST || 'localhost',
    env: process.env.NODE_ENV || 'development'
  },
  
  // 데이터베이스 설정
  database: {
    host: process.env.DB_HOST || 'localhost',
    port: parseInt(process.env.DB_PORT) || 5432,
    name: process.env.DB_NAME || 'myapp',
    username: process.env.DB_USERNAME || 'postgres',
    password: process.env.DB_PASSWORD,
    ssl: process.env.DB_SSL === 'true',
    pool: {
      min: parseInt(process.env.DB_POOL_MIN) || 2,
      max: parseInt(process.env.DB_POOL_MAX) || 10
    }
  },
  
  // 인증 설정
  auth: {
    jwtSecret: process.env.JWT_SECRET,
    jwtExpiry: process.env.JWT_EXPIRY || '24h',
    bcryptRounds: parseInt(process.env.BCRYPT_ROUNDS) || 12
  },
  
  // 외부 API 설정
  api: {
    googleClientId: process.env.GOOGLE_CLIENT_ID,
    googleClientSecret: process.env.GOOGLE_CLIENT_SECRET,
    githubToken: process.env.GITHUB_TOKEN
  },
  
  // 로깅 설정
  logging: {
    level: process.env.LOG_LEVEL || 'info',
    file: process.env.LOG_FILE || 'app.log'
  },
  
  // 캐시 설정
  redis: {
    host: process.env.REDIS_HOST || 'localhost',
    port: parseInt(process.env.REDIS_PORT) || 6379,
    password: process.env.REDIS_PASSWORD
  }
};

// 필수 환경변수 검증
const requiredVars = [
  'DB_PASSWORD',
  'JWT_SECRET'
];

const missingVars = requiredVars.filter(varName => !process.env[varName]);

if (missingVars.length > 0) {
  console.error('❌ 필수 환경변수가 설정되지 않았습니다:');
  missingVars.forEach(varName => {
    console.error(`   - ${varName}`);
  });
  process.exit(1);
}

module.exports = config;
```

```javascript
// server.js
const config = require('./config');
const express = require('express');
const app = express();

console.log('🔧 서버 설정:');
console.log(`   환경: ${config.server.env}`);
console.log(`   포트: ${config.server.port}`);
console.log(`   DB 호스트: ${config.database.host}`);
console.log(`   로그 레벨: ${config.logging.level}`);

app.listen(config.server.port, () => {
  console.log(`🚀 서버가 ${config.server.port}번 포트에서 실행중입니다.`);
});
```

## 핵심 정리

1. **환경변수**: 코드 외부에서 설정하는 값들
2. **용도**: 환경별 설정, 보안 정보, 외부 API 키
3. **파일**: `.env` (개발용), 환경별 분리 가능
4. **접근**: `process.env.변수명`
5. **타입**: 모두 문자열 → 필요시 변환
6. **보안**: `.env` 파일은 git에 커밋 금지
7. **검증**: 필수 환경변수는 서버 시작시 체크
8. **구조화**: config 객체로 정리하여 관리