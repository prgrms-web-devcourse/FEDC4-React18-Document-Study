# 코드

> useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)

# 용도

외부 스토어를 구독할 수 있음

외부스토어(자체적으로 상태관리 툴을 만들어 리액트 훅과 연동시킨 상태관리 라이브러리. 리액트에서 관찰 안함)

- React 외부에 state를 보관하는 서드파티 상태 관리 라이브러리.
- 변이 가능한 값을 노출하는 브라우저 API와 그 변이 사항을 구독하는 이벤트.

내부스토어(useState, useReducer, context, props)

주로 기존의 비 React 코드와 통합해야 할 때 유용함

다른 Redux, Recoil과 다른점 뭐일까...?

-> Redux, Recoil에서는 useSyncExternalStore를 사용하지 않아도 되게 만들어놨음

useSyncExternalStore을 쓰는 이유는 concurrent feature에서 발생하는 `Tearing`이라는 이슈

Tearing은 의도치 않은 상태 불일치로 서로 일치하지 않는 시점의 UI가 렌더링되는 것

startTransition을 사용할 떄 externalStore에서 파생된 함수를 useSyncExternalStore를 사용하는 경우가 있음

[참고](https://velog.io/@jay/useSyncExternalStore)

# 외부스토어 구독

```jsx
import { useSyncExternalStore } from "react";
import { todosStore } from "./todoStore.js";

function TodosApp() {
  const todos = useSyncExternalStore(
    todosStore.subscribe,
    todosStore.getSnapshot
  );
  // ...
}
```

todo는 스냅샷을 반환.

subscribe 함수는 스토어를 구독해야하고, 구독 취소 함수를 반환해야 함

getSnapShot 함수는 스토어에서 데이터의 스냅샷을 읽어야 함

store에 변경사항이 있을 때 다시 렌더링 함

# 브라우저 api 구독

ex. navigator.onLine 이라는 속성을 통해 네트워크 연결이 활성화 되었는지 알 수 있다

```jsx
import { useSyncExternalStore } from "react";

function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  // ...
}

function getSnapshot() {
  return navigator.onLine;
}

function subscribe(callback) {
  window.addEventListener("online", callback);
  window.addEventListener("offline", callback);
  return () => {
    window.removeEventListener("online", callback);
    window.removeEventListener("offline", callback);
  };
}
```

getSnapshot도 함수 작성해야하고, subscribe도 작성해야해서 커스텀 훅으로 빼면 편함

```jsx
import { useSyncExternalStore } from "react";

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  return isOnline;
}

function getSnapshot() {
  // ...
}

function subscribe(callback) {
  // ...
}
```

# SSR

React 앱이 서버 렌더링을 사용하는 경우, React 컴포넌트는 브라우저 환경 외부에서도 실행되어 초기 HTML을 생성합니다. 이로 인해 외부 스토어에 연결할 때 몇 가지 문제가 발생합니다

- 브라우저 전용 API에 연결하는 경우 서버에 해당 API가 존재하지 않으므로 작동하지 않습니다.
- 타사 데이터 저장소에 연결하는 경우 서버와 클라이언트 간에 일치하는 데이터가 필요합니다.

ssr할떄 getServerSnapshot 사용

```jsx
import { useSyncExternalStore } from "react";

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(
    subscribe,
    getSnapshot,
    getServerSnapshot
  );
  return isOnline;
}

function getSnapshot() {
  return navigator.onLine;
}

function getServerSnapshot() {
  return true; // Always show "Online" for server-generated HTML
}

function subscribe(callback) {
  // ...
}
```

getServerSnaphsot 함수는 오직 두 가지 상황에서만 실행됨.

- HTML을 생성할 때 서버에서 실행됨
- hydration중, 즉, React가 서버 HTML을 가져와서 인터랙티브하게 만들 때 클라이언트에서 실행됨

이를 통해 앱이 상호작용하기 전에 사용될 초기 스냅샷 값을 제공할 수 있음

note : getServerSnapshot이 초기 클라이언트 렌더링에서 서버에서 반환한 것과 정확히 동일한 데이터를 반환하는지 확인하세요. 예를 들어, getServerSnapshot이 서버에 미리 채워진 스토어 콘텐츠를 반환한 경우 이 콘텐츠를 클라이언트로 전송해야 합니다. 이를 수행하는 일반적인 방법 중 하나는 서버 렌더링 중에 window.MY_STORE_DATA와 같은 글로벌을 설정하는 `<script>` 태그를 생성한 다음, 클라이언트에서 getServerSnapshot로부터 해당 글로벌을 읽어오는 것입니다. 외부 스토어에서 이를 수행하는 방법에 대한 지침을 제공해야 합니다.

# Trouble Shooting

## 무한루프

```jsx
function getSnapshot() {
  // 🔴 Do not return always different objects from getSnapshot
  // 🔴 getSnapshot에서 항상 다른 객체를 반환하지 마세요
  // 이런식으로 하면 무한루프
  return {
    todos: myStore.todos,
  };
}

function getSnapshot() {
  // ✅ You can return immutable data
  // ✅ 불변데이터는 반환할 수 있습니다
  return myStore.todos;
}
```

## 리렌더링 시 subscribe 함수 호출 피하기

불변성을 지켜줘야하기 때문에

함수를 외부로 빼거나, useCallback을 사용해라

```jsx
function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);

  // 🚩 Always a different function, so React will resubscribe on every re-render
  // 🚩 항상 다른 함수이므로 React는 매 렌더링시마다 다시 구독합니다
  function subscribe() {
    // ...
  }

  // ...
}

function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  // ...
}

// ✅ Always the same function, so React won't need to resubscribe
// ✅ 항상 동일한 함수이므로 React는 이를 재구독할 필요가 없습니다
function subscribe() {
  // ...
}

function ChatIndicator({ userId }) {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);

  // ✅ Same function as long as userId doesn't change
  const subscribe = useCallback(() => {
    // ...
  }, [userId]);

  // ...
}
```
