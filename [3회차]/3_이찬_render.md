# 코드

```jsx
render(reactNode, domNode, callback?)
```

# 용도

jsx 조각을 브라우저 DOM 노드로 렌더링

## 매개변수

reactNode: 표시하려는 React 노드. 일반적으로 `<App />`과 같은 JSX 조각이지만, createElement()로 만든 React 엘리먼트나 문자열, 숫자, null, undefined 등을 전달할 수도 있습니다.

domNode: DOM 엘리먼트. React는 이 DOM 엘리먼트 안에 전달한 reactNode를 표시합니다. 이 순간부터 React는 domNode 내부의 DOM을 관리하고 React 트리가 변경되면 이를 업데이트합니다.

선택적 callback: 함수. React는 컴포넌트가 DOM에 배치된 후에 이 함수를 호출합니다.

## 반환값

render는 보통 null을 반환합니다. 다만 전달한 reactNode가 클래스 컴포넌트인 경우에는 해당 컴포넌트의 인스턴스를 반환합니다.

# 주의사항

React 18에서는 render가 createRoot로 대체되었습니다. React 18 이상에서는 createRoot를 사용하세요.

render를 처음 호출할 때 React는 domNode 내부의 모든 기존 HTML 콘텐츠를 지운 후 React 컴포넌트를 렌더링합니다. 서버에서나 빌드 중에 React에 의해 생성된 HTML이 domNode에 포함된 경우, 대신 이벤트 핸들러를 기존 HTML에 첨부해주는 hydrate()를 사용하세요.

동일한 domNode에서 render를 두 번 이상 호출하면 React는 전달한 최신 JSX를 반영하기 위해 필요에 따라 DOM을 업데이트합니다. React는 이전에 렌더링된 트리와 “매칭”하여 재사용할 수 있는 부분과 다시 만들어야 하는 부분을 결정합니다. 동일한 domNode에서 render를 다시 호출하면, 루트 컴포넌트에서 set 함수를 호출하는 것과 비슷하게 불필요한 DOM 업데이트를 하지 않습니다.

앱이 React로 완전히 빌드된 경우, 앱에 render 호출이 한 번만 있을 가능성이 높습니다. (프레임워크를 사용하는 경우 프레임워크가 이 호출을 대신 수행할 수 있습니다.) 컴포넌트의 자식이 아닌 DOM 트리의 다른 부분(예: 모달 또는 툴팁)에서 JSX 조각을 렌더링하려는 경우 render 대신 createPortal을 사용하세요.

# 사용법

## 루트 컴포넌트 렌더링

```jsx
import { render } from "react-dom";

const domNode = document.getElementById("root");
render(<App />, domNode);
```

React는 domNode에 `<App />`을 표시하고 그 안에 있는 DOM을 관리합니다.

React로 완전히 빌드된 앱은 일반적으로 루트 컴포넌트에 render호출이 하나만 있습니다. 페이지의 일부에 React를 “뿌려서” 사용하는 페이지에는 render호출이 필요한 만큼 많이 있을 수도 있습니다.

## 여러개의 루트 렌더링하기

```jsx
import "./styles.css";
import { render } from "react-dom";
import { Comments, Navigation } from "./Components.js";

render(<Navigation />, document.getElementById("navigation"));

render(<Comments />, document.getElementById("comments"));
```

렌더링된 트리는 unmountComponentAtNode()로 파괴할 수 있습니다.

## 렌더링된 트리 업데이트하기

동일한 DOM 노드에서 render를 두 번 이상 호출할 수도 있습니다. 컴포넌트 트리 구조가 이전에 렌더링된 것과 일치하는 한, React는 그 state를 유지합니다. input에 타이핑하더라도 매초마다 반복되는 render 호출로 인해 업데이트가 손상되지 않는 것을 확인해 보세요:

```jsx
//index.js
import { render } from "react-dom";
import "./styles.css";
import App from "./App.js";

let i = 0;
setInterval(() => {
  render(<App counter={i} />, document.getElementById("root"));
  i++;
}, 1000);

//App.js
export default function App({ counter }) {
  return (
    <>
      <h1>Hello, world! {counter}</h1>
      <input placeholder="Type something here" />
    </>
  );
}
```

render를 여러 번 호출하는 경우는 흔하지 않습니다. 보통은 컴포넌트 내부에서 state를 업데이트하는 경우가 많습니다.
