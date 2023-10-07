## useRef(initialValue)

컴포넌트의 최상위 레벨에서 `useRef`를 호출하여 ref를 선언한다.

```jsx
import { useRef } from 'react';

function MyComponent() {
  const intervalRef = useRef(0);
  const inputRef = useRef(null);
  // ...
```

### useRef 매개변수

`initialValue`: ref 객체의 `current` 프로퍼티 초기 설정 값이며, 어떤 유형의 값이든 지정할 수 있다. **이 인자는 초기 렌더링 이후부터는 무시된다.**

### useRef 반환값

`useRef`는 **단일 프로퍼티**를 가진 객체를 반환한다

`current`: 처음에는 전달한 `initialValue`로 설정되며 나중에 다른 값으로 바꿀 수 있다. ref 객체를 JSX 노드의 `ref` 속성으로 React에 전달하면 React는 `current` 프로퍼티를 설정한다.

```jsx
import { useRef } from 'react';

function MyComponent() {
  const inputRef = useRef(null);
  // ...
	return <input ref={inputRef} />;
}
```

## useRef 사용시 주의사항

1) `ref.current` 프로퍼티는 state와 달리 **변이할 수 있다.** 그러나 렌더링에 사용되는 객체(예: state의 일부)를 포함하는 경우 해당 객체를 변이해서는 안된다.

2) `ref.current` 프로퍼티를 변경해도 React는 컴포넌트를 다시 렌더링하지 않는다. ref는 **일반 JavaScript 객체이기 때문에** React는 사용자가 언제 변경했는지 알지 못한다

3) 초기화를 제외하고는 **렌더링 중**에 `ref.current`를 **쓰거나 읽지 마라.** 이렇게 하면 컴포넌트의 동작을 예측할 수 없게 다.

## 사용법

### ref로 값 참조하기

컴포넌트의 최상위 레벨에서 `useRef`를 호출하여 하나 이상의 ref 선언한다.

```jsx
import { useRef } from 'react';

function Stopwatch() {
  const intervalRef = useRef(0);
  // ...
  // intervalRef => { current : null}
```

`useRef`는 처음에 제공한 초기값으로 설정된 단일 `current` 프로퍼티가 있는 ref 객체를 반환합니다.

다음 렌더링에서 `useRef`는 동일한 객체를 반환한다. 정보를 저장하고 나중에 읽을 수 있도록 `current` 속성을 변경할 수 있다. state가 머릿속에 떠오를 수 있지만, 둘 사이에는 중요한 차이점이 있다!

**차이점**

**ref를 변경해도 리렌더링이 일어나지 않는다(리렌더링을 촉발하지 않는다)** 즉, ref는 컴포넌트의 시각적 출력에 영향을 미치지 않는 정보(화면에 보이지 않는 정보)를 저장하는 데 적합하다. 

예를 들어, **interval ID**를 저장했다가 나중에 불러와야 하는 경우 ref에 넣을 수 있다. (화면에 보여줘야하는 값이 아니기 때문에 useState가 아닌 ref를 사용함) 

**ref 내부의 값을 업데이트하려면** `current` 프로퍼티를 수동으로 변경해야 한다

```jsx
function handleStartClick() {
  const intervalId = setInterval(() => {
    // ...
  }, 1000);
  intervalRef.current = intervalId;
}
```

나중에 **ref에서 해당 interval ID를 읽어** 해당 interval을 취소할 수 있다

```jsx
function handleStopClick() {
  const intervalId = intervalRef.current;
  clearInterval(intervalId);
}
```

**ref를 사용하면 보장되는 것**

- (렌더링할 때마다 재설정되는 일반 변수와 달리) **리렌더링 사이에 정보를 저장**할 수 있다.
- (state 변수와 달리) 변경해도 **리렌더링이 일어나지 않는다.**
- (정보가 공유되는 외부 변수와 달리) 각각의 컴포넌트에 **로컬로 저장된다.**

※ ref를 변경해도 다시 렌더링되지 않으므로 화면에 표시되는 정보를 저장하는 데는 ref가 적합하지 않다. 대신 state를 사용하세요 

[**Examples of useRef**](https://react-ko.dev/reference/react/useRef#examples-value)

### 😈 실수하기 좋은 ref의 함정

**렌더링 중에는 `ref.current`를 쓰거나 *읽지* 마라**

React는 컴포넌트의 본문이 순수 함수처럼 동작하기를 기대한다.

- 입력값들(props, state, context)이 동일하면 완전히 동일한 JSX를 반환해야한다. (순수함수의 특징 1)
- 컴포넌트 함수 호출 순서가 달라지거나, 다른 인수를 사용하여 호출하게 되더라도, 다른 호출의 결과에 영향을 미치지 않아야 한다.
    
    (순수함수의 특징 2)
    

**렌더링 중에** ref를 읽거나 쓰면 이러한 기대가 깨진다.

```jsx
function MyComponent() {
  // ...
  // 🚩 Don't write a ref during rendering
  // 🚩 렌더링 중에 ref를 작성하지 마세요.
  myRef.current = 123;
  // ...
  // 🚩 Don't read a ref during rendering
  // 🚩 렌더링 중에 ref를 읽지 마세요.
  return <h1>{myOtherRef.current}</h1>;
}
```

**대신 이벤트 핸들러나 Effect에서** ref를 읽거나 쓸 수 있다.

**렌더링 중에 무언가를 읽거나 써야만 하는 경우,** 대신 state를 사용해라

### ref로 DOM 조작하기

ref를 사용하여 DOM을 조작하는 것은 특히 일반적이다. React에는 이를 위한 기본 지원이 있다.

```jsx
import { useRef } from 'react';

function MyComponent() {
  const inputRef = useRef(null);

	function handleClick() {
    inputRef.current.focus();
  } // 3. 이제 DOM 노드는 <input>에 접근해 focus()와 같은 메서드를 호출할 수 있다. 

  // ...
  return <input ref={inputRef} />;
	// 1. ref 속성으로 조작하려는 DOM 노드의 JSX에게 ref 객체를 전달함
  // 2. DOM 노드를 생성하고 화면에 그린 후, React는 ref객체의 current 프로퍼티를 DOM 노드로 설정한다.

	// 4. 노드가 화면에서 제거되면 React는 current 프로퍼티를 다시 null로 설정한다!
```

[ref로 DOM 조작하기 – React](https://react-ko.dev/learn/manipulating-the-dom-with-refs)

[**examples**](https://react-ko.dev/reference/react/useRef#examples-dom)

### ref 콘텐츠 재생성 피하기

React는 **초기에 ref 값을 한 번 저장**하고, **다음 렌더링부터는 이를 무시한다.** `new VideoPlayer()`의 결과는 초기 렌더링에만 사용되지만, 호출 자체는 이후의 모든 렌더링에서도 여전히 계속 이뤄지며 이는 값비싼 객체를 생성하는 경우 낭비일 수 있다.

이 문제를 해결하려면 대신 다음과 같이 ref를 초기화할 수 있다

```jsx
function Video() {
  const playerRef = useRef(null);

  if (playerRef.current === null) {
    playerRef.current = new VideoPlayer();
  }
```

일반적으로 **렌더링 중에 `ref.current`를 쓰거나 읽는 것은 허용되지 않습니다.** 하지만 이 경우에는 결과가 항상 동일하고 초기화 중에만 조건이 실행되므로 충분히 예측할 수 있으므로 괜찮다.(초기화 중에만 조건이 실행된다는 의미 === 초기화된 playerRef.current는 null이기 때문에 해당 조건에 해당 하는 코드는 초기화일 때만 실행됨)

### useRef를 초기화할 때 null 검사를 피하는 방법

→ 초기화할 때 null 검사를 안하는 대신 ref를 읽거나 쓸 때마다 “절대 null이 아닌 ref”를 반환해주는 함수를 만들어서 사용하는 것

```jsx
function Video() {
  const playerRef = useRef(null);

  function getPlayer() {
    if (playerRef.current !== null) {
      return playerRef.current;
    }
    const player = new VideoPlayer();
    playerRef.current = player;
    return player;
  }
```

## 문제 해결 Troubleshooting

### 1. ****커스텀 컴포넌트에 대한 ref를 얻을 수 없어요****

```jsx
const inputRef = useRef(null);

return <MyInput ref={inputRef} />; // Error
```

<aside>
⚠️ Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()? 경고: 함수 컴포넌트에는 ref를 지정할 수 없습니다. 이 ref에 접근하려는 시도는 실패합니다. React.forwardRef()를 사용하려고 하셨나요?

</aside>

기본적으로 컴포넌트는 **내부의 DOM 노드에 대한 ref를 외부로 노출하지 않는다.**

이 문제를 해결하려면 ref를 가져오고자 하는 컴포넌트를 찾고 같이 `forwardRef`로 감싸라

```jsx
import { forwardRef } from 'react';

const MyInput = forwardRef(({ value, onChange }, ref) => {
  return (
    <input
      value={value}
      onChange={onChange}
      ref={ref}
    />
  );
});

export default MyInput;
```

그러면 부모 컴포넌트가 ref를 가져올 수 있다!