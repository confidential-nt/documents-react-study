# ✏ Managing State : Preserving and Resetting State

state는 컴포넌트 간에 격리(분리)된다.
React는 UI 트리에서 어떤 컴포넌트가 어떤 state에 속하는지를 추적한다.
따라서 state를 언제 보존하고 언제 초기화할지 제어할 수 있다.

### state는 트리의 한 위치에 묶인다

React는 UI 컴포넌트 구조에 대한 렌더 트리를 빌드한다.
컴포넌트에 state를 부여하면 그 컴포넌트 내부에 state가 존재한다고 생각할 수 있지만, 실제로는 React 내부에서 유지된다.
React는 렌더링 트리에서 해당 컴포넌트의 위치에 따라 보유한 각 state를 올바른 컴포넌트와 연결한다.

```
import { useState } from 'react';

export default function App() {
  const counter = <Counter />;
  return (
    <div>
      {counter}
      {counter}
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

위 코드의 경우 트리로 표시하면 div 태그 하위에 Counter 컴포넌트가 각각 따로 존재한다.
각 트리에서 고유한 위치에서 렌더링 되기 때문에 이 둘은 별개로 여겨진다.
이렇게 각 컴포넌트는 완전히 분리된 state를 갖게 된다.
위 코드에서는 독립적인 score와 hover state를 갖게 되어 어느 것을 클릭해도 서로에게 영향을 미치지 않는다.

_React는 트리의 같은 위치에 같은 컴포넌트를 렌더링하는 한 그 state를 유지한다._

```
import { useState } from 'react';

export default function App() {
  const [showB, setShowB] = useState(true);
  return (
    <div>
      <Counter />
      {showB && <Counter />}
      <label>
        <input
          type="checkbox"
          checked={showB}
          onChange={e => {
            setShowB(e.target.checked)
          }}
        />
        Render the second counter
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

위 예제를 보면 카운터를 모두 증가시킨 후 [Render the second counter]를 해제하면 두 번째 컴포넌트가 제거된다.
다시 선택해 추가하면 score가 0으로 초기화 되어 DOM에 추가된다.

즉 컴포넌트가 제거되거나 같은 위치에서 다른 컴포넌트가 렌더링 되는 경우 React는 해당 컴포넌트의 state를 삭제해버린다.

### 동일한 위치의 동일한 컴포넌트는 state를 유지한다.

```
import { useState } from 'react';

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <Counter isFancy={true} />
      ) : (
        <Counter isFancy={false} />
      )}
      <label>
        <input
          type="checkbox"
          checked={isFancy}
          onChange={e => {
            setIsFancy(e.target.checked)
          }}
        />
        Use fancy styling
      </label>
    </div>
  );
}

function Counter({ isFancy }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }
  if (isFancy) {
    className += ' fancy';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

[Use fancy styling]을 선택하고 해제해도 카운터 state는 재설정되지 않는다.
isFancy가 true / false 뭐든간에 div의 첫 번째 자식은 <Counter />이기 때문이다.
위에서 말했지만 위치가 바뀌거나 다른 컴포넌트가 렌더링 되면 state가 초기화 되는데, 반대로 뒤집어 _같은 위치의 같은 컴포넌트는 state가 유지된다._ React 관점에서는 isFancy가 뭐든간에 같은 위치에 있는 같은 컴포넌트라 동일하게 보는 것이다.

### 동일한 위치의 다른 컴포넌트는 state를 초기화 한다.

```
import { useState } from 'react';

export default function App() {
  const [isPaused, setIsPaused] = useState(false);
  return (
    <div>
      {isPaused ? (
        <p>See you later!</p>
      ) : (
        <Counter />
      )}
      <label>
        <input
          type="checkbox"
          checked={isPaused}
          onChange={e => {
            setIsPaused(e.target.checked)
          }}
        />
        Take a break
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

체크박스 클릭 시 <Counter />가 <p>로 바뀌어버린다.

```
{isPaused ? (
        <p>See you later!</p>
      ) : (
        <Counter />
      )}
```

같은 위치지만 isPaused의 값에 따라 <p>가 될 수도 <Counter />가 되어버릴 수도 있다.
이렇게 한쪽으로 설정되면 반대쪽은 제거되고 그 state는 소멸된다.

또 같은 위치에 다른 컴포넌트가 렌더링 되면 전체 하위 트리의 state가 재설정된다. (당연함)
리렌더링 사이에 state를 유지하려면, 트리의 구조가 일치해야 한다.
즉 트리의 구조가 다르면 React는 컴포넌트를 제거할 때 state를 파괴하기 때문이다.

그렇기에 컴포넌트 함수 정의를 중첩하면 안되는 것이다.

```
import { useState } from 'react';

export default function MyComponent() {
  const [counter, setCounter] = useState(0);

  function MyTextField() {
    const [text, setText] = useState('');

    return (
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
    );
  }

  return (
    <>
      <MyTextField />
      <button onClick={() => {
        setCounter(counter + 1)
      }}>Clicked {counter} times</button>
    </>
  );
}
```

버튼을 클릭할 때마다 입력 state가 사라져버리는데, 이는 MyComponent를 렌더링할 때 MyTextField 함수가 생성되기 때문이다.
같은 위치에서 다른 컴포넌트를 렌더링하기에 하위의 모든 state도 초기화 되어 버리고, 이 때문에 버그나 성능 문제가 발생하게 된다.
따라서 항상 컴포넌트 함수는 최상위 수준에서 선언하고 정의를 중첩하지 않아야 한다.

### 동일한 위치에서 state 재설정하기

컴포넌트의 state를 리셋하고 싶다면 어떻게 해야 할까?

```
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter person="Taylor" />
      ) : (
        <Counter person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{person}'s score: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

차례가 바뀔 때 점수가 초기화 되어야 하는데 점수를 나타내는 score가 보존된다.
차례를 바꿀 때 state를 재설정 하는 방법으로는 컴포넌트를 다른 위치에 렌더링하거나, 각 컴포넌트에 key를 부여하는 것이다.

```
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA &&
        <Counter person="Taylor" />
      }
      {!isPlayerA &&
        <Counter person="Sarah" />
      }
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{person}'s score: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

컴포넌트를 다른 위치에 렌더링하면 이렇게 isPlayer의 값에 따라 렌더링된다.
따라서 Scoreboard 아래 Counter 위치가 바뀌면서 리셋이 된다.
이런 방식은 같은 위치에 몇 개의 독립적인 컴포넌트만 렌더링 할 때 편리하다. (개수가 적을 때)

```
import { useState } from 'react';

export default function Scoreboard() {
  const [isPlayerA, setIsPlayerA] = useState(true);
  return (
    <div>
      {isPlayerA ? (
        <Counter key="Taylor" person="Taylor" />
      ) : (
        <Counter key="Sarah" person="Sarah" />
      )}
      <button onClick={() => {
        setIsPlayerA(!isPlayerA);
      }}>
        Next player!
      </button>
    </div>
  );
}

function Counter({ person }) {
  const [score, setScore] = useState(0);
  const [hover, setHover] = useState(false);

  let className = 'counter';
  if (hover) {
    className += ' hover';
  }

  return (
    <div
      className={className}
      onPointerEnter={() => setHover(true)}
      onPointerLeave={() => setHover(false)}
    >
      <h1>{person}'s score: {score}</h1>
      <button onClick={() => setScore(score + 1)}>
        Add one
      </button>
    </div>
  );
}
```

2번째 방법으로 key를 사용한다.
React는 key를 통해 컴포넌트들을 구분하므로 다르다고 인지할 수 있다.
key가 다르다 === 컴포넌트가 다르다 === state가 유지되지 않는다.

key는 전역적으로 고유하지 않으며 부모 내에서의 위치만 지정한다는 점을 기억하자.

### key로 form 재설정하기

key로 state를 재설정하는 것은 form을 다룰 때 특히 유용하다.

```
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat contact={to} />
    </div>
  )
}

const contacts = [
  { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
  { id: 1, name: 'Alice', email: 'alice@mail.com' },
  { id: 2, name: 'Bob', email: 'bob@mail.com' }
];
```

입력란에 뭔가 입력한 후 다른 수신자들을 선택해도 입력 state가 유지된다.
다른 수신자들을 클릭할 때는 입력 state가 리셋되도록 하고 싶다.

<Chat key={to.id} contact={to} />처럼 key를 전해주자.
다른 수신자를 선택하면 key가 바뀌므로 그 아래 트리의 모든 state를 포함해 처음부터 재생성한다.
React 역시 DOM을 다시 생성하게 된다.

---

참고
https://react-ko.dev/learn/preserving-and-resetting-state
