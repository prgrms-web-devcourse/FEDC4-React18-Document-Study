## useState

### useState hook의 매개변수

`initialState` : 어떤 타입도 가능, 초기에 state를 설정할 값,

**initialState타입이 함수라면?**

- 초기화 함수라고 부른다.
- 해당 함수는 순수함수여야 한다.
- 반환 값이 반드시 있어야 한다.
- 인자를 받지 않는 함수여야 한다.

### useState hook의 **반환값**

useState는 정확히 두 개의 값을 가진 배열을 반환한다

1. **current state**

처음 렌더링 후 current state는 처음 렌더링 중에 전달한 initialState와 일치한다.

1. **set function**

state를 다른 값으로 업데이트하고 리렌더링을 촉발할 수 있는 함수이다.

### useState hook 사용시 주의사항

1. useState는 훅이므로 컴포넌트의 최상위 레벨이나 직접 만든 훅에서만 호출할 수 있다.

반복문, 조건문 안에서는 호출할 수 없다.

```jsx
if (condition) {
  const [value, setValue] = useState(0);
}
// 🚫 호출할 수 없음
```

2. strict Mode에서 React는 **초기화 함수**를 두 번 호출한다. 초기화 함수가 순수하다면 동작에 영향을 미치지 않는다.

**반대로 생각하면 초기화 함수가 순수하지 않다면?**

## `set` functions (setSomething)

set함수를 사용하면 state를 다른 값으로 업데이트하고 리렌더링을 촉발할 수 있다.

1. nextState를 직접 전달하는 방법

```jsx
setName("Taylor");
```

1. 이전 state로부터 계산하여 다음 React를 도출하는 함수를 전달하는 방법

```jsx
setAge((a) => a + 1);
```

### setFunction의 매개변수

`nextState` : state가 될 값

모든 데이터 타입이 허용되지만, 함수 타입은 좀 다르게 작동한다.

**함수 타입이라면?**

업데이터 함수로 취급한다.

이 함수는 **순수**해야 한다.

**대기 중인 state를 유일한 인수로 사용**해야 한다.

반드시 **nextState를 반환**해야 한다.

※ React는 업데이터 함수를 queue에 넣고 컴포넌트를 리렌더링한다. 다음 렌더링 중에 React는 queue에 있는 모든 업데이터를 이전 state에 적용하여 다음 state(nextState)를 계산한다.

**Example**

```jsx
function handleClick() {
  setAge(age + 1);
  setAge(age + 1);
  setAge(age + 1);
}
```

`setAge(age + 1)`을 세 번 호출한다. 이때 age의 값은 뭐가 될까?

- **정답**
  age는 45가 아닌 43이 된다.
  ```jsx
  function handleClick() {
    setAge(age + 1); // setAge(42 + 1)
    setAge(age + 1); // setAge(42 + 1)
    setAge(age + 1); // setAge(42 + 1)
  }
  ```
  set 함수를 연속 호출하면, 실행될 때 마다 state가 업데이트되는게 아니라 해당 함수를 queue에 넣고 이전 age인 42에 일괄 적용한다.

### setFunction 반환값

set 함수는 반환값이 없다.

### setFunction 사용시 주의사항

**1) set함수는 다음 렌더링에 대한 state변수만 업데이트한다.**

set 함수를 호출한 후에도 state 변수에는 여전히 호출 전 화면에 있던 이전 값이 담겨있다.

**Q. state를 업데이트 했지만 로그에는 계속 이전 값이 표시되는 이유는?**

```jsx
function handleClick() {
  console.log(count); // 0

  setCount(count + 1); // Request a re-render with 1
  // 1로 리렌더링 요청
  console.log(count); // Still 0!
  // 아직 0입니다!

  setTimeout(() => {
    console.log(count); // Also 0!
    // 여기도 0이고요!
  }, 5000);
}
```

state가 **snapshot**처럼 동작하기 때문이다.

> snapshot : 특정 시간에 데이터 저장 장치의 상태를 별도의 파일이나 이미지로 저장하는 기술

state를 업데이트하면 새로운 state 값으로 다른 렌더링을 요청하지만 **이미 실행 중인 이벤트 핸들러의 count 변수에는 영향을 미치지 않는다.**

nextState를 사용해야 한다면, set함수에 전달하기 전에 변수에 저장할 수 있다.

```jsx
const nextCount = count + 1;
setCount(nextCount);

console.log(count); // 0
console.log(nextCount); // 1
```

2. 사용자가 제공한 새로운 값이 `Object.is` 에 의해 현재 state와 동일하다고 판정되면 React는 컴포넌트와 그 자식들을 리렌더링하지 않는다.

3. state 업데이트를 일괄처리한다. 모든 이벤트 핸들러가 실행되고 `set` 함수를 호출한 후에 화면을 업데이트 한다. → 단일 이벤트 중에 여러 번 리렌더링을 방지한다.

드물지만 DOM에 접근하기 위해 React가 화면을 더 일찍 업데이트하도록 강제해야 하는 경우, `flushSync`를 사용할 수 있다.

4. 렌더링 도중 `set` 함수를 호출하는 것은 현재 렌더링 중인 컴포넌트 내에서만 허용된다. React는 해당 출력을 버리고 즉시 새로운 state로 다시 렌더링을 시도한다. 이 패턴은 거의 필요하지 않지만 **이전 렌더링의 정보를 저장하는 데 사용할 수 있다.**

5. Strict Mode에서 React는 의도치않은 불순물을 찾기 위해 **업데이터 함수를 두 번 호출한다**. 이는 개발 환경 전용 동작이며 상용 환경에는 영향을 미치지 않는다. 만약 업데이터 함수가 순수하다면 이것은 동작에 영향을 미치지 않는다. 호출 중 하나의 결과는 무시된다.

## 사용법

### 컴포넌트에 state 추가하기

```jsx
import { useState } from 'react';

function MyComponent() {
  const [age, setAge] = useState(42);
  const [name, setName] = useState('Taylor');
// ...
```

화면에 내용을 업데이트 하려면 nextState로 set 함수를 호출한다.

```jsx
function handleClick() {
  setName("Robin");
}
```

React는 nextState를 저장하고 새로운 값으로 컴포넌트를 다시 렌더링한 후 UI를 업데이트한다.

**[Basic useState examples](https://react-ko.dev/reference/react/useState#examples-basic)**

### 💡다시 강조

`set` 함수를 호출해도 이미 실행 중인 코드의 현재 state는 변경되지 **않는다**

```jsx
function handleClick() {
  setName("Robin");
  console.log(name); // Still "Taylor"!
  // 아직 "Taylor"입니다!
}
```

`set`함수는 **다음** 렌더링에서 반환할 `useState`에만 영향을 다.

### 이전 state 기반으로 state 업데이트 하기

```jsx
function handleClick() {
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
}
```

**우리가 원하는 결과를 얻고자 한다면?**

setAge에 값이 아닌 **업데이터 함수**를 전달하자

```jsx
function handleClick() {
  setAge((a) => a + 1); // setAge(42 => 43)
  setAge((a) => a + 1); // setAge(43 => 44)
  setAge((a) => a + 1); // setAge(44 => 45)
}
```

**대기 중인 state**를 가져와서 next state를 계산한다.

React는 업데이터 함수를 큐에 넣는다. 그러면 다음 렌더링 중에 동일한 순서로 호출한다.

**여기서 가지는 의문 : 항상 업데이터 함수를 사용하는 것이 더 좋나요?**

일단 두 접근 방식 사이에는 차이가 없다.

React는 클릭과 같은 의도적인 사용자 액션에 대해 항상 다음 클릭 전에 `age` state 변수가 업데이트 되도록 한다. 즉, 클릭 핸들러가 이벤트 핸들러를 시작할 때 “오래된” `age`를 볼 위험은 없습니다.

```jsx
function handleClick() {
  setAge((a) => a + 1);
}

function handleClick() {
  setAge(age + 1);
}
```

다만 동일한 이벤트 내에서 여러 업데이트를 수행하는 경우에는 업데이터가 도움이 될 수 있다. 혹은 state 변수 자체에 접근하는 것이 어려운 경우에도 유용하다. (리렌더링을 최적화할 때 이 문제(state 변수 자체에 접근하기 어려운 문제?)가 발생할 수 있다)

친절한 문법보다 일관성을 더 선호한다면 설정하려는 state가 이전 state에서 계산되는 경우 항상 업데이터를 작성하는 것이 합리적일 것이다. 만약 어떤 state가 _다른_ state 변수의 이전 state로부터 계산되는 경우라면, 이를 하나의 객체로 결합하고 reducer를 사용하는 것이 좋다.

**[Updater function examples](https://react-ko.dev/reference/react/useState#examples-basic)**

### 객체 및 배열 state 업데이트

state에는 객체와 배열도 넣을 수 있다. React에서 state는 읽기 전용으로 간주되므로 **기존 객체를 수정하는게 아니라, 교체해야한다**.

예를 들어, state에 `form` 객체가 있는 경우

```jsx
form.firstName = "Taylor"; // 🚫 Bed
```

새로운 객체를 생성하여 전체 객체를 교체해라

spread 문법을 사용하여 객체를 복사한 후 바꾸고 싶은 property 값을 수정하면 된다.

```jsx
setForm({
  ...form,
  firstName: "Taylor",
});

// ✅ Good
```

주의할 점! spread 문법은 얉게 복사하므로 1단계 깊이만 복사할 수 있다.

**왜? Why?**

React는 최적화를 위해서 이전 프로퍼티나 state가 다음 프로퍼티나 state와 동일한 경우 렌더링과 같은 작업을 건너뛴다. 근데 객체 같은 경우 내부에서 수정이 이루어져도 주소 값의 변화가 없기 때문에 React는 state 값의 변화가 없다고 확신한다. (배열도 마찬가지)

[**examples of objects and arrays in state**](https://react-ko.dev/reference/react/useState#examples-objects)

### 초기 state 다시 생성하지 않기

React는 초기 state를 한 번 저장하고 다음 렌더링부터는 이를 무시한다. 즉, 리렌더링할 때는 `createInitialTodos()`값이 무시된다.

```jsx
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos());
  // ...
```

`createInitialTodos()`의 결과는 초기 렌더링에만 사용되지만, 여전히 렌더링을 **할 때 마다** 이 함수를 호출하게 된다. ⇒ 이는 큰 배열을 생성하거나 값비싼 계산을 수행하는 경우 낭비가 될 수 있다.

해결방법 : `useState`에 **초기화 함수로 전달해라**

함수를 호출한 결과가 아닌 함수 자체를 전달하게 되면 React는 초기화 중에만 함수를 호출하고 이 후에는 호출하지 않고 무시하게 된다.

**[examples](https://react-ko.dev/reference/react/useState#examples-initializer)**

### key로 state 재설정하기

list를 렌더링할 때 `key` 속성을 사용되지만 `key` 속성은 다른 용도로도 사용됩니다.

**컴포넌트에 다른 `key`를 전달하여 컴포넌트의 state를 재설정**할 수 있다. 이 예제에서는 Reset 버튼이 `version` state 변수를 변경하고, 이를 `Form`에 `key`로 전달한다. `key`가 변경되면 React는 `Form` 컴포넌트(및 그 모든 자식)를 처음부터 다시 생성하므로 state가 초기화된다.

### 이전 렌더링에서 얻은 정보 저장하기

보통은 이벤트 핸들러에서 state를 업데이트합니다. 하지만 드물게 렌더링에 대한 응답으로 state를 조정해야 하는 경우도 있습니다. 예를 들어, props가 변경될 때 state 변수를 변경하고 싶을 수 있습니다.

## 문제 해결

### 1. state를 업데이트 했지만 로그에는 계속 이전 값이 표시돼요!

```jsx
function handleClick() {
  console.log(count); // 0

  setCount(count + 1); // Request a re-render with 1
  // 1로 리렌더링 요청
  console.log(count); // Still 0!
  // 아직 0입니다!

  setTimeout(() => {
    console.log(count); // Also 0!
    // 여기도 0이고요!
  }, 5000);
}
```

그 이유는 state가 스냅샷처럼 동작하기 때문이다. state를 업데이트하면 새로운 state 값으로 다른 렌더링을 요청하지만 이미 실행 중인 이벤트 핸들러의 `count` 변수에는 영향을 미치지 않는다.

### 2. \***\*state를 업데이트해도 화면이 바뀌요\*\***

```jsx
obj.x = 10;
```

보통 객체나 배열의 state를 직접 변경(변이)했기 때문이다.

### 3. \***\*“리렌더링 횟수가 너무 많아요”\*\***

전형적으로 이는 *렌더링 중*에 state를 무조건적으로 설정하고 있음을 의미 한다.

컴포넌트가 렌더링 → state 설정(렌더링 유발) → 렌더링→ state 설정(렌더링 유발) 등의 루프에 들어가 있는 것! 이 문제는 이벤트 핸들러를 지정하는 과정에서 실수로 발생하는 경우가 많다.

```jsx
// 🚩 Wrong: calls the handler during render
// 🚩 잘못된 방법: 렌더링 동안 핸들러 요청
return <button onClick={handleClick()}>Click me</button>;

// ✅ Correct: passes down the event handler
// ✅ 올바른 방법: 이벤트 핸들러로 전달
return <button onClick={handleClick}>Click me</button>;

// ✅ Correct: passes down an inline function
// ✅ 올바른 방법: 인라인 함수로 전달
return <button onClick={(e) => handleClick(e)}>Click me</button>;
```

이 에러의 원인을 찾을 수 없는 경우, 콘솔에서 에러 옆에 있는 화살표를 클릭한 뒤 JavaScript 스택에서 에러의 원인이 되는 특정 `set` 함수 호출을 찾아봐라

### 4. \***\*state의 값을 함수로 설정하려고 하면 설정은 안되고 대신 호출돼요\*\***

state에 함수를 넣을 수는 없다.

```jsx
const [fn, setFn] = useState(someFunction);

function handleClick() {
  setFn(someOtherFunction);
}
```

함수를 값으로 전달하면 React는 `someFunction`을 초기화 함수로 여기고, `someOtherFunction`은 업데이터 함수라고 받아들이며, 따라서 이들을 호출해서 그 결과를 저장하려고 시도한다. 정말로 함수를 저장하길 원한다면, 두 경우 모두 함수 앞에 `() =>`를 넣어야 합니다. 그러면 React는 전달한 함수를 값으로써 저장한다.

```jsx
const [fn, setFn] = useState(() => someFunction);

function handleClick() {
  setFn(() => someOtherFunction);
}
```
