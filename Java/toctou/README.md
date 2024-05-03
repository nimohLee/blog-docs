### TOC TOU 경쟁조건
**TOCTOU(Time Of Check to Time Of Use)** 경쟁 조건은 컴퓨터 보안 및 프로그래밍에서 자주 발생하는 일반적인 문제 중 하나이다. 이 문제는 시스템이 두 단계로 나뉘어 수행될 때 발생하는데, 하나는 어떤 조건을 검사하는 단계이고 다른 하나는 그 조건에 기반하여 어떤 동작을 실행하는 단계이다. 각각을 순서대로 1단계, 2단계라 칭하겠다. 만약 이 두 작업 사이에 상태가 변경된다면, 원래의 검사가 무의미해져 프로그램이 예기치 않은 행동 혹은 예외를 발생시키게 된다.


### 발생 상황 예시
TOCTOU RaceCondition이 발생하는 상황의 몇 가지 예시를 들어보겠다.

#### 1. 파일 시스템 접근
보통 파일에 접근하거나 수정할 때, 해당 파일이 존재하는 지 확인하고 접근하게 된다.
```java
if (file.exists()) {
	// 파일 접근 및 수정
}
```
앞서 알아본 TOCTOU에 의하면, 검사와 검사에 기반한 동작 실행의 단계로 나뉘어져있으면 TOCTOU 경쟁조건이 발생한다고 하였다. 위 코드가 바로 그 상황이다.
이 경우 파일의 존재 유무를 **먼저 검사**(1단계)하고, **파일이 존재하면**(2단계) 파일에 접근한다. 만약 파일의 존재 유무를 확인한 결과 파일이 존재했는데 파일에 접근하기 전, 그러니까 **파일 존재 확인과 그에 따른 파일 접근 사이**에 파일이 수정되거나 삭제되면 예기치 않은 예외가 발생할 수 있다.
이 예시는 단계가 있는 상황이라는 점에서 다른 모든 예시와 사실 상 유사하다.

#### 2. 사용자 권한 검사
**사용자 권한을 검사**(1단계)하고 **그 권한에 따라 동작을 실행**(2단계)하는 로직 역시 TOCTOU 경쟁조건이 발생한다. 사용자 권한을 검사한 당시에는 `관리자`였는데, 동작을 실행하기 직전에 `일반 사용자`로 변경되었다면 이 역시 예기치 못한 예외가 발생할 수 있다.

#### 3. 인터넷 거래
**재고를 확인**(1단계)하고 **재고에 따라 구매 처리 및 재고 소진하는 등의 거래**(2단계)에서도 TOCTOU 경쟁조건이 발생한다. 이 부분은 사용자의 실질적인 손해가 걸려있기 때문에, 좀 더 신중할 필요가 있다.

#### 4. 멀티스레딩 환경
3번은 멀티스레딩과 관련이 있는 예시이다. A 스레드가 **상태확인**(1단계)을 하고 **수정**(2단계)을 해야하는데, B 스레드 역시 동시에 진행하는 경우 발생한다.
```java
public class Counter {
	private int count;

	public void increment() {   // count 수정
		count++;                // count 상태 확인
	}

}
```
이 경우 명시적으로 `if`문을 사용하는 단계는 없지만, 상태에 따른 로직을 수행한다는 맥락에서 TOCTOU 경쟁조건으로 분류된다.
```
각각의 스레드는 동시호출됨

1. A스레드 incremnet() 호출 count 상태 확인. count = 1
2. B스레드 incremnet() 호출 count 상태 확인. count = 1    
3. A스레드 count 수정. count++   => count = 2
4. B스레드 count 수정. count++   => count = 2
```
만약 `count`의 초기값이 1이라면 `increment`가 두 번 호출되기 때문에 모든 작업이 수행된 후 `count`는 3이 되어야 한다. 하지만 스레드는 동시에 작동하기 때문에 동시에 `increment()`를 호출하는 경우, `Counter` 객체의 `count`의 상태를 두 스레드 모두 1로 읽어 결과적으로 count가 2가 되는 경우가 발생할 수 있다.

이외에도 TOCTOU 경쟁조건이 발생하는 경우는 많다.

### 해결 방법
TOCTOU는 파일시스템, 프로세스, 네트워크 등 다양한 방면에서 발생한다. 그 중 가장 흔하게 발생하는 멀티스레드 환경에서의 해결방법에 대해 알아보자. 나는 자바에서 TOCTOU 문제에 직면했기 때문에, 자바 위주의 해결 방법을 설명하겠다.

#### synchronized 사용
자바에서는 메서드 시그니처에 `synchronized`라는 키워드를 붙일 수 있다. `public synchronized void increment()`
해당 키워드가 붙은 메서드는 한 번에 하나의 스레드만 접근할 수 있게 된다.

```java
public synchronized void increment() {
	count++;
}
```

다시 멀티스레딩에서 다뤘던 예제를 보자. 위와 같이 `synchronized`를 붙여줌으로써 해당 메서드는 다음과 같이 호출된다.
```
각각의 스레드는 동시호출됨

1. A스레드 incremnet() 호출 count 상태 확인. count = 1
2. A스레드 count 수정. count++   => count = 2
3. B스레드 incremnet() 호출 count 상태 확인. count = 2    
5. B스레드 count 수정. count++   => count = 3
```
해당 메서드는 동기화되어 우리가 A 스레드의 메서드 접근이 모두 종료되면, B 스레드가 접근하게 된다. 결과적으로 `count`는 우리가 처음에 예상했던 3이 된다.

참고로 `synchronized`는 다음과 같이 `synchronized block`으로도 사용될 수 있다.
```java
public void increment() {
	synchronized(this) {
		count++;
	}
}
```

주의할 점은, `synchronized`를 내부적으로 락을 사용하여 스레드의 접근을 관리하기 때문에 남발하게 되면 크고 작은 오버헤드가 발생할 수 있다.

#### Lock 인터페이스 사용
`java.util.concurrent.locks.Lock` 인터페이스를 사용하는 방법도 있다. 예시는 다음과 같다
```java
import java.util.concurrent.locks.ReentrantLock;
public class Counter { 
	private final ReentrantLock lock = new ReentrantLock(); 
	private int count = 0; 
	public void increment() { 
		lock.lock(); 
		try { 
			count++; 
		} finally { 
			lock.unlock();
		} 
	 }
}
```
`ReentrantLock`을 사용하여 명시적으로 락을 걸어 해당 메서드를 보호한다. 이 방법은 명시적으로 락을 제어함으로써 좀 더 유연하게 동기화를 제어할 수 있지만, `ReentrantLock`를 참조해야 하고 `Lock`을 해제해주기 위해 `try-with-resource`구문을 써야하는 등 코드가 조금 복잡해질 수도 있다.

#### Atomic Variables 사용
자바에는 변수의 원자성을 보장해주는 `Atomic`이 있다.
예를 들면 기본 타입으로는 `AtomicInteger`, `AtomicBoolean`, `AtomicLong` 등이 있고 참조 타입은 `AtomicReference`가 있다. 자세한 API는 [자바 11 Atomic 공식문서](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/package-summary.html)를 참조하기를 바란다.
```java
import java.util.concurrent.atomic.*; 
public class Counter {
		private AtomicInteger count = new AtomicInteger(0); 
		
		public void increment() { 
			count.incrementAndGet(); 
		}
}
```
`Atomic`은 말 그대로 원자성을 보장해주기 때문에 Thread safety하다. 원자성을 보장해준다는 것은 더 이상 쪼갤 수 없다는 것인데, 여기서는 작업을 쪼갤 수 없다는 의미가 된다.
중요한 것은 원자성을 보장하고 Thread safety한다고 각 쓰레드가 서로 다른 값을 가지고 있는 것은 아니다. 공통의 값을 가지고서 멀티 스레드의 접근 시에 동시성문제를 해결한 것이지, ThreadLocal 처럼 각 스레드가 다른 값을 가지고 있다는 것은 아니다.


### 결론
자바에서 싱글 스레드 환경(main 스레드(메서드)만 존재)에서는 TOCTOU와 같은 경쟁조건이 발생하지 않는다. 애초에 하나의 스레드에서만 진행되기 때문에 경쟁 대상이 없다. 하지만 스프링을 사용할 때는 다르다. 스프링은 기본적으로 멀티스레드로 동작한다. 그런데 개발할 때에 개발자는 싱글스레드 애플리케이션을 만드는 것처럼 쓰레드에 대한 직접적인 생성, 호출, 제거를 할 필요는 없다. 그 이유는 HTTP 요청에 따라 서블릿 컨테이너에서 쓰레드를 생성하거나 쓰레드 풀로 부터 할당받아 사용한다. 따라서 웹 개발에서는 동시성 문제를 인지하고 방어하는 것이 중요하다.
많은 상황에 발생할 수 있지만 특히 스프링의 빈은 싱글톤 인스턴스이기 때문에 애플리케이션에 하나의 인스턴스만 존재하는데, 만약 스프링 빈에 상태값이 존재한다면 동시성 문제가 발생한다.

최근 동시성 문제에 대해 공부하고 있는데, 백엔드 개발잘면 동시성 문제에 대해 잘 알아야 관점이 넓어지고 성장이 빨라질 것 같다는 생각이 찐하게 든다.