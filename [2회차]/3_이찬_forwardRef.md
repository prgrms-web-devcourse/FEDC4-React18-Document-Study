# 코드

> const SomeComponent = forwardRef(render)

# 용도

forwardRef를 사용하면 컴포넌트가 ref를 사용하여 부모 컴포넌트에 DOM 노드를 노출할 수 있습니다.

그러니까 컴포넌트가 ref를 받아 자식 컴포넌트로 전달하도록 하려면 forwardRef()를 호출하면 됨.

그러니까 ref를 props로 받고싶으면 써라.

```jsx
import { forwardRef } from "react";

const MyInput = forwardRef(function MyInput(props, ref) {
  return (
    <label>
      {props.label}
      <input ref={ref} />
    </label>
  );
});
```

## 매개변수

props: 부모 컴포넌트가 전달한 props입니다.

ref: 부모 컴포넌트가 전달한 ref 속성입니다. ref는 객체나 함수일 수 있습니다. 부모 컴포넌트가 ref를 전달하지 않은 경우 null이 됩니다. 받은 ref를 다른 컴포넌트에 전달하거나 useImperativeHandle에 전달해야 합니다.

## 반환값

forwardRef는 JSX에서 렌더링할 수 있는 React 컴포넌트를 반환합니다. 일반 함수로 정의된 React 컴포넌트와 달리, forwardRef가 반환하는 컴포넌트는 ref prop을 받을 수 있습니다.

# 부모 컴포넌트에 DOM 노드 노출하기

기본적으로 각 컴포넌트의 DOM 노드는 비공개입니다. 그러나 때로는 포커싱을 허용하기 위해 부모에 DOM 노드를 노출하는 것이 유용할 수 있습니다. 이를 선택하려면 컴포넌트 정의를 forwardRef()로 감싸면 됩니다:

```jsx
import { forwardRef } from "react";

const MyInput = forwardRef(function MyInput(props, ref) {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} ref={ref} />
    </label>
  );
});

function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <MyInput label="Enter your name:" ref={ref} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

컴포넌트 내부의 DOM 노드에 대한 ref를 노출하면 나중에 컴포넌트의 내부를 변경하기가 더 어려워진다는 점에 유의하세요. 일반적으로 버튼이나 텍스트 input과 같이 재사용 가능한 저수준 컴포넌트에서 DOM 노드를 노출하지만, 아바타나 댓글 같은 애플리케이션 레벨의 컴포넌트에서는 노출하고 싶지 않을 것입니다.

# 여러 컴포넌트를 통해 ref 전달하기

ref를 DOM 노드로 전달하지 않고 MyInput과 같은 커스텀 컴포넌트로 전달할 수 있습니다:

```jsx
const FormField = forwardRef(function FormField(props, ref) {
  // ...
  return (
    <>
      <MyInput ref={ref} />
      ...
    </>
  );
});
```

하위 컴포넌트들은 forwardRef 계속 써야함

# DOM 노드 대신 명령형 핸들 노출하기

전체 DOM 노드를 노출하는 대신 보다 제한된 메서드 집합을 사용하여 명령형 핸들이라고 하는 사용자 정의 객체를 노출할 수 있습니다. 이렇게 하려면 DOM 노드를 보유할 별도의 ref를 정의해야 합니다:

```jsx
//App.js

import { useRef } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
    // This won't work because the DOM node isn't exposed:
    // DOM 노드가 노출되지 않았기 때문에 작동하지 않습니다:
    // ref.current.style.opacity = 0.5;
  }

  return (
    <form>
      <MyInput label="Enter your name:" ref={ref} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}

// Myinput.js
import { forwardRef, useRef, useImperativeHandle } from "react";

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(
    ref,
    () => {
      return {
        focus() {
          inputRef.current.focus();
        },
        scrollIntoView() {
          inputRef.current.scrollIntoView();
        },
        // opacity() {
        //   inputRef.current.style.opacity = 0.5;
        // },
      };
    },
    []
  );

  return <input {...props} ref={inputRef} />;
});

export default MyInput;
```

일부 컴포넌트가 MyInput에 대한 ref를 받으면 DOM 노드 대신 { focus, scrollIntoView } 객체만 받습니다. 이를 통해 DOM 노드에 대해 노출하는 정보를 최소한으로 제한할 수 있습니다.

# 주의사항

ref를 과도하게 사용하지 마세요. 노드로 스크롤하기, 노드에 초점 맞추기, 애니메이션 트리거하기, 텍스트 선택하기 등 prop으로 표현할 수 없는 필수적인 동작에만 ref를 사용해야 합니다.

prop으로 무언가를 표현할 수 있다면 ref를 사용해서는 안 됩니다. 예를 들어, Modal 컴포넌트에서 { open, close }와 같은 명령형 핸들을 노출하는 대신 `<Modal isOpen={isOpen} />`과 같이 prop isOpen을 사용하는 것이 더 좋습니다. Effect는 props를 통해 명령형 동작을 노출하는 데 도움이 될 수 있습니다.

# Trouble Shooting

컴포넌트가 forwardRef로 감싸져 있지만, 컴포넌트의 ref는 항상 null입니다.

이거는 받은 ref를 실제로 사용하는 것을 잊어버렸을 떄 나타남

로직이 조건부일 경우 ref가 null일 수 있음

```jsx
const MyInput = forwardRef(function MyInput({ label, showInput }, ref) {
  return (
    <label>
      {label}
      <Panel isExpanded={showInput}>
        <input ref={ref} />
      </Panel>
    </label>
  );
});
```
