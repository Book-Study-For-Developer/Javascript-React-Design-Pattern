### 알아볼 것

- 자바스크립트에서 볼 수 있는 대표적인 안티 패턴 유형

### 4.1 안티 패턴이란?

- 문제 상황에 대한 잘못된 해결책
- 문제 상황에서 벗어나 올바른 해결책에 이르는 방법
  ![](https://mblogthumb-phinf.pstatic.net/MjAyMDA1MTJfNzAg/MDAxNTg5Mjc0NDU2NTU4.aHiux3u_6yEJ7opMh1pQv4LPQtVurmqYXrN5s6ZtXZQg.qUCaqr0EAyO8C58K69Gu-QtQmphV5P7Ec7mEDeRWTIwg.JPEG.dltpfud00/1589274455408.jpg?type=w800)

어떤 코드도 특정 상황에서는 ‘안티 패턴’이 될 수 있다는 걸 명심하자.

### 4.2 자바스크립트 안티 패턴

- 전역 컨텍스트에서 변수 정의하여 전역 네임스페이스 오염시키기
- setTimeout, setInterval에 함수가 아닌 문자열 전달로 eval() 실행되게 하기
  https://html.spec.whatwg.org/multipage/timers-and-user-prompts.html#dom-settimeout
  > The effect of these steps ensures that the string compilation done by setTimeout() and setInterval() behaves equivalently to that done by eval().
  > 이 단계들의 효과는 setTimeout()과 setInterval()에 의해 수행되는 문자열 컴파일이 eval()과 동일하게 작동하도록 보장합니다.
- Object 클래스의 프로토타입 수정하기 (특히 나쁜 안티 패턴)
- 자바스크립트를 인라인으로 사용하여 유연성 떨어뜨리기
- `document.createElement` 대신 `document.write` 사용하기
