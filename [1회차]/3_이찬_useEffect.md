리액트 훅은 함수 컴포넌트에서도 클래스 컴포넌트와 동일한 기능들을 제공하기 위해서 나온 것

리액트 훅은 함수 컴포넌트의 렌더링 과정 중간에 끼어 들어가 다양한 기능들을 사용할 수 있도록 해줌

훅은 무조건 함수형 컴포넌트의 최상위 레벨에서만 호출해야한다(조건문 같은 곳에서 사용하면 안된다. 클래스 컴포넌트에서 사용해도 안된다.)

---

사용법

- 외부 시스템에 연결하기
- 커스텀 훅으로 Effect 감싸기
- React가 아닌 위젯 제어하기
- Effect로 데이터 페칭하기
- 반응형 의존성 지정
- Effect의 이전 state를 기반으로 state 업데이트하기
- 불필요한 객체 의존성 제거하기
- 불필요한 함수 의존성 제거하기
- Effect에서 최신 props 및 state 읽기
- 서버와 클라이언트에 서로 다른 콘텐츠 표시하기

useEffect(setup, dependencies?)

직역하면 sideEffect(순수함수가 아닌 함수가 발생시키는 액션)를 사용한다는 뜻?

> 컴포넌트의 최상위 레벨에서 useEffect를 호출하여 Effect를 선언

```jsx
import { useEffect } from "react";
import { createConnection } from "./chat.js";

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState("https://localhost:1234");

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);
  // ...
}

// 액션을 일으키는 함수들을 다 effect로 빼는듯
```

useEffect(setup,dependencies?) 이렇게 구성이 되고,

setup 함수는 **_컴포넌트가 DOM에 추가되면_** 셋업함수를 실행한다.

의존성의 변경되어 리렌더링이 발생하면, 클린업 함수가 있는 경우, 이전값(?)으로 클린업 함수를 실행한 다음, 새 값으로 셋업 함수를 실행한다.

의존성은 props, state, 컴포넌트 **_내부_** 에서 직접 선언한 모든 변수와 함수를 포함 Object.is 로 비교. 의존성을 지정하지 않으면 리렌더링마다 Effect가 다시 실행됨

> Object.is에서 같은 값이라고 판정되는 경우

- both undefined
- both null
- both true or both false
- both strings of the same length with the same characters in the same order
- both the same object (**meaning both values reference the same object in memory**)
- both BigInts with the same numeric value
- both symbols that reference the same symbol value
- both numbers and
  - both +0
  - both -0
  - both NaN
    or both non-zero, not NaN, and have the same value

useEffect는 undefined를 반환한다. sideEffect만 실행시키고 종료되는 함수가 undefined를 반환하는 거랑 같은 맥락인듯.

useEffect 쓸 때 주의사항

- 컴포넌트의 최상위 레벨에서만 쓰자
- 외부 시스템과 동기화하려는 목적이 아니라면 필요 없을 수도 있다.
- strict 모드 켜서 문제가 발생하면 클린업 기능을 구현해야 한다.(strict 모드 쓰면 셋업 + 클린업을 한번 실제 셋업전에 하는데 스트레스 테스트임)
- 의존성 중 일부가 컴포넌트 내부에 정의된 **객체** 또는 **함수**이면 Effect가 필요 이상으로 실행될 수 있음

---

스냅샷 : 렌더링 당시의 state를 사용해 계산

```jsx
function handleClick() {
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
}
// 이런거 써도 한번 밖에 나이 증가 안됨

function handleClick() {
  setAge((a) => a + 1); // setAge(42 => 43)
  setAge((a) => a + 1); // setAge(43 => 44)
  setAge((a) => a + 1); // setAge(44 => 45)
}
//이렇게 써야 증가됨
//a=>a+1 이 업데이트 함수라서 대기중인 state를 가져와서 다음 state를 계산하기 때문
```

---

위와 같이 age + 1 대신에 a=>a+1을 써줘야 하는 경우가 있을 수 있음. (1초마다 count를 늘려서 표시하고 싶은데 의존성에 count가 들어있으면 셋업, 클린업, 셋업, 클린업... 이래야 하니까 c=>c+1을 써서 리액트 업데이트 큐에서 state를 쓰고 의존성은 제거) 아니면 useEffectEvent 같은거 쓰거나

- Effect가 상호작용(클릭같은거)으로 인한 것이 아니라면, 업데이트된 화면이 먼저 그려지고 Effect를 실행함. Effect가 시각적인 작업을 하고 있고, 지연이 눈에 띄는 경우(깜빡임) [useLayoutEffect](https://react-ko.dev/reference/react/useLayoutEffect)(화면이 다시 그려지는 것을 방지. 근데 useEffect보다 속도 좀 느림)를 쓰자.

- 상호작용으로 인해 Effect가 발생해도 브라우저는 Effect 내부의 state 업데이트 처리 전에 화면을 다시 그릴 수 있다. 만약 화면을 다시 칠하지 못하도록 차단해야 하는 경우라면 useLayoutEffect를 쓰자

사용법

- [외부 시스템에 연결하기](https://react-ko.dev/reference/react/useEffect#connecting-to-an-external-system). 예시에서는 채팀방 사례가 나온다.계속 채팅방을 옮겨다닐때 연결 끊고 다시 받고 해야하니까.

  - 그리고 마지막에 "useEffect 한번만 수행될 수 있게 하는 방법이 있을까요?" 에 대한 대답으로 "한번만 실행시키는 방법을 찾지 말고 두번 실행되도 멀쩡하게 코드 짜라" 라고 함
  - 모든 Effect를 독립적인 프로세스로 작성하고 한 번에 하나의 셋업/클린업 주기만 생각해라

- note

  - setInterval() 및 clearInterval()로 관리되는 타이머.
  - window.addEventListener() 및 window.removeEventListener()를 사용하는 이벤트 구독.
  - animation.start() 및 animation.reset()과 같은 API가 있는 타사 애니메이션 라이브러리.
    와 같은 것들은 Effect 쓰자
  - DOM 조작시에도 사용(모달제어같은거)

- 커스텀 훅으로 빼면 더 추상화 레벨이 낮은 함수를 만들 수 있다.
- React가 아닌 위젯 제어
- 데이터 페칭 할 때(조건경합이 일어나지 않는다고 하는데, 데이터가 불러와지는 타이밍에 상관이 없어진다는 뜻 같다.)

  - 그런데 useEffect에서 데이터 페칭을 많이하면 최적화가 어려워질 수도 있음
  - ssr에 쓰지못한다던지, 캐싱을 할 수 없다던지, 상용구 코드(보일러플레이트)가 많이 필요하다던지
  - 그래서 framework의 데이터페칭 메커니즘을 이용한다던지, client-side 캐시를 사용하거나 구축한다던지(리액트 쿼리, 리액트라우터 등등) 하자

- 의존성이 필요없는 경우는 명시적으로 빈 배열을 넣어주자. 아예 안넣어줄수도 있지만 리렌더링이 필요하지 않은 상황에서도 리렌더링이 일어날 수 있다.

- 렌덜이 중에 생성된 객체를 의존성으로 하지 말고, useEffect 내부에서 객체를 생성하자. 그래야 리렌더링시 불필요하게 useEffect가 사용되는 경우를 막을 수 있다.

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const options = { // 🚩 This object is created from scratch on every re-render
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options); // It's used inside the Effect
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // 🚩 As a result, these dependencies are always different on a re-render
  // ...
```

- 함수도 마찬가지로 useEffect 내에서 쓰자

! state가 useEffect 안에서 업데이트 되고 그 state가 의존성이면 무한렌더링에 걸린다
