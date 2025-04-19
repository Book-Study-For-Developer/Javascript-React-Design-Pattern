## 안티 패턴이란?

안티 패턴이란 겉으로만 해결책처럼 생긴 패턴을 뜻한다.

안티 패턴의 두 가지 개념

- 문제 상황에 대한 잘못된 해결책
- 문제 상황에서 벗어나 올바른 해결책에 이르는 방법

> [!NOTE]
>
> 🤔 이것도 해석이 와닿지 않아 찾아본 원글
>
> https://en.wikipedia.org/wiki/Anti-pattern
>
> 1. The anti-pattern is a commonly-used process, structure or pattern of action that, despite initially appearing to be an appropriate and effective response to a problem, has more bad consequences than good ones.
>
> 안티패턴은 문제에 대한 적절하고 효과적인 대응책처럼 보이지만, 좋은 결과보다 **나쁜 결과가 더 많은 흔히 사용되는 프로세스, 구조 또는 행동 패턴**입니다
>
> 2. Another solution exists to the problem the anti-pattern is attempting to address. This solution is documented, repeatable, and proven to be effective where the anti-pattern is not.
>
> **안티패턴이 해결하려는 문제에 대한 또 다른 해결책이 존재**합니다. 이 해결책은 문서화되어 있고, 반복 가능하며, 안티패턴이 해결되지 않는 부분에서도 효과가 입증되었습니다.
>
> 해석이 약간 비약되어 있는게 있는 느낌..

## 자바스크립트 안티 패턴

신속한 구현을 위해 임시방편의 코드를 선택할 수 있다.

자바스크립트는 느슨한 타입 언어이기 때문에 특히 더 조심해야 한다.

자바스크립트의 안티패턴 예시)

- 전역 컨텍스트에서 수많은 변수를 정의하여 전역 네임스페이스를 오염시키기
- setTimeout이나 setInterval에 함수가 아닌 문자열을 전달해서 내부적으로 eval()실행되게 하기
- Object 클래스의 프로토타입을 수정하기
- 자바스크립트를 인라인으로 사용하여 유연성 떨어트리기
- document.createElement 대신 document.write 사용하기

<hr/>

🤔 각각의 안티패턴들을 실제 코드의 예시로 살펴보기

1.

```tsx
// 전역 오염 예시
var userName = "John";
var isLoggedIn = true;
var theme = "dark";
function doSomething() {
  console.log("Doing something...");
}

// ⚠️ 전역 스코프가 뒤죽박죽 됨.
// 다른 라이브러리나 코드와 충돌 날 가능성 있음.
```

2.

```tsx
// 문자열로 전달된 코드 (내부적으로 eval 사용)
setTimeout("alert('Hello from string!')", 1000);
// string으로 전달하면 내부적으로 eval을 실행함
setTimeout(() => {
  eval("alert('Hello!')");
}, 1000);
setInterval("console.log('Tick')", 2000);
```

⇒ 스크립트인 문자열을 넘기면 eval을 실행하는지는 처음 알았네요. 😱

3.

```tsx
// Object.prototype 오염시키기
Object.prototype.isEmpty = function () {
  return Object.keys(this).length === 0;
};

const obj = { a: 1 };
console.log(obj.isEmpty()); // false

// ⚠️ 모든 객체에 isEmpty가 생겨버림!
console.log({}.isEmpty()); // true
```

4.

```tsx
<!-- 인라인 JS 사용 -->
<button onclick="alert('Clicked!')">Click me</button>
```

5.

```tsx
// document.write 사용
document.write("<h1>This is dangerous</h1>");
```
