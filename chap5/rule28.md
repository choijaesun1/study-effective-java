## Rule 28. 한정적 와일드 카드를 써서 API유연성을 높여라

- 불변 자료형 : List<Type1>, List<Type2>는 어떤 상-하위 관계 성립 안됨.
- 때로는 불변 자료형보다 높은 유연성이 필요할 때가 있다.
- Ex) Stack

  - pushAll :

  ```JAVA
  public class Stack<E> {
    public Stack();
    public void push(E e);
    public E pop();
    public boolean isEmpty();
  }

  // 와일드 카드 자료형을 사용하지 않는 pushAll 메서드 - 문제가 있다. : Iterable 이 가리키는 원소의 자료형과 스택 원소의 자료형이 일치해야함!
  public void pushAll(Iterable<E> src) {
    for(E e : src) {
      push(e);
    }
  }
  //오류가 없을 것 같지만 형인자 자료형은 불변이기 때문에 오류 발생!
  Stack<Number> numberStack = new Stack<Number>();
  Iterable<Integer> integers = ...;
  numberStack.pushAll(integers);
  ```

  ```JAVA
  // 와일드 카드 자료형을 사용해서 Stack 의 E 의 하위 자료형이 들어갈꺼라고 명시하자
  public void pushAll(Iterable<? extends E> src) {
    for(E e : src) {
      push(e);
    }
  }
  ```

  - popAll

  ```JAVA
  // 와일드카드 자료형 없이 구현한 popAll 메서드 - 문제가 있다.
  public void popAll(Collection<E> dst) {
    while(!isEmpty) {
      dst.add(pop());
    }
  }

  // 하위 자료형 관계가 성립하지 않으므로 오류!
  Stack<Number> numberStack = new Stack<Number>();
  Collection<Object> objects  = ...;
  numberStack.popAll(objects);
  ```

  ```JAVA
  // E의 상위 자료형이라고 명시
  // E의 소비자 구실을 하는 인자에 대한 와일드카드 자료형
  public void popAll(Collection<? super E> dst) {
    while(!isEmpty()) {
      dst.add(pop());
    }
  }
  ```

  - **유연성을 최대화 하려면, 객체 생산자나 소비자 구실을 하는 메서드 인자의 자료형은 와일드카드 자료형으로 하라**
  - _Produce - Extends, Consumer - Super_ : T가 생산자 (<? extends T>), T가 소비자 (<? super T>)

#### 이 규칙을 함수에 적용해보자!

```JAVA
// rule.25
static <E> E reduce(List<E> list, Function<E> f,E initVal)

//이 메서드의 경우 list 인자를 E 생산자로만 사용하므로 ? extends E 로 선언
static <E> E reduce (List<? extends E> list, Function<E> f, E initVal)
```

```JAVA
// rule.27
public static <E> Set<E> union(Set<E> s1, Set<E> s2)

// s1, s2 모두 생산자
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)

// 될 것 같지만 안되는 코드. 컴파일러가 자료형을 정확하게 유추하지 못하는 경우!
Set<Integer> integers = ...;
Set<Double> doubles = ...;
Set<Number> numbers = union(integers, doubles);

Set<Number> numbers = Union.<Number>union(integers, doubles);// 이렇게 하면 된다는데 뭔지 모르겠다 ~.~
```

**반환 값에는 와일드 카드 자료형을 쓰면 안된다! 클래스 사용자가 와일드카드 자료형에 대해 고민하게 하면 안된다**

```JAVA
// rule.27
public static <T extends Comparable<T>> T max (List<T> list)

// list는 생산자, Comparable 은 T 객체를 소비
public static <T extends Comparable<? super T>> T max (List<? extends T> list)

// 근데 이걸 기존 방식대로 구현하면 Iterator에서 에러남. (iterator 메서드가 Iterator<T>를 반환하지 않는다고 함!)
// 이렇게 고치면 됨!
public static <T extends Comparable<? super T>> T max(List<? extends T> list){
  Iterator<? extends T> list.iterator();
  ...
}
```
**Comparable, Comparator 은 항상 <? super T>를 사용해야 한다.**

#### 형인자와 와일드 카드 사이의 이원성(duality)

- ex) swap

```JAVA
// swap 메서드를 선언하는 두가지 방법
public static <E> void swap(List<E> list, int i, int j)
public static void swap(List<?> list, int i , int j)
```

**원칙은, 형인자가 메서드 선언에 단 한군데 나타난다면 해당 인자를 와일드 카드로 바꾸라**

```JAVA
// 두번째 방법으로 했을 때의 문제 : List<?>에는 null 이 외의 어떤 값도 넣을 수 없다.
public static void swap(List<?> list, int i, int j) {
  list.set(i, list.set(j, list.get(i)));// 리스트에서 꺼냈는데 리스트로 넣을 수 없는 문제 발생.
}
```

```JAVA
public static void swap(List<?> list, int i, int j) {
  swapHelper(list, i, j);
}

//와일드카드 자료형을 포착하기 위한 private helper 메서드 : 클라이언트는 모른다 ㅎㅎ 와일드 카드를 쓴줄 알겠지 캬캬
private static <E> void swapHelper(List<E> list, int i, int j) {
  list.set(i, list.set(j, list.get(i)));
}
```

### 요약
- API 에는 와일드카드 자료형을 사용하라. 유연해진다!
- 생산자, 소비자의 기본규칙을 암기하라
