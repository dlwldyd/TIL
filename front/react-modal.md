# React Modal
```javascript
import Modal from 'react-modal';

interface ModalInfo {
    isOpen: boolean,
    setIsOpen: React.Dispatch<React.SetStateAction<boolean>>,
    content: JSX.Element,
}

function ModalForm({isOpen, setIsOpen, content}: ModalInfo) {

    const onClose = () => {
        setIsOpen(isOpen => !isOpen);
    }

    return(
        <Modal
            isOpen={isOpen} // true면 모달이 열리고 false면 모달이 닫힌다.
            onRequestClose={onClose} // 모달이 닫히는(isOpen이 false가 되는) 조건이다. modal 바깥을 클릭하거나 esc를 눌러서 끄는 거는 디폴트로 포함된다.
            style={{ // 모달의 css이다.
                overlay: { // 모달 외부의 css
                position: 'fixed',
                top: 0,
                left: 0,
                right: 0,
                bottom: 0,
                backgroundColor: 'rgba(0, 0, 0, 0.75)'
                },
                content: { // 모달 내부의 css
                width: '500px',
                height: '300px',
                position: 'absolute',
                top: '50%',
                left: '50%',
                marginLeft: '-250px',
                marginTop: '-150px',
                border: '1px solid #ccc',
                background: '#fff',
                overflow: 'auto',
                WebkitOverflowScrolling: 'touch',
                borderRadius: '4px',
                outline: 'none',
                padding: '20px',
                }
            }}>
            {content}
        </Modal>
    )
}
```
* react-modal을 사용하면 손쉽게 모달창을 만들 수 있다.
* 모달이 닫히는 조건에서 esc를 눌러서 닫는 것과 모달 외부를 클릭해서 닫는 것은 기본으로 포함되어 있다.