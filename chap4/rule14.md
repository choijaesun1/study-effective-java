## Rule 14. public 클래스 안에는 public 필드를 두지 말고 접근자 메서드를 사용하라.

#### A Trash!

```JAVA
// 이런 저급한 클래스는 절대 누구에게 보여주지 말 것
class Point {
  public double x;
  public double y;
}
```
이런 클래스는 Data를 직접 조작할 수 있어서 캡슐화의 이점을 누릴 수 없다.(Rule.13)

_private 필드로 바꾸고 public Getter로 바꿔주어야 마땅하다_

_변경 가능하다면 setter도 제공!_

```JAVA
class Point {
  private double x;
  private double y;

  public Point(double x, double y) {
    this.x = x;
    this.y = y;
  }

  public double getX() { return x; }
  public double getY() { return y; }

  public void setX(double x) { this.x = x; }
  public void setY(double y) { this.y = y; }
}
```

하지만 package-private / private 중첩 클래스의 멤버필드는 경우에 따라서 public으로 선언하여 코드 가독성을 높이기도 한다.
