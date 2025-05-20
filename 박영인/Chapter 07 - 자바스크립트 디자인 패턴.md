# 생성 패턴
생성 패턴은 **유연하고 재사용 가능한 방식으로 객체를 만드는 방법을 제공**하는 디자인 패턴이다.
**생성 로직을 캡슐화하여 코드의 유연성과 유지보수성**을 높여준다.

## 생성자 패턴
생성자(Constructor)는 객체가 새로 만들어진 뒤 초기화 하는데 사용하는 메서드이다.
new 키워드와 함께 호출!

### 객체 생성
**자바스크립트에서 객체를 만들 때 사용하는 일반적인 방법들**
```js
// 1. 리터럴 표기법
const newObject = {};
// 2. Object.create()
const newObject = Object.create(Object.prototype);
// 3. new 키워드
const newObject = new Object();
```

**객체 속성 할당하는 방법**
```js
// 1. 도트 표기법
newObject1.someKey = "Hello World"; // 속성 할당
var key1 = newObject1.someKey;      // 속성 가져오기

// 2. 대괄호 표기법
newObject1["anotherKey"] = "Hello Again"; // 속성 할당
var key2 = newObject1["anotherKey"];      // 속성 가져오기

// 3. Object.defineProperty로 속성 정의
Object.defineProperty(newObject1, "controlledKey", {
  value: "This is controlled",
  writable: true,       // 값 변경 가능
  enumerable: true,     // 열거 가능 (for...in 등에서 노출됨)
  configurable: true    // 삭제 및 재정의 가능
});

// 4. Object.defineProperties로 여러 속성을 한 번에 정의
Object.defineProperties(newObject1, {
  someKey: {
    value: "Hello World",
    writable: true
  },
  anotherKey: {
    value: "Foo bar",
    writable: false
  }
});

// 속성 접근은 도트 표기법이나 대괄호 표기법 둘 다 사용 가능
console.log(newObject1.someKey);        // "Hello World"
console.log(newObject1["anotherKey"]);  // "Foo bar"
```

> 근데 Object.defineProperty, Object.defineProperties 처음본다..
> 많이 사용하는가? 👉 일반적으로 자주 쓰이는 방법은 아님
> 
> ⁉️ **언제 Object.defineProperty()를 쓰는가?**
> 속성의 동작을 명시적으로 제어하고 싶을 때:
> - `writable: false`: 읽기 전용, 편집 불가
> - `enumerable: false`: for...in이나 Object.keys()에 나타나지 않게
> - `configurable: false`: 속성을 삭제하거나 재정의하지 못하게

### 생성자의 기본특징
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

// 사용법:

// 새로운 Car 인스턴스 생성
let civic = new Car('Honda Civic', 2009, 20000);
let mondeo = new Car('Ford Mondeo', 2010, 5000);

// 브라우저 콘솔에서 결과 확인
console.log(civic.toString());
console.log(mondeo.toString());
```
Car 생성자로 객체를 생성할 때 마다 toString()같은 함수가 새로 정의되어 리소스 낭비가 생긴다.

### 프로토타입을 가진 생성자
프로토타입 객체는 함수나 클래스의 모든 인스턴스가 공통으로 사용할 수 있는 메서드를 정의하는 데 사용된다.
생성자를 통해 생성된 객체들은 동일한 프로토타입 객체를 공유한다.
```js
class Car {
  constructor(model, year, miles) {
    this.model = model;
    this.year = year;
    this.miles = miles;
  }
}

// 프로토타입 객체의 재정의를 피하기 위해 Object.prototype 대신
// Object.prototype.newMethod 형태를 사용하는 것에 유의하세요.
// 기존에 이미 정의된 프로토타입 객체를 유지하기 위해서입니다.
Car.prototype.toString = function () {
  return `${this.model} has done ${this.miles} miles`;
};

// 사용법:
let civic = new Car('Honda Civic', 2009, 20000);
let mondeo = new Car('Ford Mondeo', 2010, 5000);

console.log(civic.toString());
console.log(mondeo.toString());
```
이 예제에서 car 인스턴스들은 모두 같은 toString()함수를 공유한다.

## 모듈 패턴
모듈은 프로젝트를 구성하는 코드 단위를 체계적으로 분리, 관리하는데 효과적으로 활용된다.

### 객체 리터럴
> 평소에 그냥 객체 정의하는 방식

```js
const myModule = {
  name: "myModule",
  greet: function() {
    console.log("Hello!");
  }
};
myModule.greet();
```

### 모듈 패턴의 비공개(Private) 처리 방식
모듈 패턴은 **클로저를 활용해 내부 상태와 구현 세부사항을 감추는 방법**이다.
주로 즉시 실행 함수(IIFE)를 사용해 객체를 반환하여 클로저를 형성하거나, 모듈 스코프를 통해 내부 데이터를 은닉한다.
이렇게 하면 외부에 **공개할 메서드와 숨기고 싶은 데이터**를 명확히 구분할 수 있어, 전역 스코프를 오염시키지 않고 다른 코드와의 충돌도 피할 수 있다.
다만, 반환된 객체에 포함된 속성이나 메서드는 여전히 외부에서 접근할 수 있기 때문에 **완전한 비공개가 필요한 경우**에는 **WeakMap**을 활용해 데이터를 숨기는 방식이 사용된다.


### 모듈 패턴 예제
```js
 // 비공개: 외부 스코프에 있는 카운터 변수
let counter = 0;

// 모듈 정의
const testModule = {
  // 카운터 증가
  incrementCounter() {
    return counter++;
  },

  // 카운터 초기화
  resetCounter() {
    console.log(`counter value prior to reset: ${counter}`);
    counter = 0;
  },
};

// 디폴트 export
export default testModule;

/* 사용 예시:
import testModule from './testModule';

// 카운터 증가
testModule.incrementCounter();

// 카운터 값 확인하고 리셋
// 출력: counter value prior to reset: 1
testModule.resetCounter();
*/
```

### 모듈 패턴의 변형
**믹스인 가져오기 변형**
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
전역 스코프에 정의된 요소를 모듈로 분리해 **의도된 인터페이스만 노출**하고, **가져오는 쪽에서 원하는 이름으로 사용할 수 있는 유연함**을 제공한다.


**장점**
- 코드 간 의존성을 줄이고 모듈 간 독립성을 유지할 수 있음
- 원하는 것만 export 하여 의도치 않은 변수나 함수 노출 방지, 전역 오염 차단
- 잘 구조화된 모듈은 테스트, 확장, 유지보수에 유리함

**단점**
- 공개/비공개 멤버를 서로 다르게 접근, 관리해야하기 때문에 코드 유지보수가 복잡해짐
- 비공개 멤버는 테스트 코드에서 다루기 어렵고, 오류를 고칠 때 복잡도를 높임

### WeakMap을 사용하는 최신 모듈 패턴
```js
// WeakMap을 활용한 비공개 멤버 처리
let _counter = new WeakMap();

class Module {
  constructor() {
    _counter.set(this, 0);
  }

  incrementCounter() {
    let counter = _counter.get(this);
    counter++;
    _counter.set(this, counter);
    return _counter.get(this);
  }

  resetCounter() {
    console.log(`counter value prior to reset: ${_counter.get(this)}`);
    _counter.set(this, 0);
  }
}

const testModule = new Module();

// 사용법:

// 카운터 증가
testModule.incrementCounter();

// 카운터 값을 확인하고 리셋
// 출력: counter value prior to reset: 1
testModule.resetCounter();
```

> [!NOTE]
    > **WeakMap**
    > WeakMap은 **키를 객체로만 가질 수 있는 Map 구조**이며, 키로 참조된 객체가 더 이상 사용되지 않으면 **자동으로 WeakMap에서도 값이 제거(GC)된다.**
    > forEach, for...of, .size 같은 순회가 불가하다.


## 노출 모듈 패턴
노출 모듈 패턴은 모든 변수와 함수를 비공개 스코프에 정의한 뒤, 외부에 노출할 항목만 공개 객체에 연결하여 접근을 허용하는 패턴이다.

### 노출 모듈 패턴 예제
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

// 비공개 함수와 속성에 접근하는 공개 포인터
const myRevealingModule = {
  start: publicFunction,
  increment: publicIncrement,
  count: publicGetCount,
};

export default myRevealingModule;

// 사용법:
import myRevealingModule from './myRevealingModule';

myRevealingModule.start();
```

### 장점
- 코드의 일관성이 유지되며, 가독성이 향상된다.

### 단점
- 비공개 멤버를 참조하는 공개 속성은 외부에서 수정할 수 없다. 공개 속성을 수정하더라도 참조된 내부 구현에는 영향을 주지 않는다.


## 싱글톤 패턴
싱글톤 패턴은 **클래스의 인스턴스가 단 하나만 존재하도록 제한**하는 디자인 패턴이다.
전역에서 접근하고 공유해야 하는 **유일한 객체가 필요할 때 유용**하다.
이미 인스턴스가 존재하는 경우에는 **새로 생성하지 않고 기존 인스턴스를 반환**한다.

ES2015 이후에는 **모듈 내보내기(export)** 를 통해 싱글톤 인스턴스를 외부에 노출할 수 있다.
새로운 인스턴스를 직접 생성할 수는 없지만, **get/set 메서드를 통해 상태를 읽거나 변경**할 수 있다.

### 싱글톤 구현
 **✅ 올바른 싱글톤 구현 예제 (MySingleton)**

```js
// 싱글톤에 대한 참조를 가지는 인스턴스
let instance;

// 비공개 메서드와 변수
const privateMethod = () => {
  console.log('I am private');
};
const privateVariable = "I'm also private";
const randomNumber = Math.random();

// 싱글톤 클래스
class MySingleton {
  // 싱글톤 인스턴스가 이미 존재한다면 참조를 반환하고,
  // 존재하지 않으면 생성합니다.
  constructor() {
    if (!instance) {
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

export default MySingleton;
```

**❌ 잘못된 싱글톤 구현 예제 (MyBadSingleton)**
```js
// 싱글톤에 대한 참조를 가지는 인스턴스
let instance;

// 싱글톤 클래스
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
```

**📦 사용법 및 출력결과**
```js
import MySingleton from './MySingleton';
import MyBadSingleton from './MyBadSingleton';

const singleA = new MySingleton();
const singleB = new MySingleton();

console.log(singleA.getRandomNumber() === singleB.getRandomNumber());
// true 출력

const badSingleA = new MyBadSingleton();
const badSingleB = new MyBadSingleton();

console.log(badSingleA.getRandomNumber() !== badSingleB.getRandomNumber());
// true 출력
// 서로 다른 인스턴스가 생성됨
```

<hr>

### 싱글톤 패턴의 적합성
- 클래스의 인스턴스는 하나만 있어야 하며, 전역같이 눈에 잘 보이는 곳에 위치시켜 접근을 용이하게 해야한다.
- 싱글톤의 인스턴스는 서브클래싱을 통해서만 확장할 수 있어야 한다. 코드의 수정 없이 확장된 인스턴스를 사용할 수 있어야 한다.

#### 서브클래싱 예제
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
 
👉 constructor는 **팩토리 메서드처럼 동작**하며,
FooSingleton은 BasicSingleton의 **서브클래스로서 동일한 인터페이스를 구현**한다.

따라서 인스턴스를 사용하는 외부 코드는
this.instance가 **어떤 서브클래스인지 알 필요 없이** 그대로 사용할 수 있고,
**인스턴스를 사용하는 코드를 수정할 필요도 없다.**

<hr>


싱글톤에서는 **지연 실행이 중요**하다. 인스턴스를 언제 초기화할지 판단해, **동적 초기화 순서로 인한 문제를 방지**해야 한다.
정적 인스턴스로 싱글톤을 구현하더라도, **사용 시점까지 리소스나 메모리를 소비하지 않도록** 지연 생성하는 것이 바람직하다.

하지만 이렇게 **지연 초기화된 정적 객체**를 사용할 경우, 항상 같은 순서로 초기화되는지를 **명확히 확인**해야 한다.
정적 객체는 **초기화 순서를 명시적으로 제어하기 어렵기 때문에**, 이런 구조가 많아지면 **초기화 의존성**이 증가하고 **확장성과 유지보수성이 떨어진다**.

👉 **정적 객체와 싱글톤은 각각 유용하지만, 목적에 맞게 구분해서 사용해야 하며 남용하거나 혼용해서는 안 된다.**


### 적합한 싱글톤 패턴 사용 상황
```js
// 예시: const options = { name: "test", pointX: 5 };
class Singleton {
  constructor(options = {}) {
    // 싱글톤에 속성을 할당합니다.
    this.name = 'SingletonTester';
    this.pointX = options.pointX || 6;
    this.pointY = options.pointY || 10;
  }
}

// 인스턴스를 담을 변수
let instance;

// 정적 변수와 메서드의 구현
const SingletonTester = {
  name: 'SingletonTester',
  // 인스턴스를 가져오는 메서드
  // 싱글톤 객체의 싱글톤 인스턴스를 반환합니다. -> 함수 실행 시점에 지연 초기화
  getInstance(options) {
    if (instance === undefined) {
      instance = new Singleton(options);
    }
    return instance;
  },
};

const singletonTest = SingletonTester.getInstance({
  pointX: 5,
});

// 값을 확인하기 위해 pointX를 출력합니다.
// 출력: 5
console.log(singletonTest.pointX);
```


<hr>

자바스크립트에서 싱글톤이 필요하다는 것은, **설계를 다시 점검해봐야 한다는 신호일 수 있다.**
경우에 따라 굳이 싱글톤 클래스를 만들지 않고, **단순히 객체를 직접 생성해서 사용하는 것으로도 충분**할 수 있다.
리액트라면 싱글톤 대신 컴포넌트가 전역상태를 직접 변경할 수 없는 **전역상태관리 도구**를 이용하는게 더 좋을 수 있다.

### 자바스크립트에서 싱글톤 클래스 사용할 때의 단점
#### 싱글톤임을 파악하는 것이 힘들다
큰 규모의 코드베이스에서는 어떤 클래스가 싱글톤인지 알아보기 어렵다.
잘못 판단하면 **싱글톤을 일반 클래스처럼 사용하여 중복 인스턴스를 생성**할 수 있다.

> ? 다른 언어는 쉽나? -> 다른 언어는 구조가 자바스크립트 보다는 명확하다고 한다..

#### 테스트하기 힘들다
**의존성이 숨겨져 있고**, 인스턴스가 고정돼 있어 모킹(mocking)이나 대체가 어렵다.
특히 여러 인스턴스를 만들어야 하거나, **의존성을 분리해야 하는 테스트 상황**에서는 장애가 된다.

#### 신중한 조정이 필요하다
싱글톤은 전역 상태(예: 사용자 정보, 쿠키 등)를 저장하는 데 자주 쓰인다.
이런 데이터가 **초기화된 이후에만 사용되어야 하므로**, 실행 순서를 정확히 제어해야 한다.
앱이 커질수록 **의존성과 실행 타이밍을 맞추는 일이 점점 어려워진다.**


## 프로토타입 패턴
**프로토타입 패턴**은 **이미 존재하는 객체를 복제하여**, 템플릿처럼 활용해 **새로운 객체를 생성하는 패턴**이다.
이 패턴은 **프로토타입 기반 상속**을 바탕으로 하며, 기존 객체를 그대로 복제하여 새로운 객체를 만드는 방식으로 동작한다.

또한, 프로토타입 객체 내의 메서드는 **각 객체에 개별 복사되지 않고**, **공통된 prototype을 참조**하므로 **메모리 사용 측면에서 효율적이다.**

Object.create()는 기존 객체를 기반으로 새로운 객체를 만들 수 있게 해주는 **프로토타입 기반 상속 도구**이다.
이를 통해 **명확하게 프로토타입을 지정하여 객체를 생성**할 수 있다.

```js
const myCar = {
  name: 'Ford Escort',
  drive() {
    console.log("Weeee. I'm driving!");
  },
  panic() {
    console.log("Wait. How do you stop this thing?");
  }
};

// yourCar는 myCar를 프로토타입으로 가지는 새로운 객체이다.
const yourCar = Object.create(myCar);
console.log(yourCar.name);  // 'Ford Escort'
```


## 팩토리 패턴
팩토리 패턴은 객체 생성을 **전용 팩토리(함수 또는 클래스)에 위임**하여, **생성자를 사용하지 않고** 객체를 생성할 수 있도록 하는 패턴이다.
어떤 타입의 객체가 필요한지 팩토리에게 **알려주기만 하면**, 팩토리는 적절한 인스턴스를 반환한다.
특히 객체 생성 과정이 복잡하거나 앱 구조에 따라 유동적인 경우에 유리하다.

### 팩토리 패턴 구현
```js
// Types.js - 백그라운드에서 사용되는 클래스
// 자동차를 정의하는 클래스
class Car {
  constructor({ doors = 4, state = 'brand new', color = 'silver' } = {}) {
    this.doors = doors;
    this.state = state;
    this.color = color;
  }
}

// 트럭을 정의하는 클래스
class Truck {
  constructor({ state = 'used', wheelSize = 'large', color = 'blue' } = {}) {
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
      case 'car':
        this.vehicleClass = Car;
        break;
      case 'truck':
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
  vehicleType: 'car',
  color: 'yellow',
  doors: 6,
});

// 자동차가 vehicleClass/prototype Car로 생성되었는지 확인
// 출력: true
console.log(car instanceof Car);
// color: "yellow", doors: 6, state: "brand new" 인 자동차 객체 출력
console.log(car);
```

### 팩토리 패턴을 사용하면 좋은 상황
- **객체나 컴포넌트 생성 과정이 복잡**할 때
- **다양한 인스턴스를 상황에 맞게 편하게 생성**해야 할 때
- **공통 속성을 가진 여러 개의 객체나 컴포넌트**를 반복적으로 만들어야 할 때
- **덕 타이핑(duck typing)** 방식의 **유연한 API 설계**가 필요한 경우
- **객체 간 의존성을 줄이고(디커플링)**, 유연한 구성 방식이 필요한 경우


### 팩토리 패턴을 사용하면 안 되는 상황
팩토리 패턴을 **잘못 적용하면 오히려 복잡도가 증가**할 수 있다:
- **단순히 객체 하나만 만들면 되는 상황**인데도 불필요하게 팩토리를 쓰는 경우
- 팩토리 패턴은 **생성 과정을 인터페이스 뒤로 감추므로**, **단위 테스트가 복잡해지거나 유지보수 비용이 올라갈 수 있음**

### 추상 팩토리 패턴
**추상 팩토리 패턴**은 동일한 목적을 가진 여러 팩토리들을 하나의 그룹으로 묶어 캡슐화하는 생성 패턴이다.
이 패턴을 사용하면 객체가 어떻게 생성되는지에 대한 세부 사항을 알 필요 없이 객체를 사용할 수 있다.
객체 생성 방식에 영향을 주지 않으면서 다양한 종류의 관련 객체들을 함께 다뤄야 할 경우, 추상 팩토리 패턴을 사용하는 것이 효과적이다.

#### 추상 팩토리 구현
```js
class AbstractVehicleFactory {
  constructor() {
    // 차량 타입을 저장하는 곳, Class를 넣어둔다.
    this.types = {};
  }

  getVehicle(type, customizations) {
    const Vehicle = this.types[type];
    return Vehicle ? new Vehicle(customizations) : null;
  }

  registerVehicle(type, Vehicle) {
    const proto = Vehicle.prototype;
    // 차량 기능을 충족하는 클래스만 등록
    if (proto.drive && proto.breakDown) {
      this.types[type] = Vehicle;
    }
    return this;
  }
}

// 사용법:
const abstractVehicleFactory = new AbstractVehicleFactory();

abstractVehicleFactory.registerVehicle('car', Car);
abstractVehicleFactory.registerVehicle('truck', Truck);

// 추상 차량 타입으로 새 자동차를 인스턴스화
const car = abstractVehicleFactory.getVehicle('car', {
  color: 'lime green',
  state: 'like new',
});

// 비슷한 방법으로 트럭도 인스턴스화
const truck = abstractVehicleFactory.getVehicle('truck', {
  wheelSize: 'medium',
  color: 'neon yellow',
});
```


# 구조 패턴
구조 패턴은 **클래스나 객체 간의 관계를 정의하고 구성**하여, 시스템의 **유지보수성과 확장성**을 높이는 데 목적이 있는 디자인 패턴이다.

## 퍼사드 패턴
퍼사드 패턴은 **복잡한 시스템 내부 구조를 감추고**, 외부에는 **단순하고 일관된 인터페이스를 제공**하는 구조 패턴이다. (퍼사드: 실제 모습을 숨기고 꾸며낸 겉모습만을 드러내는 것)

jQuery와 같은 라이브러리는 퍼사드 패턴을 통해 **코어 내부 메서드나 DOM API를 직접 다룰 필요성을 줄여준다.**
> lodash, axios 등의 라이브러리, react hook들도 전부 퍼사드인 것 같다.

복잡한 하위 시스템에 **직접 접근하지 않고 간접적으로 상호작용함으로써**, 에러 발생 가능성을 줄이고, **더 쉽게 사용할 수 있다는 장점**이 있다. 

### 퍼사드 예제
브라우저 별 이벤트를 처리하는 로직을 숨겼다.
```js
const addMyEvent = (el, ev, fn) => {
  if (el.addEventListener) {
    el.addEventListener(ev, fn, false);
  } else if (el.attachEvent) {
    el.attachEvent(`on${ev}`, fn);
  } else {
    el[`on${ev}`] = fn;
  }
};
```

## 믹스인 패턴
**믹스인 패턴**은 여러 클래스 간에 **공통 기능을 재사용**할 수 있도록 **여러 클래스에 기능을 섞어주는 패턴**이다.

### 믹스인 패턴 예제
```js
const Flyable = {
  fly() {
    console.log("I'm flying!");
  }
};

const Swimmable = {
  swim() {
    console.log("I'm swimming!");
  }
};

class Animal {}

Object.assign(Animal.prototype, Flyable, Swimmable);

const duck = new Animal();
duck.fly();   // 출력: I'm flying!
duck.swim();  // 출력: I'm swimming!
```

## 서브클래싱
**서브 클래스**는 부모 클래스를 확장하는, 즉, **부모 클래스를 상속받은 자식 클래스**를 뜻한다.
**서브클래싱**이란 부모 클래스의 속성과 메서드를 상속받아 새로운 클래스를 정의하는 것을 의미한다. 즉, 부모 클래스를 기반으로 기능을 확장하거나 재정의하여 자식 클래스를 만드는 것이다.

### 서브클래싱 예제
Person 클래스를 상속한 Superhero 서브클래스를 정의하는 **서브클래싱(subclassing)** 예제이다.
```js
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.gender = "male";
  }
}

// Person의 새 인스턴스는 이처럼 쉽게 생성됩니다.
const clark = new Person('Clark', 'Kent');

// 서브 클래스 생성 (서브클래싱)
class Superhero extends Person {
  constructor(firstName, lastName, powers) {
    // 부모 클래스의 생성자를 호출합니다.
    super(firstName, lastName);
    this.powers = powers;
  }
}

// Superhero 인스턴스를 만듭니다.
const SuperMan = new Superhero('Clark', 'Kent', ['flight', 'heat-vision']);
console.log(SuperMan);

// power를 가진 Person을 출력합니다.
```


## 믹스인
자바스크립트에서는 기능의 확장을 위해 믹스인의 상속을 이용한다.
믹스인은 최소한의 복잡성으로 객체의 기능을 빌리거나 상속할 수 있게 한다.

자바스크립트의 클래스는 부모 클래스를 하나만 가질 수 있지만 여러 클래스 기능을 섞는 믹스인 으로 해결할 수 있다.

### 믹스인 예제
```js
// superclass에 기능을 붙여준다.
const MyMixins = (superclass) =>
  class extends superclass {
    moveUp() {
      console.log('move up');
    }

    moveDown() {
      console.log('move down');
    }

    stop() {
      console.log('stop! in the name of love!');
    }
  };
  
class MyAnimator extends MyMixin(CarAnimator){}
```

### 장점
- **중복을 줄이고 재사용성을 높임**
- 여러 객체 인스턴스 간 기능을 공유해 **유지보수와 기능 구현을 단순화**할 수 있음

### 단점
- 프로토타입에 직접 기능을 주입하면 함수나 메서드의 **출처가 불확실해짐**
- 코드의 흐름과 구조가 복잡해지고 **예측하기 어려운 동작**이 발생할 수 있음


## 데코레이터 패턴
**데코레이터 패턴**은 기존 객체에 **기능을 동적으로 추가하면서도**, **객체의 구조를 변경하지 않고 확장**할 수 있게 해주는 구조 패턴이다. 보통 객체를 감싸는 wrapping 방식으로 사용한다.

주로 기능 확장이 잦거나 다양한 조합의 객체가 필요한 경우, 기존 객체에 여러 기능을 조합해서 추가하고 싶을 때 사용한다.
ex) 게임에서 아이템 등급 올라갈 수록 기능이 추가되는 경우..

### 데코레이터 패턴 예제
```js
class Coffee {
  cost() {
    return 3;
  }
}

class MilkDecorator {
  constructor(coffee) {
    this.coffee = coffee;
  }
  cost() {
    return this.coffee.cost() + 1;
  }
}

class SugarDecorator {
  constructor(coffee) {
    this.coffee = coffee;
  }
  cost() {
    return this.coffee.cost() + 0.5;
  }
}

// 사용
let myCoffee = new Coffee();
myCoffee = new MilkDecorator(myCoffee);
myCoffee = new SugarDecorator(myCoffee);

console.log(myCoffee.cost()); // 출력: 4.5
```

### 인터페이스
『**Pro JavaScript Design Patterns**』에서는 데코레이터 패턴을 **같은 인터페이스를 가진 객체 안에 새 객체를 넣어 사용하는 방식**, 즉 **같은 형태를 유지하면서 기능을 점진적으로 확장하는 방식**으로 소개하고 있다.
여기서 **인터페이스는 문서화의 역할을 하며**, **재사용성과 코드의 안정성**을 높이는 데 중요한 역할을 한다.

**Interface Class 코드**
https://github.com/Apress/pro-javascript-design-patterns/blob/master/Source%20Code/Chapter02/2.05%20-%20The%20Interface%20class.js

**Typescript면 그냥 이렇게**
```ts
interface MacBook {
  addCase(): void;
  getPrice(): number;
}

class MacBookPro implements MacBook {
  addCase() {
    console.log('케이스 추가');
  }

  getPrice() {
    return 1000;
  }
}
```

### 추상 데코레이터
**데코레이터 클래스들의 공통적인 기반(부모 클래스) 역할**을 한다. 이 클래스는 원래 객체를 감싸고, **인터페이스를 그대로 유지하면서 기능을 위임**하는 틀을 제공한다.

데코레이터마다 중복되는 코드 (예: this.macbook, getPrice(), addCase() 등)를 **공통화**하고 구조를 강제한다.

```js
// MacBook 추상 데코레이터 클래스
class MacBookDecorator {
  constructor(macbook) {
    Interface.ensureImplements(macbook, MacBook);
    this.macbook = macbook;
  }

  // ...

  addCase() {
    return this.macbook.addCase();
  }

  getPrice() {
    return this.macbook.getPrice();
  }
}

// MacBookDecorator를 사용해 CaseDecorator를 확장
class CaseDecorator extends MacBookDecorator {
  constructor(macbook) {
    super(macbook);
  }

  addCase() {
    return `${this.macbook.addCase()}Adding case to macbook`;
  }

  getPrice() {
    return this.macbook.getPrice() + 45.0;
  }
}

// 맥북 인스턴스 생성
const myMacBookPro = new MacBookPro();
// 출력: 900.00
console.log(myMacBookPro.getPrice());

// 맥북 데코레이터 추가
const decoratedMacBookPro = new CaseDecorator(myMacBookPro);
// 945.00 반환
console.log(decoratedMacBookPro.getPrice());
```

여기서 MacBookDecorator는 기능을 직접 구현하지 않지만, 내부에 있는 macbook에게 **모든 기능을 위임** 한다. 이후 CaseDecorator, RamDecorator 같은 구체 데코레이터들이 이 클래스를 **extends**해서 사용한다. 
(**기존 기능은 그대로 유지하면서**, 필요한 데코레이터에서 **일부 메서드만 오버라이드해서 확장**하여 사용한다.)

### 장점
수많은 서브클래스에 의존할 필요가 없다.
👉 기능을 필요할 때 데코레이터로 조합하면 되므로 **유연하고 확장성이 높다.**

### 단점
비슷한 객체가 네임스페이스에 반복적으로 추가되기 때문에, 적절한 관리가 없으면 구조가 복잡해지고 유지보수가 어려워질 수 있다.


## 플라이웨이트 패턴
플라이웨이트 패턴은 반복적이고 비효율적인 데이터 사용을 최적화하기 위한 구조적 설계 방식이다.
연관된 객체들이 데이터를 공유하도록 하여, **메모리 사용을 최소화**하는 것을 목표로 한다.

**내재적 정보**는 변하지 않고 **공유**할 수 있는 값이며 플라이웨이트 객체 내부에 존재한다.
**외재적 정보**는 매번 바뀌며 **외부에서 개별적으로 주입**되는 값이다.


### 플라이웨이트 패턴 예제
책에 대한 인스턴스를 책 대출이 일어날 때 마다 계속 생성하면 메모리에 부담을 주고 성능 저하로 이어진다.
플라이웨이트 패턴을 이용하여 메모리 문제를 최적화 할 수 있다.

**책 대출 관리 시스템**
```js
// Book: 플라이웨이트 적용 객체
class Book {
  constructor({ title, author, genre, pageCount, publisherID, ISBN }) {
    this.title = title;
    this.author = author;
    this.genre = genre;
    this.pageCount = pageCount;
    this.publisherID = publisherID;
    this.ISBN = ISBN;
  }
}

// BookFactory: 고유한 내부 데이터에 대해 하나의 복사본만 생성되게 하는 플라이웨이트 팩토리
const existingBooks = {};

class BookFactory {
  createBook({ title, author, genre, pageCount, publisherID, ISBN }) {
    const existingBook = existingBooks[ISBN];
    if (!!existingBook) {
      return existingBook;
    } else {
      const book = new Book({ title, author, genre, pageCount, publisherID, ISBN });
      existingBooks[ISBN] = book;
      return book;
    }
  }
}


// BookRecordManager: 외부 데이터의 상태를 관리하는 싱글톤 관리자
const bookRecordDatabase = {};

class BookRecordManager {
  addBookRecord({ id, title, author, genre, pageCount, publisherID, ISBN,
                  checkoutDate, checkoutMember, dueReturnDate, availability }) {
    const bookFactory = new BookFactory();
    const book = bookFactory.createBook({ title, author, genre, pageCount, publisherID, ISBN });

    bookRecordDatabase[id] = {
      checkoutMember,
      checkoutDate,
      dueReturnDate,
      availability,
      book,
    };
  }

  updateCheckoutStatus({ bookID, newStatus, checkoutDate, checkoutMember, newReturnDate }) {
    const record = bookRecordDatabase[bookID];
    record.availability = newStatus;
    record.checkoutDate = checkoutDate;
    record.checkoutMember = checkoutMember;
    record.dueReturnDate = newReturnDate;
  }

  extendCheckoutPeriod(bookID, newReturnDate) {
    bookRecordDatabase[bookID].dueReturnDate = newReturnDate;
  }

  isPastDue(bookID) {
    const currentDate = new Date();
    return currentDate.getTime() > Date.parse(bookRecordDatabase[bookID].dueReturnDate);
  }
}
```

책 자체에 대한 정보는 **내재적 정보**로, 모든 인스턴스에서 **공통되고 변하지 않는 값**이다.
반면, 책의 대출 상태처럼 사용자에 따라 달라지는 정보는 **외재적 정보**에 해당한다.

Book 인스턴스는 **ISBN을 기준으로 단 하나만 생성**되며, 이후 동일한 책 정보가 필요한 경우 해당 인스턴스를 **공유**한다. 이를 통해 **중복 객체 생성을 방지하고**, **메모리를 효율적으로 사용할 수 있다**.


**DOM 이벤트 중앙 제어**
```js
<div id="container">
  <div class="toggle">
    More Info (Address)
    <span class="info" style="display: none">
      This is more information
    </span>
  </div>
  <div class="toggle">
    Even More Info (Map)
    <span class="info" style="display: none">
      <iframe src="MAPS_URL"></iframe>
    </span>
  </div>
</div>

<script>
  const stateManager = {
    fly() {
      document
        .getElementById('container')
        .addEventListener('click', (event) => {
          const toggleDiv = event.target.closest('.toggle');
          if (toggleDiv) {
	        // 여러개의 이벤트에 여러함수를 등록하지 않아, 메모리 효율이 올라간다.
            this.handleClick(toggleDiv);
          }
        });
    },

    handleClick(elem) {
      const span = elem.querySelector('span');
      if (span) {
        span.style.display =
          span.style.display === 'none' ? 'block' : 'none';
      }
    },
  };
  // 이벤트 등록
  stateManager.fly();
</script>
```
**중앙 집중식 이벤트 위임은** 플라이웨이트 패턴처럼 **공통 로직(핸들러)을 공유함으로써** **메모리 절약, 성능 향상, 유지보수 효율**을 얻는 구조적 장점이 있다.


# 행위 패턴
행위 패턴은 객체간의 의사소통을 돕는 패턴이다.
## 관찰자 패턴
관찰자 패턴(Observer Pattern)은 **어떤 객체의 상태 변화**가 있을 때, **그에 의존하는 다른 객체들에게 자동으로 알림을 보내는 방식**을 말한다. 주로 **이벤트 기반 시스템**에서 많이 사용된다.

###  핵심 개념

- **주체 Subject
	- 상태를 가지고 있으며, 상태가 바뀌면 관찰자들에게 알림을 보냄
	- 관찰자 리스트를 관리하고, 추가/삭제를 가능하게 함
	- → 예: 유튜브 채널
    
- **관찰자 Observer**
	- 주체를 구독하고 있다가, 변화가 생기면 알림을 받고 대응
	- 주체의 상태 변화 알림을 감지하는 update 인터페이스를 제공
	- → 예: 유튜브 구독자

### 예제
```js
class Subject {
  constructor() {
    this.observers = [];
  }

  subscribe(observer) {
    this.observers.push(observer);
  }

  notify(data) {
    this.observers.forEach(observer => observer.update(data));
  }
}

class Observer {
  update(data) {
    console.log("업데이트 수신:", data);
  }
}

const subject = new Subject();
const observer = new Observer();

subject.subscribe(observer);
subject.notify("새로운 이벤트");
```

### 왜 사용해야하는가?
결합도를 낮추고 유연함을 얻기 위함

### 사용 예시
#### IntersectionObserver
```js
const observer = new IntersectionObserver((entries) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting) {
      console.log('요소가 보이기 시작했습니다!');
    }
  });
});

observer.observe(document.querySelector('#target'));
```
- **Subject**: DOM 요소의 위치를 연산하는 브라우저 → 뷰포트와의 교차 상태가 변화하면 알림!
- **Observer**: observer에 등록된 callback!? → 이게 update 함수 같은 느낌!
- **Notification**: 요소가 뷰포트에 들어오거나 나갈 때 자동으로 콜백(callback)이 호출됨

전통적인 구조와는 조금 다르지만, 주체/관찰자 관계가 있는 관찰자 패턴의 개념을 그대로 따른다.
브라우저가 레이아웃 계산 이후에 DOM 요소의 위치를 확인하고 교차여부를 판단하고 콜백을 호출한다.

#### tanstack query
https://lurgi.tistory.com/170
https://tech.kakaoent.com/front-end/2023/230720-react-query/

**tanstack query에서 관찰자 함수를 실행하는 부분**
https://github.com/TanStack/query/blob/main/packages/query-core/src/query.ts#L624


## 발행/구독 패턴
**발행/구독 패턴**은 시스템에서 **이벤트를 발생시키는 주체(발행자, Publisher)** 와 **그 이벤트에 반응하는 주체(구독자, Subscriber)** 사이를 **직접 연결하지 않고**, **중간 매개체(Event Broker)** 를 통해 간접적으로 연결하는 구조입니다.

이때 발행자와 구독자는 서로를 **직접 알지 못하며**, **중간 매개자(Broker 또는 Event Bus)**가 메시지를 중계합니다.

### 핵심 개념
- **발행자 Publisher**
	- 어떤 이벤트가 발생했는지를 알리는 주체
	- 구독자가 누군지, 몇 명인지, 어떤 방식으로 처리하는지는 신경 쓰지 않음
- **구독자 Subscriber**
	- 관심 있는 이벤트를 미리 등록해두고(event subscribe), 해당 이벤트가 발생하면 알림을 받아 처리
	- 발행자가 누구인지는 몰라도됨
- **중재자 Event Broker
	- 발행자와 구독자 사이의 중재자
	- 이벤트 발생 시 어떤 구독자에게 알릴지를 관리

### 예제
```js
class EventBus {
  constructor() {
    this.events = {};
  }

  subscribe(event, fn) {
    (this.events[event] ||= []).push(fn);
  }

  publish(event, data) {
    (this.events[event] || []).forEach(fn => fn(data));
  }
}

const bus = new EventBus();

bus.subscribe("news", data => {
  console.log("뉴스 수신:", data);
});

bus.publish("news", "GPT-4o 출시");
```
 
### 사용 예시
#### JavaScript CustomEvent + DOM EventTarget
```js
const bus = new EventTarget();

bus.addEventListener('event:login', (e) => {
  console.log('로그인 이벤트 수신:', e.detail);
});

bus.dispatchEvent(new CustomEvent('event:login', { detail: { user: 'admin' } }));
```

#### kafka
```js
// Publisher
kafkaProducer.send({ topic: 'orders', messages: [{ value: 'order1' }] });

// Consumer
kafkaConsumer.subscribe({ topic: 'orders' });
kafkaConsumer.run({
  eachMessage: async ({ message }) => {
    console.log('Received:', message.value.toString());
  }
});
```


### 관찰자 패턴 vs 발행/구독 패턴

| **구분**       | **관찰자 패턴 (Observer)**                 | **발행/구독 패턴 (Pub/Sub)**    |
| ------------ | ------------------------------------- | ------------------------- |
| **알림 전달 방식** | 관찰 대상(Subject)이 직접 구독자(Observer)에게 알림 | 중개자(Broker)를 통해 간접적으로 전달  |
| **결합도**      | 비교적 높음 (주체가 구독자를 직접 알고 있음)            | 낮음 (주체와 구독자가 서로 모름)       |
| **구조**       | 1:N 직접 연결                             | N:M 간접 연결 (브로커를 통해)       |
| **유연성**      | 상대적으로 낮음                              | 매우 높음 (확장성 우수)            |
| **알림 흐름 제어** | 주체(Subject)가 제어                       | 중개자(Broker)가 제어           |

관찰자 패턴은 **단순하고 즉각적인 상태 변화 감지**가 필요하거나 객체간의 관계가 명확한 경우에 더 적합하다.
발행/구독 패턴은 **복잡하거나 분산된 시스템 간의 비동기 통신**에 더 어울린다.

발행/구독 패턴은 너무 유연해서 남발하는 경우 “누가 뭐에 반응하는지 추적이 어려워지는” 문제가 생길 수 있다.



## 중재자 패턴

**중재자 패턴(Mediator Pattern)** 은 **행동(Behavioral) 디자인 패턴** 중 하나로, 객체들 간의 **직접적인 의존 관계를 제거하고**, 객체들 사이의 **상호작용을 중재자 객체(Mediator)** 를 통해 처리하게 하는 패턴이다.


### 핵심 개념
객체들 사이의 복잡한 **N:N 통신을**, 하나의 중재자를 통해 **1:N 통신으로 단순화**하는 패턴

- **동료 객체 Component**: 서로 통신하려는 객체들
- **중재자 Mediator**: 각 컴포넌트들의 요청을 받아 **다른 컴포넌트에게 알림을 전달**하는 중개자


### 예제
```js
class UIComponent {
  constructor(name) {
    this.name = name;
    this._mediator = null;
  }

  setMediator(mediator) {
    this._mediator = mediator;
  }

  notify(event, payload) {
    if (!this._mediator) {
      throw new Error("Mediator not set for component " + this.name);
    }
    this._mediator.notify(this.name, event, payload);
  }
}

class UIMediator {
  constructor() {
    this.components = new Map(); // name → component
  }

  register(name, component) {
    this.components.set(name, component);
    component.setMediator(this);
  }

  get(name) {
    return this.components.get(name);
  }

  notify(senderName, event, payload) {
    const key = `${senderName}:${event}`;
    switch (key) {
      case "button:click":
        // 중재자를 통해 다른 UI 컴포넌트의 동작을 함께 관리한다.
        this.get("textBox").clear();
        this.get("checkBox").uncheck();
        break;
        
      case "checkBox:check":
        this.get("button").enable();
        break;
        
	  // ...
    }
  }
}

// Concrete UI components
class Button extends UIComponent {
 constructor(name) {
	super(name);
    this.enabled = true;
  }
  click() {
    this.notify("click");
  }
  disable() {
    this.enabled = false;
  }
  enable() {
    this.enabled = true;
  }
}

class TextBox extends UIComponent {
  constructor(name) {
    super(name);
    this.value = "초기값";
  }

  clear() {
    this.value = "";
  }

  input(value) {
    this.value = value;
    this.notify("input", value);
  }
}

class CheckBox extends UIComponent {
  constructor(name) {
    super(name);
    this.checked = false;
  }

  check() {
    if (!this.checked) {
      this.checked = true;
      this.notify("check");
    }
  }

  uncheck() {
    if (this.checked) {
      this.checked = false;
    }
  }
}

// 사용 예시
const mediator = new UIMediator();

const button = new Button("button");
const textBox = new TextBox("textBox");
const checkBox = new CheckBox("checkBox");

mediator.register("button", button);
mediator.register("textBox", textBox);
mediator.register("checkBox", checkBox);

checkBox.check();      // 중재자를 통해 button을 enable한다.
textBox.input("hello");
button.click();        // 중재자를 통해 textBox를 clear하고, checkBox를 체크 헤제한다.

```

```jsx
import React, { useState, createContext, useContext } from 'react';

const MediatorContext = createContext(null);

export function MediatorProvider({ children }) {
  const [text, setText] = useState('');
  const [checked, setChecked] = useState(false);

  const mediator = {
    text,
    setText,
    checked,
    setChecked,

    // 중재자 역할: 버튼 클릭 시 어떤 조치를 할지 판단
    handleButtonClick: () => {
      setText('');
      setChecked(false);
    }
  };

  return (
    <MediatorContext.Provider value={mediator}>
      {children}
    </MediatorContext.Provider>
  );
}

export const useMediator = () => {
  const context = useContext(MediatorContext);
  if (!context) throw new Error('useMediator must be used within MediatorProvider');
  return context;
};
```
### 왜 사용해야하는가?
결합도를 낮추고 유연함을 얻기 위함
구성 요소 재사용 가능하게함
단일 책임 원칙 준수할 수 있음!

중재자를 안쓰면 코드 로직이 복잡해진다..
예시는 간단하지만 서로가 서로를 호출하게 되면 가독성도 안좋아지고 디버깅이 어려워진다.

```js
class Button {
  constructor(textBox, checkBox) {
    this.textBox = textBox;
    this.checkBox = checkBox;
  }
  click() {
    // 다른 컴포넌트를 직접 조작
    this.textBox.clear();
    this.checkBox.uncheck();
  }
}

// 각 객체가 서로를 복잡하게글 직접 참조함
const textBox = new TextBox();
const button = new Button(textBox, null);
const checkBox = new CheckBox(button);
button.checkBox = checkBox;
```




