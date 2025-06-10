### 네임스페이스란?
네임스페이스(Namespace)는 서로 관련된 코드 단위를  고유한 식별자로 그룹화 한 것을 의미한다.
전역 변수 남용을 피하고, 코드 충돌을 방지하며, 재사용성과 유지 보수성을 높이기 위해 사용한다.

## 네임스페이스의 기초
- 서로 다른 스크립트 파일에서 변수명이나 함수명이 충돌하는 문제를 방지
- 특히 서드파티 라이브러리, 다른 개발자의 코드와 충돌을 예방
👉 대규모 자바스크립트 어플리케이션을 관리하기 위해 필요하다.

**네임스페이스 종류**
- 단일 전역 변수
- 접두사 네임스페이스
- 객체 리터럴 표기법
- 중첩 네임스페이스
- 즉시 실행 함수 표현식 (IIFE)
- 네임스페이스 주입

## 단일 전역 변수 패턴
**하나의 전역 객체**를 생성하고 그 안에 필요한 기능을 모아두는 패턴이다.

```js
const myUniqueApplication = (() => {
  function myMethod() {
    // 내부 로직
    return;
  }

  return {
    myMethod,
  };
})();

// 사용법
myUniqueApplication.myMethod();
```

IIFE를 통해 고유한 네임스페이스를 생성하여 사용한다.
외부에서는 myUniqueApplication 객체만 접근 가능하며, 내부 구현은 은닉된다.

하지만 다른 개발자가 같은 이름(myUniqueApplication)의 전역 변수를 이미 사용하고 있다면 충돌될 수 있다.

##  접두사 네임스페이스 패턴
단일 전역 변수 충돌을 방지하기 위해, **모든 전역 식별자에 접두사(prefix)** 를 붙여 사용하는 패턴이다.

```js
const myApplication_propertyA = {};
function myApplication_myMethod() {
  // ...
}
```

하지만 접두사만 다를 뿐, 전역 변수이기 때문에 충돌 위험은 여전히 존재...

##  객체 리터럴 표기법 패턴
자바스크립트의 **객체 리터럴 표기법(Object Literal Notation)** 을 활용해, 구조화된 네임스페이스를 구성하는 패턴이다.
```js
const myApplication = {
  getInfo() {
    // ...
  },
  models: {},
  views: {
    pages: {},
  },
  collections: {},
};

// 네임스페이스에 속성을 직접 추가하는 방식
myApplication.utils = {
  export() {},
};
```

전역 네임스페이스를 오염시키지 않으면서, 코드를 논리적으로 구성할 수 있다.
중첩 네임스페이스를 구성할 때 사용하기 좋은 방식이다.
키-값 구조를 갖기에 가독성이 좋으며, 덕분에 서로 다른 로직을 쉽게 분리할 수 있고 코드 확장 기반을 제공한다.

객체가 존재하지 않는데 접근하거나 재정의하면 의도치 않게 덮어쓸 수 있으므로, 다음과 같이 존재 여부를 확인하고 초기화하는 방식이 권장된다.
```js
const myApplication = myApplication || {};

if(!myApplication) { myApplication = {} };

window.myApplication || ( window.myApplication = {} );
```

## 중첩 네임스페이스 패턴 (Nested Namespace Pattern)
객체 리터럴을 확장한 형태로, **객체 안에 또 다른 객체를 중첩**시켜 계층적으로 구성하는 패턴이다. 
이를 통해 복잡한 구조를 가지는 네임스페이스를 만들 수 있다.
```js
const myApp = myApp || {};
myApp.routers = myApp.routers || {};
myApp.model = myApp.model || {};
myApp.model.special = myApp.model.special || {};

myApp["routers"] = myApp["routers"] || {};
myApp["models"] = myApp["models"] || {};
myApp["controllers"] = myApp["controllers"] || {};
```
**중첩 구조**로 논리적으로 코드를 잘 분리할 수 있으며, 기존 객체 리터럴 표기법 패턴에 비해 전역 네임스페이스 충돌을 효과적으로 방지할 수 있다.

하지만 계층이 깊어질수록 접근 코드가 길어지는 단점이 있다.
👉 근데 단일 쓰는거랑 성능 차이는 크지 않다고 한다.

## 즉시 실행 함수 표현식 패턴
즉시 실행 함수 내부의 변수와 함수를 은닉하여 외부에서 접근할 수 없게 한다.
이를 이용해서 전역 네임스페이스 오염을 방지한다.

아래 예시처럼 네임스페이스에 메서드와 속성을 할당하여 사용한다.
```js
const namespace = namespace || {};

((o) => {
  o.foo = "foo";
  o.bar = () => "bar";
})(namespace);

console.log(namespace); // { foo: "foo", bar: [Function] }
```


**비공개, 공개 접근 권한을 부여하는 예제**
```js
((namespace, undefined) => {
  const foo = "foo";
  const bar = "bar";

  // 공개 메서드와 속성
  namespace.foobar = "foobar";
  namespace.sayHello = () => {
    speak("hello world");
  };

  // 비공개 메서드
  function speak(msg) {
    console.log(`You said: ${msg}`);
  }
})(window.namespace = window.namespace || {});
```

#### ⁉️ 왜 undefined를 2번째 인자로 전달하나!?
👉 **안전하게 undefined 값을 보장하기 위해**

자바스크립트 ES5 이전에는 **undefined라는 키워드가 재정의 가능**했다.
과거에는 undefined가 키워드가 아닌 전역 변수였다!!!

```js
// 예전 환경에서는 이런 게 가능했음
undefined = "not really undefined";

(function (namespace) {
  console.log(undefined); // 변질됨, 실제 undefined가 아님
})(window.myNamespace = {});
```

따라서 즉시 실행 함수 호출 시 두번째 인자를 임의로 전달하지 않아 함수 내부에서 진짜 undefined 값을 받을 . 수 있도록 했다.
이렇게 하면 **외부에서 undefined가 오염됐더라도 함수 내부는 안전하게 보호**된다.

## 네임스페이스 주입 패턴
this를 네임스페이스의 프록시로 활용하여, this 키워드로 네임스페이스에 메서드와 속성을 주입할 수 있게 하는 패턴이다.
call이나 apply를 이용하여 this를 명시적으로 지정해준다.

```js
const myApp = myApp || {};
myApp.utils = {};

(function () {
  let val = 5;

  this.getValue = () => val;
  this.setValue = (newVal) => {
    val = newVal;
  };
}).apply(myApp.utils);
```

근데 이거보다 다른 방법이 더 쉽고 효율적일 수 있다..!

## 고급 네임스페이스 패턴
### 중첩 네임스페이스 자동화 패턴
중첩 네임스페이스가 깊어질 수록 하위 객체를 매번 정의하기 번거로워 진다.
이런 문제를 해결하기 위해 자동화 패턴을 이용한다.

아래 처럼 문자열 인자를 받아서 중첩 네임스페이스를 생성한다.

```js
function extend(ns, ns_string) {
  const parts = ns_string.split(".");
  let parent = ns;

  for (let i = 0; i < parts.length; i++) {
    if (typeof parent[parts[i]] === "undefined") {
      parent[parts[i]] = {};
    }
    parent = parent[parts[i]];
  }

  return parent;
}

// 사용 예
const myApp = {};
const mod = extend(myApp, "modules.module2");
console.log(mod === myApp.modules.module2); // true
```

### 의존성 선언 패턴
중첩된 네임스페이스를 반복해서 직접 접근하기보다, **중간 참조 변수**를 이용하여 접근하는 패턴이다.
이와 같은 방법으로 중첩 속성 접근 시 성능이 개선된다.
```js
// 중첩된 네임스페이스에 접근하는 일반적인 방법
myApp.utilities.math.fibonacci(25);
myApp.utilities.math.sin(56);
myApp.utilities.drawing.plot(98, 50, 60);

// 로컬 변수에 참조를 저장해 사용
const utils = myApp.utilities;
const maths = utils.math;
const drawing = utils.drawing;

maths.fibonacci(25);
maths.sin(56);
drawing.plot(98, 50, 60);
```

### 심층 객체 확장 패턴
기존 네임스페이스에 새로운 객체 속성을 재귀적으로 병합하는 패턴이다.
```js
function extendObjects(destinationObject, sourceObject) {
  for (const property in sourceObject) {
    if (
      typeof sourceObject[property] === "object" &&
      !Array.isArray(sourceObject[property])
    ) {
      destinationObject[property] =
        destinationObject[property] || {};
      extendObjects(destinationObject[property], sourceObject[property]);
    } else {
      destinationObject[property] = sourceObject[property];
    }
  }
  return destinationObject;
}


// 사용 예
const myNamespace = myNamespace || {};

extendObjects(myNamespace, {
  utils: {}
});
console.log("test 1", myNamespace);

extendObjects(myNamespace, {
  hello: {
    world: {
      wave: {
        test() {
          // ...
        }
      }
    }
  }
});

// 확장한 속성에 추가적으로 할당 가능
myNamespace.hello.test1 = "this is a test";
myNamespace.hello.world.test2 = "this is another test";

console.log("test 2", myNamespace);
```


👉 요즘에는 import/export를 통한 모듈 방식을 사용할 수 있어서, 방대한 크기의 네임스페이스를 관리할 일이 많지는 않은 것 같다.