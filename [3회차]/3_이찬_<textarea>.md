# 코드

```jsx
<textarea name="postContent" />
```

# 용도

html의 textarea 태그와 같다

```jsx
export default function NewPost() {
  return (
    <label>
      Write your post:
      <textarea name="postContent" rows={4} cols={40} />
    </label>
  );
}
```

# props

[일반적인 엘리먼드 props를 지원](https://react-ko.dev/reference/react-dom/components/common#props)

`<textarea>`는 모든 공통 엘리먼트의 props를 지원합니다.

value prop을 전달함으로써 이를 제어 컴포넌트가 되게 할 수 있습니다:

value: 문자열. textarea 내부의 텍스트를 제어합니다.

value를 전달할 때는 전달된 value를 업데이트 하는 onChange 핸들러도 함께 전달해야 합니다.

`<textarea>`가 비제어 컴포넌트인 경우에는, 대신 defaultValue를 전달할 수 있습니다:

다음 `<textarea>` prop들은 비제어 및 제어 컴포넌트 모두에 영향을 미칩니다:

autoComplete: 'on' 혹은 'off'. 자동 완성 동작을 지정합니다.

autoFocus: 불리언. true일 경우 마운트시 엘리먼트에 초점이 맞춰집니다.

children: `<textarea>`는 자식 요소를 받지 않습니다. 초기값을 지정하기 위해서는 defaultValue를 사용하세요.

cols: 숫자. 표준 문자 너비를 기준으로 기본 칸 수를 지정합니다. 기본 값은 20입니다.

disabled: 불리언. true일 경우, 입력이 비활성화되고 흐릿하게 표시됩니다.

form: 문자열. 이 textarea가 속한 `<form>`의 id를 지정합니다. 생략하면 가장 가까운 상위 form이 됩니다.

maxLength: 숫자. 텍스트의 최대 길이를 지정합니다.

minLength: 숫자. 텍스트의 최소 길이를 지정합니다.

name: 문자열. 폼 제출시 해당 textarea의 이름을 지정합니다.

onChange: 이벤트 핸들러. 제어 컴포넌트로 사용할 때 필요합니다. 사용자에 의해 입력 값이 변경되는 즉시 실행됩니다. (예: 각 키 입력시 실행됨). 브라우저의 input event처럼 동작합니다.

onChangeCapture: 캡쳐 단계에 실행되는 버전의 onChange입니다.

onInput: 이벤트 핸들러. 사용자에 의해 값이 변결될 때마다 실행됩니다. 역사적인 이유로 React에서는 일반적으로 비슷하게 작동하는 onChange를 대신 사용합니다.

onInputCapture: 캡쳐 단계에 실행되는 버전의 onInput입니다.

onInvalid: 이벤트 핸들러. 폼 제출시 유효성 검사에 실패하면 발생합니다. 빌트인 invalid 이벤트와는 달리, React onInvalid 이벤트는 버블이 발생합니다.

onInvalidCapture: 캡쳐 단계에 실행되는 버전의 onInvalid입니다.

onSelect: 이벤트 핸들러. `<textarea>`의 내부 선택 영역이 변경되면 발생합니다. React는 비어있는 선택과 (선택에 영향을 줄 수 있는) 편집에 대해서도 onSelect 이벤트가 발동되도록 확장했습니다.

onSelectCapture: 캡쳐 단계에 실행되는 버전의 onSelect입니다.

placeholder: 문자열. 입력 값이 비어 있을 때 희미한 색상으로 표시됩니다.

readOnly: 불리언. true 일 경우 유저는 textarea을 수정할 수 없습니다.

required: 불리언. true일 경우 form 제출시 값이 있어야 합니다.

rows: 숫자. 표준 문자 높이를 기준으로 기본 줄 수를 지정합니다. 기본 값은 2입니다.

wrap: 'hard', 'soft', 혹은'off'. form 제출시 텍스트를 어떻게 줄바꿈할지를 지정합니다.

# 주의사항

`<textarea>something</textarea>`처럼 자식 요소를 전달하는 것은 허용되지 않습니다. 초기값은 defaultValue를 사용하세요.

문자열 value prop을 제공하면 제어 컴포넌트로 취급됩니다.

제어 컴포넌트이면서 동시에 비제어 컴포넌트일 수는 없습니다.

생명주기 동안 제어 컴포넌트와 비제어 컴포넌트 사이를 전환할 수 없습니다.

제어컴포넌트는 값을 동기적으로 업데이트 하는 onChange 이벤트 핸들러가 필요합니다.
