useDeferredValue는 UI 일부의 업데이트를 지연시킬 수 있는 React 훅입니다.

> const deferredValue = useDeferredValue(value)

사용법

- 새 콘텐츠가 로드되는 동안 오래된 콘텐츠 표시하기
- 콘텐츠가 오래되었음을 표시하기
- UI의 일부에 대해 리렌더링 연기하기

useDeferredValue(value)

컴포넌트의 최상위 레벨에서 useDeferredValue를 호출하여 지연된 버전의 값을 가져옴

```jsx
import { useState, useDeferredValue } from "react";

function SearchPage() {
  const [query, setQuery] = useState("");
  const deferredQuery = useDeferredValue(query);
  // ...
}
```

초기 렌더링 중에는 반환값이 사용자가 제공한 값. 업데이트가 발생하면 이전 값으로 리렌더링을 시도하고, 그 다음 백그라운드에서 다시 새 값으로 리렌더링을 시도.

```jsx
import { Suspense, useState, useDeferredValue } from "react";
import SearchResults from "./SearchResults.js";

export default function App() {
  const [query, setQuery] = useState("");
  const deferredQuery = useDeferredValue(query);
  return (
    <>
      <label>
        Search albums:
        <input value={query} onChange={(e) => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Loading...</h2>}>
        <SearchResults query={deferredQuery} />
      </Suspense>
    </>
  );
}
```

!note

- 여기서 인풋에 값을 입력해서 리렌더링이 일어날 경우 (a눌렀다가 ab 누른경우) 리렌더링이 일어나는 값은 여전히 a임
- 백그라운드에서 React는 query와 deferredQuery를 모두 ab로 업데이트한 상태로 리렌더링을 시도.
- 가장 최근에 제공받은 값을 사용하고, 최신에 받은 값으로 리렌더링이 되기 전에 업데이트가 일어나면 처리하던 입력값을 버림
- 디바운스처럼 입력이 완료될때까지 네트워크 요청이 안되는게 아님. 각 키 입력마다 네트워크 요청은 존재함. 단지, **보여지는** 부분에 관한 이야기임. 네트워크 요청 자체가 아니라 각 키입력에 대한 결과가 준비될때까지 결과를 표시하는 것.
- 따라서 사용자가 계속 입력해도 각 키 입력에 대한 응답이 캐시되므로 백스페이스를 누르면 instant and doesn't fetch again

```jsx
<div
  style={{
    opacity: query !== deferredQuery ? 0.5 : 1,
  }}
>
  <SearchResults query={deferredQuery} />
</div>
// 이런식으로 코드를 짜주면 시각적으로도 업데이트가 되지 않았다는 사실을 쉽게 나타내 볼 수 있다.
```

- [useDeferredValue로 UI의 일부에 대해 리렌더링을 연기할 수 있다.](https://react-ko.dev/reference/react/useDeferredValue#examples)

디바운스 쓰로틀과 어떤점이 다를까?

디바운스는 사용자가 타이핑을 멈출 때까지(n초동안) 기다렸다가 목록을 업데이트

쓰로틀은 n초에 한번 목록을 업데이트

useDeferredValue는 좀 더 리액트에 맞게 짜여져 있음

사용자의 컴퓨터 사양에 맞게 지연이 됨. 고정적 지연이 아니라는 뜻

기본적으로 useDeferredValue에 의해 수행되는 지연된 리렌더링은 사용자가 다른키를 입력하면 React가 해당 리렌더링을 중단하고 키 입력을 처리하고 다시 렌더링을 시작하는데, 디바운스와 쓰로틀은 렌더링이 키 입력을 차단하는 순간을 연기할 뿐임.

네트워크 요청과 같은 작업에는 디바운스 쓰로틀링이 더 유용할 수 있음. 어느 한쪽 선택의 문제는 아님. 같이 쓸 수도 있는것.
