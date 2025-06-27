 🤔 this가 정확히 뭔가요?

  this는 **"지금 이 함수를 호출한 주체"** 를 가리키는 키워드예요.

  핵심: this가 가리키는 것은 함수를 어떻게 호출했느냐에 따라 바뀝니다!

  ---
  1. 객체 메서드에서의 this
```js
  const person = {
    name: '홍길동',
    age: 25,
    greet: function() {
      console.log(`안녕하세요, ${this.name}입니다!`); // this = person
      console.log(`나이는 ${this.age}살입니다!`);     // this = person
    }
  };

  person.greet();
  // 안녕하세요, 홍길동입니다!
  // 나이는 25살입니다!

  // 👈 person.greet()라고 호출했으므로 this = person
```
  ---
  2. 클래스에서의 this
```js
  class User {
    constructor(name, email) {
      this.name = name;   // 👈 this = 지금 만들어지는 새 객체
      this.email = email;
    }

    introduce() {
      console.log(`이름: ${this.name}`);  // 👈 this = 이 메서드를 호출한 객체
      console.log(`이메일: ${this.email}`);
    }

    updateEmail(newEmail) {
      this.email = newEmail;  // 👈 this = 이 메서드를 호출한 객체
      console.log(`이메일이 ${this.email}로 변경되었습니다.`);
    }
  }

  const user1 = new User('김철수', 'kim@test.com');
  const user2 = new User('박영희', 'park@test.com');

  user1.introduce();
  // 이름: 김철수
  // 이메일: kim@test.com
  // 👈 user1.introduce()이므로 this = user1

  user2.introduce();
  // 이름: 박영희
  // 이메일: park@test.com
  // 👈 user2.introduce()이므로 this = user2

  user1.updateEmail('kim.new@test.com');
  // 이메일이 kim.new@test.com로 변경되었습니다.
  // 👈 user1.updateEmail()이므로 this = user1
```
  ---
  3. this가 헷갈리는 경우들

  A) 함수를 변수에 저장해서 호출
```js
  const person = {
    name: '홍길동',
    greet: function() {
      console.log(`안녕하세요, ${this.name}입니다!`);
    }
  };

  // 정상 호출
  person.greet(); // 안녕하세요, 홍길동입니다! (this = person)

  // 함수를 변수에 저장 후 호출
  const greetFunction = person.greet;
  greetFunction(); // 안녕하세요, undefined입니다! (this = 전역객체 또는 undefined)

  // 👈 person.greet()가 아니라 greetFunction()으로 호출했으므로 this가 달라짐!

  B) 콜백 함수에서의 this

  class Timer {
    constructor(name) {
      this.name = name;
      this.count = 0;
    }

    start() {
      console.log(`${this.name} 타이머 시작!`);

      // ❌ 문제 있는 코드
      setInterval(function() {
        this.count++;  // this가 Timer 객체가 아님!
        console.log(`${this.name}: ${this.count}`); // undefined: NaN
      }, 1000);
    }

    startCorrect() {
      console.log(`${this.name} 타이머 시작!`);

      // ✅ 해결 방법 1: 화살표 함수 사용
      setInterval(() => {
        this.count++;  // 화살표 함수는 부모의 this 사용
        console.log(`${this.name}: ${this.count}`); // 정상 작동
      }, 1000);
    }

    startCorrect2() {
      console.log(`${this.name} 타이머 시작!`);

      // ✅ 해결 방법 2: this를 변수에 저장
      const self = this;
      setInterval(function() {
        self.count++;
        console.log(`${self.name}: ${self.count}`);
      }, 1000);
    }
  }

  const timer = new Timer('내 타이머');
  timer.startCorrect(); // 정상 작동
```
  ---
  4. 화살표 함수에서의 this
  중요: 화살표 함수는 자기만의 this를 만들지 않고 부모의 this를 사용해요!

```js
  class Button {
    constructor(element) {
      this.element = element;
      this.clickCount = 0;
    }

    // 일반 함수
    handleClickNormal() {
      this.element.addEventListener('click', function() {
        this.clickCount++; // ❌ this = button 요소 (Button 객체가 아님)
        console.log(this.clickCount); // undefined
      });
    }

    // 화살표 함수
    handleClickArrow() {
      this.element.addEventListener('click', () => {
        this.clickCount++; // ✅ this = Button 객체
        console.log(`클릭 횟수: ${this.clickCount}`); // 정상 작동
      });
    }
  }
```

🎯 왜 일반 함수에서는 this가 달라질까요?

  핵심 개념부터
  
```js
  // 이 함수를 누가 호출하느냐에 따라 this가 달라져요!
  function sayThis() {
    console.log('this는:', this);
  }

  const obj = { name: '객체', sayThis: sayThis };

  sayThis();     // this = window (브라우저) 또는 global (Node.js)
  obj.sayThis(); // this = obj 객체
```

  ---
  1. addEventListener에서 일어나는 일

```js
  class Button {
    constructor(element) {
      this.element = element;  // DOM 요소 (예: <button>)
      this.clickCount = 0;
    }

    handleClickNormal() {
      console.log('handleClickNormal에서 this:', this); // Button 객체 ✅

      this.element.addEventListener('click', function() {
        console.log('콜백 함수에서 this:', this); // button DOM 요소 ❌
        // ☝️ addEventListener가 이 함수를 "button 요소 컨텍스트"로 호출함!
      });
    }
  }

  왜 이런 일이 일어날까요?

  addEventListener는 내부적으로 이렇게 동작해요:
  // addEventListener의 내부 동작 (단순화)
  element.addEventListener = function(event, callback) {
    // 이벤트 발생시...
    callback.call(element); // 👈 element를 this로 해서 callback 호출!
  };
```
  ---
  2. 단계별로 자세히 보기

  Step 1: 객체 생성과 메서드 호출
```js
  // HTML: <button id="myBtn">클릭</button>
  const button = document.getElementById('myBtn');
  const btnInstance = new Button(button);

  btnInstance.handleClickNormal(); // 👈 이때까지는 this = btnInstance

  Step 2: addEventListener 등록

  handleClickNormal() {
    console.log('1. 여기서 this:', this); // Button {element: button, clickCount: 0}

    this.element.addEventListener('click', function() {
      console.log('2. 여기서 this:', this); // <button id="myBtn">클릭</button>
      // ☝️ 브라우저가 이 함수를 button 요소로 호출함!
    });
  }
```

  Step 3: 실제 클릭 이벤트 발생

```js
  // 사용자가 버튼을 클릭하면 브라우저가 내부적으로 이렇게 실행:
  function 내부브라우저동작() {
    const buttonElement = document.getElementById('myBtn');
    const callback = /* 등록된 콜백 함수 */;

    // callback을 buttonElement 컨텍스트로 호출
    callback.call(buttonElement); // 👈 this = buttonElement
  }
```

  ---
  3. 실제 예시로 확인해보기

```html
  <!DOCTYPE html>
  <html>
  <body>
    <button id="testBtn">테스트 버튼</button>

    <script>
      class Button {
        constructor(element) {
          this.element = element;
          this.clickCount = 0;
          this.name = 'Button 인스턴스';
        }

        handleClickNormal() {
          console.log('=== 일반 함수 버전 ===');
          console.log('메서드에서 this.name:', this.name); // "Button 인스턴스"

          this.element.addEventListener('click', function() {
            console.log('콜백에서 this:', this);
            console.log('콜백에서 this.name:', this.name); // undefined
            console.log('콜백에서 this.id:', this.id); // "testBtn" (button의 id)

            // this.clickCount++; // 에러! button 요소에는 clickCount가 없음
          });
        }

        handleClickArrow() {
          console.log('=== 화살표 함수 버전 ===');
          console.log('메서드에서 this.name:', this.name); // "Button 인스턴스"

          this.element.addEventListener('click', () => {
            console.log('콜백에서 this:', this);
            console.log('콜백에서 this.name:', this.name); // "Button 인스턴스"
            console.log('콜백에서 this.clickCount:', this.clickCount); // 0, 1, 2, ...

            this.clickCount++; // 정상 동작!
            console.log('클릭 횟수:', this.clickCount);
          });
        }
      }

      const btn = document.getElementById('testBtn');
      const btnInstance = new Button(btn);

      // 둘 다 등록해서 비교해보기
      btnInstance.handleClickNormal();
      btnInstance.handleClickArrow();
    </script>
  </body>
  </html>
```
  ---
  4. 화살표 함수는 왜 다를까요?

  화살표 함수의 특별한 규칙:
  - 화살표 함수는 자기만의 this를 만들지 않아요
  - 대신 정의된 위치의 this를 그대로 사용해요
```js
  class Button {
    constructor(element) {
      this.element = element;
      this.clickCount = 0;
    }

    handleClickArrow() {
      // 여기서 this = Button 인스턴스

      this.element.addEventListener('click', () => {
        // 화살표 함수는 여기서 정의됨
        // 따라서 this = 정의된 위치의 this = Button 인스턴스
        console.log(this.clickCount); // Button 인스턴스의 clickCount
      });
    }
  }
```
  ---
  5. 일반 함수를 고치는 방법들

  방법 1: 화살표 함수 사용 (추천)
```js
  handleClick() {
    this.element.addEventListener('click', () => {
      this.clickCount++; // ✅ 정상 동작
    });
  }
```
  방법 2: this를 변수에 저장
```js
  handleClick() {
    const self = this; // this를 변수에 저장

    this.element.addEventListener('click', function() {
      self.clickCount++; // self 사용
      console.log(self.clickCount);
    });
  }
```

  방법 3: bind() 사용
```js
  handleClick() {
    this.element.addEventListener('click', function() {
      this.clickCount++;
      console.log(this.clickCount);
    }.bind(this)); // 👈 this를 바인딩
  }
```
  ---
  6. 실제 작동하는 예시
```js
  // Node.js나 브라우저에서 테스트 가능한 예시
  class Counter {
    constructor() {
      this.count = 0;
      this.name = 'Counter';
    }

    // 일반 함수 - 문제 있음
    startNormal() {
      console.log('startNormal에서 this.name:', this.name); // "Counter"

      setTimeout(function() {
        console.log('setTimeout 콜백에서 this:', this); // global 객체 또는 undefined
        console.log('setTimeout 콜백에서 this.name:', this.name); // undefined
        // this.count++; // 에러!
      }, 1000);
    }

    // 화살표 함수 - 정상 동작
    startArrow() {
      console.log('startArrow에서 this.name:', this.name); // "Counter"

      setTimeout(() => {
        console.log('setTimeout 콜백에서 this:', this); // Counter 인스턴스
        console.log('setTimeout 콜백에서 this.name:', this.name); // "Counter"
        this.count++; // 정상 동작!
        console.log('카운트:', this.count);
      }, 1000);
    }
  }

  const counter = new Counter();
  counter.startNormal(); // 문제 발생
  counter.startArrow();  // 정상 동작
```
  ---
  7. 핵심 정리
```js
  class MyClass {
    constructor() {
      this.value = 42;
    }

    method() {
      // 여기서 this = MyClass 인스턴스

      // ❌ 일반 함수: 호출하는 쪽에서 this 결정
      someAsyncFunction(function() {
        console.log(this.value); // undefined (this가 바뀜)
      });

      // ✅ 화살표 함수: 정의된 위치의 this 사용
      someAsyncFunction(() => {
        console.log(this.value); // 42 (this 유지됨)
      });
    }
  }
```
  기억할 점:
  - 일반 함수: "누가 나를 호출했느냐"에 따라 this 결정
  - 화살표 함수: "내가 어디서 정의되었느냐"의 this 사용


  ---
  5. Express에서의 this 사용 예시
```js
  class UserController {
    constructor() {
      this.users = [
        { id: 1, name: '홍길동' },
        { id: 2, name: '김철수' }
      ];
    }

    // 사용자 목록 조회
    getUsers = (req, res) => {  // 👈 화살표 함수로 this 바인딩
      res.json({
        success: true,
        users: this.users  // this = UserController 인스턴스
      });
    }

    // 사용자 생성
    createUser = (req, res) => {
      const { name } = req.body;
      const newUser = {
        id: this.users.length + 1,  // this = UserController 인스턴스
        name: name
      };

      this.users.push(newUser);

      res.status(201).json({
        success: true,
        user: newUser
      });
    }

    // 사용자 삭제
    deleteUser = (req, res) => {
      const userId = parseInt(req.params.id);

      this.users = this.users.filter(user => user.id !== userId);

      res.json({
        success: true,
        message: '사용자가 삭제되었습니다'
      });
    }
  }

  // Express에서 사용
  const userController = new UserController();

  app.get('/users', userController.getUsers);       // this가 제대로 바인딩됨
  app.post('/users', userController.createUser);
  app.delete('/users/:id', userController.deleteUser);
```
  ---

  6. this 바인딩 방법들

  A) bind() 사용
```js
  class Logger {
    constructor(name) {
      this.name = name;
    }

    log(message) {
      console.log(`[${this.name}] ${message}`);
    }
  }

  const logger = new Logger('API');

  // ❌ this가 사라짐
  const logFunction = logger.log;
  logFunction('테스트'); // [undefined] 테스트

  // ✅ bind로 this 고정
  const boundLogFunction = logger.log.bind(logger);
  boundLogFunction('테스트'); // [API] 테스트

  // Express에서 활용
  app.get('/test', logger.log.bind(logger)); // this가 logger로 고정됨

  B) call(), apply() 사용

  const obj1 = { name: '객체1' };
  const obj2 = { name: '객체2' };

  function greet(message) {
    console.log(`${this.name}: ${message}`);
  }

  // call: this를 직접 지정해서 함수 호출
  greet.call(obj1, '안녕하세요!'); // 객체1: 안녕하세요!
  greet.call(obj2, '반갑습니다!'); // 객체2: 반갑습니다!

  // apply: call과 같지만 인수를 배열로 전달
  greet.apply(obj1, ['안녕하세요!']); // 객체1: 안녕하세요!
```
  ---

  7. 실전 디버깅 팁

  this가 뭔지 확인하는 방법

```js
  class MyClass {
    constructor(name) {
      this.name = name;
    }

    testThis() {
      console.log('this는 뭘까요?', this);
      console.log('this.name:', this.name);
      console.log('this의 타입:', typeof this);
      console.log('this의 생성자:', this.constructor.name);
    }
  }

  const obj = new MyClass('테스트');
  obj.testThis();
  // this는 뭘까요? MyClass { name: '테스트' }
  // this.name: 테스트
  // this의 타입: object
  // this의 생성자: MyClass

  this 문제 해결 체크리스트

  // 1. 메서드를 올바른 방식으로 호출했는가?
  obj.method(); // ✅ 올바름
  const fn = obj.method; fn(); // ❌ this가 사라짐

  // 2. 콜백 함수에서는 화살표 함수를 사용했는가?
  setTimeout(() => this.doSomething(), 1000); // ✅ 올바름
  setTimeout(function() { this.doSomething(); }, 1000); // ❌ this가 다름

  // 3. 클래스 메서드는 화살표 함수로 정의했는가?
  class MyClass {
    method = () => { /* this 안전 */ }  // ✅ 올바름
    method() { /* this 위험할 수 있음 */ }   // ❌ 상황에 따라 다름
  }
```

  ---

  8. server.js에서의 this 사용

  server.js를 보면 클래스를 사용하지 않고 함수형으로 작성되어 있어서 this를 거의
  사용하지 않아요. 하지만 만약 클래스로 리팩토링한다면:
```js
  class DeploymentServer {
    constructor() {
      this.logs = [];
      this.app = express();
      this.setupMiddleware();
      this.setupRoutes();
    }

    setupMiddleware() {
      this.app.use(express.json());
      this.app.use((req, res, next) => {
        this.logRequest(req); // this = DeploymentServer 인스턴스
        next();
      });
    }

    logRequest(req) {
      const logEntry = {
        method: req.method,
        url: req.url,
        timestamp: new Date().toISOString()
      };
      this.logs.push(logEntry); // this = DeploymentServer 인스턴스
    }

    setupRoutes() {
      // 화살표 함수로 this 바인딩 보장
      this.app.post('/deploy', this.handleDeploy);
      this.app.get('/status', this.handleStatus);
    }

    handleDeploy = async (req, res) => {
      try {
        this.logRequest(req); // this = DeploymentServer 인스턴스
        // 배포 로직...
      } catch (error) {
        // 에러 처리...
      }
    }

    handleStatus = (req, res) => {
      res.json({
        totalRequests: this.logs.length, // this = DeploymentServer 인스턴스
        recentLogs: this.logs.slice(-5)
      });
    }
  }

  const server = new DeploymentServer();
```