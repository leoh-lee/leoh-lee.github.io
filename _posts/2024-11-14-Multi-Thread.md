---
title: "Multi Thread"
date: "2023-09-18 23:40:19"
categories: [Java]
tags: [Java]
---

# 멀티 스레드

**스레드**란 하나의 프로세스 내의 실행흐름을 말하며 **멀티 스레드**란 하나의 프로세스에서 여러 개의 실행흐름(스레드)으로 작업을 수행하는 것을 말한다. 즉, 하나의 프로그램에서 두 개 이상의 동작을 동시에 수행하기 위해 필요한 것이다.  
예를 들어 채팅을 하면서 첨부파일을 보낸다던가하는 것이다. 자바는 고맙게도 언어 단에서 멀티 스레드를 지원하기 때문에 구현되어있는 클래스와 인터페이스를 사용하여 쉽게 멀티 스레드를 처리할 수 있다.

## 메인 스레드

자바 애플리케이션을 실행하면 제일 먼저 `main()` 메서드가 실행된다. main 메서드가 바로 **_main thread_**인 셈이다.  
Thread를 따로 사용하지 않고 main 메서드만을 가지고 모든 작업을 수행한다면 싱글스레드로 동작하는 것이다.

```java
	public static void main(String[] args) {
    	// ... Main Thread 수행
    }
```

## 작업 스레드 생성

메인 스레드 이외의 스레드는 작업 스레드로 분류한다.  
생성하는 방법은 다음과 같다.

### Thread 클래스로 직접 생성

```java
	Thread thread = new Thread(Runnable target);
```

Runnable은 스레드 실행을 위한 인터페이스이기 때문에 실제로는 Runnable의 구현체가 들어가야한다.

```java
	public class threadA implement Runnable {
    	@Override
        public void run() {
        	//...수행할 작업
        }
    }
```

Runnable 인터페이스는 추상메서드 run()을 가지고 있다. Thread 객체를 통해 Thread를 시작하면, 해당 Runnable의 구현체에서 run() 을 찾아 실행한다.

```java
	Thread threadA = new Thread(new ThreadA());
	thread.start(); // 주입된 Runnable 구현체 ThreadA에서 run()을 실행
```

이렇게 클래스를 통해 Runnable을 직접 구현하지 않고 익명 구현체를 사용하는 방법도 있다.

```java
	Thread threadA = new Thread(new Runnable(){
    	@Override
        public void run() {
        	//...수행할 작업
        }
    });
```

### Thread의 자식 클래스 생성

Thread를 상속받아서 Thread를 생성할 수도 있다. Thread 클래스 역시 Runnable의 구현체이기 때문에 run()을 가지고 있기때문이다.

```java
	public void ThreadB extends Thread {
    	@Override
        public void run() {
        	// 스레드가 실행할 내용 ...
        }
    }
```

```java
	Thread thread = new ThreadB();
    thread.start();
```

이 방법 역시 따로 클래스를 만들어 구현할 필요 없이 익명 객체를 사용하는 방법도 있다.

```java
	Thread thread = new Thread() {
    	@Override
        public void run() {
        	// 스레드가 실행할 내용...
        }
    }
    thread.start();
```

## 스레드의 이름

스레드에도 각각 이름이 있다.  
가장 유명한 메인 스레드는 `main` 이며, 나머지 작업 스레드들은 기본적으로 Thread-n ( Thread-1, Thread-2 ... )의 이름을 가진다.  
때문에, 해당 스레드에 대해 확실한 실행 로그를 확인하기 위해서는 Thread에 이름을 지정해줄 필요가 있다.  
방법은 다음과 같다.

```java
	threadA.setName("스레드이름");
```

`setName()`은 Thread 객체의 인스턴스 메서드이며 `getName()`을 통해서 이름을 가져올 수도 있다.

```java
	Thread threadA = new Thread(){
    	@Override
        public void run() {
        	setName("threadA");
            Thread thread = Thread.currentThread();
            System.out.println(thread.getName());
        }
    }
    threadA.start();
    
    // 결과
    threadA
```

> `currentThread()`는 Thread 클래스의 정적 메서드이다. 현재 위치에서 활성화된 스레드를 가져온다.

## 스레드의 상태

스레드는 다음과 같은 상태를 가질 수 있다.

- 실행 대기
- 실행
- 일시정지
- 종료

스레드의 `start()`가 실행되면 해당 스레드는 우선 실행 대기 상태가 된다. 프로세스의 작업 스케쥴링에 의해 스레드가 종료될 때까지 해당 스레드는 대기 하다가 실행으로 갔다가 실행 대기로 가기를 반복한다.

또, 스레드를 일시 정지 상태로 만들 수 있다.

해당 상대로 만들기 위한 메서드들은 다음과 같다.

- 일시 정지: sleep(), wait(), join()
- 일시 정지 해제: notify(), notifyAll(), interrupt()
- 실행 대기: yield()

### 일시 정지 상태로 만들기

스레드를 일시 정지 상태로 만드는 메서드는 앞서 말했 듯 `sleep()`, `wait()`, `join()`이 있다.

#### `sleep()`

```java
	Thread.sleep(1000);
```

`sleep()`은 Thread 클래스의 정적 메서드이며, 현재 스레드를 매개변수 만큼 일시 정지한다. 매개변수는 밀리세컨드(1/1000)단위이며 1000이 1초이다.

#### `join()`

`sleep()`은 시간을 기준으로 일시 정지한다. 만약 어떤 행위의 완료를 기준으로 일시 정지하고 싶다면 `join()`을 사용할 수 있다.

```java
	public class JoinThread extends Thread {
    	private int sum = 0;
        
        public int getSum() {
        	return sum;
        }
        
    	@Overried
        public void run() {
        	for(int i=1; i<=1000; i++) {
            	sum+=i;
            }
        }
    }
```

JoinThread는 1에서 1000까지 모두 더하는 스레드 작업을 수행하는 객체이다.

```java
	JoinThread joinThread = new JoinThread();
    System.out.println(joinThread.getSum()); // 0
    joinThread.start(); // --> 더하기 반복문 시작
    
    try {
    	joinThread.join(); // --> joinThread 객체의 run()이 끝날 때까지 현재 스레드 일시정지
    } catch(Exception e) {}
    
    System.out.println(joinThread.getSum()); // 500500
    
```

만약 위 코드에서 `join()`을 지운다면 0, 0이 출력될 것이다.

#### wait()와 notify(), notifyAll()

wait()는 동기화 메서드 혹은 동기화 블럭에서 사용하는 Thread가 아닌 Object의 메서드이다.

> 💡 동기화 메서드란?
> 
> - 여러 스레드가 하나의 객체에 동시에 접근하는 경우 동시성 문제가 발생한다. 예를 들어 어떤 객체의 메서드를 통해 해당 객체의 데이터를 변경한 뒤 10초 뒤 해당 데이터를 가져오려고 하는데, 그 10초안에 다른 스레드가 해당 객체의 메서드를 통해 데이터를 변경해버리면 기존에 사용하던 스레드는 전혀 예상치못한 값을 얻게된다.
> - 따라서 단 하나의 스레드만 접근할 수 있도록 제한하는 메서드가 동기화 메서드(synchronized method)이다.
> - 동기화 메서드 혹은 블럭을 어떤 스레드가 사용 중이라면, 그 객체에 속한 다른 동기화 메서드나 블럭도 다른 스레드는 접근할 수 없다.
> - 단, 동기화 메서드가 아닌 메서드는 스레드와 무관하게 접근 가능하다.

```java
	// 동기화 메서드
	public synchronized void method() {
    	// 단 하나의 스레드만 실행 가능
    }
```

```java
	// 동기화 블럭
	public void method() {
    	synchronized(공유객체) {
        	// 단 하나의 스레드만 실행 가능
        }
    }
```

`wait()`는 이러한 동기화 코드내에서 사용된다. 동기화 메서드를 통한 간단한 예제를 보자.

```java
	public synchronized void method() {
    	// 스레드의 실행...
    	notify(); // wait()로 인한 일시 정지 상태의 다른 스레드 하나를 실행 대기 상태로 변경
        
       try {
       	wait();  // 현재 스레드는 일시 정지 상태로 변경
       } catch(Exception e) {}
    }
```

위와 같은 방법으로 synchronized method를 사용하여 `notify()`와 `wait()`, 하나의 객체를 공유하면서도 동시성 문제를 해결할 수 있다.  
wait()로 인해 일시 정지된 모든 스레드의 일시 정지를 해제하고 싶다면 `notifyAll()`을 사용하면 된다.

## 스레드의 종료

Thread객체는 스레드를 종료하는 `stop()`을 인스턴스 메서드로 가지고 있으나, 해당 메서드는 예상치 못한 버그 등으로 인해 deprecated 되었다.  
그렇다면 어떤 방법으로 자원을 반환하고 안전하게 스레드를 종료할 수 있을까?

### 1. boolean flag 사용

간단하게 스레드를 종료하고자 할 때 해당 스레드 객체의 boolean flag값을 변경하여 스레드를 종료하는 방법이 있다.

```java
	public class StopThread extends Thread {
    
      private boolean stop = false;

      public void setStop(boolean stop) {
          this.stop = stop;
      }

      @Override
      public void run() {
          while (!stop) {
              System.out.println("run");
          }
          System.out.println("자원을 반환하고 스레드를 종료합니다.");
      }
}
```

위 클래스는 `stop` 필드가 true가 되면 스레드의 수행이 종료된다.

```java
	    StopThread stopThread = new StopThread();

        stopThread.start();

        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {}

        stopThread.setStop(true);
        
        // 실행결과
        run
        run
        .
        .
        .
        자원을 반환하고 스레드를 종료합니다.
```

실제로 2초 뒤에 `stop`을 true로 변경하면 `run()`의 while문을 빠져나와 스레드가 종료된다.

### 2. `interrupt()` 사용

Thread의 `interrupt()` 를 호출하면 해당 스레드의 일시 정지 상태가 될 때, InterruptedException() 예외가 발생시켜 스레드가 종료할 수 있다.

```java
public class InterruptThread extends Thread {
    @Override
    public void run() {
        try {
          while (true) {
              System.out.println("run");
              Thread.sleep(1);
          }
        } catch (InterruptedException e) {}
        System.out.println("자원을 반환하고 스레드를 종료합니다.");
    }
}
```

```java
	    InterruptThread interruptThread = new InterruptThread();
        interruptThread.start();
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {}
        interruptThread.interrupt();
        
        // 실행 결과
        run
        run
        .
        .
        자원을 반환하고 스레드를 종료합니다.
```

Interrupt 스레드는 1ms씩 계속해서 일시 정지 상태에 들어가지만 스레드가 종료되진 않다가 2초 후 `interruptThread.interrupt()`가 실행되면 예외가 발생하여 스레드가 종료된다.

## 데몬스레드

기본적으로 스레드는 메인 스레드가 종료되더라도 다른 작업 스레드가 수행 중이라면 애플리케이션은 종료되지 않는다. 다만 특수한 경우 다른 작업 스레드가 어떤 스레드의 종료 시 함께 종료되어야하는 경우가 있다.  
예를 들어 음악 플레이어에서 메인 스레드가 종료됐는데 음악 재생 스레드만 살아있다면 좀 이상하고 자원 낭비일 것이다.

데몬스레드를 지정하는 방법은 다음과 같다.

```java
	// 간단하게 바로 위의 예제에서 사용한 클래스를 사용하겠다.
	InterruptThread thread = new InterruptThread();
    
    thread.setDaemon(true);
```

데몬 스레드로 지정할 스레드의 `setDaemon(true)`를 호출하면 현재 스레드에 종속하는 데몬 스레드로 만들 수 있다. 현재 스레드가 종료되면 해당 스레드도 함께 종료된다.

# 나가며

이번 포스팅에서는 자바에서의 멀티 스레드가 무엇인 지, 멀티 스레드의 생성, 이름, 상태, 종료에 대해 알아보았다.

> 🙏 참조: 이것이 자바다 (신용권, 임경균 저)