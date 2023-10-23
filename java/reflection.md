# reflection 사용법
```java
@NoArgsConstructor
public class TestClass {

    private Integer x;

    public List<String> methodName(List<String> args) {
        return return args;
    }
}
```
## 메소드 실행
```java
TestClass testClass = new TestClass();
TestClass.getMethod("methodName", List.class).invoke(testClass, args);
```
* getMethod 로 첫 번째 파라미터에 해당하는 메소드 정보를 가져온다.
* getMethod 의 두 번째 이후의 파라미터는 해당 메소드의 파라미터 타입이다.
* invoke 의 첫 번째 파라미터는 해당 메소드를 실행할 객체를 넣는다.
* invoke 의 두 번째 이후의 파라미터에는 해당 메소드에 들어갈 파라미터 값을 넣는다.
* getMethod 의 두 번째 이후 파라미터 개수와 invoke 의 두 번째 이후 파라미터 개수는 같아야한다.
## 생성자를 통한 객체 생성
```java
Class<TestClass> class = TestClass.class;
TestClass testClass = class.getConstructor().newInstance();
```
* newInstance() 안의 파라미터에 따라 불러오는 생성자가 다르다.
* 만약 해당하는 생성자가 없으면 Exception 발생
## field 값 수정
```java
TestClass testClass = new TestClass();
Field[] declaredFields = testClass.getDeclaredFields();
for (Field field : declaredFields) {
    field.setAccessible(true); // private field 수정하려면 필수
    field.set(testClass, 10);
}


Field field = testClass.getField("x");
field.setAccessible(true);
field.set(testClass, 10);
```
* getDeclaredFields() 를 통해 모든 field 정보를 가져올 수 있다.
* getField({fieldName}) 을 사용하면 특정 field 정보를 가져올 수 있다.
* 만약 private field 의 값을 수정하려면 setAccessible()을 해줘야한다.
* set()의 첫 번째 파라미터에는 해당 field 값을 수정하려는 실제 객체를 넣는다.
* set()의 두 번째 파라미터에는 수정할 값을 넣는다.