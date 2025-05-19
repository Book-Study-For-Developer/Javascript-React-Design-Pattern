## 07. 자바스크립트 디자인 패턴

> ##### [COLUMN] 패턴 선택하기
>
> 최고의 패턴이 무엇인지에 대한 정답은 없다. 각 코드와 애플리케이션마다 요구하는 부분이 다를 수 있기 때문이다.
> 대신 패턴이 실질적인 구현에 도움이 되는지를 고려해야 한다.
> 예를들어, 어떤 프로젝트는 관찰자 패턴의 디커플링 (애플리케이션의 부분별 의존성을 줄이는 것)이 적합할 수 있지만, 어떤 프로젝트는 소규모라 디커플링이 중요하지 않을 수 있습니다.
> 디자인 패턴에 대해 정확히 알고 사용할 때를 잘 파악하고 있으면 올바른 디자인 패턴을 아키텍처에 적용하기 수월해 질 것입니다.

### 07-1. 생성 패턴

> 생성 패턴은 객체를 생성하는 방법을 다룹니다.

- 생성자 패턴
- 모듈 패턴
- 노출 모듈 패턴
- 싱글톤 패턴
- 프로토타입 패턴
- 팩토리 패턴

### 07-2. 생성자 패턴

> 생성자(Constructor)는 객체가 새로 만들어진 뒤 초기화하는 데에 사용되는 필요한 메서드 입니다.

ES2015 이후로는 생성자를 가진 클래스를 만들 수 있게 되었습니다.

- 자바스크립트에서는 거의 모든 것이 객체입니다. 그리고 class는 자바스크립트가 가진 프로토타입의 상속을 이용한 문법적 설탕(Syntactic Sugar) 이기도 합니다.
  - 문법적 설탕이란 복잡한 개념을 더 간격하고 이해하기 쉽게 표현하는 방법을 뜻합니다.

#### 07-2.1. 객체 생성

자바스크립트에는 새로운 객체를 생성할때 사용하는 방식이 세가지 존재합니다.

1. 리터럴 표기법을 사용하여 빈 객체 생성

```js
const newBoky = {};
```

2. Object.create() 메서드를 사용하여 빈 객체 생성

```js
const newBoky = Object.create(Object.prototype);
```

3. new 키워드를 사용하여 빈 객체 생성

```js
const newBoky = new Object();
```

---

다음과 같은 방법으로 만들어진 객체에 키와 값을 할당할 수 있습니다.

##### ECMAScript 3 호환 방식

1. 도트 Dot(.) 문법

```js
// 속성 할당
newBoky.someKey = "Hello Javascript!";

// 속성 가져오기
var key = newBoky.someKey;
```

2. 대괄호 문법

```js
// 속성 할당
newBoky["someKey"] = "Hello amigo";

// 속성 가져오기
var key = newBoky["someKey"];
```

##### ECMAScript 5 만 호환되는 방식

3. Object.defineProperty

```js
// 속성 할당하기
Object.defineProperty(newObject, "someKey", {
  value: "for more control of the property's behavior"
  writable: true,
  enumerable: true,
  configurable: true
})
```

4. 앞선 방법이 조금 불편하다면 이렇게도 가능

```js
var defineProp = function (obj, key, value) {
  config.value = value;
  Object.defineProperty(obj, key, config);
};

// 사용 방법
// 빈 객체 "person" 생성
var person = Object.create(null);

// 속성 할당
defineProp(person, "car", "Delorean");
defineProp(person, "dateOfBirth", "1981");
defineProp(person, "hasBeard", false);
```

5. Object.defineProperties

```js
// 속성 할당
Object.defineProperties(newObject, {
  someKey: {
    value: "Hello",
    writable: true,
  },
  anotherKey: {
    value: "bokeeeey",
    writable: false,
  },
});
```

---

아래 방법과 같이 객체를 상속할 수도 있습니다.
(ES2015+ 문법 사용)

```js
// 'person' 객체를 상속하는 'driver' 객체를 생성합니다.
const driver = Object.create(person);

// 속성을 할당합니다.
defineProp(driver, "topSpeed", "100mph");

// 상속받은 속성 값을 가져옵니다.
console.log(driver.dateOfBirth);

// 할당한 속성 값을 가져옵니다.
console.log(driver.topSpeed);
```

#### 07-2.2. 생성자의 기본 특징

ES2015에 도입된 JS의 class는 객체 템플릿을 정의하고 캡슐화 및 상속을 구현할 수 있게 했습니다.

- 클래스는 새 객체를 초기화하는 `constructor()`라는 이름의 메서드를 가지고 있어야 합니다.
- 또한 `new` 키워드는 생성자를 호출할 수 있으며, 생성자 내부에서 사용된 `this` 키워드는 새로 생성된 해당 객체를 가리킵니다.

```js
class Car {
  constructor(model, year, miles) {
    this.model = model;
    this.year = year;
    this.miles = miles;
  }

  toString() {
    return `${this.model} has done ${this.miles} miles`;
  }
}

// 새로운 car 인스턴스 생성
let civic = new Car("Honda Civic", 2009, 20000);
let mondeo = new Car("Ford Mondeo", 2010, 5000);

// 브라우저 콘솔에서 확인
console.log(civic.toString());
console.log(mondeo.toString());
```

> 해당 class constructor 생성자로 객체를 생성하는 방식에는 몇가지 문제가 있습니다.
> 하나는 상속이 어려워진다는 점이고, 다른 하나는 Car 생성자로 객체를 생성할 때마다 toString()과 같은 함수를 새로 정의한다는 점입니다.
> **Car 인스턴스는 모두 동일한 함수를 공유해야 하므로 이 방법은 효과적이지 않습니다.**

#### 07-2.3. 프로토타입을 가진 생성자

자바스크립트의 프로토타입 객체는 함수나 클래스 등 특정 객체의 모든 인스턴스 내에 공통 메서드를 쉽게 정의할 수 있게 합니다.

```js
class Car {
  constructor(model, year, miles) {
    this.model = model;
    this.year = year;
    this.miles = miles;
  }
}

// 프로토타입 객체의 재정의를 피하기 위해 Object.prototype 대신
// Object.prototype.newMethod 형태를 사용하고 있음에 유의하세요.
// 기존에 이미 정의된 프로토타입 객체를 유지하기 위해서 입니다.

Car.prototype.toString = function () {
  return `${this.model} has done ${this.miles} miles`;
};
```

오 이건 처음보는 초식이네요!
항상 전자의 방법으로 메서드를 정의해주지 않았나 생각합니다.

다시 생각해보니 평소 스켈레톤은 이런 방식으로 썻던거같네요

```tsx
export default function SomeComponent() {
  return (
    //... 대충 컴포넌트 구성
  )
}

function SomeComponentSkeleton() {
  return (
    //.... 대충 스켈레톤 구성
  )
}

SomeComponent.Skeleton = SomeComponentSkeleton
```

### 07-3. 모듈 패턴

> 모듈은 애플리케이션 아키텍처의 핵심 구성 요소이며 구성하는 코드 단위를 체계적으로 분리 및 관리하는 데 호과적으로 활용됩니다.

초기 자바스크립트에서는 다음과 같은 방법으로 모듈을 구현했습니다.

- 객체 리터럴 표기법
- 모듈 패턴: 빌드타임에서 확인 가능 => 컴파일시 정적 분석 가능 => 트리쉐이킹이 가능해짐
- AMD 모듈
- CommonJS 모듈: 런타임중에 실행된다 => 실행 시점(런타임 레벨)에서 판단 가능

모듈에 대한 분석은 뒷장에서 심층으로 다루니까 가볍게 넘어가자

#### 본 장에서는 모듈 패턴의 개념과 역사에 초점을 맞추어 살펴보자!

#### 07-3.1. 객체 리터럴

> 객체 리터럴 표기법은 객체는 중괄호( { } ) 안에서 key와 value를 쉼표(,)로 구분하여 객체를 정의하는 벙법

객체 리터럴은 선언 시 new 연산자를 필요로 하지 않으며, 중괄호를 통해 객체 블록의 시작을 명시합니다.

```js
const 객체리터럴방식으로만든객체 = {
  variableKey: variableValue
  functionKey() {
    //...... 월요일 좋아
  }
}
```

```js
const myModule = {
  myProperty: 'someValue'
// 객체 리터럴은 속성으로 값과 메서드를 모두 가질 수 있습니다.
// 예를 들어 객체 안에 객체를 다시 생성할 수도 있습니다.
myConfig: {
  useCaching: true,
   language: 'en'
  },
  // 간단한 메서드 예시
saySomething() {
  console.log('동해물과 백두산이');
  },
// 현재 객체의 속성 값을 사용하는 메서드
reportMyConfig() {
  console.log(`마르고 닳도록: ${this.myConfig.useCaching ? "하느님이 보우하사": "우리 나라 만세"}`)
  }

  // 현재 객체의 속성 값을 덮어씌우는(override) 메서드
  updateMyConfig(newConfig) {
    if (typeof newConfig === 'object') {
      this.myConfig = newConfig;
      console.log(this.myConfig.language);
    }
  }
}
```

객체 리터럴을 사용하면 코드를 캡슐화하여 깔끔하고 체계적을 정리 가능함.

### 07-3.2. 모듈 패턴

> 모듈 패턴은 과거 class의 캡슐화를 위해 처음 고안된 패턴입니다.

- 과거에는 앱 개발에서 재사용 가능한 로직은 분할하고 관리하기 위해 개별 스크립트에 의존하였고, 그 결과 하나의 HTML 파일에서 10~20개의 스크립트를 각각 수동으로 가져와야 하는 경우가 빈번했음.
- 객체를 활용하는 모듈 패턴은 그저 `공개` 및 `비공개` 메서드를 가진 로직을 캡슐화 하는 방법 중 하나였음.
- 시간이 지남에 따라 여러 커스텀 모듈 시스템이 등장하면서 객체, 함수, 클래스, 변수 등을 구성하여 다른 파일에 쉽게 내보내거나 가져올 수 있게 되었습니다.
  - 이를 통해 서로 가른 모듈 간 클래스 또는 함수명 충돌을 방지할 수 있게 되었음.

#### 비공개

> 모듈 패턴은 클로저(closure)를 활영하여 `private` 상태와 구성을 캡슐화합니다.

- 공개 및 비공개 메서드와 변수를 묶어 전역 스코프로의 유출을 방지하고, 다른 개발 인터페이스와의 충돌을 예방합니다.
- 모듈 패턴을 사용한다면 공개 API만을 노출하고 나머지는 클로저 내부에 비공개로 유지할 수 있습니다.
- 다른 앱에서 사용해야 하는 부분만 노출하면서 핵심 작업은 보호하는 깔끔하고 체계적인 구조를 구축 가능.
- 지금은 class 내부에서 `#` 접근 제한자를 사용하여 비공개를 적용할 수 있지만, 해당 개념이 없었던 과거에는 함수 스코프를 이용하여 비공개 개념을 구현했습니다.

```js
let counter = 0;

const testModule = {
  incrementCounter() {
    return counter++;
  },
  resetCounter() {
    console. log(`counter value prior to reset: ${counter}`);
    counter = 0;
  }
},

// 변수명을 정하지 않고 디폴트 default로서 내보내는 방법입니다.
export default testModule;

// 사용 방법:
// 모듈을 가져올 경로를 설정합니다.
import testModule from './testModule';

// 카운터 증가
testModule.incrementCounter();

// 카운터 값을 확인하고 리셋
// 출력: counter value prior to reset: 1
testModule.resetCounter();
```

> 가물가물해서 다시 정리해보는 클로저
>
> - 클로저는 비공개 변수를 가질 수 있는 환경에 있는 함수를 뜻합니다.
> - 부모 함수 내부에 선언된 중첩 함수가 있는 경우, 부모 함수가 실행을 완료한 후에도 함수가 부모 범위(=== scope) 의 변수를 "기억"할 수 있도록 합니다. (실행스택에서는 없어지지만 내부 함수가 선언되었을 때 lexical scoping에 의해 정해진 scope chain은 부모함수와 전역 변수 객체이기 때문에 해당 scope의 변수를 이용할 수 있게 됨)

##### 다른 예제로 장바구니를 구현해보겠습니다.

모듈 자체는 basketModule이라는 전역 변수 안에 독립적으로 존재하고 있습니다.
basket 배열은 모듈 내부에서 비공개로 유지되기에 다른 파일에서 읽어 들일 수 없습니다.
즉, 모듈의 클로저 내부에서 보호되고 있기에 addItem() 이나 getItem()처럼 같은 스코프 안에 존재하는 메서드만이 접근할 수 있습니다.

```js
// 비공개 변수 및 함수
const basket = [];

const doSomethingPrivate = () => {
  //...
};

const doSomethingElsePrivate = () => {
  //...
};

// 다른 파일에 공개할 객체 생성
const basketModule = {
  // Add items to our basket
  addItem(values) {
    basket.push(values);
  },

  // basket의 길이 가져오기
  getItemCount() {
    return basket.length;
  },

  // 비공개 함수를 공개 함수로 감싸 다른 이름으로 사용하기
  doSomething() {
    doSomethingPrivate();
  },
  // basket에 담긴 아이템의 합계 가져오기
  // reduce() 메서드를 사용하면 배열의 아이템을 하나의 값으로 줄일 수 있습니다.
  getTotal() {
    return basket.reduce((currentSum, item) => item.price + currentSum, 0);
  },
};

export default basketModule;
```

모듈을 살펴보면 **객체를 내보낸다는 것을 눈치채셨을 겁니다.** 내보내는 객체는 basketModule경로에 자동으로 할당되어 다음과 같이 사용할 수 있습니다.

```js
// 경로로부터 모듈을 가져옵니다.
import basketModule from "./basketModule";

// basketModule 에서 사용할 수 있는 공개 API 객체를 가져옵니다.
basketModule.addItem({
  item: "bread",
  price: 0.5,
});

basketModule.addItem({
  item: "butter",
  price: 0.3,
});

// 출력: 2
console.log(basketModule.getItemCount());

// 출력: 0.8
console.log(basketModule.getTotal());

// 그런데 다음은 동작하지 않습니다:

// 출력: undefined
// 왜냐하면 basket 변수는 공개 API에 포함되지 않았기 때문입니다.
console.log(basketModule.basket);

// basket은 baketModule의 클로저 내부에서 보호되어 접근할 수 없기에
// 이 코드도 동작하지 않습니다.
console.log(basket);
```

해당 메서드들은 basketModule로 네임스페이스가 지정됩니다. 부모함수 내부에 중첩 함수가 포함되어 있는 구조는 다음과 같은 이점을 제공합니다.

- 비공개 자유성: 모듈 내부에서만 사용 가능한 비공개 함수를 자유롭게 만들 수 있습니다. 다른 파일에서 접근 할 수 없기에 완전한 비공개를 실현할 수 있습니다.
- 디버깅 용이성: 대개 함수는 선언되고 이름으로 정해지므로, 어떤 함수가 예외를 발생시켰는지 알아내려고 할 때 디버거에서 콜 스택(call stack) 을 찾기 쉬워집니다..?

###### 해석 추가

- 해당 예시의 핵심은 closure를 활용하여 `basket` 배열과 `doSumethingPrivate()`, `doSomethingElsePrivate()`함수를 모듈 외부에서 접근할 수 없는 비공개 패턴을 적용
- JS의 lexical scoping 특성을 활용하여 모듈 내에서 선언된 변수와 함수는 해당 모듈의 스코프 내에서만 접근할 수 있는 구조
- 내부에서 선언한 함수 이름으로 인해 익명함수를 사용한 방식보다 디버거에서 콜스택을 볼 때 함수 이름으로 추적 가능한 내용인듯 === 오류가 발생하면 내부 함수명이 명확하게 표시되어 쉽게 파악 가능

### 07-3.3. 모듈 패턴의 변형

#### 믹스인 가져오기 Mixin import 변형

- 믹스인으로 변형된 패턴은 유틸 함수나 외부 라이브러리 같은 전역 스코프에 있는 요소를 모듈 내부의 고차 함수에 인자로 전달 할 수 있게 합니다. === 전역 스코프 요소를 가져와 이름을 지정(alias)할 수 있습니다.

```js
// utils.js
export const min = (arr) => Math.min(...arr);

// privateMethods.js
import { min } from "./utils";

export const privateMethod = () => {
  console.log(min([10, 5, 100, 2, 1000]));
};

// myModule.js
import { privateMethod } from "./privateMethods";

const myModule = () => ({
  publicMethod() {
    privateMethod();
  },
});

export default myModule;

// main.js
import myModule from "./myModule";

const moduleInstance = myModule();
moduleInstance.publicMethod();
```

#### 내보내기 export 변형

- 이름을 따로 지정해주지 않고 전역 스코프로 변수를 내보내는 변형.

```js
const module = {
  // ... 대충 코드 내용
};

export default module;
```

#### 장단점 정리

장점

- 자바스크립트의 관점으로 볼 때 모듈 패턴이 조금 더 OOP 지식을 가진 초보 개발자가 이해하기 쉽다.
- Mixin import 를 예시로 보면 모듈 사이의 의존성을 관리하면서 전역 요소를 원하는 만큼 범위를 설정하여 코드의 유지보수를 용이하게 하며 독립적으로 구성할 수 있도록 해줍니다.
- 비공개 패턴 지원 가능 === 공개되면 안되는 코드 캡슐화 가능
- export를 사용하여 노출범위를 제한할 수 있으므로 불필요한 전역 스코프 오염을 방지할 수 있음
- 네임스페이스 오염 방지 === 같은 변수명 충돌 방지

단점

- 공개와 비공개 멤버를 서로 다르게 접근해야 한다. 공개 여부를 수정하고 싶으면 해당 파일로 이동하여 각각 수정을 해주어야만 함.
- 나중에 새로 추가한 메서드에서는 비공개 멤버에 접근할 수 없음.
- 테스트에서 비공개 멤버는 제외됨
- 디버깅이나 핫픽스 진행기 복잡도 상승 === 비공개 멤버의 의존성이 공개 메서드와 얽혀있으므로 유연 수정하기 어려워진다

모듈 패턴에 대해 더 알고싶다면 Ben Cherry의 글을 참조해보자
[자바스크립트 모듈 패턴에 대한 깊은 연구 - 번역](https://cyberx.tistory.com/104)

### 07-3.4. WeakMap을 사용하는 최신 모듈 패턴

> ES6에서 도입된 WeakMap 객체는 약한 참조를 가진 key - value의 쌍으로 이루어진 집합체 입니다.
> WeakMap 객체는 기본적으로 key가 약하게 유지되는 map으로 참조되지 않는 key는 가비지컬렉션의 대상이 됩니다.

> ##### WeakMap을 사용하여 비공개 멤버 흉내내기
>
> WeakMap을 사용하면 비공개 데이터를 객체에 연결할 수 있고, 다음과 같은 이점을 누릴 수 있습니다.
>
> - Map과 비교했을 때, WeakMap은 키로 사용되는 객체에 대한 강력한 참조를 보유하지 않으므로 객체와 그 메타데이터가 동일한 수명을 공유하고, 따라서 메모리 누수를 방지할 수 있습니다.
> - 클로저와 비교했을 때, 하나의 WeakMap을 생성자에서 생성한 모든 인스턴스에 재사용할 수 있으므로 메모리 효율성이 더 높고, 같은 클래스의 다른 인스턴스가 서로의 비공개 멤버를 읽을 수 있습니다.
>
> 레퍼런스
> [MDN - javascript WeakMap](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/WeakMap)

### 07-3.5. 최신 라이브러리와 모듈

React와 같은 자바스크립트 라이브러리를 만들 때 모듈 패턴을 사용할 수도 있습니다.

> TMI
> 최근 패키지 자동화 배포 공부중에 가볍게 해봤던 내용인거같네요
> https://www.npmjs.com/package/@sadang-turtleneck-new-ui/ui?activeTab=versions
> 사실 별거없는 이스터에그입니다. :)

### 07-4. 노출 모듈 패턴

- 가져온 모듈의 공개 변수나 메서드에 접근하기 위해 메인 객체의 이름을 반복해서 사용해야 한다는 점에 답답함을 느끼면서 탄생한 패턴. 사실 만든이는 객체 리터럴 표기법을 사용해 요스를 공개하는 것조차 마음에 들지 않았다고 한다...
- 그 결과 모든 함수와 변수를 비공개 스코프에 정의하고, 공개하고 싶은부분만 포인터를 통해 비공개 요소에 접근할 수 있게 해주는 익명 객체를 반환하는 패턴이 짜잔 탄생했습니다.

> ES2015+ 에서는 module scope 안에 정의된 함수와 변수는 비공개 처리됩니다.
> 그리고 export와 import를 통해 공개 여부를 결정합니다.

노출 모듈 패턴을 사용하면 더 구체적인 이름을 붙여 비공개 요소를 공개로 내보낼 수 있습니다.

```js
let privateCounter = 0;

const privateFunction = () => {
  privateCounter++;
};

const publicFunction = () => {
  publicIncrement();
};

const publicIncrement = () => {
  privateFunction();
};

const publicGetCount = () => privateCounter;

// 비공개 함수와 속성에 접근하는 공개 프록시
const myRevealingModule = {
  start: publicFunction,
  increment: publicIncrement,
  count: publicGetCount,
};

export default myRevealingModule;

// 사용처:
import myRevealingModule from "./myRevealingModule";

myRevealingModule.start();
```

#### 장단점

- 노출 모듈 패턴을 사용하면 코드의 일관성이 유지됩니다. 또한 모듈의 가장 아래에 위치한 공개 객체(공개 포인터를 의미하는듯)를 더 알아보기 쉽게 바꾸어 가독성을 향상시킵니다.
- 단점으로는 비공개 함수를 참조하는 공개 함수를 수정할 수 없다는 것입니다.
  - 비공개 함수가 비공개 구현을 참고하기 때문에 발생하며, 수정을 해도 함수가 변경될 뿐 참조된 구현이 변경이 되는것이 아니기 때문입니다.
  - 비공개 변수를 참조하는 공개 객체 멤버 또한 수정이 불가능합니다.
  - 따라서 기존 모듈 패턴보다 취약하고 수정이나 변화에 하드할 수 있으므로 사용에 주의해야 합니다.

### 07-5. 싱글톤 패턴

> Singleton 패턴은 클래스의 인스턴스가 오직 하나만 존재하도록 제한하는 패턴입니다.

- **싱글톤 패턴은 전역에서 접근 및 공유**해야 하는 단 하나의 객체가 필요할 때 유용합니다.
- 싱글톤 패턴을 구현하려면 이미 존재하는 인스턴스가 없어야 합니다. 인스턴스가 이미 존재하는 경우네는 해당 인스턴스의 참조를 반환합니다.
- 싱글톤 패턴은 정적 클래스나 객체와는 다르게 초기화를 지연시킬 수 있습니다.
  - 초기화 시점에 필요한 특정 정보가 유효하지 않을 수도 있기 때문에.
- 싱글톤 클래스의 인스턴스가 이미 생성되었다는 사실을 모른다면 해당 인스턴스를 찾아 사용하기 어렵습니다.
  - 이는 싱글톤이 객체나 클래스가 아닌 구조이기 때문입니다.
  - 클로저 변수 자체가 클로저가 아니라 클로저를 제공하는 함수 스코프가 클로저를 뜻한다는 것과 유사합니다.

#### 구현 방법

- ES2015+ 에서는 자바스크립트 클래스의 전역 인스턴스를 단 한 번만 생성하는 싱글톤 패턴을 구현할 수 있습니다.
- 또한 module export를 통해 싱글톤 인스턴스를 외부에서 사용할 수 있게 노출할 수도 있습니다.
- 새로운 클래스 인스턴스를 생성할 수는 없으나 클래스 내에 공개된 get, set 메서드를 통해 인스턴스를 읽거나 수정할 수 있습니다.

```js
// 싱글톤에 대한 참조를 가지는 인스턴스
let instance;

// 비공개 메서드와 변수
const privateMethod = () => {
 console.log('I am private');
};
const privateVariable = `I'm also private`;
const randomNumber = Math.random();  // 클래스 외부에 정의된 클로저 변수는 모든 인스턴스가 동일한 값을 공유함

// 싱글톤
class MySingleton {
 // 싱글톤 인스턴스가 이미 존재한다면 참조를 반환하고
 // 존재하지 않으면 생성합니다.
 constructor() {
   if (!instance) {
     // 공개된 속성
     this.publicProperty = 'I am also public';
     instance = this;
   }

   return instance;
 }

 // 공개 메서드
 publicMethod() {
   console.log('The public can see me!');
 }

 getRandomNumber() {
   return randomNumber;
 }
}
// [ES2015+] 이름 없이 기본 값으로 내보내기
export default MySingleton;

// 싱글톤에 대한 참조를 가지는 인스턴스
let instance;

// 싱글톤
class MyBadSingleton {
 // 항상 새로운 싱글톤 인스턴스를 생성
 constructor() {
   this.randomNumber = Math.random();
   instance = this;

   return instance;
 }

 getRandomNumber() {
   return this.randomNumber;
 }
}

export default MyBadSingleton;

// 사용법:
import MySingleton from './MySingleton';
import MyBadSingleton from './MyBadSingleton';

const singleA = new MySingleton();
const singleB = new MySingleton();
console.log(singleA.getRandomNumber() === singleB.getRandomNumber());
// true 출력 === 같은 인스턴스를 참조

const badSingleA = new MyBadSingleton();
const badSingleB = new MyBadSingleton();
console.log(badSingleA.getRandomNumber() !== badSingleB.getRandomNumber());
// true 출력 === 각자 다른 상태를 가지게 됨

// 참고: 무작위 수를 사용하고 있기에 수학적으로
// 가능성은 낮지만 두 숫자가 같을 수도 있습니다.
// 그렇다 하더라도 앞선의 예제는 유효합니다.

// 추가: 번수위치도 다시보면 클래스 외부에서 클로저 변수로 저장하는 것과 인스턴스 속성으로 저장해주는 차이가 존재한다...!
```

**싱글톤의 특징은 인스턴스에 대한 전역 접근을 허용한다는 것입니다.**
`GoF의 디자인 패턴`에서는 싱글톤 패턴의 **적합성**을 다음과 같이 이야기합니다.

- 클래스의 인스턴스는 정확히 하나만 있어야 하며 눈에 잘 보이는 곳에 위치시켜 접근을 용이하게 해야 합니다.
- 싱글톤의 인스턴스는 서브클래싱을 통해서만 확장할 수 있어야 하고, 코드의 수정 없이 확인된 인스턴스를 사용할 수 있어야 합니다.

#### 위에서 언급한 두번째 특징은 코드로 보는게 이해가 더 빠름!

```js
constructor() {
 if (this._instance == null) {
   if (isFoo()) {
     this._instance = new FooSingleton();
   } else {
     this._instance = new BasicSingleton();
   }
 }

 return this._instance;
}
```

- constructor은 팩토리 메서드와 비슷하게 기능하기 때문에, 인스턴스에 접근하는 코드를 직접 수정하지 않아도 됩니다.(내부에서 알아서 동작한다는 의미인듯) `FooSingleton`이 `BasicSingleton`의 서브 클래스가 되어 동일한 인터페이스를 구현할 것입니다.

#### 싱글톤 패턴은 유용하지만 JS환경에서는 설계를 다시 생각해봐야 한다는 신호일 수도 있습니다..!

JS환경에서는 객체를 직접적으로 생성할 수 있으므로, 객체를 생성하기 위해 클래스를 정의해야 하는 C++, JAVA 등과는 다르게 싱글톤 클래스를 만드는 대신 직접 객체 하나를 생성할 수도 있다는 의미입니다.
따라서 JS환경에서 싱글톤 클래스를 사용하는 것에는 다음과 같은 단점들도 함께 존재합니다.

- 싱글톤임을 파악하는 것이 힘들다.
  규모가 있는 모듈을 가져오는 경우, 어떤 클래스가 싱글톤 클래스인지 알아내기 어렵습니다. 일반 클래스와 혼동하여 어러 객체를 인스턴스화 하는 싱글톤의 특징과 알맞지 않은 방법으로 사용할 가능성이 있습니다.
- 테스트하기 힘들다. ㅠㅠ
  싱글톤은 숨겨진 의존성, 여러 인스턴스 생성의 제한, 의존성 대체의 어려움 등 다양한 문제로 테스트 과정에서는 어려움이 존재할 수 있습니다.
- 신중한 조정이 필요하다.
  싱글톤의 일상적인 사용 사례로는 전역 범위에 걸쳐 필요한 데이터를 저장하는 것이 있습니다.
  (여러 컴포넌트에서 사용할 수 있는 사용자 인증 정보나 쿠키 등등)
  따라서 데이터가 유효하게 된 뒤에는 사용할 수 있도록 올바른 실행 순서를 구현해주어야 하는데, 애플리케이션의 크기와 복잡성이 커집에 따라 어려우질 수 있습니다.

### 07-5.1. 리액트의 상태 관리

React를 통해 개발을 한다면 싱글톤 대신 ContextAPI나 Redux 같은 전역 상태관리 도구를 이용하여 개발할 수도 있습니다.
싱글톤과는 달리, 전역 상태 관리 도구는 변경이 불가능한 읽기 전용 상태를 제공합니다.
하지만 반대로 이야기하면 컴포넌트가 전역 상태를 직접 변경할 수 없게 만들어 의도치 않은 상태 변경을 예방할 수 있습니다.

### 07-6. 프로토타입 패턴

> GoF는 프로토타입 패턴을 이미 존재하는 객체를 복제해 만든 템플릿을 기반으로 새 각체를 생성하는 패턴이라고 정의했습니다.

- 프로토타입 패턴은 프로토타입의 상속을 기반으로 합니다.
- 프로토타입 상속은 클래스처럼 따로 정의되는것이 아닌, 이미 존재하는 다른 객체를 복제하여 새로운 객체를 만들어냅니다.
- 객체 내에 함수를 정의할 때(method?) 복사본이 아닌 참조로 생성되어 모든 자식 객체가 동일한 함수를 가리키게 할 수 있습니다. === 상속을 구현하는 쉬운 방법이면서 성능에서의 이점도 존재함

ps. 프로토타입은 가볍게 읽고 넘어갔습니다. :)

### 07-7. 팩토리 패턴

> 팩토리 패턴은 객체를 생성하는 생성 패턴의 하나입니다. 다른 패턴과 달리 생성자를 필요로 하지 않지만, 필요한 타입의 팩토리 객체를 생성하는 다른 방법을 제공합니다.

- 팩토리 패턴은 동적인 요소나 애플리케이션 구조에 깊게 의지하는 등의 상황처럼 객체 생성 과정이 복잡할때 특히 유용합니다.

```js
// Types.js - 백그라운드에서 사용하는 클래스
// 자동차를 정의하는 클래스
class Car {
  constructor({ doors = 4, state = "brand new", color = "silver" } = {}) {
    this.doors = doors;
    this.state = state;
    this.color = color;
  }
}

// 트럭을 정의하는 클래스
class Truck {
  constructor({ state = "used", wheelSize = "large", color = "blue" } = {}) {
    this.state = state;
    this.wheelSize = wheelSize;
    this.color = color;
  }
}

// FactoryExample.js
// 차량 팩토리를 정의
class VehicleFactory {
  constructor() {
    this.vehicleClass = Car;
  }

  // 새 차량 인스턴스를 생성하는 팩토리 함수
  createVehicle(options) {
    const { vehicleType, ...rest } = options;

    switch (vehicleType) {
      case "car":
        this.vehicleClass = Car;
        break;
      case "truck":
        this.vehicleClass = Truck;
        break;
      // 해당되지 않으면 VehicleFactory.prototype.vehicleClass에 Car를 할당
    }

    return new this.vehicleClass(rest);
  }
}

// 자동차를 만드는 팩토리의 인스턴스 생성
const carFactory = new VehicleFactory();
const car = carFactory.createVehicle({
  vehicleType: "car",
  color: "yellow",
  doors: 6,
});

// 자동차가 vehicleClass/prototype Car로 생성되었는지 확인
// 출력: true
console.log(car instanceof Car);
// color: "yellow", doors: 6, state: "brand new" 인 자동차 객체 출력
console.log(car);
```

#### 07-7.1. 팩토리 패턴을 사용하면 좋은 상황

- 객체나 컴포넌트의 생성 과정이 높은 복잡성을 가지고 있을 때
- 상황에 맞춰 다양한 객체 인스턴스를 편리하게 생성할 수 있는 방법이 필요할 때
- 같은 속성을 공유하는 여러 개의 작은 객체 또는 컴포넌트를 다뤄야 할 때
- 덕 타이핑(duck typing)같은 API 규칙만 충족하면 되는 다른 객체의 인스턴스와 함께 객체를 구성할 때, 또한 디커플링에도 유용.

#### 07-7.2. 팩토리 패턴을 사용하면 안되는 상황

- 잘못된 상황에 팩토리 패턴을 적용한다면 애플리케이션의 복잡도가 크게 증가할 수 있습니다.
- 객체 생성 인터페이스 제공이 작업중인 라이브러리나 프레임워크의 설계 목표가 아니라면 차라리 위험을 피해 생성자를 사용하는 것이 좋습니다.

#### 07-7.3. 추상 팩토리 패턴

> 추상 팩토리 패턴은 같은 목표를 가진 각각의 팩토리들을 하나의 그룹으로 캡슐화하는 패턴입니다. 또한 객체가 어떻게 생성되는지에 대한 세부사항을 알 필요 없이 객체를 사용할 수 있게 합니다.

- 객체의 생성 과정에 영향을 받지 않아야 하거나 여러 타입의 객체로 작업해야 하는 경우에 추상 팩토리를 사용하면 좋습니다.

> 아키텍처 설계에서 의존성 역전 원칙(DIP)과 인터페이스 분리 원칙(ISP)와 연관되어있는듯한 개념인듯
>
> - 의존성 역전 원칙(DIP): 고수준 모듈은 저수준 모듈에 의존해서는 안 되며, 둘 다 추상화에 의존해야 함
> - 인터페이스 분리 원칙(ISP): 클라이언트는 자신이 사용하지 않는 인터페이스에 의존해서는 안 됨
>
> 책에서는 VehicleFactory 를 예시로 들었지만 HTTP 클라이언트를 예시로 본다면 추상화된 패턴을 도입한다면 Axios나 Fetch, Ajax 등에 대한 세부사항을 알 필요 없이 의존성과 결합성도 낮추고 유연하게 라이브러리 변경 가능한 구조 설계가 가능할듯

#### 07-8. 구조 패턴 (Structural)

> 구조 패턴은 클래스와 객체의 구성을 다룹니다. 상속의 개념을 통해 인터페이스와 객체를 구성하여 새로운 기능을 추가할 수 있는 것처럼 말입니다.

자바스크립트의 구조 Structural 패턴들

- 퍼사드 패턴
- 믹스인 패턴
- 데코레이터 패턴
- 플라이웨이트 패턴

#### 07-9. 퍼사드 패턴

> 퍼사드란 실제 모습을 숨기고 꾸며면 겉모습만을 세상에 드러내는 것을 뜻합니다.

- 퍼사드 패턴은 심층적인 복잡성을 숨기고, 사용하기 편리한 높은 수준의 인터페이스를 제공하는 패턴입니다.
- jQuery 같은 라이브러이에서 흔히 볼 수 있는 구조 패틴입니다. 퍼사드 특징을 가지거나 제한된 추상화 메서드만을 공개하여 사용할 수 있도록 합니다.
- 숨겨진 하위 시스템이 아닌 바깥에 나타난 퍼사드와 직접 상호작용 할 수 있습니다.
  - 예시로 jQuery 코어의 많은 내부 메서드를 직접 찾아 실행하는 대신 쉽게 공개된 인터페이스를 사용하게 됨.
  - 다른 예시로 DOM API와 상태를 나타내는 변수를 직접 다룰 필요성을 줄여주기도 함.
- 퍼사드의 장점으로는 사용하기 쉽다는 점(높은 수준의 인터페이스 제공)과 패턴 구현에 필요한 코드의 양이 적다는 점입니다.

```js
// privateMethods.js
const _private = {
  i: 5,
  get() {
    console.log(`current value: ${this.i}`);
  },
  set(val) {
    this.i = val;
  },
  run() {
    console.log('running');
  },
  jump() {
    console.log('jumping');
  },
};

export default _private;

// module.js
import _private from './privateMethods.js';

const module = {
 facade({ val, run }) {
   _private.set(val);
   _private.get();
   if (run) {
     _private.run();
   }
 },
};

export default module;

// index.js
import module from './module.js';

// 출력: "current value: 10" and "running"
module.facade({
 run: true,
 val: 10,
});
```

예제 코드에서 `module.facade()`는 모듈 내부에서 비밀스런 동작을 실행하지만 사용자는 내부에서 무슨 일이 벌어지는지 몰라도 됩니다. 구현 수준의 세부사항을 알지 않고도 훨씬 쉽게 사용할 수 있게 된 것 입니다.

> API가 대표적인 퍼사드 패턴인듯...?
> 내부 로직은 숨기고 사용하기 쉬운 인터페이스를 제공 받으며 구현 세부사항의 변경으로부터 클라이언트 코드를 보호
> HAL과도 유사한것 같고....

### 07-10. 믹스인 패턴

> 전통적인 프로그래밍 언어에서 믹스인은 서브클래스가 쉽게 상속받아 기능을 재사용할 수 있도록 하는 클래스 입니다.

### 07-11. 서브클래싱

> ES2015+에서 도입된 기능을 통해 기존 또는 부모 클래스를 확장활 수도, 부모 클래스의 메서드를 호출할 수도 있게 되었습니다.
> 부모 클래스를 확장하는 자식 클래스를 서브클래스라고 합니다.

- 서브클래싱이란 부모 클래스 객체에서 속성을 상속받아 새로운 객체를 만드는 것을 뜻합니다.
  - 서브클래스는 부모 클래스에서 먼저 정듸된 메서드를 override하는 것도 가능합니다.
  - 서브클래스의 메서드는 override된 부모 클래스의 메서드를 호출할 수도 있는데, 이를 **메서드 체이닝** 이라고 부릅니다.
  - 마찬가지로 부모 클래스의 생성자를 호출할 수도 있는데, 이를 **생성자 체이닝** 이라고 부릅니다.

```js
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.gender = "male";
  }
}
// Person의 새 인스턴스는 이처럼 생겨 보입니다.
const clark = new Person("Clark", "Kent");

class Superhero extends Person {
  constructor(firstName, lastName, powers) {
    // 부모 클래스의 생성자를 호출합니다.
    super(firstName, lastName);
    this.powers = powers;
  }
}

// Superhero 인스턴스를 만듭니다.
const SuperMan = new Superhero("Clark", "Kent", ["flight", "heat-vision"]);
console.log(SuperMan);

// power를 가진 Person을 출력합니다.

// Superhero 클래스의 생성자는 Person 클래스를 확장해 인스턴스를 생성합니다.
// 따라서 Person 클래스의 기본값을 Superhero 클래스에서 새로 값을 오버이드하여 할당할 수 있습니다.
```

### 07-12. 또 다시 기어나온 믹스인...

![](https://i.namu.wiki/i/IyzybdMMTgDt7yG6RnioaEPK1G_5QizLkkzl8IntRH_TTTFmxL7wIQzpL11s1eeK-WM2NSX57YnbpjIvI8Yuiw.webp)

> 자바스크립트에서는 기능의 확장을 위해 믹스인의 상속을 이용합니다.
> 새롭게 만들어지는 클래스는 부모 클래스로부터 메서드와 속성을 부여받습니다.
> 또한 자신만의 속성과 메서드를 정의할 수도 있습니다.

- 자바스크립트의 클래스는 부모 클래스를 하나만 가질 수 있지만 여러 클래스의 기능을 섞는 것으로 문제를 해결할 수 있습니다.
- 자바스크립트에서 클래스는 표현식(expression)뿐만 아니라 문(statement)으로도 사용할 수 있습니다.
  - 이 표현식은 평가될 때마다 새로운 클래스를 반환합니다.
  - extends 절은 클래스 생성자를 반환하는 임의의 표현식을 혀용할 수도 있습니다.

```js
// 동적으로 부모 클래스를 받아 확장하는 믹스인 함수 생성
const MyMixins = (superclass) =>
  class extends superclass {
    moveUp() {
      console.log("move up");
    }

    moveDown() {
      console.log("move down");
    }

    stop() {
      console.log("stop! in the name of love!");
    }
  };

// ===== 사용처 =====

// CarAnimator 생성자의 기본 구조
class CarAnimator {
  moveLeft() {
    console.log("move left");
  }
}

// PersonAnimator 생성자의 기본 구조
class PersonAnimator {
  moveRandomly() {
    /*...*/
  }
}

// MyMixins을 사용하여 CarAnimator 확장
class MyAnimator extends MyMixins(CarAnimator) {}

// carAnimator의 새 인스턴스 생성
const myAnimator = new MyAnimator();
myAnimator.moveLeft();
myAnimator.moveDown();
myAnimator.stop();

// 출력:
// move left
// move down
// stop! in the name of love!
```

#### 07-12.1. 장단점

- 믹스인을 사용하여 함수의 중복을 줄이고 재사용성을 높일 수 있습니다.
- 애플리케이션에서 객체 인스턴스 사이에 공유되는 기능이 있다면 믹스인을 통해 기능을 공유하여 중복을 피하고 고유 기능을 구현하는데 집중할 수 있습니다.
- 단, 믹스인의 단점이 있다면 몇몇의 개발자들은 클래스나 객체의 프로토타입에 기능을 주입하는 것을 나쁜 방법이라고 여깁니다. === 고수준에서 저수준으로 의존성이 흘러가는것이 문제인건가
  - 프로토타입의 오염과 함수 출처에 대한 불확실성 초래

### 07-13. 데코레이터 패턴 (Decorator)

> 데코레이터 패턴은 코드 재사용을 목표로 하는 구조 패턴입니다. 믹스인과 마찬가지로 객체 서브클래싱의 또 다른 방법이라고 생각하면 됩니다.
>
> - 기본적으로 데코레이터는 기존 클래스에 동적으로 기능을 추가하기 위해 사용합니다.
> - 데코레이터 자체는 기본 기능에 필수적이지 않다는 생각이었습니다. 필수적이었다면 부모 클래스에 이미 구현되었을 것입니다.

- 데코레이터를 사용하면 기존 시스템의 내부 코드를 힘겹게 수정하지 않고도 기능을 추가할 수 있게 됩니다.
- 데코레이터 사용의 주된 이유는 애플리케이션의 기능이 다양한 타입의 객체를 필요로 할 수도 있기 때문입니다. (다양한 유형과 타입의 사람이 존재하듯이 다양한 타입으로 정의할 기능을 필요로 하는 경우가 존재함)
- 데코레이터 패턴은 객체의 생성을 신경쓰지 않는 대신 기능의 확장에 조금 더 초점을 둡니다.
- 프로토타입 상속에 의지하기 보다는 하나의 베이스가 되는 클래스에 추가 기능을 제공하는 데코레이터 객체를 점진적으로 추가합니다. 이는 서브클래싱 대신 베이스 객체에 속성이나 메서드를 추가하여 간소화 하겠다는 아이디어 입니다.

```js
// Vehicle 생성자
class Vehicle {
  constructor(vehicleType) {
    // 일부 합리적인 기본값
    this.vehicleType = vehicleType || "car";
    this.model = "default";
    this.license = "00000-000";
  }
}

// 기본 Vehicle에 대한 테스트 인스턴스
const testInstance = new Vehicle("car");
console.log(testInstance);

// 출력:
// vehicle: car, model:default, license: 00000-000

// 데코레이트할 새로운 차량 인스턴스를 생성합니다.
const truck = new Vehicle("truck");

// Vehicle에 추가하는 새로운 기능
truck.setModel = function (modelName) {
  this.model = modelName;
};

truck.setColor = function (color) {
  this.color = color;
};

// 같 설정자와 같 함달이 올바르게 작동하는지 테스트
truck.setModel("CAT");
truck.setColor("blue");

console.log(truck);

// 출력:
// vehicle:truck, model:CAT, color: blue

// "vehicle"이 변경되지 않았음을 보여줍니다.
const secondInstance = new Vehicle("car");
console.log(secondInstance);

// 출력:
// vehicle: car, model:default, license: 00000-000
```

### 07-15. 장단점

- 데코레이터 패턴의 객체는 새로운 기능으로 감싸져 확장되거나 `데코레이트`될 수 있으며 베이스 객체가 변경될 걱정 없이 사용할 수 있습니다.
- 더 넓은 의미에서 보았을 대 수 많은 서브클래스에 의존할 필요도 없습니다.
- 그러나, 네임 스페이스에 작고 비슷한 객체를 추가하기 때문에 잘 관리하지 않는다면 애플리케이션의 구조를 무척 복잡하게 만들 수 있습니다.
  - 이 패턴에 익숙치 않은 다른 개발자가 패턴 사용의 목적을 파악하기 어렵게 된다면 관리가 힘들어 질 수도 있지만, 문서화나 패턴에 대한 이해도를 높이면 해결 가능하긴 함.

### 07-16. 플라이웨이트 패턴 (Flyweight)

> 플라이웨이트 패턴은 반복되고 느리고 비효율적으로 데이터를 공유하는 코드를 최적화하는 전통적인 구조적 해결 방법입니다. 연관된 객체끼리 데이터를 공유하면서 애플리케이션의 메모리를 최소화하는 목적을 가지고 있습니다.

- 플라이웨이트 패턴의 데이터 공유 방식은 여러 비슷한 객체나 데이터 구조에서 공통으로 사용되는 부분만을 하나의 외부 객체로 내보내는 것으로 이루어집니다.
  - 각 객체에 데이터를 저장하기 보다는 하나의 의존 외부 데이터에 모아서 저장할 수 있습니다.

#### 07-16.1. 사용법

플라이웨이트 패턴의 사용법으로는 두 가지 방법이 있습니다.

1. 데이터 레이어에서 메모리에 저장된 수 많은 비슷한 객체 사이로 데이터를 공유하는 것.
2. DOM 레이어에도 플라이웨이트를 적용할 수 있습니다. 비슷한 동작을 하는 이벤트 핸들러를 모든 자식 요소에 등록하기 보다는 부모 요소 같은 중앙 이벤트 관리자에게 맡기는 방법이 있습니다.

#### 07-16.2. 데이터 공유

전통적인 플라이웨이트 패턴에 대해 알고 넘어가야 할 두가지 개념이 있습니다.

- 내재적 intrinsic 상태
- 외재적 extrinsic 상태

내재적 정보는 객체 내부 메서드에 필요한 것이며, 없으면 절대로 동작하지 않습니다.
반면, 외재적 정보는 제거되어 외부에 저장될 수 있습니다.

- 같은 내재적 정보를 지닌 객체를 팩토리 메서드를 사용해 만들어진 하나의 공유된 객체로 대체할 수 있습니다. === 저장된 내부 데이터 양을 경량화 할 수 있음
  - 이미 공통 부분으로 인스턴스화된 객체를 재사용하면 되기 때문에 객체의 내재적 정보가 다를 경우에만 새로운 객체 복사본을 생성하면 됩니다.
- 외재적 정보를 다룰 때에는 따로 관리자를 사용합니다.
  - 관리자는 다양한 방법으로 구현 가능하지만 한 가지 방법만 꼽아본다면 플라이웨이트 객체와 내재적 상태를 보관하는 중앙 데이터베이스를 관리자로 사용하는 것입니다.
