# 코드

```jsx
const html = renderToString(reactNode);
```

# 용도

React 트리를 HTML 문자열로 렌더링합니다.

서버에서 renderToString을 호출하면 앱을 HTML로 렌더링합니다.

클라이언트에서 hydrateRoot를 호출하면 서버에서 생성된 HTML을 상호작용 가능하게 만듭니다.

```jsx
import { renderToString } from "react-dom/server";

const html = renderToString(<App />);
```

## 매개변수

reactNode: HTML로 렌더링하려는 React 노드입니다. 예: <App />과 같은 JSX 노드

## 반환값

HTML 문자열

# 주의사항

renderToString은 Suspense를 제한적으로 지원합니다. 컴포넌트가 일시 중단되면 renderToString은 즉시 HTML로 폴백을 전송합니다.

renderToString은 브라우저에서도 작동은 하지만, 클라이언트 코드에서 사용하는 것은 권장하지 않습니다.

# 사용법

## React 트리를 HTML 문자열로 렌더링하기

```jsx
import { renderToString } from "react-dom/server";

// The route handler syntax depends on your backend framework
// 라우트 핸들러 구문은 백엔드 프레임워크에 따라 다릅니다.
app.use("/", (request, response) => {
  const html = renderToString(<App />);
  response.send(html);
});
```

이렇게 하면 React 컴포넌트의 초기 비대화형 HTML 출력물이 생성됩니다. 클라이언트에서는 서버에서 생성된 HTML을 hydrate하고 상호작용 가능하게 만들기 위해 hydrateRoot를 호출해야 합니다.

# 대안

## renderToString을 스트리밍 메서드로 마이그레이션하기

renderToString은 즉시 문자열을 반환하므로 스트리밍이나 데이터 대기를 지원하지 않습니다.

가능한 모든 기능을 갖춘 다음 대체 서비스 중 하나를 사용할 것을 권장합니다:

Node.js를 사용하는 경우, renderToPipeableStream을 사용하세요.

Deno 또는 최신 엣지 런타임에서 웹 스트림을 사용하는 경우, renderToReadableStream을 사용하세요.

서버 환경이 스트림을 지원하지 않는 경우에는 renderToString을 계속 사용할 수 있습니다.

## 클라이언트 코드에서 renderToString 제거하기

클라이언트에서 일부 컴포넌트를 HTML로 변환하기 위해 renderToString을 사용하는 경우가 있습니다.

```jsx
// 🚩 Unnecessary: using renderToString on the client
import { renderToString } from "react-dom/server";

const html = renderToString(<MyIcon />);
console.log(html); // For example, "<svg>...</svg>"
```

클라이언트에서 react-dom/server를 가져오면 번들 크기가 불필요하게 증가하므로 피해야 합니다. 브라우저에서 일부 컴포넌트를 HTML로 렌더링해야 하는 경우 createRoot를 사용하여 DOM에서 HTML을 읽어오세요

```jsx
import { createRoot } from "react-dom/client";
import { flushSync } from "react-dom";

const div = document.createElement("div");
const root = createRoot(div);
flushSync(() => {
  root.render(<MyIcon />);
});
console.log(div.innerHTML); // For example, "<svg>...</svg>"
```

flushSync 호출은 innerHTML 속성을 읽기 전에 DOM을 업데이트하기 위해 필요합니다.

# 문제 해결

## 컴포넌트에 Suspense를 도입하면 HTML에 항상 폴백만 보입니다

renderToString은 Suspense를 완전히 지원하지 않습니다.

일부 컴포넌트가 일시 중단되는 경우(예: lazy으로 정의되었거나 데이터를 가져오는 경우) renderToString은 해당 콘텐츠가 해결될 때까지 기다리지 않습니다. 대신 renderToString은 그 위에서 가장 가까운 <Suspense> 바운더리를 찾아 HTML에서 fallback prop을 렌더링합니다. 콘텐츠는 클라이언트 코드가 로드될 때까지 표시되지 않습니다.

이 문제를 해결하려면 권장 스트리밍 솔루션 중 하나를 사용하세요. 이들은 서버에서 확인되는 대로 콘텐츠를 청크로 스트리밍하므로, 클라이언트 코드가 전부 로드되기 전에도 페이지가 점진적으로 채워지는 것을 볼 수 있습니다.
