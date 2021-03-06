# 메시지, 국제화
## 메시지
messages.properties 라는 메시지 관리용 파일을 만들고 다양한 메시지를 한 곳에서 관리하도록 하는 기능을 메시지 기능이라 한다. 메시지 관리 기능을 사용하려면 스프링이 제공하는 MessageSource를 스프링 빈으로 등록하면 되는데, 스프링 부트를 사용하면 스프링 부트가 MessageSource를 자동으로 스프링 빈으로 등록한다. 스프링 부트를 사용하면 다음과 같이 메시지 소스를 설정할 수 있다.
```
spring.messages.basename=messages
```
스프링 부트 메시지 소스 기본 값은 spring.messages.basename=messages이기 때문에 스프링 부트와 관련된 별도의 설정을 하지 않으면 messages 라는 이름으로 기본 등록된다. 따라서 messages_en.properties, messages_ko.properties, messages.properties 등의 파일만 등록하면 자동으로 인식된다.
### 사용법
```properties
item=상품
item.id=상품 ID
item.itemName=상품명
item.price=가격 {0}
item.quantity=수량
```
```html
<label for="itemName" th:text="#{item.itemName}"></label>
<!--매개변수 전달, won은 "원" 이라는 String-->
<label for="price" th:text="#{item.price(${won})}"></label>
<!--메시지가 없는 경우에는 NoSuchMessageException 이 발생한다-->
<label for="errorTest" th:text="#{errorTest}"></label>
```
* 타임리프가 렌더링 될 때 messageSource의 getMessage 함수가 실행되면서 해당 되는 메시지를 찾아온다. 만약 메시지가 없는 경우에는 NoSuchMessageException 이 발생한다.
* 매개변수를 전달할 수 있다.
## 국제화
```properties
#messages.properties
item=상품
item.id=상품 ID
item.itemName=상품명
item.price=가격
item.quantity=수량
```
```properties
#messages_en_US.properties
item=Item
item.id=Item ID
item.itemName=Item Name
item.price=Price
item.quantity=Quantity
```
```html
<label for="itemName" th:text="#{item.itemName}"></label>
```
* Accpet-language 헤더를 기반으로 국제화 파일을 선택한다. Accpet-language가 en_US 의 경우 messages_en_US -> messages_en -> messages 순서로 찾는다.
* Locale Resolver의 구현체를 변경해서 쿠키나 세션기반의 Locale 선택기능을 사용할 수 있다. 이러한 방식을 사용하면 사용자가 직접 원하는 언어를 선택할 수 있도록 바꿀 수 있다.