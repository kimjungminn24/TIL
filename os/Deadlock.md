- 데드락 (교착상태)
- 여러 스레드가 서로가 점유하고 있는 자원을 기다리면서 무한 대기 상태에 빠지는 현상

### 데드락 네가지 조건

1. 상호 배제 : 자원은 한번에 오직 하나의 프로세스(스레드)만 사용할 수 있음
2. 점유 대기 :  프로세스가 자원을 점유한 상태에서 다른 자원을 추가로 요청하며 대기하는 상태
3. 비선점 : 다른 프로세스가 점유 중인 자원을 강제로 뺏을 수 없음
4. 순환 대기 : 서로 자원을 기다리는 프로세스들이 원형으로 대기하고 있는 상황

### Resource-Allocation Graph (RAG)

- 프로세스와 자원의 관계를 그래프로 시각화한 것
- 데드락 발생 가능성을 분석하기 위해 사용함
- 정점 : 프로세스 노드(P1,P2) , 자원 노드 (R1,R2)
- 간선 : 요청 간선(프로세스가 자원을 요청하고 있는 상태) , 할당 간선(자원이 프로세스에 할당된 상태)
- 사이클이 발생하면 데드락 가능성이 높아짐

```markdown
[P1] ──▶ [R1] ──▶ [P2] ──▶ [R2] ──▶ [P1]
 ↑                                  ↓
 └───────────── 사이클 ─────────────┘

P1 -> R1 : P1이 자원 R1요청중
R1 -> P2 : R1은 P2에게 할당됨 
P1, P2가 서로의 자원을 기다리는 상태이므로 데드락 발생 
// 자원에 여러 인스턴스가 존재하면 데드락이 아닐 수 있음
```

### 데드락 문제에 대한 3가지 대처법

1. 무시 (Ignore) → 데드락은 드물게 발생하니 신경 쓰지 않고 운영함
2. 예방 또는 회피 (Prevention / Avoidance) → 데드락의 발생 조건 중 하나 이상을 제거하거나, 데드락이 발생할 가능성이 있는 위험 상태를 피함
3. 회복 (Recovery) → 데드락이 이미 발생한 후, 이를 탐지하고 복구함

### 데드락 예방 (prevention)

- 데드락이 일어날 수 있는 4가지 조건 중 하나를 제거해서 데드락 자체가 일어나지 않도록 사전에 차단
- 상호 배제 : 모든 자원을 공유 가능하게 만들자 → 하지만 대부분의 자원은 공유 불가능 ( 현실적으로 제거 불가능)
- 점유대기: 프로세스가 자원을 점유한 상태에서 다른 자원을 추가 요청하지 못하게함→ 시작 전 필요한 모든 자원을 한번에 요청하게 강제함 → 자원이 비효율적으로 낭비될 수 있음
- 비선점 : 자원이 점유 중이라도 다른 프로세스가 강제로 빼앗을 수 있게함 → 낮은 우선순위 프로세스가 점유 중이면 선점하여 높은 우선순위 프로세스에 할당 → 복잡한 상태 롤백이 필요할 수 있음
- 순환 대기  : 자원에 일정한 순서를 부여하고 프로세스가 자원을 항상 순서대로만 요청하게 제한 → 가장 현실적

```java
void transaction(Account from, Account to, double amount) {
    mutex lock1 = get_lock(from);
    mutex lock2 = get_lock(to);

    //  순서를 고려하지 않고 락을 획득
    acquire(lock1);
    acquire(lock2);

    withdraw(from, amount);
    deposit(to, amount);

    release(lock2);
    release(lock1);
}
// 만약 쓰레드 A: transaction(A, B), 쓰레드 B: transaction(B, A)를 동시에 실행하면
// A는 lock(A) → lock(B), B는 lock(B) → lock(A) 요청
// 서로가 상대 락을 기다리며 데드락 발생 가능

    // 항상 같은 순서로 락을 획득하도록 정렬
    if (from.id < to.id) {
        lock1 = get_lock(from);  // id가 더 작은 쪽 먼저
        lock2 = get_lock(to);
    } else {
        lock1 = get_lock(to);
        lock2 = get_lock(from);
    }
// 현실적 한계 -> 락 순서 규칙을 다른 코드에서도 항상 지켜야함
// 자원의 수가 많거나 동적으로 결정되면 정렬이 복잡해짐

```

### 데드락 회피 (avoidance)

- 데드락이 생길 수 있는 상황이라면 아예 자원 할당을 하지않음
- 자원이 요청되었을 때 미래에 데드락이 발생할 가능성이 있는지 미리 계산함
- 만약 위험한 상태가 된다면 자원 할당을 거부하고 기다리게 만듬
- 이를 위해 각 프로세스가 최대 얼만큼 자원을 요청할 수 있는지 미리 알아야함
    - 각 프로세스가 필요로 하는 할 수 있는 최대 자원 수
    - 각 프로세스가 현재 할당 받은 자원 수
    - 시스템 전체의 사용 가능한 자원 수
- 안전한 상태 (safe state) : 지금 자원을 요청한 프로세스에게 줄 수 있고 나머지 프로세스들도 순서만 잘 정하면 모두 자원을 받아 끝낼 수 있는 상태 → 데드락을 피할 수 있는 방법 → 항상 안전한 상태에서 머물자
- 자원 요청 시점에만 작동함 → 요청마다 돌려야하므로 연산 비용이 큼

### 은행원 알고리즘

- 데드락 회피 대표 알고리즘
- 은행원이 모든 고객에게 대출을 해줄 때, 미래의 모든 요청도 감당할 수 있는지 판단하고 자원을 할당하는 방식

- Available : 현재 사용 가능한 자원 수
- Max : 각 프로세스가 최대로 필요로 하는 자원 수
- Allocation : 각 프로세스가 현재 할당 받은 자원 수
- Need = Max - Allocation : 앞으로 각 프로세스가 더 필요로 하는 자원 수
- 동작 순서
    - 프로세스가 자원 요청(Request)
        
        → 요청 수 ≤ Need 이고, 요청 수 ≤ Available 인지 확인
        
    - 일단 자원을 준 것처럼 시뮬레이션
        - Available -= Request
        - Allocation += Request
        - Need -= Request
    - Safety Algorithm 수행 → 위 시뮬레이션 결과가 Safe State인지 확인
    - Safe라면 요청 승인, Unsafe라면 거절 및 롤백

### 데드락 예방과 회복

- 데드락이 발생한 후 감지하고 복구
- 복구 - 프로세스 중단, 자원 선점, 전체 재시작
