# 비동기 프로그래밍 패턴

<br />

- 비동기 자바스크립트 프로그래밍은 브라우저가 이벤트에 응답하여 다른 코드를 실행하는 동안, 백그라운드에서 오랜 시간이 걸리는 작업을 수행하게 해줌

<br />

## 9.1 비동기 프로그래밍

### 동기
- 자바스크립트에서 동기 코드는 블로킹 방식으로 실행됨
- 코드에서 다음 줄의 문장은 현재 문장의 실행이 완료된 후에 실행 가능

<br />

### 비동기
- 비동기 코드는 논블로킹 방식으로 실행됨
- 자바스크립트 엔진은 현재 실행 중인 코드가 다른 작업을 기다리는 동안 백그라운드에서 해당 비동기 코드를 실행할 수 있음
- 네트워크 요청, 데이터베이스 읽기/쓰기, 입출력 작업에 적합
- async, await, promise는 비동기 코드를 동기 코드처럼 작동하도록 작성할 수 있음

<br />

## 9.2 배경
- 자바스크립트에서 콜백 함수는 다른 함수에 인수로 전달되어, 비동기 작업이 완료된 후 실행됨
- 콜백 함수는 주로 네트워크 요청이나 사용자 입력과 같은 비동기 작업의 결과를 처리하기 위해 사용됨
  
<br />

## 9.3 프로미스 패턴
- 프로미스는 비동기 작업의 결과를 나타내는 객체
- 대기, 완료, 거부 세 가지 상태를 가짐

<br />

### 9.3.1 프로미스 체이닝
여러 개의 프로미스를 함께 연결할 수 있음

<br />

### 9.3.2 프로미스 에러 처리
catch 메서드를 사용하여 프로미스 체인의 실행 중에 발생할 수 있는 에러 처리

<br />

### 9.3.3 프로미스 병렬 처리
Promise.all 메서드를 사용하여 여러 프로미스를 동시에 실행할 수 있게 해줌

<br />

### 9.3.4 프로미스 순차 실행
Promise.resolve 메서드를 사용하여 프로미스를 순차적으로 실행할 수 있게 해줌


> ❓ **순차 실행을 위해 resolve 메서드를 사용하는 것이 맞을까?** 
> - 프로미스가 아닐 때 해당 값을 resolve한 새로운 프로미스를 반환하는 역할
> - 빈 프로미스를 만들어줌으로써 프로미스 체이닝을 시작할 수 있는 기반을 만들어주기는 하지만 순차 실행을 위한 메서드로 설명하는 것이 맞는지 잘 모르겠음 
> - `Array.prototype.reduce()` 메서드와 `async/await` + `for...of`을 사용하여 순차 실행이 가능하다고 함
> - forEach()는 순차 실행 보장 x
> ```ts
> const urls = ['a', 'b', 'c'];
> urls.reduce((acc, url) => {
>   return acc.then(() => fetch(url));
> }, Promise.resolve());
> ```
> ```ts
> async function runSequentially(tasks) {
>   for (const task of tasks) {
>     await task(); // task는 함수: () => Promise
>   }
> }
```

<br />

### 9.3.5 프로미스 메모이제이션
캐시를 사용하여 프로미스 함수 호출의 결과값을 저장함

```ts
const cache = new Map();

function memoizedMakeRequest(url) {
  if (cache.has(url)) return cache.get(url);

  return new Promise((resolve, reject) => {
    fetch(url)
    .then(response => response.json())
    .then(data => {
      cache.set(url, data);
      resolve(data);  
    })
    .catch(error => reject(error));
  })
}
```
```ts
const button = document.querySelector('button');

button.addEventListner('click', () => {
  memoizedMakeRequest('http://example.com/')
  .then(data => console.log(data))
  .catch(error => console.error(error));
})
```
- 버튼을 클릭하면 memoizedMakeRequest 함수가 호출됨
- 요청한 URL이 이미 캐시에 존재한다면 캐시된 데이터가 반환됨
- 캐시에 존재하지 않는다면 새로운 요청이 발생하고, 나중에 같은 요청이 들어올 때를 대비해 캐시에 저장됨

<br />

### 9.3.6 프로미스 파이프라인
프로미스와 함수형 프로그래밍 기법을 활용해 비동기 처리의 파이프라인 생성

<br />

### 9.3.7 프로미스 재시도
프로미스가 실패할 때 다시 시도할 수 있음

<br />

### 9.3.8 프로미스 데코레이터
고차 함수를 사용하여 프로미스에 적용할 수 있는 데코레이터를 생성함

<br />

### 9.3.9 프로미스 경쟁
- 여러 프로미스를 동시에 실행하고 가장 먼저 완료되는 프로미스의 결과 반환
- Promise.race()

<br />

## 9.4 async/await 패턴
비동기 코드를 동기 코드처럼 작성할 수 있게 해주는 자바스크립트 기능
  
<br />

### 9.4.2 비동기 반복
for-await-of 반복문을 사용하여 비동기 반복 가능 객체를 순회할 수 있도록 함

```ts
async function createAsyncIterable() {
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

<br />

### 9.4.3 비동기 에러 처리
try-catch 블록을 사용하여 에러 처리


<br />

### 9.4.4 비동기 병렬
Promise.all 메서드를 사용하여 여러 비동기 작업을 동시에 실행할 수 있게 함

