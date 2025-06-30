# NestJS ORM & Prisma 완전정복

## 📚 목차
1. [ORM이란?](#orm이란)
2. [Prisma 소개](#prisma-소개)
3. [설치 및 초기 설정](#설치-및-초기-설정)
4. [Prisma 스키마 작성](#prisma-스키마-작성)
5. [NestJS와 Prisma 통합](#nestjs와-prisma-통합)
6. [CRUD 작업](#crud-작업)
7. [관계 정의 및 조회](#관계-정의-및-조회)
8. [고급 쿼리](#고급-쿼리)
9. [실습 예제](#실습-예제)

---

## ORM이란?

### 개념
**ORM = Object-Relational Mapping**
"데이터베이스 테이블을 JavaScript 객체처럼 다룰 수 있게 해주는 도구"

### 비유로 이해하기
```
전통적인 방법 (SQL)        →    ORM 방법
─────────────────              ─────────────────
복잡한 외국어                     한국어로 번역
"SELECT * FROM users"           user.findAll()
"INSERT INTO users..."          user.create({...})
"UPDATE users SET..."           user.update({...})
```

### ORM 없이 vs ORM 사용

#### ❌ 어려운 방법: 직접 SQL 작성
```javascript
const mysql = require('mysql2');

const connection = mysql.createConnection({
  host: 'localhost',
  user: 'root',
  password: 'password',
  database: 'myapp'
});

// 사용자 조회
app.get('/users', (req, res) => {
  connection.query('SELECT * FROM users', (error, results) => {
    if (error) {
      return res.status(500).json({ error: error.message });
    }
    res.json(results);
  });
});

// 새 사용자 생성
app.post('/users', (req, res) => {
  const { name, email } = req.body;
  const sql = 'INSERT INTO users (name, email, created_at) VALUES (?, ?, NOW())';
  
  connection.query(sql, [name, email], (error, results) => {
    if (error) {
      return res.status(500).json({ error: error.message });
    }
    res.json({ id: results.insertId, name, email });
  });
});
```

**문제점:**
- SQL 문법 외워야 함 😵
- 오타 나면 에러 💥
- 복잡한 관계 처리 어려움 🤯
- 데이터베이스마다 문법 다름 😱

#### ✅ 쉬운 방법: ORM 사용
```javascript
const { User } = require('./models');

// 사용자 조회
app.get('/users', async (req, res) => {
  try {
    const users = await User.findAll();  // SQL 자동 생성!
    res.json(users);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// 새 사용자 생성
app.post('/users', async (req, res) => {
  try {
    const { name, email } = req.body;
    const user = await User.create({ name, email });  // INSERT 자동!
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

**장점:**
- JavaScript 문법으로 간단 😊
- 자동 완성 지원 💡
- 에러 줄어듦 ✅
- 어떤 DB든 같은 코드 🚀

---

## Prisma 소개

### Prisma란?
**Prisma = 차세대 ORM**
"SQL을 전혀 몰라도 데이터베이스를 쉽게 다룰 수 있게 해주는 최신 도구"

### Prisma vs 기존 ORM
```
기존 ORM (TypeORM)         →    Prisma
─────────────────              ─────────────────
복잡한 데코레이터                 간단한 스키마 파일
@Entity() @Column()            schema.prisma 하나로 끝
수동 타입 정의                   자동 타입 생성
복잡한 설정                     설정 거의 없음
```

### Prisma의 핵심 특징

#### A) 스키마 파일 하나로 모든 것 관리
```prisma
// schema.prisma - 이 파일 하나로 끝!
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id    Int     @id @default(autoincrement())
  name  String
  email String  @unique
  posts Post[]
}

model Post {
  id     Int    @id @default(autoincrement())
  title  String
  user   User   @relation(fields: [userId], references: [id])
  userId Int
}
```

#### B) 자동 타입 생성
```typescript
// TypeScript 타입이 자동으로 생성됨!
const user: User = await prisma.user.create({
  data: {
    name: "홍길동",
    email: "hong@example.com"
  }
});
// User 타입이 자동으로 만들어져서 자동완성 지원!
```

---

## 설치 및 초기 설정

### 1단계: NestJS 프로젝트 생성
```bash
# NestJS CLI 설치 (전역)
npm i -g @nestjs/cli

# 새 프로젝트 생성
nest new my-blog-api

# 프로젝트 폴더로 이동
cd my-blog-api

# 서버 실행 테스트
npm run start:dev
# http://localhost:3000 접속해서 "Hello World!" 확인
```

### 2단계: Prisma 설치
```bash
# Prisma 관련 패키지 설치
npm install prisma @prisma/client

# Prisma 초기화
npx prisma init
```

### 3단계: 데이터베이스 설정 (Docker)
```bash
# PostgreSQL 컨테이너 실행
docker run -d \
  --name nest-postgres \
  -e POSTGRES_PASSWORD=password123 \
  -e POSTGRES_DB=nestjs_blog \
  -e POSTGRES_USER=postgres \
  -p 5432:5432 \
  postgres:15

# 실행 확인
docker ps
```

### 4단계: 환경변수 설정
```bash
# .env 파일
DATABASE_URL="postgresql://postgres:password123@localhost:5432/nestjs_blog"
```

### 생성되는 폴더 구조
```
my-blog-api/
├── src/
│   ├── app.controller.ts
│   ├── app.module.ts
│   ├── app.service.ts
│   └── main.ts
├── prisma/
│   └── schema.prisma          # ← Prisma 모델 정의
├── .env                       # ← 데이터베이스 연결 정보
├── package.json
└── node_modules/
```

---

## Prisma 스키마 작성

### 기본 사용자 모델
```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"  // 또는 "mysql", "sqlite"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  age       Int?     // 선택적 필드 (nullable)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

### 관계가 있는 모델들
```prisma
model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  createdAt DateTime @default(now())
  
  // 관계: 한 사용자가 여러 게시글
  posts     Post[]
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  createdAt DateTime @default(now())
  
  // 관계: 게시글은 한 사용자에게 속함
  author    User     @relation(fields: [authorId], references: [id])
  authorId  Int
}
```

### 데이터베이스 생성 및 동기화
```bash
# 데이터베이스에 테이블 생성
npx prisma db push

# 또는 마이그레이션 파일 생성 (운영환경 권장)
npx prisma migrate dev --name init

# Prisma Client 생성 (타입 포함)
npx prisma generate
```

---

## NestJS와 Prisma 통합

### Prisma 서비스 만들기
```typescript
// src/prisma/prisma.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit {
  async onModuleInit() {
    await this.$connect();
  }
}
```

### User 서비스
```typescript
// src/user/user.service.ts
import { Injectable } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { User } from '@prisma/client';

@Injectable()
export class UserService {
  constructor(private prisma: PrismaService) {}

  // 모든 사용자 조회
  async findAll(): Promise<User[]> {
    return this.prisma.user.findMany();
  }

  // 특정 사용자 조회
  async findOne(id: number): Promise<User | null> {
    return this.prisma.user.findUnique({
      where: { id }
    });
  }

  // 새 사용자 생성
  async create(data: { name: string; email: string; age?: number }): Promise<User> {
    return this.prisma.user.create({
      data
    });
  }

  // 사용자 수정
  async update(id: number, data: { name?: string; email?: string; age?: number }): Promise<User> {
    return this.prisma.user.update({
      where: { id },
      data
    });
  }

  // 사용자 삭제
  async remove(id: number): Promise<User> {
    return this.prisma.user.delete({
      where: { id }
    });
  }
}
```

### User 컨트롤러
```typescript
// src/user/user.controller.ts
import { Controller, Get, Post, Put, Delete, Body, Param, ParseIntPipe } from '@nestjs/common';
import { UserService } from './user.service';
import { User } from '@prisma/client';

@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get()
  async findAll(): Promise<User[]> {
    return this.userService.findAll();
  }

  @Get(':id')
  async findOne(@Param('id', ParseIntPipe) id: number): Promise<User> {
    return this.userService.findOne(id);
  }

  @Post()
  async create(@Body() createUserDto: { name: string; email: string; age?: number }): Promise<User> {
    return this.userService.create(createUserDto);
  }

  @Put(':id')
  async update(
    @Param('id', ParseIntPipe) id: number,
    @Body() updateUserDto: { name?: string; email?: string; age?: number }
  ): Promise<User> {
    return this.userService.update(id, updateUserDto);
  }

  @Delete(':id')
  async remove(@Param('id', ParseIntPipe) id: number): Promise<User> {
    return this.userService.remove(id);
  }
}
```

---

## CRUD 작업

### 기본 CRUD 작업들
```typescript
import prisma from './prisma.service';

// 📝 Create (생성)
const newUser = await prisma.user.create({
  data: {
    name: "홍길동",
    email: "hong@example.com",
    age: 25
  }
});

// 📖 Read (조회)
// 모든 사용자
const users = await prisma.user.findMany();

// 특정 사용자
const user = await prisma.user.findUnique({
  where: { id: 1 }
});

// 조건으로 찾기
const adults = await prisma.user.findMany({
  where: {
    age: {
      gte: 18  // 18세 이상
    }
  }
});

// 🔄 Update (수정)
const updatedUser = await prisma.user.update({
  where: { id: 1 },
  data: { age: 26 }
});

// ❌ Delete (삭제)
await prisma.user.delete({
  where: { id: 1 }
});
```

---

## 관계 정의 및 조회

### 관계 조회하기
```typescript
// 사용자와 그의 모든 게시글 조회
const userWithPosts = await this.prisma.user.findUnique({
  where: { id: 1 },
  include: {
    posts: true  // 관련된 게시글도 함께 가져오기
  }
});

console.log(userWithPosts);
// {
//   id: 1,
//   name: "홍길동",
//   email: "hong@example.com",
//   posts: [
//     { id: 1, title: "첫 번째 글", content: "..." },
//     { id: 2, title: "두 번째 글", content: "..." }
//   ]
// }
```

### 선택적 필드만 조회
```typescript
// 필요한 필드만 선택해서 조회
const users = await this.prisma.user.findMany({
  select: {
    id: true,
    name: true,
    email: true,
    // age는 제외
    posts: {
      select: {
        title: true  // 게시글의 제목만
      }
    }
  }
});
```

---

## 고급 쿼리

### 복잡한 조건
```typescript
// 여러 조건 조합
const result = await this.prisma.user.findMany({
  where: {
    OR: [
      { name: { contains: "김" } },      // 이름에 "김"이 포함
      { age: { gte: 30 } }               // 또는 30세 이상
    ],
    AND: [
      { email: { endsWith: "@gmail.com" } },  // 그리고 Gmail 사용자
      { createdAt: { gte: new Date('2024-01-01') } }  // 그리고 2024년 이후 가입
    ]
  },
  orderBy: {
    createdAt: 'desc'  // 최신 가입자 순
  },
  take: 10,  // 최대 10명
  skip: 0    // 첫 페이지
});
```

### 집계 함수
```typescript
// 통계 정보
const stats = await this.prisma.user.aggregate({
  _count: true,        // 총 사용자 수
  _avg: { age: true }, // 평균 나이
  _max: { age: true }, // 최고 나이
  _min: { age: true }  // 최소 나이
});

console.log(stats);
// {
//   _count: 150,
//   _avg: { age: 25.5 },
//   _max: { age: 65 },
//   _min: { age: 18 }
// }
```

---

## 실습 예제

### 간단한 블로그 API 스키마
```prisma
// prisma/schema.prisma
model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  createdAt DateTime @default(now())
  
  posts     Post[]
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  createdAt DateTime @default(now())
  
  author    User     @relation(fields: [authorId], references: [id])
  authorId  Int
}
```

### 테스트 API 호출
```bash
# 사용자 생성
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{"name": "홍길동", "email": "hong@example.com"}'

# 게시글 생성
curl -X POST http://localhost:3000/posts \
  -H "Content-Type: application/json" \
  -d '{"title": "첫 번째 글", "content": "안녕하세요!", "authorId": 1}'

# 사용자와 게시글 함께 조회
curl http://localhost:3000/users/1?include=posts
```

---

## 유용한 도구들

### Prisma Studio
```bash
# 웹 기반 데이터베이스 관리 도구 실행
npx prisma studio
```

브라우저에서 http://localhost:5555로 접속하면:
- 테이블 데이터 시각적으로 확인
- 데이터 추가/수정/삭제 가능
- 관계 그래프로 보기
- SQL 쿼리 없이 GUI로 관리

### 자주 사용하는 Docker 명령어들
```bash
# 컨테이너 상태 확인
docker ps -a

# 컨테이너 로그 확인
docker logs nest-postgres

# 컨테이너 중지
docker stop nest-postgres

# 컨테이너 시작
docker start nest-postgres

# 컨테이너 삭제 (데이터도 함께 삭제됨)
docker rm -f nest-postgres

# PostgreSQL에 직접 접속 (선택사항)
docker exec -it nest-postgres psql -U postgres -d nestjs_blog
```

---

## 핵심 정리

### Prisma의 장점
- **간단함**: 스키마 파일 하나로 모든 설정
- **타입 안전**: 자동 타입 생성으로 오타 방지
- **자동 완성**: IDE에서 완벽한 자동 완성 지원
- **시각적 도구**: Prisma Studio로 데이터 관리

### NestJS와 조합
- **의존성 주입**으로 깔끔한 코드
- **TypeScript**와 완벽 호환
- **데코레이터** 활용한 API 구조

### 모델과 테이블
- **모델** = 데이터베이스 테이블의 설계도
- **과정**: 모델 작성 → 테이블 생성 → 타입 생성 → 객체 사용
- **한 모델** = 한 테이블 = 한 TypeScript 타입

---

> 💡 **다음 단계**: 실제 프로젝트에서 NestJS + Prisma로 완전한 REST API를 구축해보세요!