회사 프로젝트 마무리 단계에서 보안 취약점 검사를 진행했는데 SQL Injection 에 관한 취약점이 다량 발견되었다.  
SQL Injection은 말 그대로 화면 등에서 SQL을 주입하여 개발자의 의도와 다르게 SQL문을 수행 시키는 것이다.

## 문제

```
SELECT id
FROM users
WHERE
    id = 'inputId' AND pw = 'inputPw'
```

라는 SQL이 구현됐다 해보자.

만약 사용자가 로그인 화면에서 id에 inputId'-- 라고 넣으면 어떻게 될까?

```
SELECT id
FROM users
WHERE
    id = 'inputId'-- AND pw = 'inputPw'
```

결론적으로 --가 패스워드를 검증하는 AND절을 주석 처리하여 ID만 알면 로그인이 되어버린다.  
이는 치명적인 보안 상의 문제이며 사용자의 개인정보가 위험하다.

혹은 이런 방법도 있다.

```
SELECT id
FROM USER
WHERE
    id = 'inputId' AND PASSWORD = '1' OR '1' = '1'
```

위 방법은 id에는 input이라고 넣고 비밀번호에 `1' or '1' = '1`이라 넣은 것이다.  
이러면 WHERE 절을 OR 조건으로 바꾸어서 '1' = '1' 로 조건을 통과하게 된다. 이 역시 매우 치명적이다.

SELECT 문 뿐만이 아니라 INSERT, UPDATE, DELETE도 마찬가지다.

```
INSERT 
    BOARD(id, contents)
VALUES
    (
        'inputId'
        , '글의 내용')
```

위의 SQL문은 게시판에 글을 등록하는 매우 간단한 INSERT 쿼리이다. 여기에는 치명적인 취약점이 있다.

```
INSERT 
    BOARD(id, contents)
VALUES
    (
        'inputId'
        , '글의 내용'); DELETE FROM BOARD --)
```

글의 내용에 `글의 내용'); DELETE FROM BOARD --` 라고 적은 것이다. 이렇게 하면 INSERT한 다음 BOARD 게시판 자체를 삭제해버린다.

## 방어

### 입력 값 유효성 검사

가장 기본적인 대응 전략이다.  
사용자가 입력한 외부 입력 값에 대하여 항상 유효성 검사를 한다.

#### 블랙 리스트 방식

문제가 될만한 요소들을 사전에 지정하여 막아버리는 방법이다.

##### 1\. SQL 쿼리의 구조를 변경 시키는 문자 제한

SQL 예약어인 `UNION`, `GROUP BY`, `COUNT()`등의 함수와 세미콜론, 주석 등의 문자를 제한한다. 이러한 문자가 입력되었을 시에 다른 공백 등으로 치환한다.

##### 2\. 공백 후에도 정교한 체크

`SESELECTLECT`를 주입했을 때 중간에 있는 SELECT를 공백으로 치환(SE"SELECT"LECT) 하더라도 SELECT가 남는다. 이런 예외 사항까지 정교하게 방어할 필요가 있다.

#### 화이트 리스트 방식

화이트 리스트 방식은 블랙 리스트 방식과 반대로 허용된 문자를 제외하고는 모두 막는 방법이다. 때문에 블랙 리스트 방식보다 강력하다.

### Prepared Statement 사용

Prepared Statement를 활용하면 쿼리의 문법 처리 과정이 **미리 컴파일** 되어 있기 때문에 외부 입력 값으로 SQL 관련 구문이나 특수 문자가 들어와도 그것은 SQL 문법으로서 역할을 할 수 없게 된다.

```
SELECT * FROM USER
WHERE ID = ? AND PASSWORD = ?
```

위 쿼리와 같이 어떤 값이 들어갈 지 물음표로 대체되어 있다.  
`Prepared Statement`는 이 부분을 미리 컴파일하고 ?에 해당하는 데이터를 바인딩 하기 때문에 데이터는 SQL 문법으로서의 역할을 할 수 없다.

##### mybatis에서의 활용

mybatis에서 쿼리에 데이터를 바인딩하는 방법에는 `${}`와 `#{}`가 있다.  
`${}`은 Statement의 역할을 한다. 값 그 자체를 쿼리에 적용한 후 컴파일 하기 때문에 SQL Injection이 발생할 수 있다.  
`#{}`은 Prepared Statement를 사용하기 때문에 SQL Injection을 방어한다. 방어가 가능한 이유는  
다음과 같다.

```
-- ${} 사용
SELECT * FROM USER
WHERE ID = admin AND PASSWORD = 1234

-- #{} 사용
SELECT * FROM USER
WHERE ID = 'admin' AND PASSWORD = '1234'
```

ID 값에는 `admin`을, PASSWORD 값에는 `1234`를 넣었다.  
`${}`로 작성된 코드는 값 그 자체가 나온다.  
그렇기 때문에 실제로 사용하려면 쿼리 자체를 `'${id}'` 의 형태로 작성해야하는데 이 때 SQL Injection이 발생한다.  
id 값에 `admin'--` 라고 넣어버리면 위에서 본 대로 ID만 알아도 로그인이 된다.

```
-- ${} 사용
SELECT * FROM USER
WHERE ID = 'admin'-- AND PASSWORD = 1234
```

`#{}`은 따옴표를 씌운상태로 컴파일이 되기 때문에

```
-- ${} 사용
SELECT * FROM USER
WHERE ID = 'admin'--' AND PASSWORD = 1234
```

가 되어 `admin'--`가 모두 문자열로 취급된다.