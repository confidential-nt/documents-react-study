# Lifecycle of Reactive Effects

## 들어가면서

Effect는 컴포넌트와 다른 생명주기를 가짐. 컴포넌트는 마운트, 업데이트 또는 마운트 해제할 수 있음. **Effect는 동기화를 시작하고 나중에 동기화를 중지하는 두 가지 작업만 할 수 있음.** 이 사이클은 '시간이 지남에 따라 변하는 props와 state에 의존하는' Effect의 경우 여러 번 발생할 수 있음. React는 Effect의 의존성을 올바르게 지정했는지 확인하는 린터 규칙을 제공. 이렇게 하면 Effect가 최신 props와 state에 동기화됨.

## Effect의 생명주기

모든 React 컴포넌트는 동일한 생명주기를 거침:

- 컴포넌트는 화면에 추가될 때 마운트됨.
- 컴포넌트는 새로운 props나 state를 받으면 업데이트됨. 이는 보통 상호작용에 대한 응답으로 발생.
- 화면에서 제거되면 컴포넌트가 마운트 해제됨.

이는 컴포넌트에 대해 생각하는 좋은 방법이지만 Effect에 대해서는 아님. **대신 각 Effect를 컴포넌트의 생명주기와 독립적으로 생각해야.** . Effect는 외부 시스템을 현재 props 및 state에 동기화하는 방법을 설명함. 코드가 변경됨에 따라, 이 동기화를 더 자주 또는 덜 자주 수행해야. (동기화를 조정하는 게 필요할 때가 있음.)

이 점을 설명하기 위해 컴포넌트를 채팅 서버에 연결하는 Effect를 예로 들어보자.

```js
const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

보다시피 Effect의 본문에는 동기화 시작 방법이 명시되어 있음. 그리고 Effect에서 반환되는 클린업 함수는 동기화를 중지하는 방법을 지정.

직관적으로 React는 컴포넌트가 마운트될 때 동기화를 시작하고 컴포넌트가 마운트 해제될 때 동기화를 중지할 것이라고 생각할 수 있음. **하지만 때로는 컴포넌트가 마운트된 상태에서 동기화를 여러 번 시작하고 중지해야 할 수도 있음.**

이러한 동작이 필요한 이유와 발생 시기, 그리고 이러한 동작을 제어할 수 있는 방법을 살펴보자.

- 참고:
  - 일부 Effect는 클린업 함수를 전혀 반환하지 않음. 대부분의 경우 함수를 반환하고 싶겠지만, 그렇지 않은 경우 React는 아무 작업도 하지 않는 빈 클린업 함수를 반환한 것처럼 동작함.

## 동기화가 두 번 이상 수행되어야 하는 이유

이 ChatRoom 컴포넌트가 사용자가 드롭다운에서 선택한 roomId prop을 받는다고 가정해 보자. 처음에 사용자가 "general" 채팅방을 roomId로 선택했다고 가정해 보자. 앱에 "general" 채팅방이 표시됨.

```js
const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId /* "general" */ }) {
  // ...
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

UI가 표시되면 React가 Effect를 실행하여 동기화를 시작. "general" 방에 연결됨.

```js
function ChatRoom({ roomId /* "general" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Connects to the "general" room
    connection.connect();
    return () => {
      connection.disconnect(); // Disconnects from the "general" room
    };
  }, [roomId]);
  // ...
```

지금까지는 괜찮.

이후, 사용자가 드롭다운에서 다른 방을 선택함.(예: "travel"). 먼저 React가 UI를 업데이트.

```js
function ChatRoom({ roomId /* "travel" */ }) {
  // ...
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

다음에 어떤 일이 일어날지 생각해 보자. 사용자는 UI에서 "travel"이 선택된 대화방임을 알 수 있음. 하지만 지난번에 실행된 Effect는 여전히 "general"대화방에 연결되어 있음. **roomId prop이 변경되었기 때문에 이전에 Effect("general" 방에 연결)가 수행한 작업이 더 이상 UI와 일치하지 않음.**

이 시점에서 우리는 React가 두 가지 작업을 수행하기를 원함.

1. 이전 roomId와의 동기화 중지 ("general" 룸에서 연결 해제)
2. 새 roomId와 동기화 시작 ("travel" 룸과 연결)

**다행히 우리는 이미 이 두 가지를 수행하는 방법을 React에 가르침.** Effect의 본문은 동기화를 시작하는 방법을 지정하고, 클린업 함수는 동기화를 중지하는 방법을 지정함. 이제 React가 해야 할 일은 올바른 순서로, 올바른 props와 state로 호출하기만 하면 됨. 정확히 어떻게 일어나는지 살펴보자.

## React가 Effect를 재동기화 하는 방법

ChatRoom컴포넌트의 roomId prop이 새로운 값을 받았다는 것을 기억. 이전에는 "general"이었지만 이제는 "travel". 다른 방에 다시 연결하려면 React가 Effect를 다시 동기화해야.

동기화를 중지하기 위해, React는 "general" 방에 연결한 후 Effect가 반환한 클린업 함수를 호출함. roomId가 "general"이므로, 클린업 함수는 "general" 방에서 연결을 끊음.

```js
function ChatRoom({ roomId /* "general" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Connects to the "general" room
    connection.connect();
    return () => {
      connection.disconnect(); // Disconnects from the "general" room
    };
    // ...
```

그러면 React는 **이 렌더링 중에 우리가 제공한 Effect를 실행.** 이번에는 roomId가 "travel"이므로 (클린업 함수가 호출되기 전까지) "travel"채팅방과 동기화되기 시작함.

```js
function ChatRoom({ roomId /* "travel" */ }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Connects to the "travel" room
    connection.connect();
    // ...

```

덕분에 이제 사용자가 UI에서 선택한 방과 동일한 방에 연결됨.

컴포넌트가 다른 roomId로 다시 렌더링할 때마다 Effect가 다시 동기화됨. 예를 들어, 사용자가 roomId를 "travel"에서 "music"으로 변경한다고 가정.. React는 다시 클린업 함수를 호출하여 Effect 동기화를 중지("travel" 방에서 연결을 끊습니다). 그런 다음 새 roomId prop으로 본문을 실행하여 다시 동기화를 시작("music" 방에 연결).

마지막으로 사용자가 다른 화면으로 이동하면 ChatRoom이 마운트 해제됨. 이제 연결 상태를 유지할 필요가 전혀 없음. React는 마지막으로 Effect의 동기화를 중지하고 "music" 채팅방에서 연결을 끊음.

## Effect의 관점에서 생각하기

ChatRoom 컴포넌트의 관점에서 일어난 모든 일을 요약.

1. roomId가 "general"로 설정된 상태로 ChatRoom이 마운트됨
1. roomId가 "travel"로 설정된 상태로 ChatRoom이 업데이트됨
1. roomId가 "music"로 설정된 상태로 ChatRoom이 업데이트됨
1. ChatRoom 마운트 해제됨

컴포넌트 생명주기의 각 시점에서 Effect는 서로 다른 작업을 수행.

1. Effect가 "general" 방에 연결됨
1. Effect가 "general" 방과의 연결이 끊어지고 "travel" 방에 연결됨
1. Effect가 "travel" 방과의 연결이 끊어지고 "music" 방에 연결됨
1. Effect가 "music" 방과의 연결이 끊어짐

이제 Effect 자체의 관점에서 무슨 일이 일어났는지 생각해 보자.

```js
useEffect(() => {
  // Your Effect connected to the room specified with roomId...
  const connection = createConnection(serverUrl, roomId);
  connection.connect();
  return () => {
    // ...until it disconnected
    connection.disconnect();
  };
}, [roomId]);
```

이 코드의 구조는 어떤 일이 일어났는지 겹치지 않는 기간의 연속으로 보는 데 영감을 줄 수 있음.

1. Effect가 "general" 방에 연결됨 (연결이 끊어질 때까지)
1. Effect가 "travel" 방에 연결됨 (연결이 끊어질 때까지)
1. Effect가 "music" 방에 연결됨 (연결이 끊어질 때까지)

이전에는 컴포넌트의 관점에서 생각했음. **컴포넌트의 관점에서 보면 Effect를 “렌더링 후” 또는 “마운트 해제 전”과 같은 특정 시점에 실행되는 “콜백” 또는 “생명주기 이벤트”로 생각하기 쉬웠음. 이러한 사고 방식은 매우 빠르게 복잡해지므로 피하는 것이 가장 좋다.**

**대신 항상 한 번에 하나의 시작/중지 사이클에만 집중. 컴포넌트를 마운트, 업데이트 또는 마운트 해제하는 것은 중요하지 않음. 동기화를 시작하는 방법과 중지하는 방법만 설명하면 됨. 이 작업을 잘 수행하면 필요한 횟수만큼 Effect를 시작하고 중지할 수 있는 탄력성을 확보할 수 있음.**

**JSX를 생성하는 렌더링 로직을 작성할 때 컴포넌트가 마운트되는지 업데이트되는지 생각하지 않는 것을 떠올리면 이해가 쉬울 것.** 화면에 무엇이 표시되어야 하는지 설명하면 나머지는 React가 알아서 처리함.

Effect를 작성할 때, 그냥 동기화 시 수행할 작업 - 동기화 중지 시 수행할 작업만 생각하라는 의미인듯.

## React가 Effect의 재동기화 가능 여부를 확인하는 방법

다음은 실제 사용 가능한 예제. “Open chat”을 눌러 ChatRoom 컴포넌트를 마운트 해보라.

```js
import { useState, useEffect } from "react";
import { createConnection } from "./chat.js";

const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
  return <h1>Welcome to the {roomId} room!</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState("general");
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        Choose the chat room:{" "}
        <select value={roomId} onChange={(e) => setRoomId(e.target.value)}>
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? "Close chat" : "Open chat"}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

컴포넌트가 처음 마운트될 때 3개의 로그가 표시됨.

1. ✅ Connecting to "general" room at https://localhost:1234... (development-only)
1. ❌ Disconnected from "general" room at https://localhost:1234. (development-only)
1. ✅ Connecting to "general" room at https://localhost:1234...

처음 두 개의 로그는 개발 모드 전용. 개발 모드에서 React는 항상 각 컴포넌트를 한 번씩 다시 마운트.

**개발 모드에서 React는 즉시 강제로 동기화를 수행하여 Effect가 다시 동기화될 수 있는지 확인함.** 도어락이 작동하는지 확인하기 위해 문을 열었다가 한 번 더 닫는 것과 비슷. React는 개발 중에 Effect를 한 번 더 시작하고 중지하여 클린업 함수를 잘 구현했는지 확인함. -> 동기화 했다가 중지하고 다시 동기화하는 과정이 정상적으로 잘 이루어지냐...

실제로 Effect가 다시 동기화되는 주된 이유는 Effect가 사용하는 일부 데이터가 변경된 경우임. 위의 예제에서도 roomId가 변경되면 Effect가 다시 동기화되는 것을 확인할 수 있음.

그러나 재동기화가 필요한 더 특이한 경우도 있음. 예를 들어, 채팅이 열려 있는 상태에서 위의 샌드박스에서 serverUrl을 편집해 보라. 코드 편집에 대한 응답으로 Effect가 어떻게 다시 동기화되는지 주목하라. 앞으로 React는 재동기화에 의존하는 더 많은 기능을 추가할 수 있음.

## React가 Effect의 재동기화 필요성을 인식하는 방법

roomId가 변경되었을 때 React가 Effect를 다시 동기화해야 한다는 것을 어떻게 알았는지 궁금할 것. 그 이유는 의존성 목록에 roomId를 포함시킴으로써 코드가 roomId에 의존하고 있음을 React에 알렸기 때문.

```js
function ChatRoom({ roomId }) { // The roomId prop may change over time
                                // roomId prop은 시간에 따라 바뀔 수 있음
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // This Effect reads roomId
                                                            // 이 Effect는 roomId를 읽음
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]); // So you tell React that this Effect "depends on" roomId
                // 이 Effect가 roomId에 의존함을 React에 알림.
  // ...
```

작동 방식은 다음과 같음.

1. 당신은 roomId는 prop이므로 시간이 지남에 따라 변경될 수 있다는 것을 알고 있음.
1. 당신은 Effect가 roomId를 읽는다는 것을 알았음. (따라서 해당 로직은 나중에 변경될 수 있는 값에 의존함).
1. 당신은 이 때문에 Effect의 의존성으로 지정함(roomId가 변경될 때 다시 동기화되도록).

컴포넌트가 다시 렌더링될 때마다 React는 사용자가 전달한 의존성 배열을 살펴봄. 배열의 값 중 하나라도 이전 렌더링 중에 전달한 동일한 지점의 값과 다르면 React는 Effect를 다시 동기화.

예를 들어, 초기 렌더링 중에 ["general"]을 전달했고 나중에 다음 렌더링 중에 ["travel"]을 전달한 경우, React는 "general"과 "travel"을 비교. 이 값은 (Object.is와 비교했을 때) 다른 값이기 때문에 React는 Effect를 다시 동기화할 것. 반면 컴포넌트가 다시 렌더링될 때 roomId가 변경되어있지 않은 경우 Effect는 동일한 방에 연결된 상태로 유지됨.

## 각각의 Effect는 별도의 동기화 프로세스를 나타냅니다.

이 로직은 이미 작성한 Effect와 동시에 실행되어야 하므로 **관련 없는 로직을 Effect에 추가하지 말 것.** 예를 들어, 사용자가 chat room을 방문할 때 분석 이벤트를 전송하고 싶다고 가정해 보자. 이미 roomId에 의존하는 Effect가 있으므로 바로 그 Effect에서 분석 이벤트 호출을 추가하고 싶을 수 있음.

```js
function ChatRoom({ roomId }) {
  useEffect(() => {
    logVisit(roomId);
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

하지만 나중에 이 Effect에 연결을 다시 설정해야 하는 다른 의존성을 추가한다고 가정. 이 Effect가 다시 동기화되면 의도하지 않은 동일한 방에 대해 logVisit(roomId)도 호출하게 됨. **방문을 기록하는 것은 연결과는 별개의 프로세스.** 그렇기 때문에 두 개의 개별 Effect로 작성해야 함.

```js
function ChatRoom({ roomId }) {
  useEffect(() => {
    logVisit(roomId);
  }, [roomId]);

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    // ...
  }, [roomId]);
  // ...
```

**코드의 각 Effect는 별도의 독립적인 동기화 프로세스를 나타내야**

위의 예시에서는 한 Effect를 삭제해도 다른 Effect의 로직이 깨지지 않음. 이는 서로 다른 것을 동기화하므로 분리하는 것이 합리적이라는 것을 나타냄. 반면 일관된 로직을 별도의 Effect로 분리하면 코드가 “더 깔끔해” 보일 수 있지만 유지 관리가 더 어려워짐. **따라서 분리할때는 코드가 더 깔끔해 보이는지 여부가 아니라 프로세스가 동일한지 또는 분리되어 있는지를 고려해야 함.**

## Effect는 ‘반응형(Reactive) 값’에 반응합니다

다음 코드에서 Effect는 두 개의 변수(serverUrl 및 roomId)를 읽지만, 의존성에는 오직 roomId만 지정.

```js
const serverUrl = "https://localhost:1234";

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

serverUrl을 의존성에 지정할 필요가 없는 이유가 무엇일까?

**이는 리렌더링으로 인해 serverUrl 이 변경되지 않기 때문.** 컴포넌트가 어떤 이유로 몇 번이나 다시 렌더링하든 항상 동일함. serverUrl 은 절대 변하지 않으므로 의존성으로 지정하는 것은 의미가 없음. 결국, 의존성은 시간이 지남에 따라 변경될 때만 무언가를 수행함.

반면 roomId는 다시 렌더링할 때 달라질 수 있음. **컴포넌트 내부에서 선언된 props, state 및 기타 값은 렌더링 중에 계산되고 React 데이터 흐름에 참여하기 때문에 반응형.**

만약 serverUrl이 state 변수라면, 반응형일 것. 반응형 값은 의존성에 포함되어야 함.

serverUrl을 의존성으로 포함하면 serverUrl가 변경된 후 Effect가 다시 동기화되도록 할 수 있음.

예시)

```js
import { useState, useEffect } from "react";
import { createConnection } from "./chat.js";

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState("https://localhost:1234");

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);

  return (
    <>
      <label>
        Server URL:{" "}
        <input
          value={serverUrl}
          onChange={(e) => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState("general");
  return (
    <>
      <label>
        Choose the chat room:{" "}
        <select value={roomId} onChange={(e) => setRoomId(e.target.value)}>
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

roomId 또는 serverUrl과 같은 반응형 값을 변경할 때마다 Effect가 채팅 서버에 다시 연결됨.

## 빈 의존성을 가지고 있는 Effect의 의미

serverUrl과 roomId를 모두 컴포넌트 외부로 이동하면 어떻게 될까?

```js
const serverUrl = "https://localhost:1234";
const roomId = "general";

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ All dependencies declared
  // ...
}
```

이제 Effect의 코드는 반응형 값을 사용하지 않으므로 의존성이 비어 있을 수 있음.

컴포넌트의 관점에서 생각해보면, 빈 [] 의존성 배열은 이 Effect가 컴포넌트가 마운트될 때만 채팅방에 연결되고 컴포넌트가 마운트 해제될 때만 연결이 끊어진다는 것을 의미. (React는 개발 모드에서 로직을 테스트하기 위해 한 번 더 동기화한다는 점을 기억).

**하지만 Effect의 관점에서 생각하면 마운트 및 마운트 해제에 대해 전혀 생각할 필요가 없음. 중요한 것은 Effect가 동기화를 시작하고 중지하는 작업을 지정한 것. 현재는 반응형 의존성이 없습니다. 하지만 사용자가 시간이 지남에 따라 roomId 또는 serverUrl을 변경하기를 원한다면(그래서 반응형이어야 한다면) Effect의 코드는 변경되지 않음. 의존성에 추가하기만 하면 됨.**

## 컴포넌트 본문에서 선언된 모든 변수는 반응형입니다

props와 state만 반응형 값인 것은 아님. 이들로부터 계산하는 값들 역시 반응형. props나 state가 변경되면 컴포넌트가 다시 렌더링되고 그로부터 계산된 값도 변경됨. 그렇기 때문에 Effect가 사용하는 컴포넌트 본문의 모든 변수는 Effect 의존성 목록에 있어야 함.

사용자가 드롭다운에서 채팅 서버를 선택할 수도 있고, 설정에서 기본 서버를 구성할 수도 있다고 가정해 보자. 이미 settings state를 context에 넣어서 해당 context에서 읽었다고 가정해 보자. 이제 props에서 선택한 서버와 context에서 기본 서버를 기준으로 serverUrl을 계산함.

```js
function ChatRoom({ roomId, selectedServerUrl }) {
  // roomId is reactive
  const settings = useContext(SettingsContext); // settings is reactive
  const serverUrl = selectedServerUrl ?? settings.defaultServerUrl; // serverUrl is reactive
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Your Effect reads roomId and serverUrl
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]); // So it needs to re-synchronize when either of them changes!
  // ...
}
```

이 예제에서 serverUrl은 prop이나 state 변수가 아님. 렌더링 중에 계산하는 일반 변수. 하지만 렌더링 중에 계산되므로 리렌더링으로 인해 변경될 수 있음. 따라서 serverUrl은 반응형 변수.

**컴포넌트 내부의 모든 값(컴포넌트 본문의 props, state, 변수 포함)은 반응형.** 모든 반응형 값은 다시 렌더링할 때 변경될 수 있으므로 반응형 값을 Effect의 의존성으로 포함시켜야 함.

즉, Effect는 컴포넌트 본문의 모든 값에 “반응”함.

- 참고: 전역 또는 변이 가능한 값이 의존성이 될 수 있나요?
  - 변이 가능한 값(전역 변수 포함, 특히 외부에 있는 값들..)은 not reactive하다.
  - location.pathname과 같은 변이 가능한 값은 의존성이 될 수 없음. 이 값은 변이 가능하므로 React 렌더링 데이터 흐름 외부에서 언제든지 바뀔 수 있음. 이 값을 변경해도 컴포넌트가 다시 렌더링되지 않음.
  - 따라서 이를 의존성에 지정하더라도 React는 이 값이 변경될 때 Effect를 다시 동기화해야 하는지 알 수 없음.
  - 또한 렌더링 도중(의존성을 계산할 때) 변경 가능한 데이터를 읽는 것은 렌더링의 순수성을 깨뜨리기 때문에 React의 규칙을 위반함.
  - 대신, useSyncExternalStore를 사용하여 외부의 변경 가능한 값을 읽고 구독해야 함.
  - ref.current와 같이 변이 가능한 값 또는 이 값으로부터 읽은 것 역시 의존성이 될 수 없음.
  - useRef가 반환하는 ref 객체 자체는 의존성이 될 수 있지만, current 프로퍼티는 의도적으로 변이 가능함. 이를 통해 리렌더링을 촉발하지 않고도 무언가를 추적할 수 있음.
  - 하지만 이를 변경하더라도 리렌더링을 촉발하지는 않기 때문에, 이는 반응형 값이 아니며, React는 이 값이 변경될 때 Effect를 다시 실행해야 할지 알 수 없음.
  - 이 페이지 아래에서 배우게 되겠지만, 린터는 이러한 문제를 자동으로 확인해 줌.

## React는 모든 반응형 값을 의존성으로 지정했는지 검토합니다

린터가 React에 맞게 구성된 경우, Effect 코드에서 사용되는 모든 반응형 값이 해당 의존성으로 선언되었는지 확인함. 예를 들어, 다음 코드는 roomId 나 serverUrl가 모두 반응형이므로 린트 오류.

```js
import { useState, useEffect } from "react";
import { createConnection } from "./chat.js";

function ChatRoom({ roomId }) {
  // roomId is reactive
  const [serverUrl, setServerUrl] = useState("https://localhost:1234"); // serverUrl is reactive

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // <-- Something's wrong here!

  return (
    <>
      <label>
        Server URL:{" "}
        <input
          value={serverUrl}
          onChange={(e) => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState("general");
  return (
    <>
      <label>
        Choose the chat room:{" "}
        <select value={roomId} onChange={(e) => setRoomId(e.target.value)}>
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

이것은 React 오류처럼 보일 수 있지만 실제로는 코드의 버그를 지적하는 것. roomId와 serverUrl은 시간이 지남에 따라 변경될 수 있지만, 변경 시 Effect를 다시 동기화하는 것을 잊어버리고 있음. 결과적으로 사용자가 UI에서 다른 값을 선택한 후에도 초기 roomId 와 serverUrl에 연결된 상태로 유지됨.

버그를 수정하려면 린터의 제안에 따라 Effect의 의존성 요소로 roomId 및 serverUrl을 지정하면 됨.

- 참고: 어떤 경우에는 컴포넌트 내부에서 값이 선언되더라도 절대 변하지 않는다는 것을 React가 알고 있음. 예를 들어, useState에서 반환된 설정자 함수와 useRef에서 반환된 ref 객체는 리렌더링 시 변경되지 않도록 보장되는 안정적인 값. 안정적인 값은 반응하지 않으므로 린터를 사용하면 목록에서 생략할 수 있음. 그러나 이러한 값을 포함하는 것은 허용됨. 변경되지 않으므로 상관없음.

## 재동기화를 원치 않는 경우엔 어떻게 해야 하나요?

이전 예제에서는 roomId와 serverUrl 를 의존성으로 나열하여 린트 오류를 수정함.

그러나 대신 이러한 값이 반응형 값이 아니라는 것, 즉,리렌더링의 결과로 변경될 수 없다는 것을 린터에 “증명”할 수 있음. 예를 들어, serverUrl과 roomId가 렌더링에 의존하지 않고 항상 같은 값을 갖는다면 컴포넌트 외부로 옮길 수 있음. 이제 의존성이 될 필요가 없음.

**또한 Effect 내부로 이동할 수도 있음. 렌더링 중에 계산되지 않으므로 반응하지 않음**

```js
function ChatRoom() {
  useEffect(() => {
    const serverUrl = "https://localhost:1234"; // serverUrl is not reactive
    const roomId = "general"; // roomId is not reactive
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ All dependencies declared
  // ...
}
```

Effect는 반응형 코드 블록. 내부에서 읽은 값이 변경되면 다시 동기화됨. 상호작용당 한 번만 실행되는 이벤트 핸들러와 달리 Effect는 동기화가 필요할 때마다 실행됨.

의존성을 “선택”할 수는 없음. 의존성에는 Effect에서 읽은 모든 반응형 값이 포함되어야 함. 린터가 이를 강제함. **때때로 이로 인해 무한 루프와 같은 문제가 발생하거나 Effect가 너무 자주 다시 동기화될 수 있음. 린터를 억제하여 이러한 문제를 해결하지 말 것. 대신 시도할 수 있는 방법은 다음과 같음.**

- **Effect가 독립적인 동기화 프로세스를 나타내는지 확인.** Effect가 아무것도 동기화하지 않는다면 불필요한 것일 수 있음. 여러 개의 독립적인 것을 동기화하는 경우 분할하라.

- ‘반응’하지 않고 Effect를 재동기화하지 않으면서 props나 state의 최신 값을 읽으려면, Effect를 반응하는 부분(Effect에 유지)과 반응하지 않는 부분(Effect Event라는 것으로 추출)으로 분리할 수 있음. 이벤트와 Effect를 분리하는 방법(다음 챕터)에 대해 읽어볼 것.

- 객체와 함수를 의존성으로 사용하지 마라. 렌더링 중에 오브젝트와 함수를 생성한 다음 Effect에서 읽으면 렌더링할 때마다 오브젝트와 함수가 달라짐. 그러면 매번 Effect를 다시 동기화해야 함. Effect에서 불필요한 의존성을 제거하는 방법에 대해 읽어볼 것. (다다음 챕터)

- 참고: 린터는 우리의 친구이지만 그 힘은 제한되어 있음.
  - **린터는 의존성이 잘못되었을 때만 알 수 있음. 각 사례를 해결하는 최선의 방법은 알지 못함.** 만약 린터가 의존성을 제안하지만 **이를 추가하면 루프가 발생한다고 해서 린터를 무시해야 한다는 의미는 아님. 해당 값이 반응적이지 않고 의존성이 될 필요가 없도록 Effect 내부(또는 외부)의 코드를 변경해야 한다는 뜻.**
  - 기존 코드베이스가 있는 경우 이와 같이 린터를 억제하는 Effect가 있을 수 있음. (하지 말 것을 권장.)
    ```js
    useEffect(() => {
      // ...
      // 🔴 Avoid suppressing the linter like this:
      // eslint-ignore-next-line react-hooks/exhaustive-deps
    }, []);
    ```
  - 다음 페이지들에서는 규칙을 위반하지 않고 이 코드를 수정하는 방법을 알아보자.
