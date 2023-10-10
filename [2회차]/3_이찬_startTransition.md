# 코드

> startTransition(scope)

# 용도

UI를 차단하지 않고 state를 업데이트할 수 있게 하기 위함

```jsx
import { startTransition } from "react";

function TabContainer() {
  const [tab, setTab] = useState("about");

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```

## 매개변수

scope: 하나 이상의 set 함수를 호출하여 일부 state를 업데이트하는 함수. React는 매개변수 없이 scope를 즉시 호출하고 scope 함수 호출 중에 동기적으로 예약된 모든 state 업데이트를 트랜지션으로 표시합니다. 이는 논블로킹이고, 원치 않는 로딩을 표시하지 않을 것입니다.

## 반환값

아무것도 반환하지 않음

## 주의사항

- startTransition은 트랜지션이 ‘pending’인지 추적하는 방법을 제공하지 않습니다. 트랜지션이 진행 중일 때 ‘pending’ 표시기를 보여주고 싶다면, 대신 useTransition이 필요합니다.

- 해당 state의 set 함수에 접근할 수 있는 경우에만 업데이트를 트랜지션으로 감쌀 수 있습니다. 일부 prop이나 커스텀 훅 값에 대한 응답으로 트랜지션을 시작하려면, 대신 useDeferredValue를 사용해보세요.

- startTransition에 전달하는 함수는 동기식이어야 합니다. React는 이 함수를 즉시 실행하여, 실행하는 동안 발생하는 모든 state 업데이트를 트랜지션으로 표시합니다. 나중에 더 많은 state 업데이트를 수행하려고 하면(예: 타임아웃), 트랜지션으로 표시되지 않습니다.

- 트랜지션으로 표시된 state 업데이트는 다른 state 업데이트에 의해 중단됩니다. 예를 들어, 트랜지션 내에서 차트 컴포넌트를 업데이트한 다음, 차트가 다시 렌더링되는 도중에 입력을 시작하면 React는 입력 업데이트를 처리한 후 차트 컴포넌트에서 렌더링 작업을 다시 시작합니다.

- 트랜지션 업데이트는 텍스트 입력을 제어하는 데 사용할 수 없습니다.

- 진행 중인 트랜지션이 여러 개 있는 경우, React는 현재 트랜지션을 함께 일괄 처리합니다. 이는 향후 릴리스에서 제거될 가능성이 높은 제한 사항입니다.

# state 업데이트를 논블로킹 트랜지션으로 표시하기

state 업데이트는 startTransition 호출로 감싸 트랜지션으로 표시할 수 있습니다:

```jsx
import { startTransition } from "react";

function TabContainer() {
  const [tab, setTab] = useState("about");

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```

트랜지션을 사용하면 느린 기기에서도 사용자 인터페이스 업데이트를 반응성 있게 유지할 수 있습니다.

트랜지션을 사용하면 재렌더링 도중에도 UI가 반응성을 유지합니다. 예를 들어, 사용자가 탭을 클릭했다가 마음이 바뀌어 다른 탭을 클릭하면 첫 번째 리렌더링이 완료될 때까지 기다릴 필요 없이 다른 탭을 클릭할 수 있습니다.

# useTransition과 다른 점?

useTransition은 훅이므로 컴포넌트나 커스텀 훅 내부에서만 호출할 수 있습니다. 다른 곳(예: 데이터 라이브러리)에서 트랜지션을 시작해야 하는 경우, 대신 독립형 startTransition을 호출하세요.
