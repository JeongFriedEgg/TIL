# CH.10 로그는 반드시 필요한 내용만 찍자.
## System.out.println()의 문제
- 성능에 영향을 많이 주는 경우가 빈번히 발생한다.
- 특히 커널 CPU를 많이 사용하는데, 윈도에서 화면에 출력할 때 커널 CPU를 많이 점유하기 때문이다.

| | | 응답 시간 | 개선율 |
| :---: | :---: | :---: | :---: |
| 화면 선택시 총 소요 시간 | 변경전 | 1,242ms | - |
| | 변경 1 | 893ms | 39% |
| | 변경 2 | 504ms | 146% |
- 변경 1은 앞으로 설명할 로거를 사용하면서 로그 사용 여부를 false로 했을 경우이며, 변경 2는 모든 로그를 주석 처리하고 `System.out.printn()`을 제거한 경우이다.
- 변경 1을 반영하자 성능이 39% 개선되었고, 변경 2를 반영하자 146%가 개선되었다.
- 파일이나 콘솔에 로그를 남길 경우를 생각해보자.
- 내용이 완전히 프린트되거나 저장될 때까지, 뒤에 프린트하려는 부분은 대기할 수밖에 없다.
- 그렇게 되면 애플리케이션에서는 대기시간이 발생한다.
- 이 대기시간은 시스템의 속도에 의존적이다.
- 만약 디스크에 로그를 남긴다면, 서버 디스크의 RPM이 높을수록 로그의 처리속도는 빨라질 것이다.
- 또한 많은 서비스들이 통계성 데이터를 로그에 쌓고 처리하려고 한다.
- 적절한 로거를 사용해서 데이터를 쌓는 것은 좋겠지만, `System.out.printin()`으로 로그를 쌓는 것은 적절하지 않다.
## System.out.format() 메서드
- `format(String format, Object... args)`: 지정된 포맷으로 프린트를 한다. 뒤에 있는 매개변수를 쉼표로 나열하도록 되어있다.
- `format(locale l, String format, Object... args)`: 위의 메서드와 동일하되 가장 앞에 지역 정보를 포함한다. 지역에 따라 다른 형태의 데이터를 프린트할 수 있다.
- String을 더해서 처리하는 것과 Formatter를 사용하는 것 중 어느 방식이 더 빠른지 살펴보자.
```java
public class StringFormat extends SimpleBenchmark{
    public static void main(String[] args) {
        Runner.main(StringFormat.class,args);
    }
    String a="aaa",b="bbb",c="ccc";
    long d=1,e=2,f=3;
    String data;

    public void timeStringAdd(int repeats) {
        for(int reps=0;reps<repeats;reps++) {
            data=a+"+" +b+"+" +c+"+" +d+"+" +e+"+" +f;
        }
    }

    public void timeFormat(int repeats) {
        for(int reps=0;reps<repeats;reps++) {
            data=String.format("%s %s %s %d %d %d",a,b,c,d,e,f);
        }
    }
}
```
- `String`을 더하는 문장은 다음과 같이 변환된다.
```java
(new StringBuilder(String.valueOf(a))).append("*")
.append(b).append(" ").append(c).append(" ")
.append(d).append(" ").append(e).append(" ").append(f).toString();
```
- `format()` 메서드를 사용하는 문장은 다음과 같이 변환된다.
```java
data = String.format("% %s %s %d% %d %d", new Object[] {
    a, b, c, Long. valuef(d), Long. valueOf(e), Long.valueof(f)
});
```
- 컴파일시 변환된 부분을 보면 새로운 Object 배열을 생성하여 그 값을 배열에 포함시키도록 되어있다.
- 게다가 long 값을 Object 형태로 나타내기 위해서 Long 클래스의 valueOf()` 메서드를 사용하고 있다.
- `Formatter` 클래스에서는 %가 들어가 있는 `format` 문자열을 항상 파싱(parsing)하기 때문에 문자열을 그냥 더하는 것보다 성능이 좋을 수 없다.
## 로그를 더 간결하 게 처리하는 방법
- 디버그용 로그가 꼭 필요할 때는 어떻게 해야 할까?
- 로거(Logger)를 사용하여 로그를 처리하는 것이다.
- 로거를 사용하기 힘든 상황이라면 어떻게 해야 할까?
- 자체 로거클래스를 만드는 방법과 시스템로그를 컴파일 할 때 삭제되도록 하는 방법이다. 자체 로거클래스를 만드는 방법보다는 이미 만들어져 있는 로거를 사용하는 것이 훨씬 효율적이므로, 이 방법은 추천하고 싶지 않다.
- 파일에 쓰는 로그는 Log4j나 JDK에서 제공하는 로거를 사용하는 것이 직접 구현하는 것보다 낫다.
```java
public class LogRemoveSample {
    public LogRemoveSample() {
    }
    public ArrayList getList() {
        ArrayList retList=new ArrayList(10);
        //중간생략
        System.out.format("LogRemoveSample.getList(): size=%d\n",
                          retList.size());
        return retList;
    }
}
```
- 클래스가 컴파일될 때 시스템 로그가 삭제되도록 하려면 아래와 같이 해볼 수 있다.
```java
public class LogRemoveSample {
    private final boolean printFlag=false;
    public LogRemoveSample() {}
    public ArrayList getList() {
        ArrayList retList=new ArrayList(10);
        //중간생략
        if(printFlag) {
            System.out.format("LogRemoveSample.getList(): size=%d\n",
                              retList.size());
        }
        return retList;
    }
}
```
- 두 줄 이상에 걸쳐서 시스템 로그를 프린트하는 부분을 그냥 일괄 변경해서 주석 처리하면, 100% 컴파일 오류가 발생하기 때문이다.
- 이 소스를 컴파일한 클래스를 역컴파일해 보면, if 문장과 그 안에 있는 문장은 찾을 수가 없을 것이다.
- 이미 컴파일러에서 그 부분은 실행 시 필요가 없다고 생각하고 삭제하기 때문이다.
- 대신 이렇게 해 놓으면, 모든 소스를 찾아 다니면서 printFlag를 변경해 주어야 하는 단점이 있다. 그래서 다음과 같이 간단한 flag 정보를 갖는 클래스를 만들어 관리하면 약간 더 편리하다.
```java
public class LogFlag {
    public static final boolean printFlag=false;
}
```
```java
if(LogFlag.printFlag) {
    System.out.format("LogRemoveSample.getList() : size=%d\n",retList.size());
}
```
- 매번 if 문장으로 막는 것보다 간단하게 사용하기 위해서는 좀더 보완을 해서 다음과 같이 클래스를 만들면 된다.
- 이 소스처럼 만들면 System.out.println()를 매번 입력하거나 복사해서 붙여넣을 필요가 없다.
- 또한 `printFlag`에 따라서 로그를
- 남길지, 남기지 않을지를 결정할 수 있다. 다만 이 소스의 단점은 `printFlag`를 수정하기 위해서 다시 컴파일해야 한다는 점과 어차피 log() 메서드 요청을 하기 위해서 메시지 문자열을 생성해야 한다는 것이다.
## 로거 사용 시의 문제
- 컴파일 시에 로그를 제거하는 방법을 사용하지 않는한, 로그를 프린트하든 하지 않든, 로그를 삭제하기 위한 한 줄을 처리하기 위해서는 어차피 객체를 생성해야 한다.
- 즉 운영시 로그레벨을 올려놓는다고 해도, 디버그용 로그 메시지는 간단한 문자든 간단한 쿼리든 상관없이 하나이상의 객체가 필요하다.
```java
logger.info("query="+query);
logger.info("result="+resultHashMap);
```
- 로그를 이렇게 처리하면, 분명히 info() 메서드가 호출이 될 것이다.
- 또한 호출되는 메서드에 문자열이 전달되어야하기 때문에 괄호안에 있는 값들을 문자열로 변환하는 작업이 반드시 수행된 다음, 메서드 호출이 진행된다.
- 두 번째에 있는 `resultHashMap`과 같이 `HashMap`의 더하기 연산을 위해서는 `HashMap`에 있는 모든 데이터를 확인한다.
- 그리고 문자열로 더하는 작업을 마친 후에 "result="라는 문자열과 더하는 연산이 수행된다.
- 가장 좋은 방법은 디버그용 로그를 제거하는 것이다. 하지만 그렇지 못한 것이 현실이다.
- 그래서 이 경우에는 시스템 로그의 경우처럼 로그처리여부를 처리하는 것이 좋다.
```java
if(logger.isLoggable(Level.INFO)) {
    //로그처리
}
```
- 이렇게 if 문장으로 처리하면 로그를 위한 불필요한 메모리 사용을 줄일 수 있어, 더 효율적으로 메시지를 처리할 수 있다.
## 로그를 깔끔하게 처리하게 도와주는 Slf4j와 LogBack
```java
public class Wombat {
    final Logger logger = LoggerFactory.getLogger(Wombat.class);
    Integer t;
    Integer oldT;
    public void setTemperature(Integer temperature) {
        oldT = t;
        t = temperature;

        logger.debug("Temperature set to {}. Old temperature was {}.",
                     t, oldT);
        if(temperature.intValue() > 50) {
            logger.info("Temperature has risen above 50 degrees.");
        }
    }
}
```
- 기존의 로거들은 앞 절에서 이야기한대로 출력을 위해서 문자열을 더해 전달해줘야만 했다.
- 하지만, slf4j는 `format` 문자열에 중괄호를 넣고, 그 순서대로 출력하고자 하는 데이터들을 콤마로 구분하여 전달해준다.
- 이렇게 전달해 주면 로그를 출력하지 않을 경우 필요없는 문자열 더하기 연산이 발생하지 않는다.
- 게다가 slf4j는 자바의 기본 로거를 비롯하여 Log4j, 아파치 commons 로깅 등과 연계하여 사용할 수 있도록 되어있다.
- LogBack 로거는 예외의 스텍정보를 출력할 때 해당 클래스가 어떤 라이브러리(jar 파일)를 참고하고 있는지도 포함하여 제공하기 때문에 쉽게 관련된 클래스를 확인할 수 있다.
## 예외 처리도 이렇게
- 예외가 발생하면 `Exception` 클래스에 기본 정보가 전달된다.
- 만약 `e.printStackTrace()`를 호출하게 되면 스택정보를 확인하고, 확인된 정보를 콘솔에 프린트한다.
- 콘솔에 찍힌 이 로그를 알아보기가 힘들다. 왜냐하면 여러 스레드에서 콘솔에 로그를 프린트하면 데이터가 섞이기 때문이다.
- 자바의 예외 스택 정보는 로그를 최대 100개까지 프린트하기 때문에 서버의 성능에도 많은 부하를 준다.
- 스택 정보를 가져오는 부분에서는 거의 90% 이상이 CPU를 사용하는 시간이고, 나머지 프린트하는 부분에서는 대기시간이 소요된다. 
- 그래도 `printStackTrace()`에서 출력해주는 데이터가 필요할 때가 있다.
- 예외를 메시지로 처리하면 실제 사용자들은 한 줄의 오류 메시지나 오류 코드만을 보게 되기 때문에 장애를 처리하기가 쉽지 않기 때문이다.
```java
try {
    ...
} catch (Exception e) {
    StackTraceElement[] ste=e.getStackTrace();
    String className=ste[0].getClassName();
    String methodName=ste[0].getMethodName();
    int lineNumber=ste[0].getLineNumber();
    String fileName=ste[0].getFileName();
    logger.severe("Exception : "+e.getMessage());
    logger.severe(className+"."+methodName+""+fileName+""+lineNumber+" line");
}
```