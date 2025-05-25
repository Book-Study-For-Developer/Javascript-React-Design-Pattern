🤔 책의 초판이 2012년이었군요.. 어쩐지

### 자바스크립트의 비동기 특징

아직도 주구장창 듣는 “이벤트 루프” 덕분에 싱글스레드인 자바스크립트에서도 비동기 동작이 가능합니다. (최신에는 Web Worker를 통해 멀티 스레드 구현이 가능)

![image](https://github.com/user-attachments/assets/6e106ef7-0a16-44a3-b69a-b66d49b888d9)

- Call Stack: 자바스크립트 엔진(V8)이 코드 실행을 위해 사용하는 메모리
- Heap: 동적으로 생성된 자바스크립트 객체가 저장되는 공간
- Wep Api: 브라우저에서 제공하는 API 모음 (동기적, 비동기적 모두 포함)
  - DOM: HTML 문서의 구조와 내용을 표현하고 조작할 수 있는 객체
  - XMLHttpRequest: 서버와 비동기적으로 데이터를 교환할 수 있는 객체 (axios가 이를 사용)
  - Timer Api: 일정한 시간 간격으로 함수를 실행시키는 메소드 제공
  - Console API: 개발자 도구에서 콘솔 기능 제공
  - Canvas API: canvas 태그를 통해 그릴수 있도록 하는 메소드 제공
- Callback 큐: 비동기적 작업이 완료되면 실행되는 함수들이 대기하는 큐
  - (macro) Task Queue: `setTimeout`, `setInterval`, `fetch`, `addEventListener` 등의 비동기 함수가 들어가는 큐
  - Microtask Queue: `promise`, `MutationObserver` 비동기 함수가 들어가는 큐
  - 우선순위: Microtask Queue > Task Queue, 각 큐안에 들어가는 순서는 어떤 비동기 작업이냐에 따라 달라짐
  - AnimationFrame Queue: `requestAnimationFrame` 자바스크립트 애니메이션 동작을 제어하는 메서드가 등록되는 큐
- Event Loop: 비동기 함수들을 적절히 실행시키는 관리자

### 이벤트 루프가 동작하는 방식?

이벤트 루프는 비동기 함수 작업을 Web API에 옮기는 역할을 하고, **작업이 완료되면 콜백 큐에 적재한 후 다시 자바스크립트 엔진에 적재하는 역할**을 한다.

이벤트 루프는 Call Stack이 현재 실행중인 작업이 있는지, Task Queue에 대기 중인 작업이 있는지 **계속해서 확인하는 루프**를 돌린다.

```tsx
// 이벤트 루프를 나타내는 슈도 코드
while (queue.waitForMessage()) {
  queue.processNextMessage();
}
```

더 자세히 보고 싶다면?

- 글이 좋아요! ⇒ [https://inpa.tistory.com/entry/🔄-자바스크립트-이벤트-루프-구조-동작-원리](https://inpa.tistory.com/entry/%F0%9F%94%84-%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EB%A3%A8%ED%94%84-%EA%B5%AC%EC%A1%B0-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC)
- 영상이 좋아요! ⇒ https://dev.to/lydiahallie/javascript-visualized-event-loop-3dif

## 프로미스 패턴

🤔 대부분 알고 있는 내용들과 문법이라 처음 본 내용만 작성하였습니다.

### 프로미스 메모이제이션 패턴

프로미스 메모이제이션 패턴은 캐시를 사용하여 프로미스 함수 호출의 결과 값을 저장하는 것 (이를 통해 중복된 요청을 방지할 수 있다.)

```tsx
const cache = new Map();

function memoizedMakeRequest(url) {
	if (cache.has(url)) {
		return cache.get(url);
	}

	return new Promise((resolve, reject) => {
		fetch(url).then(response => response.json())
				.then(data => {
				  // 데이터 요청이 성공하면 cache에 저장
					cache.set(url, data);
					resolce(data);
				})
				.catch(error => reject(error);
	})
}
```

🤔 tanstack query가 하는 역할도 같으려나?

https://github.com/TanStack/query/blob/main/packages/query-core/src/queryCache.ts#L95-L131

코드를 살펴보니 동일하게 map을 사용하고 있고 queryHash값이 있는지로 판별하고 있다.

### 프로미스 재시도

```tsx
function makeRequestWithRetry(url) {
  // 재시도 횟수 저장
  let attempts = 0;

  const makeRequest = () =>
    new Promise((resolve, reject) => {
      fetch(url)
        .then((response) => response.json())
        .then((data) => resolve(data))
        .catch((error) => reject(error));
    });

  const retry = (error) => {
    attempts++;
    if (attempts >= 3) {
      throw new Error("Request failed after 3 attempts.");
    }
    console.log(`Retrying request: attempt ${attempts}`);
    return makeRequest().catch(retry);
  };

  return makeRequest().catch(retry);
}
```

tanstack query에서 retry는 어떻게 할까?

https://github.com/TanStack/query/blob/main/packages/query-core/src/retryer.ts#L167-L203

비슷한 방식으로 진행한다. 😮

### 프로미스 데코레이터

데코레이터 패턴은 고차 함수를 사용하여 프로미스에 적용할 수 있는 데코레이터를 생성한다.

```tsx
function logger(fn) {
  return function (...args) {
    console.log("Starting function...");
    return fn(...args).then((result) => {
      console.log("Function completed.");
      return result;
    });
  };
}

const makeRequestWithLogger = logger(makeRequest);

document.getElementById("fetchWithLogger").addEventListener("click", () => {
  makeRequestWithLogger(apiUrl)
    .then((data) => printResult(data))
    .catch((error) => {
      output.innerText = `Error: ${error.message}`;
    });
});
```

logger라는 데코레이터를 기존 프로미스에 기능을 추가

## Async/Await 패턴

### 비동기 반복

비동기 반복 패턴은 for-await-of 반복문을 사용하여 비동기 반복 기능 객체를 순회할 수 있도록 한다.

```tsx
async function* createAsyncIterable() {
	yeild 1;
	yeild 2;
	yeild 3;
}

async function main() {
	for await (const value of createAsyncIterable()) {
		console.log(value);
	}
}
```

제네레이터 문법을 활용하여 순회를 사용

🤔 이런 패턴은 어디에 쓰이는걸까?

책 코드 예시를 살펴보니 while문을 돌며 innerText를 추가했다.

```tsx
const output = document.getElementById("output");

async function* createAsyncIterable() {
  yield 1;
  yield 2;
  yield 3;
}

async function main() {
  try {
    const asyncIterable = createAsyncIterable();
    const asyncIterator = asyncIterable[Symbol.asyncIterator]();

    while (true) {
      const { done, value } = await asyncIterator.next();

      if (done) {
        break;
      }

      output.innerText += `${value}\n`;
    }
  } catch (error) {
    output.innerText = `Error: ${error.message}`;
  }
}

document.getElementById("startAsyncIteration").addEventListener("click", () => {
  output.innerText = "";
  main();
});
```

이걸 보고 든 생각은 요새 AI가 대답을 스트리밍 형태로 하는데 이럴때도 사용할 듯 하다.

최근에 tanstack query에서 [streamedQuery](https://tanstack.com/query/latest/docs/reference/streamedQuery)가 나왔는데 [공식 문서의 예시](https://tanstack.com/query/latest/docs/framework/react/examples/chat?path=examples%2Freact%2Fchat%2Fsrc%2Fchat.ts)를 살펴보니 실제로 제네레이터로 위와 같은 패턴을 쓰고 있었다! 😱

```tsx
function chatAnswer(_question: string) {
  return {
    async *[Symbol.asyncIterator]() {
      const answer = answers[Math.floor(Math.random() * answers.length)];
      let index = 0;
      while (index < answer.length) {
        await new Promise((resolve) =>
          setTimeout(resolve, 100 + Math.random() * 300)
        );
        yield answer[index++];
      }
    },
  };
}
```
