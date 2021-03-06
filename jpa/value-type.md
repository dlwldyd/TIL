# 값 타입
## 임베디드 타입
```java
@Embeddable
public class Address {

    private String street;
    private String city;
    private String zipcode;

    //값 타입이 임베디드 타입일 경우 기본생성자 필수
    public Address() {
    }

    public Address(String street, String city, String zipcode) {
        this.street = street;
        this.city = city;
        this.zipcode = zipcode;
    }

    //equals() and hashCode() 만들어 주는게 좋음
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Address address = (Address) o;
        return Objects.equals(getStreet(), address.getStreet()) && Objects.equals(getCity(), address.getCity()) && Objects.equals(getZipcode(), address.getZipcode());
    }

    @Override
    public int hashCode() {
        return Objects.hash(getStreet(), getCity(), getZipcode());
    }

    ...

}
```
```java
@Entity
public class Member {
    
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    //기본 값 타입
    @Column(name = "USERNAME")
    private String name;

    //임베디드 타입
    @Embedded
    private Address homeAddress;

    @Embedded
    @AttributeOverrides(
            {@AttributeOverride(name = "city", column = @Column(name = "WORK_CITY")),
            @AttributeOverride(name = "street", column = @Column(name = "WORK_STREET")),
            @AttributeOverride(name = "zipcode", column = @Column(name = "WORK_ZIPCODE"))})
    private Address workAddress;

    // private String street;
    // private String city;
    // private String zipcode;

    ...

}
```
* 연관된 기본 값 타입을 모아 하나의 객체로 만들어 임베디드 타입으로 만들 수 있다.
* 기본 값 타입을 모아 하나의 객체로 만들어도 매핑하는 테이블에는 아무런 변화도 없다.
* @Embeddable을 임베디드 타입을 정의하는 객체에 붙이고 @Embedded를 임베디드 타입을 사용하는 곳에 붙이면 된다.
* 기본 생성자는 필수이다. 단 기본생성자를 통해 객체를 생성할 일이 없으면(왠만하면 없다) public이 아닌 protected로 기본생성자를 만드는게 더 안전하다.
* equals() and hashCode()를 오버라이드 하는 편이 좋다.(임베디드 타입 간 비교 시 필요, 비교하지 않으면 굳이 오버라이드 하지는 않아도 됨)
* 임베디드 타입을 정의하는 객체에 거기에 속한 기본 값 타입을 이용하여 의미있는 메서드를 만들 수 있다.
* 재사용성이 높다.
* 임베디드 타입은 setter를 만들지 않고 불변객체로 만드는 것이 좋다. 왜냐하면 하나의 객체를 여러 엔티티가 공유하는 경우에 setter를 통해 값을 바꾼다면 다른 엔티티에도 update 쿼리가 나갈 수 있기 때문이다. 만약 값을 바꿀 필요가 있다면 새로운 객체를 생성해서 할당하자.
* 하나의 엔티티에 같은 임베디드 타입을 여러번 사용할 때는 그대로 사용하면 컬럼명이 중복되어 예외가 발생하기 때문에 @AttributeOverrides 를 통해 컬럼명을 바꿔줘야 한다.
* 임베디드 타입은 해당 객체가 엔티티가 아니라 임베디드 타입이라 판단될 때만 사용해야한다.
## 값 타입 컬렉션
```java
@Entity
public class Member {
    
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @ElementCollection(fetch = FetchType.LAZY)
    @CollectionTable(name = "FAVORITE_FOOD", joinColumns = @JoinColumn(name = "MEMBER_ID"))
    @Column(name = "FOOD_NAME")
    private Set<String> favoriteFoods = new HashSet<>();

/*값 타입 컬렉션에서 List는 사용하지 않는 것이 좋다.
    
    @ElementCollection(fetch = FetchType.LAZY)
    @CollectionTable(name = "ADDRESS", joinColumns = @JoinColumn(name = "MEMBER_ID"))
    @AttributeOverrides(
            {@AttributeOverride(name = "city", column = @Column(name = "OLD_CITY")),
                    @AttributeOverride(name = "street", column = @Column(name = "OLD_STREET")),
                    @AttributeOverride(name = "zipcode", column = @Column(name = "OLD_ZIPCODE"))})
    private List<Address> addressHistory = new ArrayList<>();
*/
    
    ...

}
```
* 값 타입 컬렉션을 사용하면 값 타입 컬렉션을 위한 테이블이 생성되고 해당 테이블의 기본 키는 기본적으로 모든 컬럼이 된다.
* 값 타입 컬렉션에서는 List를 사용하지 않는 것이 좋다. 왜냐하면 Set 같은 경우에는 같은 값이 중복으로 들어갈 수가 없고 Map의 경우에는 같은 값이 들어가더라도 Map의 키는 중복되지 않는다. 따라서 테이블의 row를 식별 가능하기 때문에 remove를 통해 원하는 값을 삭제할 수 있다. 하지만 List의 경우에는 같은 값이 중복해서 들어갈 수 있기 때문에 테이블의 row를 식별할 수 없다. 따라서 List에서 값을 remove 하면 엔티티의 외래키(여기서는 MEMBER_ID)와 일치하는 모든 row를 삭제하고 전부 다시 add한다.
* 값 타입 컬렉션은 지연로딩 전략을 기본 값으로 사용한다.
* 값 타입 컬렉션은 엔티티를 일대다 매핑 후 casacde = CascadeType.ALL, orphanRemoval = true 로 설정한 것처럼 동작한다.
* 값 타입 컬렉션은 엔티티가 아닌 값 타입이라 판단될 때만 사용하고, 엔티티라 판단되면 일대다 매핑 후 casacde = CascadeType.ALL, orphanRemoval = true로 설정해서 사용하면 된다.