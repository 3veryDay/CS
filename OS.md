# 운영체제(OS) 핵심 요약

## 목차
1. [TLB](#1-tlb-translation-look-aside-buffer)
2. [Thrashing](#2-thrashing-스레싱)
3. [불연속 할당 (Memory Allocation)](#3-불연속-할당)
4. [프로세스와 컴파일 과정](#4-프로세스와-컴파일-과정)
5. [PCB와 스레드](#5-pcb-process-control-block)
6. [프로세스 상태](#6-프로세스-상태)
7. [IPC (Inter Process Communication)](#7-ipc)

---

## 1. TLB (Translation Look-aside Buffer)
> **캐시의 일종**

- `Hierarchy page table`을 사용하여 VA(Virtual Address)를 PA(Physical Address)로 변환하는 과정은 시간이 많이 소요됨.
- **TLB**에 VPN(Virtual Page Number)이 존재하면, Page Table을 거치지 않고 바로 물리 주소 참조가 가능하여 속도가 향상됨.

## 2. Thrashing (스레싱)
OS가 Page Fault를 처리하느라 CPU를 소모하는 상태 (실제 작업보다 페이징에 시간 낭비).

### 원인
- 과도한 멀티태스킹
- **Working Set 크기 초과** (실행 중인 프로세스의 작업 집합이 실제 물리 메모리보다 클 때)
- 메모리 부족

### 문제점
- 시스템 응답 속도 저하
- 전체 처리량(Throughput) 급감

### 해결방법
- 메모리 관리 정책 변경 (LRU, FIFO, LFU 등)
- Working Set 크기 조정
- Process 우선순위 조정 (일부 프로세스 중단)

## 3. 불연속 할당
메모리를 연속적으로 할당하지 않고, 쪼개어 서로 다른 공간에 할당하는 방식.

### 3-1. Paging (페이징)
동일한 크기의 `Page`로 나누어 메모리에 분산 할당하는 방식.
- **장점:** 외부 단편화(External Fragmentation) 없음.
- **단점:** 주소 변환이 복잡하고 잦음. 내부 단편화(Internal Fragmentation) 발생 가능.

### 3-2. Segmentation (세그멘테이션)
`Segment`라는 **의미 단위**로 나누는 방식.
- Code, Data, Stack, Heap 등이 세그먼트 단위가 됨.
- 내부적으로 더 작은 세그먼트(함수 단위 등)로 쪼개기 가능.
- **Segment Table**을 사용하여 관리.
- **장점:** 내부 단편화 발생하지 않음.
- **단점:** 외부 단편화 문제 발생 가능.

### 3-3. Paged Segmentation
- 세그멘테이션을 기본으로 하되, 세그먼트를 다시 페이징하여 관리하는 방식.

## 4. 프로세스와 컴파일 과정

1. **전처리 (Preprocessing)**
   - `#include`와 같은 매크로 처리, 주석 제거
   - \```bash
     gcc -E hello.c -o hello.i
     \```
   - 결과: `.i` 파일

2. **컴파일 (Compilation)**
   - C언어를 어셈블리어로 변환, 문법 오류 검사
   - \```bash
     gcc -S hello.i -o hello.s
     \```
   - 결과: `.s` 파일

3. **어셈블리 (Assembly)**
   - 어셈블리 코드를 기계어(Machine Code)로 변환
   - \```bash
     gcc -c hello.s -o hello.o
     \```
   - 결과: `.o` (Object File)

4. **링킹 (Linking)**
   - 여러 목적 파일(.o)과 라이브러리를 합쳐 실행 파일 생성
   - **정적 라이브러리:** 실행 파일에 포함됨.
   - **동적 라이브러리:** 실행 시 DLL 등을 통해 참조.

## 5. PCB (Process Control Block)

**실행 예시 (오버워치 실행)**
1. HDD의 실행 파일이 RAM의 `사용자 영역`으로 로드됨 (Code, Data, Stack, Heap 할당).
2. PC(Program Counter), Register State, 우선순위 등을 포함한 **PCB**가 생성되어 커널 영역의 `Process Table`에 등록됨.
3. PCB의 PC 정보를 바탕으로 실행 시작.
4. **Context Switching** 발생 시:
   - 현재 상태를 기존 PCB에 저장.
   - 다음 실행할 프로세스(예: 디스코드)의 PCB 정보를 CPU로 로드.

### Linux에서의 Thread 관리
리눅스에서는 스레드도 하나의 프로세스처럼 관리하며, PCB들이 Linked List로 연결됨.
- **공유 자원:** Code, Data, Heap
- **개별 자원:** Stack, Register
- **Thread Group:** 같은 프로세스 내 스레드들은 `TGID`(Thread Group ID)로 묶임.
  - 개별 스레드는 고유한 `PID`를 가짐.
  - `top` 명령어는 TGID를 기준으로 대표 PID만 보여줌.

## 6. 프로세스 상태
<img width="1042" alt="image" src="https://github.com/user-attachments/assets/79479700-7bcd-4632-9654-527e7222edba" />

### Dispatch
`Ready Queue`에 있는 프로세스가 `Running` 상태로 전환되는 순간.
1. **Scheduler:** "너 실행해!" (선택)
2. **Dispatcher:** CPU 제어권 넘겨줌 (Context Switching 수행)

## 7. IPC (Inter Process Communication)

### 1. 공유 메모리 (Shared Memory)
- `MappedByteBuffer` 등 사용.
- 디스크 파일을 RAM에 직접 매핑(`mmap`).
- **Zero-copy:** 커널을 거치지 않아 매우 빠름.
- **단점:** 동기화 문제(Race Condition) 발생 가능.
\```java
FileChannel channel = FileChannel.open(Paths.get("ipc.dat"), ...);
MappedByteBuffer buffer = channel.map(FileChannel.MapMode.READ_WRITE, 0, 1024);
buffer.putInt(100); // 즉시 공유됨
\```

### 2. 메시지 전달 (Message Passing)
커널을 통해 데이터를 송수신하는 방식.

#### 2-1. Pipe
- 단방향 통신 (FIFO).
- **자바 (Thread 간):** `PipedInputStream` / `PipedOutputStream` (connect 필요).
- **자바 (Process 간):** `System.in` / `System.out` (Standard IO).
- **리눅스:** `|` 명령어와 동일.
\```java
Process p = new ProcessBuilder("cat").start();
OutputStream pipeToChild = p.getOutputStream(); // Write
InputStream pipeFromChild = p.getInputStream(); // Read
\```

#### 2-2. Signal
- 비동기적 이벤트 알림 (예: `SIGINT`, `SIGKILL`).
- 자바: `Shutdown Hook`을 통해 리소스 정리 가능.

#### 2-3. Socket
- 네트워크를 통한 양방향 통신 (IP, Port).
- **TCP:** `Socket`, `ServerSocket`.
- **UDP:** `DatagramSocket` (비연결형, 빠름, 스트리밍).
- **Unix Domain Socket:** 같은 머신 내 통신 시 파일 시스템 경로 사용 (가장 빠름).

#### 2-4. RPC (Remote Procedure Call)
- 다른 컴퓨터의 함수를 내 것처럼 호출.
- **RMI (Java):** 자바 간 통신.
- **gRPC (Google):** Protocol Buffers 사용, 고성능, 마이크로서비스 표준.
- *참고: RPC의 개념이 웹으로 확장된 것이 REST API.*
- |RPC | REST API|
  |---|------|
  |행위 중심|명사(resource) 중심|
  |/doSomething|/something|
  |binary, 빠름|json, 읽기 쉽지만, 무거움|
  |캐싱이 어려움|캐싱이 쉬움|
  |결합도 높음|결합도 높음|
  |내부 서버 간 통신(MSA 구조)|외부 공개 API, FE와 BE 연동|


