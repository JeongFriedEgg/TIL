# CH.20 모니터링 API인 JMX
## JMX란?
- JMX는 Java Management Extensions의 약자이다.
- JMX의 4단계 레벨
  - 인스트루먼테이션 레벨(Instrumentation Level)
  - 에이전트 레벨(Agent Level)
  - 분산 서비스 레벨(Distributed Services Level)
  - 추가 가능한 관리용 프로토콜 API들(Additional Management Protocol APIs)
### 인스트루먼테이션 레벨
- 하나 이상의 MBeans(Management Beans, 관리 빈즈)를 제공한다.
- 이 MBeans에서 필요한 리소스들의 정보를 취합하여 에이전트로 전달하는 역할을 한다.
### 에이전트 레벨
- 에이전트를 구현하기 위한 스펙이 제공되어 있다.
- 에이전트는 리소스를 관리하는 역할을 수행한다.
- 보통 에이전트는 모니터링이 되는 서버와 같은 장비에 위치한다.
- 에이전트는 MBean 서버와 MBeans를 관리하는 서비스의 집합으로 구성되어 있다.
### 분산 서비스 레벨
- JMX 관리자를 구현하기 위한 인터페이스와 컴포넌트를 제공한다.
- 여러 에이전트에서 제공하는 정보를 관리할 수 있는 화면과 같은 부분을 여기서 담당한다.
## MBean에 대해서 조금만 더 자세히 알아보자
- 표준 MBean(Standard MBean): 변경이 많지 않은 시스템을 관리하기 위한 MBean이 필요한 경우 사용한다.
- 동적 MBean(Dynamic MBean): 애플리케이션이 자주 변경되는 시스템을 관리 하기 위한 MBean이 필요한 경우 사용한다.
- 모델 MBean(Model MBean): 어떤 리소스나 동적으로 설치가 가능한 MBean이 필요한 경우 사용한다.
- 오픈 MBean(Open MBean): 실행 중에 발견되는 객체의 정보를 확인하기 위한 MBean이 필요할 때 사용한다. JMX의 스펙에 지정된 타입만 리턴해야 한다.
## Visual VM을 통한 JMX 모니터링
- bin 디렉터리 아래에는 java와 javac, javadoc만 있는 것이 아니다.
- 여러가지 다양한 툴이 존재하며, 그중에서 모니터링을 위한 jconsole과 jvisualvm(이하 Visual VM)이라는 툴도 존재한다.
- 이 두가지 툴 모두 JMX의 데이터를 볼 수 있도록 만들어졌다.
## 원격으로 JMX를 사용하기 위해서는...
- 원격지에 있는 서버와 통신을 하여 JMX 모니터링을 하기 위해서는 서버나 자바 애플리케이션을 시작할 때 VM 옵션을 지정해야 한다.
- 간단하게 사용하려면 다음의 3가지 옵션을 지정할 수 있다.
  - Dcom.sun.management.jmxremote.port=9003
  - Dcom.sun.management.jmxremote.ssl=false
  - Dcom.sun.management.jmxremote.authenticate=false
- 아이디와 패스워드를 지정하여 접속할 수 있도록 변경하려면 다음과 같이 지정할 수 있다.
  - Dcom.sun.management.jmxremote.port=9003
  - Dcom.sun.management.jmxremote.password.file=/파일위치/conf/jmxremote.password
  - Dcom.sun.management.jmxremote.access.file=/파일위치/conf/jmxremote.access
  - Dcom.sun.management.jmxremote.ssl=false