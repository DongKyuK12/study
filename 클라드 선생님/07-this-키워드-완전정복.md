# this 키워드 완전정복

## this가 뭔가요?

`this`는 **"현재 실행되고 있는 함수나 메서드를 호출한 객체"**를 가리키는 키워드예요.

**중요한 특징:**
- `this`의 값은 함수가 **호출되는 방식**에 따라 결정됨
- 함수가 정의된 위치가 아니라 **호출된 상황**이 중요
- JavaScript에서 가장 헷갈리는 개념 중 하나

## 1. this의 기본 동작

### 전역에서의 this

```javascript
console.log(this); // 브라우저: window 객체, Node.js: global 객체

function globalFunction() {
  console.log(this); // 브라우저: window, Node.js: global (strict mode에서는 undefined)
}

globalFunction();
```

### 일반 함수에서의 this

```javascript
// 일반 함수에서 this
function regularFunction() {
  console.log('일반 함수의 this:', this);
}

regularFunction(); // undefined (strict mode) 또는 global 객체

// 화살표 함수에서 this
const arrowFunction = () => {
  console.log('화살표 함수의 this:', this);
};

arrowFunction(); // 상위 스코프의 this (global 또는 undefined)
```

## 2. 객체 메서드에서의 this

### 메서드로 호출할 때

```javascript
const person = {
  name: '홍길동',
  age: 25,
  
  // 일반 메서드
  sayHello: function() {
    console.log(`안녕하세요, 저는 ${this.name}입니다.`); // this = person
    console.log(`나이는 ${this.age}살입니다.`);
  },
  
  // 축약 문법
  getInfo() {
    return `${this.name} (${this.age}세)`; // this = person
  },
  
  // 화살표 함수 (주의!)
  sayBye: () => {
    console.log(`안녕히 가세요, ${this.name}님`); // this ≠ person!
  }
};

person.sayHello(); // ✅ this = person
console.log(person.getInfo()); // ✅ this = person
person.sayBye(); // ❌ this ≠ person (undefined.name 에러)
```

### 메서드를 변수에 할당할 때 (주의!)

```javascript
const person = {
  name: '홍길동',
  sayHello() {
    console.log(`안녕하세요, ${this.name}님!`);
  }
};

// 정상 호출
person.sayHello(); // "안녕하세요, 홍길동님!"

// 메서드를 변수에 할당
const greet = person.sayHello;
greet(); // "안녕하세요, undefined님!" - this가 person이 아님!

// 해결 방법 1: bind 사용
const boundGreet = person.sayHello.bind(person);
boundGreet(); // "안녕하세요, 홍길동님!"

// 해결 방법 2: 화살표 함수로 감싸기
const wrapperGreet = () => person.sayHello();
wrapperGreet(); // "안녕하세요, 홍길동님!"
```

## 3. 클래스에서의 this

### 클래스 메서드

```javascript
class User {
  constructor(name, age) {
    this.name = name; // this = 새로 생성되는 인스턴스
    this.age = age;
  }
  
  // 일반 메서드
  introduce() {
    console.log(`저는 ${this.name}이고, ${this.age}살입니다.`);
  }
  
  // 화살표 함수 메서드 (클래스 필드)
  introduceArrow = () => {
    console.log(`안녕하세요, ${this.name}입니다.`);
  }
  
  // 정적 메서드
  static createAdult(name) {
    return new User(name, 18); // this = User 클래스 자체
  }
}

const user1 = new User('김철수', 30);
const user2 = new User('이영희', 25);

user1.introduce(); // this = user1
user2.introduce(); // this = user2

// 메서드를 변수에 할당
const intro = user1.introduce;
intro(); // ❌ this = undefined (에러 발생)

const introArrow = user1.introduceArrow;
introArrow(); // ✅ this = user1 (화살표 함수라서 바인딩됨)
```

## 4. 이벤트 핸들러에서의 this

### DOM 이벤트에서 this

```javascript
// HTML: <button id="myButton">클릭하세요</button>

const button = document.getElementById('myButton');
const obj = {
  count: 0,
  
  // 일반 함수 - this가 button 요소를 가리킴
  handleClick: function() {
    console.log(this); // <button> 요소
    // this.count++; // 에러! button에는 count가 없음
  },
  
  // 화살표 함수 - this가 obj를 가리킴
  handleClickArrow: () => {
    console.log(this); // obj 객체 (또는 상위 스코프)
    this.count++; // ✅ 정상 동작
  },
  
  // bind 사용
  handleClickBound: function() {
    console.log(this); // obj 객체
    this.count++;
  }
};

// 이벤트 리스너 등록
button.addEventListener('click', obj.handleClick); // this = button
button.addEventListener('click', obj.handleClickArrow); // this = obj
button.addEventListener('click', obj.handleClickBound.bind(obj)); // this = obj
```

## 5. call, apply, bind로 this 제어하기

### call 메서드

```javascript
const person1 = { name: '홍길동' };
const person2 = { name: '김철수' };

function greet(greeting, punctuation) {
  console.log(`${greeting}, ${this.name}${punctuation}`);
}

// call: 첫 번째 인수가 this, 나머지는 함수의 인수
greet.call(person1, '안녕하세요', '!'); // "안녕하세요, 홍길동!"
greet.call(person2, '반갑습니다', '.'); // "반갑습니다, 김철수."
```

### apply 메서드

```javascript
// apply: call과 비슷하지만 인수를 배열로 전달
greet.apply(person1, ['안녕하세요', '!']); // "안녕하세요, 홍길동!"
greet.apply(person2, ['반갑습니다', '.']); // "반갑습니다, 김철수."

// 배열의 최댓값 구하기 (예전 방식)
const numbers = [1, 5, 3, 9, 2];
const max = Math.max.apply(null, numbers); // this는 필요 없어서 null
console.log(max); // 9

// 현재는 스프레드 연산자 사용
const maxModern = Math.max(...numbers); // 더 간단!
```

### bind 메서드

```javascript
const person = { name: '홍길동' };

function introduce(greeting) {
  console.log(`${greeting}, 저는 ${this.name}입니다.`);
}

// bind: this가 고정된 새로운 함수 생성
const boundIntroduce = introduce.bind(person);
boundIntroduce('안녕하세요'); // "안녕하세요, 저는 홍길동입니다."

// 부분 적용(partial application)도 가능
const boundWithGreeting = introduce.bind(person, '반갑습니다');
boundWithGreeting(); // "반갑습니다, 저는 홍길동입니다."

// setTimeout에서 유용
setTimeout(boundIntroduce, 1000, '늦었지만'); // 1초 후 실행
```

## 6. 실전 예시 - Express 컨트롤러

### this 문제와 해결방법

```javascript
class UserController {
  constructor() {
    this.users = [
      { id: 1, name: '홍길동' },
      { id: 2, name: '김철수' }
    ];
  }
  
  // ❌ 일반 메서드 - this 바인딩 문제 발생
  getAllUsers(req, res) {
    console.log(this); // undefined (Express가 함수를 일반 함수로 호출)
    res.json(this.users); // 에러!
  }
  
  // ✅ 화살표 함수 - this가 자동으로 바인딩됨
  getAllUsersArrow = (req, res) => {
    console.log(this); // UserController 인스턴스
    res.json(this.users); // 정상 동작
  }
  
  // ✅ bind를 사용한 해결법
  getAllUsersBound(req, res) {
    console.log(this); // UserController 인스턴스
    res.json(this.users); // 정상 동작
  }
}

const express = require('express');
const app = express();

const userController = new UserController();

// 라우트 등록 방법들
app.get('/users1', userController.getAllUsers); // ❌ 에러 발생
app.get('/users2', userController.getAllUsersArrow); // ✅ 정상 동작
app.get('/users3', userController.getAllUsersBound.bind(userController)); // ✅ 정상 동작

// 더 깔끔한 방법: 생성자에서 bind
class BetterUserController {
  constructor() {
    this.users = [{ id: 1, name: '홍길동' }];
    
    // 생성자에서 미리 bind
    this.getAllUsers = this.getAllUsers.bind(this);
    this.createUser = this.createUser.bind(this);
  }
  
  getAllUsers(req, res) {
    res.json(this.users); // this가 올바르게 바인딩됨
  }
  
  createUser(req, res) {
    const newUser = { id: this.users.length + 1, ...req.body };
    this.users.push(newUser);
    res.json(newUser);
  }
}
```

## 7. this와 화살표 함수

### 화살표 함수의 this는 특별해요

```javascript
const obj = {
  name: '홍길동',
  
  // 일반 함수 - 호출 방식에 따라 this가 달라짐
  regularFunction: function() {
    console.log('일반 함수:', this.name);
    
    // 내부 함수에서 this 문제
    function innerFunction() {
      console.log('내부 함수:', this.name); // undefined (일반 함수)
    }
    innerFunction();
    
    // 화살표 함수로 해결
    const innerArrow = () => {
      console.log('내부 화살표:', this.name); // 홍길동 (상위 스코프의 this)
    };
    innerArrow();
  },
  
  // 화살표 함수 - 상위 스코프의 this 사용
  arrowFunction: () => {
    console.log('화살표 함수:', this.name); // undefined (global의 this)
  }
};

obj.regularFunction();
obj.arrowFunction();
```

### 배열 메서드에서 this 활용

```javascript
class NumberProcessor {
  constructor(numbers) {
    this.numbers = numbers;
    this.multiplier = 2;
  }
  
  // ❌ 일반 함수 사용시 this 문제
  processWrong() {
    return this.numbers.map(function(num) {
      return num * this.multiplier; // this = undefined
    });
  }
  
  // ✅ 화살표 함수로 해결
  processRight() {
    return this.numbers.map(num => num * this.multiplier); // this = NumberProcessor
  }
  
  // ✅ bind로 해결
  processBind() {
    return this.numbers.map(function(num) {
      return num * this.multiplier;
    }.bind(this));
  }
  
  // ✅ thisArg 매개변수 사용
  processThisArg() {
    return this.numbers.map(function(num) {
      return num * this.multiplier;
    }, this); // map의 두 번째 인수로 this 전달
  }
}

const processor = new NumberProcessor([1, 2, 3, 4, 5]);
console.log(processor.processRight()); // [2, 4, 6, 8, 10]
```

## 8. 실습 과제

다음 문제들을 해결해보세요:

### A) this 바인딩 문제 수정

```javascript
const counter = {
  count: 0,
  
  increment() {
    this.count++;
    console.log(`Count: ${this.count}`);
  },
  
  startTimer() {
    // TODO: 1초마다 increment를 호출하되, this가 counter를 가리키도록 수정
    setInterval(this.increment, 1000); // 현재는 this 문제 발생
  }
};

// counter.startTimer(); // this 문제로 에러 발생
```

### B) 이벤트 핸들러 this 문제

```javascript
class ClickCounter {
  constructor() {
    this.clickCount = 0;
  }
  
  handleClick() {
    this.clickCount++;
    console.log(`클릭 횟수: ${this.clickCount}`);
  }
  
  setupEventListener() {
    const button = document.getElementById('myButton');
    // TODO: this가 ClickCounter 인스턴스를 가리키도록 이벤트 리스너 등록
    button.addEventListener('click', this.handleClick); // 현재는 this 문제
  }
}
```

### C) 메서드 체이닝에서 this 활용

```javascript
class Calculator {
  constructor() {
    this.value = 0;
  }
  
  add(num) {
    this.value += num;
    // TODO: 메서드 체이닝이 가능하도록 return 값 설정
  }
  
  multiply(num) {
    this.value *= num;
    // TODO: 메서드 체이닝이 가능하도록 return 값 설정
  }
  
  getResult() {
    return this.value;
  }
}

// 사용 예시: calc.add(5).multiply(2).getResult() === 10
```

## 9. this 디버깅 팁

### this 값 확인하기

```javascript
function debugThis() {
  console.log('=== this 디버깅 ===');
  console.log('this:', this);
  console.log('typeof this:', typeof this);
  console.log('this === window:', this === window); // 브라우저
  console.log('this === global:', this === global); // Node.js
  console.log('this === undefined:', this === undefined);
  console.log('==================');
}

// 다양한 방식으로 호출해서 this 확인
debugThis(); // 일반 호출
({ method: debugThis }).method(); // 객체 메서드로 호출
debugThis.call({ name: 'test' }); // call로 호출
```

### this 문제 해결 체크리스트

```javascript
// ✅ this 문제 해결 체크리스트
const troubleshootThis = {
  // 1. 메서드를 변수에 할당했나?
  checkAssignment() {
    const obj = { method() { return this; } };
    const method = obj.method; // ❌ this 잃어버림
    const boundMethod = obj.method.bind(obj); // ✅ bind로 해결
  },
  
  // 2. 콜백 함수로 전달했나?
  checkCallback() {
    const obj = { 
      data: 'test',
      process() { return this.data; }
    };
    
    // ❌ this 문제
    setTimeout(obj.process, 1000);
    
    // ✅ 해결 방법들
    setTimeout(obj.process.bind(obj), 1000);
    setTimeout(() => obj.process(), 1000);
  },
  
  // 3. 화살표 함수를 잘못 사용했나?
  checkArrowFunction() {
    const obj = {
      name: 'test',
      // ❌ 화살표 함수는 자체 this가 없음
      arrow: () => this.name,
      // ✅ 일반 함수 사용
      regular() { return this.name; }
    };
  }
};
```

## 핵심 정리

**this 결정 규칙 (우선순위순):**
1. **new**: `new Func()` → 새 객체
2. **call/apply/bind**: 명시적으로 지정된 객체
3. **메서드 호출**: `obj.method()` → obj
4. **일반 함수**: strict mode에서 undefined, 아니면 global

**화살표 함수의 this:**
- 상위 스코프의 this를 그대로 사용
- bind, call, apply로 변경 불가
- 메서드로 사용시 주의 필요

**실무 해결책:**
- **bind 사용**: 명시적 바인딩
- **화살표 함수**: 콜백에서 유용
- **생성자에서 bind**: 클래스 메서드
- **변수 저장**: const self = this (구식)

**디버깅 팁:**
- console.log(this)로 확인
- 호출 방식 체크
- strict mode 확인