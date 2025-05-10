### 5.1 애플리케이션 분리의 중요성

자바스크립트 모듈에 대한 대표적인 아티클 …

[JavaScript 번들러로 본 조선시대 붕당의 이해 - 재그지그의 개발 블로그](https://wormwlrm.github.io/2020/08/12/History-of-JavaScript-Modules-and-Bundlers.html)

### 5.2 모듈 가져오기와 내보내기

```jsx
// staff.mjs

const baker = {};
const pastryChef = {};
const assistant = {};

export { baker, pastryChef, assistant };
```

```jsx
<script type="module" src="main.mjs"></script>
<script nomudule src="fallback.js"></script>
```

### 5.3 모듈 객체

```jsx
import * as Staff from "/modules/staff.mjs";

Staff.baker;
Staff.pastryChef;
```

### 5.4 외부 소스로부터 가져오는 모듈

```jsx
import { cakeFactory } from "https://..../cakeFactory.mjs";
```

### 5.5 정적으로 모듈 가져오기

- 앞의 코드 예시
- 초기 페이지 로드 시 코드를 미리 로드해야 함

### 5.6 동적으로 모듈 가져오기

```jsx
let module = await import("/modules/cakeFactory.js");
```

- 사용자 상호작용에 따라 가져올 때 특정 핸들러에서 import 후 사용하면 된다.
- 화면에 보이면 가져오기: `IntersectionObserver` API 사용

### 5.7 서버에서 모듈 사용하기

- Node 15.3.0 버전 이상에서 모듈을 지원
- type을 module로 적용하면 `.mjs`와 `.js` 파일을 모듈로 취급

```json
{
  ...
  "type": "module",
  ...
}
```

### 5.8 모듈을 사용하면 생기는 이점

- 한 번만 실행
  - 의존성 트리의 가장 내부에 위치한 모듈부터 실행된다.
  - ES 모듈 시스템은 **정적 분석**이 가능하므로, 브라우저나 번들러가 **전체 모듈 그래프를 먼저 파악한 뒤**, **모든 import 순서를 맞춰서 실행**
    - 순서를 지키지 않으면, 아직 정의되지 않은 함수/변수를 참조할 수 있어 오류 발생
    - 그래서 항상 **의존성이 먼저 실행되도록** 설계
- 자동으로 지연 로드
- 유지보수와 재사용
- 네임스페이스를 제공
- 사용하지 않는 코드 제거 (트리쉐이킹)

| **구분**           | `<script defer>`      | `<script type="module">`                    |
| ------------------ | --------------------- | ------------------------------------------- |
| 실행 시점          | DOMContentLoaded 직전 | 동일                                        |
| import/export 사용 | 불가                  | 가능                                        |
| 스코프             | 전역 (`window`)       | 모듈 스코프                                 |
| 중복 실행          | 여러 번 가능          | 한 번만 실행됨                              |
| CORS 제한          | 없음                  | 있음 (외부 모듈은 same-origin or CORS 필요) |

### 5.9 생성자, 게터, 세터를 가진 클래스

- 모듈과 클래스의 차이점
  - 모듈은 가져오기 내보내기
  - 클래스는 class 키워드를 통해 정의

### 5.10 자바스크립트 프레임워크와 클래스

- 리액트는 클래스 대체제로 hooks를 도입
- 여전히 클래스는 널리 사용되고 있다.
