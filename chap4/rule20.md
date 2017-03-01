## Rule 20. 태그 달린 클래스 대신 클래스 계층을 활용하라

 - 두가지 이상의 기능을 가지고 있으며 그중 어떤 기능을 제공하는지 표시하는 태그가 달린 클래스
 ```JAVA
 class Figure {
    enum Shape {
        RECTANGLE, CIRCLE
    };

    // 어떤 모양인지 나타내는 태그 필드
    final Shape shape;

    // RECTANGLE 일때만 쓰는 필드
    double length;
    double width;

    // CIRCLE 일때만 쓰는 필드
    double radius;

    // CIRCLE 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // RECTANGLE 생성자
    Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch (shape) {
        case RECTANGLE:
            return length * width;
        case CIRCLE:
            return Math.PI * (radius * radius);
        default:
            throw new AssertionError();
        }
    }
}
 ```
 - enum 선언 태그 필드 switch 문등 상투적 코드가 반복
 - 객체를 만들때마다 필요없는 기능을 위한 필드도 함께 생성 되어 메모리 낭비
 - 무슨 기능을 제공하는 것인지 불분명
 - **태그 기반 클래스는 클래스 계층을 얼기설기 흉내 낸 것일 뿐**

#### 추상 클래스로 정의하고 계승

```JAVA
abstract class Figure {
    abstract double area();
}

class Circle extends Figure {
    final double radius;

    Circle(double radius) {
        this.radius = radius;
    }

    double area() {
        return Math.PI * (radius * radius);
    }
}

class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }

    double area() {
        return length * width;
    }
}
```

- 코드가 간단 명료, 명시적
  - 필요없는 필드 존재하지 않음
- 프로그래머의 실수로 발생하는 오류 감소
- 자료형 간의 자연스러운 계층 관계를 반영할 수 있어 유연성이 높아지고 컴파일 시 형 검사를 하기에 용이

##### 태그 필드가 있는 클래스를 만나게 된다면, 리팩터링을 통해 클래스 계층으로 변환할 방법이 없는지 고민해보라
