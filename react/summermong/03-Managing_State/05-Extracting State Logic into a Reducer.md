# ✏ Managing State : Extracting State Logic into a Reducer

여러 개의 state 업데이트가 여러 이벤트 핸들러에 분산되어 있는 컴포넌트는 과부하가 걸릴 수 있다.
이 때 reducer라는 단일 함수로 컴포넌트 외부의 모든 state 업데이트 로직을 통합할 수 있다.

## reducer로 state 로직 통합하기

컴포넌트가 복잡해지면 state가 업데이트 되는 경우를 한 눈에 파악하기가 어려워진다.
이런 경우 복잡성을 줄이고 state가 업데이트 되는 모든 로직을 접근하기 쉽게 '한 곳'에 모아 관리하는 게 좋다.
이를 위해 state 로직을 컴포넌트 외부의 reducer라는 단일 함수로 옮길 수 있다.

reducersms state를 관리하는 또 다른 방법으로, useState -> useReducer로의 마이그레이션은 3단계로 진행된다.

1. state 설정을 action 전달로 변경
2. reducer 함수 작성
3. 컴포넌트에서 reducer 사용

```
import { useState } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

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
  {id: 0, text: 'Visit Kafka Museum', done: true},
  {id: 1, text: 'Watch a puppet show', done: false},
  {id: 2, text: 'Lennon Wall pic', done: false},
];
```

이런 코드가 있을 때 useReducer로 마이그레이션을 해보자.

### 1. state 설정을 action 전달로 변경

먼저 모든 state 설정 로직을 제거한다.
handleAddTask, handleChangeTask, handledeleteTask 3개의 이벤트 핸들러에 state 설정 대신 action을 전달한다.
dispatch() 안에 액션 객체를 전달해 아래와 같이 수정한다.

```
function handleAddTask(text) {
  dispatch({
    type: 'added',
    id: nextId++,
    text: text,
  });
}

function handleChangeTask(task) {
  dispatch({
    type: 'changed',
    task: task,
  });
}

function handleDeleteTask(taskId) {
  dispatch({
    type: 'deleted',
    id: taskId,
  });
}
```

### 2. reducer 함수 작성

reducer 함수는 2개의 매개변수를 갖는데. (state, action)
그리고 이 함수는 다음 state를 반환한다.

```
function tasksReducer(tasks, action) {
  if (action.type === 'added') {
    return [
      ...tasks,
      {
        id: action.id,
        text: action.text,
        done: false,
      },
    ];
  } else if (action.type === 'changed') {
    return tasks.map((t) => {
      if (t.id === action.task.id) {
        return action.task;
      } else {
        return t;
      }
    });
  } else if (action.type === 'deleted') {
    return tasks.filter((t) => t.id !== action.id);
  } else {
    throw Error('Unknown action: ' + action.type);
  }
}
```

taskReducer라는 함수는 tasks라는 state와 action 객체를 매개 변수로 갖고 있다.
만약 action의 type이 'added'라면 어떤 배열을 반환하고, 'changed'라면 또 다른 배열을 반환하는 형태다.
이 reducer 함수는 state를 매개변수로 갖기 때문에 컴포넌트 밖에서 선언할 수 있다.

위 코드는 if/else로 구분했지만 switch 구문을 사용하는 게 더 보편적이고 가독성이 좋다.

reduce 연산은 배열을 가지고 많은 값들을 하나의 값으로 누적한다.
reduce로 넘기는 함수가 reducer로 알려져 있는데, 지금까지의 결과와 현재의 아이템을 갖고 다음 결과를 반환하는 개념이 유사하다.
React의 reducer도 지금까지의 state와 action을 가지고 다음 state를 반환하고, action들을 state로 모으게 되기 때문이다.

### 3. 컴포넌트에서 reducer 사용하기

useReducer hook을 import하고 useState 대신 useReducer로 바꾸어준다.
useReducer는 reducer 함수와 초기 state를 파라미터로 받아 state 값과 dispatch 함수를 반환한다.

const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
이런 식으로 tasksReducer라는 reducer 함수, 초기 state인 initialTasks를 매개변수로 사용한다.

```
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
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
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: 'Visit Kafka Museum', done: true},
  {id: 1, text: 'Watch a puppet show', done: false},
  {id: 2, text: 'Lennon Wall pic', done: false},
];
```

## useState와 useReducer 비교하기

- 코드 크기: useReducer를 사용하면 reducer 함수와 action 전달 부분을 모두 적어야 해서 코드가 길어진다. 하지만 많은 이벤트 핸들러가 비슷한 방식으로 state를 업데이트 한다면 useReducer가 더 효율적일 수 있다.

- 가독성: 간단히 state를 업데이트 한다면 useState가 더 좋겠지만, 업데이트 로직이 복잡해지면 가독성이 매우 나빠진다. 이 경우 useReducer가 훨씬 깔끔하게 코드를 관리할 수 있다.

- 디버깅: useState에 버그가 있으면 디버깅하기 어렵지만 useReducer는 reducer에 콘솔 로그를 추가해 디버깅할 수 있다. (물론 더 많은 코드를 살펴봐야 한다.)

- 테스팅: reducer는 컴포넌트에 의존하지 않는 순수함수로, 복잡한 state 업데이트 로직의 경우 reducer가 특정 초기 state와 action에 대해 특정 state을 반환한다고 단언할 수 있다.

- 개인 취향: 사바사

결론적으로 무조건적으로 무언가 더 낫다는 아니며 필요에 따라 사용하면 된다.

## 그외

useReducer를 잘 작성하기 위해서는 1. 반드시 순수해야 하며 2. 각 action은 여러 데이터가 변경되더라도 하나의 사용자 상호작용을 설명해야 한다.
Immer 라이브러리로 reducer를 더 간결하게 만들 수 있다.

---

참고
https://react-ko.dev/learn/extracting-state-logic-into-a-reducer
