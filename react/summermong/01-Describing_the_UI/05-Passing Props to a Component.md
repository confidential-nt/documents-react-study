# ✏️ Describing the UI : Passing Props to a Component

### Familiar props

props란 img 태그에 전달하는 className, alt, src, width...처럼 JSX 태그에 전달하는 정보를 말한다.
(사용자가 정의한 컴포넌트는 어떤 props든 전달할 수 있지만 img 태그처럼 전달할 수 있는 props가 미리 정의된 경우도 있다.)

---

### Passing props to a component

```
export default function Profile() {
  return (
    <Avatar />
  );
}
```

Profile 컴포넌트는 자식 컴포넌트인 Avata에 어떤 props를 전달하려고 한다.
이때 2단계에 걸쳐 전달할 수 있다.

1. 자식 컴포넌트에 props 전달하기

예제에서는 person(객체)와 size(숫자)를 전달한다.
person 뒤의 이중 괄호는 JSX 중괄호 안의 객체로 이해하면 된다.

```
export default function Profile() {
  return (
    <Avatar
      person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}
      size={100}
    />
  );
}
```

2. 자식 컴포넌트 내부에서 props 읽기

전달한 props들은 function Avatar 뒤의 ({}) 안에서 쉼표로 구분해 읽을 수 있다.
이렇게 읽으면 Avatar 코드 내에서 변수처럼 사용할 수 있다.

```
function Avatar({ person, size }) {
  // person and size are available here
}
```

```
import { getImageUrl } from './utils.js';

function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}

export default function Profile() {
  return (
    <div>
      <Avatar
        size={100}
        person={{
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
      />
    </div>
  );
}

```

props를 사용하면 부모 컴포넌트와 자식 컴포넌트를 독립적으로 생각할 수 있다.
React에서 데이터는 주로 위에서 아래로(props drilling) 컴포넌트 트리를 통해 전달한다.
이는 부모 컴포넌트에서 자식 컴포넌트로 데이터를 전달하는 방식을 의미한다.

상위 컴포넌트에서 하위 컴포넌트로 데이터를 전달할 때, 데이터는 props(속성)라는 형태로 전달된다.
부모 컴포넌트에서 자식 컴포넌트로 props를 통해 데이터를 전달함으로써 컴포넌트 간에 정보를 공유하고 상태를 조정할 수 있다.

이러한 데이터 흐름은 React의 단방향 데이터 바인딩(one-way data binding) 원칙에 기반하고 있다.
부모 컴포넌트에서 자식 컴포넌트로만 데이터를 전달하고, 자식 컴포넌트에서는 직접 부모 컴포넌트의 상태를 변경하지 않는다.
데이터는 부모에서 자식으로만 흐르며, 자식 컴포넌트는 받은 props를 사용하여 렌더링하거나 다른 동작을 수행한다.

즉 props는 함수의 인자와 동일한 역할을 한다.
보통은 props 자체를 필요로 하지 않기에 구조 분해 할당 하여 개별로 사용한다.

---

### Specifying a default value for a prop

값이 지정되지 않았을 때 prop에 기본값을 주려면 변수 뒤에 =와 함께 기본값을 넣어 구조 분해 할당을 해줄 수 있다.

```
function Avatar({ person, size = 100 }) {
  // ...
}
```

size prop이 없이 렌더링 된다면 size는 100으로 설정된다.
이런 기본값은 size prop가 없거나, size={undefined}로 전달될 때 사용된다.
하지만 만약 size={null} || size={0}으로 전달된다면 설정한 기본값은 사용되지 않는다.
(null, 0은 **명시적으로 값이 없다는 값**을 나타내는 것이기 때문에)

---

### Forwarding props with the JSX spread syntax

가끔 전달되는 props들은 반복적일 수 있는데, 이 경우 전개 구문을 사용하면 코드가 더 간결해진다.

```
function Profile({ person, size, isSepia, thickBorder }) {
  return (
    <div className="card">
      <Avatar
        person={person}
        size={size}
        isSepia={isSepia}
        thickBorder={thickBorder}
      />
    </div>
  );
}
```

```
function Profile(props) {
  return (
    <div className="card">
      <Avatar {...props} />
    </div>
  );
}
```

---

### Passing JSX as children

JSX 태그 내에 컨텐츠를 중첩하면 부모 컴포넌트는 children이라는 prop를 받을 수 있다.
아래 코드는 Card 컴포넌트가 <Avarta />라는 children prop를 받아 렌더링한 것이다.

```
import Avatar from './Avatar.js';

function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

export default function Profile() {
  return (
    <Card>
      <Avatar
        size={100}
        person={{
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
      />
    </Card>
  );
}
```

---

### How props change over time

props는 항상 고정된 것이 아니다.
즉 컴포넌트의 데이터를 '처음에만' 반영하는 것이 아니라 시간에 따라 매번 다른 props를 받는다.

하지만 props는 불변으로 CS에서 변경할 수 없다는 뜻으로 사용된다.
컴포넌트는 props를 변경해야 할 때마다 부모 컴포넌트에 다른 props(객체)를 전달하도록 요청한다.
그러면 이전의 props는 버려지고(참조 끊기) JS 엔진은 기존의 props가 차지했던 메모리를 회수한다. (=가비지 콜렉팅)

되도록 props를 변경하려 하지 말고, set State를 사용해야 한다.

---

참고: https://react-ko.dev/learn/passing-props-to-a-component
