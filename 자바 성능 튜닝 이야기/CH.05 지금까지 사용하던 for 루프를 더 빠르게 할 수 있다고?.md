# CH.05 지금까지 사용하던 for 루프를 더 빠르게 할 수 있다고?
## 조건문에서의 속도는?
- if문 안에는 boolean 형태의 결과 값만 사용할 수 있다.
- switch문은 JDK 6까지는 byte, short, char, int 이렇게 네 가지 타입을 사용한 조건분기만 가능했지만, JDK 7부터는 String도 사용 가능하다.
- 일반적으로 if문에서 분기를 많이 하면 시간이 많이 소요된다고 생각한다.
- if문 조건안에 들어가는 비교 구문에서 속도를 잡아먹지 않는 한, if 문장 자체에서는 그리 많은 시간이 소요되지 않는다.
```java
@State(Scope.Thread)
@BenchmarkMode({ Mode.AverageTime })
@OutputTimeUnit(TimeUnit.MICROSECONDS)
public class ConditionIf {
    int LOOP_COUNT=1000;

    @GenerateMicroBenchmark
    public void randomOnly() {
        Random random=new Random();
        int data=1000+random.nextInt();
        for(int loop=0; loop<LOOP_COUNT; loop++) {
            resultProcess("dummy");
        }
    }

    @GenerateMicroBenchmark
    public void if10() {
        Random random=new Random();
        String result=null;
        int data=1000+random.nextInt();
        for(int loop=0; loop<LOOP_COUNT; loop++) {
            if(data<50) { result="50"; }
            else if(data<150) { result="150"; }
            else if(data<250) { result="250"; }
            else if(data<350) { result="350"; }
            else if(data<450) { result="450"; }
            else if(data<550) { result="550"; }
            else if(data<650) { result="650"; }
            else if(data<750) { result="750"; }
            else if(data<850) { result="850"; }
            else if(data<950) { result="950"; }
            else { result="over"; }
            resultProcess(result);
        }
    }

    @GenerateMicroBenchmark
    public void if100() {
        Random random=new Random();
        String result=null;
        int data=1000+random.nextInt();
        for(int loop=0; loop<LOOP_COUNT; loop++) {
            if(data<50) { result="50"; }
            else if(data<150) { result="150"; }
            else if(data<250) { result="250"; }
            //중간에 총 100개의 if문은 생략했다.
            else if(data<9950) { result="9950"; }
            else { result="over"; }
            resultProcess(result);
        }
    }

    String current;
    public void resultProcess(String result) {
        current=result;
    }
}
```
- `randomOnly()` 메서드를 만든 이유는, if가 있는 경우와 없는 경우를 비교하기 위한 기준이 필요하기 때문이다.

| 대상 | 응답 시간 (마이크로초) |
|:---|---|
| `randomOnly` | 0.46 |
| if 10개 | 5 |
| if 100개 | 63 |
- 결과를 보면 if문 10개를 거치는 경우, 없을 때보다 10배의 시간이 소요된다.
- 100개일 경우에는 140배 이상의 시간이 더 소요된다.
- 이 예제 코드는 if문이 10개라 할지라도 LOOP_COUNT라는 반복횟수는 1,000이므로, 총 10,000번의 if문을 거친 결과가 if10()의 값이라는 점이다.
- 그러므로, if가 하나만 있을 경우에는 기존에 있는 코드 대비 약 "응답 시간/10,000"만큼 더 소요가 된다고 볼 수 있으므로 아주 큰 성능 저하가 발생한다고 보기는 어렵다.
- switch는 숫자 비교 시 if보다 가독성이 좋아지므로 정해져 있는 숫자로 분기를 할 때는 swtich를 권장한다.
```java
public int getMonthNumber(String str) {
    int month=-1;
    switch(str) {
        case "January": month=1;
        break;
        case "February": month=2;
        break;
        case "March": month=3;
        break;
        case "April": month=4;
        break;
        case "May": month=5;
        break;
        case "June": month=6;
        break;
        case "July": month=7;
        break;
        case "August": month=8;
        break;
        case "September": month=9;
        break;
        case "October": month=10;
            break;
        case "November": month=11;
            break;
        case "December": month=12;
            break;
    }
    return month;
}
```
- JDK 6까지만 해도 switch-case문에서는 주로 정수와 enum을 처리할 수 있었는데, 어떻게 JDK 7에서는 String을 비교할까?
- 그 답은 `int` 정수를 리턴하는 `Object` 클래스에 선언되어 있는 `hashCode`라는 메서드에 있다.
- `String`에서 `Overriding`한 `hashCode` 메서드는 문자열을 `int` 값으로 구분하여 switch-case 문에서 사용하는 것이다.
```java
public int getMonthNumber(java.lang.String);
Code:
    0: iconst_m1
    1: istore_2
    2: aload_1
    3: dup
    4: astore_3
    5: invokevirtual #20  // Method java/lang/String.hashCode:()I
    8: lookupswitch { // 12
        -199248958: 116
        -162006966: 128
        -25881420: 140
             77125: 152
           2320440: 164
           2320482: 176
          43165376: 188
        63478374: 200
        74113571: 212
        626483269: 224
        1703773522: 236
        1972131363: 248
        default: 324
    }
    116: aload_3
    117: ldc           #26         // String February
    119: invokevirtual #28         // Method java/lang/String.equals:(Ljava/lang/Object;)Z
    122: ifne          265
    125: goto          324
    128: aload_3
    129: ldc           #32         // String January
    131: invokevirtual #28         // Method java/lang/String.equals:(Ljava/lang/Object;)Z
    134: ifne          260
    137: goto          324
```
- 여기에서 중간에 굵은 글씨로 표현한 부분을 보면 숫자들이 나열된 것을 볼 수 있다.
- 이 숫자들이 January 부터 December까지를 hashCode() 메서드로 변환한 것이다.
- 컴파일하면서 case 문에 있는 각 값들을 hashCode로 변환하고, 그 값이 작은 것부터 정렬한 다음에 String의 equals() 메서드를 사용하여 실제 값과 동일한지 비교한다.
- 숫자들이 정렬되어 있다는 점이다. switch-case 문은 작은 숫자부터 큰 숫자를 비교하는 게 가장 빠르다.
- 대상이 되는 case의 수가 적으면 상관 없지만, 많으면 많을수록 switch-case에서 소요되는 시간이 오래 걸린다.
## 반복 구문에서의 속도는?
```java
public void test(ArrayList<String> list) {
    boolean flag=true;
    int idx=0;
    do {
        if(list.get(idx).equals("A")) flag=false;
    } while(flag);
}
```
- 만약 ArrayList v의 첫 번째 값이 'A'이면 정상적으로 수행이 되겠지만, 그렇지 않으면 해당 애플리케이션은 서버를 재시작하거나 스레드를 강제 종료시킬 때까지 계속 반복문을 수행할 것이다.
```java
for (int loop=0; loop<list.size(); loop++)
```
- 이렇게 코딩을 하는 습관은 좋지 않다. 매번 반복하면서 `list.size()` 메서드를 호출하기 때문이다.
- 이럴 때는 다음과 같이 수정하여야 한다.
```java
int listSize=list.size();
for(int loop=0; loop<listSize; loop++)
```
- 이렇게 하면 필요 없는 size() 메서드 반복 호출이 없어지므로 더 빠르게 처리된다.
- JDK 5.0부터는 다음과 같이 For-Each라고 불리는 for 루프를 사용할 수 있다.
```java
ArrayList<String> list=new ArrayList<String>();
-
for(String str : list)
```
- For-Each를 사용하면 별도로 형변환하거나 get() 메서드 또는 elementAt() 메서드를 호출할 필요 없이 순서에 따라서 String 객체를 for 문장 안에서 사용할 수 있으므로 매우 편리하다.
- 단, 이 방식은 데이터의 첫 번째 값부터 마지막까지 처리해야 할 경우에만 유용하다.
```java
@State(Scope.Thread)
@BenchmarkMode({ Mode.AverageTime })
@OutputTimeUnit(TimeUnit.MICROSECONDS)
public class ForLoop {

    int LOOP_COUNT=100000;
    List<Integer> list;

    @Setup
    public void setUp() {
        list=new ArrayList<Integer>(LOOP_COUNT);

        for(int loop=0; loop<LOOP_COUNT; loop++) {
            list.add(loop);
        }
    }

    @GenerateMicroBenchmark
    public void traditionalForLoop() {
        int listSize=list.size();
        for(int loop=0; loop<listSize; loop++) {
            resultProcess(list.get(loop));
        }
    }

    @GenerateMicroBenchmark
    public void traditionalSizeForLoop() {
        for(int loop=0; loop<list.size(); loop++) {
            resultProcess(list.get(loop));
        }
    }

    @GenerateMicroBenchmark
    public void timeForEachLoop() {
        for(Integer loop:list) {
            resultProcess(loop);
        }
    }

    int current;
    public void resultProcess(int result) {
        current=result;
    }
}
```
| 대상 | 응답 시간 (마이크로초) |
|---|---|
| for | 410 |
| for 크기 반복 비교 | 413 |
| for-each | 481 |
## 반복 구문에서의 필요 없는 반복
```java
public void sample(DataVO data,String key) {
    TreeSet treeSet2=null;
    treeSet2=(TreeSet)data.get(key);
    if(treeSet2 !=null) {
        for(int i=0; i<treeSet2.size(); i++) {
            DataVO2 data2=(DataVO2)treeSet2.toArray()[i];
            // ...
        }
    }
}
```
- 이 소스의 문제는 toArray() 메서드를 반복해서 수행한다는 것이다.
- 이 코드는 `toArray()` 메서드가 반복되지 않도록 for문 앞으로 옮기는 것이 좋다.
```java
public void sample(DataVO data,String key) {
    TreeSet treeSet2=null;
    treeSet2=(TreeSet)data.get(key);
    if(treeSet2 !=null) {
        DataVO2 [] dataVO2=(DataVO2[]) treeSet2.toArray();
        int treeSet2Size= treeSet2.size();
        for(int i=0; i<treeSet2Size; i++) {
            DataVO2 data2=dataVO2[i];
            // ...
        }
    }
}
```