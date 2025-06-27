# super 키워드와 상속

## 상속이 뭔가요?

상속은 **"기존 클래스의 특징을 물려받아서 새로운 클래스를 만드는 것"**이에요.

**비유:**
- 부모 클래스 = 스마트폰 설계도 📱
- 자식 클래스 = 게이밍폰 설계도 🎮
- 상속 = 스마트폰 기능 + 게임 특화 기능

## 1. 기본 상속 문법

### extends 키워드로 상속하기

```javascript
// 부모 클래스 (기본 클래스)
class Animal {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  
  eat() {
    return `${this.name}이(가) 음식을 먹습니다.`;
  }
  
  sleep() {
    return `${this.name}이(가) 잠을 잡니다.`;
  }
  
  getInfo() {
    return `이름: ${this.name}, 나이: ${this.age}`;
  }
}

// 자식 클래스 (파생 클래스)
class Dog extends Animal {
  constructor(name, age, breed) {
    super(name, age); // 부모 생성자 호출
    this.breed = breed; // 자식만의 속성
  }
  
  // 자식만의 메서드
  bark() {
    return `${this.name}이(가) 멍멍 짖습니다!`;
  }
  
  // 부모 메서드 재정의 (오버라이드)
  eat() {
    return `강아지 ${this.name}이(가) 사료를 먹습니다.`;
  }
}

// 사용 예시
const myDog = new Dog('멍멍이', 3, '골든리트리버');

console.log(myDog.getInfo());    // 부모로부터 상속받은 메서드
console.log(myDog.bark());       // 자식만의 메서드
console.log(myDog.eat());        // 재정의된 메서드
console.log(myDog.sleep());      // 부모 메서드 그대로 사용
```

## 2. super 키워드 완전 이해

### super의 세 가지 사용법

```javascript
class Vehicle {
  constructor(brand, model, year) {
    this.brand = brand;
    this.model = model;
    this.year = year;
    this.speed = 0;
  }
  
  start() {
    return `${this.brand} ${this.model}이(가) 시동을 겁니다.`;
  }
  
  accelerate(amount) {
    this.speed += amount;
    return `속도가 ${amount}km/h 증가했습니다. 현재 속도: ${this.speed}km/h`;
  }
  
  getDescription() {
    return `${this.year}년식 ${this.brand} ${this.model}`;
  }
}

class Car extends Vehicle {
  constructor(brand, model, year, doors) {
    // 1. super() - 부모 생성자 호출
    super(brand, model, year);
    this.doors = doors;
  }
  
  start() {
    // 2. super.method() - 부모 메서드 호출
    const parentResult = super.start();
    return `${parentResult} 자동차 시스템이 준비되었습니다.`;
  }
  
  accelerate(amount) {
    // 부모 메서드 호출 후 추가 로직
    const result = super.accelerate(amount);
    
    if (this.speed > 120) {
      return `${result} (과속 주의!)`;
    }
    return result;
  }
  
  getFullDescription() {
    // 3. super 프로퍼티 접근
    const baseDesc = super.getDescription();
    return `${baseDesc} (${this.doors}도어)`;
  }
}

// 사용 예시
const myCar = new Car('현대', '소나타', 2023, 4);

console.log(myCar.start());              // 부모 + 자식 로직
console.log(myCar.accelerate(50));       // 50km/h
console.log(myCar.accelerate(80));       // 130km/h (과속 주의!)
console.log(myCar.getFullDescription()); // 전체 설명
```

## 3. 다단계 상속

### 3단계 상속 구조

```javascript
// 1단계: 기본 클래스
class Device {
  constructor(brand, model) {
    this.brand = brand;
    this.model = model;
    this.isOn = false;
  }
  
  turnOn() {
    this.isOn = true;
    return `${this.brand} ${this.model}이(가) 켜졌습니다.`;
  }
  
  turnOff() {
    this.isOn = false;
    return `${this.brand} ${this.model}이(가) 꺼졌습니다.`;
  }
}

// 2단계: 모바일 기기
class MobileDevice extends Device {
  constructor(brand, model, batteryCapacity) {
    super(brand, model);
    this.batteryCapacity = batteryCapacity;
    this.batteryLevel = 100;
  }
  
  turnOn() {
    if (this.batteryLevel <= 0) {
      return '배터리가 부족합니다. 충전해주세요.';
    }
    return super.turnOn() + ' 모바일 시스템이 로드되었습니다.';
  }
  
  useBattery(amount) {
    this.batteryLevel = Math.max(0, this.batteryLevel - amount);
    return `배터리 ${amount}% 사용. 남은 배터리: ${this.batteryLevel}%`;
  }
  
  charge() {
    this.batteryLevel = 100;
    return '충전이 완료되었습니다.';
  }
}

// 3단계: 스마트폰
class Smartphone extends MobileDevice {
  constructor(brand, model, batteryCapacity, storage) {
    super(brand, model, batteryCapacity);
    this.storage = storage;
    this.installedApps = ['전화', '메시지', '설정'];
  }
  
  turnOn() {
    const result = super.turnOn();
    return `${result} 앱 ${this.installedApps.length}개가 준비되었습니다.`;
  }
  
  installApp(appName) {
    this.installedApps.push(appName);
    this.useBattery(2); // 앱 설치시 배터리 2% 소모
    return `${appName} 앱이 설치되었습니다.`;
  }
  
  makeCall(number) {
    if (!this.isOn) {
      return '먼저 전원을 켜주세요.';
    }
    this.useBattery(5); // 통화시 배터리 5% 소모
    return `${number}로 전화를 겁니다.`;
  }
}

// 사용 예시
const iPhone = new Smartphone('Apple', 'iPhone 15', 4000, '256GB');

console.log(iPhone.turnOn());              // 모든 단계의 turnOn 메서드 실행
console.log(iPhone.installApp('게임'));     // 배터리 사용
console.log(iPhone.makeCall('010-1234-5678')); // 통화 + 배터리 사용
console.log(iPhone.useBattery(10));        // 현재 배터리 상태
```

## 4. 추상 클래스 패턴

### 기본 구조만 제공하는 클래스

```javascript
// 추상 클래스 (직접 인스턴스화하지 않을 클래스)
class Shape {
  constructor(color) {
    if (this.constructor === Shape) {
      throw new Error('Shape은 추상 클래스입니다. 직접 인스턴스화할 수 없습니다.');
    }
    this.color = color;
  }
  
  // 추상 메서드 (자식 클래스에서 반드시 구현해야 함)
  getArea() {
    throw new Error('getArea 메서드는 자식 클래스에서 구현해야 합니다.');
  }
  
  getPerimeter() {
    throw new Error('getPerimeter 메서드는 자식 클래스에서 구현해야 합니다.');
  }
  
  // 공통 메서드
  getInfo() {
    return `색상: ${this.color}, 면적: ${this.getArea()}, 둘레: ${this.getPerimeter()}`;
  }
  
  paint(newColor) {
    this.color = newColor;
    return `색상이 ${newColor}로 변경되었습니다.`;
  }
}

// 구체적인 자식 클래스들
class Rectangle extends Shape {
  constructor(color, width, height) {
    super(color);
    this.width = width;
    this.height = height;
  }
  
  // 추상 메서드 구현
  getArea() {
    return this.width * this.height;
  }
  
  getPerimeter() {
    return 2 * (this.width + this.height);
  }
}

class Circle extends Shape {
  constructor(color, radius) {
    super(color);
    this.radius = radius;
  }
  
  // 추상 메서드 구현
  getArea() {
    return Math.PI * this.radius * this.radius;
  }
  
  getPerimeter() {
    return 2 * Math.PI * this.radius;
  }
}

// 사용 예시
const rectangle = new Rectangle('빨강', 10, 5);
const circle = new Circle('파랑', 7);

console.log(rectangle.getInfo()); // 색상: 빨강, 면적: 50, 둘레: 30
console.log(circle.getInfo());    // 색상: 파랑, 면적: 153.94, 둘레: 43.98

// const shape = new Shape('검정'); // 에러! 추상 클래스는 인스턴스화 불가
```

## 5. 실전 예시 - Express 컨트롤러 상속

### 기본 컨트롤러와 특화 컨트롤러

```javascript
// 기본 컨트롤러 클래스
class BaseController {
  constructor() {
    this.items = [];
    this.nextId = 1;
  }
  
  // 공통 메서드들
  getAll(req, res) {
    res.json({
      success: true,
      count: this.items.length,
      data: this.items
    });
  }
  
  getById(req, res) {
    const id = parseInt(req.params.id);
    const item = this.items.find(item => item.id === id);
    
    if (!item) {
      return res.status(404).json({
        success: false,
        message: this.getNotFoundMessage()
      });
    }
    
    res.json({
      success: true,
      data: item
    });
  }
  
  delete(req, res) {
    const id = parseInt(req.params.id);
    const index = this.items.findIndex(item => item.id === id);
    
    if (index === -1) {
      return res.status(404).json({
        success: false,
        message: this.getNotFoundMessage()
      });
    }
    
    const deletedItem = this.items.splice(index, 1)[0];
    
    res.json({
      success: true,
      data: deletedItem,
      message: this.getDeleteMessage()
    });
  }
  
  // 추상 메서드들 (자식 클래스에서 구현)
  getNotFoundMessage() {
    throw new Error('getNotFoundMessage를 구현해주세요');
  }
  
  getDeleteMessage() {
    throw new Error('getDeleteMessage를 구현해주세요');
  }
  
  validateCreateData(data) {
    throw new Error('validateCreateData를 구현해주세요');
  }
}

// 사용자 컨트롤러
class UserController extends BaseController {
  constructor() {
    super();
    this.items = [
      { id: 1, name: '홍길동', email: 'hong@example.com', role: 'user' },
      { id: 2, name: '김관리', email: 'admin@example.com', role: 'admin' }
    ];
    this.nextId = 3;
  }
  
  // 추상 메서드 구현
  getNotFoundMessage() {
    return '사용자를 찾을 수 없습니다';
  }
  
  getDeleteMessage() {
    return '사용자가 삭제되었습니다';
  }
  
  validateCreateData(data) {
    const { name, email } = data;
    const errors = [];
    
    if (!name || name.length < 2) {
      errors.push('이름은 2자 이상이어야 합니다');
    }
    
    if (!email || !email.includes('@')) {
      errors.push('올바른 이메일 형식이 아닙니다');
    }
    
    return errors;
  }
  
  // 사용자 특화 메서드
  create(req, res) {
    const errors = this.validateCreateData(req.body);
    
    if (errors.length > 0) {
      return res.status(400).json({
        success: false,
        message: '유효성 검사 실패',
        errors: errors
      });
    }
    
    const { name, email, role = 'user' } = req.body;
    
    // 중복 이메일 체크
    const existingUser = this.items.find(user => user.email === email);
    if (existingUser) {
      return res.status(409).json({
        success: false,
        message: '이미 존재하는 이메일입니다'
      });
    }
    
    const newUser = {
      id: this.nextId++,
      name,
      email,
      role
    };
    
    this.items.push(newUser);
    
    res.status(201).json({
      success: true,
      data: newUser,
      message: '사용자가 생성되었습니다'
    });
  }
  
  // 부모 메서드 확장
  getAll(req, res) {
    const { role } = req.query;
    
    let filteredItems = this.items;
    
    if (role) {
      filteredItems = this.items.filter(user => user.role === role);
    }
    
    res.json({
      success: true,
      count: filteredItems.length,
      data: filteredItems,
      filter: role ? { role } : null
    });
  }
}

// 상품 컨트롤러
class ProductController extends BaseController {
  constructor() {
    super();
    this.items = [
      { id: 1, name: '노트북', price: 1000000, category: 'electronics' },
      { id: 2, name: '마우스', price: 30000, category: 'electronics' }
    ];
    this.nextId = 3;
  }
  
  // 추상 메서드 구현
  getNotFoundMessage() {
    return '상품을 찾을 수 없습니다';
  }
  
  getDeleteMessage() {
    return '상품이 삭제되었습니다';
  }
  
  validateCreateData(data) {
    const { name, price, category } = data;
    const errors = [];
    
    if (!name || name.length < 1) {
      errors.push('상품명은 필수입니다');
    }
    
    if (!price || price <= 0) {
      errors.push('가격은 0보다 커야 합니다');
    }
    
    if (!category) {
      errors.push('카테고리는 필수입니다');
    }
    
    return errors;
  }
  
  // 상품 특화 메서드
  create(req, res) {
    const errors = this.validateCreateData(req.body);
    
    if (errors.length > 0) {
      return res.status(400).json({
        success: false,
        message: '유효성 검사 실패',
        errors: errors
      });
    }
    
    const { name, price, category } = req.body;
    
    const newProduct = {
      id: this.nextId++,
      name,
      price: parseInt(price),
      category
    };
    
    this.items.push(newProduct);
    
    res.status(201).json({
      success: true,
      data: newProduct,
      message: '상품이 생성되었습니다'
    });
  }
}

// Express 앱에서 사용
const express = require('express');
const app = express();

app.use(express.json());

const userController = new UserController();
const productController = new ProductController();

// 사용자 라우트
app.get('/users', userController.getAll.bind(userController));
app.get('/users/:id', userController.getById.bind(userController));
app.post('/users', userController.create.bind(userController));
app.delete('/users/:id', userController.delete.bind(userController));

// 상품 라우트
app.get('/products', productController.getAll.bind(productController));
app.get('/products/:id', productController.getById.bind(productController));
app.post('/products', productController.create.bind(productController));
app.delete('/products/:id', productController.delete.bind(productController));

app.listen(3000, () => {
  console.log('상속 기반 컨트롤러 서버가 3000번 포트에서 실행중입니다.');
});
```

## 6. 메서드 오버라이딩 심화

### super를 활용한 메서드 확장

```javascript
class Logger {
  log(message) {
    const timestamp = new Date().toISOString();
    console.log(`[${timestamp}] ${message}`);
  }
  
  error(message) {
    this.log(`ERROR: ${message}`);
  }
}

class FileLogger extends Logger {
  constructor(filename) {
    super();
    this.filename = filename;
  }
  
  // 부모 메서드 확장
  log(message) {
    // 부모의 콘솔 로그 실행
    super.log(message);
    
    // 추가로 파일에도 저장 (가상의 코드)
    this.writeToFile(message);
  }
  
  writeToFile(message) {
    console.log(`파일 ${this.filename}에 저장: ${message}`);
  }
  
  // 새로운 메서드 추가
  warn(message) {
    this.log(`WARNING: ${message}`);
  }
}

class DatabaseLogger extends FileLogger {
  constructor(filename, dbConnection) {
    super(filename);
    this.dbConnection = dbConnection;
  }
  
  // 더 확장
  log(message) {
    // 부모의 로그 (콘솔 + 파일)
    super.log(message);
    
    // 추가로 데이터베이스에도 저장
    this.saveToDatabase(message);
  }
  
  saveToDatabase(message) {
    console.log(`데이터베이스에 저장: ${message}`);
  }
}

// 사용 예시
const basicLogger = new Logger();
const fileLogger = new FileLogger('app.log');
const dbLogger = new DatabaseLogger('app.log', 'db-connection');

basicLogger.log('기본 로그');      // 콘솔만
fileLogger.log('파일 로그');       // 콘솔 + 파일
dbLogger.log('DB 로그');           // 콘솔 + 파일 + DB
```

## 7. 실습 과제

다음 상속 구조를 구현해보세요:

### A) 게임 캐릭터 상속

```javascript
// 기본 캐릭터 클래스
class Character {
  constructor(name, hp, attack) {
    this.name = name;
    this.hp = hp;
    this.maxHp = hp;
    this.attack = attack;
  }
  
  // TODO: 공격 메서드 구현
  attackTarget(target) {
    // 데미지 계산하고 target의 hp 감소
  }
  
  // TODO: 회복 메서드 구현
  heal(amount) {
    // hp 회복 (maxHp 초과 불가)
  }
  
  isAlive() {
    return this.hp > 0;
  }
}

// 전사 클래스
class Warrior extends Character {
  constructor(name) {
    // TODO: super 호출 (hp: 120, attack: 25)
    // TODO: 방어력 속성 추가
  }
  
  // TODO: 방어 스킬 구현
  defend() {
    // 방어 모드 토글
  }
  
  // TODO: 강력한 공격 오버라이드
  attackTarget(target) {
    // 기본 공격 + 전사 특화 보너스
  }
}

// 마법사 클래스
class Mage extends Character {
  constructor(name) {
    // TODO: super 호출 (hp: 80, attack: 35)
    // TODO: 마나 속성 추가
  }
  
  // TODO: 마법 공격 구현
  castSpell(target) {
    // 마나 소모해서 강력한 공격
  }
  
  // TODO: 마나 회복 구현
  restoreMana(amount) {
    // 마나 회복
  }
}
```

### B) 은행 계좌 상속

```javascript
// 기본 계좌 클래스
class Account {
  constructor(accountNumber, owner, balance = 0) {
    // TODO: 계좌 정보 초기화
  }
  
  deposit(amount) {
    // TODO: 입금 처리
  }
  
  withdraw(amount) {
    // TODO: 출금 처리 (잔액 부족시 에러)
  }
  
  getBalance() {
    return this.balance;
  }
}

// 적금 계좌
class SavingsAccount extends Account {
  constructor(accountNumber, owner, balance, interestRate) {
    // TODO: super 호출 + 이자율 설정
  }
  
  // TODO: 이자 계산 메서드
  calculateInterest() {
    // 현재 잔액에 이자율 적용
  }
  
  // TODO: 출금 제한 오버라이드
  withdraw(amount) {
    // 한 번에 최대 100만원까지만 출금 가능
  }
}

// 체크 계좌 (당좌예금)
class CheckingAccount extends Account {
  constructor(accountNumber, owner, balance, overdraftLimit) {
    // TODO: super 호출 + 마이너스 한도 설정
  }
  
  // TODO: 마이너스 출금 가능한 출금 오버라이드
  withdraw(amount) {
    // 잔액 + 마이너스 한도까지 출금 가능
  }
}
```

## 핵심 정리

**상속의 핵심 개념:**
1. **extends**: 클래스 상속 키워드
2. **super()**: 부모 생성자 호출 (자식 생성자 첫 줄)
3. **super.method()**: 부모 메서드 호출
4. **오버라이딩**: 부모 메서드 재정의
5. **확장**: 부모 기능 + 자식 기능

**super 사용 규칙:**
- 생성자에서 `super()`는 `this` 사용 전에 호출
- 메서드에서 `super.methodName()`으로 부모 메서드 접근
- 부모의 기능을 완전히 대체하거나 확장 가능

**상속 설계 원칙:**
- **is-a 관계**: 자식은 부모의 한 종류
- **공통 기능**: 부모 클래스에 공통 기능 배치
- **특화 기능**: 자식 클래스에 특화 기능 추가
- **코드 재사용**: 중복 코드 제거

**실무 활용:**
- API 컨트롤러 기본 구조
- 에러 클래스 계층 구조
- 데이터베이스 모델 기본 클래스
- 유틸리티 클래스 확장