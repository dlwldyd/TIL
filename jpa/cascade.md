# 영속성 전이, 고아 객체
## 영속성 전이
```java
@Entity
public class Parent {

    @Id
    @GeneratedValue
    @Column(name = "PARENT_ID")
    private Long id;

    private String name;

    @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL)
    private List<Child> childList = new ArrayList<>();

    public void addChild(Child child) {
        childList.add(child);
        child.setParent(this);
    }

    ...

}
```
```java
@Entity
public class Child {

    @Id
    @GeneratedValue
    @Column(name = "CHILD_ID")
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    private Parent parent;

    ...

}
```
```java
Parent parent = new Parent();
Child child1 = new Child();
Child child2 = new Child();

parent.addChild(child1);
parent.addChild(child2);

entityManager.persist(parent);
//entityManager.persist(child1); 필요 없음
//entityManager.persist(child2); 필요 없음

entityManager.remove(parent);
//entityManager.remove(child1); 필요 없음
//entityManager.remove(child2); 필요 없음
```
* child 엔티티가 parent 엔티티 하나와만 연관관계에 있고 lifecycle이 같을 때 사용해야 한다.(다른 엔티티와도 연관관계가 있으면 사용하면 안됨)
* CascadeType.ALL : Parent 를 persist, remove 할 때 연관된 객체도 전부 persist, remove 된다.
* CascadeType.PERSIST : Parent 를 persist 할 때 연관된 객체도 전부 persist 된다.
* CascadeType.REMOVE : Parent 를 remove 할 때 연관된 객체도 전부 remove 된다.
* 나머지는 잘 사용하지 않는다.
## 고아 객체
```java
@Entity
public class Parent {

    @Id
    @GeneratedValue
    @Column(name = "PARENT_ID")
    private Long id;

    private String name;

    @OneToMany(mappedBy = "parent", orphanRemoval = true)
    private List<Child> childList = new ArrayList<>();

    public void addChild(Child child) {
        childList.add(child);
        child.setParent(this);
    }

    ...

}
```
```java
@Entity
public class Child {

    @Id
    @GeneratedValue
    @Column(name = "CHILD_ID")
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    private Parent parent;

    ...

}
```
```java
Parent parent = new Parent();
Child child1 = new Child();
Child child2 = new Child();

parent.addChild(child1);
parent.addChild(child2);

entityManager.persist(parent);
entityManager.persist(child1);
entityManager.persist(child2);

entityManager.remove(parent);
//entityManager.remove(child1); 필요 없음
//entityManager.remove(child2); 필요 없음

entityManager.persist(parent);
entityManager.persist(child1);
entityManager.persist(child2);

Parent findParent = entityManager.find(Parent.class, parent.getId());
List<Child> childList = findParent.getChildList();
childList.remove(1);
childList.remove(0);

//entityManager.remove(child1); 필요 없음
//entityManager.remove(child2); 필요 없음
```
* orphanRemoval이 true일 때 childList 에서 빠진 엔티티는 Child 테이블에서 삭제된다.(@OneToMany, @OneToOne에만 있음)
* parent를 삭제하면 childList에 있는 모든 child가 테이블에서 삭제된다.(CascadeType.REMOVE 처럼 동작)
* child 엔티티가 parent 엔티티와만 연관관계에 있을 때 사용해야한다.