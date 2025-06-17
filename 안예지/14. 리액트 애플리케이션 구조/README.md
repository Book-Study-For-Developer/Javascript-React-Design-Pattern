# 리액트 애플리케이션 구조

## 일반적인 리액트 애플리케이션 구조

리액트 애플리케이션은 다양한 방식으로 구성될 수 있지만, 일반적으로 다음과 같은 기본 구조를 가집니다.

### 기본 폴더 구조
```
my-react-app/
├── public/                 # 정적 파일들
│   ├── index.html         # HTML 템플릿
│   ├── favicon.ico        # 파비콘
│   └── manifest.json      # PWA 설정
├── src/                   # 소스 코드
│   ├── components/        # 재사용 가능한 컴포넌트
│   ├── pages/            # 페이지 컴포넌트
│   ├── hooks/            # 커스텀 훅
│   ├── utils/            # 유틸리티 함수
│   ├── services/         # API 호출 로직
│   ├── styles/           # 글로벌 스타일
│   ├── assets/           # 이미지, 폰트 등
│   ├── contexts/         # React Context
│   ├── types/            # TypeScript 타입 정의
│   ├── constants/        # 상수 정의
│   ├── App.tsx           # 루트 컴포넌트
│   ├── index.tsx         # 진입점
│   └── setupTests.ts     # 테스트 설정
├── package.json          # 의존성 및 스크립트
├── tsconfig.json         # TypeScript 설정
├── tailwind.config.js    # Tailwind CSS 설정
└── README.md             # 프로젝트 문서
```

### Next.js 프로젝트 구조
```
src/
├── app/                  # App Router (Next.js 13+)
│   ├── layout.tsx
│   ├── page.tsx
│   └── (routes)/
├── components/
│   ├── ui/              # 기본 UI 컴포넌트
│   └── features/        # 기능별 컴포넌트
├── lib/
│   ├── utils.ts
│   └── validations.ts
├── hooks/
├── types/
└── styles/
```

## 파일 구조의 중요성

리액트 애플리케이션의 파일 구조는 프로젝트의 유지보수성, 확장성, 그리고 팀 협업에 큰 영향을 미칩니다. 잘 설계된 파일 구조는 다음과 같은 이점을 제공합니다:

- **가독성 향상**: 코드를 찾고 이해하기 쉬워집니다
- **개발 효율성**: 관련 파일들을 빠르게 찾을 수 있습니다
- **유지보수성**: 기능 변경 시 영향 범위를 쉽게 파악할 수 있습니다
- **팀 협업**: 일관된 구조로 팀원들이 쉽게 적응할 수 있습니다

## 콜로케이션(Colocation)이란?

콜로케이션은 **관련된 파일들을 물리적으로 가까운 곳에 배치하는 원칙**입니다. 리액트에서는 특정 기능이나 컴포넌트와 관련된 모든 파일들(컴포넌트, 스타일, 테스트, 타입 정의 등)을 같은 디렉토리에 위치시키는 것을 의미합니다.

### 콜로케이션의 핵심 아이디어

> "Things that change together should live together"
> 
> 함께 변경되는 것들은 함께 있어야 한다

## 콜로케이션의 장점

### 1. 인지 부하 감소
```
components/
  UserProfile/
    UserProfile.tsx          # 컴포넌트
    UserProfile.test.tsx     # 테스트
    UserProfile.module.css   # 스타일
    types.ts                 # 타입 정의
    hooks.ts                 # 관련 훅
```

관련된 모든 파일이 한 곳에 있어 컨텍스트 스위칭이 줄어듭니다.

### 2. 개발 속도 향상
- 파일을 찾는 시간 단축
- IDE의 자동완성과 네비게이션 기능 활용
- 관련 파일들 간의 빠른 이동

### 3. 리팩토링 용이성
```
# 기능 삭제 시
rm -rf components/UserProfile/

# 기능 이동 시
mv components/UserProfile/ features/user/
```

### 4. 의존성 관리
```typescript
// UserProfile/index.ts
export { UserProfile } from './UserProfile';
export type { UserProfileProps } from './types';
export { useUserProfile } from './hooks';
```

## 실제 적용 예시

### 나쁜 구조 (기술별 분리)
```
src/
  components/
    Header.tsx
    UserProfile.tsx
    ProductCard.tsx
  styles/
    header.css
    user-profile.css
    product-card.css
  tests/
    Header.test.tsx
    UserProfile.test.tsx
    ProductCard.test.tsx
  types/
    user.ts
    product.ts
```

### 좋은 구조 (콜로케이션 적용)
```
src/
  components/
    Header/
      Header.tsx
      Header.module.css
      Header.test.tsx
      index.ts
    UserProfile/
      UserProfile.tsx
      UserProfile.module.css
      UserProfile.test.tsx
      types.ts
      hooks.ts
      index.ts
    ProductCard/
      ProductCard.tsx
      ProductCard.module.css
      ProductCard.test.tsx
      types.ts
      index.ts
```

## 콜로케이션 적용 방법

### 1. 컴포넌트 기반 구조
```
ComponentName/
  ├── ComponentName.tsx      # 메인 컴포넌트
  ├── ComponentName.test.tsx # 테스트 파일
  ├── ComponentName.stories.tsx # 스토리북 파일
  ├── ComponentName.module.css # 스타일 파일
  ├── types.ts              # 타입 정의
  ├── hooks.ts              # 관련 훅
  ├── utils.ts              # 유틸리티 함수
  └── index.ts              # 배럴 익스포트
```

### 2. 기능 기반 구조
```
features/
  auth/
    components/
      LoginForm/
      SignupForm/
    hooks/
      useAuth.ts
    services/
      authApi.ts
    types/
      auth.types.ts
  user/
    components/
      UserProfile/
      UserSettings/
    hooks/
      useUser.ts
    services/
      userApi.ts
```

### 3. 페이지 기반 구조
```
pages/
  HomePage/
    HomePage.tsx
    components/
      Hero/
      Features/
      Testimonials/
    hooks/
      useHomePage.ts
    HomePage.module.css
```

## 추가로 고려할 점

### 1. 적절한 export 활용
```typescript
// components/UserProfile/index.ts
export { UserProfile } from './UserProfile';
export type { UserProfileProps } from './types';
export { useUserProfile } from './hooks';

// 사용 시
import { UserProfile, useUserProfile } from '@/components/UserProfile';
```

### 2. 적절한 추상화 레벨 유지
```
components/
  ui/                    # 재사용 가능한 기본 컴포넌트
    Button/
    Input/
    Modal/
  features/              # 비즈니스 로직이 포함된 컴포넌트
    UserProfile/
    ProductCard/
  layout/                # 레이아웃 컴포넌트
    Header/
    Sidebar/
    Footer/
```

### 3. 공통 관심사 분리
```
src/
  shared/
    hooks/
      useLocalStorage.ts
      useDebounce.ts
    utils/
      formatters.ts
      validators.ts
    constants/
      api.ts
      routes.ts
```

## 주의사항과 한계

### 1. 과도한 중첩 방지
```
# 좋지 않은 예
components/UserProfile/Avatar/ProfilePicture/Image/

# 개선된 예
components/UserProfile/Avatar/
```

### 2. 공유 로직의 적절한 위치
- 여러 컴포넌트에서 사용되는 로직은 상위 디렉토리로
- 재사용성과 콜로케이션 사이의 균형 유지

### 3. 팀 컨벤션 중요성
- 팀 전체가 동의하는 구조 사용
- 일관성 있는 네이밍 컨벤션
- 문서화된 가이드라인 유지