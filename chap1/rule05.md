## Rule 5. 불필요한 객체는 만들지 마라

####1. 기능적으로 동일한 객체는 필요할때 만드는 것 보다 재사용하는 편이 낫다. 변경불가능한 객체는 언제나 재사용할 수 있다. (Rule.15)

```JAVA
 String s= new String("sth");
```
 이 코드는 실행 될 때마다 새로운 쓸데없는 객체를 만들게 된다.
 ```JAVA
  String s = "sth";
 ```
 이렇게 하면 실행할 때마다 객체를 만드는 대신 동일한 객체를 사용한다.게다가 같은 가상 머신에서 실행되는 모든 코드가 해당 객체를 재사용하게 된다.
 (문자열 상수를 지정하는 경우는 해당 문자열을 자바 가상 머신 전체에서 공유한다고 함.)

####2. (규칙1) Static Factory Method를 사용하면 불필요한 객체 생성을 줄일 수 있다. (ex> Boolean.valueOf(String))
####3. 변경 가능한 객체를 재사용할 수 있다. (변경할 일이 없다면)
 - 나쁜 예
 ```JAVA
 public class Person{
     private final Date birthDate;

     //다른 필드와 메서드, 생성자는 생략
     //이렇게 하면 안된다!
     public boolean isBabyBoomer(){
         //생성 비용이 높은 객체를 쓸데없이 생성한다.
         Calendar gtmCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));

         gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
         Date boomStart = gmtCal.getTime();
         gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
         Date boomEnd = gmtCal.getTime();

         return birthDate.compareTo(boomStart) >= 0 && birthDate.compareTo(boomEnd) < 0;
     }
 }
 ```

 - 개선한 코드 : Calendar, TimeZone 그리고 Date 객체를 클래스가 초기화 될 때 한 번만 만든다.
 ```JAVA
 public class Person{
     private final Date birthDate;
     //다른 필드와 메서드, 생성자는 생략
     /**
      * 베이비 붐 시대의 시작과 끝
      */

     // 누가봐도 상수!! 코드가 읽기 좋아졌다.
     private static final Date BOOM_START;
     private static final Date BOOM_END;

     static{
         Calendar gtmCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));

         gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
         BOOM_START = gmtCal.getTime();
         gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
         BOOM_END = gmtCal.getTime();
     }

     public boolean isBabyBoomer(){
         return birthDate.compareTo(BOOM_START) >= 0 && birthDate.compareTo(BOOM_END) < 0;
     }
 }
 ```

 이 경우 isBabyBoomer() 가 호출 되지 않았을 때 쓸데 없이 초기화 된 것. lazy initialization(초기화 지연)을 활용하여 Method가 처음 호출되었을 때 초기화 할 수 있지만(Rule.71)
 구현이 복잡해지고 성능개선도 어려움(Rule.55)

####4. 재사용 여부가 분명하지 않은 경우 (재사용하지만 변경되어도 상관 없는 경우..?)
- View 라고 부르는 Adapter는 후면객체가 관리하는 것 이외의 정보는 따로 저장하지 않으므로 여러 객체를 만들 필요가 없다.
- Map의 keySet 은 항상 같은 Set 객체를 반환 한다. 후면객체인 Map 객체가 동일하기 때문에 반환된 객체 하나가 변경되면 다른 객체들도 변한다. 그래도 상관없다. 왜냐면 keySet을 어차피 같은 객체를 관리 하는 거니까!

####5. autoboxing을 통해 자바의 기본 자료형과 그 객체 표현형을 섞어 사용할 수 있는데 이를 잘못 사용하면 성능의 큰 영향을 주게 된다.
```JAVA
public static void main(String[] args){
    Long sum = 0L; // 동작은 하지만 Long 객체가 계속 생성된다. (이 친구도 String 처럼 변경되는 것이 아니라 계속 생기나?)
    for(long i = 0; i< Integer.MAX_VALUE; i++){
        sum += i;
    }
    System.out.println(sum);
}}
```
 따라서 객체 표현형 대신 기본 자료형을 사용하고, 생각지도 못한 자동 객체화가 발생하지 않도록 유의하라.

####5. 객체 생성비용이 극단적으로 크지 않다면 직접 관리하는 객체 pool은 사용하지 않는 것이 좋다.
- 어이없네 이제 와서 객체 생성 비용이 데이터베이스 연결 모듈쯤은 되야한다네? (Rule.39)
- 코드가 어지러워지고, 메모리 요구량이 증가하며 성능도 떨어짐
