# 네임스페이스 패턴 (Namespace Pattern)

## 네임스페이스란?

JavaScript는 기본적으로 전역 범위(global scope)에 변수를 선언하게 되며, 여러 스크립트가 혼합되면 **이름 충돌(name collision)** 문제가 발생할 수 있습니다.
이를 방지하기 위해 **네임스페이스(namespace)** 를 정의하여, 변수나 함수가 소속될 "공간"을 만들어 전역 범위를 더럽히지 않고 코드를 구조화합니다.

---

## 네임스페이스 패턴의 종류

1. 단일 전역 변수 패턴
2. 접두사 네임스페이스 패턴
3. 객체 리터럴 표기법 패턴
4. 중첩 네임스페이스 패턴
5. 즉시 실행 함수 표현식 (IIFE)
6. 네임스페이스 주입 패턴
7. 고급 패턴: 자동화 / 의존성 선언 / 객체 확장

---

## 11.2 단일 전역 변수 패턴

```tsx
const MyApp = (() => {
  const privateVar = 'private';

  return {
    publicMethod: () => console.log(privateVar),
  };
})();

MyApp.publicMethod(); // "private"
```

### 💬 부연설명

- IIFE(즉시 실행 함수 표현식)를 활용해 네임스페이스를 만들고, 그 안에 필요한 기능을 반환하여 `MyApp`에 할당합니다.
- 내부 변수 `privateVar`는 외부에서 접근할 수 없는 캡슐화(encapsulation)를 갖습니다.
- **작은 유틸리티 모듈**, **위젯 단위의 독립적 기능**을 구성할 때 유용합니다.
- 단점은 동일한 이름의 전역 네임스페이스가 존재하면 덮어쓰기 위험이 있다는 점입니다.

---

## 11.3 접두사 네임스페이스 패턴

```tsx
const MyApp_config = {};
const MyApp_utils = {};
function MyApp_log(message: string) {
  console.log(message);
}
```

### 💬 부연설명

- 전역 이름의 충돌을 피하기 위해 일관된 접두사(`MyApp_`)를 붙입니다.
- 구조화는 되어 있지 않지만, 전역 네임스페이스의 범람을 막는 데 최소한의 도움을 줍니다.
- **아주 단순한 프로젝트나 레거시 스크립트에서 빠르게 적용** 가능하지만, 유지보수와 가독성 측면에서는 권장되지 않습니다.
- 정리된 객체 형태가 아니므로 자동 완성 기능의 도움을 받기 어렵습니다.

---

## 11.4 객체 리터럴 표기법 패턴

```tsx
const MyApp = {
  config: {
    apiUrl: '/api',
  },
  utils: {
    log(message: string) {
      console.log(message);
    },
  },
};

MyApp.utils.log('Hello');
```

### 💬 부연설명

- 객체 리터럴을 사용해 명확한 네임스페이스 구조를 형성합니다.
- 코드 가독성이 뛰어나며, `MyApp.utils.log()`처럼 체계적으로 접근 가능합니다.
- 가장 일반적이며, **팀 협업 시 코드 컨벤션 통일 및 오토 컴플리션에 매우 유리**합니다.
- 객체 리터럴로 선언되기 때문에 파일 간 결합도가 낮고, 테스트와 리팩토링도 용이합니다.

---

## 11.5 중첩 네임스페이스 패턴

```tsx
const MyApp = MyApp || {};
MyApp.modules = MyApp.modules || {};
MyApp.modules.router = MyApp.modules.router || {};

MyApp.modules.router.navigate = (url: string) => {
  console.log(`Navigating to ${url}`);
};
```

### 💬 부연설명

- 중첩 구조를 통해 도메인, 책임, 기능별로 구분된 모듈화가 가능합니다.
- `MyApp.modules.router`와 같이 **세부 기능을 분리하여 유지보수성을 높임**.
- 중복 선언을 방지하기 위해 `|| {}`를 사용하는 것이 핵심 포인트입니다.
- 대형 프로젝트나 라이브러리 설계에서 많이 활용됩니다.

---

## 11.6 즉시 실행 함수 표현식 (IIFE)

```tsx
const Namespace = Namespace || {};

((ns) => {
  const privateData = 'secret';
  ns.utils = {
    getSecret: () => privateData,
  };
})(Namespace);

console.log(Namespace.utils.getSecret()); // "secret"
```

### 💬 부연설명

- IIFE는 외부로부터 내부 변수를 은닉할 수 있어 **보안성과 모듈성을 동시에 확보**할 수 있습니다.
- 외부 네임스페이스 객체(`Namespace`)를 인자로 전달하여 그 안에 기능을 추가하는 방식이기 때문에 **유연한 확장이 가능**합니다.
- 외부에서 `Namespace.utils.getSecret()`처럼 접근할 수 있지만 내부 `privateData`는 완벽히 은닉됩니다.

---

## 11.7 네임스페이스 주입 패턴

```tsx
const MyApp = MyApp || {};
MyApp.utils = {};

(function () {
  let internalValue = 0;

  this.set = (val: number) => {
    internalValue = val;
  };
  this.get = () => internalValue;
}).apply(MyApp.utils);

MyApp.utils.set(10);
console.log(MyApp.utils.get()); // 10
```

### 💬 부연설명

- `this`를 네임스페이스로 간주하여 메서드를 동적으로 "주입"합니다.
- 코드가 실행될 당시의 컨텍스트에 따라 유연하게 동작하기 때문에, **라이브러리 또는 플러그인 확장 시 유용**합니다.
- 단점은 컨텍스트(`this`)를 명확하게 인식하지 못하면 디버깅이 어려울 수 있습니다.

---

## 11.8 고급 네임스페이스 패턴

### 11.8.1 중첩 네임스페이스 자동화

```tsx
function extendNamespace(root: any, namespaceStr: string) {
  const parts = namespaceStr.split('.');
  let current = root;

  for (const part of parts) {
    current[part] = current[part] || {};
    current = current[part];
  }

  return current;
}

const MyApp = {};
extendNamespace(MyApp, 'utils.drawing.canvas');

MyApp.utils.drawing.canvas.paint = () => console.log('Paint!');
```

### 💬 부연설명

- 중첩 객체를 직접 하나씩 생성하지 않고, 문자열 기반으로 **자동 생성하는 유틸리티 함수**입니다.
- 대규모 프로젝트에서 **모듈 생성 자동화 및 초기화 시 유용**하며, 코드 반복을 줄일 수 있습니다.

---

### 11.8.2 의존성 선언 (로컬 참조)

```tsx
// 성능 개선을 위한 로컬 참조 방식
const utils = myApp.utils;
const math = utils.math;

math.calculate();
```

### 💬 부연설명

- 중첩된 네임스페이스에 자주 접근하는 경우, **지역 변수로 캐싱**하면 속도와 가독성을 동시에 향상시킬 수 있습니다.
- 특히 **루프 안에서 객체에 반복 접근 시 성능 최적화에 효과**적입니다.
- 단점: 네임스페이스 구조가 바뀌면 참조도 함께 수정해야 하므로 의존성 관리가 필요합니다.

---

### 11.8.3 심층 객체 확장 (Deep Extend)

```tsx
function deepExtend(target: any, source: any): any {
  for (const key in source) {
    if (
      source[key] &&
      typeof source[key] === 'object' &&
      !Array.isArray(source[key])
    ) {
      target[key] = target[key] || {};
      deepExtend(target[key], source[key]);
    } else {
      target[key] = source[key];
    }
  }

  return target;
}

const base = { config: { debug: false } };
const override = { config: { verbose: true } };

const merged = deepExtend(base, override);
```

### 💬 부연설명

- 두 개 이상의 네임스페이스나 설정 객체를 병합할 때 사용되는 재귀적 함수입니다.
- **플러그인 구성, 환경 설정, 다국어 번역 파일 통합 등에서 매우 자주 사용**됩니다.
- 기본값을 유지하면서 새로운 값을 덮어쓰는 방식으로 유연한 확장을 가능하게 합니다.

---

## 11.9 권장하는 패턴

> ✅ 대규모 프로젝트에서는 객체 리터럴 + 중첩 네임스페이스 + 자동화 조합이 가장 안정적입니다.
> ✅ **소규모 프로젝트나 위젯 개발**에는 `IIFE + 단일 전역 변수` 패턴이 가볍고 효과적입니다.
> 🔄 확장이 자주 일어나는 시스템에서는 `네임스페이스 주입 + 심층 확장` 패턴이 유용합니다.
