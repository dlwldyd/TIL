# State
## State를 사용하기 전
```javascript
const root = document.getElementById("root");
let cnt = 0;

function cntUp() {
    cnt += 1;
    ReactDOM.render(<Container/>, root);
}
const Container = () => (
    <div>
        <h3>Total Clicks : {cnt}</h3>
        <button onClick={cntUp}>Click me</button>
    </div>
);
ReactDOM.render(<Container/>, root);
```
* State를 사용하지 않으면 이벤트로 인해 업데이트된 데이터를 화면에 보여주기 위해 ReactDOM.render(<Container/>, root)를 호출해 다시 화면을 렌더링 해줘야 한다.
## State 사용
```javascript
const root = document.getElementById("root");
const Container = () => {
    const [cnt, setCnt] = React.useState(0);
    function cntUp() {
        // setCnt(cnt + 1); 값을 넣을 수도 있다.
        setCnt((cnt) => cnt + 1); //함수를 넣을 수도 있다. State의 첫 번째 인자값을 사용하는 경우 이 방법이 위의 방법보다 더 좋은 방법
    }
    return (
        <div>
            <h3>Total Clicks : {cnt}</h3>
            <button onClick={cntUp}>Click me</button>
        </div>
    );   
}
ReactDOM.render(<Container/>, root);
```
* State를 사용하면 이벤트 발생시 호출되는 함수마다 화면을 렌더링 하는 ReactDOM.render()를 호출할 필요가 없다.
* React.useState()는 ReactDOM.render()에 들어갈 컴포넌트 내에 넣어야 한다. 컴포넌트 밖에 넣으면 동작하지 않는다.
* React.useState()의 반환값은 길이 2인 배열인데 첫번째 인자는 렌더링할 데이터이고, 2번째 인자는 modifier(자바의 setter라 생각하면 된다.)이다.
* modifier가 호출되면 자동으로 ReactDOM.render(<Container/>, root)가 호출된다.