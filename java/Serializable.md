# 직렬화(Serializable)
* 자바 시스템 내부에서 사용하는 객체를 외부의 자바 시스템에서도 사용할 수 있도록 byte 형태로 데이터를 변환하고 byte 형태로 변환된 데이터를 다시 객체로 변환하는 기술이다. 정확히 말하면 객체를 byte 형태로 변환 하는 것을 직렬화라 하고, byte 형태로 변환된 데이터를 다시 객체로 변환하는 것을 역직렬화라 한다. 우리가 만든 객체를 파일에 쓰고 다른 자바 시스템이 해당 파일을 읽도록 하거나 해당 객체를 다른 서버로 보내거나 받을 수 있도록 하려면 반드시 Serializable 인터페이스를 구현해야 한다.
```java
@Getter
@Setter
@ToString
@AllArgsConstructor
public class Member implements Serializable {

    // private static final long serialVersionUID = 9814965416451L;
    private String username;
    private String password;
    // transient private String password;
}
```
```java
public class JavaSystemA {
    public static void main(String[] args) {
        JavaSystemA javaSystemA = new JavaSystemA();
        String fullPath = "C:/test/test.md";

        Member member = new Member("user", "1111");
        javaSystemA.saveObject(fullPath, member);
    }

    public void saveObject(String fullPath, Member member) {
        FileOutputStream fos = null;
        ObjectOutputStream oos = null;
        try {
            fos = new FileOutputStream(fullPath);
            oos = new ObjectOutputStream(fos);
            oos.writeObject(member);
            System.out.println("Write Success");
        } catch (Exception e) { 
            e.printStackTrace();
        } finally {
            if (oos != null) {
                try {
                    oos.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            if (fos != null) {
                try {
                    fos.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```
```java
public class JavaSystemB {
    public static void main(String[] args) {
        JavaSystemB javaSystemB = new JavaSystemB();
        String fullPath = "C:/test/test.md";
        javaSystemB.loadObject(fullPath);
    }

    public void loadObject(String fullPath) {
        FileInputStream fis = null;
        ObjectInputStream ois = null;
        try {
            fis = new FileInputStream(fullPath);
            ois = new ObjectInputStream(fis);
            Object obj = ois.readObject();
            Member member = (Member)obj;
            System.out.println(member);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (ois != null) {
                try {
                    ois.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
        if (fis != null) {
            try {
                fis.close();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}
```
* A라는 자바시스템에서 Member 객체를 파일의 형태로 저장하고 B라는 자바시스템이 해당 파일을 읽을 때 만약 "Member"라는 같은 이름을 가진 객체라 하더라도 serialVersionUID 값이 다르다면 다른 클래스라고 인식하기 때문에 해당 파일을 읽을 수 없다.
* serialVersionUID 값이 같더라도 변수의 개수나 타입이 다르다면 이 경우에도 다른 클래스라고 인식해 해당 파일을 읽을 수 없다.
* serialVersionUID 값은 직접 넣어줄 수도 있고(직접 넣는게 권장됨), 직접 넣지 않더라도 컴파일 시 자동으로 생성된다.
* transient 예약어를 사용하여 선언한 변수는 Serializable의 대상에서 제외된다. 만약 transient 예약어를 사용한 변수에 값을 저장했다 할지라도 다른 자바 시스템에서 객체의 해당 변수를 읽을 때 값을 읽을 수 없다.
```java
@Getter
@Setter
@ToString
@AllArgsConstructor
public class Member implements Serializable {

    private String username;
    private String password;
}
```
```java
@Getter
@Setter
@ToString
@AllArgsConstructor
public class MemberChild extends Member {
    private int age;
}
```
* 부모 클래스인 Member가 Serializable을 구현했을 때 부모 클래스의 자식 클래스인 MemberChild도 직렬화가 가능하다.
* MemberChild를 직렬화하면 부모 클래스의 username, password도 같이 직렬화가 된다.
```java
@Getter
@Setter
@ToString
@AllArgsConstructor
public class Member {

    private String username;
    private String password;
}
```
```java
@Getter
@Setter
@ToString
@AllArgsConstructor
public class MemberChild extends Member implements Serializable {
    private int age;
}
```
* 부모 클래스인 Member가 Serializable을 구현하지 않았기 때문에 자식 클래스인 MemberChild를 직렬화 할 때 username, password는 직렬화 대상에서 제외된다.
```java
@Getter
@Setter
@ToString
@AllArgsConstructor
public class Member implements Serializable {

    private String username;
    private String password;
    // private Object name = new Object();
    private Object name = new String("이름");
}
```
* Member 객체의 변수인 username과 password는 String 타입이고, String은 Serializable을 구현하는 객체이기 때문에 직렬화가 가능하다. 하지만 name의 타입인 Object는 Serializable을 구현하는 객체가 아니기 때문에 직렬화 할 수가 없다. 따라서 Member 객체의 직렬화를 시도하면 NotSerializableException이 발생한다.
* name의 타입이 Object라 하더라도 name이 가리키는 실제 객체가 직렬화 가능한 객체라면 Member 객체의 직렬화가 가능하다.