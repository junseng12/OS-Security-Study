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

![image](https://github.com/user-attachments/assets/cc228596-3384-4be8-b308-60b962d8ad59)

![image](https://github.com/user-attachments/assets/5f7cb42b-44b2-4a12-8a5e-7293996d98da)

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

**Critical Section:** 공유 자원에 동시에 접근하면 문제가 발생할 수 있는 코드 영역

**필수 조건**

- Mutual Exclusion (상호 배제)
- Progress (진행 보장)
- Bounded Waiting (한정된 대기)

---

### 📐 Peterson’s Solution

두 개의 프로세스 간 Critical Section 접근 제어를 위한 소프트웨어 기반 해법

```c
flag[i] = true;
turn = j;
while (flag[j] && turn == j);
// critical section
flag[i] = false;
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
