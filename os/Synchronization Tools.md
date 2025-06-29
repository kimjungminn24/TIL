

- [프로세스 동기화](#프로세스-동기화)
- [동기화 문제의 해결책](#동기화-문제의-해결책)
- [뮤텍스와 세마포어](#뮤텍스와-세마포어)
- [모니터](#모니터와-자바-동기화)


## 프로세스 동기화

### 생산자-소비자 문제

- 동시성 환경에서 공유 자원에 대해 생산자와 소비자가 비동기적으로 접근할 때 발생하는 동기화 문제
- 생산자는 데이터를 만들어 버퍼에 넣음
- 소비자는 버퍼에서 데이터를 꺼내 소비함
- 둘은 독립적으로 비동기적으로 동작함 → 생산자는 소비자와 상관없이 생산하고, 소비자도 따로 소비함
- 하지만 공유자원(버퍼)를 동시에 접근하면 문제가 생김 → 서로 작업 순서를 예측할 수 없기 때문에 데이터가 덮어써지거나 중복 소비될 수 있음
- count ++은 단순한 한줄이 아님 → (count 값을 레지스터에 로드, 레지스터에 1을 더함, 레지스터 값을 count에 다시 저장) 3단계 연산을 함 → count ++,count—를 하는 과정에서 데이터 무결성이 깨짐

### 경쟁 조건 (Race Condition)

- 둘 이상의 프로세스(또는 스레드)가 동시에 공유 자원에 접근하여 그 실행 순서에 따라 결과가 달라지는 상황
- 발생 원인: 공유 자원에 동시에 접근/수정
- 해결 방법: 동기화(Synchronization) → 임계 구역(Critical Section)을 정의하고 한 번에 하나의 스레드만 해당 구역에 접근하도록 제한

### 임계 구역 (Critical Section)

- 공유 자원에 접근하는 코드 영역
- 여러 스레드가 동시에 접근하면 경쟁 조건(Race Condition)이 발생하므로 반드시 동시에 하나만 접근하도록 보호해야 함

### 임계 구역 문제 (Critical Section Problem)

- 경쟁 조건을 방지하기 위해 ,"임계 구역에 동시에 하나의 프로세스만 들어가도록 만드는 문제"
- 이 문제를 정상적으로 해결하려면, 3가지 조건(요구사항) 반드시 만족해야 함
- 임계 구역 문제의 세 가지 요구 사항
    - 상호 배제 :  두 개 이상의 프로세스가 동시에 임계 구역에 들어가서는 안됨
    - 진행 : 임계 구역에 진입하려는 프로세스가 없다면, 나머지 대기 중인 프로세스는 불필요하게 기다리지 않고 즉시 진입할 수 있어야 함
    - 한정 대기 : 어떤 프로세스도 임계 구역에 진입하기 위해 무한정 기다려서는 안됨
- 싱글 코어에서는 인터럽트가 발생하지 않도록 막음 → context switching이 일어나지 않도록 막음 → 멀티 프로세서(멀티코어) 에서는 다른 코어에서 동시에 접근할 수 있으므로 상호 배제를 보장하기는 어려움
- Non-preemptive 비선점형 커널
    - 커널이 실행을 시작하면 자기 작업이 끝날 때까지 절대 중단되지 않음→ 커널 코드 실행 중에는 다른 프로세스가 CPU를 빼앗을 수 없음 → 동기화가 쉬움→  성능문제로 최근에는 사용하지 않음
- preemptive 선점형 커널
    - 커널이 실행 중일 때라도 운영체제가 인터럽트나 스케줄링에 의해 현재 실행 중인 테스크를 중단하고 다른 테스크를 전환할 수 있는 구조 → 동기화 기법 필요
- 커널이 실행 중이란 말은 CPU가 커널모드에서 커널 코드를 실행 중이란 뜻임
    - 시스템 호출 발생 (read(),write(),fork()) , 인터럽트 발생, 예외 발생 이런일이 생기면 CPU가 사용자 모드에서 커널모드로 전환해 커널 코드 실행

## 동기화 문제의 해결책

### 소프트웨어 해결책 ( 피터슨 알고리즘)

- 두 개의 프로세스가 임계 구역에 동시에 들어가지 못하도록 하기 위한 상호 배제 알고리즘
- 두 개의 프로세스가 flag배열과 turn 변수를  사용하여 서로 번갈아 가며 임계 구역에 진입하도록 조절함
- load(읽기)와 store(쓰기) 연산의 정확한 순서와 메모리 반영에 의존하는데 현대 CPU와 컴파일러는 이 순서를 보장하지 않기 때문에 정상적인 동기화 결과가 보장되지 않음
- 상호배제,데드락발생x,기아x  보장함

```java
// 공유 변수
bool flag[2] = {false, false};
int turn = 0;

// P0 (프로세스 0)
void process0() {
    while (true) {
        // 진입 의사 표시
        flag[0] = true; //다른 프로세스는 flag[1] = true;
        turn = 1; //다른 프로세스는 turn = 0;

        // 대기: P1이 진입 원하고 지금은 P1 차례면 기다림
        while (flag[1] && turn == 1); //다른 프로세스는 flag[0] && turn==0

        // --- Critical Section ---
        printf("P0: critical section\n");
        // -------------------------

        // 나옴
        flag[0] = false;

        // Remainder Section
        printf("P0: doing other work\n");
    }
}

```

### 하드웨어 해결책

- Atomicity 원자성
    - 한 번에 하나의 단위 연산으로 수행되는 연산
    - 중간에 컨텍스트 스위칭이나 인터럽트가 개입할 수 없음
    - test_and_set : 공유 변수의 값을 읽고 동시에 사용 중으로 설정하는 원자 연산, 락을 구현할 때 사용
    - compare_and_swap : 지정된 메모리 값이 기대한 값과 일치하면 새로운 값으로 교체함
- atomic variables
    - compare_and_swap 명령어는 원자 변수같은 도구를 만들기 위해 사용됨
    - 정수, 불리언 같은 기본 자료형에 대해 원자적인 연산을 수행할 수 있음
    - 경쟁 조건이 발생할 수 있는 상황에서 상호 배제를 보장하기 위해 사용할 수 있음
    - 하나의 변수에 대한 동기화가 필요할 때 유용
    
    ```java
    public class NormalCounter {
        static int count = 0;  // 일반 변수
    
        public static void main(String[] args) throws InterruptedException {
            Thread t1 = new Thread(() -> {
                for (int i = 0; i < 10000; i++) {
                    count++;  //  여러 스레드가 동시에 접근 → 값 꼬임 가능성 있음
                }
            });
    
            Thread t2 = new Thread(() -> {
                for (int i = 0; i < 10000; i++) {
                    count++;  //  마찬가지로 경쟁 조건 발생
                }
            });
    
            t1.start();
            t2.start();
            t1.join();
            t2.join();
    
            System.out.println("최종 count 값: " + count);  
            //  이 값은 20000이 아닐 수 있음 → 경쟁 조건(Race Condition) 발생
        }
    }
    
    import java.util.concurrent.atomic.AtomicInteger;
    
    public class AtomicCounter {
        static AtomicInteger count = new AtomicInteger(0);  //  원자 변수
    
        public static void main(String[] args) throws InterruptedException {
            Thread t1 = new Thread(() -> {
                for (int i = 0; i < 10000; i++) {
                    count.incrementAndGet();  //  원자적으로 증가 → 안전함
                }
            });
    
            Thread t2 = new Thread(() -> {
                for (int i = 0; i < 10000; i++) {
                    count.incrementAndGet();  //  경쟁 없이 동시에 사용 가능
                }
            });
    
            t1.start();
            t2.start();
            t1.join();
            t2.join();
    
            System.out.println("최종 count 값: " + count.get());  
            //  항상 20000 → 동기화 문제 없음
        }
    }
    
    ```

## 뮤텍스와 세마포어

### 뮤텍스

- 임계 구역 진입을 제어하기 위해 사용
- 한 번에 하나의 스레드만 접근 가능 → 상호 배제 보장
- 보통 락을 못얻으면 대기 상태가 됨
- busy wating : 락을 얻을 때 까지 무한 루프를 돌면서 기다림 → CPU를 점유한채 기다림 → 자원낭비
- spin lock : busy waiting 을 활용한 뮤텍스의 일종 , 컨텍스트 스위칭을 하지 않고 락을 얻을 때 까지 루프를 돌면서 기다림
- 스핀락은 멀티코어에서 유용할 수 있음 → 컨텍스트 스위칭 비용  없이 즉시 락을 획득할 수 있음
- 뮤텍스는 락을 획득한 스레드만 락을 해제할 수 있음 → 소유권 개념 존재

### 세마 포어

- 값이 0 이하가 되면 해당 스레드는 대기 상태로 전환됨 (block 상태)
- 이진 세마포어는 값이 0 또는 1만 가지며, 사실상 뮤텍스처럼 동작함
- 세마포어는 소유권 개념이 없음 → 자원을 획득하지 않은 스레드도 signal 호출 가능
- wait과 signal을 통해 조건과 자원 접근을 제어하여 동기화를 보장함
- 세마포어는 소유권 개념이 없음 → 자원을 획득하지 않은 스레드도 signal 호출 가능
- 일반적으로 세마포어는 busy waiting 대신 스레드를 block 상태로 전환하여 CPU 낭비를 방지함

```java
// 세마포어 변수 선언 및 초기화
semaphore S = 3;  // 초기값 3: 동시에 최대 3개 스레드 허용 (예: 자원 3개)

// 세마포어 감소 연산 (P 연산 또는 wait)
P(S) {
    while (S <= 0) {
        // 자원이 없으므로 기다림
        wait();  // 또는 busy waiting
    }
    S -- ;  // 자원 하나 차지
}

// 세마포어 증가 연산 (V 연산 또는 signal)
V(S) {
    S ++;  // 자원 반환
    signal();   // 기다리던 스레드 깨움 (있다면)
}

// 어떤 스레드의 실행 흐름
Thread() {
    P(S);               // 세마포어 획득 → 자원 요청
    // ---- 임계 구역 시작 ----
    access_shared_resource();
    // ---- 임계 구역 끝 ----
    V(S);               // 세마포어 해제 → 자원 반환
}

```

## 모니터와 자바 동기화

### 모니터

- 세마포어는 타이밍 에러가 잘 발생함 →  순서를 안지키거나 시그널과 웨이트를 잘못 쓰면 의도하지 않은 실행 순서 발생 → 이러할 가능성을 줄이자 → 모니터
- 공유자원에 대한 동기화를 안전하게 처리하기 위해 자료(데이터)+메서드(연산)+상호배제(뮤텍스)를 캡슐화한 구조
- ADT(추상 자료형)처럼 동작함 → 데이터는 숨기고 정의된 메서드만 통해서 접근
- 임계 구역 보호는 내부적으로 자동 처리됨
- 내부에 조건 변수를 활용해 wait과 signal 구현
- 조건 변수 : 특정 조건이 만족할 때 까지 대기하게함 (ex ) 큐가 비어있음)

### 자바에서는 모니터락을 제공함

- synchronized : 임계 영역에 해당하는 코드 블록을 선언할 때 사용하는 자바 키워드
- 해당 코드 블록(임계영역)에는 모니터락을 획득해야 진입 가능
- 모니터락을 가진 객체 인스턴스를 지정할 수 있음
- 메소드에 선언하면 메소드 코드 블록 전체가 임계 영역으로 지정됨 → 모니터락을 가진 객체 인스턴스는 this 객체 인스턴스임
- wait() , notify() : java.lang.Object 클래스에 선언됨 : 모든 자바 객체가 가진 메소드임
- 쓰레드가 어떤 객체의 wait() 메서드를 호출하면 해당 객체의 모니터락을 획득하기 위해 대기 상태로 진입함
- notify () 메소드를 호출하면 해당 객체 모니터에 대기중인 쓰레드 하나를 깨움
- notfiyAll() 메소드를 호출하면 해당 객체 모니터에 대기중인 쓰레드 전부를 깨움

```java
class SharedBuffer {
    private int data;
    private boolean hasData = false;

    // 생산자가 호출
    public synchronized void produce(int value) throws InterruptedException {
        while (hasData) {
            wait(); // 소비자가 소비할 때까지 대기 (모니터 락 반납)
        }
        data = value;
        hasData = true;
        System.out.println("Produced: " + value);
        notify(); // 대기 중인 소비자 스레드 하나를 깨움
    }

    // 소비자가 호출
    public synchronized int consume() throws InterruptedException {
        while (!hasData) {
            wait(); // 생산자가 생산할 때까지 대기 (모니터 락 반납)
        }
        hasData = false;
        System.out.println("Consumed: " + data);
        notify(); // 대기 중인 생산자 스레드 하나를 깨움
        return data;
    }
}

```

> 참고 : 주니온님의 인프런 운영체제 공룡책 강의를 참고하였습니다.

