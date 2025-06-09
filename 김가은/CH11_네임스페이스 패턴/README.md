# 네임스페이스 패턴

- 네임스페이스는 코드 단위를 고유한 식별자로 그룹화한 것
- 하나의 식별자를 여러 네임스페이스에서 참조할 수 있고, 각 식별자는 중첩된 네임스페이스의 계층구조를 가질 수 있음
- 자바스크립트는 기본적으로 네임스페이스를 지원하지 않지만, 객체와 클로저를 활용해 비슷한 효과를 얻을 수 있음

<br />

## 11.2 단일 전역 변수 패턴
- 하나의 전역 변수를 주요 참조 객체로 사용하는 방식
- 다른 개발자가 같은 이름의 전역 변수를 사용하고 있을 경우 충돌 발생할 수 있음
```js
const myUniqueApplication = (() => {
  function myMethod() {
    return;
  }

  return { myMethod };
})();

myUniqueApplication.myMethod(); 
```

<br />

## 11.3 접두사 네임스페이스 패턴
- 고유한 접두사를 선정한 다음 모든 메서드, 변수, 객체를 이 접두사 뒤에 붙여서 정의
- 애플리케이션이 커짐에 따라 많은 전역 객체가 생성됨
> 🙋‍♂️ 변수명도 필요 이상으로 길어지고, 규모가 커짐에 따라 접두사 관리도 힘들어질 것 같아 별로 사용하고 싶지 않은 패턴...
```js
const myApplication_propetyA = {};
const myApplication_propetyB = {};

function myApplication_myMethod() {

}
```

<br />

## 11.4 객체 리터럴 표기법 패턴
- 깊은 중첩까지 지원하는 구조를 구현할 때 유용함
- 애플리케이션 내이 서로 다른 로직이나 기능을 쉽게 캡슐화하여 깔끔하게 분리 가능
```js
const myApplication = {
  getInfo() {

  },
  models: {},
  views: {
    pages: {},
  },
  collections: {},
};
```

<br />

## 11.5 중첩 네임스페이스 패턴
- 같은 이름의 네임스페이스가 존재하더라도, 하위에 중첩된 네임스페이스까지 일치할 가능성이 낮기 때문에 충돌 위험도 낮음

<br />

## 11.6 즉시 실행 함수 표현식 패턴
- 자바스크립트에서는 즉시 실행 함수로 정의된 내부의 변수와 함수 모두 외부에서 접근할 수 없으므로 쉽게 코드의 은닉성 구현 가능
- 즉시 실행 함수를 사용하면 확장성을 비교적 쉽게 구현 가능
```js
// 네임스페이스의 이름과 undefined를 인자로 전달
((namespace, undefined) => {
  // 비공개 속성들
  const foo = 'foo';
  const bar = 'bar';

  // 공개 메서드와 속성
  namespace.foobar = 'foobar';
  namespace.sayHello = () => {
    speak('hello world');
  };

  // 비공개 메서드
  function speak(msg) {
    console.log(`You said: ${msg}`);
  }
  
  // 전역 네임스페이스에 namespace가 있는지 검사하고, 없을 경우에 window.namespace에 객체 리터럴 할당
})((window.namespace = window.namespace || {}));
```

<br />

## 11.7 네임 스페이스 주입 패턴
- 함수 내에서 this를 네임스페이스의 프록시로 활용하여 특정 네임스페이스에 메서드와 속성 주입
```js
// 전역 네임스페이스 객체 생성
const myApp = myApp || {};
// 서브 네임스페이스 생성
myApp.utils = {};

(function() {
  // getValue, setValue를 통해서만 접근 가능한 은닉된 변수
  let val = 5;

  this.getValue = () => val;

  this.setValue = (newVal) => {
    val = newVal;
  };

  this.tools = {};
}).apply(myApp.utils);

(function() {
  this.diagnose = () => 'diagnosis';
}).apply(myApp.utils.tools);
```

- ES6+ 방식으로 리팩토링
- this나 .apply() 없이도 구현 가능
```js
const myApp = {
  utils: (() => {
    let val = 5;

    const tools = {
      diagnose: () => 'diagnosis',
    };

    return {
      getValue: () => val,
      setValue: (newVal) => val = newVal,
      tools,
    };
  })(),
};
```

<br />

## 11.8 고급 네임스페이스 패턴

### 11.8.1 중첩 네임스페이스 자동화 패턴
- 중첩 네임스페이서 패턴은 추가하고자 하는 계층이 늘어날수록 최상위 네임스페이스에 더 많은 하위 객체들이 정의되어야 함
- 하나의 문자열 인자를 받아서 파싱한 뒤에 필요한 객체를 기반 네임스페이스에 자동으로 추가할 수 있음

```js
const myApp = {};

function extend(ns, ns_string) {
  const parts = ns_string('.');
  let parent = ns;
  let pl;

  pl = parts.length;

  for (let i = 0; i < pl; i++) {
    if (typeof parent[parts[i]] === 'undefined') {
      parent[parts[i]] = {};
    }

    parent = parent[parts[i]];
  }

  return parent;
}

const mod = extend(myApp, 'modules.module2');
console.log(mod);

extend(myApp, 'modueA.moduleB.moduleC.moduleD');
console.log(myApp);
```

<br />

### 11.8.2 의존성 선언 패턴
- 객체에 대한 로컬 참조는 전체적인 조회 시간을 단축함

```js
// 중첩된 네임스페이스에 접근하는 일반적 방법
myApp.utilities.math.fibonacchi(25);
myApp.utilities.math.sin(56);

// 로컬 변수 사용하는 방식
const utils = myApp.utilities;

const maths = utils.math;
const drawing = utils.drawing;

maths.fbonacchi(25);
maths.sin(56);
```

### 11.8.3 심층 객체 확장 패턴
- 객체 리터럴 표기법으로 선언된 네임스페이스는 다른 객체와 쉽게 확장 또는 병합될 수 있음
- 병합 이후에는 두 네임스페이스의 속성과 함수 모두를 동일한 네임스페이스에서 접근할 수 있음
- Lodash.js의 extend() 메서드 사용하면 간단함
  - [assignIn()](https://lodash.com/docs/4.17.15#assignIn) 메서드로 변경됨
  - 객체 간의 속성을 복사(병합)할 때 사용되는 메서드, 상속된 속성까지 복사함  
    | 메서드          | 복사 대상      | 중첩 병합 여부 | 주로 사용하는 상황       |  
    | ------------ | ---------- | -------- | ---------------- |  
    | `_.assign`   | 자체 속성만     | ❌ 아니요    | 일반적인 객체 병합       |  
    | `_.assignIn` | 자체 + 상속 속성 | ❌ 아니요    | 클래스 인스턴스 병합 등    |  
    | `_.merge`    | 자체 속성만     | ✅ 예      | 중첩 객체까지 병합 필요할 때 |  

<br />

## 11.9 권장하는 패턴
- 작가 개인적으로는 중첩 네임스페이스 방법 추천
- 즉시 실행 함수 표현식 패턴과 단일 전역 변수 패턴은 중소 규모의 애플리케이션에서는 잘 작동
