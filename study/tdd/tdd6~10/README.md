## 6장 - 모두를 위한 평등
6장에서는 `Dollar`와 `Franc` 공통 부분을 찾아 상위 클래스인 `Money`로 옮긴다. 그 과정에서 이미 공통된 부분인 `amount` 인스턴스 변수를 `Money`로 옮겨 주었다.
```java
class Dollar extends Money {
//  private int amount; // Money로 이동
  ...
}

class Franc extends Money {
// private int amount; // Money로 이동
  ...
}

abstract class Money {
  protected int amount;
  ...
}
```
이 때, 하위 클래스에서 올라온 `amount`는 `protected`로 선언되어, 실질적으로 하위 클래스에서 `private`을 사용하는 것과 동일하게 만들어주었다.

그리고 각 객체에서 구현이 서로 달랐던 `equals()`를 동일하게 만들어주어 `Money`로 올려주었다. 동일할 수 있는 구현부들을 최대한 동일하게 만들어서 상위 클래스에 올린 것이다.

이 때 `Money`에 올라간 `equals()`는 `public`으로 선언되었다. 당연한 얘기지만 공통부분이 애초에 `public`이라면 상위 클래스에서도 `public`일 것이다. 그런데 만약 `private`이었다면 조금 고민을 해봐야 한다.

공통 부분을 상위 클래스로 올린 다음에도 하위 클래스에서 해당 부분을 사용해야 한다면 상위 클래스에서 `protected`로 선언해야하고, 사용하지 않는다면 `private`으로 해주어도 된다.

## 7장 - 사과와 오렌지
지금 `equals()`를 보면 `Dollar`와 `Franc`를 동등 비교할 수 없다.
```java
@Override
    public boolean equals(Object obj) {
        Money money = (Money) obj;
        return amount == money.amount
                && getClass().equals(money.getClass());
    }
```
메서드는 `equals` 이지만 실제로는 동등성 비교가 아니라 동일한 **타입**인 지 비교하기 때문에 `dollar.equals(franc)`은 실패한다. 
저자는 이를 확인하고 통화 개념을 적용할까 하다가 일단 넘어갔다. 통화 개념을 도입할 충분한 동기가 없었기 때문이다. 더 많은 동기가 있기 전에는 더 많은 설계를 도입하지 않기로 했다.

## 8장 - 객체 만들기
이번 장에서는 `times()`를 공통화 하려고 한다.
```java
Franc times(int multiplier) {
  return new Franc(amount * multiplier);
}

Dollar times(int multiplier) {
  return new Dollar(amount * multiplier);
}
```
뭔가 비슷한 듯 애매하다. 잘 보면 어차피 `Franc`과 `Dollar`가 `Money`의 상속을 받는 클래스들이기 때문에 조금씩 `Money`로 치환할 수 있다.

```java
Money times(int multiplier) {
  return new Franc(amount * multiplier);
}

Money times(int multiplier) {
  return new Dollar(amount * multiplier);
}
```
근데 return 부분을 `Money`로 바꿔주기에는 아직 `Franc`과 `Dollar`를 구분할 방법이 없다. 좀 더 기다려보자. 

이렇게 하위 클래스에 대한 직접적인 참조를 하나씩 줄여가면, 하위 클래스를 제거하는 데에 한 걸음씩 나아간다.

```java
abstract class Money {
  ...
  public Dollar dollar(int amount) {
    return new Dollar(amount);
  }

  public Franc franc(int amount) {
    return new Franc(amount);
  }
  ...
}
```

이렇게 객체를 생성하여 반환해주는 메서드를 팩터리 메서드라 한다. 
갑자기 생각난건데,내가 알기로는 이펙티브 자바 가장 첫번 째 주제가 **생성자 대신 정적 팩터리 메서드를 고려하라**이다.
이펙티브 자바를 항상 5장까지 읽고 덮었기 때문에 기억이 났다.

```java
class Dollar {
  ...
  static Dollar of(int amount) {
    return new Dollar(amount);
  }
  ...
}

// client
Dollar amountOfDollar = Dollar.of(5);
```

정적 팩터리 메서드를 사용하면 생성자가 이름을 가질 수 있어서 가독성이 좋아진다. 하지만 생성자를 사용하지 않아서 상속하기가 어렵다는 단점이 있다. 다른 장단점이 많지만 주제와 다른 얘기므로 넘어가겠다.

다시 돌아와서, 
이제 클라이언트 코드에서 하위 클래스를 직접 참조하여 객체를 생성할 필요가 없다.

```java
Money dollar = Money.dollar(5);
Money franc = Franc.dollar(5);
```

## 9장 ~ 10장
`times()`를 공통화하려면 `Franc`과 `Dollar`를 구분할 수 있어야 한다.  그러기 위해 `currency`라는 개념을 도입한다.

리팩터링 후 남은 코드는 다음과 같다.

```java
public class Money {
    private final int amount;
    protected String currency;

    public Money(int amount, String currency) {
        this.amount = amount;
        this.currency = currency;
    }

    public static Money dollar(int amount) {
        return new Dollar(amount, "USD");
    }

    public static Money franc(int amount) {
        return new Franc(amount, "CHF");
    }

    public Money times(int multiplier) {
        return new Money(amount * multiplier, currency);
    };

    @Override
    public boolean equals(Object obj) {
        Money money = (Money) obj;
        return amount == money.amount
                && currency().equals(money.currency);
    }

    public String currency() {
        return currency;
    };

    public String toString() {
        return amount + "  " + currency;
    }
}


public class Dollar extends Money{
    public Dollar(int amount, String currency) {
        super(amount, currency);
    }
}

public class Franc extends Money{
    public Franc(int amount, String currency) {
        super(amount, currency);
    }
}
```

이제 생성자와 팩터리 메서드가 혼용된다. 현재 `Money` 클래스는 `equals`로 `currency`가 달러인지 프랑인 지를 비교하기 때문에 `currency`만 있으면 `Franc`와 `Dollar` 객체가 더 이상 필요가 없다. 
아마 다음 장에서 하위 클래스들을 모두 없애고 팩터리 메서드도 지우지 않을까 예상해본다.

## 정리
6~10장에서는 주로 하위 클래스를 상위의 추상 클래스로 공통화하는 작업을 수행하였다. 이 과정에서 조금씩 프로덕트 코드가 변할 때마다 테스트를 실행해주는 것이 인상깊었다.

인터페이스만 사용하고 추상클래스를 잘 사용하지 않았는데 생각보다 많이 사용해도 될 것 같다는 느낌이다.
