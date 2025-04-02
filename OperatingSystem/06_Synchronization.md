# 📘 운영체제 6장 – Synchronization Tools

> 프로세스, CPU, Core 간의 동기화 문제를 다루는 장.

---

## 🗂️ 목차

- [학습 목표](#학습-목표)
- [개념 요약](#개념-요약)
  - [Background](#background)
  - [Critical-Section Problem](#critical-section-problem)
  - [Peterson’s Solution](#petersons-solution)
  - [하드웨어 기반 동기화](#하드웨어-기반-동기화)
  - [Mutex Locks](#mutex-locks)
  - [Semaphores](#semaphores)
  - [Monitors](#monitors)
  - [Liveness 문제](#liveness-문제)

---

## 🚀 학습 목표

- Critical Section 문제 설명 및 Race Condition 이해
- 하드웨어 기반 동기화 방법 (Memory Barriers, CAS, Atomic Variables)
- Mutex Lock, Semaphore, Monitor를 이용한 Critical Section 해결법
- 다양한 상황 (low, moderate, high contention)에서의 도구 평가

---

## 📌 개념 요약

### 🔹 Background

**동기화 (Synchronization)**  
여러 프로세스나 쓰레드가 동시에 실행되면 성능이 향상되고 모듈화가 쉬워지지만, **동시에 접근하면 데이터 일관성(Data Inconsistency) 문제가 발생**한다. 따라서 동기화는 필수적이다.

**Race Condition**  
여러 프로세스가 동일 데이터에 접근할 때, 수행 순서에 따라 결과가 달라지는 문제.

![image](https://github.com/user-attachments/assets/f2ab49e4-9598-4391-9e05-7bb3eed78d00)


> 예시: Producer-Consumer 문제  
> 생산자와 소비자가 공유 버퍼를 사용할 때, 동기화 없이 동시에 접근하면 버퍼 오염, 데이터 유실 등의 문제가 발생할 수 있다.

### Producer
```c
while (true) {
/* produce an item in next produced */ 
while (counter == BUFFER_SIZE) ; /* do nothing */ 
buffer[in] = next_produced; 
in = (in + 1) % BUFFER_SIZE; 
counter++; 
}
```

### Consumer
```c
while (true) {
while (counter == 0) ; /* do nothing */ 
next_consumed = buffer[out]; 
out = (out + 1) % BUFFER_SIZE; 
counter--; /* consume the item in next consumed */ 
}
```

---

### 🚧 Critical-Section Problem

**Critical Section** : **공유 자원(변수, table, file 등)**에 동시에 접근하면 문제가 발생할 수 있는 코드 영역

**필수 조건**

- ① Mutual Exclusion (상호 배제)
  독점권 행사(부여)
- ② Progress (진행 보장)
  아무도 그 내부에 존재하지 않을 때, 들어갈 수 있어야 함
- ③ Bounded Waiting (한정된 대기)
  Critical-Section에 누가 있는 경우 기다리지만, 그 시간에 한정이 있어야 함

> > 커널의 preemptive, non-preemptive 두 가지 접근법
> > **Preemptive** : 어느 proc가 CPU를 차지하여 동작하더라도 더 우선순위 높은 것이 그 CPU 사용권 빼앗을 수 있음
> > **Non-preemptive** : 어느 proc가 CPU 차지하여 동작 시에 그 CPU 할당 종료(커널 모드를 빠져나갈 때, block 상태일 때, 스스로 CPU 양보할 때)까지 다른 proc에서 사용권 가져올 수 없음

---

### 📐 Peterson’s Solution

두 개의 프로세스 간 Critical Section 접근 제어를 위한 소프트웨어 기반 해법

```c
flag[i] = true;
//turn = i라 설정하면, 나중에 수행되는 j가 turn값을 overwrite하면서 j가 늦게 수행되었음에도 j가 사용할 차례가 된다는 것은 불공정(상대방 차례임을 명시하는 것이 공정)
turn = j;
// i의 입장에서 j보다 한 발 늦은 것
//j의 flag가 false되거나 i turn이 되는 것을 무한 대기
// 이 조건에서 벗어났다는 것은 i가 critical section에 들어가도 된다는 것
while (flag[j] && turn == j);


/* critical section */
//i process가 critical section 내 활동을 모두 수행한 상태
flag[i] = false;
/* remainder section */
```

---

### ⚙️ 하드웨어 기반 동기화

#### Disable Interrupts

단일 프로세서 환경에서 인터럽트를 비활성화하여 독점 실행 보장

#### Memory Barriers

Weakly ordered 메모리 모델에서 메모리 접근 순서를 강제

#### Atomic Instructions

- Test-and-Set

```c
boolean test_and_set(boolean *target) {
    boolean rv = *target;
    *target = true;
    return rv;
}
```

- Compare-and-Swap

```c
int compare_and_swap(int *value, int expected, int new_value) {
    int temp = *value;
    if (*value == expected)
        *value = new_value;
    return temp;
}
```

---

### 🔒 Mutex Locks

- acquire(), release() 함수를 통해 상호 배제 구현
- busy waiting (spin lock) 발생 가능

---

### 🔑 Semaphores

- 자원 수를 정수 값으로 관리
- wait(), signal() 연산
- Binary Semaphore와 Counting Semaphore로 구분

**Busy-waiting vs No-busy-waiting**

- Busy: 빠르지만 CPU 낭비
- No-busy: 효율적인 자원 관리

---

### 🖥️ Monitors

- 고급 언어 수준에서의 동기화 기법
- 공유 데이터와 동기화 메서드를 하나의 추상적 데이터 타입으로 캡슐화

---

### ♻️ Liveness 문제

- Deadlock: 자원 순환 대기
- Starvation: 자원 독점으로 인한 무한 대기
- Priority Inversion: 낮은 우선순위 작업이 자원을 점유하여 높은 우선순위 작업이 대기

**해결:** Priority Inheritance Protocol
