# props
## props란
```javascript
const root = document.getElementById("root");
{/*function Btn(props) {
    return <button style={{
        backgroundColor: "tomato",
        color: "white",
        padding: "10px 20px",
        border: 0,
        borderRadius: 10
    }}>{props.text}</button>
}*/}
function Btn({text = "button", big, onClick}) {
    return <button style={{
        backgroundColor: "tomato",
        color: "white",
        padding: "10px 20px",
        border: 0,
        borderRadius: 10,
        fontSize: big ? 18 : 12
    }}
    onClick={onClick}>{text}</button>
}
function Container() {
    return (
        <div>
            {/*Btn({text:"Save Changes" big:true})*/}
            <Btn text="Save Changes" big={true}  onClick={changeValue}/>
            {/*Btn({text:"Confirm"})*/}
            <Btn text="Confirm"/>
        </div>
        );
}
ReactDOM.render(<Container/>, root);
```
* props를 통해 컴포넌트는 부모 컴포넌트로 부터 값을 전달받을 수 있다.(여기서의 부모 컴포넌트는 DOM 구조상의 부모 컴포넌트가 아니라 해당 컴포넌트를 감싸고 있는 함수를 말함)
* 오직 props로 전달되기 때문에 부모 컴포넌트에서 style 속성이나 이벤트 리스너 같은 것을 바로 추가할 수 없다. 자식 컴포넌트가 props로 받은 값을 속성으로 추가하는 작업을 따로 해줘야 한다.
## React.memo
```javascript
const root = document.getElementById("root");
function Btn({text, onClick}) {

    //props의 값이 바뀌지 않으면 리렌더링 되지 않기 대문에 Btn()이 호출되지 않고, 따라서 이 함수도 호출되지 않는다.
    console.log(text, "changed");

    return <button style={{
        backgroundColor: "tomato",
        color: "white",
        padding: "10px 20px",
        border: 0,
        borderRadius: 10
    }}
    onClick={onClick}>{text}</button>
}

// React.memo()
const MemorizedBtn = React.memo(Btn);

function Container() {
    const [value, setValue] = React.useState("Save Changes");
    const changeValue = () => setValue("Revert Changes");
    return (
        <div>
            {/*버튼을 누르면 값이 바뀌는 버튼*/}
            <MemorizedBtn text={value} onClick={changeValue}/>
            {/*버튼을 눌러도 값이 바뀌지 않는 버튼*/}
            <MemorizedBtn text="Confirm"/>
        </div>
        );
}
ReactDOM.render(<Container/>, root);
```
* SaveChanges 버튼을 누르면 value 값이 바뀌기 때문에 Container 컴포넌트가 리렌더링되고, Container 컴포넌트의 자식 컴포넌트인 Btn 컴포넌트들 역시 전부 리렌더링 된다. 하지만 첫 번째 버튼은 버튼의 텍스트가 "Save Changes"에서 "Revert Changes"로 바뀌기 때문에 리렌더링 할 필요가 있지만 두 번째 버튼은 아무것도 바뀌지 않기 때문에 리렌더링 할 필요가 없다. 이러한 쓸데없이 리렌더링 하는 작업을 없애기 위해 사용하는 것이 React.memo() 이다. React.memo()를 사용하면 props가 바뀐 컴포넌트만 리렌더링을 하고, props의 값이 바뀌지 않으면 리렌더링 하지 않는다.
## PropTypes
```javascript
const root = document.getElementById("root");
    function Btn({text, fontSize}) {
        return <button style={{
            backgroundColor: "tomato",
            color: "white",
            padding: "10px 20px",
            border: 0,
            borderRadius: 10,
            fontSize: fontSize
        }}>{text}</button>
    }
    // 어떤 타입의 값이 들어가야 하는지 명시
    Btn.propTypes = {
        text: PropTypes.string,
        fontSize : PropTypes.number
    };
    function Container() {
        return (
            <div>
                <Btn text="Save Changes" fontSize={16}/>
                <Btn text="Confirm"/>
            </div>
            );
    }
    ReactDOM.render(<Container/>, root);
```
* 자바스크립트는 타입을 명시하지 않기 때문에 props에 들어가지 말아야할 타입의 값이 들어갈 수 있다. 하지만 PropTypes를 사용하여 props에 어떤 타입의 값이 들어가야 하는지 명시함으로써 들어가지 말아야할 타입의 값이 들어가는 것을 방지할 수 있다.(실행이 안되는게 아니라 콘솔에서 경고를 해준다)