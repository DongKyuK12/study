# 라우터와 HTTP 메서드

## 라우터가 뭔가요?

라우터는 **"URL 경로에 따라 다른 처리를 하는 시스템"**이에요.

**비유:**
- 라우터 = 교통 신호등 🚦
- URL = 목적지 주소
- HTTP 메서드 = 교통수단 (자동차, 버스, 지하철)

```
GET /users     → 사용자 목록 조회
POST /users    → 새 사용자 생성
PUT /users/1   → 1번 사용자 수정
DELETE /users/1 → 1번 사용자 삭제
```

## 1. HTTP 메서드들

### 주요 HTTP 메서드

```javascript
const express = require('express');
const app = express();

app.use(express.json());

// GET - 데이터 조회 (읽기)
app.get('/users', (req, res) => {
  res.json({ message: '사용자 목록 조회' });
});

// POST - 데이터 생성 (쓰기)
app.post('/users', (req, res) => {
  res.json({ message: '새 사용자 생성' });
});

// PUT - 데이터 전체 수정 (수정)
app.put('/users/:id', (req, res) => {
  res.json({ message: `${req.params.id}번 사용자 전체 수정` });
});

// PATCH - 데이터 부분 수정 (수정)
app.patch('/users/:id', (req, res) => {
  res.json({ message: `${req.params.id}번 사용자 부분 수정` });
});

// DELETE - 데이터 삭제 (삭제)
app.delete('/users/:id', (req, res) => {
  res.json({ message: `${req.params.id}번 사용자 삭제` });
});

app.listen(3000);
```

### HTTP 메서드별 특징

| 메서드 | 용도 | 데이터 전송 | 안전성 | 멱등성 |
|--------|------|-------------|--------|--------|
| GET | 조회 | URL 파라미터 | 안전 | 멱등 |
| POST | 생성 | 요청 본문 | 위험 | 비멱등 |
| PUT | 전체수정 | 요청 본문 | 위험 | 멱등 |
| PATCH | 부분수정 | 요청 본문 | 위험 | 비멱등 |
| DELETE | 삭제 | 없음/본문 | 위험 | 멱등 |

## 2. 라우트 매개변수 (Route Parameters)

### URL 경로에서 값 추출하기

```javascript
const express = require('express');
const app = express();

// 기본 매개변수
app.get('/users/:id', (req, res) => {
  const userId = req.params.id;
  res.json({
    message: '사용자 조회',
    userId: userId
  });
});

// 여러 매개변수
app.get('/users/:id/posts/:postId', (req, res) => {
  const { id, postId } = req.params;
  res.json({
    message: '특정 사용자의 특정 게시글',
    userId: id,
    postId: postId
  });
});

// 선택적 매개변수
app.get('/products/:id?', (req, res) => {
  if (req.params.id) {
    res.json({ message: `${req.params.id}번 상품 조회` });
  } else {
    res.json({ message: '모든 상품 조회' });
  }
});

// 와일드카드
app.get('/files/*', (req, res) => {
  const filePath = req.params[0]; // * 부분이 들어감
  res.json({
    message: '파일 조회',
    path: filePath
  });
});

app.listen(3000);
```

**테스트:**
```bash
curl http://localhost:3000/users/123
curl http://localhost:3000/users/123/posts/456
curl http://localhost:3000/products/789
curl http://localhost:3000/products
curl http://localhost:3000/files/images/profile.jpg
```

## 3. 쿼리 파라미터 (Query Parameters)

### URL 뒤에 ? 다음에 오는 데이터

```javascript
// GET /search?q=nodejs&limit=10&page=1
app.get('/search', (req, res) => {
  const query = req.query.q;          // 'nodejs'
  const limit = req.query.limit;      // '10' (문자열!)
  const page = req.query.page;        // '1' (문자열!)
  
  res.json({
    query: query,
    limit: parseInt(limit) || 20,      // 숫자로 변환, 기본값 20
    page: parseInt(page) || 1,         // 숫자로 변환, 기본값 1
    results: []
  });
});

// 더 깔끔한 방법
app.get('/products', (req, res) => {
  const {
    category = 'all',           // 기본값 설정
    minPrice = 0,
    maxPrice = 999999,
    sort = 'name',
    order = 'asc'
  } = req.query;
  
  res.json({
    filters: {
      category,
      priceRange: {
        min: parseInt(minPrice),
        max: parseInt(maxPrice)
      },
      sort,
      order
    }
  });
});
```

**테스트:**
```bash
curl "http://localhost:3000/search?q=express&limit=5&page=2"
curl "http://localhost:3000/products?category=electronics&minPrice=100&maxPrice=500&sort=price&order=desc"
```

## 4. 실전 RESTful API 만들기

### 완전한 CRUD API

```javascript
const express = require('express');
const app = express();

app.use(express.json());

// 가짜 데이터베이스
let products = [
  { id: 1, name: '노트북', price: 1000, category: 'electronics' },
  { id: 2, name: '마우스', price: 30, category: 'electronics' },
  { id: 3, name: '책상', price: 200, category: 'furniture' }
];

let nextId = 4;

// 1. 전체 상품 조회 (필터링, 페이징 지원)
app.get('/products', (req, res) => {
  const {
    category,
    minPrice,
    maxPrice,
    search,
    page = 1,
    limit = 10
  } = req.query;
  
  let filteredProducts = [...products];
  
  // 카테고리 필터
  if (category) {
    filteredProducts = filteredProducts.filter(p => p.category === category);
  }
  
  // 가격 필터
  if (minPrice) {
    filteredProducts = filteredProducts.filter(p => p.price >= parseInt(minPrice));
  }
  
  if (maxPrice) {
    filteredProducts = filteredProducts.filter(p => p.price <= parseInt(maxPrice));
  }
  
  // 검색 필터
  if (search) {
    filteredProducts = filteredProducts.filter(p => 
      p.name.toLowerCase().includes(search.toLowerCase())
    );
  }
  
  // 페이징
  const startIndex = (parseInt(page) - 1) * parseInt(limit);
  const endIndex = startIndex + parseInt(limit);
  const paginatedProducts = filteredProducts.slice(startIndex, endIndex);
  
  res.json({
    success: true,
    data: paginatedProducts,
    pagination: {
      current: parseInt(page),
      total: Math.ceil(filteredProducts.length / parseInt(limit)),
      count: paginatedProducts.length,
      totalItems: filteredProducts.length
    }
  });
});

// 2. 특정 상품 조회
app.get('/products/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const product = products.find(p => p.id === id);
  
  if (!product) {
    return res.status(404).json({
      success: false,
      message: '상품을 찾을 수 없습니다'
    });
  }
  
  res.json({
    success: true,
    data: product
  });
});

// 3. 새 상품 생성
app.post('/products', (req, res) => {
  const { name, price, category } = req.body;
  
  // 유효성 검사
  if (!name || !price || !category) {
    return res.status(400).json({
      success: false,
      message: '이름, 가격, 카테고리는 필수입니다'
    });
  }
  
  if (typeof price !== 'number' || price <= 0) {
    return res.status(400).json({
      success: false,
      message: '가격은 0보다 큰 숫자여야 합니다'
    });
  }
  
  const newProduct = {
    id: nextId++,
    name,
    price,
    category
  };
  
  products.push(newProduct);
  
  res.status(201).json({
    success: true,
    data: newProduct,
    message: '상품이 생성되었습니다'
  });
});

// 4. 상품 전체 수정
app.put('/products/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const productIndex = products.findIndex(p => p.id === id);
  
  if (productIndex === -1) {
    return res.status(404).json({
      success: false,
      message: '상품을 찾을 수 없습니다'
    });
  }
  
  const { name, price, category } = req.body;
  
  // 모든 필드가 필요 (PUT은 전체 교체)
  if (!name || !price || !category) {
    return res.status(400).json({
      success: false,
      message: '모든 필드(이름, 가격, 카테고리)가 필요합니다'
    });
  }
  
  products[productIndex] = {
    id,
    name,
    price,
    category
  };
  
  res.json({
    success: true,
    data: products[productIndex],
    message: '상품이 수정되었습니다'
  });
});

// 5. 상품 부분 수정
app.patch('/products/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const productIndex = products.findIndex(p => p.id === id);
  
  if (productIndex === -1) {
    return res.status(404).json({
      success: false,
      message: '상품을 찾을 수 없습니다'
    });
  }
  
  const { name, price, category } = req.body;
  
  // 제공된 필드만 업데이트
  if (name !== undefined) products[productIndex].name = name;
  if (price !== undefined) products[productIndex].price = price;
  if (category !== undefined) products[productIndex].category = category;
  
  res.json({
    success: true,
    data: products[productIndex],
    message: '상품이 수정되었습니다'
  });
});

// 6. 상품 삭제
app.delete('/products/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const productIndex = products.findIndex(p => p.id === id);
  
  if (productIndex === -1) {
    return res.status(404).json({
      success: false,
      message: '상품을 찾을 수 없습니다'
    });
  }
  
  const deletedProduct = products.splice(productIndex, 1)[0];
  
  res.json({
    success: true,
    data: deletedProduct,
    message: '상품이 삭제되었습니다'
  });
});

app.listen(3000, () => {
  console.log('상품 API 서버가 3000번 포트에서 실행중입니다.');
});
```

## 5. Express Router로 라우트 분리하기

### 라우터 파일 분리

```javascript
// routes/products.js
const express = require('express');
const router = express.Router();

// /products로 시작하는 라우트들
router.get('/', (req, res) => {
  res.json({ message: '모든 상품' });
});

router.get('/:id', (req, res) => {
  res.json({ message: `${req.params.id}번 상품` });
});

router.post('/', (req, res) => {
  res.json({ message: '상품 생성' });
});

module.exports = router;
```

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.json({ message: '모든 사용자' });
});

router.get('/:id', (req, res) => {
  res.json({ message: `${req.params.id}번 사용자` });
});

module.exports = router;
```

```javascript
// server.js
const express = require('express');
const app = express();

// 라우터 import
const productRoutes = require('./routes/products');
const userRoutes = require('./routes/users');

app.use(express.json());

// 라우터 연결
app.use('/products', productRoutes);
app.use('/users', userRoutes);

app.listen(3000);
```

## 6. 라우트 매개변수 검증

### 매개변수 유효성 검사

```javascript
// 숫자 ID 검증
app.param('id', (req, res, next, id) => {
  const numericId = parseInt(id);
  
  if (isNaN(numericId) || numericId <= 0) {
    return res.status(400).json({
      success: false,
      message: 'ID는 양의 정수여야 합니다'
    });
  }
  
  req.params.id = numericId; // 숫자로 변환된 값 저장
  next();
});

// 이제 모든 :id 매개변수가 자동으로 검증됨
app.get('/users/:id', (req, res) => {
  // req.params.id는 이미 숫자로 변환됨
  res.json({ userId: req.params.id });
});

app.get('/products/:id', (req, res) => {
  // req.params.id는 이미 숫자로 변환됨
  res.json({ productId: req.params.id });
});
```

## 7. 라우트 패턴 매칭

### 고급 라우트 패턴

```javascript
// 정확한 문자열 매칭
app.get('/about', (req, res) => {
  res.send('About 페이지');
});

// 문자열 패턴 (와일드카드)
app.get('/ab*cd', (req, res) => {
  res.send('abcd, abxcd, abRANDOMcd 등 매칭');
});

// 선택적 문자
app.get('/ab?cd', (req, res) => {
  res.send('acd, abcd 매칭');
});

// 하나 이상 반복
app.get('/ab+cd', (req, res) => {
  res.send('abcd, abbcd, abbbcd 등 매칭');
});

// 그룹화
app.get('/ab(cd)?e', (req, res) => {
  res.send('abe, abcde 매칭');
});

// 정규표현식
app.get(/.*fly$/, (req, res) => {
  res.send('butterfly, dragonfly 등 fly로 끝나는 모든 경로');
});

// 배열로 여러 경로
app.get(['/users', '/members'], (req, res) => {
  res.send('사용자 또는 멤버 페이지');
});
```

## 8. 미들웨어와 라우터 조합

### 라우트별 미들웨어 적용

```javascript
// 인증 미들웨어
const requireAuth = (req, res, next) => {
  const token = req.headers.authorization;
  if (!token) {
    return res.status(401).json({ error: '토큰이 필요합니다' });
  }
  next();
};

// 관리자 권한 미들웨어
const requireAdmin = (req, res, next) => {
  // 토큰에서 역할 확인 (실제로는 JWT 파싱 등)
  const isAdmin = req.headers['x-role'] === 'admin';
  if (!isAdmin) {
    return res.status(403).json({ error: '관리자 권한이 필요합니다' });
  }
  next();
};

// 로깅 미들웨어
const logRequest = (req, res, next) => {
  console.log(`${req.method} ${req.originalUrl} - ${new Date()}`);
  next();
};

// 라우트에 미들웨어 적용
app.get('/public', (req, res) => {
  res.json({ message: '누구나 접근 가능' });
});

app.get('/private', requireAuth, (req, res) => {
  res.json({ message: '인증된 사용자만 접근 가능' });
});

app.get('/admin', requireAuth, requireAdmin, (req, res) => {
  res.json({ message: '관리자만 접근 가능' });
});

app.post('/users', logRequest, requireAuth, (req, res) => {
  res.json({ message: '로깅 + 인증 후 사용자 생성' });
});

// 여러 미들웨어를 배열로
app.delete('/users/:id', [logRequest, requireAuth, requireAdmin], (req, res) => {
  res.json({ message: '사용자 삭제 (모든 검증 통과)' });
});
```

## 9. API 테스트 예시

### 완전한 API 테스트

```bash
# 1. 모든 상품 조회
curl http://localhost:3000/products

# 2. 필터링된 상품 조회
curl "http://localhost:3000/products?category=electronics&minPrice=50&maxPrice=200"

# 3. 페이징된 상품 조회
curl "http://localhost:3000/products?page=2&limit=5"

# 4. 특정 상품 조회
curl http://localhost:3000/products/1

# 5. 새 상품 생성
curl -X POST http://localhost:3000/products \
  -H "Content-Type: application/json" \
  -d '{"name": "키보드", "price": 80, "category": "electronics"}'

# 6. 상품 전체 수정
curl -X PUT http://localhost:3000/products/1 \
  -H "Content-Type: application/json" \
  -d '{"name": "게이밍 노트북", "price": 1500, "category": "electronics"}'

# 7. 상품 부분 수정
curl -X PATCH http://localhost:3000/products/1 \
  -H "Content-Type: application/json" \
  -d '{"price": 1200}'

# 8. 상품 삭제
curl -X DELETE http://localhost:3000/products/1
```

## 10. 실습 과제

다음 API를 구현해보세요:

```javascript
// 블로그 API 구현
// POST   /posts          - 게시글 생성
// GET    /posts          - 게시글 목록 (페이징, 검색 지원)
// GET    /posts/:id      - 특정 게시글 조회
// PUT    /posts/:id      - 게시글 수정
// DELETE /posts/:id      - 게시글 삭제
// GET    /posts/:id/comments - 게시글의 댓글 목록
// POST   /posts/:id/comments - 댓글 생성

const posts = [
  {
    id: 1,
    title: '첫 번째 게시글',
    content: '내용입니다',
    author: '홍길동',
    createdAt: new Date(),
    comments: [
      { id: 1, content: '좋은 글이네요', author: '김철수' }
    ]
  }
];

// TODO: 위 API들을 구현해보세요!
```

## 핵심 정리

**HTTP 메서드 용도:**
- **GET**: 데이터 조회 (안전, 멱등)
- **POST**: 데이터 생성 (위험, 비멱등) 
- **PUT**: 전체 수정 (위험, 멱등)
- **PATCH**: 부분 수정 (위험, 비멱등)
- **DELETE**: 삭제 (위험, 멱등)

**라우트 패턴:**
- **매개변수**: `/users/:id`
- **선택적**: `/users/:id?`
- **와일드카드**: `/files/*`
- **정규식**: `/regex/`

**RESTful API 설계 원칙:**
1. 명사 사용 (`/users`, `/products`)
2. HTTP 메서드로 동작 표현
3. 일관된 응답 형식
4. 적절한 상태 코드 사용
5. 필터링, 페이징, 정렬 지원