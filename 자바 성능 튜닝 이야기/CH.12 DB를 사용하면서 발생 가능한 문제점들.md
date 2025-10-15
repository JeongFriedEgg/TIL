# CH.12 DB를 사용하면서 발생 가능한 문제점들
## DB Connection과 Connection Pool, DataSource
- JDBC 관련 API는 클래스가 아니라 인터페이스이다.
- JDK의 API에 있는 java.sql 인터페이스를 각 DB 벤더에서 상황에 맞게 구현하도록 되어있다.
- 같은 인터페이스라고 해도, 각 DB 벤더에 따라서 처리되는 속도나 내부처리방식은 상이하다.
```java
try{
    Class.forName("oracle.jdbc.driver.OracleDriver");
    Connection con = DriverManager.getConnection(
            "jdbc:oracle:thin:@ServerIP:1521:SID", "ID", "Password");
    PreparedStatement ps = con.prepareStatement("SELECT ... where id=?");
    ps.setString(1,id);
    ResultSet rs = ps.executeQuery();
    // 중간데이터 처리 부분생략
} catch(ClassNotFoundException e){
    System.out.println("드라이버 load fail");
    throw e;
} catch(SQLException e){
    System.out.println("Connection fail");
    throw e;
} finally {
    rs.close();
    ps.close();
    con.close();
}
```
- 가장 느린 부분은 `Connection` 객체를 얻는 부분이다.
- 같은 장비에 DB가 구성되어 있다고 하더라도, DB와 WAS 사이에는 통신을 해야하기 때문이다.
- `Connection` 객체를 생성하는 부분에서 발생하는 대기시간을 줄이고, 네트워크의 부담을 줄이기 위해서 사용하는 것이 DB Connection Pool이다.
- `Statement`와 거의 동일하게 사용할 수 있는 `Statement` 인터페이스의 자식 클래스로 `PreparedStatement`가 있다.
- `Statement`와 `PreparedStatement`의 가장 큰 차이점은 캐시(cache) 사용 여부이다.
- `Statement`를 사용할 때와 `PreparedStatement`를 처음 사용할 때의 프로세스
  1. 쿼리 문장 분석
  2. 컴파일
  3. 실행
- `Statement`를 사용하면 매번 쿼리를 수행할 때마다 1-3 단계를 거치게 되고, `PreparedStatement`는 처음 한 번만 세 단계를 거친 후 캐시에 담아서 재사용을 한다는 것이다.
- 쿼리를 수행하는 메서드는 여러가지 있는데, 그중 많이 사용하는 것이 `executeQuery()`, `executeUpdate()`, `execute()` 메서드이다.
- `executeQuery()` 메서드는 select 관련 쿼리를 수행한다. 수행 결과로 요청한 데이터 값이 `ResultSet` 객체의 형태로 전달된다.
- `executeUpdate()` 메서드는 select 관련 쿼리를 제외한 DML(INSERT, UPDATE, DELETE 등) 및 DDL(CREATE TABLE, CREATE VIEW 등) 쿼리를 수행한다. 결과는 int 형태로 리턴된다.
- `execute()` 메서드는 쿼리의 종류와 상관없이 쿼리를 수행한다. `execute()` 메서드의 수행결과는 `ResultSet`이 아닌 `boolean` 형태의 데이터를 리턴하는데, 만약 데이터가 있을 경우에는 true를 리턴하여 `getResultSet()` 메서드를 사용하여 결과값을 받을 수 있다.
## DB를 사용할 때 닫아야 하는 것들
- 일반적으로 각 객체를 얻는 순서는 Connection, Statement, ResultSet 순이며, 객체를 닫는 순서는 ResultSet, Statement, Connection 순이다.
- ResultSet 객체가 닫히는 경우
  - `close()` 메서드를 호출하는 경우
  - GC의 대상이 되어 GC되는 경우
  - 관련된 `Statement` 객체의 `close()` 메서드가 호출되는 경우
- `ResultSet` 인터페이스에서 `close()` 메서드를 호출하는 이유는, 자동으로 호출되기 전에 관련된 DB와 JDBC리소스를 해제하기 위함이다.
- `Statement` 객체는 `Connection` 객체를 `close()`한다고 해서 자동으로 닫히지 않는다.
- `Statement`가 닫히는 경우
  - close() 메서드를 호출하는 경우
  - GC의 대상이 되어 GC 되는 경우
- `Connection` 객체가 닫히는 경우
  - close() 메서드를 호출하는 경우
  - GC의 대상이 되어 GC되는 경우
  - 치명적인 에러가 발생하는 경우
- `Connection`은 대부분 Connection Pool을 사용하여 관리된다.
- 시스템이 기동되면 지정된 개수만큼 연결하고, 필요할 때 중가시키도록 되어 있다.
- 증가되는 최대값 또한 지정하도록 되어 있다.
- 사용자가 증가해 더 이상 사용할 수 있는 연결이 없으면, 여유가 생길 때까지 대기한다.
- 그러다가 어느정도 시간이 지나면 오류가 발생한다.
- 그러므로 `close()` 메서드를 호출하여 연결을 닫아야 한다.
```java
try {
    // 상단 부분 생략
    Connection con=...;
    PreparedStatement ps=...;
    ResultSet rs=...;
    // 중간 생략
    rs=null;
    ps=null;
    con=null;
} catch(Exception e) {
    -
}
```
- close가 될 수도 있고 되지 않을수도 있다.
- 어차피 null로 치환하면 GC의 대상이 되긴 한다. 하지만 언제 GC될지 모르기 때문에 좋지 않은 방법이다.
```java
try {
    // 상단 부분 생략
    Connection con=...;
    PreparedStatement ps=...;
    ResultSet rs=...;
    // 중간 생략
    rs.close();
    ps.close();
    con.close();
} catch(Exception e) {
    -
}
```
- 예외가 발생하지 않으면 정상적으로 close되겠지만, 예외가 발생하면 어떤 객체도 close되지 않는다.
```java
Connection con=null;
PreparedStatement ps=null;
ResultSet rs=null;
try {
    // 상단 부분 생략
    con=...;
    ps =...;
    rs=...;
    // 중간 생략
} catch(Exception e) {
    -
} finally {
    rs.close();
    ps.close();
}
try {
    con.close();
} catch (Exception e) {
    -
}
```
- 쿼리를 수행하는 부분에서 오류가 발생할 때 예외를 던져버리면, 해당 `Connection` 클래스의 객체는 절대 닫히지 않을수도 있다.
```java
Connection con=null;
PreparedStatement ps=null;
ResultSet rs=null;
try {
    // 상단 부분 생략
    con=...;
    ps =...;
    rs=...;
    // 중간 생략
} catch(Exception e) {
    ...
} finally {
    try{rs.close();} catch(Exception rse){}
    try{ps.close();} catch(Exception pse){}
    try{con.close();} catch(Exception cone){}
}
```
- 이와 같은 방법으로 처리를 해주어야 한다.
## JDK 7에서 등장한 AutoClosable 인터페이스
- `AutoClosable` 인터페이스에는 리턴 타입이 `void`인 `close()` 메서드 단 한개만 선언되어 있다.
- try-with-resources 문장으로 관리되는 객체에 대해서 자동적으로 close() 처리를 한다.
- `InteruptedException`을 던지지 않도록 하는 것을 권장한다.
- 이 close() 메서드를 두 번 이상 호출할 경우 뭔가 눈에 보이는 부작용이 나타나도록 해야 한다.
```java
public String readFile(String fileName) throws IOException {
    FileReader reader=new FileReader(new File(fileName));
    BufferedReader br=new BufferedReader(reader);
    String data=null;
    try {
        data=br.readLine();
    } finally {
        if (br!=null) br.close();
    }
    return data;
}
```
- JDK 7부터는 try 블록이 시작될 때 소괄호 안에 close() 메서드를 호출하는 객체를 생성해주면 간단하게 처리할 수 있다.
```java
public String readFileNew(String fileName) throws IOException {
    FileReader reader=new FileReader(new File(fileName));
    try(BufferedReader br = new BufferedReader(reader)) {
        return br.readLine();
    }
}
```
- 별도로 finally 블록에서 close() 메서드를 호출할 필요가 없어졌다는 의미다.
## ResultSet.last() 메서드
- 이 메서드를 수행하는 이유는 뭘까?
```java
rs.last();
int totalCount=rs.getRow();
ResultArray[] result=new ResultArray[totalCount];
```
- 전체 데이터 개수를 확인하고 배열에 담아서 사용하기 위해서라면 그나마 양호하다.
- 배열을 Vector로 변경하고 사용하면 되기 때문이다.
- 하지만 게시판과 같은 화면을 구성할 때 전체건수를 확인하기 위해서 이렇게 사용하는 경우도 있다.
- 전체건수를 확인하기 위해서는 쿼리를 한 번더 던져서 확인하는 것이 훨씬 빠르다.
- `rs.last()` 메서드의 수행시간은 데이터의 건수 및 DB와의 통신속도에 따라서 달라진다.
- 건수가 많으면 많을수록 대기 시간이 중가한다.
- `is.next()`를 수행할 때와 비교할 수 없을 정도로 속도차이가 나기 때문에, 이 메서드의 사용은 자제해야 한다.
## JDBC를 사용하면서 유의할만한 몇 가지 팁
- `setAutoCommit()` 메서드를 사용하여 자동커밋여부를 지정하는 작업은 반드시 필요할 때만 하자.
- 배치성 작업을 할 때는 `Statement` 인터페이스에 정의되어 있는 `addBatch()` 메서드를 사용하여 쿼리를 지정하고, `executeBatch()` 메서드를 사용하여 쿼리를 수행하자.
- 가져오는 데이터의 수가 정해져 있을 경우에는 `Statement`와 `ResultSet` 인터페이스에 있는 `setFetchSize()` 메서드를 사용하여 원하는 개수를 정의하자.
- 한 건만 필요할 때는 한 건만 가져오자.