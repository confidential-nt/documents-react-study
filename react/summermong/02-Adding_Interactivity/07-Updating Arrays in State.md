# ✏️ Adding Interactivity : Updating Arrays in State

JS에서 배열은 변경 가능하지만, state에 저장할 때는 변경이 불가능한 것으로 취급해야 한다.
이는 객체와 마찬가지로 state에 저장된 배열을 업데이트 하려면 변이가 아닌 새 배열을 만들어 사용해야 함을 의미한다.

### 변이 없이 배열 업데이트하기

JS에서 배열은 객체이므로 state 배열은 '읽기 전용'으로 취급해야 한다.
즉 배열 내부의 항목을 재할당하거나, push(), pop()과 같이 배열을 '변이'하는 메서드도 사용해선 안된다.
대신 배열을 업데이트 할 때마다 새 배열을 전달하기 위해 **filter(), map()과 같이 비변이 메서드를 호출해 새 배열을 반환**하면 된다.

|      | 비추천 (배열 직접 변이)      | 추천 (새 배열 반환)        |
| ---- | ---------------------------- | -------------------------- |
| 추가 | push, unshift                | concat, [...arr] 전개 구문 |
| 삭제 | pop, shift, **splice**       | filter, **slice**          |
| 교체 | splice, arr[i]... assignment | map                        |
| 정렬 | reverse, sort                | 배열을 복사한 후 정렬      |

추가: 전개 구문은 배열의 시작(unshift), 끝(push)의 기능 모두 수행할 수 있다.
제거: 배열에서 항목을 제거하는 가장 쉬운 방법은 filtering으로, 해당 항목을 포함하지 않는 새 배열을 생성하는 filter를 사용할 수 있다.
변경: 배열의 일부 또는 모든 항목을 변경할 때 map으로 새로운 배열을 만들 수 있다.
삽입: 시작도 끝도 아닌 특정 위치에 삽입하기 위해 전개 구문과 slice() 메서드를 쓸 수 있다. (조각 내기))

```
...
const initialArtists = [
  { id: 0, name: 'Marta Colvin Andrade' },
  { id: 1, name: 'Lamidi Olonade Fakeye'},
  { id: 2, name: 'Louise Nevelson'},
];

export default function List() {
  const [name, setName] = useState('');
  const [artists, setArtists] = useState(
    initialArtists
  );

  function handleClick() {
    const insertAt = 1; // Could be any index
    const nextArtists = [
      // Items before the insertion point:
      ...artists.slice(0, insertAt),
      // New item:
      { id: nextId++, name: name },
      // Items after the insertion point:
      ...artists.slice(insertAt)
    ];
    setArtists(nextArtists);
    setName('');
  }
```

**예제 확인**

---

참고
https://react-ko.dev/learn/updating-arrays-in-state
