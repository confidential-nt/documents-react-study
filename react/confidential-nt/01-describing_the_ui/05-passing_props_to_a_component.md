## Passing Props to a Component

### 들어가면서

컴포넌트는 props를 통해 통신한다. 부모 컴포넌트는 자식 컴포넌트에게 props를 통해 값을 줄 수 있다.
props는 HTML 어트리뷰트를 생각나게 할 수 있는데, props에는 자바스크립트에서 값으로 취급되는 모든 것이 들어갈 수 있다.

### Passing props to a component

props를 사용하면 부모 컴포넌트와 자식 컴포넌트를 독립적으로 생각할 수 있게 된다. 즉, 부모 컴포넌트 입장에선 자식 컴포넌트가 **props를 어떻게 내부에서 사용하는지 생각할 필요 없이 props의 값을 수정할 수 있으며, 자식 컴포넌트 입장에서도 부모 컴포넌트와 관계없이 자식 컴포넌트 안에서 props를 사용하는 방식을 바꿀 수 있다.**

props는 함수의 인수와 동일한 역할을 한다. **사실 props는 함수 컴포넌트의 유일한 인자다.** React 컴포넌트 함수는 하나의 인자, 즉,props 객체를 받는다.

### Specifying a default value for a prop

```javascript
function Avatar({ person, size = 100 }) {
  // ...
}
```

위와 같이 기본 값 줄 수 있는데, props를 아예 안줬거나 props에 undefined를 줬을때만 작동.

### Forwarding props with the JSX spread syntax

그들의 모든 props를 자식 컴포넌트에 전달할 때, 전개 구문을 통해 props를 전달하는 게 합리적일 수도. 특히나 아래의 경우, Profile 컴포넌트는 props를 직접적으로 사용하지 않기 때문에..더욱 합리적이라 할 수 있다.

```javascript
function Profile(props) {
  return (
    <div className="card">
      <Avatar {...props} />
    </div>
  );
}
```

**남용하지 말아라.** 상황에 따라 조심히 판단하여 제한적으로 사용하라.

### Passing JSX as children

컴포넌트를 유연하게 만들어주는 패턴. 부모 컴포넌트는 children 내부에서 무엇이 렌더링 되는지 알아야 할 필요가 없다.

### How props change over time

**컴포넌트는 시간에 따라 다른 props를 받을 수 있다.** **Props는 항상 고정되어 있지 않다.**(state가 일단 그렇기 때문에...) Props는 컴포넌트의 데이터를 처음에만 반영하는 것이 아니라 **모든 시점에 반영한다.**

리액트는 불변성 표방 -> 컴포넌트가 props를 변경해야 하는 경우 **부모 컴포넌트에 다른 props, 즉,새로운 객체를 전달하도록 “요청”해야 한다.**

“props 변경”을 시도하지 말라. 선택한 색을 변경하는 등 사용자 입력에 반응해야 하는 경우에는 나중에 나올 set state가 필요할 것.
