## Rule 17. 계승을 위한 설계와 문서를 갖추거나, 그럴 수 없다면 계승을 금지하라.

#### 재정의 가능 메서드를 내부적으로 어떻게 사용하는지(self-use) 반드시 문서에 남겨라

 - 재정의 가능한 메서드가 호출 되는 모든 상황을 문서로 남겨라
 - 긍까 내가 재정의를 했는데 이게 클라이언트가 모르는 곳에서 호출 될 수 도 있으니 그걸 남기라는 말인듯

#### 문서만 제대로 썼다고 계승에 적합한 설계가 되는 것은 아니다.

 - 클래스 내부 동작에 개입할 수 있는 훅(hooks)을 신중하게 고른 protected 메서드 형태로 제공해야 한다.

#### 클래스를 테스트할 유일한 방법은 하위 클래스를 직접 만들어 보는 것

 - 하위 클래스를 한 3개쯤 만들어보고 그 중 하나는 다른 사람이 만들어보라
 - 어떤 멤버를 protected로 해야하는지 실수는 없었는지가 눈에 보인다.
 - protected는 한번 릴리스 하면 영원히 고수해야 하기 때문에 꼭꼭 하위클래스를 만드는 테스트를 하자!

#### 계승을 위한 설계시 유의할 점

##### 1. 생성자는 재정의 가능 메서드를 호출해서는 안된다.

 - 잘못된 사례

 ```JAVA
 public class Super {
   //생성자가 재정의 가능 메서드를 호출하는 잘못된 사례
   public Super() {
       overrideMe();
   }
   public void overrideMe(){ }
}
public final class Sub extends Super{

   private final Date date;

   Sub(){
       date = new Date();
   }

   //상위 클래스 생성자가 호출하게 되는 재정의 메서드
   @Override public void overrideMe(){
       System.out.println(date);
   }

   public static void main(String[] args){
       Sub sub = new Sub();
       sub.overrideMe();
     }
}
 ```

결과는 null 이 출력된다. (원래는 NPE가 발생하는데 println에서 이를 처리해주는거임)

##### 2. Clonable 이나 Serializable 을 구현하기로 했다면 clone 이나 readObject 메서드 안에서 재정의 가능한 메서드를 호출하지 않도록 주의해야한다.

- readObject안에서 재정의 가능한 메서드를 호출하도록 한다면 하위 클래스 객체가 완전히 역직렬화가 되기 전에 해당 메서드가 실행된다.
- clone의 경우는 복사본의 객체 상태를 미처 수정하기 전에 해당 메서드가 실행

##### 3. Serializable 인터페이스를 구현하는 계승용 클래스에 readResolve와 writeReplace 메서드가 있다면 이 메서드들은 private이 아니라 protected로 선언해야 한다.

- (..? Serializable 많이 안써봐서 모르겠당... 와 닿진 않네욤..)

### 결론 :

- 계승을 반드시 허용해야 한다고 느껴지면, 재정의 가능 메서드는 절대로 호출하지 않도록 하고 그 사실을 반드시 문서에 남겨라.

- 문서화 하지 않은 클래스에 대한 하위 클래스는 만들지 않도록 하라.
