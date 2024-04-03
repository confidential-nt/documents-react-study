### 들어가면서

지난번엔 내가 짠 코드를 리팩토링했다. 당시 내 머릿속엔 해당 코드가 어떻게 동작하는지 생생했기 때문에, 테스트 코드를 작성하지 않고 리팩토링을 진행했더랬다. 그러나 이번엔 남이 작성한 코드를 리팩토링 해야했고, 아무것도 없이 쌩으로 리팩토링을 진행하기엔 위험하다고 판단. 따라서 테스트 코드를 작성하고 리팩토링을 하기로 마음 먹었다.

### 이 글을 작성하는 의도

1. 저는 남이 작성한 코드를 리팩토링하기 위해, 이러한 과정을 거쳤습니다. 와 같은 경험 및 문제 해결 과정을 공유하기 위해
2. 역시나 지나가면서 이 글을 읽는 분들의 공짜 피드백을 받기 위해

### jest 대신 vitest를 선택한 이유

공식문서를 읽어보면 vite를 사용한다면, vitest를 사용하는 편이 훨씬 더 매끄럽다고 한다. (설정을 함께 쓸 수 있다는 점 때문에?) 예전에 vite 환경에서 jest로 테스트 한번 해보려다가 고생을 많이 했었던 기억 때문에, 좀 더 매끄럽게 테스트를 하고 싶어서 vitest를 선택했다.

~~근데 사실 지금 와서 생각해보면 jest를 쓰든 vitest를 쓰든 크게 다른 점은 없는 것 같다. 어차피 vite를 사용한다면 CRA와는 다르게 테스트 환경을 하나부터 열까지 다 직접 세팅해야하므로..~~

### 리팩토링 하고 싶었던 부분

기존 Task 컴포넌트는 다음과 같은 코드로 이루어져있다.

```javascript
import { useState, ChangeEvent, useEffect } from "react";
import Checkbox from "@mui/material/Checkbox";
import { MdOutlineCreate, MdOutlineSaveAlt } from "react-icons/md";
import { HiOutlineBackspace, HiOutlineTrash } from "react-icons/hi";
import { TodoItem } from "../../pages/Todo";

type Props = {
  todo: TodoItem,
  onUpdate: (todo: TodoItem) => void,
  onDelete: (todo: TodoItem) => void,
};

function Task({ todo, onUpdate, onDelete }: Props) {
  const { id, title, status } = todo;
  const [isEditing, setIsEditing] = useState(false);
  const [editedContent, setEditedContent] = useState(title);

  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    const updatedStatus = e.target.checked ? "completed" : "active";
    onUpdate({ ...todo, status: updatedStatus });

    const updatedTodo = { ...todo, status: updatedStatus };
    localStorage.setItem(`todo_${id}`, JSON.stringify(updatedTodo));
  };

  const handleDelete = () => onDelete(todo);

  const handleEditChange = (e: ChangeEvent<HTMLInputElement>) => {
    setEditedContent(e.target.value);
  };

  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    const updatedTodo = { ...todo, title: editedContent };
    localStorage.setItem(`todo_${id}`, JSON.stringify(updatedTodo));
    onUpdate(updatedTodo);
    setIsEditing(false);
  };

  useEffect(() => {
    const savedTodo = localStorage.getItem(`todo_${id}`);
    if (savedTodo) {
      const parsedTodo = JSON.parse(savedTodo);
      setEditedContent(parsedTodo.title);
    }
  }, [id]);

  if (isEditing) {
    return (
      <li className="flex justify-between items-center w-30 h-12 shrink-0 border-black border-2 rounded-[15px] shadow-standard bg-white my-4">
        <div className="w-full justify-left mx-4 items-center">
          <form onSubmit={handleSubmit}>
            <input
              type="text"
              value={editedContent}
              onChange={handleEditChange}
              className="w-full h-[2rem] flex p-1 border-none text-gray-400"
            />
          </form>
        </div>
        <span className="flex justify-right items-center">
          <MdOutlineSaveAlt
            type="submit"
            onClick={handleSubmit}
            className="w-6 h-6 shrink-0 mx-1 cursor-pointer"
          />
          <HiOutlineBackspace
            onClick={() => setIsEditing(false)}
            className="w-6 h-6 shrink-0 mx-1 cursor-pointer"
          />
        </span>
      </li>
    );
  }
  return (
    <li className="flex justify-between items-center w-30 h-12 shrink-0 border-black border-2 rounded-[15px] shadow-standard bg-white my-4">
      <div className="flex justify-left items-center">
        <Checkbox
          id={id.toString()}
          checked={status === "completed"}
          onChange={handleChange}
          color="secondary"
          className="cursor-pointer"
        />
        <label
          htmlFor={id.toString()}
          className="flex justify-left items-center"
          style={{
            textDecoration: status === "completed" ? "line-through" : "none",
          }}
        >
          {title}
        </label>
      </div>
      <span className="flex justify-right items-center">
        <MdOutlineCreate
          onClick={() => setIsEditing(true)}
          className="w-6 h-6 shrink-0 mx-1 cursor-pointer"
        />
        <HiOutlineTrash
          onClick={handleDelete}
          className="w-6 h-6 shrink-0 mx-1 cursor-pointer"
        />
      </span>
    </li>
  );
}

export default Task;
```

Task 컴포넌트는 isEditing이 true라면, 사용자가 투두 아이템을 수정할 수 있도록 form을 보여주고 false라면 기존의 사용자가 입력한 투두 아이템을 보여준다.

내가 가장 리팩토링하고 싶었던 부분은

1. isEditing에 따른 분기처리의 단순화
2. 분기처리와 관련하여, 중복되는 코드 최소화

였다.

### 테스트 코드 작성

내가 수정할 부분과 관련하여, 기능들이 제대로 동작하는지 테스트하는 테스트 코드를 다음과 같이 작성하였다.

```javascript
import { afterEach, describe, expect, it, vi } from "vitest";
import { render, screen } from "@testing-library/react";
import { userEvent } from "@testing-library/user-event";
import Task from "../Task";
import { TodoItem } from "../../../types/Todo.types";

const TODO_TITLE = "할일 목록 항목";

const todo = {
  id: 1,
  title: TODO_TITLE,
  status: "active",
  start: new Date("2024-03-01"),
  end: new Date("2024-03-15"),
} as TodoItem;

describe("Task Test", () => {

  const onDelete = () => {}
  const onUpdate = () => {}

  it("스냅샷 테스트", () => {
    const { asFragment } = renderTask();
    expect(asFragment()).toMatchSnapshot();
  });

  it("할 일이 표시되어야한다", () => {
    renderTask();
    expect(screen.getByText(TODO_TITLE)).toBeInTheDocument();
  });

  it("수정중일때는 수정중일때의 UI가 보여야한다", async () => {
    renderTask();
    await userEvent.click(screen.getByLabelText("수정 버튼"));
    expect(screen.getByDisplayValue(TODO_TITLE)).toBeInTheDocument();
  });

  function renderTask() {
    return render(<Task todo={todo} onDelete={onDelete} onUpdate={onUpdate} />);
  }
});

```

위와 같은 테스트 코드를 작성해야겠다고 생각한 이유는 수정해야할 부분이 isEditing과 관련된 분기처리 구문인데, 이것들이 리팩토링 후에도 정상적으로 동작한다면 isEditing이 false일때는 투두아이템을, isEditing이 true일때는 투두 아이템의 타이틀을 value로 가지는 input을 띄워야하기 때문이다.

### 리팩토링 결과

위와 같은 테스트 코드를 작성했고, 테스트 코드가 없을 때보다 훨씬 안정감과 자신감을 가진 채 다음과 같이 코드를 리팩토링할 수 있었다.

Task.tsx

```javascript
import { useState, ChangeEvent, useEffect } from "react";
import TaskContent from "./TaskContent";
import TaskForm from "./TaskForm";
import { TodoItem } from "../../types/Todo.types";

type Props = {
  todo: TodoItem,
  onUpdate: (todo: TodoItem) => void,
  onDelete: (todo: TodoItem) => void,
};

function Task({ todo, onUpdate, onDelete }: Props) {
  const { id, title } = todo;
  const [isEditing, setIsEditing] = useState(false);
  const [editedContent, setEditedContent] = useState(title);

  const handleChange = (e: ChangeEvent<HTMLInputElement>) => {
    const updatedStatus = e.target.checked ? "completed" : "active";
    onUpdate({ ...todo, status: updatedStatus });

    const updatedTodo = { ...todo, status: updatedStatus };
    localStorage.setItem(`todo_${id}`, JSON.stringify(updatedTodo));
  };

  const handleDelete = () => onDelete(todo);

  const handleEditChange = (e: ChangeEvent<HTMLInputElement>) => {
    setEditedContent(e.target.value);
  };

  const handleSubmit = (event: React.FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    const updatedTodo = { ...todo, title: editedContent };
    localStorage.setItem(`todo_${id}`, JSON.stringify(updatedTodo));
    onUpdate(updatedTodo);
    setIsEditing(false);
  };

  useEffect(() => {
    const savedTodo = localStorage.getItem(`todo_${id}`);
    if (savedTodo) {
      const parsedTodo = JSON.parse(savedTodo);
      setEditedContent(parsedTodo.title);
    }
  }, [id]);

  return (
    <li className="flex justify-between items-center w-30 h-12 shrink-0 border-black border-2 rounded-[15px] shadow-standard bg-white my-4">
      // 문제의 부분. 분기처리를 단순화 시키고 중복되는 부분을 최소화했다.
      {isEditing ? (
        <TaskForm
          onEditChange={handleEditChange}
          onSubmit={handleSubmit}
          onChangeEditState={() => setIsEditing(false)}
          editedContent={editedContent}
        />
      ) : (
        <TaskContent
          {...todo}
          onChange={handleChange}
          onChangeEditState={() => setIsEditing(true)}
          onDelete={handleDelete}
        />
      )}
    </li>
  );
}

export default Task;
```

### 테스트 코드를 작성하면서 얻어걸린 것들

#### 타입 수정

테스트 코드를 작성하면서 리팩토링 해야 할 부분을 추가적으로 알아내기도 한다. 테스트 코드를 위한 타입을 파악하는 과정에서, TodoItem과 관련된 타입이 다음과 같은 형태를 가지고 있음을 파악했다.

```javascript
export type TodoItem = {
  id: number,
  title: string,
  status: string,
  start: Date,
  end: Date,
};
```

status는 string과 같은 포괄적인 타입보다는 분명 더 좁은 범위의 타입을 가지고 있는 게 좋을 것이란 판단을 했으며, 실제 코드에서도 그렇게 사용되고 있을 것이라는 생각이 들었다. 기존 코드를 파악한 결과, 역시 status는 "active"와 "completed" 딱 두 가지의 값을 가지고 있었다. 그래서 다음과 같이 타입을 수정했고, 혼란을 방지하기 위해 Todo.type.ts 파일로 따로 분리해주었다.

```javascript
export type TodoItem = {
  id: number,
  title: string,
  status: "completed" | "active",
  start: Date,
  end: Date,
};
```

#### aria-label

리액트 테스팅 라이브러리는 버튼 내부 text content와 같이 비교적 잘 변하지 않는 부분을 사용해 테스트를 하게 해주고, 특히 **사용자의 관점**에서 테스트를 하게 해준다.

테스트를 하면서 한 가지 마주쳤던 문제는, 다음과 같이 Task 내부에 들어있는 삭제 버튼 및 수정 버튼이 button 태그 내부에 감싸져있지 않고, button 태그로 감싸도 해당 두 버튼을 특정할 방법이 이 상태에서는 없다는 것이었다. (+ 뭐하는 버튼인지 설명하지 않기 때문에 웹 접근성 측면에서도 그닥 좋지 않다)

```javascript
<span className="flex justify-right items-center">
  <MdOutlineCreate
    onClick={() => setIsEditing(true)}
    className="w-6 h-6 shrink-0 mx-1 cursor-pointer"
  />
  <HiOutlineTrash
    onClick={handleDelete}
    className="w-6 h-6 shrink-0 mx-1 cursor-pointer"
  />
</span>
```

저 두 버튼을 특정하기 위한 html을 새로 내부에 추가할까 하는 생각이 잠시 들었지만, 리액트 테스팅 라이브러리가 제공하는 api를 떠올리고는 다른 방법을 사용하기로 했다. 리액트 테스팅 라이브러리는 사용자의 관점에서 테스트 할 수 있도록 만든다. 이 사용자의 관점에는 웹 접근성도 포함된다. 따라서 리액트 테스팅 라이브러리의 api 웹 접근성을 높이는 방향으로 코드를 수정하도록 유도한다.

내가 떠올린 api는 screen.getByLabelText 다. 이게 뭐냐면 aria-label에 적힌 값에 따라 엘리먼트를 가져오는 것이다. 두 버튼에 aria-label을 명시하면 추가적인 html 없이 두 버튼을 특정할 수 있고, 웹 접근성도 향상할 수 있게 된다.

다음과 같이 수정했다.

```javascript
<span className="flex justify-right items-center">
  <button aria-label="수정 버튼" onClick={onChangeEditState}>
    <MdOutlineCreate className="w-6 h-6 shrink-0 mx-1 cursor-pointer" />
  </button>
  <button
    aria-label="삭제 버튼"
    className="w-6 h-6 shrink-0 mx-1 cursor-pointer"
    onClick={onDelete}
  >
    <HiOutlineTrash />
  </button>
</span>
```

### 테스트 코드를 작성하면서 마주친 난관

1. vite에서의 vitest에는 toBeInTheDocument가 없다
   toBeInTheDocument라는 api는 jest-dom이 제공하는 것이며, dom과 관련된 테스트를 할 때 수월하게 할 수 있게 하는 matchers들을 제공하는 아이인데, vitest에는 당연히 기본적으로 jest-dom이 없기 때문에 따로 확장을 시켜줘야한다. 테스트 환경을 구성할 때 작성해야하는 setup.ts 파일에서 다음과 같이.

```javascript
import { expect, afterEach } from "vitest";
import { cleanup } from "@testing-library/react";
import * as matchers from "@testing-library/jest-dom/matchers"; // 다른 사람은 어떤지 모르겠지만, default export를 하면 matchers를 인식하지 못해서 named export로 바꿨더니 정상적으로 동작했다.

expect.extend(matchers); // vitest의 expect가 toBeInTheDocument를 사용할 수 있도록 확장.

afterEach(() => {
  cleanup();
});
```

2. 이대로 무사히 테스트 코드 작성하는가 싶더니 toBeInTheDocument의 타입을 인식 못하는 문제
   사실 당연한 문제다. ~~(당시에는 당연한 줄 몰랐지만..)~~ vitest의 expect에는 기본적으로 위와 같은 추가적인 matchers들이 포함되어있지 않다. 그런데 Typescript 환경에서 Typescript는 기본적으로 타입 정의를 참조하여 코드의 타입 검사를 수행한다. 이때 expect 함수와 같은 전역 객체에 새로운 메서드나 속성이 추가되면, 해당 타입 정의를 TypeScript가 인식할 수 있도록 명시해주어야한다. 어떻게 명시하느냐? tsconfig.json 파일의 "types" 속성을 수정해주면 된다. 여기서 "types" 속성의 역할은 TypeScript 컴파일러에게 사용할 타입 정의 파일들의 목록을 알려주는 것이다. 이렇게하면 types 설정을 통해 TypeScript 컴파일러가 @testing-library/jest-dom 라이브러리의 타입 정의를 인식하게 되어, toBeInTheDocument() 함수를 포함한 추가적인 매처들이 타입 검사에서 오류 없이 정상적으로 인식된다.

```javascript
{
  "compilerOptions": {
    "target": "ES2020",
    "types": ["@testing-library/jest-dom"],
      // ...

  }
```

### 테스트 코드 추가

목표는 달성했으나 테스트 코드가 Task가 제공하는 기능들을 모두 테스트 하고 있는가? 에 관한 의문이 들어서 다음처럼, 몇 가지 테스트 코드를 더 추가했다.

```
import { afterEach, describe, expect, it, vi } from "vitest";
import { render, screen } from "@testing-library/react";
import { userEvent } from "@testing-library/user-event";
import Task from "../Task";
import { TodoItem } from "../../../types/Todo.types";

const TODO_TITLE = "할일 목록 항목";

const todo = {
  id: 1,
  title: TODO_TITLE,
  status: "active",
  start: new Date("2024-03-01"),
  end: new Date("2024-03-15"),
} as TodoItem;

describe("Task Test", () => {
  const onDelete = vi.fn();
  const onUpdate = vi.fn();

  afterEach(() => {
    onDelete.mockReset();
    onUpdate.mockReset();
  });

  it("스냅샷 테스트", () => {
    const { asFragment } = renderTask();
    expect(asFragment()).toMatchSnapshot();
  });

  it("할 일이 표시되어야한다", () => {
    renderTask();
    expect(screen.getByText(TODO_TITLE)).toBeInTheDocument();
  });

  it("수정중일때는 수정중일때의 UI가 보여야한다", async () => {
    renderTask();
    await userEvent.click(screen.getByLabelText("수정 버튼"));
    expect(screen.getByDisplayValue(TODO_TITLE)).toBeInTheDocument();
  });

  it("삭제 버튼을 누르면 onDelete 콜백이 호출되어야 한다", async () => {
    renderTask();
    await userEvent.click(screen.getByLabelText("삭제 버튼"));
    expect(onDelete).toBeCalledTimes(1);
  });

  it("체크 박스에 체크하면 onUpdate 콜백이 호출되어야 한다", async () => {
    renderTask();
    await userEvent.click(screen.getByRole("checkbox"));
    expect(onUpdate).toBeCalledTimes(1);
  });

  it("체크 박스에 체크하면 할 일의 status가 'active'에서 'completed'로 바뀌어야한다", async () => {
    renderTask();
    await userEvent.click(screen.getByRole("checkbox"));
    expect(onUpdate).toBeCalledWith({ ...todo, status: "completed" });
  });

  it("할 일의 이름을 수정하면 해당 수정사항이 반영되어야한다", async () => {
    const text = " - 완료";
    renderTask();
    await userEvent.click(screen.getByLabelText("수정 버튼"));
    const input = screen.getByDisplayValue(TODO_TITLE);
    await userEvent.type(input, `${text}{enter}`);
    expect(onUpdate).toBeCalledWith({ ...todo, title: TODO_TITLE + text });
  });

  function renderTask() {
    return render(<Task todo={todo} onDelete={onDelete} onUpdate={onUpdate} />);
  }
});

```

### 마무리

어찌저찌 마무리는 하게 되었지만 몇 가지의 고민이 있다.

1. 테스트 코드 리팩토링에 대한 욕심이 생겼다. swr의 테스트 코드를 보면 component render을 위한 유틸 함수를 따로 만들어놨던데 나중에 내 테스트 코드의 규모가 커진다면 나도 그렇게 해보고 싶다.
2. onDelete, onUpdate를 테스트 코드 상에서 전역적으로 만들 필요가 있나 싶기도 하다. 그냥 it 내부에서 그때 그때 필요할때만 mocking 하고, resetMocks 옵션을 설정 파일에서 따로 준다면 무리없이 동작하지 않을까 싶은데 일단 냅뒀다.
3. 성공하는 케이스만 다루는 테스트 코드는 의미가 없다고 하는데 어떤 실패하는 케이스를 다뤄야할지 고민이다. 역시 테스트 코드 짜는 것은 매우 어려운 것 같다. 테스트 코드를 위한 테스트 코드를 작성하고 있진 않은지 매우 걱정이 된다.

느낀 점도 있다.

테스트 코드를 짜고 리팩토링을 하면서, 기존 코드에 대한 이해가 훨씬 풍부해졌다는 것이다. 기존 코드를 유지보수 해야하는 입장이라면 이와 같은 과정을 왜 거쳐야 하는지 몸소 느낄 수 있었다.
