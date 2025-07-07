# 1장. 자바스크립트 심화

## 런타임 구조

### 콜 스택 (Call Stack)

자바스크립트는 단일 스레드 프로그래밍 언어 -> 한 번에 하나의 작업만 처리
콜 스택은 실행 중인 코드의 위치를 추적하는 데이터 구조

**자바스크립트는 슈퍼맨 한명이 모든 일을 한다**

- 호출 스택이 하나만 존재
- 병렬 작업 처리 불가
- 함수 호출 시 스택에 push, 함수 종료 시 pop (LIFO)

**블로킹 연산이 발생하면 끝난다!**

오래 걸리는 동기적 작업 -> 전체 실행을 차단 (무한 루프, 큰 파일의 동기적 읽기 등)

```js
const [state, setState] = useState(0);

setState(엄청 큰 값); // setState는 대표적인 블로킹 연산 -> 큰 값을 넣으면 무한 루프에 빠질 수 있음
```

```js
const main = () => {
  a();
};
const a = () => {};

main();

// 만약 main()이 실행되면
// stack에 main 함수 추가
// main 함수 내에서 a() 함수 호출
// stack에 a 함수 추가
// a 함수 종료 -> stack에서 a 함수 제거
// main 함수 종료 -> stack에서 main 함수 제거

// 구조
// ...
// e(); --- 이게 끝나기 전까지는
// d(); --- 이게 끝나기 전까지는
// c(); --- 이게 끝나기 전까지는
// b(); --- 이게 끝나기 전까지는
// a(); --- 이게 끝나기 전까지는
// main(); --- 스택에 남아있음 -> 메모리 차지
```

스택 오버플로우 : 엔진이 허용하는 범위보다 더 많은 스택이 쌓여 초과된 상태 -> 메모리 초과 오류

#### 스레드 안정성 (Thread Safety)

단일 스레드이기에, 위의 단점에도 불구하고...

- 경쟁 조건이나 데드락 같은 문제 발생 X
- 공유 자원 접근에 대한 복잡성 감소

경쟁 조건 : 여러 스레드가 동시에 접근하여 데이터를 변경하려고 할 때, 실행 순서에 따라 결과가 달라지는 현상

일반적으로 JS에서는 발생하지 않지만, 비동기 코딩에서는 코드를 이상하게 짜면 발생할 수 있음

```js
// 가상의 멀티스레드 코드 예시 (실제 JS 아님)
let balance = 100; // 공유 자원

// 스레드 1
function withdraw(amount) {
  if (balance >= amount) {
    balance -= amount;
  }
}

// 스레드 2
function deposit(amount) {
  balance += amount;
}

// 두 스레드가 동시에 실행되면?
// 입금 스레드가 먼저 실행되지 않으면, 출금 스레드가 실행되지 않을 수 있다
```

#### 데드락 (Deadlock)

프론트엔드에서 데드락을 마주할 상황은 많지 않지만, JS 벡엔드 상황만 가도 데드락이 발생할 수 있음

두 개 이상의 스레드가 서로의 자원을 기다리며 영원히 블록되는 상태 -> 무한 대기 상태

필요한 4가지 조건 (모두 만족해야 데드락 발생)

지하철에서 만나기로 한 두 내향인

- 상호 배제
  - 연락은 한 명이 먼저 해야하나, 동시에 할 수 없다고 생각 (먼저 전화하면 민폐일거야, 오고 있는 중일거야)
- 점유 대기
  - 둘 다 "나는 도착했고, 너의 연락을 기다리는 중" 이라고 생각
- 비선점
  - 상대방에게 전화를 걸기보다 받기를 기다림. "기다리는 중"이라는 입장 유지
- 순환 대기
  - 내가 너의 연락을 기다리고 있고, 너가 내 연락을 기다리고 있는 상황

"아무거나 좋아" 커플의 메뉴 정하기

- 상호 배제
  - 둘 다 "난 아무거나 좋아 너 맘대로 해"라고 생각
- 점유 대기
  - 둘 다 "난 아무거나 좋아 너 맘대로 해"라고 생각
- 비선점
  - 서로 아무 것도 먼저 정하지 않음
- 순환 대기
  - 상대의 응답을 기다리며 대기

데드락의 문제

- 시스템이 정지된 것처럼 보임
- 외부 개입 없이는 해결되지 않음

```js
// 비동기 코드에서의 데드락 예시
// 콜백 지옥이나 잘못된 비동기 흐름으로 인한 '교착상태' 유사 현상

function transfer(a, b, amount, callback) {
  a.withdraw(amount, () => {
    b.deposit(amount, callback);
  });
}

// a 계좌에서 amount원을 출금 -> b 계좌에 입금 -> callback 함수 호출
// 만약 b.desposit이 pending 상태라면?
// a.withdraw이 종료되지 않음
// 데드락 발생
```

데드락을 피하는 방법

- 자원 획득 순서 통일
- 타임아웃 설정
- 가능하다면, 동시성을 피하기

#### 콜 스택 Summary

콜 스택은

- 원시타입 값을 저장하고
- 함수 호출의 실행 컨텍스트를 저장하고
- 요청이 동기적으로 실행된다. -> 경쟁 조건에 따른 데드락이 일반적으로는 단일 스레드이기 때문에 안 생기지만, 비동기 코딩에서는 코드를 이상하게 짜면 생길 수 있음

### 원시타입

원시 타입 : 객체가 아니며 메스드를 가지지 않는 기본적인 데이터 타입, 실행 컨텍스트 (콜 스택)에 함께 저장됨

- number (2^53 - 1까지 표현 가능, 이 이상부터 Infinity 반환) (53비트는 정수, 1비트는 부호)
- string (문자열)
- boolean (true, false)
- undefined (값이 없음)
- null
- symbol (ES6에서 추가됨)
- BigInt (ES2020에서 추가됨) (값의 범위가 무한대, But 값이 커지면 커질수록 성능 저하)

원시 타입은 변경 불가능(immutable)하며, 변수에 할당될 때 값 자체가 복사된다.

```js
let a = 10;
let b = a;

b = 20;
console.log(a); // 10 (변화 없음)
console.log(b); // 20
```

반면 객체는 참조(주소)가 복사됨 (깊은 복사)

```js
let obj1 = { num: 10 };
let obj2 = obj1; // 참조 복사

obj2.num = 20; // obj1.num도 20으로 변경됨
console.log(obj1.num); // 20, 함께 영향을 받음
```

### 실행 컨텍스트

실행 가능한 코드에 제공할 환경 정보를 모아놓은 객체

- this
- 변수 객체 (Variable Object, VO)
- 스코프 체인

로 구성된다.

#### this

"나"로 비유할 수 있다. 현재 실행 컨텍스트를 참조하는 키워드. 생활 속에서 "나"라는 말은 누구나 쓸 수 있지만, 호출한 사람에 따라 "나"의 존재가 달라지듯이 **this도 어디서 호출되었느냐에 따라 달라진다.**

#### 변수 객체

현재 실행 컨텍스트에서 선언된 변수, 함수, 매개변수 등을 저장하는 객체로, 엔진이 코드를 해석할 때 참조하는 "데이터 저장소"의 역할을 한다.

- 전역 컨텍스트에서 VO는 전역 객체 (브라우저의 window, Node.js의 global)과 동일하고,
- 함수 컨텍스트에서 VO는 활성 객체 (Activation Object, AO)와 동일하고, 함수의 지역 변수, arguments 객체 등을 포함한다
- 변수/함수 선언은 호이스팅 (Hoisting)에 의해 런타임 전에 VO에 미리 등록된다.

```js
// Hoisting 예시 (면접 질문 예상)

console.log(a); // undefined (호이스팅)
var a = 10;

// => 자바스크립트 엔진은 다음과 같이 해석한다

var a = undefined; // 전역 컨텍스트에 a 등록 (undefined로 초기화)
console.log(a); // undefined
a = 10; // 런타임에서 실제 값 할당
console.log(a); // 10

// ------------------------------------------------------------
// 만약 var가 아니라 let이나 const라면?

console.log(a); // ReferenceError: a is not defined, 왜냐하면 let이나 const는 호이스팅이 안 되기 때문
let a = 10;
```

#### 스코프 체인

현재 실행 컨텍스트의 변수 객체와 외부 렉시컬 환경 (Outer Lexical Environment)을 참조하는 링크드 리스트, 변수를 참조할 때 스크프 체인을 따라 상위 스코프로 탐색한다.

- 렉시컬 스코핑 (Lexical Scoping) 기반 : 함수가 정의된 위치에 따라 스코프가 결정됨
- 체인의 최상위는 항상 전역 객체 (브라우저의 window, Node.js의 global)
- [[OuterEnv]] 또는 [[Scope]] 프로퍼티로 외부 환경을 참조함

```js
// 스코프 체인 예시

function outer() {
  var x = 10;
  function inner() {
    console.log(x); // 10 (스코프 체인 사용 -> outer의 x 탐색 후 참조)
  }
  inner();
}
```

### 다시 실행 컨텍스트로 돌아와서...

위 세 개념은 하나의 함수가 실행될 때 참조해야하는 정보의 집합이다. 참조 값을 모두 찾고 environmentRecord에 저장한 뒤, 값이 존재하지 않을 경우 스코프 체인을 따라 상위 스코프로 탐색한다. (Lexical 환경의 외부 환경을 참조한다) (outerEnvironmentReference)

이를 통해 자바스크립트 코드의 환경과 실행 순서를 보장한다.

1. 필요한 환경 정보를 모아 객체(VO)를 구성
2. 이를 콜 스택에 누적 (push)
3. 가장 위에 쌓여있는 컨텍스트와 관련 있는 코드를 실행 (pop)
4. 실행 컨텍스트가 활성화되는 시점에 선언된 변수를 Hoisting 처리
5. 외부 환경 정보 구성
6. this 값 설정

#### environmentRecord

현재 컨텍스트와 관련된 코드의 식별자 정보(매개변수 식별자, 함수 자체, 함수 내부 식별자) 저장

Hoisting 발생 시, Host Object : 전역 실행 컨텍스트는 변수 객체를 생성하는 대신 전역 객체를 활용한다. (예시 : 브라우저의 window, Node.js의 global)

#### outerEnvironmentReference

현재 호출된 함수가 선언될 당시의 Lexical Environment를 참조 -> 스코프, 스코프 체인을 형성

#### 렉시컬 스코프 (정적 스코프) (어휘 범위)

함수가 정의되는 위치(코드 상의 위치)에 따라 스코프가 결정된다. (변수의 유효 범위가 정해진다.) JS의 렉시컬 스코프는 **함수가 태어난 곳 (함수 정의 위치)**를 기준으로 변수의 접근 범위를 정하는 규칙이다.

특징 3가지

1. 정적(Static) 스코프 : 함수가 정의된 위치에서 스코프가 결정
2. 스코프 체인 (Scope Chain) : 변수를 찾을 때 자신의 스코프 -> 상위 스코프 -> ... -> 전역 순으로 탐색
3. 클로저 기반 : 함수가 자신의 정의된 환경을 기억한다.

렉시컬 스코프의 효과 (면접 질문 예상)

1. 코드 예측성 : 함수가 어디서 호출되었는지 예측 가능
2. 함수 생성 시점의 환경 보존 : 함수가 생성될 때 정의된 환경을 기억하고 있어, 함수가 실행될 때 그 환경을 참조할 수 있다. (클로저)
3. private 변수를 모사하여 사용 가능 : 함수 내부에서 선언된 변수는 외부에서 접근 불가능하며, 함수 내부에서만 접근 가능 using 클로저

```js
// 예시 1 : 함수의 중첩과 스코프 체인

const outer = () => {
  const name = "Alice"; // outer 스코프에 name 변수 선언

  const inner = () => {
    console.log(name); // inner 함수는 자신의 스코프 -> outer 스코프 -> 전역 스코프 순으로 탐색하여 name 변수를 찾아 출력, 즉, 만약 outer 스코프에 name 변수가 없다면 전역 스코프에서 찾는다.
  };

  inner();
};

outer(); // Alice 출력
```

```js
// 예시 2 : 클로저와 렉시컬 스코프

const createCounter = () => {
  let count = 0;

  return () => {
    count++;
    console.log(count);
  };
};

const counter = createCounter();
counter(); // 1
counter(); // 2 (렉시컬 스코프 -> count의 상태가 유지된다)

// 반환된 함수는 createCounter 함수의 스코프를 기억한다 -> count 변수를 공유하며 상태를 유지한다.
```

```js
// 예시 3 : 렉시컬 스코프 vs 동적 스코프

const name = "Global Alice";

const sayName = () => {
  console.log(name);
};

const outer = () => {
  const name = "Outer Alice";
  sayName(); // 렉시컬 스코프에서 Global Alice 출력 (사용된 시점이 아닌, 선언된 시점을 따른다)
};

outer();

// 렉시컬 스코프 : sayName은 정의되는 시점의 스코프(전역)를 따른다. (Global Alice)
// 동적 스코프라면, 호출 위치(사용된 시점)에 따라 스코프가 결정되었을 것이다. (Outer Alice) 허나 JS는 렉시컬 스코프를 따른다.
```

| 비교 항목      | 렉시컬 스코프                 | 동적 스코프              |
| -------------- | ----------------------------- | ------------------------ |
| 결정 시점      | 함수 정의 시 (코드 작성 단계) | 함수 호출 시 (런타임)    |
| 변수 탐색 방식 | 정적(코드 구조 기반)          | 동적(호출 스택 기반)     |
| 주요 언어      | JavaScript, Python, C 등      | Bash, Perl(일부 모드) 등 |
| 예측 가능성    | 높음                          | 낮음 (호출 경로에 의존)  |

```js
// 예시 4 : 화살표 함수와 this의 렉시컬 바인딩

const person = {
  name: "Bob",
  greet: function () {
    setTimeout(() => {
      console.log(`Hello, ${this.name}!`); // 화살표 함수는 렉시컬 스코프를 따르기 때문에, this는 함수가 정의된 위치의 this를 참조한다. 즉, 화살표 함수는 렉시컬 this를 사용한다. (따라서, setTimeout의 this는 person 객체를 참조하게 된다.)
    }, 1000);
  },
};

person.greet(); // Hello, Bob

const person = {
  name: "Bob",
  greet: function () {
    setTimeout(function () {
      console.log(`Hello, ${this.name}!`); // 일반 함수는 렉시컬 스코프를 따르지 않기 때문에, this는 함수가 호출된 위치의 this를 참조한다. 즉, 일반 함수는 동적 this를 사용한다.
    }, 1000);
  },
};

person.greet(); // Hello, undefined! (즉 this가 전역 객체를 참조하게 되고, 이 안에는 name이라는 프로퍼티가 없기 때문에 undefined가 출력된다.)
```

#### this의 5가지 주요 바인딩 규칙

```js
// 1. 기본 바인딩 (단독/일반 함수 호출)
function showThis() {
  console.log(this); // 브라우저 : window 객체, Node.js : global 객체
}
showThis(); // 엄격 모드 ('use strict' 사용 시) : undefined
```

```js
// 2. 암시적 바인딩 (객체 메서드 호출)
const obj = {
  name: "Alice",
  greet() {
    console.log(`Hello, ${this.name}!`);
  },
};
obj.greet(); // Hello, Alice! (this는 obj 객체를 참조한다.)
```

```js
// 3. 명시적 바인딩 (call, apply, bind)

function introduce() {
  console.log(`I'm ${this.name}`); // 그냥 호출하면 this는 전역 객체 참조 -> I'm undefined
}

const person = {
  name: "Bob",
};

introduce.call(person); // I'm Bob (this는 person 객체를 참조한다.)
introduce.apply(person); // I'm Bob (this는 person 객체를 참조한다.)

const boundFunc = introduce.bind(person);
boundFunc(); // I'm Bob (this는 person 객체를 참조한다.)
```

call, apply, bind : JS에서 함수의 this를 명시적으로 설정할 때 사용하는 메서드

공통점

- 모두 Function.prototype에 정의된 메서드
- 모두 첫 번째 인자로 this로 사용할 값을 받음
- 모두 원래 함수의 this를 변경해서 실행하거나 새 함수로 반환

차이점

- call : 인자를 쉼표로 구분하여 전달할 수 있음
- apply : 인자를 배열로 전달할 수 있음
- bind : 위 두 메서드와 달리 즉시 실행하지 않고, 새로운 함수를 반환함

```js
function introduce(a, b) {
  console.log(`I'm ${this.name}, ${a} ${b}`);
}

const person = { name: "Bob" };

introduce.call(person, "nice", "to meet you"); // I'm Bob, nice to meet you
introduce.apply(person, ["nice", "to meet you"]); // I'm Bob, nice to meet you
const boundFunc = introduce.bind(person);
boundFunc("nice", "to meet you"); // I'm Bob, nice to meet you
```

```js
// 4. new 바인딩 (생성자 함수)
function Person(name) {
  this.name = name;
}
const charlie = new Person("Charlie");
console.log(charlie.name); // Charlie (this는 charlie 객체를 참조한다.)
```

```js
// 5. 화살표 함수의 this
const outerThis = this; // this는 전역 객체를 참조한다.
const arrowFunc = () => {
  console.log(this === outerThis); // true (화살표 함수는 렉시컬 스코프를 따르기 때문에, this는 함수가 정의된 위치의 this, 즉 전역 객체를 참조한다.)
};
const obj = {
  method: arrowFunc,
};
obj.method(); // true
```

function () {} 과 화살표 함수 () => {} 의 차이점

- function () {}: 일반 함수로, 자체적으로 this 할당 (4번 예시 참고)
- () => {} : 화살표 함수로, 자신의 this를 가지지 않고 상위 스코프의 this를 참조 (5번 예시 참고)

정적 컨텍스트 (Lexical Context) 는 소스코드가 작성된 그 문맥에 실행 컨텍스트나 호출 컨텍스트에 의해 결정됨. 즉 함수가 실행된 위치가 아닌, 정의된 위치에서의 컨텍스트를 참조함.

- 일반 함수 : 메서드, 생성자 함수, 동적 this가 필요한 상황에 사용
- 화살표 함수 : 콜백, 클로저, 정적 this가 필요한 상황에 사용

```js
// 1. 생성자 함수 new로 사용하는 상황

function Person(name) {
  this.name = name;
}
const alice = new Person("Alice"); // 정상 동작

const Person = (name) => {
  this.name = name; // 화살표 함수는 자체적으로 this를 가지지 않기 때문에, 상위 스코프의 this를 참조한다.
};
const alice = new Person("Alice"); // TypeError: Person is not a constructor
```

```js
// 2. 객체 메서드에서의 적합성

const obj = {
  value: 10,
  increment: function () {
    this.value++;
  },
};
obj.increment();
console.log(obj.value); // 11

const obj = {
  value: 10,
  increment: () => {
    this.value++; // 화살표 함수는 자체적으로 this를 가지지 않기 때문에, 상위 스코프의 this(전역 객체)를 참조한다.
  },
};
obj.increment();
console.log(obj.value); // 10 (변화 없음)
```

```js
// 3. prototype 존재 여부

function Car() {}
console.log(Car.prototype); // { ... } (일반 함수는 prototype 존재)

const Car = () => {};
console.log(Car.prototype); // undefined
```

```js
// 4. this의 바인딩

const person = {
  name: "Alice",
  greet: function () {
    console.log(`Hello, ${this.name}!`);
  },
};
person.greet(); // Hello, Alice! (this는 person 객체를 참조한다.)
setTimeout(person.greet, 100); // Hello, undefined! (this는 전역 객체를 참조한다.) 왜냐하면 일반 함수는 호출 시점에 따라 this가 결정되기 때문이다. 즉, 호출 시점에 따라 this가 결정되기 때문에, setTimeout의 콜백 함수는 전역 객체를 참조하게 된다.

const person = {
  name: "Alice",
  greet: () => {
    console.log(`Hello, ${this.name}!`);
  },
};
person.greet(); // Hello, undefined! (this는 person이 아니고, 화살표 함수는 자체적으로 this를 가지지 않기 때문에, 상위 스코프의 this(전역 객체)를 참조한다.)
```

## 힙 (Heap)

힙은 거대한 창고이다. 객체, 배열 등 참조 타입의 데이터들은 모두 힙에 저장된다. 동적 메모리 할당을 위해 관리되는 메모리 영역이다. -> 스택과 함께 메모리 레이아웃을 구성하는 주요 요소이다.

참조 타입 : 객체, 배열, 함수 등 참조 값을 가지는 데이터. 원시 타입보다는 크기가 크고, 메모리 주소를 가지고 있다. 느리지만 메모리 주소를 가지고 있기 때문에 빠르게 접근할 수 있다.

힙 vs 스택

| 비교 항목   | 힙 (Heap)                 | 스택 (Stack)          |
| ----------- | ------------------------- | --------------------- |
| 할당 방식   | 동적 (런타임 중 결정)     | 정적 (컴파일 시 결정) |
| 관리 주체   | 개발자/가비지 컬렉터      | 시스템 (자동 관리)    |
| 접근 속도   | 상대적으로 느림           | 빠름                  |
| 메모리 크기 | 큰 공간, 유연한 확장      | 제한적, 고정 크기     |
| 사용 사례   | 객체, 배열, 동적 자료구조 | 지역 변수, 함수 호출  |

힙 메모리 주의사항

1. 메모리 누수 (Memory Leak) : 메모리 할당 후 해제를 하지 않으면 계속 차지 (Ex. 불필요한 전역 변수 사용)

```js
function createLeak() {
  globalVar = new Array(1000000); // 100만 개의 요소를 가진 배열 생성 후 전역 변수에 할당 -> 이러면 가비지 컬렉터가 globalVar을 해제하지 않고 계속 차지하게 된다.
}
```

2. Fragmentation : 빈번한 할당/해제 -> 메모리 영역이 조각나 효율성이 떨어짐 (기본적으로 제공되는 map, filter, reduce 등의 메서드는 내부적으로 배열을 새로 생성하여 반환하기 때문에 메모리 조각남, 주의 필요!)
3. 성능 오버헤드 : 스택보다 할당/해제 속도가 느림

어떻게 메모리 누수를 분석하는가? : Chrome DevTools와 라이브러리 활용, 코드 레벨에서 메모리 누수를 관리 (WeakMap, WeakSet, 클로저 등)

힙의 특징

1. 동적 할당 (Dynamic Allocation) : 런타임 중에 메모리 크기가 결정 (new, malloc, 객체, 배열 등), 스택과 달리 사용자가 직접 관리
2. 생명 주기 (Lifetime) : 메모리 할당 후 해제 시점이 명확하지 않음 (명시적으로 해제되기 전까지 유지) (가비지 컬렉터에 의해 관리)
3. 크기 유연성 : 필요에 따라 메모리를 확장하거나 축소 가능
4. 전역 접근 가능 : 힙에 할당된 데이터는 프로그램의 어디서나 접근 가능

힙 메모리 누수의 주요 원인

1. 불필요한 전역 변수

```js
window.leakyData = new Array(1000000); // 전역변수는 가비지 컬렉팅되지 않는다.
```

2. 분리된 DOM 요소 (Detached DOM)

```js
const element = document.getElement("div");
document.body.appendChild(element);
document.body.removeChild(element); // DOM에서 해당 요소가 제거됨
// 하지만 element 변수는 여전히 참조 중 -> 메모리 누수 발생
```

3. 클로저에 갇힌 큰 데이터

클로저란, 함수가 외부 변수를 참조하고 있는 상태에서 함수가 종료되어도 외부 변수가 사라지지 않는 현상을 말한다. 클로저에 갇혔다는 말의 뜻은, 외부 변수가 사라지지 않는다는 뜻이다.

```js
function createLeak() {
  const hugeData = new Array(1000000); // 클로저에 갇혀 GC되지 않음
  return () => console.log("Leak!");
}
const leakyFunc = createLeak(); // hugeData가 계속 메모리에 남는다.
```

이 예시 코드에서 hugeData는 클로저에 갇혀 있기 때문에, 가비지 컬렉터가 해제하지 않는다. 왜냐하면 leakyFunc가 종료되어도 hugeData는 참조 중이기 때문이다.

4. 미해제된 이벤트 리스너 / 타이머

```js
element.addEventListener("click", heavyFunction); // 이벤트 리스너가 계속 메모리에 남는다.
const timer = setInterval(heavyFunction, 1000); // 타이머가 계속 메모리에 남는다. clearInterval(timer)를 통해 해제해야 한다.
```

## 이벤트 루프 (Event Loop)

JS는 단일 스레드(Single Thread)로 동작한다. 허나 이러면 성능이 졸라 느려진다. JS는 비동기 처리를 하지 않는 한 절대로 좋은 성능을 낼 수 없다. 따라서 JS는 이벤트 루프를 사용하여 비동기 작업과 논블로킹 IO를 구현하여 멀티 스레드처럼 동작하게 한다.

슈퍼 커피 종업원 예시 : 직원 1명이 슈퍼맨이어서 주문받고 커피 만들고, 청소, 결제 등 혼자서 다한다. 허나 커피 포트의 물을 끓이는 시간은 직원이 할 수 없다. 이때

- 동기적 처리 : 물이 끓을 때까지 멍하니 기다리고 있는다. (`await 물끓이기();`)
- 비동기적 처리 : 물을 올려놓고 다른 일을 먼저 처리하고 있다가, 물이 다 끓었다는 신호(콜백)가 오면 커피를 만든다.

이벤트 루프의 구성

- 콜스택
- 힙
- 태스크 큐
- 마이크로태스크 큐
- 렌더링 파이프라인 (브라우저일 경우)

### 태스크 큐 (Macrotask Queue)

- setTimeout, setInterval, DOM 이벤트, I/O 등의 콜백이 대기하는 큐
- 매크로태스크 라고도 불림
- 렌더링 파이프라인 이후에 실행됨

### 마이크로태스크 큐 (Microtask Queue)

- Promise, queueMicrotask, MutationObserver 등의 콜백이 대기하는 큐
- 매크로태스크 큐보다 우선순위가 높음 (즉, 코드 상에서 setTimeout이 Promise보다 위에 있더라도 Promise가 먼저 실행된다.)
- 렌더링 파이프라인 이전에 실행됨

```js
console.log("start"); // 1. 동기 코드

setTimeout(() => console.log("timeout"), 0); // 4. 매크로태스크

Promise.resolve()
  .then(() => console.log("promise1")) // 3. 마이크로태스크
  .then(() => console.log("promise2")); // 3. 마이크로태스크

console.log("end"); // 2. 동기 코드

// 출력 순서 : start, end, promise1, promise2, timeout
```

### 렌더링 파이프라인 (브라우저 한정)

- CSS 계산 -> 레이아웃 -> 페인팅 작업이 마이크로태스크 이후, 매크로태스크 이전에 수행됨
- 마이크로태스크 큐 -> 렌더링 파이프라인 -> 태스크 큐 순으로 실행됨

### 이벤트 루프의 동작 순서

1. 호출 스택이 비어있는지 확인
   - 스택이 비어있다면 다음 단계 진행
2. 마이크로태스크 큐 처리
   - 큐에 있는 모든 마이크로태스크를 순차적으로 실행
   - Ex. Promise.then(), await의 후속 처리
3. 렌더링 (브라우저)
   - UI 업데이트가 필요한 경우 실행
4. 매크로태스크 큐 처리
   - 한 개의 매크로태스크만 실행한 다음 다시 이벤트 루프 시작
   - Ex. setTimeout, 클릭 이벤트 등

즉, JS는 콜 스택 -> 마이크로태스크 -> 브라우저 렌더링 -> 매크로태스크 순으로 실행된다.

## 콜백과 Promise

JS의 용도 : 처음에는 아무런 의미가 없었음. 아주 단순한 수준의 언어. 웹상에서 아주 단순한 처리들을 담당. 여러 기업이 경쟁하다보니 형식이 통일되지도 않았음. 그래서 복잡한 비동기 처리가 고려사항이 아니었음.

- 콜백 : 함수를 인자로 전달하는 전통적인 방식
- Promise : 비동기 상태를 객체로 관리

### 비동기란?

코드의 실행이 순차적으로 진행되지 않고, 작업이 완료되는 것을 기다리지 않고 다음 작업을 수행하는 방식을 의미. -> 동시에 여러 작업을 처리하거나, 기다리는 시간을 효율적으로 활용하는 것이 핵심임.

예시 : 백엔드에 API 요청을 보내고, 응답 올 때까지 아무런 프로세스를 진행하지 않는다면? 백엔드 요청을 동기적으로 기다리게 되기에 서비스 퍼포먼스가 기하급수적으로 떨어진다.

### 콜백 (Callback)

- 함수를 인자로 전달하는 방식, 비동기 작업 완료 후 실행

```js
function fetchData(callback) {
  setTimeout(() => {
    callback("데이터 로드 완료!");
  }, 1000);
}

fetchData((data) => {
  console.log(data); // 1초 후에 "데이터 로드 완료!" 출력
});
```

#### 단점 : 콜백 지옥 (Callback Hell)

Promise의 등장 이전에는 코드를 다음과 같이 작성해야 했음.

```js
fs.readFile("file.txt", (err, data) => {
  if (err) throw err;
  fs.readFile("file2.txt", (err, data) => {
    if (err) throw err;
    fs.writeFile("output.txt", data1 + data2, (err) => {
      if (err) throw err;
      console.log("작업 완료!");
    });
  });
});
```

- 어떤거 끝나면 어떤거 할지 하나하나 다 정해준다 -> 명령형 프로그래밍

주니어 개발자의 실수 : if, callback으로 depth가 깊어진다. 코드 가독성이 떨어진다. 이를 해결하기 위해선, bracket이 3번 이상 깊어지면 쎄함을 느끼자.

### Promise의 탄생 (ES6 표준에 도입됨)

2015년, 즉 10년 전만해도 콜백 지옥으로 코드 쓸 일이 많았음.

옛날에 nodejs가 처음 나왔을 때 Paypal (일론머스크 + 피터틸)이 주도적으로 사용. 결제 시스템이다 보니 절차 의존적인 상황이 많이 발생 했음. (사용자의 금액 확인 -> 상대 계좌 확인 -> 송금 요청) 이러한 상황들이 모두 콜백 지옥으로 구현됨.

#### Promise의 정의

1. 비동기 작업의 최종 완료/실패를 나타내는 객체
2. then()으로 성공, catch()로 실패 처리
3. 3개의 상태 중 하나를 가짐
   - 대기 (Pending) : 비동기 작업이 진행 중인 상태
   - 이행 (Fulfilled) : 비동기 작업이 성공한 상태
   - 거절 (Rejected) : 비동기 작업이 실패한 상태

연쇄적인 콜백 지옥을 .then()의 메서드 체이닝 방식으로 사용해 가독성 향상

```js
fetchData()
  .then((data) => processData(data))
  .then((result) => saveData(result))
  .catch((error) => console.error(error)); // 실패 시 처리
```

```js
// Promise 쉽게 접근하는 법
const fetchData = () => {
  // some works
  return new Promise();
};

fetchData()
  .then(() => {}) // 성공 시 실행
  .catch(() => {}); // 실패 시 실행
```

#### Promise의 비동기 제어 패턴

##### 병렬 처리 : Promise.all(ES6, ES2015)

```js
Promise.all([fetchData1(), fetchData2(), fetchData3()]).then(
  ([data1, data2, data3]) => {
    console.log(data1, data2, data3); // 모든 비동기 작업이 완료된 후 실행
  }
);
```

```js
// 많이 쓰이지만, 좋은게 아니다! 하나라도 실패하면 모두 실패한다. 이는 사용성을 해친다.
const imageUpload = [
  "1.webp", // 실패
  "2.jpg", // 성공
];

Promis.all([
  uploadWebp(), // 저장 실패
  uploadJpg(),
]) // JPG 이거는 계속 재요청되니까 서버에 불필요한 저장이 발생
  .then()
  .catch((err) => console.log(err)); // 에러 처리가 불명확 : 클라이언트는 왜 오류인지 잘 모름 -> 재요청
```

#### Promise.allSettled(ES2020)

Promise.all()의 문제점을 해결하기 위해 등장한 메서드. 모든 비동기 작업이 완료된 후 결과를 배열로 반환한다. Promise.all()보다 정교하게 비동기 관리를 할 수 있다.

즉, 웬만해서는 Promise.all() 대신 Promise.allSettled()를 사용하자.

```js
const promises = [
  Promise.resolve("성공"),
  Promise.reject("실패"),
  Promise.resolve("또 다른 성공"),
];

Promise.allSettled(promises).then((results) => {
  results.forEach((result) => {
    if (result.status === "fulfilled") {
      console.log("성공 : ", result.value);
    } else {
      console.log("실패 : ", result.reason);
    }
  });
});
```

#### 경쟁 처리 : Promise.race (ES6, ES2015)

뭐든 제일 먼저 끝나는거 확인했으면 끝!

```js
Promise.race([fetchFastAPI(), fetchSlowAPI()]).then((result) => {
  console.log(result); // 가장 빠른 결과만 반환
});
```

단점 : 사용되지 않는 함수(fetchSlowAPI)를 호출하게 된다.

##### 할 수 있는 질문들

1. 만약 race에서 첫번째 요청은 완료됐으나 이어서 진행되는 요청에서 에러가 발생했다면 catch가 발생되는가?

   - 동작 : 이미 첫번째 프로미스가 완료(성공 혹은 실패)된 후에는 나머지 프로미스에서 발생하는 에러는 무시된다.
   - 이유 : Promise.race는 첫번째로 settled된 프로미스의 결과만을 처리하며, 나머지 프로미스의 결과는 관심 대상이 아니다.

2. 만약 하나도 완료되기 전에 특정 요청이 에러를 반환하면 나머지 진행 중인 요청은 계속 진행되는가?

   - 동작 : 어떤 프로미스가 가장 먼저 거부(reject)되면 나머지 진행 중인 요청은 무시되고 Promise.race는 즉시 해당 에러를 반환한다.
   - 이유 : 거부도 "완료(settled)" 상태이기 때문에, 가장 먼저 거부된 프로미스가 Promise.race의 결과를 결정한다.

```js
// Promise.race의 대표적인 예시 외우자!

function fetchWithiTimeout(url, options = {}, timeout = 5000) {
  // API 호출 프로미스
  const fetchPromise = fetch(url, options).then((response) => {
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    return response.json();
  });

  // 타임아웃 프로미스
  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(
      () => reject(new Error(`Request timed out after ${timeout}ms`)),
      timeout
    );
  });

  // Timeout과 프로미스를 race를 붙여 관리하게 되면, 해당 프로미스가 시간보다 늦어지면 실패 처리를 할 수 있다.
  return Promise.race([fetchPromise, timeoutPromise]);
}
```

타임 아웃 설정 API 호출 과정에서는 보통 모듈을 호출하니 안써도 되지 않나요? 내장 timeout들이 있을텐데요? (Axios, ...)

Promise.race는 API의 타임아웃에만 사용하면 된다. (X)
Promise.race는 모든 형태의 비동기 요청에 타임아웃과 같은 로직을 주입할 때 사용할 수 있다. (O)

완성도가 높은 제품이라면 타임아웃을 클라이언트에서도 이중으로 걸어두는 것이 안전하다.

##### AbortController 활용하기

웹 요청을 클라이언트가 취소할 수 있도록 해준다.

```js
function fetchWithAbortTimeout(url, options = {}, timeout = 5000) {
  const controller = new AbortController();
  const { signal } = controller; // signal : 취소 요청을 보낼 수 있는 신호

  // API 호출 프로미스
  const fetchPromise = fetch(url, { ...options, signal }) // fetch에서 signal을 주입
    .then((response) => {
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      return response.json();
    })
    .catch((error) => {
      if (error.name === "AbortError") {
        console.log(`Request aborted after ${timeout}ms`);
      }
      throw error;
    });

  // 타임아웃 프로미스
  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(() => {
      controller.abort(); // timeout Promise 내에서 controller를 취소 요청
      reject(new Error(`Request timed out after ${timeout}ms`));
    }, timeout);
  });

  return Promise.race([fetchPromise, timeoutPromise]);
  // 하수 : 그냥 fetch만 사용한다
  // 중수 : 타임아웃 처리를 추가한다
  // 고수 : AbortController의 Signal을 이용해서 시간이 오래 걸리는 Promise를 취소
}
```

프론트엔드 면접 과제에서 코드 평가를 하게 될 경우 반드시 나오는 놈들

1. request
2. global state
3. props drilling

react-query, useSWR -> data, isLoading 이런 거는 누구나 다한다! 보고싶은 것은

- error, edge case 핸들링 -> AbortController

웬만해서 쓰자! 코드 평가에서 좋은 점수를 받을 수 밖에 없다.

#### Promise.any (ES2021)

주어진 프로미스 중 하나라도 성공하면 즉시 반환 (모두 실패할 때에만 reject)
뭐라도 하나만 성공하면 되는 상황에서 사용

다양한 시도를 통해 하나만 실행되면 되는 시나리오에서 사용

1. 다중 CDN 주소에 대해 리소스를 호출한다
2. 사용자 위치 정보를 획득한다
   - 브라우저의 Geolocation API
   - IP 기반 위치
   - 사용자 설정 위치
3. 실시간 데이터 소스 연결
   - 웹 소캣
   - SSE (Server-Send Events)
   - HTTP 풀링

CDN이란?

- CDN : Content Distribution Network
- 정적 자원들에 대해 접근을 빨리 해줌
- AWS S3 이미지 저장 in 미국 서부, AWS CloudFront, CloudFlare, Akamai 등
- Edge Location 존재 -> 대륙마다 유럽, 아시아, 미국 등에서 호출된 자원에 대한 캐싱 처리 -> 데이터 전송 속도 향상 + 트래픽이 커지면 일정 구간부터 비용이 저렴해진다.

```js
// Promise.any는 동일한 자원을 다른 위치에서 요구할 때 사용한다.
// Promise.race를 쓰면 안되는 이유 : 둘 중 하나가 실패 요청을 빠르게 해버리면 그냥 실패한다. (다중 리소스 호출에 대해서는 any를 사용한다)
Promise.any([
  requestImage("south korea", "1.jpg"),
  requestImage("eastern usa", "1.jpg"),
]);
```

Promise.any와 Promise.race는 유사하다. 하지만 Promise.any는 모든 프로미스가 실패해야 실패 처리를 하고, Promise.race는 빨리 도착한 요청을 반환한다.
즉, 어떤 녀석이 더 좋은지는 개발자가 잘 판단하여야 한다.

또한 작동 방식이 앞선 요청이 완료되었더라도 이후 요청이 계속 유지되므로 불필요한 요청 중단을 처리해주어야 한다. 이때 AborController를 통해 중단하는 것이 Best Practice이다. 즉, 병렬처리에 대해서는 항상 AbortController의 사용을 고려하자.

```js
// requestImage 함수에는 controller.signal이 주입되었다고 가정
Promise.any([
  requestImage("south korea", "1.jpg"),
  requestImage("eastern usa", "1.jpg"),
]).then(() => {
  controller.abort();
});
```

#### Promise.withResolvers (ECMAScript2024)

가장 최신 스펙, Promise를 promise, resolve, reject로 나눈다.

```js
const { promise, resolve, reject } = Promise.withResolvers();

// 나중에 resolve/reject 호출 가능
setTimeout(() => resolve("완료!"), 1000);

promise.then((result) => console.log(result));
```

```js
new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(1);
  }, 1000);
})
  .then((result) => console.log(result))
  .catch((error) => console.log(error));

// resolve를 다른 곳에서 관리하고 싶다!
// ->

let res = null;
let rej = null;
new Promise((resolve, reject) => {
  res = resolve;
  rej = reject;
});

// 이후 resolve가 필요한 곳에서
function a() {
  res(1);
}

// 이런 식으로 쓰면 위에 res, rej, Promise의 정의가 반복된다.
// ->

const { resolve, reject } = Promise.withResolvers();

resolve(1);
```

즉, Promise의 resolve/reject를 외부에서 제어해야 할 때 유용하다. 기존 new Promise() 패턴의 대안이다. 중요한 최신 문법 패턴이니, 잘 알아두도록 하자.

### p-limit을 이용한 순차적 실행

pLimit()에 넣은 인자 만큼 한 번에 처리할 프로미스 개수를 한정하여, 프로미스의 순차적 처리가 필요한 경우 사용할 수 있다.

```js
import pLimit from "p-limit";

const limit = pLimit(1); // pLimit(1) : 동시에 1개의 작업만 실행

const input = [
  limit(() => fetchSomething("foo")),
  limit(() => fetchSomething("bar")),
  limit(() => doSomething("baz")),
];

// Promise.all로 모든 작업을 동시에 실행시켰음에도 불구하고, limit은 순차적으로 실행된다.
const result = await Promise.all(input);
console.log(result);
```

#### 싱클턴 패턴이 적용된 p-limit

pLimit을 여러개 정의해서 혼동하지 말고, 단일 인스턴스로 단 한 번만 생성해서 전역에서 사용하자.

```js
import pLimit from "p-limit";

class LimiterSingleton {
  private static instance: LimiterSingleton;
  private limit: ReturnType<typeof pLimit>;

  private constructor() {
    this.limit = pLimit(1); // 동시에 1개만 실행
  }

  public static getInstance(): LimiterSingleton {
    if (!LimiterSingleton.instance) {
      LimiterSingleton.instance = new LimiterSingleton();
    }
    return LimiterSingleton.instance;
  }

  public run<T>(fn : () => Promise<T>) : Promise<T> {
    return this.limit(fn);
  }
}

export default LimiterSingleton;
```

## Web API

웹 브라우저에서 제공되는 기본기능들, 별도의 선언이나 참조 없이 전역에서 접근 가능함.

왜 리액트에서 Fetch를 참조없이 사용 가능한가? -> fetch가 Web API이기 때문이다.

### 대표적인 Web API

#### Fetch

서버와 HTTP 통신 (REST API 호출)

#### LocalStorage/SessionStorage

클라이언트 측 데이터 저장

#### Intersection Observer API

뷰포트에서 요소의 가시성 관찰 (무한 스크롤, 레이지 로딩)
ex : 구글 애드센스 (광고가 실제로 사용자에게 보였는지 모니터링)

```js
const observer = new IntersectionObserver((entries) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting) {
      loadContent();
    }
  });
});
observer.observe(document.querySelector(".footer"));
// footer를 감시하여 뷰포트에 들어오면 보이는 entry들마다 loadContent() 함수를 호출한다.
```

#### Web Workers

메인 스레드 블로킹 방지 (무거운 계산 작업), 메인 스레드, 싱글 스레드에서 진행되기 어려운 작업들을 넣는다. 별도의 스레드로 처리되기에, 데드락 상황을 고려해야 한다.

사용해야하는 상황

- 클라이언트에서 filter, map, reduce 등 연산이 헤비한 경우
- 낙관적 처리가 필요할 때 (백그라운드 처리가 필요할 때) (이미지 전처리) : 안 되었지만 된 것이라 가정하고 일을 진행하는 것.

사진 업로드 시 용량이 큰 경우가 많다. 서버가 원본 그대로 저장하는 것은 부담스럽다. -> 용량의 상한선을 정하고 resize, optimize, crop을 한다. 또는 png > jpg, webp를 한 후 서버에게 보낸다. 이때 이런 처리가 시간이 좀 걸린다. 이것이 블로킹 상황을 야기한다. 따라서, 이미지 전처리 같은 것을 Web Worker에서 처리하고, 완료가 되지 않았더라도 완료된거라 가정하고 보여준다. (프로필 이미지 변경 등)

```js
const worker = new Worker("worker.js"); // 실행하고자 하는 JS 파일, worker.js
worker.postMessage(data);
worker.onmessage = (event) => console.log(event.data);
```

#### Geolocation API

사용자 위치 정보 획득 after 허용 요청

```js
navigator.geolocation.getCurrentPosition(
  (position) => console.log(position.coords),
  (error) => console.error(error)
);
```

#### Canvas API

동적 그래픽 생성 및 조작

```js
const ctx = canvas.getContext("2d");
ctx.fillStyle = "red";
ctx.fillRect(10, 10, 100, 100);
```

#### WebSocket API

실시간 양방향 통신

```js
const socket = new WebSocket("wss://echo.websocket.org");
socket.onmessage = (event) => console.log(event.data); // 메세지 수신 콜백
socket.send("Hello, WebSocket!"); // 메세지 송신
```

#### Clipboard API

클립보드 작업

```js
navigator.clipboard
  .writeText("복사할 텍스트")
  .then(() => console.log("복사 완료!"));
```

최신 기능으로는 클립보드에 이미지 복사도 가능

```js
const blob = await fetch("image.png").then((res) => res.blob());
await navigator.clipboard.write([new ClipboardItem({ "image/png": blob })]);
```

#### navigator.share 기능

보통 웹 개발 시 링크 공유는 Clipboard API를 이용해 클립보드에 복사하는 정도로 구현되지만, 웹 공유 자체 기능을 사용하면 네이티브 친화적인 공유가 가능해진다.

프론트 개발 시 사용자의 클립보드에 주소를 복사하는 클립보드 복사와 콘텐츠를 직접적으로 공유하는 navigator.share를 병행하여 UI로 분리해서 제공해주는게 가장 좋다.

단, share 기능은 모든 브라우저에서 지원하는 것은 아니다. 따라서, 사용하기 전에 if문으로 확인을 한번 해주어야 한다.

```js
// 웹 공유 기능 지원 여부
if (navigator.share) {
  navigator.share({
    title: "공유 타이틀",
    url: "https://example.com",
  });
}
```

지원읋 해주지 않는 경우, 공유용 모달을 따로 만들어서 활성화시켜주자.

#### navigator.getBattery 기능

배터리 상태 조회 (스마트폰 환경)

navigator 객체의 각 기능은 브라우저 실행 환경에 따라 사용 가능한 기능에 차이가 있음. 따라서 사용 전에 기능이 존재하는지 검사 후 호출하는 것이 안전함. 또한 기능이 없을 경우 예외처리가 꼭 필요함.

```js
navigator.getBattery().then((battery) => {
  console.log(`배터리 레벨: ${battery.level * 100}%`);
  battery.addEventListener("levelchange", updateBatteryLevelUI);
});
```

#### navigator.mediaDevices 기능

카메라 접근 권한 요청하는 기능

```js
navigator.mediaDevices
  .getUserMedia({ video: true })
  .then((stream) => {
    videoElement.srcObject = stream;
  })
  .catch((error) => {
    console.error("미디어 장치 오류", error);
  });
```

### 브라우저 호환성 이슈 대처법

```js
// 기능 감지 패턴
function checkFeature(feature) {
  return feature in navigator;
}

// 사용 예시
if (checkFeature("geolocation")) {
  // 위치 서비스 사용
} else {
  alert("이 브라우저는 위치 서비스를 지원하지 않습니다.");
}

// 폴백 구현 예제 (Geolocation)
function getLocation() {
  if (navigator.geolocation) {
    // 표준 API 사용
  } else {
    // IP 기반 위치 조회 폴백
    fetch("https://ipinfo.io/json").then((res) => res.json());
  }
}
```

사용자의 위치나 카메라, 클립보드 기능 등은 모두 권한을 받아야 하기 때문에 해당 권한이 제대로 승인되었는지 확인 후 사용되도록 코드가 작성되어야 한다.

```js
// 권한 상태 확인
navigator.permissions
  .query({ name: "geolocation" })
  .then((status) => console.log(status.state));
```

### Local Storage

브라우저에서 데이터를 유지해야 하는 상황일 때 가장 많이 사용되는 저장소로, 세션 스토리지는 탭이 닫히면 사라지기 때문에 비교적 덜 사용되는 반면, 로컬 스토리지는 브라우저가 닫혀도 유지되기 때문에 널리 사용된다.

#### 전역 커스텀 훅을 통한 로컬 스토리지 예시

```js
import { useEffect, useState } from "react";

// 파싱 로직, stringify 로직이 반복되기에 커스텀 훅으로 빼서 사용하는 것이 좋다.
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

  const setValue = (value) => {
    try {
      const valueToStore =
        value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue];
}
```

로컬 스토리지를 사용하기 위해서는 브라우저에서 window를 통해 접근해야 하므로, `typeof window !== "undefined"` 조건문으로 많이 판별한다.

```js
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(initialValue);

  useEffect(() => {
    // 맨 처음 마운트되면
    try {
      if (typeof window !== "undefined") {
        const item = window.localStorage.getItem(key);
        setStoredValue(item ? JSON.parse(item) : initialValue);
      }
    } catch (error) {
      console.error(error);
    }
  }, [key, initialValue]);

  // rest of hook ...
}
```

로컬 스토리지 사용시에는 데이터를 serializa, deserialize 과정이 필요하다.

```js
// Storing
localStorage.setItem("key", JSON.stringify(data));

// Retrieving
const data = JSON.parse(localStorage.getItem("key"));
```

로컬 스토리지 사용시 나오는 에러는 크게

- 접근 에러
- 한계 용량 에러 (QuotaExceededError)

로 나뉜다. 이에 대한 에러 캐칭을 적절히 해주어야 한다. 특히 로컬 스토리지 관련 훅 같은거 만들 때 Set 부분에서.

클라이언트 사이드 캐싱을 꼭 해주어야 할까? -> 그렇다! 필요한 상황

- 서버에서 GET 요청 리스트를 보여줄 때
- 이미 한번 가져왔는데 또 가져오면 안 좋은 상황 (중복 요청이 걱정되는 상황)

```js
try {
  localStorage.setItem("key", JSON.stringify(data));
} catch (error) {
  if (error instanceof DOMException && error.name === "QuotaExceededError") {
    // handle storage full error
    console.error("로컬 스토리지 용량 초과");
  } else {
    // handle other errors
    console.error("로컬 스토리지 접근 에러", error);
  }
}
```

보통은 클린업 시나리오를 포함하여 로컬 스토리지를 영구적으로 보관하지 않도록 관리해주는게 좋다. 어떤 코드던, 데이터가 메모리에 할당되었을 때의 시나리오 뿐만 아니라 메모리에서 해제되었을 때의 시나리오도 섬세하게 고려해주어야 한다.

### Session Storage

세션 스토리지는 로컬 스토리지와 달리 탭을 닫으면 날아간다. 그래서 보통 잘 안쓴다. 이거 쓸바엔 세션 스토리지를 쓰낟. 하지만 무조건 사용되는 몇가지 대표 아이디어는 기억해두자. 목적에 따라 잘 사용하면 매우 강력한 도구이다.

#### 폼 데이터 임시 저장 (옛날 패턴)

```js
// 폼 입력 시 저장
form.addEventListener("input", (e) => {
  sessionStorage.setItem("formData", JSON.stringify(formData));
});

// 페이지 로드 시 복구
window.addEventListener("load", () => {
  const savedData = JSON.parse(sessionStorage.getItem("formData"));
  if (savedData) {
    populateForm(savedData);
  }
});
```

#### 사용자 진행 상태 추적

```js
// 다음 단계로 이동 시
sessionStorage.setItem("currentStep", "step3");

// 페이지 재방문 시
const currentStep = sessionStorage.getItem("currentStep");
showStep(currentStep);
```

사용자가 폼을 순서대로 입력하는 페이지가 있을 때, 1개 이상의 폼을 서비스에서 동시에 입력 가능한 경우 전역 상태 관리를 쓰지 않고 세션 스토리지를 쓰는게 오히려 멀티 탭 환경에 대응하기에 좋다.

예시 (에어비앤비 예약 페이지) : 여러 페이지를 켜놓았다. 한 곳에서는 A라는 숙소에서, 한 곳에서는 B라는 숙소에서 묵는 예약을 동시에 진행하기 위해. 만약 페이지에서 로컬 스토리지 내의 하나의 상태에 의존한다면, 분리된 처리가 되지 않는다. 하지만 세션 스토리지를 이용한다면? 두 페이지에서 각각 예약을 진행할 수 있다.

arirbnb.com/seoul/예약 - a라는 숙소 정보가 보여지고 있음
arirbnb.com/seoul/예약 - b라는 숙소 정보가 보여지고 있음

동일한 라우트에 대하여 동일한 로컬 스토리지가 사용되기에, 로컬 스토리지가 오버라이드될 위험이 존재한다.

따라서 프론트엔드에서 개발할 때 라우팅은 사용자가 입력한 데이터를 쿼리 파라미터로 전달하는 것이 좋다.

예시 : /서울/예약?2025-06-01

이를 탭 단에서 관리하는 것이 세션이다. 스텝 등 탭 상에서만 일시적으로 유효한 데이터를 저장한다.

정리하자면, 결제 프로세스 페이지를 일괄 라우트로 지정하고, 전역 상태 관리를 통해서 관리하게 되면 취약점이 발생한다.
따라서, 세션 스토리지 + URL 쿼리 스트링을 조합해 탭과 사용자의 요청 페이지의 정보를 완전히 분리하여 그것에 대응되는 데이터가 전달되도록 설정하자.

#### 멀티탭 환경에서의 상태 관리

```js
// 탭별로 독립적인 작업 진행 가능
const tabId = Math.random().toString(36).substring(2);
sessionStorage.setItem("currentTabId", tabId);
```

현재는 전역 상태 관리나 useState 등 컴포넌트 수준 상태 관리가 많이 사용되다보니, 세션 스토리지는 멀티탭 상황에 대응하기 아주 좋은 방법이 된다.

#### 페이지 전환 간 데이터 전달

```js
// 페이지 이동 전
sessionStorage.setItem(
  "tempData",
  JSON.stringify({ from: "pageA", data: someData })
);

// 대상 페이지에서
const tempData = JSON.parse(sessionStorage.getItem("tempData"));
sessionStorage.removeItem("tempData"); // 클린업 프로세스
```

#### 임시 대용량 데이터 저장

```js
function generateReport(data) {
  sessionStorage.setItem("reportData", JSON.stringify(processData(data)));
  window.open("/report-viewer");
}
```

#### 보편적으로 사용될 수 있는 세션 스토리지 전역 커스텀 훅

```js
import { useEffect, useState } from "react";

function useSessionStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const storedValue =
      typeof window !== "undefined" ? window.sessionStorage.getItem(key) : null;
    return storedValue ? JSON.parse(storedValue) : initialValue;
  });

  useEffect(() => {
    window.sessionStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue];
}

// 사용 예시
function CheckoutProcess() {
  const [shippingInfo, setShippingInfo] = useSessionStorage("shippingInfo", {});
  // ...
}
```

세션 스토리지의 보편적 사용 목적

1. 일시적인 데이터 저장이 필요할 때
2. 사용자 세션 동안만 유지되어야 할 정보
3. 민감하지 않지만 탭별로 독립적인 데이터
4. 사용자가 브라우저를 닫으면 자동으로 삭제되어야 하는 데이터 (클린업을 해주자!)
