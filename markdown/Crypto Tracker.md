# Setup

### 목표

- CoinPaprika API로 암호화폐 목록 및 상세정보 가져오기
- 2개 화면 구성: Home(코인 리스트), Coin(코인 상세)
- URL에 따라 코인 이동: `/bitcoin`, `/bitcoin/chart`, `/bitcoin/information`
- React Query로 데이터 fetching 효율화

### 설치

`npm install react-router-dom@5.3.0 --legacy-peer-deps react-query`
`npm install react@18.2.0 react-dom@18.2.0 typescript@4.9.5 --save`

> `--legacy-peer-deps`는 `react-scripts@5.0.1`과의 호환성 문제 해결을 위해 필요
>
> `react`,`react-dom`,`typescript`는 의존성 문제 해결을 위해 다운그레이드

### 화면 구성

#### 1. Home 화면

코인 리스트 보여주기

#### 2. Coin 화면

URL 파라미터에 따라 해당 코인 정보 보여주기

### 라우팅 설정

```tsx
//Router.tsx
import { BrowserRouter, Switch, Route } from 'react-router-dom';
import Coins from './Coins';
import Coin from './Coin';

function Router() {
  return (
    <BrowserRouter>
      <Switch>
        <Route path="/:coinId">
          <Coin />
        </Route>
        <Route path="/">
          <Coins />
        </Route>
      </Switch>
    </BrowserRouter>
  );
}

export default Router;
```

Switch는 첫 번째 매칭되는 Route만 렌더링

`/:coinId`는 변수 경로 → `/bitcoin`, `/btc` 등

```tsx
//App.tsx
import Router from './Router';

function App() {
  return <Router />;
}

export default App;
```

### useParams

useParams는 현재 URL에서 동적 경로 파라미터를 추출해서 객체 형태로 반환합니다.

```tsx
import { useParams } from 'react-router-dom';

interface RouteParams {
  coinId: string;
}

function Coin() {
  const { coinId } = useParams<RouteParams>();
  return <h1>Coin: {coinId}</h1>;
}
```

#### 왜 타입 지정이 필요할까?

타입스크립트는 useParams()의 반환 타입을 **기본적으로 알 수 없습니다.**
→ 따라서 명시적으로 제네릭 타입을 넘겨야 사용 가능

```tsx
const { coinId } = useParams<{ coinId: string }>(); // 안전하고 편리
```

### 요약

| 항목                | 설명                          |
| ------------------- | ----------------------------- |
| `Route path="/:id"` | URL 파라미터 지정             |
| `useParams()`       | URL 변수 값 가져오기          |
| `Switch`            | 한 번에 하나의 Route만 렌더링 |

![](https://velog.velcdn.com/images/hhyukk/post/a010652a-f40e-40c3-9a12-226b61054541/image.png)

# Styles

### Reset CSS 적용

#### 목적

- 브라우저마다 다른 **기본 스타일 초기화** (margin, padding 등 제거)
- HTML 요소들의 스타일을 **공통 상태로 맞춤**

#### 방법

- `styled-components`의 `createGlobalStyle` 사용
- `<GlobalStyle />` 컴포넌트를 App에서 렌더링하여 전역에 스타일 적용

```tsx
import { createGlobalStyle } from 'styled-components';

const GlobalStyle = createGlobalStyle`
  /* 모든 태그 초기화 */
  html, body, h1, h2, h3, p, div, span, ... {
    margin: 0;
    padding: 0;
    border: 0;
    ...
  }

  * {
    box-sizing: border-box;
  }

  body {
    line-height: 1;
  }

  a {
    text-decoration: none;
  }
`;
```

### 글로벌 폰트 설정

[구글 폰트](https://fonts.google.com)에서 웹 폰트 `Source Sans Pro`를 가져와 사용

`@import`를 `GlobalStyle` 내부에 추가

```tsx
const GlobalStyle = createGlobalStyle`
  @import url('https://fonts.googleapis.com/css2?family=Source+Sans+Pro:wght@300;400&display=swap');

  body {
    font-family: 'Source Sans Pro', sans-serif;
  }
`;
```

### createGlobalStyle 사용 이유

`styled-components`는 기본적으로 **스타일을 컴포넌트 단위로 고립시킴**

Reset이나 폰트처럼 **전역에 적용되어야 할 스타일**은 `createGlobalStyle`을 통해 처리

한 번만 렌더링되며 전체 앱에 전역 스타일을 주입함

### Theme 설정

#### 테마 타입 정의

```tsx
// styled.d.ts
import 'styled-components';

declare module 'styled-components' {
  export interface DefaultTheme {
    bgColor: string;
    textColor: string;
    accentColor: string;
  }
}
```

#### 테마 객체 작성

```tsx
// theme.ts
import { DefaultTheme } from 'styled-components';

export const theme: DefaultTheme = {
  bgColor: '#2f3640',
  textColor: '#f5f6fa',
  accentColor: '#44bd32',
};
```

#### App에서 ThemeProvider 적용

```tsx
import { ThemeProvider } from 'styled-components';
import { theme } from './theme';

function App() {
  return (
    <ThemeProvider theme={theme}>
      <>
        {' '}
        //Fragment
        <GlobalStyle />
        <Router />
      </>
    </ThemeProvider>
  );
}
```

#### Fragment

**React 컴포넌트는 하나의 부모 요소만 반환**할 수 있음

하지만 **여러 요소를 나란히 반환하고 싶을 때**가 많음

#### 예시

```tsx
return (
  <GlobalStyle />
  <Router />
);
```

이건 에러 발생

> 여러 형제 요소를 하나의 부모 없이 반환할 수 없음

### Theme 값 글로벌 스타일에 적용

```tsx
const GlobalStyle = createGlobalStyle`
  ...
  body {
    background-color: ${(props) => props.theme.bgColor};
    color: ${(props) => props.theme.textColor};
  }

  a {
    text-decoration: none;
  }
`;
```

```tsx
//routes/Coins.tsx
import styled from 'styled-components';

const Title = styled.h1`
  color: ${(props) => props.theme.accentColor};
`;

function Coins() {
  return <Title>코인</Title>;
}
export default Coins;
```

![](https://velog.velcdn.com/images/hhyukk/post/0769cb7b-3e5f-497d-986e-138922666de8/image.png)

![](https://velog.velcdn.com/images/hhyukk/post/c6a6ea25-f4ad-4b05-b4db-2984c7573d3a/image.png)
