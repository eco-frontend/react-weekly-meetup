학습 내용

- [`useState`](https://ko.react.dev/reference/react/useState) 훅으로 state 변수를 추가하는 방법
- useState 훅이 반환하는 한쌍의 값
- 둘 이상의 state 변수를 추가하는 방법
- state를 지역적이라고 불리는 이유

## 일반 변수로 충분하지 않은 경우

- 아래 코드는 조각 이미지를 렌더링하는 컴포넌트이다.
- next 버튼을 클릭하면, index가 변경되어 다음 조각품이 표시된다.
    - 하지만 이 방법은 작동하지 않는다.

<aside>
⚠️ 1. 렌더링 사이에 데이터를 **유지**한다.
2. react가 새로운 데이터로 컴포넌트를 렌더링하도록 **유발**한다.

</aside>

- 예제 코드
    
    ```jsx
    //App.js
    
    import { sculptureList } from './data.js';
    
    export default function Gallery() {
      let index = 0;
    
      function handleClick() {
        index = index + 1;
      }
    
      let sculpture = sculptureList[index];
      return (
        <>
          <button onClick={handleClick}>
            Next
          </button>
          <h2>
            <i>{sculpture.name} </i> 
            by {sculpture.artist}
          </h2>
          <h3>  
            ({index + 1} of {sculptureList.length})
          </h3>
          <img 
            src={sculpture.url} 
            alt={sculpture.alt}
          />
          <p>
            {sculpture.description}
          </p>
        </>
      );
    }
    
    //data.js
    
    조각상 배열
    ```
    
    - `handleClick` 함수 가 지역변수 index 를 업데이트하고 있습니다.
        
        그러나 두 가지 요인으로 인해 해당 변경 사항이 눈에 보이지 않게 됩니다.
        
        1. 지역 변수는 렌더링간에 유지되어지지 않는다.
            - react가 컴포넌트를 두번째 렌더링할 때 지역변수에 대한 변경사항은 고려하지 않고 처음부터 렌더링한다.
        2. 지역 변수를 변경해도 렌더링이 작동하지 않는다.
            - react는 새 데이터로 구성요소를 다시 렌더링하는 것을 인식하지 못함
    - 컴포넌트를 렌더링하기 위해서 두가지가 필요
        1. 렌더링 사이에 데이터를 **유지**한다.
        2. react가 새로운 데이터로 컴포넌트를 렌더링하도록 **유발**한다.
    - useState 훅은 앞서 말한 이 두 가지를 제공한다.
        1. state 변수
        2. state Setter(setState) 함수

## State 변수 추가하기

- state 변수 추가 시 파일 상단에 React에서 useState를 가져온다.
    - 아래의 단계처럼 변경한다.

```jsx
import { useState } from 'react';

///1 번째 단계
let index = 0;

///2 번째 단계
const [index, setIndex] = useState(0);
```

- index는 state 변수이고, setIndex는 setter 함수가 된다.

<aside>
⚠️ 여기서 `[]` 문법을 [배열 구조 분해](https://javascript.info/destructuring-assignment)라고 하며, 배열로부터 값을 읽을 수 있게 해줍니다. `useState`가 반환하는 배열에는 항상 두 개의 항목이 있다.

</aside>

- 아래와 같이 변경했을 때 handleClick이 정상작동한다.

```jsx
function handleClick() {
  setIndex(index + 1);
}
```

## 첫 번째 훅 만나기

- React에서 useState와 같이 `use` 로 시작하는 다른 함수를 모두 훅이라고 한다.
- 훅은 오직 React가 오직 [렌더링](https://react.dev/learn/state-a-components-memory) 중일 때만 사용할 수 있는 특별한 함수이고, 이를 통해 React기능을 “연결”할 수 있다.
    - 렌더링은  - 다음페이지에서 자세히 알아본다.
    

<aside>
⚠️ **훅(`use`로 시작하는 함수들)은 컴포넌트의 최상위 수준 또는 [커스텀 훅](https://react.dev/learn/reusing-logic-with-custom-hooks)에서만 호출할 수 있다.**

</aside>

- 조건문, 반복문 또는 기타 중첩 함수 내부에서는 훅을 호출할 수 없다.
- 훅은 함수이지만 컴포넌트의 필요에 대한 무조건적인 선언으로 생각하면 도움이 됩니다.
- 파일 상단에서 모듈을 “import”하는 것과 유사하게 컴포넌트 상단에서 React 기능을 “Use”합니다.
    
    ```jsx
    // 1
    const Component = () => {
    	useState();
    	
    	const handleClick = () => {
    		// const [state, setState] = useState();
    	}
    	
    	useEffect(() => {},[])
    	
    	return (<></>)
    }
    
    // 2
    const Component = () => {
    	const handleClick = ()  => {
    		state
    	}
    	
    	const [state, setState] = useState();
    	
    	return (<button onClick={handleClick}></button>)
    }
    
    // 3
    const Component = () => {
    	const handleClick = () => {
    	}
    	
    	useEffect(() => 
    	
    	useState();
    	
    	return (<></>)
    }
    
    // 4 -> not pure
    const Component = () => {
    	const handleClick = () => {
    	}
    	
    	if (props.a && props.b) return null
    	
    	useEffect(() => 
    	
    	useState();
    	
    	return (<></>)
    }
    ```
    

## useState 해부

- useState를 호출하면, React 에 컴포넌트가 무언가를 기억하기를 원한다고 말한다.
    - 위에서는 index를 기억하기를 원하기 때문

<aside>
⚠️ 이 쌍의 이름은 `const [something, setSomething]`과 같이 지정하는 것이 규칙이다.
state: loading
setState: setLoading → setFreezing 알 수 없음 x

</aside>

- useState의 유일한 인수는 state 변수의 **초깃값이다.** 위에서는 0

순서를 살펴보면 아래와 같다.

1. 컴포넌트가 처음 렌더링 된다. index의 초깃값으로 useState를 사용해 0을 전달
2. State를 업데이트한다. 사용자가 버튼을 클릭하면 setIndex(index+1)를 호출한다.
3. 컴포넌트가 두번째로 렌더링된다. React는 index + 1을 하여 1을 반환한다.
    1. 
    1. **Your component’s second render.** React still sees `useState(0)`, but because React *remembers* that you set `index` to `1`, it returns `[1, setIndex]` instead.
4. 위와 같은 상태가 반복된다. 

```jsx
prev: setIndex(index + 1); -> 호출 되어서 리렌더링이라는 동작을 수행함

index + 1 이라는 인자

// 1
// useState(0)
// second render phase => +1 index = 1
index: 0
setIndex(param) //param => index + 1 = 1
	rerender()

index: 1

current: 리렌더링 동작이 수행됨, 이때 index + 1이 어디로 넘어가서 1이 반환되는걸까..?
```

## 컴포넌트에 여러 State 변수 지정하기

- 하나의 컴포넌트에 원하는 만큼의 State 변수를 가질 수 있다.
- 아래 컴포넌트는 숫자 타입 index와 “show Details”를 클릭했을 때 토글 되는 showMore
- 코드
    
    ```jsx
    import { useState } from 'react';
    import { sculptureList } from './data.js';
    
    export default function Gallery() {
      const [index, setIndex] = useState(0);
      const [showMore, setShowMore] = useState(false);
    
      function handleNextClick() {
        setIndex(index + 1);
      }
    
      function handleMoreClick() {
        setShowMore(!showMore);
      }
    
      let sculpture = sculptureList[index];
      return (
        <>
          <button onClick={handleNextClick}>
            Next
          </button>
          <h2>
            <i>{sculpture.name} </i> 
            by {sculpture.artist}
          </h2>
          <h3>  
            ({index + 1} of {sculptureList.length})
          </h3>
          <button onClick={handleMoreClick}>
            {showMore ? 'Hide' : 'Show'} details
          </button>
          {showMore && <p>{sculpture.description}</p>}
          <img 
            src={sculpture.url} 
            alt={sculpture.alt}
          />
        </>
      );
    }
    ```
    
    - 이 예제 처럼 index와 showMore이 서로 연관되어있지 않는 경우 여러개의 state 변수를 가지는 것이 좋다.
    - 두 state 변수를 자주 함께 변경하는 경우에는 서로 합치는 것이 더 좋을 수 있다.
        - 서로 다른 부수효과가 계속 발생
    
    - DEEP DIVE ⚠️
        
        React는 어떤 state를 반환할지 어떻게 알 수 있을까요?
        
        - useState 호출이 어떤 state 변수를 참조하는지에 대한 정보를 받지 못한다는 것을 눈치챘을 것입니다.
        - useState에 전달되는 “식별자”가 없는데 어떻게 어떤 변수를 반환할지 알 수 있을까요?
        - 함수를 파싱하는 것과 같은 magic에 의존하는걸까요?
            - 아닙니다.
        - 간결한 구문을 구현하기 위해 훅은 동일한 컴포넌트의 **모든 렌더링에서 안정적인 호출 순서에 의존한다.**
        - ”최상위 수준에서만 훅 호출”을 따르면, 훅은 항상 같은 순서로 호출되기 때문에 잘 작동한다.   [린트 플러그인](https://www.npmjs.com/package/eslint-plugin-react-hooks) 에서 대부분의 실수도 잡아준다.
        - 더욱 자세한 내용은 링크([리액트 훅: 마법이 아닌 배열](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e))에서 확인가능하다
        
        ```jsx
        let componentHooks = [];
        let currentHookIndex = 0;
        
        //구현 해보기 issue
        // How useState works inside React (simplified).
        function useState(initialState) {
          let pair = componentHooks[currentHookIndex];
          if (pair) {
            // This is not the first render,
            // so the state pair already exists.
            // Return it and prepare for next Hook call.
            currentHookIndex++;
            return pair;
          }
        
          // This is the first time we're rendering,
          // so create a state pair and store it.
          pair = [initialState, setState];
        
          function setState(nextState) {
            // When the user requests a state change,
            // put the new value into the pair.
            pair[0] = nextState;
            updateDOM();
          }
        
          // Store the pair for future renders
          // and prepare for the next Hook call.
          componentHooks[currentHookIndex] = pair;
          currentHookIndex++;
          return pair;
        }
        
        function Gallery() {
          // Each useState() call will get the next pair.
          const [index, setIndex] = useState(0);
          const [showMore, setShowMore] = useState(false);
        
          function handleNextClick() {
            setIndex(index + 1);
          }
        
          function handleMoreClick() {
            setShowMore(!showMore);
          }
        
          let sculpture = sculptureList[index];
          // This example doesn't use React, so
          // return an output object instead of JSX.
          return {
            onNextClick: handleNextClick,
            onMoreClick: handleMoreClick,
            header: `${sculpture.name} by ${sculpture.artist}`,
            counter: `${index + 1} of ${sculptureList.length}`,
            more: `${showMore ? 'Hide' : 'Show'} details`,
            description: showMore ? sculpture.description : null,
            imageSrc: sculpture.url,
            imageAlt: sculpture.alt
          };
        }
        
        function updateDOM() {
          // Reset the current Hook index
          // before rendering the component.
          currentHookIndex = 0;
          let output = Gallery();
        
          // Update the DOM to match the output.
          // This is the part React does for you.
          nextButton.onclick = output.onNextClick;
          header.textContent = output.header;
          moreButton.onclick = output.onMoreClick;
          moreButton.textContent = output.more;
          image.src = output.imageSrc;
          image.alt = output.imageAlt;
          if (output.description !== null) {
            description.textContent = output.description;
            description.style.display = '';
          } else {
            description.style.display = 'none';
          }
        }
        
        let nextButton = document.getElementById('nextButton');
        let header = document.getElementById('header');
        let moreButton = document.getElementById('moreButton');
        let description = document.getElementById('description');
        let image = document.getElementById('image');
        let sculptureList = [{
          name: 'Homenaje a la Neurocirugía',
          artist: 'Marta Colvin Andrade',
          description: 'Although Colvin is predominantly known for abstract themes that allude to pre-Hispanic symbols, this gigantic sculpture, an homage to neurosurgery, is one of her most recognizable public art pieces.',
          url: 'https://i.imgur.com/Mx7dA2Y.jpg',
          alt: 'A bronze statue of two crossed hands delicately holding a human brain in their fingertips.'  
        }, {
          name: 'Floralis Genérica',
          artist: 'Eduardo Catalano',
          description: 'This enormous (75 ft. or 23m) silver flower is located in Buenos Aires. It is designed to move, closing its petals in the evening or when strong winds blow and opening them in the morning.',
          url: 'https://i.imgur.com/ZF6s192m.jpg',
          alt: 'A gigantic metallic flower sculpture with reflective mirror-like petals and strong stamens.'
        }, {
          name: 'Eternal Presence',
          artist: 'John Woodrow Wilson',
          description: 'Wilson was known for his preoccupation with equality, social justice, as well as the essential and spiritual qualities of humankind. This massive (7ft. or 2,13m) bronze represents what he described as "a symbolic Black presence infused with a sense of universal humanity."',
          url: 'https://i.imgur.com/aTtVpES.jpg',
          alt: 'The sculpture depicting a human head seems ever-present and solemn. It radiates calm and serenity.'
        }, {
          name: 'Moai',
          artist: 'Unknown Artist',
          description: 'Located on the Easter Island, there are 1,000 moai, or extant monumental statues, created by the early Rapa Nui people, which some believe represented deified ancestors.',
          url: 'https://i.imgur.com/RCwLEoQm.jpg',
          alt: 'Three monumental stone busts with the heads that are disproportionately large with somber faces.'
        }, {
          name: 'Blue Nana',
          artist: 'Niki de Saint Phalle',
          description: 'The Nanas are triumphant creatures, symbols of femininity and maternity. Initially, Saint Phalle used fabric and found objects for the Nanas, and later on introduced polyester to achieve a more vibrant effect.',
          url: 'https://i.imgur.com/Sd1AgUOm.jpg',
          alt: 'A large mosaic sculpture of a whimsical dancing female figure in a colorful costume emanating joy.'
        }, {
          name: 'Ultimate Form',
          artist: 'Barbara Hepworth',
          description: 'This abstract bronze sculpture is a part of The Family of Man series located at Yorkshire Sculpture Park. Hepworth chose not to create literal representations of the world but developed abstract forms inspired by people and landscapes.',
          url: 'https://i.imgur.com/2heNQDcm.jpg',
          alt: 'A tall sculpture made of three elements stacked on each other reminding of a human figure.'
        }, {
          name: 'Cavaliere',
          artist: 'Lamidi Olonade Fakeye',
          description: "Descended from four generations of woodcarvers, Fakeye's work blended traditional and contemporary Yoruba themes.",
          url: 'https://i.imgur.com/wIdGuZwm.png',
          alt: 'An intricate wood sculpture of a warrior with a focused face on a horse adorned with patterns.'
        }, {
          name: 'Big Bellies',
          artist: 'Alina Szapocznikow',
          description: "Szapocznikow is known for her sculptures of the fragmented body as a metaphor for the fragility and impermanence of youth and beauty. This sculpture depicts two very realistic large bellies stacked on top of each other, each around five feet (1,5m) tall.",
          url: 'https://i.imgur.com/AlHTAdDm.jpg',
          alt: 'The sculpture reminds a cascade of folds, quite different from bellies in classical sculptures.'
        }, {
          name: 'Terracotta Army',
          artist: 'Unknown Artist',
          description: 'The Terracotta Army is a collection of terracotta sculptures depicting the armies of Qin Shi Huang, the first Emperor of China. The army consisted of more than 8,000 soldiers, 130 chariots with 520 horses, and 150 cavalry horses.',
          url: 'https://i.imgur.com/HMFmH6m.jpg',
          alt: '12 terracotta sculptures of solemn warriors, each with a unique facial expression and armor.'
        }, {
          name: 'Lunar Landscape',
          artist: 'Louise Nevelson',
          description: 'Nevelson was known for scavenging objects from New York City debris, which she would later assemble into monumental constructions. In this one, she used disparate parts like a bedpost, juggling pin, and seat fragment, nailing and gluing them into boxes that reflect the influence of Cubism’s geometric abstraction of space and form.',
          url: 'https://i.imgur.com/rN7hY6om.jpg',
          alt: 'A black matte sculpture where the individual elements are initially indistinguishable.'
        }, {
          name: 'Aureole',
          artist: 'Ranjani Shettar',
          description: 'Shettar merges the traditional and the modern, the natural and the industrial. Her art focuses on the relationship between man and nature. Her work was described as compelling both abstractly and figuratively, gravity defying, and a "fine synthesis of unlikely materials."',
          url: 'https://i.imgur.com/okTpbHhm.jpg',
          alt: 'A pale wire-like sculpture mounted on concrete wall and descending on the floor. It appears light.'
        }, {
          name: 'Hippos',
          artist: 'Taipei Zoo',
          description: 'The Taipei Zoo commissioned a Hippo Square featuring submerged hippos at play.',
          url: 'https://i.imgur.com/6o5Vuyu.jpg',
          alt: 'A group of bronze hippo sculptures emerging from the sett sidewalk as if they were swimming.'
        }];
        
        // Make UI match the initial state.
        updateDOM();
        
        ```
        
    
    ## State는 격리되고 비공개로 유지된다.
    
    - State는 화면에서 컴포넌트에 지역적이다.
    - 동일한 컴포넌트를 두 번 렌더링한다면, 각 복사본은 **완전히 격리된 State**를 가진다
    
    ```jsx
    import Gallery from './Gallery.js';
    
    export default function Page() {
      return (
        <div className="Page">
          <Gallery />
          <Gallery />
        </div>
      );
    }
    ```
    
    - Page 컴포넌트가 Gallery의 state에 대해 “알지” 않는다. (독립적으로 작동)
        - Props와 달리, **state는 선언한 컴포넌트에 완전히 비공개다**
        - 추후 [컴포넌트 간 State 공유](https://react.dev/learn/sharing-state-between-components)는 다시 다룬다.
        
    
    ### 요약)
    
    - 컴포넌트가 렌더링 간에 어떤 정보를 “기억”할 때 state 변수를 사용한다.
    - state 변수는 useState 훅을 호출하여 선언한다.
    - 훅은 use로 시작하는 특별한 함수들이다.
    - 훅은 import와 마찬가지로 반드시 호출되어야 한다. useState를 포함한 훅을 호출하는 것은 컴포는트나 훅의 최상위 수준에서만 유효하다.
    - useState 훅은 현재 state와 이를 업데이트할 함수로 이루어진 한쌍을 반환한다.
    - 여러개의 state 변수를 가질 수 있다. React 내부에서 그들을 순서대로 매칭한다.
    - state는 컴포넌트에 비공개이다. 두 곳에서 렌더링 하더라도 각각 복사본은 고유한 state를 가진다.
