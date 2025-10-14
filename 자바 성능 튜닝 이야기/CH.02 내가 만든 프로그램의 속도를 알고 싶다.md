# CH.02 내가 만든 프로그램의 속도를 알고 싶다
## 프로파일링 툴이란?
- 운영용 서버를 진단 및 모니터링하기 위해서 사용한다.
- APM 툴을 프로파일링 툴과 비교하면 프로파일링 툴은 개발자용 툴이고, APM 툴은 운영 환경용 툴이라고 할 수 있다.
- 프로파일링 툴
  - 소스 레벨의 분석을 위한 툴이다. 
  - 애플리케이션의 세부 응답시간까지 분석할 수 있다.
  - 메모리 사용량을 객체나 클래스, 소스의 라인단위까지 분석할 수 있다.
  - 가격이 APM 물에 비해서 저렴하다.
  - 보통 사용자수 기반으로 가격이 정해진다.
  - 자바 기반의 클라이언트 프로그램 분석을 할 수 있다.
- APM 툴
  - 애플리케이션의 장애상황에 대한 모니터링 및 문제점 진단이 주 목적이다.
  - 서버의 사용자 수나 리소스에 대한 모니터링을 할 수 있다.
  - 실시간 모니터링을 위한 물이다.
  - 가격이 프로파일링 툴에 비하여 비싸다.
  - 보통 CPU 수를 기반으로 가격이 정해진다.
  - 자바 기반의 클라이언트 프로그램 분석이 불가능하다.
- 프로파일링 툴은 대부분 느린 메서드, 느린 클래스를 찾는 것을 목적으로 하지만, APM툴은 목적에 따라 용도가 상이하다.
- 어떤 APM툴은 문제점 진단에 강한 한편, 다른 APM툴은 시스템 모니터링 및 운영에 강하다.
- 프로파일링 툴이 기본적으로 제공하는 기능
  - 웅답 시간 프로파일링 기능
    - 응답 시간을 측정
    - 하나의 클래스 내에서 사용되는 메서드 단위의 응답시간을 측정한다.
  - 메모리 프로파일링
    - 메모리 프로파일링을 하는 주된 이유는 잠깐 사용하고 GC의 대상이 되는 부분을 찾거나, 메모리 부족 현상(Memory Leak)이 발생하는 부분을 찾기 위함이다.
    - 클래스 및 메서드 단위의 메모리 사용량이 분석된다.
- CPU시간은 CPU를 점유한 시간을 의미하고, 대기시간은 CPU를 점유하지 않고 대기하는 시간을 의미한다.
- CPU시간과 대기시간을 더하면 실제 소요시간(Clock time)이 된다.
- CPU시간은 툴에 따라서 스레드(Thread) 시간으로 표시되기도 한다.
- 일단 툴에서 분석을 하려면 해당 메서드가 수행이 되어야 한다.
- 문제가 되는 메서드가 수행되어야 하므로, 메모리 부족 현상이 가장 분석하고 찾아내기 어렵다.
## System 클래스
- 모든 System 클래스의 메서드는 static으로 되어 있고, 그 안에서 생성된 in, out, err과 같은 객체들도 static으로 선언되어 있으며, 생성자(Constructor)도 없다.
- `static void arraycopy(Object src, int sicPos, Object dest, int destPos, int length)`
  - 특정 배열을 복사할 때 사용한다.
  - src는 복사 원본 배열, dest는 복사한 값이 들어가는 배열이다.
  - srcPos는 원본의 시작 위치, destPos는 복사본의 시작 위치, length는 복사하는 개수이다.
- JVM에서 사용할 수 있는 설정은 하나는 속성 (Property)값이고, 다른 하나는 환경(Environment)값이다.
- 속성은 JVM에서 지정된 값들이고, 환경은 장비(서버)에 지정되어 있는 값들이다.
- Properties를 사용하는 메서드
  - `static Properties getProperties()`
    - 현재 자바 속성값들을 받아온다.
  - `static String getProperty(String key)`
    - Key에 지정된 자바 속성값을 받아온다.
  - `static String getPropertyString key, String def)`
    - key에 지정된 자바 속성값을 받아온다. def는 해당 key가 존재하지 않을 경우 지정할 기본값이다.
  - `static void setProperties(Properties props)`
    - props 객체에 담겨있는 내용을 자바 속성에 지정한다.
  - `static String setProperty(String key, String value)`
    - 자바 속성에 있는 지정된 key의 값을 value값으로 변환한다.
- env를 사용하는 메서드
  - `static Map (String,String) getenv()`
    - 현재 시스템 환경 값 목록을 스트링 형태의 맵으로 리턴한다.
  - `static String getenv(String name)`
    - name에 지정된 환경변수의 값을 얻는다.
- 절대로 사용해서는 안되는 메서드
  - `static void gc()`
    - 자바에서 사용하는 메모리를 명시적으로 해제하도록 GC를 수행하는 메서드다.
  - `static void exit(int status)`
    - 현재 수행중인 자바 VM을 멈춘다.
  - `static void runFinalization()`
    - Object 객체에 있는 finalize()라는 메서드는 자동으로 호출되는데, 가비지 콜렉터가 알아서 해당 객체를 더 이상 참조할 필요가 없을 때 호출한다.
## System.currentTimeMillis와 System.nanoTime
- `static long currentTimeMillis()`
  - 현재의 시간을 ms로 리턴한다.
- UTC라는 시간표준체계를 따르는데, 1970년 1월 1일부터의 시간을 long 타입으로 리턴해 준다.
```java
public class CompareTimer {
    public static void main(String[] args) {
        CompareTimer timer = new CompareTimer();
        for (int loop = 0; loop < 10; loop++) {
            timer.checkNanoTime();
            timer.checkCurrentTimeMillis();
        }
    }

    private DummyData dummy;

    public void checkCurrentTimeMillis() {
        long startTime = System.currentTimeMillis();
        dummy = timeMakeObjects();
        long endTime = System.currentTimeMillis();
        long elapsedTime = endTime - startTime;
        System.out.println("milli=" + elapsedTime);
    }

    public void checkNanoTime() {
        long startTime = System.nanoTime();
        dummy = timeMakeObjects();
        long endTime = System.nanoTime();
        double elapsedTime = (endTime - startTime) / 1000000.0;
        System.out.println("nano=" + elapsedTime);
    }

    public DummyData timeMakeObjects() {
        HashMap<String, String> map = new HashMap<String, String>(1000000);
        ArrayList<String> list = new ArrayList<String>(1000000);
        return new DummyData(map, list);
    }
}
```
```java
class DummyData {
    private HashMap<String, String> map;
    private ArrayList<String> list;

    public DummyData(HashMap<String, String> map, ArrayList<String> list) {
        this.map = map;
        this.list = list;
    }
}
```
```text
nano=6.720132
milli=7
nano=6.047872
milli=39
nano=1.989248
milli=2
nano=1.937061
milli=21
nano=1.896381
milli=2
nano=3.386365
milli=4
nano=6.049927
milli=6
nano=6.127591
milli=39
nano=2.051296
milli=2
nano=2.310174
milli=2
```
- `System.currentTimeMillis()`로 측정한 결과가 `System.nanoTime()`으로 측정한 것보다 더 느린 것으로 나타난다.
- 추가로 초기에 성능이 느리게 나온 이유는 여러 가지이지만, 클래스가 로딩되면서 성능저하도 발생하고, JIT Optimizer가 작동하면서 성능 최적화도 되기 때문이라고 보면 된다.
- 메서드의 성능을 측정하는 방법
  - JMH
  - Caliper
  - JUnitPerf
  - JUnitBench
  - ContiPerf
- JMH와 Caliper를 제외한 나머지 툴들은 JUnit으로 만든 테스트 코드들을 실행하는데 사용된다.
- 앞서 살펴본 HashMap과 ArrayList 객체 생성속도를 JMH로 확인하는 예제코드
```java
@BenchmarkMode({Mode.AverageTime})
@OutputTimeUnit(TimeUnit.MILLISECONDS)
public class CompareTimerJMH {

    @GenerateMicroBenchmark
    public DummyData makeObject() {
        HashMap<String, String> map = new HashMap<String, String>(1000000);
        ArrayList<String> list = new ArrayList<String>(1000000);
        return new DummyData(map, list);
    }
}
```
- JMH는 클래스 선언시 반드시 어노테이션을 지정할 필요는 없다. 하지만, 그렇게 하면 기본 옵션으로 수행되기 때문에 평균응답시간을 측정하기 위해서 `@BenchmarkMode`로 옵션을 지정하였다.
- `@OutputTimeUnit` 어노테이션을 사용하여 출력되는 시간단위를 밀리초로 지정하였다.
- `@GenerateMicroBenchmark`는 측정대상이 되는 메서드를 선언할 때 사용한다. 해당 클래스에 메서드가 많이 있더라도 이 어노테이션을 지정하지 않으면 테스트 대상에서 제외된다.
```text
$ java -jar target/microbenchmarks.jar .*.CompareTimerJMH.* -i 3 -r 1s
# Measurement Section
# Runtime (per iteration): 1s
# Iterations: 3
# Thread counts (concurrent threads per iteration): [1, 1, 1]
# Threads will synchronize iterations
# Benchmark mode: Average time, time/op
# Running: com.perf.timer.CompareTimerJMH.makeObjectWithSize1000000
# Warmup Iteration 1 (3s in 1 thread): 2.694 msec/op
# Warmup Iteration 2 (3s in 1 thread): 2.112 msec/op
# Warmup Iteration 3 (3s in 1 thread): 2.331 msec/op
# Warmup Iteration 4 (3s in 1 thread): 2.186 msec/op
# Warmup Iteration 5 (3s in 1 thread): 2.126 msec/op
Iteration 1 (1s in 1 thread): 2.015 msec/op
Iteration 2 (1s in 1 thread): 2.125 msec/op
Iteration 3 (1s in 1 thread): 2.127 msec/op

Run result "makeObjectWithSize1000000": 2.089 ±(95%) 0.160 ±(99%) 0.369 msec/op

Run statistics "makeObject": min = 2.015, avg = 2.089, max = 2.127, stdev = 0.064

Run confidence intervals "makeObjectWithSize1000000": 95% [1.929, 2.249], 99% [1.720, 2.458]

Benchmark                                Mode Thr Cnt Sec  Mean  Mean error Units
c.p.t.CompareTimerJMH.makeObjectWithS... avgt 1 3 1 2.089 
0.369 msec/op
```
- 평균과 표준편차를 중심으로 보면 된다. 즉, 이 실험은 2.089ms 정도가 소요되었다고 보면 된다.