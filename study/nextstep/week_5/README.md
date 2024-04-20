어느덧 6주가 지났다. 이 과정을 진행하는 동안 시간이 정말 빠르게 흘렀다. 배운 점도 많고, 이번 주가 아마 마지막 주일 것 같은데 미션 두 개가 아직 남았다. 둘 다 마지막 단계를 리퀘스트하고 코드 리뷰를 주고받는 중이라 기간 내에 끝낼 수 있지 않을까 싶다.
과정이 완전히 종료되면 회고록도 따로 작성해야겠다.

## 피드백 정리
### 빌더 패턴
지금껏 빌더 패턴을 즐겨 사용해왔다. 책 이펙티브 자바의 Item2는 
> 생성자에 매개변수가 많다면 빌더를 고려하라

이다.
스프링부트 프로젝트에서 `Lombok` 라이브러리를 사용하면 매우 간단하게 빌더패턴을 사용할 수 있다. 때문에, 클래스에 매개변수가 많은 경우 잘 사용했다.

수강신청 미션을 진행하며 매개변수가 많아지자 평소처럼 빌더패턴을 사용하여 객체를 생성했다. 그러자 리뷰어님께서 빌더패턴의 장단점에 대해 여쭤보셨다. 

#### 장점
내가 생각한 빌더 패턴의 장점은 다음과 같다.

(1) 객체를 좀 더 직관적으로 생성할 수 있다.

(2) 객체를 좀 더 유연하게 생성할 수 있다.

물론 정적 팩터리 메서드도 꽤 직관적일 순 있으나 빌더 패턴만큼은 아니라고 생각한다.
```java
// 정적 팩터리 메서드
User.of("nimoh", "genius", 10);

// 빌더 패턴
User.builder()
    .name("nimoh")
    .nickName("genius")
    .age(10)
    .build();
```
빌더 패턴의 생성 코드가 좀 길어졌으나, 더 직관적이다. 반면 정적 팩터리 메서드는 `"nimoh"`가 이름인 지 `"genius"`가 이름인 지 한 번에 확인하기가 어렵다.(1)


또한, 정적 팩터리 메서드는 `"nimoh"`와 `"genius"`의 위치가 바뀌면 이름이 `"genius"`가 되고 별명이 `"nimoh"`가 된다. 빌더패턴은 매개변수에 대해 직접 할당하는 느낌으로 생성할 수 있기 때문에 순서는 관계없으므로 좀 더 유연하다.(2)

#### 단점
리뷰에 대한 답글을 작성하는 당시에 생각나는 단점은 있었는데 객체를 생성하는 코드가 복잡해지고 클래스 코드가 길어지고 복잡해진다는 것이다.

```java
// 정적 팩터리 메서드
User.of("nimoh", "genius", 10);

// 빌더 패턴
User.builder()
    .name("nimoh")
    .nickName("genius")
    .age(10)
    .build();
```
다시 이 코드를 보면 직관적이긴 하지만 꽤 복잡하다. 매개변수가 많으면 더 두드러질 것이다. 또한 빌더 패턴을 구현하는 코드도 꽤 복잡하다.

```java
class User {
    private final String name; // 필수 
    private String nickName;
    private final int age;    // 필수

    public User(String name, String nickName, int age) {
        this.name = name;
        this.nickName = nickName;
        this.age = age;
    }

    // 여기부터 모두 빌더패턴 구현 코드이다.

    public static UserBuilder builder() {
        return new UserBuilder();
    }

    public static class UserBuilder {
        private String name;
        private String nickName;
        private int age;

        public UserBuilder name(String name) {
            this.name = name;
            return this;
        }

        public UserBuilder nickName(String nickName) {
            this.nickName = nickName;
            return this;
        }

        public UserBuilder age(int age) {
            this.age = age;
            return this;
        }

        public User build() {
          return new User(name, nickName, age);
        }
    }
}
```
매개변수가 3개라서 다행이지 더 많으면 계속 늘어난다. 꽤나, 아니 많이 복잡하다.
사실 이 부분은 `Lombok` 라이브러리를 사용하면 매우 간단하게 빌더패턴을 사용할 수 있기때문에 실무에서 큰 문제가 되지는 않는다.

여기서 단점이 끝났다면 나는 평생 빌더패턴만 사용했을 것이다. 내 생각에 큰 단점을 찾아내고 말았다. 해당 리뷰에 대한 답을 남기고 나서, 무의식적으로 빌더패턴 장단점을 생각하며 회사에서 또 빌더패턴을 사용하던 중 갑자기 치명적인 단점을 찾았다. 바로 **어떤 필드(멤버변수)가 필수값인 지 알 수가 없다는 것**이다.

다시 `User` 클래스를 보자.
```java
class User {
    private String name;
    private String nickName;
    private int age;

    public User(String name, String nickName, int age) {
        this.name = name;
        this.nickName = nickName;
        this.age = age;
    }

    // 여기부터 모두 빌더패턴 구현 코드이다.

    public static UserBuilder builder() {
        return new UserBuilder();
    }

    public static class UserBuilder {
        private String name;
        private String nickName;
        private int age;

        public UserBuilder name(String name) {
            this.name = name;
            return this;
        }

        public UserBuilder nickName(String nickName) {
            this.nickName = nickName;
            return this;
        }

        public UserBuilder age(int age) {
            this.age = age;
            return this;
        }

        public User build() {
          return new User(name, nickName, age);
        }
    }
}
```
이제 `name`과 `age`는 필수 필드이다. 빌더 패턴으로 해당 클래스를 만들어보자.
```java
User user = User.builder().nickName("genius").build();
```
이제 어쩔 것인가? 물론 내부 구현에서 필수 값을 검증해서 예외를 던지는 등 예외처리는 할 수 있다. 그런데 그건 그 클래스 내부 사정이다. 생성자 혹은 정적 팩터리 메서드로 생성하면 필수값 누락은 곧 매개변수가 누락이므로 컴파일 에러가 난다. 또, IDE의 도움으로 어떤 필드가 생성에 필요한 지 알 수 있다. 그런데 빌더패턴은 불가능하다. 직접 해당 클래스에 들어가서 작성되어있는 주석을 확인하거나 예외처리를 확인해봐야 한다.

#### 그래서 결론은?
이렇듯 빌더패턴은 매우 유연하면서도 유연하지 않은 방법이다. 따라서, 매개변수가 많을 때 사용하면 매우 유용하지만 아무데나 사용하면 안된다. 만약 필수값에 대한 검증이 불가피할 경우 차라리 생성자나 팩터리 메서드 사용을 유지하고, 유사한 매개변수끼리 새로운 객체로 묶는 것이 훨씬 가독성, 객체지향 측면에서 좋은 선택일 수 있다.

### 점진적 리팩터링
이 부분은 피드백은 아니고, NEXTSTEP의 자바지기 박재성님이 꽤나 강조하시는 부분이다. 점진적인 리팩터링이라는 것은 달리는 자동차가 멈추거나 고장나지 않게 바퀴를 갈아끼우는 것이다. 그렇기 때문에 **IS-AS와 TO-BE의 공존**이 중요하다. 보통 리팩터링이라고 하면 IS-AS를 지워버리고 TO-BE를 작성한다. 그렇게 하면 엄청난 사이드 이펙트가 발생할 것이다. 컴파일 에러 뿐만 아니라 예상치 못한 충격의 런타임 에러까지 발생할 수 있다. 정말 기도메타로 개발 및 배포해야한다.

그 대신, 점진적으로 리팩터링을 하면 사이드 이펙트를 최소화 할 수 있다. 간단한 예를 들어보자.
```java
class Board {
    private List<Post> posts;

    public Board(List<Post> posts) {
        this.posts = posts;
    }

    public void deletePost(Long postId) {
        posts.remove(postId);
    }
}
```
예제를 위해 정말정말 간단한 코드를 작성했다. 게시글(Post)의 리스트를 가진 게시판(Board)이 있다. 이 때, `List<Post>`를 일급컬렉션인 `Posts`로 리팩터링하려고 한다. 그냥 무작정 `List<Post>`를 `Posts`로 다 변경해버릴까? 만약 시스템의 모든 구석구석을 1도 빠짐없이 파악하고 있다면 그렇게 하라. 그게 아니면 점진적으로 리팩터링을 하자.
```java
class Board {
    private List<Post> posts;// 우선 삭제하지 않음
    private Posts posts2;   // 변경할 필드를 생성 (AS-IS와 TO-BE 공존)

    public Board(List<Post> posts) { // 동일한 매개변수를 받아, Posts로 자연스럽게 변경
        this.posts2 = new Posts(posts); 
    }

    public void deletePost(Long postId) {
        posts2.remove(postId);  // Posts 객체에 기존 메서드를 이관
    }
}
```
`Posts`는 이미 구현했다는 가정하에 말하겠다. 우선 임시로 `posts2` 필드를 생성하고 생성자를 변경했다. 생성자의 매개변수가 변경되지 않았기 때문에 `Board`를 생성하던 외부에서는 기존 코드를 수정할 필요없다. 

점진적 리팩터링에서 말하는 AS-IS(`List<Post>`)와 TO-BE(`Posts`)가 잠깐 공존하게 되었다. 이렇게 한 단계씩 점진적으로 수정하다보면 어느새 IDE는 `List<Post>`의 `posts`가 더 이상 사용되지 않는다고 알려줄 것이다. 그러면 그제서야 AS-IS를 제거해주면 된다.

## 6주차 회고
NEXT STEP의 미션을 진행하면서 가장 가슴이 답답했던 부분은 `indent`(들여쓰기)를 1로 하라는 규칙이었다. 일반적인 로직은 그냥 메서드로 분리하거나 클래스로 분리하면 간단하게 해결되었다. 하지만 `while` 문의 경우에는 참 어려웠다. 내부에서 `if`문 하나만 들어가도 `indent`가 2가 되어버렸기 때문에 참 난감했다. 뿐만 아니라 반복을 끝내주는 시점인 `break`가 명시적으로 필요한 경우가 많았는데 이럴 때마다 골치 아팠다. 반복문 대신 `Stream API`로 처리할 수 있는 부분은 처리했고, 나머지는 재귀로 작성했다. 리뷰어마다 재귀를 불호하거나 while문을 불호하거나 다들 의견이 달라서 나 역시 조금 머릿속이 복잡했다. 

사실 이 이야기의 핵심은 `while`을 쓰나 재귀를 쓰냐는 것이 아니다. 과정이 끝나고 앞으로 개발하면서 이런 비슷한 부분에 대한 고민을 많이 하게될텐데, 내 철학이 잡혀나가야한다는 것이다. 개발 방법에 Best Practice는 있지만 정답은 없다. 어떤 코드건 방법이건 트레이드 오프가 있는 법이고, 우리가 속히 아는 클린 코드가 누구에게는 더티 코드가 될 수도 있는 것이다. 아직은 내가 이제 갓 1년이 지난 개발자이기 때문에 언어나 프레임워크에 대한 철학이 잡히지는 않았다. 책에서나 누가 좋다고 하면 나도 좋은 것 같고 따라야할 것만 같다. 하지만 내가 성장하기 위해서는 남이 말하는 것을 곧이곧대로 따를 게 아니라 "역시 과연 좋은 게 맞을까?" 라는 고민을 해야할 것이다.