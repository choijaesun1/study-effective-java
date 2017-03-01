## 인터페이스는 자료형을 정의할 때만 사용하라

인터페이스를 구현하는 클래스를 만들게 되면, 그 인터페이스는 해당 클래스의 객체를 참조할 수 있는 자료형 역할을 하게 된다.

**상수 인터페이스** 는 인터페이스를 잘못 사용한 것

```JAVA
public interface PhysicalConstants {
    static final double AVOGADROS_NUMBER = 6.02214199e23;
    static final double BOLTZMANN_CONSTANT = 1.3806503e-23;
    static final double ELECTRON_MASS = 9.10938188e-31;
}
```

상수를 API의 일부로 공개하고 싶을 때 더 좋은 방법 :

 - 상수가 강하게 연결되어있는 경우는 그냥 그 안으로 넣는다.
 - Enum 사용
 - 그렇지 않다면 유틸리티 클래스에 넣어서 공개
 ```JAVA
 public class PhysicalConstants {
    private PhysicalConstants(){} // 객체 생성을 막음
    
    public static final double AVOGADROS_NUMBER = 6.02214199e23;
    public static final double BOLTZMANN_CONSTANT = 1.3806503e-23;
    public static final double ELECTRON_MASS = 9.10938188e-31;
}
 ```
