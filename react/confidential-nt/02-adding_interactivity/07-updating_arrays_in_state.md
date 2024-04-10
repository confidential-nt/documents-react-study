# Updating Arrays in State

## 들어가면서

JavaScript에서 배열은 변경 가능하지만 state에 저장할 때는 변경이 불가능한 것으로 취급해야.

**객체와 마찬가지**로, state에 저장된 배열을 업데이트하려면, 새로운 배열을 만들고(또는 기존 배열을 복사본을 만듦) 새 배열을 사용하도록 state를 설정해야.

## 변이 없이 배열 업데이트하기

JavaScript에서 배열은 객체의 또 다른 종류일 뿐. React state의 배열은 읽기 전용으로 취급해야함. 객체와 마찬가지.

즉, arr[0] = 'bird'와 같이 배열 내부의 항목을 재할당해서는 안 되며, push() 및 pop()과 같이 배열을 변이하는 메서드도 사용해서는 안됨.

대신 배열을 업데이트할 때마다 state 설정 함수에 새 배열을 전달하고 싶을 것. 그렇게 하려면 state의 원래 배열에서 filter() 및 map()과 같은 비변이 메서드를 호출하여 새 배열을 만들면됨.

React state 내에서 배열을 다룰 때는 **왼쪽 열의 메서드를 피하고** 대신 오른쪽 열의 메서드를 선호해야. 다음을 참고하여 코드를 작성할 것.

피해야할 것 / 권장되는 것
adding: push, unshift / concat, spread syntax
removing: pop, shift, splice / filter, slice
replacing: splice, arr[i] = ...assignment / map
sorting: reverse, sort / 배열을 복사한 다음 처리

또는 두 열의 메서드를 모두 사용할 수 있는 Immer를 사용할 수도.

참고: slice와 splice는 이름이 비슷하지만 매우 다르다.

- slice는 배열 또는 배열의 일부를 복사할 수 있음.
- splice는 배열을 항목을 삽입하거나 삭제하기 위해 변이. => 권장되지 않음.
- 두 개 안헷갈리도록 조심.

## 배열에 추가하기

push 와 같이 변이시키는 거 말고, 전개 구문 같은 거 사용하라는 뜻. 전개구문 하나로 push(끝에 추가) / unshift(앞에 추가) 하는 거 다 수행가능.

## 배열에서 제거하기

filter 사용해라.

## 배열 변경하기

map 사용해라.

## 배열에서 항목 교체하기

ar[0] = 'bird'와 같은 할당 쓰지 말고 map 써라.

map 호출 내에서 두 번째 인수로 항목의 인덱스를 받게됨. 이를 사용하여 원래 항목(첫 번째 인수)을 반환할지 아니면 다른 것을 반환할지 결정할 수 있음.

## 배열에 삽입(insert)하기

**때로는 시작도 끝도 아닌 특정 위치에 항목을 삽입하고 싶을 때가 있음.** 이를 위해 ... 배열 전개 구문과 slice() 메서드를 함께 사용할 수 있음.

예시)

```javascript
const nextArtists = [
  // Items before the insertion point:
  ...artists.slice(0, insertAt),
  // New item:
  { id: nextId++, name: name },
  // Items after the insertion point:
  ...artists.slice(insertAt),
];
setArtists(nextArtists);
```

이 예에서 삽입하려는 요소는 항상 인덱스 1에 삽입됨.

## 배열에 다른 변경 사항 적용하기

전개 구문과 map() 및 filter()와 같은 비변이 메서드만으로는 할 수 없는 일이 몇 가지 있음. 예를 들어, 배열을 반전시키거나 정렬하고 싶을 수 있음. JavaScript reverse() 및 sort() 메서드는 원래 배열을 변이하므로 직접 사용할 수 없음.

대신, 배열을 먼저 복사한 다음 변이하면 됨.

예시)

```javascript
const nextList = [...list];
nextList.reverse();
setList(nextList);
```

복사본이 생겼으므로 nextList.reverse() 또는 nextList.sort()와 같은 변이 메서드를 사용하거나 **nextList[0] = "something"으로 개별 항목을 할당할 수도 있음.**

그러나 배열을 복사하더라도 배열 내부의 기존 항목을 직접 변이할 수는 없음. **이는 얕은 복사가 이루어져 새 배열에는 원래 배열과 동일한 항목이 포함되기 때문.** 따라서 복사된 배열 내부의 객체를 수정하면 기존 state를 변이하는 것.

다음과 같은 코드가 문제가 됨.

```javascript
const nextList = [...list];
nextList[0].seen = true; // Problem: mutates list[0]
setList(nextList);
```

nextList 와 list는 서로 다른 배열이지만, **nextList[0]**과 **list[0]**은 같은 객체를 가리킴. 따라서 nextList[0].seen을 변경하면 list[0].seen도 변경하는 것입니다. 이것은 state를 변이하므로 피해야.

중첩된 JavaScript 객체 업데이트와 비슷한 방법으로 이 문제를 해결할 수 있음. 변경하려는 개별 항목을 변이하는 대신 복사하는 것.

## 배열 내부의 객체 업데이트하기

객체는 실제로 배열 “내부”에 위치하지 않는다. 코드에서는 “내부”에 있는 것처럼 보일 수 있지만 배열의 각 객체는 배열이 “가리키는” 별도의 값. 그렇기 때문에 list[0]과 같이 중첩된 필드를 변경할 때 주의해야. 다른 사람의 작품 목록이 배열의 동일한 요소를 가리킬 수 있음.

**중첩된 state를 업데이트할 때는 업데이트하려는 지점부터 최상위 수준까지 복사본을 만들어야 함.**

다음 예에서는 두 개의 개별 작품 목록의 초기 state가 동일. 두 목록은 분리되어야 하지만 변이로 인해 state가 실수로 공유되어 한 목록의 체크박스를 선택하면 다른 목록에 영향을 미침.

즉, 원치않는 버그가 발생하는 것.

```javascript
import { useState } from "react";

let nextId = 3;
const initialList = [
  { id: 0, title: "Big Bellies", seen: false },
  { id: 1, title: "Lunar Landscape", seen: false },
  { id: 2, title: "Terracotta Army", seen: true },
];

export default function BucketList() {
  const [myList, setMyList] = useState(initialList);
  const [yourList, setYourList] = useState(initialList);

  function handleToggleMyList(artworkId, nextSeen) {
    const myNextList = [...myList];
    const artwork = myNextList.find((a) => a.id === artworkId);
    artwork.seen = nextSeen;
    setMyList(myNextList);
  }

  function handleToggleYourList(artworkId, nextSeen) {
    const yourNextList = [...yourList];
    const artwork = yourNextList.find((a) => a.id === artworkId);
    artwork.seen = nextSeen;
    setYourList(yourNextList);
  }

  return (
    <>
      <h1>Art Bucket List</h1>
      <h2>My list of art to see:</h2>
      <ItemList artworks={myList} onToggle={handleToggleMyList} />
      <h2>Your list of art to see:</h2>
      <ItemList artworks={yourList} onToggle={handleToggleYourList} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map((artwork) => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={(e) => {
                onToggle(artwork.id, e.target.checked);
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}
```

문제는 다음 코드에 있음.

```javascript
const myNextList = [...myList];
const artwork = myNextList.find((a) => a.id === artworkId);
artwork.seen = nextSeen; // Problem: mutates an existing item
setMyList(myNextList);
```

어떻게 고칠까? map을 사용하여 이전 항목에 대한 변이 없이 업데이트된 버전으로 대체할 수 있음.

```javascript
setMyList(
  myList.map((artwork) => {
    if (artwork.id === artworkId) {
      // Create a *new* object with changes
      return { ...artwork, seen: nextSeen };
    } else {
      // No changes
      return artwork;
    }
  })
);
```

일반적으로 방금 만든 객체만 변이해야. 이미 있는 state의 객체를 다루는 경우에는 복사본을 만들어야

## Immer로 간결한 업데이트 로직 작성하기

중첩 배열을 변이 없이 업데이트하는 작업은 객체를 다룰 때와 같이 약간 반복적일 수 있음.

- 일반적으로 state를 몇 레벨 이상 깊이 업데이트할 필요는 없음. **state 객체가 매우 깊다면 다르게 재구성하여 평평하게 만드는 것이 좋음**
- state **구조를 변경하고 싶지 않다면, Immer를 사용.**

```javascript
import { useState } from "react";
import { useImmer } from "use-immer";

let nextId = 3;
const initialList = [
  { id: 0, title: "Big Bellies", seen: false },
  { id: 1, title: "Lunar Landscape", seen: false },
  { id: 2, title: "Terracotta Army", seen: true },
];

export default function BucketList() {
  const [myList, updateMyList] = useImmer(initialList);
  const [yourList, updateYourList] = useImmer(initialList);

  function handleToggleMyList(id, nextSeen) {
    updateMyList((draft) => {
      const artwork = draft.find((a) => a.id === id);
      artwork.seen = nextSeen;
    });
  }

  function handleToggleYourList(artworkId, nextSeen) {
    updateYourList((draft) => {
      const artwork = draft.find((a) => a.id === artworkId);
      artwork.seen = nextSeen;
    });
  }

  return (
    <>
      <h1>Art Bucket List</h1>
      <h2>My list of art to see:</h2>
      <ItemList artworks={myList} onToggle={handleToggleMyList} />
      <h2>Your list of art to see:</h2>
      <ItemList artworks={yourList} onToggle={handleToggleYourList} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map((artwork) => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={(e) => {
                onToggle(artwork.id, e.target.checked);
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}
```

Immer를 사용하면 이제 artwork.seen = nextSeen과 같은 변이도 괜찮.

마찬가지로 push() 및 pop()과 같은 변이 메서드를 draft의 콘텐츠에 적용할 수 있음.

백그라운드에서 Immer는 항상 사용자가 draft에 적용한 변경 사항에 따라 다음 state를 처음부터 다시 구성함. 이렇게 하면 이벤트 핸들러가 state를 변이하지 않고도 매우 간결하게 유지됨.
