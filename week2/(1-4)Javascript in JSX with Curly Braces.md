# Javascript in JSX with Curly Braces

JSX는 자바스크립트 파일 안에 HTML과 유사한 마크업을 작성할 수 있게하는 문법이고, 렌더링 로직과 내용을 같은 곳에서 작성할 수 있다.

때때로 당신은 작은 자바스크립로직을 추가하길 원하거나 동적 프로퍼티를 마크업 내에 표현하기를 원한다.

이러한 상황에서 당신은 **curly braces**를 이용하여 자바스크립트를 JSX에서 이용할 수 있다.

- quotes를 이용해서 string을 전달하는 법
- 자바스크립트 변수를 표현하는 방법
- 자바스크립트 함수를 호출하는 법
- 자사브크립트 객체를 사용하는 법

## Passing string with quotes

JSX에 string 속성 값을 전달할 때에는 `‘` 또는 `“` 를 이용해서 전달 할 수 있다.

그러나 동적으로 지정된 텍스트를 전달하고자 한다면? 자바스크립트 변수를 이용하고 `{ }` 로 감싸면된다.

중괄호는 마크업에서 바로 자바스크립트를 작업할 수 있도록 합니다.

## Using curly braces: A window into the JavaScript world

JSX는 자바스크립트를 작성하는 특별한 방법입니다. 중괄호를 이용해서 자바스크립트를 내부에 작성할 수 있게 합니다. 자바스크립트 표현식은 중괄호 안에서 동작한다.

중괄호를 사용할 수 있는 위치는 2군데 존재한다.

1. 텍스트를 JSX 태그안에서 바로 사용할 때
2. `=` 뒤에 바로 오는 속성에서 사용할 때 → `<Profile title={} />`

## Using “double curlies”: CSS and other object in JSX

string, number, 자바스크립트 표현식을 추가할 때에는 JSX 안에 object로 전달 할 수 있다.

객체도 중괄호와 같이 표현한다. 그래서 자바스크립트 격체를 JSX에 전달하기 위해서는 또 다른 중괄호 쌍으로 감싸야한다

인라인 CSS 스타일링을 할 때 리액트는 인라인 스타일을 사용할 필요가 없음(대부분의 경우에는 CSS 클래스가 잘 작동하기 때문에)

만약 인라인 스타일이 필요하게 된다면, style 속성에 객체를 전달 할 수 있다.

인라인 스타일의 프로퍼티는 카멜케이스로 작성한다.

```jsx
<ul style="background-color: black">

<ul style={{ backgroundColor: 'black' }}>
```

## More fun with JavaScript objects and curly braces

여러 표현식을 하나의 객체 안으로 이동할 수 있고 중괄호 안에서 JSX 에서 참조 할 수 있음.

컴포넌트안에서 person의 값들을 사용할 수 있습니다.

```jsx
const person = {
	name: 'Gregorio Y. Zara',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
}

<div style={person.theme}>
	<h1>{person.name}'s Todos</h1>
```

JSX는 자바스크립트를 이용해서 로직과 데이터를 구성할 수 있기 때문에 템플릿 언어로서 매우 미니멀하다
