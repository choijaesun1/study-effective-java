## int 상수 대신 enum을 사용하라

- enum이 나오기 전 - 지극히 불만족!

```JAVA
// int 를 사용한 enum 패턴
public static final int APPLE_FUJI = 0; ...
```
  - 형 안전성 면에서도 구리고 편의상도 좋지 않다.

  ```JAVA
  public static final int APPLE_FUJI = 0; ...
  public static final int ORANGE_FUJI = 0; ...
  // 컴파일러는 Apple 이 필요한 곳에 Orange를 넣어도 에러 안뱉는다.
  ```
  - int enum 상수를 순차적으로 이용하거나 그룹의 크기를 알아낼 좋은 방법도 없다.
  - String enum 패턴도 있는데 이게 더 나쁨!
    - equals 비교해야하니까 성능도 떨어지고 문자열 상수를 하드코딩 하다가 실수가 있을 수도 있다.

### Enum (after java 1.5)

- 다른 언어들과 달리 그냥 int 값이 아니다!
- 열거 상수별로 하나의 객체를 public static final 필드 형태로 제공하는 것!
- 객체를 생성하거나 계승을 통해 확장할 수 없으므로 자료형의 개체 수는 엄격히 통제된다.

#### Enum의 장점

##### 1. 형 안정성을 제공한다.

##### 2. 같은 이름의 상수를 namespace 를 통해 분리하여 공존할 수 있도록 한다.

##### 3. toString 메서드를 호출하면 인쇄 가능 문자로 변환할 수 있다.

##### 4. 임의의 메서드나 필드도 추가할 수 있고 인터페이스를 구현할 수도 있다.
- Object에 정의된 모든 고품질 메서드들이 포함되어있고(3장) Comparable interface와 Serializable interface가 구현되어있다.

**근데 왜 이렇게 할 수 있도록 설계 했을까?** - 상수에 데이터를 연계시키면 좋겠다! ex) ```APPLE("red") ```

#### 예제

```JAVA
//데이터와 연산을 구비한 enum 자료형
public enum Planet {
  MERCURY(3.302e+23, 2.439e6),
  VENUS,
  EARTH,
  MARS,
  JUPITER,
  SATURN,
  URANUS,
  NEPTUNE;

  private final double mass;
  private final double radius;
  private final double surfaceGravity;

  // 중력 상수
  private static final double G = 6.67399E-11;

  // constructor
  Planet (double mass, double radius) {
    this.mass = mass;
    this.radius = radius;
    surfaceGravity = G * mass / (radius * radius);
  }

  public double mass() {return mass;}
  public double radius() {return radius;}
  public double surfaceGravity() {return surfaceGravity;}

  public double surfaceWeight(double mass) {
    return mass * surfaceGravity; // F=ma
  }
}
```

### Enum 활용

**enum상수에 데이터를 넣으려면 객체 필드를 선언하고 생성자를 통해 받은 데이터를 그 필드에 저장하면 된다.**

1. enum 은 원래 변경 불가능 하므로 final 로 선언해야한다. (사알 다음부터 이래 해야지!)

2. 상수기 선언된 순서대로 반환해주는 **values()** 와 **toString()** 이 있고 toString이 맘에 안들면 재정의 하면된다.

3. enum에 정의한 메서드를 클라이언트에게까지 공개할 특별한 이유가 없다면 private이나 package-private로 선언하라(Rule.13)

4. 일반적으로 쓰일 Enum : 최상위 public, 특정 클래스에서만 쓰일 것이라면 멤버클래스로 선언(Rule.22)

#### 상수들이 제각기 다른 방식으로 동작 하도록 해야할 때.

- ex) 사칙연산이 자기가 표현하는 연산을 실제로 실행해야한다고 가정

```JAVA
// 별로 ~.~
//자기 값에 따라 분기하는 Enum 자료형
public enum Operation {
  PLUS, MINUS, TIMES, DIVIDE;

  //this 상수가 나타내는 산술 연산 실행
  double apply(double x, double y) {
    switch(this) {
      case PLUS : return x+y;
      case MINUS : return x-y;
      case TIMES : return x*y;
      case DIVIDE : return x/y;
    }
    throw new AssertError("Unknown op : " +this);
  }
}
```
**상수별 몸체 클래스를 활용하자!**

```JAVA
public enum Operation {
  PLUS (double apply(double x, double y) {return x+y;}),
}

//상수별 클래스 몸체와 별도로 데이터를 갖는 enum 자료형
public enum Operation {
  PLUS("+") {
    double apply(double x, double y) {return x+y;}
  }...;

  private final String symbol;

  Operation(String symbol) {this.symbol = symbol;}

  @Override
  public String toString() {return symbol;}

  abstract double apply(double x, double y);
}

// 이거 디기 많이 쓰임
```

#### Enum 에서 쓰면 좋은 메서드

1. valueOf(String) : 상수 이름을 상수 그자체로 변환하는 역할

2. toString() : 얘를 재정의 하는 경우에는 **fromString** 메서드를 작성해서 toString이 뱉어내는 문자열을 다시 enum 상수로 변환할 수단을 제공해야할 지 생각해 보라.

```JAVA
//enum 자료형에 대한 fromString메서드 구현
private static final Map<String, Operation> stringToEnum = new HashMap<>();

// 얘는 상수가 만들어지고 난 다음에 실행된다. enum 생성자 안에서는 static field에 접근할 수 없다.(컴파일 시점에 상수인 static 필드 제외.)
// 그래서 상수 생성자에서 자기 스스로를 맵 안에다 넣게 하는건 안된다.
static {
  for(Operation op : values()){
    stringToEnum.put(op.toString(), op);
  }
}

// 문자열이 주어지면 그에 대한 Operation 상수 반환. 잘못된 문자열이면 null 반환
public static Operation fromString(String symbol) {
  return stringToEnum.get(symbol);
}

```

#### 상수별 메서드의 단점

1. enum끼리 공유하는 코드를 만들기가 어렵다.

```JAVA
//enum 상수에따라 분기하는 switch문을 이용해서 코드 공유 - 좋은 방법인가?
enum PayrollDay {
  MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
  private static final int HOURS_PER_SHIFT = 0;

  double pay(double hoursWorked, double payRate) {
    double basePay = hoursWorked * payRate;

    double overtimePay; //초과 근무 수당

    switch(this) {
      case SATURDAY : case SUNDAY :
        overtimePay = hoursWorked * payRate / 2;
        break;
        default:
          overtimePay = hoursWorked <= HOURS_PER_SHIFT ? 0 : (hoursWorked - HOURS_PER_SHIFT) * payRate / 2;
    }

    return basePay + overtimePay;
  }
  // enum 이 추가 되었는데 까먹고 case 처리를 안해줬다면?
}
```

**개선 코드 - 전략 enum을 인자로 받도록**

```JAVA
enum PayrollDay {
  MONDAY(PayType.WEEKDAY), TUESDAY(PayType.WEEKDAY), WEDNESDAY(PayType.WEEKDAY),
  THURSDAY(PayType.WEEKDAY), FRIDAY(PayType.WEEKDAY),
  SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND);

  private final PayType payType;
  PayrollDay (PayType PayType) {this.payType = payType;}

  double pay(double hoursWorked, double payRate) {
    return payType.pay(hoursWorked, payRate);
  }

  private enum PayType {
    WEEKDAY {
      double overtimePay(double hours, double payRate) {
        return hours <= HOURS_PER_SHIFT ? 0 : (hours - HOURS_PER_SHIFT) * payRate / 2;
      }
    },
    WEEKEND {
      double overtimePay(double hours, double payRate) {
        return hours * payRate / 2;
      }
    };

    private static final int HOURS_PER_SHIFT = 8;

    abstract double overtimePay(double hrs, double payRate);

    double pay(double hoursWorked, double payRate) {
      double basePay = hoursWorked * payRate;
      return basePay + overtimePay(hoursWorked, payRate);
    }
  }
}
```

##### 그럼 switch 문은 언제 씀? - 외부 enum 자료형 상수별로 달리 동작하는 코드를 만들어야할 때

```JAVA
// 기존 enum자료형에 없는 메서드를 switch 문을 사용해 구현한 사례
public static Operation inverse(Operation op) {
  switch(op) {
    case PLUS : return Operation.MINUS;
    ...
  }
}
```

결국 요약이 다!!
