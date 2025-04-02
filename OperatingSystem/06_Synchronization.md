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

> 예시: Producer-Consumer 문제  
> 생산자와 소비자가 공유 버퍼를 사용할 때, 동기화 없이 동시에 접근하면 버퍼 오염, 데이터 유실 등의 문제가 발생할 수 있다.  
> ![Producer-Consumer 문제 예시](./images/producer-consumer.png)

---

### 🚧 Critical-Section Problem

**Critical Section** : **공유 자원(변수, table, file 등)**에 동시에 접근하면 문제가 발생할 수 있는 코드 영역

**필수 조건**git add

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

- Algorithm for Process Pi(Pj인 경우 i와 j를 뒤바꾸면 됨

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

> > - Peterson’s Solution 문제점(CPU나 compiler 개입 시 정상 동작 불가능)
> >   일반적인 경우, 제대로 동작하나 CPU나 compiloptimization을 위해 Peterson’s Solution program code의 수행 순서 바꿀 수 있음
> >   (이때, multithread는 제대로 동작 X - race condition 발생)

---

### ⚙️ 하드웨어 기반 동기화

#### Disable Interrupts

단일 프로세서 환경에서 인터럽트를 비활성화하여 독점 실행 보장

##### 그 밖의 세 가지 형태 hardware support

- A. Memory Barriers
  Weakly ordered 메모리 모델에서 메모리 접근 순서를 강제 (Weakly ordered의 일종)
  memory barrier를 선언해놓는 그 순간까지는 업데이트를 지연시켜도 된다는 것
- B. Hardware instructions
- C. Atomic variables

> > Memory model : 컴퓨터 구조가 AP을 실행할 수 있도록 보장하는 메모리
> >
> > - Memory models 두 가지 존재(Multiprocessor, computer 환경에서 데이터 공유 시, 쓰는 전략)
> >   ➢ Strongly ordered
> >   한 CPU가 Write하면, 다른 CPU가 즉시 확인할 수 있어야 함 (간단하고 문제 없겠지만, 성능에 악영향을 미침)
> >   ➢ Weakly ordered
> >   한 CPU가 Write해도, 즉시 전파할 필요가 없음(문제가 생기지 않을 때까지 전파 X)
> >   ex. 어떤 data를 업데이트 했는데, 다른 CPU나 컴퓨터에서 이를 사용하지 않을 경우

#### B. Hardware instructions 코드 예제

하드웨어 명령어(Test-and-Set, Compare-and-Swap 등)는 인터럽트 없이 Instruction Cycle 동안 원자적으로 실행되며, 이는 수행 후 인터럽트가 체크되는 구조와 복합 동작을 단일 명령어로 제공하는 하드웨어 기능 덕분이다. 이를 통해 Critical Section을 보호할 수 있다.

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

#### Atomic Variables 코드 예제

어떤 값에 대해서 변화를 시키는 동작이 atomic하게 이루어지는 변수

- Increment

```c
void increment(atomic_int *v)
{
	int temp;
	do {
		temp = *v;
	}
	//v값을 temp에 넣고 v == temp면, temp를 하나 증가시켜 v에다 넣음
	(v == temp가 아니면 그 사이에 누군가가 끼여 값에 변화를 일으켰다는 것)
	while (temp != (compare_and_swap(v,temp,temp+1));
}
```

---

### 🔒 Mutex Locks

- acquire(), release() 함수를 통해 상호 배제 구현
- busy waiting (spin lock) 발생 가능

---

### 🔑 Semaphores

- 자원 수를 정수 값으로 관리
- wait(), signal() 연산 사용
- Binary Semaphore와 Counting Semaphore로 구분

**Busy-waiting vs No-busy-waiting**

- Busy Waiting: CPU 계속 사용, 짧은 대기 시 효율적
- No-Busy-Waiting: 대기 큐 사용, block()/wakeup()로 효율적 자원 관리

- Wait, Signal 코드 예제

```c
wait(semaphore *S) {
    S->value--;
    if (S->value < 0) { // 자원 없으면 대기 리스트 추가
        add this process to S->list;
        block();
    }
}

signal(semaphore *S) {
    S->value++;
    if (S->value <= 0) { // 대기 프로세스 있으면 깨움
        remove a process P from S->list;
        wakeup(P);
    }
}
```

**문제**: Deadlock, Priority Inversion 가능

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
