배열은 Javascript에서 변경이 가능하지만, state로 저장할 때에는 변경할 수 없도록 처리해야 한다.

객체와 마찬가지로, state에 저장된 배열을 업데이트 하고 싶을 땐, 새 배열을 생성(기존 배열의 복사본을 생성) 한 뒤, 이 새 배열을 state로 두어 업데이트해야 합니다.

학습 내용

- React State에서 배열의 항목을 추가, 삭제 또는 변경하는 방법
- 배열 내부의 객체를 업데이트하는 방법
- Immer로 덜 반복해서 배열을 복사하는 방법

## 변경하지 않고 배열 업데이트 하기

- Javascript에서 배열은 다른 종류의 객체.
- 객체와 마찬가지로 react state에서 배열은 읽기 전용으로 처리해야 한다.
    - arr[0] = ‘bird’ 와 같이 배열 내부의 항목을 재할당해서는 안된다.
    - push() 또는 pop()과 같은 함수로 배열을 변경해서는 안된다.

- 대신 배열을 업데이트 할 때 마다 새로운 배열을 state 함수에 전달해야한다.
- state의 원본 배열을 변경하지 않는 filter(), map()과 같은 함수를 사용하여 원본 배열로부터 새 배열을 만들 수 있다.
- 이후 새 배열들을 state에 설정합니다.

- 아래 배열 연산에 대한 참조표 이다.
    - react state 내에서 배열을 다룰 땐, 왼쪽을 피하고 오른쪽을 선호해야 한다.

|  | 비선호 (배열을 변경) | 선호 (새 배열을 반환) |
| --- | --- | --- |
| 추가 | push, unshift | concat, […arr] 전개 (https://react.dev/learn/updating-arrays-in-state#adding-to-an-array) |
| 제거 | pop, shift, splice | filter, slice (https://react.dev/learn/updating-arrays-in-state#removing-from-an-array) |
| 교체 | splice, arr[i] = ... 할당 | map (https://react.dev/learn/updating-arrays-in-state#replacing-items-in-an-array) |
| 정렬 | reverse, sort | 배열을 복사(https://react.dev/learn/updating-arrays-in-state#making-other-changes-to-an-array) | 최근에 나온 toReversed, toSorted |
- 두 열의 함수를 모두 사용할 수 있도록 하는 immer를 사용할 수 있다.

<aside>
⚠️ 주의! slice와 splice 함수 이름은 비슷하지만 매우 다르다.

</aside>

- slice  ⇒ 배열 또는 그 일부를 복사할 수 있다.
- splice ⇒ 배열을 변경한다. (항목을 추가하거나 제거)
    - react에서, state의 객체나 배열을 변경하지 않는게 좋기 때문에 slice를 자주사용하게 될 것 이다.
    - 객체 업데이트에서 왜 권장하지 않는지 이유를 설명한다. [(이전 챕터)](https://react.dev/learn/updating-objects-in-state)

## 배열에 항목 추가하기

push()는 배열을 변경합니다. (권고하지 않는 방식)

```jsx
import { useState } from 'react';

let nextId = 0;

export default function List() {
  const [name, setName] = useState('');
  const [artists, setArtists] = useState([]);

  return (
    <>
      <h1>Inspiring sculptors:</h1>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={() => {
        setName('');
        artists.push({
          id: nextId++,
          name: name,
        });
      }}>Add</button>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}
```

- 기존에 존재하던 항목들을 뒤에 새 항목을 포함하는 새로운 배열을 만들어 사용해주세요.
    - 아래와 같이 전개(spread)를 사용해서 할 수 있습니다.
    
    ```jsx
    setArtists( // 아래의 새로운 배열로 state를 변경합니다.
      [
        ...artists, // 기존 배열의 모든 항목에,
        { id: nextId++, name: name } // 마지막에 새 항목을 추가합니다.
      ]
    );
    
    // 기존 배열앞에 배치하는 방법
    setArtists([
      { id: nextId++, name: name }, // 추가할 항목을 앞에 배치하고,
      ...artists // 기존 배열의 항목들을 뒤에 배치합니다.
    ]);
    ```
    

## 배열에서 항목 제거하기

- 배열에서 항목을 제거하는 가장 쉬운 방법은 필터링 하고 새 배열을 제공하는 것이다.
- 아래와 같이 filter를 사용 할 수 있다.

```jsx
import { useState } from 'react';

let initialArtists = [
  { id: 0, name: 'Marta Colvin Andrade' },
  { id: 1, name: 'Lamidi Olonade Fakeye'},
  { id: 2, name: 'Louise Nevelson'},
];

export default function List() {
  const [artists, setArtists] = useState(
    initialArtists
  );

  return (
    <>
      <h1>Inspiring sculptors:</h1>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>
            {artist.name}{' '}
            <button onClick={() => {
              setArtists(
                artists.filter(a =>
                  a.id !== artist.id
                )
              );
            }}>
              Delete
            </button>
          </li>
        ))}
      </ul>
    </>
  );
}

// artist.id가 다른 것으로만 남은 artists로 구성된 배열을 새로 생성하고 담는다.
setArtists(
  artists.filter(a => a.id !== artist.id)
);
```

## 배열 반환하기

- 배열의 일부 또는 전체 항목을 변경하고자 한다면, map()을 사용해 새로운 배열을 만들 수 있다.
- map()에 전달할 함수는 데이터나 인덱스를 기반으로 항목을 어떻게 처리할지 결정할 수 있다.

```jsx
import { useState } from 'react';

let initialShapes = [
  { id: 0, type: 'circle', x: 50, y: 100 },
  { id: 1, type: 'square', x: 150, y: 100 },
  { id: 2, type: 'circle', x: 250, y: 100 },
];

export default function ShapeEditor() {
  const [shapes, setShapes] = useState(
    initialShapes
  );

  function handleClick() {
    const nextShapes = shapes.map(shape => {
      if (shape.type === 'square') {
        // 변경시키지 않고 반환합니다.
        return shape;
      } else {
        // 50px 아래로 이동한 새로운 원을 반환합니다.
        return {
          ...shape,
          y: shape.y + 50,
        };
      }
    });
    // 새로운 배열로 리렌더링합니다.
    setShapes(nextShapes);
  }

  return (
    <>
      <button onClick={handleClick}>
        Move circles down!
      </button>
      {shapes.map(shape => (
        <div style={{
          background: 'purple',
          position: 'absolute',
          left: shape.x,
          top: shape.y,
          borderRadius:
            shape.type === 'circle'
              ? '50%' : '',
          width: 20,
          height: 20,
        }} />
      ))}
    </>
  );
}

```

## 배열 내 항목 교체하기

- 배열에서 하나 이상의 항목을 교체하는 경우가 흔하다.
- arr[0] = ‘bird’와 같은 할당은 원본 배열을 변경하기 떄문에, map()을 권장한다.

```jsx
import { useState } from 'react';

let initialCounters = [
  0, 0, 0
];

export default function CounterList() {
  const [counters, setCounters] = useState(
    initialCounters
  );

  function handleIncrementClick(index) {
    const nextCounters = counters.map((c, i) => {
      if (i === index) {
        // 클릭된 counter를 증가시킵니다.
        return c + 1;
      } else {
        // 변경되지 않은 나머지를 반환합니다.
        return c;
      }
    });
    setCounters(nextCounters);
  }

  return (
    <ul>
      {counters.map((counter, i) => (
        <li key={i}>
          {counter}
          <button onClick={() => {
            handleIncrementClick(i);
          }}>+1</button>
        </li>
      ))}
    </ul>
  );
}

```

## 배열에 항목 삽입하기

- 시작도, 끝도 아닌 위치에 항목을 삽입하고 싶을 때 전개 연산자(… ,spread)구문과 slice 함수를 함께 사용할 수 있다.
- slice는 배열의 “일부분”을 잘라 낼 수 있다.

- 예제는 항상 배열의 첫번째에 삽입

```jsx
import { useState } from 'react';

let nextId = 3;
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
    const insertAt = 1; // 모든 인덱스가 될 수 있습니다.
    const nextArtists = [
      // 삽입 지점 이전 항목
      ...artists.slice(0, insertAt),
      // 새 항목
      { id: nextId++, name: name },
      // 삽입 지점 이후 항목
      ...artists.slice(insertAt)
    ];
    setArtists(nextArtists);
    setName('');
  }

  return (
    <>
      <h1>Inspiring sculptors:</h1>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={handleClick}>
        Insert
      </button>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}

```

## 배열에 기타 변경 적용하기

- 전개 구문과 map(), filter() 같은 비-변경 함수들로만으로는 할 수 없는 일이 몇 가지 있다.
- 예를 들어 배열을 뒤집거나 정렬하고 싶을 수 있습니다.
- javaScript의 reverse() 및 sort() 함수는 원본 배열을 변경시키므로 사용할 수 없다.
    - 배열을 복사한 뒤 변경하면 된다.

```jsx
import { useState } from 'react';

const initialList = [
  { id: 0, title: 'Big Bellies' },
  { id: 1, title: 'Lunar Landscape' },
  { id: 2, title: 'Terracotta Army' },
];

export default function List() {
  const [list, setList] = useState(initialList);

  function handleClick() {
	  // 새로운 배열로 복사
    const nextList = [...list];
		//새로운 배열을 거꾸로 뒤집음
    nextList.reverse();
    setList(nextList);
  }

  return (
    <>
      <button onClick={handleClick}>
        Reverse
      </button>
      <ul>
        {list.map(artwork => (
          <li key={artwork.id}>{artwork.title}</li>
        ))}
      </ul>
    </>
  );
}
```

<aside>
⚠️ 주의! 아래와 같이 배열을 복사하더라도, 배열 내부에 기존 항목을 직접 변경 X

</aside>

- `nextList`와 `list`는 서로 다른 배열이지만, `nextList[0]`과 `list[0]`은 동일한 객체를 가리킨다.
- 아래에서 배열 내부의 객체 업데이트 하는 방법을 소개합니다.

```jsx
const nextList = [...list];
nextList[0].seen = true; // 문제: list[0]을 변경시킵니다.
setList(nextList);
```

## 배열 내부의 객체 업데이트 하기

- 객체는 실제로 배열 “내부”에 위치하지 않는다.
- 중첩된 State를 업데이트 할 때, 업데이트 하려는 지점부터 최상위 레벨까지의 복사본을 만들어야 한다.
- 두개의 개별 artwork 목록들은 초기 state가 같다.
- 두 리스트는 분리 되어야 하지만 변경으로 인해 두 목록의 state가 공유 된다.

- 문제의 코드는 어디일까요?

```jsx
import { useState } from 'react';

let nextId = 3;
const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
  { id: 2, title: 'Terracotta Army', seen: true },
];

export default function BucketList() {
  const [myList, setMyList] = useState(initialList);
  const [yourList, setYourList] = useState(
    initialList
  );

  function handleToggleMyList(artworkId, nextSeen) {
    const myNextList = [...myList];
    const artwork = myNextList.find(
      a => a.id === artworkId
    );
    artwork.seen = nextSeen;
    setMyList(myNextList);
  }

  function handleToggleYourList(artworkId, nextSeen) {
    const yourNextList = [...yourList];
    const artwork = yourNextList.find(
      a => a.id === artworkId
    );
    artwork.seen = nextSeen;
    setYourList(yourNextList);
  }

  return (
    <>
      <h1>Art Bucket List</h1>
      <h2>My list of art to see:</h2>
      <ItemList
        artworks={myList}
        onToggle={handleToggleMyList} />
      <h2>Your list of art to see:</h2>
      <ItemList
        artworks={yourList}
        onToggle={handleToggleYourList} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map(artwork => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={e => {
                onToggle(
                  artwork.id,
                  e.target.checked
                );
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

- 문제는 아래의 코드 때문 입니다.

```jsx
const myNextList = [...myList];
const artwork = myNextList.find(a => a.id === artworkId);
artwork.seen = nextSeen; // 문제: 기존 항목을 변경시킵니다.
setMyList(myNextList);
```

- 아래와 같이 바꿔주게 된다면, 서로 배열을 다르게 변경할 수 있다.

```jsx
setMyList(myList.map(artwork => {
  if (artwork.id === artworkId) {
    // 변경된 *새* 객체를 만들어 반환합니다.
    return { ...artwork, seen: nextSeen };
  } else {
    // 변경시키지 않고 반환합니다.
    return artwork;
  }
}));
```

```jsx
const myNextList = [...myList]; // 1 myList[0] myList[1] 얕은복사
const myNextList = myList.map(v => v) // 2. 복사아니고 그대로 ((a) => a)()
const myNextList = myList.map(v => ({...v})) // 3 myList[0].value 복사
```

# Immer로 간결한 업데이트 로직 작성하기

변경 없이 중첩된 배열을 업데이트 하는 것은 [이전 객체와 마찬가지](https://react.dev/learn/updating-objects-in-state#write-concise-update-logic-with-immer)로 약간 반복적일 수 있다.

- 일반적으로 깊은 레벨까지 state 업데이트 할 필요가 없다. (평탄화)
- state의 구조를 변경하고 싶지 않다면 immer를 사용할 수 있다.

```jsx
import { useState } from 'react';
import { useImmer } from 'use-immer';

let nextId = 3;
const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
  { id: 2, title: 'Terracotta Army', seen: true },
];

export default function BucketList() {
  const [myList, updateMyList] = useImmer(
    initialList
  );
  const [yourArtworks, updateYourList] = useImmer(
    initialList
  );

  function handleToggleMyList(id, nextSeen) {
    updateMyList(draft => {
      const artwork = draft.find(a =>
        a.id === id
      );
      artwork.seen = nextSeen;
    });
  }

  function handleToggleYourList(artworkId, nextSeen) {
    updateYourList(draft => {
      const artwork = draft.find(a =>
        a.id === artworkId
      );
      artwork.seen = nextSeen;
    });
  }

  return (
    <>
      <h1>Art Bucket List</h1>
      <h2>My list of art to see:</h2>
      <ItemList
        artworks={myList}
        onToggle={handleToggleMyList} />
      <h2>Your list of art to see:</h2>
      <ItemList
        artworks={yourArtworks}
        onToggle={handleToggleYourList} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map(artwork => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={e => {
                onToggle(
                  artwork.id,
                  e.target.checked
                );
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}

//immer를 사용하면 아래와 같이 사용해도 괜찮다.
//하지만 개인적으로 보았을 땐 모듈로인해 악습관을 들이는 것 같습니다.
updateMyTodos(draft => {
  const artwork = draft.find(a => a.id === artworkId);
  artwork.seen = nextSeen;
});
```

- 이전 객체 챕터에서 배웠듯이 특수 draft 객체를 변경하기 때문이다.
- push()와 pop()같은 변경함수도 적용할 수 있다.

### 요약

- 배열을 state로 만들 수 있지만 변경하면 안된다.
- 배열을 변경하는 대신 배열의 새로운 버전을 만들고, state를 업데이트 해야한다.
- `[...arr, newItem]` 배열 전개 구문을 사용하여 새 항목을 포함한 배열을 생성할 수 있다.
- `filter()`와 `map()`을 사용하여 필터링된 항목들이나 변환된 항목들을 가진 배열을 만들 수 있다.
- Immer를 사용하여 코드 간결성을 유지할 수 있습니다.