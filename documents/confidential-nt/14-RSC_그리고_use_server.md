## RSC란?

### 들어가면서

번들링 전에 '별도의 환경'에서 미리 렌더링하는 새로운 유형의 컴포넌트. (아직 실험단계)

이 별도의 환경이란? 서버다. 서버 컴포넌트는 CI 서버에서 빌드 시 한 번 실행하거나 각 요청에 대해 실행할 수 있음.

### 서버 없는 서버 컴포넌트

서버 컴포넌트는 빌드 시점에 실행하여 파일 시스템에서 읽거나 정적 콘텐츠를 가져올 수 있음. 혹은 CMS에서 가져올수도. -> 서버 없어도 되는 경우.

서버 컴포넌트가 없는 경우, 클라이언트의 정적 데이터를 Effect를 사용하여 가져오는 것이 일반적.

```javascript
// bundle.js
import marked from "marked"; // 35.9K (11.2K gzipped)
import sanitizeHtml from "sanitize-html"; // 206K (63.3K gzipped)

function Page({ page }) {
  const [content, setContent] = useState("");
  // NOTE: loads *after* first page render.
  useEffect(() => {
    fetch(`/api/content/${page}`).then((data) => {
      setContent(data.content);
    });
  }, [page]);

  return <div>{sanitizeHtml(marked(content))}</div>;
}
```

```javascript
// api.js
app.get(`/api/content/:page`, async (req, res) => {
  const page = req.params.page;
  const content = await file.readFile(`${page}.md`);
  res.send({ content });
});
```

이 패턴은 사용자가 페이지 수명 동안 변경되지 않는 정적 콘텐츠를 렌더링하기 위해 추가로 75K(gzip)의 라이브러리를 다운로드하고 파싱해야 하며 페이지가 로드된 후 데이터를 가져오기 위한 두 번째 요청을 기다려야 함을 의미.

서버 컴포넌트를 사용하면 빌드 시점에 이러한 컴포넌트를 한 번만 렌더링할 수 있음.

```javascript
import marked from "marked"; // Not included in bundle
import sanitizeHtml from "sanitize-html"; // Not included in bundle

async function Page({ page }) {
  // NOTE: loads *during* render, when the app is built.
  const content = await file.readFile(`${page}.md`);

  return <div>{sanitizeHtml(marked(content))}</div>;
}
```

그런 다음 렌더링된 결과물은 HTML로 변화되어 CDN에 업로드될 수 있겠지.

첫 페이지 로드 시 콘텐츠가 표시(사용자가 빠르게 컨텐츠를 볼 수 있음)되며 번들에는 정적 콘텐츠를 렌더링하는 데 필요한 라이브러리가 포함되지 않는다.

### 서버 있는 서버 컴포넌트

서버 컴포넌트는 서버에서 실행될 수 있으니까 따로 API 빌드할 필요 없이 바로 서버 내부 데이터 레이어에 액세스 가능. 애플리케이션이 번들링되기 전에 렌더링되며, 데이터와 JSX를 클라이언트 컴포넌트에 props로 전달할 수 있음.

서버 컴포넌트가 없으면 Effect에서 클라이언트에서 동적 데이터를 가져오는 것이 일반적.

```javascript
// bundle.js
function Note({ id }) {
  const [note, setNote] = useState("");
  // NOTE: loads *after* first render.
  useEffect(() => {
    fetch(`/api/notes/${id}`).then((data) => {
      setNote(data.note);
    });
  }, [id]);

  return (
    <div>
      <Author id={note.authorId} />
      <p>{note}</p>
    </div>
  );
}

function Author({ id }) {
  const [author, setAuthor] = useState("");
  // NOTE: loads *after* Note renders.
  // Causing an expensive client-server waterfall.
  useEffect(() => {
    fetch(`/api/authors/${id}`).then((data) => {
      setAuthor(data.author);
    });
  }, [id]);

  return <span>By: {author.name}</span>;
}
```

```javascript
// api
import db from "./database";

app.get(`/api/notes/:id`, async (req, res) => {
  const note = await db.notes.get(id);
  res.send({ note });
});

app.get(`/api/authors/:id`, async (req, res) => {
  const author = await db.authors.get(id);
  res.send({ author });
});
```

서버 컴포넌트를 사용하면 데이터를 바로 읽고 컴포넌트에서 렌더링할 수 있음.

```javascript
import db from "./database";

async function Note({ id }) {
  // NOTE: loads *during* render.
  const note = await db.notes.get(id);
  return (
    <div>
      <Author id={note.authorId} />
      <p>{note}</p>
    </div>
  );
}

async function Author({ id }) {
  // NOTE: loads *after* Note,
  // but is fast if data is co-located.
  const author = await db.authors.get(id);
  return <span>By: {author.name}</span>;
}
```

그런 다음 번들러는 데이터, 렌더링된 서버 컴포넌트 및 동적 클라이언트 컴포넌트를 하나의 번들로 결합.

서버 컴포넌트는 서버에서 다시 가져와서 데이터에 액세스하고 다시 렌더링하여 동적으로 만들 수 있음.(SSR, ISR...) 이 새로운 애플리케이션 아키텍처는 서버 중심 멀티 페이지 앱의 단순한 '요청/응답' 정신 모델과 클라이언트 중심 단일 페이지 앱(SPA)의 두 가지 장점을 모두 제공.

### 서버 컴포넌트에 상호작용 더하기

서버 컴포넌트는 브라우저로 전송되지 않으므로 useState와 같은 대화형 API를 사용할 수 없음. 서버 컴포넌트에 상호작용성을 추가하려면 "use client" 지시문을 사용하여 클라이언트 컴포넌트와 함께 컴포넌트를 작성하면 됨.

- 참고:
  - 서버 컴포넌트에 대한 directives 는 없음.
  - 흔히 오해하는 게, "use server"가 서버 컴포넌트에 대한 지시어는 라고 오해하는 것. "use server" 지시어는 서버 액션에 사용됨.

다음 예에서는 state를 사용해 expanded state를 전환하는 Expandable Client Components를 Notes Server Components에서 가져옴.

```javascript
// Server Component
import Expandable from "./Expandable";

async function Notes() {
  const notes = await db.notes.getAll();
  return (
    <div>
      {notes.map((note) => (
        <Expandable key={note.id}>
          <p note={note} />
        </Expandable>
      ))}
    </div>
  );
}
```

```javascript
// Client Component
"use client";

export default function Expandable({ children }) {
  const [expanded, setExpanded] = useState(false);
  return (
    <div>
      <button onClick={() => setExpanded(!expanded)}>Toggle</button>
      {expanded && children}
    </div>
  );
}
```

이것은 먼저 Notes를 서버 컴포넌트로 렌더링한 다음, 번들러에게 클라이언트 컴포넌트인 Expandable에 대한 번들을 만들도록 지시하는 방식으로 작동함. 브라우저에서 클라이언트 컴포넌트는 프롭으로 전달된 서버 컴포넌트의 출력을 볼 수 있음. (서버 컴포넌트가 클라이언트 컴포넌트에게 프롭을 전달할 수 있다는 말)

```javascript
<head>
  <!-- the bundle for Client Components -->
  <script src="bundle.js" />
</head>
<body>
  <div>
    <Expandable key={1}>
      <p>this is the first note</p>
    </Expandable>
    <Expandable key={2}>
      <p>this is the second note</p>
    </Expandable>
    <!--...-->
  </div>
</body>
```

### 서버 컴포넌트로 컴포넌트 비동기화

서버 컴포넌트는 async/await을 사용하여 컴포넌트를 작성하는 새로운 방법을 도입함. 비동기 컴포넌트에서 기다리면 React는 일시 중단하고 promise가 해결될 때까지 기다렸다가 렌더링을 재개. 이는 서버/클라이언트의 경계에서 Suspense에 대한 스트리밍 지원(데이터 받아오는거..)과 함께 작동.

서버에서 프로미스를 생성하고 클라이언트에서 대기할 수도 있음.

```javascript
// Server Component
import db from "./database";

async function Page({ id }) {
  // Will suspend the Server Component.
  const note = await db.notes.get(id);

  // NOTE: not awaited, will start here and await on the client.
  const commentsPromise = db.comments.get(note.id);
  return (
    <div>
      {note}
      <Suspense fallback={<p>Loading Comments...</p>}>
        <Comments commentsPromise={commentsPromise} />
      </Suspense>
    </div>
  );
}
```

```javascript
// Client Component
"use client";
import { use } from "react";

function Comments({ commentsPromise }) {
  // NOTE: this will resume the promise from the server.
  // It will suspend until the data is available.
  const comments = use(commentsPromise);
  return comments.map((commment) => <p>{comment}</p>);
}
```

note 콘텐츠는 페이지가 렌더링하는 데 중요한 데이터이므로 서버에서 기다림. 댓글은 우선 순위가 낮으므로 서버에서 promise를 시작하고 use API를 사용해 클라이언트에서 기다림. 그러면 note 콘텐츠가 렌더링되는 것을 차단하지 않음.

비동기 컴포넌트는 클라이언트에서 지원되지 않으므로 use로 promise를 기다림.

## use server에 대하여(Directives)

자세한 건 공식 문서 참고.

서버 액션을 사용하기 위한 지시문.

서버 액션이란? 클라이언트에서 호출 가능한 서버 측 함수를 지정하는 지시문.

'use server'는 서버 측 파일에서만 사용 가능. 결과 서버 액션은 props을 통해 클라이언트 컴포넌트에 전달할 수 있습니다.

특히 form action과 사용할 때, 코드가 좀 더 간결해질 수 있음.

useTransition과도 합이 좋다고 함.

서버 액션을 사용할 때는 데이터를 모두 신뢰 불가능한 데이터로 취급해야 하며(즉, 보안에 유의.) 들어오는 데이터, 나가는 데이터 모두 직렬 가능해야함.

- 참고: 직렬화 가능하다?(serializable)
  - "직렬화 가능하다"는 데이터를 저장하거나 전송하기 위해 연속적인 데이터 형태로 변환하는 과정을 말함. 예를 들어, 객체, 데이터 구조, 또는 다른 메모리 내용을 파일 형식이나 데이터 스트림으로 변환하여 네트워크를 통해 다른 컴퓨터로 보내거나 나중에 사용하기 위해 저장할 수 있음. 이 과정을 통해 복잡한 데이터 구조나 객체 상태를 파일이나 메모리에 저장한 뒤, 필요할 때 그 상태를 다시 복원(역직렬화)하여 사용할 수 있음. 직렬화는 프로그래밍에서 중요한 개념으로, 데이터의 지속성(persistence)을 관리하거나 객체의 상태를 쉽게 교환할 수 있게 해줌.
  - ex) 예를 들어, 온라인 쇼핑을 하고 장바구니에 상품을 담는다고 생각. 장바구니에 담은 상품의 목록을 다른 페이지로 넘어갈 때도 계속 유지하기 위해서는 서버와 클라이언트 간에 데이터를 전송해야. 이 데이터를 전송하기 위해서는 "장바구니" 객체(상품의 목록, 각 상품의 수량 등의 정보를 포함)를 직렬화 과정을 통해 전송가능한 형태로 변환해야. 이를테면, 이 객체를 문자열 형태의 JSON 포맷으로 변환하는 것이 직렬화의 한 예.
  - JSON 문자열로 변환된 장바구니 데이터는 서버로 보내지거나 브라우저의 로컬 저장소에 저장될 수 있음. 나중에 이 데이터를 다시 사용하려면, JSON 문자열을 원래의 객체 구조로 역직렬화하여 장바구니 페이지에서 다시 활용할 수 있음. 이렇게 데이터를 원활하게 전송하고 저장하는 과정이 직렬화와 역직렬화.

코드 예시 1) 서버 컴포넌트와 함께 사용하기(form)

```javascript
// App.js

async function requestUsername(formData) {
  "use server";
  const username = formData.get("username");
  // ...
}

export default function App() {
  return (
    <form action={requestUsername}>
      <input type="text" name="username" />
      <button type="submit">Request</button>
    </form>
  );
}
```

코드 예시 2) 서버 컴포넌트, 클라이언트 컴포넌트 각각 분리된 곳에서 사용하기(form, 리턴 값 사용하기)

```javascript
// requestUsername.js
"use server";

export default async function requestUsername(formData) {
  const username = formData.get("username");
  if (canRequest(username)) {
    // ...
    return "successful";
  }
  return "failed";
}
```

```javascript
// UsernameForm.js
"use client";

import { useActionState } from "react";
import requestUsername from "./requestUsername";

function UsernameForm() {
  const [state, action] = useActionState(requestUsername, null, "n/a");

  return (
    <>
      <form action={action}>
        <input type="text" name="username" />
        <button type="submit">Request</button>
      </form>
      <p>Last submission request returned: {state}</p>
    </>
  );
}
```

코드 예시 3) form이 아닌 곳에서 사용하기

```javascript
import incrementLike from "./actions";
import { useState, useTransition } from "react";

function LikeButton() {
  const [isPending, startTransition] = useTransition();
  const [likeCount, setLikeCount] = useState(0);

  const onClick = () => {
    startTransition(async () => {
      const currentCount = await incrementLike();
      setLikeCount(currentCount);
    });
  };

  return (
    <>
      <p>Total Likes: {likeCount}</p>
      <button onClick={onClick} disabled={isPending}>
        Like
      </button>;
    </>
  );
}
```

```javascript
// actions.js
"use server";

let likeCount = 0;
export default async function incrementLike() {
  likeCount++;
  return likeCount;
}
```

## Next.js 같은 프레임워크 없이 사용할 수 있는가?

결론: 당연히 그렇다.

그런데 좀 많이 복잡하다. 이 복잡한 일련의 과정을 Next.js가 쉽게 할 수 있도록 감싸준거겠지..

[이 아티클](https://dev.to/one-beyond/react-server-components-without-any-frameworks-5a8p)을 읽으면 대충 어떻게 하는지 알 수 있다. 다음엔 한번 Next.js 없이 서버 컴포넌트 써보는 도전을 해보고 싶기도 하다. 혹은 추가적으로 Next.js 없는 SSR이라던가.. 한번 도전? ㅋㅋ

## 출처

- [React Server Components](<https://react.dev/reference/rsc/server-components#noun-labs-1201738-(2)>)
- [use server](https://react.dev/reference/rsc/use-server)
- [React Server Components without any frameworks](https://dev.to/one-beyond/react-server-components-without-any-frameworks-5a8p)
