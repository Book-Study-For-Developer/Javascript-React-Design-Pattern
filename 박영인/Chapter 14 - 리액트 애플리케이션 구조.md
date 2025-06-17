## 14.1 소개
리액트 애플리케이션 구조는 일반적으로 두 가지 방식이 많이 사용된다:
- 기능별 그룹화
- 파일 유형별 그룹화

### 14.1.1 기능별 그룹화
비즈니스 흐름 또는 애플리케이션 도메인을 기준으로 폴더를 나누는 방식이다.
비즈니스 흐름에 맞게 폴더를 구성하며, 각 기능에 필요한 CSS, 컴포넌트, 테스트, 라이브러리 등을 하나의 폴더에 모아둔다.

```
common/
  Avatar.js
  Avatar.css
  ErrorUtils.js
  ErrorUtils.test.js

product/
  index.js
  product.css
  price.js
  product.test.js

checkout/
  index.js
  checkout.css
  checkout.test.js
```

#### 장점
- 변경 사항이 생기면 관련 파일이 한 폴더에 모여 있어 유지보수하기 쉽다.
- 특정 기능에 대한 코드가 집중되어 있어 추적과 관리가 편하다.
#### 단점
- 여러 모듈 간에 공통으로 사용하는 코드의 중복이 발생할 수 있다.
- 재사용성과 일관성이 떨어질 수 있다.

### 14.1.2 파일 유형별 그룹화
파일의 종류(CSS, JS, 이미지, 테스트 등)에 따라 폴더를 나누는 방식이다. 
관련된 기능의 파일이라도 각각 다른 폴더에 저장된다.

> 예전엔 거의 이렇게 썼던거 같은데..!

```
css/
  global.css
  checkout.css
  product.css

lib/
  date.js
  currency.js
  gtm.js

pages/
  product.js
  productlist.js
  checkout.js
```

#### 장점
- **표준 구조 재사용**: 다른 프로젝트에서도 동일한 구조로 적용 가능하다.
- **새로운 팀원의 빠른 적응**: 익숙한 구조 덕분에 빠르게 파일을 찾고 이해할 수 있다.
- **공통 스타일 관리 용이**: 전역 스타일이나 공통 컴포넌트를 수정하여 애플리케이션 전체에 반영할 수 있다.
#### 단점
- 하나의 기능을 수정하려면 여러 폴더를 오가야 한다.
- 기능이 많아질수록 폴더 수가 증가해 파일 탐색이 어려워진다.

### 14.1.3 도메인 및 공통 컴포넌트 기반의 혼합 그룹화
파일 유형별, 기능별 그룹화의 장점을 모두 살린 방식이다. 전체 앱에서 공통으로 쓰이는 컴포넌트는 components에, 각 도메인에 특화된 기능은 domain 폴더에 정리한다.

#### 장점
- 자주 변경되는 관련 파일들을 한곳에 두면서도, 전역 컴포넌트 재사용이 가능하다.

#### 구조의 확장 방식
- **평면적 구조**: 도메인 폴더 안에 관련된 파일만 나열하는 방식
- **중첩 구조**: 도메인 안에 다시 하위 도메인을 구성하여 계층적으로 구성

```
// 평면 구조
domain/
  product.js
  product.css
  product.test.js
  checkout.js
  checkout.css
  checkout.test.js

// 중첩 구조
domain/
  product/
    productType/
      features.js
      features.css
      size.js
    price/
      listprice.js
      discount.js
```

📌 **Note**: 3~4단계 이상의 중첩은 경로 관리가 어려워지므로 피하는 것이 좋다.


### 여담
저희는 최상위 폴더는 파일 유형별로 나뉘고, 내부에서 도메인 기능별로 나뉘는 혼합형 방식으로 폴더를 관리합니다!
domain 폴더 내부에는 비즈니스 로직이 각 도메인 별로 모여 있는 구조입니다.

> 대부분 혼합형이실거 같네여!!

```
src/
├── components/
├── constants/
├── context/
├── domain/
│   ├── project/
│   ├── testSuite/
│   └── mockServer/
│       ├── service/
│       ├── model/
│       └── hooks/
├── hooks/
├── pages/
│   ├── project/
│   ├── testSuite/
│   └── mockServer/
├── utils/
```

### 읽을거리
Teo - 프론트엔드 폴더구조 고찰
- https://velog.io/@teo/separation-of-concerns-of-frontend
- https://velog.io/@teo/fsd
- https://velog.io/@teo/folder-structure