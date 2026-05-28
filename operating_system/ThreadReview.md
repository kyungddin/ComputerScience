## 운영체제 스레드 개념 정리

### 스레드란?

**프로세스** 내에서 실제로 실행되는 흐름의 단위. 하나의 프로세스는 최소 1개의 스레드를 가짐.

| | 프로세스 | 스레드 |
|--|--|--|
| 메모리 | 독립된 주소 공간 | 프로세스 내 메모리 공유 |
| 생성 비용 | 높음 | 낮음 |
| 통신 | IPC 필요 | 공유 메모리로 직접 통신 |
| 독립성 | 강함 | 약함 (하나가 죽으면 전체 영향) |

---

### 스레드가 공유하는 것 vs 독립적인 것

**공유 (같은 프로세스 내)**
- 코드(Code) 영역
- 데이터(Data) 영역 — 전역변수, static 변수
- 힙(Heap) 영역

**독립 (스레드마다 따로 가짐)**
- **스택(Stack)** — 지역변수, 함수 호출 정보
- **레지스터** — PC(Program Counter), SP 등
- **스레드 ID**

---

### 핵심 동기화 개념

```
Critical Section (임계 구역)
  └── 공유 자원에 접근하는 코드 구간

해결 조건 3가지:
  1. Mutual Exclusion (상호 배제) — 한 번에 하나만
  2. Progress (진행) — 누군가는 들어갈 수 있어야 함
  3. Bounded Waiting (한정 대기) — 무한 기다림 없어야 함

주요 도구:
  - Mutex (뮤텍스) — lock/unlock, 소유권 있음
  - Semaphore (세마포어) — 카운팅, 소유권 없음
  - Monitor — 고수준 동기화 (Java synchronized 등)
```

---

### 사용자 수준 vs 커널 수준 스레드

- **사용자 수준 스레드** — 라이브러리가 관리, 커널은 모름 → 하나가 블로킹되면 전체 블로킹
- **커널 수준 스레드** — OS가 직접 관리 → 진정한 병렬 실행 가능, 생성 비용 있음
- **혼합형 (M:N)** — 현대 OS 대부분 채택

---

### 동기 & 블로킹

```markdown
동기  + 블로킹   → 가장 일반적 (순서대로 실행, 기다림)
동기  + 논블로킹 → 폴링 방식 (기다리지 않고 완료 여부를 계속 확인)
비동기 + 블로킹  → 거의 안 씀 (효과가 없음)
비동기 + 논블로킹 → Node.js, I/O 이벤트 루프 방식
```

---

### std::thread

```cpp
#include <thread>
#include <iostream>

void task(int id) {
    std::cout << "스레드 " << id << " 실행 중\n";
}

int main() {
    std::thread t1(task, 1);
    std::thread t2(task, 2);

    t1.join();  // 메인이 t1 끝날 때까지 대기
    t2.join();

    return 0;
}
```

### std::mutex (동기화)

```cpp
#include <thread>
#include <mutex>

int count = 0;
std::mutex mtx;

void increment() {
    for (int i = 0; i < 1000; i++) {
        std::lock_guard<std::mutex> lock(mtx);  // 스코프 벗어나면 자동 해제
        count++;
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);
    t1.join();
    t2.join();
    // count == 2000 보장
}
```