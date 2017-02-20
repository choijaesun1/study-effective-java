## Rule 15. 변경 가능성을 최소화 하라

변경 불가능 클래스 = 객체를 수정할 수 없는 클래스. ex) String, BigInteger, BigDecimal...

### 변경 불가능한 객체를 만드는 규칙

##### 1. 객체 상태를 변경하는 메서드(Setter 등) 을 제공하지 않는다.

##### 2. 계승할 수 없도록 한다. : final 선언

##### 3. 모든 필드를 final 로 선언한다.

##### 4. 모든 필드를 private으로 선언한다.

##### 5. 변경 가능 컴포넌트에 대한 독점적 접근권을 보장한다.

- 클래스에 포함된 변경 가능 객체에 대한 참조를 클라이언트가 획득할 수 없어야 함.
- 생성자, 접근자(getter?), readObject 메서드 안에서는 방어적 복사본을 만들어야함 (Rule.39)

```JAVA
public final class Complex{
    private final double re;
    private final double im;

    public Complex(double re, double im){
        this.re = re;
        this.im = im;
    }

    // 대응되는 수정자(setter)가 없는 접근자들(getter)
    public double realPart() { return re; }
    public double imaginaryPart() { return im; }

    public Complex add(Complex c){
        return new Complex(re + c.re, im + c.im);
    }

    public Complex subtract(Complex c){
        return new Complex(re - c.re, im - c.im);
    }

    ...
}
```

사칙 연산은 각각 this 객체를 변경하는 대신 새로운 Complex 객체를 만들어 반환. => 함수형 접근법

- 함수형 접근법 : 피 연산자를 변경하는 대신, 연산을 적용한 결과를 새롭게 만들어 반환
- cf) 절차적, 명령형 접근법 : 피연산자에 일정한 절차를 적용하여 그 상태를 바꿈.

### 변경 불가능한 객체 장점

#### 1. 단순하다

 - 생성될 때 부여된 한 가지 상태만 작는다.

#### 2. Thread-Safe

 - 동기화도 필요없고(변경되는 값이 없으니) 자유롭게 공유
   - 자유롭게 공유하므로 방어적 복사본을 만들 필요도 없고 clone 메서드나 복사 생성자 또한 만들 필요도 없다.
 - 구현 방법
   - public static final
   - 자주 사용하는 객체를 Cache 하여 이미 존재하는 객체가 거듭 생성 되는 것을 방지 (Rule.1)

#### 3. 그 내부도 공유할 수 있다.

 - ex) BigInteger
   - 부호 : int, 값 : int []
   - negate method = 부호만 바꿔서 새로운 BigInteger 반환. But 배열은 공유하므로 복사되지 않음

#### 4. 다른 객체의 구성 요소로도 훌륭하다.

  - map의 key나 집합의 원소로 활용하기 좋다.
    - 변경 되지 않으므로 맵, 집합의 불변식이 깨질 걱정을 하지 않아도 됨.

### 변경 불가능한 객체 단점

#### 1. 값 마다 별도의 객체를 만들어야 한다.

따라서 객체 생성 비용이 높을 수 있다. 큰 객체라면 더더욱!

_이를 해결하는 방법_
 - 다단계 연산 가운데 자주 요구 되는 것을 기본 연산으로 제공 => But 클라이언트가 이 연산을 어떻게 쓸지 정확하게 예상 가능할 때만!
 - 변경가능한 public 동료 클래스를 제공하자 (ex> StringBuilder) - 단! 성능의 개선이 확실할 때

### 변경 불가능한 객체 구현 하는 또다른 방법

#### 1. class를 final로 선언하는 대신, 모든 생성자를 private or package-private로 선언하고 public 정적팩터리 제공

```JAVA
//생성자 대신 정적 팩토리 메서드를 제공하는 변경 불가능 클래스
public class Complex{
    private final double re;
    private final double im;

    //private 생성자
    private Complex(double re, double im){
        this.re = re;
        this.im = im;
    }

    //정적 팩토리 메서드
    public static Complex valueOf(double re, double im){
        return new Complex(re, im);
    }

    ...//나머지 동일
}
```

여러 개의 package-private 구현 클래스를 활용 할 수 있게 되므로 유연성도 가장 좋다.(뭔말??)

위의 코드는 패키지 외부에서 보면 변경 불가능한 클래스와 마찬가지!

### 변경 불가능한 객체로 만들지 않으면?

 진짜 이 객체인지 하위 객체인지를 확인해야 하고 하위 객체라면, 해당 객체가 변경 가능한 객체라는 가정하에 방어적 복사를 해야한다.(Rule.39)

```JAVA
public static BigInteger safeInstance(BigInteger val) {
  if(val.getClass() != BigInteger.class) {
    return new BigInteger(val.toByteArray());
  }
  return val;
}
```

근데 꼭 막 다 final 로 만들어야하는건 아님! 유연하게 비 final 필드를 만들수도 있음.

ex) PhoneNumber의 hashCode 메서드 : 처음으로 호출 되는 순간 계산한 해시코드를 캐싱.(재계산 비용을 줄인다.)

=> 계산 계속해도 같은 값 반환 될꺼! 

=>왜냐면 객체가 불변하기 때문!

### Serialize 주의(Rule.76) -?

변경 불가능 클래스가 Serializable 인터페이스를 구현하도록 했고, 해당 클래스에 변경 가능 객체를 참조하는 필드가 있다면,

 readObject나 readResolve 메서드를 제공하거나 아니면 ObjectOutputStream.writeUnshared나 ObjectInputStream.readInshared 메서드를 반드시 사용해야 한다.

 안그러면 공격자가 변경 불가능 클래스로 부터 변경 가능 객체를 만들 수 있게 된다.

### 요약

1. 변경 가능한 클래스로 만들 타당한 이유가 없다면, 반드시 변경 불가능 클래스로 만들어야 한다.
2. 변경 불가능한 클래스로 만들 수 없다면 변경 가능성을 최대한 제한하라.
3. 가능한 모든 필드는 final로 선언하라.
4. 객체 상태를 변화시키는 메스드를 제공하지 마라 (public 초기화 메서드, 정적팩터리 메서드, 재초기화 메서드)
