# 7. 자바스크립트 디자인패턴

## 7.1 생성 패턴

- 생성자 패턴
- 모듈 패턴
- 노출 모듈 패턴
- 싱글톤 패턴
- 프로토타입 패턴

## 7.2 생성자 패턴

생성자 패턴은 객체를 생성하고 초기화하는 특별한 메서드를 통해 유사한 객체를 일관된 방식으로 만들 수 있게 해주는 패턴입니다.
자바스크립트에서는 함수를 생성자로 사용해 객체의 초기 상태와 동작을 정의하고, new 키워드를 통해 인스턴스화합니다. 이 패턴은 코드 재사용성을 높이고 객체 생성 로직을 캡슐화하는 데 유용합니다.

객체를 생성하는 방법

```tsx
// 리터럴 표기법
const newObject = {};

// Object.create() 메서드
const newObject = Object.create(Object.prototype);

// new 키워드
const newObject = new Object();
```

객체에 키와 값을 할당하는 방법

```tsx
// Dot 문법
newObject.someKey = 'some value';
// 대괄호 문법
newObject['someKey']= 'some value';

// ES5만 호환되는 방식 - defineProperty 속성 활용
Object.defineProperty(newObject, 'someKey', {
	value: 'some value',
	writable: true,
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

## 7.2.3 프로토타입을 가진 생성자
프로토타입 객체의 재정의를 피하기 위해 Object.prototype 대신 Object.prototype.newMethod 형태를 사용하고 있음에 유의합시다.


기존에 정의된 프로토타입 객체를 유지하기 위해서 입니다.

```javascript
// 안 좋은 방식 - 프로토타입 객체 전체를 재정의
// 이렇게 하면 기존의 내장 메서드(toString, valueOf 등)가 모두 사라짐
Object.prototype = {
  newMethod: function() { 
    console.log("새 메서드 실행"); 
  }
  // 기존 내장 메서드들이 사라져서 문제 발생!
};

// 좋은 방식 - 기존 프로토타입은 유지하면서 새 메서드만 추가
Object.prototype.newMethod = function() { 
  console.log("새 메서드 실행"); 
};
// 이 방식은 toString, valueOf 등의 기존 내장 메서드를 유지하면서 새로운 메서드만 추가함
```

### 부록: 자바스크립트와 다른 언어의 생성자 비교

각 프로그래밍 언어마다 클래스 및 생성자를 구현하는 방식에 차이가 있습니다.

#### 자바스크립트 (JavaScript)
```javascript
class Person {
  constructor(name) {
    this.name = name;
  }

  greet() {
    console.log(`Hello, ${this.name}`);
  }
}

const p = new Person('Alice');
p.greet();
```
- **특징**
  - `constructor`라는 고정된 메서드 이름 사용 (변경 불가)
  - 인스턴스 초기화 전용
  - 생성자를 명시하지 않으면 기본 생성자(빈 constructor)가 자동으로 제공됨
  - 오버로딩 불가능 (자바스크립트는 메서드 오버로딩 개념이 없음)

#### 자바 (Java)
```java
public class Person {
  private String name;

  // 생성자 - 클래스 이름과 동일해야 함
  public Person(String name) {
    this.name = name;
  }

  public void greet() {
    System.out.println("Hello, " + this.name);
  }
}
```
- **특징**
  - 클래스 이름과 동일한 이름의 메서드가 생성자
  - 오버로딩 가능 (매개변수가 다른 여러 생성자 선언 가능)

#### Python
```python
class Person:
    def __init__(self, name):
        self.name = name

    def greet(self):
        print(f"Hello, {self.name}")
```
- **특징**
  - 고정된 이름 `__init__` 사용 (언어 규칙상 변경 불가)
  - 하나만 존재, 오버로딩은 언어 차원에서 지원 안 함 (디폴트 인자를 활용하거나 로직으로 처리)

#### 비교 요약
| 언어 | 생성자 명명 규칙 | 오버로딩 가능 여부 |
|------|-------------------|-------------------|
| JavaScript | `constructor` 고정 | 불가능 (JS는 오버로딩 개념 없음) |
| Java | 클래스 이름과 동일 | 가능 |
| Python | `__init__` 고정 | 불가능 (가변 인자 처리로 우회) |
| TypeScript | `constructor` 고정 (JS와 동일) | 불가능 (오버로딩 시그니처만 가능, 구현은 하나) |

#### 결론 및 실무적 차이
- **JS/Python**
  - 고정된 이름을 사용하며 오버로딩이 불가능
  - 생성자 논리를 가변 인자 처리나 조건 분기로 유연하게 구현

- **Java**
  - 클래스명과 동일한 생성자명
  - 오버로딩 지원으로 다양한 초기화 방식 제공

자바스크립트는 함수 기반 언어의 특성을 그대로 유지하면서도 클래스 문법을 흉내낸 것이므로, 자바와 문법만 비슷할 뿐 내부 동작과 유연성은 함수 지향적 특성이 더 강합니다.

### 부록: 자바스크립트가 constructor 키워드를 사용하는 이유

#### 왜 "클래스명과 같은 이름의 생성자"를 자바스크립트는 채택하지 않았을까?

##### 1. 자바스크립트는 원래 클래스 기반 언어가 아니었음
자바스크립트는 함수 기반 프로토타입 언어로 출발했습니다. 클래스 문법은 ES6(2015)에 와서야 도입됐고, 사실상 함수 기반을 감춘 문법적 설탕(Syntax Sugar)입니다.

자바스크립트 엔진은 여전히 이렇게 동작합니다.

```javascript
// 이렇게 작성하면
class Person {
  constructor(name) {
    this.name = name;
  }
}

// 사실상 이렇게 동작합니다
function Person(name) {
  this.name = name;
}
```

자바스크립트 클래스는 진짜 클래스가 아니고, 함수 기반으로 동작하는 걸 포장한 문법일 뿐입니다.

##### 2. 클래스명과 메서드명 충돌 위험
만약 이런 식으로 작성한다고 가정해봅시다.

```javascript
class Person {
  person() {  // 클래스명과 같은 메서드명
    console.log('Something else');
  }
}
```

자바스크립트는 소문자/대문자를 구분하지만, 문법적으로 클래스 이름과 메서드 이름이 독립적이기 때문에 **"클래스명과 같은 메서드명을 얼마든지 쓸 수 있습니다."**

이게 가능하다 보니, 메서드 이름과 생성자 이름을 일치시키는 규칙을 만들면 혼란이 발생할 여지가 큽니다. 그래서 "생성자는 반드시 constructor"라고 고정된 키워드를 부여해 "이게 생성자입니다"라고 명확하게 선언하게 만든 것입니다.

##### 3. 자바스크립트는 클래스 이름이 재할당 가능
자바는 클래스 이름이 타입으로 고정되어 있지만, 자바스크립트는 클래스도 변수이기 때문에 재할당이 가능합니다.

```javascript
class Person {}
Person = 'not a class anymore';  // 이런 재할당이 가능합니다
```

이런 언어적 특성상, 클래스명과 메서드명을 일치시키는 것보다 고정된 constructor 키워드를 사용하는 게 더 안전합니다.

##### 4. 동적 클래스 선언을 지원
자바스크립트는 클래스 이름조차 동적으로 생성할 수 있습니다.

```javascript
const className = 'Person';
const DynamicClass = class {
  constructor() {}
};
```

자바는 클래스명과 생성자명이 일치해야 하기 때문에 이런 동적 선언이 불가능하지만, 자바스크립트는 이름이 고정되지 않아도 되기 때문에 고정된 constructor 키워드를 사용하는 게 합리적입니다.

#### 클래스명과 생성자명을 일치시키면 발생하는 문제들

자바처럼 클래스명과 생성자명을 일치시킬 수도 있었겠지만, 자바스크립트 언어 특성(함수 기반, 동적 선언, 변수 재할당 가능성) 때문에 명확하게 고정된 constructor 키워드를 도입하는 게 더 안전하고 일관된 선택이었습니다.

## 부록: JavaScript Class는 문법적 설탕(Syntactic Sugar)인가? 에 대한 논의

### 문법적 설탕(Syntactic Sugar)이란?

문법적 설탕은 프로그래밍 언어에서 기존 기능을 더 읽기 쉽고 표현하기 쉽게 만들어주는 문법을 의미합니다. 즉, 언어의 기능과 표현력은 그대로 유지하면서 개발자가 코드를 더 간결하게 작성할 수 있도록 도와주는 구문입니다.

자바스크립트에서 클래스가 문법적 설탕이다 라는 표현에 반대하는 의견이 종종 있었던 걸로 기억합니다.

1. **단순 설탕 이상이라는 의견**

   - 클래스는 `new` 키워드 없이 호출하면 오류가 발생
   - 항상 strict mode로 실행
   - `super` 키워드를 통한 부모 클래스 메서드 호출 지원
   - 비공개 필드와 메서드(`#` 접두사) 지원

2. **문법적 설탕이라는 의견**
   - 내부적으로 여전히 프로토타입 기반으로 동작
   - 자바스크립트의 클래스는 결국 함수 타입

```javascript
// ES5 방식의 프로토타입 기반 코드
function Person(name) {
  this.name = name;
}

Person.prototype.sayHi = function () {
  console.log("Hi, I am " + this.name);
};

// ES6 방식의 클래스 기반 코드 (문법적 설탕)
class Person {
  constructor(name) {
    this.name = name;
  }

  sayHi() {
    console.log(`Hi, I am ${this.name}`);
  }
}
```

## 부록: Object.defineProperty 개요

자바스크립트에서 Assignment(할당) 연산자는 객체의 속성(property)을 설정하거나 변경할 때 사용됩니다.
Object.defineProperty()의 writable 속성은 이 할당 연산자를 통한 값 변경 가능 여부를 결정합니다.

```javascript
const obj = {};

// writable: false로 설정하면 할당 연산자로 값을 변경할 수 없음
Object.defineProperty(obj, "readOnly", {
  value: 10,
  writable: false,
});

// writable: true로 설정하면 할당 연산자로 값을 변경할 수 있음
Object.defineProperty(obj, "writable", {
  value: 20,
  writable: true,
});

obj.readOnly = 100; // 무시됨 (strict mode에서는 에러 발생)
console.log(obj.readOnly); // 10

obj.writable = 200; // 정상 작동
console.log(obj.writable); // 200
```

### 부록: Object.defineProperty의 모든 속성 옵션

Object.defineProperty()로 속성을 정의할 때 다음과 같은 속성 옵션들을 설정할 수 있습니다.

1. **데이터 속성 설명자(Data Property Descriptor)**

   - **value**: 속성에 연결된 값
   - **writable**: 할당 연산자를 통해 값을 변경할 수 있는지 여부 (boolean)
   - **enumerable**: for...in 루프나 Object.keys() 등에서 열거 가능한지 여부 (boolean)
   - **configurable**: 속성 설명자를 변경하거나 속성을 삭제할 수 있는지 여부 (boolean)

2. **접근자 속성 설명자(Accessor Property Descriptor)**

   - **get**: 속성을 읽을 때 호출되는 함수 (getter)
   - **set**: 속성에 값을 할당할 때 호출되는 함수 (setter)
   - **enumerable**: for...in 루프나 Object.keys() 등에서 열거 가능한지 여부 (boolean)
   - **configurable**: 속성 설명자를 변경하거나 속성을 삭제할 수 있는지 여부 (boolean)

```javascript
const person = {};

// 데이터 속성 설명자 사용
Object.defineProperty(person, "name", {
  value: "John", // 속성 값
  writable: true, // 값 변경 가능
  enumerable: true, // 열거 가능
  configurable: true, // 설명자 변경 및 삭제 가능
});

// 접근자 속성 설명자 사용
Object.defineProperty(person, "fullName", {
  get: function () {
    return this.firstName + " " + this.lastName;
  },
  set: function (value) {
    const parts = value.split(" ");
    this.firstName = parts[0];
    this.lastName = parts[1];
  },
  enumerable: true,
  configurable: true,
});

// 사용 예시
person.firstName = "John";
person.lastName = "Doe";
console.log(person.fullName); // "John Doe"

person.fullName = "Jane Smith";
console.log(person.firstName); // "Jane"
console.log(person.lastName); // "Smith"
```

### 부록: 속성 옵션들의 기본값

속성 설명자의 옵션을 지정하지 않으면 다음과 같은 기본값이 적용됩니다.

```javascript
// 기본값 예시
Object.defineProperty(obj, "prop", {
  value: undefined,
  writable: false,
  enumerable: false,
  configurable: false,
});
```

반면, 일반적인 할당(`obj.prop = value`)으로 속성을 정의하면 모든 플래그가 true로 설정됩니다.

```javascript
const obj = {};
obj.prop = "value";

// 위 코드는 다음과 동일합니다.
Object.defineProperty(obj, "prop", {
  value: "value",
  writable: true,
  enumerable: true,
  configurable: true,
});
```

이러한 속성 옵션들을 적절히 활용하면 객체의 속성을 보다 세밀하게 제어할 수 있습니다.

## 7.3 모듈 패턴

### 7.3.1 모듈 패턴의 개요

초기 자바스크립트에서는 다음과 같은 방식으로 모듈을 구현했습니다.

- 객체 리터럴 표기법
- 모듈 패턴
- AMD 모듈
- CommonJS 모듈

ES2015(ES6) 이전에는 모듈 내보내기 기능을 지원하는 CommonJS 모듈이나 AMD 모듈이 주로 사용되었습니다.

[객체 리터럴에 대한 글](https://github.com/rmurphey/rmurphey/blob/master/public/blog/using-objects-to-organize-your-code.md)

모듈 패턴의 핵심 기능

- **캡슐화**: 객체 리터럴을 사용하여 코드를 캡슐화할 수 있습니다.
- **클로저 활용**: 모듈 패턴은 클로저를 활용해 비공개 상태와 구성을 캡슐화합니다.
- **공개 API**: 모듈 패턴을 사용하면 공개 API만 노출하고 나머지는 클로저 내부에 비공개로 유지할 수 있습니다. 이를 통해 다른 애플리케이션이 사용해야 하는 부분만 노출하고, 핵심 작업은 보호하는 깔끔하고 체계적인 구조를 구축할 수 있습니다.
- **비공개 구현**: ES2019(ES10) 이전의 자바스크립트에서는 접근 제한자(# 해시)를 지원하지 않아 비공개라는 개념이 존재하지 않았습니다. 당시에는 함수 스코프를 이용해 비공개 개념을 구현했습니다.
- **WeakMap 활용**: 반환된 객체에 포함된 변수를 비공개하려면 WeakMap()을 사용할 수 있습니다. WeakMap은 객체만 키로 설정할 수 있으며 순회가 불가능하기에 객체의 참조를 통해서만 접근이 가능합니다.

### 7.3.3 WeakMap을 사용한 private data 관리 예제

```javascript
const privateData = new WeakMap();

class Person {
  constructor(name, age) {
    privateData.set(this, {
      name: name,
      age: age
    });
  }

  getName() {
    return privateData.get(this).name;
  }
  
  getAge() {
    return privateData.get(this).age;
  }
  
  setName(name) {
    const data = privateData.get(this);
    data.name = name;
  }
}

const person = new Person('더글라스', 30);
console.log(person.getName()); // '더글라스'
console.log(person.getAge());  // 30

console.log(person.name);      // undefined
console.log(privateData.get(person).name); // '더글라스' (실제로는 이렇게 접근하지 않음)

person.setName('으아아');
console.log(person.getName()); // '으아아'

// 객체 참조가 사라지면 WeakMap의 데이터도 가비지 컬렉션의 대상이 됨
let tempPerson = new Person('익명의 인물', 25);
console.log(tempPerson.getName()); // '익명의 인물'

// tempPerson 참조를 제거하면 WeakMap에 저장된 관련 데이터도 자동으로 메모리에서 해제됨
tempPerson = null;
```

### 7.3.4 모듈 패턴 구현 예시

```javascript
// 비공개 변수와 함수
const basket = [];

const doSomethingPrivate = () => {
  //...
};

const doSomethingElsePrivate = () => {
  //...
};

// 공개 객체 생성
const basketModule = {
  // 장바구니에 아이템 추가
  addItem(values) {
    basket.push(values);
  },

  // 장바구니의 아이템 개수 반환
  getItemCount() {
    return basket.length;
  },

  // 비공개 함수에 대한 공개 별칭
  doSomething() {
    doSomethingPrivate();
  },

  // 장바구니 아이템의 총 가치 계산
  // reduce() 메서드는 배열의 각 요소에 함수를 적용하여 단일 값으로 줄입니다
  getTotal() {
    return basket.reduce((currentSum, item) => item.price + currentSum, 0);
  },
};

export default basketModule;
```

### 7.3.5 모듈 패턴의 변형

#### 1. 믹스인 가져오기 변형
Vue에서 자주 볼 수 있는 방식입니다. 다만 이 방식은 코드의 복잡도를 높일 수 있습니다. Vue의 Mixin을 생각해보면 이해하기 쉽습니다.

#### 2. 내보내기 변형
전역 스코프에 변수로 '비공개'를 선언하고 모듈에서 이를 사용하는 방식입니다.

```javascript
const privateVariable = "";
const privateMethod = () => {};

const module = {
  publicMethod: () => {
    console.log(privateVariable);
  },
};
```

### 7.3.6 모듈 패턴의 장단점

#### 장점
- 모듈 간 의존성 관리가 용이하며 전역 요소를 원하는 만큼 넘겨주어 독립적으로 만들 수 있습니다.
- export를 이용해 원하는 것들만 노출하여 비공개를 지원할 수 있습니다.
- 이름 충돌에 대한 걱정이 줄어듭니다.

#### 단점
- 공개와 비공개 요소를 서로 다르게 접근해야 합니다.
- 공개/비공개 여부가 바뀐다면 파일에서 각각 수정해야 합니다.
- 자동화 단위 테스트에서 비공개 함수들은 테스트가 어렵고, 오류 수정 시 코드 복잡도를 높일 수 있습니다.



## 7.4 노출 모듈 패턴

### 7.4.1 노출 모듈 패턴의 개념

노출 모듈 패턴은 기존 모듈 패턴에서 개선된 버전으로, 공개하고 싶은 부분만 포인터를 통해 비공개 요소에 접근할 수 있도록 합니다.

```javascript
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

// 비공개 함수와 속성에 대한 공개 포인터 노출
const myRevealingModule = {
  setName: publicSetName,
  greeting: publicVar,
  getName: publicGetName,
};

export default myRevealingModule;
```

### 7.4.2 노출 모듈 패턴의 한계

#### 단점
- 비공개 함수를 참조하는 공개 함수를 수정할 수 없습니다.
  - 비공개 함수가 비공개 구현을 참조했기 때문에, 수정해도 함수가 변경될 뿐 참조된 구현이 변경되지 않습니다.
- 비공개 변수를 참조하는 공개 객체 멤버도 수정이 어렵습니다.
  - `myRevealingModule`의 수정이 어렵습니다.

## 7.5 싱글톤 패턴

### 7.5.1 싱글톤 패턴의 정의

싱글톤 패턴은 클래스의 인스턴스가 오직 하나만 존재하도록 제한하는 패턴입니다. 싱글톤 패턴을 사용하면 인스턴스에 대한 전역 접근을 허용하게 됩니다.

자바스크립트에서는 객체를 직접적으로 생성할 수 있기 때문에 **싱글톤 클래스를 만드는 대신 객체 하나를 그냥 생성하는 것이 가능하므로 설계를 다시 고려**해야 할 수도 있습니다.

### 7.5.2 싱글톤 패턴 예제

```javascript
// 기본적인 싱글톤 패턴
const Singleton = (function() {
  let instance;
  
  function init() {
    // 싱글톤 객체
    return {
      name: "싱글톤 인스턴스",
      getTime() {
        return new Date().toLocaleTimeString();
      }
    };
  }
  
  return {
    getInstance() {
      if (!instance) {
        instance = init();
      }
      return instance;
    }
  };
})();

// 사용 예시
const a = Singleton.getInstance();
const b = Singleton.getInstance();

console.log(a === b); // true - 동일한 인스턴스
console.log(a.getTime()); // 현재 시간 출력
```

### 7.5.3 모던 자바스크립트에서의 싱글톤 패턴

```javascript
// logger.js - ES 모듈 싱글톤
class Logger {
  constructor() {
    // 이미 인스턴스가 있으면 그 인스턴스 반환
    if (Logger.instance) {
      return Logger.instance;
    }
    
    this.logs = [];
    this.count = 0;
    Logger.instance = this;
  }
  
  log(message) {
    const timestamp = new Date().toISOString();
    const logEntry = `[${timestamp}] [로그 #${++this.count}]: ${message}`;
    this.logs.push(logEntry);
    console.log(logEntry);
    return this.count;
  }
  
  getLogs() {
    return [...this.logs];
  }
  
  clearLogs() {
    this.logs = [];
    return "모든 로그가 삭제되었습니다.";
  }
}

// 모듈로 내보내기
export default new Logger();

// 사용 예시
// import logger from './logger.js';
// logger.log("애플리케이션이 시작되었습니다.");
// logger.log("사용자가 로그인했습니다.");
// console.log(logger.getLogs()); // 모든 로그 출력
```

### 7.5.4 싱글톤 패턴의 장단점

#### 장점
- 하나의 인스턴스만 생성되므로 메모리와 자원을 절약할 수 있습니다.
- 전역적으로 접근 가능한 인스턴스를 제공합니다.
- 객체가 한 번만 초기화되도록 보장합니다.

#### 단점
- **식별의 어려움**: 큰 모듈을 import할 때 어떤 클래스가 싱글톤 클래스인지 파악하기 어렵습니다.
- **테스트의 어려움**: 싱글톤은 숨겨진 의존성, 여러 인스턴스 생성의 어려움, 의존성 대체의 어려움 등으로 테스트하기 어려울 수 있습니다.
- **조정의 복잡성**: 전역 범위에 걸쳐 필요한 데이터(인증 정보, 쿠키 등)를 저장할 때 흔히 사용되지만, 이 데이터가 유효한 뒤에 사용할 수 있도록 올바른 실행 순서를 구현하는 작업이 필수적입니다.

## 7.6 프로토타입 패턴

### 7.6.1 프로토타입 패턴의 개념

프로토타입 패턴은 이미 존재하는 객체를 복제해 만든 템플릿을 기반으로 새 객체를 생성하는 패턴입니다. 이는 자바스크립트의 프로토타입 상속을 기반으로 합니다.

프로토타입 패턴은 상속을 구현하는 쉬운 방법일 뿐 아니라 성능에서도 이점이 있습니다. 객체 내에 함수를 정의할 때 복사본이 아닌 참조로 생성되어 모든 자식 객체가 동일한 함수를 가리키게 할 수 있기 때문입니다.

### 7.6.2 Object.create의 활용

Object.create를 사용하면 다른 객체로부터 상속할 수 있으며, 차등 상속과 같은 고급 개념을 쉽게 구현할 수 있습니다.

```javascript
const myCar = {
  name: 'Ford',
  drive() {
    console.log('운전 중...');
  }
};

const yourCar = Object.create(myCar);
```

위 코드에서 `yourCar`를 생성할 때 `myCar`의 속성들이 모두 `yourCar`에 그대로 상속됩니다.

## 7.7 팩토리 패턴

### 7.7.1 팩토리 패턴의 개념

팩토리 패턴은 객체를 만들어서 반환하는 패턴입니다. 이는 기존 객체를 복제하는 것이 아닌 새로운 객체를 생성하는 방식입니다.

### 7.7.2 팩토리 패턴의 적용 시나리오

#### 팩토리 패턴을 사용하면 좋은 상황
- 객체나 컴포넌트의 생성 과정이 높은 복잡성을 가지고 있을 때
- 상황에 맞춰 다양한 객체 인스턴스를 편리하게 생성할 수 있는 방법이 필요할 때
- 같은 속성을 공유하는 여러 개의 작은 객체 또는 컴포넌트를 다뤄야 할 때
- 덕 타이핑 같은 API 규칙만 충족하면 되는 다른 객체의 인스턴스와 함께 객체를 구성할 때
- 디커플링(decoupling = 결합도 감소)이 필요할 때

#### 팩토리 패턴을 사용하지 않는 것이 좋은 상황
- 객체 생성 인터페이스 제공이 설계 목표가 아니라면 자칫 불필요하게 코드 복잡도를 높일 수 있습니다.

### 7.7.3 추상 팩토리 패턴

추상 팩토리 패턴은 객체가 어떻게 생성되는지에 대한 세부사항을 알 필요 없이 객체를 사용할 수 있게 하는 패턴입니다. 이 패턴은 객체의 생성 과정에 영향을 받지 않아야 하거나 여러 타입의 객체로 작업해야 하는 경우 유용합니다.

```javascript
class AbstractVehicleFactory {
  constructor() {
    // 차량 타입을 저장하는 곳
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
```

## 7.8 구조 패턴

구조 패턴은 객체들이 더 큰 구조를 형성하도록 조합하는 방법을 제공합니다. 주요 구조 패턴으로는 다음과 같은 것들이 있습니다.

- 퍼사드 패턴
- 믹스인 패턴
- 데코레이터 패턴
- 플라이웨이트 패턴

## 7.9 퍼사드 패턴

### 7.9.1 퍼사드 패턴의 개념

Facade란 실제 모습을 숨기고 꾸며낸 겉모습만을 세상에 드러내는 것을 의미합니다. 퍼사드 패턴은
- 심층적인 복잡성은 숨기고, 사용하기 편리한 높은 수준의 인터페이스를 제공하는 패턴입니다.
- 복잡한 하위 시스템(여러 모듈, 기능)을 단일 인터페이스로 감싸 "간단하게 사용하도록 돕는 구조"입니다.

자바스크립트 생태계에서는 라이브러리나 프레임워크의 "단일 API 노출" 전략에서 많이 사용됩니다.

### 7.9.2 퍼사드 패턴의 핵심 특징

- **단순화된 인터페이스 제공**: 복잡한 하위 시스템을 단일 인터페이스로 추상화합니다.
- **의존성 최소화**: 사용자가 내부 구현을 알 필요가 없습니다.
- **사용 편의성 증가**: 복잡한 로직을 단순한 API로 사용할 수 있게 합니다.

### 7.9.3 퍼사드 패턴의 구조

```
클라이언트 → 퍼사드 → 다양한 하위 시스템 컴포넌트
```

### 7.9.4 예시: Axios - HTTP 요청의 복잡성을 감추는 퍼사드

Axios는 복잡한 HTTP 클라이언트 기능을 간단한 인터페이스로 제공합니다.

내부적으로 다음과 같은 복잡한 시스템으로 구성됩니다.
- **XMLHttpRequest/HTTP 클라이언트**: 브라우저와 Node.js 환경 모두에서 HTTP 통신 처리
- **Promise 처리**: 비동기 요청을 직관적으로 다루기 위한 Promise 기반 API
- **요청/응답 인터셉터**: 요청이나 응답을 자동으로 변환하는 미들웨어 시스템
- **설정 병합 로직**: 전역, 인스턴스, 요청별 설정을 지능적으로 병합
- **어댑터 시스템**: 환경에 따라 적절한 HTTP 처리 메커니즘 선택

#### 퍼사드 인터페이스 - 단순하고 직관적인 API

```javascript
// 복잡한 내부 구현을 감춘 간단한 인터페이스
axios.get('/api/users')
  .then(response => console.log(response.data))
  .catch(error => console.error(error));
```

#### 퍼사드 API 유형

##### 1. 기본 설정 API
모든 옵션을 직접 구성할 수 있는 유연한 인터페이스:

```javascript
axios({
  url: '/api/users',
  method: 'GET',
  headers: { 'Authorization': 'Bearer token' },
  timeout: 5000,
  withCredentials: true
});
```

##### 2. 간편 메서드 API
자주 사용하는 패턴을 위한 간결한 인터페이스:

```javascript
// GET 요청
axios.get('/api/users');

// POST 요청
axios.post('/api/users', { name: 'John', email: 'john@example.com' });

// 간편 메서드는 내부적으로 기본 API를 호출합니다
// axios.get(url) → axios({ method: 'get', url })
```

### Axios 내부 아키텍처

| 구성 요소 | 설명 | 퍼사드가 감추는 복잡성 |
|---------|------|-------------------|
| **어댑터(Adapter)** | 환경에 따라 XHR 또는 Node.js HTTP 모듈 선택 | 환경별 HTTP 통신 세부 구현 |
| **인터셉터(Interceptor)** | 요청/응답 가로채기 및 변형 | 요청 전처리, 응답 후처리 로직 |
| **Config 병합** | 다양한 레벨의 설정 지능적 병합 | 복잡한 설정 우선순위 결정 로직 |
| **Promise 체인** | 인터셉터와 어댑터를 연결한 비동기 파이프라인 | 비동기 실행 순서 관리 |
| **취소 토큰** | 요청 취소 메커니즘 | AbortController 또는 취소 토큰 구현 |

퍼사드 패턴의 핵심 가치는 "필요한 것만 보여주고 복잡한 것은 감춘다"는 원칙에 있습니다. Axios는 이 원칙에 따라 HTTP 통신의 복잡성을 단순한 인터페이스로 제공합니다.

### 7.9.5 퍼사드 패턴의 장단점

#### 장점
- **코드 복잡성 감소**: 사용자는 복잡한 시스템의 세부사항을 몰라도 됩니다.
- **의존성 분리**: 클라이언트 코드와 하위 시스템 간 결합도가 감소합니다.
- **테스트 용이성**: 퍼사드를 통해 모의 객체(Mock)를 주입할 수 있습니다.

#### 단점
- **유연성 제한**: 퍼사드가 제공하지 않는 고급 기능 사용이 어려울 수 있습니다.
- **오버헤드 가능성**: 불필요한 기능까지 포함할 수 있습니다.

### 7.9.6 실무 적용 시나리오
- 복잡한 라이브러리 래핑
- 레거시 코드 현대화
- 마이크로서비스 API 게이트웨이
- 다양한 API를 통합하는 클라이언트

## 7.10 믹스인 패턴

### 7.10.1 믹스인 패턴의 개념

믹스인 패턴은 최소한의 복잡성으로 객체의 기능을 빌리거나 상속할 수 있도록 도와주는 패턴입니다. 코드 재사용을 위한 효과적인 방법을 제공합니다.

실제 코드 예시는 다음 링크에서 확인하실 수 있습니다. [믹스인 예제 코드](https://github.com/addyosmani/learning-jsdp/blob/main/ch07/mixins_superclassing.html)

## 7.11 서브클래싱

서브클래싱은 부모 클래스 객체에서 속성을 받아 새로운 객체를 만드는 것을 의미합니다. 자바스크립트에서는 상속을 구현하는 주요 방법 중 하나입니다.

## 7.12 믹스인

### 7.12.1 믹스인의 상세 개념

믹스인은 최소한의 복잡성으로 객체의 기능을 빌리거나 상속할 수 있게 해주는 패턴입니다. 믹스인은 다른 여러 클래스를 아우르며 쉽게 공유할 수 있는 속성과 메서드를 가진 클래스입니다.

자바스크립트의 클래스는 부모 클래스를 하나만 가질 수 있지만, 믹스인을 통해 여러 클래스의 기느을 섞어서 문제를 해결할 수 있습니다.

#### 장점
- 코드 재사용성 증가
- 다중 상속과 유사한 효과를 낼 수 있음
- 기능 모듈화가 용이함

#### 단점
- 작은 객체들이 많이 생성되어 코드 복잡도가 증가할 수 있음
- 데코레이터 체인이 길어질수록 디버깅이 어렵고 관리가 힘들어짐
- 특정 데코레이터가 적용된 상태를 확인하기 어려울 수 있음
- 순서에 의존적인 데코레이터를 사용할 경우 예상치 못한 결과가 발생할 수 있음
- 클라이언트 코드가 데코레이터 패턴의 구현 세부사항을 알아야 할 경우가 있음

### 7.12.3 React와 믹스인
React는 v0.13 이전에 `createClass`로 컴포넌트를 만들 때 믹스인을 사용했으나, 현재는 사용하지 않습니다.

#### 믹스인 폐기 이유
- **이름 충돌**: 여러 믹스인이 동일한 이름의 메서드/상태를 사용할 때 충돌 발생
- **유지보수 어려움**: 믹스인 간 의존성이 복잡해지면 코드 추적이 어렵습니다
- **ES6 클래스 비호환**: 새로운 클래스 문법은 믹스인을 자연스럽게 지원하지 않음

#### 현재 권장하는 대안
```javascript
// 1. 고차 컴포넌트(HOC)
const withFeature = (Component) => {
  return class extends React.Component {
    render() {
      return <Component {...this.props} extraProp={value} />;
    }
  };
};

// 2. Hooks (React 16.8+)
function useFeature() {
  return { result };
}

function MyComponent() {
  const { result } = useFeature();
}
```
이 대안들은 믹스인보다 코드 추적이 쉽고 컴포넌트 간 결합도를 낮추는 장점이 있습니다.

## 7.13 데코레이터 패턴

### 7.13.1 데코레이터 패턴의 개념

데코레이터 패턴은 재사용을 목적으로 하는 구조 패턴으로, 객체 서브클래싱의 다른 방식을 제공합니다. 이 패턴은 기존 클래스에 동적으로 기능을 추가하기 위해 사용됩니다.

데코레이터 패턴의 주요 목적은 객체의 기본 기능을 확장하는 것입니다. 이는 상속을 통하지 않고도 런타임에 객체의 동작을 수정할 수 있게 합니다.

### 7.13.2 인터페이스의 역할

인터페이스는 객체가 가져야 할 메서드를 정의하는 방법을 제공합니다. 자바스크립트에서는 명시적인 인터페이스가 없지만, 암묵적인 인터페이스 구현을 통해 비슷한 효과를 얻을 수 있습니다.

#### 타입스크립트에서의 인터페이스 사용 예제

```typescript
// 인터페이스 정의
interface Vehicle {
  start(): void;
  stop(): void;
  getSpeed(): number;
}

// 인터페이스를 구현하는 클래스
class Car implements Vehicle {
  getCost(): number {
    throw new Error("Method not implemented.");
  }
  
  getDescription(): string {
    throw new Error("Method not implemented.");
  }

  start(): void {
    throw new Error("Method not implemented.");
  }
  
  stop(): void {
    throw new Error("Method not implemented.");
  }
}

// 사용 예시
const myCar: Vehicle = new Car();
myCar.start();
```

### 7.13.3 추상 데코레이터

추상 데코레이터는 데코레이터 패턴을 구조화하여 컴포넌트를 감싸는 추상 클래스를 정의하고, 그 하위에 구체적인 데코레이터들을 구현하는 방식입니다. 이 패턴은 코드 중복을 줄이고 데코레이터 간의 공통 로직을 추상 클래스에 모을 수 있게 해줍니다.

커피 주문 시스템을 예로 들면,

```typescript
// 컴포넌트 인터페이스
interface Coffee {
  getCost(): number;
  getDescription(): string;
}

// 기본 커피 구현
class SimpleCoffee implements Coffee {
  getCost(): number { return 10; }
  getDescription(): string { return "기본 커피"; }
}

// 추상 데코레이터
abstract class CoffeeDecorator implements Coffee {
  protected coffee: Coffee;
  
  constructor(coffee: Coffee) {
    this.coffee = coffee;
  }
  
  getCost(): number {
    return this.coffee.getCost();
  }
  
  getDescription(): string {
    return this.coffee.getDescription();
  }
}

// 구체 데코레이터들
class MilkDecorator extends CoffeeDecorator {
  getCost(): number {
    return this.coffee.getCost() + 2;
  }
  
  getDescription(): string {
    return this.coffee.getDescription() + ", 우유 추가";
  }
}

class SugarDecorator extends CoffeeDecorator {
  getCost(): number {
    return this.coffee.getCost() + 1;
  }
  
  getDescription(): string {
    return this.coffee.getDescription() + ", 설탕 추가";
  }
}

// 사용 예시
const coffee = new SimpleCoffee();
console.log(coffee.getDescription()); // "기본 커피"
console.log(coffee.getCost()); // 10

const coffeeWithMilk = new MilkDecorator(coffee);
console.log(coffeeWithMilk.getDescription()); // "기본 커피, 우유 추가"
console.log(coffeeWithMilk.getCost()); // 12

const coffeeWithMilkAndSugar = new SugarDecorator(coffee);
console.log(coffeeWithMilkAndSugar.getDescription()); // "기본 커피, 우유 추가, 설탕 추가"
console.log(coffeeWithMilkAndSugar.getCost()); // 13
```

장점
- 데코레이터 간 공통 로직을 추상 클래스에 모아 코드 중복 방지
- 객체 상속 없이 기능 확장 가능
- 런타임에 동적으로 객체 기능 조합 가능

단점
- 작고 비슷한 객체들이 많이 생성되어 코드 복잡도가 증가할 수 있음
- 패턴에 익숙하지 않은 다른 개발자가 패턴의 사용 목적을 파악하기 어렵게 되어 관리가 힘들어짐
- 특정 데코레이터가 적용된 상태를 확인하기 어려울 수 있음

## 7.16 플라이웨이트 패턴
플라이웨이트 패턴은 반복되고 느리며 비효율적으로 데이터를 공유하는 코드를 최적화하는 전통적인 구조적 해결 방법입니다. 
이 패턴은 많은 수의 유사한 객체를 필요할 때 메모리 사용을 최소화하는 데 도움이 됩니다.

### 7.16.1 플라이웨이트 패턴 사용법
DOM레이어에도 적용 가능함. 비슷한 동작 하는 이벤트 핸들러 모든 자식 요소에게 맡기보다는 부모 요소 같은 중앙 이벤트관리자에게 맡기는 방법이 있겠음.

### 7.16.2 데이터 공유

#### 내재적 상태 (intrinsic)
- 객체의 내부 메서드에 필요한 것으로, 이것이 없으면 객체가 동작하지 않습니다.
- 여러 컨텍스트에서 공유될 수 있는 고정된 정보입니다.

#### 외재적 상태 (extrinsic)
- 외재적 정보는 제거되어 외부에 저장될 수 있습니다.
- 컨텍스트에 따라 변하는 정보입니다.
- 외재적 정보 다룰 때에는 따로 관리자를 사용합니다. - 플라이웨이트 객체와 내재적 상태 보관하는 중앙 데이터베이스를 관리자로 구현한 것.

### 7.16.3 플라이웨이트 패턴의 구현 요소

- **플라이웨이트**: 외부 상태를 받아 작동할 수 있게 하는 인터페이스입니다.
- **구체적(concrete) 플라이웨이트**: 인터페이스를 실제로 구현하고 내부 상태를 저장합니다. 컨텍스트 사이에서 공유될 수 있어야 하며, 외부 상태를 조작할 수 있어야 합니다.
- **플라이웨이트 팩토리**: 개별 인스턴스가 필요할 때 재사용할 수 있도록 관리합니다.

### 7.16.4 플라이웨이트 패턴 예제

플라이웨이트 패턴은 수많은 유사한 객체를 효율적으로 관리할 때 유용합니다. 아래 예시는 도서관 시스템에서 책 정보를 관리하는 방법을 보여줍니다.

#### 자바스크립트 예제
```javascript
// 플라이웨이트 객체
class Book {
  constructor(title, author, genre, ISBN) {
    this.title = title;
    this.author = author;
    this.genre = genre;
    this.ISBN = ISBN;
  }
}

// 플라이웨이트 팩토리
class BookFactory {
  constructor() {
    this.books = {};
  }

  createBook(title, author, genre, ISBN) {
    // 이미 존재하는 책은 재사용
    if (!this.books[ISBN]) {
      this.books[ISBN] = new Book(title, author, genre, ISBN);
    }
    return this.books[ISBN];
  }

  getCount() {
    return Object.keys(this.books).length;
  }
}

// 컨텍스트 클래스
class Library {
  constructor() {
    this.factory = new BookFactory();
    this.records = [];
  }

  addBook(title, author, genre, ISBN, isAvailable) {
    const book = this.factory.createBook(title, author, genre, ISBN);
    this.records.push({ book, isAvailable });
  }

  getAvailableBooks() {
    return this.records
      .filter(record => record.isAvailable)
      .map(record => record.book.title);
  }

  getStats() {
    return {
      totalBooks: this.records.length,
      uniqueBooks: this.factory.getCount()
    };
  }
}

// 사용 예시
const library = new Library();

// 책 추가 (일부 책은 ISBN이 같음)
library.addBook("해리 포터", "J.K. 롤링", "판타지", "isbn-1", false);
library.addBook("해리 포터", "J.K. 롤링", "판타지", "isbn-1", true);
library.addBook("1984", "조지 오웰", "디스토피아", "isbn-2", true);
library.addBook("1984", "조지 오웰", "디스토피아", "isbn-2", false);
library.addBook("어린 왕자", "생텍쥐페리", "문학", "isbn-3", true);

// 결과 확인
console.log("총 책 수:", library.getStats().totalBooks);  // 5
console.log("고유 책 객체 수:", library.getStats().uniqueBooks);  // 3
console.log("대출 가능한 책:", library.getAvailableBooks());  // ["해리 포터", "1984", "어린 왕자"]
```



### 플라이웨이트 패턴의 실제 응용 사례

1. **웹 브라우저에서 글꼴 렌더링**: 같은 글꼴의 문자를 매번 새로 생성하지 않고 공유하여 메모리 사용량 감소

2. **게임 개발에서의 객체 풀링**: 게임 내 다수의 유사한 객체(예: 총알, 입자 효과)를 효율적으로 관리

3. **문서 편집기**: 같은 서식의 문자나 단락 스타일을 공유하여 대용량 문서 처리 속도 향상

4. **지도 애플리케이션**: 유사한 지형 타일이나 아이콘을 재사용하여 메모리 사용 최적화


### 7.16.5 자바스크립트에서 implements와 덕펀칭 활용하기

자바스크립트는 기본적으로 인터페이스를 지원하지 않지만, 덕펀칭을 통해 유사하게 구현할 수 있습니다.

```javascript
// 인터페이스 구현 기능 추가 (덕펀칭 활용)
Function.prototype.ImplementsFor = function(...interfaces) {
  const proto = this.prototype;
  
  interfaces.forEach(interface => {
    for (const method in interface) {
      // 인터페이스 메서드가 클래스에 구현되었는지 확인
      if (typeof interface[method] === 'function' && typeof proto[method] !== 'function') {
        throw new Error(`${this.name}는 ${method} 메서드를 구현해야 합니다`);
      }
    }
  });
  return this;
};

// 사용 예시
const IComparable = { compareTo: function(obj) {} };
const ISortable = { sort: function() {} };

function MyClass() { /* 생성자 */ }
MyClass.ImplementsFor(IComparable, ISortable);

// 필요한 메서드 구현
MyClass.prototype.compareTo = function(obj) { return 0; };
MyClass.prototype.sort = function() { /* 구현 */ };
```

### 덕펀칭 Duck Punching 🐤

덕펀칭은 자바스크립트에서 기존 객체의 동작을 수정하거나 확장하는 기법입니다. 이름은 "오리를 치는 것"처럼 이미 존재하는 객체의 메서드나 속성을 런타임에 변경하는 행위를 비유한 것입니다.

```javascript
// 예시: String에 새 메서드 추가
String.prototype.reverse = function() {
  return this.split('').reverse().join('');
};

console.log("Hello".reverse()); // "olleH"
```

주로 프로토타입 체인을 활용해 기존 객체의 메서드 교체, 네이티브 객체에 새 메서드 추가, 특정 조건에서 메서드 동작 변경 등에 사용됩니다. 

강력한 기법이지만, 코드 예측 가능성과 디버깅 난이도를 고려해 신중하게 사용해야 합니다.

## 7.17 행위 패턴

행위 패턴은 객체 간의 상호 작용과 책임 분배에 관한 패턴입니다. 주요 행위 패턴으로는 다음과 같은 것들이 있습니다.

- **관찰자 패턴**: 객체 간 일대다 의존성을 정의하여 한 객체의 상태가 변경되면 다른 객체들에게 자동으로 알립니다.
- **중재자 패턴**: 객체 간의 복잡한 통신을 캡슐화하여 객체들이 서로 직접 참조하지 않고도 상호작용할 수 있게 합니다.
- **커맨드 패턴**: 요청을 객체로 캡슐화하여 매개변수화된 클라이언트를 큐에 넣고, 요청에 대한 작업을 지원합니다.

이러한 행위 패턴들은 객체 간의 의사소통과 협력을 구조화하는 방법을 제공합니다.

## 7.18 관찰자 패턴
한 객체가 변경 될 때 다른 객체들에 변경되었음을 알릴 수 있게 해주는 패턴
누가 자신을 구독하는지 알 필요 없이 알림을 보낼 수 있습니다.
한 객체를 관찰하는 여러 객체들이 존재하며, 주체의 상태가 변화하면 관찰자들에게 자동으로 알림을 보냅니다.
ES2015+ 에서는 notify와 update 메서드를 사용하는 자바스크립트 클래스르 통해 주체와 관찰자르 구현함으로써 관찰자 패턴을 만들 수 있습니다.

관찰자 패턴의 구성 요소
- 주체(Subject): 관찰 대상이 되는 객체로, 관찰자들의 목록을 관리하고 상태 변화가 있을 때 모든 관찰자에게 알림을 보냅니다.
- 관찰자(Observer): 주체의 상태 변화를 감지하고 반응하는 인터페이스를 정의합니다.
- 구체적 주체(Concrete Subject): 주체 인터페이스를 구현한 실제 클래스로, 관찰 대상이 되는 상태와 데이터를 포함합니다.
- 구체적 관찰자(Concrete Observer): 관찰자 인터페이스를 구현한 실제 클래스로, 주체의 상태 변화에 따라 특정 동작을 수행합니다.

#### 관찰자 패턴 예제 코드

```javascript 
// 인터페이스 정의
interface Observer { update(data: any): void; }
interface Subject {
  subscribe(observer: Observer): void;
  unsubscribe(observer: Observer): void;
  notify(data: any): void;
}

// 뉴스 발행자 구현
class NewsPublisher implements Subject {
  private observers: Observer[] = [];
  private latestNews = '';

  subscribe(observer: Observer): void {
    if (!this.observers.includes(observer)) this.observers.push(observer);
  }

  unsubscribe(observer: Observer): void {
    const index = this.observers.indexOf(observer);
    if (index !== -1) this.observers.splice(index, 1);
  }

  notify(data: any): void {
    this.observers.forEach(observer => observer.update(data));
  }

  publishNews(news: string): void {
    this.latestNews = news;
    console.log(`새 뉴스: ${news}`);
    this.notify(news);
  }

  getLatestNews(): string { return this.latestNews; }
}

// 구독자 구현
class EmailSubscriber implements Observer {
  constructor(private email: string) {}
  update(news: string): void {
    console.log(`${this.email}로 전송: ${news}`);
  }
}

class MobileSubscriber implements Observer {
  constructor(private userId: string) {}
  update(news: string): void {
    console.log(`사용자 ${this.userId}에게 알림: ${news}`);
  }
}

// 사용 예시
const publisher = new NewsPublisher();
const email1 = new EmailSubscriber("user1@example.com");
const email2 = new EmailSubscriber("user2@example.com");
const mobile = new MobileSubscriber("user_123");

// 구독 등록
publisher.subscribe(email1);
publisher.subscribe(email2);
publisher.subscribe(mobile);

// 뉴스 발행
publisher.publishNews("관찰자 패턴 대세!");

// 구독 해지
publisher.unsubscribe(email2);

// 새 뉴스 발행
publisher.publishNews("구독 해지한 사용자는 알림 받지 않음");
```

### 7.19 발행/구독 패턴
실제 자바스크립트 환경에서는 발행/구독 패턴이라는 변형된 형태의 구현이 더 널리 사용됨. 관찰자 패턴과 유사하지만 차이점 존재함.
발행/구독 패턴에서는 이벤트 알림을 원하는 구독자와 이벤트를 발생시키는 발행자 사이에 토픽/이벤트 채널을 둠. 이러한 이벤트 시스템을 통해 애플리케이션에 특화된 이벤트 정의가 가능하고 구독자에게 필요한 값이 포함된 커스텀 인자 전달 가능함.

#### 발행/구독 패턴의 상세 설명

발행/구독(Pub/Sub) 패턴은 관찰자 패턴의 확장된 형태로 볼 수 있으며, 다음과 같은 특징을 가집니다.

1. **이벤트 채널의 존재**: 발행자와 구독자 사이에 이벤트 채널(또는 이벤트 버스, 메시지 브로커)이 있어 직접적인 의존성이 없습니다.
2. **느슨한 결합**: 발행자는 구독자에 대해 알 필요가 없고, 구독자도 발행자를 알 필요가 없습니다.
3. **다중 이벤트 타입**: 여러 종류의 이벤트(토픽)을 정의하고 구독자는 관심 있는 이벤트만 선택적으로 구독할 수 있습니다.
4. **비동기적 통신**: 대부분의 발행/구독 시스템은 비동기 방식으로 메시지를 전달합니다.

#### 관찰자 패턴과 발행/구독 패턴의 차이점

| 특성 | 관찰자 패턴 | 발행/구독 패턴 |
|------|------------|--------------|
| 결합도 | 주체가 관찰자의 존재를 알고 있음 (직접 알림) | 발행자와 구독자는 서로를 모름 (간접 알림) |
| 중간 레이어 | 없음 (주체가 직접 관찰자에게 알림) | 이벤트 채널/브로커 존재 |
| 통지 방식 | 모든 관찰자에게 동일한 알림 전송 | 토픽/이벤트 타입에 따라 선택적 구독 가능 |
| 확장성 | 주체-관찰자 간 1:n 관계 | m:n 관계 가능 (여러 발행자, 여러 구독자) |
| 구현 복잡성 | 상대적으로 단순 | 중간 채널 구현으로 인해 더 복잡 |

#### 자바스크립트에서의 발행/구독 패턴 예제
```javascript
// 간단한 발행/구독 시스템
class EventBus {
  constructor() {
    this.topics = {};
  }

  // 이벤트 구독
  subscribe(topic, callback) {
    if (!this.topics[topic]) this.topics[topic] = [];
    const index = this.topics[topic].push(callback) - 1;
    
    // 구독 취소 함수 반환
    return {
      unsubscribe: () => {
        this.topics[topic].splice(index, 1);
        if (this.topics[topic].length === 0) delete this.topics[topic];
      }
    };
  }

  // 이벤트 발행
  publish(topic, data) {
    if (!this.topics[topic]?.length) return;
    this.topics[topic].forEach(callback => setTimeout(() => callback(data), 0));
  }
  
  // 토픽 정보 조회
  getTopics() {
    return Object.keys(this.topics).map(topic => ({
      topic,
      count: this.topics[topic].length
    }));
  }
}

// 사용 예시
const bus = new EventBus();

// 구독 등록
const userSub = bus.subscribe('user:login', user => 
  console.log(`로그인: ${user.name}`));

bus.subscribe('notification', msg => 
  console.log(`알림: ${msg.message}`));

bus.subscribe('user:login', user => 
  console.log(`로그인 시간: ${new Date().toLocaleTimeString()}`));

// 발행
bus.publish('user:login', { name: '홍길동', id: 1234 });
bus.publish('notification', { message: '새 메시지 도착', id: 5678 });

// 구독 취소
userSub.unsubscribe();

// 취소 후 발행 (첫 번째 콜백은 실행되지 않음)
bus.publish('user:login', { name: '김철수', id: 5678 });

console.log('활성 토픽:', bus.getTopics());
```

#### 실무 사례: 액세스 토큰 갱신을 위한 발행/구독 패턴 📝

실무에서 Axios interceptor를 활용해 토큰 갱신 처리 구현 시 발행/구독 패턴을 적용했던 사례가 있어서 들고와봤습니다.

```typescript
// 핵심 발행/구독 패턴 구현 부분
// 1. 구독자 목록 관리
let isRefreshing = false;
let refreshSubscribers: ((token: string) => void)[] = []; 

// 2. 발행자: 토큰 갱신 결과를 모든 구독자에게 알림
const onRefreshed = (token: string) => {
  refreshSubscribers.forEach((callback) => callback(token));
  refreshSubscribers = [];
};

// 3. 구독자 등록 함수
const addRefreshSubscriber = (callback: (token: string) => void) => {
  refreshSubscribers.push(callback);
};

// 응답 인터셉터 내부 구현
async (error: AxiosError) => {
  // 401 인증 오류 처리
  if (error.response?.status === 401) {
    const originalRequest = error.config as any;
    
    // 이미 토큰 갱신 진행 중인 경우: 현재 요청을 구독자로 등록
    if (isRefreshing) {
      return new Promise((resolve, reject) => {
        addRefreshSubscriber((token: string) => {
          if (token) {
            // 새 토큰으로 원래 요청 재시도
            originalRequest.headers.Authorization = `Bearer ${token}`;
            resolve(axios(originalRequest));
          } else {
            // 토큰 갱신 실패 시 요청도 실패 처리
            reject(new Error('토큰 갱신 실패'));
          }
        });
      });
    }
    
    // 첫 번째 401 에러: 토큰 갱신 프로세스 시작
    isRefreshing = true;
    
    try {
      // 토큰 갱신 요청 
      const newToken = await authService.refreshToken();
      
      // 갱신 성공: 새 토큰을 모든 대기 중인 요청에 발행
      onRefreshed(newToken);
      
      // 원래 요청 재시도
      originalRequest.headers.Authorization = `Bearer ${newToken}`;
      return axios(originalRequest);
    } catch (error) {
      // 갱신 실패: 빈 토큰을 발행하여 모든 요청이 실패하도록 함
      onRefreshed("");
      return Promise.reject(error);
    } finally {
      isRefreshing = false;
    }
  }
  
  return Promise.reject(error);
}
```

#### 패턴 설명

위 코드에서 발행/구독 패턴은 다음과 같이 구현되어 있습니다.

1. **발행/구독 메커니즘**
   - `refreshSubscribers` 배열: 토큰 갱신을 기다리는 요청들의 콜백 함수를 저장하는 구독자 목록
   - `addRefreshSubscriber()`: 구독자를 등록하는 함수
   - `onRefreshed()`: 토큰 갱신 결과를 모든 구독자에게 알리는 발행 함수

2. **주요 시나리오**
   - 여러 API 요청이 동시에 401 오류를 받으면 첫 번째 요청만 갱신을 시도하고 나머지는 구독자로 등록
   - 갱신이 성공하면 새 토큰이 모든 구독자에게 전파되어 자동으로 원래 요청을 재시도
   - 갱신이 실패하면 빈 토큰이 전파되어 모든 요청이 실패 처리됨

3. **패턴 사용으로 얻은 효과**
   - Race Condition 방지: 여러 요청이 동시에 401을 받아도 토큰 갱신은 한 번만 수행
   - 자원 효율성: 중복 갱신 요청 없이 단일 갱신으로 모든 요청 처리
   - 우아한 에러 처리: 갱신 결과가 모든 대기 요청에 일관되게 전파됨


### 관찰자 패턴과 발행/구독 패턴의 장단점
장점
- **느슨한 결합**: 컴포넌트 간의 직접적인 의존성을 줄여줍니다.
- **확장성**: 새로운 구독자/관찰자를 기존 코드 수정 없이 쉽게 추가할 수 있습니다.
- **코드 재사용성**: 직접 연결이 아닌 주체와 관찰자의 관계로 결합도를 낮춰 코드의 관리와 재사용성을 높일 수 있습니다.

단점
- **예측 가능성 저하**: 발행자와 구독자의 연결을 분리함으로써, 애플리케이션의 특정 부분들이 기대하는 대로 동작한다는 것을 보장하기 어려워질 수 있습니다.
- **의존성 파악 어려움**: 구독자들이 서로의 존재에 대해 전혀 알 수 없고, 발행자를 변경하는 데 드는 비용을 파악하기 어렵습니다.
- **추적 복잡성**: 구독자와 발행자 관계가 동적으로 결정되기에 어떤 구독자가 어떤 발행자에 의존하는지 추적하기 어려워질 수 있습니다.

### 리액트 생태계에서의 관찰자 패턴
RxJS - 관찰자 패턴 사용하는 대표적인 라이브러리
RxJS(Reactive Extensions for JavaScript)는 관찰자 패턴을 기반으로 한 강력한 반응형 프로그래밍 라이브러리입니다. 이 라이브러리는 비동기 데이터 스트림과 이벤트 기반 프로그래밍을 선언적으로 처리할 수 있게 해줍니다.

**RxJS의 핵심 개념**
- **Observable**: 시간에 따라 발생하는 이벤트나 데이터의 스트림을 나타냅니다.
- **Observer**: Observable을 구독하고 데이터를 받아 처리하는 객체입니다.
- **Subscription**: Observable 구독의 실행을 나타내며, 구독을 취소하는 방법을 제공합니다.
- **Operators**: 함수형 프로그래밍 방식으로 Observable을 변환하는 순수 함수들입니다(map, filter, merge 등).
- **Subject**: Observable이자 Observer 역할을 동시에 할 수 있는 특별한 유형입니다.

**RxJS 활용 예시**
```javascript
import { fromEvent } from 'rxjs';
import { map, debounceTime, distinctUntilChanged, switchMap } from 'rxjs/operators';

// 검색 입력 처리
const search$ = fromEvent(document.querySelector('#search'), 'input').pipe(
  map(e => e.target.value),
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(term => fetch(`/api/search?q=${term}`).then(res => res.json()))
);

// 구독
const sub = search$.subscribe(results => {
  renderResults(results);
});

// 정리
function unsubscribe() {
  sub.unsubscribe();
}
```

## 중재자 패턴

### 중재자 패턴의 개념

중재자 패턴(Mediator Pattern)은 객체 간의 복잡한 통신과 제어를 캡슐화하는 행위 패턴입니다. 이 패턴은 객체 간의 직접적인 참조를 제거하고, 중재자라는 중앙 객체를 통해 모든 상호작용이 이루어지도록 합니다.

중재자 패턴의 핵심 아이디어는 "객체들이 서로 직접 통신하지 않고 중재자를 통해 통신한다"는 것입니다. 이를 통해 객체 간의 결합도를 낮추고 코드의 유지보수성을 높일 수 있습니다.

### 중재자 패턴의 구성 요소

1. **중재자(Mediator)**: 객체 간 통신을 조정하는 인터페이스를 정의합니다.
2. **구체적 중재자(Concrete Mediator)**: 중재자 인터페이스를 구현하고 동료 객체 간의 조정 로직을 담당합니다.
3. **동료(Colleague)**: 중재자와 통신하는 객체들의 인터페이스를 정의합니다.
4. **구체적 동료(Concrete Colleague)**: 동료 인터페이스를 구현하고 다른 동료와의 통신이 필요할 때 중재자를 통해 처리합니다.

### 중재자 패턴 예제: 채팅방 기능

다음은 채팅방을 구현한 중재자 패턴의 예제입니다.
```javascript
// 중재자 인터페이스
interface ChatMediator {
  send(message: string, sender: User): void;
  add(user: User): void;
}

// 참여자 클래스
class User {
  constructor(private name: string, private mediator: ChatMediator) {
    this.mediator.add(this);
  }

  getName(): string { return this.name; }

  send(message: string): void {
    console.log(`${this.name} 보냄: ${message}`);
    this.mediator.send(message, this);
  }

  receive(message: string, sender: User): void {
    console.log(`${this.name} 받음: ${message} (발신: ${sender.getName()})`);
  }
}

// 구체적 중재자 구현
class ChatRoom implements ChatMediator {
  private users: User[] = [];

  add(user: User): void {
    this.users.push(user);
    console.log(`${user.getName()} 입장`);
  }

  send(message: string, sender: User): void {
    // 발신자 제외 모든 사용자에게 전달
    this.users
      .filter(user => user !== sender)
      .forEach(user => user.receive(message, sender));
  }
}

// 사용 예시
const room = new ChatRoom();
const user1 = new User('철수', room);
const user2 = new User('영희', room);
const user3 = new User('민수', room);

user1.send('안녕하세요!');
user2.send('반갑습니다!');
```
위 예제에서
- `ChatMediator`는 중재자 인터페이스입니다.
- `ChatRoom`은 구체적 중재자로, 사용자 간의 메시지 전달을 관리합니다.
- `User`는 동료 클래스로, 중재자를 통해 다른 사용자와 통신합니다.

### 중재자 패턴의 장단점

#### 장점
1. **결합도 감소**: 객체 간 직접 참조가 없어 결합도가 낮아집니다.
2. **재사용성 향상**: 동료 객체는 중재자와만 통신하므로 다른 중재자와 함께 재사용하기 쉽습니다.
3. **유지보수성 개선**: 객체 간 통신 로직이 중앙화되어 유지보수가 용이합니다.
4. **단일 책임 원칙**: 통신 로직이 중재자에 집중되어 동료 객체는 자신의 핵심 기능에만 집중할 수 있습니다.

#### 단점
1. **중재자 복잡화**: 시스템이 복잡해질수록 중재자가 복잡해지고 유지보수가 어려워질 수 있습니다.
2. **중앙 집중화**: 중재자가 단일 실패 지점(Single Point of Failure)이 될 수 있습니다.
3. **성능 오버헤드**: 모든 통신이 중재자를 거치므로 직접 통신보다 약간의 성능 저하가 발생할 수 있습니다.

## 커맨드 패턴

### 커맨드 패턴의 개념

커맨드 패턴(Command Pattern)은 요청을 객체로 캡슐화하여 클라이언트와 수신자 간의 의존성을 분리하는 행위 패턴입니다. 이 패턴을 사용하면 요청을 큐에 저장하거나 로그로 기록하고, 작업을 취소하는 등의 기능을 쉽게 구현할 수 있습니다.

커맨드 패턴의 핵심은 "하나의 행동을 객체로 표현"하는 것입니다. 이를 통해 명령을 매개변수화하고, 지연 실행, 명령 큐, 실행 취소(Undo) 등 다양한 기능을 구현할 수 있습니다.

### 커맨드 패턴의 구성 요소

1. **커맨드(Command)**: 작업 수행에 필요한 인터페이스를 선언합니다.
2. **구체적 커맨드(Concrete Command)**: 커맨드 인터페이스를 구현하고 수신자 객체와 작업을 연결합니다.
3. **수신자(Receiver)**: 실제 작업을 수행하는 객체입니다.
4. **호출자(Invoker)**: 커맨드 객체를 저장하고 실행합니다.
5. **클라이언트(Client)**: 구체적 커맨드 객체를 생성하고 수신자를 지정합니다.

### 커맨드 패턴 예제: 리모컨 시스템
다음은 TV와 조명을 제어하는 리모컨 시스템의 예제입니다.

```javascript
// 커맨드 인터페이스
interface Command {
  execute(): void;
  undo(): void;
}

// 수신자 클래스들
class Device {
  protected isOn = false;
  constructor(protected name: string) {}
  
  on(): void {
    this.isOn = true;
    console.log(`${this.name} 켜짐`);
  }
  
  off(): void {
    this.isOn = false;
    console.log(`${this.name} 꺼짐`);
  }
}

class TV extends Device {
  setVolume(level: number): void {
    console.log(`${this.name} 볼륨: ${level}`);
  }
}

class Light extends Device {}

// 커맨드 구현
class PowerCommand implements Command {
  constructor(
    private device: Device,
    private powerOn: boolean
  ) {}
  
  execute(): void {
    this.powerOn ? this.device.on() : this.device.off();
  }
  
  undo(): void {
    this.powerOn ? this.device.off() : this.device.on();
  }
}

// 리모컨 (호출자)
class Remote {
  private commands: Command[][] = [];
  private lastCommand: Command | null = null;
  
  addButton(onCmd: Command, offCmd: Command): number {
    const slot = this.commands.length;
    this.commands.push([onCmd, offCmd]);
    return slot;
  }
  
  press(slot: number, on: boolean): void {
    if (slot >= this.commands.length) return;
    const cmd = this.commands[slot][on ? 0 : 1];
    cmd.execute();
    this.lastCommand = cmd;
  }
  
  undo(): void {
    this.lastCommand?.undo();
  }
}

// 사용 예시
const tv = new TV("거실TV");
const light = new Light("주방등");

const remote = new Remote();

// 버튼 등록
const tvBtn = remote.addButton(
  new PowerCommand(tv, true),
  new PowerCommand(tv, false)
);

const lightBtn = remote.addButton(
  new PowerCommand(light, true), 
  new PowerCommand(light, false)
);

// 리모컨 사용
remote.press(tvBtn, true);     // 거실TV 켜짐
remote.press(lightBtn, true);  // 주방등 켜짐
remote.undo();                 // 주방등 꺼짐
remote.press(tvBtn, false);    // 거실TV 꺼짐
```

### 커맨드 패턴의 장단점

#### 장점
1. **단일 책임 원칙**: 작업을 수행하는 객체와 작업을 호출하는 객체를 분리합니다.
2. **개방/폐쇄 원칙**: 기존 코드를 수정하지 않고 새로운 커맨드를 추가할 수 있습니다.
3. **작업 취소(Undo)**: 이전 상태를 저장하면 작업 취소 기능을 쉽게 구현할 수 있습니다.
4. **작업 큐 및 로그**: 명령을 객체로 저장하여 큐에 넣거나 로그로 기록할 수 있습니다.
5. **트랜잭션**: 여러 명령을 하나의 트랜잭션으로 묶어 원자적으로 실행하거나 취소할 수 있습니다.

#### 단점
1. **코드 복잡성 증가**: 간단한 작업에도 여러 클래스가 필요하여 코드가 복잡해질 수 있습니다.
2. **클래스 수 증가**: 각 명령마다 별도의 클래스가 필요하여 클래스 수가 많아질 수 있습니다.
