# ✏️ Adding Interactivity : Updating Objects in State

state는 객체를 포함해 JS의 어떤 값이든 저장할 수 있다.
**하지만 state에 있는 객체를 직접 '변경'해서는 안된다.**
직접 변경하는 것 대신에 복사하여 state를 사용해야 한다.

### 변이란?

JS의 어떤 값이든 state에 저장할 수 있다.
숫자, 문자열, 불리언 값 같은 것은 불변, 즉 변이할 수 업석나 읽기 전용이다.
따라서 다시 렌더링해서 값을 바꿀 수 있다.

```
const [x, setX] = useState(0);
setX(5);
```

이것은 x state를 0에서 5로 변경(새로 할당)한 것이지, 0이라는 숫자 자체를 변경한 것은 아니다. 이처럼 JS에서는 빌트인 원시 자료형 값을 '변경'할 수 없다. 이를 불변성이라고 한다.

그렇다면 객체는 어떨까?

```
const [position, setPosition] = useState({ x: 0, y: 0 });
position.x = 5;
```

객체 자체의 내용을 변경하는 것은 가능하다. 이를 변이(mutation)라고 한다.
state의 객체는 기술적으로 바꿀 수 있지만, 원시 자료형 값과 같이 불변하는 것처럼 취급해야 한다. 즉 객체 역시 교체를 해줘야 한다.

### state는 읽기 전용으로 취급해라

state 설정자 함수를 사용해 객체를 수정하지 않고, 직접 객체에 접근해 수정하게 되면 React는 변경된 부분을 인지하지 못해 렌더링이 발생하지 않는다.

이전에 다뤘다시피 React는 아래와 같은 과정을 거쳐 상태 변화를 감지하고 렌더링을 수행한다.

**[설정자 함수 호출 -> 상태 변경 감지 -> 가상 돔에서 새로운 상태와 이전 상태를 비교해 바뀐 부분 식별 -> 실제 돔에서 그 부분을 업데이트]**

이 때 설정자 함수를 호출해 상태를 변화하지 않으면 바뀐 부분을 인지하지 못하게 되므로 React가 적절한 업데이트를 할 수 없게 된다.

단, state와 무관한 객체 변경은 당연히 가능하다.
변경하는 행위는 이미 state가 있는 기존 객체를 변경할 때만 문제가 된다.
이렇게 방금 생성한 객체를 변경하는 건 다른 코드가 아직 해당 객체를 참조하지 않았기에 === 영향을 미치지 않기에 상관없다.

```
const nextPosition = {};
nextPosition.x = e.clientX;
nextPosition.y = e.clientY;
setPosition(nextPosition);
```

### 그렇다면 어떻게 복사하는가?

**전개 구문을 사용해 객체를 복사한다.**

전개 구문은 얕은 구문으로 한 단계 깊이만 복사한다.
속도는 빠르지만 중첩된 프로퍼티를 업데이트 하기 위해서는 두 번 이상 사용해야 한다.

```
const originalObject = {
  name: 'John',
  age: 30,
  address: {
    city: 'New York',
    country: 'USA'
  }
};

// originalObject를 얕은 복사하여 새로운 객체를 생성
const updatedObject = { ...originalObject };
```

### 중첩된 객체는 어떻게 업데이트하나?

```
// 잘못된 예시

const [person, setPerson] = useState({
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Blue Nana',
    city: 'Hamburg',
    image: 'https://i.imgur.com/Sd1AgUOm.jpg',
  }
});

person.artwork.city = 'New Delhi';

```

```
// 올바른 예시

const nextArtwork = { ...person.artwork, city: 'New Delhi' };
const nextPerson = { ...person, artwork: nextArtwork };
setPerson(nextPerson);

```

또는 immer 라이브러리를 사용하면 된다.
immer에서 제공하는 프록시 draft는 특수한 유형의 객체라 사용자가 수행하는 작업을 기록하기 때문에 자유롭게 수정 할 수 있다. 이는 immer가 내부적으로 draft의 어떤 부분이 바뀌었는지 파악하고 해당 부분을 포함한 새로운 객체를 생성하는 원리로 가능하다.

```
import produce from 'immer';

// 원본 객체
const originalObject = {
  name: 'John',
  age: 30,
  address: {
    city: 'New York',
    country: 'USA'
  }
};

// immer 라이브러리를 사용하여 객체를 업데이트
const updatedObject = produce(originalObject, draft => {
  draft.address.city = 'Los Angeles';
});

console.log(updatedObject);
```

---

참고
https://react-ko.dev/learn/updating-objects-in-state
