## Rule 16. 계승하는 대신 구성하라

#### 계승은 코드 재사용을 돕는 강력한 도구지만 통제할 수 있는 범위 안에서 사용해야한다.
 - 상위 클래스가 속한 패키지 밖에서 계승을 시도하는 것은 위험
 - 이번 절에서는 implements는 해당되지 않는다.

### 계승은 왜 위험한가

##### 1. 메서드 호출과 달리, 계승은 캡슐화 원칙을 위반한다.

 - 상위 클래스의 구현은 Release가 거듭되면서 바뀔 수 있는데 이때 하위 클래스 코드가 깨질 수 있다
 - 따라서 상위 클래스를 설계할 때 계승 될 것을 고려해야하고 문서도 작성해야한다.

 - ex )

 ```JAVA
 //계승을 잘못 사용한 사례
public class InstrumentedHashSet<E> extends HashSet<E>{
    //요소를 삽입하려 한 횟수
    private int addCount = 0;

    //생성자
    public InstrumentedHashSet() {}

    public InstrumentedHashSet(int initCap, float loadFactor){
        super(initCap, loadFactor);
    }

    @Override
    public boolean add(E e){
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c){
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount(){
        return addCount;
    }
}
 ```

 이 코드에서 ```addAll(Arrays.asList("1","2","3"))``` 하면 addCount 는 3이 될 것 같지만 실제로는 **6**

 왜냐면 addAll 은 HashSet에서 add를 사용하고 있기 때문이다. (HashSet 문서에는 명시 되어있지 않음)

 addAll 메서드 재정의를 삭제하면 문제가 해결 되지만 만약 이후 HashSet의 addAll에서 add 함수 활용을 안하게 되면 InstrumentedHashSet 클래스는 깨질 수 있음.

 아니면 하위 클래스의 addAll이 재정의한 add를 호출하면 바른결과가 나올 것이지만 항상 이 방법을 쓸 수 있는 것도 아니고 효율적이지도 않다.

##### 2. 다음 릴리스에는 상위 클래스에 새로운 메서드가 추가될 수 있다.

- ex ) 만약 어떤 하위 클래스에서 상위 클래스의 특정 기능을 수행하는 모든 메서드를 재정의 해야한다고 했을 때, 상위 클래스에 해당 기능을 수행하는 메서드가 나중에 추가되게 되면 하위 클래스의 기능 수행에 문제가 생기게 된다.

##### _계승이 위험한 이유는 메서드 재정의 때문만은 아니다._

 - 하위 클래스에서 어떤 메서드를 사용하고 있는데 상위클래스에서 같은 시그니처인데 반환 자료형만 다른 경우도 깨지게 된다.

 - 이 경우는 약도 없는게 하위 클래스에서 메서드를 추가할 당시 만들어진 규약이 아니기 때문에 어떻게 사용하고 있을지 예측할수가 없다.

### 계승 대신 구성하라 (Composition)

##### 기존 클래스가 새 클래스의 일부가 되도록 한다.
 -  새로운 클래스에 포함된 각각의 메서드는 기존 클래스에 있는 메서드 가운데 필요한 것을 호출해서 그 결과를 반환 (**전달(forwarding)**)
 - 기존 클래스에 다른 메서드가 추가되더라도 새로 만든 클래스에 영향 없음

 - 구성 : 클래스 자신, 재사용이 가능한 전달 클래스(모든 전달 메서드를 포함한다.)

 - ex ) 계승 -> 구성

```JAVA
 // 계승 대신 구성을 사용하는 포장(wrapper) 클래스
 class InstrumentedSet<E> extends ForwardingSet<E>{
   // Set 객체를 감싸고 있다.

  private int addCount = 0;

  public InstrumentedSet(Set<E> s) {
    super(s);
  }

  @Override
  public boolean add(E e){
    addCount++;
    return super.add(e);
  }

  @Override
  public boolean addAll(Collection<? extends E> c){
    addCount += c.size();
    return super.addAll(c);
  }

  public int getAddCount(){
    return addCount;
  }
}

// 재사용 가능한 전달(forwarding)(위임) 클래스
class ForwardingSet<E> implements Set<E> {

  // Set 객체
  private final Set<E> s;

  public ForwardingSet(Set<E> s){
    this.s = s;
  }

  @Override
  public int size() { return s.size(); }
  @Override
  public boolean isEmpty() { return s.isEmpty();}
  @Override
  public boolean contains(Object o) { return s.contains(o);}
  @Override
  public Iterator<E> iterator() {return s.iterator();}

}
```

  - 이런 설계는 안정적이고 유연하다.
  - 계승을 이용한 접근법 : 한 클래스에만 적용 가능하고, 상위 클래스 생성자 마다 별도의 생성자를 구현해야함.
  - Wrapper Class : 어떤 Set 구현도 원하는 대로 수정할 수 있고, 이미 있는 생성자도 쓸 수 있다. (어떤 Set이던 Setting)

  ```JAVA
  Set<Date> s =new InstrumentedSet<Date>(new TreeSet<Date>(cmp));
  Set<E> s2 = new InstrumentedSet<E>(new HashSet<E>(capacity));
  ```

 - 이런 패턴을 장식자 패턴이라고도 하고 구성과 전달 기법을 아울러 위임이라고 부르기도한다.

### 굳이 찾은 단점

 - 역호출 프레임워크와 함께 사용하기에는 적합하지 않다.
   - 포장된 객체는 포장 객체에 대해서는 모르기 때문에 자신에 대한 참조를 넘김(this)
   - 따라서 역 호출 과정에서 포장 객체는 제외된다.

### 요약

#### 1. 계승은 상하위 클래스간 IS-A 관계가 성립할 때만 계승.

#### 2. 그렇다 하더라도 계승은 문제를 일으킬 수 있으므로 구성과 전달 기법을 사용하라

#### 3. 포장 클래스 구현에 적당한 인터페이스가 있다면 더더욱!
