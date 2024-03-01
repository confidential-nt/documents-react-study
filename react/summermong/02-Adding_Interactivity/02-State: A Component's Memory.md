# ✏️ Adding Interactivity : State: A Component's Memory

컴포넌트는 상호작용의 결과로 화면의 내용을 변경해야 하는 경우가 많다.
폼에 입력한 값, 캐러셀의 다음 이미지, 장바구니에 추가된 아이템 등을 기억해야 하는데, 이런 종류의 컴포넌트별 메모리를 **state(상태)**라고 한다.

### 어떤 경우에 state가 필요한가?

```
import { sculptureList } from './data.js';

export default function Gallery() {
  let index = 0;

  function handleClick() {
    index = index + 1;
  }

  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleClick}>
        Next
      </button>
      <h2>
        <i>{sculpture.name} </i>
        by {sculpture.artist}
      </h2>
      <h3>
        ({index + 1} of {sculptureList.length})
      </h3>
      <img
        src={sculpture.url}
        alt={sculpture.alt}
      />
      <p>
        {sculpture.description}
      </p>
    </>
  );
}
```

handleClick 이벤트 핸들러가 지역 변수 index를 업데이트 하지만 [지역 변수는 렌더링 사이에 유지되지 않는다] && [지역 변수를 변경해도 렌더링이 발동되지 않는다]는 이유 때문에 변경사항이 업데이트 되지 않는다.

컴포넌트를 새 데이터로 업데이트 하기 위해서는 [렌더링 사이에 데이터를 유지 해야 한다] && [새로운 데이터로 리렌더링이 되어야 한다]는 2가지 작업이 필요하기에 useState를 사용한다.

useState는 렌더링 사이에 데이터를 유지하는 state 변수와 이를 업데이트하고 리렌더링을 야기하는 state 설정자 함수를 제공한다.

### 동작 방식

use-로 시작하는 함수(hook)은 React가 렌더링 중일 때만 사용할 수 있는 특별한 함수이므로 컴포넌트 최상위 레벨(또는 커스텀 훅)에 호출하기 위해 useState를 import 해 컴포넌트의 최상위에서 호출해준다.

useState로 컴포넌트에게 특정 상태를 기억하고 업데이트 할 수 있게 한다.
아래처럼 변수명을 지정하는 것이 관례다.

```
const [index, setIndex] = useState(0);
```

위 코드의 useState의 유일한 인수는 state 변수의 초기값인 0이다.
컴포넌트가 렌더링 될 때마다 useState는 **두 개의 값을 포함하는 배열**을 제공한다.
따라서 컴포넌트 처음 렌더링([0, setIndex]) => state 업데이트 => 업데이트된 값으로 리렌더링하는 방식으로 동작된다.

### state에 대해서

하나의 컴포넌트에 많은 유형의 state 변수를 가질 수 있는데 서로 연관이 없다면 여러 개의 state 변수를 갖는 것이 좋다. 하지만 연관이 있다면 두 변수를 하나로 합치는 것이 더 좋을 수 있다. (추후 읽을 예정)

state는 **컴포넌트의 인스턴스에 지역적이다.**
이 말은 동일한 컴포넌트를 두 군데에서 렌더링하면 각 사본은 완전히 격리된 state를 갖게 됨을 의미한다.
이것이 일반 변수와 state의 차이점으로, state는 특정 함수 호출에도, 코드의 특정 위치에도 얽매이지 않지만 화면상 특정 위치에서 지역적이다.

props와 달리 state는 해당 변수를 선언한 컴포넌트 외에 완전히 비공개이며 부모 컴포넌트에서 변경할 수 없다.
=> state는 컴포넌트 외부에서 직접 접근할 수 없어 내부 상태를 보호하고 부작용 없이 사용할 수 있다.
=> state를 변경하려면 해당 컴포넌트 내 state 설정자 함수를 사용해 변경해야 한다.

만약 다른 두 컴포넌트의 state를 동기화 하려면 자식 컴포넌트에서 state를 제거하고, 두 컴포넌트가 공유하는 가장 가까운 부모 컴포넌트에 추가하여 수행할 수 있다. (부모 컴포넌트에서 state 변경 후 자식 컴포넌트에 props로 전달하면 되기 때문)

---

출처: https://react-ko.dev/learn/state-a-components-memory
