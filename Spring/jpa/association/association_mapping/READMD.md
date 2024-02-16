
## 연관관계

### 연관관계의 방향
연관관계는 방향이 있다. 연관 상대 객체를 참조하고 있는 쪽에서 상대 객체 쪽으로 방향이 흘러간다.

#### 단방향 매핑
```java
@Entity

```

### 연관관계 매핑

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
