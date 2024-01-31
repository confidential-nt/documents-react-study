## Conditional Rendering

### 들어가면서

컴포넌트는 서로 다른 조건에 따라 다르게 보여줘야 하는 경우가 있다.
이때 if, &&, ? : (삼항연산자) 등을 사용할 수 있다.

### 조건부로 반환하는 JSX

if-else 사용하기

### null을 사용해 조건부로 아무것도 반환하지 않기

어떤 상황에서는 아무것도 렌더링하고 싶지 않을 수도. -> 그런데 컴포넌트는 무언가를 반환해야 -> null을 반환하면 됨.

그런데 **'return null' 자체는 좀 일반적이지 않고 개발자를 놀라게할 수 있으므로 주의해야한다고..** 부모 컴포넌트의 JSX에 컴포넌트를 조건부로 포함하거나 제외하는 경우가 더 많고 그 방법들은 다음과 같다.

### 조건을 포함한 JSX

```javascript
if (isPacked) {
  return <li className="item">{name} ✔</li>;
}
return <li className="item">{name}</li>;
```

위 코드를 봐라. 중복이 있다. 해롭지는 않지만 코드를 유지 관리하기 어렵게 만들 수 있음. 이런 상황에서는 조건부로 약간의 JSX를 포함시켜 덜 반복적이게 만들 수 있다.

#### 조건(삼항) 연산자(? :)

```javascript
return <li className="item">{isPacked ? name + " ✔" : name}</li>;
```

- 참고: 위에 나온 예제와 이 예제는 완전히 동일한 내용인가? 완전히 동등한가? 동일하다. OOP에 익숙하다면, 위 예제들은 <li>의 서로 다른 두 “인스턴스”를 생성할 수 있기 때문에 미묘하게 다르다고 생각할 수 있으나, JSX 요소는 내부 state를 보유하지 않고 실제 DOM 노드가 아니기 때문에 “인스턴스”가 아니다. 청사진일 뿐..

단, 이 스타일 남용하면 코드 더러워짐. 적당히 사용하도록하고, 컴포넌트가 지저분해지면 자식 컴포넌트로 빼라.

#### 논리 AND 연산자(&&)

React 컴포넌트 내에서 **조건이 참일 때 일부 JSX를 렌더링하거나 그렇지 않으면 아무것도 렌더링하지 않으려 할 때** 자주 사용

```javascript
return (
  <li className="item">
    {name} {isPacked && "✔"}
  </li>
);
```

"isPacked이면 (&&) 체크 표시를 렌더링하고, **그렇지 않으면 아무것도 렌더링하지 않습니다**" 라는 뜻.

왼쪽(조건)이 true이면 오른쪽(이 경우 체크 표시)의 값을 반환. 하지만 조건이 false이면 표현식 전체가 false가 됨.**React는 false를 null이나 undefined와 마찬가지로 JSX 트리상의 “구멍”으로 간주하고**, 그 자리에 아무것도 렌더링하지 않는다.

- 주의사항: **&&의 왼쪽에 숫자를 넣지 마라.**
  - &&의 특성상 왼쪽이 0이면 falsy한 값으로 판정되긴 하는데, 단축 평가 때문에 0이 반환됨 -> **UI에 0이 표시됨.**
  - 흔히 하는 실수 중 하나는 messageCount && \<p>New messages\</p>와 같은 코드를 작성하는 것. messageCount가 0일 때 아무것도 렌더링하지 않는다고 생각하기 쉽지만, 실제로는 0 자체를 렌더링하게 됨.
  - 이 문제를 해결하려면 왼쪽을 불리언으로 만들어라: messageCount > 0 && \<p>New messages\</p> -> 왜냐? false가 반환되면 null과 마찬가지로 아무것도 렌더링 안한다고 했으니까.
  - !!messageCount &&\<p>New messages\</p>도 가능.

#### 변수에 조건부로 JSX 할당하기

아 이런거 너무 복잡하다. 귀찮다. 알기 어렵다. -> if문 + let의 조합 이용.

```javascript
let itemContent = name;
if (isPacked) {
  itemContent = name + " ✔";
}
return <li className="item">{itemContent}</li>;
```

이 스타일은 가장 장황하지만 가장 유연하기도 함.
