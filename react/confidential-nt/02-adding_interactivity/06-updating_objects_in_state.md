# Updating Objects in State

## 들어가면서

state는 객체를 포함해서, 어떤 종류의 JavaScript 값이든 저장할 수 있음.
하지만 React state에 있는 객체를 직접 변경해서는 안됨. 대신 객체를 업데이트하려면 새 객체를 생성하고(혹은 기존 객체의 복사본을 만들고), 해당 복사본을 사용하도록 state를 설정해야함.

## 변이란 무엇인가요?

객체 state를 살펴보자:

```javascript
const [position, setPosition] = useState({ x: 0, y: 0 });
```

기술적으로 객체 자체의 내용을 변경하는 것은 가능. 이를 **변이(mutation)** 라고 함.

React state의 객체는 기술적으로는 변이할 수 있지만, 숫자, 불리언(boolean), 문자열과 같이 불변하는 것처럼 취급해야 함.

객체를 직접 변이하는 대신, 항상 교체해야 함.

## state를 읽기 전용으로 취급하세요

다시 말해 state에 넣는 모든 JavaScript 객체를 읽기 전용으로 취급해야

state 변이는 경우에 따라 작동할 수 있지만 권장하지 않는다.

```javascript
onPointerMove={e => {
  position.x = e.clientX;
  position.y = e.clientY;
}}
```

위 코드에서는 포지션 객체를 수정하고 있지만, state 설정자 함수를 사용하지 않으면 React는 객체가 변이되었다는 사실을 알지 못함. 그래서 React는 아무 반응도 하지 않음.

실제로 리렌더링을 촉발하려면 새 객체를 생성하고 state 설정자 함수에 전달하라. 아래 처럼.

```javascript
onPointerMove={e => {
  setPosition({
    x: e.clientX,
    y: e.clientY
  });
}}
```

참고: Local mutation is fine (지역 변이는 괜찮습니다)

- 이와 같은 코드는 기존 객체의 state를 수정하기 때문에 문제가 됨.

  ```javascript
  position.x = e.clientX;
  position.y = e.clientY;
  ```

- 그러나 이와 같은 코드는 방금 생성한 새로운 객체를 변이하는 것이기 때문에 완전히 괜찮

  ```javascript
  const nextPosition = {};
  nextPosition.x = e.clientX;
  nextPosition.y = e.clientY;
  setPosition(nextPosition);
  ```

- 사실 이렇게 작성하는 것과 완전히 동일

  ```javascript
  setPosition({
    x: e.clientX,
    y: e.clientY,
  });
  ```

  변이는 이미 state가 있는 기존 객체를 변경할 때만 문제가 됨. 방금 생성한 객체를 변경해도 다른 코드가 아직 참조하지 않으므로 괜찮. 객체를 변경해도 해당 객체에 의존하는 다른 객체에 실수로 영향을 미치지 않음. 이를 “지역 변이(local mutation)“라고 함. 렌더링하는 동안에도 지역 변이를 수행할 수 있음. 아래 코드 기억나심?

  ```javascript
  function Cup({ guest }) {
    return <h2>Tea cup for guest #{guest}</h2>;
  }

  export default function TeaGathering() {
    let cups = [];
    for (let i = 1; i <= 12; i++) {
      cups.push(<Cup key={i} guest={i} />);
    }
    return cups;
  }
  ```

## 전개 구문을 사용하여 객체 복사하기

종종 기존 데이터를 새로 만드는 객체의 일부로 포함시키고 싶을 때가 있음. 예를 들어, form에 있는 하나의 필드만 업데이트하고 다른 모든 필드는 이전 값을 유지하고 싶을때라든지..

이럴 때 ... 객체 전개 구문 써라.

```javascript
setPerson({
  ...person, // Copy the old fields
  // 이전 필드를 복사합니다.
  firstName: e.target.value, // But override this one
  // 단, first name만 덮어씌웁니다.
});
```

각 입력 필드에 대해 별도의 state 변수를 선언하지 않을 수 있음. 큰 양식의 경우 올바르게 업데이트하기만 하면 모든 데이터를 객체에 그룹화하여 보관하는 것이 매우 편리

... 전개 구문은 “얕은” 구문으로, 한 단계 깊이만 복사한다는 점에 유의. 속도는 빠르지만 중첩된 프로퍼티를 업데이트하려면 두 번 이상 사용해야 한다는 뜻이기도 함.

```javascript
import { useState } from "react";

export default function Form() {
  const [person, setPerson] = useState({
    firstName: "Barbara",
    lastName: "Hepworth",
    email: "bhepworth@sculpture.com",
  });

  function handleFirstNameChange(e) {
    setPerson({
      ...person,
      firstName: e.target.value,
    });
  }

  function handleLastNameChange(e) {
    setPerson({
      ...person,
      lastName: e.target.value,
    });
  }

  function handleEmailChange(e) {
    setPerson({
      ...person,
      email: e.target.value,
    });
  }

  return (
    <>
      <label>
        First name:
        <input value={person.firstName} onChange={handleFirstNameChange} />
      </label>
      <label>
        Last name:
        <input value={person.lastName} onChange={handleLastNameChange} />
      </label>
      <label>
        Email:
        <input value={person.email} onChange={handleEmailChange} />
      </label>
      <p>
        {person.firstName} {person.lastName} ({person.email})
      </p>
    </>
  );
}
```

참고: 여러 필드에 단일 이벤트 핸들러 사용하기

- 객체 내에서 [ 및 ] 중괄호를 사용하여 동적 이름을 가진 프로퍼티를 지정할 수도 있음.
- 다음은 동일한 예시이지만 세 개의 다른 이벤트 핸들러 대신 단일 이벤트 핸들러를 사용한 예시.

  ```javascript
  import { useState } from "react";

  export default function Form() {
    const [person, setPerson] = useState({
      firstName: "Barbara",
      lastName: "Hepworth",
      email: "bhepworth@sculpture.com",
    });

    function handleChange(e) {
      setPerson({
        ...person,
        [e.target.name]: e.target.value,
      });
    }

    return (
      <>
        <label>
          First name:
          <input
            name="firstName"
            value={person.firstName}
            onChange={handleChange}
          />
        </label>
        <label>
          Last name:
          <input
            name="lastName"
            value={person.lastName}
            onChange={handleChange}
          />
        </label>
        <label>
          Email:
          <input name="email" value={person.email} onChange={handleChange} />
        </label>
        <p>
          {person.firstName} {person.lastName} ({person.email})
        </p>
      </>
    );
  }
  ```

## 중첩된 객체 업데이트하기

다음과 같은 중첩된 객체 구조를 생각해 봐라.

```javascript
const [person, setPerson] = useState({
  name: "Niki de Saint Phalle",
  artwork: {
    title: "Blue Nana",
    city: "Hamburg",
    image: "https://i.imgur.com/Sd1AgUOm.jpg",
  },
});
```

person.artwork.city를 업데이트하려면 변이를 사용하여 업데이트하는 방법이 명확 -> 그러나 동작 안함.

city를 변경하려면 먼저 새 artwork 객체(이전 artwork의 데이터로 미리 채워진)를 생성한 다음 새 artwork을 가리키는 새로운 사람 객체를 생성해야 함.

```javascript
const nextArtwork = { ...person.artwork, city: "New Delhi" };
const nextPerson = { ...person, artwork: nextArtwork };
setPerson(nextPerson);
```

또는 단일 함수 호출로 작성할 수도 있음

```javascript
setPerson({
  ...person, // Copy other fields
  artwork: { // but replace the artwork
             // 대체할 artwork를 제외한 다른 필드를 복사합니다.
    ...person.artwork, // with the same one
    city: 'New Delhi' // but in New Delhi!
                      // New Delhi'는 덮어씌운 채로 나머지 artwork 필드를 복사합니다!
  })
```

참고: 객체는 실제로 중첩되지 않습니다

- 이와 같은 객체는 코드에서 “중첩”되어 나타납니다.

  ```javascript
  let obj = {
    name: "Niki de Saint Phalle",
    artwork: {
      title: "Blue Nana",
      city: "Hamburg",
      image: "https://i.imgur.com/Sd1AgUOm.jpg",
    },
  };
  ```

- 그러나 “중첩”은 객체의 동작 방식을 고려해보자면 정확한 방식은 아님.
- 코드가 실행될 때 “중첩된” 객체 같은 것은 존재하지 않음.
- **실제로는 서로 다른 두 개의 객체를 보고 있는 것.**

  ```javascript
  let obj1 = {
    title: "Blue Nana",
    city: "Hamburg",
    image: "https://i.imgur.com/Sd1AgUOm.jpg",
  };

  let obj2 = {
    name: "Niki de Saint Phalle",
    artwork: obj1,
  };
  ```

- obj1은 obj2의 “내부”에 있지 않음. 예를 들어, obj3도 obj1을 “가리킬” 수 있음.

  ```javascript
  let obj1 = {
    title: "Blue Nana",
    city: "Hamburg",
    image: "https://i.imgur.com/Sd1AgUOm.jpg",
  };

  let obj2 = {
    name: "Niki de Saint Phalle",
    artwork: obj1,
  };

  let obj3 = {
    name: "Copycat",
    artwork: obj1,
  };
  ```

- obj3.artwork.city를 변이하면 obj2.artwork.city와 obj1.city 모두에 영향을 미침. obj3.artwork, obj2.artwork, obj1이 동일한 객체이기 때문. 객체를 “중첩된” 객체라고 생각하면 이 점을 이해하기 어려움. 실은 프로퍼티를 사용하여 서로를 “가리키는” 별도의 객체.

## Immer로 간결한 업데이트 로직 작성

state가 깊게 중첩된 경우 그것을 flat하게 만드는 것을 고려할 수 있음. 하지만 state 구조를 변경하고 싶지 않다면 중첩된 전개 구문보다 더 간편한 방법을 선호할 수 있음. Immer는 변이 구문을 사용하여 작성하더라도 자동으로 사본을 생성해주는 편리한 인기 라이브러리. Immer를 사용하면 작성하는 코드가 “규칙을 깨고” 객체를 변이하는 것처럼 보임.

```javascript
updatePerson((draft) => {
  draft.artwork.city = "Lagos";
});
```

하지만 일반 변이와 달리 이전 state를 덮어쓰지 않는다.

참고: Immer는 어떻게 동작하나요?

- Immer에서 제공하는 draft는 프록시라는 특수한 유형의 객체.
- 사용자가 수행하는 작업을 “기록”
- 그렇기 때문에 원하는 만큼 자유롭게 수정할 수 있음.
- Immer는 내부적으로 draft의 어떤 부분이 변경되었는지 파악하고 편집 내용이 포함된 완전히 새로운 객체를 생성.

Immer 사용 예시

1. npm install use-immer를 실행하여 Immer를 의존성으로 추가.
2. import { useState } from 'react'를 import { useImmer } from 'use-immer'로 바꾸기.

```javascript
import { useImmer } from "use-immer";

export default function Form() {
  const [person, updatePerson] = useImmer({
    name: "Niki de Saint Phalle",
    artwork: {
      title: "Blue Nana",
      city: "Hamburg",
      image: "https://i.imgur.com/Sd1AgUOm.jpg",
    },
  });

  function handleNameChange(e) {
    updatePerson((draft) => {
      draft.name = e.target.value;
    });
  }

  function handleTitleChange(e) {
    updatePerson((draft) => {
      draft.artwork.title = e.target.value;
    });
  }

  function handleCityChange(e) {
    updatePerson((draft) => {
      draft.artwork.city = e.target.value;
    });
  }

  function handleImageChange(e) {
    updatePerson((draft) => {
      draft.artwork.image = e.target.value;
    });
  }

  return (
    <>
      <label>
        Name:
        <input value={person.name} onChange={handleNameChange} />
      </label>
      <label>
        Title:
        <input value={person.artwork.title} onChange={handleTitleChange} />
      </label>
      <label>
        City:
        <input value={person.artwork.city} onChange={handleCityChange} />
      </label>
      <label>
        Image:
        <input value={person.artwork.image} onChange={handleImageChange} />
      </label>
      <p>
        <i>{person.artwork.title}</i>
        {" by "}
        {person.name}
        <br />
        (located in {person.artwork.city})
      </p>
      <img src={person.artwork.image} alt={person.artwork.title} />
    </>
  );
}
```

이벤트 핸들러가 얼마나 간결해졌는지 주목. 단일 컴포넌트에서 useState와 useImmer를 원하는 만큼 맞춰 사용할 수 있음. 특히 state에 중첩이 있고 객체를 복사하면 코드가 반복되는 경우 업데이트 핸들러를 간결하게 유지하는 데 Immer는 좋은 방법.

참고: React에서 state 변이를 권장하지 않는 이유는 무엇인가요?

1. 디버깅: 렌더링 사이에 state가 어떻게 변경되었는지 명확하게 확인할 수 있습니다.
2. 최적화: 일반적인 React 최적화 전략은 이전 프로퍼티나 state가 다음 프로퍼티나 state와 동일한 경우 작업을 건너뛰는 것에 의존. state를 변이하지 않는다면 변경이 있었는지 확인하는 것이 매우 빠름. 만약 prevObj === obj 라면, 내부에 변경된 것이 없다는 것을 확신할 수 있음.
3. 새로운 기능: 우리가 개발 중인 새로운 React 기능은 state가 스냅샷처럼 취급되는 것에 의존. 과거 버전의 state를 변이하는 경우 새로운 기능을 사용하지 못할 수 있음.
4. 요구 사항 변경: 실행 취소/다시 실행 구현, 변경 내역 표시, 사용자가 양식을 이전 값으로 재설정할 수 있도록 하는 것과 같은 일부 애플리케이션 기능은 아무것도 변이되지 않은 state에서 더 쉽게 수행할 수 있음. 과거의 state 복사본을 메모리에 보관하고 필요할 때 재사용할 수 있기 때문. 변경 접근 방식으로 시작하면 나중에 이와 같은 기능을 추가하기 어려울 수 있음.
5. 더 간단한 구현: React는 변이에 의존하지 않기 때문에 객체에 특별한 작업을 할 필요가 없음. 많은 “반응형” 솔루션처럼 프로퍼티를 가로채거나, 항상 프록시로 감싸거나, 초기화할 때 다른 작업을 할 필요가 없음. 이것이 바로 React를 사용하면 추가 성능이나 정확성의 함정 없이 아무리 큰 객체라도 state에 넣을 수 있는 이유이기도 함.

- 실제로는 React에서 state를 변이해서라도 잘 “빠져나갈” 수 있겠지만, state의 불변성을 유지하는 접근 방식을 염두에 두고 개발된 새로운 React 기능을 잘 사용할 수 있기 위해서, 그렇게 하지 말 것을 강력히 권장. 제발.
