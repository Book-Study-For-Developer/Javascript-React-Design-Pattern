# 5. 최신 자바스크립트 문법과 기능

## 5.1 애플리케이션 분리의 중요성

### 모듈형 자바스크립트

모듈형 자바스크립트는 느슨한 결합을 통해 의존성을 낮추어 애플리케이션의 유지보수를 용이하게 만듭니다. 모듈화된 코드는 다음과 같은 이점을 제공합니다.

- 재사용 가능한 코드 블록 생성
- 코드의 캡슐화와 범위 제한
- 의존성 관리 용이
- 코드 구조화 개선

ES5 이전의 JavaScript는 모듈을 자연스럽게 가져오는 방법이 없었습니다. 이로 인해 전역 네임스페이스 오염이나 의존성 관리 문제가 자주 발생했습니다. 이러한 문제를 해결하기 위해 AMD와 CommonJS 모듈 패턴이 등장했으며, 이는 초기 자바스크립트에서 모듈화 구현을 위해 가장 많이 사용된 패턴이 되었습니다.

ES6(ES2015) 및 이후 버전(ESNext)에서 모듈 관련 기능이 정식으로 추가되었습니다. ECMAScript의 문법과 기능 정의를 담당하는 TC39 표준 기구는 대규모 개발을 위한 자바스크립트의 진화 과정을 주시해 왔으며, 모듈형 자바스크립트 작성을 위한 더 나은 기능의 필요성을 인지하고 있었기에 이러한 발전이 가능했습니다.

## 5.2 모듈 가져오기와 내보내기

자바스크립트 모듈 시스템의 핵심은 `import`와 `export` 문입니다.

- `import` 문은 내보내기된 모듈을 지역 변수로 가져오거나 이름을 바꿔서 가져올 수 있습니다.
- `export` 문은 지역 모듈을 외부에서 읽을 수 있지만 수정할 수 없도록 만들어주며, 이름을 바꿔서 내보내는 것도 가능합니다.

### 모듈 파일 확장자

`.mjs`는 모듈 파일과 기존 스크립트를 구분하기 위해 쓰이는 모듈 전용 확장자입니다. 이 확장자를 사용해 런타임 및 빌드 도구에 해당 파일이 모듈임을 명시적으로 알릴 수 있습니다.

### 모듈 예제

```javascript
// staff.mjs - 모듈 내보내기 예제
export const baker = {
    bake(item) {
        console.log(`Woo! I just baked ${item}`);
    }
};
```

```javascript
// cakeFactory.mjs - 모듈 가져오기 및 내보내기
import baker from "/modules/staff.mjs";

export const oven = {
    makeCupcake(toppings) {
        baker.bake("cupcake", toppings);
    },
    makeMuffin(size) {
        baker.bake("muffin", size);
    }
};
```

```javascript
// baker.mjs - 모듈 사용 예제
import { cakeFactory } from "/modules/cakeFactory.mjs";

cakeFactory.oven.makeCupcake("sprinkles");
cakeFactory.oven.makeMuffin("large");
```

### 모듈 사용 방법

1. **단번에 내보내기**: 여러 값을 한 번에 내보낼 수 있습니다.
   ```javascript
   export { func1, func2, class1, variable1 };
   ```

2. **필요한 모듈만 가져오기**: 모듈에서 필요한 부분만 선택적으로 가져올 수 있습니다.
   ```javascript
   import { func1, variable1 } from "./module.js";
   ```

3. **브라우저에 모듈임을 알리기**: `script` 태그에 `type="module"`을 명시하여 브라우저에게 해당 스크립트가 모듈임을 알립니다.
   ```html
   <script type="module" src="app.js"></script>
   ```

4. **대체 스크립트 제공**: `nomodule` 속성은 브라우저에게 해당 스크립트가 모듈이 아님을 알려줍니다. 이는 모듈 문법을 사용하지 않는 대체 스크립트에 유용하며, 모듈을 지원하지 않는 레거시 브라우저에서도 기능이 제대로 작동할 수 있도록 합니다.
   ```html
   <script type="module" src="app.js"></script>
   <script nomodule src="app-legacy.js"></script>
   ```
   최신 브라우저는 폴리필이 필요하지 않지만, 레거시 브라우저에서는 트랜스파일된 코드가 필요하기에 대체 스크립트가 사용될 수 있습니다.

## 5.3 모듈 객체

모듈의 모든 내보내기를 하나의 객체로 가져올 수 있습니다.

```javascript
import * as Staff from "/modules/staff.mjs";

// 사용 예시
Staff.baker.bake("cookies");
Staff.manager.schedule();
```

이렇게 하면 하나의 네임스페이스 객체를 통해 모듈의 모든 기능에 접근할 수 있어, 복잡한 모듈에서 여러 기능을 사용할 때 유용합니다.

## 5.4 외부 소스로부터 가져오는 모듈

### 정적 가져오기

ES2015부터는 외부 소스에서 원격 모듈을 쉽게 가져올 수 있게 되었습니다.

```javascript
import { cakeFactory } from "https://example.com/modules/cakeFactory.mjs";
```

이 방식은 CDN이나 외부 호스팅된 라이브러리를 직접 가져올 때 유용합니다. 하지만 네트워크 의존성과 보안 이슈에 주의해야 합니다.

## 5.5 정적으로 모듈 가져오기

정적 가져오기는 메인 코드 실행 전에 모듈을 다운로드하고 실행해야 하기 때문에 성능 문제가 생길 수 있습니다. 특히 대규모 애플리케이션에서는 초기 로딩 시간이 길어질 수 있어 사용자 경험에 영향을 줄 수 있습니다.

```javascript
// 정적 가져오기 - 페이지 로드 시 즉시 로드됨
import { heavyFunction } from "./heavyModule.js";
```

## 5.6 동적으로 모듈 가져오기

동적 모듈 가져오기는 지연 로딩(Lazy loading)을 구현하는 방법입니다. 링크나 버튼 클릭 시 필요한 모듈만 로드하게 만들어 초기 로딩 시간을 줄일 수 있습니다.

`import(url)`은 요청된 모듈의 네임스페이스 객체에 대한 프로미스 객체를 반환합니다. 이 프로미스 객체는 모듈 자체와 모든 모듈 의존성을 가져온 후 인스턴스화하고 평가한 뒤에 완료됩니다.

```javascript
form.addEventListener("submit", e => {
    e.preventDefault();
    import("/modules/cakeFactory.js")
        .then((module) => {
            module.oven.makeCupcake("sprinkles");
            module.oven.makeMuffin("large");
        })
        .catch(error => {
            console.error("모듈 로딩 중 오류 발생:", error);
        });
});
```

동적 가져오기는 `async/await` 구문과 함께 사용할 수도 있습니다.

```javascript
async function loadAndUseCakeFactory() {
    try {
        let module = await import("/modules/cakeFactory.js");
        module.oven.makeCupcake("chocolate");
    } catch (error) {
        console.error("모듈 로딩 실패:", error);
    }
}
```

이 방식을 사용하면 모듈이 실제로 필요할 때만 다운로드되고 실행됩니다.

### 5.6.1 사용자 상호작용에 따라 가져오기

동적 가져오기는 사용자 상호작용에 따라 필요한 기능만 로드할 때 매우 유용합니다. 예를 들어, 정렬 기능이 필요할 때만 관련 라이브러리를 로드할 수 있습니다.

```javascript
// 정렬 버튼 클릭 시 lodash.sortBy 모듈 동적 로드
sortButton.addEventListener('click', async () => {
    try {
        const { sortBy } = await import('https://cdn.jsdelivr.net/npm/lodash-es@4.17.21/sortBy.js');
        const sortedItems = sortBy(items, 'name');
        renderItems(sortedItems);
    } catch (error) {
        console.error('정렬 모듈 로딩 실패:', error);
    }
});
```

### 5.6.2 화면에 보이면 가져오기

사용자가 항상 페이지의 모든 부분을 보는 것은 아니기 때문에, 화면에 실제로 보이는 컴포넌트의 모듈만 로드하는 것이 효율적입니다. IntersectionObserver API를 사용하면 컴포넌트가 화면에 보이는지 감지할 수 있고, 이를 통해 모듈을 동적으로 로드할 수 있습니다.

```javascript
// 지연 로딩을 위한 Intersection Observer 사용 예제
const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
        if (entry.isIntersecting) {
            const lazyComponent = entry.target;
            
            import(`./components/${lazyComponent.dataset.component}.js`)
                .then(module => {
                    const component = new module.default(lazyComponent);
                    component.render();
                    
                    observer.unobserve(lazyComponent);
                })
                .catch(err => console.error('컴포넌트 로딩 오류:', err));
        }
    });
}, {
    rootMargin: '100px' // 요소가 화면에 나타나기 100px 전에 미리 로드 시작
});

// 지연 로드할 컴포넌트들 관찰 시작
document.querySelectorAll('.lazy-component').forEach(component => {
    observer.observe(component);
});
```

## 5.7 서버에서 모듈 사용하기

Node.js 15.3.0 버전 이상에서는 ECMAScript 모듈을 정식으로 지원합니다. 모듈 기능은
정식으로 릴리즈되었고 npm 패키지 생태계와 호환됩니다.

Node.js에서 모듈을 사용하는 방법

1. **package.json에 모듈 타입 선언**
   ```json
   {
     "name": "my-package",
     "type": "module"
   }
   ```

2. **파일 확장자로 구분**
   Node.js는 `type`이 `module`로 설정되어 있다면 `.mjs`와 `.js`로 끝나는 파일을 자바스크립트 모듈로 취급합니다. 반면 `.cjs` 확장자는 항상 CommonJS 모듈로 취급됩니다.

3. **ES 모듈 사용 예시**
   ```javascript
   // main.mjs
   import { readFile } from 'fs/promises';

   const content = await readFile('./package.json', 'utf8');
   console.log(JSON.parse(content));
   ```

## 5.8 모듈을 사용하면 생기는 이점

모듈 시스템은 다음과 같은 다양한 이점을 제공합니다.

1. **한 번만 실행된다**: 의존성 트리의 가장 내부에 위치한 모듈이 먼저 실행됩니다. 이는 의존성 관리를 예측 가능하게 만들고, 초기화 순서 문제를 해결합니다.

2. **자동으로 지연 로드된다**: 일반 스크립트 파일은 즉시 로드되지 않게 하려면 `defer` 속성을 붙여야 하지만, 모듈은 자동으로 지연되어 로드됩니다. 필요에 따라 `async` 속성을 추가하여 모듈이 문서 파싱을 차단하지 않고 비동기적으로 로드되도록 할 수도 있습니다.
   ```html
   <script type="module" src="app.js"></script> <!-- 자동으로 defer와 같은 효과 -->
   <script type="module" async src="analytics.js"></script> <!-- 비동기 로드 -->
   ```

3. **유지보수와 재사용이 쉽다**: 코드를 기능별로 분리하여 작성하면 각 모듈은 한 가지 일만 담당하게 되어 유지보수와 재사용이 용이해집니다.

4. **네임스페이스를 제공한다**: 모듈은 자체 네임스페이스를 가지므로 전역 네임스페이스 오염 문제를 해결합니다. 이는 대규모 애플리케이션에서 이름 충돌 문제를 방지합니다.

5. **사용하지 않는 코드를 제거한다**: 빌드 도구는 모듈 의존성 트리를 분석하여 실제로 사용하지 않는 코드(데드 코드)를 제거할 수 있습니다. 이는 트리 쉐이킹(tree shaking)이라고 하며, 번들 크기를 최적화하는 데 도움이 됩니다.

## 5.9 생성자, 게터, 세터를 가진 클래스

ES2015에서 추가된 자바스크립트의 클래스는 `class` 키워드를 통해 사용할 수 있습니다. 이전에는 프로토타입 상속을 통해 클래스와 유사한 기능을 구현했지만, ES2015 이상에서는 모든 것을 함수로 만드는 것을 피하고 새로운 식별자를 도입했습니다.

### 클래스 기본 문법

```javascript
class Cake {
    // 생성자 메서드
    constructor(flavor, size) {
        this.flavor = flavor;
        this.size = size;
        this._price = 0; // 내부 프로퍼티
    }
    
    // 게터 메서드
    get price() {
        return this._price;
    }
    
    // 세터 메서드
    set price(value) {
        if (value < 0) {
            throw new Error('가격은 음수가 될 수 없습니다');
        }
        this._price = value;
    }
    
    // 인스턴스 메서드
    describe() {
        return `${this.size} ${this.flavor} 케이크, 가격: ${this.price}원`;
    }
}

const chocolateCake = new Cake('초콜릿', '대형');
chocolateCake.price = 15000; // 세터 호출
console.log(chocolateCake.price); // 게터 호출: 15000
console.log(chocolateCake.describe()); // 대형 초콜릿 케이크, 가격: 15000원
```

### 클래스 상속

자바스크립트의 클래스는 프로토타입 기반이며, `extends` 키워드를 사용하여 상속을 구현할 수 있습니다.

```javascript
class SpecialCake extends Cake {
    constructor(flavor, size, occasion) {
        // 부모 클래스 생성자 호출
        super(flavor, size);
        this.occasion = occasion;
        this.price = 20000; // 기본 가격 설정
    }
    
    // 메서드 오버라이딩
    describe() {
        return `${this.occasion}용 ${super.describe()}`;
    }
}

const birthdayCake = new SpecialCake('바닐라', '중형', '생일');
console.log(birthdayCake.describe()); // 생일용 중형 바닐라 케이크, 가격: 20000원
```

`super` 키워드는 부모 클래스의 생성자와 메서드를 호출할 수 있게 해줍니다. 상속 패턴을 사용할 때 특히 유용합니다.

### 비공개 클래스 멤버

최신 자바스크립트(ES2020 이상)에서는 클래스 내부 멤버를 비공개로 정의할 수 있습니다. 프로퍼티나 메서드 이름 앞에 `#` 기호를 붙여 비공개 멤버를 만들 수 있습니다.

```javascript
class BankAccount {
    #balance = 0; // 비공개 프로퍼티
    
    constructor(owner) {
        this.owner = owner;
    }
    
    #validateAmount(amount) { // 비공개 메서드
        if (amount <= 0) throw new Error('금액은 양수여야 합니다');
    }
    
    deposit(amount) {
        this.#validateAmount(amount);
        this.#balance += amount;
        return this.#balance;
    }
    
    withdraw(amount) {
        this.#validateAmount(amount);
        if (amount > this.#balance) throw new Error('잔액이 부족합니다');
        this.#balance -= amount;
        return this.#balance;
    }
    
    get balance() {
        return this.#balance;
    }
}

const account = new BankAccount('홍길동');
account.deposit(10000);
console.log(account.balance); // 10000
// console.log(account.#balance); // 오류: 비공개 필드에 직접 접근 불가
```

### 정적 메서드와 프로퍼티

`static` 키워드를 사용하여 정적 메서드와 프로퍼티를 정의할 수 있습니다. 정적 멤버는 클래스를 초기화하지 않고도 사용할 수 있습니다.

```javascript
class MathUtils {
    static PI = 3.14159;
    
    static square(x) {
        return x * x;
    }
    
    static cube(x) {
        return x * x * x;
    }
}

console.log(MathUtils.PI); // 3.14159
console.log(MathUtils.square(4)); // 16
console.log(MathUtils.cube(3)); // 27
```

## 5.10 자바스크립트 프레임워크와 클래스

현대 자바스크립트 프레임워크에서 클래스는 다양하게 활용됩니다. 특히 React에서는 클래스 컴포넌트와 함수형 컴포넌트 두 가지 방식을 모두 지원합니다.

React Hooks는 함수형 컴포넌트에서 클래스를 사용하지 않고도 React의 상태와 라이프사이클 기능을 다룰 수 있도록 만들어졌습니다. 하지만 클래스 컴포넌트도 여전히 많은 프로젝트에서 사용되고 있습니다.

### React 클래스 컴포넌트 예시

```jsx
import React, { Component } from 'react';

class Counter extends Component {
    constructor(props) {
        super(props);
        this.state = {
            count: 0
        };
        this.increment = this.increment.bind(this);
    }
    
    increment() {
        this.setState(state => ({
            count: state.count + 1
        }));
    }
    
    componentDidMount() {
        console.log('컴포넌트가 마운트되었습니다');
    }
    
    render() {
        return (
            <div>
                <p>현재 카운트: {this.state.count}</p>
                <button onClick={this.increment}>증가</button>
            </div>
        );
    }
}
```

### React 함수형 컴포넌트와 Hooks 예시

```jsx
import React, { useState, useEffect } from 'react';

function Counter() {
    const [count, setCount] = useState(0);
    
    useEffect(() => {
        console.log('컴포넌트가 마운트되었습니다');
    }, []);
    
    return (
        <div>
            <p>현재 카운트: {count}</p>
            <button onClick={() => setCount(count + 1)}>증가</button>
        </div>
    );
}
```

클래스 기반 컴포넌트는 더 많은 보일러플레이트 코드가 필요하지만, 함수형 컴포넌트와 Hooks는 더 간결하고 읽기 쉬운 코드를 작성할 수 있게 해줍니다. 두 방식 모두 자바스크립트의 현대적 기능을 활용하여 웹 애플리케이션을 구축하는 데 사용됩니다.

