## 10. 모듈형 자바스크립트 디자인 패턴

> #### [들어가며]
>
> 확장 가능한 자바스크립트 환경에서 모듈형 이란 서로 의존성이 낮은 기능들이 모듈로써 저장된 형태를 뜻합니다.
> 이러한 느슨한 결합은 의존성을 제거하여 애플리케이션의 유지보수를 용이하게 만들어줍니다.
> 이번 장에서는 앞선 장에서 보았던 방식들과 다르게, ES5 (모듈, 클래스 등의 기능을 지원하지 않는 버젼) 문법을 사용하는 AMD, CJS, UMD 세가지 방식에 대해 살펴보겠습니다.

### 10-01. 스크립트 로더에 대한 참고사항

- AMD와 CJS 같은 모듈형 자바스크립트를 이해하기 위해서는 스크립트 로더에 대한 선행이 필수적입니다.

> 스크립트 로더는 모듈형 자바스크립트를 구현하지 위한 핵심적인 도구입니다. 호환가능한 스크립트 로더를 사용해야만 모듈형 자바스크립트를 구현할 수 있었습니다. ex) RequireJS, curl.js 등등

### 10-02. AMD

> AMD: Asynchronous Module Definition 형식
> 모듈과 의존성 모두를 비동기적으로 로드할 수 있도록 설계된 모듈 정의 방식입니다.

- AMD 형식의 주요 목표는 개발자들이 활용할 수 있는 모듈형 자바스크립트 솔루션을 제공하는 것입니다.
- 비동기적이면서도 높은 유연성을 갖춤 === 의존성을 느슨하게 해주는 장점 존재
- 당시 많은 개발자가 AMD형식을 선호했음 === 당시 구현하기 힘들었던 JS Moduler 를 위한 주춧돌이 될 수 있었음.

#### 10-02.1. 모듈 알아보기

AMD에서 주목할만한 두가지 개념이 존재합니다.

- 모듈 정의를 구현하는 define 메서드
  - define 메서드는 이름이 있는 모듈 혹은 익명 모듈을 정의하는데 사용됩니다.
- 의존성 로딩을 처리하는 require 메서드
  - require는 일반적으로 최상위 자바스크립트 파일이나 모듈 내에서 의존성을 동적으로 가져오고자 할 때 사용됩니다
  - CJS 파일에서 import대신 require를 사용하는 방식을 뜻하는듯?

#### 어째서 AMD가 모듈형 자바스크립트 작성에 더 좋을까?

why?!

- 유연한 모듈 정의 방식에 대한 명확한 솔루션 제공
- 기존에 많이 사용되고 있는 전역 네임스페이스나 `<script>`태그 방식에 비해 훨씬 더 구조화되어 있음.
- 독립적인 모듈과 의존성을 명확히 선언 가능
- 일부 대체 솔루션 (ex: CJS) 에 비해 더 효과적이라는 주장이 존재함
  - AMD는 다른 크로스 도메인, 로컬환경, 디버깅 등에서 문제가 없으며, 서버사이드 툴을 사용할 필요도 없음.
  - 대부분의 AMD 로더는 빌드 과정 없이 브라우저에서 모듈을 로딩하는 것을 지원함.
- 여러 모듈을 하나의 파일로 가져오기 위한 `전송 (transport)`방식을 제공. (다른 방식에서는 전송 형식을 아직 도입하지 못함)
- 스크립트의 Lazy-load 를 지원함

### 10-03. CommonJS

> CJS 는 서버 사이드에서 모듈을 선언하는 간단한 API를 지정하는 모듈 제안입니다.
> AMD 와는 다르게 I(nput)/O(utput), 파일시스템, 프로미스 등 더욱 광범위한 부분을 다룹니다.

- 여기에서 서버사이드 모듈이라는 부분이 이해가 잘 안되었는데, Node.js 환경을 뜻하는듯 합니다.
- AMD는 브라우저 중심인 반면 CJS는 서버사이드까지 커버가 가능하다는것이죠. 또는 서버 중심으로 출시되기도 한것같고...

##### 모듈시스템에도 근본적인 차이가 존재하는것으로 알고있습니다.

- cjs는 런타임이 실행되는 시점에 정의가 됨
  - 자바스크립트 실행 시점에 알수 있는 (런타임 레벨) 시간에서 판단 할 수 있는 장점 존재
- mjs는 빌드타임에서 확인 가능한 시점이 존재함 ⇒ static 분석이 가능해짐 ⇒ 트리쉐이킹 등등 코드가 더 가벼워 질 수 있는 장점이 존재할 수 있음

##### GPT를 조금 duck punching해본 결과 node에서 CJS를 채택한 이유도 연관되는것 같네요

- 당시 2009년도에는 MJS가 존재하지 않았음
- 브라우저용 AMD만 존재함
- 서버에서 JS를 사용할 표준이 없었음
- 서버사이드의 특이한 환경도 작용했음
  - 런타임 유연성 필요: production인지 development인지 다른 런타임 분기 존재
  - 조건부 로딩
  - 동적 확장성
  - 환경별 분기
    등등..

#### 10-03.3. Node.js 환경에서의 CJS

기본적으로 Node.js는 다음과 같은 파일들을 CJS 모듈로 인식합니다.

- .cjs 확장자를 가진 파일
- package.json 파일 안에 type값이 commonjs로 되어있는 경우 .js 파일도 해당됨
- package.json 파일 안에 type을 명시하지 않은 경우 .js 파일도 해당됨 (Monorepo 설정시 헤메었던 기억이 있네요)
- .mjs, .cjs, .json, .node, .js 이외의 확장자를 가진 파일

### 10-04. AMD vs CJS: 동상이몽

- AMD는 브라우저 우선 접근 방식을 채택하여 비동기 동작과 간소화된 하위 호환성을 선택한 반면, 파일 input/output 에 대한 개념은 없습니다.
- 또한 객체, 함수, 생성자, 문자열, JSON 등 다양한 형태의 모듈을 지원하여 브라우저에서 자체적으로 실행된다는 면에서 대단히 유연한 포멧입니다.

- 반면 CJS는 서버 우선 접근 방식을 취하며 동기적으로 동작, 전역 변수와의 독립성 그리고 미래의 서버 환경을 고려합니다.
- 이러한 특징이 의미하는 바는 CJS가 unwrapped 언래핑된 모듈을 지원하기 때문에 ES2015+에 조금 더 가깝게 느껴진다는 것 입니다.

### 10-05. 마치며.

- package 듀얼타입 삭제 및 MJS로만 사용하도록 설정 필수...!
