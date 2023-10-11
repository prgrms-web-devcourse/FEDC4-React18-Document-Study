# 코드

```jsx
<select>
  <option value="someOption">Some option</option>
  <option value="otherOption">Other option</option>
</select>
```

# 용도

html의 option 태그와 같다

select 태그 안에 옵션을 렌더링 가능

```jsx
<select>
  <option value="someOption">Some option</option>
  <option value="otherOption">Other option</option>
</select>
```

# props

[일반적인 엘리먼드 props를 지원](https://react-ko.dev/reference/react-dom/components/common#props)

disabled: boolean값. true 인 경우 옵션을 선택할 수 없으며 흐리게 보입니다.

label: string값. 옵션의 의미를 지정합니다. 지정하지 않으면 옵션 내부의 텍스트가 사용됩니다.

value: 폼에서 부모 요소인 select를 제출할 때, 이 옵션이 선택된 경우에 사용됩니다.

# 주의사항

React는 `<option>`에서 selected 속성을 지원하지 않습니다. 대신 비제어 셀렉트 박스의 경우에는 `<select defaultValue>`로, 제어 셀렉트 박스의 경우에는 `<select value>`로, 이 옵션의 value를 부모에 전달하세요.
