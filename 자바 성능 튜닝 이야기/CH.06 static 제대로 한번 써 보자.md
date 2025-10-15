# CH.06 static 제대로 한번 써 보자.
## static의 특징
```java
public class VariableTypes {
    int instanceVariable;
    static int classVariable;

    public void method(int parameter) {
        int localVariable;
    }
}
```
- static으로 선언한 classVariable은 클래스 변수라고 한다.
- 왜냐하면, 그 변수는 '개체의 변수'가 되는 것이 아니라 '클래스의 변수'가 되기 때문이다.
- 100개의 VariableTypes 클래스의 인스턴스를 생성하더라도, 모든 객체가 classVariable에 대해서는 동일한 주소의 값을 참조한다.
```java
public class StaticBasicSample2 {
    static String staticVal;
    static {
        staticVal = "Static Value";
        staticVal = StaticBasicBasicSample.staticInt + "";
    }

    public static void main(String[] args) {
        System.out.println(StaticBasicSample2.staticVal);
    }

    static {
        staticVal = "Performance is important !!!";
    }
}
```
- static 초기화 블록은 위와 같이 클래스 어느 곳에나 지정할 수 있다.
- 이 static 블록은 클래스가 최초 로딩될 때 수행되므로 생성자 실행과 상관없이 수행된다.
- 또한 위의 예제와 같이 여러 번 사용할 수 있으며, 이와 같이 사용했을 때 staticVal 값은 마지막에 지정한 값이 된다.
- static 블록은 순차적으로 읽혀진다는 의미이다.
```text
# 출력
Performance is important !!!
```
- static의 특징은 다른 JVM에서는 static이라고 선언해도 다른 주소나 다른 값을 참조하지만, 하나의 JVM이나 WAS 인스턴스에서는 같은 주소에 존재하는 값을 참조한다는 것이다.
- 그리고 GC의 대상도 되지 않는다.
- 그러므로 static을 잘 사용하면 성능을 뛰어나게 향상시킬 수 있지만, 잘못 사용하면 예기치 못한 결과를 초래하게 된다.
## static 잘 활용하기
### 자주 사용하고 절대 변하지 않는 변수는 final static으로 선언하자.
- 자주 사용되는 로그인 관련 쿼리들이나 간단한 목록 조회 쿼리를 final static으로 선언하면 적어도 1바이트 이상의 객체가 GC 대상에 포함되지 않는다.
- 간단한 데이터들도 static으로 선언할 수 있지만, 템플릿 성격의 객체를 static으로 선언하는 것도 성능향상에 많은 도움이 된다.
- Velocity를 사용할 때가 좋은 예이다.
- Velocity란 자바 기반의 프로젝트를 수행할 때, UI가 될 수 있는 HTML뿐만 아니라 XML, 텍스트 등의 템플릿을 정해놓고, 실행시 매개변수 값을 던져서 원하는 형식의 화면을 동적으로 구성할 수 있도록 도와주는 컴포넌트이다.
- Velocity 기반의 성능을 테스트해 보면 템플릿을 읽어오는 부분에서 시간이 가장 많이 소요된다.
```java
// 메서드 앞부분 생략
try {
    Template template = Velocity-getTemplate("TemplateFileName");
// 이하 생략
```
- 템플릿 파일을 읽어서 파싱(parsing)하기 때문에 서버의 CPU에 부하가 많이 발생하고 대기시간도 많아진다.
- 그러므로 수행하는 메서드에서 이 부분을 분리하여 다음과 같이 수정해야 한다.
```java
//클래스 앞부분 생략
static Template template;

static {
    try {
        template = Velocity.getTemplate("TemplateFileName");
    } catch (Exception e) {
        //exception 처리
    }
}
```
- 이렇게 처리하면 화면을 요청할 때마다 템플릿 객체를 파싱하여 읽을 필요가 없다.
- 클래스가 로딩될 때 한 번만 파싱하므로 성능이 엄청나게 향상된다.
- 그런데 만약, 해당 template 내용이 지속적으로 변경되는 부분이라면 이와 같이 코드를 작성할 경우 A화면이 보여야 하는 사용자에게 B화면이 보일 수도 있다.
### 설정 파일 정보도 static으로 관리하자.
- 클래스의 객체를 생성할 때마다 설정 파일을 로딩하면 엄청난 성능저하가 발생하게 된다.
- 이럴 때는 반드시 static으로 데이터를 읽어서 관리해야 한다.
### 코드성 데이터는 DB에서 한 번만 읽자.
- 큰 회사의 부서코드나 큰 쇼핑몰의 상품코드처럼 양이 많고 자주 바뀔 확률이 높은 데이터를 제외하고, 부서가 적은 회사의 코드나, 건수가 그리 많지 않되 조회빈도가 높은 코드성 데이터는 DB에서 한 번만 읽어서 관리하는 것이 성능 측면에서 좋다.
```java
public class CodeManager {
    private HashMap<String, String> codeMap;
    private static CodeDAO cDAO;
    private static CodeManager cm;

    static {
        cDAO = new CodeDAO();
        cm = new CodeManager();
        if (!cm.getCodes()) {
            //에러처리
        }
    }

    private CodeManager() {
    }

    public static CodeManager getInstance() {
        return cm;
    }

    private boolean getCodes() {
        try {
            codeMap = cDAO.getCodes();
            return true;
        } catch (Exception e) {
            return false;
        }
    }

    public boolean updateCodes() {
        return cm.getCodes();
    }

    public String getCodeValue(String code) {
        return codeMap.get(code);
    }
}
```
- 내부적으로 cm 객체를 static으로 선언하여, 생성자가 아닌 getInstance() 메서드를 통해서 CodeManager 클래스에 접근하도록 했다. 
- codeMap이라는 HashMap 객체에 우리가 필요한 코드 정보들을 담을 예정이다.
- CodeDAO 클래스는 DB에서 코드정보를 갖고 오도록 되어있는, 기존에 사용하던 DAO 클래스이다.
- 클래스가 메모리에 로드되면 static 초기화 블록에서 cDAO 객체 및 cm 객체를 초기화하고, getCodes() 메서드를 호출한다.
- 모든 코드정보는 codeMap에 저장된다.
- 이제부터 코드정보를 가져올 때는 getCodeValue() 메서드를 호출하여 메모리에서 코드정보를 읽어온다.
- 그런데 이와 같은 클래스를 만들어서 코드를 가져오는 일은 가장 큰 문제가 되기도 한다.
- 만약 서버 인스턴스가 하나만 있다면 코드가 변경되는 것을 걱정할 필요가 없다.
- 수정되자마자 updateCodes() 메서드를 호출하면 끝이기 때문이다.
- 하지만 서로 다른 JVM에 올라가 있는 코드정보는 수정된 코드와 상이하므로 그 부분에 대한 대책을 마련해 놓아야만 한다.
- 만약 코드가 절대 변경되지 않고, 혹시 코드가 변경될 경우 서버를 재시작한다면 이 부분에 대해 걱정할 필요는 전혀 없다.
- 이러한 JVM 간에 상이한 결과가 나오는 것을 방지하기 위해서 요즘에는 mem-cached, EhCache 등의 캐시(Cache)를 많이 사용한다.
## static 잘못 쓰면 이렇게 된다.
```java
public class BadQueryManager {
    private static String queryURL = null;

    public BadQueryManager(String badUrl) {
        queryURL = badUrl;
    }

    public static String getSql(String idSql) {
        try {
            FileReader reader = new FileReader();
            HashMap<String, String> document = reader.read(queryURL);
            return document.get(idSql);
        } catch (Exception ex) {
            System.out.println(ex);
        }
        return null;
    }
}
```
- 만약 어떤 화면에서 BadQueryManager의 생성자를 통해서 queryURL을 설정하고 getSql() 메서드를 호출하기 전에, 다른 queryURL을 사용하는 화면의 스레드에서 BadQueryManager의 생성자를 호출하면 어떤 일이 발생할까? 
- 그때부터는 시스템이 오류를 발생시킨다. 먼저 호출한 화면에서는 생성자를 호출했을 때의 URL을 유지하고 있을 것이라 생각하고 getSql() 메서드를 호출하겠지만, 이미 그 값은 변경되고 난 후다.
```java
public class BadQueryManager {
    private static String queryURL = null;

    public BadQueryManager(String badUrl) {
        queryURL = badUrl;
    }

    public static String getSql(String idSql) {
        //중간생략
        HashMap<String, String> document = reader.read(queryURL);
        //이하생략
    }
}
```
- getSql() 메서드와 queryURL을 static으로 선언한 것이 잘못된 부분이다.
- 웹 환경이기 때문에 여러 화면에서 호출할 경우에 queryURL은 그때 그때 바뀌게 된다.
- queryURL은 static으로 선언했기 때문에 클래스의 변수이지 객체의 변수가 아니다.
- 모든 스레드에서 동일한 주소를 가리키게 되어 문제가 발생한 것이다.
```java
// 중간생략
private static boolean successFlag;
// 이하생략
```
- 서블릿에서 웅시자의 합격 여부를 잠깐 담아놓기 위해 SuccessFlag로 지정해 놓은 부분이다.
- 물론 이 소스는 개발자가 본인의 PC에서 테스트할 때는 전혀 문제가 되지 않는다.
- 그러나 만약 수십명이 동시에 자신이 정보를 확인하기 위해서 위의 서블릿을 호출하는 경우를 생각해 보자.
- successFlag를 true로 처리해 놓은 상황에서 다른 사용자의 요청이 처리되어 false로 바뀐다면, 그 사람은 완전히 다른 결과를 받게 된다.
## static과 메모리 릭 
- 어떤 클래스에 데이터를 `Vector`나 `ArrayList`에 담을 때 해당 `Collection` 객체를 `static`으로 선언하면 어떻게 될까?
- 만약 지속적으로 해당 객체에 데이터가 쌓인다면, 더 이상 GC가 되지 않으면서 시스템은 `OutOfMemoryError`를 발생시킨다.
- 더 이상 사용 가능한 메모리가 없어지는 현상을 메모리 릭(Memory Leak)이라고 하는데, `static`과 `Collection` 객체를 잘못 사용하면 메모리 릭이 발생한다.