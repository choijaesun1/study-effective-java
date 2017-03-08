## Rule 23. 새 코드에는 무인자 제네릭 자료형을 사용하지 마라

### 제네릭 자료형

- 제네릭 클래스 or 인터페이스 = 제네릭 자료형
- 제네릭 자료형은 형인자 자료형의 집함을 정의 ex) List<String>
- 제네릭 자료형은 새로운 무인자 자료형을 정의
  - 무인자 자료형 : 실 형인자 없이 사용되는 제네릭 자료형 ex) List<E>에서 무인자 자료형은 List
    - ex)

    ```JAVA
    //무인자 컬렉션 자료형, 앞으로는 이렇게 하면 안 된다.
    /**
     * 내 우표 컬렉션. Stamp 객체만 보관한다.
     */
     private final Collection stamps = ...;
     // 이 컬렉션에 엉뚱한 자료형의 객체를 넣어도 컴파일 시에는 문제가 없다.
    ```
    이럴 때, 다른 객체를 넣으면 Exception 발생할 수 있다. **오류는 컴파일 시 찾을 수 있으면 제일 좋다.**

- 제네릭을 쓰면, 컴파일러에게 컬렉션에 담길 객체의 자료형이 무엇인지 선언 가능

```JAVA
// 형인자 컬렉션 자료형 - 형 안전성 확보
private final Collection<Stamp> stamps = ...;
// java 1.5 이상
```
- 무인자 자료형을 쓰면 형 안전성이 사라지고, 제네릭의 장점 중 하나인 표현력 측면에서 손해를 보게 된다.
- 새로만드는 코드에서는 List와 같은 무인자 자료형을 쓰면 안 되지만, 아무객체나 넣을 수 있는 List<Object> 는 넣어도 좋다.
  - 차이 : List 와 같은 무인자 자료형은 사용하면 형 안전성을 잃게 되지만, List<Object> 와 같은 형인자 자료형을 쓰면 그렇지 않다.

  ```JAVA
  // 실행 도중에 오류를 일으키는 무인자 자료형(List)의 예
  public static void main(String[] args) {
    List<String> strings = new ArrayList<String>();
    unsafeAdd(strings, new Integer(42));

    // 형변환 에러가 날꺼야!
    String s = strings.get(0); // 컴파일러가 자동으로 형변환
  }

  // List<Object> 로 하면 컴파일 에러!
  private static void unsafeAdd(List list, Object o){
    list.add(o);
  }
  ```

- 비한정적 와일드 카드 자료형
 - 제네릭 자료형을 쓰고 싶으나 실제형인자가 무엇인지는 모르거나 신경쓰고 싶지 않을때는 형인자로 ? 를 쓰면된다.

 ```JAVA
 // 비한정적 와일드 카드 자료형 - 형 안전성과 유연성 만족
 static int numElementsInCommon(Set<?> s1, Set<?> s2) {
      int result = 0;
      for(Object o1 : s1){
          if(s2.contains(01)){
              result++;
          }
      }
     return resultt;
}
 ```

- 와일드 카드 자료형은 안전, 무인자자료형은 안전하지 않음.
- Collection<?> 에는 null 이외에 어떤원소도 넣을수가 없다.
- 이게 싫으면 제네릭 메서드(Rule27) 이나 한정적 와일드카드 자료형(Rule 28)을 쓰면된다.


### 무인자 자료형을 써도 되는 경우

#### 1. 클래스 리터럴에는 반드시 무인자 자료형을 사용해야한다.
 - List.class 는 되지만 List<String>.class 는 안된다.

#### 2. instanceof 연산자 사용 시

```JAVA
if( o instanceof Set) {
  Set<?> m = (Set<?>) 0;// 와일드 카드 자료형
}
```

### 요약

- 무인자 자료형을 쓰면 프로즈램 실행 도중에 예외가 발생할 수 있으므로 자제하자..!
- Set은 무인자 자료형으로 제네릭 자료형 시스템의 일부가 아니다. Set<?>, Set<Object>는 안전하지만 Set은 안전하지 못하다.
