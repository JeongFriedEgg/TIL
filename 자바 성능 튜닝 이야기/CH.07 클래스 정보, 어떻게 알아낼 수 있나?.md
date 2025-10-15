# CH.07 클래스 정보, 어떻게 알아낼 수 있나?
## reflection 관련 클래스들
- 자바 API에는 reflection이라는 패키지가 있다.
- 이 패키지에 있는 클래스들을 사용하면 JVM에 로딩되어 있는 클래스와 메서드 정보를 읽어올 수 있다.
### Class 클래스
- Object 클래스에 있는 getClass() 메서드를 이용하는 것이 일반적이다.
- `String getName()`: 클래스의 이름을 리턴한다.
- `Package getPackage()`: 클래스의 패키지 정보를 패키지 클래스 타입으로 리턴한다.
- `Field[] getFields()`: public으로 선언된 변수 목록을 Field 클래스 배열 타입으로 리턴한다.
- `Field getField(String name)`: public으로 선언된 변수를 Field 클래스 타입으로 리턴한다. 
- `Field[] getDeclaredFields()`: 해당 클래스에서 정의된 변수 목록을 Field 클래스 배열 타입으로 리턴한다. 
- `Field getDeclaredField(String name)`; name과 동일한 이름으로 정의된 변수를 Field 클래스 타입으로 리턴한다. 
- `Method[] getMethods()`: public으로 선언된 모든 메서드 목록을 Method 클래스 배열 타입으로 리턴한다. 해당 클래스에서 사용 가능한 상속받은 메서드도 포함된다. 
- `Method getMethod(String name, Class... parameterTypes)`: 지정된 이름과 매개변수 타입을 갖는 메서드를 Method 클래스 타입으로 리턴한다. 
- `Method[] getDeclaredMethods()`: 해당 클래스에서 선언된 모든 메서드 정 보를 리턴한다.
- `Method getDeclaredMethod(String name, Class... parameterTypes)`: 지정된 이름과 매개변수 타입을 갖는 해당 클래스에서 선언된 메서드를 Method 클래스 타입으로 리턴한다. 
- `Constructor[] getConstructors()`: 해당 클래스에 선언된 모든 public 생성자의 정보를 Constructor 배열 타입으로 리턴한다.
- `Constructor[] getDeclaredConstructors()`: 해당 클래스에서 선언된 모든 생성자의 정보를 Constructor 배열 타입으로 리턴한다. 
- `int getModifiers()`: 해당 클래스의 접근자(modifier) 정보를 int 타입으로 리턴한다. 
- `String toString()`: 해당 클래스 객체를 문자열로 리턴한다.
```java
String currentClassName=this.getClass().getName();
```
- getName() 메서드는 패키지 정보까지 리턴해 준다.
### Method 클래스
- Method 클래스에는 생성자가 없으므로 Method 클래스의 정보를 얻기 위해서는 Class 클래스의 getMethods() 메서드를 사용하거나 getDeclaredMethod() 메서드를 써야 한다.
- `Class<?> getDeclaringClass()`: 해당 메서드가 선언된 클래스 정보를 리턴한다.
- `Class <?> getReturnType()`: 해당 메서드의 리턴 타입을 리턴한다.
- `Class <?>[] getParameterTypes()`: 해당 메서드를 사용하기 위한 매개변수의 타입들을 리턴한다.
- `String getName()`: 해당 메서드의 이름을 리턴한다. 
- `int getModifiers()`: 해당 메서드의 접근자 정보를 리턴한다. 
- `Class<?>[] getExceptionTypes()`: 해당 메서드에 정의되어 있는 예외 타입들을 리턴한다. 
- `Object invoke(Object obj, Object... args)`: 해당 메서드를 수행한다. 
- `String toGenericString()`: 타입 매개변수를 포함한 해당 메서드의 정보를 리턴한다.
- `String toString()`: 해당 메서드의 정보를 리턴한다.
### Field 클래스
- Field 클래스는 클래스에 있는 변수들의 정보를 제공하기 위해서 사용한다.
- `int getModifiers()`: 해당 변수의 접근자 정보를 리턴한다. 
- `String getName()`: 해당 변수의 이름을 리턴한다.
- `String toString()`: 해당 변수의 정보를 리턴한다.
## reflection 관련 클래스를 사용한 예
## reflection 클래스를 잘못 사용한 사례
```java
public String checkClass(Object src) {
    if (src.getClass().getName().equals("'java.math.BigDecimal")) {
        // 데이터 처리
    }
    //이하 생략
}
```
- 이렇게 사용할 경우 응답속도에 그리 많은 영향을 주지는 않지만, 많이 사용하면 필요없는 시간을 낭비하게 된다.
```java
public String checkClass(Object src) {
    if (src instanceof java.math.BigDecimal) {
        //데이티 처리
    }
    //이하 생략
}
```
- `instanceof`를 사용하니 소스가 훨씬 간단해졌다.

| 대상 |응답 시간 (마이크로초)|
|:---|---|
|  instanceof 사용  |0.167|
|Reflection 사용|1.022|