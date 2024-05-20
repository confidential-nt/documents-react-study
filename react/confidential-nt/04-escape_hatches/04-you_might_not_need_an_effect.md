# You Might Not Need an Effect

## 들어가면서

Effect는 React 패러다임에서 벗어날 수 있는 탈출구. Effect를 사용하면 React의 “외부로 나가서” 컴포넌트를 React가 아닌 위젯, 네트워크 또는 브라우저 DOM과 같은 외부 시스템과 동기화할 수 있음. 외부 시스템이 관여하지 않는 경우(예: 일부 props나 state가 변경될 때 컴포넌트의 state를 업데이트하려는 경우)에는 Effect가 필요하지 않음. 불필요한 Effect를 제거하면 코드를 더 쉽게 따라갈 수 있고, 실행 속도가 빨라지며, 오류 발생 가능성이 줄어듦.

## 불필요한 Effect를 제거하는 방법

Effect가 필요하지 않은 흔한 경우는 두 가지가 있음.

- **렌더링을 위해 데이터를 변환하는 경우 Effect는 필요하지 않음.** 예를 들어, 목록을 표시하기 전에 필터링하고 싶다고 가정. 목록이 변경될 때 state 변수를 업데이트하는 Effect를 작성하고 싶을 수 있음. 하지만 이는 비효율적. 컴포넌트의 state를 업데이트할 때 React는 먼저 컴포넌트 함수를 호출해 화면에 표시될 내용을 계산. 다음으로 이러한 변경 사항을 DOM에 “commit”하여 화면을 업데이트하고, 그 후에 Effect를 실행. 만약 Effect “역시” state를 즉시 업데이트한다면, 이로 인해 전체 프로세스가 처음부터 다시 시작될 것! 불필요한 렌더링을 피하려면 모든 데이터 변환을 컴포넌트의 최상위 레벨에서 하라. 그러면 props나 state가 변경될 때마다 해당 코드가 자동으로 다시 실행될 것.
- **사용자 이벤트를 처리하는 데에 Effect는 필요하지 않음.** 예를 들어, 사용자가 제품을 구매할 때 /api/buy POST 요청을 전송하고 알림을 표시하고 싶다고 하자. 구매 버튼 클릭 이벤트 핸들러에서는 정확히 어떤 일이 일어났는지 알 수 있음. 반면 Effect는 사용자가 무엇을 했는지(예: 어떤 버튼을 클릭했는지)를 알 수 없음. 그렇기 때문에 일반적으로 사용자 이벤트를 해당 이벤트 핸들러에서 처리.

한편 외부 시스템과 동기화하려면 Effect가 필요. 예를 들어, jQuery 위젯을 React state와 동기화하는 Effect를 작성할 수 있음. 또한 검색 결과를 현재의 검색 쿼리와 동기화하기 위해 데이터 요청을 Effect로 처리할 수 있음. 최신 프레임워크는 컴포넌트에 직접 Effects를 작성하는 것보다 더 효율적인 빌트인 데이터 페칭 메커니즘을 제공한다는 점을 명심.

이에 대한 구체적인 예 몇 가지를 살펴보자.

## props 또는 state에 따라 state 업데이트하기

firstName과 lastName이라는 두 개의 state 변수가 있는 컴포넌트를 가정. 이 두 변수를 연결하여 fullName을 계산하고 싶음. 또한, firstName 또는 lastName이 변경될 때마다 fullName이 업데이트되기를 원함. 가장 먼저 생각나는 것은 fullName state 변수를 추가하고 Effect에서 업데이트하는 것일 수 있음.

```javascript
function Form() {
  const [firstName, setFirstName] = useState("Taylor");
  const [lastName, setLastName] = useState("Swift");

  // 🔴 Avoid: redundant state and unnecessary Effect
  // 🔴 이러지 마세요: 중복 state 및 불필요한 Effect
  const [fullName, setFullName] = useState("");
  useEffect(() => {
    setFullName(firstName + " " + lastName);
  }, [firstName, lastName]);
  // ...
}
```

이는 필요 이상으로 복잡하고 비효율적. state 변수와 Effect를 모두 제거 하라.

```javascript
function Form() {
  const [firstName, setFirstName] = useState("Taylor");
  const [lastName, setLastName] = useState("Swift");
  // ✅ Good: calculated during rendering
  // ✅ 좋습니다: 렌더링 과정 중에 계산
  const fullName = firstName + " " + lastName;
  // ...
}
```

**기존 props나 state에서 계산할 수 있는 것이 있으면 state에 넣지 마라. 대신 렌더링 중에 계산하라.** 이렇게 하면 코드가 더 빨라지고(추가적인 “계단식” 업데이트를 피함), 더 간단해지고(일부 코드 제거), 오류가 덜 발생합니다(서로 다른 state 변수가 서로 동기화되지 않아 발생하는 버그를 피함). 이 접근 방식이 생소하게 느껴진다면, React로 사고하기에서 state에 들어가야할 내용이 무엇인지 확인.

## 고비용 계산 캐싱하기

아래 컴포넌트는 props로 받은 todos를 filter prop에 따라 필터링하여 visibleTodos를 계산. 이 결과를 state 변수에 저장하고 Effect에서 업데이트하고 싶을 수도 있을 것.

```javascript
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState("");

  // 🔴 Avoid: redundant state and unnecessary Effect
  // 🔴 이러지 마세요: 중복 state 및 불필요한 Effect
  const [visibleTodos, setVisibleTodos] = useState([]);
  useEffect(() => {
    setVisibleTodos(getFilteredTodos(todos, filter));
  }, [todos, filter]);

  // ...
}
```

앞의 예시에서와 마찬가지로 이것은 불필요하고 비효율적. state와 Effect를 제거.

```javascript
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState("");
  // ✅ This is fine if getFilteredTodos() is not slow.
  // ✅ getFilteredTodos()가 느리지 않다면 괜찮습니다.
  const visibleTodos = getFilteredTodos(todos, filter);
  // ...
}
```

일반적으로 위 코드는 괜찮. 하지만 getFilteredTodos()가 느리거나 todos가 많을 경우, newTodo와 같이 관련 없는 state 변수가 변경되더라도 getFilteredTodos()를 다시 계산하고 싶지 않을 수 있음.

이럴 땐 값비싼 계산을 useMemo 훅으로 감싸서 캐시(또는 “메모화 (memoize)“)할 수 있음.

```javascript
import { useMemo, useState } from "react";

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState("");
  const visibleTodos = useMemo(() => {
    // ✅ Does not re-run unless todos or filter change
    // ✅ todos나 filter가 변하지 않는 한 재실행되지 않음
    return getFilteredTodos(todos, filter);
  }, [todos, filter]);
  // ...
}
```

한 줄로 작성하려면 다음과 같이.

```javascript
import { useMemo, useState } from "react";

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState("");
  // ✅ Does not re-run getFilteredTodos() unless todos or filter change
  // ✅ todos나 filter가 변하지 않는 한 getFilteredTodos()가 재실행되지 않음
  const visibleTodos = useMemo(
    () => getFilteredTodos(todos, filter),
    [todos, filter]
  );
  // ...
}
```

이렇게 하면 todos나 filter가 변경되지 않는 한 내부 함수가 다시 실행되지 않기를 원한다는 것을 React에 알림. 그러면 React는 초기 렌더링 중에 getFilteredTodos()의 반환값을 기억. 그 다음부터는 렌더링 중에 todos나 filter가 다른지 확인. 지난번과 동일하다면 useMemo는 마지막으로 저장한 결과를 반환. 같지 않다면, React는 내부 함수를 다시 호출하고 그 결과를 저장.

useMemo로 감싸는 함수는 렌더링 중에 실행되므로, 순수 계산에만 작동.

- 참고: 계산이 비싼지는 어떻게 알 수 있나요?
  - 일반적으로 수천 개의 객체를 만들거나 반복하는 경우가 아니라면 비용이 많이 든다고 보지 않을 것. 좀 더 확신을 얻고 싶다면 콘솔 로그를 추가하여 코드에 소요된 시간을 측정할 수 있음.
    ```javascript
    console.time("filter array");
    const visibleTodos = getFilteredTodos(todos, filter);
    console.timeEnd("filter array");
    ```
  - 측정하려는 상호작용을 수행(예: input에 입력). 그러면 filter array : 0.15ms 라는 로그가 콘솔에 표시되는 것을 보게될 것. 기록된 전체 시간이 상당하다면(예: 1ms 이상) 해당 계산은 메모해 두는 것이 좋을 수 있음. 실험삼아 해당 계산을 useMemo로 감싸서 해당 상호작용에 대해 총 로그된 시간이 감소했는지 여부를 확인할 수 있음.
    ```javascript
    console.time("filter array");
    const visibleTodos = useMemo(() => {
      return getFilteredTodos(todos, filter); // Skipped if todos and filter haven't changed
    }, [todos, filter]);
    console.timeEnd("filter array");
    ```
  - **useMemo는 첫 번째 렌더링을 더 빠르게 만들지는 않음.** 업데이트 시 불필요한 작업을 건너뛰는 데에만 도움이 될 뿐.
  - 컴퓨터가 사용자보다 빠를 수 있으므로 인위적으로 속도 저하를 일으켜서 성능을 테스트하는 것도 좋은 생각. 예를 들어, Chrome에서는 CPU 스로틀링 옵션을 제공
  - 또한 개발 중에 성능을 측정하는 것은 정확한 결과를 제공하지는 않는다는 점에 유의. (예를 들어, Strict Mode를 켜면, 각 컴포넌트가 한 번이 아닌 두 번씩 렌더링되는 것을 볼 수 있음) 가장 정확한 타이밍을 얻으려면 상용 앱을 빌드하고 사용자가 사용하는 것과 동일한 기기에서 테스트하라.

## prop이 변경되면 모든 state 재설정하기

다음 ProfilePage 컴포넌트는 userId prop을 받음. 이 페이지에는 코멘트 input이 포함되어 있으며, comment state 변수를 사용하여 그 값을 보관. 어느 날, 한 프로필에서 다른 프로필로 이동할 때 comment state가 재설정되지 않는 문제를 발견함. 이 문제를 해결하려면 userId가 변경될 때마다 comment state 변수를 지워줘야 함.

```javascript
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState("");

  // 🔴 Avoid: Resetting state on prop change in an Effect
  // 🔴 이러지 마세요: prop 변경시 Effect에서 state 재설정 수행
  useEffect(() => {
    setComment("");
  }, [userId]);
  // ...
}
```

이것은 ProfilePage와 그 자식들이 먼저 오래된 값으로 렌더링한 다음 새로운 값으로 다시 렌더링하기 때문에 비효율적. 또한 ProfilePage 내부에 어떤 state가 있는 모든 컴포넌트에서 이 작업을 수행해야 하므로 복잡. 예를 들어, 댓글 UI가 중첩되어 있는 경우 중첩된 하위 댓글 state들도 모두 지워야 할 것.

그 대신 명시적인 키를 전달해 각 사용자의 프로필이 개념적으로 다른 프로필이라는 것을 React에 알릴 수 있음. 컴포넌트를 둘로 나누고 바깥쪽 컴포넌트에서 안쪽 컴포넌트로 key 속성을 전달.

```javascript
export default function ProfilePage({ userId }) {
  return <Profile userId={userId} key={userId} />;
}

function Profile({ userId }) {
  // ✅ This and any other state below will reset on key change automatically
  // ✅ key가 변하면 이 컴포넌트 및 모든 자식 컴포넌트의 state가 자동으로 재설정됨
  const [comment, setComment] = useState("");
  // ...
}
```

일반적으로 React는 같은 컴포넌트가 같은 위치에서 렌더링될 때 state를 유지. userId를 key로 Profile 컴포넌트에 전달하는 것은 곧, userId가 다른 두 Profile 컴포넌트를 state를 공유하지 않는 별개의 컴포넌트들로 취급하도록 React에게 요청하는 것. **React는 (userId로 설정한) key가 변경될 때마다 DOM을 다시 생성하고 state를 재설정하며, Profile 컴포넌트 및 모든 자식들의 state를 재설정할 것.** 그 결과 comment 필드는 프로필들을 탐색할 때마다 자동으로 지워짐.

위 예제에서는 외부의 ProfilePage 컴포넌트만 export하였으므로 프로젝트의 다른 파일에서는 오직 ProfilePage 컴포넌트에만 접근 가능. ProfilePage를 렌더링하는 컴포넌트는 key를 전달할 필요 없이 일반적인 prop으로 userId만 전달하고 있음. ProfilePage가 내부의 Profile 컴포넌트에 key로 전달한다는 사실은 내부에서만 알고 있는 구현 세부 사항.

## props가 변경될 때 일부 state 조정하기

때론 prop이 변경될 때 state의 전체가 아닌 일부만 재설정하거나 조정하고 싶을 수 있음.

다음의 List 컴포넌트는 items 목록을 prop으로 받고, selection state 변수에 선택된 항목을 유지합니다. items prop이 다른 배열을 받을 때마다 selection을 null로 재설정하고 싶음.

```javascript
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 🔴 Avoid: Adjusting state on prop change in an Effect
  // 🔴 이러지 마세요: prop 변경시 Effect에서 state 조정
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}
```

이것 역시 이상적이지 않음. items가 변경될 때마다 List와 그 하위 컴포넌트는 처음에는 오래된 selection값으로 렌더링됨. 그런 다음 React는 DOM을 업데이트하고 Effects를 실행함. 마지막으로 setSelection(null) 호출은 List와 그 자식 컴포넌트를 다시 렌더링하여 이 전체 과정을 재시작하게 됨.

Effect를 삭제하는 것으로 시작. 대신 렌더링 중에 직접 state를 조정.

```javascript
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // Better: Adjust the state while rendering
  // 더 나음: 렌더링 중에 state 조정
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```

이렇게 이전 렌더링의 정보를 저장하는 것은 이해하기 어려울 수 있지만, Effect에서 동일한 state를 업데이트하는 것보다는 나음. 위 예시에서는 렌더링 도중 setSelection이 직접 호출됨. React는 return문과 함께 종료된 직후에 List를 다시 렌더링함. 이 시점에서 React는 아직 List의 자식들을 렌더링하거나 DOM을 업데이트하지 않았기 때문에, List의 자식들은 기존(stale)의 selection 값에 대한 렌더링을 건너뛰게 됨.

**렌더링 도중 컴포넌트를 업데이트하면, React는 반환된 JSX를 버리고 즉시 렌더링을 다시 시도함.** React는 계단식으로 전파되는 매우 느린 재시도를 피하기 위해, **렌더링 중에 동일한 컴포넌트의 state만 업데이트할 수 있도록 허용.** **렌더링 도중 다른 컴포넌트의 state를 업데이트하면 오류가 발생.** 동일 컴포넌트가 무한으로 리렌더링을 반복 시도하는 상황을 피하기 위해 items !== prevItems와 같은 조건이 필요한 것. 이런 식으로 state를 조정할 수 있긴 하지만, 다른 side effect(DOM 변경이나 timeout 설정 등)은 이벤트 핸들러나 Effect에서만 처리함으로써 컴포넌트의 순수성을 유지해야 함.

**이 패턴은 Effect보다 효율적이지만, 대부분의 컴포넌트에는 필요하지 않음.** 어떻게 하든 props나 다른 state들을 바탕으로 state를 조정하면 데이터 흐름을 이해하고 디버깅하기 어려워질 것. **항상 key로 모든 state를 재설정하거나 렌더링 중에 모두 계산할 수 있는지를 확인하라.** 예를 들어, 선택한 item을 저장(및 재설정)하는 대신, 선택한 item의 ID를 저장할 수 있음.

```javascript
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ Best: Calculate everything during rendering
  // ✅ 가장 좋음: 렌더링 중에 모든 값을 계산
  const selection = items.find((item) => item.id === selectedId) ?? null;
  // ...
}
```

이제 state를 “조정”할 필요가 전혀 없음. 선택한 ID를 가진 항목이 목록에 있으면 선택된 state로 유지됨. 그렇지 않은 경우 렌더링 중에 계산된 selection 항목은 일치하는 항목을 찾지 못하므로 null이 됨. 이 방식은 items에 대한 대부분의 변경과 무관하게 ‘selection’ 항목은 그대로 유지되므로 대체로 더 나은 방법.

## 이벤트 핸들러 간 로직 공유

해당 제품을 구매할 수 있는 두 개의 버튼(구매 및 결제)이 있는 제품 페이지가 있다고 하자. 사용자가 제품을 장바구니에 넣을 때마다 알림을 표시하고 싶음. 두 버튼의 클릭 핸들러에 모두 showNotification() 호출을 추가하는 것은 반복적으로 느껴지므로 이 로직을 Effect에 배치하고 싶을 수 있음.

```javascript
function ProductPage({ product, addToCart }) {
  // 🔴 Avoid: Event-specific logic inside an Effect
  // 🔴 이러지 마세요: Effect 내부에 특정 이벤트에 대한 로직 존재
  useEffect(() => {
    if (product.isInCart) {
      showNotification(`Added ${product.name} to the shopping cart!`);
    }
  }, [product]);

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo("/checkout");
  }
  // ...
}
```

이 Effect는 불필요. 또한 버그를 촉발할 가능성이 높음. 예를 들어, 페이지가 새로고침 될 때마다 앱이 장바구니를 “기억”한다고 가정해 봅시다. 카트에 제품을 한 번 추가하고 페이지를 새로고침 하면 알림이 다시 표시됨. 또한 해당 제품 페이지를 새로고침할 때에도 여전히 알림이 계속 등장. 이는 페이지를 불러올 때 product.isInCart가 이미 true이므로 위의 Effect가 showNotification()을 호출하기 때문

**어떤 코드가 Effect에 있어야 하는지 이벤트 핸들러에 있어야 하는지 확실치 않은 경우, 이 코드가 실행되어야 하는 이유를 자문해 보라.** **컴포넌트가 사용자에게 표시되었기 때문에 실행되어야 하는 코드에만 Effect를 사용하라.** 이 예제에서는 페이지가 표시되었기 때문이 아니라, 사용자가 버튼을 눌렀기 때문에 알림이 표시되어야 함. Effect를 삭제하고 공유 로직을 두 이벤트 핸들러에서 호출하는 함수에 넣어라.

```javascript
function ProductPage({ product, addToCart }) {
  // ✅ Good: Event-specific logic is called from event handlers
  // ✅ 좋습니다: 이벤트 핸들러 안에서 각 이벤트별 로직 호출
  function buyProduct() {
    addToCart(product);
    showNotification(`Added ${product.name} to the shopping cart!`);
  }

  function handleBuyClick() {
    buyProduct();
  }

  function handleCheckoutClick() {
    buyProduct();
    navigateTo("/checkout");
  }
  // ...
}
```

## POST요청 보내기

이 Form 컴포넌트는 두 종류의 POST 요청을 전송. 마운트될 때에는 분석 이벤트를 보냄. 양식을 작성하고 제출 버튼을 클릭하면 /api/register 엔드포인트로 POST 요청을 보냄.

```javascript
function Form() {
  const [firstName, setFirstName] = useState("");
  const [lastName, setLastName] = useState("");

  // ✅ Good: This logic should run because the component was displayed
  // ✅ 좋습니다: '컴포넌트가 표시되었기 때문에 로직이 실행되어야 하는 경우'에 해당
  useEffect(() => {
    post("/analytics/event", { eventName: "visit_form" });
  }, []);

  // 🔴 Avoid: Event-specific logic inside an Effect
  // 🔴 이러지 마세요: Effect 내부에 특정 이벤트에 대한 로직 존재
  const [jsonToSubmit, setJsonToSubmit] = useState(null);
  useEffect(() => {
    if (jsonToSubmit !== null) {
      post("/api/register", jsonToSubmit);
    }
  }, [jsonToSubmit]);

  function handleSubmit(e) {
    e.preventDefault();
    setJsonToSubmit({ firstName, lastName });
  }
  // ...
}
```

이전 예제와 동일한 기준을 적용.

분석 POST 요청은 Effect에 남아 있어야 함. 분석 이벤트를 전송하는 이유는 form이 표시되었기 때문.

그러나 /api/register POST 요청은 form이 표시되어서 발생하는 것이 아님. 특정 시점, 즉,사용자가 버튼을 누를 때만 요청을 보내고 싶을 것. 이 요청은 해당 상호작용에서만 발생해야. 두 번째 Effect를 삭제하고 해당 POST 요청을 이벤트 핸들러로 이동.

```javascript
function Form() {
  const [firstName, setFirstName] = useState("");
  const [lastName, setLastName] = useState("");

  // ✅ Good: This logic runs because the component was displayed
  // ✅ 좋습니다: '컴포넌트가 표시되었기 때문에 로직이 실행되어야 하는 경우'에 해당
  useEffect(() => {
    post("/analytics/event", { eventName: "visit_form" });
  }, []);

  function handleSubmit(e) {
    e.preventDefault();
    // ✅ Good: Event-specific logic is in the event handler
    // ✅ 좋습니다: 이벤트 핸들러 안에서 특정 이벤트 로직 호출
    post("/api/register", { firstName, lastName });
  }
  // ...
}
```

어떤 로직을 이벤트 핸들러에 넣을지 Effect에 넣을지 선택할 때, 사용자 관점에서 어떤 종류의 로직인지에 대한 답을 찾아야. 이 로직이 특정 상호작용으로 인해 발생하는 것이라면 이벤트 핸들러에 넣어라. 사용자가 화면에서 컴포넌트를 보는 것이 원인이라면 Effect에 넣어라.

## 연쇄 계산

때로는 다른 state를 바탕으로 또다른 state를 조정하는 Effect를 연쇄적으로 사용하고 싶을 수도 있음.

```javascript
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);
  const [isGameOver, setIsGameOver] = useState(false);

  // 🔴 Avoid: Chains of Effects that adjust the state solely to trigger each other
  // 🔴 이러지 마세요: 오직 서로를 촉발하기 위해서만 state를 조정하는 Effect 체인
  useEffect(() => {
    if (card !== null && card.gold) {
      setGoldCardCount(c => c + 1);
    }
  }, [card]);

  useEffect(() => {
    if (goldCardCount > 3) {
      setRound(r => r + 1)
      setGoldCardCount(0);
    }
  }, [goldCardCount]);

  useEffect(() => {
    if (round > 5) {
      setIsGameOver(true);
    }
  }, [round]);

  useEffect(() => {
    alert('Good game!');
  }, [isGameOver]);

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    } else {
      setCard(nextCard);
    }
  }

  // ...
```

이 코드에는 두 가지 문제가 있음.

한 가지 문제는 매우 비효율적이라는 점. 컴포넌트(및 그 자식들)은 체인의 각 set 호출 사이에 다시 렌더링해야 함. 위의 예시에서 최악의 경우(setCard → 렌더링 → setGoldCardCount → 렌더링 → setRound → 렌더링 → setIsGameOver → 렌더링)에는 하위 트리의 불필요한 리렌더링이 세 번이나 발생.

속도가 느리지는 않더라도, 코드가 발전함에 따라 작성한 ‘체인’이 새로운 요구사항에 맞지 않는 경우가 발생할 수 있음. 게임 이동의 기록을 단계별로 살펴볼 수 있는 방법을 추가한다고 가정해 보자. 각 state 변수를 과거의 값으로 업데이트하여 이를 수행할 수 있음. 하지만 ‘카드’ state를 과거의 값으로 설정하면 다시 Effect 체인이 촉발되고 표시되는 데이터가 변경됨. 이와 같은 코드는 종종 경직되고 취약.

이 경우 렌더링 중에 계산 가능한 것은 계산하고 이벤트 핸들러에서 state를 조정하는 것이 좋음.

```javascript
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);

  // ✅ Calculate what you can during rendering
  // ✅ 가능한 것을 렌더링 중에 계산
  const isGameOver = round > 5;

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    }

    // ✅ Calculate all the next state in the event handler
    // ✅ 이벤트 핸들러에서 다음 state를 모두 계산
    setCard(nextCard);
    if (nextCard.gold) {
      if (goldCardCount <= 3) {
        setGoldCardCount(goldCardCount + 1);
      } else {
        setGoldCardCount(0);
        setRound(round + 1);
        if (round === 5) {
          alert('Good game!');
        }
      }
    }
  }

  // ...
```

훨씬 더 효율적. 또한 게임 기록을 볼 수 있는 방법을 구현하면 이제 다른 모든 값을 조정하는 Effect 체인을 촉발시키지 않고도 각 state 변수를 과거의 움직임으로 설정할 수 있음. 여러 이벤트 핸들러 간에 로직을 재사용해야 하는 경우 함수를 추출하여 해당 핸들러에서 함수를 호출할 수 있음.

이벤트 핸들러 내부에서 state는 스냅샷처럼 동작함을 기억. 예를 들어, setRound(round + 1)를 호출한 후에도 round 변수는 사용자가 버튼을 클릭한 시점의 값을 반영. 계산에 다음 값을 사용해야 하는 경우 const nextRound = round + 1과 같이 수동으로 정의해야.

이벤트 핸들러에서 직접 다음 state를 계산할 수 없는 경우도 있음. 예를 들어, 이전 드롭다운의 선택 값에 따라 다음 드롭다운의 옵션이 달라지는 form을 상상해 보자. 이를 네트워크와 동기화해야 한다면 Effect 체인이 적절할 것.

## 애플리케이션 초기화하기

일부 로직은 앱이 로드될 때 한 번만 실행되어야

최상위 컴포넌트의 Effect에 배치하고 싶을 수도 있음.

```javascript
function App() {
  // 🔴 Avoid: Effects with logic that should only ever run once
  // 🔴 이러지 마세요: 한 번만 실행되어야 하는 로직이 포함된 Effect
  useEffect(() => {
    loadDataFromLocalStorage();
    checkAuthToken();
  }, []);
  // ...
}
```

그러나 이 함수가 개발 중에 두 번 실행된다는 것을 금방 알게 될 것. 이 함수는 두 번 호출되도록 설계되지 않았기 때문에 인증 토큰이 무효화되는 등의 문제가 발생할 수 있음. 일반적으로 컴포넌트는 다시 마운트할 때 복원력이 있어야 합니다. 여기에는 최상위 App 컴포넌트도 포함됨.

상용 환경에서는 실제로 다시 마운트되지 않을 수 있지만, 모든 컴포넌트에서 동일한 제약 조건을 따르면 코드를 이동하고 재사용하기가 더 쉬워짐. **일부 로직이 컴포넌트 마운트당 한 번이 아니라 앱 로드당 한 번 실행되어야 하는 경우,** 최상위 변수를 추가하여 이미 실행되었는지 여부를 추적.

```javascript
let didInit = false;

function App() {
  useEffect(() => {
    if (!didInit) {
      didInit = true;
      // ✅ Only runs once per app load
      // ✅ 앱 로드당 한 번만 실행됨
      loadDataFromLocalStorage();
      checkAuthToken();
    }
  }, []);
  // ...
}
```

모듈 초기화 중이나 앱 렌더링 전에 실행할 수도 있음.

```javascript
if (typeof window !== "undefined") {
  // Check if we're running in the browser.
  // 브라우저에서 실행중인지 확인
  // ✅ Only runs once per app load
  // ✅ 앱 로드당 한 번만 실행됨
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

컴포넌트를 import할 때 최상위 레벨의 코드는 렌더링되지 않더라도 일단 한 번 실행됨. 임의의 컴포넌트를 임포트할 때 속도 저하나 예상치 못한 동작을 방지하려면 이 패턴을 과도하게 사용하지 마라. 앱 전체 초기화 로직은 App.js와 같은 루트 컴포넌트 모듈이나 애플리케이션의 엔트리 포인트에 유지.

## state변경을 부모 컴포넌트에 알리기

참 또는 거짓일 수 있는 내부 isOn state를 가진 Toggle 컴포넌트를 작성하고 있다고 가정. 토글하는 방법에는 몇 가지가 있음(클릭 또는 드래그). Toggle 내부 state가 변경될 때마다 부모 컴포넌트에 알리고 싶어서 onChange 이벤트를 prop으로 받고 Effect에서 이를 호출하도록 함.

```javascript
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  // 🔴 Avoid: The onChange handler runs too late
  // 🔴 이러지 마세요: onChange 핸들러가 너무 늦게 실행됨
  useEffect(() => {
    onChange(isOn);
  }, [isOn, onChange]);

  function handleClick() {
    setIsOn(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      setIsOn(true);
    } else {
      setIsOn(false);
    }
  }

  // ...
}
```

앞서와 마찬가지로 이것은 이상적이지 않다. 먼저 Toggle이 state를 업데이트하고, React가 화면을 업데이트. 그런 다음 React는 부모 컴포넌트로부터 전달받은 onChange 함수를 호출하는 Effect를 실행.(너무 늦음) 이제 부모 컴포넌트가 자신의 state를 업데이트하고, 다른 렌더 패스를 실행. 이보다는, 모든 것을 단일 명령 안에서 처리하는 것이 더 좋음.

Effect를 삭제하고, 대신 동일한 이벤트 핸들러 내에서 두 컴포넌트의 state를 업데이트 하자.

```javascript
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  function updateToggle(nextIsOn) {
    // ✅ Good: Perform all updates during the event that caused them
    // ✅ 좋습니다: 이벤트 발생시 모든 업데이트를 수행
    setIsOn(nextIsOn);
    onChange(nextIsOn);
  }

  function handleClick() {
    updateToggle(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      updateToggle(true);
    } else {
      updateToggle(false);
    }
  }

  // ...
}
```

이 접근에 따르면 Toggle 컴포넌트 및 부모 컴포넌트 모두 이벤트가 발생하는 동안 state를 업데이트. React는 서로 다른 컴포넌트에서 일괄 업데이트를 함께 처리하므로, 렌더링 과정은 한 번만 발생.

(React는 여러 컴포넌트에서 동시에 발생하는 state 업데이트를 모아서 한 번에 처리하는 최적화 메커니즘을 가지고 있음. 이를 "batching"이라고 함. 이는 React가 성능을 최적화하기 위해 동일한 이벤트 루프에서 발생하는 여러 state 업데이트를 모아서 한 번에 렌더링하는 것. 위의 최적화 덕분에, Toggle 컴포넌트와 부모 컴포넌트 모두의 state가 같은 이벤트 처리 루프에서 업데이트되면, React는 이들을 개별적으로 렌더링하지 않고 한 번에 렌더링함.결과적으로, 여러 state 업데이트가 발생하더라도 렌더링은 한 번만 이루어져서 성능이 향상됨.)

state를 완전히 제거하고 그 대신 부모 컴포넌트로부터 isOn을 받을 수도.

```javascript
// ✅ Also good: the component is fully controlled by its parent
// ✅ 좋습니다: 부모 컴포넌트에 의해 완전히 제어됨
function Toggle({ isOn, onChange }) {
  function handleClick() {
    onChange(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      onChange(true);
    } else {
      onChange(false);
    }
  }

  // ...
}
```

**“state 끌어올리기”는 부모 컴포넌트가 부모 자체의 state를 토글하여 Toggle을 완전히 제어할 수 있게 해줌. 이는 부모 컴포넌트가 더 많은 로직을 포함해야 하지만, 전체적으로 걱정해야 할 state가 줄어든다는 것을 의미.** 두 개의 서로 다른 state 변수를 동기화하려고 할 때마다, 대신 state를 끌어올려 보자.

## 부모에게 데이터 전달하기

다음 Child 컴포넌트는 일부 데이터를 페치한 다음 Effect에서 Parent 컴포넌트에 전달

```javascript
function Parent() {
  const [data, setData] = useState(null);
  // ...
  return <Child onFetched={setData} />;
}

function Child({ onFetched }) {
  const data = useSomeAPI();
  // 🔴 Avoid: Passing data to the parent in an Effect
  // 🔴 이러지 마세요: Effect에서 부모에게 데이터 전달
  useEffect(() => {
    if (data) {
      onFetched(data);
    }
  }, [onFetched, data]);
  // ...
}
```

**React에서 데이터는 부모 컴포넌트에서 자식 컴포넌트로 흐름.** 화면에 뭔가 잘못된 것이 보이면, 컴포넌트 체인을 따라 올라가서 어떤 컴포넌트가 잘못된 prop을 전달하거나 잘못된 state를 가지고 있는지 찾아냄으로써 정보의 출처를 추적할 수 있음. **자식 컴포넌트가 Effect에서 부모 컴포넌트의 state를 업데이트하면, 데이터 흐름을 추적하기가 매우 어려워짐.** 자식과 부모 컴포넌트 모두 동일한 데이터가 필요하므로, 대신 부모 컴포넌트가 해당 데이터를 페치해서 자식에게 전달하도록 하라.

```javascript
function Parent() {
  const data = useSomeAPI();
  // ...
  // ✅ Good: Passing data down to the child
  // ✅ 좋습니다: 자식에게 데이터 전달
  return <Child data={data} />;
}

function Child({ data }) {
  // ...
}
```

## 외부 스토어 구독하기

때로는 컴포넌트가 React state 외부의 일부 데이터를 구독해야 할 수도 있음. 서드파티 라이브러리나 브라우저 빌트인 API에서 데이터를 가져와야 할 수도 있음. 이 데이터는 React가 모르는 사이에 변경될 수도 있는데, 그럴 땐 수동으로 컴포넌트가 해당 데이터를 구독하도록 해야. 이 작업은 종종 Effect에서 수행함.

```javascript
function useOnlineStatus() {
  // Not ideal: Manual store subscription in an Effect
  // 이상적이지 않음: Effect에서 수동으로 store 구독
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function updateState() {
      setIsOnline(navigator.onLine);
    }

    updateState();

    window.addEventListener("online", updateState);
    window.addEventListener("offline", updateState);
    return () => {
      window.removeEventListener("online", updateState);
      window.removeEventListener("offline", updateState);
    };
  }, []);
  return isOnline;
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

여기서 컴포넌트는 외부 데이터 저장소(이 경우 브라우저 navigator.onLine API)에 구독함. 이 API는 서버에 존재하지 않으므로(따라서 초기 HTML을 생성하는 데 사용할 수 없으므로) 처음에는 state가 true로 설정됨. 브라우저에서 해당 데이터 저장소의 값이 변경될 때마다 컴포넌트는 해당 state를 업데이트함.

이를 위해 Effect를 사용하는 것이 일반적이지만, **React에는 외부 저장소를 구독하기 위해 특별히 제작된 훅이 있음. Effect를 삭제하고 useSyncExternalStore호출로 대체**

```javascript
function subscribe(callback) {
  window.addEventListener("online", callback);
  window.addEventListener("offline", callback);
  return () => {
    window.removeEventListener("online", callback);
    window.removeEventListener("offline", callback);
  };
}

function useOnlineStatus() {
  // ✅ Good: Subscribing to an external store with a built-in Hook
  // ✅ 좋습니다: 빌트인 훅에서 외부 store 구독
  return useSyncExternalStore(
    subscribe, // React won't resubscribe for as long as you pass the same function
    // React는 동일한 함수를 전달하는 한 다시 구독하지 않음
    () => navigator.onLine, // How to get the value on the client
    // 클라이언트에서 값을 가져오는 방법
    () => true // How to get the value on the server
    // 서버에서 값을 가져오는 방법
  );
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

이 접근 방식은 변경 가능한 데이터를 Effect를 사용해 React state에 수동으로 동기화하는 것보다 오류 가능성이 적음. 일반적으로 위의 useOnlineStatus()와 같은 커스텀 훅을 작성하여 개별 컴포넌트에서 이 코드를 반복할 필요가 없도록 함. React 컴포넌트에서 외부 store를 구독하는 방법에 대해 자세히 읽어보자.

## 데이터 페칭하기

많은 앱이 데이터 페칭을 시작하기 위해 Effect를 사용함. 이와 같은 데이터 페칭 Effect를 작성하는 것은 매우 일반적.

```javascript
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    // 🔴 Avoid: Fetching without cleanup logic
    // 🔴 이러지 마세요: 클린업 없이 fetch 수행
    fetchResults(query, page).then((json) => {
      setResults(json);
    });
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

이 페치를 이벤트 핸들러로 옮길 필요는 없음.

이벤트 핸들러에 로직을 넣어야 했던 앞선 예제와 모순되는 것처럼 보일 수 있음. 하지만 페치해야 하는 주된 이유가 타이핑 이벤트가 아니라는 점을 생각해 보라. 검색 입력은 URL에 미리 채워져 있는 경우가 많으며, 사용자는 input을 건드리지 않고도 앞뒤로 탐색할 수 있음.

page와 query가 어디에서 오는지는 중요하지 않음. 이 컴포넌트가 표시되는 동안 현재의 page 및 query에 대한 네트워크의 데이터와 results의 동기화가 유지되면 됨. 이것이 Effect인 이유

다만 위 코드에는 버그가 있음. "hello"를 빠르게 입력한다고 하자. 그러면 query가 "h"에서 "he", "hel", "hell", "hello"로 변경됨. 이렇게 하면 각각 페칭을 수행하지만, 어떤 순서로 응답이 도착할지는 보장할 수 없음. 예를 들어, "hell" 응답은 "hello" 응답 이후에 도착할 수 있음. 이에 따라 마지막에 호출된 setResults()로부터 잘못된 검색 결과가 표시될 수 있음. **이를 “경쟁 조건”이라고 함.** **서로 다른 두 요청이 서로 “경쟁”하여 예상과 다른 순서로 도착한 경우임.**

경쟁 조건을 수정하기 위해서는 오래된 응답을 무시하도록 클린업 함수를 추가해야

```javascript
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);
  useEffect(() => {
    let ignore = false;
    fetchResults(query, page).then((json) => {
      if (!ignore) {
        setResults(json);
      }
    });
    return () => {
      ignore = true;
    };
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

이렇게 하면 Effect가 데이터를 페치할 때 마지막으로 요청된 응답을 제외한 모든 응답이 무시됨.

데이터 페칭을 구현할 때 경합 조건을 처리하는 것만 어려운 것은 아님. 응답을 캐시하는 방법(사용자가 Back을 클릭하고면 스피너 대신 이전 화면을 즉시 볼 수 있도록), 서버에서 페치하는 방법(초기 서버 렌더링 HTML에 스피너 대신 가져온 콘텐츠가 포함되도록), 네트워크 워터폴을 피하는 방법(데이터를 페치해야 하는 하위 컴포넌트가 시작하기 전에, 위의 모든 부모가 데이터 페치를 완료할 때까지 기다릴 필요가 없도록) 등도 고려해볼 사항.

이런 문제는 React뿐만 아니라 모든 UI 라이브러리에 적용됨. 이러한 문제를 해결하는 것은 간단하지 않기 때문에 최신 프레임워크들은 컴포넌트에서 직접 Effect를 작성하는 것보다 더 효율적인 빌트인 데이터 페칭 메커니즘을 제공.

프레임워크를 사용하지 않고(또한 직접 만들고 싶지 않고) Effect에서 데이터 페칭을 보다 인체공학적으로 만들고 싶다면, 다음 예시처럼 페칭 로직을 커스텀 훅으로 추출하는 것을 고려해 보라.

```javascript
function SearchResults({ query }) {
  const [page, setPage] = useState(1);
  const params = new URLSearchParams({ query, page });
  const results = useData(`/api/search?${params}`);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}

function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    let ignore = false;
    fetch(url)
      .then((response) => response.json())
      .then((json) => {
        if (!ignore) {
          setData(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [url]);
  return data;
}
```

또한 오류 처리와 콘텐츠 로딩 여부를 추적하기 위한 로직을 추가하고 싶을 것. 이와 같은 훅을 직접 빌드하거나 React 에코시스템에서 이미 사용 가능한 많은 솔루션 중 하나를 사용할 수 있음. 이 방법만으로는 프레임워크 빌트인 데이터 페칭 메커니즘을 사용하는 것만큼 효율적이지는 않겠지만, **데이터 페칭 로직을 커스텀 훅으로 옮기면 나중에 효율적인 데이터 페칭 전략을 채택하기가 더 쉬워짐.**

일반적으로 Effects를 작성해야 할 때마다 위의 useData와 같이 좀 더 선언적이고 목적에 맞게 만들어진 API를 사용하여 기능을 커스텀 훅으로 추출할 수 있는지 잘 살펴보라. **컴포넌트에서 원시 useEffect 호출이 적을수록 애플리케이션을 유지 관리하기가 더 쉬워짐.**
