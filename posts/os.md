# 교재 정리

교재: Operating Systems Principles & Practice, Thomas Anderson & Michael Dahlin, 2nd Ed. Recursive Books, 2014.

### _계속 정리 중..._

# I Kernels and Processes
## 1 Introduction
<p align="center">
    <img src="/assets/os/1/1-1.png" width=100% height=100% text-align=center />
</p>
간단한 웹 서버에 운영체제가 관여하는 바는 깊다. 
- 웹서버가 받은 쿼리에 응답하는 어플리케이션과의 통신
- 두 유저가 동시에 요청했을 때의 처리
- 성능을 위해 응답의 복제본을 캐싱할 때, 어떻게 동시에 들어오는 여러 요청들에 일관적인 캐시 데이터를 보여줄 수 있을지
- 커스텀을 위해 웹서버가 클라이언트에 실행 가능한 스크립트를 보내줄 때, 어떻게 클라이언트 입장에서 스크립트를 안전하게 실행할지
- 웹페이지가 저장된 디스크에서 웹서버가 어떻게 이 파일의 위치를 특정해서 읽어올 수 있는지
- 하드웨어를 변경했을 때, 웹서버 코드가 매 번 하드웨어에 맞춰 재작성 되어야 할 지

### 1.1 What Is An Operating System?
<p align="center">
    <img src="/assets/os/1/1-3.png" width=100% height=100% text-align=center />
</p>
운영체제는 유저와 하드웨어 사이에 위치해서 하드웨어를 제어하는 방식을 유저가 사용하기 편리하고 안전하게 노출하는 계층이다.  
하드웨어는 atomic instruction, privilege level, MMU 같은 primitive를 제공하고, 운영체제는 이를 이용해 동기화와 fault isolation을 구현한다.

보다시피 운영체제 계층은 크게 두 개로 나뉘는데, 하드웨어 추상화 계층(Hardware Abstraction Layer)과 커널-유저 인터페이스로 나뉜다. 
- 하드웨어 추상화 계층: ARM, AMD, Intel 등 다양한 하드웨어에서 노출하는 API를 감싸서 파일시스템, 네트워킹, 스레드 관련 함수가 하드웨어는 신경쓰지 않고 HAL에서 노출하는 API를 호출하기만 하면 되도록 하드웨어를 추상화해준다. 
- 커널-유저 인터페이스: 유저 프로그램이 사용하는 기능들이 있다. 시스템 라이브러리가 해당 기능을 호출하고, 유저 프로그램은 시스템 라이브러리의 API를 호출함으로써 커널 영역을 건드릴 수 있게 된다. 

#### 1.1.1 Resource Sharing: Operating System as Referee  
운영체제는 단일 CPU로도 여러 프로그램을 동시에 돌아가게 해준다. 정확히는, 동시에 돌아가는 것처럼 보이게 한다. 이를 위해서 몇 가지를 수행한다.
- 자원 할당: 여러 프로그램을 동시에 돌리기 위해서 한정된 자원을 효율적으로, 그리고 안전하게(악의적인 프로그램이 다른 프로그램에 영향을 미치지 못하게) 할당해줘야 한다.
- 소통: 만약 여러 프로그램이 특정 목적을 위해 서로 소통해야 한다면, 그것도 가능하게 해 줘야 한다. 

#### 1.1.2 Masking Limitations: Operating System as Illusionist  
**가상화**를 통해 한정된 자원을 최대한 이용한다. 각 프로세스는 메모리, CPU 등 자신이 모든 하드웨어를 완전히 점유하고 있다고 생각함.   
CPU, 메모리뿐만 아니라 소프트웨어적 도움을 통해 거의 대부분의 하드웨어는 가상화될 수 있다. 
- 물리적 네트워크는 패킷 손실이 가능하지만 패킷 손실이 없음을 보장하는 가상화된 네트워크를 만들 수 있다(TCP).
- 메모리 및 디스크는 각 하드웨어 특성에 따라 제각각의 단위와 방법으로 블럭을 읽고 출력하는데, 이를 다루기 용이한 byte-addressable 한 형태로 래핑해줄 수 있다.
- 동일한 ISA 내에서 ABI를 통일함으로써, 하드웨어 내부 차이를 숨긴다. 덕분에 어플리케이션을 변경하지 않고 다양한 기기에서 돌아가게 할 수 있다. 애뮬레이션을 사용한다면 ARM, x86 같은 CPU 아키텍처도 가상화할 수 있다.

위 모든 특성을 조합하면, **컴퓨터 그 자체를 가상화**할 수도 있다. OS(Guest OS)를 어플리케이션으로써 다른 OS(Host OS)위에서 돌아가게 만들 수 있다. 이를 **가상머신(Virtual Machine)**이라 부른다. Guest OS는 일반적인 운영체제처럼 자원을 할당하고 프로세스들을 스케줄링하지만, 이 OS가 사용하는 CPU, RAM, 디스크 등의 자원들은 모두 실제 물리적 자원이 아니고, Host OS에 의해 가상화된 자원들이다. 

#### 1.1.3 Providing Common Services: Operating System as Glue
아랫단의 복잡성을 가리고 프로그램이 사용할 API만 노출해 줘서, 다양한 하드웨어 위에서도 어플리케이션의 코드를 변경할 필요없게 해준다. 하드웨어 추상화 계층이 이 역할을 함.

#### 1.1.4 Operating System Design Patterns
Referee, Illusionist, Glue로 나뉘어지는 역할은 비단 OS 디자인에만 비유되지 않고, 다양한 도메인에 적용될 수 있다.

<p align="center">
    <img src="/assets/os/1/1-6.png" width=100% height=100% text-align=center />
</p>
대표적인 예로 브라우저를 들 수 있다. 브라우저는 하나의 어플리케이션이지만, 이 또한 referee, illusionist, glue를 고려하여 설계되었다. 브라우저는 여러 탭을 열어둘 수 있으며, 본질적으로 스크립트를 동적으로 실행함으로써 실행되기 때문이다.
- Referee: 여러 탭들 중 유저가 머물러 있는 탭에 어떻게 즉각적인 반응을 줘야한다. 악의적인 스크립트가 실행되면서 브라우저에 악영향을 끼치는 것을 막아야 한다(Sandboxing).
- Illusionist: 분산된 웹 서버 환경에서 브라우저와 소통하던 웹 서버가 죽었을 때, 정상 작동하는 다른 웹 서버로 연결을 복구해야 한다. 그래서 유저는 기존 웹 서버가 죽은지 모르게 한다.
- Glue: OS와 하드웨어 플랫폼에 상관없이 주어진 스크립트를 일관적으로 실행해서 유저에게 보여줘야 한다.

<p align="center">
    <img src="/assets/os/1/1-7.png" width=100% height=100% text-align=center />
</p>

또한 DBMS가 있다. OS와 DBMS는 닮은점이 많다고 생각이 드는데, 디스크에 정보를 기록하는 형식이 거의 비슷하기 때문이다.


- Referee: OS와 다르게 DBMS는 여러 유저가 동시에 사용할 수 있기에 이쪽의 동시성 이슈와 보안도 고려해야 한다. 
- Illusionist: 명령을 수행하는 중에 하드웨어가 고장나더라도, **유저가 관찰하는 상태가 일관되도록** 해야한다.


### 1.2 Operating System Evaluation
운영체제가 잘 설계되었는지 평가하는 요소들이다. 이 요소들은 운영체제뿐만 아니라 다른 시스템 디자인의 평가 요소로도 쓰일 수 있다. 결국 다 돌고 돈다.

#### 1.2.1 Reliability and Availability
- Reliability(신뢰성): 의도된 대로 정확히 실행되었는지 
운영체제와 같은 정밀한 시스템에서 신뢰성은 달성하기 매우 어려운데, 모든 컴포넌트가 맞물러 동작하기 때문이다. 하나만 잘못되어도 기대하던 동작이 불가능해지고 전반적인 시스템이 돌아가지 않게 한다. 신뢰성을 높이기 위해 일반적으로 테스트를 도입해서 코드가 기대하는 일반적인 경로와 결과를 따르도록 확인한다. 하지만, 운영체제와 같이 아주 복잡한 시스템에서는 아주 작은 취약점을 이용해서 일반적인 경로에서 벗어나 생각지 못한 경로를 실행하는 것으로 신뢰성을 망가뜨릴 수 있다. 

_추가로 찾아본 부분_: 일반적인 경로에서 벗어나 취약점을 공략하는 예시로 리눅스의 Dirty COW 취약점이 있다. 이는 Copy On Write, mmap, 멀티스레드 상황을 이용해 의도적으로 경합 조건을 만들어 읽기 전용 파일을 수정할 수 있게 되는 취약점이다. [참고](https://blog.naver.com/skinfosec2000/220931851792)

- Availability(가용성): 시스템이 사용 가능한 정도
책에서는 가용성을 유저의 입력에 대응하는 척도로 보았다. 즉, 시스템이 해킹되어서 신뢰할 수 없어져도 유저의 입력에 반응한다면 가용성이 있다고 본다. 분산 시스템에서는 가용성이 유저의 요청에 최신의 데이터일 필요 없이 응답 자체를 내뱉을 수 있는지 여부(CAP theorem의 A)인데, 이와 비슷한 개념으로 봐도 될 것 같다. 

시스템이 고장나면 가용성은 줄어든다. 시스템이 켜져있는 상태가 지속될수록 가용성이 높아지므로, 가용성은 시스템이 얼마나 자주 고장나는지 - mean time to failure(MTTF), 그리고 얼마나 빨리 복구되는지 - mean time to repair(MTTR)을 가용성을 판단하는 척도로 사용한다.

#### 1.2.2 Security
컴퓨터의 작동이 악의적인 사용자에게 공격되지 않게 해야한다. 그러나 모든 컨퓨터 시스템은 보안 상 완벽하지 않다. 특히, 운영체제처럼 복잡한 시스템은 더욱 그렇다. 

그럼에도 불구하고, 운영체제는 Fault Isolation(결함 격리)을 통해 최대한 악의적인 공격이 시스템 전체를 망가뜨리지 않도록 한다. 결함 격리는 신뢰할 수 없는 스크립트를 가상화된 머신에서 실행시켜서 최악의 경우에 가상화된 머신만 해킹되도록 한다. 다만, 모든 프로세스를 격리해서 실행하면 프로세스 간 통신을 하지 못한다. 이를 가능하게 하기 위해 어떤 프로그램은 어떤 공유 데이터에 접근/변경할 수 있는지 등의 보안 정책을 유지한다. 이 보안 정책에 문제가 있다면 악용되어 시스템을 해킹하는데 사용될 수 있다. 공격 면적을 최대한 줄여야 하겠다.

#### 1.2.3 Portability
#### 1.2.4 Performance
#### 1.2.5 Adoption
#### 1.2.6 Design Tradeoffs
### 1.3 Operating Systems: Past, Present, and Future
#### 1.3.1 Impact of Technology Trends
#### 1.3.2 Early Operating Systems
#### 1.3.3 Multi-User Operating Systems
#### 1.3.4 Time-Sharing Operating Systems
#### 1.3.5 Modern Operating Systems
#### 1.3.6 Future Operating Systems
Exercises
## 2 The Kernel Abstraction
### 2.1 The Process Abstraction
### 2.2 Dual-Mode Operation
#### 2.2.1 Privileged Instructions
#### 2.2.2 Memory Protection
#### 2.2.3 Timer Interrupts
### 2.3 Types of Mode Transfer
#### 2.3.1 User to Kernel Mode
#### 2.3.2 Kernel to User Mode
### 2.4 Implementing Safe Mode Transfer
#### 2.4.1 Interrupt Vector Table
#### 2.4.2 Interrupt Stack
#### 2.4.3 Two Stacks per Process
#### 2.4.4 Interrupt Masking
#### 2.4.5 Hardware Support for Saving and Restoring Registers
### 2.5 Putting It All Together: x86 Mode Transfer
### 2.6 Implementing Secure System Calls
### 2.7 Starting a New Process
### 2.8 Implementing Upcalls
### 2.9 Case Study: Booting an Operating System Kernel
### 2.10 Case Study: Virtual Machines
### 2.11 Summary and Future Directions
### Exercises
## 3 The Programming Interface
### 3.1 Process Management
#### 3.1.1 Windows Process Management
#### 3.1.2 UNIX Process Management
### 3.2 Input/Output
### 3.3 Case Study: Implementing a Shell
### 3.4 Case Study: Interprocess Communication
#### 3.4.1 Producer-Consumer Communication
#### 3.4.2 Client-Server Communication
### 3.5 Operating System Structure
#### 3.5.1 Monolithic Kernels
#### 3.5.2 Microkernel
### 3.6 Summary and Future Directions
### Exercises

# v2

# v3
# 9.
With a high-speed local area network such as a data
center, the latency to fetch a page of data from the memory of a nearby computer is
much faster than fetching it from disk. Cooperative Caching (70p)

Local disk or non-volatile memory. For client machines, local disk or non-volatile
flash memory can serve as backing store when the system runs out of memory. In
turn, the local disk serves as a cache for remote disk storage. For example, web
browsers store recently fetched web pages in the client file system to avoid the cost of
transferring the data again the next time it is used; once cached, the browser only
needs to validate with the server whether the page has changed before rendering the
web page for the user.
Q. 어떻게 serverside 에 페이지가 바뀐 적이 있는지 validate하나? HTTP 전송 한 번 하고 들어오는 헤더에서 retently-updated 같은 값을 확인함으로써?

Thrashing  
A program thrashes if the cache is too small to hold
its working set, so that most references are cache misses. Each time there is a cache miss, we need to evict
a cache block to make room for the new reference. However, the new cache block may in turn be evicted
before it is reused.
The word “thrash” dates from the 1960’s, when disk drives were as large as washing machines. If a
program’s working set did not fit in memory, the system would need to shuffle memory pages back and forth
to disk. This burst of activity would literally make the disk drive shake violently, making it very obvious to
everyone nearby why the system was not performing well.

Fully Associative 캐시 단점  
메모리 크기는 계속 증가 (Moore's Law)
증가한 메모리 크기에 맞춰 캐시 라인 크기를 늘리든, 엔트리 개수를 늘리든 해야 함
1. 라인 크기를 늘림 
    - 프로그램의 spatial locality 한계가 있어서 큰 라인을 가져오더라도 활용할 가능성 낮음
    - 멀티프로세서에서 다른 프로세스가 해당 캐시 라인을 invalidate 할 가능성 높아짐 (false sharing)
2. 캐시 엔트리 개수를 늘림
    - Fully Associative의 동시 병렬 검색을 위해서 필요한 Comparator가 증가된 엔트리 수만큼 더 필요함 

File I/O by two methods  
1. Using system calls read/write
- 디스크에 있는 파일은 유지한 채로 적절한 자료구조로 변형한 복사본을 커널 메모리에 올리고, 유저 버퍼에도 올려줌
- 디스크 <-> 커널 메모리 <-> 유저 프로세스 버퍼
- read/write 시스템 콜 이용 시 커널이 개입해 유저 버퍼에 값을 올려주고, 유저 버퍼의 작성 값을 커널 메모리에 담고 있었다가 디스크로 flush
2. Memory Mapped File
- memory의 특정 공간에 파일을 매핑시켜 파일을 program segement로 다룰 수 있게 함.
    - system call이 아닌 load write 등 일반 명령어 사용해 읽고 쓰기 가능
    - copy본에 작업하는 게 아니라, 디스크에 바로 작업함
        - 메모리와 디스크를 대응시킴: mmap() 시스템 콜의 도움을 받음
        - 덕분에 메모리에 작업 -> 디스크에 적용
        - 일반적으로 write-back 으로 변경 사항이 바로 디스크에 적용되지는 않음
    - 일반적인 load, write로 실행된다는 것의 의미: 만약 파일이 메모리에 올라와있지 않다면 page fault 나고 디스크에서 로드 후 아무 일 없던 것처럼 재실행 됨
        - 프로그램을 처음 시작할 때도 적용됨: executable을 메모리에 올릴 때 모든 파일들을 올릴 필요 없이 필요한 첫 페이지만 올려서 빠르게 실행한 후 뒤에 필요한 페이지들은 그때그때 페이지 폴트를 통해 로드함 


주기적으로 추방 가능성이 높은 Dirty 페이지를 백그라운드에서 디스크에 복사함으로써 Clean으로 만든다 (p91)
- 나중에 필요할 때 해당 dirty 페이지가 추방된다면 dirty이므로 디스크에 변경 사항을 적용해야 하고, 재개까지 시간이 오래 걸림
- 그래서 미리 추방 가능성이 높은 페이지를 찾고, 이게 dirty라면 디스크로 동기화 해 줌으로써 clean 상태로 만들고, 나중에 필요 시 디스크 I/O 없이 바로 페이지 갈아끼울 수 있음


페이지가 수정되어 디스크에 쓰여질 때 페이지 테이블의 dirty bit를 초기화한다. 이 때 그 가상주소에 해당하는 TLB 엔트리를 삭제(shootdown)한다. 왜?
- TLB도 엔트리 별 dirty bit을 관리하고 있기 때문이다.
    - 그럼 페이지 테이블에서처럼 dirty bit만 초기화하면 되지 않나? 왜 엔트리 전체를 삭제하는거지? 비효율적인데?
    - TLB는 크기가 작고 극히 일부분의 명령만 수행할 수 있기에 엔트리 안에 있는 특정 위치(dirty bit)를 조작할 수 없다. 따라서 dirty bit을 초기화 시키기 위해 엔트리 삭제를 할 수밖에 없다. 물론 dirty bit만 조작할 수 있다면 효율적이다.


Swapping
- 돌아가고 있는 프로세스들의 메모리 요구량 총계를 메모리가 감당하지 못할 때 특정 프로세스가 차지하는 모든 메모리 페이지를 disk로 flushing 해 다른 프로세스에 나눠줌으로써 메모리 요구를 맞춰주는 방식
- 만약 그대로 놔둔다면?
    - 메모리가 부족하기 때문에 프로세스의 메모리 요구를 맞추기 위해 다른 프로세스가 active하게 사용 중인 메모리 페이지를 disk로 flush해야 한다. 이 과정이 계속 반복되면서 throughput 극도로 낮아지는 문제 발생함.


# 10. Advanced Memory Management
## Zero-Copy I/O. How do we improve the performance of transferring blocks of data
between user-level programs and hardware devices? 


## Virtual Machines. How do we execute an operating system on top of another
operating system, and how can we use that abstraction to introduce new operating
system services?  
- shadow page table: Host OS가 Guest User Process의 가상 메모리 -> Host 물리 메모리를 매핑해주는 페이지 테이블 
    - Guest User Program 가상 메모리 <-> Host 물리 메모리 매핑
    - 호스트 입장에서 똑같은 유저 프로세스인 Guest OS와 Guest User Program을 구분하기 위해 사용함. 구분하지 않는다면 Guest User 프로세스가 Guest Kernel 메모리 영역을 수정하려할 때 호스트 입장에서는 똑같은 유저 프로세스 메모리 영역이므로 정상이라 판단할 수 있게 되고, 게스트 프로그램이 게스트 커널 장악 가능.
    1. Guest Page Table (Guest OS가 관리)
        - Guest 가상 주소 -> Guest 물리 주소
        - Guest OS가 자유롭게 수정
        - 실제 사용은 안 됨 (참조용)

    2. Shadow Page Table (VMM이 관리)
        - Guest 가상 주소 -> Host 물리 주소
        - Guest OS의 page table 변경 추적
        - 권한도 올바르게 복사
        - CPU가 실제로 사용

- Multiple copies of the same page: 같은 내용의 페이지를 여러 개 두지 말고, 메모리 상에서 "단 하나"만 둔 채, 이를 사용할 예정의 페이지들이 해당 단 하나의 페이지를 참조하게 하고, write 가 일어날 때에야 새로운 공간에 작성함으로써 공간을 줄이는 방법
- Compression of unused pages: 비슷한 내용을 가진 페이지들을 압축해서 메모리 사용량을 줄이는 방법. 비슷한 내용의 페이지가 두 개(A, B) 있을 때, 압축할 페이지(B)에서 원본 페이지(A)와 다른 부분을 빼내고 이 부분(B')만 따로 저장한다. 이후에 압축된 페이지가 참조될 때 원본 페이지(A)와 압축된 부분(B')을 사용해서 압축한 페이지(B)를 복원해서 사용한다.
    - 이 방법이 효과적인 이유는 디스크로 내려가지 않고 메모리에서 압축 및 복원 작업을 수행하기 때문임. 메모리 공간이 부족해서 B 페이지가 디스크로 내려갔다면 B를 나중에 읽을 때 Disk I/O를 수행해야 하기 떄문


## Fault Tolerance. How can we make applications resilient to machine crashes?
### Checkpoint and restart
프로세스의 메모리 상태를 일정 주기로 디스크에 저장하고 나중에 불러올 수 있게 한다. 어떻게?
- 메모리 통째로 디스크에 복사: 경합 문제 때문에 복사 시간동안 메모리 변경 막기 위해 프로세스 진행을 블락해야 됨. 느리고 쌉손해
- Copy-On-Write로 경합 문제를 없앤다면? 메모리 페이지들을 Read-only로 변경하고 복사를 프로세스 진행과 병렬적으로 수행시킨다. 프로세스가 Read-only 페이지를 변경할 때 COW하면 안전하게 프로세스 상태 저장 가능. 그래도 메모리 자체를 복사하는 것이므로 느리다.
- 메모리 자체를 복사하지 않고 diff만 남길 수는 없나? 메모리 변경 명령어들만을 저장한다면 메모리를 통째로 복사하는 것보다 시간이 적게 걸릴 수 있다. 불러올 때는 저장한 명령어들을 순차적으로 실행하면 된다. 다만 이렇게 되면 매 메모리 write 마다 커널 트랩을 발생(컨텍스트 스위치)시켜야 하고 디스크에 작성(이건 백그라운드에서 가능하긴 할듯)해야 하기 때문에 엄청 느려질 수 있다. 프로세스 진행 상한이 프로세서의 속도가 아니라 트랩 핸들러의 속도에 막힌다. 또한 순차적으로 로그를 쌓는 형태이므로 삭제한 예전 내용을 보존하는 문제도 있을 수 있다.
    > So if you write a memo insulting your boss, and then edit it to tone it down, it is best to save a completely new version of your file
before you send it off!
- 명령어 단위가 아니라 좀 더 큰 단위로 가자. 페이지 단위로 변경 사항을 저장하자. 모든 페이지를 read-only 로 변경한다. write 시 트랩 발생시켜 dirty bit을 세팅한다. 이후 이 페이지의 write는 트랩 없이 쭉 진행 가능하다. 체크포인팅 시 dirty bit이 켜져있는 페이지들만 디스크로 복사한다. 이렇게 되면 전체 페이지 복사할 필요 없고 write마다 트랩이 발생하지도 않는다. 



## Security. How can we contain malicious applications that can exploit unknown faults inside the operating system? 
내 운영체제와 같은 환경의 가상 머신에서 앱을 다운받고 돌린다. 
> And of course, reinstalling your system after it has become infected with a virus is even slower!
회사 차원에서 대규모로 돌리는 것을 virtual machine honeypot 이라고 한다. 
한계로는 
- 바로 시스템을 탈취하려 드는 바이러스의 경우 쉽게 알 수 있지만, 가만히 잠복해있는 경우 이게 바이러스인지 아닌지 판단이 어렵다. 
- 가상머신뿐만 아니라 더 나아가 호스트로 탈출해서 호스트 머신까지 감염시키는 바이러스가 있다면 큰일날 것이다. 찾아보니 실제로 그런 경우가 꽤 되는 것 같다. [Virtual Machine Escape](https://en.wikipedia.org/wiki/Virtual_machine_escape)


## User-Level Memory Management. How do we give applications control over how
their memory is managed? 
DB, Garbage Collector, Sandbox 등 많은 어플리케이션들은 메모리 기대치에 따라 작동을 다르게 한다. 이 기대치와 page evict, allocation 등 운영체제가 제공하는 메모리 관리 정책과 괴리가 생기면 불필요한 페이징, thrashing 등 문제가 생긴다. 이를 해결하기 위해,
- Pinned page: 어플리케이션에서 특정 가상 페이지를 물리 페이지에 고정시켜 운영체제가 evict하지 못하게 한다.
- User-level pagers: 커널이 페이지 폴트를 핸들링하지 않고 유저 어플리케이션에서 정의된 페이지 폴트 핸들러에게 처리를 위임한다. 어플리케이션을 잘 알고 있는 유저 핸들러가 어떤 페이지를 evict하고 불러올지 결정한다. 


# v4 Persistent Storage

## File Systems: Introduction and Overview
disk와 같은 non-volatile storage는 random access가 안 됨. bulk로 가져와야 함. 물리적으로 모터를 움직여야 함. 디스크 random access 시간(=10ms) >>> DRAM 시간(=100ns)
지속되어야 하는 데이터는 disk에 넣어야 하는데 사용자가 다루기 쉬운 구조, 디스크 접근을 최소화 한 운영체제의 기능이 파일 시스템임 

File: 이름붙여진 데이터들의 집합. 걍 임의의 길이의 바이트 배열임. 
- 무작위 바이트 배열이라면 어떻게 OS가 파일을 실행할 수 있지? 
두 가지로 나뉜다. binary file 혹은 script file
- binary file이면 파일을 파싱해서 code, data 섹션들 메모리로 로드하고 시작한다. 
- script file이면 그 스크립트를 실행할 interpreter을 불러와서 얘한테 맡겨야 한다. 
어떻게 OS가 구분하지?
- 리눅스 기준 ELF 바이너리는 0x7f, 0x45, 0x4c와 0x46로 시작한다. 
- 스크립트 파일은 #! 로 시작한다. ex) #! bin/sh
- .exe, .pl 등 확장자로 구분할 수도 있다.

Directory: 파일에 이름을 부여해주는 것. 파일과 다르게 바이트를 저장하지 않음. 디렉토리는 파일을 가질수도 디렉토리를 가질수도 있음 -> 계층

Volumn: 파일과 디렉토리가 저장된 논리적 디스크 ex) USB 내에 있는 파일 시스템. 논리적 구조 덕분에 한 volumn을 volumn에 붙일 수 있다. 이를 마운트(mount)라 한다. USB를 컴퓨터에 연결하면 USB 볼륨이 컴퓨터 볼륨에 마운트되어 컴퓨터 볼륨에서 USB에 저장된 내부 파일을 보거나 수정할 수 있다.


11.2 API
11.3 Software Layers
11.3.1 API and Performance
file system API and also provide caching and write buffering 제공
- Block cache: Disk의 파일 내용 많이 버퍼에 가져와 캐시 -> 작은 요청 여러 번 하더라도 syscall or Disk IO 발생하지 않음. 다른 프로세스들 간 동기화 되어야 함.
- Prefetching: 해서 나중에 쓸 것으로 보이는 디스크 블록들도 메모리에 미리 가져옴

11.3.2 Device Drivers: Common Abstractions
OS와 하드웨어 I/O 사이를 이어줌. OS가 하드웨어를 몰라도 같은 명령으로 동일한 기능을 할 수 있게 하드웨어를 추상화 해 줌. 그래서 하드웨어 제작사가 해당 하드웨어를 위한 Device Driver도 만드는게 보통임. 
_Challenge: device driver reliability_
커널 루틴이 의존하고, 하드웨어와 상호작용 해야하기 때문에, 디바이스 드라이버는 운영체제의 일부로써 실행된다. 따라서 드라이버가 결함이 있다면 운영체제에까지 악영향을 미칠 수 있게 된다. 따라서 어렵겠지만 드라이버도 유저 프로세스들처럼 false isolation을 잘 시켜서 실행해야 한다. 

11.3.3 Device Access
하드웨어는 메모리와 구조부터 다르다. byte addressable도 아니다. 어떻게 디바이스 드라이버는 하드웨어에 엑세스하고 CPU, 메모리와 소통할 수 있을까? 세 가지 방법 덕분에 가능하다. 
### Memory-mapped I/O.
<p align="center">
    <img src="/assets/io-device.png" width=100% height=100% text-align=center />
</p>
디바이스 드라이버는 위 사진처럼 I/O Bus와 Memory Bus 두 버스와 연결되어있다. 그래서 하드웨어에서 버스 트래픽이 발생하면 그에 맞는 디바이스 드라이버가 작업을 해서 메모리 버스에 신호를 보내주는 식이다. 
그렇다면 어떻게 CPU는 하드웨어 신호를 인지할 수 있을까?
물리 메모리의 특정 영역을 디바이스 드라이버에게 각각 할당한다. 해당 주소로의 Read/Write 명령은 메인 메모리로 가지 않고 디바이스 컨트롤러의 레지스터로 간다. 이렇게 CPU와 드라이버가 소통할 수 있다.
```asm
MOV register, memAddr // To read
MOV memAddr, register // To write
```

### DMA
CPU 개입 없이 하드웨어와 메모리 간 데이터를 주고받는 방식
1. CPU가 memory mapped I/O를 통해 장치의 DMA 레지스터에 어떤 데이터를 어디로, 얼마나 보낼 지 설정한다
2. 장치가 자신의 DMA 엔진을 통해 메모리 버스를 사용해 데이터를 메모리로 직접 전송한다
3. 완료 시 인터럽트를 발생시켜 CPU에게 알린다

### Interrupts
I/O 디바이스의 DMA가 끝났을 때 인터럽트로 완료를 알린다. 다른 옵션으로는 CPU가 memory mapped된 영역을 계속 read하는 Polling 방식이 있는데, 디바이스 접근은 느릴 뿐더러 완료 시점이 불규칙하기 때문에 인터럽트를 주로 쓴다.

11.3.4 Putting It All Together: A Simple Disk Request
유저 프로세스가 `read()` 시스템 콜을 호출했을 때 어떻게 실행되는지 생각해보자.
1. 커널은 호출한 스레드를 wait 큐에 넣고 block 시킨다.
2. 커널은 MMIO(Memory Mapped I/O)를 이용해 아래 정보를 디스크에 보낸다.
    - 필요한 데이터 정보(위치, 크기)
    - 어느 영역으로 보낼 지(DMA 세팅)
3. 디스크가 데이터를 읽고 DMA로 메모리에 값을 복사해 넣는다.
4. 디스크는 인터럽트를 발생시킨다.
5. 커널의 인터럽트 핸들러는 복사된 데이터가 있는 커널 버퍼의 데이터를 유저 버퍼로 복사한다.
6. 커널의 인터럽트 핸들러는 1번에서 block 시켰던 스레드를 ready로 변경하고 종료한다.
7. 스레드가 스케줄링 되었을 때 준비된 데이터를 사용할 수 있다.

11.4 Summary and Future Directions
Exercises


12 Storage Devices
디스크는 DRAM과 다르게 랜덤 엑세스 시간이 엄청 느리고 한 번에 몇 천 바이트를 읽어온다. 같은 디렉토리 안에 있는 파일들을 인접한 섹터에 보관해놓으면 sequential reading이 되기에 파일 시스템 디자인 시 디스크의 특징을 잘 알아야 함. 

12.1 Magnetic Disk
디스크의 최소 저장 단위는 섹터(Sector)다. 이는 보통 512B로 한 번의 읽기/쓰기 단위가 512B로 일어난다. 한 바이트를 변경하고 싶어도 500배의 공간에 접근해야 한다.
왜 더 줄이지 못하는가?
- 하드웨어는 섹터 별로 자체 에러 수정 코드(Error Collection Code)를 관리하는데, 단위가 작아지만 ECC 오버헤드가 커지기 때문

12.1.1 Disk Access and Performance
### Seek
모터 돌려서 읽고자 하는 트랙으로 이동

### Rotate
트랙 안에서 원하는 섹터로 이동
최적화: 새로운 track으로 바뀌고 목표 섹터로 갈 때, 원하는 섹터 말고도 헤드가 지나가면서 접근한 섹터들의 내용도 모두 읽어서 디스크 버퍼에 기록해 놓는다. 다음에 미리 읽어놓은 부분 요청이 들어왔을 때 full-ratation 하지 않고 바로 응답할 수 있게 됨.

### Transfer 
원하는 섹터에서 rotating 하면서 read/write 함. Seek, Rotate 시간보다 짧음. 
디스크 <-> 디스크의 버퍼 메모리 <-> 호스트의 메모리
디스크의 바깥쪽 트랙은 안쪽 트랙보다 면적이 넓으므로 바깥쪽 트랙의 transfer rate가 안쪽보다 더 크다. 그래서 transfer rate를 보면 하나의 값이 아닌 50~128 MB/s 처럼 범위로 되어있는 것이다.
Random이 아니라 순차적 읽기를 하면 Seek, Rotation은 더 할 필요없이 Transfer 만 여러 번 하면 되기 때문에 비용이 급격히 낮아진다.

12.1.2 Case Study: Toshiba MK3254GSY


12.1.3 Disk Scheduling
inner disk <-> outer disk 간 딜레이, rotation time 등으로 디스크에 들어오는 read/write 요청들을 out of order로 스케줄링 하는게 의미가 생김
12.2 Flash Storage
12.3 Summary and Future Directions
Exercises

13 Files and Directories
파일 이름과 디스크에 저장된 block의 offset 만으로 어떻게 메타데이터와 내용을 가진 계층적인 파일 시스템을 만들 수 있을까?
그러면서도 spatical locality를 잘 성취해야 하고, 다양한 명령에 범용적으로 기능해야 하며, OS나 하드웨어가 크래시 되어도 정보는 잘 저장되어야 한다.

13.1 Implementation Overview
파일 이름과 block offset 을 physical storage block에 매핑한다.
directories, index structures, free space maps, and locality heuristics 네 가지 테크닉을 사용한다.

<p align="center">
    <img src="/assets/file-systems-map.png" width=100% height=100% text-align=center />
</p>

directories와 index structures는 file name을 물리 디스크 블락으로 매핑시켜주는 데 사용된다.
free space maps는 효율적인 공간 활용을 위해 빈 공간 정보를 추적한다. 
다양한 locality heuristics 하에서 위 요소들이 구현될 수 있다.

13.2 Directories: Naming Data
디렉토리는 `파일 이름 -> 파일 값` 매핑 정보들을 가지는 파일이다.
원래는 단순 링크드 리스트로 관리했는데 파일 개수가 많아지면 링크드 리스트의 조회가 비효율적이게 되어 B+ 트리 구조를 채택하게 됨.

hard link: Leaf 노드의 포인터 값을 기존에 있는 파일의 값으로 설정하면 됨
soft link: 참조하고자 하는 파일 이름으로 대체
아래 예시 디렉토리 안에 있는 파일 세 개는 모두 같은 파일을 가리킴.
foo.txt, bar.txt는 hard link, baz.txt는 soft link 되어있음
foo.txt를 unlink해도 bar.txt에서 원본 파일에 접근 가능하지만 foo.txt를 softlinking 하는 baz.txt에서는 접근 오류가 남
```
foo.txt 871 
bar.txt 871
baz.txt foo.txt 
``` 

13.3 Files: Finding Data
Tree index는 Linked List 보다 Random access가 좋음
13.3.1 FAT: Linked List
File Allocation Table dictionary 사용해 다음 파일 블럭 값 갖게 함. 
남는 공간은 FAT의 zerod entry sequential search 하면서 얻음(next fit). 간단, but fragmentation 확률 높음

13.3.2 FFS: Fixed Tree
Unix Fast File System
inode 리스트 안에 파일 별 inode element 가짐. 파일 메타데이터, 각 물리 파일 블럭의 포인터를 가짐. 크기를 고려해 direct, indirect, double, triple indirect block pointers 를 가짐. 수 TB의 파일 지원 가능. 
이 엔트리들은 inode 파일 안에 모두 sequential하게 위치에 있어 disk에서 정보를 읽어들일 때 transfer만 하면 되므로 순차 탐색 가능, 탐색 시간 줄어듦. 
Q. inode 파일이 12 direct, 1 indirect, 1 double indirect, 1 triple indirect, and 1 quadruple indirect pointers로 구성된다면 이 인덱스 구조가 지원하는 최대 파일 용량은? Assuming 4 KB blocks and 8-byte pointers.
A. direct는 4KB => 12*4KB = 3 * 2^14B
파일 페이지 하나가 가질 수 있는 포인터 개수 = 2^12/2^3 = 2^9개
indirect = 2^9 * 4KB = 2^21 B
double indirect = 2^9 * 2^9 * 4KB = 2^30 B
triple = 2^(9*3) * 4KB = 2^39 B = 0.5 TB
quadrruple = 2^(9*4) * 4KB = 2^48 B = 256 TB
In total, ~= 256.5 TB

13.3.3 NTFS: Flexible Tree With Extents
디스크 전체의 비트맵을 메모리에 유지하는 것은 공간 상 비현실적이다. 탐색 비용도 큼
13.3.4 Copy-On-Write File Systems
디스크는 Random small write 보다 large sequential write할 때 성능이 더 좋다. 디스크의 용량(bandwidth)은 쉽게 커지는 반면, 탐색 및 작성을 위한 디스크 회전 속도(seek time, rotational latency)는 그렇지 않기 때문.

메모리 캐시로 읽기를 disk I/O 없이 쳐내는 것과 다르게 메모리 쓰기는 데이터 영속성을 위해 짧은 주기로 flush 해줘야 하기 때문에 병목은 읽기보다 쓰기에 몰림. 
쓰기 비용 >> 읽기 비용이기 때문에 쓰기를 최적화 해야 함

특정 위치에 고정되어 있는 inode 배열을 이 COW 시스템에서는 파일처럼 관리해서 고정되어있지 않게 바꾼다. write가 일어나서 블럭이 하나 생성되었다면, 그 블럭부터 root 까지 향하면서 만나는 모든 노드를 새롭게 작성한다. 이 때 디스크에 sequential하게 작성됨. 한 번에 작성해야 하는 데이터가 많지만, 
> sequential write 의 소요 시간 >>>> 그때그때 random write를 여러 번 하는 시간   

이므로 많은 데이터를 쓰더라도 random access를 피하는 것이 더 나음. 

<p align="center">
    <img src="/assets/COW-inode-write.png" width=100% height=100% text-align=center />
</p>

### ZFS
COW를 구현한 파일 시스템 오픈소스 

#### ZFS index structures
<p align="center">
    <img src="/assets/zfs-update.png" width=100% height=100% text-align=center />
</p>

Q. 위 사진과 같이 하나의 블럭 write가 일어나면 모든 부모가 새롭게 써지는 블럭 이외 다른 블럭으로 포인터는 유지한 채로 새로운 위치에 써진다. 그렇다는 말은 기존 부모의 값들을 디스크에서 읽어야 하지 않을까? 즉, COW 과정에서 random read 가 많이 일어나지 않나?

A. random read가 발생하는 것은 맞지만 블럭 작성 시 대부분의 node 정보는 메모리에 올라와 있기에 디스크 random read는 일어나지 않는다. 메모리에서 읽어오고 디스크 순차 작성 가능함.

#### ZFS space map.
아무리 비트 하나로 4KB 짜리 free 블럭을 관리한다고 해도 디스크 용량이 커진다면 메모리에 보관해야 할 비트맵의 크기도 선형적으로 커지는 문제가 있다. ex) 1PB 디스크의 free 블럭을 관리하기 위한 비트맵의 크기 = 2^50 / 2^12 = 2^38 = 256GB

Q. 비트맵 모두를 메모리에 올리지 않고 쪼개서 일부만 메모리에 올리면 되지 않나?
A. 그렇게 하면 할당은 메모리에 올라온 비트맵을 이용함으로써 효율적으로 할 수 있을 것이다. 그러나 이미 할당된 후 할당 해제되는 블럭의 경우, 메모리에 올라와 있지 않은 비트맵의 블럭일 가능성이 높으므로, 이 때는 disk random access 및 write 가 일어나는 문제가 있다. 

ZFS에서는 비트맵 크기, free의 랜덤 disk 엑세스 문제를 해결하기 위해,
- 블럭단위가 아닌 블럭을 묶은 block group 단위로 각각 free blocks를 관리
- 블럭 별 비트맵이 아닌 연속적으로 비어있는 블럭들을 하나로 묶은 범위 형태인 extents를 관리
- free 시 disk에 접근하지 않고 메모리에 append-only log로 정보만 남겨놓는다. 다른 block group이 나중에 메모리에 올라왔을 때 로그를 읽으면서 해당 block group을 업데이트하고, 메모리에서 빠질 때 자연스럽게 디스크에 업데이트 되도록 한다.


13.4 Putting It All Together: File and Directory Access
일반적인 inode 파일 구조에서 특정 파일(`/foo/bar/baz`)을 읽기까지의 과정을 생각해보자. 이름: inode number 매핑을 생각하면서 진행하면 된다. inode 파일을 읽고, 거기에 있는 파일 블럭 포인터를 읽은 후 그 쪽 파일을 읽는다. 만약, 파일이라면 내용이 써져 있을 것이고, 디렉토리라면 name: inode number 매핑이 저장되어 있을 것이다. 이렇게 쭉쭉 따라가면 된다.
<p align="center">
    <img src="/assets/file-system-example.png" width=100% height=100% text-align=center />
</p>
root 폴더는 inode number = 2로 고정되어 있으니


13.5 Summary and Future Directions
Exercises


14 Reliable Storage
지금까지는 하드웨어는 100% 데이터를 영구적으로 안전하게 보유할 수 있다고 생각했다. 그런게 어떻게 이것이 가능할까? 이제부터 그것을 알아보자. 
Reliability: 의도한 기능을 할 수 있는지, 하드웨어의 경우 데이터를 읽어들이고 작성할 수 있는지 여부 <- 우리가 알아볼 부분
Availability: 요청에 빠르게 응답할 수 있는지, 이 경우 I/O 요청에 얼마나 빠르게 응답하는지 여부

Reliability를 위협하는 것은 두 가지가 있다.
1. Update 중간에 종료되는 경우
2. 저장된 데이터가 오염되는 경우

아래 방법들로 각각을 해결할 수 있다.
1. 원자적(Atomic) 쓰기
2. 중복성(Redundancy) 성취

14.1 Transactions: Atomic Updates
먼저 파일이 생성될 때 어떤 일이 일어나는지 알아보자.
1. inode bitmap에서 free block을 찾고 allocated로 상태를 바꾼다.
2. inode entry를 할당하고 초기화한다.
3. 파일이 만들어진 디렉토리에 1에서 할당받은 inode number를 filename 과 매핑한다.
즉, 세 번의 disk write가 일어난다. 또한, 아직 파일의 메타데이터를 알려주는 inode만 만들어졌고 실물 파일은 만들어지지 않은 상태다. 어플리케이션에서 write() 시스템 콜로 파일에 실제로 내용을 작성하면,
1. free data block bitmap에서 free block을 찾고 allocated로 바꾼다.
2. data block을 할당하고 주어진 내용으로 초기화한다. 
3. inode entry의 data 필드가 2에서 새로 만든 block을 참조하게 한다.
이것도 3번의 disk write가 일어난다. 이후 write()가 호출될 때도 위와 같이 3번의 write가 일어난다. 기존 내용을 덮어쓰는 것이 아닌 새로운 블럭을 할당받고 append 하는 식으로 작동하기 때문

이처럼 하나의 operation을 할 때 write 가 여러 번 일어난다는 뜻은 reliability의 관점에서 봤을 때 그 다음 write가 일어나기 전에 시스템이 종료됐을 때를 고려해야 한다는 의미가 된다.

14.1.1 Ad Hoc Approaches
이런 문제를 위해 작성된 범용적이지 않은 해결 방법 시스템이다. 시스템이 종료되고 재부팅될 때 모든 파일의 메타데이터를 살펴보고 이상이 있는 부분을 롤백함으로써 reliability를 달성한다. 예를 들면, 파일을 생성할 때 2번 inode entry를 할당한 뒤 시스템이 종료되었다면, 해당 inode 엔트리는 어느 디렉토리에도 속하지 않은 상태가 된다. 이 시스템은 모든 파일을 살펴보면서 이런 이상 현상을 발견하고 적절하게 조치한다. 이 예에서는 할당되었지만 어디에도 속하지 않은 inode entry를 할당 해제함으로써 해결한다. 이런 시스템의 예시로는 FFS(Unix Fast File System)의 fsck(file system check)가 있다. 

예상할 수 있다시피 이런 시스템에서는 명확한 단점이 있다. 
- 모든 가능한 상황을 고려하고 복구하는 시나리오를 짜야 하므로 시스템이 엄청나게 복잡해질 수 있다. 
- 이 시스템이 동작하기 위해서는 각각의 쓰기 작업이 차례차례 디스크에 작성 완료된 후 다음 쓰기를 작성해야 한다. 즉, bulk sequential write를 하지 못하고 개별 random write를 해야 한다. 비용이 높아진다.
- 마지막이자 이 시스템이 힘을 잃은 가장 큰 이유는, 재부팅 될 때마다 디스크의 모든 파일을 읽어야 하기 때문이다. 디스크 용량이 기하급수적으로 커짐에 따라 현대 서버에서 매 부팅마다 모든 파일을 읽히는 비용은 말이 안되게 되었다.


14.1.2 The Transaction Abstraction
하드웨어에서와 같이 여기서 말하는 트랜잭션도 여러 가지의 작업을 한 개로 묶어서 한 번에 처리해주는 기능이다.

14.1.3 Implementing Transactions
트랜잭션이 끝났다고 해서 그 결과가 바로 디스크에 반영되는 것은 아니다. Commit과 Write back이 이뤄져야 디스크에 변경 사항이 반영된다.  
Commit은 일련의 트랜잭션이 끝난 후 디스크의 로그 영역에 변경된 데이터가 적혀진 상태로, 크래시가 나더라도 디스크에 적힌 로그를 보면서 트랜잭션이 끝난 상태로 되돌릴 수 있는 durability를 보장한다. 중요한 점은 아직 변경된 파일은 메모리에만 있고 디스크에는 적용되지 않는다. 
Write back 단계에서 메모리에 있는 commit된 변경 사항을 디스크로 적용시킨다. 이를 `redo logging`이라 한다.

예상하겠지만 이렇게 하는 이유는 성능 때문인데, commit할 때마다 디스크에 동기화하면 disk write가 트랜잭션마다 일어나기 때문에 비효율적이기 때문이다. 따라서 최소한의 durability 를 보장하기 위해 작은 로그를 특정 디스크 공간에 저장해두고 실제로 변경 사항을 디스크에 반영시키는 것은 비동기적으로 적절한 시간에 여러 변경사항이 함께 묶여 진행된다. 이외에도 변경 사항들을 한꺼번에 디스크에 적용함으로써 bulk writing의 이점도 가져갈 수 있다.

Q. Commit 시 로그에 담긴 데이터와 Write back 시에 적히는 데이터가 같나? 같다면 공간 낭비 아닌가? 왜 그렇게 하는가?
A. 맞다. 로그와 write back 시 쓰이는 데이터는 실제로 같아야 한다. 그게 Commit의 목적이기 때문인데, commit 후 크래시가 났다면 로그의 데이터를 보면서 디스크를 복구해야 하기 때문이다. write back이 끝난 로그는 지움으로써 공간을 확보한다.

Q. commit 후, write back 전에 크래시가 났다고 가정해보자. 복구 시 이 변경 사항은 디스크에 반영해야 하나, 반영하지 않아야 하나?
A. 반영해야 한다. 복구 시 작성된 로그를 따라가면서 디스크에 적용한다. 흥미로운 점은 DBMS의 복구 방법은 이런 파일 시스템의 복구 방법과 다르다는 것인데, DBMS는 비슷한 상황에서 롤백을 하기도 한다. 복구 시 파일 시스템은 롤백이 필요 없고, DBMS는 롤백이 필요해 DBMS의 트랜잭션 로깅이 파일 시스템의 로깅보다 훨씬 복잡하다.

Q. 트랜잭션이 쌓이면서 로그가 비대해질 것이다. 로그가 비대해지면 디스크 공간뿐만 아니라, 크래시에서 복구(recovery)할 때 읽어야 하는 로그의 양이 많아지고, 이는 시간 상으로도 손해가 클 것 같다. 어떻게 해결할까?  
A. 트랜잭션을 진행하면서 로그의 양을 줄임으로써 해결한다. Write back이 끝난 로그는 이미 디스크에 적용되어 있으므로, 그 다음에 진행하면서 크래시가 일어나도 디스크에 그대로 데이터가 남아 있을 것이다. 따라서 write back 까지 진행된 트랜잭션에 대한 로그는 지워도 된다.   
내부 로그 가비지 콜렉터(Log GC)가 트랜잭션 로그 데이터의 각 트랜잭션을 보면서 
1. 커밋되었는지
2. 트랜잭션과 연관된 dirty page들이 모두 write back(=flush) 되었는지   

확인하고 해당하는 트랜잭션은 로그에서 지운다. 

트랜잭션 로그의 생애 주기는 다음과 같이 정리할 수 있다.  
> 트랜잭션 실행 -> 로그 기록 -> Commit -> (시간차) Write back -> Garbage collection 가능

<p align="center">
    <img src="/assets/transactional-log-structure.png" width=100% height=100% text-align=center />
</p>

위와 같이 메모리에서 log head, log tail 포인터를 관리한다. 종종 메모리의 log head 포인터를 디스크의 것과 동기화시킨다. 이 때 가비지 콜렉터는 디스크의 log head 포인터 이전의 공간을 메모리에서 안전하게 해제할 수 있다. 



14.1.4 Transactions and File Systems
이전에 시도했던 명령의 순서를 조정하고 재부팅 시 디스크를 스캔해서 잠재적 문제를 고치는 Adhoc 방법의 트랜잭션은 하드디스크 용량이 커짐에 따라 공간/시간 비용이 높아져 가능하지 않게 되었다. 

파일 시스템은 FFS와 NTFS와 같은 전통적인 파일 시스템과 ZFS와 같은 COW(Copy On Write)를 가진 파일 시스템 두 종류로 나눌 수 있다. 어떻게 둘 다 adhoc 방법에서 벗어나 효과적으로 consistency를 만족하는지 살펴보자.

#### 전통적인 파일 시스템
- Journaling: 저널링은 파일 생성, 삭제 등 파일의 메타데이터를 변경할 때 트랜잭션을 이용해 redo log에 변경 내역을 기록해 consistency를 보장한다. 그러나 메타데이터가 아닌 파일의 내용 자체를 변경할 때는 log에 기록하지 않는데, 내용은 메타데이터 변경보다 용량이 커서 로그에 다 담기에 부담되기 때문이다. 따라서 저널링 정책에서는 파일 내용의 consistency는 보장되지 않는다. 파일의 내용을 작성하다 시스템이 종료되면 어떤 상태로 재부팅될 지 알 수 없다.
- Logging: 파일의 내용도 모두 redo log에 기록함으로써 모든 consistency를 보장한다.

#### COW 파일 시스템
생각해보면 COW의 특성에 따라 이 친구는 애초에 consistency가 보장된다. 기존 파일의 내용을 덮어씌우지 않고 새로운 공간에 데이터를 다 만들어 놓은 후 root의 포인터만 변경해주기 때문이다. 기존의 데이터 구조를 덮어쓰는 동작을 단 한 번(root의 포인터 변경) 수행하므로 atomic하다.

하지만, 살펴봤듯 매 요청마다 새로운 data block부터 root에 이르기까지 path에 있는 모든 블럭들을 새로 디스크에 만드는 것은 비효율적이다. ZFS는 몇 가지 방법으로 이를 최적화했다. 
1. Batch Update: 요청 하나하나만이 아니라 여러 개를 모아서 한 번에 디스크를 업데이트한다. 규모의 이점을 누릴 수 있는데,
- 디스크에서는 여러 번의 랜덤 쓰기보다 한 번의 큰 쓰기가 훨씬 낫다
- 운이 좋아서 이어진 구역을 한 번에 작성하게 되면 더 효과적이다
2. Intent log: 워드 프로세서에서 사용자가 저장 키를 입력했을 때처럼 어플리케이션이 파일 시스템에게 강제로 디스크와 동기화를 요청할 수 있다. 이 때 변경 사항이 많다면 동기화까지 몇 초 이상 걸릴 수 있고, 어플리케이션을 blocking한다. 사용성을 높이기 위해 실제로 동기화를 진행하지 않고, Intent log라는 디스크 특정 위치에 있어 빠르게 접근할 수 있는 로그에 변경 사항들을 작성만 하고 디스크에는 동기화를 진행하지 않음으로써 동기화 시간을 줄인다. Intent log는 본질적으로 redo log처럼 동작해, 어플리케이션이 다시 켜졌을 때 Intent log와 맞게끔 디스크를 업데이트한다. 정리하면, 
- 사용자가 동기화를 명시적으로 요청했을 때만 작동
- Intent Log는 디스크의 지정된 구역에 위치해 접근이 빠르다 -> 작성이 빠르다
- 단일 변경 사항이 큰 경우에는 `intent log에 작성 -> 나중에 disk에 재작성`으로 두 번 작성하는 비용을 피하기 위해, log가 아닌 처음부터 디스크의 다른 구역에 작성 후, 나중에 그 파일을 가리키는 disk의 indirect block 포인터만 변경한다. _근데 이건 redo log도 사용하는 테크닉인데, 이러면 intent log와 redo log의 차이가 없지않나 싶다_ 
그러면 intent log와 redo log의 차이점은?
- 애초에 intent log는 COW 파일시스템에, redo log는 COW가 아닌 전통적 파일 시스템에 있는 로그다. 둘이 공존하지 않음
- intent log는 `fsync()`와 같이 명시적으로 파일 시스템-디스크 싱크 요청이 들어왔을 때 전체를 디스크에 랜덤하게 쓰기 보다는 그 요청 하나에 대해서만 나중에 참고할 수 있게 디스크에 로그만 빠르게 작성함으로써 요청 latency를 줄이는 기능
- redo log는 모든 변경사항을 저장함
 
14.2 Error Detection and Correction
14.2.1 Storage Device Failures and Mitigation
Storage Device는 물리적인 다양한 이유로 일부, 혹은 전체가 망가질 수 있음
- 하드 디스크: 테이프 스크레치, 헤드 손상 등
- 플래시 디스크: 특정 셀의 전자 과충전 으로 옆 셀에 영향을 미침
정보가 촘촘히 저장되어있기에, 이런 오류는 독립적으로 일어나지 않고 일단 발생하면 연쇄적으로 전파되어 장애 확률을 높일 수 있음. 또한, 디스크가 처음 만들어졌을 때 오히려 오류 확률이 높고, 오래 사용되었을 때는 당연히 디스크도 수명이 있으니 오류 확률이 높아짐. 즉, 오류 확률은 독립적이지 않고, 일관되지 않음.

14.2.2 RAID: Multi-Disk Redundancy for Error Correction
이런 물리적 한계 때문에 디스크는 필연적으로 신뢰성(Reliability)이 떨어질 수밖에 없음. 완전무결한 디스크를 만드는 것은 현실적으로 불가능하고, 만든다 하더라도 비쌀 것이므로 다른 방법으로 end-to-end reliability를 보장함. 바로 저렴한 디스크를 여러 개 사용해서 같은 정보를 여러 번 저장하는 것임. 데이터를 서로간에 잘 배치해서, 하나가 오류가 발생했을 때 다른 것들이 정상이라면 오류를 스스로 고칠 수 있는 시스템을 만든 것이 여기서 알아볼 RAID다.

- Mirroring: 동일한 데이터를 다른 디스크에도 넣음 -> single point failure 방지
- Rotating parity: 데이터를 여러 개로 쪼개서 여러 디스크에 나눠 넣고 나눠 넣은 데이터들을 관리하는 데이터(parity)도 다른 디스크에 넣음. 즉, N개 데이터를 관리한다면 N+1개의 디스크가 필요함. parity는 쪼개 넣은 N개의 데이터들을 XOR한 값으로, 데이터 하나가 없어져도 parity와 그 외의 데이터들을 XOR하면 복구 가능함. 데이터 하나를 업데이트할 때 데이터와 parity가 있는 디스크 둘 다에 접근해서 읽고, 값 변경 후 써야 함 -> 4 I/O

> **왜 4 I/O ?**  
> 기존 데이터 읽기, 기존 parity 읽기, 새 데이터 쓰기, 새 parity 쓰기를 해야 하므로 I/O가 네 번 일어난다. Parity > 업데이트는 `new_parity = old_parity XOR old_data XOR new_data` 공식을 사용하기 때문에 기존 값들을 읽어야 한다.

여기서 생각해야 할 점은 parity가 저장될 디스크를 정하는 것이다. 하나의 디스크에만 parity를 저장하게 된다면 쓰기 작업이 일어날 때마다 parity가 저장된 디스크도 항상 접근해야 한다. 이 디스크가 쉽게 병목될 수 있다. 따라서 parity는 여러 디스크에 골고루 분산해준다. 

데이터 유실 시나리오
하나의 디스크의 데이터만 유실되었을 때는 다른 디스크들을 이용해 데이터를 복구할 수 있다. 하지만 RAID를 적용해도 같은 stripe 에 있는 디스크 두 개 이상에 문제가 생기면 복구할 수 없다.
- 디스크 두 개 이상에서 데이터가 유실되면 데이터를 복구할 수 없다. 
- 혹은 디스크 하나가 고장났고 이를 복구하기 위해 다른 디스크들을 이용할 때, 읽기 오류가 발생해도 데이터를 복구할 수 없다.
- 아니면 디스크 하나가 고장났고 이를 복구하기 위해 다른 디스크들을 참조하는 중 운 없게도 다른 디스크가 또 고장나는 경우 데이터를 복구할 수 없게 된다.

이를 방지하고 RAID의 reliability 를 높이기 위해서는
- redundancy를 늘려 두 개의 디스크가 고장나도 복구할 수 있게 한다.
    - Google File System(GFS)의 경우 하나의 데이터 블록을 세 개의 다른 디스크에 저장한다.
- 읽기 오류를 줄인다.
    - 주기적으로 디스크를 스캔하면서 잘못된 값을 갖고 있는지 확인하고 수정한다. 이를 Scrubbing이라 한다.
        - 수정할 수 있다면, 일시적인 읽기 문제였으므로 이제 문제 없음
        - 수정할 수 없다면, 해당 부분은 이제 못 쓰게 되었으므로 디스크의 다른 공간을 이 문제의 공간에 매핑해서 사용하게 한다.
    - 혹은 읽기 오류 확률이 적은 디스크를 사용할 수도 있다.
- 복구 시간 자체를 줄여 그 사이에 문제가 생기는 가능성을 줄인다.
    - 복구 시간이란 디스크가 죽었을 때, 새로운 디스크로 물리적으로 교환하고 데이터를 다른 디스크들에서 읽어 와 채워넣는 과정이다.
    - 미리 예비 디스크를 준비시켜 놓고, 디스크가 죽었을 때 자동으로 그 디스크로 치환되게 설계 해 놓으면, 적어도 물리적 디스크 교환 시간은 줄일 수 있다. 
        - 그럼에도 데이터를 다른 디스크에서 읽어 와 채워넣는 시간이 적지 않다.
    - 아예 백지 디스크에 데이터를 채워넣는 RAID 시스템을 벗어나 복구 시간을 줄인 시스템이 있다. Hadoop File System(HDFS)인데, 복구를 하나의 새로운 디스크를 대상으로 중앙화하지 않고 여러 디스크에 유실된 데이터를 넣어줌으로써 병렬성을 이용한다.
        - HDFS는 디스크가 고장났을 때 RAID 처럼 새로운 디스크를 넣지 않는다. 복제해놓은 데이터 블록을 다른 디스크에 복제함으로써 reliability를 유지한다.
        - 같은 데이터 블록을 **랜덤한** 세 개의 디스크에 넣어 놓고 이 복제 개수를 추적한다. 디스크 장애 등으로 복제 개수가 줄어들었을 때, 이를 감지하고 유실된 데이터가 저장되어 있던 디스크들에서 다른 디스크로 블록을 복제해 넣음으로써 복제 개수를 복구하게 된다. 
        - 데이터들이 랜덤하게 여러 디스크들에 나눠져 있으므로 읽어오는 디스크와 작성하는 디스크가 서로 다르다. RAID에서는 작성하는 디스크가 단 하나였지만 여기서는 여러 디스크에 복구 부하를 분산하게 된다.


14.2.3 Software Integrity Checks
RAID가 잡지 못하는 오류가 있을 수 있고 RAID 윗 단인 소프트웨어에서도 오류를 감지할 수 있으면 추가 reliability를 제공할 수 있을 것이다. 오류를 발견하면 아랫단인 RAID에게 해당 부분 복구하라고 요청한다.

- Block integrity metadata: 일정 크기의 파일마다 checksum을 넣고 파일을 읽을 때 checksum을 체크해서 파일이 문제없는지 확인한다.
- File system fingerprints: 파일 시스템이 root inode 기준으로 트리를 구성한다는 특징에 착안해 각 노드는 자신의 자식 노드들의 포인터뿐만 아니라 checksum까지 보유한다. Read 할 때 자식 파일의 checksum 계산 후 맞는지 확인한다. Leaf를 Write할 때는 부모 노드까지 타고 가면서 새로운 포인터와 checksum을 넣고 root node까지 전파한다.