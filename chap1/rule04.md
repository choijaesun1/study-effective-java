## Rule 4. 객체 생성을 막을 때는 private 생성자를 사용하라
 - java.util.Collections, java.utils.Arrays 등 처럼 객체를 만들려는 목적이 아닌 유틸리티 클래스 들이 있다. 이러한 클래스는 객체를 만들 목적의 클래스가 아니다.
 - abstact 로 선언해도 하위 클래스를 정의하는 순간 객체 생성 가능.
 - abstact 선언하여 api 로 제공하게 되면 abstract니까 계승해서 써라 이렇게 이해할 수 있다.
 
 ### private 생성자를 넣어 객체 생성을 방지하자!
 - 예제
 
 ```JAVA
 public class UtiliyClass {
 // 기본 생성자가 자동으로 생성되지 못하도록 하여 객체 생성 방지
  private UtiliyClass() {
    // 실수로 객체를 생성하게 되면 바로 알 수 있다.
    throw new AssertionError();
  }
  //..
 }
```
