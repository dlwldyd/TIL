# JSX
## JSX 기초
```javascript
...

<body>
</body>
<script type="text/babel">
    const body = document.querySelector("body");
    const h2 = (
        <h2>Click here</h2>
    );
    const btn = (
        <button id="btn" onClick={() => console.log("Clicked")}>Button</button>
    );
    ReactDOM.render(h2, body);
    ReactDOM.render(btn, body);
</script>

...

```
* html과 유사하게, 그리고 쉽게 react 요소를 만들기 위해 사용하는 문법이다.
* 이벤트 리스너는 속성 부분에 넣는다.
* 이벤트 리스너를 넣기 위해서는 이벤트 리스너의 이름 앞에 "on"을 붙여줘야한다. ex)"Click" -> "onClick", "MouseEnter" -> "onMouseEnter"
## 여러 요소 묶기
```javascript
...

<body>
</body>
<script type="text/babel">
    const body = document.querySelector("body");
    const H2 = () => (
        <h2>Click here</h2>
    );
    const Btn = () => (
        <button id="btn" onClick={() => console.log("Clicked")}>Button</button>
    );
    const Container = () => (
        <div>
            <H2/>
            <Btn/>
        </div>
    );
    ReactDOM.render(<Container/>, body);
</script>

...

```
* 여러 컴포넌트를 하나의 컴포넌트에 넣어서 묶을 때 다른 컴포넌트에 넣을 컴포넌트는 함수로 정의해야하고 함수 이름을 대문자로 정의해야한다.(소문자로 하면 html 태그로 인식함)
## JSX 내 html 속성값
* class -> className
* for -> htmlFor