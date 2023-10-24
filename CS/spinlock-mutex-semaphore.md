# 스핀락, 뮤텍스, 세마포어

자료 출처 - 쉬운코딩 https://www.youtube.com/watch?v=gTkvX2Awj6g

- race condition : 여러 프로세스 / 스레드가 동시에 같은 데이터를 조작할 때 타이밍이나 접근 순서에 따라 결과가 달라질 수 있는 상황.

- 동기화: 여러 프로세스 / 스레드를 동시에 실행해도 공유 데이터의 일관성을 유지하는 것

- 임계 영역: 공유 데이터의 일관성을 보장하기 위해 하나의 프로세스 / 스레드만 진입해서 실행 가능한 영역

mutual exclusion(상호배제)을 보장할 수 있을까?
임계 영역 전 후로 lock/unlock을 걸어 하나의 프로세스 / 스레드만 진입할 수 있도록 정해준다.

```c++
do {
	acquire lock
		critical section
	release lock
		remainder section
} while (True)
```

Thread 의 예제로 아래와 같이 코드를 만들 수 있다:

```c++
volatile int lock = 0; // global

void critical() {
	while (test_and_set(&lock) == 1);
	... critical section
	lock = 0;
}
```

```c++
int TestAndSet(int* lockPtr) {
	int oldLock = *lockPtr;
	*lockPtr = 1;
	return oldLock;
}
```

여기서 TestAndSet은 CPU atomic 명령어다. 실행 중간에 간섭(interrupt)나 중단 받지 않는다.
같은 메모리 영역에 대해 동시에 실행되지 않는다.
멀티코어 환경에서 서로 다른 스레드가 해당 명령어를 실행해도, 동시에 실행 될 수는 없다.

Mutex클래스로 lock / unlock 예시:

```c++
class Mutex {
	int value = 1;
	int guard = 0;
}

Mutex::lock() {
	while (test_and_set(&guard));
	if (value == 0) {
		//... 현재 스레드를 큐에 넣음;
		guard = 0; & go to sleep
	} else {
		value = 0;
		guard = 0;
	}
}

Mutex::unlock() {
	while (test_and_set(&guard));
	if (큐에 하나라도 대기중이라면){
		// 그중에 하나를 깨운다;
	}else {
		value = 1;
	}
	guard = 0;
}

mutex -> lock();
... critical section
mutex -> unlock();


```

Summary: 뮤텍스 (mutex는 락을 가질 수 있을 때 까지 휴식)

스핀락이 좋을 때:
멀티 코어 환경이고, 임계영역에서의 작업이 컨텍스트 스위칭보다 더 빨리 끝난다면 스핀락이 뮤텍스보다 더 이점이 있다.

_내생각 : 폴링도 스핀락과 비슷한 방식으로 반복적인 동작을 수행하는 것 같다. 스핀락은 공유 자원에 대한 접근을 조절하는 데 사용되는 동기화 메커니즘이며 폴링은 주기적으로 상태를 확인하는 방식이다. 폴링은 데이터 동기화나 이벤트 전파와 관련이 있을 수 있지만, 스핀락은 공유자원의 상호배제를 위해 명시적으로 설계됐다._

Semaphore(세마포)
signal mechanism을 가진 하나 이상의 프로세스 / 스레드가 critical section에 접근 가능하도록 하는 장치

```c++
class Semaphore {
	int value = 1;
	int guard = 0;
}
Semaphore::wait() {
	while(test_and_set(&guard));
	if(value == 0) {
		...현재 스레드를 큐에 넣음
		guard = 0; & go to sleep
	}else {
		value -= 1;
		guard = 0;
	}
}
Semaphore:: signal() {
	while(test_and_set(&guard));
	if(큐에 하나라도 대기중이라면) {
		그 중에 하나를 깨워서 준비 시킨다.
	}else {
		value += 1;
	}
	guard = 0;
}
```

세마포어는 순서를 정해줄 때 사용된다.

```c++
task1
semaphore -> signal()
task2
semaphore -> wait()
task3
```

여기서 task3는 semaphore가 signal를 받아야 실행할 수 있다.

뮤텍스와 이진 세마포는 사실상 똑같지는 않다.
뮤텍스는 락을 가진 자만 락을 해제 할 수 있지만, 세마포는 그렇지 않다.

뮤텍스는 priority inheritance 속성을 가진다. 세마포는 그 속성이 없다.

우선순위 스케줄링을 할 때 우선순위가 높은 프로세스가 낮은 프로세스의 lock에 의해 task를 수행하지 못하기에, 이를 해결하기 위해 낮은 프로세스의 우선순위를 높은 프로세스의 우선순위 만큼 올려주고 lock을 풀 수 있도록 해준다. 이를 priority inheritance다.

세마포어는 누가 lock을 해제할지 알 수 없기에 뮤텍스와 다르다.

상호 배제만 필요하다면 뮤텍스를, 작업 간의 실행 순서 동기화가 필요하다면 세마포를 권장한다.
