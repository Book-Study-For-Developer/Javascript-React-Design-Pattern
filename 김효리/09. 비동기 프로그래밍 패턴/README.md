## 9.1 비동기 프로그래밍

### ✅ 동기(Synchronous)

- 코드가 **한 줄씩 순서대로 실행**됨
- 하나가 끝날 때까지 다음 줄로 안 넘어감 → **막힘(블로킹)** 발생 가능

### ✅ 비동기(Asynchronous)

- 시간이 오래 걸리는 작업(예: 서버 요청)을 기다리지 않고 다음 코드 실행
- 작업이 끝나면 **콜백이나 프로미스로 결과 전달**

```jsx
function makeRequest(url, callback) {
  fetch(url)
    .then((response) => response.json())
    .then((data) => callback(null, data))
    .catch((error) => callback(error));
}

makeRequest('http://example.com/', (error, data) => {
  if (error) {
    console.error(error);
  } else {
    console.log(data);
  }
});
```

⚠️ 이렇게 콜백을 계속 중첩해서 쓰면 콜백 지옥(callback hell) 발생

→ 읽기 힘들고 유지보수가 어려움

```jsx
async function makeRequest(url) {
  try {
    const response = await fetch(url);
    const data = await response.json();
    console.log(data);
  } catch (error) {
    console.log(error);
  }
}

makeRequest('http://example.com/');
```

## 9.3 프로미스 패턴

프로미스는 Promise 생성자를 사용하여 만들 수 있고, 이 생성자는 함수를 인수로 받는다.

인수로 받는 이 함수는 resolve, reject 두 개의 인수를 전달받는다.

```jsx
function makeRequest(url) {
  return new Promise((resolve, reject) => P
    fetch(url)
      .then(response => response.json())
      .then(data => resolve(data))
      .catch(error => reject(error));
  });
}

makeRequest('http://example.com/')
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

### 9.3.1 프로미스 체이닝

```jsx
function makeRequest(url) {
  return new Promise((resolve, reject) => {
    fetch(url)
      .then((response) => resonse.json())
      .then((data) => resolve(data))
      .catch((error) => reject(error));
  });
}

function processData(data) {
  return processedData;
}

makeRequest('http://example.com/')
  .then((data) => processData(data))
  .then((processedData) => console.log(processedData))
  .catch((error) => console.error(error));
```

### 🧩 유용한 패턴들

| 패턴                     | 설명                                                    |
| ------------------------ | ------------------------------------------------------- |
| `Promise.all([...])`     | 여러 작업을 병렬로 실행 후 모두 완료되면 결과 전달      |
| `Promise.race([...])`    | 제일 먼저 끝나는 하나의 작업 결과만 받음                |
| `Promise.resolve(value)` | 이미 값이 있는 프로미스 즉시 생성                       |
| `Map` + 프로미스         | 요청 결과를 저장해두는 **메모이제이션(캐싱)** 구현 가능 |

### 9.3.6 프로미스 파이프라인

```jsx
function transform1(data) {
  return transformedData;
}

function fransform2(data) {
  return transformedData;
}

makeRequest('http://example.com/')
  .then((data) => pipeline(data).then(transform1).then(transform2))
  .then((transformedData) => console.log(transformedData))
  .catch((error) => console.error(error));
```

- 프로미스와 함수형 프로그래밍 기법을 활용하여 비동기 처리의 파이프라인을 생성한다.
- 🤔 파이프라인의 개념을 잘 모른다….
- `pipeline`은 데이터를 받아서 여러 변환 과정을 **파이프 형태**로 연결

📌 **"파이프라인"이란?**

- 데이터를 **단계적으로 변환**하는 흐름
- 각 단계는 이전 결과를 입력으로 받아 처리

### 9.3.7 프로미스 재시도

- 클로저를 사용하여 retry를 할 수 있다.

### 9.3.8 프로미스 데코레이터

```jsx
function logger(fn) {
  return function (...args) {
    console.log('함수.시작.');
    return fn(...args).then((result) => {
      console.log('함수. 끝.');
      return result;
    });
  };
}

const makeRequestWithLogger = logger(makeRequest);

makeRequestWithLogger('http://example.com/')
  .then((data) => console.log(data))
  .catch((error) => console.error(error));
```

→ 어떤 함수를 감싸서 **공통 로직 추가** 가능 (로깅, 타이머, 등)

## 9.4 async/await 패턴

- `async` 함수는 항상 프로미스를 반환
- `await`는 프로미스를 기다림
- `try/catch`로 오류 처리 가능 → **가독성이 좋고 명확함**

### 9.4.2 비동기 반복 (`for await...of`)

```jsx
async function* createAsyncIterable() {
  yield 1;
  yield 2;
  yield 3;
}

async function main() {
  for await (const value of createAsyncIterable()) {
    console.log(value);
  }
}
```

- `async function*`은 **비동기 제너레이터 함수**
- `for await...of`로 하나씩 값을 **기다리면서 출력** 가능

### 사용 이유?

- 예: **스트리밍**, 대용량 데이터, 파일 읽기 등에서 유용

```jsx
(1) createAsyncIterable() 호출 → async iterator 반환
(2) for await 루프 시작
(3) yield 1 → 출력: 1
(4) yield 2 → 출력: 2
(5) yield 3 → 출력: 3
(6) 순회 종료
```

### 9.4.8 async/await 파이프라인

```jsx
async function transform1(data) {
  return transformedData;
}

async function fransform2(data) {
  return transformedData;
}

async function main() {
  const data = await makeRequest('http://example.com/');
  const transformedData = await pipeline(data)
    .then(transform1)
    .then(transform2);

  console.log(transformedData);
}
```

→ 프로미스 체이닝을 `await` 안에서 사용한 구조

### 9.4.10 async/await 데코레이터

```jsx
function asyncLogger(fn) {
  return async function (...args) {
    console.log("비동기.함수.시작");
    const result = await fn(...args);
    console.log("비동기.함수.끝.");
    return result;
  };
}

@asyncLogger
async function main() {
  const data = await makeRequest("http://example.com/");
  console.log(data);
}
```

- `@asyncLogger`는 `main` 함수를 `asyncLogger(main)`으로 감싼다는 의미.
- 🤔 이건 진짜진짜 처음 보는 문법 …..

GPT 답변:

`@asyncLogger` 문법은 **TypeScript**나 Babel에서만 지원됨

- 일반 JS에서는 `const wrapped = asyncLogger(main);`처럼 사용

(이거 맞나요 ?)

## 9.5 실용적인 예제 더보기

| 예제           | 설명                                       |
| -------------- | ------------------------------------------ |
| HTTP 요청      | fetch, axios 등을 사용                     |
| 파일 읽기/쓰기 | Node.js에서 `fs/promises` 사용             |
| 순차 실행      | `.then()` 또는 `await`로 순서 보장         |
| 병렬 실행      | `Promise.all()`                            |
| 캐싱           | 요청 결과를 `Map`에 저장해 재사용          |
| 이벤트 처리    | `async/await`로 클릭 처리 등               |
| 자동 재시도    | 실패 시 반복 요청 로직 작성                |
| 데코레이터     | 로깅, 측정, 예외 처리 등을 공통적으로 적용 |

## 📌 전체 흐름 요약

| 방식        | 설명                                  | 예                     |
| ----------- | ------------------------------------- | ---------------------- |
| 콜백        | 복잡하고 유지 어려움                  | `fetch(..., callback)` |
| 프로미스    | 더 깔끔하지만 체이닝이 길어질 수 있음 | `.then().catch()`      |
| async/await | 가장 깔끔하고 예외 처리 쉽다          | `await fetch()`        |
| 데코레이터  | 함수에 공통 기능 부여                 | `@logger`              |
