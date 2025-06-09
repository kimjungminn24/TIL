### 대표적인 동기화 문제들

1. BoundedBuffer-problem = Producer-consumer
2. Readers-Writers
3. Dining-Philospphers

### Bounded-Buffer 문제

- 버퍼 : 고정된 크기 n의 공유 공간
- 생산자 : 데이터를 생성하여 버퍼에 넣음
- 소비자 : 버퍼에서 데이터를 꺼내 사용
- 제약 조건
  - 버퍼가 가득차면 생산자는 대기해야함
  - 버퍼가 비어 있으면 소비자는 대기해야함
  - 동시에 여러 스레드가 접근하지 못하도록 동기화 필요

```java
int n; // 버퍼 크기
semaphore mutex =1; // 상호배제를 위한 바이너리 세마포어,바이너리 세마포어 = 뮤텍스
semaphore empty =n; // 빈 공간 수를 나타냄 (초기엔 n)
semaphroe full =0 // 채워진 버퍼 수를 나타냄 (초기엔 0)

//생산자 동작
while (true) {
    // 아이템 생산

    //버퍼에 빈 공간이 있을 때까지 대기
    wait(empty);

    // 임계 영역 진입 (동시에 여러 생산자가 insert하지 않도록 보호)
    wait(mutex);

    // 공유 버퍼에 아이템 삽입

    signal(mutex);    // 뮤텍스반납, 임계 영역 해제
    signal(full);     // 채워진 칸 증가
}

// 소비자 동작
while (true) {
    wait(full);       // full 기다림, 채워진 아이템 있는지 확인
    wait(mutex);      // 임계 영역 진입 (동시에 여러 소비가자 사용하지 않도록 보호)

    // 공유 버퍼에서 아이템 꺼냄

    signal(mutex);    // 임계 영역 해제
    signal(empty);    // 빈 칸 증가

      // 아이템 사용
}
```

### Readers-Writers 문제

- 여러 프로세스가 공유자원에 접근함
- 이 중 일부는 읽기 전용이고 일부는 쓰기 작업을 수행함
- 여러 reader는 동시에 읽어도 문제가 없지만 writer는 혼자만 접근해야함
- 어떻게 reader는 동시에 허용하고 writer는 단독으로 허용할것인가?
  1. reader 우선 : reader는 바로 접근 가능, writer는 모든 reader가 종료될 때까지 대기,reader가 많으면 writer가 굶주릴 수 있음
  2. writer 우선 : writer가 접근하려고 하면 reader들도 대기하게 만듦, writer가 맣으면 reader가 굶주릴 수 있음
  3. 공정성 또는 fifo 접근 : 접근 순서를 공정하게 관리하여 기아 문제 해결

```java
// 공유 자원 접근을 제어하는 세마포어
// writer는 이 세마포어를 사용해 자원을 단독 점유함
// reader는 첫 번째 접근 시 획득하고, 마지막 reader가 나갈 때 해제
semaphore rw_mutex = 1;

// read_count 값을 안전하게 변경하기 위한 상호배제용 세마포어
// 여러 reader들이 동시에 read_count를 수정하지 않도록 보호함
semaphore mutex =1;

// 현재 공유 자원에 접근 중이 reader의 수
// 첫 번째 reader는 rw_mutex를 획득하고, 마지막 reader 가 나갈때 해제
int read_count = 0;

//writer 동작
while (true) {
    wait(rw_mutex);        // 공유 자원을 단독으로 접근하기 위해 잠금
    //  데이터 쓰기 작업 수행
    signal(rw_mutex);      // 자원 사용 후 해제
}

// Reader 프로세스 동작
while (true) {

    wait(mutex);               // read_count에 대한 접근을 보호 (상호배제)
    read_count++;              // 현재 자원에 접근하는 Reader 수 증가
    if (read_count == 1) {
        wait(rw_mutex);        // 첫 번째 Reader가 자원에 접근하기 전 Writer 차단
                               // → Writer가 들어오지 못하게 공유 자원 잠금
    }
    signal(mutex);            // read_count 수정 완료 → mutex 해제

    //  공유 자원 읽기 작업 수행 (다른 Reader와는 동시 접근 가능)

    wait(mutex);              // read_count 감소를 위해 다시 상호배제 진입
    read_count--;             // 자원 접근 중인 Reader 수 감소
    if (read_count == 0) {
        signal(rw_mutex);     // 마지막 Reader가 나가면 Writer에게 자원 개방
    }
    signal(mutex);            // read_count 수정 완료 → mutex 해제
}
```

- writer가 임계 영역을 점유한 상태에서, n명의 reader가 접근을 시도하면
  - 첫번째 reader는 rw_mutex에서 대기중, 나머지 reader들은 mutex에서 대기중
  - 모두 writer가 끝나기를 기다리게 되어 reader 병목 현상 발생 가능함
- 해결법 : 공정성 보장, reader-writer locks
  - reader lock을 획득하면 여러개의 프로세스가 진입할 수 있음
  - write lock 을 획득하면 하나의 프로세스만 진입할 수 있음

## 식사하는 철학자들 문제

- 5명의 철학자가 원형 식탁에 앉아 있음
- 각자 왼손과 오른손에 젓가락이 필요하며, 젓가락은 총 5개
- 철학자들은 다음 상태를 반복함:
  1. 생각함
  2. 배고픔
  3. 먹음
- 먹기 위해서는 양쪽(왼쪽, 오른쪽) 젓가락을 모두 획득해야 함

### 발생할 수 있는 문제

1. Race Condition (경쟁 상태)

- 같은 젓가락을 동시에 잡으려 하면 충돌 발생
- 해결: 젓가락 사용을 임계 구역(Critical Section)으로 설정

2. Deadlock (교착 상태)

- 모두가 동시에 왼쪽 젓가락을 먼저 잡으면
- 오른쪽 젓가락을 얻지 못해 모두 무한 대기 상태
- 서로 락을 점유한 채 대기하면서 데드락 발생

### 해결 방법

1. 철학자 수를 N-1명으로 제한 (ex: 4명만 식사 시도 가능)

   - 항상 하나 이상의 젓가락 여유가 있어 데드락 방지

   ```java
   import java.util.concurrent.Semaphore;

   class Philosopher extends Thread {
       private final int id;
       private final DiningTable1 table;

       public Philosopher(int id, DiningTable3 table) {
           this.id = id;
           this.table = table;
       }

       public void run() {
           try {
               while (true) {
                   System.out.println("Philosopher " + id + " is thinking.");
                   Thread.sleep(100);
                   table.eat(id);
               }
           } catch (InterruptedException e) {
               Thread.currentThread().interrupt();
           }
       }
   }

   class DiningTable1 {
       private final Semaphore[] chopsticks = new Semaphore[5]; // 각 젓가락은 1개만 사용 가능
       private final Semaphore limit = new Semaphore(4); // 최대 4명까지만 식사 시도 허용

       public DiningTable1() {
           for (int i = 0; i < 5; i++) {
               chopsticks[i] = new Semaphore(1); // 젓가락 하나당 세마포어 1개 (동시 사용 금지)
           }
       }

       public void eat(int id) throws InterruptedException {
           limit.acquire(); // 5명 중 4명까지만 식사 시도 → 데드락 방지

           chopsticks[id].acquire();               // 왼쪽 젓가락 획득
           chopsticks[(id + 1) % 5].acquire();     // 오른쪽 젓가락 획득

           System.out.println("Philosopher " + id + " is eating.");
           Thread.sleep(100); // 식사 중

           chopsticks[id].release();               // 왼쪽 젓가락 반납
           chopsticks[(id + 1) % 5].release();     // 오른쪽 젓가락 반납

           limit.release(); // 식사 인원 수 제한 해제 → 다른 철학자에게 기회 부여
       }
   }

   ```

2. 양쪽 젓가락이 모두 가능할 때만 집기

   - 한쪽만 가능하면 다시 생각하기로 돌아감

   ```java
   class DiningTable2 {
       private final Semaphore[] chopsticks = new Semaphore[5]; // 젓가락 5개

       public DiningTable2() {
           for (int i = 0; i < 5; i++) {
               chopsticks[i] = new Semaphore(1); // 각각 1명만 사용할 수 있도록 제한
           }
       }

       public void eat(int id) throws InterruptedException {
           while (true) {
               // 먼저 왼쪽 젓가락을 non-blocking으로 시도
               if (chopsticks[id].tryAcquire()) {
                   // 왼쪽 성공 후, 오른쪽도 시도
                   if (chopsticks[(id + 1) % 5].tryAcquire()) {
                       // 둘 다 성공했을 경우 식사 시작
                       System.out.println("Philosopher " + id + " is eating.");
                       Thread.sleep(100);

                       // 식사 후 젓가락 반환
                       chopsticks[id].release();
                       chopsticks[(id + 1) % 5].release();
                       break; // 식사 성공 → 반복 종료
                   } else {
                       // 오른쪽 젓가락이 실패했으면 왼쪽도 놓고 재시도
                       chopsticks[id].release();
                   }
               }

               // 잠깐 생각하고 다시 시도 (기회를 양보)
               Thread.sleep(50);
           }
       }
   }

   ```

3. 비대칭 전략 사용

   - 홀수 철학자는 왼쪽 → 오른쪽 순
   - 짝수 철학자는 오른쪽 → 왼쪽 순
   - 순환 대기 조건을 깨트려 데드락 방지

   ```java
   class DiningTable3 {
       private final Semaphore[] chopsticks = new Semaphore[5]; // 젓가락 5개

       public DiningTable3() {
           for (int i = 0; i < 5; i++) {
               chopsticks[i] = new Semaphore(1); // 각 젓가락은 1개만 사용 가능
           }
       }

       public void eat(int id) throws InterruptedException {
           int left = id;                 // 왼쪽 젓가락 번호
           int right = (id + 1) % 5;      // 오른쪽 젓가락 번호

           // 철학자 번호가 짝수면 오른쪽 먼저, 홀수면 왼쪽 먼저
           if (id % 2 == 0) {
               chopsticks[right].acquire(); // 오른쪽 젓가락 먼저 획득
               chopsticks[left].acquire();  // 그 다음 왼쪽 획득
           } else {
               chopsticks[left].acquire();  // 왼쪽 젓가락 먼저 획득
               chopsticks[right].acquire(); // 그 다음 오른쪽 획득
           }

           System.out.println("Philosopher " + id + " is eating.");
           Thread.sleep(100); // 식사 중

           chopsticks[left].release();      // 왼쪽 젓가락 반납
           chopsticks[right].release();     // 오른쪽 젓가락 반납
       }
   }

   ```

→ 이 3가지 방법은 데드락은 방지하지만, Starvation는 여전히 가능함

### 기아까지 해결하는 방법

1. 모니터 기반 상태 추적 방식

   - 철학자의 상태를 THINKING, HUNGRY, EATING으로 관리
   - 조건 변수(wait/notify)로 철학자 간의 식사 조건을 제어
   - 공정하게 기회를 부여하며 기아 방지 가능

   ```java
   class DiningMonitor {
       enum State { THINKING, HUNGRY, EATING }

       private final State[] state = new State[5];      // 철학자 상태 배열
       private final Object[] self = new Object[5];     // 각 철학자의 조건 변수

       // 초기 상태 설정
       public DiningMonitor() {
           for (int i = 0; i < 5; i++) {
               state[i] = State.THINKING;
               self[i] = new Object(); // 각 철학자별 개별 대기 객체
           }
       }

       //왼쪽 철학자 번호
       private int left(int i) {
           return (i + 4) % 5;
       }

   		//오른쪽 철학자 번호
       private int right(int i) {
           return (i + 1) % 5;
       }

   	// 철학자가 젓가락을 집는 동작
       public void takeForks(int i) throws InterruptedException {
           synchronized (this) {
               state[i] = State.HUNGRY;   // 배고픔 상태로 전환
               test(i);                   // 먹을 수 있는지 검사

               if (state[i] != State.EATING) {
   		            //양쪽 이웃이 식사 중이라 젓가락을 못잡을우
                   // 자신의 조건 변수 객체에서 대기
                   synchronized (self[i]) {
                       self[i].wait();
                   }
               }
           }
       }

   		//철학자가 젓가락을 내려놓음
       public void putForks(int i) {
           synchronized (this) {
               state[i] = State.THINKING; // 다시 생각하는 상태로 전환
               // 이웃 철학자들이 먹을 수 있는지 검사
               test(left(i));
               test(right(i));
           }
       }

   		//철학자 i가 먹을수 있는지 검사하고, 가능하면 상태 변경 및 깨우기
       private void test(int i) {
           // 자신이 배고프고, 이웃이 먹고 있지 않다면
           if (state[i] == State.HUNGRY &&
               state[left(i)] != State.EATING &&
               state[right(i)] != State.EATING) {

               state[i] = State.EATING; // 식사 시작
               synchronized (self[i]) {
                   self[i].notify();    // 자신을 깨움
               }
           }
       }
   }

   class Philosopher extends Thread {
       private final int id;
       private final DiningMonitor monitor;

       public Philosopher(int id, DiningMonitor monitor) {
           this.id = id;
           this.monitor = monitor;
       }

       public void run() {
           try {
               while (true) {
                   think(); //생각
                   monitor.takeForks(id); //젓가락 집기
                   eat(); // 식사
                   monitor.putForks(id); // 내려놓기
               }
           } catch (InterruptedException e) {
               Thread.currentThread().interrupt();
           }
       }

       private void think() throws InterruptedException {
           System.out.println("Philosopher " + id + " is thinking...");
           Thread.sleep(100);
       }

       private void eat() throws InterruptedException {
           System.out.println("Philosopher " + id + " is eating!");
           Thread.sleep(100);
       }
   }

   public class DiningPhilosophersMonitorTest {
       public static void main(String[] args) {
           DiningMonitor monitor = new DiningMonitor();

           for (int i = 0; i < 5; i++) {
               new Philosopher(i, monitor).start();
           }
       }
   }

   ```

   > 참고 : 주니온님의 인프런 운영체제 공룡책 강의를 참고하였습니다.
