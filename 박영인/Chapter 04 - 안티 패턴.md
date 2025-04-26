## 4.1 안티 패턴이란?
**안티패턴**이란, 처음에는 효과적인 해결책처럼 보이지만,
시간이 지나면서 **유지보수나 확장성 측면에서 문제를 일으키는 잘못된 패턴**이다.

어떤 설계 방식이 안티패턴인지 빠르게 인지할 수 있다면, 잘못된 디자인 패턴을 도입하는 실수를 줄일 수 있다.  
따라서 안티패턴은 반면교사로 삼을 수 있도록 문서화해 두는 것이 중요하다.

### 사례
#### 🎬 초기 상황
처음에는 application의 비즈니스 로직을 담은 useApplicationService라는 커스텀 훅을 projectId, applicationId를 훅 선언 시점에 넘기는 방식으로 설계했다. 이전에는 application 관리 페이지가 따로 존재하였고, 그렇기 때문에 이 구조는 하나의 application을 단일하게 다룰 때는 직관적이고 편리했다.

```js
const { update, delete } = useApplicationService({ projectId, applicationId });
```

#### ⚠️ 문제 발생
하지만 이후 UI가 개편되면서, 한 페이지에서 **여러 개의 application을 동시에 수정하거나 삭제**해야 하는 요구사항이 생겼다.
이때 훅이 **하나의 applicationId로 고정되어 있는 구조**라서, 매번 훅을 새로 생성하거나 임시로 applicationId를 갈아끼우는 방식으로 우회해야 했고, 코드도 복잡해졌다.

#### ❌ 왜 안티패턴인가?
- **훅의 사용 범위가 지나치게 제한적이다**
  특정 `applicationId`에 고정된 구조는, 복수의 항목을 다뤄야 하는 UI에서 활용이 어렵다.
- **요구사항 변화에 유연하게 대응할 수 없다**
  UI 요구사항이 바뀔 경우 훅 전체를 수정하거나 비정상적인 우회 로직이 생기기 쉽다.

#### ✅ 교훈
훅 설계 시 **입력값을 선언 시점에 고정하는 방식은 변화에 취약하다.** 
UI는 언제든지 변할 수 있으니 유연성 있는 설계를 해야한다!!
반복적이고 다양한 개체를 다루는 상황에서는 **실행 시점에 동적으로 값을 전달할 수 있도록 유연하게 설계하는 것이 바람직하다.**

```js
const { deleteApplication } = useApplicationService();
deleteApplication(applicationId);
```

## 4.2 자바스크립트 안티 패턴
#### 전역 컨텍스트에서 수많은 변수를 정의하여 전역 네임스페이스를 오염 시키기
 ```js
// 전역 변수
var user = 'admin';
function login() { ... }
function logout() { ... }
var count = 0;
```
위 코드가 다른 스크립트와 합쳐지며 다른 선언들과 충돌이 날 수 있다.

#### setTimeout이나 setInterval에 함수가 아닌 문자열을 전달해서 내부적으로 eval() 실행되게 하기
```js
setTimeout("alert('Hello')", 1000);
setInterval("doSomething()", 2000);
```
문자열로 넘기면 내부에서 eval로 실행이 된다.
보안, 성능, 디버깅 면에서 문제가 된다!!

> 문자열 넘기면 eval로 실행이 되다니..!

#### Object 클래스의 프로토타입을 수정하기
```js
Object.prototype.sayHello = function () {
  console.log('Hello!');
};

const user = { name: 'Alice' };
// 예상 가능
user.sayHello(); // 실행됨

// ❌ 예상 불가능..
for (let key in user) {
  console.log(key); // 'name', 'sayHello' 같이 예상치 못한 키까지 순회됨
}
```

#### 자바스크립트를 인라인으로 사용하여 유연성을 떨어뜨리기
```html
<button onclick="alert('Hello')">Click Me</button>
```
수정이 복잡해진다. 재사용이 불가하다. 가독성이 떨어진다.

#### document.write 사용하기
document.write()는 브라우저가 HTML을 파싱하는 도중에 **직접 문서를 덮어쓰는 방식으로 출력**을 삽입하는 API이다.

```html
<script>
  document.write('<h1>Hello World</h1>');
</script>
```

HTML 파싱 도중 document.write()가 실행되면 그 동안 파싱이 중단되며 렌더링이 차단된다. -> 성능 저하

> [!NOTE]
> document.createElement()는 렌더링 차단이 없다!
비동기적으로 HTML 파싱을 마친 후 레이아웃/페인트 주기에 DOM으로 반영된다.