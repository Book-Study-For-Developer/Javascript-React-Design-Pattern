## 생성 패턴

### 생성자 패턴

객체가 새로 만들어진 뒤 초기화하는 데에 사용되는 특별한 메서드

객체를 생성하는 방법

```tsx
// 리터럴 표기법을 사용하여 빈 객체 생성
const newObject = {};

// Object.create() 메서드를 사용하여 빈 객체 생성
const newObject = Object.create(Object.prototype);

// n🤔w 키워드를 사용하여 빈 객체를 생성
const newObject = new Object();
```

객체에 키워 값을 할당하는 방법

```tsx
// 객체에 직접 할당하기
newObject.someKey = 'some value';
newObject['someKey']= 'some value';

// defineProperty 속성 활용
Object.defineProperty(newObject, 'someKey', {
	value: 'some value',
	// assignment 연산자를 통해 속성(property)를 바꿀 수 있음
	writable: true,
	// 열거할 수 있는 속성을 부여
	enumarable: true,
	configuarable: true,
})

// defineProperties 활용
Object.defineProperties(newObject, {
	'someKey': {
		value: 'some value',
		writable: true,
		enumarable: true,
		configuarable: true,
	},
	...
})
```

🤔 writable, enumarable, configuarable 속성 알아보기

**writable**

`writable`이 false이면 `obj.key = value` 방식으로는 값을 변경할 수 없다.

```tsx
const obj = {
  key1: "value1",
  key2: "value2",
};

Object.defineProperty(obj, "key1", {
  writable: false,
});

obj.key1 = "value3"; // 에러는 발생되지 않지만 value는 그대로
```

**configuarable**

**`configurable`** 속성은 객체에서 해당 속성의 삭제 가능 여부와 descriptor의 다른 속성들의 변경 가능 여부를 결정

**enumarable**

`enumerable`이 false이면 `Object.keys`나 `for...in`을 사용하여 **객체의 키를 열거할 수 없다**. 심지어 단순히 객체를 출력만 할 때에도 해당 속성이 보이지 않는다. 하지만 객체 내에서 완전히 사라진 것은 아니기 때문에 `obj.key`와 같이 직접 접근할 수는 있다.

```tsx
const obj = {
  key1: "value1",
  key2: "value2",
};

Object.defineProperty(obj, "key1", {
  enumerable: false,
});

console.log(Object.keys(obj)); // ['key2']
```

### 생성자의 특징

간단한 생성자 패턴을 적용한 Car 클래스이다.

```tsx
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

// 인스턴스 생성
const civic = new Car("Honda Civic", 2009, 2009);
const mondeo = new Car("Ford Mondeo", 2010, 2010);
```

- 상속이 어려워짐
- Car 생성자로 객체를 생성할 때 마다 toString()과 같은 함수를 새로 정의
  - 모두 동일한 함수를 공유하기 때문에 효과적이지 않다.

프로토타입 객체를 통해 새롭게 생성되는 인스턴스에도 동일한 속성을 부여할 수 있다.

```tsx
Car.prototype.toString = function () {
  return `${this.model} has done ${this.miles} miles`;
};
```

🤔 프로토타입을 통해 정의하면 오히려 코드의 위험도가 올라갈 것 같아서 효과적이지 않는 방법일지라도 클래스내에 함수를 선언해서 쓸 듯 합니다.

## 모듈 패턴

모듈은 프로젝트를 구성하는 코드 단위를 분리 및 관리하는데 효과적으로 활용되는 요소이다. (흔히 컴포넌트 분리하는 것들도 여기에 해당)

**객체 리터럴 표기법을 활용하여 모듈을 정의**

```tsx
const myModule = {
	myProperty: 'someValue',
	myConfig: {
		useCashing: true,
		language: 'en',
	},
	...
}
```

객체 리터럴에 대한 글:

https://github.com/rmurphey/rmurphey/blob/master/public/blog/using-objects-to-organize-your-code.md

- 객체 리터럴을 사용하면 코드를 캡슐화가 가능
- 모듈 패턴은 클로저를 활용하여 ‘private’ 상태로 캡슐화를 진행
  - public, private을 활용하여 전역 스코프로의 유출을 방지한다.
- 클로저를 통해 비공개를 구현하기 때문에 반환되는 객체의 경우에는 공개되어 다른 곳에서도 사용이 가능
  - 반환되는 객체까지 비공개로 하기 위해서는 WeakMap을 사용하면 된다.
  - WeakMap은 객체만 키로 설정할 수 있어 순회가 불가능하기에 객체의 참조를 통해서만 접근이 가능하다.

```tsx
const privateData = new WeakMap<object, { secret: string }>();

class SecretBox {
  // WeakMap을 활용하여 반환되는 객체로 직접 꺼내올 수는 없다.
  constructor(secret: string) {
    privateData.set(this, { secret });
  }

  revealSecret() {
    const data = privateData.get(this);
    return data?.secret;
  }
}

const box = new SecretBox("비밀 메시지");

console.log(box.secret); // undefined
console.log(box.revealSecret()); // "비밀 메시지"
console.log(privateData.get(box)); // { secret: '비밀 메시지' } (WeakMap에 직접 접근해야 알 수 있음)
```

**모듈 패턴 예시 살펴보기**

```tsx
// https://github.com/addyosmani/learning-jsdp/blob/main/ch07/modules/basket.mjs

const basket = [];

const doSomethingPrivate = () => {
  //...
};

const doSomethingElsePrivate = () => {
  //...
};

// Create an object exposed to the public
const basketModule = {
  // Add items to our basket
  addItem(values) {
    basket.push(values);
  },

  // Get the count of items in the basket
  getItemCount() {
    return basket.length;
  },

  // Public alias to a private function
  doSomething() {
    doSomethingPrivate();
  },

  // Get the total value of items in the basket
  // The reduce() method applies a function against an accumulator and each
  // element in the array (from left to right) to reduce it to a single value.
  getTotal() {
    return basket.reduce((currentSum, item) => item.price + currentSum, 0);
  },
};

export default basketModule;

// 사용법
basketModule.addItem({ item: "bread", price: 0.5 });

console.log(basketModule.basket);
```

basket 은 export되지 않고 모듈 내에서 클로져로 보호되고 있다.

특징:

- basketModule이란 네임스페이스가 지정된다.
- 비공개 자유성: 모듈 내부에서만 사용 가능한 비공개 함수를 자유롭게 만들 수 있다.
- 디버깅 용이성: 어떤 함수가 예외를 발생시켰는지 알아낼때 디버깅으로 찾기 쉬워진다.

### 모듈 패턴의 변형

**믹스인 가져오기 변형**

유틸 함수나 외부 라이브러리 같은 전역 스코프에 있는 요소를 모듈 내부의 고차 함수에 인자로 전달 할 수 있게 한다.

`utils.js => privateMethods.js => myModule.js` 순서대로 함수를 전달하여 사용한다.

**내보내기 변형**

전역 스코프에 변수로 ‘비공개’를 선언하고 모듈에서 이를 사용하는 것이다.

```tsx
const privatVariable = "";
const privateMethod = () => {};

const module = {
  publicMethod: () => {
    console.log(privateVariable);
  },
};
```

장점

- 모듈 사이의 의존성을 관리하고 전역 요소를 원하는 만큼 넘겨주어 독립적으로 만들어준다.
- 원하는 것들만 export를 이용해 비공개를 지원할 수 있다.
  - 같은 이름에 대한 걱정도 사라짐

단점

- 공개와 비공개를 서로 다르게 접근해야한다.
- 공개/비공개 여부가 바뀐다면 파일에서 각각 바꿔줘야 한다.
- 자동화 단위 테스트에서 비공개 함수들은 제외되고 오류를 고칠 때 코드 복잡도를 높인다.

**WeakMap을 사용하는 모듈 패턴**

WeakMap 객체는 참조되지 않는 키는 GC에 의해 사라진다.

```tsx
const basket = new WeakMap();
const doSomethingPrivate = new WeakMap();
const doSomethingElsePrivate = new WeakMap();

class BasketModule {
  constructor() {
    basket.set(this, []);
    doSomethingPrivate.set(this, () => {
      console.log("Doing something...");
    });
    doSomethingElsePrivate.set(this, () => {
      console.log("Doing something else...");
    });
  }
  doSomething() {
    doSomethingPrivate.get(this)();
  }
  doSomethingElse() {
    doSomethingElsePrivate.get(this)();
  }
  addItem(values) {
    const basketData = basket.get(this);
    basketData.push(values);
    basket.set(this, basketData);
  }
  getItemCount() {
    return basket.get(this).length;
  }
  getTotal() {
    return basket
      .get(this)
      .reduce((currentSum, item) => item.price + currentSum, 0);
  }
}

const basketModule = new BasketModule();
```

### 노출 모듈 패턴

기존 모듈 패턴에서 개선된 버전이며, 공개하고 싶은 부분만 포인터를 통해 비공개 요소에 접근할 수 있게 해주는 익명객체를 반환하는 패턴이다.

```tsx
let privateVar = "Rob Dodson";
const publicVar = "Hey there!";

const privateFunction = () => {
  console.log(`Name:${privateVar}`);
};

const publicSetName = (strName) => {
  privateVar = strName;
};

const publicGetName = () => {
  privateFunction();
};

// Reveal public pointers to
// private functions and properties
const myRevealingModule = {
  setName: publicSetName,
  greeting: publicVar,
  getName: publicGetName,
};

export default myRevealingModule;
```

장점:

- 코드의 일관성이 유지되고, 가장 아래쪽에 위치한 공개 객체를 더 알아보기 쉽게 바꾼다.

단점:

- 비공개 함수를 참조하는 공개 함수를 수정할 수 없다.
  - 비공개 함수가 비공개 구현을 참조했기 때문에, 수정을 해도 함수가 변경될 뿐 참조된 구현이 변경되는 것이 아니다.
- 비공개 변수를 참조하는 공개객체 멤버도 수정할 수 없다.
  - `myRevealingModule` 수정이 불가

### 싱글톤 패턴

클래스의 인스턴스가 오직 하나만 존재하도록 제한하는 패턴이다. 이미 존재하는 인스턴스가 있을 경우 해당 인스턴스를 반환해야 한다. 또한, 초기화를 지연시킬 수 있는데 초기화 시점에 필요한 특정 정보가 유효하지 않을 수 있기 떄문이다.

```tsx
// https://github.com/addyosmani/learning-jsdp/blob/main/ch07/modules/MySingleton.mjs

let instance;

// Private methods and variables
const privateMethod = () => {
  console.log("I am private");
};
const privateVariable = "Im also private";
const randomNumber = Math.random();

// Singleton
class MySingleton {
  // Get the Singleton instance if one exists
  // or create one if it doesn't
  constructor() {
    if (!instance) {
      // Public property
      this.publicProperty = "I am also public";
      instance = this;
    }

    return instance;
  }

  // Public methods
  publicMethod() {
    console.log("The public can see me!");
  }

  getRandomNumber() {
    return randomNumber;
  }
}
```

특징

- 싱글톤 패턴을 사용하면 인스턴스에 대한 전역 접근을 허용하는 것이다.

```tsx
constructor() {
	if (this._instance == null) {
		if(isFoo()) {
			this._instance = new FooSingleton();
		}	else {
			this._instance = new BasicSingleTon();
		}
	}

	retunr this._instance;
}
```

싱글톤에서는 지연 실행이 중요하다. 코드가 언제나 같은 순서로 실행되는지 확인해야 한다. 그럴 때 확인해야하는 코드가 많아지면 확장성이 떨어진다.

🤔 무슨 말인지 이해가 안가니 코드로 살펴보기..

이 코드는 싱글톤을 사용했을때 아주 적합한 코드

```tsx
// 싱글톤 객체
class Singleton {
  constructor(options = {}) {
    this.name = "SingletonTester";
    this.pointX = options.pointX || 6;
    this.pointY = options.pointY || 10;
  }
}

let instance;

// 정적 객체
const SingletonTester = {
  name: "SingletonTester",
  getInstance(options) {
    if (instance === undefined) {
      instance = new Singleton(options);
    }

    return instance;
  },
};

const singletonTest = SingletonTester.getInstance({ pointX: 5 });
```

자바스크립트에서는 객체를 직접적으로 생성할 수 있기 때문에 **싱글톤 클래스를 만드는 대신 객체 하나를 그냥 생성이 가능하기에 설계를 다시 생각**해봐야 할 수도 있다.

**단점**

- 싱글톤임을 파악하는 것이 힘들다.
  - 큰 모듈을 import할 때 어떤 클래스가 싱글톤 클래스인지 파악이 어렵다.
- 테스트하기 힘들다.
  - 싱글톤은 숨겨진 의존성, 여러 인스턴스 생성의 어려움, 의존성 대체의 어려움 등 테스트하기에 어려울 수 있다.
- 신중한 조정이 필요하다.
  - 가장 흔하게 사용되는 예시로는 전역 범위에 걸쳐 필요한 데이터를 저장하는 것이 있다. (인증 정보, 쿠키 등) 이 데이터가 유효한 뒤에 사용할 수 있도록 올바른 실행 순서를 구현하는 일이 필수

🤔 책의 말이 어려워 다시 생각해보기

자바스크립트에서 싱글톤 패턴을 사용할 때에는 어떤 클래스가 싱글톤인지 알기 어렵다. 그렇기 때문에 정말 필요한 곳에서만 사용을 하고, 특히나 하나의 인스턴스를 활용하기 때문에 **여러 곳에서 인스턴스를 참조**하고 있다.

그러면 **실행 순서가 보장이 되어야 하는 곳에서는 실행순서를 보장할 수 있는 코드가 구현**되어 있어야 한다.

### 프로토타입 패턴

이미 존재하는 객체를 복제해 만든 템플릿을 기반으로 새 객체를 생성하는 패턴

프로토타입 패턴은 프로토타입 상속을 기반으로 한다.

프로토타입 패턴은 상속을 구현하는 쉬운 방법일 뿐만 아니라 **성능에서의 이점**도 챙길 수 있다.
객체 내에 함수를 정의할 때 복사본이 아닌 **참조로 생성되어 모든 자식 객체가 동일한 함수를 가리키게 할 수** 있기 때문

### Object.create

다른 객체로부터 상속할 수 있게 해주는 차등 상속과 같은 고급 개념을 쉽게 구현할 수 있게 해준다.

`const yourCar = Object.create(myCar);` 사용시 myCar의 속성들이 모두 yourCar에 그대로 상속이 된다.

🤔 책의 말이 너무 어려워 다른 예제로 살펴보기

```tsx
const carPrototype = {
  wheels: 4,
  drive() {
    console.log(`Driving with ${this.wheels} wheels`);
  },
};

const myCar = Object.create(carPrototype);
myCar.drive(); // "Driving with 4 wheels"

// 새로 만든 객체에 추가 프로퍼티도 설정 가능
myCar.color = "red";
console.log(myCar.color); // "red"
```

즉 하나의 prototype 근본이 되는 것들로 계속해서 만드는 패턴

### 팩토리 패턴

팩토리 패턴은 객체를 직접 만들어서 반환하는 패턴이다. (프로토타입 패턴과는 다르게 **기존 객체를 복제하는 게 아닌 새롭게 객체를 만드는 것**)

위의 프로토타입 패턴과의 비교 예시

```tsx
function createCar(name, wheels) {
  return {
    name,
    wheels
    drive() {
      console.log(`Driving with ${this.wheels} wheels`);
    }
  };
}

const myCar = createCar('Car1', 4);
myCar.drive(); // "Driving with 4 wheels"
```

팩토리 패턴을 사용하면 좋은 상황 🥇

- 객체나 컴포넌트의 생성 과정이 높은 복잡성을 가지고 있을 때
- 상황에 맞춰 다양한 객체 인스턴스를 편리하게 생성할 수 있는 방법이 필요할 때
- 같은 속성을 공유하는 여러 개의 작은 객체 또는 컴포넌트를 다뤄야 할 때
- 덕 타이핑 같은 API 규칙만 충족하면 되는 다른 객체의 인스턴스와 함께 객체를 구성할 때, 또한 디커플링(decoupling = 결합도 ⬇️) 에도 유용

**🎯 쉽게 요약**

- **Prototype Pattern**
  - “얘 좋네? **복사해서 하나 더 쓰자**!”
- **Factory Pattern**
  - “필요한 거 맞춰서 **새로 만들어줄게**!”

**추상 팩토리 패턴**

객체가 어떻게 생성되는지에 대한 세부사항을 알 필요 없이 객체를 사용할 수 있게 하는 패턴

```tsx
class AbstractVehicleFactory {
  constructor() {
    // Storage for our vehicle types
    this.types = {};
  }

  getVehicle(type, customizations) {
    const Vehicle = this.types[type];
    return Vehicle ? new Vehicle(customizations) : null;
  }

  registerVehicle(type, Vehicle) {
    const proto = Vehicle.prototype;
    // drive, breakDown 을 가지고 있는지 체크
    if (proto.drive && proto.breakDown) {
      this.types[type] = Vehicle;
    }
    return this;
  }
}
```

## 구조 패턴

### 퍼사드 패턴

Facade란 실제 모습을 숨기고 꾸며낸 겉모습만을 세상에 드러내는 것

⇒ 심층적인 복잡성은 숨기고, 사용하기 편리한 높은 수준의 인터페이스를 제공하는 패턴

🤔 책에서는 jQuery를 예시로 들고와서 패스,,

우리가 흔히 사용하는 라이브러리들은 대부분 퍼사드 패턴을 충족한다고 생각한다.

최근에 suspenseive 라이브러리 코드를 봤음

⇒ https://github.com/toss/suspensive/tree/main/packages/react-query-5/src

react-query를 추상화하여 쉽게 사용할 수 있도록 제공

### 믹스인 패턴

서브클래스가 쉽게 상속받아 기능을 재사용할 수 있도록 하는 클래스

서브 클래싱이란?

- 부모 클래스 객체에서 속성을 상속받아 새로운 객체를 만드는 것을 뜻한다.

🤔 말이 어려운것 같은데,, 그냥 extends 해서 만든 클래스라고 보면 될듯 ?

**믹스인**

믹스인은 최소한의 복잡성으로 객체의 기능을 빌리거나 상속할 수 있도록 도와준다.

코드로 살펴보기 ⇒ https://github.com/addyosmani/learning-jsdp/blob/main/ch07/mixins_superclassing.html

🤔 scss의 mixin과 동일한 느낌으로 보면 될 것 같다.

공통된 함수를 하나 만들어서 재사용성을 높이는 느낌?

`class Car extends MyMixins(CarAnimator)` ⇒ Car 클래스에 공통된 애니메이션을 주입

장점

- 객체 인스턴스 사용이 공유되는 기능이 있다면 중복을 피하고 고유 기능을 구현하는 데에 집중할 수 있다.

단점

- 클래스나 객체의 프로토타입에 기능을 주입하는 것은 나쁜 방법

### 데코레이터 패턴

코드 재사용을 목표로 하는 구조 패턴, 데코레이터를 사용하면 기존 시스템의 내부 코드를 힘겹게 바꾸지 않고 기능을 추가할 수 있게 된다.

🤔 리액트에서의 데코레이터 패턴 ⇒ HOC

리액트에서는 기존 컴포넌트에 새로운 기능을 래핑(데코레이트)해서 리턴하는 Hoc 패턴이 여기에 속한다고 보인다.

```tsx
function withAuth<P>(WrappedComponent: React.ComponentType<P>) {
  return (props: P) => {
    const isAuthenticated = true; // 이걸 실제로는 context나 쿠키에서 가져오겠지

    if (!isAuthenticated) {
      return <div>로그인이 필요합니다</div>;
    }

    return <WrappedComponent {...props} />;
  };
}
```

🤔 인터페이스 코드 살펴보기

책에서는 인터페이스 클래스를 통해 코드의 안전성을 높이는 패턴을 소개한다.

거기에 쓰인 인터페이스 코드가 책에는 없어서 따로 살펴봤습니다.

https://github.com/addyosmani/learning-jsdp/blob/main/ch07/decorators_interface.html#L22

에 있는 코드를 약간 변형하였습니다.

```tsx
type MethodName = string;

class Interface {
  name: string;
  methods: MethodName[];

  constructor(name: string, methods: MethodName[]) {
    if (methods.length === 0) {
      throw new Error(
        "Interface constructor expects at least one method name."
      );
    }
    this.name = name;
    this.methods = [];

    for (const method of methods) {
      // string만 속성이 가능
      if (typeof method !== "string") {
        throw new Error(
          "Interface constructor expects method names as strings."
        );
      }
      this.methods.push(method);
    }
  }

  // 객체에 인터페이스가 구현되어있는지 체크하는 함수
  static ensureImplements(object: unknown, ...interfaces: Interface[]): void {
    // 최소 1개의 속성을 받아야 함
    if (interfaces.length === 0) {
      throw new Error(
        "Function Interface.ensureImplements expects at least one Interface argument."
      );
    }

    for (const interfaceInstance of interfaces) {
      if (!(interfaceInstance instanceof Interface)) {
        throw new Error(
          "Arguments to Interface.ensureImplements must be instances of Interface."
        );
      }

      for (const method of interfaceInstance.methods) {
        if (typeof (object as Record<string, unknown>)[method] !== "function") {
          throw new Error(
            `Object does not implement the ${interfaceInstance.name} interface. Method ${method} was not found.`
          );
        }
      }
    }
  }
}
```

데코레이트 패턴 주의해야할 점

- 네임 스페이스에 작고 비슷한 객체를 추가하기 때문에, 구조가 매우 복잡해질 수 있다.

### 플라이웨이트 패턴

반복되고 느리고 비효율적으로 데이터를 공유하는 코드를 최적화하는 전통적인 구조적 해결 방법

**내재적 상태**

- 객체의 내부 메서드에 필요한 것이라 없으면 동작하지 않는다.

**외재적 상태**

- 외재적 정보는 제거되어 외부에 저장이 될 수 있다.

구현 방법

- 플라이웨이트
  - 외부의 상태를 받아 작동할 수 있게 하는 인터페이스
- 구체적(concrete) 플라이웨이트
  - 인터페이스를 실제로 구현하고 내부 상태를 저장한다. 컨텍스트 사이에서 공유될 수 있어야 하며, 외부 상태를 조작할 수 있어야 한다.
- 플라이웨이트 팩토리
  - 개별 인스턴스가 필요할 때 재사용할 수 있도록 관리한다.

🤔 덕펀칭 살펴보기

![image.png](attachment:3dbc643a-ccc8-44e8-a297-4007c29c7a17:image.png)

⇒ https://dev.to/chunkybyte/duck-punching-in-javascript-with-example-4nkf

유연하게 기능을 확장할 수 있지만 안정성에 영향을 줄 수 있다.

인터페이스 타입스크립트 버전

```tsx
interface SuperclassOrInterface {
  [key: string]: any;
}

class InterfaceImplementation {
  static implementsFor<T extends object>(
    superclassOrInterface: T
  ): typeof InterfaceImplementation {
    Object.setPrototypeOf(
      this.prototype,
      typeof superclassOrInterface === "function"
        ? superclassOrInterface.prototype
        : superclassOrInterface
    );
    return this;
  }
}
```

예시 코드 살펴보기 ⇒ https://github.com/addyosmani/learning-jsdp/blob/main/ch07/flyweight.html

**플라이웨이트 변환하기**

외부 상태 관라하기 코드 ⇒

https://github.com/addyosmani/learning-jsdp/blob/main/ch07/flyweight_extrinsic_states.html

https://github.com/addyosmani/learning-jsdp/blob/main/ch07/flyweight_extrinsic_states.html#L148C13-L148C31 에서 외부 상태 관리중

- 외부에서 상태를 관리하기 때문에 메모리를 절약할 수 있다.
