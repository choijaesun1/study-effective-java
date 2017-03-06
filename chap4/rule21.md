## Rule 21. 전략을 표현하고 싶을 때는 함수 객체를 사용하라

### 함수 객체

  - 자바는 함수 포인터를 지원하지 않지만 객체 참조를 통해 비슷한 효과를 달성할 수 있다.
  - 인자로 전달된 객체에 뭔가를 하는 메서드를 정의할 수 있는데 이 메서드만을 가지고 잇는 객체는 해당 메서드의 포인터 구실을 한다. (**함수 객체**)
  - ex)

  ```JAVA
  // 이 객체는 문자열을 비교하는 데 사용될 수 있는 실행 가능 전략
  class StringLengthComparator{
    public int compare(String s1, String s2){
      return s1.length() - s2.length();
   }
}
  ```
#### stateless 클래스라 필드가 필요 없으므로 싱글턴 패턴으로 쓸데 없는 객체 생성을 피한다

  ```JAVA
  class StringLengthComparator{
   private StringLengthComparator() {}
   public static final StringLengthComparator INSTANCE = new StringLengthComparator();
    public int compare(String s1, String s2){
      return s1.length() - s2.length();
   }
}
  ```

  - 근데 이 친구는 인자의 자료형이 맞아야 해서 다른 전략을 전달할 수 없다.

#### **실행 가능 전략 클레스에 어울리는 전략 인터페이스를 정의할 필요가 있다.**

  ```JAVA
  public interface Comparator<T>{
   public int compare(T t1, T t2);
}

class StringLengthComparator implements Comparator<String>{
  //.. 위와 동일
}
  ```

  - 실행 가능한 익명 클래스로 정의하는 경우
  ```JAVA
  Arrays.sort(stringArray, new Comparator<String>(){
    public int compare(String s1, String s2){
      return s1.length() - s2.length();
   }
}
  ```
  - but 쓸 때마다 새로운 객체

#### 실행 가능 전략 클래스를 public으로 공개하지 않아도 된다. 반복적으로 사용된다면 public static 필드를 갖는 호스트 클래스를 정의하는 것도 방법

  ```JAVA  
Class Host{
 private static class StrLenCmp implements Comparator<String>, Serializable{
    public int compare(String s1, String s2){
       return s1.length() - s2.length();
    }
 }

 public static final Comparator<String> STRING_LENGTH_COMPARATOR = new StrLenCmp(); //얘가 공개 되는거!!
 ...
}

  ```
