# CH.18 GC가 어떻게 수행되고 있는지 보고 싶다.
## 자바 인스턴스 확인을 위한 jps
- jps는 해당 머신에서 운영중인 JVM의 목록을 보여준다.
```text
jps [-q] [-mlvV] [-Joption] [‹hostid›]
```
- -q: 클래스나 JAR 파일명, 인수등을 생략하고 내용을 나타낸다(단지 프로세스 id만 나타난다).
- -m: main 메서드에 지정한 인수들을 나타낸다.
- -l: 애플리케이션의 main 클래스나 애플리케이션 JAR 파일의 전체 경로 이름을 나타낸다.
- -V: JVM에 전달된 자바 옵션목록을 나타낸다.
- -V: JVM의 플래그 파일을 통해 전달된 인수를 나타낸다.
- -Joption: 자바옵션을 이 옵션뒤에 지정할 수 있다.
- 아무 옵션없이 jps를 입력하면 현재 서버에서 수행되고 있는 자바 인스턴스들의 목록이 나타난다.
## GC 상황을 확인하는 jstat
- jstat는 GC가 수행되는 정보를 확인하기 위한 명령어이다.
```text
C:\jdk1.5\bin>jstat -gcnew 2464 1000

  SOC      S1C      SOU    S1U      TT   MTT  DSS      EC       EU       YGC   YGCT
960.0    960.0    710.5    0.0       1   15  480.0    8128.0   2438.6     6    0.089
960.0    960.0    710.5    0.0       1   15  480.0    8128.0   2438.6     6    0.089
960.0    960.0    710.5    0.0       1   15  480.0    8128.0   2438.6     6    0.089
```
```text
jstat -<option> [-t] [-h<lines>] <vmid› [<interval> [<count>]]
```
- -: 수행 시간을 표시한다.
- -h:lines: 각 열의 설명을 지정된 라인 주기로 표시한다.
- interval: 로그를 남기는 시간의 차이(밀리초 단위임)를 의미한다.
- count: 로그 남기는 횟수를 의미한다.
- 〈option〉의 종류
  - class: 클래스 로더에 대한 통계
  - compiler: 핫스팟 JIT 컴파일러에 대한 통계
  - gc: GC 힙 영역에 대한 통계
  - gccapacity: 각 영역의 허용치와 연관된 영역에 대한 통계
  - gccause: GC의 요약 정보와, 마지막 GC와 현재 CC에 대한 통계
  - gcnew: 각 영역에 대한 통계
  - gcnewcapacity: Young 영역과 관련된 영역에 대한 통계
  - gcold: Old와 Perm 영역에 대한 통계
  - gcoldcapacity: Old 영역의 크기에 대한 통계
  - gcpermcapacity: Perm 영역의 크기에 대한 통계
  - gcutil: GC에 대한 요약정보
  - printcompilation: 핫스팟 컴파일 메서드에 대한 통계
- jstat에서 프린트되는 결과를 사용하여 그래프를 그리면 GC가 처리되는 추이를 알아볼 수 있으므로 편리하다.
- jstat을 로그로 남겨 분석하는 데는 한계가 있다.
- 로그를 남기는 주기에 GC가 한 번 발생할 수도 있고, 10번 발생할수도 있기 때문이다.
- 정확한 분석을 하고자 할 때는 뒷부분에 있는 verbosegc 옵션 사용을 권장한다.
## jstat 명령에서 GC튜닝을 위해서 가장 유용한 옵션은 두 개
- 가장 애용하는 옵션은 -gcutil과 -gccapacity이다.
```java
public class GCMaker {

    public static void main(String[] args) throws Exception{
        GCMaker maker=new GCMaker();
        for(int loop=0;loop<120;loop++) {
            maker.makeObject();
            Thread.sleep(1000);
            System.out.print(".");
        }
    }
    private void makeObject() {
        Integer[] intArr=new Integer[1024000];
        ArrayList<Integer> list=new ArrayList<Integer>(1024000);
        for(int loop=0;loop<1024;loop++) {
            intArr[loop]=loop;
            list.add(loop);
        }
    }
}
```
- gccapacity 옵션은 현재 각 영역에 할당되어 있는 메모리의 크기를 KB 단위로 나타낸다.
```text
$ jstat -gccapacity 3580

NGCMN  NGCMX  NGC    SOC    S1C    EC     OGCMN  OGCMX
OGC    OC     PGCMN  PGCMX  PGC    PC     YGC    FGC
43072.0 689472.0 40832.0 5632.0 64.0 29568.0 86144.0 1379008.0
86144.0 86144.0 21248.0 83968.0 21248.0 21248.0 7 0
```
- NGC로 시작하는 것은 New (Young) 영역의 크기 관련, OGC로 시작하는 것은 Old 영역 크기 관련, PGC로 시작하는 것은 Perm 영역 크기 관련 정보이다.
- S0C, S1C, EC, OC, PC는 각각 Survivor0, Survivor1, Eden, Old, Perm 영역의 현재 할당된 크기를 나타낸다.
- MN, MX, C로 끝나는 항목들은 각각 Min, Max, Committed를 의미한다.
- gcutil 옵션은 힙 영역의 사용량을 %로 보여준다.
```text
$ jstat -gcutil 3580 1s

S0     S1     E      O      P      YGC    YGCT   FGC    FGCT   GCT
0.00   9.40   99.46  0.01   11.89  3      0.020  0      0.000  0.020
10.00  0.00   25.05  0.01   11.89  4      0.023  0      0.000  0.023
10.00  0.00   49.80  0.01   11.89  4      0.023  0      0.000  0.023
```
- YGC은 Young 영역의 GC 횟수, YGCT는 Young 영역의 GC가 수행된 누적시간(초)이다.
- O는 Old, P는 Perm 영역을 의미하며, 이 두개 영역중 하나라도 GC가 발생하면 FGC의 횟수가 중가하고, FGCT 시간이 올라가게 된다.
- GCT는 Young GC가 수행된 시간인 YGCT와 Full GC가 수행된 시간인 FGCT의 합이다.
## 원격으로 JVM 상황을 모니터링하기 위한 jstatd
- jstatd 명령어를 사용하면 원격 모니터링을 할 수 있지만, 중지하면 서버가 가동 중일 경우에도 원격 모니터링이 불가능하다.
```text
jstatd [-nr] [-p port] [-n rminame]
```
- nr: RMI registry가 존재하지 않을 경우 새로운 RMI 레지스트리를 jstatd 프로세스 내에서 시작하지 않는 것을 정의하기 위한 옵션이다.
- p: RMI 레지스트리를 식별하기 위한 포트번호.
- n : RMI 객체의 이름을 지정한다. 기본이름은 JStatRemoteHost이다.
```text
jstatd -J-Djava.security.policy=all.policy -p 2020
```
```text
C: \jdk1.5\bin>jps minspc: 2020
2904 Bootstrap
3500 Jps
```
- jps를 통하여 프로세스 아이디를 확인하였으니 이제 jstat를 사용하여 서버의 GC 상황을 모니터링 해보자.
```text
C:\jdk1.7\bin>jstat -gcutil 2904@minspc:2020 1000

S0     S1     E      O      P      YGC    YGCT   FGC    FGCT   GCT
36.63  0.00   85.32  81.87  68.61  10897  16.334 25     1.513  17.847
0.00   14.92  0.00   82.40  68.61  10935  16.377 25     1.513  17.890
15.15  0.00   0.00   82.40  68.61  11047  16.486 25     1.513  17.999
0.00   15.66  0.00   82.88  68.61  11156  16.597 25     1.513  18.110
```
- 2904는 프로세스 번호, minspc는 서버 명, 2020은 포트번호다.
- 원격지의 서버를 이 방식으로 사용하려면 해당 포트를 방화벽에서 열어주어야 한다.
## verbosegc 옵션을 이용하여 gc 로그 남기기
- GC를 분석하기 위한 명령어로는 가장 쉬울, verbosegc라는 옵션이 있다.
- 자바 수행시에 간단히 -verbosegc라는 옵션을 넣어주면 된다.
```text
[GC 8128K->848K(130112K), 0.0090257 secs]
```
- Young 영역에 마이너 GC가 발생했으며, 8,128kbyte에서 848kbyte로 축소되었다.
- 전체 할당된 크기는 130,112kbyte이며, GC 수행시간은 0.0090257초이다.
### PrintGCTimeStamps 옵션
- 이렇게 옵션을 주고 수행하면 언제 GC가 발생되었는지 알 수 없다.
- 이 경우를 대비해서 verbosegc 옵션과 함께 사용할 수 있는-XX:+PrintGCTime Stamps 옵션이 있다.
```text
-verbosegc -XX:+PrintGCTimeStamps 옵션 적용 후
0.668: [GC 8128K->848K(130112K), 0.0089713 secs]
1.073: [GC 8976K->1453K(130112K), 0.0088953 secs]
1.496: [GC 9581K->2242K(130112K), 0.0111185 secs]
1.857: [GC 10370K->2993K(130112K), 0.0095970 secs]
2.136: [GC 11121K->3699K(130112K), 0.0081142 secs]
5.911: [GC 11827K->4387K(130112K), 0.0163130 secs]
6.070: [GC 12511K->4594K(130112K), 0.0069344 secs]
6.085: [GC 12722K->4604K(130112K), 0.0012633 secs]
6.095: [GC 12727K->4592K(130112K), 0.0011932 secs]
```
- verbosegc 옵션만 적용했을 때와 비교하면, 가장 좌측에 수행한 시간이 포함되는 것이 눈에 띈다.
- 서버가 기동되기 시작한 이후부터 해당 GC가 수행될 때까지의 시간을 로그에 포함하기 때문에 언제 GC가 발생되었는지 확인할 수 있다.
### PrintHeapAtGC 옵션
```text
-verbosegc -XX:+PrintGCTimeStamps -XX:+PrintHeapAtGC 옵션 적용 후
{Heap before GC invocations=1 (full 0):
PSYoungGen      total 37696K, used 29307K [...]
  eden space 32320K, 90% used [...]
  from space 5376K, 0% used [...]
  to   space 5376K, 0% used [...]
ParOldGen       total 86144K, used 0K [...]
  object space 86144K, 0% used [...]
PSPermGen       total 21248K, used 2521K [...]
  object space 21248K, 11% used [...]
3.536: [GC 29307K->4649K(123840K), 0.0708125 secs]
Heap after GC invocations=1 (full 0):
PSYoungGen      total 37696K, used 4649K [...]
  eden space 32320K, 0% used [...]
  from space 5376K, 86% used [...]
  to   space 5376K, 0% used [...]
ParOldGen       total 86144K, used 0K [...]
  object space 86144K, 0% used [...]
PSPermGen       total 21248K, used 2521K [...]
  object space 21248K, 11% used [...]
}
```
- GC가 한 번 수행될 때 각 영역(Eden 영역, Survivor 영역, Old 영역, 전체 영역)에서 얼마나 많은 메모리 영역을 사용하고 있는지 상세하게 볼 수 있다.
- 이 결과의 특징은 Before라고 되어있는 블록에는 GC전의 메모리 상황을, After라고 되어있는 블록에는 GC 후의 메모리 상황을 제공한다는 것이다.
```text
-verbosegc-xx:+printGCTimestamps xx:+printGCDetails 옵션 적용 후
0.719: [GC 0.719: [DefNew: 8128K->848K(9088K), 0.0090986 secs]
8128K->848K(130112K), 0.0092344 secs]
•••
63.616: [Tenured: 112634K->9923K(121024K), 0.0633237 secs]
121416K->9923K(130112K), 0.0634597 secs]
```
- 각 서버에 알맞은 분석 툴
  - GC Analyzer
  - IBM GC 분석기
  - HPjtune
## 어설프게 아는 것이 제일 무섭다
- 정말 메모리릭이 있는 경우는 필자의 경험상 전체 자바 애플리케이션 1%도 안 된다.
- 분당 1건 정도의 요청이 들어오는 시스템은 하루에 Full GC가 한 번 일어날까 말까 할 수도 있다.
- 메모리를 2GB로 지정한 시스템에 초당 1건이 요청되는 시스템에서 한 번 요청이 올 때 10MB의 메모리가 생성된다고 가정하자.
- 이 시스템의 Old 영 역이 1% 증가하려면 얼마나 기다려야 할까? 
- 한 번 요청올 때 생성되는 10MB의 메모리는 Eden 영역에 쌓일 것이다.
- 이 데이터가 Survivor 영역으로 넘어가고 Old 영역까지 넘어갈 확률은 얼마나 될까? 
- 보통의 경우 JVM이 자동으로 지정해주는 Young 영역과 Old 영역의 비율은 1:2 ~ 1:9 정도다.
- 그러면, 2GB에서는 100~ 300MB 정도가 Young 영역에 할당될 것이다.
- 이 시스템의 Old 영역이 1% 증가하려면 얼마나 기다려야 할까?
- 정답은 없지만, 적어도 5분에서 2시간 정도 소요될 수 있다.
- 5분에 1%라면, 한 시간에 12%고, 9시간 정도가 되어야 100%에 도달하여 Full GC가 발생하게 될 것이다.
- 메모리 럭이 발생하는지 확인하는 가장 확실한 방법은 verbosegc를 남겨서 보는 방법이다.
- 그리고, 간단하게 확인할 수 있는 또 한가지 방법은 Full GC가 일어난 이후에 메모리 사용량을 보는 것이다.
- 정확하게 이야기해서 Full GC가 수행된 후에 Old 영역의 메모리 사용량을 보자.
- 만약 사용량이 80% 이상이면 메모리 릭을 의심해야 한다.
- stat과 verbosegc 로그 결과를 갖고 이야기하자.
- 그냥 보면 jstat의 Old 영역은 항상 올라가는 것이 정상이다.
- Full GC가 발생한 이후의 메모리 사용량으로 메모리 릭 여부를 판단하자.