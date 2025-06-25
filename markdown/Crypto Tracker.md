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

# Home

### 기본 구조 및 스타일링

```tsx
//Coins.tsx
// styled-components 이용한 기본 레이아웃 구성
const Container = styled.div`
  padding: 0px 20px;
  max-width: 480px;
  margin: 0 auto;
`;

const Header = styled.header`
  height: 10vh;
  display: flex;
  justify-content: center;
  align-items: center;
`;

const Title = styled.h1`
  font-size: 48px;
  color: ${(props) => props.theme.accentColor};
`;
```

모바일 화면처럼 중앙 정렬, 최대 너비 설정

제목 스타일 지정

### 코인 목록 렌더링

#### 초기에는 하드코딩된 배열로 렌더링

```tsx
const coins = [
  { id: 'btc-bitcoin', name: 'Bitcoin', ... },
  { id: 'eth-ethereum', name: 'Ethereum', ... },
];

...

{coins.map((coin) => (
  <Coin key={coin.id}>{coin.name} &rarr;</Coin>
))}
```

map을 사용하여 코인 리스트를 `<li>`로 렌더링

key는 `coin.id` 사용

### 코인 상세 페이지로 이동

`a href` **→ 새로고침 발생**

`react-router-dom`의 `<Link>` 사용

```tsx
import { Link } from 'react-router-dom';

<Link to={`/${coin.id}`}>{coin.name} &rarr;</Link>;
```

#### 스타일 개선

```tsx
const Coin = styled.li`
  background-color: white;
  color: ${(props) => props.theme.bgColor};
  border-radius: 15px;
  margin-bottom: 10px;
  a {
    padding: 20px;
    transition: color 0.2s ease-in;
    display: block;
  }
  &:hover {
    a {
      color: ${(props) => props.theme.accentColor};
    }
  }
`;
```

전체 li 클릭 가능하도록 a를 `display: block` 처리

마우스 hover 시 색상 전환

### 기본 링크 스타일 제거

```tsx
//App.tsx
const GlobalStyle = createGlobalStyle`
  a {
    text-decoration: none;
    color: inherit;
  }
`;
```

### 코인 데이터 API로 가져오기

[coinpaprika API](https://api.coinpaprika.com/v1/coins) 사용

#### 데이터 타입 선언

```tsx
interface CoinInterface {
  id: string;
  name: string;
  symbol: string;
  rank: number;
  is_new: boolean;
  is_active: boolean;
  type: string;
}
```

#### State 사용

```tsx
const [coins, setCoins] = useState<CoinInterface[]>([]);
const [loading, setLoading] = useState(true);
```

#### useEffect로 데이터 fetch

```tsx
useEffect(() => {
  (async () => {
    const response = await fetch('https://api.coinpaprika.com/v1/coins');
    const json = await response.json();
    setCoins(json.slice(0, 100)); // 상위 100개만 사용
    setLoading(false);
  })();
}, []);
```

비동기 즉시 실행 함수 `(async () => {...})()`로 API 호출

useEffect를 사용해 컴포넌트 마운트 시 데이터 요청

> **"마운트(Mount)"**란 컴포넌트가 처음으로 화면(DOM)에 나타나는 시점

### 로딩 상태 처리

```tsx
const Loader = styled.span`
  text-align: center;
  display: block;
`;

return (
  <Container>
    <Header>
      <Title>코인</Title>
    </Header>
    {loading ? (
      <Loader>Loading...</Loader>
    ) : (
      <CoinsList>
        {coins.map((coin) => (
          <Coin key={coin.id}>
            <Link to={`/${coin.id}`}>{coin.name} &rarr;</Link>
          </Coin>
        ))}
      </CoinsList>
    )}
  </Container>
);
```

# Route States

Crypto Icon API로 암호화폐를 위한 코인의 아이콘을 가져올 수있음

```tsx
<Link to={`/${coin.id}`}>
  <Img src={`https://cryptoicon-api.pages.dev/icons/128/color/${coin.symbol.toLowerCase()}.png`} />
  {coin.name} &rarr;
</Link>
```

### Link에 state 전달하기

React Router의 `<Link>` 컴포넌트로 이동할 때 state 객체를 통해 데이터를 함께 보낼 수 있음.

```tsx
<Link
  to={{
    pathname: `/${coin.id}`,
    state: { name: coin.name },
  }}
>
  ...
</Link>
```

이 방식은 URL에 드러나지 않고, 메모리 상으로만 전달됨.

즉, 사용자에게 보이지 않는 "Behind the Scenes" 전달 방식.

### state 받기

`useLocation()` 훅을 사용하여 전달된 state에 접근할 수 있음.

```tsx
const { state } = useLocation<RouteState>();
```

### 예외 상황: 새로고침 또는 직접 URL 접근 시

사용자가 `/bitcoin` 같은 URL을 직접 입력하거나 새로고침하면 이전 페이지를 거치지 않았기 때문에 state가 존재하지 않음 → undefined 에러 발생
`Cannot read properties of undefined (reading 'name')`

안전하게 접근하기 위해 optional chaining 사용

```tsx
<Title>{state?.name || 'Loading...'}</Title>
```

### 요약

| 기능             | 설명                                   |
| ---------------- | -------------------------------------- |
| `Link`의 `state` | 컴포넌트 간 "비가시적" 데이터 전달     |
| `useLocation()`  | 받은 state 읽기                        |
| `state?.name`    | 직접 접근 시 발생할 수 있는 오류 방지  |
| 사용자 경험 향상 | 로딩 시간 최소화, 즉각적인 데이터 표시 |
