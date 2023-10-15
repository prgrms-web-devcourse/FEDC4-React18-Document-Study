# 코드

const root = hydrateRoot(domNode, reactNode, options?)

# 용도

hydrateRoot를 사용하면 이전에 react-dom/server로 생성한 HTML 콘텐츠가 있는 브라우저 DOM 노드 내에 React 컴포넌트를 표시할 수 있습니다.

hydrateRoot를 호출하여 서버 환경에서 React가 이미 만들어둔 HTML에 React를 “붙입니다”.

```jsx
import { hydrateRoot } from "react-dom/client";

const domNode = document.getElementById("root");
const root = hydrateRoot(domNode, reactNode);
```

React는 domNode 내부에 있는 HTML에 붙어서 그 내부의 DOM을 직접 관리할 것입니다. 온전히 React만으로 만든 App에서는 보통 단일 루트 컴포넌트에 대해 hydrateRoot를 단 한 번만 호출할 것입니다.

# hydrateRoot(domNode, reactNode, options?)

## 매개변수

domNode: 서버에서 루트 엘리먼트로 렌더링된 DOM element.

reactNode: 앞서 존재하는 HTML에 렌더링하기 위한 “React 노드”. 주로 ReactDOM Server의 renderToPipeableStream(`<App />`)처럼 메서드로 렌더링된 `<App />`같은 JSX 조각들입니다.

선택적 options: React 루트에 옵션을 주기 위한 객체.

- 선택적 onRecoverableError: React가 에러에서 자동으로 회복되었을 때 호출하는 콜백 함수.

- 선택적 identifierPrefix: React가 ID로 사용하는 접두사로 useId로 만들어진 값. 한 페이지에 여러 루트를 사용할 경우 충돌을 피하는 데에 유용합니다. 반드시 서버에서 사용한 것과 동일한 값이어야 합니다.

## 반환값

hydrateRoot는 두 메서드가 포함된 객체를 반환합니다: render와 unmount.

## 주의사항

hydrateRoot()는 서버에서 렌더링된 내용과 후에 렌더링된 내용이 동일할 것을 기대합니다. 동일하지 않은 부분들은 버그로 인식하고 직접 고쳐줘야 합니다.

개발 환경에서는 React가 hydration 중에 동일하지 않은 부분에 대해 경고해 줍니다. 속성이 동일하지 않을 경우 해당 속성이 올바르게 적용될 것이라고 보장할 수 없습니다. 마크업이 동일하지 않는 경우는 드물고, 모든 마크업을 검증하는 비용은 굉장히 비싸므로, 이 동작은 성능상 중요합니다.

hydrateRoot는 App에서 단 한 번만 호출하게 될 것입니다. 만약 프레임워크를 사용한다면 프레임워크가 대신 호출해줄 수도 있습니다.

App이 미리 렌더링된 HTML 없이 클라이언트에서만 렌더링하는 경우에는 hydrateRoot()를 쓸 수 없습니다. 대신 createRoot()를 사용해주세요.

# root.render(reactNode)

hydrate된 React 루트부터 내부 컴포넌트를 새로운 React 컴포넌트로 갱신하려면 root.render를 호출하세요. 브라우저 DOM 요소들도 함께 갱신됩니다.

```jsx
root.render(<App />);
```

React는 hydrate된 root부터 내부를 `<App />`으로 갱신합니다.

## 매개변수

reactNode: 갱신하고 싶은 “React 노드”. 주로 `<App /`>과 같은 JSX를 매개변수로 넘기지만, createElement()로 만든 React 엘리먼트나 문자열, 숫자, null, undefined 등을 넘겨도 됩니다.

## 반환값

root.render는 undefined를 반환합니다.

## 주의사항

hydrate가 끝나기 전에 root.render를 호출하면 React는 서버에서 렌더링된 HTML을 모두 없애고 클라이언트에서 렌더링된 컴포넌트들로 완전히 교체합니다.

# root.unmount()

root.unmount를 호출해 React 루트부터 그 하위에 렌더링된 트리를 삭제합니다.

```jsx
root.unmount();
```

온전히 React만으로 만든 앱은 root.unmount를 호출할 경우가 거의 없습니다.

주로 React 루트부터 혹은 그 상위에서부터 시작된 DOM node들을 다른 코드에 의해 DOM에서 삭제되어야 하는 경우 유용합니다. 예를 들어, jQuery 탭 패널이 활성화 되어 있지 않은 탭을 DOM에서 지운다고 가정해봅시다. 탭이 지워지면, React 루트와 그 내부를 포함해 그 안의 모든 것이 지워지게 되고 DOM에서 또한 지워지게 됩니다. root.unmount를 호출해 React에게 삭제된 컨텐츠들을 “그만” 다루라고 알려주어야 합니다. 그렇지 않으면 삭제되어버린 React 루트 내부의 컴포넌트들은 삭제되지 않을 것이며, `“구독”처럼 컴퓨팅 자원을 자유롭게 놓아주지 못하게 됩니다.`

root.unmount를 호출하면 루트 내부의 모든 컴포넌트를 언마운트하고 루트 DOM 노드에서 React를 “떼어”냅니다. 루트 내부의 이벤트 핸들러와 state까지 모두 포함해 언마운트 및 삭제됩니다.

## 매개변수

root.unmount는 매개변수를 받지 않습니다.

## 반환값

render returns null.
render는 null을 반환합니다.

## 주의사항

root.unmount를 호출하면 루트부터 그 안의 모든 컴포넌트가 언마운트되고 루트 DOM 노드에서 React를 “떼어냅니다.”

root.unmount를 한번 호출한 이후엔 root.render를 루트에 다시 사용할 수 없습니다. 언마운트된 루트에 다시 root.render를 호출하려고 한다면 “마운트되지 않은 루트를 업데이트할 수 없습니다.”라는 에러를 던지게 됩니다.

# 사용법

## 서버에서 렌더링된 HTML을 hydrate하기

앱의 HTML을 react-dom/server로 생성했다면, 클라이언트에서 hydrate해주어야 합니다.

```jsx
import { hydrateRoot } from "react-dom/client";

hydrateRoot(document.getElementById("root"), <App />);
```

이렇게 하면 서버 HTML을 브라우저 DOM 노드에서 React 컴포넌트를 이용해 hydrate 해줍니다. 주로 앱을 시작할 때 단 한 번 실행하게 될 것입니다. 프레임워크를 사용중이라면 프레임워크가 알아서 실행해줄 것입니다.

앱을 hydrate하기 위해 React는 서버에서 만들어 진 HTML에 컴포넌트의 로직을 “붙입니다”. Hydration을 거치면 서버에서 만들어진 초기 HTML 스냅샷이 브라우저에서 완전히 인터랙티브한 앱으로 바뀌게 됩니다.

hydrateRoot를 다시 호출하거나 다른 곳에서 더 호출할 필요는 없습니다. 이 시점부터 React가 애플리케이션의 DOM을 다루게 됩니다. UI를 갱신하기 위해선 대신 useState를 사용하세요.

### 함정

hydrateRoot에 전달한 React 트리는 서버에서 만들었던 React 트리 결과물과 동일해야 합니다.

이는 사용자 경험을 위해서 중요합니다. 유저는 서버에서 만들어진 HTML을 JavaScript 코드가 로드될 때까지 둘러보게 됩니다. 서버 렌더링은 HTML 스냅샷을 보여줌으로써 앱의 로딩이 더 빠르게 느껴지도록 합니다. 갑자기 다른 컨텐츠를 보여주게 되면 이러한 환상이 깨져버릴 것입니다. 따라서 서버에서 렌더링한 결과물과 클라이언트에서 최초로 렌더링한 결과물이 같아야 하는 것입니다.

다음은 hydration 에러가 발생하는 대표적인 사례들입니다:

루트 노드 안쪽에서 React로 작성된 HTML 주위에 새 줄(new line) 등의 추가적인 공백이 있는 경우.

typeof window !== 'undefined'과 같은 조건을 렌더링 로직에서 사용한 경우.

window.matchMedia와 같은 브라우저 전용 API를 렌더링 로직에 사용한 경우.

서버와 클라이언트에서 서로 다른 데이터를 렌더링한 경우.

React는 몇몇 hydration 에러는 복구하긴 하지만, 다른 버그들과 마찬가지로 반드시 고쳐줘야 합니다. 운이 좋으면 그저 느려지기만 할 뿐이겠지만, 최악의 경우 이벤트 핸들러가 다른 엉뚱한 엘리먼트에 붙게 될 수도 있습니다.

## 문서 전체를 hydrate하기

앱을 온전히 React만으로 작성한 경우 `<html>` 태그를 포함해 JSX로 된 전체 문서를 렌더링할 수 있습니다.

```jsx
function App() {
  return (
    <html>
      <head>
        <meta charSet="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <link rel="stylesheet" href="/styles.css"></link>
        <title>My app</title>
      </head>
      <body>
        <Router />
      </body>
    </html>
  );
}
```

전체 문서를 hydrate하기 위해선 글로벌 변수인 document를 hydrateRoot의 첫번째 인자로 넘기세요:

```jsx
import { hydrateRoot } from "react-dom/client";
import App from "./App.js";

hydrateRoot(document, <App />);
```

## 불가피한 hydration 불일치 에러 억제하기

어떤 엘리먼트의 속성이나 텍스트 컨텐츠가 서버와 클라이언트에서 서로 다를 수밖에 없는 경우(예: 타임스탬프), hydration 불일치 경고를 안보이게 할 수도 있습니다.

해당 element에서 hydration 경고를 끄기 위해서는 suppressHydrationWarning={true}를 추가하세요:

```jsx
// index.html
<!--
  HTML content inside <div id="root">...</div>
  was generated from App by react-dom/server.
-->
<div id="root"><h1>Current Date: <!-- -->01/01/2020</h1></div>

// index.js
import './styles.css';
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document.getElementById('root'), <App />);

// App.js
export default function App() {
  return (
    <h1 suppressHydrationWarning={true}>
      Current Date: {new Date().toLocaleDateString()}
    </h1>
  );
}
```

이것은 한 단계 아래까지만 적용되며, 예외적인 탈출구일 뿐입니다. 남용하지 마세요. 텍스트 컨텐츠가 아닌 한 React는 잘못된 부분을 수정하지 않을 것이며, 갱신이 일어나기 전까지는 불일치한 상태로 남아있을 것입니다.

## 클라이언트와 서버의 컨텐츠가 서로 다른 경우 처리하기

의도적으로 서버와 클라이언트에서 서로 다른 내용을 렌더링하길 원한다면, 서버와 클라이언트에서 서로 다른 방법으로 렌더링하면 됩니다. 클라이언트에서 다른 것을 렌더링하고자 하는 컴포넌트에서, Effect에서 true로 할당되는 isClient같은 state 변수를 읽도록 하면 됩니다.

```jsx
// index.html
<!--
  HTML content inside <div id="root">...</div>
  was generated from App by react-dom/server.
-->
<div id="root"><h1>Is Server</h1></div>

// index.js
import './styles.css';
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document.getElementById('root'), <App />);

// App.js
import { useState, useEffect } from "react";

export default function App() {
  const [isClient, setIsClient] = useState(false);

  useEffect(() => {
    setIsClient(true);
  }, []);

  return (
    <h1>
      {isClient ? 'Is Client' : 'Is Server'}
    </h1>
  );
}
```

이렇게 하면 처음엔 서버와 동일한 결과물을 렌더링해서 불일치 문제를 피하고, hydration 후에 새로운 결과물이 동기적으로 렌더링 됩니다.

이 방법은 두 번 렌더링해야 하므로 상대적으로 hydration이 느려집니다. 통신 상태가 느릴 경우의 사용자 경험을 유의하세요. 초기 HTML이 렌더링되고 한참 이후에야 JavaScript 코드를 불러오게 될 수 있고, 이 경우 hydration 이후 즉시 다른 UI를 렌더링하게 되면 사용자로서는 UI가 삐걱거리는 것처럼 인식될 수도 있습니다.

## hydrate된 루트 컴포넌트 업데이트하기

루트의 hydrate가 끝난 이후에 root.render를 호출해 루트 컴포넌트를 업데이트 할 수 있습니다. createRoot와 달리 초기 컨텐츠가 이미 HTML로 렌더링 되어 있으므로, 보통은 사용할 필요가 없습니다.

hydrate 후 어떤 시점에 root.render를 호출했을 때 컴포넌트의 트리 구조가 이전에 렌더링했던 구조와 일치한다면, React는 state를 그대로 유지합니다. 다음 예제에서 input에 어떻게 타이핑하든 관계 없이, 매 초 반복되는 render 호출로 인한 업데이트가 아무런 문제를 일으키지 않음을 주목하세요:

```jsx
// index.html
<!--
  All HTML content inside <div id="root">...</div> was
  generated by rendering <App /> with react-dom/server.
-->
<div id="root"><h1>Hello, world! <!-- -->0</h1><input placeholder="Type something here"/></div>

// index.js
import { hydrateRoot } from 'react-dom/client';
import './styles.css';
import App from './App.js';

const root = hydrateRoot(
  document.getElementById('root'),
  <App counter={0} />
);

let i = 0;
setInterval(() => {
  root.render(<App counter={i} />);
  i++;
}, 1000);

// App.js
export default function App({counter}) {
  return (
    <>
      <h1>Hello, world! {counter}</h1>
      <input placeholder="Type something here" />
    </>
  );
}
```

hydrate된 루트에 root.render를 호출하는 것은 흔한 일은 아닙니다. 내부 컴포넌트 중 한 곳에서 state를 업데이트하는 것이 일반적입니다.
