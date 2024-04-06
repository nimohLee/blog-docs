이번 주는 진도를 많이 못 뺐다. 회사 회식과 뭐 이것저것 하다보니 할 시간이 많이 없었다(핑계). 사다리 미션을 하고 있는데 개인적으로 난이도가 제일 어려워서 헤매는 중이다. 무엇이 어렵냐면, 이번 미션은 객체지향 보다는 어떤 식으로 사다리를 **구현**할 것인지에 대한 것이 주가 되어 많이 난감했다. 지금까지 객체지향, 책임과 역할 등을 고민해왔는데 이번 미션은 조금 달랐다. Optional, Stream, Lambda 등이 주가 되는 미션인데 사실 아직 잘 모르겠다. 일단 피드백 받은 것을 정리하겠다.

## 피드백 정리
### 1. `setter`는 **금지** `getter`는 **지양**
커리큘럼 초반에 `getter`를 사용하지 말라는 규칙이 있어서 최대한 사양하지 않으려고 노력했다. 가끔 어쩔 수 없이 (출력 로직 등에서) `getter`를 만들어야할 때도 있었다. 이럴 때마다 정말 스트레스(?)를 많이 받았다. `getter`를 만들지 말라하는데, 출력 책임과 도메인은 분리해야겠고... 그럴려면 `getter`가 꼭 필요했다. 

결론은 "`setter`와 `getter`를 분리해서 생각하라" 였다. 제목처럼 `setter`는 **금지**이다. 도메인 객체에 `setter`가 있다면 더 이상 예측할 수 없는 객체가 된다. 객체의 상태는 그 객체에게 메시지를 보내, 행위로 하여금 변경되게 하는 것이 객체지향의 기본 철칙이다. 하지만 `setter`는 메시지라기 보다는 객체의 상태를 객체 외부(클라이언트)에서 직접 집어 넣는 것이다. 따라서 객체는 응집도가 떨어지고, 더 이상 자율적이지 않게 된다.

하지만 `getter`는 <span style="color: tomato;">금지</span>가 아니라 <span style="color: orange;">지양</span>이다. 최대한 사용하지 않고 메시지로 객체와 소통하기 위해 아등바등 해야한다. 그래도 `getter`가 필요하다면 만들어라. 즉, `getter`는 **최후의 보루**이다.

### 2.생성자를 많이 생성하는 것은 좋다.
리팩토링을 하다보면, 클라이언트 코드를 모조리 수정해야 하는 경우가 있다. 이 때에는 점진적인 리팩토링이 중요하다. AS-IS와 TO-BE를 공존하도록 만드는 것이다. 다음과 같은 예시를 보자.
```java
public class Test {
  private int testInt;
  private String testString;
  
  public Test(int testInt, String testString) {
    this.testInt = testInt;
    this.testString = testString;
  }
}
```

예를들어 생성자로 정수 `testInt`와 문자열 `testString`을 받아 생성하는 `Test` 클래스가 있다. 만약, testString 부분에 정수타입(int, Integer)이 들어와야하면 다음과 같이 할 수 있을 것이다.

```java
public class Test {
  private int testInt;
  private String testString;
  
  public Test(int testInt, int testString) {
    this(testInt, String.valueOf(testString));
  }

  public Test(int testInt, String testString) {
    this.testInt = testInt;
    this.testString = testString;
  }
}
```
이렇게 생성자를 하나 더 만들어 유연하게 변경할 수 있을 것이다. 이렇게되면 현재 잘 돌아가는 클라이언트 코드에서 사이드 이펙트가 터질 위험이 없기 때문에 안정적이고 점진적으로 리팩토링이 가능하다. 결국에는 `testString`이 정수로만 들어오게 된다면 자연스럽게 생성자를 변경해도 된다.

잘 설계된 객체는 인스턴스 변수와 메서드가 적고, 생성자가 많은 객체이다. 이 것을 염두에 두고 설계 및 개발하도록 하자.

### 3. 서비스 계층과 도메인 로직을 분리하자.
보통 웹 개발을 하면 서비스 계층에서 많은 비즈니스 로직이 수행한다.

```java
@Service
@RequiredArgsConstructor
public class TestService {
  private QuestionRepository questionRepository;

  public void test(NsUser loginUser, long questionId) {
     Question question = questionRepository.findById(questionId).orElseThrow(NotFoundException::new);

    NsUser owner = question.getOwner();
    
    if (owner.getId().equals(loginUser.getId())) {
      throw new IllegalArgumentException("질문을 삭제할 권한이 없습니다.");
    }
    ...
  }
}
```
이게 바로 SI식 코드이다. 내가 너무 SI를 비하하는 걸 수도 있는데 최소한 내가 일하는 환경은 이렇다.
객체(사실 객체라 할만한 것은 DTO 밖에 없다.)에는 생성자, 인스턴스 변수, `getter`, `setter`만 있고 그 어떤 메서드도 없다. 그렇기 때문에 인스턴스 변수를 모두 서비스 레이어로 가져와서 처리한다.

이것은 전혀 객체지향적이지 않다. 심지어 절차지향적이다. 나 역시 회사 코드를 보며 객체지향이라고 생각드는 부분이 거의 없었다. 보통 계층형으로 개발하기 때문에 `Controller`, `Service`, `Repository`만 지키고, 객체지향이라는 것은 어디서도 느낄 수 없었다. 심지어 나는 JPA가 아닌 Mybatis를 통해 DB에 접근해서, 엔티티도 없었다. 그러다 보니 모든 로직이 서비스 레이어에 몰리게 되었고, 그것이 SI에서는 읽기 쉬운 좋은 코드가 된다. (우리 회사만 그런 건지는 잘 모르겠다.)

갑자기 회사 욕이 됐는데 나중에 하자. 정리하자면, 객체를 사용하는 로직은 그 객체에 있으면 된다. 상태와 행위가 같은 곳에 위치할수록 좋은 코드이다. 위의 코드를 조금 바꿔보자.
```java
@Service
@RequiredArgsConstructor
public class TestService {
  private QuestionRepository questionRepository;

  public void test(NsUser loginUser, long questionId) {
     Question question = questionRepository.findById(questionId).orElseThrow(NotFoundException::new);
    
    question.isNotOwnerThenThrow(loginUser);
    ...
  }
}

public class Question {
  private NsUser owner;

  ...

  public void isNotOwnerThenThrow(NsUser loginUser) {
    if (!owner.equals(loginUser)) {
      throw new CannotDeleteException("질문을 삭제할 권한이 없습니다.");
    }
  }
}
```
급하게 만드느라 위 코드도 그리 좋은 코드는 아니지만, 어쨌든 `Question`의 owner인 지를 판단하는 로직을 `Question`으로 넣어줬다. 서비스 레이어에서는 `Question`를 헤집어 `owner`를 가져올 필요 없이 메시지만 보내주면 된다. 이 연습을 위해서는 `getter`와 `setter`를 무작정 만들지 않는 것이 중요하다. `getter`가 있으면 무의식적으로 사용하게 되더라.


## 두 번째이자 마지막 모짝미 (모여서 짝 미션)
어제는 두 번째 모짝미를 다녀왔다. 내 사다리가 지금 휘청거려서, 어쩔 수 없이 마지막 미션의 1단계를 같이 했다. 같이 미션을 한 짝은 서비스 기업에 다니시고, NEXTSTEP 과정도 회사 복지로 등록하셨다고 한다. 나는 강의와 커리큘럼으로 내 돈 몇 백만원을 쓰고 있기 때문에 참 부러웠다. 물론 내가 내 성장과 자기계발을 위해 사용한 돈은 전혀 아깝지 않지만, 그 돈으로 다른 곳에 쓰거나 투자할 수도 있었기에 아쉬움이 없진 않았다.

그래서 많은 자극을 받고 왔다. 좋은 서비스 기업에 가는 것에 대한 열망이 좀 더 생겼다. 그 분 역시 내 생각엔 좋은 서비스 기업에 다니고 계셨는데, 나는 개발자 인맥이 전혀 없기 때문에 그 분과 친하게 지내고 싶었다. 하지만 이성이어서 치근덕대는 것 같을까봐 다가가기가 좀 그랬다... 어쩌다보니 초등학생 일기장처럼 됐네..🥲? 

## 4주차 회고
사다리 미션이 개인적으로 너무 어려워서 진도가 잘 안 나간다. 앞의 두 미션은 객체지향이 주가 됐다면, 이번 미션은 구현의 문제인 듯 하다. 게다가 주제는 함수형 프로그래밍인데 아직 어느 부분에서 사용해야하는 지 감이 안 잡힌다.

오늘은 회사에 대한 아쉬운 점도 적어보려고 한다.
스프링 StreoType이 아닌 모든 객체가 다 DTO로 통용되어 사용되는 중이고, 선배 개발자는 `VO`와 `DTO`를 동일한 것으로 간주하고 `DTO`라는 명칭만 쓰자는 컨벤션을 공유하셨다.
도메인 객체라 할만한 객체가 없다. PL은 객체가 많아지는 것을 싫어하셔서 핀잔을 주신다. 객체보다는 `Map` 사용을 선호하신다. 과정에서 배우는 것과 실무 동료들의 마인드가 너무나 달라서, 괴리감이 오기도 한다.

깃 컨벤션은 이름과 날짜를 넣고 내용을 작성하는 것이다. 커밋에 어차피 이름과 날짜가 남는데 왜 굳이..? 라는 생각이다.

이런 개발환경 속에서 더 좋은 곳으로 발돋움할 수 있을까? 개발자 네트워크도 없는 내가?.. 아직 잘 모르겠다. 내가 하고 있는 이 하루하루의 노력들이 언제 빛을 바랄 지도 모르겠다.

그래도 포기하지 않으려고 한다. 개발자의 성장은 잠잠하다가 갑자기 수직상승한다고 어디선가 본 적 있다. 그건 일만하는 개발자가 아니라 스스로 학습하고 성장하려고 해야하는 것이라 생각한다. 내 얘기가 되도록 꾸준히 해보자. 

> 오늘 하루를 나중에 후회하지 않도록

