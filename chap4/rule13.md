## Rule 13. 클래스와 멤버의 접근 권한은 최소화하라.
#### 잘 설계된 모듈 = 구현 세부사항을 전부 API 뒤쪽에 감춤. = 정보 은닉, 캡슐화

### 정보 은닉이 중요한 이유

#### 모듈간 Coupling 을 낮춘다. 개별적으로 개발, 시험, 최적화, 변경...

  - 병렬적으로 개발할 수 있어서 개발 속도가 올라간다.
  - 유지 보수도 Good
  - 무조건 좋은 성능을 보장하는 것은 아니지만 효과적으로 성능 튜닝을 가능하게 한다.
  - SW 재사용성을 높인다. 의존성이 낮으니까!
  - 대규모 시스템 개발 과정의 risk 도 낮춘다.
    - 전체적으로는 성공적이지 않더라도 모듈은 건질 수 있다.

### 자바에서 정보은닉 실현하기

- 접근 제어 매커니즘 : class, interface, 멤버들의 접근 권한(**access modifier** 에 의해 결정)을 규정

### 정보 은닉의 원칙

#### 1. 각 클래스와 멤버는 가능한 접근 불가능 하도록 만들어라.

  - SW가 동장하는 범위 안에서 가장 낮은 접근 권한을 설정하라.
  - **최상위 레벨 클래스와 interface** 는 가능한 package-private으로 선언 해라!
    - 이래야 API의 일부가 아니라 내부 로직에 속하므로 다음번 릴리즈에 Client 코드를 안깨뜨릴 수 있다. (public이 되면 API의 일부가 된다.)
    - 사용자 Class 가 하나뿐이라면 private 중첩 class를 고래해 보라.
  - **Member**
    - Access modifier
      - private : 클래스 내부에서만 접근 가능.
      - package-private : 같은 패키지 내의 아무 클래스나 사용가능. (default)
      - protected : 자신이랑 하위클래스만 사용 가능. 선언된 클래스와 같은 패키지에 있는 클래스에서도 사용가능 (그래..?)
      - public : 어디서든!
    - 여튼 결국 접근 가능 권한을 최소화 해라. 예를 들어 package-private으로 멤버를 선언해야한다면 시스템 재설계를 고려하여 의존성을 끊어낼 수 있도록한다.
    - private, package-private은 API의 일부가 아니지만 Serializeable을 구현하는 클래스의 멤버라면 공개 API 속으로 새어나갈(leak) 수 있다.(Rule 74,75) - 뭔소리지 아침에 봐야지
    - protected는 자제하자. 공개적인 약속과도 같다! 이미 한번 나가면 영원히 유지 해야 한다.
  - **Method**
    - 상위 클래스 메서드를 재정의 할 때 원래 메서드의 접근 권한보다 낮은 권한을 설정할수 없다.
      - 상위 클래스를 사용할 수 있는 곳 => 하위 클래스도 사용 가능해야함.

#### 2. 객체 Field 는 절대 public으로 선언하면 안된다.

 - 비-final 필드나 변경 가능 객체에 대한 final 참조 필드를 public으로 선언하면 메서드를 통하지 않고도 필드 값을 변경할 수 있게 되므로 그 필드와 관계된 불변식을 강제할 수 없다.

 - 변경 가능 public 필드를 가진 클래스는 다중 Thread에 안전하지 않다.

 - 변경 불가능한 객제를 참조하는 final 필드여도 공개 API의 일부가 되므로 삭제/수정할 수 없게 된다.

 - static도 마찬가지 이지만 어떤 상수들이 클래스로 추상화된 결과물의 핵심적 부분을 구성한다고 하면 public static final 로 선언하여 공개할 수 있다.
   - But 이런 필드는 반드시 기본 자료형을 가지고나, 변경 불가능한 객체를 참조해야함.

#### 3. public static final 배열 필드를 두거나, 배열 필드를 반환하는 접근자를 정의하면 안된다.

 - 보안의 문제를 초래할 수 있는 코드

 ```JAVA
 // VALUES 는 변하지 않지만 VALUE 안에 데이터가 변할수가 있다.
 public static final Thing[] VALUES={..};
 ```

 - 문제를 해결할 수 있는 방법

   - 수정 불가능한 list를 반환

   ```JAVA
   private static final Thingp[ PRIVATE_VALUES = {..};
   public static final List<Thing>VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
   ```

   - 복사 반환
   ```JAVA

   public static final Thing[] values(){
      return PRIVATE_VALUES.clone();
   }
   ```
  - 방법 선택은 Client가 어떤 자료형을 반환 받기를 원하는가, 어느쪽이 더 나은 성능을 보장하는가.

### 요약

1. 접근 권한은 가능한 낮추가.
2. 최소한의 public API 를 설계하고 다른 Class, interface, member는 API에서 제외 시켜라
3. public static final 필드를 제외한 어떤 필드도 public으로 선언하지 마라.
4. public static final 필드가 참조하는 객체는 변경 불가능한 객체로 만들라.
