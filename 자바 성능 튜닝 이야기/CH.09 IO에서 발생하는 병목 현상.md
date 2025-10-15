# CH.09 IO에서 발생하는 병목 현상
## 기본적인 IO는 이렇게 처리한다
- 자바에서 입력과 출력은 스트림(stream)을 통해서 이루어진다.
- 일반적으로 IO라고 하면 파일 IO만을 생각할 수 있는데, 어떤 디바이스를 통해 이뤄지는 작업을 모두 IO라고 한다.
- 네트워크를 통해서 다른 서버로 데이터를 전송하거나, 다른 서버로부터 데이터를 전송받는 것도 IO에 포함된다.
- IO는 성능에 영향을 가장 많이 미친다.
- IO에서 발생하는 시간은 CPU를 사용하는 시간과 대기시간 중 대기시간에 속하기 때문이다.
- IO와 관련된 디바이스가 느리면 느릴수록 애플리케이션의 속도는 느려진다.
- 입력과 관련된 스트림들은 `java.io.InputStream` 클래스로부터 상속받았다.
  - `ByteArrayInputStream`: 바이트로 구성된 배열을 읽어서 입력 스트림을 만든다. 
  - `FileInputStream`: 이미지와 같은 바이너리 기반의 파일의 스트림을 만든다.
  - `FilterInputStream`: 여러 종류의 유용한 입력 스트림의 추상 클래스이다.
  - `ObjectInputStream`: `ObjectOutputStream`을 통해서 저장해 놓은 객체를 읽기 위한 스트림을 만든다.
  - `PipedInputStream`: `PipedOutputStream`을 통해서 출력된 스트림을 읽어서 처리하기 위한 스트림을 만든다.
  - `SequenceInputStream`: 별개인 두 개의 스트림을 하나의 스트림으로 만든다.
- 문자열 기반의 스트림을 읽기 위해서 사용하는 클래스는 이와는 다르게 `java.io.Reader` 클래스의 하위 클래스들이다.
  - `BufferedReader`: 문자열 입력 스트림을 버퍼에 담아서 처리한다. 일반적으로 문자열 기반의 파일을 읽을 때 가장 많이 사용된다.
  - `CharArrayReader`: char의 배열로 된 문자 배열을 처리한다.
  - `FilterReader`: 문자열 기반의 스트림을 처리하기 위한 추상 클래스이다.
  - `FileReader`: 문자열 기반의 파일을 읽기 위한 클래스이다.
  - `InputStreamReader`: 바이트 기반의 스트림을 문자열 기반의 스트림으로 연결하는 역할을 수행한다.
  - `PipedReader`: 파이프 스트림을 읽는다.
  - `StringReader`: 문자열 기반의 소스를 읽는다.
- 바이트 단위로 읽거나, 문자열 단위로 읽을 때 중요한 것은 한번 연(open한) 스트림은 반드시 닫아주어야 한다는 것이다.
- 스트림을 닫지 않으면 나중에 리소스가 부족해질 수 있다.
```java
public class BasicIOReadUtil {
    public static ArrayList readCharStream(String fileName)throws Exception{
        ArrayList<StringBuffer> list=new ArrayList<StringBuffer>();
        FileReader fr=null;
        try {
            fr=new FileReader(fileName);//FileReader 객체 생성
            int data=0;
            // 한 줄씩 데이터를 담을 StringBuffer 생성
            StringBuffer sb=new StringBuffer();
            while((data=fr.read())!=-1) {
                if(data=='\n' || data=='\r') {
                    list.add(sb);
                    sb=new StringBuffer();
                } else {
                    sb.append((char)data);
                }
            }
        } catch (IOException e) {
            System.err.println(e.getMessage());
            throw e;
        } catch (Exception e) {
            System.err.println(e.getMessage());
            throw e;
        } finally {
            if(fr!=null) fr.close();
        }
        return list;
    }

    public static void main(String args[]) throws Exception{
        String fileName="C:\\10MBFile";
        Stopwatch sw=new Stopwatch();
        sw.start();
        ArrayList list1=BasicIOReadUtil.readCharStream(fileName);
        System.out.println(sw);
        System.out.println(list1.size());
    }
}
```
- `readCharStream` 메서드는 지정된 파일을 받으면 해당 파일을 읽는다.
- 읽은 내용을 일단 `StringBuffer`에 담고, 줄이 바뀔 경우 `ArrayList`에 담아서 리턴하도록 되어있다.
- 파일을 처리할 때는 되도록이면 `IOException`을 따로 구분하여 처리하는 것이 좋다.
```java
public static String readCharStreamWithBuffer(String fileName) Exception{
    StringBuffer retSB=new StringBuffer();
    FileReader fr=null;
    try {
        fr=new FileReader(fileName);
        int bufferSize=1024*1024;
        char readBuffer[]=new char[bufferSize];
        int resultSize=0;
        while((resultSize=fr.read(readBuffer))!=-1) {
            if(resultSize==bufferSize) {
                retSB.append(readBuffer);
            } else {
                for(int loop=0;loop<resultSize;loop++) {
                    retSB.append(readBuffer[loop]);
                }
            }
        }
        // 이하 예외 처리 생략
        return retSB.toString();
    }
}
```
- 특정 배열에 읽은 데이터를 저장한 후 그 데이터를 사용하면, 더 빠르게 처리할 수 있다.
- 매개변수가 있는 read 메서드에서는 파일에서 읽은 Char 배열의 개수가 리턴된다.
- 이 메서드를 수행하면 약 400ms로 양호한 응답속도가 나온다.
- char의 배열을 버퍼로 사용했기 때문에, 해당 배열을 초기화하지 않으면 리턴되는 문자열에 필요하지 않은 값들이 포함된다.
- 간단하게 `FileReader`를 사용하여 파일을 읽었는데, 이러한 방식은 별로 사용되지 않는다.
- 문자열 단위로 읽는 것은 굉장히 비효율적이기 때문이다.
- 문자열 단위로 읽는 방식에 대한 해결 방법에는 `BufferedReader` 클래스가 있다.
```java
public static ArrayList<String> readBufferedReader(String fileName)
throws Exception{
    ArrayList<String> list=new ArrayList<String>();
    BufferedReader br=null;
    try {
        br=new BufferedReader(new FileReader(fileName));
        String data;
        while((data=br.readLine())!=null) {
            list.add(data);
        }
    } catch (Exception e) {
        System.err.println(e.getMessage());
        throw e;
    } finally {
        if(br!=null) br.close();
    }
    return list;
}
```
- `BufferedReader` 클래스는 다른 `FileReader` 클래스와 마찬가지로 문자열 단위나 문자열 배열 단위로 읽을 수 있는 기능을 제공하지만, 추가로 라인단위로 읽을 수 있는 `readLine` 메서드를 제공한다.
- 실제 응답 속도도 약 350ms로, 약간 빨라진다.

||버퍼 없이 FileReader|버퍼 포함한 FileReader| BufferedReader 사용시 |
|---|---|---|-------------------|
|응답 속도|2,480ms|400ms| 350ms             |
## IO에서 병목이 발생한 사례
- 사용자의 요청이 발생할 때마다 매번 파일을 읽도록 되어 있는 시스템도 있다.
```java
String configUrl;
public Vector getRoute(String type) {
    if(configUrl == null) {
        configUrl = this.getClass().getResource("/xxx/config.xml");
    }
    Object obj = new DaoUtility(configUrl, "1");
    // ...
}
```
- 이 소스는 어떤 경로를 확인하는 시스템의 일부이다.
- 경로 하나를 가져오기 위해서 매번 configUrl을 DaoUtility에 넘겨준다.
- DaoUtility에서는 요청이 올 때마다 config.xml 파일을 읽고 파싱하여 관련 DB 쿼리 데이터를 읽는다.
- 서버에는 엄청난 IO가 발생할 것이며, 응답시간이 좋지 않으리라는 점도 쉽게 예상할 수 있다.
## 그럼 NIO의 원리는 어떻게 되는 거지?
- 자바를 사용하여 하드 디스크에 있는 데이터를 읽을 때의 프로세스
  1. 파일을 읽으라는 메서드를 자바에 전달한다. 
  2. 파일명을 전달받은 메서드가 운영체제의 커널에게 파일을 읽어달라고 요청한다.
  3. 커널이 하드 디스크로부터 파일을 읽어서 자신의 커널에 있는 버퍼에 복사하는 작업을 수행한다. DMA에서 이 작업을 하게 된다. 
  4. 자바에서는 마음대로 커널의 버퍼를 사용하지 못하므로, JVM으로 그 데이터를 전달한다.
  5. JVM에서 메서드에 있는 스트림 관리 클래스를 사용하여 데이터를 처리한다.
- 자바에서는 3번 복사작업을 할 때에나 4번 전달작업을 수행할 때 대기하는 시간이 발생할 수밖에 없다.
- 이러한 단점을 보완하기 위해서 NIO가 탄생했다.
- 3번 작업을 자바에서 직접 통제하여 시간을 더 단축할 수 있게 한 것이다.
- NIO
  - 버퍼의 도입
  - 채널의 도입
  - 문자열의 엔코더와 디코더 제공
  - Perl 스타일의 정규 표현식에 기초한 패턴 매칭 방법 제공
  - 파일을 잠그거나 메모리 매핑이 가능한 파일 인터페이스 제공
  - 서버를 위한 복합적인 Non-blocking IO 제공
## DirectByteBuffer를 잘못 사용하여 문제가 발생한 사례
- `ByteBuffer`는 네트워크나 파일에 있는 데이터를 읽어들일 때 사용한다.
- `ByteBuffer` 객체를 생성하는 메서드에는 `wrap()`, `allocate()`, `allocateDirect()`가 있다.
- `allocateDirect()` 메서드는 데이터를 자바 JVM에 올려서 사용하는 것이 아니라, OS 메모리에 할당된 메모리를 Native한 JNI로 처리하는 `DirectByteBuffer` 객체를 생성한다.
```java
public class DirectByteBufferCheck {
    public static void main(String[] args) {
        DirectByteBufferCheck check = new DirectByteBufferCheck();
        for (int loop = 1; loop < 1024000; loop++) {
            check.getDirectByteBuffer();
            if (loop % 100 == 0) {
                System.out.println(loop);
            }
        }
    }

    public ByteBuffer getDirectByteBuffer() {
        ByteBuffer buffer;
        buffer = ByteBuffer.allocateDirect(65536);
        return buffer;
    }
}
```
- `getDirectByteBuffer()` 메서드에서는 `ByteBuffer` 클래스의 `allocateDirect()` 메서드를 호출함으로써 `DirectByteBuffer` 객체를 생성한 후 리턴해 준다.
```text
$ jstat -gcutil 5376 5s
  S0     S1     E      O      P       YGC    YGCT     FGC    FGCT     GCT
 0.00   0.00   4.00   5.01  11.58       4    0.107     4    0.287    0.394
 0.00   0.00   2.00   5.01  11.58       5    0.131     5    0.351    0.482
 0.00   0.00  10.00   5.01  11.63       5    0.131     5    0.351    0.482
 0.00   0.00  10.00   5.01  11.63       6    0.154     6    0.405    0.559
 0.00   0.00  10.00   5.01  11.63       7    0.177     7    0.461    0.638
```
- FGC 값을 보면 거의 5~10초에 한 번씩 Full GC가 발생하는 것을 볼 수 있다.
- 그런데 왼쪽에서 네 번째에 있는 O라고 되어 있는 Old 영역의 메모리는 중가하지 않는다.
- 그 이유는 `DirectByteBuffer`의 생성자 때문이다.
- JVM에 있는 코드에 `System.gc()` 메서드가 있기 때문에 해당 생성자가 무차별적으로 생성될 경우 GC가 자주 발생하고 성능에 영향을 줄 수밖에 없다.
- 가능하다면 singleton 패턴을 사용하여 해당 JVM에는 하나의 객체만 생성하도록 하는 것을 권장한다.
## lastModified() 메서드의 성능 저하
- File 클래스에 있는 `lastModified()`라는 메서드를 사용해왔다.
- 이 메서드를 사용하면 최종 수정된 시간을 밀리초 단위로 제공한다.
- lastModified() 메서드 처리 절차
  1. `System.getSecurityManager()` 메서드를 호출하여 `SecurityManager` 객체 얻어옴
  2. 만약 null이 아니면 `SecurityManager` 객체의 `checkRead()` 메서드 수행 
  3. File 클래스 내부에 있는 `FileSystem`이라는 클래스의 객체에서 `getLastModifiedTime()` 메서드를 수행하여 결과 리턴
- 하지만 이 작업을 반복하는 형태의 서비스를 제공한다면 이야기는 달라진다.
- 이 작업은 IO 작업을 수반하기 때문에 OS의 IO의 영향을 많이 받을 수 밖에 없다.

```java
public class WatcherThread extends Thread{
    String dirName;
    public WatcherThread(String dirName) {
        this.dirName=dirName;
    }

    public void run() {
        System.out.println("Watcher is started");
        filewatcher();
        System.out.println("Watcher is ended");
    }
    public void filewatcher() {
        try {
            Path dir = Paths.get(dirName);
            WatchService watcher = FileSystems.getDefault().newWatchService();
            dir.register(watcher, ENTRY_CREATE, ENTRY_DELETE, ENTRY_MODIFY);

            WatchKey key;

            for(int loop=0;loop<4;loop++) {
                key = watcher.take();
                String watchedTime=new Date().toString();
                List<WatchEvent<?>> eventList = key.pollEvents();
                for (WatchEvent<?> event : eventList) {
                    Path name = (Path) event.context();
                    if (event.kind() == ENTRY_CREATE) {
                        // Do something when created
                        System.out.format("%s created at %s%n", name,
                                watchedTime);
                    } else if (event.kind() == ENTRY_DELETE) {
                        // Do something when deleted
                        System.out.format("%s deleted at %s%n", name,
                                watchedTime);
                    } else if (event.kind() == ENTRY_MODIFY) {
                        // Do something when modified
                        System.out.format("%s modified at %s%n", name,
                                watchedTime);
                    }
                }
                key.reset();
            }
        } catch (Exception e) {
            // catch 블록 내용 생략
        }
    }
}
```

1. Path 객체를 생성해서 모니터링할 디렉터리를 지정한다.
2. WatchService 클래스의 watcher라는 객체를 생성한다.
3. dir이라는 Path 객체의 register라는 메서드를 활용하여 파일이 생성, 수정, 삭제되는 이벤트를 처리하도록 지정하였다.
4. watcher 객체의 take() 메서드를 호출하면 해당 디렉터리에 변경이 있을 때까지 기다리다가, 작업이 발견되면 key라는 WatchKey클래스의 객체가 생성된다. 마치 Socket 관련 객체에 accept() 메서드처럼 어떤 이벤트가 생길 때까지 낚시줄을 던져놓고 기다리고 있는 상황이라고 생각하면 된다. 
5. 파일에 변화가 생겼다면 이벤트의 목록을 가져온다. 
6. 이벤트를 처리한 다음에 key 객체를 reset한다.