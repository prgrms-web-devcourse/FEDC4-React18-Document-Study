사용법

- 트리 깊숙히 데이터 전달하기
- context를 통해 전달된 데이터 업데이트하기
- fallback 기본값 지정하기
- 트리 일부에 대한 context 재정의 하기
- 객체 및 함수 전달 시 리렌더링 최적화

문제 해결

- 컴포넌트가 provider의 값을 인식하지 못합니다
- 기본값이 다른데도 context에 항상 undefined만 얻습니다

컴포넌트의 최상위 레벨(?)에서 useContext를 호출하여 context를 읽고 구독합니다.

context : 멀리 떨어져 있는 상위 트리라도 그 안에 있는 전체 트리에 일부 데이터를 제공할 수 있게 해줍니다.

| ![no-context](image.png) | ![context](image-1.png) |
| ------------------------ | ----------------------- |

createContext로 생성
사용사례

- 테마(다크모드 같은거)
- 현재 계정(로그인정보)
- 라우팅정보
- state 관리

context자체는 정보를 보유하지 않음. 컴포넌트에서 제공하거나 읽을 수 있는 정보의 종류를 나타낼 뿐

useContext는 "호출하는" 컴포넌트에 대한 context값을 반환함

이 값은 호출한 컴포넌트에서 트리상 위에 있는 가장 가까운 SomeContext.Provider에 전달된 value

이런 provider가 없으면 반환되는 값은 해당 context에 대해 createContext에 전달한 defaultValue가 된다.

반환된 값은 항상 최신 값임. React는 context가 변경되면 context를 읽는 컴포넌트를 자동으로 리렌더링함

컨텍스트를 제공하는 컴포넌트 === <Context.Provider> -> 반드시 useContext() 호출을 수행하는 컴포넌트의 위에 있어야 함

해당 컨텍스트를 사용하는 자식들에 대해 전부 자동 리렌더링

memo로 리렌더링을 건너뛰어도 새로운 context값을 수신하는 자식들을 막지 못함

context를 제공하는 데 사용하는 SomeContext와 context를 읽는 데 사용하는 SomeContext가 정확하게 동일한 객체인 경우에만 작동.

useContext()는 항상 그것을 호출하는 컴포넌트 위의 가장 가까운 provider를 찾습니다. useContext()를 호출하는 컴포넌트 내의 provider는 고려하지 않습니다.

```jsx
/*
App에서 컨텍스트를 생성하고 Form을 감쌈 Form의 하위 컴포넌트들은 컨텍스트를 구독할 준비가 됨
그래서 Panel과 Button은 Form에서 직접적으로 props를 주지 않아도 context의 값을 사용할 수 있음
*/
import { createContext, useContext } from "react";

const ThemeContext = createContext(null);

export default function MyApp() {
  return (
    <ThemeContext.Provider value="dark">
      <Form />
    </ThemeContext.Provider>
  );
}

function Form() {
  return (
    <Panel title="Welcome">
      <Button>Sign up</Button>
      <Button>Log in</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = "panel-" + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  );
}

function Button({ children }) {
  const theme = useContext(ThemeContext);
  const className = "button-" + theme;
  return <button className={className}>{children}</button>;
}
```

> 이거를 이제 컴포넌트들 마다 파일로 분리하고 싶으면 export const ThemeContext... , import ThemeContext... 해주면 됩니다. 그러면 하위 컴포넌트 들에서 다 쓸 수 있다.

context를 업데이트하려면 state와 결함해야함

값과 객체를 context를 통해 전달할 수 있음. state, setState전달 가능

다중 context가능

단일 컴포넌트로 providers 추출 가능

context와 reducer를 통한 확장

[일정관리 예시](https://react-ko.dev/learn/scaling-up-with-reducer-and-context)

React가 부모 트리에서 특정 context의 provider들을 찾을 수 없는 경우, useContext()가 반환하는 context 값은 해당 context를 생성할 때 지정한 기본값과 동일합니다:

```jsx
const ThemeContext = createContext(null);
```

기본값은 절대 변경되지 않음 절대. context를 업데이트하려면 state를 사용해라

트리의 일부분을 다른 값의 provider로 감싸 해당 부분에 대한 context를 재정의할 수 있습니다.

```jsx
<ThemeContext.Provider value="dark">
  ...
  <ThemeContext.Provider value="light">
    <Footer />
  </ThemeContext.Provider>
  ...
</ThemeContext.Provider>
```

테마를 재정의 하거나, 자동으로 중첩되는 제목들을 위해 씁니다.

```jsx
import { useContext } from "react";
import { LevelContext } from "./LevelContext.js";

export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

객체 및 함수 전달 시 리렌더링 최적화

```jsx
import { useCallback, useMemo } from "react";

function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  const login = useCallback((response) => {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }, []);

  const contextValue = useMemo(
    () => ({
      currentUser,
      login,
    }),
    [currentUser, login]
  );

  return (
    <AuthContext.Provider value={contextValue}>
      <Page />
    </AuthContext.Provider>
  );
}
```

위와 같은 방식으로 useCallback 과 useMemo를 활용해서 context 리렌더링을 최적화 해줄 수도 있습니다.
