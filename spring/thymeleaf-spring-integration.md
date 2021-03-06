# 타임리프, 스프링 통합
```java
@Getter
@Setter
@AllArgsConstructor
public class Item {

    private Long id;
    private String itemName;
    private Integer price;
    private Integer quantity;
    private boolean open; //판매 여부
    private List<String> regions; //등록 지역
    private ItemType itemType; //상품 종류
    private String deliveryCode; //배송 방식

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}
```
* 이 문서에서 사용하는 Item 객체
## object, field
```html
<!--컨트롤러에서 model.addAttribute("item", new Item());로 Item객체 전달-->
<form th:action th:object="${item}" method="post">
    <div>
        <label for="itemName">상품명</label>
        <input type="text" th:field="*{itemName}" class="form-control" placeholder="이름을 입력하세요">
    </div>
    <div>
        <label for="price">가격</label>
        <input type="text" th:field="*{price}" class="form-control" placeholder="가격을 입력하세요">
    </div>
    <div>
        <label for="quantity">수량</label>
        <input type="text" th:field="*{quantity}" class="form-control" placeholder="수량을 입력하세요">
    </div>
</form>
```
* th:object="${item}" : \<form> 에서 사용할 객체를 지정한다. 선택 변수 식( *{...} )을 적용할 수 있다.
* \*{itemName}은 ${item.itemName}과 같다.
* th:field="*{itemName}" : th:field 는 id , name , value 속성을 모두 자동으로 만들어준다.
  + id : th:field 에서 지정한 변수 이름과 같다. id="itemName"
  + name : th:field 에서 지정한 변수 이름과 같다. name="itemName"
  + value : th:field 에서 지정한 변수의 값을 사용한다. value=""
* th:field가 적용된 객체는 자동으로 타입 컨버팅 된다.
## 히든 필드
### 타임리프를 사용하지 않을 경우
```html
<form th:action th:object="${item}" method="post">
    <div>판매 여부</div>
    <div>
        <div class="form-check">
            <input type="checkbox" id="open" name="open" class="form-check-input">
            <label for="open" class="form-check-label">판매 오픈</label>
        </div>
    </div>
</form>
```
```
item.open=true //체크 박스를 선택하는 경우
item.open=null //체크 박스를 선택하지 않는 경우
```
* 체크박스를 체크하면 HTML Form에서 open=on 이라는 값이 넘어간다. 스프링은 on 이라는 문자를 true 타입으로 변환해준다. 반면에 HTML에서 체크박스를 선택하지 않고 폼을 전송하면 open 이라는 필드 자체가 서버로 전송되지 않는다.
* 체크박스 해제를 인식하기 위해서는 \<input type="hidden" name="_open" value="on"/>라는 히든필드가 필요하다. 이 경우 스프링은 체크 박스 헤제를 인식하여 item.open에 false를 할당한다.
### 타임리프를 사용할 경우
```html
<form th:action th:object="${item}" method="post">
    <div>판매 여부</div>
    <div>
        <div class="form-check">
            <input type="checkbox" th:field="*{open}" class="form-check input">
            <label for="open" class="form-check-label">판매 오픈</label>
        </div>
    </div>
</form>
```
```html
<!--렌더링 결과-->
<div>판매 여부</div>
<div>
    <div class="form-check">
        <input type="checkbox" id="open" class="form-check-input" name="open" value="true">
        <input type="hidden" name="_open" value="on"/>
        <label for="open" class="form-check-label">판매 오픈</label>
    </div>
</div>
```
* 타임리프를 사용하면 체크박스의 히든 필드를 자동을 생성해준다.
## 체크박스, Map 사용
```java
@ModelAttribute("regions")
public Map<String, String> regions() {
    Map<String, String> regions = new LinkedHashMap<>();
    regions.put("SEOUL", "서울");
    regions.put("BUSAN", "부산");
    regions.put("JEJU", "제주");
    return regions;
}
```
```html
<form th:action th:object="${item}" method="post">
    <div>
        <div>등록 지역</div>
        <div th:each="region : ${regions}" class="form-check form-check-inline">
            <input type="checkbox" th:field="*{regions}" th:value="${region.key}" class="form-check-input">
            <label th:for="${#ids.prev('regions')}" th:text="${region.value}" class="form-check-label">서울</label>
        </div>
    </div>
</form>
```
```html
<!--렌더링 결과-->
<div>
    <div>등록 지역</div>
    <div class="form-check form-check-inline">
        <input type="checkbox" value="SEOUL" class="form-check-input" id="regions1" name="regions">
        <input type="hidden" name="_regions" value="on"/>
        <label for="regions1" class="form-check-label">서울</label>
    </div>
    <div class="form-check form-check-inline">
        <input type="checkbox" value="BUSAN" class="form-check-input" id="regions2" name="regions">
        <input type="hidden" name="_regions" value="on"/>
        <label for="regions2" class="form-check-label">부산</label>
    </div>
    <div class="form-check form-check-inline">
        <input type="checkbox" value="JEJU" class="form-check-input" id="regions3" name="regions">
        <input type="hidden" name="_regions" value="on"/>
        <label for="regions3" class="form-check-label">제주</label>
    </div>
</div>
```
* 멀티 체크박스는 같은 이름의 여러 체크박스를 만들 수 있다. 그런데 문제는 이렇게 반복해서 HTML 태그를 생성할 때, 생성된 HTML 태그 속성에서 name 은 같아도 되지만, id 는 모두 달라야 한다. 따라서 타임리프는 체크박스를 each 루프 안에서 반복해서 만들 때 임의로 1 , 2 , 3 숫자를 뒤에 붙여준다.
* 타임리프는 #ids.prev(...) , #ids.next(...), #ids.seq(...) 을 이용해서 동적으로 생성되는 id 값을 사용할 수 있다.
## 라디오 버튼, enum 사용
```java
@Getter
@RequiredArgsConstructor
public enum ItemType {
    BOOK("도서"), FOOD("음식"), ETC("기타");

    private final String description;
}
```
```java
@ModelAttribute("itemTypes")
public ItemType[] itemTypes() {
    return ItemType.values();
}
```
```html
<form th:action th:object="${item}" method="post">
    <div>
        <div>상품 종류</div>
        <div th:each="type : ${itemTypes}" class="form-check form-check-inline">
            <input type="radio" th:field="*{itemType}" th:value="${type.name()}" class="form-check-input">
            <label th:for="${#ids.prev('itemType')}" th:text="${type.description}" class="form-check-label">
                BOOK
            </label>
        </div>
    </div>
</form>
```
```html
<!--렌더링 결과-->
<div>
    <div>상품 종류</div>
    <div class="form-check form-check-inline">
        <input type="radio" value="BOOK" class="form-check-input" id="itemType1" name="itemType">
        <label for="itemType1" class="form-check-label">도서</label>
    </div>
    <div class="form-check form-check-inline">
        <input type="radio" value="FOOD" class="form-check-input" id="itemType2" name="itemType" checked="checked">
        <label for="itemType2" class="form-check-label">음식</label>
    </div>
    <div class="form-check form-check-inline">
        <input type="radio" value="ETC" class="form-check-input" id="itemType3" name="itemType">
        <label for="itemType3" class="form-check-label">기타</label>
    </div>
</div>
```
* 라디오 버튼은 이미 선택이 되어 있다면, 수정시에도 항상 하나를 선택하도록 되어 있으므로 체크박스와 달리 별도의 히든 필드를 생성하지 않는다.
## 셀렉트박스, List 사용
```java
@Getter
@Setter
@AllArgsConstructor
public class DeliveryCode {
    private String code;
    private String displayName;
}
```
```java
@ModelAttribute("deliveryCodes")
public List<DeliveryCode> deliveryCodes() {
    List<DeliveryCode> deliveryCodes = new ArrayList<>();
    deliveryCodes.add(new DeliveryCode("FAST", "빠른 배송"));
    deliveryCodes.add(new DeliveryCode("NORMAL", "일반 배송"));
    deliveryCodes.add(new DeliveryCode("SLOW", "느린 배송"));
    return deliveryCodes;
}
```
```html
<form th:action th:object="${item}" method="post">
    <div>
        <div>배송 방식</div>
        <select th:field="*{deliveryCode}" class="form-select">
            <option value="">==배송 방식 선택==</option>
            <option th:each="deliveryCode : ${deliveryCodes}" th:value="${deliveryCode.code}" th:text="${deliveryCode.displayName}">FAST</option>
        </select>
    </div>
</form>
```
```html
<!--렌더링 결과-->
<div>
    <div>>배송 방식</div>
    <select class="form-select" id="deliveryCode" name="deliveryCode">
        <option value="">==배송 방식 선택==</option>
        <option value="FAST">빠른 배송</option>
        <option value="NORMAL">일반 배송</option>
        <option value="SLOW">느린 배송</option>
    </select>
</div>
```
