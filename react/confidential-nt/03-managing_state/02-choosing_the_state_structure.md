# Choosing the State Structure

## 들어가면서

state를 잘 구조화해야 나중에 버그가 없다. 구조화 잘 하는 팁을 알아보자.

## state 구조화 원칙

state를 사용할 때는 얼마나 많은 state 변수를 사용할지, 데이터의 모양은 어떻게 할지에 대해 선택해야.

좋은 구조 없이 그냥 사용해도 프로그램을 돌아갈 수 있게는 하지만 but...업보는 무섭다.

더 나은 선택을 할 수 있도록 안내하는 몇 가지 원칙들:

1. **관련 state를 그룹화하라.** 항상 두 개 이상의 state 변수를 동시에 업데이트하는 경우 단일 state 변수로 병합하는 것이 좋다.
2. **state의 모순을 피하라.** 여러 state 조각이 서로 모순되거나 ‘불일치’할 수 있는 방식으로 state를 구성하면 실수가 발생할 여지가 생김.
3. **불필요한 state를 피하라.** 렌더링 중에 컴포넌트의 props나 기존 state 변수에서 일부 정보를 계산할 수 있다면 해당 정보를 해당 컴포넌트의 state에 넣지 않아야.
4. **state 중복을 피하라.** 동일한 데이터가 여러 state 변수 간에 또는 중첩된 객체 내에 중복되면 동기화 state를 유지하기가 어렵다.
5. **깊게 중첩된 state는 피하라.** 깊게 계층화된 state는 업데이트하기 쉽지 않음. 가능하면 state를 평평한 방식으로 구성하는 것이 좋다.

이러한 원칙의 목표: **실수 없이 state를 쉽게 업데이트할 수 있도록 하는 것**

state에서 불필요하거나 중복된 데이터를 제거하면 모든 데이터가 동기화 상태를 유지하는 데 도움이 됨. 이는 데이터베이스 구조를 ‘정규화’하는 것과 유사.

> **“state를 최대한 단순하게 만들되, 그보다 더 단순해서는 안 됩니다.”**

이러한 원칙이 실제로 어떻게 적용되는지 살펴보자.

## 관련 state 그룹화하기

단일 state 변수를 사용할지 다중 state 변수를 사용할지 고민할 때가 있음.

둘 중 뭐가 나아보이니?

```javascript
// 1.
const [x, setX] = useState(0);
const [y, setY] = useState(0);

// 2.
const [position, setPosition] = useState({ x: 0, y: 0 });
```

기술적으로는 이 두 가지 접근 방식 중 하나를 사용할 수 있음. 하지만 두 개의 state 변수가 항상 함께 변경되는 경우에는 하나의 state 변수로 통합하는 것이 좋다.

그래야 커서를 움직일 때, 좌표가 모두 업데이트가 될테니까. 그래야 옳게 동기화 될테니까.

데이터를 객체나 배열로 그룹화하는 또 다른 경우는 필요한 state의 조각 수를 모를 때. 예를 들어, 사용자가 사용자 정의 필드를 추가할 수 있는 양식이 있을 때 유용.

## state의 모순을 피하세요

다음은 isSending 및 isSent state 변수가 있는 호텔 피드백 양식.

```javascript
import { useState } from "react";

export default function FeedbackForm() {
  const [text, setText] = useState("");
  const [isSending, setIsSending] = useState(false);
  const [isSent, setIsSent] = useState(false);

  async function handleSubmit(e) {
    e.preventDefault();
    setIsSending(true);
    await sendMessage(text);
    setIsSending(false);
    setIsSent(true);
  }

  if (isSent) {
    return <h1>Thanks for feedback!</h1>;
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>How was your stay at The Prancing Pony?</p>
      <textarea
        disabled={isSending}
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <br />
      <button disabled={isSending} type="submit">
        Send
      </button>
      {isSending && <p>Sending...</p>}
    </form>
  );
}

// Pretend to send a message.
function sendMessage(text) {
  return new Promise((resolve) => {
    setTimeout(resolve, 2000);
  });
}
```

이 코드는 작동하긴 하지만, “불가능한” state의 설정을 허용하고 있음.

예를 들어, setIsSent와 setIsSending 중 어느 하나만 호출하면, isSending과 isSent가 동시에 true가 되는 상황이 발생할 수 있음. -> 위의 코드 예시의 경우는 괜찮지만, 만약 실수로 setIsSent와 setIsSending 중 어느 하나만 호출하면 모순된 상황이 발생할 수 있다는 것.

이런 경우, 컴포넌트가 복잡할수록 무슨 일이 일어났는지 파악하기가 더 어려워짐.

**isSending과 isSent는 동시에 true가 되어서는 안되므로, 세 가지 유효한 state 중 하나를 취할 수 있는 status 라는 state변수 하나로 대체하는 것이 좋다.** (사전에 혼란 방지하자.)

status는 typing(초기), sending, sent가 될 수 있음.

```javascript
import { useState } from "react";

export default function FeedbackForm() {
  const [text, setText] = useState("");
  const [status, setStatus] = useState("typing");

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus("sending");
    await sendMessage(text);
    setStatus("sent");
  }

  const isSending = status === "sending";
  const isSent = status === "sent";

  if (isSent) {
    return <h1>Thanks for feedback!</h1>;
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>How was your stay at The Prancing Pony?</p>
      <textarea
        disabled={isSending}
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <br />
      <button disabled={isSending} type="submit">
        Send
      </button>
      {isSending && <p>Sending...</p>}
    </form>
  );
}

// Pretend to send a message.
function sendMessage(text) {
  return new Promise((resolve) => {
    setTimeout(resolve, 2000);
  });
}
```

가독성을 위해 일부 상수를 선언할 수 있음.

```javascript
const isSending = status === "sending";
const isSent = status === "sent";
```

## 불필요한 state를 피하세요

**렌더링 중에 컴포넌트의 props나 기존 state 변수에서 일부 정보를 계산할 수 있는 경우 해당 정보를 컴포넌트의 state에 넣지 않아야.**

예를 들어, 이 양식을 살펴보자. 작동하긴 하지만, 불필요한 state가 있진 않은가?

```javascript
import { useState } from "react";

export default function Form() {
  const [firstName, setFirstName] = useState("");
  const [lastName, setLastName] = useState("");
  const [fullName, setFullName] = useState("");

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    setFullName(e.target.value + " " + lastName);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
    setFullName(firstName + " " + e.target.value);
  }

  return (
    <>
      <h2>Let’s check you in</h2>
      <label>
        First name: <input value={firstName} onChange={handleFirstNameChange} />
      </label>
      <label>
        Last name: <input value={lastName} onChange={handleLastNameChange} />
      </label>
      <p>
        Your ticket will be issued to: <b>{fullName}</b>
      </p>
    </>
  );
}
```

fullName은 불필요함. **렌더링 중에 언제든지 firstName과 lastName에서 fullName을 계산할 수 있으므로** state에서 제거해야.

이렇게 수정하라.

```javascript
import { useState } from "react";

export default function Form() {
  const [firstName, setFirstName] = useState("");
  const [lastName, setLastName] = useState("");

  const fullName = firstName + " " + lastName;

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
  }

  return (
    <>
      <h2>Let’s check you in</h2>
      <label>
        First name: <input value={firstName} onChange={handleFirstNameChange} />
      </label>
      <label>
        Last name: <input value={lastName} onChange={handleLastNameChange} />
      </label>
      <p>
        Your ticket will be issued to: <b>{fullName}</b>
      </p>
    </>
  );
}
```

여기서 fullName 은 state 변수가 아님. 대신 렌더링 중에 계산되는 것.

따라서 변경 핸들러는 이를 업데이트하기 위해 특별한 작업을 수행할 필요가 없음. setFirstName 또는 setLastName을 호출하면 다시 렌더링을 촉발하고 다음 fullName이 새 데이터에서 계산됨.

참고: props를 state에 그대로 미러링하지 마라.

- 다음 코드는 중복 state의 일반적인 예시:
  ```javascript
  function Message({ messageColor }) {
  const [color, setColor] = useState(messageColor);
  ```
- 여기서 color state 변수는 messageColor props로 초기화됨. **문제는 부모 컴포넌트가 나중에 다른 messageColor 값(예: blue 대신 red)을 전달하면 color state 변수가 업데이트되지 않는다는 것!** **state는 첫 번째 렌더링 중에만 초기화됨**
- 그렇기 때문에 state 변수에 일부 prop을 ‘미러링’하면 혼란을 초래할 수 있음. 대신 코드에서 messageColor prop을 직접 사용하라.
  ```javascript
  function Message({ messageColor }) {
  const color = messageColor;
  ```
- **props를 state로 ‘미러링’하는 것은 특정 prop에 대한 모든 업데이트를 무시하려는 경우에만 의미가 있음.** 관례에 따라 prop 이름을 initial 또는 default로 시작하여 새 값이 무시됨을 명확히 하라:
  ```javascript
  function Message({ initialColor }) {
  // The `color` state variable holds the *first* value of `initialColor`.
  // Further changes to the `initialColor` prop are ignored.
  const [color, setColor] = useState(initialColor);
  ```

## state 중복을 피하세요

이 메뉴 목록 컴포넌트를 사용하면 여러 가지 여행용 간식 중 하나를 선택할 수 있음

```javascript
import { useState } from "react";

const initialItems = [
  { title: "pretzels", id: 0 },
  { title: "crispy seaweed", id: 1 },
  { title: "granola bar", id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(items[0]);

  return (
    <>
      <h2>What's your travel snack?</h2>
      <ul>
        {items.map((item) => (
          <li key={item.id}>
            {item.title}{" "}
            <button
              onClick={() => {
                setSelectedItem(item);
              }}
            >
              Choose
            </button>
          </li>
        ))}
      </ul>
      <p>You picked {selectedItem.title}.</p>
    </>
  );
}
```

현재 선택된 항목은 selectedItem state 변수에 객체로 저장됨. 그러나 이것은 좋지 않음. **selectedItem의 내용은 items 목록 내의 항목 중 하나와 동일한 객체.** 즉, 항목 자체에 대한 정보가 두 곳에 중복됨.

이것이 왜 문제가 될까? 각 항목을 편집 가능하게 만들어 보겠음:

```javascript
import { useState } from "react";

const initialItems = [
  { title: "pretzels", id: 0 },
  { title: "crispy seaweed", id: 1 },
  { title: "granola bar", id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(items[0]);

  function handleItemChange(id, e) {
    setItems(
      items.map((item) => {
        if (item.id === id) {
          return {
            ...item,
            title: e.target.value,
          };
        } else {
          return item;
        }
      })
    );
  }

  return (
    <>
      <h2>What's your travel snack?</h2>
      <ul>
        {items.map((item, index) => (
          <li key={item.id}>
            <input
              value={item.title}
              onChange={(e) => {
                handleItemChange(item.id, e);
              }}
            />{" "}
            <button
              onClick={() => {
                setSelectedItem(item);
              }}
            >
              Choose
            </button>
          </li>
        ))}
      </ul>
      <p>You picked {selectedItem.title}.</p>
    </>
  );
}
```

먼저 항목에서 “Choose”을 클릭한 다음 편집하면 입력은 업데이트되지만 하단의 레이블에 편집 내용이 반영되지 않는 것을 확인할 수 있음. 이는 state가 중복되어 있고 selectedItem을 업데이트하는 것을 잊었기 때문.

selectedItem도 업데이트할 수 있지만 중복을 제거하는 것이 더 쉬운 수정 방법. 이 예제에서는 selectedItem 객체(items 내부의 객체 복사본을 생성하는) 대신 selectedId를 state로 유지한 다음 items 배열에서 해당 ID를 가진 항목을 검색하여 selectedItem을 가져옴.

```javascript
import { useState } from "react";

const initialItems = [
  { title: "pretzels", id: 0 },
  { title: "crispy seaweed", id: 1 },
  { title: "granola bar", id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedId, setSelectedId] = useState(0);

  const selectedItem = items.find((item) => item.id === selectedId);

  function handleItemChange(id, e) {
    setItems(
      items.map((item) => {
        if (item.id === id) {
          return {
            ...item,
            title: e.target.value,
          };
        } else {
          return item;
        }
      })
    );
  }

  return (
    <>
      <h2>What's your travel snack?</h2>
      <ul>
        {items.map((item, index) => (
          <li key={item.id}>
            <input
              value={item.title}
              onChange={(e) => {
                handleItemChange(item.id, e);
              }}
            />{" "}
            <button
              onClick={() => {
                setSelectedId(item.id);
              }}
            >
              Choose
            </button>
          </li>
        ))}
      </ul>
      <p>You picked {selectedItem.title}.</p>
    </>
  );
}
```

이렇게 하니까 중복은 사라지고 필수 state만 유지됨!

이제 선택한 항목을 편집하면 아래 메시지가 즉시 업데이트됨. 이는 setItems가 리렌더링을 촉발하고 items.find(...)가 업데이트된 제목의 항목을 찾기 때문. 선택한 ID만 필수적이므로 선택한 항목을 state로 유지할 필요가 없음. 나머지는 렌더링 중에 계산할 수 있음.

## 깊게 중첩된 state는 피하세요

행성, 대륙, 국가로 구성된 여행 계획을 상상해 보라. 이 예제에서처럼 중첩 객체와 배열을 사용하여 state를 구조화하고 싶을 수 있음:

```javascript
export const initialTravelPlan = {
  id: 0,
  title: "(Root)",
  childPlaces: [
    {
      id: 1,
      title: "Earth",
      childPlaces: [
        {
          id: 2,
          title: "Africa",
          childPlaces: [
            {
              id: 3,
              title: "Botswana",
              childPlaces: [],
            },
            {
              id: 4,
              title: "Egypt",
              childPlaces: [],
            },
            {
              id: 5,
              title: "Kenya",
              childPlaces: [],
            },
            {
              id: 6,
              title: "Madagascar",
              childPlaces: [],
            },
            {
              id: 7,
              title: "Morocco",
              childPlaces: [],
            },
            {
              id: 8,
              title: "Nigeria",
              childPlaces: [],
            },
            {
              id: 9,
              title: "South Africa",
              childPlaces: [],
            },
          ],
        },
        {
          id: 10,
          title: "Americas",
          childPlaces: [
            {
              id: 11,
              title: "Argentina",
              childPlaces: [],
            },
            {
              id: 12,
              title: "Brazil",
              childPlaces: [],
            },
            {
              id: 13,
              title: "Barbados",
              childPlaces: [],
            },
            {
              id: 14,
              title: "Canada",
              childPlaces: [],
            },
            {
              id: 15,
              title: "Jamaica",
              childPlaces: [],
            },
            {
              id: 16,
              title: "Mexico",
              childPlaces: [],
            },
            {
              id: 17,
              title: "Trinidad and Tobago",
              childPlaces: [],
            },
            {
              id: 18,
              title: "Venezuela",
              childPlaces: [],
            },
          ],
        },
        {
          id: 19,
          title: "Asia",
          childPlaces: [
            {
              id: 20,
              title: "China",
              childPlaces: [],
            },
            {
              id: 21,
              title: "India",
              childPlaces: [],
            },
            {
              id: 22,
              title: "Singapore",
              childPlaces: [],
            },
            {
              id: 23,
              title: "South Korea",
              childPlaces: [],
            },
            {
              id: 24,
              title: "Thailand",
              childPlaces: [],
            },
            {
              id: 25,
              title: "Vietnam",
              childPlaces: [],
            },
          ],
        },
        {
          id: 26,
          title: "Europe",
          childPlaces: [
            {
              id: 27,
              title: "Croatia",
              childPlaces: [],
            },
            {
              id: 28,
              title: "France",
              childPlaces: [],
            },
            {
              id: 29,
              title: "Germany",
              childPlaces: [],
            },
            {
              id: 30,
              title: "Italy",
              childPlaces: [],
            },
            {
              id: 31,
              title: "Portugal",
              childPlaces: [],
            },
            {
              id: 32,
              title: "Spain",
              childPlaces: [],
            },
            {
              id: 33,
              title: "Turkey",
              childPlaces: [],
            },
          ],
        },
        {
          id: 34,
          title: "Oceania",
          childPlaces: [
            {
              id: 35,
              title: "Australia",
              childPlaces: [],
            },
            {
              id: 36,
              title: "Bora Bora (French Polynesia)",
              childPlaces: [],
            },
            {
              id: 37,
              title: "Easter Island (Chile)",
              childPlaces: [],
            },
            {
              id: 38,
              title: "Fiji",
              childPlaces: [],
            },
            {
              id: 39,
              title: "Hawaii (the USA)",
              childPlaces: [],
            },
            {
              id: 40,
              title: "New Zealand",
              childPlaces: [],
            },
            {
              id: 41,
              title: "Vanuatu",
              childPlaces: [],
            },
          ],
        },
      ],
    },
    {
      id: 42,
      title: "Moon",
      childPlaces: [
        {
          id: 43,
          title: "Rheita",
          childPlaces: [],
        },
        {
          id: 44,
          title: "Piccolomini",
          childPlaces: [],
        },
        {
          id: 45,
          title: "Tycho",
          childPlaces: [],
        },
      ],
    },
    {
      id: 46,
      title: "Mars",
      childPlaces: [
        {
          id: 47,
          title: "Corn Town",
          childPlaces: [],
        },
        {
          id: 48,
          title: "Green Hill",
          childPlaces: [],
        },
      ],
    },
  ],
};
```

이제 이미 방문한 장소를 삭제하는 버튼을 추가하고 싶다고 가정해 보자. 어떻게 해야 할까? 중첩된 state를 업데이트하려면 변경된 부분부터 위쪽까지 객체의 복사본을 만들어야. 깊게 중첩된 장소를 삭제하려면 해당 장소의 상위 장소 체인 전체를 복사해야. 이러한 코드는 매우 장황할 수 있음.

**state가 너무 깊게 중첩되어 업데이트하기 어려운 경우 “flat”하게 만드는 것을 고려해 보자.**

다음은 이 데이터를 재구성할 수 있는 한 가지 방법. 각 place가 하위 place의 배열을 갖는 트리 구조 대신, 각 place가 자식 place ID의 배열을 보유하도록 할 수 있음. 그런 다음 각 place ID에서 해당 place로의 매핑을 저장.

이 데이터 재구성은 데이터베이스 테이블을 떠올리게 할 수 있음.:

```javascript
export const initialTravelPlan = {
  0: {
    id: 0,
    title: "(Root)",
    childIds: [1, 42, 46],
  },
  1: {
    id: 1,
    title: "Earth",
    childIds: [2, 10, 19, 26, 34],
  },
  2: {
    id: 2,
    title: "Africa",
    childIds: [3, 4, 5, 6, 7, 8, 9],
  },
  3: {
    id: 3,
    title: "Botswana",
    childIds: [],
  },
  4: {
    id: 4,
    title: "Egypt",
    childIds: [],
  },
  5: {
    id: 5,
    title: "Kenya",
    childIds: [],
  },
  6: {
    id: 6,
    title: "Madagascar",
    childIds: [],
  },
  7: {
    id: 7,
    title: "Morocco",
    childIds: [],
  },
  8: {
    id: 8,
    title: "Nigeria",
    childIds: [],
  },
  9: {
    id: 9,
    title: "South Africa",
    childIds: [],
  },
  10: {
    id: 10,
    title: "Americas",
    childIds: [11, 12, 13, 14, 15, 16, 17, 18],
  },
  11: {
    id: 11,
    title: "Argentina",
    childIds: [],
  },
  12: {
    id: 12,
    title: "Brazil",
    childIds: [],
  },
  13: {
    id: 13,
    title: "Barbados",
    childIds: [],
  },
  14: {
    id: 14,
    title: "Canada",
    childIds: [],
  },
  15: {
    id: 15,
    title: "Jamaica",
    childIds: [],
  },
  16: {
    id: 16,
    title: "Mexico",
    childIds: [],
  },
  17: {
    id: 17,
    title: "Trinidad and Tobago",
    childIds: [],
  },
  18: {
    id: 18,
    title: "Venezuela",
    childIds: [],
  },
  19: {
    id: 19,
    title: "Asia",
    childIds: [20, 21, 22, 23, 24, 25],
  },
  20: {
    id: 20,
    title: "China",
    childIds: [],
  },
  21: {
    id: 21,
    title: "India",
    childIds: [],
  },
  22: {
    id: 22,
    title: "Singapore",
    childIds: [],
  },
  23: {
    id: 23,
    title: "South Korea",
    childIds: [],
  },
  24: {
    id: 24,
    title: "Thailand",
    childIds: [],
  },
  25: {
    id: 25,
    title: "Vietnam",
    childIds: [],
  },
  26: {
    id: 26,
    title: "Europe",
    childIds: [27, 28, 29, 30, 31, 32, 33],
  },
  27: {
    id: 27,
    title: "Croatia",
    childIds: [],
  },
  28: {
    id: 28,
    title: "France",
    childIds: [],
  },
  29: {
    id: 29,
    title: "Germany",
    childIds: [],
  },
  30: {
    id: 30,
    title: "Italy",
    childIds: [],
  },
  31: {
    id: 31,
    title: "Portugal",
    childIds: [],
  },
  32: {
    id: 32,
    title: "Spain",
    childIds: [],
  },
  33: {
    id: 33,
    title: "Turkey",
    childIds: [],
  },
  34: {
    id: 34,
    title: "Oceania",
    childIds: [35, 36, 37, 38, 39, 40, 41],
  },
  35: {
    id: 35,
    title: "Australia",
    childIds: [],
  },
  36: {
    id: 36,
    title: "Bora Bora (French Polynesia)",
    childIds: [],
  },
  37: {
    id: 37,
    title: "Easter Island (Chile)",
    childIds: [],
  },
  38: {
    id: 38,
    title: "Fiji",
    childIds: [],
  },
  39: {
    id: 40,
    title: "Hawaii (the USA)",
    childIds: [],
  },
  40: {
    id: 40,
    title: "New Zealand",
    childIds: [],
  },
  41: {
    id: 41,
    title: "Vanuatu",
    childIds: [],
  },
  42: {
    id: 42,
    title: "Moon",
    childIds: [43, 44, 45],
  },
  43: {
    id: 43,
    title: "Rheita",
    childIds: [],
  },
  44: {
    id: 44,
    title: "Piccolomini",
    childIds: [],
  },
  45: {
    id: 45,
    title: "Tycho",
    childIds: [],
  },
  46: {
    id: 46,
    title: "Mars",
    childIds: [47, 48],
  },
  47: {
    id: 47,
    title: "Corn Town",
    childIds: [],
  },
  48: {
    id: 48,
    title: "Green Hill",
    childIds: [],
  },
};
```

이제 state가 “flat”(“정규화”라고도 함)해졌으므로 중첩된 항목을 업데이트하는 것이 더 쉬워졌음.

지금 장소를 제거하려면 두 단계의 state만 업데이트하면 됨:

- 부모 장소의 업데이트된 버전은 제거된 ID를 childIds 배열에서 제외해야 합니다.
- 루트 ‘테이블’ 객체의 업데이트된 버전에는 상위 위치의 업데이트된 버전이 포함되어야 합니다.

다음은 이를 수행하는 방법의 예:

```javascript
import { useState } from "react";
import { initialTravelPlan } from "./places.js";

export default function TravelPlan() {
  const [plan, setPlan] = useState(initialTravelPlan);

  // ** 주목 **  위에서 말하는 내용이 이 내용.//
  function handleComplete(parentId, childId) {
    const parent = plan[parentId];
    // Create a new version of the parent place
    // that doesn't include this child ID.
    const nextParent = {
      ...parent,
      childIds: parent.childIds.filter((id) => id !== childId),
    };
    // Update the root state object...
    setPlan({
      ...plan,
      // ...so that it has the updated parent.
      [parentId]: nextParent,
    });
  }

  const root = plan[0];
  const planetIds = root.childIds;
  return (
    <>
      <h2>Places to visit</h2>
      <ol>
        {planetIds.map((id) => (
          <PlaceTree
            key={id}
            id={id}
            parentId={0}
            placesById={plan}
            onComplete={handleComplete}
          />
        ))}
      </ol>
    </>
  );
}

function PlaceTree({ id, parentId, placesById, onComplete }) {
  const place = placesById[id];
  const childIds = place.childIds;
  return (
    <li>
      {place.title}
      <button
        onClick={() => {
          onComplete(parentId, id);
        }}
      >
        Complete
      </button>
      {childIds.length > 0 && (
        <ol>
          {childIds.map((childId) => (
            <PlaceTree
              key={childId}
              id={childId}
              parentId={id}
              placesById={placesById}
              onComplete={onComplete}
            />
          ))}
        </ol>
      )}
    </li>
  );
}
```

state를 원하는 만큼 중첩할 수 있지만 “flat”하게 만들면 많은 문제를 해결할 수 있음. state를 더 쉽게 업데이트할 수 있고 중첩된 객체의 다른 부분에 중복이 생기지 않도록 할 수 있음.

참고 : 메모리 사용량 개선하기

- 이상적으로는 ‘테이블’ 객체에서 삭제된 항목(및 그 하위 항목!)도 제거하여 메모리 사용량을 개선하는 것이 좋음.
- 이 버전은 그렇게 함. 또한 Immer를 사용하여 업데이트 로직을 더욱 간결하게 만듦.

  ```javascript
  import { useImmer } from "use-immer";
  import { initialTravelPlan } from "./places.js";

  export default function TravelPlan() {
    const [plan, updatePlan] = useImmer(initialTravelPlan);

    function handleComplete(parentId, childId) {
      updatePlan((draft) => {
        // Remove from the parent place's child IDs.
        const parent = draft[parentId];
        parent.childIds = parent.childIds.filter((id) => id !== childId);

        // Forget this place and all its subtree.
        deleteAllChildren(childId);
        function deleteAllChildren(id) {
          const place = draft[id];
          place.childIds.forEach(deleteAllChildren);
          delete draft[id];
        }
      });
    }

    const root = plan[0];
    const planetIds = root.childIds;
    return (
      <>
        <h2>Places to visit</h2>
        <ol>
          {planetIds.map((id) => (
            <PlaceTree
              key={id}
              id={id}
              parentId={0}
              placesById={plan}
              onComplete={handleComplete}
            />
          ))}
        </ol>
      </>
    );
  }

  function PlaceTree({ id, parentId, placesById, onComplete }) {
    const place = placesById[id];
    const childIds = place.childIds;
    return (
      <li>
        {place.title}
        <button
          onClick={() => {
            onComplete(parentId, id);
          }}
        >
          Complete
        </button>
        {childIds.length > 0 && (
          <ol>
            {childIds.map((childId) => (
              <PlaceTree
                key={childId}
                id={childId}
                parentId={id}
                placesById={placesById}
                onComplete={onComplete}
              />
            ))}
          </ol>
        )}
      </li>
    );
  }
  ```

때로는 중첩된 state의 일부를 하위 컴포넌트로 이동하여 state 중첩을 줄일 수도 있음. 이 방법은 항목에 마우스 커서가 있는지 여부와 같이 저장할 필요가 없는 임시 UI state에 적합.
