# ✏️ Describing the UI : Conditional Rendering

### 삼항 연산자

상품의 포장 여부를 확인하는 isPacked의 prop에 따라 체크 표시를 주려고 한다.

```
function Item({ name, isPacked }) {
  if (isPacked) {
    return <li className="item">{name} ✔</li>;
  }
  return <li className="item">{name}</li>;
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item
          isPacked={true}
          name="Space suit"
        />
        <Item
          isPacked={true}
          name="Helmet with a golden leaf"
        />
        <Item
          isPacked={false}
          name="Photo of Tam"
        />
      </ul>
    </section>
  );
}
```

이런 식으로 prop에 따라 JSX 트리를 다르게 반환하는 경우가 있다.
하지만 prop의 값만 다를 뿐 코드의 중복이 보인다.
이러한 중복은 크리티컬하진 않지만 유지보수에 어려움이 있다.
이와 같은 경우에 조건(삼항) 연산자로 보다 가독성 좋은 코드를 작성할 수 있다.

```
return (
  <li className="item">
    {isPacked ? name + ' ✔' : name}
  </li>
);
```

이처럼 **조건 ? 결과 1 : 결과 2** 와 같은 형태로 코드가 더 깔끔해진다.
하지만 삼항 연산자도 지나치게 남용하면 컴포넌트가 지저분해지는 문제가 있으므로 때에 맞게 사용해야 한다.

---

### 논리 AND 연산자

React 컴포넌트 내 조건이 참일 때 일부 JSX만 렌더링 하려는 경우 사용한다.

```
return (
  <li className="item">
    {name} {isPacked && '✔'}
  </li>
);
```

삼항 연산자 예제와 똑같은 결과를 나타낸다.
&& 표현식은 조건이 true면 오른쪽 값을 반환한다.
하지만 조건이 false라면 표현식 전체가 false가 된다.
React는 false를 null, undefined처럼 인지해 아무 것도 렌더링하지 않게 된다.

---

### 변수에 조건부로 JSX 할당하기

if문과 변수를 사용해 조건부로 JSX를 할당할 수 있다.

```
function Item({ name, isPacked }) {
  let itemContent = name;
  if (isPacked) {
    itemContent = name + " ✔";
  }
  return (
    <li className="item">
      {itemContent}
    </li>
  );
}

export default function PackingList() {
  return (
    <section>
      <h1>Sally Ride's Packing List</h1>
      <ul>
        <Item
          isPacked={true}
          name="Space suit"
        />
        <Item
          isPacked={true}
          name="Helmet with a golden leaf"
        />
        <Item
          isPacked={false}
          name="Photo of Tam"
        />
      </ul>
    </section>
  );
}
```

---

React에서는 JS로 분기 로직을 제어한다.
if문으로 조건부로 JSX 표현식을 반환할 수 있다.
중괄호로 일부 JSX를 변수에 조건부로 저장한 뒤 다른 JSX 안에 포함시킬 수 있다.

---

참고: https://react-ko.dev/learn/conditional-rendering
