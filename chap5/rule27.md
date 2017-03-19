## Rule 27. 가능하면 제네릭 메서드로 만들 것

- static 유틸리티 메서드는 특히 제네릭화 하기 좋은 후보 ex> Collections의 binarySearch나 sort같은

- ex) 두집합의 union을 반환하는 메서드

```JAVA
//무인자 자료형 사용 - 권할 수 없는 방법(Rule.23) - 무인자 자료형은 사용하지 마라
public static Set union(Set s1, Set s2){
  Set result = new HashSet(s1);
  result.addAll(s2);
  return result;
}
```
**형인자를 선언하는 형인자 목록은 메서드의 수정자와 반환값 자료형 사이에 둔다.**

```JAVA
// 이 예제에서 형인자 목록은 <E>이고 반환값 자료형은 Set<E>이다.
// 제네릭 메서드
public static <E> Set<E> union(Set<E> s1, Set<E> s2) {
  Set<E> result = new HashSet<E>(s1);
  result.addAll(s2);
  return result;
}
// 경고없이 잘 컴파일 된다. 다만 입력과 반환값이 전부 같다는 것!. 한정적 와일드카드 자료형을 쓰면 좀 더 유연해질 수 있다.(Rule.28)
```

#### 제네릭 싱글턴 패턴
- 변경이 불가능 하지만 많은 자료형에 적용 가능한 객체를 만들어야 할때. ex> Collections.emptySet
- 예졔)

```JAVA
public interface UnaryFunction<T> {
  T apply (T arg);
}
```
항등함수는 무상태 함수이므로 필요할 때마다 새함수를 만드는 것은 낭비! **자료형 정보는 컴파일이 끝나면 삭제 되기 때문에 제네릭 싱글턴 하나만 있으면된다.**

```JAVA
//제네릭 싱글턴 팩터리 패턴
private static UnaryFunction<Object> IDENTITY_FUNCTION =
  new UnaryFunction<Object>() {
    public Object apply(Object art) {return arg;}
  };

// IDENTITY_FUNCTION 은 무상태 객체고 형인자는 비한정 인자이므로 모든 자료형이 같은 객체를 공유해도 안전하다
@SupperessWarnings("unchecked")
public static <T> UnaryFunction<T> identityFunction() {
  return (UnaryFunction<T>) IDENTITY_FUNCTION;
}

// 제네릭 싱글턴 사용 예제
public static void main(String [] args) {
  String [] strings = {"A", "B", "C"};
  UnaryFunction<String> sameString = identityFunction();

  for(String s : strings) {
    System.out.println(sameString.apply(s));
  }

  Number [] numbers = {1,2,3};
  UnaryFunction<Number> sameNumber = identityFunction();

  for(Number n : numbers) {
    System.out.println(sameNumber.apply(n));
  }
}
```

#### 재귀적 자료형 한정

- 흔히 Comparable인터페이스와 함께 흔히 쓰인다.

```JAVA
public interface Comparable<T> {
  int compareTo(T o);
}

// 자료형이 비교가능해야하는데 이러한 조건을 다음과 같이 표현
// 재귀적 자료형 한정을 통해 상호 비교 가능성 표현
public static <T extends Comparable<T>> T max (List<T> list) {
  Iterator<T> i = list.iterator();
  T result = i.next();
  while(i.hasNext()) {
    T t = i.next();
    if(t.compareTo(result) > 0) {
      result = t;
    }
  }
  return t;
}
// 자료형 한정 자기 자신과 비교가능한 모든 자료형 T
```
