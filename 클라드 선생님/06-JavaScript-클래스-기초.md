# JavaScript 클래스 기초

## 클래스가 뭔가요?

클래스는 **"비슷한 특징을 가진 객체들을 만들기 위한 설계도"**예요.

**비유:**
- 클래스 = 붕어빵 틀 🧇
- 객체(인스턴스) = 실제 붕어빵들 🧇🧇🧇
- 생성자 = 붕어빵을 만드는 과정

## 1. 클래스 기본 문법

### 클래스 선언하기

```javascript
class User {
  // 생성자 - 객체를 만들 때 실행됨
  constructor(name, age) {
    this.name = name;  // 속성(프로퍼티)
    this.age = age;
  }
  
  // 메서드 - 객체의 기능
  sayHello() {
    return `안녕하세요, 저는 ${this.name}입니다.`;
  }
  
  getAge() {
    return `저는 ${this.age}살입니다.`;
  }
  
  // 나이 증가
  birthday() {
    this.age++;
    return `생일 축하합니다! 이제 ${this.age}살이네요.`;
  }
}

// 클래스로 객체 만들기
const user1 = new User('홍길동', 25);
const user2 = new User('김철수', 30);

console.log(user1.sayHello()); // "안녕하세요, 저는 홍길동입니다."
console.log(user2.getAge());   // "저는 30살입니다."
console.log(user1.birthday()); // "생일 축하합니다! 이제 26살이네요."
```

## 2. 생성자(Constructor) 이해하기

### 생성자의 역할

```javascript
class Product {
  constructor(name, price, category) {
    // 초기값 설정
    this.name = name;
    this.price = price;
    this.category = category;
    this.createdAt = new Date(); // 생성 시간 자동 설정
    this.id = Math.random().toString(36); // 랜덤 ID 생성
    
    console.log(`${name} 상품이 생성되었습니다.`);
  }
  
  getInfo() {
    return {
      id: this.id,
      name: this.name,
      price: this.price,
      category: this.category,
      createdAt: this.createdAt
    };
  }
  
  updatePrice(newPrice) {
    const oldPrice = this.price;
    this.price = newPrice;
    return `가격이 ${oldPrice}원에서 ${newPrice}원으로 변경되었습니다.`;
  }
}

// 객체 생성 - 생성자가 자동으로 호출됨
const laptop = new Product('노트북', 1000000, '전자제품');
const mouse = new Product('마우스', 30000, '전자제품');

console.log(laptop.getInfo());
console.log(mouse.updatePrice(25000));
```

### 기본값이 있는 생성자

```javascript
class Book {
  constructor(title, author, pages = 100, year = new Date().getFullYear()) {
    this.title = title;
    this.author = author;
    this.pages = pages;      // 기본값: 100
    this.year = year;        // 기본값: 현재 연도
    this.isRead = false;     // 고정 기본값
  }
  
  read() {
    this.isRead = true;
    return `${this.title}을(를) 다 읽었습니다!`;
  }
  
  getDescription() {
    return `${this.title} (${this.author}, ${this.year}년, ${this.pages}페이지)`;
  }
}

// 다양한 방법으로 생성
const book1 = new Book('JavaScript 완벽 가이드', '김개발'); // 기본값 사용
const book2 = new Book('Node.js 마스터', '이백엔드', 300, 2023); // 모든 값 지정

console.log(book1.getDescription());
console.log(book2.getDescription());
```

## 3. 메서드 종류들

### A) 인스턴스 메서드 (일반 메서드)

```javascript
class Calculator {
  constructor() {
    this.result = 0;
  }
  
  // 인스턴스 메서드 - 각 객체마다 다르게 동작
  add(number) {
    this.result += number;
    return this;  // 메서드 체이닝을 위해 자기 자신 반환
  }
  
  subtract(number) {
    this.result -= number;
    return this;
  }
  
  multiply(number) {
    this.result *= number;
    return this;
  }
  
  getResult() {
    return this.result;
  }
  
  reset() {
    this.result = 0;
    return this;
  }
}

// 사용법
const calc = new Calculator();
const result = calc.add(10).multiply(2).subtract(5).getResult(); // 15

console.log(result); // 15
```

### B) 정적 메서드 (Static Method)

```javascript
class MathUtils {
  // 정적 메서드 - 클래스 이름으로 직접 호출
  static add(a, b) {
    return a + b;
  }
  
  static subtract(a, b) {
    return a - b;
  }
  
  static multiply(a, b) {
    return a * b;
  }
  
  static getRandomNumber(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
  }
  
  static isPrime(num) {
    if (num < 2) return false;
    for (let i = 2; i <= Math.sqrt(num); i++) {
      if (num % i === 0) return false;
    }
    return true;
  }
}

// 정적 메서드 사용 - new 없이 클래스 이름으로 직접 호출
console.log(MathUtils.add(5, 3));           // 8
console.log(MathUtils.getRandomNumber(1, 10)); // 1~10 사이 랜덤 수
console.log(MathUtils.isPrime(17));         // true

// ❌ 인스턴스로는 호출할 수 없음
// const math = new MathUtils();
// math.add(5, 3); // 에러 발생!
```

## 4. getter와 setter

### 속성에 접근하는 특별한 방법

```javascript
class Temperature {
  constructor(celsius) {
    this._celsius = celsius; // _는 private을 의미하는 관례
  }
  
  // getter - 속성처럼 접근 가능
  get celsius() {
    return this._celsius;
  }
  
  get fahrenheit() {
    return (this._celsius * 9/5) + 32;
  }
  
  get kelvin() {
    return this._celsius + 273.15;
  }
  
  // setter - 속성처럼 설정 가능
  set celsius(value) {
    if (value < -273.15) {
      throw new Error('절대영도보다 낮을 수 없습니다!');
    }
    this._celsius = value;
  }
  
  set fahrenheit(value) {
    this.celsius = (value - 32) * 5/9;
  }
}

// 사용법
const temp = new Temperature(25);

console.log(temp.celsius);    // 25 (getter 호출)
console.log(temp.fahrenheit); // 77 (getter 호출)
console.log(temp.kelvin);     // 298.15 (getter 호출)

// setter 사용
temp.celsius = 30;           // setter 호출
console.log(temp.fahrenheit); // 86

temp.fahrenheit = 100;       // setter 호출
console.log(temp.celsius);   // 37.77...
```

## 5. 실전 예시 - Express 라우터 클래스

### 라우터를 클래스로 만들기

```javascript
class UserController {
  constructor() {
    this.users = [
      { id: 1, name: '홍길동', email: 'hong@example.com' },
      { id: 2, name: '김철수', email: 'kim@example.com' }
    ];
    this.nextId = 3;
  }
  
  // 모든 사용자 조회
  getAllUsers(req, res) {
    res.json({
      success: true,
      count: this.users.length,
      data: this.users
    });
  }
  
  // 특정 사용자 조회
  getUserById(req, res) {
    const id = parseInt(req.params.id);
    const user = this.users.find(u => u.id === id);
    
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
  }
  
  // 새 사용자 생성
  createUser(req, res) {
    const { name, email } = req.body;
    
    if (!name || !email) {
      return res.status(400).json({
        success: false,
        message: '이름과 이메일은 필수입니다'
      });
    }
    
    const newUser = {
      id: this.nextId++,
      name,
      email
    };
    
    this.users.push(newUser);
    
    res.status(201).json({
      success: true,
      data: newUser
    });
  }
  
  // 사용자 수정
  updateUser(req, res) {
    const id = parseInt(req.params.id);
    const userIndex = this.users.findIndex(u => u.id === id);
    
    if (userIndex === -1) {
      return res.status(404).json({
        success: false,
        message: '사용자를 찾을 수 없습니다'
      });
    }
    
    const { name, email } = req.body;
    this.users[userIndex] = { id, name, email };
    
    res.json({
      success: true,
      data: this.users[userIndex]
    });
  }
  
  // 사용자 삭제
  deleteUser(req, res) {
    const id = parseInt(req.params.id);
    const userIndex = this.users.findIndex(u => u.id === id);
    
    if (userIndex === -1) {
      return res.status(404).json({
        success: false,
        message: '사용자를 찾을 수 없습니다'
      });
    }
    
    const deletedUser = this.users.splice(userIndex, 1)[0];
    
    res.json({
      success: true,
      data: deletedUser,
      message: '사용자가 삭제되었습니다'
    });
  }
}

// Express와 함께 사용
const express = require('express');
const app = express();

app.use(express.json());

// 컨트롤러 인스턴스 생성
const userController = new UserController();

// 라우트 연결 (bind 사용해서 this 유지)
app.get('/users', userController.getAllUsers.bind(userController));
app.get('/users/:id', userController.getUserById.bind(userController));
app.post('/users', userController.createUser.bind(userController));
app.put('/users/:id', userController.updateUser.bind(userController));
app.delete('/users/:id', userController.deleteUser.bind(userController));

app.listen(3000, () => {
  console.log('서버가 3000번 포트에서 실행중입니다.');
});
```

## 6. 클래스 vs 객체 리터럴

### 언제 클래스를 사용할까요?

```javascript
// ❌ 간단한 데이터는 객체 리터럴 사용
const simpleUser = {
  name: '홍길동',
  age: 25
};

// ✅ 복잡한 로직이나 여러 인스턴스가 필요하면 클래스 사용
class ComplexUser {
  constructor(name, age) {
    this.name = name;
    this.age = age;
    this.loginHistory = [];
    this.preferences = {};
  }
  
  login() {
    this.loginHistory.push(new Date());
    return `${this.name}님이 로그인했습니다.`;
  }
  
  setPreference(key, value) {
    this.preferences[key] = value;
  }
  
  getLoginCount() {
    return this.loginHistory.length;
  }
}

// 여러 사용자 생성
const users = [
  new ComplexUser('홍길동', 25),
  new ComplexUser('김철수', 30),
  new ComplexUser('이영희', 28)
];

users.forEach(user => {
  console.log(user.login());
});
```

## 7. 실습 과제

다음 클래스들을 구현해보세요:

### A) 은행 계좌 클래스

```javascript
class BankAccount {
  constructor(owner, initialBalance = 0) {
    // TODO: 계좌 소유자, 잔액, 거래 내역 초기화
  }
  
  deposit(amount) {
    // TODO: 입금 처리 및 거래 내역 기록
  }
  
  withdraw(amount) {
    // TODO: 출금 처리 (잔액 부족시 에러)
  }
  
  getBalance() {
    // TODO: 현재 잔액 반환
  }
  
  getTransactionHistory() {
    // TODO: 거래 내역 반환
  }
}

// 사용 예시
const account = new BankAccount('홍길동', 10000);
console.log(account.deposit(5000));    // "5000원이 입금되었습니다. 잔액: 15000원"
console.log(account.withdraw(3000));   // "3000원이 출금되었습니다. 잔액: 12000원"
console.log(account.getBalance());     // 12000
```

### B) 쇼핑카트 클래스

```javascript
class ShoppingCart {
  constructor() {
    // TODO: 장바구니 아이템 배열 초기화
  }
  
  addItem(product, quantity = 1) {
    // TODO: 상품 추가 (이미 있으면 수량 증가)
  }
  
  removeItem(productId) {
    // TODO: 상품 제거
  }
  
  updateQuantity(productId, quantity) {
    // TODO: 수량 변경
  }
  
  getTotalPrice() {
    // TODO: 총 가격 계산
  }
  
  getItemCount() {
    // TODO: 총 아이템 개수
  }
  
  clear() {
    // TODO: 장바구니 비우기
  }
}
```

## 8. 클래스 상속 미리보기

다음 강의에서 배울 내용이지만 미리 맛보기:

```javascript
// 기본 동물 클래스
class Animal {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    return `${this.name}이(가) 소리를 냅니다.`;
  }
}

// 강아지 클래스 - Animal을 상속
class Dog extends Animal {
  speak() {
    return `${this.name}이(가) 멍멍 짖습니다.`;
  }
  
  fetch() {
    return `${this.name}이(가) 공을 가져옵니다.`;
  }
}

const myDog = new Dog('멍멍이');
console.log(myDog.speak()); // "멍멍이가 멍멍 짖습니다."
console.log(myDog.fetch()); // "멍멍이가 공을 가져옵니다."
```

## 핵심 정리

**클래스의 핵심 개념:**
1. **생성자(constructor)**: 객체 초기화
2. **메서드**: 객체의 기능/행동
3. **속성**: 객체의 데이터
4. **정적 메서드**: 클래스 레벨의 기능
5. **getter/setter**: 속성 접근 제어

**클래스 사용 시기:**
- 비슷한 객체를 여러 개 만들어야 할 때
- 복잡한 로직이 필요할 때
- 상태(데이터)와 행동(메서드)을 함께 관리할 때
- 재사용 가능한 컴포넌트가 필요할 때

**실무 활용:**
- API 컨트롤러 클래스
- 데이터베이스 모델 클래스
- 유틸리티 클래스
- 비즈니스 로직 클래스