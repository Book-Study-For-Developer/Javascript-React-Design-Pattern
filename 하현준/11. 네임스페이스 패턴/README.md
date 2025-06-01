## 네임스페이스의 기초

대규모 스크립트나 애플리케이션에서는 네임스페이스를 사용하지 않으면, 페이지 내 다른 스크립트와 변수 또는 메서드 이름이 **충돌하여 실행되지 않을사수 있다.**

네임스페이스의 종류

- 단일 전역 변수
- 접두사 네임스페이스
- 객체 리터럴 표기법
- 중첩 네임스페이스
- 즉시 실행 표현식
- 네임스페이스 주입

## 단일 전역 변수 패턴

하나의 전역 변수를 선언하여 주요 참조 객체로 활용하는 패턴이다.

```tsx
const myUniqueApplication = (() => {
	function myMethod() {
		return;
	}

	return {
		myMethod;
	}
})();
```

즉시 실행 함수를 사용하여 고유 네임스페이스를 생성한다.

- 그러나 myUniqueApplication을 **이미 사용하고 있다면 충돌**이 발생할 수 있다.

## 접두사 네임스페이스 패턴

단일 전역 변수를 할 경우 겹치는 문제가 발생하니 prefix를 붙이는 것이 패턴이다.

```tsx
const myApplication_propertyA = {};
const myApplication_propertyB = {};
```

myApplication이라는 prefix를 붙여서 겹치는 가능성을 줄이는 패턴이다.

🤔 이것도 결국 도찐개찐 아닌가 싶다.. ㅋㅋ, 전역 변수 사용 멈춰~

## 객체 리터럴 표기법 패턴

객체 리터럴 문법을 사용하여 키 자체를 네임스페이스가 될 수 있도록 하는 패턴이다.

```tsx
const myApplication = {
	getInfo() {
		//...
	}

	models: {},
	views: {
		pages: {},
	}
}
```

우리가 흔히 객체를 정의해서 객체 내부에서 키와 value들을 관리하는 패턴으로 보면 될 듯하다.

- 이렇게 객체로 관리하게 되면 중첩되어 사용할 수 있으니 충돌할 일이 줄어든다고 한다..
- 그럼에도 네임스페이스가 존재할 수 있으니 이를 확인하고 초기화하는 패턴이 몇가지 있다.

```tsx
// ❗️BAD
const myApplication = {};

// 1
let myApplication = myApplication || {};
// 2
if (!myApplication) {
  myApplication = {};
}
// 3
window.myApplication || (window.myApplication = {});
// 4
let myApplication = myApplication === undefined ? {} : myApplication;
```

🤔 이건 타입스크립트로 썼다면 아마 다들 해보셨을 패턴 같다. 저는 마지막을 선호합니다.

## 중첩 네임스페이스 패턴

하위의 중첩된 네임스페이스까지 정확히 일치하긴 어렵기 때문에 객체 리터럴을 발전시킨 패턴이다.

```tsx
YAHOO.util.Dom.getElementByClassName("test");
```

어찌보면 documentAPI들도 거대한 네임스페이스로 보면 될 듯하다.

document 하위로 중첩된 네임스페이스로 메소드를 실행하니까?

## 즉시 실행 함수 표현식 패턴

즉시 실행 함수를 통해 내부에 정의된 변수나 함수를 하나의 네임스페이스로 묶는 패턴이다.

```tsx
const namespace = namespace || {};

((o) => {
  o.foo = "foo";
  o.bar = () => "bar";
})(namespace);
```

기본적인 네임스페이스 + IIFE를 사용하는 패턴이다.

```tsx
((namespace, undefined) => {
  const foo = "foo";
  const bar = "bar";
  namespace.foobar = "foobar";
  namespace.sayHello = () => {
    speak("Hello World");
  };
  namespace.sayGoodbye = () => {
    speak("Goodbye peeps");
  };
  function speak(msg) {
    console.log(`You said: ${msg}`);
  }
})((window.namespace2 = window.namespace2 || {}));
```

🤔 여기서 두번째 인자에 `undefined`를 넣는 이유는 **ES5 이전에는 undefined가 변경될 수 있었기 때문에 강제로 넣는게 필요**하다고 이야기한다.

그 이유?: ES5이전에는 undefined가 예약어였지만, 상수가 아니였다고 한다.

즉 이런 괴상한 짓이 가능한것

```tsx
undefined = "oops!";
```

그러면 undefined의 값이 변경되는 것이다.. 😱 (undefined가 undefined가 아닌 것)

## 네임 스페이스 주입 패턴

함수 내에서 this를 네임스페이스의 프록시로 활용하여 특정 네임스페이스에 메서드와 속성을 ‘주입’ 하는 패턴이다.

```tsx
const myApp = {};
myApp.utils = {};
(function () {
  let val = 5;
  this.getValue = () => val;
  this.setValue = (newVal) => {
    val = newVal;
  };
  this.tools = {};
}).apply(myApp.utils);

// tools 네임스페이스에 새로운 의존성을 주입하는 것
(function () {
  this.diagnose = () => "Diagnosis";
}).apply(myApp.utils.tools);
```

장점: 기능적인 동작을 쉽게 적용할 수 있다는 점

단점: 같은 목적의 더 쉽고 간편한 패턴이 있다.

## 고급 네임스페이스 패턴

### 중첩 네임스페이스 자동화 패턴

중첩 네임스페이스의 depth가 깊어지게 될 수 있다.
ex)`application.utilities.drawing.canvas.2d`

중첩된 객체의 깊이가 깊어질 수록 매우 귀찮아질 것이다. 이를 자동화 할 수 있는 함수를 만들면 편해진다.

```tsx
function extend(ns, nsString) {
  const parts = nsString.split(".");
  let parent = ns;
  let pl;

  pl = parts.length;

  for (let i = 0; i < pl; i++) {
    // .을 기준으로 쪼갰으니 각각 하나씩 돌면서 undefined면 추가하면서 중첩된 객체를 만든다.
    if (typeof parent[parts[i]] === "undefined") {
      parent[parts[i]] = {};
    }

    parent = parent[parts[i]];
  }

  return parent;
}

// 사용법
const mod = extend(myApp, "modules.module2");
```

### 의존성 선언 패턴

객체에 대한 로컬 참조가 전체적인 조회 시간을 단축한다.

```tsx
myApp.utilites.math.fibonacci(25);
myApp.utilites.math.sin(56);
myApp.utilites.drawing.plot(98, 50, 60);

// 로컬 변수에 캐싱한 참조를 사용
const utils = myApp.utilities;

const math = uiitls.math;
const drawing = utils.drawing;
```

🤔 그냥 가독성을 좋게 하기위해서 하는 편인데, 이게 성능에 영향을 주는지는 몰랐다. 또 이걸 의존성 선언 패턴이라고 하는것도 처음알았다.

### 심층 객체 확장 패턴

객체 리터럴 표기법으로 선언된 네임스페이스를 다른 객체와 확장하는 패턴이다. (jQuery의 extends 문법과 동일한 기능이라고 한다.)

```tsx
function extendObjects(destinationObject, sourceObject) {
  for (const property in sourceObject) {
    if (
      sourceObject[property] &&
      typeof sourceObject[property] === "object" &&
      !Array.isArray(sourceObject[property])
    ) {
      destinationObject[property] = destinationObject[property] || {};
      extendObjects(destinationObject[property], sourceObject[property]);
    } else {
      destinationObject[property] = sourceObject[property];
    }
  }
  return destinationObject;
}
```

⇒ lodash/[es-toolkit](https://github.com/toss/es-toolkit/blob/main/docs/ko/reference/compat/object/extend.md)의 extends하고 같은 기능이니 그것을 활용하는 것으로..

🤔 현재는 assignIn으로 네이밍이 변경된 것으로 보이네요. (https://lodash.com/docs/4.17.15#assign)
