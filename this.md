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