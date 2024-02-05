### 들어가면서

취업을 위해 신규 프로젝트보단 기존 프로젝트를 리팩토링하는 걸 더 추천한다는 많은 분들의 조언을 듣고, 부트캠프에서 우수상을 수상한 프로젝트를 리팩토링 해보기로 마음 먹었다.

(사실 기존 프로젝트를 리팩토링 하는 것이 배울 게 더 많을 것 같긴 했다.)

테스트 코드 없이도, 가장 익숙한 내가 개발한 부분을 먼저 리팩토링 하기로 마음 먹었다. 사실 걸리적거리는 부분이 있기도 했고...

그것은 바로 Props Drilling 이었다.

이 프로젝트에서는 디렉토리-파일 구조를 보여주기 위한 TreeView가 필요했는데, 이것을 만드는 과정에서 마감에 쫓겨 정신없이 만들다보니 Props Drilling이 있어도 그냥 눈물을 머금고 무시하고 진행했더랬다.

'언젠간 꼭, 해결해야지!' 다짐했으나 그 언제가 언제가 될지는 몰랐는데... 이렇게 또 여러가지 상황들이 이 문제를 해결하게끔 만든다. ~~(사실 똥은 따끈따끈 할 때 치우는 게 제일인 법이지만)~~

### 이 글을 쓰는 의도

1. 기존 프로젝트의 리팩토링을 어디서부터 시작해야하는지 모르겠는 나 같은 사람들을 위해...'저는 이런 식으로 해결했습니다'와 같은 경험을 공유하기위해.
2. ~~누군가의 무료 조언을 얻기 위해.~~

### 해결하고 싶었던 부분

Memo.tsx

```javascript

import { Suspense, lazy, useCallback, useEffect, useState } from "react";
import {
  Directory,
  Memo as MemoType,
  onCreateArgs,
  onDeleteArgs,
  onMoveArgs,
  onRenameArgs,
} from "../types/Memo.types";
import { useUserContext } from "../context/UserContext";
import { getAllMemoStoreQuery } from "../service/database/api";
import { useLiveQuery } from "dexie-react-hooks";
import TreeViewHOC from "../components/memo/TreeViewHOC";
import useMemoActions from "../hooks/useMemoActions";
import { LuFolderTree } from "react-icons/lu";
import { useAuthContext } from "../context/AuthContext";
import useMemoStore from "../hooks/useMemoStore";
import useMemoActionsInServer from "../hooks/useMemoActionsInServer";
import { AFTER_AUTH_KEY } from "../common/local-storage";

const TextEditor = lazy(() => import("../components/memo/TextEditor"));

export default function Memo() {
  const [isDrawerOpened, setDrawerOpened] = useState(false);

  const [memo, setMemo] = useState<MemoType | null>(null);
  const [directory, setDirectory] = useState<string | null>(null);

  const { tempUserId } = useUserContext();
  const { user } = useAuthContext();
  const { onCreate, onDelete, onMove, onRename } = useMemoActions(directory);
  const {
    onCreate: onCreateInServer,
    onDelete: onDeleteInServer,
    onMove: onMoveInServer,
    onRename: onRenameInServer,
  } = useMemoActionsInServer(directory);

  const memoStore = useLiveQuery(async () => {
    if (tempUserId) {
      const result = await getAllMemoStoreQuery(tempUserId);
      return result;
    }
  }, [tempUserId]) as Directory | undefined;

  const { uploadLocalMemoStoreToServer, memoStoreQuery } = useMemoStore();

  const initializeAppAfterFirstLogin = useCallback(() => {
    const afterAuth = localStorage.getItem(AFTER_AUTH_KEY);

    if (!afterAuth) {
      memoStore &&
        uploadLocalMemoStoreToServer.mutate(
          { memoStore },
          {
            onSuccess: () =>
              localStorage.setItem(AFTER_AUTH_KEY, AFTER_AUTH_KEY),
          }
        );
    }
  }, [memoStore]);

  useEffect(() => {
    if (user) {
      initializeAppAfterFirstLogin();
    }
  }, [initializeAppAfterFirstLogin, user]);

  const toggleDrawer =
    (open: boolean) => (event: React.KeyboardEvent | React.MouseEvent) => {
      if (
        event &&
        event.type === "keydown" &&
        ((event as React.KeyboardEvent).key === "Tab" ||
          (event as React.KeyboardEvent).key === "Shift")
      ) {
        return;
      }

      setDrawerOpened(open);
    };

  const onClickMemo = (memo: MemoType | null) => {
    setMemo(memo);
  };

  const onClickDirectory = (id: string | null) => {
    setDirectory(id);
  };

  const handleCreate = ({ type, ...args }: onCreateArgs) => {
    if (user) {
      onCreateInServer({ type, ...args });
      return null;
    }
    onCreate({ type, ...args });

    return null;
  };
  const handleDelete = ({ ids, nodes }: onDeleteArgs) => {
    if (user) {
      onDeleteInServer({ ids, nodes });
      return;
    }
    onDelete({ ids, nodes });
  };
  const handleRename = ({ id, name, node }: onRenameArgs) => {
    if (user) {
      onRenameInServer({ id, name, node });
      return;
    }
    onRename({ id, name, node });
  };
  const handleMove = ({
    dragIds,
    dragNodes,
    parentId,
    ...args
  }: onMoveArgs) => {
    if (user) {
      onMoveInServer({ dragIds, dragNodes, parentId, ...args });
      return;
    }
    onMove({ dragIds, dragNodes, parentId, ...args });
  };

  return (
    <section className="h-screen pt-3 md:pt-2 md:h-auto md:flex">
      <button
        onClick={toggleDrawer(!isDrawerOpened)}
        className="fixed z-20 bottom-28 left-5 md:hidden bg-violet-700 p-2 rounded-full"
      >
        <LuFolderTree className="text-white" />
      </button>
      <TreeViewHOC
        className="md:hidden"
        onClickDirectory={onClickDirectory}
        onClickMemo={onClickMemo}
        onCreate={handleCreate}
        onDelete={handleDelete}
        onRename={handleRename}
        onMove={handleMove}
        onClose={toggleDrawer(false)}
        onOpen={toggleDrawer(true)}
        open={isDrawerOpened}
        memoStore={memoStoreQuery.data || memoStore}
      />
      <div className="md:grow">
        <Suspense fallback={<p>loading...</p>}>
          <TextEditor memo={memo} />
        </Suspense>
      </div>
      <TreeViewHOC
        className="hidden md:block md:ml-3"
        onClickDirectory={onClickDirectory}
        onClickMemo={onClickMemo}
        onCreate={handleCreate}
        onDelete={handleDelete}
        onRename={handleRename}
        onMove={handleMove}
        onClose={toggleDrawer(false)}
        onOpen={toggleDrawer(true)}
        open={isDrawerOpened}
        memoStore={memoStoreQuery.data || memoStore}
      />
    </section>
  );
}
```

위와 같이 딱 봐도 한눈에 파악하기 어려운 코드가 있다.

다른 건 다 무시하고, onClickDirectory, onClickMemo, onCreate ....와 같이 TreeViewHOC에 전달되는 Props에 주목하자.

TreeViewHOC.tsx

```javascript
import React from "react";
import Drawer from "./drawer/Drawer";
import TreeView from "./TreeView";
import {
  Directory,
  Memo,
  onCreateArgs,
  onDeleteArgs,
  onMoveArgs,
  onRenameArgs,
} from "../../types/Memo.types";
import { IdObj } from "react-arborist/dist/types/utils";

type Props = {
  className: string;
  onClose?: (event: React.KeyboardEvent | React.MouseEvent) => void;
  onOpen?: (event: React.KeyboardEvent | React.MouseEvent) => void;
  open?: boolean;
  memoStore?: Directory;
  onClickMemo: (memo: Memo | null) => void;
  onClickDirectory: (id: string | null) => void;
  onCreate: ({
    parentId,
    index,
    type,
    parentNode,
  }: onCreateArgs) => IdObj | Promise<IdObj | null> | null;
  onDelete: ({ ids, nodes }: onDeleteArgs) => void;
  onRename: ({ id, name, node }: onRenameArgs) => void;
  onMove: ({
    dragIds,
    dragNodes,
    parentId,
    parentNode,
    index,
  }: onMoveArgs) => void;
};

export default function TreeViewHOC({
  className,
  onClose,
  onOpen,
  open,
  memoStore,
  onCreate,
  onDelete,
  onRename,
  onMove,
  onClickDirectory,
  onClickMemo,
}: Props) {
  if (!memoStore) return null;

  if (className.includes("md:hidden") && onClose && onOpen && open) {
    return (
      <Drawer
        onClose={onClose}
        onOpen={onOpen}
        open={open}
        className={className}
      >
        <TreeView
          memoStore={memoStore}
          onCreate={onCreate}
          onDelete={onDelete}
          onRename={onRename}
          onMove={onMove}
          onClickDirectory={onClickDirectory}
          onClickMemo={onClickMemo}
        />
      </Drawer>
    );
  }

  if (className.includes("md:block")) {
    return (
      <TreeView
        memoStore={memoStore}
        className={className}
        onCreate={onCreate}
        onDelete={onDelete}
        onRename={onRename}
        onMove={onMove}
        onClickDirectory={onClickDirectory}
        onClickMemo={onClickMemo}
      />
    );
  }
}
```

TreeViewHOC는 내부적으로 TreeView를 호출하고 있다. TreeView에 TreeViewHOC에 전달되었던 몇몇 Props들이 그대로 전달되고 있는 것을 알 수 있다.

TreeView.tsx

```javascript
import { Tree } from "react-arborist";
import {
  Directory,
  Memo,
  onCreateArgs,
  onDeleteArgs,
  onMoveArgs,
  onRenameArgs,
} from "../../types/Memo.types";
import { transformData } from "../../utils/memo";
import Node from "./node/Node";
import { IdObj } from "react-arborist/dist/types/utils";
import TreePannel from "./TreePannel";

const treeClassname = "tree";

type Props = {
  memoStore: Directory;
  className?: string;
  onClickMemo: (memo: Memo | null) => void;
  onClickDirectory: (id: string | null) => void;
  onCreate: ({
    parentId,
    index,
    type,
    parentNode,
  }: onCreateArgs) => IdObj | Promise<IdObj | null> | null;
  onDelete: ({ ids, nodes }: onDeleteArgs) => void;
  onRename: ({ id, name, node }: onRenameArgs) => void;
  onMove: ({
    dragIds,
    dragNodes,
    parentId,
    parentNode,
    index,
  }: onMoveArgs) => void;
};

export default function TreeView({
  memoStore,
  className,
  onCreate,
  onDelete,
  onRename,
  onMove,
  onClickDirectory,
  onClickMemo,
}: Props) {
  return (
    <>
      {memoStore && (
        <div className={className}>
          <TreePannel onCreate={onCreate} />
          <Tree
            data={(transformData(memoStore) as Directory).children}
            onCreate={onCreate}
            onDelete={onDelete}
            onRename={onRename}
            onMove={onMove}
            className={`${treeClassname}`}
            onClick={(e) => {
              if ((e.target as HTMLElement).classList.contains(treeClassname)) {
                onClickDirectory(null);
              }
            }}
          >
            {(props) => (
              <Node
                {...props}
                node={props.node}
                onClickMemo={onClickMemo}
                onClickDirectory={onClickDirectory}
              />
            )}
          </Tree>
        </div>
      )}
    </>
  );
}
```

그리고 위에서 언급했던 Props들은 TreeView에서 Tree라는 react-arborist의 컴포넌트에 전달되고 있음을 알 수 있다.

보면 알겠지만, 전달되고 있는 Props도 상당히 많고, depth도 꽤 깊다.

구현할 때 당연히 불편함을 느꼈지만, 당장 앱이 깨지는 게 두려워서 개선할 엄두를 내지 못하고 있었다.

### 첫번째 스텝. 작은 똥부터 치우기

명확한 목표가 있었음에도, 막상 리팩토링하려니 어디서부터 해야할지 모르겠어서 막막했다. 그러다가 모 리팩토링 강의에서 작은 것부터 먼저 해결하라고 교육받은 것이 기억 나, 일단 당장에 눈에 거슬리는, 가장 눈에 띄는 더러운 부분부터 치우기로 했다.

Memo.tsx를 보면 알겠지만, 다음과 같이 컴포넌트가 굳이 알 필요 없는 로직들이 늘어서있는 것을 볼 수 있다.

Memo.tsx

```javascript
const { onCreate, onDelete, onMove, onRename } = useMemoActions(directory);
const {
  onCreate: onCreateInServer,
  onDelete: onDeleteInServer,
  onMove: onMoveInServer,
  onRename: onRenameInServer,
} = useMemoActionsInServer(directory); // 사용자 로그인 여부에 따라 메모를 indexedDB or 서버에서 불러온다.

// ....

const handleCreate = ({ type, ...args }: onCreateArgs) => {
  if (user) {
    onCreateInServer({ type, ...args });
    return null;
  }
  onCreate({ type, ...args });

  return null;
};
const handleDelete = ({ ids, nodes }: onDeleteArgs) => {
  if (user) {
    onDeleteInServer({ ids, nodes });
    return;
  }
  onDelete({ ids, nodes });
};
const handleRename = ({ id, name, node }: onRenameArgs) => {
  if (user) {
    onRenameInServer({ id, name, node });
    return;
  }
  onRename({ id, name, node });
};
const handleMove = ({ dragIds, dragNodes, parentId, ...args }: onMoveArgs) => {
  if (user) {
    onMoveInServer({ dragIds, dragNodes, parentId, ...args });
    return;
  }
  onMove({ dragIds, dragNodes, parentId, ...args });
};
```

위 로직들을 전부 useAllActions라는 hooks로 묶어 따로 빼주었다.

useAllActions.tsx

```javascript
import { AuthenticatedUser } from "../types/Auth.types";
import {
  onCreateArgs,
  onDeleteArgs,
  onMoveArgs,
  onRenameArgs,
} from "../types/Memo.types";
import useMemoActions from "./useMemoActions";
import useMemoActionsInServer from "./useMemoActionsInServer";

export default function useAllActions(
  directory: string | null,
  user?: AuthenticatedUser | null
) {
  const { onCreate, onDelete, onMove, onRename } = useMemoActions(directory);
  const {
    onCreate: onCreateInServer,
    onDelete: onDeleteInServer,
    onMove: onMoveInServer,
    onRename: onRenameInServer,
  } = useMemoActionsInServer(directory);

  const handleCreate = ({ type, ...args }: onCreateArgs) => {
    if (user) {
      onCreateInServer({ type, ...args });
      return null;
    }
    onCreate({ type, ...args });

    return null;
  };
  const handleDelete = ({ ids, nodes }: onDeleteArgs) => {
    if (user) {
      onDeleteInServer({ ids, nodes });
      return;
    }
    onDelete({ ids, nodes });
  };
  const handleRename = ({ id, name, node }: onRenameArgs) => {
    if (user) {
      onRenameInServer({ id, name, node });
      return;
    }
    onRename({ id, name, node });
  };
  const handleMove = ({
    dragIds,
    dragNodes,
    parentId,
    ...args
  }: onMoveArgs) => {
    if (user) {
      onMoveInServer({ dragIds, dragNodes, parentId, ...args });
      return;
    }
    onMove({ dragIds, dragNodes, parentId, ...args });
  };

  return {
    handleCreate,
    handleDelete,
    handleRename,
    handleMove,
  };
}
```

그랬더니 이제 Memo.tsx측에서는 CRUD 로직에 대해 알 필요가 없어졌고, 사용자 로그인 여부에 따라 메모를 불러오는 책임에서도 멀어졌다.

다시 Memo.tsx로 돌아가, 사용자가 처음 앱에 로그인 했을 때 앱을 초기화하는 부분도, 굳이 Memo.tsx에 있을 필요는 없다.

Memo.tsx

```javascript
// ...
const initializeAppAfterFirstLogin = useCallback(() => {
  const afterAuth = localStorage.getItem(AFTER_AUTH_KEY);

  if (!afterAuth) {
    memoStore &&
      uploadLocalMemoStoreToServer.mutate(
        { memoStore },
        {
          onSuccess: () => localStorage.setItem(AFTER_AUTH_KEY, AFTER_AUTH_KEY),
        }
      );
  }
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, [memoStore]);

useEffect(() => {
  if (user) {
    initializeAppAfterFirstLogin();
  }
}, [initializeAppAfterFirstLogin, user]);
```

따라서 아래와 같이 함수로 따로 빼주었다.

memo.ts

```javascript
export function initializeAppAfterFirstLogin(
  memoStore?: MemoStore,
  mutateFunction?: MutationFunction
) {
  const afterAuth = localStorage.getItem(AFTER_AUTH_KEY);

  if (!afterAuth) {
    memoStore &&
      mutateFunction &&
      mutateFunction(
        { memoStore },
        {
          onSuccess: () => localStorage.setItem(AFTER_AUTH_KEY, AFTER_AUTH_KEY),
        }
      );
  }
}
```

여기까지 리팩토링하고보니, Memo.tsx에서 또 다른 부분을 hooks로 따로 뺄 수 있지 않을까 싶었다.

그렇게 발견한 부분이 다음과 같이 Dexie.js를 사용하여 indexedDB에서 데이터를 불러오고 있는 부분이었다.

Memo.tsx

```javascript
// ...
const memoStore = useLiveQuery(async () => {
    if (tempUserId) {
      const result = await getAllMemoStoreQuery(tempUserId);
      return result;
    }
  }, [tempUserId]) as Directory | undefined;
```

이것도 따로 hooks로 빼주었다.

useIndexedDBMemoStore.tsx

```javascript
import { useLiveQuery } from "dexie-react-hooks";
import { getAllMemoStoreQuery } from "../service/database/api";
import { Directory } from "../types/Memo.types";

export default function useIndexedDBMemoStore(tempUserId?: string) {
  const memoStore = useLiveQuery(async () => {
    if (tempUserId) {
      const result = await getAllMemoStoreQuery(tempUserId);
      return result;
    }
  }, [tempUserId]) as Directory | undefined;

  return {
    memoStore,
  };
}
```

이렇게까지 하고 나니 이전보다 Memo.tsx가 훨 깔끔해졌고, 코드의 흐름 파악이 더 쉬워졌으며 비로소 길이 보이기 시작했다. **무엇보다 따로 빼둔 hooks들은 이제 재사용이 가능해졌기에, Props Drilling을 막기 위해 어떻게 Context API를 구성할지가 보이기 시작했다.**

### 두번째 스텝. Context API를 통한 Props Drilling 해결

~~요즘엔 Context API 잘 안쓴다고 하던데 잘 모르겠다. 하지만 정말 TreeView에 한해서만 쓰는 props들은 Context API로 해결하는 게 맞는 것 같아서 Context API로 해결했다.~~

TreeViewContext.tsx

```javascript
import React, { createContext, useContext, useState } from "react";
import {
  Directory,
  Memo,
  onCreateArgs,
  onDeleteArgs,
  onMoveArgs,
  onRenameArgs,
} from "../types/Memo.types";
import { useAuthContext } from "./AuthContext";
import useAllActions from "../hooks/useAllActions";

type Props = {
  children: React.ReactElement,
  memoStore?: Directory,
  onClickMemo: (memo: Memo | null) => void,
  onClose: (
    event: React.KeyboardEvent<Element> | React.MouseEvent<Element, MouseEvent>
  ) => void,
  onOpen: (
    event: React.KeyboardEvent<Element> | React.MouseEvent<Element, MouseEvent>
  ) => void,
  open: boolean,
};

type TreeViewContextProps = {
  onCreate: (args: onCreateArgs) => null,
  onDelete: (args: onDeleteArgs) => void,
  onRename: (args: onRenameArgs) => void,
  onMove: (args: onMoveArgs) => void,
  onClickDirectory: (id: string | null) => void,

  memoStore?: Directory,
  onClickMemo: (memo: Memo | null) => void,
  onClose: (
    event: React.KeyboardEvent<Element> | React.MouseEvent<Element, MouseEvent>
  ) => void,
  onOpen: (
    event: React.KeyboardEvent<Element> | React.MouseEvent<Element, MouseEvent>
  ) => void,
  open: boolean,
};

const TreeViewContext =
  createContext <
  TreeViewContextProps >
  {
    onCreate: () => null,
    onDelete: () => {},
    onRename: () => {},
    onMove: () => {},
    onClickDirectory: () => {},
    onClickMemo: () => {},
    onClose: () => {},
    onOpen: () => {},
    open: false,
    memoStore: undefined,
  };

export default function TreeViewContextProvider({
  children,
  memoStore,
  onClickMemo,
  onClose,
  onOpen,
  open,
}: Props) {
  // 바깥에서부터 받아야하는 거 빼고 전부 이 안에서...
  const [directory, setDirectory] = (useState < string) | (null > null);
  const { user } = useAuthContext();

  const { handleCreate, handleDelete, handleMove, handleRename } =
    useAllActions(directory, user);

  return (
    <TreeViewContext.Provider
      value={{
        onCreate: handleCreate,
        onDelete: handleDelete,
        onMove: handleMove,
        onRename: handleRename,
        onClickDirectory: (id: string | null) => {
          setDirectory(id);
        },
        onClickMemo,
        onClose,
        onOpen,
        open,
        memoStore,
      }}
    >
      {children}
    </TreeViewContext.Provider>
  );
}

export function useTreeViewContext() {
  return useContext(TreeViewContext);
}
```

위와 같이 Context API를 구성했다. Memo.tsx에서도 사용해야해서 바깥에서 Props를 통해 받아와야하는 Props들 빼고는 전부 TreeViewContext 안으로 끌어왔다.

그랬더니 Memo.tsx가 다음과 같이 비교적 가벼워졌다.

Memo.tsx

```javascript
import { Suspense, lazy, useEffect, useState } from "react";
import { Memo as MemoType } from "../types/Memo.types";
import { useUserContext } from "../context/UserContext";
import TreeViewHOC from "../components/memo/TreeViewHOC";
import { LuFolderTree } from "react-icons/lu";
import { useAuthContext } from "../context/AuthContext";
import useMemoStore from "../hooks/useMemoStore";
import { initializeAppAfterFirstLogin } from "../utils/memo";
import useIndexedDBMemoStore from "../hooks/useIndexedDBMemoStore";
import TreeViewContextProvider from "../context/TreeViewContext";

const TextEditor = lazy(() => import("../components/memo/TextEditor"));

export default function Memo() {
  const [isDrawerOpened, setDrawerOpened] = useState(false);
  const [memo, setMemo] = useState<MemoType | null>(null);

  const { tempUserId } = useUserContext();
  const { user } = useAuthContext();

  const { memoStore } = useIndexedDBMemoStore(tempUserId);

  const { uploadLocalMemoStoreToServer, memoStoreQuery } = useMemoStore();

  useEffect(() => {
    if (user) {
      initializeAppAfterFirstLogin(
        memoStore,
        uploadLocalMemoStoreToServer.mutate
      );
    }
  }, [memoStore, user, uploadLocalMemoStoreToServer]);

  const toggleDrawer =
    (open: boolean) => (event: React.KeyboardEvent | React.MouseEvent) => {
      if (
        event &&
        event.type === "keydown" &&
        ((event as React.KeyboardEvent).key === "Tab" ||
          (event as React.KeyboardEvent).key === "Shift")
      ) {
        return;
      }

      setDrawerOpened(open);
    };

  const onClickMemo = (memo: MemoType | null) => {
    setMemo(memo);
  };

  return (
    <section className="h-screen pt-3 md:pt-2 md:h-auto md:flex">
      <button
        onClick={toggleDrawer(!isDrawerOpened)}
        className="fixed z-20 bottom-28 left-5 md:hidden bg-violet-700 p-2 rounded-full"
      >
        <LuFolderTree className="text-white" />
      </button>
      <TreeViewContextProvider
        memoStore={memoStoreQuery.data || memoStore}
        onClickMemo={onClickMemo}
        onClose={toggleDrawer(false)}
        onOpen={toggleDrawer(true)}
        open={isDrawerOpened}
      >
        <>
          <TreeViewHOC className="md:hidden" />
          <div className="md:grow">
            <Suspense fallback={<p>loading...</p>}>
              <TextEditor memo={memo} />
            </Suspense>
          </div>
          <TreeViewHOC className="hidden md:block md:ml-3" />
        </>
      </TreeViewContextProvider>
    </section>
  );
}
```

그리고 마침내, 내 신경을 거슬리게 하던 Props Driiling도 해결이 됐다.

TreeViewHOC.tsx

```javascript
import Drawer from "./drawer/Drawer";
import TreeView from "./TreeView";
import { useTreeViewContext } from "../../context/TreeViewContext";

type Props = {
  className: string,
};

export default function TreeViewHOC({ className }: Props) {
  const { memoStore, onClose, onOpen, open } = useTreeViewContext();

  if (!memoStore) return null;

  if (className.includes("md:hidden") && onClose && onOpen && open) {
    return (
      <Drawer
        onClose={onClose}
        onOpen={onOpen}
        open={open}
        className={className}
      >
        <TreeView />
      </Drawer>
    );
  }

  if (className.includes("md:block")) {
    return <TreeView className={className} />;
  }
}
```

그렇게 많은 Props를 받아왔었는데 이제 달랑 'className'하나만 받아오는 것에 주목하자!

TreeView도 마찬가지다.

TreeView.tsx

```javascript
import { Tree } from "react-arborist";
import { Directory } from "../../types/Memo.types";
import { transformData } from "../../utils/memo";
import Node from "./node/Node";

import TreePannel from "./TreePannel";
import { useTreeViewContext } from "../../context/TreeViewContext";

const treeClassname = "tree";

type Props = {
  className?: string;
};

export default function TreeView({ className }: Props) {
  const {
    memoStore,
    onClickMemo,
    onClickDirectory,
    onCreate,
    onDelete,
    onMove,
    onRename,
  } = useTreeViewContext();
  return (
    <>
      {memoStore && (
        <div className={className}>
          <TreePannel onCreate={onCreate} />
          <Tree
            data={(transformData(memoStore) as Directory).children}
            onCreate={onCreate}
            onDelete={onDelete}
            onRename={onRename}
            onMove={onMove}
            className={`${treeClassname}`}
            onClick={(e) => {
              if ((e.target as HTMLElement).classList.contains(treeClassname)) {
                onClickDirectory(null);
              }
            }}
          >
            {(props) => (
              <Node
                {...props}
                node={props.node}
                onClickMemo={onClickMemo}
                onClickDirectory={onClickDirectory}
              />
            )}
          </Tree>
        </div>
      )}
    </>
  );
}
```

### 세번째 스텝. 여기서 만족하지 않는다. 나는 Memo.tsx가 아예 TreeView 어쩌구를 몰랐으면 좋겠어!

여기서 만족하지 못했다. Props Drilling을 해결하고보니, 다른 거슬리는 게 생겼다.

Memo.tsx

```javascript
<TreeViewContextProvider
        memoStore={memoStoreQuery.data || memoStore}
        onClickMemo={onClickMemo}
        onClose={toggleDrawer(false)}
        onOpen={toggleDrawer(true)}
        open={isDrawerOpened}
      >
          // .....

```

Memo.tsx 안에서, TreeViewContextProvider와 그 안에 있는 컴포넌트가 너무 뭉탱이로 자리잡고 있다는 것이었다. 이것들을 한꺼번에 옮겨버리고 싶은데 그렇게 할 수 없었다. **왜냐하면 Memo.tsx에는 onClickMemo, onClose, onOpen, open과 같은 내부 상태가 있는데, (각각 TreeView에 있는 메모를 클릭했을 때, 그리고 모바일 화면에서 Drawer가 열렸을때 닫혔을때의 동작에 관여하는 상태들이다.) TreeViewContextProvider와 이 상태는 강하게 커플링되어있다는 문제가 있었다.** 어쩔 수 없이 ContextProvider 안에 넣지 못하고 빼둔 것인데, 달리 방도가 없을까 생각하다가,

그때, 내 머리를 번뜩 스쳐지나가는 게 있었으니...

요즘 좋은 프론트엔드 아키텍쳐에 대해서 공부를 하고 있는데, 공통적으로 나온 내용을 보면 '컴포넌트는 UI에 관해서만 관여하게 하고, 특히나 상태 관련, 상호작용 관련 로직은 따로 빼둬라'이다.

이것을 적용해보기로 했다.

우선 다음과 같이 두 가지 훅을 만들었다. drawer, memo와 관련된 이 상태들은 TreeView 안에서만 쓰는 게 아니라고 생각하여 recoil을 통해 간단하게 구현했다.

useDrawer.tsx

```javascript
import { useRecoilState } from "recoil";
import { drawerState } from "../state/atoms";

export default function useDrawer() {
  const [isDrawerOpened, setDrawerOpened] = useRecoilState(drawerState);

  const toggleDrawer =
    (open: boolean) => (event: React.KeyboardEvent | React.MouseEvent) => {
      if (
        event &&
        event.type === "keydown" &&
        ((event as React.KeyboardEvent).key === "Tab" ||
          (event as React.KeyboardEvent).key === "Shift")
      ) {
        return;
      }

      setDrawerOpened(open);
    };
  return {
    toggleDrawer,
    isDrawerOpened,
  };
}
```

useMemoes.tsx (useMemo와의 혼동을 피하기위해 이렇게 지었다..ㅋ)

```javascript
import { Memo } from "../types/Memo.types";
import { useRecoilState } from "recoil";
import { memoState } from "../state/atoms";

export default function useMemoes() {
  const [memo, setMemo] = (useRecoilState < Memo) | (null > memoState);

  const onClickMemo = (memo: Memo | null) => {
    setMemo(memo);
  };

  return {
    memo,
    onClickMemo,
  };
}
```

이렇게하니, Memo.tsx에서 관련 코드를 모두 지워버릴 수 있었고, 다음과 같이 수정할 수 있었다.

Memo.tsx

```javascript
<TreeViewContextProvider>
// ...
```

그리고 TreeViewContextProvider와 그 안에 있는 children은 이제 자유의 몸이므로 하나의 컴포넌트로 통째로 묶어 추출할 수 있다.

### 최종본

Memo.tsx

```javascript
import { useEffect } from "react";
import { useUserContext } from "../context/UserContext";
import { LuFolderTree } from "react-icons/lu";
import { useAuthContext } from "../context/AuthContext";
import useMemoStore from "../hooks/useMemoStore";
import { initializeAppAfterFirstLogin } from "../utils/memo";
import useIndexedDBMemoStore from "../hooks/useIndexedDBMemoStore";
import useDrawer from "../hooks/useDrawer";
import TreeViewContainer from "../components/memo/TreeViewContainer";

export default function Memo() {
  const { tempUserId } = useUserContext();
  const { user } = useAuthContext();

  const { isDrawerOpened, toggleDrawer } = useDrawer();

  const { memoStore } = useIndexedDBMemoStore(tempUserId);

  const { uploadLocalMemoStoreToServer } = useMemoStore();

  useEffect(() => {
    if (user) {
      initializeAppAfterFirstLogin(
        memoStore,
        uploadLocalMemoStoreToServer.mutate
      );
    }
  }, [memoStore, user, uploadLocalMemoStoreToServer]);

  return (
    <section className="h-screen pt-3 md:pt-2 md:h-auto md:flex">
      <button
        onClick={toggleDrawer(!isDrawerOpened)}
        className="fixed z-20 bottom-28 left-5 md:hidden bg-violet-700 p-2 rounded-full"
      >
        <LuFolderTree className="text-white" />
      </button>
      <TreeViewContainer />
    </section>
  );
}
```

몰라보게 깔끔해진 모습!

### 마무리

이게 맞나? 하는 불안감이 들기도 하지만, 아무튼 내가 원하는대로 당장은 리팩토링이 됐으니 기분이 좋다.

다음엔 내가 작성하지 않은 코드를, 테스트코드와 함께 리팩토링할 예정이다.

지나가시다 이 글을 읽고 어..그거 그렇게 하는 거 아닌데요..하는 생각이 드신다면 부디 태클 걸어주시길 부탁드립니다. 피드백은 언제나 환영입니다.
