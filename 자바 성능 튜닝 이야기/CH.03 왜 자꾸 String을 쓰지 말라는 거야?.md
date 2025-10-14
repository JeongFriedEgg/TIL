# CH.03 왜 자꾸 String을 쓰지 말라는 거야?
- GC는 Garbage Collection 의 약자로, 자바에서 사용하는 한정된 공간의 메모리가 꽉 찼을 때 더 이상 필요없는 객체들을 제거하는 작업을 의미한다.
## String 클래스를 잘못 사용한 사례
- 자바에서 가장 많이 사용하는 객체 1위는 `String` 클래스, 2위는 `Collection` 관련 클래스라고 생각한다.
```java
String strSQL = "";
strSQL += "select * ";
strSQL += "from ( ";
strSQL += "select A_column, ";
strSQL += "B_column, ";
```
- 요즘은 myBatis, Hibernate와 같은 데이터 매핑 프레임워크를 사용하지만, 예전에는 보통 이렇게 쿼리를 작성했다.
- 이렇게 쿼리를 작성하면, 개발시에는 좀 편할지 몰라도 메모리를 많이 사용하게 된다는 문제가 있다.
## StringBuffer 클래스와 StringBuilder 클래스
- `StringBuffer` 클래스나 `StringBuilder` 클래스에서 제공하는 메서드는 동일하다.
- `StringBuffer` 클래스는 스레드에 안전하게(ThreadSafe) 설계되어 있으므로, 여러 개의 스레드에서 하나의 `StringBuffer` 객체를 처리해도 전혀 문제가 되지 않는다.
- 하지만 `StringBuilder`는 단일 스레드에서의 안전성만을 보장한다.
- 여러 개의 스레드에서 하나의 StringBuilder 객체를 처리하면 문제가 발생한다.

| 생성자                               |                                                      |
|-----------------------------------|------------------------------------------------------|
| `StringBuffer()`                  | 아무 값도 없는 StringBuffer 객체를 생성한다. 기본 용량은 16개의 char이다.  |
| `StringBuffer(CharSequence seq)`  | CharSequence를 매개변수로 받아 그 seq 값을 갖는 StringBuffer를 생성한다. |
| `StringBuffer(int capacity)`      | capacity에 지정한 만큼의 용량을 갖는 StringBuffer를 생성한다.         |
| `StringBuffer (String str)`       | str의 값을 갖는 StringBuffer를 생성한다.                       |
- CharSequence는 인터페이스이다.
- 이 인터페이스를 구현한 클래스로는 `CharBuffer`, `String`, `StringBuffer`, `StringBuilder`가 있다.
- StringBuffer나 StringBuilder로 값을 만든 후 굳이 `toString`을 수행하여 필요없는 객체를 만들어서 넘겨주기보다는 `CharSequence`로 받아서 처리하는 것이 메모리 효율에 더 좋다.
- `append()` 메서드는 말 그대로 기존값의 맨 끝 자리에 넘어온 값을 덧붙이는 작업을 수행하고, insert() 메서드는 지정된 위치 이후에 넘어온 값을 덧붙이는 작업을 수행한다.
- `append()` 메서드 내에서 +를 이용해 문자열을 더하면 `StringBuffer`를 사용하는 효과가 전혀 없다.
- 그러므로 되도록이면 `append()` 메서드를 이용하여 문자열을 더하기 바란다.
## String vs StringBuffer vs StringBuilder
```java

final String AValue="abcde";

for(int outLoop=0;outLoop<10;outLoop++) {
    
    String a = new String();
    StringBuffer b = new StringBuffer();
    StringBuilder c = new StringBuilder();

    for(int loop=0;loop<10000;loop++) {
        a+=AValue;
    }

    for(int loop=0;loop<10000;loop++) {
        b.append(AValue);
    }

    String temp = b.toString();
    
    for(int loop=0;loop<10000;loop++) {
        c.append(AValue);
    }

    String temp2 = c.toString();
}

System.out.println("OK");
System.out.println(System.currentTimeMillis());
```
- 소스를 JSP로 만든 이유는 이 코드를 java 파일로 만들어 반복작업을 수행할 경우, 클래스를 메모리로 로딩하는데 소요되는 시간이 발생하기 때문이다.
- 회계 프로그램과 같이 쿼리가 굉장히 복잡한 시스템의 경우 보통 퀴리 하나가 적어도 4~5페이지 정도 된다.
- 일부 시스템의 경우는 기존 웹 모듈을 사용하기 위해서 배치(batch) 프로그램을 자바 기반으로 작성하는 경우도 많다.

| 주요 소스 부분                                      | 응답 시간 (ms)       | 비고    |
|-----------------------------------------------|------------------|-------|
| `a+=aValue;`                                  | 95,801.41ms      | 95초   |
| `b.append(aValue); String temp=b.toString();` | 247.48ms 14.21ms | 0.24초 |
| `c.append(aValue); String temp2=b.toString();` | 174.17ms 13.38ms | 0.17초 |

|주요 소스 부분| 메모리 사용량(bytes)        |생성된 임시 객체수| 비고             |
|---|-----------------------|-----------|----------------|
|`a+=aValue;`| 100,102,000,000       | 4,000,000 | 약 95Gb         |
|`b.append(aValue); String temp=b.toString();`| 29,493,600 10,004,000 | 1,200 200 | 약 28Mb 약 9.5Mb |
|`c.append(aValue); String temp2=b.toString();`| 29,493,600 10,004,000 | 1200 200  | 약 28Mb 약 9.5Mb |
- a에 aValue를 더하면 새로운 String 클래스의 객체가 만들어지고, 이전에 있던 a객체는 필요없는 쓰레기 값이 되어 GC 대상이 되어 버린다.
- 값이 더해지기 전의 a객체는 a+=aValue;를 수행하면서 사라지고(쓰레기가 되고), 새로운 주소와 'abcdeabcde'라는 값을 갖는 a객체가 생성된다.
- 이런 작업이 반복 수행되면서 메모리를 많이 사용하게 되고, 웅답 속도에도 많은 영향을 미치게 된다.
- 앞에서도 보았지만, GC를 하면 할수록 시스템의 CPU를 사용하게 되고 시간도 많이 소요된다.
- `StringBuffer`나 `StringBuilder`는 `String`과는 다르게 새로운 객체를 생성하지 않고, 기존에 있는 객체의 크기를 증가시키면서 값을 더한다.
- String은 짧은 문자열을 더할 경우 사용한다.
- `StringBuffer`는 스레드에 안전한 프로그램이 필요할 때나, 개발중인 시스템의 부분이 스레드에 안전한지 모를 경우 사용하면 좋다.
- 만약 클래스에 static 으로 선언한 문자열을 변경하거나, singleton으로 선언된 클래스(VM에 객체가 하나만 생성되는 클래스)에 선언된 문자열일 경우에는 이 클래스를 사용해야만 한다.
- `StringBuilder`는 스레드에 안전한지의 여부와 전혀 관계없는 프로그램을 개발할 때 사용하면 좋다.
- 만약 메서드 내에 변수를 선언했다면, 해당 변수는 그 메서드 내에서만 살아있으므로, `StringBuilder`를 사용하면 된다.
## 버전에 따른 차이