# **3장 - Processes**

운영체제의 주요 관심사는 사용자 프로그램 실행이지만 커널 내부가 아닌 사용자 공간에서 가장 잘 수행되는 다양한 시스템 작업도 고려되어야 한다. 따라서 시스템은 프로세스 모음, 사용자 코드 실행, 운영체제 코드 실행으로 구성된다. 이 장에서는 프로세스의 개념, 운영 체제에서 프로세스의 표현 방식 및 작동 방식에 대해 설명한다.

## **3.1 Process Concept**

### **3.1.1 The Process**

- 프로세스는 메모리에 실행중인 프로그램이다.
  - 프로그램은 디스크에 저장된 명령 목록을 의미하므로 프로세스와 다르다.
  - 동일한 프로그램을 실행한다고 하더라도 텍스트섹션은 동일하지만 데이터, 힙 및 스택 섹션은 다르다.
  - 프로세스는 코드의 실행환경일 수 있는데 Java 코드는 JVM으로 프로세스로 실행되는 것이 그 예이다.

![Process in Memory](https://user-images.githubusercontent.com/16266103/113502908-ddc96480-9569-11eb-8c03-395b27bbdfbe.png)

>  출처: Operating System Concepts 10th Edition

- 프로세스 메모리 레이아웃은 일반적으로 여러 섹션으로 나뉘며 아래와 같다.
  - Text section : 프로그램 코드
  - Data section : 전역 변수
  - Heap section : 동적으로 할당되는 메모리
  - Stack section : 함수의 매개변수, 복귀 주소와 로컬 변수 등
- 텍스트 및 데이터 섹션의 크기는 고정이지만 스택이나 힙 섹션은 프로그램 실행 중 동적으로 축소 및 증가할 수 있다.
  - 함수를 호출할 때마다 함수 매개 변수, 로컬 변수 및 반환 주소를 포함하는 activation record가 스택에 push된다. 컨트롤이 함수에서 반환되면 activation record가 스택에서 pop 된다.
  - 메모리가 동적으로 할당될 때 힙이 증가하고 메모리가 시스템에 반환되면 힙이 축소된다. 운영체제는 힙 섹션과 스택 섹션이 겹치지 않도록 해준다.

### **3.1.2 Process State**

![Diagram of Process State](https://user-images.githubusercontent.com/16266103/113502910-e1f58200-9569-11eb-8f7b-92f772757083.png)

>  출처: Operating System Concepts 10th Edition

- 프로세스가 실행되면 상태가 변경된다. 프로세스의 상태는 부분적으로 해당 프로세스의 현재 활동에 의해 정의된다.
- 프로세스는 아래의 상태들을 따른다.
  - New : 프로세스 생성
  - Running : 명령어 실행
  - Waiting : 프로세스가 어떤 사건(I/O 등)이 일어나기를 기다림
  - Ready : 프로세스가 처리기에 할당되기를 기다림
  - Terminated : 프로세스 실행 종료
- 운영체제에 따라 상태의 이름이 다를 수 있다. 한 번에 하나의 프로세스만 프로세서의 코어에서 실행할 수 있다는 것을 알아야 한다.

### **3.1.3 Process Control Block**

![Process Control Block (PCB)](https://user-images.githubusercontent.com/16266103/113502912-e6219f80-9569-11eb-90e1-3600f56a6cdf.png)

>  출처: Operating System Concepts 10th Edition

- 각 프로세스는 OS에 의해 PCB(Process Control Block 혹은 Task Control Block)로 표현된다.
- PCB는 아래의 정보들을 가지고 있다.
  - Process state : New, Running, Waiting, Ready, Terminated
  - Program counter : 프로세스가 다음에 실행할 명령어의 주소를 가리킴
  - CPU registers : CPU가 요청을 처리하는 데이터의 임시저장 공간
  - CPU-scheduling information : 프로세스 우선순위, 스케쥴 큐에 대한 포인터 및 다른 스케쥴 Parameter에 관한 정보
  - Memory-management information : Base, Limit Register의 정보 혹은 Segment Table에 관한 정보
  - Accounting information : CPU가 사용된 양과 시간, 시간 제한, 프로세스 번호
  - I/O status information : 해당 프로세스에 할당된 I/O 장치의 리스트 및 Open된 파일에 대한 정보
- PCB는 accounting data와 함께 프로세스를 시작하거나 다시 시작하는데 필요한 데이터의 저장소 역할을 한다.

![CPU Switch From Process to Process](https://user-images.githubusercontent.com/16266103/113502913-e9b52680-9569-11eb-9948-fb572168d98b.png)

>  출처: Operating System Concepts 10th Edition

- 위 그림은 Context Switch에 대해서 설명하고 있다.
- 프로세스가 실행되고 있는 상황에서 interrupt 요청으로 다음 우선 순위의 프로세스가 실행되어야 할 때 기존 프로세스의 상태 또는 레지스터의 값(Context)를 저장하고 CPU가 다음 프로세스를 수행하도록 새로운 프로세서의 상태 또는 레지스터 값(Context)를 교체하는 작업을 Context Switch라고 한다.
- Context는 CPU가 해당 프로세스를 실행하기 위한 정보들이며 PCB에 저장된다.
  - 정보: 프로세스 상태, 프로그램 카운터, 레지스터, 프로세스 번호 등
- Context Switch가 잦아지면 그 순간 CPU는 작업을 못하기 때문에 오버헤드가 발생한다.
- Context Switch가 발생하는 interrupt는 아래와 같다.
  - I/O request
  - time slice expired
  - fork a child
  - wait for an interrupt
  - ...
- Context Switch를 하는 주체는 OS 스케쥴러이다.

### **3.1.4 Threads**

- 프로세스가 실행될 때 1개의 스레드를 포함하고 있다.
- 스레드를 사용하면 하나의 프로세스에서 여러개의 작업을 실행할 수 있다.
- PCB에 프로그램 카운터 및 스레드 정보(메모리 한계, 레지스터 정보 등)를 가지게끔 해야한다.
- 다음 챕터인 4장에서 스레드와 동시성에 대해서 자세하게 다룬다.

## **3.2 Process Scheduling**

**Process Scheduling**은 CPU core에서 다음에 실행할 프로세스를 선택하는 것이다.

**목표는?**

- 최대 CPU 사용률로 process를 실행하는 것.
- 프로세스를 CPU core로 빠르게 전환하는 것.

현재 메모리에 있는 프로세스 수를 **The Degree of Multiprogramming** 라고 한다.

일반적으로 대부분의 프로세스는 I/O bound process와 CPU bound process로 설명된다.

**I/O Bound VS CPU Bound**

- I/O bound process : 연산보다 I/O에 더 시간이 들어가는 프로세스. CPU burst가 작다.
- CPU bound process : 연산에 주로 시간이 들어가는 프로세스. CPU burst가 크다.

> **CPU burst** : 프로세스에서 CPU 사용 구간.

### **3.2.1 Scheduling Queues**

주요 process scheduling queue로 **Ready Queue**와 **Wait Queue**가 있다.

![The ready queue and wait queues.](https://user-images.githubusercontent.com/24274424/114288650-e01d4880-9aac-11eb-9624-ff263682b9aa.png)

>  출처: Operating System Concepts 10th Edition

- Ready Queue
  - 주 메모리에 상주하며 실행 준비, 대기중인 모든 프로세스 집합
- Wait Queue
  -  이벤트를 기다리는 프로세스 세트 (ex. I/O)

프로세스는 다양한 대기열 간에 마이그레이션(migration) 된다.

![Queueing-diagram representation of process scheduling.
](https://user-images.githubusercontent.com/24274424/114288711-e4e1fc80-9aac-11eb-8c16-a15ce03d850a.png)

>  출처: Operating System Concepts 10th Edition

- 프로세스는 I/O Request 후 I/O Wait queue로 이동한다.
- 프로세스는 새로운 자식 프로세스를 생성한 다음 자식의 종료를 기다리는 동안 Wait queue에서 대기한다.
- 프로세스는 Interrupt의 결과로 core에서 강제로 제거되거나 시간 만료된 후 Ready queue에 다시 들어갈 수 있다.

위 두 경우에서 프로세스는 결국 Wait 상태에서 Ready 상태로 전환된 다음 다시 Ready queue로 간다. 프로세스는 종료될 때까지 이 주기는 계속된다.

### **3.2.2 CPU Scheduling**

프로세스는 lifetime 동안 Ready queue와 많은 Wait queue 사이를 옮겨다닌다.

CPU Scheduler 역할은 Ready queue에 있는 프로세스 중 하나를 CPU Core에 할당하는 것이다.

프로세스에게 CPU를 강제로 제거하고 다른 프로세스가 실행되도록 예약할 수 있는 디자인이다. 이러한 디자인으로 CPU scheduler는 100μs마다 실행한다(무조건 정해진 시간이 있는 건 아니다).

#### **Swapping**

Swapping이라는 중간 형태의 스케줄링을 가진다. 핵심 아이디어는 메모리에서 프로세스를 제거하여 **The Degree of Multiprogramming**(메모리상 프로세스의 수)를 줄이는 것이다.

- 프로세스를 메모리에서 디스크로 **swapped out**하여 현재 상태 저장
- 디스크에서 메모리로 **swapped in**하여 상태 복원한다.

Swapping은 일반적으로 메모리가 *오버커밋*되어 해제되어야하는 경우에만 필요한다(추가적으로 9장에서 자세히 설명).

> OverCommitted(오버커밋) : 실제 메모리 총량을 넘는 메모리를 요구를 허용하는 것.

### **3.2.3 Context Switch**

![CPU Switch From Process to Process](https://user-images.githubusercontent.com/16266103/113502913-e9b52680-9569-11eb-9948-fb572168d98b.png)

>  출처: Operating System Concepts 10th Edition

*1.2.1절* Interrupt에서는 OS가 현재 작업에서 CPU core를 바꿔 커널 루틴을 실행하게된다. Interrupt가 발생하면, 실행중인 프로세스와 현재 Context를 저장해 해당 Context를 복원하여 기본적으로 프로세스를 일시정지 이후로 다시 시작한다.

- CPU가 다른 프로세스로 전환 할 때 시스템은 이전 프로세스의 상태를 저장하고 Context 스위치를 통해 새 프로세스에 대해 저장된 상태를 load해야 한다.
- PCB에 프로세스의 Context 표시한다.
  - Context는 PCB에 표시되며 정보들은 [3.1.3 Process Control Block](#3.1.3-Process-Control-Block)에서 확인 가능하다.
- Context Switching 시간은 Pure Overhead다. 전환하는 동안 시스템이 유용한 작업을 하지 않는다.
  - OS와 PCB가 복잡할수록 Context 전환이 길어진다.
- 하드웨어 지원에 따라 시간이 달라진다.
  - 일부 하드웨어는 CPU 당 여러 레지스터 세트(한 번에 로드되는 여러 Context)를 제공한다.
- 운영체제가 복잡할수록 Context Switching 중에 수행해야하는 작업의 양도 많아진다.

### MULTITASKING IN MOBILE SYSTEMS

#### **IOS**

- 모바일 장치에 제약으로 인해 IOS 초기 버전은 사용자 애플리케이션 멀티테스킹을 제공하지 못했다. 다른 모든 사용자 응용 프로그램이 일시중지된 동안 하나만 실행되었다.
- 운영체제 작업은 멀티태스킹이 가능했다.
- IOS 4부터 사용자 애플리케이션에 제한된 형태의 멀티태스킹을 제공하면서 단일 foreground 애플리케이션이 여러 background 애플리케이션과 동시에 실행될 수 있게 되었다.
- 모바일 장치용 하드웨어가 **더 큰 메모리 용량**, **다중 처리 코어** 및 **더 긴 배터리 수명**을 제공하기 시작하면서 IOS의 후속 버전은 더 적은 제한으로 멀티태스킹을 위한 더 풍부한 기능을 지원한다.
- 예를 들어, iPad 태블릿의 더 큰 화면에서는 분할 화면으로 알려진 기술인 두 개의 foreground 앱을 동시에 실행할 수 있다.

#### **Android**

- Android는 예전부터 멀티 태스킹을 지원했으며, background에서 실행할 수 있는 애플리케이션 유형에 제한을 두지 않았다.
- 애플리케이션이 background에서 처리해야하는 경우 애플리케이션은 background 프로세스를 대신하여 service를 사용한다.
- 스트리밍 오디오 응용 프로그램을 생각해보면, 응용 프로그램이 background로 이동하는 경우 service는 background 응용 프로그램을 대신하여 오디오 장치 드라이버에 오디오 데이터를 계속 보낸다.
- 실제로 background 응용 프로그램이 일시 중단된 경우에도 service 계속 실행된다. service에는 사용자 인터페이스가 없고 메모리 사용량이 적으므로 모바일 환경에서 멀티 태스킹을 위한 효율적인 기술을 제공한다.

<details>
<summary> 질문(2021.04.11) </summary>
  
- Pure Overhead에 대해서
- 레지스터가 빨라요? 메모리가 빨라요?
  - 레지스터요.
- 아래의 경우는 다른가
  - 모든 프로세스는 CPU에 올라가 있다가 wait 갔다가 ready를 간다.
- 심각한 오류에 빠지면 어떻게 되나.
  - 리부팅이 될 거 같다.
- 블루스크린
  - 데드락 같은 상태
- Wait queue를 거치지 않는 경우가 있나.
  - 바로 갈 수 있다.
- Swapping
  - 물리적인 메모리가 안 나올때 + 죽이고 싶지 않을 때 디스크로 옮기고 메모리를 확보.
- Swapping 문제가 없나?
  - 속도가 느릴 수 있다.
  - 휘발성 비휘발성 관련 문제는 없다.
  - 디스크 수명에 영향을 준다.
  - 메모리에 압축을 해놓고 프로세스를 실행할 때 압축을 풀어서 사용한다.
</details>

## **3.3 Operations on Processes**

대부분의 시스템에서 프로세스는 동시에 실행될 수 있고, 이들은 동적으로 생성되거나 삭제될 수 있다. 따라서 시스템은 프로세스의 생성, 삭제에 관한 매커니즘을 제공할 수 있어야한다.

### **3.3.1 Process Creation**

- 시스템의 실행 과정에서 프로세스는 여러 개의 새 프로세스를 생성할 수 있다. 부모 프로세스는 자식 프로세스를 생성하게 되고, 이 과정을 거쳐 프로세스 트리를 형성할 수 있다.
- 일반적으로 프로세스는 고유한 번호인 Process Indentifier(PID)를 통해 식별되고 관리된다.
- PID는 [PCB](#3.1.3-Process-Control-Block)에 들어있다.
![fork, exec flow chart](https://user-images.githubusercontent.com/24209005/115135028-44617e80-a050-11eb-9171-3e539e0c8cde.png)

>  출처: Operating System Concepts 10th Edition

- `fork()`시스템 콜을 호출하면 부모 프로세스는 본인과 동일한 자식 프로세스를 생성한다.
- 자식 프로세스는 `exec()`시스템 콜을 통해 본인의 메모리 공간을 새로운 프로그램으로 대체한다.
- 부모 프로세스는 `wait()`시스템 콜을 호출하고 자식 프로세스가 종료될 때까지 기다린다.

해당 그림은 `fork&exec` 방식의 프로세스 생성 방식으로 모든 경우에 대해 제네릭한 경우는 아니다.

- `fork&exec` 방식의 생성 방식: 동일한 문맥이 아니라 다른 문맥의 프로그램을 실행시키고 싶을 때 exec를 실행시키면 문맥이 바뀐다.
  - 부모 프로세스 fork 후, 자식에서 새로운 프로그램 실행
- 윈도우는 부모프로세스에서 자식 프로세스가 실행시킬 프로그램을 미리 결정한다. 즉, 생성 시점부터 결정이 된 상태가 된다.
  - 부모에서 자식에서 실행시킬 프로그램 지정 후, 자식 생성

#### A Tree of Processes in Linux

![Process Tree](https://user-images.githubusercontent.com/24209005/115135057-6529d400-a050-11eb-980c-44008257e38f.png)

>  출처: Operating System Concepts 10th Edition

- 프로세스는 계층 구조를 갖는다는게 중요하다. 즉, 하나의 부모 프로세스가 여러 개의 자식 프로세스를 가진다.

#### C Program Forking Separate Process

```c
#include <sys/types.h>
#include <stdio.h>
#include <unistd.h>

int main()
{
  pid_t pid;

  /* fork a child process */
  pid = fork();

  if (pid < 0) { /* error occurred */
    fprintf(stderr, "Fork Failed");
    return 1;
  }
  else if (pid == 0) { /* child process */
    execlp("/bin/ls", "ls", NULL);
  }
  else { /* parent process */
    /* parent will wait for the child to complete */
    wait(NULL);
    printf("Child Complete");
  }

  return 0;
}
```

>  소스 코드 출처: Operating System Concepts 10th Edition

- `fork()`시스템 콜은 부모 프로세스에겐 자식 프로세스의 PID를, 자식 프로세스에겐 0을 반환한다.
- 그리고 반환된 pid 값을 통해 자식 프로세스의 `exec()`시스템 콜 호출 여부와 부모 프로세스의 `wait()`시스템 콜 호출 여부를 결정한다.
- 부모 프로세스와 자식 프로세스는 동시에 동작한다.

### **3.3.2 Process Termination**

일반적인 프로세스의 종료 과정은 다음과 같다.

- 자식 프로세스는 종료 시에 `exit()`시스템 콜을 호출한다.
- 자식 프로세스에서 부모 프로세스로 상태 데이터와 PID를 반환한다.
- 프로세스의 리소스는 시스템에 의해 할당 해제됩니다.

또 다른 방법으로 부모 프로세스는 `abort()`시스템 콜을 호출하여 자식 프로세스의 실행을 종료할 수 있다. 이와 같이 종료하는 경우는 다음과 같다.

- 자식 프로세스가 할당 된 리소스를 초과했을 때
- 자식 프로세스에게 할당 된 작업이 더 이상 불필요할 때
- 부모 프로세스가 종료되었을 때

부모 프로세스는 `wait()`시스템 콜을 통해 자식 프로세스의 종료를 기다리게 되는데 이 때 종료 된 프로세스는 부모 프로세스에게 상태 정보와 PID를 반환한다. 이 때 다음과 같은 문제가 발생할 수 있다.

#### Zombie Process

![Zombie Process](https://user-images.githubusercontent.com/24209005/115135108-b0dc7d80-a050-11eb-84ef-3f10ab21bd04.png)

좀피 프로세스는 대기중인 부모 프로세스가 없는 경우 `wait()`시스템 콜을 호출하지 않은 경우)에 해당된다. 즉, 부모 프로세스가 자식 프로세스의 종료 상태를 회수하지 않았을 경우 자식 프로세스는 좀비 프로세스가 된다. 부모 프로세스가 좀비 프로세스의 종료상태를 `wait()`시스템 콜을 통해 회수하게 되면 좀비 프로세스는 제거된다.

커널의 입장에서는 부모 프로세스가 자식의 반환값을 알 때 까지 이 정보를 저장해야 할 의무가 있다. 자식 프로세스가 종료된 이후에 부모 프로세스가 자식 프로세스의 상태를 알고 싶을 수 있기 때문에 커널은 자식 프로세스가 종료되더라도 최소한의 정보(프로세스 ID, 프로세스 종료 상태 등)를 가지고 있게 된다. 이게 회수가 안 되면 커널에는 자식 프로세스의 상태 값과 PID를 가진 상태로 남아있는데 이것이 좀비 프로세스라고 보면 된다. 따라서 `wait()`로 자식의 상태 값을 회수하기 전까지는 좀비 프로세스 상태이다.

단, 좀비 프로세스가 존재한다는 것이 반드시 나쁜 것은 아니다. 왜냐하면 프로세스가 종료되면 반드시 좀비 프로세스의 과정을 한 번은 거치게 된다. 그 이유는 [3.1.2 Process State](#3.1.2-Process-State)의 `Running`, `Waiting`, `Ready` 상태의 flow를 보면 이해할 수 있다.

- `Running` 상태의 부모 프로세스에서 `wait()`시스템 콜을 호출하면 부모 프로세스는 `Waiting`상태로 접어든다.
- 자식 프로세스가 죽으면 커널은 `Waiting`상태에 있는 부모 프로세스를 `Ready` 상태로 변환하고 `Ready Queue`에 집어넣는다.
- `Ready Queue`에 있던 부모 프로세스가 `Running`상태로 바뀌기 전까지는 자식 프로세스가 좀비 프로세스일 수 밖에 없다.

#### Orphan Process

![Orphan Process pt 자료](https://user-images.githubusercontent.com/24209005/115135117-c0f45d00-a050-11eb-9beb-3a72cdb6ce7b.png)

`wait()`시스템 콜을 호출하지 않고 부모 프로세스가 종료 된 경우 해당 프로세스는 고아 프로세스가 된다. 즉, 부모 프로세스가 자식 프로세스보다 먼저 종료되면 자식 프로세스는 고아 프로세스가 된다.

부모 프로세스가 자식 프로세스보다 먼저 종료되면 init 프로세스가 자식 프로세스 새로운 부모 프로세스가 된다.
종료되는 프로세스가 발생할 때 커널은 이 프로세스가 누구의 부모 프로세스인지 확인한 후, 커널이 자식 프로세스의 PPID를 init 프로세스의 PID(1)로 바꾼다. 이는 프로세스가 트리 구조를 가지기 때문에 가능하다.

#### 3.3.2.1 Android Process Hierarchy

모바일 환경에 자원 제약이 있기 때문에 모바일 OS는 기존 프로세스를 종료해야 할 수 있다.
이에 따라 안드로이드에서는 프로세스를 우선 순위를 가장 높은 것부터 가장 낮은 것까지 분류하고 있다.

- Foreground Process: 현재 동작되고 있으며 스크린에서 보여지는 프로세스
- Visible Process: 직접적으로 보여지지는 않지만 Foreground Process에 참조하고 있는 프로세스
- Service Process: 백그라운드에서 동작하며 사용자에게 노출되는 프로세스 (ex. 음악 듣기: 백그라운드에 있는데 사용자가 인지할 수 있다)
- Background Process: 백그라운드에서 동작하지만 사용자에게 노출되지 않는 프로세스(ex. 푸쉬 알람 대기)
- Empty Process: 이외의 비활성화된 모든 프로세스

<details>
<summary> 질문(2021.04.18) </summary>
  
- PID는 PCB에 들어있다.
  - 맞다. 리눅스의 경우 C 구조체 안에 들어있다.
- 3.3.1의 fork, exec flow chart 그림은 모든 경우에 대한 제네릭한 경우는 아니다.
  - fork/exec 방식의 생성 방식: 동일한 문맥이 아니라 다른 문맥의 프로그램을 실행시키고 싶을 때 exec를 실행시키면 문맥이 바뀐다.(부모 -> 자식에 복사, 자식에서 새로운 프로그램 실행)
  - 윈도우는 부모프로세스에서 자식 프로세스가 실행시킬 프로그램을 미리 결정한다. 즉, 생성 시점부터 결정이 된 상태가 된다.(부모에서 자식에서 실행시킬 프로그램 지정 -> 자식 생성)
- 트리 그림 중요한가?
  - 계층 구조를 갖는다는게 중요(하나의 부모와 여러개의 자식 프로세스)
  - 고아프로세스 이해에 도움이 된다(고아가 된 후에 부모 프로세스 PID=1로 바꿔주는 것이기 때문에)
- wait를 하지 않았을 경우 좀비프로세스가 되는것인가
  - 커널의 입장에서는 부모 프로세스가 자식의 반환값을 알 때 까지 이 정보를 저장해야 할 의무가 있다. 자식이 어떻게 죽었는지 알아야 하기 때문에(부모가 자식의 상태를 회수 할 때 까지 커널에 저장된다. 이게 회수가 안 되면 커널에 찌꺼기 상태로 남아있는 좀비 프로세스가 된다)
  - 따라서 wait로 자식의 status를 회수하기 전까지는 좀비 프로세스 상태이다.
  - 좀비가 존재한다는 것이 반드시 나쁜 것은 아니다.
    - 반드시 좀비가 한 번은 된다.
      - 러닝에서 웨이트를 호출하면 부모는 웨이팅 상태
      - 자식이 죽으면 커널은 웨이팅 상태에 있는 부모 프로세스를 레디 상태로 변환, 레디 큐에 집어넣음
      - 레디큐에 있다가 러닝 상태로 바뀔때까지는 자식 프로세스가 좀비일 수 밖에 없다.
- 서비스 프로세스와 백그라운드 프로세스의 차이?
  - 서비스 : 음악 듣기(백그라운드에 있는데 사용자가 인지할 수 있다)
  - 백그라운드 : 푸쉬 알람 대기
- 모바일에서 프로세스 관리의 핵심은 자원 제약이 있을 경우 우선순위가 낮은 것을 제거해가며 자원을 확보하는 방식을 사용한다는 것

</details>

### Multiprocess Architecture(Chrome Browser)

전통적인 브라우저들은 싱글 프로세스로 작동됐다. 하지만 탭 기능이 생기며 문제가 발생했는데, 만약 웹 사이트에 문제가 생기면 브라우저가 모두 닫히거나 망가졌다.

>- 멀티 스레드의 약점-> 하나가 고장나면 전체 프로세스에 영향을 끼침
>- 장점은 메모리의 데이터와  힙 영역을 직접 공유하기 때문에 IPC가 필요없다.

![크롬예시](https://user-images.githubusercontent.com/24666330/116804033-9af1b100-ab56-11eb-80f9-1a2a11bee81f.png)

크롬은 3가지 다른 타입의 프로세스로 나뉜다.

- 브라우저 : ui, I/O(디스크, 네트워크)를 담당하는 프로세스. 크롬이 실행되면 생성되고, 단 하나만 존재한다.
- 렌더러(Renderer) : HTML과 자바스크립트로 웹 페이지를 보여준다. 새로운 탭에서 웹 사이트가 열릴 때마다 새로운 렌더러가 생성된다.
  - 보안을 위해 디스크와 네트워크 I/O작업을 제한시킨 샌드박스 위에서 돌아간다.
- 플러그인 : 말 그대로 플러그인이다.

멀티 프로세스의 장점은 각 프로세스가 독립적이기 때문에 서로에게 영향을 끼치기 어렵다. 특히 각 탭마다 새로운 프로세스가 생기기 때문에 각 탭에서 열린 웹 사이트들은 서로 격리(isolation)되어있다.

> [크롬 소개글(출시 당시)](https://peaceharmony.tistory.com/entry/%EA%B5%AC%EA%B8%80%EC%9D%98-%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80-%ED%81%AC%EB%A1%ACGoogle-Chrome%EC%9D%84-%EC%86%8C%EA%B0%9C%ED%95%A9%EB%8B%88%EB%8B%A4)

## 3.4 Interprocess Communication

시스템 내부의 프로세스는 독립적(independent)으로 동작하지만, 협력(cooperating)할 수도 있다.

책에서는 데이터 공유를 기준으로 협력을 구분한다. 만약 서로 공유하는 데이터가 없으면 해당 프로세스들은 서로 독립적이다. 반면, 서로 데이터를 공유하는 프로세스들은 협력하고 있는 것이다.

협력하는 이유는 아래와 같다.

- 정보 공유 : 여러 어플리케이션이 같은 정보를 원할 수 있다. 예를 들어, 복사 붙여넣기 같은 경우 여러 프로세스에서 동시에(concurrent) 같은 데이터가 사용되어야 한다.
- 속도 향상 : 특정 작업(task)의 속도를 빠르게 하기 위해 하위 작업(sub task)을 나눌 수 있다. 병렬적(parallel)으로 실행할 수 있기 때문에 멀티 코어 환경에서 속도가 향상된다.

> 동시성(concurrency) : 여러개의 작업이 함께 실행되는 것이다.
>
> 병렬성(parallelism) : 동시간에 여러개의 작업이 실행되는 것이다.
>
> 동시성의 함께라는 말 때문에 헷갈릴 수 있는데 함께 실행된다고 해서 같은 시점에 실행되어야 하는 것이 아니다. 여러개의 작업을 실행시키고 CPU가 각 작업을 돌아가면서 처리한다면 이는 동시에 실행된다고 할 수 있는 것이다. 즉, 싱글 프로세스에서 멀티 스레드를 구현하기 위한 논리적인 개념에 가깝다. 자세한 설명은 4.2에 나온다.

- 모듈화 : 2장에서 언급되었다고 한다(찾아봐야함).

> 협력 하는 이유 중 편리성(Convenience)이 9판과 PPT 자료에는 남아있지만, 10판에는 삭제되었다. 사용자 편의성이라 이해하면 된다.

협력을 위해서는 IPC(Interprocess Communication)가 필요하다.

> 위에서, 프로세스간 협력은 데이터를 공유하는 것이라 했다. 하지만, 프로세스에 접근하는 것은 커널영역에서만 가능하다. 즉, 프로세스끼리 직접 서로의 데이터에 접근할 수 없다. 따라서 다른 방법이 필요한데, 이 방법이 IPC다.

IPC에는 크게 두 가지 모델이 있다.

> 두 가지 모델 모두 많이 사용된다고 한다.

![IPC모델](https://user-images.githubusercontent.com/24666330/116804039-a93fcd00-ab56-11eb-9a9c-5342cfb1c18b.png)
> 그림에 나온 메세지 큐는 IPC기법이 아닌, 논리적인 자료구조를 말하는 것이다.

- 공유 메모리(Shared memory)

  - 프로세스끼리 공유하는 메모리 영역을 설정

  - 속도가 빠르다. 메세지 패싱은 일반적으로 시스템 콜을 이용해서 구현되기 때문에 시간이 더 소요되기 때문이다.
    
    > 시스템 콜을 사용하지 않는 것은 아니지만, 공유 메모리 영역 설정에 필요한 것이다. 이후에는 커널의 도움이 필요없다.

- 메세지 전달(Message passing)

  - 서로 메세지를 주고 받음

  - 데이터 충돌(conflict)이 일어나지 않기 때문에 작은 데이터 전달에 적합하고 구현하기도 쉽다.
    
    > distributed shared memory를 지원하면 충돌을 고려하지 않아도 된다고 한다.

자세한 것은 3.5와 3.6장에서 다룬다.

## 3.5 IPC in Shared-Memory Systems

공유 메모리를 이용한 IPC는 공유할 영역 설정이 필요하다. 보통 주소 공간(address space)에 공유 메모리 영역(region)을 다루는 부분(segment)이 따로 있다고 한다. 만약 다른 프로세스가 공유메모리의 데이터를 사용하고 싶다면 해당 주소 공간에 연결(attach)돼야 한다.

> 여기서 segment는 구분지어진 영역을 나타내는 것이다(예를 들어 힙 영역을 heap segment라고 부르기도 한다).
>
> 리눅스나 유닉스 시스템에서는 모든 것을 파일로 생각한다. 공유 메모리도 한 프로세스가 공유 메모리를 생성하면 파일 형태로 생성이 되는데, 공유메모리로 인식할 수 있는 특수한 파일로 만들어질 것이다. **OS에서는 이런 종류의 파일을 모아두는 위치가 따로 있는데, 이런 부분을 말하는 것이다.**
>
> 리눅스, 유닉스 설정을 보면 공유 메모리 크기 설정을 할 수가 있는데, 이게 그 방증이다.

기본적으로 OS는 프로세스가 다른 프로세스의 메모리에 접근하는 것을 막는다. 두개 이상의 프로세스가 이 제약을 깨기로 동의해야 공유 메모리 사용이 가능하다. 공유 메모리를 사용하면 공유된 영역을 이용해서 정보를 교환할 수 있다. 데이터의 형태와 위치를 프로세스가 직접 결정하기 때문에 OS가 제어하지 않는다. 따라서 같은 위치에 동시에 쓰기 작업(writing)을 하지 않도록 보장하는 것도 프로세스의 책임이다. 즉 동기화(Synchronization)가 필요하다.

### Producer-Consumer Problem

프로세스 협력의 대표적인 패러다임으로 Producer-Consumer Problem이 있다. producer 프로세스는 consumer에서 사용할 정보를 제공한다.

> 컴파일러가 어샘블러에서 읽을 어샘블리 코드를 생성하고, 어샘블러가 로더에서 사용할 객체 모듈을 생성하는 것을 예로 들 수 있다.

producer-consumer는 client-server로 바꿔서 생각해볼 수 있다. 서버를 producer, 클라이언트를 consumer라고 생각해볼 수 있다. 예를 들어, 웹 서버가 정적 파일을 클라이언트로 보내주면 클라이언트가 이를 렌더링한다.

공유 메모리는 Producer-Consumer Problem을 해결하는 방법 중 하나다. producer와 consumer를 동시에 실행시키기 위해서 공유 메모리를 버퍼로 사용하는데, producer가 버퍼를 채우고 consumer가 버퍼를 비우는 방식이다.

consumer가 버퍼에 들어있는 다른 내용을 꺼내 쓰는 동안 producer가 쓰기 작업할 수 있다. 아직 완전히 쓰여지지 않은 항목을 꺼내면 안 되기 때문에, 반드시 동기화되어야 한다.

두 가지 버퍼를 사용할 수 있다.

- unbounded-buffer : 사이즈가 고정되지 않음. producer는 계속 쓸 수 있지만, consumer는 버퍼에 아무 것도 없으면 대기상태가 된다.

    > unbounded-buffer를 실제로 사용하는 모델이 흔치 않다. 만약 사용한다면, 커널이 개입할 여지가 생길 수 있다. 
    >
    > 참고로 POSIX 표준에는 버퍼 사이즈가 정해져있다.

- bounded-buffer : 사이즈가 고정된다. producer는 버퍼가 가득 차면 대기해야 하고, consumer는 버퍼가 비었으면 대기해야 한다.

#### bounded buffer로 구현 예시

> 아래 예시는 동기화가 고려되지 않았다. 이는 6장과 7장에서 마저 다룬다고 한다.

```c
#define BUFFER_SIZE 10
typedef struct {
 . . .
} item;

item buffer[BUFFER_SIZE];
int in = 0;
int out = 0;
```

`in == out`(in 만큼 out되면)이면 버퍼가 비어있는 것이다. 또한, `((in + 1) % BUFFER SIZE) == out`이면 버퍼가 가득 찬 것이다.

> - 서큘러 형식으로 구현되어있기 때문에 in과 out이 만나는 부분을 시작점으로 잡아야 편할 것이다.
> - 또한 이러한 구현에서 in과 out 처리를 따로 해주지 않으려면 Buffer - 1까지만 사용할 수 있다.

```c
// Producer
item next_produced;
while (true) { 
    /* produce an item in next produced */ 
    while (((in + 1) % BUFFER_SIZE) == out) {
        // 버퍼가 가득 차있으면 빈 while문을 돌려 대기상태로 만든다.
    }
    buffer[in] = next_produced; 
    in = (in + 1) % BUFFER_SIZE; 
} 
```

```c
// Consumer
item next_consumed;
while (true) {
    while (in == out) {
        // 버퍼가 비었으면 대기상태
    }
    next_consumed = buffer[out]; 
    out = (out + 1) % BUFFER_SIZE;
    /* consume the item in next consumed */ 
} 
```

---

질문

- 애플의 여러 기기간 복사-붙여넣기 기능은 공유 메모리인가?
    - 클라우드 방식을 사용할 것이다.
    - 이러한 방식은 커널 내부에서 공유되는 영역을 사용하는 것이 아니기 때문에 공유 메모리 방식이라 할 수 없다.


## 3.6 IPC in Message-Passing Systems

이번 챕터에서는 프로세스 간 통신 방법에 대해서 알아본다.

`3.5 IPC in Shared-Memory Systems` 을 통해 shared-memory 환경에서 cooperating 프로세스가 어떻게 소통할 수 있는지 확인했다. 이 scheme은 이러한 프로세스들이 메모리 영역을 공유하도록 요구하고 shared-memory에 접근 및 조작하기 위한 코드는 애플리케이션 프로그래머에 의해 명시적으로 작성되어야 한다. 동일한 효과를 달성하는 또 다른 방법은 운영 체제가 메시지 전달 기능을 통해 서로 통신하는 프로세스를 협력할 수 있는 수단을 제공하는 것이다.

```c
item next consumed;
while (true) {
	while (in == out)
		; /* do nothing */
	next consumed = buffer[out];
	out = (out + 1) % BUFFER SIZE;
	/* consume the item in next consumed */
}
```

> Figure 3.13 The consumer process using shared memory.

메시지 전달 기능은 적어도 두 가지 operation을 제공한다.

- send(message)
- receiver(message)

link와 send() / receive() 작업을 논리적으로 구현하는 몇 가지 방법은 아래와 같다.

- Direct or indirect communication
- Synchronous or asynchronous communication
- 자동 혹은 명시적 buffering

위와 관련된 이슈들을 아래에서 살펴보게 될 것이다.

### **3.6.1 Naming**

프로세스가 통신하려면 반드시 서로를 알아볼 수 있는(특정 프로세스를 한정 지을 수 있는) 수단이 필요하다. direct 혹은 indirect 통신 방법이 존재한다.

**direct communication**

통신할 프로세스들은 서로의 이름에 대해 명확히 알아야 한다. (sender, recipient 모두)

- send(P, message) : Send a message to process P
- receive(Q, message) : Receive a message from process Q

이 scheme에서 send()와 receive()는 아래와 같이 정의된다.

- 통신을 원하는 두 프로세스 간 이 link는 자동으로 생성됨
- link는 정확히 두 프로세스와 연관되어 있음
- 다시 말해, 두 프로세스 간에는 정확히 하나의 링크만 존재함

**symmetry, asymmetry**

- symmetry : 발신자 프로세스와 수신자 프로세스 모두 통신하기 위해 다른 프로세스의 이름을 지정해야 함
- asymmetry : 여기서는 발신인 이름만 수신인으로 지정하고 수신인은 발신인의 이름을 지정할 필요가 없음

이 scheme에서 send()와 receive()는 아래와 같이 정의된다.

- send(P, message) : Send a message to process P
- receive(id, message) : 모든 프로세스로부터 메시지를 수신합니다. 변수 ID는 통신이 수행된 프로세스의 이름으로 설정
  - 여기서 id는 통신이 발생하는 프로세스 set을 말함
- 단점 (symmetric, asymmetric 모두)
  - 프로세스를 특정하기 때문에 모듈성이 떨어지게 됨
  - pid를 바꾸면 새로 바뀐 pid를 찾기 위해 old id를 모두 찾을 수밖에 없음

**Indirect communication**

- indirect communication : 통신 프로세스들이 **mailbox**나 **포트**를 통해 메시지를 주고받는 것
- mailbox : message를 넣거나 뺄 수 있는 object이며 추상적으로 볼 수 있음
  - 각 mailbox는 고유 id를 가짐
  - ex: POSIX message queues

이 scheme에서 send()와 receive()는 아래와 같이 정의된다.

- send(A, message) : Send a message to mailbox A
- receive(A, message) : Receive a message from mailbox A

이 scheme의 communication link에는 다음과 같은 속성이 있다.

- 공유된 mailbox를 가지고 있어야 두 프로세스 간 링크가 성립됨
- 두 프로세스가 통신하기 위해 하나 이상의 mailbox를 이용할 수 있음
- link는 두 프로세스 이상과 관련되어 있을 수 있음 (즉, 한 mailbox는 2개 이상의 프로세스가 통신하기 위해 존재)

이제 P1, P2 및 P3 프로세스가 모두 mailbox A를 공유한다고 가정해보자. 프로세스 P1은 A에게 메시지를 보내고, P2와 P3는 모두 A로부터 receive()을 실행한다. 어느 프로세스가 P1에서 보낸 메시지를 수신할 것인가? 답은 다음 중 어떤 방법을 선택하느냐에 따라 달라진다.

- 한 mailbox 3개 이상의 프로세스가 접근을 허용함으로써 생긴 문제
- 이 답은 어떤 방법을 선택할 것인지에 따라 결과가 달라짐
  - link(mailbox)에 2개 프로세스만 연관되도록 함
  - 한 번에 한 개의 프로세스만 receive()를 허용
  - System이 어떤 프로세스가 선택하게 할 것인지 결정함 따라서 이때 누가 선택할 것인지 알고리즘을 정해야 하고 System이 sender에 대한 receiver를 확인할 수 있어야 함

**mailbox 소유자(mailbox가 누구의 memory address에 속하는지)**

- Process의 경우
  - 프로세스 통신 간 고유의 mailbox를 소유하기 때문에 mailbox의 소유자를 알 수 있으므로 혼동이 없음
  - 하지만 프로세스가 삭제될 때 mailbox도 삭제되므로 mailbox가 삭제되었는지 알림 받아야 함
  - mailbox를 소유하는 프로세스가 종료되면 mailbox는 사라짐 따라서 이후에 이 mailbox로 메시지를 보내는 모든 프로세스에는 mailbox가 더 이상 존재하지 않음을 알려야 함

- Operating System의 경우
  - 독립적으로 존재하게 된다. 즉, 어떤 프로세스에도 특정하게 attach 되지 않음
  - 이에 따라 OS는 반드시 다음과 같은 절차를 밟도록 해야 함
    - Create a new mailbox.
    - Send and receive messages through the mailbox
    - Delete a mailbox
  - 새로운 mailbox를 만든 프로세스가 mailbox의 default 소유자가 됨 (초깃값)
  - 하지만, ownership과 메시지를 받는 권한은 적절한 시스템 콜을 통해 다른 프로세스에 전달될 수 있다. 이를 통해 각 mailbox 다중 receiver가 존재할 수 있게 됨

### **3.6.2 Synchronization**

메시지를 읽는 중에 데이터가 업데이트될 수 있는 상황이라고 해보자. synchronization 문제가 발생하게 된다.

프로세스 간 통신은 send()와 receive() 원시형을 통해 이루어진다. 구현을 위한 다양한 설계 옵션이 있다. 메시지 전달은 blocking 혹은 nonblocking(동기 및 비동기라고도 한다)일 수 있다. (관계의 동기, 비동기 개념은 다양한 운영체제 알고리즘에 적용됨)

**Blocking send**

receiving process 나 mailbox가 데이터를 받는 중에는 sending process가 block 된다.

**Nonblocking send**

sending process가 메시지를 전송하고 작업을 다시 시작한다.

**Blocking receive**

message를 읽을 수 있을 때까지 receiver를 block 한다.

**Nonblocking receive**

수신자가 유효한 메시지 또는 null을 검색한다.

```c
message next produced;
while (true) {
	/* produce an item in next produced */
	send(next produced);
}
```

> Figure 3.14 The producer process using message passing

send()와 receive()의 blocking을 통해 생산자-소비자 문제 해결할 수 있다.

- 생산자가 blocking send()를 사용하면 receiver와 mailbox에 도착할 때까지 대기 상태가 됨
- 소비자는 receive()를 쓰더라도 blocking send()에 의해 block 된다. available 한 경우 block 해제하여 읽을 수 있음

### **3.6.3 Buffering**

direct 혹은 indirect로 통신할 때 메시지는 프로세스에 있는 임시 queue와 통신한다. 기본적으로 이러한 queue는 아래 3가지 방법으로 구현한다.

**Zero capacity**

큐의 최대 길이가 0이기 때문에 link에 대기 중인 메시지가 있을 수 없다. 이런 경우 sender는 수신인이 메시지를 수신할 때까지 block 해야 한다.

**Bounded capacity**

queue의 길이가 유한하다. 따라서 최대 n개의 메시지를 queue에 넣어 둘 수 있다. sender가 메시지를 보낼 때 queue가 비어 있으면 queue에 메시지가 보관되며, sender는 대기하지 않고 데이터를 보낼 수 있게 된다.

하지만 link의 용량이 유한하기 때문에 link가 꽉 차면 sender는 반드시 빈 공간이 생길 때까지 block 처리되어야 한다.

```c
message next consumed;
while (true) {
	receive(next consumed);
	/* consume the item in next consumed */
}
```

> Figure 3.15 The consumer process using message passing.

**Unbounded capacity**

queue의 길이가 무한하기 때문에 sender는 block 되지 않는다.

<details>
<summary> 질문(2021.05.02) </summary>

- 3.6장의 내용은 컨셉인가? conceptual 한가?
 - unbounded capacity를 보면 이건 conceptual 한 것이다.
 - 그렇지만 일부는 conceptual 한 게 아니라 구현이 되어있다.
- 각 구현에 대해 장단점 및 패러다임이 있는가?
 - 응용 단에서 IPC를 사용할 때 구현한다기보다는 어떤 성격을 갖고 있는가가 중요하다. 따라서 적절하게 골라 써야 한다.
 - message queue, pipe 등의 IPC는 각각 다른 성격, 특성을 가지고 있으므로 구현하는 시나리오에 맞게 선택해야 한다.
 - IPC에 뭐가 있는지 찾아보고 구현하면 된다.
- 3.6장이 IPC 구현이랑 매칭되는가?
 - 섞이는 느낌이다. 와닿지 않고 conceptual 한 부분이 있다.

</details>

## Examples of IPC Systems

### 3.7.4 Pipes

Pipe란 두 프로세스가 통신할 수 있는 통로(도관)다.

초기 UNIX 시스템에서 초기 IPC 메커니즘 중 하나다. 몇 가지 제한사항이 있지만, 프로세스가 서로 통신할 수 있는 간단한 방법을 제공한다. 

Pipe를 구현할 때는 4가지를 고려해야한다.

- 양방향 통신을 허용해야 하나, 단방향 통신으로 하나?
- 양방향 통신이 허용되는 경우 반이중(데이터는 한 번에 한 방향으로만 이동할 수 있음) 또는 전이중(데이터가 동시에 양방향으로 이동할 수 있음)인가?
- 통신 프로세스간에 관계 (예 : 부모-자식)가 존재해야하나?
- 네트워크를 통해 통신 할 수 있나? 아니면 통신 프로세스가 동일한 시스템에 있어야 하나?

#### 3.7.4.1 Ordinary Pipes

- 표준 producer–consumer 방식으로 통신한다. 
- Producer는 Pipe의 한쪽 끝(쓰는 쪽)에서 쓰고, Consumer는 다른 끝(읽는 쪽)에서 읽는다.
- 단방향 통신만 허용하며, 양방향 통신이 필요한 경우 서로 다른 방향으로 데이터를 전송하는 두 개의 Pipes를 사용한다. 

$$
pipe(int fd[])
$$

UNIX 시스템에서 이 함수를 사용하여 구성된다.

이 함수는 `int fd []` 파일 서술자에 액세스되는 Pipe를 만든다. `fd [0]` 은 Pipe의 읽는 쪽이고 `fd [1]`는 쓰는 쪽이다. UNIX는 Pipe를 특수한 유형의 파일로 취급하며, `read ()` 및 `write ()` 시스템 호출을 사용하여 Pipe에 접근할 수 있다.

> **파일 서술자(file descriptor)**: 컴퓨터 프로그래밍 분야에서 파일 서술자(file descriptor) 또는 파일 기술자는 특정한 파일에 접근하기 위한 추상적인 키이다. 이 용어는 일반적으로 POSIX 운영 체제에 쓰인다.

Ordinary Pipe는 생성한 프로세스 외부에서 액세스할 수 없다. 일반적으로 부모 프로세스는 Pipe를 생성하고, `fork ()`를 통해 자식 프로세스와 통신하는 Pipe를 만들어 통신한다.

그림 3.20은 fd 배열의 파일 서술자와 부모 및 자식 프로세스의 관계를 보여준다. 이 그림에서 알 수 있듯이 부모가 Pipe의 쓰기 끝 (fd [1])에 쓰는 모든 쓰기는 Pipe의 읽기 끝 (fd [0])에서 자식이 읽을 수 있다.

<img src="https://user-images.githubusercontent.com/24274424/117546193-32c63200-b064-11eb-9dc7-7678ffbb6dc5.png" alt="3-20">

작성자가 파일의 끝을 닫았을 때 Pipe에서 읽는 프로세스가 파일 끝 (read () return 0)을 감지 할 수 있는지 확인하는 것은 Pipe의 중요한 단계다.

Windows 시스템의 Ordinary pipes를 Anonymous Pipes라고 하며 UNIX와 유사하게 작동한다. 

단방향이며 통신 프로세스간에 부모-자식 관계를 사용한다.

그림 3.23은 자식과 통신하기 위해 Anonymous Pipes를 만드는 부모 프로세스를 보여준다.

<img src="https://user-images.githubusercontent.com/24274424/117546189-31950500-b064-11eb-958f-96547c5e810f.png" alt="3-23">

Ordinary pipes에는 UNIX 및 Windows 시스템 모두에서 통신 프로세스간에 상위-하위 관계가 필요한다. 즉, Ordinary Pipe는 동일한 시스템의 프로세스 간 통신에만 사용할 수 있다.

#### 3.7.4.2 Named Pipes

Ordinary pipes는 한 쌍의 프로세스가 통신할 수 있도록 하는 간단한 메커니즘을 제공한다. 그러나 프로세스가 서로 통신하는 동안에만 존재한다. UNIX 및 Windows 시스템 모두에서 프로세스가 통신을 마치고 종료되면 더 이상 존재하지 않는다.

Named pipes는 훨씬 더 강력한 통신 도구를 제공한다.

- 통신은 양방향일 수 있으며, 부모-자녀 관계가 필요하지 않는다. 
- 여러 프로세스 사이를 통신할 수 있다. 
- 실제로 일반적인 시나리오에서 Named pipes에는 여러 작성자(Writer)가 있으며, Named pipes는 통신 프로세스가 완료된 후에도 계속 존재한다. 

파일 시스템에서 삭제될 때까지 계속 존재한다. FIFO는 양방향 통신을 허용하고, 반이중 전송만 허용된다. 데이터가 양방향으로 이동해야하는 경우 일반적으로 두 개의 FIFO가 사용되야하며, 통신 프로세스는 동일한 시스템에 있어야 한다. 기계간 통신이 필요한 경우 소켓 (섹션 3.8.1)을 사용해야 한다.

Windows 시스템의 Named pipes는 UNIX에 비해 더 풍부한 통신 메커니즘을 제공한다. 

전이중 통신이 허용되며 통신 프로세스는 동일하거나 다른 시스템에 상주할 수 있다. UNIX FIFO는 byte-oriented 데이터만 전송할 수 있는 반면, Windows 시스템은 byte-oriented 또는 message-oriented 데이터를 허용한다.

<details>
<summary> 질문(2021.05.16) </summary>

- ordinary pipe의 pipe 함수에 대한 자세한 설명해줄 수 있는지?
  - 읽기를 위한 descriptor하나와 쓰기를 위한 descriptor하나로 구성
  - fd는 처음 open된 file의 id값이라고 보면 된다.
  - 처음 생성될 때는 0, 1번이 생성되고 그 이후 open되는 채널들은 3, 4... 이런식으로 생성됨 (id값이기 때문에 unique 하도록)
- ordinary pipe의 그림 이해가 잘 안된다. 파이프가 2개가 있어야하는거 아닌가?
  - 처음 생성될 때는 양방향으로 구성이 되지만 실제로 구현을 하다보면 연결을 끊고 단방향으로 바꾼 뒤 사용
- named pipe에서 양방향이 된다고 읽었는데 반이중 전송만 허용한다는게 무슨 의미인지?
  - 전이중을 하기 위해서는 결국 2개가 필요
</details>

## 3.8 Communication in Client–Server Systems

[3.4-Interprocess-Communication](#3.4-Interprocess-Communication)에서 프로세스간에 `Shared Memeory`와 `Message Passing`방법을 이용해 통신하는 방법을 살펴봤다. 이 기술들은 `client-server` 시스템에서 통신하는 방법으로도 사용될 수 있다. 이번에는 `client-server` 시스템에서 통신하는 다른 두 가지 방법인 `Sockets`과 `Remote Procedure Call(RPC)`를 살펴볼 것이다.

### 3.8.1 Sockets

소켓은 통신을 위한 Endpoint라고 정의할 수 있다. 한 쌍의 프로세스는 각 프로세스에 하나씩 한 쌍의 소켓을 사용하여 통신하며 포트 번호와 연결된 IP 주소로 구성되어 있다. 일반적으로 소켓은 `client-server` 아키텍처를 사용한다.서버는 지정된 포트를 수신하여 들어오는 클라이언트 요청을 기다리고 요청이 수신되면 서버는 클라이언트 소켓의 연결을 수락하여 연결을 완료한다.

클라이언트 프로세스가 연결 요청을 시작하면 호스트 컴퓨터에 의해 1024보타 큰 임의의 숫자로 포트가 할당된다.

![Sockets](https://user-images.githubusercontent.com/24209005/118385392-621f0500-b649-11eb-875c-09d5ed68bbc3.png)

예를 들어, IP 주소가 `146.86.5.20` 인 호스트 X의 클라이언트가 주소 `161.25.19.8`의 웹 서버 와 연결을 설정하려는 경우 호스트 X 주소에 포트 1625로 할당 될 수 있다.

각 소켓의 연결은 고유해야하기 한다. 이에 따라 호스트 X의 다른 프로세스가 동일한 웹 서버와 다른 연결을 설정하려는 경우 1024보다 크고 1625가 아닌 포트 번호가 할당된다.

#### Sockets in Java

Java에는 3가지 유형의 Sockets이 있다.

- 연결형(TCP)
- 비연결형(UDP)
- MulticastSocket Class: 데이터를 여러 수신자에게 보낼 수 있다.

```java
// Server
import java.net.*;
import java.io.*;

public class DateServer {
  public static void main(String[] args) {
    try {
      ServerSocket sock = new ServerSocket(6013);
      /* now listen for connections */
      while (true) {
        Socket client = sock.accept();
        PrintWriter pout = new PrintWriter(client.getOutputStream(), true);
        /* write the Date to the socket */
        pout.println(new java.util.Date().toString());
        /* close the socket and resume */
        /* listening for connections */
        client.close();
      }
    } catch (IOException ioe) {
      System.err.println(ioe);
    }
  }
}
```

```java
// Client
import java.net.*;
import java.io.*;

public class DateClient {
  public static void main(String[] args) {
    try {
      /* make connection to server socket */
      Socket sock = new Socket("127.0.0.1", 6013);
      InputStream in = sock.getInputStream();
      BufferedReader bin = new BufferedReader(new InputStreamReader(in));
      /* read the date from the socket */
      String line;
      while ((line = bin.readLine()) != null) {
        System.out.println(line);
      }
      /* close the socket connection*/
      sock.close();
    } catch (IOException ioe) {
      System.err.println(ioe);
    }
  }
}
```

1. 서버 프로세스는 `println()` 메서드를 호출하여 클라이언트에 날짜를 보낸다.
2. 소켓에 날짜를 기록하면 서버는 클라이언트에 대한 소켓을 닫고 더 많은 요청을 다시 수신한다.
3. 클라이언트는 소켓을 만들고 서버가 수신하는 포트에 연결하여 서버와 통신한다.
4. 클라이언트는 소켓을 만들고 포트 6013의 IP 주소 `127.0.0.1`에서 서버와의 연결을 요청한다.
5. 일단 연결되면 클라이언트는 일반 스트림 I/O 문을 사용하여 소켓에서 읽을 수 있다.
6. 서버로부터 날짜를받은 후 클라이언트는 소켓을 닫고 종료한다.

- IP 주소 `127.0.0.1`은 루프백으로 알려진 특수 IP 주소로 컴퓨터가 자기 자신을 참조하는 IP이다.
- 이 메커니즘을 사용하면 동일한 호스트의 클라이언트와 서버가 TCP/IP 프로토콜을 사용하여 통신 할 수 있다.
- `127.0.0.1`은 날짜 서버를 실행하는 다른 호스트의 IP 주소 혹은 실제 도메인 명으로도 대체될 수 있다.

소켓을 사용하는 통신은 일반적이고 효율적이지만 분산 프로세스 간의 저수준 통신으로 간주된다. 그 이유는 소켓이 통신 스레드간에 구조화되지 않은 바이트 스트림을 교환하는 것만 허용하기 때문이다. 다음 섹션에서는 조금 더 수준 높은 통신 방법인 `Remote Procedure Calls(RPC)`를 살펴보도록 하겠다.


<details>
<summary> 질문(2021.05.16) </summary>

- 소켓은 low level이라는건?
  - 소켓을 통해 직접적으로 통신을 한다기 보단 실제로는 소켓을 wrapping한 무언가를 통해 통신 => 인터페이스가 있지 않다? 바이트 스트림만을 공유한다?
  - 웹 소켓을 생각해보면 결국 기본적인 소켓을 감싸는 형태

</details>

### 3.8.2 Remote Procedure Calls

> MSA나 분산 환경에서 많이 사용됨

RPC는 네트워크로 연결 된 서비스끼리 프로시저를 호출할 때 사용할 수 있도록 추상화시킨 원격 서비스다. 프로세스가 별도의 시스템에서 실행되는 환경이기 때문에 메세지 기반 통신을 사용해야 한다.

IPC 메세지와 달리 RPC 통신에서는 단순히 패킷화 된 데이터를 주고받는 것이 아니다. 각 메세지를 특정 포트를 수신하는 RPC 데몬으로 보내고, 각 메세지에는 실행할 기능(function)을 특정하는 구분자와 매개변수가 들어있다. 메세지의 요청에 따라 기능이 실행되고 해당 기능의 출력이 요청자에게 다시 전송된다.

시스템은 보통 하나의 네트워크 주소를 가지기 때문에 포트 번호로 서비스를 구분한다. 예를 들어, 내가 원하는 서버의 기능이 3027 포트에 접근하면 실행되도록 되어있다면 해당 포트로 메세지를 보내야한다.

RPC의 semantics(어떻게 번역해야할지?)는 로컬에서 프로시저를 호출하는 것 처럼 원격에 있는 프로시저를 사용할 수 있도록 한다. RPC 시스템은 stub이라는 것을 이용해 세부 사항을 감춘다.

- 클라이언트의 stub은 서버의 프로시저를 찾고, 파라미터를 마샬링하여 전송하는 역할을 한다.

- 서버의 stub은 메세지를 수신하고 파라미터를 디마샬링 한다. 그 후 해당하는 프로시저를 실행시킨다. 만약 반환 값이 필요하다면 같은 방법으로 클라이언트에 되돌려 보낸다.

    > 마샬링 : 객체를 통신에 적합한 상태로 만들고 전달하는 것. 참고 -[직렬화와 마샬링](https://m.blog.naver.com/PostView.naver?blogId=wonggss&logNo=220850781090&proxyReferer=https:%2F%2Fwww.google.com%2F)

> 윈도우에서 stub 코드는 MIDL(Microsoft Interface Definition Language)이라는 특수한 형태로 작성하면 컴파일의 결과로 나온다.

**RPC를 위해 고려해야할 부분**

- 데이터 포멧

    매개변수로 보내지는 데이터가 마샬링 될 때, 시스템 별로 표현법이 다른 것을 고려해야한다. 빅 엔디언(big-endian)을 사용하는 시스템과 리틀 엔디언(little-endian)을 사용하는 시스템에 따라 달라져야 하는데, 이를 해결하기 위해서 독립적인 데이터 표현법을 사용한다. 그 중 하나가 XDR(External Data Representation)인데, 클라이언트와 서버에서 각각 아키텍쳐에 맞게 포매팅 해준다.

- 실패 처리

    RPC는 네트워크를 이용한다는 점을 고려해야한다. 네트워크에 이상이 생겨 실패하거나, 두 번 이상 실행되는 경우가 생길 수도 있다. 하지만, 한 번이 아닐 가능성이 있으면(at most once) 안 된다. 이를 해결하기 위해 OS는 메세지가 반드시 한 번만(exactly once) 도착해야 하는 것을 보장해줘야 한다.

    우선, 각 메세지에 타임스탬프를 찍어서 중복 실행을 막는 방법이 있는데, 여러번 실행되는 것은 막을 수 있지만 반드시 한 번 실행되는 것을 보장해주지는 않는다. 이를 해결하기 위해 RPC 호출이 수신 및 실행되었다고 클라이언트에 ACK 메세지를 되돌려 보내준다.

- 실행 위치 파악

    원격 서버에 소스가 위치하기 때문에 실행 위치를 알아야 된다. RPC는 포트를 알면 어디에 소스가 위치하는 것인 줄 아는 것과 같기 때문에 서버의 포트 번호를 알아내면 된다. 크게 두 가지 방법이 있다.

  - 고정 된 포트 번호를 사용하거나
  - rendezvous(혹은 matchmaker)를 사용한다.

    > RPC 포트 번호로 가동되는 데몬 프로세스로, 어떤 포트를 실행해야할지 알려준다.
    > ![rpc-matchmaker](https://user-images.githubusercontent.com/24666330/119587690-28e55280-be0a-11eb-8906-2de1f8c362df.png)


#### 3.8.2.1 Android RPC

RPC가 일반적으로 분산 시스템의 클라이언트-서버 구성에 사용되지만 같은 시스템 내부에서 IPC로 사용할 수도 있다. 안드로이드의 바인더 프레임워크에는 다른 프로세스의 서비스를 요청할 수 있도록 다양한 IPC 메커니즘이 사용된다.

안드로이드는 application component라는 것을 기본 빌딩 블록으로 사용한다. 이것은 안드로이드 애플리케이션에 유틸을 전달해주고, 여러 애플리케이션 컴포넌트를 결합하여 앱의 기능으로 사용할 수 있도록 해준다. 이러한 컴포넌트 중에 서비스라는게 있는데, ui없이 백그라운드에서 돌아간다. 이는 오랫동안 실행되는 작업이나 원격 프로세스 작업을 수행할 때 사용된다. 예를 들어서 음악을 백그라운드로 실행시키거나, 다운로드를 받는 동안 다른 작업을 할 수 있도록 해주는 것이다.

클라이언트 앱이 서비스의 `bindService()` 메소드를 호출하면 해당 컴포넌트가 묶이면서(bound) 메세지 패싱이나 RPC를 사용하여 클라이언트-서버 커뮤니케이션을 할 수 있는 상태가 된다. 묶인 서비스는 `Service` 클래스를 반드시 상속(extend)해야 하고, `onBind()` 메소드를 반드시 구현(implement)해야 한다. 이 메소드는 `Messenger` 서비스를 리턴하는데,  클라이언트에 단방향으로 메세지를 보내준다. 클라이언트는 `Message` 객체의 `replyTo` 필드를 이용하여 서비스의 응답이 어디로 와야하는지 알려줘야 한다.

RPC를 위해서 `onBind()` 메소드는 반드시 원격 객체의 메소드를 나타내는 인터페이스를 리턴해야 한다. 이 것은 stub파일을 만들기 위해 AIDL(Android Interface Definition Language)를 사용한다. 클라이언트 인터페이스를 원격 서비스에 제공한다.

```java
interface RemoteService {
    boolean remoteMethod(int x, double y);
}
```

RemoteService.aidl 을 위와 같이 작성하면 RemoteService.java 파일을 생성하고, 해당 파일에 있는 메소드가 stub이 된다. 그리고 서버에서 이 인터페이스를 구현(implement)하여 서버에 있는 해당 메소드를 호출하면 서버에 정의된 소스가 나오도록 한다.

클라이언트가 `bindService()` 를 호출하면 `onBind()` 메소드가 호출되고, 리모트 서비스에 있는 stub을 리턴한다. 그러면 클라이언트에서 해당 메소드를 실행시킨다.

```java
RemoteService service;
...
service.remoteMethod(3, 0.14);
```

내부적으로 안드로이드 바인더 프레임워크가 매개변수 마샬링을 처리 및 전송, 서비스의 구현부분을 호출하고, 반환값을 클라이언트 프로세스로 다시 보내준다.

<details>
<summary> 질문(2021.05.23) </summary>

- 윈도우만 특정한 이유?
-> 옛날 윈도우 app과 현재 윈도우 app이 다른데, 현재도 사용되는 것인지 모르겠다.

- GRPC
  프런트앤드 컴포넌트에서 나온것
  구글에서 RPC프레임워크 만듬
  MSA 구조를 만들때 서버에서 해당 프레임워크를 쓰면 좋다고 홍보했다고 한다.

  각 MSA 패키지들의 컨트롤러가 있어야 하는 것일까?
  서버-클라이언트 구조이므로

  -> matchmaker가 있으니 필요없지 않을까?

  msa에서는 서비스간에 네트워크 통신을 해야하기 때문에 성능이 떨어지는데 이걸 gRPC로 커버한다고 하네요

  그리고 msa내의 서비스들이 다른 언어로 되어있어도 grpc로 통신하게되면 무조건 protobuf형태여서 인터페이스가 하나로 통일된대요

  참고로 rest를 이용해서 msa 통신을 하면 엄청 느리다.

  -> 특별한 컴파일이 따로 필요할 것 같진 않다.

- RPC를 많이 사용하나?

    -> RPC를 실제로 쓰는 것은 보지 못했다. 안드로이드 바인더는 유명하다. 시스템 공부를 할 때 가장 중요한 개념이기 때문에

</details>
