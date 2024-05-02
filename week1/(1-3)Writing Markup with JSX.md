JSX 는 문법적 확장. HTML 친화적인 마크업을 javscript 파일 내에서 작성할 수 있도록 함.

## Why React mixes markup with rendering logic

과거의 웹 개발에서는 HTML, CSS, JavaScript를 각각 HTML은 컨텐츠, CSS는 디자인, JavaScript는 로직으로 나누어서 작성했었습니다.

현재로 와서는 웹이 더 복잡하고 인터렉티브한 액션들이 많아지게 되었고, 점점 더 로직이 컨텐츠를 결정하게 되면서 결국 자바스크립트가 HTML 컨텐츠를 생성하게 되었습니다.

이러한 이유로 리액트에서는 마크업과 렌더링 로직을 한 곳에서 작성하도록 합니다. 

## How JSX is different from HTML

JSX는 HTML과 유사하게 보이는 Javascript 문법 확장입니다. 리액트 컴포넌트는 마크업이 포함된 자바스크립트 함수이며, 브라우저에 마크업을 표현하기 위해서 JSX를 사용합니다.

JSX는 HTML보다는 조금 더 엄격한 규칙을 가지고있으며 동적인 정보를 노출할 수 있습니다.

React와 JSX는 함께 쓰이지만 독립적으로도 사용할 수 있으며 JSX는 문법 확장, React는 자바스크립트 라이브러리 입니다.

## How to display information with JSX

JSX를 작성하는 규칙은 3가지 입니다.

1. single root element
    
    여러개의 요소를 반환하기 위해서는 하나의 부모 태그로 감싸야합니다.
    
    부모 태그를 마크업에서 제외해야하는 경우는 Fragment로 불리는 empty tag를 대신해서 사용할 수 있습니다. Fragment는 브라우저 HTML 트리에 흔적을 남기지 않고 항목을 그룹화 합니다.
    
    하나의 부모 태그로 감싸야하는 이유는 JSX는 내부적으로 자바스크립트 객체로 변환됩니다.
    
    자바스크립트에서 하나의 함수가 두개의 객체를 반환 할 수 없기 때문에 JSX도 부모 태그를 감싸는 것 없이 리턴할 수 없다는 것을 의미합니다.
    
    ```jsx
    const func = () => {
    	return [{},{}]
    }
    
    const jsx = () => {
    	return <><img /><img /></>
    }
    ```
    
2. close all the tags
    
    JSX 는 명시적으로 모든 태그가 닫혀있어야합니다. self-closing 태그의 경우도 명시해주어야합니다
    
    ```jsx
    <img src=""> -> <img src="" />
    ```
    
3. camelCase ~~all~~ most of the things
    
    JSX가 자바스크립트로 변환되고 JSX에서 작성한 속성들은 자바스크립트 객체의 키가 됩니다.
    
    ```jsx
    <img class >
    <svg stroke-width />
    
    =>
    
    img: {
    	className: ''
    	'aria-label': ''
    }
    
    svg: {
      strokeWidth: ''
    }
    
    ```
    
    이러한 속성들을 종종 변수로 읽어들이길 원할수도 있습니다. 그러나 자바스크립트는 변수 이름에 대한 제한 사항이 있습니다. 예를 들면 이름 규칙에서 dash(-) 나 예약어 등은 사용할 수 없습니다.
    
    이러한 이유로 리액트에서는 HTML과 SVG 속성들을 camelCase로 작성합니다.(ex. `stroke-width` → `strokeWidth`). `class` 도 예약어이기 때문에 리액트에서는 `className` 을 대신 사용합니다.
    
    [Element: className property - Web APIs | MDN](https://developer.mozilla.org/en-US/docs/Web/API/Element/className)
    
    `aria-*` 및 `data-*` 속성은 역사적인 이유로 HTML에서와 같이 대시를 사용하여 작성합니다.
    

과거의 웹 개발자들은 content = HTML, design = CSS, logic = JavaScript 로 사용했었습니다.

웹 컨텐츠 마크업은 HTML에 작성하고 페이지에 관련한 로직은 자바스크립트에 작성하는 식으로 로직이 분리되어있었습니다.

웹이 더 인터렉티브하게 되었고, 점점 더 로직이 컨텐츠를 결정하게 되었습니다. 결국 자바스크립트가 HTML을 담당하게 되었습니다.

이러한 이유로 리액트에서는 렌더링 로직과 마크업을 같은 곳(컴포넌트)에서 작성하도록 합니다.

각 리액트 컴포넌트는 브라우저에 리액트를 렌더하기 위한 약간의 마크업이 포함된 자바스크립트 함수입니다. 

리액트 컴포넌트는 마크업을 표현하기 위해 JSX라는 문법 확장을 사용하고 있습니다.

JSX는 HTML 처럼 보이지만 조금 더 엄격하고 동적인 정보를 노출할 수 있습니다. 이를 가장 잘 이해하는 방법은 HTML 마크업을 JSX 마크업으로 변환해보는 것 입니다.

JSX와 React는 각각 다릅니다. 두개가 같이 쓰이긴 하지만 독립적으로도 사용할 수 있습니다.

JSX는 구문확장이고 React는 자바스크립트 라이브러리 입니다.

JSX 규칙

1. single root element
    
    여러개의 요소를 반환하기 위해서는 하나의 부모태그로 감싸야합니다.
    
    부모 태그를 마크업에서 제외해야한다면, Fragment 라고 불리는 `<></>` 를 대신해서 쓸 수 있습니다.
    
    Fragment 는 브라우저 HTML 트리에 흔적을 남기지 않고 항목을 그룹화 할 수 있습니다.
    
    JSX는 HTML 처럼 보이지만 내부적으로 자바스크립트 객체로 변환됩니다. 즉 하나의 함수에 배열로 감싸는 것이 아니라면 두개의 객체를 리턴할 수 없습니다. 이것으로 JSX 태그를 두 개를 다른 태그 또는 Fragment로 감싸는것 없이 리턴할 수 없다는 것을 설명할 수 있습니다.
    

1. Close all the tags
    
    JSX 는 모든 태그가 명시적으로 닫혀있어야 합니다. self-closing 태그도 명시적으로 닫아야합니다.
    
    ```jsx
    <img src=""> -> <img src="" />
    ```
    

1. camelCase ~~all~~ most of the things
    
    JSX가 자바스크립트로 변환되고 JSX에서 작성한 속성들은 자바스크립트 객체의 키가 됩니다.
    
    이러한 속성들을 종종 변수로 읽어들이길 원할수도 있습니다. 그러나 자바스크립트는 변수 이름에 대한 제한 사항이 있습니다. 예를 들면 이름 규칙에서 dash(-) 나 예약어 등은 사용할 수 없습니다.
    
    이러한 이유로 리액트에서는 HTML과 SVG 속성들을 camelCase로 작성합니다.(ex. `stroke-width` → `strokeWidth`). `class` 도 예약어이기 때문에 리액트에서는 `className` 을 대신 사용합니다.
    
    `aria-*` 및 `data-*` 속성은 역사적인 이유로 HTML에서와 같이 대시를 사용하여 작성합니다.
    

참고

[HTML to JSX](https://transform.tools/html-to-jsx)
