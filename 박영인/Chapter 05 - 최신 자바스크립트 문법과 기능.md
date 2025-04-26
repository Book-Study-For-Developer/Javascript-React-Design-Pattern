## 5.1 애플리케이션 분리의 중요성
자바스크립트 모듈은 ES2015(ES6) 출시와 함께 표준화되었다.
애플리케이션을 모듈형으로 잘게 분리된 모듈로 구성하면, 느슨한 결합이 이루어진다. 이 느슨한 결합은 의존성을 낮추어 애플리케이션의 유지보수를 용이하게 만들어 준다.


## 5.2 모듈 가져오기와 내보내기
모듈을 사용하면 독립적인 단위로 코드를 분리할 수 있다. 그리고 코드의 재사용성을 높여 다른 애플리케이션에도 동일한 기능을 적용할 수 있게 한다.

import 문을 통해 내보내기 된 모듈을 가져올 수 있으며, export 문을 통해 지역 모듈을 외부에서 읽을 수 있게 된다.

> 기본 import/export 코드 예시: 간단한 import/export라 생략😙

파일 끝에서 내보내고 싶은 모듈을 객체로 정리하여 하나의 export 문으로 내보낼 수 있다.
```js
const baker = {
	// ...
}
const pastryChef = {
	// ...
}
const assistant = {
	// ...
}

export {baker, pastryChef, assistant};
```

가져올 땐 사용하고 싶은 모듈만을 가져오는 것도 가능하다.
```js
import {baker, assistant} from "/modules/staff.mjs"
```

script 태그에서 type에 "module"을 명시하여 브라우저에게 모듈임을 알릴 수 있다.
모듈을 지원하지 않는 구형 브라우저는 type="module"을 인식하지 못하고 무시한다. 이 경우를 대비해 nomodule 속성을 사용하여 대체 스크립트를 제공할 수 있다.

```html
<script type="module" src="main.mjs"></script>
<script nomodule src="fallback.js"></script>
```



## 5.3 모듈 객체
`* as 객체 이름` 방식으로 모듈을 객체 하나로 가져올 수 있다. 
Staff 라는 네임 스페이스로 관리 가능!

```js
import * as Staff from "/modules/staff.mjs";

export const oven = {
    makeCupcake(toppings) {
	    Staff.baker.bake("cupcake", toppings);
    },
    makePastry(mSize) {
	    Staff.pastryChef.make("pastry", type);
    }
}
```


## 5.4 외부 소스로부터 가져오는 모듈
import를 통해 외부 소스로부터 모듈을 쉽게 가져올 수 있다.

```js
import {cakeFactory} from "https://example.com";

cakeFactory.oven.makeCupcake("sprinkels");
```


## 5.5 정적으로 모듈 가져오기
위의 예시들 같이 기본적인 import 방식은 **정적 가져오기**다.
정적 가져오기의 경우, 초기 페이지 로딩 시 모듈을 함께 다운로드하고 코드를 미리 로드하므로 성능에 영향을 줄 수 있다.


## 5.6 동적으로 모듈 가져오기
모듈을 동적으로 필요한 시점에 로드하면 초기 로딩 시간을 줄일 수 있는 장점이 있다.
주로 사용자와의 상호작용이 발생하거나, 요소가 화면에 나타날 때 모듈을 동적으로 로드한다.

동적 가져오기는 `import(url)`과 같은 함수 형태로 사용되며, Promise 객체를 반환한다.  
이 방식을 사용하면 모듈을 코드 실행 중 필요한 시점에 비동기로 로드할 수 있으며,  
로딩이 완료된 후 `.then()` 또는 `await`를 통해 모듈 내용을 사용할 수 있다.

```js
form.addEventListener("submit", e => {
	e.preventDefault()
	import("/modules/cakeFactory.js")
		.then((module) => {
			module.oven.makeCupcake("sprinkles")
			module.oven.makeMuffin("large")
	})
})

let module = await import("/modules/cakeFactory.js")
```


### 5.6.1 사용자 상호작용에 따라 가져오기
다이얼로그나 비디오 같은 기능은 페이지 로드 시점에 필요하지 않으니 사용자 상호작용에 따라 모듈을 로드하는 것이 좋다.

```js
btn.addEventListener("click", e => {
	e.preventDefault()
	import("lodash.sortby")
		.then((module) => module.default)
		.then(sortInput())
		.catch(err => { console.log(err) })
})
```

> [!NOTE]
> Q. 사용자와 상호작용하는 시점마다 모듈을 불러오면 동작이 느려지는 문제는 없나?
> 
> A. 매번 로드되지 않고, 처음 로드 이후로는 캐싱된다!
> 사용자가 접근할 가능성이 있을 때(마우스 올릴 때 등) 모듈을 미리 로드하여 딜레이를 느끼지 못하게 처리한다.


### 5.6.2 화면에 보이면 가져오기
스크롤에 따라 나타나는 기능처럼, 처음에는 숨겨져 있다가 나중에 노출되는 요소의 경우  
해당 요소가 화면에 나타날 때 모듈을 로드하는 것이 효율적이다.  
이때 IntersectionObserver API를 사용하면 요소가 화면에 보이는지를 감지할 수 있다.



## 5.7 서버에서 모듈 사용하기
node 15.3.0 이상 부터 자바스크립트 모듈을 지원한다.
node는 type이 module이면 .mjs, .js 파일을 자바스크립트 모듈로 인식한다.


## 5.8 모듈을 사용하면 생기는 이점
##### 한번만 실행된다.
모듈 스크립트는 처음 한번만 실행된다. 
하나의 모듈을 여러 곳에서 import 하더라도, 해당 모듈을 **한 번만 평가하고 캐싱**한다.

모듈 의존성 트리의 가장 내부에 위치한 모듈이 먼저 로드/평가 된다. 이후 해당 모듈에 의존하는 상위 모듈이 순차적으로 평가된다.
이러한 방식 덕분에 상위 모듈은 하위 모듈이 준비된 상태에서 안전하게 사용할 수 있다!!

##### 자동으로 지연 로드된다.
HTML에서 `<script type="module">`을 사용할 경우, 브라우저는 해당 스크립트를 **자동으로 지연(=비동기) 로드**하게 된다.

##### 유지보수와 재사용이 쉽다.
모듈은 독립적으로 실행될 수 있는 코드 조각으로 관리되어, 여러 다른 함수에서 동일한 코드를 재사용할 수 있다.

##### 네임스페이스를 제공한다.
모듈은 관련된 변수와 상수들을 자체적인 네임스페이스 내에서 정의하므로, 전역 스코프를 오염시키지 않고 충돌을 방지할 수 있다.

##### 사용하지 않는 코드를 제거한다.
모듈을 통해 코드를 가져오면 웹팩, 롤업 같은 번들러를 써서 사용하지 않는 모듈을 자동으로 제거(트리쉐이킹)할 수 있다.


> [!NOTE]
> **모듈 캐싱**
> 브라우저나 런타임(Node.js 등)이 한 번 로드하고 평가한 모듈을 메모리에 저장(캐싱) 해두고, 같은 모듈을 다시 import 할 때 재평가하지 않고 캐시된 결과를 재사용하는 동작을 말합니다.
> <br>
> **어떤 값이 캐싱되는가?**
> - 모듈 스코프 내의 실행 결과
> 	- 변수, 함수, 클래스 선언
> 	- 실행 시점에 수행된 부수효과(예: console.log, DOM 조작, 초기화 등)
> - export된 값 → 값 자체가 아닌 참조가 전달됨(동일 메모리 참조를 공유)
> 



## 5.9 생성자, 게터, 세터를 가진 클래스
ES2015에서 클래스가 추가되었다.

class에서 getter, setter는 다음과 같이 동작한다. 속성처럼 사용하지만 내부적으론 함수로 실행된다.
```js
class Cake {
  constructor(name) {
    this._name = name;
  }

  get name() {
    console.log('getter 호출');
    return this._name;
  }

  set name(value) {
    console.log('setter 호출');
    this._name = value;
  }
}

const cake = new Cake('chocolate');
console.log(cake.name);     // getter 호출 → chocolate
cake.name = 'cheese';       // setter 호출
```

`extends`와 `super` 키워드를 사용하여 클래스 상속과 부모 클래스의 생성자 또는 메서드를 호출할 수 있다.  

`#` 기호를 속성 앞에 붙이면 해당 속성은 클래스 외부에서 접근할 수 없는 **private 멤버**가 된다. 주로 보안이나 캡슐화가 중요해서 클래스 내부 상태를 **외부에서 직접 바꾸지 못하도록** 보호하고 싶을 때 사용한다.

또한, `static` 키워드를 사용하면 인스턴스가 아닌 클래스 자체에 속한 **정적 메서드나 프로퍼티**를 정의할 수 있다. 클래스 관련 공통 유틸함수나 모든 인스턴스가 공유해야하는 값을 저장할 때 사용한다.

> 기본적인 class 문법이라 예시는 생략! 😙


## 5.10 자바스크립트 프레임워크와 클래스
react hooks가 나오면서 클래스를 사용하지 않고도 컴포넌트 상태와 라이프사이클을 관리할 수 있게 되었다.



