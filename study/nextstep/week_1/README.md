## 1주차

### 객체지향 생활 체조 원칙

객체지향 생활 체조 원칙은 소트웍스 앤솔러지 책에서 다루고 있는 내용으로 객체지향 프로그래밍을 잘 하기 위한 9가지 원칙을 제시하고 있다.

이 책에서 주장하는 9가지 원칙은 다음과 같다.

-   규칙 1: 한 메서드에 오직 한 단계의 들여쓰기(indent)만 한다.
-   규칙 2: else 예약어를 쓰지 않는다.
-   규칙 3: 모든 원시값과 문자열을 포장한다.
-   규칙 4: 한 줄에 점을 하나만 찍는다.
-   규칙 5: 줄여쓰지 않는다(축약 금지).
-   규칙 6: 모든 엔티티를 작게 유지한다.
-   규칙 7: 3개 이상의 인스턴스 변수를 가진 클래스를 쓰지 않는다.
-   규칙 8: 일급 콜렉션을 쓴다.
-   규칙 9: 게터/세터/프로퍼티를 쓰지 않는다.

이 원칙 중에 내가 제일 안되는 것은 규칙3과 규칙 8, 규칙 9이다. 나머지는 실무에서도 신경쓰면서 개발을 하고 있는데 3,8,9는 아무리 신경써도 실무에 쉽게 적용하기가 어려웠다. 또 실무에서는 상사들이 자바 파일이 많아지는 것을 선호하지 않는다. 이번 과정을 통해서라도 3,8,9를 매우 집중하여 클린한 객체지향 코드를 작성해보고자 한다.

### 문자열 덧셈 계산기

가장 첫 단계에서는 문자열로 들어오는 식을 구분자로 나누어 더하는 계산기를 구현했다.  
예를 들어 `"1,2,3"`이 들어오면 `,`를 구분자로 하고 1+2+3의 결과인 6을 반환해야 한다.

이 과정은 매우 간단하기 때문에 나름 잘 구현했다. 첫 단계다 보니 리뷰어님들도 모두 소소한 피드백만 해주시고 머지 해주셨다.

#### 질문에 대한 답변

-   질문

```text
메서드의 책임을 줄이고자 메서드 추출을 많이 했습니다. 이 때 private 메서드가 너무 많이 생기는데, 괜찮은건가요? private 메서드는 테스트를 하기 어려워서 그리 좋은 방법 같지 않다고 생각합니다.
```

-   답변

```text
메서드를 작게 나누는 것은 좋다고 생각해요  
너무 많은 private 메서드가 생성된다면 클래스의 책임이 크지 않은가 고민해 볼 수 있을 것 같은데요  
즉 리팩토링의 신호라고 볼 수도 있을 것 같아요  
지금 고민하신 부분은 모든 미션에서 자연스럽게 다루어지니 직접 경험해 보시면 좋을 것 같습니다 😃
```

-   정리

너무 많은 private 메서드가 생성된다면 리팩터링의 신호라는 점이 인상깊었다. 메서드 분리를 하다보면 항상 private 메서드가 너무 많아져서 고민이었었는데, 이것을 신호로 보고 객체를 분리할 수 있겠다고 생각했다.

### 자동차 경주

처음으로 `getter`를 아예 안 써보려니 머리에 쥐가 났다. 조영호님의 `객체지향의 사실과 오해`같은 책을 보면 "객체에게 무언가를 달라고 요청하지말고, 무언가를 하라고 시켜라" 라고 한다. `getter`를 사용하지 않는 것인데, `getter`로 데이터를 가져와서 비즈니스 로직을 수행하면 그것은 객체지향이 아니라 절차지향이다.  
데이터와 프로세스를 서로 다른 곳에 위치하면 자율적인 객체지향이 깨지게 된다.

여기까지 이론적인 내용이다.

너무 머리아팠다. "**객체가 무슨 책임까지 가지고 있어도 되는가**" 에 대한 경계를 결정하기가 어려웠다.

#### 리뷰

1. `Random`과 같은 객체는 매번 생성할 필요 없다.

```
class RandomUtil {
  private static final BOUND = 9;

  public static int randomNumberZeroToNine() {
        return new Random().nextInt(BOUND);
  }
}
```

`randomNumberZeroToNine()`는 요청이 들어올 때마다 `Random` 인스턴스를 새로 생성해서 `nextInt`를 실행한다. 굳이 그럴 필요가 없는 게 `Random` 객체가 불변객체도 아니고 유틸성이기 때문에 하나의 인스턴스만 사용해도 아무 문제가 없을 것이다.

```
class RandomUtil {
  private static final BOUND = 9;
  private static final Random random = new Random();

  public static int randomNumberZeroToNine() {
        return random.nextInt(BOUND);
  }
}
```

클래스 변수로 올릴 수 있을 것이다. 하지만 이게 끝이 아니다. 지금 변수 `random`은 `static final`로 상수이다. 나는 지금껏 원시값만 상수라고 생각하고 대문자 컨벤션을 적용했는데, 객체도 예외는 아니다.  
이를 수정해주자!

```
class RandomUtil {
  private static final BOUND = 9;
  private static final Random RANDOM = new Random();

  public static int randomNumberZeroToNine() {
        return RANDOM.nextInt(BOUND);
  }
}
```

2.  요구사항에 유연하게 개발하기

1번 피드백에서 잠깐 봤던 `randomNumberZeroToNine` 메서드는 0에서 9까지의 숫자를 랜덤으로 가져오는 메서드이다. 만약 요구사항이 0~9가 아니라 0~10 사이의 숫자를 랜덤으로 가져와야한다면 어떻게 해야할까?

`randomNumberZeroToTen`이라는 메서드를 새로 만들거나, 기존 메서드를 수정해야할 것이다. 그러면 수정할 게 너무 많아진다.

```
class RandomUtil {
  private static final Random RANDOM = new Random();

  public static int randomNumberZeroTo(int bound) {
        return RANDOM.nextInt(bound);
  }
}
```

대신, 위와 같이 바운더리 숫자를 파라미터로 받도록 수정했다. 이렇게 되면 0~9든 0~10이든 파라미터로 넘어오기 때문에 요구사항에 유연하고 재사용할 수 있게 된다.

또, 어떤 숫자가 4 이상인 지 확인하는 `Judgement` 클래스가 있다.

```
  class Judgement {
    private static final int FOUR = 4;

    public static boolean isNumberGreaterThanFour(int number) {
        return number >= FOUR;
    }
  }
```

이 경우 역시 요구사항에 유연하지 못하다. 만약 4가 아니라 5보다 큰 값을 체크한다면? 상수 `FOUR`를 `FIVE` 로 바꾸고 메서드명을 `isNumberGreaterThanFive()` 로 바꿀 것인가? 정말 별로다.

객체지향 SOLID 원칙 중에는 **OCP(Open-Closed Principle)** 가 있다. 우리말로 개방-폐쇄 원칙인데, 자세한 내용은 [객체의 협력과 역할 그리고 책임](https://nimoh.tistory.com/18)에서 볼 수 있다.  
이 원칙의 핵심 골자는 **_변경에는 닫혀있고 확장에는 열려있어야 한다_** 는 것이다.

`isNumberGreaterThanFive`는 변화에 열려있고 확장에는 닫혀있다. 정말 GC. Garbage Code 이다. 나는 확장에 열린 코드를 적용하기 위해 Judgement 클래스를 과감하게 버리고 새로운 인터페이스를 하나 만들었다.

```
public interface MoveStrategy {
    boolean movable();
}
```

이 인터페이스의 구현체들은 이동전략을 가질 것이다. 여기서 하나의 소소한 피드백이 들어왔는데, 하나의 인터페이스에 하나의 메서드만 있다면 함수형 인터페이스이다. 즉, `@FunctionalInterface`를 명시적으로 붙여주면 좋다.

```
@FunctionalInterface
public interface MoveStrategy {
    boolean movable();
}
```

이런 함수형 인터페이스의 구현체는 람다식으로 표현할 수 있다.

이제 랜덤 숫자가 4 이상이면 전진하는 전략의 구현체를 만들어보자.

```
public class RandomNumberStrategy implements MoveStrategy {

    private static final int MOVE_STANDARD = 4;
    private static final int BOUND = 10;

    @Override
    public boolean movable() {
        return RandomUtil.randomNumber(BOUND) >= MOVE_STANDARD;
    }
}
```

이제 이 전략은 변경에 닫혀 있고, 확장에 열려있다.

왜냐하면, 요구사항이 또 변경되어 이번에는 숫자가 아닌 다른 기준에 따라 자동차가 전진하는 경우 `RandomNumberStrategy`를 **변경**하는 것이 아니라, `MoveStrategy` 인터페이스의 구현체를 새로 생성(**확장**)하면 된다.

이렇게 전략패턴으로 가져가니 테스트 역시 매우 수월해졌다.

```
    @Test
    void play() {
        // 익명 구현체 사용
        /* 
        MoveStrategy moveStrategy = new MoveStrategy() {
            @Override
            public boolean movable() {
                return true;
            }
        };
        */

        // 람다식 사용
        MoveStrategy alwaysMoveStrategy = () -> true;

        Car car1 = new Car();
        car1.play(moveStrategy);
        car1.play(moveStrategy);

        Car car2 = new Car();
        car2.play(moveStrategy);
        car2.play(moveStrategy);

        assertThat(car1).isEqualTo(car2);

    }
```

기존에는 `play()`를 테스트하기 위해서는 랜덤값에 의존할 수 밖에 없었는데, 함수형 인터페이스를 사용함으로써 항상 전진하는 전략을 주입해줄 수 있게 되었다.

또, 함수형 인터페이스를 사용하기 때문에 익명구현체를 직접 구현하지 않고 람다식을 사용하여 간단하게 나타낼 수 있게 되었다.

3.  NPE(Null Pointer Exception)이 생기지 않게 개발하기

```
    private Integer getInputtedNumber() {
        try {
            int result = scanner.nextInt();
            scanner.nextLine();     // 버퍼 제거
            return result;
        } catch (InputMismatchException e) {
            System.out.println("숫자를 입력해주세요");
            scanner.nextLine();     // 버퍼 제거
        }
        return null;
    }
```

`Scanner`를 통해 숫자를 입력받고, 예외가 발생하면 null을 리턴하는 메서드가 있었다. 리뷰어님은 null을 리턴하면 잠재적으로 NPE가 존재한다는 뜻이기 때문에 null을 최대한 사용하지 않는 것을 권장하셨다.

## 정리

리뷰어 분이 너무 꼼꼼히 리뷰를 해주셔서 블로그에 어디까지 정리해야할 지 모르겠다. 이 외에도 좀 더 자잘한 내용이 많은데, 큰 토픽은 모두 다룬 듯 하다.

처음 코드리뷰를 받아보면서 느낀 점이 있다. 바로, "리뷰 받은 내용을 까먹지 말자" 이다. 뭐 당연한 말을 하냐~ 할 수도 있다. 리뷰 받을 때는 아하 그렇군. 하고 피드백을 반영한다. 하지만 리뷰받은 것을 계속 생각하고 있지 않으면, 다음 단계의 비슷한 상황에서 똑같은 실수를 하게 될 게 뻔하다. 이걸 경계해야 한다.

벌써 1주차가 지나갔는데, 나름 재미있다. 좋은 코드를 위해 정말 머리를 싸매가며 힘들기도 했지만, 싸맨만큼 좋은 리뷰도 받을 수 있어서 매우 만족한다.

다음 주 토요일에는 오프라인에서 만나는 것도 있는데 꼭 참여해보고 싶다.