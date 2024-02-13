
### 연관관계의 종류

#### `@OneToMany` (일대다 관계)
일대다 관계는 위에서 본 `team`과 `member`의 관계이다. 



#### `@OneToOne` (일대일 관계)
예를 들면 사람과 사물함의 관계다.
한 명의 사람은 하나의 사물함을 가지고, 하나의 사물함 역시 한 명의 사람에게 사용된다. (일반적이지 않은 예외는 무시한다)

```java
@Entity
public class Locker {

    @Id
    @GeneratedValue
    private Long id;

}


@Entity
public class Member {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    @OneToOne  // 일대일 매핑
    @JoinColumn(name = "locker_id")
    private Locker locker;

}
```
