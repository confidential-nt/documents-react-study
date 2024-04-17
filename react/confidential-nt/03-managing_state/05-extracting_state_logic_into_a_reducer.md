# Extracting State Logic into a Reducer

## 들어가면서

여러 개의 state 업데이트가 여러 이벤트 핸들러에 분산되어 있는 컴포넌트는 과부하가 걸릴 수 있음.(can get overwhelming. -> 좀 압도적이라서 읽기가 힘들고 파악이 안되고..이렇다는 뜻인듯) 이러한 경우, reducer 라고 하는 단일 함수를 통해 컴포넌트 외부에 모든 state 업데이트 로직을 통합할 수 있음.

## reducer로 state 로직 통합하기

컴포넌트가 복잡해지면 컴포넌트의 state가 업데이트되는 다양한 경우를 한눈에 파악하기 어려워질 수 있음. 예를 들어, 아래의 TaskApp 컴포넌트는 state에 tasks 배열을 보유하고 있으며, 세 가지의 이벤트 핸들러를 사용하여 task를 추가, 제거 및 수정함.

```javascript
import { useState } from "react";
import AddTask from "./AddTask.js";
import TaskList from "./TaskList.js";

export default function TaskApp() {
  const [tasks, setTasks] = useState(initialTasks);

  function handleAddTask(text) {
    setTasks([
      ...tasks,
      {
        id: nextId++,
        text: text,
        done: false,
      },
    ]);
  }

  function handleChangeTask(task) {
    setTasks(
      tasks.map((t) => {
        if (t.id === task.id) {
          return task;
        } else {
          return t;
        }
      })
    );
  }

  function handleDeleteTask(taskId) {
    setTasks(tasks.filter((t) => t.id !== taskId));
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: "Visit Kafka Museum", done: true },
  { id: 1, text: "Watch a puppet show", done: false },
  { id: 2, text: "Lennon Wall pic", done: false },
];
```

각 이벤트 핸들러는 state를 업데이트하기 위해 setTasks를 호출함. **컴포넌트가 커질수록 여기저기 흩어져 있는 state 로직의 양도 늘어남.** **복잡성을 줄이고 모든 로직을 접근하기 쉽게 한 곳에 모으려면, state 로직을 컴포넌트 외부의 reducer라고 하는 단일 함수로 옮길 수 있음.**

Reducer는 state를 관리하는 다른 방법. useState에서 useReducer로 마이그레이션하는 방법은 세 단계로 진행됨.

1. state를 설정하는 것에서 action들을 전달하는 것으로 변경하기
2. reducer 함수 작성하기
3. 컴포넌트에서 reducer 사용하기

## Step 1: state 설정을 action들의 전달로 바꾸기

현재 이벤트 핸들러는 state를 설정하여 수행할 작업(state 설정 logic)을 지정하고 있음.

```javascript
function handleAddTask(text) {
  setTasks([
    ...tasks,
    {
      id: nextId++,
      text: text,
      done: false,
    },
  ]);
}

function handleChangeTask(task) {
  setTasks(
    tasks.map((t) => {
      if (t.id === task.id) {
        return task;
      } else {
        return t;
      }
    })
  );
}

function handleDeleteTask(taskId) {
  setTasks(tasks.filter((t) => t.id !== taskId));
}
```

모든 state 설정 로직을 제거. -> 이제 세 개의 이벤트 핸들러만 남음.

- 사용자가 “Add”를 누르면 handleAddTask(text)가 호출됨.
- 사용자가 task를 토글하거나 “Save”를 누르면 handleChangeTask(task)가 호출됨.
- 사용자가 “Delete”를 누르면 handleDeleteTask(taskId)가 호출됨.

reducer를 사용한 state 관리는 state를 직접 설정하는 것과 약간 다름. **state를 설정하여 React에게 “무엇을 할 지”를 지시하는 대신, 이벤트 핸들러에서 “action”을 전달하여 “사용자가 방금 한 일”을 지정.** (state 업데이트 로직은 다른 곳에 있음!) **즉, 이벤트 핸들러를 통해 “tasks를 설정”하는 대신 “task를 추가/변경/삭제”하는 action을 전달하는 것.** 이러한 방식이 사용자의 의도를 더 명확하게 설명함. (MVI?)

```javascript
function handleAddTask(text) {
  dispatch({
    type: "added",
    id: nextId++,
    text: text,
  });
}

function handleChangeTask(task) {
  dispatch({
    type: "changed",
    task: task,
  });
}

function handleDeleteTask(taskId) {
  dispatch({
    type: "deleted",
    id: taskId,
  });
}
```

dispatch 함수에 넣어준 객체를 “action” 이라고 함. 이 객체는 일반적인 JavaScript 객체. 여기에 무엇을 넣을지는 우리가 결정하지만, 일반적으로 무슨 일이 일어났는지 에 대한 최소한의 정보를 포함해야. (dispatch 함수 자체는 이후 단계에서 추가할 것.)

- 참고: action 객체는 어떤 형태든 될 수 있음.
  - 그렇지만 무슨 일이 일어나는지 설명하는 문자열 타입의 type을 지정하고 추가적인 정보는 다른 필드를 통해 전달하도록 작성하는게 일반적. type은 컴포넌트에 따라 다르므로 이 예에서는 'added' 또는 'added_task'를 사용하면 됨. 무슨 일이 일어나는지를 설명하는 이름을 선택해라.

## Step 2: reducer 함수 작성하기

reducer 함수에 state 로직을 둘 수 있음. 이 함수는 두 개의 매개변수를 가지는데, 하나는 현재 state이고 하나는 action 객체. 그리고 이 함수가 다음 state를 반환함:

```javascript
function yourReducer(state, action) {
  // return next state for React to set
}
```

React는 reducer로부터 반환된 것을 state로 설정할 것.

state를 설정하는 로직을 이벤트 핸들러에서 reducer 함수로 옮기기 위해서 다음과 같이 진행해 보자.

1. 현재의 state(tasks)를 첫 번째 매개변수로 선언.
2. action 객체를 두 번째 매개변수로 선언.
3. 다음 state를 reducer 함수에서 반환. (React가 state로 설정할 것.)

아래는 모든 state 설정 로직을 reducer 함수로 옮긴 내용.

```javascript
function tasksReducer(tasks, action) {
  if (action.type === "added") {
    return [
      ...tasks,
      {
        id: action.id,
        text: action.text,
        done: false,
      },
    ];
  } else if (action.type === "changed") {
    return tasks.map((t) => {
      if (t.id === action.task.id) {
        return action.task;
      } else {
        return t;
      }
    });
  } else if (action.type === "deleted") {
    return tasks.filter((t) => t.id !== action.id);
  } else {
    throw Error("Unknown action: " + action.type);
  }
}
```

reducer 함수는 state(tasks)를 매개변수로 갖기 때문에, 컴포넌트 밖에서 reducer 함수를 선언할 수 있음. 이렇게 하면 들여쓰기 단계도 줄이고 코드를 읽기 쉽게 만들 수 있음.

- 참고: 위에 있던 코드는 if/else 구문을 사용. 그러나 reducer 안에서는 switch 구문을 사용하는 게 일반적.

  - 결과는 똑같지만 switch 구문이 한눈에 봐도 읽기 더 편함.
  - 이 문서의 나머지 부분에서는 switch를 사용해서 reducer 함수를 작성한다.
  - case 블럭을 모두 중괄호 { 와 }로 감싸는 걸 추천. 이렇게 하면 다양한 case들 안에서 선언된 변수들이 서로 충돌하지 않음. 또한, 하나의 case는 보통 return으로 끝나야. 만약 return을 잊는다면 이 코드는 다음 case에 빠지게 될 것이고, 이는 실수로 이어질 수 있음.

- 참고: 왜 reducer라고 부를까요?

  - reducer들이 비록 컴포넌트 안에 있는 코드의 양을 “줄여주긴” 하지만, 이건 사실 배열에서 사용하는 reduce() 연산을 따서 지은 이름.
  - reduce() 연산은 배열을 가지고 많은 값들을 하나의 값으로 “누적”할 수 있음.
  - **reduce로 넘기는 함수가 “reducer”로 알려져 있음.** **지금까지의 결과 와 현재의 아이템 을 가지고, 다음 결과 를 반환.** React reducer는 이 아이디어와 똑같은 예시. React reducer도 지금까지의 state 와 action 을 가지고 다음 state 를 반환. 이런 방식으로 시간이 지나면서 action들을 state로 모으게 됨.
  - 심지어 reduce() 메서드를 initialState와 actions 배열을 사용해서 reducer로 최종 state를 계산할 수도 있음:

    ```javascript
    import tasksReducer from "./tasksReducer.js";

    let initialState = [];
    let actions = [
      { type: "added", id: 1, text: "Visit Kafka Museum" },
      { type: "added", id: 2, text: "Watch a puppet show" },
      { type: "deleted", id: 1 },
      { type: "added", id: 3, text: "Lennon Wall pic" },
    ];

    let finalState = actions.reduce(tasksReducer, initialState);

    const output = document.getElementById("output");
    output.textContent = JSON.stringify(finalState, null, 2);
    ```

  - 이 작업을 직접 할 필요는 없겠지만, 이것은 React가 하는 것과 비슷

## Step 3: 컴포넌트에서 reducer 사용하기

마지막으로, 컴포넌트에 tasksReducer를 연결해야. React에서 useReducer Hook을 import. 그런 다음, useState 대신 useReducer로 바꿔라.

```javascript
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

useReducer Hook은 useState와 비슷. 초기 state 값을 전달해야 하며, 그 결과로 state 값과 state 설정자 함수(useReducer의 경우 dispatch 함수)를 반환. 하지만 조금 다른 점이 있음.

useReducer Hook은 두 개의 인자를 받음.

1. reducer 함수
2. 초기 state

그리고 아래 내용을 반환

1. state값
2. dispatch 함수 (사용자의 action을 reducer에 “전달”해주는 함수)

최종본

```javascript
import { useReducer } from "react";
import AddTask from "./AddTask.js";
import TaskList from "./TaskList.js";

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: "added",
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: "changed",
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: "deleted",
      id: taskId,
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case "added": {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case "changed": {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case "deleted": {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error("Unknown action: " + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: "Visit Kafka Museum", done: true },
  { id: 1, text: "Watch a puppet show", done: false },
  { id: 2, text: "Lennon Wall pic", done: false },
];
```

원한다면, reducer를 다른 파일로 분리하는 것도 가능.

이렇게 관심사를 분리하면 컴포넌트 로직을 더 쉽게 읽을 수 있음. 이제 이벤트 핸들러는 action을 전달하여 무슨 일이 일어났는지 만 지정하고, reducer 함수는 action에 대한 응답으로 state가 어떻게 변경되는지를 결정

## useState와 useReducer 비교하기

Reducer도 좋은 점만 있는 것은 아님. 다음은 useState 와 useReducer 를 비교할 수 있는 몇 가지 방법

- 코드 크기: 일반적으로 useState를 사용하면 미리 작성해야 하는 코드가 줄어듦. useReducer를 사용하면 reducer 함수 와 action을 전달하는 부분 모두 작성해야. **하지만 많은 이벤트 핸들러가 비슷한 방식으로 state를 업데이트하는 경우 useReducer를 사용하면 코드를 줄이는 데 도움이 될 수 있음.**
- 가독성: useState로 간단한 state를 업데이트 하는 경우 가독성이 좋음. 그렇지만 state의 구조가 더욱 복잡해지면, 컴포넌트의 코드의 양이 부풀어 오르고 한눈에 읽기 어려워질 수 있음. 이 경우 useReducer를 사용하면 업데이트 로직이 어떻게 동작 하는지와 이벤트 핸들러를 통해 무엇이 일어났는지 를 깔끔하게 분리할 수 있음.
- 디버깅: useState에 버그가 있는 경우, state가 어디서 잘못 설정되었는지, 그리고 왜 그런지 알기 어려울 수 있음. **useReducer를 사용하면, reducer에 콘솔 로그를 추가하여 모든 state 업데이트와 왜 (어떤 action으로 인해) 버그가 발생했는지 확인할 수 있음.** **각 action이 정확하다면, 버그가 reducer 로직 자체에 있다는 것을 알 수 있습니다. 하지만 useState를 사용할 때보다 더 많은 코드를 살펴봐야.**
- 테스팅: reducer는 컴포넌트에 의존하지 않는 순수한 함수. 즉, 별도로 분리해서 내보내거나 테스트할 수 있음. 일반적으로 보다 현실적인 환경(e2e)에서 컴포넌트를 테스트하는 것이 가장 좋지만, 복잡한 state 업데이트 로직의 경우 reducer가 특정 초기 state와 action에 대해 특정 state를 반환한다고 단언하는 것이 유용할 수 있음.
- 개인 취향: 어떤 사람은 reducer를 좋아하고 어떤 사람은 싫어함. ㅇㅋ. 취향의 문제니까. useState 와 useReducer는 언제든지 앞뒤로 변환할 수 있으며, 서로 동등함!

일부 컴포넌트에서 **잘못된 state 업데이트로 인해 버그가 자주 발생하고 코드에 더 많은 구조를 도입하려는 경우 reducer를 사용하는 것이 좋음.** **모든 컴포넌트에 reducer를 사용할 필요는 없으니 자유롭게 섞어서 사용하라. 심지어 같은 컴포넌트에서 useState와 useReducer를 함께 사용할 수도**

## reducer 잘 작성하기

reducer를 작성할 때 다음 두 개의 팁을 기억

- reducer는 반드시 순수해야. state 설정 함수와 비슷하게, reducer는 렌더링 중에 실행됨! (action은 다음 렌더링까지 대기.) 즉, reducer는 반드시 순수해야. 즉, 입력 값이 같다면 결과 값도 항상 같아야. 요청을 보내거나 timeout을 스케쥴링하거나 사이드 이펙트(컴포넌트 외부에 영향을 미치는 작업)을 수행해서는 안됨. reducer는 객체 및 배열을 변이 없이 업데이트해야.
- 각 action은 여러 데이터가 변경되더라도, 하나의 사용자 상호작용을 설명해야. 예를 들어, 사용자가 reducer가 관리하는 5개의 필드가 있는 양식에서 ‘재설정’을 누른 경우, 5개의 개별 set_field action보다는 하나의 reset_form action을 전송하는 것이 더 합리적. 모든 action을 reducer에 기록하려면 어떤 상호작용이나 응답이 어떤 순서로 일어났는지 재구성할 수 있을 만큼 로그가 명확해야. 이는 디버깅에 도움이 됨!

## Immer를 사용하여 간결한 reducer 작성하기

일반 state의 객체와 배열을 변경할 때와 마찬가지로 Immer 라이브러리를 사용해 reducer를 더 간결하게 만들 수 있음. 여기서 useImmerReducer를 사용하면 push 또는 arr[i] = 할당으로 state를 변이할 수 있음

```javascript
import { useImmerReducer } from "use-immer";
import AddTask from "./AddTask.js";
import TaskList from "./TaskList.js";

function tasksReducer(draft, action) {
  switch (action.type) {
    case "added": {
      draft.push({
        id: action.id,
        text: action.text,
        done: false,
      });
      break;
    }
    case "changed": {
      const index = draft.findIndex((t) => t.id === action.task.id);
      draft[index] = action.task;
      break;
    }
    case "deleted": {
      return draft.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error("Unknown action: " + action.type);
    }
  }
}

export default function TaskApp() {
  const [tasks, dispatch] = useImmerReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: "added",
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: "changed",
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: "deleted",
      id: taskId,
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: "Visit Kafka Museum", done: true },
  { id: 1, text: "Watch a puppet show", done: false },
  { id: 2, text: "Lennon Wall pic", done: false },
];
```

reducer는 순수해야 하므로 state를 변이하지 않아야. 하지만 Immer는 안전하게 변이할 수 있는 특별한 draft 객체를 제공. 내부적으로 Immer는 사용자가 변경한 draft로 state의 복사본을 생성. 이 방식을 통해 **useImmerReducer로 관리되는 reducer는 첫 번째 인수를 변경할 수 있고, state를 반환할 필요가 없음.**
