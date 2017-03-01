## Rule 18. 추상 클래스 대신 인터페이스를 사용하라

#### 추상 클래스와 인터페이스의 차이점

##### 1. 추상클래스는 구현된 메서드를 포함할 수 있지만 인터페이스는 아니다.

###### 8 부터는 default 메서드를 통해 인터페이스에도 구현을 일부 포함시킬 수 있다.

```JAVA
public interface DoIt {
   void doSomething(int i, double x);
   int doSomethingElse(String s);
   default boolean didItWork(int i, double x, String s) {
       // Method body
   }  
}
```
##### 2. 추상클래스의 구현을 위해서는 계승을 해야 한다.

 - 다중 상속이 허용되지 않으므로 제약이있다.

#### 인터페이스를 써야하는 이유

##### 1. 이미 있는 클래스를 개조해서 새로운 인터페이스를 구현하는 것은 간단하다.

 - 그러나 새로 도입된 추상 클래스를 확장하도록 기존 클래스를 개조할 수 없다.
    - 여기 밑에 있는 설명 이해 안감;;

##### 2. 인터페이스는 mixin 을 정의하는데 이상적이다.

  - mixin : 클래스가 주 자료형 이외에 추가호 구현할 수 있는 자료형으로 어떤 선택적 기능을 제공한다는 사실을 선언하기 위해 쓰인다.
  - ex) Comparable interface
  - 만약 내가 만든 클래스가 Comparable을 구현하면 내클래스 기능 + Comparable 의 기능도 제공한다 는 뜻..?

##### 3. 인터페이스는 비 계층적인 자료형 프레임워크를 만들 수 있도록 한다.

 - ex) 작곡가이면서 가수인 경우
 ```JAVA
 public interface Singer {
   AudioClip sing(Song s);
 }
 public interface Songwriter {
   Song compose(boolean hit);
 }

 public interface SingerSongwriter extends Singer, Songwriter {
   AudoiClip strum();
   void actSensitive();
 }
 ```
 - 이러한 경우를 계층으로 표현하려고 하면 계층 구조가 복잡함.
 - 필요한 속성이 n개가 있다면 조합의 가지수는 2^n 이 되고 이런 문제를 조합 증폭이라 한다.

##### 4. 인터페이스를 사용하면 포장 클래스 숙어를 통해 안전하면서도 강력한 기능 개선이 가능하다.

 - 추상클래스를 사용해 자료형을 정의하면 계승 이외에 수단을 이용할 수 없다.
 - (Rule.16이랑 같은 말인듯)

##### 5. 추상 골격 구현 클래스를 중요 인터페이스마다 두면 인터페이스의 장점과 추상클래스의 장점을 결합할 수 있다.

 - 인터페이스로는 자료형을 정의하고, 구현은 골격 구현 클래스가
 - 관습적으로 골격 구현 클래스의 이름은 Abstract _Interface_ 라 한다.
 - ex)
 ```JAVA
 // int 배열을 Integer 객체의 리스트 처럼 볼 수 있도록 하는 어댑터 패턴 적용사례
 static List<Integer> intArrayAsList(final int[] a){
   if(a==null) throw new NullPointerException();
   return new AbstractList<Integer>(){
      public Integber get(int i){
        return a[i];
      }
      @Override
      public Integer set(int i, Integer val){
        int oldVal = a[i];
        a[i] = val;
        return oldVal;
      }
      public int size(){
         return a.length;
      }
   };
}
// 객체 자동 변환때문에 성능은 좋지 않다.
 ```
  - 이 코드에서 반환되는 값은 외부에서 접근이 불가능한 익명의 클래스라는 점에 주의 (Rule.22)
  - 골격 구현 클래스가 있다면 해당 클래스를 사용해 인터페이스를 구현하는 것이 가장 분명한 프로그래밍 방법   
  - 골격 구현 클래스를 상속하도록 기존 클래스를 변경할 수 없다면, 인터페이스를 직접 구현해도 된다.
    - 이러한 경우에는 골격 구현 클래스를 계승하는 private inner class를 정의하고 인터페이스 메서드에 대한 호출은 해당 중첩 클래스 객체로 전달 (모의 가상 상속)
  - __골격 구현 클래스 만드는 법__
    - 기본 메서드는 추상 메서드로 선언
    - 나머지 메서드는 구현
  - 간단 구현 이라는 것도 있음 (추상클래스 아님)

#### 추상 클래스가 인터페이스보다 나은 점

##### 인터페이스 보다 추상클래스가 발전시키기 쉽다.
 - 다음 릴리스에 언제든 새로운 메서드 추가 가능
 - 인터페이스를 구현하려면 모든 메서드를 해야하니까 기존 코드 에러나겠지..
 - 그러므로 인터페이스 설계에 신중해야함.
