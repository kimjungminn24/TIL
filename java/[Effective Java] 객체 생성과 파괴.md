# 아이템 1. 생성자 대신 정적 팩터리 메서드를 고려하라

- `static`으로 선언된 메서드
- 객체 생성 로직을 숨기고, 재사용하거나 조건에 따라 반환 객체를 달리할 수 있음

```java
public class Point {
    private final int x;
    private final int y;

    private Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    //정적 팩토리 메서드
    public static Point of(int x, int y) {
        return new Point(x, y);
    }
}
```

### 장점

- 의미 있는 이름을 가질 수 있음 : 생성자와 달리 메서드 이름을 통해 반환 객체의 성격을 명확하게 표현할 수 있다.
- 객체를 매번 새로 생성 X : 동일한 인스턴스를 재사용하거나 캐싱이 가능해, 불필요한 객체 생성을 줄이고 성능을 높일 수 있다.
- 반환 타입의 하위 타입 반환 : 인터페이스를 반환하고 실제 구현은 감출 수 있어, 유연하고 깔끔한 API 설계가 가능하다.

```java
List<String> list = Collections.unmodifiableList(originalList);
// 반환 타입은 List 인터페이스지만 내부적으로는 UnmodifiableRandomAccessList 같은 구현체가 반환된다.
// 클라이언트는 구현체를 몰라도되며, 인터페이스만으로 충분히 사용할 수 있다.
// API 변경 없이 내부 구현을 자유롭게 바꿀 수 있다.
```

- 입력값에 따라 다양한 객체를 반환 : 조건에 따라 서로 다른 구현체를 반환할 수 있어, 확장성과 유연성이 높다.

반환할 클래스가 나중에 정의돼도 된다
서비스 제공자 프레임워크(SPF)와 같이 모듈 간 결합도를 낮춘 구조에 적합하다.

### 단점

- 상속 불가: 보통 생성자를 private으로 막아두기 때문에 하위 클래스를 만들 수 없다
- 문서화 어려움: 일반 메서드처럼 보이므로 API 문서에서 찾기 어려울 수 있다

### 명명 규칙 요약

| 메서드 이름                 | 의미                 | 예시                        |
| --------------------------- | -------------------- | --------------------------- |
| `from`                      | 형 변환              | `Date.from(instant)`        |
| `of`                        | 여러 매개변수 → 객체 | `EnumSet.of(...)`           |
| `valueOf`                   | 기본 타입 → 객체     | `Boolean.valueOf(true)`     |
| `instance 또는 getInstance` | 인스턴스 반환        | `StackWalker.getInstance()` |
| `create 또는 newInstance`   | 매번 새 객체 반환    | `Array.newInstance(...)`    |
