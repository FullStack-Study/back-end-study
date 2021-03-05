# CPU 스케줄링 (CPU scheduling)

CPU 스케줄링은 단기스케줄러가 CPU와 메모리 사이의 스케줄링을 하는것이다. 단기스케줄러는 실행중이던 프로세스가 대기상태가 되면, 대기중인 프로세스로부터 CPU를 회수하고 새로운 프로세스에게 CPU를 할당한다. 이때 CPU를 할당한 프로세스를 결정하는데, Ready Queue에 있는 프로세스 중 `running` 시킬 프로세스를 정하는 작업이다.

Ready Queue는 반드시 **선입선출(FIFO)** 방식의 큐가 아닐 수도 있다.

Ready Queue는

- 선입선출 큐
- 우선순위 큐
- 트리
- 또는 단순히 순서가 없는 연결 리스트 형태로 구성될 수 있다.

Queue에 있는 레코드는 일반적으로 각 프로세스의 PCB형태이다.

### 스케줄러 디스패치(Dispatcher)

![프로세스 상태](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/10aa87ce-25ea-43a4-8eeb-753c003955ce/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210305%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210305T124124Z&X-Amz-Expires=86400&X-Amz-Signature=a5035b6a7be21bd4d50f3ee2952caf4a6fb0caebf4c7e508ed80ae931f1664b2&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

CPU 스케줄링에서 프로세스가 `ready` → `running` 로 상태전이 하도록 관리하는 작업을 디스패치라고한다. 이 작업을 하는 것이 **디스패처(dispatcher)** 모듈이다.

디스패처는 다음과 같은 작업을 한다.

- context switching
- 사용자 모드로 전환
- 프로그램을 재개하기 위해 사용자 프로그램의 적절한 위치로 이동(jump)

디스패처는 모든 프로세스가 context switching 할 때 호출되므로, 가능한 빨리 작업을 마쳐야 한다.

디스패처가 하나의 프로세스를 정지하고 다른 프로세스의 수행을 시작하는 데 까지 소요되는 시간을 **디스패치 지연(dispatch latency)**라고 한다.

## 6.1. 선점(Preemptive), 비선점(Non-Preemptive)

### CPU 스케줄링은 다음 네가지 상황에서 발생 할 수 있다.

1. 한 프로세스가 **running** 상태에서 **waiting** 상태로 전환될 때 (ex. 입출력 요청이나 자식 프로세스가 종료되기를 기다리기 위해 `wait()`을 호출 할 때)
2. 프로세스가 **`running`** 상태에서 **`ready`** 상태로 전환될 때 (ex. 인터럽트 발생)
3. 프로세스가 **`waiting`** 상태에서 **`ready`** 상태로 전환될 때 (ex. 입출력 종료)
4. 프로세스가 **`종료(terminated)`** 될 때

### 6.1.1. 선점 스케줄링 (Preemptive Scheduling)

- 1, 4의 상황에서 한 프로세스가 실행중일 때 CPU 스케줄러가 다른 프로세스에게 CPU를 넘겨줄 수 없다. waiting 상태이거나 terminated 상태가 되면 CPU 스케줄러에 의한 스케줄링이 이루어질 수 있다. 이때는 비선점(또는 협조적) 방식으로 스케줄링이 발생한다.
- 선점 스케줄링은 데이터가 다수의 프로세스에 의해 공유될 때 **경쟁 조건(race condition)** 이 발생할 수 있다. 한 프로세스가 자료를 갱신하고 있는 동안 두 번째 프로세스가 선점되었을 때, 두 번째 프로세스가 데이터를 읽으려고 하면, 데이터 일관성이 깨지게 된다.

- 장점
    - 높은 우선순위를 가진 프로세스들이 빠른 처리를 요구하는 시스템에서 유용함
    - 응답시간이 비교적 빠르다.
    - 대화식 시분할 시스템에 적합하다.
- 단점
    - 나중에 들어오는 프로세스들이 높은 우선순위를 가지는 경우 잦은 context switching으로 인한 오버헤드를 초래한다.

### 6.1.2. 비선점 스케줄링 (Non-Preemptive Scheduling)

- 선점 방식 스케줄링에서는 CPU 스케줄러가 우선순위에 따라 CPU를 할당받아 실행중인 프로세스를 ready로 전환시키고 더 우선순위가 높은 프로세스에게 CPU를 할당할 수 있다.
- 장점
    - 응답시간을 예상할 수 있다.
    - 모든 프로세스에 대한 요구를 공정하게 처리한다.
- 단점
    - 짧은 작업을 수행하는 프로세스 앞에 긴 작업을 수행하는 프로세스가 있는 경우 평균 대기시간이 길어진다.

## 6.2. 스케줄링 알고리즘

### 스케줄링 기준

- CPU 이용률(utilization): CPU 실행 시간 중 CPU를 사용중인 시간 비율이다. CPU가 바쁘다면 효율적으로 사용하고 있다는 뜻이다. 따라서 이용률이 높을수록 좋다. 실제 시스템에서는 40~90% 정도의 수치를 가진다.
- 처리량(throughput): 단위 시간 당 완료된 프로세스의 개수이다. 처리량이 높을수록 효율적이다.
- 총 처리 시간(turnaround time): 프로세스를 실행하는데 소요된 시간이다. 프로세스 실행 시작 시간부터 종료 시간까지 걸리는 시간이다.

    (메모리 접근시간+대기큐에서의 대기 시간+CPU에서 실행시간+I/O에 걸린 시간)

- 대기 시간(waiting time): 프로세스가 Ready Queue에서 기다리는 시간의 합이다. 대기시간이 길어지면 starvation과 같은 문제가 발생하므로 평균적으로 짧을수록 좋다.
- 응답 시간(response time): 프로세스가 요청된 후 첫 번째 응답을 받기까지 걸리는 시간이다. 주로 대화식 시스템(interactive system)에서 기준으로 사용한다. 응답시간은 짧을수록 좋고, 균일하면 예측하기 좋기 때문에 변동폭이 작을수록 좋다.

CPU가 일을 처리하고, I/O 작업 요청에 대한 응답을 기다리는 일을 반복한다고 가정하자. 이 때 CPU burst와 I/O burst만 있다고 가정한다. 프로세스가 I/O burst 일 때는 CPU가 놀고 있으므로 이때 다른 프로세스에게 CPU를 넘겨주는 것이 좋다.

![CPU burst / IO burst](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/51b0fc8b-f3ea-4f57-906f-a5a2e8645d22/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210305%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210305T124202Z&X-Amz-Expires=86400&X-Amz-Signature=c8bc40ace05d07091eccc29cdf66f5bfc0d3f7fdfd3819d10850fa4c1afe8617&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

- CPU burst time: CPU가 프로세스에 할당되어 작업을 처리하는 시간
- I/O burst time: 입출력 요청에 대한 응답을 기다리는 시간

## 6.3. 비선점형 스케줄링 알고리즘

### 6.3.1. 선입 선처리 스케줄링 (FCFS: First Come First Served Scheduling)

- CPU를 먼저 요청하는 프로세스가 CPU를 먼저 할당 받는다.
- FIFO(선입선출) 큐로 관리한다.
- 프로세스가 Ready Queue에 진입하면, 이 프로세스의 PCB를 큐에 입력한다.
- CPU burst time이 긴 프로세스가 CPU를 독점하는 **호위 효과(convey effect)** 가 발생한다.

    짧은 프로세스들이 먼저 처리되지 않고 긴 대기시간을 가지므로 CPU와 장치 이용률이 저하된다.

### 6.3.2. 최단 작업 우선 스케줄링(SJF: Shortest Job First Scheduling)

- CPU burst time이 짧은 프로세스가 먼저 실행된다. 평균 대기시간이 짧아진다.
- CPU burst time이 긴 프로세스는 우선순위가 밀리게 되어 무한정 대기상태에 빠지는 **기아상태(Starvation)** 가 발생할 수 있다.
- 선점형 SJF 알고리즘은 SRT 스케줄링이라고 불린다.

### 6.3.3. 우선순위 스케줄링(Priority Scheduling)

- 프로세스에 우선순위가 있으며, 우선순위가 높은 프로세스에 먼저 CPU를 할당한다.

    SJF도 우선순위 스케줄링 알고리즘 중의 하나이다.

- 우선순위가 같은 프로세스들은 선입선처리(FCFS)로 처리된다.
- 0이 높은 우선순위를 가지는지, 낮은 우선순위를 가지는지는 시스템에 따라 다르다.
- 계속해서 높은 우선순위의 프로세스가 CPU를 요청한다면, 낮은 우선순위의 프로세스들이 처리되지 못하기 때문에 **기아상태(starvation)** 가 발생한다.
- 기아상태는 **aging** 을 통해 해결할 수 있다. aging은 프로세스가 Ready Queue에 머무르는 시간이 길어지면 우선순위가 높아지도록 하는 기법이다.

## 6.4. 선점형 스케줄링 알고리즘

### 6.4.1. 최소 잔류 시간 우선 스케줄링(SRT: Shortest Remaining Time First)

- SJF의 선점형 방식 스케줄링 알고리즘이다.
- 가장 짧은 CPU burst time을 가진 프로세스가 먼저 실행된다.
- 남은 처리 시간이 더 짧은 프로세스가 Ready Queue에 진입하면 선점하여 우선 수행한다.
- 짧은 작업이 많은 경우 SJF보다 평균 대기시간이 줄어들지만, 긴 작업이 많은 경우 SJF보다 대기 시간이 길어진다.
- SRT 역시 긴 작업 시간을 가진 프로세스는 낮은 우선순위가 유지되기 때문에 무한정 대기하게 되어 **기아상태(starvation)** 가 발생한다.

### 6.4.2. 라운드 로빈 스케줄링(RR: Round Robin Scheduling)

- 대화식 사용자를 위한 시분할 시스템(Time sharing System)을 위해 고안되었다. 응답시간을 예측할 수 있으며, 응답시간이 빠르기 때문에 공정한 스케줄링이다.
- **시간 할당량(time quantum)** 이라고 하는 작은 단위의 시간을 정의한다. 일반적으로 10에서 100밀리초의 time quantum을 사용한다.
    - time quantum이 매우 길면, FCFS와 비슷하게 동작한다.
    - time quantum이 매우 작으면, context switching이 자주 발생하기 때문에, 오버헤드가 발생한다. time quantum은 context switching 시간에 비해 적당히 커야한다.
    - time quantum가 CPU burst의 80% 이상이면 turnaround time이 개선된다.
- Ready Queue는 원형 큐로 동작한다.
- CPU 스케줄러는 Ready Queue에서 첫번째 프로세스를 선택해 한 번의 time quantum 이후에 인터럽트를 걸도록 타이머를 설정 한 후, 프로세스를 dispatch한다.
- FCFS 처럼 먼저 CPU를 요청한 프로세스가 먼저 할당되지만, CPU burst time이 time quantum보다 긴 프로세스의 경우, 프로세스가 time quantum 동안 CPU를 사용하고 다음 프로세스가 CPU를 선점 한다.
- CPU burst time이 time quantum 보다 작은 경우, 프로세스의 작업이 끝나면 프로세스는 자발적으로 CPU를 반납한다. 스케줄러는 바로 Ready Queue의 다음 프로세스에게 CPU를 할당한다.