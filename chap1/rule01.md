## Rule 1. 생성자 대신 정적 팩토리 메서드를 사용할 수 없는지 생각해 보라

### Static Factory Method ?

 - Constructor를 통한 객체 생성 이외에 객체를 반환, 생성 해줄 수 있는 Method..?
public static method 로서 외부 클래스에서 바로 접근할 수 있는 method 로, 생성자의 역할을 하는 친구 ( 참고 : http://aroundck.tistory.com/2 )
 - ex> java Boolean 객체
```
public static Boolean valueOf(boolean b) {
	return b? Boolean.TRUE : Boolean.FALSE;
}
```
### 장점
####생성자와는 달리 Static Factory Method에는 이름이 있다.
 - 생성자는 어떤 객체가 만들어지는지 정확하게 알 수 없지만 Static Factory Method는 용도에 따라 작명해줄 수 있으므로 용도를 명확하게 하여 코드의 가독성을 높일 수 있다.
```
// Constructor
public Person()
public Person(String name)

//Static Factory Method
public static Person getInstance();
public static Person getInstanceWithName(String name);
```
 - 중복 시그니처를 사용할 수 있다.

```
// Constructor (불가)
public Person()
public Person(String name)
public Person(String address)

//Static Factory Method
public static Person getInstance();
public static Person getInstanceWithName(String name);
public static Person getInstanceWithAddress(String address);
```
 - 따라서 Naming 이 중요!

####생성자와는 달리 호출할 때마다 새로운 객체를 생성할 필요는 없다.
- 이미 만들어둔 객체를 활용할 수도 있고, Cache 해놓고 재사용하여 같은 객체가 불필요하게 중복으로 생성되는 일을 피할 수 있다.
- 객체 수를 제어할 수 있다.
- 객체를 생성하는 비용이 큰 경우 성능의 차이

####생성자와는 달리 반환값 자료형의 하위자료형 객체를 반환할 수 있다.
 ```
// Constructor = 해당 Class 의 객체만을 반환

// Static Factory Method
public static Person getInstance (JobType type) {
    if(type.equals(JobType.STUDENT)) {
      return new Student();
    }else {
      return new Faculty();
    }
}
```
 - 메서드에 주어지는 인자를 이용하면 어떤 객체를 만들지 동적으로 결정 할 수 있으므로 유지보수하거나 확장하기 쉽다.
 - java.util.EnumSet 예를 보면 API를 사용하는 사용자가 어떤 객체가 반환되는지 알 필요가 없다. EnumSet의 하위 클래스가 반환 될 것이라는 것만 중요! (코드참조 : https://github.com/kkd927/effective-java-study/blob/master/chap2/rule1.md )

####형인자 자료형 객체를 만들 때 편하다.
 - 현재 1.7부터 다이아몬드 연산자 (<>) 를 통해서 생성자에 자료형 유추가 가능해졌다.

###단점
####public이나 protected로 선언된 생성자가 없으므로 하위 클래스를 만들 수 없다.
 - 계승(inheritance) 대신 구성(Compostition) 기법을 쓰도록 장려한다.(Rule.16)

####정적팩터리 메서드가 다른 정적 메서드와 확인히 구분되지 않는다.
 - 주석을 통해 구분 짓거나 Naming에 신경을 써야한다.
