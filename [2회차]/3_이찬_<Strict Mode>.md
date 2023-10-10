# 코드

```jsx
<StrictMode>
  <App />
</StrictMode>
```

# 용도

개발 중에 컴포넌트에서 흔히 발생하는 버그를 조기에 발견할 수 있음.

Strict Mode는 다음과 같은 개발 전용 동작들을 활성화 함.

- 한번 더 렌더링(불완전한 렌더링으로 인한 버그 찾기 위해)
- Effect를 한 번 더 실행함 (클린업 누락돼서 버그 발생 찾기위해)
- 지원 중단된 api의 사용 여부를 확인하기 위해

이러한 모든 검사는 개발 환경 전용이며 상용 빌드에는 영향을 미치지 않음

# 전체 앱에서 사용하기

```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";

const root = createRoot(document.getElementById("root"));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

# 앱의 일부에 Strict Mode 사용 설정하기

```jsx
import { StrictMode } from "react";

function App() {
  return (
    <>
      <Header />
      <StrictMode>
        <main>
          <Sidebar />
          <Content />
        </main>
      </StrictMode>
      <Footer />
    </>
  );
}
```

# 개발환경에서 이중 렌더링으로 발견된 버그 수정하기

React는 작성하는 모든 컴포넌트가 순수한 함수라고 가정합니다. 즉, 작성하는 React 컴포넌트는 동일한 입력(props, state, context)이 주어졌을 때 항상 동일한 JSX를 반환해야 합니다.

이 규칙을 위반하는 컴포넌트는 예측할 수 없는 동작을 하고 버그를 유발합니다. Strict Mode는 개발 환경에서 실수로 작성한 불순한 코드를 찾을 수 있도록 다음의 일부 함수(순수 함수만)를 두 번 호출합니다:

- 컴포넌트 함수 본문(최상위의 로직만 있으므로 이벤트 핸들러 내부의 코드는 포함되지 않습니다.)
- useState, set 함수, useMemo, useReducer에 전달한 함수
- constructor, render, shouldComponentUpdate와 같은 일부 클래스 컴포넌트 메서드 ([전체 목록 보기](https://ko.reactjs.org/docs/strict-mode.html#detecting-unexpected-side-effects))

순수함수가 아니면 실행시마다 다른 결과가 반환이 될 것이므로 두번 실행하면 그걸 알 수 있겠지? 라는 논리.

> [컴포넌트를 순수하게 유지하는 방법에 대해 자세히 알아보세요.](https://react-ko.dev/learn/keeping-components-pure)

# 개발 환경에서 Effect를 재실행하여 발견된 버그 수정하기

Strict Mode는 Effect의 버그를 찾는 데에도 도움이 될 수 있습니다.

모든 Effect에는 셋업 코드가 있으며 클린업 코드도 있을 수 있습니다. 일반적으로 React는 컴포넌트가 마운트될 때 (화면에 추가될 때) 셋업을 호출하고, 컴포넌트가 마운트 해제될 때 (화면에서 제거될 때) 클린업을 호출합니다.

Strict Mode가 켜져있으면 React는 개발 환경에서 모든 Effect에 대해 셋업 + 클린업 사이클을 한 번 더 실행합니다. 이것을 의아하게 여길 수도 있지만, 수동으로 잡기 어려운 미묘한 버그를 발견하는 데 도움이 됩니다.

# Strict Mode로 지원 중단 경고 수정하기

React는 <StrictMode> 트리 내의 컴포넌트가 더 이상 지원되지 않는 API중 하나를 사용하는 경우 경고를 표시합니다:

- findDOMNode. 대안을 살펴보세요.
- UNSAFE*componentWillMount와 같은 UNSAFE* 클래스 라이프 사이클 메서드들. 대안을 살펴보세요.
- 예전 컨텍스트 API들 (childContextTypes, contextTypes, getChildContext) 대안을 살펴보세요.
- 예전 문자열 ref(this.refs). 대안을 살펴보세요.
