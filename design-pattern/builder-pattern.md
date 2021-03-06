# 빌더 패턴
생성자를 통해 인스턴스를 생성하려면 생성자를 일일이 정의해 주어야하고 생성자를 너무 많이 만들기 싫으면 만약 필드가 4개가 있는 클래스의 경우 new Product(null, null, null, null) -> 이런식으로 인스턴스를 만들 수도 있다. 하지만 이렇게 되면 만약 해당 클래스에 필드가 하나 추가되는 변경점이 생길 시 생성자를 통해 인스턴스를 생성하는 코드를 전부 바꿔야해서 코드를 유지보수 하는데 어려움이 생긴다. 또한 생성자에 들어갈 변수의 순서를 맞추어 주어야 하는 번거러움이 있고 가독성도 좋지않다. 

자바빈즈 패턴(setter)를 통해 멤버변수를 정의한다면 이러한 번거로움도 없고 가독성도 좋겠지만 만약 해당 멤버변수가 한번 정의되고 난 후 값이 바뀌면 안되는 경우에는 자바빈즈 패턴(setter)을 사용하기 어렵다. 

하지만 빌더 패턴을 사용하면 인스턴스를 생성할 때 인자를 가독성 좋게 넘길 수 있고, 순서를 맞출 필요가 없으며 생성자를 너무 많이 만들 필요도 없다. 또한 값이 바뀌면 안되는 멤버 변수의 setter함수를 만들지 않아 한 번 결정된 값을 계속 유지할 수 있다.

## 빌더 패턴 적용 예시
```java
// 읽기만 가능해야 함
@Getter
@RequiredArgsConstructor
pulic class Client {
    private final String name;
    private final String favoriteColor;
    private final String favoriteItem;
    private final String favoriteBrand;
}
```
```java
// Builder
@RequiredArgsConstructor
public class ClientBuilder {
    //필수
    private final String name;
    //선택
    private String favoriteColor = "";
    private String favoriteItem = "";
    private String favoriteBrand = "";

    public ClientBuilder setFavoriteColor(String favoriteColor) {
        this.favoriteColor = favoriteColor;
        return this;
    }
    
    public ClientBuilder setFavoriteItem(String favoriteItem) {
        this.favoriteItem = favoriteItem;
        return this;
    }

    public ClientBuilder setFavoriteBrand(String favoriteBrand) {
        this.favoriteBrand = favoriteBrand;
        return this;
    }

    public build() {
        return new Client(name, favoriteColor, favoriteItem, favoriteBrand);
    }
}
```
```java
ClientBuilder clientBuilder = new ClientBuilder("John");
Client client = clientBuilder
                    .setFavoriteColor("blue")
                    .setFavoriteItem("shoes")
                    .setFavoriteBrand("Nike")
                    .build();
```
* Product(Client)에는 모든 필드이 final로 선언되었기 때문에 한번 객체를 생성하면 값을 바꿀 수 없다.
* 어떤 객체를 생성하려면 우선 그것의 빌더를 생성해야 한다. 성능이 매우매우매우 중요한 상황에서는 문제가 될 수 있다.
* 유연성이 좋다. 하나의 빌더는 여러 개의 객체를 생성하는데 사용될 수 있으며, 이러한 과정 중에 빌더의 매개변수는 다양하게 조정될 수 있다.
* 자바빈즈 패턴(setter) 처럼 가독성이 좋다.
* 생성자나 팩토리가 처리해야할 필드가 많고 필드 중 다수가 필수가 아니라면 빌더 패턴을 사용하는 것이 좋다.