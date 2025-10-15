# CH.08 synchronized는 제대로 알고 써야 한다.
## 자바에서 스레드는 어떻게 사용하나?
- 하나의 프로세스에는 여러 개의 스레드가 생성된다.
- 단일 스레드가 생성되어 종료될 수도 있고, 여러 개의 스레드가 생성되어 수행될 수도 있다.
- 스레드는 다른 말로 Lightweight Process(LWP)라고도 한다.
- 가벼운 프로세스이고, 프로세스에서 만들어 사용하고 있는 메모리를 공유한다.
- 그래서 별개의 프로세스가 하나씩 뜨는 것보다는 성능이나 자원 사용에 있어서 많은 도움이 된다.
### Thread 클래스 상속과 Runnable 인터페이스 구현
- 스레드의 구현은 Thread 클래스를 상속받는 방법과 Runnable 인터페이스를 구현하는 방법 두 가지가 있다.
- 기본적으로 Thread 클래스는 Runnable 인터페이스를 구현한 것이기 때문에 어느 것을 사용해도 거의 차이가 없다.
- Runnable 인터페이스를 구현하면 원하는 기능을 추가할 수 있다.
- 이는 장점이 될 수도 있지만, 해당 클래스를 수행할 때 별도의 스레드 객체를 생성해야 한다는 점은 단점이 될 수도 있다.
- 또한 자바는 다중 상속을 인정하지 않는다.
- 따라서 스레드를 사용해야 할 때 이미 상속받은 클래스가 존재한다면 Runnable 인터페이스를 구현해야 한다.
```java
public class RunnableImpl implements Runnable {
    public void run() {
        System.out.println("This is RunnableImpl.");
    }
}
```
```java
public class ThreadExtends extends Thread {
    public void run() {
        System.out.println("This is ThreadExtends.");
    }
}
```
- `Thread` 클래스를 상속받은 경우에는 `start()` 메서드를 호출하면 된다.
- 하지만 `Runnable` 인터페이스를 구현한 경우에는 `Thread` 클래스의 `Runnable` 인터페이스를 매개변수로 받는 생성자를 사용해서 Thread 클래스를 만든 후 `start()` 메서드를 호출해야 한다.
```java
public class RunThreads {
    public static void main(String[] args) {
        RunnableImpl ri = new RunnableImpl();
        ThreadExtends te = new ThreadExtends();
        new Thread(ri) - start();
        te.start();
    }
}
```
### sleep(), wait, join 메서드
- 현재 진행중인 스레드를 대기하도록 하기 위해서는 `sleep()`, `wait()`, `join()` 세 가지 메서드를 사용하는 방법이 있다.
- 이 세 가지 메서드는 모두 예외를 던지도록 되어 있어 사용할 때는 반드시 예외처리를 해주어야 한다.
- `sleep(long millis)`: 명시된 ms만큼 해당 스레드가 대기한다. static 메서드이기 때문에 반드시 스레드 객체를 통하지 않아도 사용할 수 있다.
- `sleep(long millis, int nanos)`: 명시된 ms + 명시된 나노 시간만큼 해당 스레드가 대기한다.
- `wait()` 메서드도 명시된 시간만큼 해당 스레드를 대기시킨다.
- `sleep()` 메서드와 다른 점은 매개변수인데, 만약 아무런 매개변수를 지정하지 않으면 `notify()` 메서드 혹은 `notifyAlI()` 메서드가 호출될 때까지 대기한다.
- `join()` 메서드는 명시된 시간만큼 해당 스레드가 죽기를 기다린다.
- 만약 아무런 매개변수를 지정하지 않으면 죽을 때까지 계속 대기한다.
### interrupt(), notify(), notifyAll() 메서드
- 앞서 명시한 세 개의 메서드를 '모두' 멈출 수 있는 유일한 메서드는 `interrupt()` 메서드다.
- `interrupt()` 메서드가 호출되면 중지된 스레드에는 `InterruptedException`이 발생한다.
- 제대로 수행되었는지 확인하려면 `interrupted()` 메서드를 호출하거나 `isInterrupted()` 메서드를 호출하면 된다.
- 두 방법의 차이는 `interrupted()` 메서드는 스레드의 상태를 변경시키지만, `isInterrupted` 메서드는 단지 스레드의 상태만을 리턴한다는 점이다.
- `notify()` 메서드와 `notifyAll()` 메서드는 모두 `wait()` 메서드를 멈추기 위한 메서드다.
- `wait()` 메서드가 호출된 후 대기 상태로 바뀐 스레드를 깨운다.
- `notify()` 메서드는 객체의 모니터와 관련있는 단일 스레드를 깨우며, `notifyAll()` 메서드는 객체의 모니터와 관련 있는 모든 스레드를 깨운다.
```java
public class Sleep extends Thread {
    public void run() {
        try {
            Thread.sleep(10000);//10초간 대기한 후 종료한다.
        } catch (InterruptedException e) {
            System.out.println("Somebody stopped me T T");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String args[]) {
        Sleep s = new Sleep();
        S.start();//스레드를 시작한다.
        try {
            int cnt = 0;
            while (cnt < 5) {
                S.join(1000); //1초씩 기다린다.
                cnt++;
                System.out.format("%d second waited\n", cnt);
            }
            if (S.isAlive()) {//스레드가 살아 있는지 확인한다.
                s.interrupt();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
## interrupt() 메서드는 절대적인 것이 아니다.
- `interrupt()` 메서드를 호출하여 특정 메서드를 중지시키려고 할 때 항상 해당 메서드가 멈출까? 정답은 '아니요'다.
- `interrupt()` 메서드는 해당 스레드가 'block'되거나 특정 상태에서만 작동한다.
```java
public class InterruptSample {
    public static void main(String[] args) throws Exception {
        InfinitThread infinit=new InfinitThread();
        infinit.start();
        Thread.sleep(2000);
        System.out.println("isInterrupted="+infinit.isInterrupted());
        infinit.interrupt();
        System.out.println("isInterrupted="+infinit.isInterrupted());
    }
}
```
```java
public class InfinitThread extends Thread {
    int value=Integer.MIN_VALUE;
    private boolean flag=true;
    public void run(){
        while(flag) {
            value++;
            if(value==Integer.MAX_VALUE) {
                value=Integer.MIN_VALUE;
                System.out.println("MAX_VALUE reached !!!");
            }
        }
    }
}
```
- `interrupt()` 메서드는 대기상태일 때에만 해당 스레드를 중단시키기 때문에 이 스레드는 멈추지 않는다.
### flag 값 수정하기
```java
public class InfinitThread extends Thread {
    int value=Integer.MIN_VALUE;
    private boolean flag=true;
    public void run(){
        while(flag) {
            value++;
            if(value==Integer.MAX_VALUE) {
                value=Integer.MIN_VALUE;
                System.out.println("MAX_VALUE reached !!!");
            }
        }
    }
    public void setFlag(boolean flag) {
        this.flag=flag;
    }
}
```
- 다른 스레드에서 `interrupt()` 메서드를 호출한 후 flag를 변경하는 방법이다.
```java
public class InterruptSample {
    public static void main(String[] args) throws Exception {
        InfinitThread infinit=new InfinitThread();
        infinit.start();
        Thread.sleep(2000);
        System.out.println("isInterrupted="+infinit.isInterrupted());
        infinit.interrupt();
        System.out.println("isInterrupted="+infinit.isInterrupted());
        infinit.setFlag(false);
    }
}
```
- 이 예제는 시작하고 2초 후에 `interrupt()` 메서드가 호출되고, flag 값이 false가 되기 때문에 바로 멈춘다.
### sleep() 추가하기
```java
public class InfinitThread extends Thread {
    int value=Integer.MIN_VALUE;
    private boolean flag=true;

    public void run(){
        while(flag) {
            value++;
            if(value==Integer.MAX_VALUE) {
                value=Integer.MIN_VALUE;
                System.out.println("MAX_VALUE reached !!!");
            }
            try {
                Thread.sleep(0,1);
            }catch(Exception e) {
                break;
            }
        }
    }
}
```
- 성능저하는 발생하지만, interrupt() 메서드가 호출되면 이 스레드는 바로 멈춘다.
## synchronized를 이해하자
- 하나의 객체에 여러 요청이 동시에 달려들면 원하는 처리를 하지도 못하고 이상한 결과가 나올 수 있다.
- 그래서 synchronized를 사용해서 동기화를 하는 것이다.
- 이 식별자를 사용하면 "천천히 한 명씩 들어와!"라고 해당 메서드나 블록에서 제어하게 된다.
- synchronized는 다음과 같이 메서드와 블록으로 사용할 수 있다.
```java
public synchronized void sampleMethod() {
    //중간생략
}

private Object obj = new Object();
public void sampleBlock() {
    synchronized (obj) {
        //중간생략
    }
}
```
- 메서드를 동기화하려면 메서드 선언부에 사용하면 된다.
- 특정 부분을 동기화하려면 해당 블록에만 선언을 해서 사용하면 된다.
- 언제 동기화를 사용해야 할까?
  - 하나의 객체를 여러 스레드에서 동시에 사용할 경우
  - static으로 선언한 객체를 여러 스레드에서 동시에 사용할 경우
## 동기화는 이렇게 사용한다 - 동일 객체 접근 시
```java
public class Contribution {
    private int amount=0;
    public void donate() {
        amount++;
    }
    public int getTotal() {
        return amount;
    }
}
```
```java
public class Contributor extends Thread {
    private Contribution myContribution;
    private String myName;
    public Contributor(Contribution contribution,String name) {
        myContribution=contribution;
        myName=name;
    }
    public void run() {
        for(int loop=0;loop<1000;loop++) {
            myContribution.donate();
        }
        System.out.format("%s total=%d\n", myName,
                          myContribution.getTotal());
    }
}
```
- 소스를 보면, 1인당 1원씩 1,000번 기부하고, 기부가 완료되면 현재까지 쌓인 기부금을 프린트하도록 되어 있다.
```java
public class ContributeTest {
    public static void main(String[] args) {
        Contributor[] crs=new Contributor[10];
        // 기부자와 기부 단체 초기화
        for(int loop=0;loop<10;loop++) {
            Contribution group=new Contribution();
            crs[loop]=new Contributor(group,"Contributor"+loop);
        }
        // 기부 실행
        for(int loop=0;loop<10;loop++) {
            crs[loop].start();
        }
    }
}
```
- 이렇게 수행하면 기부금을 받는 단체인 group 객체를 매번 새로 생성했기 때문에, 10명의 기부자가 10개의 각기 다른 단체에 기부하는 상황이 될 것이다.
```text
Contributor@ total=1000
Contributor8 total=1000
Contributor6 total=1000
Contributor4 total=1000
Contributor2 total=1000
Contributor9 total=1000
Contributor5 total=1000
Contributor3 total=1000
Contributor7 total=1000
Contributor1 total=1000
```
- 수행을 할 때마다 결과는 다르겠지만, 기부자가 돈을 낸 각 기부 단체에는 1,000원씩 기부되었을 것이다.
- 그럼 만약 기부 단체가 하나만 있을 경우에는 어떻게 될까? 앞의 Contribute Test 클래스를 다음과 같이 수정하여 매번 기부자를 생성하지 않고, 하나의 그룹을 여러 기부 단체에서 참조하도록 하면 원하는 기능이 구현될 것이다.
```java
//앞부분 생략
Contributor[] crs=new Contributor[10];
Contribution group=new Contribution();
for(int loop=0;loop<10;loop++) {
    crs[loop]=new Contributor(group, "Contributor"+loop);
}
//이하 생략
```
- 예상대로라면 각 단체에서 돈을 1,000원씩 냈기 때문에, 어떤 기부자가 마지막에 수행이 되든 기부금의 총합은 10,000원이 되어야 한다.
```text
Contributor® total=1000
Contributor8 total=9707
Contributor9 total=8707
Contributor7 total=7707
Contributor4 total=6707
Contributor5 total=5707
Contributor6 total=5676
Contributor2 total=3964
Contributor3 total=3121
Contributor1 total=2000
```
- 대부분 10,000이라는 값이 프린트되지 않고, 위 결과를 봐도 9,707이 최대 값이다.
- 이렇게 되는 이유는 10개의 Contributor 객체에서 하나의 Contribution 객체의 donate() 메서드를 동시에 접근할 수 있도록 되어있기 때문이다.
- 이 오류를 수정하기 위해서는 다음과 같이 donate() 메서드에 synchronized를 써서 동기화 식별자를 추가해야 한다.
```java
public synchronized void donate() {
    amount++;
}
```
```text
Contributor0 total=1883
Contributor8 total=10000
Contributor6 total=9074
Contributor4 total=8589
Contributor2 total=7000
Contributor9 total=6000
Contributor7 total=5888
Contributor5 total=4000
Contributor3 total=3000
Contributor1 total=2000
```
|케이스명 | 각각 단체에 기부 동기화 미사용 | 동일 단체에 기부 동기화 미사용 | 동일 단체에 기부 동기화 사용 |
|---|---|---|------------------|
|케이스 번호 | 1 | 2 | 3                |
|안정성 | O | X | O                |
|평균 응답 속도 | 1.3 ms | 1.3 ms | 10.1 ms          |
- 대부분의 프로그램에서 동기화를 부여한 메서드는 이렇게 간단하지 않다는 점이다.
- 약간의 대기시간을 주기 위해서 1번과 3번 케이스의 donate() 메서드에 1000ns씩 쉬도록 `Thread.sleep(0,1000);`을 추가하자.

|케이스명 | 각각 단체에 기부 동기화 미사용 | 동일 단체에 기부 동기화 사용 sleep 1000 ns | 각각 단체에 기부 동기화 사용 sleep 1000 ns |
|---|---|---|-------------------------------|
|케이스 번호 | 1 | 3 | 4                             |
|평균 응답 속도 | 1.953 초 | 16.105 초 | 1.954 초                       |

- 대기시간을 넣으니 응답시간이 많이 증가하였다.
- 반드시 필요한 부분에만 동기화를 사용해야 이와 같은 성능저하를 줄일 수 있을 것이다.
## 동기화는 이렇게 사용한다 - static 사용 시
- static을 사용하는 경우에 동기화를 사용한다.
```java
public class ContributionStatic {
    private static int amount=0;
    public void donate() {
        amount++;
    }
    public int getTotal() {
        return amount;
    }
}
```
- 각 단체에 기부하는 케이스를 고려해 보자.

```java
Contributor[] crs=new Contributor[10];
for(int loop=0;loop<10;loop++) {
    Contribution group=new Contribution();
    crs[loop]=new Contributor(group, "Contributor"+loop);
}
```
```text
Contributor0 total=10000
Contributor9 total=94786
Contributor7 total=84786
Contributor3 total=74786
Contributor1 total=54786
Contributor5 total=64786
Contributor8 total=52769
Contributor6 total=43097
Contributor4 total=33376
Contributor2 total=23463
```
- 각 단체에 기부하는 케이스라고 하더라도, amount를 static으로 선언하면 객체의 변수가 아닌 클래스의 변수가 된다.
- 따라서 아무리 여러 단체가 있더라도 하나의 amount에 값을 지정하게 되므로 이렇게 사용해서는 절대 안 된다.
- synchronized를 추가하고 실행
```text
Contributor0 total=16482
Contributor8 total=99814
Contributor6 total=89814
Contributor4 total=79814
Contributor2 total=69814
Contributor3 total=59814
Contributor5 total=49814
Contributor7 total=39814
Contributor9 total=29814
Contributor1 total=19814
```
- `synchronized`는 각각의 객체에 대한 동기화를 하는 것이기 때문에, 이렇게 하면 각각의 단체에 대한 동기화는 되겠지만 amount에 대한 동기화는 되지 않는다.
```java
public class ContributionStatic {
    private static int amount=0;
    public static synchronized void donate() {
        amount++;
    }
    public int getTotal() {
        return amount;
    }
}
```
- amount는 클래스 변수이므로 메서드도 클래스 메서드로 참조하도록 static을 추가해 주어야 한다.
```text
Contributor total=10000
Contributor2 total=76402
Contributor6 total=74911
Contributor1 total=64957
Contributor4 total=64621
Contributor9 total=50643
Contributor3 total=38356
Contributor5 total=97752
Contributor7 total=99267
Contributor8 total=100000
```
- 항상 변하는 값에 대해서 `static`으로 선언하여 사용하면 굉장히 위험하다.
## 동기화를 위해서 자바에서 제공하는 것들
- Lock: 실행 중인 스레드를 간단한 방법으로 정지시켰다가 실행시킨다. 상호참조로 인해 발생하는 데드락을 피할 수 있다. 
- Executors: 스레드를 더 효율적으로 관리할 수 있는 클래스들을 제공한다. 스레드 풀도 제공하므로, 필요에 따라 유용하게 사용할 수 있다. 
- Concurrent 콜렉션: 앞서 살펴본 콜렉션의 클래스들을 제공한다. 
- Atomic 변수: 동기화가 되어 있는 변수를 제공한다. 이 변수를 사용하면, synchronized 식별자를 메서드에 지정할 필요없이 사용할 수 있다.
## JVM 내에서 synchronization은 어떻게 동작할까?
- 자바의 HotSpot VM은 '자바 모니터(monitor)'를 제공함으로써 스레드들이 '상호 배제 프로토콜(mutual exclusion protocol)'에 참여할 수 있도록 돕는다.
- 자바 모니터는 잠긴 상태(lock)나 풀림(unlocked) 중 하나이며, 동일한 모니터에 진입한 여러 스레드들 중에서 한 시점에는 단 하나의 스레드만 모니터를 가질 수 있다.
- 모니터를 가진 스레드만 모니터에 의해서 보호되는 영역에 들어가서 작업을 할 수 있다.
- 여기서 보호된 영역이란 이 장에서 앞서 설명한 `synchronized`로 감싸진 블록들을 의미한다.
- HotSpot VM에서 대부분의 동기화 작업은 fast-path 코드 작업을 통해서 진행한다.
- 만약 여러 스레드가 경합을 일으키는 상황이 발생하면 이 fast-path 코드는 slow-path 코드상태로 변환된다.