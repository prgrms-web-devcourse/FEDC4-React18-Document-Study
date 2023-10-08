useMemo는 리렌더링 사이의 계산 결과를 캐시할 수 있는 React 훅입니다.

> const cachedValue = useMemo(calculateValue, dependencies)

사용법

- 비용이 많이 드는 재계산 생략하기
- 컴포넌트의 리렌더링 건너뛰기
- 다른 훅의 의존성 메모화
- 함수 메모화

컴포넌트 최상단에서 useMemo를 호출하여 리렌더링 사이의 계산 결과를 캐시합니다.

```jsx
import { useMemo } from "react";

function TodoList({ todos, tab }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

calculateValue : 순수함수여야 한다. 인자를 받지 않고, 값을 반환해야한다.
dependencies : 의존성을 Object.is 로 비교. 의존성이 바뀌기 전까지는 함수 계산 결과값 캐싱

비용이 많이 드는 재계산을 위해 사용한다.

```jsx
import { useMemo } from "react";

function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

큰 배열을 필터링하거나, 변환하거나, 고비용의 계산을 수행할 때, 데이터가 변경되지 않았을 때 등등

!note : useMemo는 성능 최적화 목적으로 사용해야 합니다. 이것 없이 코드가 작동하지 않는다면 먼저 근본적인 문제를 찾아 해결하세요. 이후에 다시 useMemo 를 추가하여 성능을 개선할 수 있습니다.

console.time, console.timeEnd 찍어서 시간 측정했을때 1ms 이상 걸리면 메모 써봄직하다. 인위적으로 성능을 느리게 해서 측정도 해보자.(사용자의 컴퓨터가 느릴 경우를 대비)

첫번째 렌더링을 빠르게 해주지는 않는다.

실제로 몇 가지 원칙을 따르면 많은 메모화가 불필요해질 수 있습니다:

컴포넌트가 다른 컴포넌트를 시각적으로 감쌀 때 JSX를 자식으로 받아들이도록 하세요. 이렇게 하면 wrapper 컴포넌트가 자체 state를 업데이트할 때 React는 그 자식 컴포넌트가 다시 렌더링할 필요가 없다는 것을 알 수 있습니다.

로컬 state를 선호하고 필요 이상으로 state를 끌어올리지 마세요. 예를 들어, 최상위 트리나 전역 state 라이브러리에 폼이나 아이템이 호버되었는지와 같은 일시적 state를 두지 마세요.

렌더링 로직을 순수하게 유지하세요. 컴포넌트를 다시 렌더링했을 때 문제가 발생하거나 눈에 띄는 시각적 아티팩트가 생성된다면 컴포넌트에 버그가 있는 것입니다! 메모화하는 대신 버그를 수정하세요.

state를 업데이트하는 불필요한 Effect를 피하세요. React 앱의 대부분의 성능 문제는 컴포넌트를 반복해서 렌더링하게 만드는 Effect에서 발생하는 업데이트 체인으로 인해 발생합니다.

Effect에서 불필요한 의존성을 제거하세요. 예를 들어, 메모화 대신 일부 오브젝트나 함수를 Effect 내부나 컴포넌트 외부로 이동하는 것이 더 간단할 때가 많습니다.

```jsx
import { memo } from "react";

const List = memo(function List({ items }) {
  // ...
});
```

이런식으로 함수형 컴포넌트도 memo를 써서 의존성(여기선 props)이 변경되지 않은 경우 리렌더링이 일어나지 않게 해줄 수 있음. 그런데 부모 컴포넌트에서 리렌더링이 일어나면 props는 객체가 리렌더링 했을 때 새로운 객채로 취급되는 것 처럼 항상 새로운 값을 받아서 memo가 무용지물이 됨. 이때 useMemo해주면 리렌더링을 피할 수 있음

```jsx
export default function TodoList({ todos, tab, theme }) {
  // Tell React to cache your calculation between re-renders...
  // 리렌더링 사이에 계산 결과를 캐싱하도록 합니다...
  // const visibleTodos = filterTodos(todos, tab); 이러면 곤란
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab] // ...so as long as these dependencies don't change...
    // ...따라서 여기의 의존성이 변경되지 않는다면 ...
  );
  return (
    <div className={theme}>
      {/* ...List will receive the same props and can skip re-rendering */}
      {/* ...List는 같은 props를 전달받게 되어 리렌더링을 건너뛸 수 있게 됩니다 */}
      <List items={visibleTodos} />
    </div>
  );
}
```

memo말고 useMemo를 사용해서 jsx 노드를 수동으로 감쌀 수 있긴한데 불편함. 조건부로 수행할 수 없다던지...

다른 훅의 의존성 메모화

맨위 코드는 useMemo 쓰는 의미가 없고,
맨아래가 중간의 코드를 개선한 버전이다.

```jsx
function Dropdown({ allItems, text }) {
  const searchOptions = { matchMode: 'whole-word', text };

  const visibleItems = useMemo(() => {
    return searchItems(allItems, searchOptions);
  }, [allItems, searchOptions]); // 🚩 Caution: Dependency on an object created in the component body
                                 // 🚩 주의: 컴포넌트 내부에서 생성한 객체 의존성
  // ...

  function Dropdown({ allItems, text }) {
  const searchOptions = useMemo(() => {
    return { matchMode: 'whole-word', text };
  }, [text]); // ✅ Only changes when text changes
              // ✅ text 변경시에만 변경됨

  const visibleItems = useMemo(() => {
    return searchItems(allItems, searchOptions);
  }, [allItems, searchOptions]); // ✅ Only changes when allItems or searchOptions changes
                                 // ✅ allItems 또는 searchOptions 변경시에만 변경됨
  // ...

  function Dropdown({ allItems, text }) {
  const visibleItems = useMemo(() => {
    const searchOptions = { matchMode: 'whole-word', text };
    return searchItems(allItems, searchOptions);
  }, [allItems, text]); // ✅ Only changes when allItems or text changes
                        // ✅ allItems 또는 text 변경시에만 변경됨
  // ...
```

함수도

```jsx
export default function Page({ productId, referrer }) {
  const handleSubmit = useMemo(() => {
    return (orderDetails) => {
      post("/product/" + productId + "/buy", {
        referrer,
        orderDetails,
      });
    };
  }, [productId, referrer]);

  return <Form onSubmit={handleSubmit} />;
}
```

이런식으로 메모화 할 수 있는데 useCallback을 쓰는게 더 낫다

troubleshooting
리렌더링할 떄마다 계산이 두번 실행 될때...
계산중에 생성된 객체를 변이 시키는 것은 괜찮다.

```jsx
const visibleTodos = useMemo(() => {
  const filtered = filterTodos(todos, tab);
  // ✅ Correct: mutating an object you created during the calculation
  // ✅ 올바름: 계산 중에 생성한 객체의 변이
  filtered.push({ id: "last", text: "Go for a walk!" });
  return filtered;
}, [todos, tab]);
```

반복문 안에서는 useMemo 못씀

```jsx
function ReportList({ items }) {
  return (
    <article>
      {items.map((item) => (
        //이 안에서 useMemo를 바로 사용할 수 없다.
        <Report key={item.id} item={item} />
      ))}
    </article>
  );
}

function Report({ item }) {
  // ✅ Call useMemo at the top level:
  // ✅ useMemo는 컴포넌트 최상단에서 호출하세요:
  const data = useMemo(() => calculateReport(item), [item]);
  return (
    <figure>
      <Chart data={data} />
    </figure>
  );
}
```
