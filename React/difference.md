# Vanilla javascript와 ReactJS의 차이
```javascript
...

<body>
    <h2>Click here</h2>
    <button id="btn">Button</button>
</body>
<script>
    const btn = document.getElementById("btn");
    function handleClick() {
        console.log("Button clicked");
    }
    btn.addEventListener("click", handleClick);
</script>

...

```
```javascript
...

<body>
</body>
<script>
    const body = document.querySelector("body");
    const h2 = React.createElement("h2", null, "Click here");
    const btn = React.createElement("button", {id: "btn", onClick: () => console.log("Clicked")}, "Button");
    ReactDOM.render(h2, body);
    ReactDOM.render(btn, body);
</script>

...

```
* interactive application을 만든다는 말은 결국은 event listener를 붙이는 일의 연속이다. 자바스크립트만 사용하는 경우에는 interactive application을 만들기 위해서 html문서에 있는 컴포넌트를 가져와서 event listener를 붙여야 한다. 반면에 ReactJS를 사용하면 자바스크립트만 사용할 때와는 정반대로 컴포넌트 자체를 코드에서 생성하고(event listener는 컴포넌트를 생성할 때 붙여준다. 따라서 자바스크립트만 사용할 때보다 한 단계 과정이 줄어든다.) 생성된 컴포넌트를 DOM구조안에 넣어준다. 따라서 ReactJS를 사용하면 자바스크립트만을 사용할 때 보다 interactive application을 만드는데 더 유리하다.
* 컴포넌트를 코드에서 생성하기 때문에 원한다면 같은 컴포넌트를 여러번 재사용할 수 있다.