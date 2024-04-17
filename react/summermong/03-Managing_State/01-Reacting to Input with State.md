# ✏ Managing State : Reacting to Input with State

React는 선언형 프로그래밍 언어로 UI를 개별적으로 직접 조작하는 대신 컴포넌트가 다양한 상태를 기술하고, 유저의 입력에 반응해 각 상태들 사이를 전환한다.

### 선언형 UI VS 명령형 UI

선언형 UI를 지향하는 React에서는 일일이 UI를 조작하지 않는다.
대신 표시할 내용을 '선언'하면 React가 UI를 업데이트할 방법을 알아낸다.

명령형은 UI를 조작하기 위한 '정확한 지침'을 작성해야 한다.
단점으로는 복잡한 시스템에서는 모든 지침(업데이트)를 확인해야 하므로 관리가 복잡해진다.

따라서 선언형 프로그래밍을 지향해야 한다.

1. 컴포넌트의 다양한 시각적 상태를 식별

- 유저의 입장에서 UI가 가질 수 있는 다양한 상태를 시각화 해야 한다.
  - 비어있음: form의 “Submit”버튼은 비활성화되어 있습니다.
  - 입력중: form의 “Submit”버튼이 활성화되어 있습니다.
  - 제출중: form은 완전히 비활성화되어있고 Spinner가 표시됩니다.
  - 성공시: form 대신 “Thank you”메세지가 표시됩니다.
  - 실패시: ‘입력중’ 상태와 동일하지만 추가로 오류 메세지가 표시됩니다.

```
export default function Form({
  // Try 'submitting', 'error', 'success':
  status = "submitting",
}) {
  if (status === "success") {
    return <h1>That's right!</h1>;
  }
  return (
    <>
      <h2>City quiz</h2>
      <p>
        In which city is there a billboard that turns air into drinkable
        water?
      </p>
      <form>
        <textarea disabled={status === "submitting"} />
        <br />
        <button disabled={status === "empty" || status === "submitting"}>
          Submit
        </button>
        {status === "error" && (
          <p className="Error">
            Good guess but a wrong answer. Try again!
          </p>
        )}
      </form>
    </>
  );
}
```

이렇게 컴포넌트에 시각적 상태가 많은 경우 아래와 같이 한 페이지에 모두 관리하는 것이 편리할 수 있다.

```
import Form from './Form.js';

let statuses = [
  'empty',
  'typing',
  'submitting',
  'success',
  'error',
];

export default function App() {
  return (
    <>
      {statuses.map(status => (
        <section key={status}>
          <h4>Form ({status}):</h4>
          <Form status={status} />
        </section>
      ))}
    </>
  );
}
```

이와 같은 페이지를 living styleguide 또는 storybook이라고 한다.

2. 상태 변화를 촉발하는 요소를 파악

- 사람의 입력(버튼 클릭, 필드 입력 등)과 컴퓨터의 입력(응답 도착, 시간 초과, 이미지 로딩 등)에 대한 응답으로 상태 변경이 일어난다.
- 두 경우 모두 state 변수를 설정해야 UI를 업데이트 할 수 있다.
  - text 입력 변경 시 text box의 콘텐츠 여부에 따라 입력중 또는 다른 값으로 전환한다든지, 네트워크 응답 성공 시 성공 state로 전환한다든지...
- 상태가 어떤 Input으로 인해 다른 상태로 변하는지 flow를 그리면 버그를 줄일 수 있다.

3. useState를 사용하여 메모리의 상태를 표현

- 상태를 useState로 표현하는데, 반드시 필수적인 상태부터 우선적으로 고려한다.
- 상태가 너무 복잡해지면 유지보수 및 관리가 어려워지고, 그로 인해 버그가 발생할 확률이 높아진다.
- 어떤 것부터 해야 하는지 모르겠다면 가능한 모든 시각적 상태를 다 다룰 수 있을 만큼 state를 추가해보고 리팩토링을 해보자!

4. 비필수적인 state 변수를 제거

- 리팩토링의 목표는 state가 유저에게 보여주길 원하는 유효한 UI를 나타내지 않는 경우를 방지한다. (보여주려고 했는데 그 UI가 안 나오는 경우를 방지)
- 리팩토링 시 고려할 3가지 질문
  - 이 상태가 역설을 야기하는가?
  - 다른 상태를 통해 같은 정보를 가져올 수 있는가?
  - 다른 상태의 역을 통해 같은 정보를 가져올 수 있는가?
- 리팩토링을 해도 모순이 발생할 수 있으므로 리듀서를 고려해보자!

5. 이벤트 핸들러를 연결하여 state를 설정

- 상태를 업데이트하는 이벤트 핸들러를 만들어보자!

---

참고
https://react-ko.dev/learn/reacting-to-input-with-state
