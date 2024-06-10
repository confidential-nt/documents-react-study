# Effect 의존성 제거하기

## 들어가면서

Effect를 작성하면 린터는 Effect의 의존성 목록에 Effect가 읽는 모든 반응형 값(예: props 및 state)을 포함했는지 확인함. 이렇게 하면 Effect가 컴포넌트의 최신 props 및 state와 동기화 상태를 유지할 수 있음. 불필요한 의존성으로 인해 Effect가 너무 자주 실행되거나 무한 루프를 생성할 수도 있음. 이 가이드를 따라 Effect에서 불필요한 의존성을 검토하고 제거하라.

## 의존성은 코드와 일치해야 합니다

Effect를 작성할 때는 먼저 Effect가 수행하기를 원하는 작업을 시작하고 중지하는 방법을 지정함.

```js
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  	// ...
}
```

그런 다음 Effect 의존성을 비워두면([]) 린터가 올바른 의존성을 제안.

린터에 표시된 내용에 따라 채우기.

Effect는 반응형 값에 “반응”함. roomId는 반응형 값이므로(리렌더링으로 인해 변경될 수 있음), 린터는 이를 의존성으로 지정했는지 확인함. roomId가 다른 값을 받으면 React는 Effect를 다시 동기화. 이렇게 하면 채팅이 선택된 방에 연결된 상태를 유지하고 드롭다운에 ‘반응’함.

## 의존성을 제거하려면 의존성이 아님을 증명하세요

Effect의 의존성을 “선택”할 수 없다는 점에 유의. Effect의 코드에서 사용되는 모든 반응형 값은 의존성 목록에 선언되어야 함. 의존성 목록은 주변 코드에 의해 결정됨.

```js
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  // This is a reactive value
  // roomId는 반응형 값임
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // This Effect reads that reactive value
    // 이 Effect는 반응형값을 읽음
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ So you must specify that reactive value as a dependency of your Effect
  // ✅ 그러므로 해당 반응형 값을 Effect의 의존성 목록에 추가해야함
  // ...
}
```

반응형 값에는 props와 컴포넌트 내부에서 직접 선언된 모든 변수 및 함수가 포함됨. roomId는 반응형 값이므로 의존성 목록에서 제거할 수 없음. 린터가 허용하지 않음.

그리고 린터가 옳을 것. roomId는 시간이 지남에 따라 변경될 수 있으므로 코드에 버그가 발생할 수 있음. 의존성에 추가안하면.

의존성을 제거하려면 해당 컴포넌트가 의존성이 될 필요가 없다는 것을 린터에 “증명”하라. 예를 들어, roomId를 컴포넌트 밖으로 이동시키면 반응하지 않고 리렌더링 시에도 변경되지 않음을 증명할 수 있음.

이제 roomId는 반응형 값이 아니므로(리렌더링할 때 변경할 수 없으므로) 의존성이 될 필요가 없음.

이것이 빈([]) 의존성 목록을 지정할 수 있는 이유. Effect는 더 이상 반응형 값에 의존하지 않으므로 컴포넌트의 props나 state가 변경될 때 Effect를 다시 실행할 필요가 없음.

## 의존성을 변경하려면 코드를 변경하세요

작업흐름에서 패턴을 발견했을 수도 있음.

1. 먼저 Effect의 코드 또는 반응형 값 선언 방식을 변경함.
1. 그런 다음, 변경한 코드에 맞게 의존성을 조정함.
1. 의존성 목록이 마음에 들지 않으면 첫 번째 단계로 돌아감. (그리고 코드를 다시 변경함).

마지막 부분이 중요. 의존성을 변경하려면 먼저 주변 코드를 변경하라. 의존성 목록은 Effect의 코드에서 사용하는 모든 반응형 값의 목록이라고 생각하면 됨. 이 목록에 무엇을 넣을지는 사용자가 선택하지 않음. 이 목록은 코드를 설명함. 의존성 목록을 변경하려면 코드를 변경하라.

이것은 방정식을 푸는 것처럼 느껴질 수 있음. 예를 들어, 의존성 제거와 같은 목표를 설정하고 그 목표에 맞는 코드를 “찾아야” 함. 모든 사람이 방정식을 푸는 것을 재미있어하는 것은 아니며, Effect를 작성할 때도 마찬가지임. 다행히도 아래에 시도해 볼 수 있는 일반적인 레시피 목록이 있음.

- 참고: 기존 코드베이스가 있는 경우 이와 같이 린터를 억제하는 Effect가 있을 수 있음.

  ```js
  useEffect(() => {
    // ...
    // 🔴 Avoid suppressing the linter like this:
    // eslint-ignore-next-line react-hooks/exhaustive-deps
  }, []);
  ```

  - 의존성이 코드와 일치하지 않으면 버그가 발생할 위험이 매우 높음. 린터를 억제하면 Effect가 의존하는 값에 대해 React에 “거짓말”을 하게 됨.
  - 대신 다음에 소개할 기술을 사용.

- 참고: 의존성 린터를 억제하는 것이 왜 위험한가요?

  - 린터를 억제하면 매우 직관적이지 않은 버그가 발생하여 찾아서 수정하기가 어려움. 한 가지 예를 들어보자.

    ```js
    import { useState, useEffect } from 'react';

    export default function Timer() {
      const [count, setCount] = useState(0);
      const [increment, setIncrement] = useState(1);

      function onTick() {
        setCount(count + increment);
      }

      useEffect(() => {
        const id = setInterval(onTick, 1000);
        return () => clearInterval(id);
        // eslint-disable-next-line react-hooks/exhaustive-deps
      }, []);

      return (
        <>
          <h1>
            Counter: {count}
            <button onClick={() => setCount(0)}>Reset</button>
          </h1>
          <hr />
          <p>
            Every second, increment by:
            <button
              disabled={increment === 0}
              onClick={() => {
                setIncrement((i) => i - 1);
              }}
            >
              –
            </button>
            <b>{increment}</b>
            <button
              onClick={() => {
                setIncrement((i) => i + 1);
              }}
            >
              +
            </button>
          </p>
        </>
      );
    }
    ```

  - “마운트할 때만” Effect를 실행하고 싶다고 가정해 보자. 빈([]) 의존성이 그렇게 한다는 것을 읽었으므로 린터를 무시하고 [] 의존성을 강제로 지정하기로 결정.
  - 이 카운터는 두 개의 버튼으로 설정할 수 있는 양만큼 매초마다 증가해야 함. 하지만 이 Effect가 아무 것도 의존하지 않는다고 React에 “거짓말”을 했기 때문에, React는 초기 렌더링에서 계속 onTick 함수를 사용. 이 렌더링에서 count는 0이었고 increment는 1이었음. 그래서 이 렌더링의 onTick은 항상 매초마다 setCount(0 + 1)를 호출하고 항상 1이 표시됨. 이와 같은 버그는 여러 컴포넌트에 분산되어 있을 때 수정하기가 더 어려움.
  - 린터를 무시하는 것보다 더 좋은 해결책은 항상 있음. 이 코드를 수정하려면 의존성 목록에 onTick을 추가해야 함. (interval을 한 번만 설정하려면 onTick을 Effect Event로 만들어라.)
  - 의존성 린트 오류는 컴파일 오류로 처리하는 것이 좋다. (그냥 그대로 둬라. 억제하지 말고) 이를 억제하지 않으면 이와 같은 버그가 발생하지 않는다. 이 페이지의 나머지 부분에서는 이 경우와 다른 경우에 대한 대안을 설명합니다.

## 불필요한 의존성 제거하기

코드를 반영하기 위해 Effect의 의존성을 조정할 때마다 의존성 목록을 살펴보아라. 이러한 의존성 중 하나라도 변경되면 Effect가 다시 실행되는 것이 합리적일까? 가끔 대답은 “아니오”
임.

- 다른 조건에서 Effect의 다른 부분을 다시 실행하고 싶을 수도 있음.
- 일부 의존성의 변경에 “반응”하지 않고 “최신 값”만 읽고 싶을 수도 있음.
- 의존성은 객체나 함수이기 때문에 의도치 않게 너무 자주 변경될 수 있음.

올바른 해결책을 찾으려면 Effect에 대한 몇 가지 질문에 답해야 함. 몇 가지 질문을 살펴보자.

## 이 코드를 이벤트 핸들러로 옮겨야 하나요?

가장 먼저 고려해야 할 것은 이 코드가 Effect가 되어야 하는지 여부.

form을 상상해 보자. 제출할 때 submitted state 변수를 true로 설정함. POST 요청을 보내고 알림을 표시해야 함. 이 로직은 submitted가 true가 될 때 “반응”하는 Effect 안에 넣었음.

```js
function Form() {
  const [submitted, setSubmitted] = useState(false);

  useEffect(() => {
    if (submitted) {
      // 🔴 Avoid: Event-specific logic inside an Effect
      post('/api/register');
      showNotification('Successfully registered!');
    }
  }, [submitted]);

  function handleSubmit() {
    setSubmitted(true);
  }

  // ...
}
```

나중에 현재 테마에 따라 알림 메시지의 스타일을 지정하고 싶으므로 현재 테마를 읽음. theme는 컴포넌트 본문에서 선언 되었기때문에 이는 반응형 값이므로 의존성으로 추가함.

```js
function Form() {
  const [submitted, setSubmitted] = useState(false);
  const theme = useContext(ThemeContext);

  useEffect(() => {
    if (submitted) {
      // 🔴 Avoid: Event-specific logic inside an Effect
      post('/api/register');
      showNotification('Successfully registered!', theme);
    }
  }, [submitted, theme]); // ✅ All dependencies declared

  function handleSubmit() {
    setSubmitted(true);
  }

  // ...
}
```

이렇게 하면 버그가 발생하게 됨. 먼저 form을 제출한 다음 어두운 테마와 밝은 테마 간 전환한다고 가정해 보자. theme가 변경되고 Effect가 다시 실행되어 동일한 알림이 다시 표시됨.

여기서 문제는 이것이 애초에 Effect가 아니어야 한다는 점. 이 POST 요청을 보내고 특정 상호작용인 form 제출에 대한 응답으로 알림을 표시하고 싶다는 것. 특정 상호작용에 대한 응답으로 일부 코드를 실행하려면 해당 로직을 해당 이벤트 핸들러에 직접 넣어야 함.

```js
function Form() {
  const theme = useContext(ThemeContext);

  function handleSubmit() {
    // ✅ Good: Event-specific logic is called from event handlers
    post('/api/register');
    showNotification('Successfully registered!', theme);
  }

  // ...
}
```

이제 코드가 이벤트 핸들러에 있고, 이는 반응형 코드가 아니므로 사용자가 form을 제출할 때만 실행됨. 이벤트 핸들러와 Effect 중에서 선택하는 방법과 불필요한 Effect를 삭제하는 방법에 대해 자세히 알아보려면 지난 문서를 읽어봐라.

## Effect가 여러 관련 없는 일을 하고 있나요?

다음으로 스스로에게 물어봐야 할 질문은 Effect가 서로 관련이 없는 여러 가지 작업을 수행하고 있는지 여부.

사용자가 도시와 지역을 선택해야 하는 배송 form을 만든다고 가정. 선택한 country에 따라 서버에서 cities 목록을 페치해서 드롭다운에 표시함.

```js
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  const [city, setCity] = useState(null);

  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [country]); // ✅ All dependencies declared

  // ...
```

Effect에서 데이터를 페칭하는 좋은 예시. country props에 따라 cities state를 네트워크와 동기화하고 있음. ShippingForm이 표시되는 즉시 그리고 country가 변경될 때마다 (어떤 상호작용이 원인이든 상관없이) 데이터를 가져와야 하므로 이벤트 핸들러에서는 이 작업을 수행할 수 없음.

이제 도시 지역에 대한 두 번째 셀렉트박스를 추가하여 현재 선택된 city의 areas을 페치한다고 가정해 보자. 동일한 Effect 내에 지역 목록에 대한 두 번째 fetch 호출을 추가하는 것으로 시작할 수 있음.

```js
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);

  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    // 🔴 Avoid: A single Effect synchronizes two independent processes
    if (city) {
      fetch(`/api/areas?city=${city}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setAreas(json);
          }
        });
    }
    return () => {
      ignore = true;
    };
  }, [country, city]); // ✅ All dependencies declared

  // ...
```

하지만 이제 Effect가 city state 변수를 사용하므로 의존성 목록에 city를 추가해야 했음. 이로 인해 사용자가 다른 도시를 선택하면 Effect가 다시 실행되어 fetchCities(country)를 호출하는 문제가 발생함. 결과적으로 불필요하게 도시 목록을 여러 번 다시 페치하게 됨.

이 코드의 문제점은 서로 관련이 없는 두 가지를 동기화하고 있다는 것

1. country props를 기반으로 cities state를 네트워크에 동기화하려고 함
1. city state를 기반으로 areas state를 네트워크에 동기화하려고 함.

로직을 두 개의 Effect로 분할하고, 각 Effect는 동기화해야 하는 props에 반응하도록 수정해야.

```js
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [country]); // ✅ All dependencies declared

  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);
  useEffect(() => {
    if (city) {
      let ignore = false;
      fetch(`/api/areas?city=${city}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setAreas(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [city]); // ✅ All dependencies declared

  // ...
```

이제 첫 번째 Effect는 country가 변경될 때만 다시 실행되고, 두 번째 Effect는 city가 변경될 때 다시 실행됨. 목적에 따라 분리했으니, 서로 다른 두 가지가 두 개의 개별 Effect에 의해 동기화됨. 두 개의 개별 Effect에는 두 개의 개별 의존성 목록이 있으므로 의도치 않게 서로를 촉발하지 않음.

최종 코드는 원본보다 길지만 Effect를 분할하는 것이 여전히 정확함. 각 Effect는 독립적인 동기화 프로세스를 나타내야 함. 이 예제에서는 한 Effect를 삭제해도 다른 Effect의 로직이 깨지지 않음. 즉, 서로 다른 것을 동기화하므로 분할하는 것이 좋음. 중복이 걱정된다면 반복되는 로직을 커스텀 Hook으로 추출하여 이 코드를 개선할 수 있음.

## 다음 state를 계산하기 위해 어떤 state를 읽고 있나요?

이 Effect는 새 메시지가 도착할 때마다 새로 생성된 배열로 messages state 변수를 업데이트함.

```js
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages([...messages, receivedMessage]);
    });
    // ...
```

messages 변수를 사용하여 모든 기존 메시지로 시작하는 새 배열을 생성하고 마지막에 새 메시지를 추가함. 하지만 messages는 Effect에서 읽는 반응형 값이므로 의존성이어야 함.

```js
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages([...messages, receivedMessage]);
    });
    return () => connection.disconnect();
  }, [roomId, messages]); // ✅ All dependencies declared
  // ...
```

그리고 messages를 의존성으로 만들면 문제가 발생.

메시지를 수신할 때마다 setMessages()는 컴포넌트가 수신된 메시지를 포함하는 새 messages 배열로 리렌더링하도록 함. 하지만 이 Effect는 이제 messages에 따라 달라지므로 Effect도 다시 동기화됨. 따라서 새 메시지가 올 때마다 채팅이 다시 연결됨. 사용자가 원하지 않을 것.

이 문제를 해결하려면 Effect 내에서 messages를 읽지 마라. **대신 업데이터 함수를 setMessages에 전달.**

```js
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
```

이제 Effect가 messages 변수를 전혀 읽지 않는 것을 알 수 있음. msgs => [...msgs, receivedMessage]와 같은 업데이터 함수만 전달하면 됨. React는 업데이터 함수를 대기열에 넣고 다음 렌더링 중에 msgs 인수를 제공. 이 때문에 Effect 자체는 더 이상 messages에 의존할 필요가 없음. 이 수정으로 인해 채팅 메시지를 수신해도 더 이상 채팅이 다시 연결되지 않는다.

## 값의 변경에 ‘반응’하지 않고 값을 읽고 싶으신가요?

- 이 섹션에서는 아직 안정된 버전의 React로 출시되지 않은 실험적인 API에 대해 설명함.

사용자가 새 메시지를 수신할 때 isMuted가 true가 아닌 경우 사운드를 재생하고 싶다고 가정해 보자.

```js
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
      if (!isMuted) {
        playSound();
      }
    });
    // ...
```

이제 Effect의 코드에서 isMuted를 사용하므로 의존성에 추가해야함.

```js
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      setMessages(msgs => [...msgs, receivedMessage]);
      if (!isMuted) {
        playSound();
      }
    });
    return () => connection.disconnect();
  }, [roomId, isMuted]); // ✅ All dependencies declared
  // ...
```

문제는 (사용자가 “Muted” 토글을 누르는 등) isMuted가 변경될 때마다 Effect가 다시 동기화되고 채팅에 다시 연결된다는 점. 이는 바람직한 사용자 경험이 아님! (이 예에서는 린터를 비활성화해도 작동하지 않습니다. 그렇게 하면 isMuted가 이전 값으로 ‘고착’됨)

이 문제를 해결하려면 Effect에서 반응해서는 안 되는 로직을 추출해야 함. 이 Effect가 isMuted의 변경에 “반응”하지 않기를 원함. 이 비반응 로직을 Effect Event로 옮기면 됨.

```js
import { useState, useEffect, useEffectEvent } from 'react';

function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [isMuted, setIsMuted] = useState(false);

  const onMessage = useEffectEvent(receivedMessage => {
    setMessages(msgs => [...msgs, receivedMessage]);
    if (!isMuted) {
      playSound();
    }
  });

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
```

Effect Event를 사용하면 Effect를 반응형 부분(roomId와 같은 반응형 값과 그 변경에 “반응”해야 하는)과 비반응형 부분(onMessage가 isMuted를 읽는 것처럼 최신 값만 읽는)으로 나눌 수 있음. 이제 Effect Event 내에서 isMuted를 읽었으므로 Effect의 의존성이 될 필요가 없음. 그 결과, “Muted” 설정을 켜고 끌 때 채팅이 다시 연결되지 않아 원래 문제가 해결됨.

### props를 이벤트 핸들러로 감싸기

컴포넌트가 이벤트 핸들러를 props로 받을 때 비슷한 문제가 발생할 수 있음.

```js
function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onReceiveMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId, onReceiveMessage]); // ✅ All dependencies declared
  // ...
```

부모 컴포넌트가 렌더링할 때마다 다른 onReceiveMessage 함수를 전달한다고 가정해 보자.

onReceiveMessage는 의존성이므로 부모가 리렌더링할 때마다 Effect가 다시 동기화됨. 그러면 채팅에 다시 연결됨. 이 문제를 해결하려면 호출을 Effect Event로 감싸라.

```js
function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  const onMessage = useEffectEvent(receivedMessage => {
    onReceiveMessage(receivedMessage);
  });

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
```

Effect Event는 반응하지 않으므로 의존성으로 지정할 필요가 없음. 그 결과, 부모 컴포넌트가 리렌더링할 때마다 다른 함수를 전달하더라도 채팅이 더 이상 다시 연결되지 않음.

### 반응형 코드와 비반응형 코드 분리

이 예제에서는 roomId가 변경될 때마다 방문을 기록하려고 함. 모든 로그에 현재 notificationCount를 포함하고 싶지만 notificationCount 변경으로 로그 이벤트가 촉발하는 것은 원하지 않음.

해결책은 다시 비반응형 코드를 Effect Event로 분리하는 것.

```js
function Chat({ roomId, notificationCount }) {
  const onVisit = useEffectEvent((visitedRoomId) => {
    logVisit(visitedRoomId, notificationCount);
  });

  useEffect(() => {
    onVisit(roomId);
  }, [roomId]); // ✅ All dependencies declared
  // ...
}
```

로직이 roomId와 관련하여 반응하기를 원하므로 Effect 내부에서 roomId를 읽음. 그러나 notificationCount를 변경하여 추가 방문을 기록하는 것은 원하지 않으므로 Effect Event 내부에서 notificationCount를 읽음. Effect Event를 사용하여 Effect에서 최신 props과 state를 읽는 방법에 대해서는 지난 문서를 통해 자세히 알아볼 것.

## 일부 반응형 값이 의도치 않게 변경되나요?

Effect가 특정 값에 ‘반응’하기를 원하지만, 그 값이 원하는 것보다 더 자주 변경되어 사용자의 관점에서 실제 변경 사항을 반영하지 못할 수도 있음. 예를 들어, 컴포넌트 본문에 options 객체를 생성한 다음 Effect 내부에서 해당 객체를 읽는다고 가정해 보자.

```js
unction ChatRoom({ roomId }) {
  // ...
  const options = {
    serverUrl: serverUrl,
    roomId: roomId
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    // ...
```

이 객체는 컴포넌트 본문에서 선언되므로 반응형 값. Effect 내에서 이와 같은 반응형 값을 읽으면 의존성으로 선언함. 이렇게 하면 Effect가 변경 사항에 “반응”하게 됨.

```js
// ...
useEffect(() => {
  const connection = createConnection(options);
  connection.connect();
  return () => connection.disconnect();
}, [options]); // ✅ All dependencies declared
// ...
```

의존성으로 선언하는 것이 중요! 이렇게 하면 예를 들어, roomId가 변경되면 Effect가 새 options으로 채팅에 다시 연결됨. 하지만 위 코드에도 문제가 있음. 이를 확인하려면 아래 샌드박스의 input에 타이핑하고 콘솔에서 어떤 일이 발생하는지 살펴보라.

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // Temporarily disable the linter to demonstrate the problem
  // eslint-disable-next-line react-hooks/exhaustive-deps
  const options = {
    serverUrl: serverUrl,
    roomId: roomId,
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]);

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <input value={message} onChange={(e) => setMessage(e.target.value)} />
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  return (
    <>
      <label>
        Choose the chat room:{' '}
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

위의 샌드박스에서 input은 message state 변수만 업데이트함. 사용자 입장에서는 이것이 채팅 연결에 영향을 미치지 않아야 함. 하지만 message를 업데이트할 때마다 컴포넌트가 리렌더링됨. 컴포넌트가 리렌더링되면 그 안에 있는 코드가 처음부터 다시 실행됨.

ChatRoom 컴포넌트를 리렌더링할 때마다 새로운 options 객체가 처음부터 새로 생성됨. React는 options 객체가 마지막 렌더링 중에 생성된 options 객체와 다른 객체임을 인식함. 그렇기 때문에 (options에 따라 달라지는) Effect를 다시 동기화하고 사용자가 입력할 때 채팅이 다시 연결됨.

이 문제는 객체와 함수에만 영향을 줌. JavaScript에서는 새로 생성된 객체와 함수가 다른 모든 객체와 구별되는 것으로 간주됨. 그 안의 내용이 동일할 수 있다는 것은 중요하지 않음.

객체 및 함수 의존성으로 인해 Effect가 필요 이상으로 자주 재동기화될 수 있음.

그렇기 때문에 가능하면 객체와 함수를 Effect의 의존성으로 사용하지 않는 것이 좋음. 대신 컴포넌트 외부나 Effect 내부로 이동하거나 원시 값을 추출해 보라.

### 정적 객체와 함수를 컴포넌트 외부로 이동

객체가 props 및 state에 의존하지 않는 경우 해당 객체를 컴포넌트 외부로 이동할 수 있음.

```js
const options = {
  serverUrl: 'https://localhost:1234',
  roomId: 'music'
};

function ChatRoom() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ All dependencies declared
  // ...
```

이렇게 하면 린터가 반응하지 않는다는 것을 증명할 수 있음. 리렌더링의 결과로 변경될 수 없으므로 의존성이 될 필요가 없음. 이제 ChatRoom을 리렌더링해도 Effect가 다시 동기화되지 않음.

이는 함수에도 적용됨.

```js
function createOptions() {
  return {
    serverUrl: 'https://localhost:1234',
    roomId: 'music'
  };
}

function ChatRoom() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ All dependencies declared
  // ...

```

### Effect 내에서 동적 객체 및 함수 이동

객체가 roomId 프로퍼티처럼 리렌더링의 결과로 변경될 수 있는 반응형 값에 의존하는 경우, 컴포넌트 외부로 끌어낼 수 없음. 하지만 Effect의 코드 내부로 이동시킬 수는 있음.

```js
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
```

이제 options이 Effect 내부에서 선언되었으므로 더 이상 Effect의 의존성이 아님. 대신 Effect에서 사용하는 유일한 반응형 값은 roomId. roomId는 객체나 함수가 아니기 때문에 의도치 않게 달라지지 않을 것이라고 확신할 수 있음. JavaScript에서 숫자와 문자열은 그 내용에 따라 비교됨.

이 수정 덕분에 입력을 수정해도 더 이상 채팅이 다시 연결되지 않음. 그러나 예상대로 roomId 드롭다운을 변경하면 다시 연결됨.

이는 함수에서도 마찬가지.

```js
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    function createOptions() {
      return {
        serverUrl: serverUrl,
        roomId: roomId
      };
    }

    const options = createOptions();
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
```

Effect 내에서 로직을 그룹화하기 위해 자신만의 함수를 작성할 수 있음. Effect 내부에서 선언하는 한, 반응형 값이 아니므로 Effect의 의존성이 될 필요가 없음.

### 객체에서 원시 값 읽기

가끔 props에서 객체를 받을 수도 있음.

```js
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // ✅ All dependencies declared
  // ...
```

렌더링 중에 부모 컴포넌트가 객체를 생성한다는 점이 위험.

```js
<ChatRoom
  roomId={roomId}
  options={{
    serverUrl: serverUrl,
    roomId: roomId,
  }}
/>
```

이렇게 하면 부모 컴포넌트가 리렌더링할 때마다 Effect가 다시 연결됨. 이 문제를 해결하려면 Effect 외부의 객체에서 정보를 읽고 객체 및 함수 의존성을 피하라.

```js
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = options;
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ All dependencies declared
  // ...
```

로직은 약간 반복적임 (Effect 외부의 객체에서 일부 값을 읽은 다음 Effect 내부에 동일한 값을 가진 객체를 만드니까). 하지만 Effect가 실제로 어떤 정보에 의존하는지 매우 명확하게 알 수 있음. 부모 컴포넌트에 의해 의도치 않게 객체가 다시 생성된 경우 채팅이 다시 연결되지 않음. 하지만 options.roomId 또는 options.serverUrl이 실제로 다른 경우 채팅이 다시 연결됨.

### 함수에서 원시값 계산

함수에 대해서도 동일한 접근 방식을 사용할 수 있음. 예를 들어, 부모 컴포넌트가 함수를 전달한다고 가정해 보자.

```js
<ChatRoom
  roomId={roomId}
  getOptions={() => {
    return {
      serverUrl: serverUrl,
      roomId: roomId,
    };
  }}
/>
```

의존성을 만들지 않으려면 (그리고 리렌더링할 때 다시 연결되는 것을 방지하려면) Effect 외부에서 호출하라. 이렇게 하면 객체가 아니며 Effect 내부에서 읽을 수 있는 roomId 및 serverUrl 값을 얻을 수 있음.

```js
unction ChatRoom({ getOptions }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = getOptions();
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ All dependencies declared
  // ...
```

이는 렌더링 중에 호출해도 안전하므로(외부에 영향을 주지 않음) 순수 함수에서만 작동함. 함수가 이벤트 핸들러이지만 변경 사항으로 인해 Effect가 다시 동기화되는 것을 원하지 않는 경우, 대신 Effect Event로 함수를 감싸라.
