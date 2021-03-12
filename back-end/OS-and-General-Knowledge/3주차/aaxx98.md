# 프로세스 동기화 (Process Synchronization)

## 1. 배경

여러개의 스레드 또는 여러개의 프로세스가 병렬적으로 실행되는 상황에서 동일한 자원에 동시에 접근한다면, 한 프로세스가 공유 data에 접근 중일 때 다른 프로세스가 data를 변경한다면 데이터 일관성을 해칠 수 있다.  
(병렬적 실행이란 time-sharing system에서 정해진 시간동안 프로세스를 동작시키고, 시간이 끝나면 context switching을 하여 여러 작업이 동시에 실행되는 것 처럼 보이도록 하는 것)  
어떤 공유 data에 접근중이면 다른 프로세스의 접근을 제한하는 것을 **프로세스 동기화(Process Synchronization)** 라고 한다.

## 2. Critical Section Problem (임계영역 문제)

-   **Critical Section**
    : 병렬적 실행에서, 프로세스 코드 중 공유 data에 접근하는 부분을 Critical Section이라고 한다.

어떤 타이밍에 Critical Section의 코드를 실행중인 프로세스는 단 하나뿐이어야한다.  
만약 두 개 이상의 프로세스가 Critical Section의 코드를 실행중이라면 **Race Condition(경쟁 조건)** 이 발생한다.

Critical Section Problem은 이 방법을 설계한 것이다.

![경쟁조건](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/15f06ea9-bb6d-483c-a3f7-f9cfa8793769/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210312%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210312T095449Z&X-Amz-Expires=86400&X-Amz-Signature=8f4af5365ecffaf250cc185b2c3caeebdb348ae1b1d5c7dc1c02dae001427555&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)
`counter++`이 완전히 끝나고 `count--`가 실행된다고 가정하면 `counter = 5`임을 예상할 수 있다.
그러나 병렬실행으로 인해 동시에 두 코드의 critical section이 실행되면 race condition이 발생하고, counter는 5가 아닌 다른값을 가지게 된다.

## 3. 해결책

### 프로세스(Pi) 구조

프로세스 Pi와 Pj가 병렬 실행 중일때라고 가정한다.

```c++
do{
    [entry section] //critical section 시작 전
    {
        [critical section]
    }
    [exit section] //critical section 종료 후
    {
        [remainder section]
    }
} while(true);

```

```c++
do{
    while(trun == j); //i 번째 차례가 아니면 무한루프
    { // turn != j 이면 critical section 진입
        [critical section]
    }
    turn = j; // Pi의 critical section 종료 후 Pj에게 차례를 넘겨준다.
    {
        [remainder section]
    }
} while(true);
```

### 조건

1. Mutual Exclusion(상호 배제)

    어떤 프로세스 Pi 가 Critical Section을 실행중이라면, 다른 프로세스들은 Critical Section을 실행할 수 없다.

2. Progress(진행)

    Critical Section을 실행중인 프로세스는 없고, 진입을 원하는 프로세스들이 있을 때 반드시 바로 프로세스를 선택하여 Critical Section을 실행해야 한다.  
    remainder section을 실행중인 프로세스가 다른 프로세스의 Critical Section 진입을 막을 수 없다.

3. Bounded Waiting(한정된 대기)

    Critical Section에 진입 요청하는 횟수가 유한하다. (언젠가는 반드시 진입할 수 있음)

### 1. Mutex Locks

lock을 얻으면 critical section을 `aquire()`로 보호하고, 실행이 끝나면 `release()`한다.  
lock은 boolean일수도 있고, 두개 이상의 프로세스(또는 스레드)를 병행 실행할 수 있다면 숫자값일 수도 있다.
어떤 공유 data에 대한 lock을 가진 프로세스가 있다면 다른 프로세스는 해당 공유 data에 접근할 수 없다.

```c++
do{
    [acquire lock] //critical section 시작 전
    {
        [critical section]
    }
    [release lock] //critical section 종료 후
    {
        [remainder section]
    }
} while(true);

```

```c++
acquire(){
    while(!lock_available); //lock 요청, lock==false이면 무한 대기
    /* busy waiting */
    lock_available = false; //lock을 얻음, 다른 프로세스들은 lock을 얻을 수 없음
}

release(){
    lock_available = true; //lock 반환, 다른 프로세스들이 lock을 얻을 수 있음
}
```

-   단점  
     `aquire()`에서 lock을 요청할 때, 다른 프로세스가 lock을 가지고 있다면, lock이 풀릴 때 까지 무한대기 한다. 이것을 **busy waiting** 또는 **spinlock**이라고 한다.  
    busy waiting은 CPU가 놀고있는 시간이 되기 때문에 CPU 이용률이 감소한다.

*   +) `aquire()`과 `release()`는 **atomic**함이 보장된다.
    > **atomic** : 실행중에 context switching이 발생하지 않음

### 2. Semaphore

Semaphore는 정수형 변수이고, 가용 자원 개수를 나타낸다. 동기화 대상이 1 이상일 때 사용한다.

공유 data가 프로세스에 의해 사용중이면 `wait()`을 통해 세마포어를 1 줄인다. (S—)

공유 data의 사용이 완료되면, `signal()`을 통해 세마포어를 1 늘린다. (S++)

`wait()`과 `signal()`은 atomic하다.

-   **Counting Semaphore**는 S≥0의 값을 가질 수 있고, 최초의 S값은 1 이상의 수이다.
-   **Binary Semaphore**는 S=0 또는 S=1의 값을 가질 수 있고, 최초의 S값은 1이다. 이때는 **Mutex lock**과 유사하게 동작한다.

```cpp
do{
    [wait]//critical section 시작 전
    {
        [critical section]
    }
    [signal]//critical section 종료 후
    {
        [remainder section]
    }
} while(true);
```

1. busy waiting을 이용한 Semaphore

    CPU 이용률이 떨어진다는 단점이 있다.

    ```cpp
    wait(S){
    	while(S<=0); // S가 1 이상이면 할당받을 자원이 남아있다는 의미
    	//busy waiting
    	S--; // 자원을 할당받음
    }

    signal(S)[
    	S++; // 자원해제
    }
    ```

2. waiting queue를 활용한 Semaphore

    waiting queue에 있는 프로세스는 CPU를 할당받아 실행될 수 없다.

    S→value가 음수라는 것은, `가용자원 < 자원을 요청하는 프로세스 수` 라는 의미이다.

    이 때 자원을 요청중인 프로세스들을 무한정 대기시키지 않고, waiting queue에서 대기하도록 하면, ready queue에 있는 다른 자원들에게 CPU를 할당하여 실행할 수 있다.

    ```cpp
    typedef struct{
    	int value; //S
    	struct process *list; //waiting queue
    } semaphore

    wait(semaphore *S){
    	S->value--; //자원 할당 요청
    	if(S->value < 0){ //남아있는 공유자원이 없다면
    		[현재 프로세스 P를 S->list(waiting queue)에 추가] //현재 프로세스는 wait 상태가 된다.
    		block(); //사용가능한 자원이 없으므로 프로세스의 진입을 막는다.
    	}
    }

    signal(semaphore *S)[
    	S->value++; //자원 해제 요청
    	if(S->value <= 0){ //남아있는 공유자원이 있다면
    		[현재 프로세스 P를 S->list(waiting queue)에서 제거]
    		wakeup(P); //waiting 상태인 프로세스를 깨운다.(ready 상태로 만든다.)
    	}
    }
    ```

-   단점

    `wait()`과 `signal()`의 호출 순서가 바뀌면 문제가 발생한다.

    -   `wait()`-critical section-`wait()` : 현재 프로세스가 critical section을 빠져나가지 못한다. **Dead lock**이 발생한다.
    -   `signal()`-critical section-`wait()` : 2개 이상의 프로세스가 동시에 critical section에 접근한다. Mutual Exclusion(상호 배제)를 보장할 수 없다.

-   Dead Lock (교착상태)

    binary semaphore인 S, Q가 있고, 프로세스 P0, P1이 각각 다음과 같은 코드를 실행한다고 하자.

    -   P0

        ```cpp
        wait(S);
        wait(Q);

        ...

        signal(S);
        signal(Q);
        ```

    -   P1

        ```cpp
        wait(Q);
        wait(S);

        ...

        signal(Q);
        signal(S);
        ```

        이때 context switching에 의해 실행 순서가 다음과 같아진다면, 다음과 같은 현상이 발생한다.

        ```cpp
        P0: wait(S); // S=0
        P1: wait(Q); // Q=0
        P0: wait(Q); // P0은 P1이 Q를 해제할 때 까지 대기한다.
        P1: wait(S); // P1은 P0이 S를 해제할 때 까지 대기한다.
        ```

        P0과 P1은 각각 서로의 프로세스가 공유자원을 해제해 줄 때 까지 대기하고 있지만, 대기중이기 때문에 자원 해제를 할 수 없는 상태이다.

        따라서 두 프로세스는 무한정 대기하게 되고, 이를 교착상태라고 한다.

### 3. 모니터

앞선 방법들은 어셈블리 언어 수준에서 적합한 동기화 해결 방식이었다. 고급 언어를 이용한 프로그램에서는 모니터를 이용해 동기화 문제를 해결한다.

Java에서는 모든 객체에 모니터를 제공한다.

```java
class BankAccount {
	int balance;
	synchronized void deposit(int amt) {
		int temp = balance + amt;
		System.out.print("+");
		balance = temp;
	}
	synchronized void withdraw(int amt) {
		int temp = balance - amt;
		System.out.print("-");
		balance = temp;
	}
	int getBalance() {
		return balance;
	}
}
```

다음 코드를 보자, 공유 data는 balance이고, 프로그램 작동중 balance의 일관성을 유지하는 것은 중요한 일이다.

balance는 `deposit()`에 의해 증가할 수도 있고 , `withdraw()`에 의해 감소할 수도 있다.

balance의 일관성을 유지하기 위해서는 두 함수가 atomic하게 작동하는 것이 보장되어야 한다.

모니터는 다음과 같이 작동한다.

-   어떤 공유 data에 대해 모니터를 지정해 놓으면, 프로세스는 data에 접근하기 위해 모니터에 진입해야 한다.
-   프로세스가 모니터에 진입하려고 할 때, 다른 프로세스가 모니터 내부에 있다면 큐에서 대기한다.

    -   배타동기 queue: 어떤 스레드에서 공유자원을 사용하는 함수를 사용중이라면, 공유자원에 접근하는 함수를 사용하려는 스레드는 배타동기 queue에서 대기한다.  
        배타동기 함수는 `synchronized` 키워드로 선언할 수 있고, 배타동기 함수가 실행중일 때는 다른 배타동기 함수를 실행할 수 없다.
    -   조건동기 queue: 새 스레드가 진입 할 수 있도록 모니터에 있던 스레드가 잠시 block되어 대기하는 공간이다.

-   `wait()`  
    모니터에 진입중인 스레드를 block하고, 조건동기 queue에 대기시킨다.

-   `notify()`, `notifyAll()`  
    조건동기 queue의 스레드중 하나를 모니터에 복귀시킨다. `notifyAll()`은 조건동기 queue의 모든 스레드를 하나씩 모니터에 진입시킨다.
