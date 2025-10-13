# CH.01 디자인 패턴, 꼭 써야 한다
## 적어도 MVC 모델은 적용해야죠
- MVC는 Model, View, Controller의 약자이다.
- 하나의 JSP나 스윙(Swing)처럼 화면에 모든 처리로직을 모아두는 것이 아니라 모델역할, 뷰역할, 컨트롤러역할을 하는 클래 를 각각 만들어서 개발하는 모델이다.
- 뷰는 사용자가 결과를 보거나 입력할 수 있는 화면이라고 생각하면 된다.
- 이벤트를 발생시키고, 이벤트의 결과를 보여주는 역할을 한다.
- 컨트롤러는 뷰와 모델의 연결자라고 생각하면 된다.
- 뷰에서 받은 이벤트를 모델로 연결하는 역할을 한다.
- 모델은 뷰에서 입력된 내용을 저장, 관리, 수정하는 역할을 한다.
```text
브라우저 ⎯ JSP ⎯ 자바 빈 ⎯ 데이터베이스
```
```text
브라우저 ⎯ 서블릿
     ↖︎    |    ↖︎
          JSP ⎯ 자바 빈 ⎯ 데이터베이스
```
- JSP 모델1은 JSP에서 자바 빈을 호출하고 데이터베이스에서 정보를 조회, 등록, 수정, 삭제업무를 한 후 결과를 브라우저로 보내주는 방식이다.
- 개발 후 프로세스 변경이 생길 경우에 수정이 어렵다는 단점이 있다.
- JSP 모델2는 MVC 모델을 정확히 따른다.
- JSP로 요청을 직접 하는 JSP 모델1과 가장 큰 차이점은 서블릿으로 요청을 한다는 것이다.
- 모델2에서는 서블릿이 컨트롤러 역할을 수행한다.
## J2EE 디자인 패턴이란?
- 시스템을 만들기 위해서 전체 중 일부 의미있는 클래스들을 묶은 각각의 집합을 디자인 패턴이라고 생각하면 된다.

![J2EE.png](../image/J2EE.png)

- 가장 윗부분은 프레젠테이션 티어이고, 중간부분은 비즈니스 티어, 하단부분은 인테그레이션 티어다.
- 위로 갈수록 화면에 가깝고, 아래로 갈수록 DB와 같은 저장소에 가깝다고 생각하면 이해하기 쉽다.
- Intercepting Filter 패턴 : 요청 타입에 따라 다른 처리를 하기 위한 패턴이다. 
- Front Controller 패턴 : 요청 전후에 처리하기 위한 컨트롤러를 지정하는 패턴이다. 
- View Helper 패턴 : 프레젠테이션 로직과 상관없는 비즈니스 로직을 헬퍼로 지정하는 패턴이다. 
- Composite View 패턴 : 최소 단위의 하위 컴포넌트를 분리하여 화면을 구성하는 패턴이다.
- Service to Worker 패턴 : Front Controller와 View Helper 사이에 디스패처를 두어 조합하는 패턴이다.
- Dispatcher View 패턴 : Front Controller와 View Helper로 디스패처 컴포넌트를 형성한다. 뷰 처리가 종료될 때까지 다른 활동을 지연한다는 점이 Service to Worker 패턴과 다르다. 
- Business Delegate 패턴 : 비즈니스 서비스 접근을 캡슐화하는 패턴이다. 
- Service Locator 패턴 : 서비스와 컴포넌트 검색을 쉽게 하는 패턴이다. 
- Session Facade 패턴 : 비즈니스 티어 컴포넌트를 캡슐화하고, 원격 클라이언트에서 접근할 수 있는 서비스를 제공하는 패턴이다. 
- Composite Entity 패턴 : 로컬 엔티티 빈과 POJO를 이용하여 큰 단위의 엔티티 객체를 구현한다. 
- Transfer Object 패턴 : 일명 Value Object 패턴이라고 많이 알려져 있다. 데이터를 전송하기 위한 객체에 대한 패턴이다. 
- Transfer Object Assembler 패턴 : 하나의 Transfer Object로 모든 타입 데이터를 처리할 수 없으므로, 여러 Transfer Object를 조합하거나 변형한 객체를 생성하여 사용하는 패턴이다. 
- Value List Handler 패턴 : 데이터 조회를 처리하고, 결과를 임시 저장하며, 결과 집합을 검색하여 필요한 항목을 선택하는 역할을 수행한다. 
- Data Access Object 패턴 : 일명 DAO라고 많이 알려져 있다. DB에 접근을 전담하는 클래스를 추상화하고 캡슐화한다. 
- Service Activator 패턴 : 비동기적 호출을 처리하기 위한 패턴이다.
```text
# Service to Worker 패턴 클래스 다이어그램


Client ⎯ <<Servlet>> Controller ⎯ Dispatcher ⎯ <<JSP>> View
                     ⎮                  ⎮              ⎮
                     └────────────── Helper ───────────┘
```
```text
# Dispatcher View 패턴 클래스 다이어그램


Client ⎯ <<Servlet>> Controller ⎯ Dispatcher ⎯ <<JSP>> View
                                                       ⎮
                                                    Helper
```
- Service to Worker 패턴과 Dispatcher View 패턴이 의미가 비슷하여 혼동될 수 있는데, 클래스 다이어그램을 보면 다음과 같은 차이가 있다.
- Dispatcher view 패턴은 Helper 클래스를 직접 컨트롤하지 않는다는 차이가 있다.
## Transfer Object 패턴
```java
public class EmployeeTO implements Serializable {
    private String empName;
    private String empID;
    private String empPhone;

    publicEmployeeTo() {
        super();
    }

    public EmployeeTO(String empName, String empID, String empPhone) {
        super();
        this.empName = empName;
        this.empID = empID;
        this.empPhone = empPhone;
    }

    public String getEmpID() {
        return empID;
    }

    public void setEmpID(String empID) {
        this.empID = empID;
    }

    public String getEmpName() {
        if (empName == null) return "";
        else return empName;
    }

    public void setEmpName(String empName) {
        this.empName = empName;
    }

    public String getEmpPhone() {
        return empPhone;
    }

    public void setEmpPhone(String empPhone) {
        this.empPhone = empPhone;
    }

    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append("empName=").append(empName).append(" empID=")
                .append(empID).append("empPhone=").append(empPhone);
        return sb.toString();
    }
}
```
- Transfer Object 패턴은 Transfer Object를 만들어 하나의 객체에 여러 타입의 값을 전달하는 일을 수행한다.
- getter() 메서드나 setter() 메서드를 사용하면 getEmpName() 메서드처럼 empName이 null 값이라도 null을 리턴하지 않고 길이가 0인 String을 리턴한다.
- Transfer Object를 잘 만들어 놓으면 각 소스에서 일일이 null 체크를 할 필요가 없기 때문에 개발할 때 오히려 더 편해진다.
- Transfer Object를 생성할 때는 반드시 toString() 메서드를 구현하기 바란다.
- 이 메서드를 구현하지 않고, EmployeeTO 객체의 toString() 메서드를 수행하면 com.pattern.EmployeeTOac1716처럼 알 수 없는 값을 리턴한다.
- 하나의 객체에 결과값을 담아올 수 있어 두번, 세번씩 요청을 하는 일이 발생하는 것을 줄여주므로, 이 패턴을 사용하기를 권장한다.
## Service Locator 패턴
```java
public class ServiceLocator {
    private InitialContext ic;
    private Map cache;
    private static ServiceLocator me;

    static {
        me = new ServiceLocator();
    }

    private ServiceLocator() {
        cache = Collections.synchronizedMap(new HashMap());
    }

    public InitialContextget InitialContext() throws Exception {
        try {
            if (ic == null) {
                ic = new InitialContext();
            }
        } catch (Exception e) {
            throw e;
        }
        return ic;
    }

    public static ServiceLocatorgetInstance() {
        return me;
    }
    // ...
}
```
- Service Locator 패턴은  DB의 DataSource를 찾을 때(lookup할 때) 소요되는 응답속도를 감소시키기 위해서 사용된다.
- 위의 소스를 간단히 보면, cache라는 Map 객체에 home 객체를 찾은 결과를 보관하고 있다가, 누군가 그 객체를 필요로 할 때 메모리에서 찾아서 제공하도록 되어 있다.
- 만약 해당 객체가 cache라는 맵에 없으면 메모리에서 찾는다.