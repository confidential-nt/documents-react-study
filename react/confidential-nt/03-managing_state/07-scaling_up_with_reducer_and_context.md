# Scaling Up with Reducer and Context

## 들어가면서

Reducer를 사용하면 컴포넌트의 state 업데이트 로직을 통합할 수 있음. Context를 사용하면 다른 컴포넌트들에 정보를 전달할 수 있음.
Reducer와 context를 함께 사용하여 복잡한 화면의 state를 관리할 수 있음.

## reducer와 context를 결합하기

Reducer의 개요에서 reducer로 state를 관리하는 방법에 대해 알아보았었음.

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
      <h1>Day off in Kyoto</h1>
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
  { id: 0, text: "Philosopher’s Path", done: true },
  { id: 1, text: "Visit the temple", done: false },
  { id: 2, text: "Drink matcha", done: false },
];
```

Reducer를 사용하면 이벤트 핸들러를 간결하고 명확하게 만들 수 있음. 그러나 앱이 커질수록 다른 어려움에 부딪힐 수 있음. 현재 tasks state와 dispatch 함수는 최상위 컴포넌트인 TaskApp에서만 사용할 수 있음. 다른 컴포넌트들에서 tasks의 리스트를 읽고 변경하려면 prop을 통해 현재 state와 state를 변경할 수 있는 이벤트 핸들러를 명시적으로 전달해야

예를 들어, 아래 TaskApp 컴포넌트에서 TaskList 컴포넌트로 task 리스트와 이벤트 핸들러를 전달

```javascript
<TaskList
  tasks={tasks}
  onChangeTask={handleChangeTask}
  onDeleteTask={handleDeleteTask}
/>
```

그리고 TaskList 컴포넌트에서 Task 컴포넌트로 이벤트 핸들러를 전달

```javascript
<Task task={task} onChange={onChangeTask} onDelete={onDeleteTask} />
```

지금처럼 간단한 예시에서는 잘 동작하지만, 수십 수백 개의 컴포넌트를 거쳐 state나 함수를 전달하기는 쉽지 않음.

그래서 이때는 context를 사용하고 싶을 것. 그러면 반복적인 “prop drilling” 없이 TaskApp 아래의 모든 컴포넌트 트리에서 tasks를 읽고 dispatch 함수를 실행할 수 있음.

Reducer와 context를 결합하는 방법은 아래와 같다.

1. Context를 생성한다.
2. State과 dispatch 함수를 context에 넣는다.
3. 트리 안에서 context를 사용한다.

## Step 1 : Context 생성하기

useReducer훅은 현재 tasks와 업데이트할 수 있는 dispatch 함수를 반환.

트리를 통해 전달하려면, 두 개의 별개의 context를 생성해야

- TasksContext는 현재 tasks 리스트를 제공.
- TasksDispatchContext는 컴포넌트에서 action을 dispatch 하는 함수를 제공.

두 context는 나중에 다른 파일에서 가져올 수 있도록 별도의 파일에서 내보내기

```javascript
import { createContext } from "react";

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

두 개의 context에 모두 기본값을 null로 전달하고 있음. 실제 값은 TaskApp 컴포넌트에서 제공.

## Step 2 : State와 dispatch 함수를 context에 넣기

이제 TaskBoard 컴포넌트에서 두 context를 모두 불러올 수 있음. useReducer()를 통해 반환된 tasks와 dispatch를 받고 아래 트리 전체에 전달.

```javascript
import { TasksContext, TasksDispatchContext } from "./TasksContext.js";

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
  // ...
  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        ...
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```

이제 prop을 통한 전달을 제거할 수 있음.

## Step 3 : 트리 안에서 context 사용하기

이제 tasks 리스트나 이벤트 핸들러를 트리 아래로 전달할 필요가 없음.

```javascript
<TasksContext.Provider value={tasks}>
  <TasksDispatchContext.Provider value={dispatch}>
    <h1>Day off in Kyoto</h1>
    <AddTask />
    <TaskList />
  </TasksDispatchContext.Provider>
</TasksContext.Provider>
```

대신 필요한 컴포넌트에서는 TaskContext에서 task 리스트를 읽을 수 있음.

task 리스트를 업데이트하기 위해서 컴포넌트에서 context의 dispatch 함수를 읽고 호출할 수 있음.

이제 TaskApp 컴포넌트는 자식 컴포넌트에, TaskList는 Task 컴포넌트에 이벤트 핸들러를 전달하지 않을 수 있음. 각 컴포넌트에서 필요한 context를 읽을 수 있습니다.

```javascript
import { useState, useContext } from "react";
import { TasksContext, TasksDispatchContext } from "./TasksContext.js";

export default function TaskList() {
  const tasks = useContext(TasksContext);
  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>
          <Task task={task} />
        </li>
      ))}
    </ul>
  );
}

function Task({ task }) {
  const [isEditing, setIsEditing] = useState(false);
  const dispatch = useContext(TasksDispatchContext);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={(e) => {
            dispatch({
              type: "changed",
              task: {
                ...task,
                text: e.target.value,
              },
            });
          }}
        />
        <button onClick={() => setIsEditing(false)}>Save</button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>Edit</button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={(e) => {
          dispatch({
            type: "changed",
            task: {
              ...task,
              done: e.target.checked,
            },
          });
        }}
      />
      {taskContent}
      <button
        onClick={() => {
          dispatch({
            type: "deleted",
            id: task.id,
          });
        }}
      >
        Delete
      </button>
    </label>
  );
}
```

State와 state를 관리하는 useReducer는 여전히 최상위 컴포넌트인 TaskApp에 있음. 그러나 tasks와 dispatch는 하위 트리 컴포넌트 어디서나 context를 불러와서 사용할 수 있음.

## 하나의 파일로 합치기

반드시 이런 방식으로 작성하지 않아도 되지만, reducer와 context를 모두 하나의 파일에 작성하면 컴포넌트들을 조금 더 정리할 수 있음. 현재, TasksContext.js는 두 개의 context만을 선언하고 있음.

이제 이 파일이 좀 더 복잡해질 예정. Reducer를 같은 파일로 옮기고 TasksProvider 컴포넌트를 새로 선언. 이 컴포넌트는 모든 것을 하나로 묶는 역할을 하게 됨.

1. Reducer로 state를 관리.
2. 두 context를 모두 자식 컴포넌트에 제공.
3. children을 prop으로 받기 때문에 JSX를 전달할 수 있음.

TaskContext.js

```javascript
import { createContext, useReducer } from "react";

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);

export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
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

const initialTasks = [
  { id: 0, text: "Philosopher’s Path", done: true },
  { id: 1, text: "Visit the temple", done: false },
  { id: 2, text: "Drink matcha", done: false },
];
```

이렇게 하면 TaskBoard 컴포넌트에서 복잡하게 얽혀있던 부분을 깔끔하게 정리할 수 있음.

```javascript
import AddTask from "./AddTask.js";
import TaskList from "./TaskList.js";
import { TasksProvider } from "./TasksContext.js";

export default function TaskApp() {
  return (
    <TasksProvider>
      <h1>Day off in Kyoto</h1>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}
s;
```

TasksContext.js에서 context를 사용하기 위한 use 함수들도 내보낼 수 있음.

```javascript
export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}
```

그렇게 만들어진 함수를 사용하여 컴포넌트에서 context를 읽을 수 있음.

```javascript
const tasks = useTasks();
const dispatch = useTasksDispatch();
```

이렇게 하면 동작이 바뀌는 건 아니지만, 다음에 context를 더 분리하거나 함수들에 로직을 추가하기 쉬워짐. 이제 모든 context와 reducer는 TasksContext.js에 있음. **이렇게 컴포넌트들이 데이터를 어디서 가져오는지가 아닌 무엇을 보여줄 것인지에 집중할 수 있도록 깨끗하게 정리할 수 있음.**

TasksProvider를 tasks를 어떻게 관리하는지 알고있는 관리자로 여겨도 됨. useTasks로 tasks를 읽을 수 있고, useTasksDispatch로 컴포넌트들에서 tasks를 업데이트 할 수 있음.

- 참고 : 커스텀 훅 안에서도 useContext 등 다른 Hook을 사용할 수 있음.

앱이 커질수록 context-reducer 조합이 더 많아질 것. 앱을 확장하고 큰 노력 없이 트리 아래에서 데이터에 접근할 수 있도록 state를 끌어올리기 위한 강력한 방법이기 때문.

## 공식 문서 발췌

> React는 변경된 value를 받는 provider부터 시작해서 해당 context를 사용하는 자식들에 대해서까지 전부 자동으로 리렌더링합니다. 이전 값과 다음 값은 >Object.is로 비교합니다. memo로 리렌더링을 건너뛰어도 새로운 context 값을 수신하는 자식들을 막지는 못합니다.

## chatGPT 발췌

> 이러한 동작은 특히 큰 애플리케이션에서 성능 저하의 원인이 될 수 있기 때문에, Context의 사용은 신중하게 고려되어야 합니다. 필요 이상으로 많은 데이터를 Context에 넣거나, 매우 자주 업데이트되는 데이터를 Context에 넣는 것은 피해야 합니다. 대신 필요한 컴포넌트에서만 최소한의 데이터를 사용하거나, 여러 개의 작은 Context로 분할하여 사용하는 방법을 고려할 수 있습니다. 아니면 Context를 사용하는 컴포넌트에 React의 React.memo를 적용해볼 수 있습니다. 이것은 props가 변경되지 않는 한 컴포넌트의 리렌더링을 방지할 수 있습니다. 그러나 이 방법은 Context의 값이 컴포넌트의 props로 직접 전달되지 않는 한 효과적이지 않을 수 있습니다.

개인적인 궁금증: useContext(MyContext) 훅을 호출하지 않고 있더라도 리렌더링 되나요?

> 아니요, useContext(MyContext) 훅을 호출하지 않는 컴포넌트는 Context의 value 변경에 의해 리렌더링되지 않습니다. 컴포넌트가 리렌더링되기 위해서는 해당 컴포넌트가 명시적으로 Context를 구독하고 있어야 하는데, 이는 useContext 훅을 사용하여 Context에 접근하거나, 컴포넌트 내에서 Context.Consumer를 사용하는 경우에 해당합니다. 만약 컴포넌트가 Context를 전혀 사용하지 않는다면, Context의 값이 변경되어도 그 컴포넌트는 영향을 받지 않고 리렌더링되지 않습니다. 이는 Context가 영향을 미치는 것이 구독하는 컴포넌트에만 한정되기 때문입니다.
