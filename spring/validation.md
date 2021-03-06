# 검증
## 자바 코드를 통한 검증
```properties
#properties 파일
range.item.price=가격은 {0} ~ {1} 까지 허용합니다.
range={0} ~ {1} 까지 허용합니다.
max.item.quantity=수량은 최대 {0} 까지 허용합니다.
max=최대 {0} 까지 허용합니다.
totalPriceMin=전체 가격은 {0}원 이상이어야 합니다. 현재 값 = {1}
```
```java
@PostMapping("/add")
public String addItem(@ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

    //검증 로직
    if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
        //오류가 발생한 객체이름, 오류필드, 사용자가 입력한 값, 타입 오류 같은 바인딩 실패인지, 검증 실패인지 구분 값, 메시지 코드, 메시지에서 사용하는 인자, 기본 오류 메시지
        bindingResult.addError(new FieldError("item", "price", item.getPrice(), false, new String[]{"range.item.price"}, new Object[]{1000, 1000000},  null));
    }
    if (item.getQuantity() == null || item.getQuantity() >= 9999) {
        bindingResult.addError(new FieldError("item", "quantity", item.getQuantity(), false, new String[]{"max.item.quantity"}, new Object[]{9999}, null));
    }

    //특정 필드가 아닌 복합 룰 검증
    if (item.getPrice() != null && item.getQuantity() != null) {
        int resultPrice = item.getPrice() * item.getQuantity();
        if (resultPrice < 10000) {
            bindingResult.addError(new ObjectError("item", new String[]{"totalPriceMin"}, new Object[]{10000, resultPrice}, null));
        }
    }

    //검증 실패시
    if (bindingResult.hasErrors()) {
        return "validation/addForm";
    }

    ...
}
```
* BindingResult는 검증하려는 데이터 뒤에 넣어야 한다.
* BindingResult가 있으면 @ModelAttribute 에 데이터 바인딩 시 오류가 발생해도 오류페이지로 이동하는 대신에 오류 정보를 BindingResult에 담아서 컨트롤러를 호출한다.
* BindingResult 객체의 addError함수를 통해 FieldError객체를 추가하면 필드 에러를 추가할 수 있다.
* ObjectError 객체를 추가하면 글로벌 에러를 추가할 수 있다.
* 메시지를 이용하여 한 곳에서 오류 메시지를 관리할 수 있다.
```java
@Component
public class ItemValidator implements Validator {

    @Override
    public boolean supports(Class<?> clazz) {
        return Item.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        Item item = (Item) target;

        //검증 로직
        if (item.getPrice() == null || item.getPrice() < 1000 || item.getPrice() > 1000000) {
            //오류 필드, 오류 코드(메시지 리졸버에 들어갈 값), 메시지에서 사용하는 인자, 기본 오류 메시지
            errors.rejectValue("price", "range", new Object[]{1000, 1000000}, null);
        }
        if (item.getQuantity() == null || item.getQuantity() >= 9999) {
            errors.rejectValue("quantity", "max", new Object[]{9999}, null);
        }


        //특정 필드가 아닌 복합 룰 검증
        if (item.getPrice() != null && item.getQuantity() != null) {
            int resultPrice = item.getPrice() * item.getQuantity();
            if (resultPrice < 10000) {
                errors.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
            }
        }
    }
}
```
```java
@Controller
@RequiredArgsConstructor
public class ValidationItemController{

    private final ItemRepository itemRepository;
    private final ItemValidator itemValidator;

    @InitBinder
    public void init(WebDataBinder dataBinder) {
        dataBinder.addValidators(itemValidator);
    }
    
    @PostMapping("/add")
    public String addItemV6(@Validated @ModelAttribute Item item, BindingResult bindingResult, RedirectAttributes redirectAttributes, Model model) {

        if (bindingResult.hasErrors()) {
            log.info("errors = {}", bindingResult);
            return "validation/addForm";
        }

        ...
    }

    ...
}
```
* 검증을 위한 객체를 따로 만들 때는 Validator 인터페이스를 상속해야한다.
* @Validated 어노테이션은 검증하려는 데이터에 넣어야 한다. @Validated 어노테이션을 넣은 객체에 자동으로 검증이 실행된다.
* @Validated 어노테이션을 사용하려면 WebDataBinder에 Validator를 추가하는 함수가 필요하다. 이 함수에는 @InitBinder 어노테이션을 지정해야한다.
* @initBinder는 해당 컨트롤러에만 영향을 준다. 다른 컨트롤러가 실행될 때에는 영향을 주지 않는다. 모든 컨트롤러에 적용하는 검증기를 등록하고 싶을 때는 다른 방식을 사용해야 한다.
* valdate함수의 매개변수 중 target 에는 검증할 데이터가 들어온다. errors 에는 Errors(BindingResult의 부모 클래스)객체가 들어온다.
* addError 함수 대신에 rejectValue 함수를 이용하여 필드에러를 추가할 수 있다. reject 함수를 이용하여 글로벌에러 또한 추가할 수 있다.
* MessageCodeResolver는 reject 함수나 rejectValue 함수에 들어간 오류코드(range, max 등)를 사용해서 메시지코드를 생성한다. 메시지코드는 우선순위가 `오류코드.객체이름.필드이름 -> 오류코드.필드이름 -> 오류코드.타입 -> 오류코드` 순이다. ex) range.item.price -> range.price -> range.java.lang.String -> range
* 타입 에러가 발생하면 우선순위가 `typeMismatch.객체이름.필드이름 -> typeMismatch.필드이름 -> typeMismatch.타입 -> typeMismatch` 순으로 오류코드를 생성해서 rejectValue 함수를 실행해 BindingResult에 오류를 추가한다.
```java
@SpringBootApplication
public class ItemServiceApplication implements WebMvcConfigurer {
    public static void main(String[] args) {
        SpringApplication.run(ItemServiceApplication.class, args);
    }
    @Override
    public Validator getValidator() {
        return new ItemValidator();
    }
}
```
* 메인 함수가 있는 클래스에 WebMvcConfigurer를 상속해서 글로벌 Validator를 등록할 수 있다.
* 직접 글로벌 Validator를 등록하면 Bean Validation을 사용할 때 스프링부트가 Bean Validator를 글로벌 Validator로 자동으로 등록해주지 않는다. 따라서 Bean Validation을 사용한다면 꼭 필요한 경우가 아닐 때는 글로벌 Validator를 등록하지 않는 것이 좋다.
```html
<div th:if="${#fields.hasGlobalErrors()}">
    <p th:each="err : ${#fields.globalErrors()}" class="field-error" th:text="${err}">글로벌 오류 메시지</p>
</div>
```
```html
<input type="text" id="price" th:field="*{price}" th:errorclass="field-error" class="form-control">
<div class="field-error" th:errors="*{price}">
    가격 오류
</div>
```
* 타임리프는 #fields 로 BindingResult 가 제공하는 검증 오류에 접근할 수 있다.
* th:errors는 해당 필드에 오류가 있는 경우에 태그를 출력한다. th:if 의 편의 버전이다.
* th:errorclass는 th:field 에서 지정한 필드에 오류가 있으면 th:classappend와 같이 class 정보를 추가한다
## Bean Validation(어노테이션 기반 검증)
```java
@Getter
@Setter
@AllArgsConstructor
public class ItemSaveForm {

    @NotBlank(message = "공백은 입력할 수 없습니다.")
    private String itemName;

    @NotNull
    @Range(min = 1000, max = 1000000)
    private Integer price;

    @NotNull
    @Max(value = 9999)
    private Integer quantity;
}
```
```java
@PostMapping("/add")
public String addItem(@Validated @ModelAttribute ItemSaveForm form, BindingResult bindingResult, RedirectAttributes redirectAttributes) {

    if (form.getPrice() != null && form.getQuantity() != null) {
        int resultPrice = form.getPrice() * form.getQuantity();
        if (resultPrice < 10000) {
            bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
        }
    }

    if (bindingResult.hasErrors()) {
        return "validation/addForm";
    }

    ...
}
```
```properties
totalPriceMin=가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}

#{0}에는 필드 이름이 들어간다.(itemName, price, quantity)
NotBlank.item.itemName=상품이름을 적어주세요
Range.item.price=가격은 {2}원에서 {1}원까지 허용됩니다.
Max.item.quantity=최대 {1}개 까지  등록할 수 있습니다.

NotBlank={0} 공백X 
Range={0}, {2} ~ {1} 허용
Max={0}, 최대 {1}
```
* Bean Validation을 사용하면 어노테이션만을 사용해서 간단하게 검증기능을 구현할 수 있다.
* 오류코드는 어노테이션 이름으로 등록된다.
* 글로벌 에러의 경우에는 @ScriptAssert를 통해 등록할 수 있지만 제약이 많아 자바코드를 통해 등록하는 것이 더 좋다.