# ✏️ Adding Interactivity : Responding to Events

React를 사용하면 JSX에 이벤트 핸들러를 추가할 수 있다.
이벤트 핸들러는 유저와의 상호작용에 반응해 발생하는 자체 함수를 말한다.

### 이벤트 핸들러 추가하기

이벤트 핸들러를 추가하기 위해서는 함수를 정의하고, 적절한 JSX 태그에 prop으로 전달한다.

```
export default function Button() {
  function handleClick() {
    alert('You clicked me!');
  }

  return (
    <button onClick={handleClick}>
      Click me
    </button>
  );
}
```

위 예시는 handleClick 함수를 정의하고 <button>에 prop으로 전달한 코드다.
이벤트 핸들러 함수는 대개 컴포넌트 안에 정의되며 handle을 접두사로 사용하는 것이 관례다.

prop으로 전달하는 것 외에도 인라인으로 이벤트 핸들러를 정의할 수 있다.
또는 화살표 함수로 간결하게 정의할 수도 있다.

```
<button onClick={function handleClick() {
  alert('You clicked me!');
}}>
```

```
<button onClick={() => {
  alert('You clicked me!');
}}>
```

단 이렇게 이벤트 핸들러에 전달되는 함수는 호출이 아니라 '전달'되어야 한다.
호출을 하게 되면 지정한 이벤트가 작동될 때가 아닌, 렌더링 중에 함수가 실행되어 버린다.

---

### 이벤트 핸들러에서 props 읽기

이벤트 핸들러는 컴포넌트 내부에서 선언되기 때문에 컴포넌트의 props에 접근할 수 있다.

---

### 이벤트 핸들러를 props로 전달하기

부모 컴포넌트가 자식의 이벤트 핸들러를 지정하고 싶을 때는, 컴포넌트가 부모로부터 받는 prop을 이벤트 핸들러로 전달한다.

```
function Button({ onClick, children }) {
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}

function PlayButton({ movieName }) {
  function handlePlayClick() {
    alert(`Playing ${movieName}!`);
  }

  return (
    <Button onClick={handlePlayClick}>
      Play "{movieName}"
    </Button>
  );
}

function UploadButton() {
  return (
    <Button onClick={() => alert('Uploading!')}>
      Upload Image
    </Button>
  );
}

export default function Toolbar() {
  return (
    <div>
      <PlayButton movieName="Kiki's Delivery Service" />
      <UploadButton />
    </div>
  );
}
```

위 예제에서 Toolbar 컴포넌트는 PlayButton, UploadButton을 렌더링한다.
PlayButton은 handlePlayClick을, UploadButton은 () => alert...을 prop으로 Button에 전달한다.
Button 컴포넌트는 onClick이라는 prop을 받고, 이를 사용해 클릭 시 전달된 함수를 호출하도록 React에 지시한다.

---

### 이벤트 핸들러 props 이름 정하기

기본 제공 컴포넌트는 브라우저 이벤트 이름만 지원하지만, 자체 컴포넌트를 빌드할 때는 이벤트 핸들러 prop의 이름을 원하는 방식으로 지정할 수 있다.

이런 경우 관례적으로 이벤트 핸들러 props는 on 접두사를 붙이고 그 뒤에 대문자가 온다. (ex. onClick)

---

### 이벤트 전파

이벤트 핸들러는 컴포넌트에 있을 수 있는 모든 하위 컴포넌트의 이벤트도 포착한다.
즉 이벤트가 발생한 곳에서 시작해 트리 위로 올라가는 것을 이벤트가 트리 위로 버블, 또는 전파된다고 한다.

```
export default function Toolbar() {
  return (
    <div className="Toolbar" onClick={() => {
      alert('You clicked on the toolbar!');
    }}>
      <button onClick={() => alert('Playing!')}>
        Play Movie
      </button>
      <button onClick={() => alert('Uploading!')}>
        Upload Image
      </button>
    </div>
  );
}
```

위 예제는 두 버튼 중 하나를 클릭하면 해당 버튼의 onClick 실행 후 부모 div의 onClick이 실행되는 코드다.

이벤트가 전파되는 것을 막으려면 e.stopPropation()을 호출하면 된다.
이벤트 핸들러는 **이벤트 객체**를 유일 인수로 받는데, 관례적으로 event의 e라고 부른다.
이 객체를 통해 이벤트에 대한 정보를 읽을 수 있고, 전파를 중지할 수도 있다.
따라서 이벤트가 상위 컴포넌트로 전파하지 못하게 하려면 e.stopPropagation을 호출한다.

---

### 기본 동작 방지

일부 브라우저 이벤트에는 연결된 기본 동작이 있다. (ex. form의 submit)
이런 기본 동작을 방지하기 위해서는 이벤트 객체에서 e.preventDefault()로 기본 동작의 발생을 막을 수 있다.

```
export default function Signup() {
  return (
    <form onSubmit={e => {
      e.preventDefault();
      alert('Submitting!');
    }}>
      <input />
      <button>Send</button>
    </form>
  );
}
```

e.stopPropagation은 위 태그에 연결된 이벤트 핸들러의 실행을 중지하고, e.preventDefault는 몇몇 이벤트의 기본 브라우저 동작을 방지하는 역할을 한다.

---

출처: https://react-ko.dev/learn/responding-to-events
