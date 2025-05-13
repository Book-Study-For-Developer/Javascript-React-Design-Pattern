## 7.2 생성자 패턴

> 유사한 객체를 여러 개 생성할 때 사용하는 디자인 패턴
> 객체 생성 로직을 하나의 생성자 함수(또는 클래스)에 정의하고, `new` 키워드로 인스턴스를 생성하는 방식

- 모듈 패턴
- 노출 모듈 패턴
- 싱글톤 패턴
- 프로토타입 패턴
- 팩토리 패턴

### 7.2.1 객체 생성

```jsx
// 방법 1: 리터럴 표기법
const newObject = {};

// 방법 2: Object.create() 메서드
const newObject = Object.create(Object.prototype);

// 방법 3: new 키워드를 사용하여 빈 객체 생성
const newObject = new Object();

// -----------------------------------------

// ECMAScript 3 호환 방식
newObject.someKey = 'Hello World';

var key = newObject.someKey;

newObject['someKey'] = 'Hello World';

// ECMAScript 5 호환 방식
Object.defineProperty(newObject, 'someKey', {
  value: '~~~~', // 기본값 undefined
  writable: true, // 기본값 false
  enumerable: true, // 기본값 false
  configurable: true, // 기본값 false
});

var defineProp = function (obj, key, value) {
  config.value = value;
  Object.defineProperty(obj, key, config);
};

var person = Object.create(null);

defineProp(person, 'car', 'Delorean');

Object.defineProperties(newObject, {
  someKey: {
    value: 'Hello',
    writable: true,
  },
  anotherKey: {
    value: 'Foo bar',
    writable: false,
  },
});
```

### 7.2.2, 7.2.3 클래스

```jsx
class Car {
  constructor(model, year, miles) {
    this.model = model;
    this.year = year;
    this.miles = miles;
  }
  // 작성 1. 메서드
  toString() {
    return `${this.model} has done ${this.miles} miles`;
  }
}

// 작성 2. 프로토타입 메서드
Car.prototype.toString = function () {
  return `${this.model} has done ${this.miles} miles`;
};

let civic = new Car('Honda Civic', 2009, 20000);

console.log(civic.toString());
```

## 7.3 모듈 패턴

- 프로젝트를 구성하는 코드 단위를 체계적으로 분리 및 관리하는 데 효과적으로 활용
- ES2019 이전까지는 접근 제한자(#, 해시)를 지원하지 않아 ‘비공개’라는 개념이 존재하지 않았다.
- 모듈 패턴은 **클로저**를 활용해 ‘비공개’ 상태와 구성을 캡슐화한다. 즉, 공개 및 비공개 메서드와 변수를 묶어 전역 스코프로의 유출을 방지한다.
- 즉시 실행 함수(IIFE)를 사용해 객체를 반환한다. (11장에서 자세히 다룸)
- 객체에 포함된 변수를 비공개하려면 `WeakMap()`을 사용하면 된다.

```jsx
// 전역 스코프로부터 보호되는 '비공개' 변수
let counter = 0;

const testModule = {
  increase() {
    return counter++;
  },
  reset() {
    console.log(`counter value prior to reset: ${counter}`);
    counter = 0;
  },
};

export default testModule;
```

### 7.3.3 모듈 패턴의 변형

믹스인 가져오기 변형

```jsx
// utils.js
export const min = (arr) => Math.min(...arr);

// privateMethods.js
import { min } from './utils';

export const privateMethod = () => {
  console.log(min([10, 5, 100, 2, 1000]));
};

// myModule.js
import { privateMethod } from './privateMethods';

const myModule = () => ({
  publicMethod() {
    privateMethod();
  },
});

export default myModule;

// main.js
import myModule from './myModule';

const moduleInstance = myModule();
moduleInstance.publicMethod();
```

> 믹스인 가져오기 패턴: 외부 유틸 함수나 라이브러리, 전역 객체에 직접 의존하지 않고, **고차 함수의 인자로 필요한 의존성을 외부에서 주입받아 사용하는 모듈 구성 방식**

✨ 모듈 패턴의 장점: 은닉성, 메모리 효율, 사용 편의성

💦 모듈 패턴의 단점: 수정할 때 복잡도를 높임 (private 멤버 수정 힘듦)

### 7.3.4 WeakMap을 사용하는 최신 모듈 패턴

> 객체를 키로만 사용할 수 있는 Map이며, 그 키 객체가 더 이상 참조되지 않으면 자동으로 GC(Garbage Collection) 대상이 됨

- 🧳용도: 비공개 데이터 저장, DOM 요소 메타데이터 연결 등

```jsx
let _counter new WeakMap();

class Module {
	constructor() {
		_counter.set(this, 0);
	}
	increase() {
		let counter = _counter.get(this);
		counter++;
		_counter.set(this, counter);

		return _counter.get(this);
	}
	reset() {
		console.log(`counter value prior to reset: ${_counter.get(this)}`);
		_counter.set(this, 0);
	}
}

const testModule =new Module();

testModule.increase();
testModule.reset();
```

| 항목    | `Map`                    | `WeakMap`                     |
| ------- | ------------------------ | ----------------------------- |
| 키      | 객체 + 원시값            | 객체만                        |
| 순회    | 가능 (`forEach`, `keys`) | ❌ 불가능                     |
| GC 연동 | ❌ 수동으로 삭제 필요    | ✅ 객체 참조 끊기면 자동 삭제 |
| 용도    | 일반적 데이터 저장       | 비공개/임시/메타 데이터 저장  |

## 7.4 노출 모듈 패턴

> 모든 함수와 변수를 비공개 스코프에 정의하고, 공개하고 싶은 부분만 포이넡를 통해 비공개 요소에 접근할 수 있게 해주는 익명 객체를 반환하는 패턴

- 내부에 정의된 함수나 변수 중에서 **공개하고 싶은 것만 명확하게 return 객체에 담아 외부에 노출**
- ES2015+에서는 export와 import로 공개 여부를 결정

✨ 장점: 코드의 일관성 유지, 가독성 향상

💦 단점: 비공개 변수를 참조하는 공개 함수, 객체 멤버 수정 불가

## 7.5 싱글톤 패턴

> 클래스의 인스턴스가 오직 하나만 존재하도록 제한하는 패턴

- 이미 존재하는 인스턴스가 없어야 한다.

```jsx
let instance;

const privateMethod = () => {
  console.log('I am private');
};
const privateVariable = 'Im also private';
const randomNumber = Math.random();

class MySingleton {
  constructor() {
    if (!instance) {
      this.publicProperty = 'I am also public';
      instance = this;
    }

    return instance;
  }

  publicMethod() {
    console.log('공개');
  }

  getRandomNumber() {
    return randomNumber;
  }
}

export default MySingleton;
```

📙 “GoF의 디자인 패턴” 싱글톤 패턴의 적합성

- 클래스의 인스턴스는 하나만 있어야 하며 접근을 용이하게 해야 합니다.
- 싱글톤의 인스턴스는 서브클래싱을 통해서만 확장할 수 있어야 합니다.

### “지연된 실행”이란?

> 객체를 미리 생성해두는 것이 아니라, 그 객체가 처음으로 필요할 때 생성하는 방식

✨ 싱글톤을 정적 인스턴스로 구현했다 하더라도, 필요할 때 까지는 리소스나 메모리를 소모하지 않도록 지연 생성될 수 있다.

### ☠️ 자바스크립트에서의 싱글톤

- 설계를 다시 생각해야 한다.
- 싱글톤임을 파악하는 것이 힘들다.
- 테스트하기 힘들다.
- 애플리케이션 크기가 커지고 복잡해질수록 어렵다.

### 7.5.1 리액트의 상태 관리

✨ 싱글톤 ⇒ Context API, 리덕스 같은 전역 상태 관리 도구를 활용하면 된다.

## 7.6 프로토타입 패턴

> 이미 존재하는 객체를 복제해 만든 템플릿을 기반으로 새 객체를 생성하는 패턴

- Object.create 활용
- class extends 활용

## 7.7 팩토리 패턴

> 생성자를 필요로 하지 않지만, 필요한 타입의 팩토리 객체를 생성하는 다른 방법을 제공한다.
> `new` 키워드를 사용하지 않음!

(책의 예제는 클래스지만 GPT에게 쉬운 예제 만들어 달라고 함 ..)

```jsx
function createUser(role) {
  if (role === 'admin') {
    return { role, permissions: ['read', 'write', 'delete'] };
  } else if (role === 'guest') {
    return { role, permissions: ['read'] };
  } else {
    return { role, permissions: ['read', 'write'] };
  }
}

const admin = createUser('admin');
const guest = createUser('guest');

console.log(admin); // { role: 'admin', permissions: [ 'read', 'write', 'delete' ] }
```

- 어떤 클래스(혹은 타입)의 인스턴스를 만들지 동적으로 결정할 때 편리

✨ 장점: 생성 과정이 복잡할 때, 다양한 상황에 맞춰 편리하게 생성할 때 용이하다.

💦 단점: 애플리케이션의 복잡도가 크게 증가

### 7.7.3 추상 팩토리 패턴

> 같은 목표를 가진 각각의 팩토리들을 하나의 그룹으로 캡슐화 하는 패턴

- 객체가 어떻게 생성되는지에 대한 세부사항을 알 필요 없다.

```jsx
class AbstractVehicleFactory {
  constructor() {
    this.types = {};
  }

  getVehicle(type, customizations) {
    const Vehicle = this.types[type];
    return Vehicle ? new Vehicle(customizations) : null;
  }

  registerVehicle(type, Vehicle) {
    const proto = Vehicle.prototype;
    if (proto.drive && proto.breakDown) {
      this.types[type] = Vehicle;
    }
    return this;
  }
}
```

---

## 7.8 구조 패턴

> 클래스와 객체의 구성을 다룬다.

- 퍼사트 패턴
- 믹스인 패턴
- 데코레이터 패턴
- 플라이웨이트 패턴

## 7.9 퍼사드 패턴

> 퍼사드는 실제 모습을 숨기고 꾸며낸 겉모습만을 세상에 드러내는 것을 말한다.
> 심층적인 복잡성을 숨기고, 사용하기 편리한 높은 수준의 인터페이스를 제공하는 패턴

- jQuery 같은 자바스크립트 라이브러리에서 흔히 볼 수 있는 구조 패턴
- 제한된 추상화 메서드만이 공개되어 사용할 수 있도록 한다.
- 사용자는 구현 수준의 세부사항을 알지 않고도 쉽게 사용할 수 있다.

(예시는 jQuery의 ready 메서드지만 … 사용해본 적이 없는 관계로)

```jsx
class VideoPlayerFacade {
  constructor() {
    this.loader = new FileLoader();
    this.decoder = new Decoder();
    this.player = new VideoPlayer();
  }

  playVideo(filename) {
    this.loader.load(filename);
    this.decoder.decode(filename);
    this.player.play();
  }
}

const videoPlayer = new VideoPlayerFacade();
videoPlayer.playVideo('movie.mp4'); // 간단하게 실행
```

## 7.11 서브클래싱

> 부모 클래스 객체에서 속성을 상속받아 새로운 객체를 만드는 것

```jsx
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = laseName;
    this.gender = 'male';
  }
}

const clark = new Person('Clark', 'Kent');

class SuperHero extends Person {
  constructor(firstName, lastName, powers) {
    super(firstName, lastName);
    this.powers = powers;
  }
}

const SuperMan = new Superhero('Clark', 'Kent', ['flight', 'heat-vision']);
console.log(SuperMan);
```

## 7.12 믹스인

자바스크립트에서는 기능의 확장을 위해 믹스인의 상속을 이용한다.

자바스크립트의 클래스는 부모 클래스를 하나만 가질 수 있지만 여러 클래스의 기능을 섞는 것으로 문제를 해결할 수 있다.

```jsx
// 동적으로 부모 클래스를 받아 확장할 수 있다.
const MyMixins = (superclass) =>
  class extends superclass {
    moveup() {
      console.log('move up');
    }
    moveDown() {
      console.log('move down');
    }
  };

class MyAnimator extends MyMixins(CarAnimator) {}
```

✨ 장점: 함수의 중복을 줄이고 재사용성을 높인다.

💦 단점: 리액트의 경우 컴포넌트의 유지보수와 재사용을 복잡하게 만드므로 고차 컴포넌트나 Hooks를 사용하는 게 좋다.

## 7.13 데코레이터 패턴

> 코드 재사용을 목표로 하는 구조 패턴

- 기존 시스템의 내부 코드를 힘겹게 바꾸지 않고도 기능을 추가할 수 있게 된다.
- 책의 맥북 예제와 비슷한 GPT 예제

```jsx
// 기본 컴포넌트
class Coffee {
  cost() {
    return 5;
  }
}

// 데코레이터 1
class MilkDecorator {
  constructor(coffee) {
    this.coffee = coffee;
  }

  cost() {
    return this.coffee.cost() + 2; // 우유 추가 비용
  }
}

// 데코레이터 2
class SugarDecorator {
  constructor(coffee) {
    this.coffee = coffee;
  }

  cost() {
    return this.coffee.cost() + 1; // 설탕 추가 비용
  }
}

// 사용
let coffee = new Coffee();
coffee = new MilkDecorator(coffee); // 우유 추가
coffee = new SugarDecorator(coffee); // 설탕 추가

console.log(coffee.cost()); // 출력: 8 (5 + 2 + 1)
```

✨ 장점: 베이스 객체가 변경될 걱정 없이 사용할 수 있따.

💦 단점: 애플리케이션의 복잡성 증가

## 7.16 플라이웨이트 패턴

> 반복되고 느리고 비효율적으로 데이터를 공유하는 코드를 최적화하는 전통적인 구조적 해결 방법
> 연관된 객체끼리 데이터를 공유하게 하면서 애플레케이션의 메모리를 최소화하는 목적을 가지고 있다.

### 내재적 상태

객체의 내부 메서드에 필요하고, 없으면 절대로 동작하지 않는다.

### 외재적 상태

외부에 저장될 수 있다.

(도서관 책 관리 예시는 너무 길어서 읽기만 했습니다!)

### 7.16.7, 7.16.8 DOM 이벤트 핸들링

- 예시가 jQuery라서 이해를 잘 못하겠음 ..

---

## 7.17 행위 패턴

> 객체 간의 의사소통을 돕는 패턴

- 관찰자 패턴
- 중재자 패턴
- 커맨드 패턴

## 7.18 관찰자 패턴

> 한 객체가 변경될 때 다른 객체들에 변경되었음을 알릴 수 있게 해주는 패턴

주체: 관찰자 리스트를 관리하고, 추가와 삭제를 가능하게 한다.

관찰자: 주체의 상태 변화 알림을 감지하는 update 인터페이스를 제공한다.

구체적 주체: 상태 변화에 대한 알림을 모든 관찰자에게 전달하고, 구체적 주체의 상태를 저장한다.

구체적 관찰자: 구체적 주체의 참조를 저장하고, 관찰자의 update 인터페이스를 구현하여 주체의 상태 변화와 관찰자의 상태 변화가 일치할 수 있도록 한다.

```jsx
class ObserverList {
  constructor() {
    this.observerList = [];
  }

  add(obj) {
    return this.observerList.push(obj);
  }

  count() {
    return this.observerList.length;
  }

  get(index) {
    if (index > -1 && index < this.observerList.length) {
      return this.observerList[index];
    }
  }

  indexOf(obj, startIndex) {
    let i = startIndex;

    while (i < this.observerList.length) {
      if (this.observerList[i] === obj) {
        return i;
      }
      i++;
    }

    return -1;
  }

  removeAt(index) {
    this.observerList.splice(index, 1);
  }
}

class Subject {
  constructor() {
    this.observers = new ObserverList();
  }

  addObserver(observer) {
    this.observers.add(observer);
  }

  removeObserver(observer) {
    this.observers.removeAt(this.observers.indexOf(observer, 0));
  }

  notify(context) {
    const observerCount = this.observers.count();
    for (let i = 0; i < observerCount; i++) {
      this.observers.get(i).update(context);
    }
  }
}

// 관찰자 클래스
class Observer {
  constructor() {}
  update() {}
}
```

🥔 예전에 자바스크립트로 노션 클로닝 프로젝트 했을 때 다른 분들이 다른 영역에서의 데이터 공유를 구독 패턴, 프록시 패턴으로 구현하셨던 게 떠올랐어요.

### 7.18.1 관찰자 패턴과 발행/구독 패턴의 차이점

- 발행/구독 패턴은 중간 매개자가 존재한다. 발행자와 구독자를 각자 독립적으로 유지한다.

✨ 장점: 코드의 관리와 재사용성을 높일 수 있다. 시스템의 구성 요소 간 결합도를 낮추는 훌륭한 도구이다.

💦 단점: 구독자들이 서로의 존재에 대해 알 수가 없고, 어떤 구독자가 어떤 발행자에 의존하는지 추적하기 어려울 수 있다.

📌 [진짜 진짜 유용했던 블로그 글](https://junilhwang.github.io/TIL/Javascript/Design/Vanilla-JS-Store/)

</aside>

### 리액트 생태계에서의 관찰자 패턴

🔗 [RxJS](https://rxjs.dev/guide/overview): 관찰자 패턴, 이터레이터 패턴, 함수형 프로그래밍 개념을 결합하여 이벤트 시퀀스 관리

## 7.19 중재자 패턴

> 하나의 객체가 이벤트 발생 시 다른 여러 객체들에게 알림을 보낼 수 있는 디자인 패턴

쉬운 예시로 관제탑이 있다. 항공기끼리 직접 통신을 하게 하지 않고, 관제탑을 통해 항공기의 이착륙을 관리한다.

또한 DOM의 이벤트 위임 또한 document가 중재자 역할을 한다.

😭 이벤트 집합 패턴과 중재자 패턴의 차이를 이해하기 쉽지 않습니다..

| 항목        | 발행/구독 패턴                      | 중재자 패턴                        |
| ----------- | ----------------------------------- | ---------------------------------- |
| 목적        | **이벤트 기반 비동기 알림**         | **객체 간 복잡한 통신 조정**       |
| 연결 구조   | 발행자 ↔ (이벤트 허브) ↔ 구독자     | 모든 객체 ↔ 중재자                 |
| 메시지 흐름 | **브로드캐스트** (누가 들을지 모름) | **타겟 조절 가능** (중재자가 선택) |
| 객체 관계   | 발행자와 구독자 **완전 분리**       | 참여 객체가 **중재자에게 의존**    |

## 7.20 커맨드 패턴

> 메서드 호출, 요청 또는 작업을 단일 객체로 캡슐화하여 추후에 실행할 수 있도록 해준다.

추상 클래스를 상속받아 필요한 기능을 모두 구현한 클래스를 구체 클래스라고 한다.

커맨트 패턴의 기본 원칙은 **명령을 내리는 객체와 명령을 실행하는 객체의 책임을 분리**하는 것

그렇다면 이 책임을 어떻게 분리할까? ⇒ 다른 객체에 위임!

```jsx
const CarManager = {
  requestInfo(model, id) {
    return `The information for ${model} with ID ${id} is foobar`;
  },

  // ...
};
```

- CarManager 객체 내부의 핵심 API가 변경되면, 메서드를 직접 호출하는 모든 객체를 수정해야 한다. ‘객체 간의 느슨한 결합’ 원칙에 위배된다.

```jsx
carManager.execute = function(name) {
	return (
		carManager[name] &&
		carManager[name].apply(carManager, [].slice.call(arguments, 1))
	);
};

CarManager.execute('arrangeViewing', 'Ferrari', '14523');
// carManager.arrangeViewing("Ferrari", "14523");

// apply, call이 좀 어려워서 최신 버전으로 GPT한테 변환해달라고 해봤습니다.
execute(method, ...args) {
  return this[method]?.(...args);
}
```
