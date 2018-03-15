# DockerStudy

## clone()을 이용한 namespace의 격리

clone()을 통한 신규 프로세스를 생성하면서 새로운 namespace생성하는 방식.
clone()은 사실상 fork()의 구현이다.

| namespace | system parameter | 격리내용 |
------ | ------ | ---------------
UTS | CLONE_NEWUTS | host네임, 도메인네임
IPC | CLONE_NEWIPC | 시그널, 메세지큐와 공유 메모리
PID | CLONE_NEWPID | 프로세스 ID
Network | CLONE_NEWNET | 네트워크장비, 네트워크 스택, 포트 등 
Mount | CLONE_NEWNS | 파일시스템 
User | CLONE_NEWUSER | 사용자 및 사용자 그룹

사용자격리는 리눅스커널 3.8 이상이고 커널컴파일 시 USER_NS 기능을 사용해야 한다.
확인이 어려우면 Ubuntu10.04버전을 사용하면 된다.
## cgroups를 이용한 자원의 제한
cgroups 는 controller groups.

### cgroups의 작용
1. 자원의 제한. cgroups의 task가 사용할 자원의 총량에 대해 제한한다. task가 이 제한을 초과하면 OOM(Out of Memory)가 발생된다.
1. 우선순위 분배. CPU 시간 할당, 디스크 IO throughput를 할당하면서 사실상 task의 우선수위를 제한한것이다.
1. 자원 통계. cgroups는 시스템 자원의 사용량에 대한 통계를 낼 수 있다. 예를 들어 cpu의 사용시간, 메모리의 사용량 등이다. 이는 과금에 대한 기능을 제공할 수 있다.
1. task 컨트롤. cgroups는 task에 대해 대기, 재실행 등 작업을 실행할 수 있다.

### cgoups 용어 설명
* task는 프로세스일수도 있고 스레드 일 수도 있다.
* cgroup은 자원에 대한 컨트롤의 단위다. 하나 혹은 여러개의 subsystem으로 그성된다.
* subsystem은 하나의 자원에 대한 컨트롤러이다. 예를 들어 CPU subsystem은 CPU 시분할에 대한 컨트롤을 할수 있고 메모리 subsystem은 메모리 사용량에 대한 컨트롤을 할 수 있다.
* hierachy. 하나의 계층은 하나의 트리모양의 cgroup으로 구성되어 있다. 매 계층은 대응되는 subsystem에 대한 자원을 컨트롤할 수 있다. 계층안의 cgroup 노드는 부모의 노드의 subsystem을 상속받는다. 하나의 시스템은 여러 hierachy를 가질 수 있다.

![계층 이미지](/src/1.png)

그림에서 Cgroup Hierarchiy A는 CPU subsystem과 CPUACCT(cgroups의 CPU사용량을 집계할 수 있는 서브시스템)subsystem을 추가 하고 cgroup cgrp1를 사용하는 task는 CPU의 시분할자원을 60% 점유 할 수 있고 cgroup cgrp2를 사용하는 task는 시분할자원을 20% 점유 할수 있다. 

### cgroups의 구조 생성 규칙
1. 같은 hierachy는 하나 혹은 여러개의 subsystem을 추가 할 수 있다.
1. subsystem은 여러개의 hierachy에 추가 될수 있지만 이때 추가하는 hierachy는 subsystem이 하나만 있어야 있다.
1. 시스템이 하나의 hierachy를 새로 생성하면 이 hierachy는 시스템의 모든 task에 추가되며 이 cgroup은 root cgroup이라고 불린다. task는 동일한 hierarchiy에서 하나의 cgroup에만 존재할 수 있고 서로 다른 hierarchiy의 cgroup에는 동시에 존재 할 수 있다. 예를 들어 메모리를 점유하는 cgroup에서 10MB용량과 10GB용량을 같는다는것은 상호 모순된다. 즉 하나만 가질 수 있다. 메모리는 10GB 사용량을 확보하고 CPU는 20%의 시분할 자원을 점유 할수 있다.
1. fork된 자식프로세스는 부모프로세스와 동일한 cgroup에 속할 수 있지만 상호 독립된 프로세스이므로 앞으로 자식 프로세스가 다른 cgroup으로 변할 수 있다.