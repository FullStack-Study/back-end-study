# 0. 들어가며

# 스프링이란 무엇인가?

스프링은 자바 엔터프라이즈 애플리케이션 개발에 사용되는 **애플리케이션 프레임워크**이다.

-   **애플리케이션 프레임워크**

    `단순함(POJO 프로그래밍)` `유연성`

    애플리케이션 개발을 빠르고 효율적으로 할 수 있도록 애플리케이션의 바탕이 되는 틀과 공통 프로그래밍 모델, 기술 API를 지원해준다.

    1. 애플리케이션 기본 틀 - 스프링 컨테이너
        - 스프링 컨테이너 또는 애플리케이션 컨텍스트 라고 불리는 스프링 런타임 엔진 제공
        - 스프링 컨테이너는 설정 정보를 참고로 애플리케이션을 구성하는 오브젝트를 생성하고 관리한다.
    2. 공통 프로그래밍 모델 - IoC/DI, 서비스 추상화, AOP

        - 프레임워크는 **프로그래밍 모델**을 지정해 애플리케이션 코드 작성 방식에 대한 기준을 제시해준다.

            - IoC/DI

                오브젝트의 생명주기와 의존관계에 대한 프로그래밍 모델

            - 서비스 추상화

                환경이나 서버, 특정 기술에 종속되지 않고 이식성, 유연성을 높여줌

            - AOP

                깔끔한 코드 유지

    3. 기술 API
        - 스프링은 개발의 다양한 계층에 바로 활용할 수 있는 기술 API를 제공한다.
        - 스프링에서 일관된 방식으로 사용할 수 있도록 지원해주는 기능, 전략 클래스 등을 제공한다.

    # 스프링 3.0의 달라진 기능

    -   Java5와 JavaEE6
    -   스프링 표현식 언어(SpEL)
    -   자바 코드를 이용한 DI 설정과 DIJ(JSR-330)
    -   OXM
    -   `@MVC`와 REST
    -   내장형 DB 지원(Derby, HSQL, H2)
    -   Converter, ConversionService, Formatter

    # 스프링 3.1에 추가된 새로운 기능

    -   강화된 자바 코드를 이용한 빈 설정
    -   런타임 환경 추상화
    -   JPA 지원 확장과 하이버네이트 4 지원
    -   새로운 DispatcherServlet 전략과 플래시 맵
    -   캐시 추상화

# 1. 오브젝트와 의존관계

# 1.1 초난감 DAO

사용자 정보를 JDBC API를 통해 DB에 저장하고 조회할 수 있는 간단한 DAO를 하나 만들어보자.

> ### DAO (Data Access Object)
>
> DB를 사용해 데이터를 조회하거나 조작하는 기능을 전담하도록 만든 오브젝트

## 1.1.1 User

```cpp
package springbook.user.domain;

public class User {
	String id;
	String name;
	String password;

	public String getld() {
		return id;
	}

	public void setld(String id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}
}
```

-   USERS 테이블 필드 구성

    | 필드명   | 타입        | 설정        |
    | -------- | ----------- | ----------- |
    | id       | VARCHAR(10) | Primary Key |
    | Name     | VARCHAR(10) | Not Null    |
    | Password | VARCHAR(10) | Not Null    |

## 1.1.2 UserDao

사용자 정보를 DB에 넣고 관리할 수 있는 DAO 클래스

사용자 정보의 등록: `add`, 조회: `get`

-   JDBC 작업 순서
    -   DB연결을 위한 Connection을 가져온다.
    -   SQL을 담은 Statement (또는 PreparedStatement)를 만든다.
    -   만들어진 State를 실행한다.
    -   조회의 경우 SQL 쿼리 실행 결과를 ResultSet으로 받아서 정보를 저장할 오브젝트(User)에 옮겨 준다.
    -   작업 중에 생성된 Connection, Statement, ResultSet 같은 리소스는 작업을 마친 후 반드시 닫아준다.
    -   JDBC API가 만들어내는 예외를 잡아서 직접 처리하거나, 메소드에 throws를 선언해서 예외가 발생하면 메소드 밖으로 던지게 한다.

```java
public class UserDao {
	public void add(User user) throws ClassNotFoundException, SQLException {
		Class.forName("org.h2.Driver");
		Connection c = DriverManager.getConnection("jdbc:h2:tcp://localhost/~/springbook", "sa", "");

		PreparedStatement ps = c.prepareStatement("insert into users(id, name, password) values (?, ?, ?)");
		ps.setString(1, user.getId());
		ps.setString(2, user.getName());
		ps.setString(3, user.getPassword());

		ps.executeUpdate();

		ps.close();
		c.close();
	}

	public User get(String id) throws ClassNotFoundException, SQLException {
		Class.forName("org.h2.Driver");
		Connection c = DriverManager.getConnection("jdbc:h2:tcp://localhost/~/springbook", "sa", "");

		PreparedStatement ps = c.prepareStatement("select * from users where id = ?");
		ps.setString(1, id);

		ResultSet rs = ps.executeQuery();
		rs.next();

		User user = new User();
		user.setId(rs.getString("id"));
		user.setName(rs.getString("name"));
		user.setPassword(rs.getString("password"));

		rs.close();
		ps.close();
		c.close();

		return user;
	}
}
```

## 1.1.3 `main()`을 이용한 DAO 테스트 코드

```java
public static void main(String[] args) throws ClassNotFoundException, SQLException {
		UserDao dao = new UserDao();

		User user = new User();
		user.setId("whiteship");
		user.setName("백기선");
		user.setPassword("married");

		dao.add(user);

		System.out.println(user.getId() + " 등록 성공");

		User user2 = dao.get(user.getId());
		System.out.println(user2.getName());
		System.out.println(user2.getPassword());
		System.out.println(user2.getId() + " 조회 성공");
	}
```

## 초난감 DAO의 문제점 (1.2 ~ 1.3.3 내용 요약)

1. 코드 중복
    - `UserDao`에서 커넥션을 받아오는 코드가 메소드 마다 중복되고 있음
        ```java
        Connection c = DriverManager.getConnection("jdbc:h2:tcp://localhost/~/springbook", "sa", "");
        ```
    - 중복 되는 코드를 메소드로 분리하여 개선 가능
2. 확장 가능성

    - 만약 다른 종류의 DB를 이용해야한다면?
    - 코드 내부 구조를 공개하지 않고 DB Connection 구조만 바꿔서 사용하고 싶다면?
    - 상속을 통한 확장 가능
        - 메소드 오버라이딩으로 구체화하는 방법 - **템플릿 메소드 패턴**
        - 슈퍼클래스에 추상 메소드로 선언한 후, 서브클래스에서 구체화하는 방법- **팩토리 메소드 패턴**

3. 상속 사용

    - 상속을 사용하게된다면 다중상속 문제가 일어날 가능성이 있음
    - 상속을 통한 상하위 클래스의 관계는 밀접하므로 느슨한 결합을 보장하지 못함
    - 확장된 기능인 다양한 DB 커넥션을 생성하는 코드를 다른 DAO 클래스에 적용할 수 없음
    - `UserDAO` 내의 getConnection 메소드를 `DB 커넥션만 담당하는 Class`로 분리하여 개선 가능
    - 인터페이스를 도입하여 확장 가능성 보장 => **전략 패턴(Strategy Pattern)**
        - 객체지향의 **다형성** 이용: 하나의 메소드 사용으로 원하는 DB Connection을 이용할 수 있어야 한다. 런타임 시점에 DB Connection의 구체적인 종류가 결정되도록 한다.

## 1.3.4 원칙과 패턴

-   개방 폐쇄 원칙(OCP, Open-Closed Principle)
    -   클래스나 모듈은 확장에는 열려 있어야 하고 변경에는 닫혀 있어야 한다.
-   높은 응집도(High Conherence)
    -   기능을 변경해야 할 때 일부 모듈의 내용만 변경할 수 있는 것
-   낮은 결합도(Loose Coupling)
    -   하나의 오브젝트에서 변경이 일어날 때, 관계를 맺고 있는 다른 오브젝트에게 변화를 요구하는 정도가 낮은 것
    -   관계를 유지하는 데 꼭 필요한 최소한의 방법만 간접적인 형태로 제공하고, 나머지는 서로 독립적이고 알 필요도 없게 만들어주는 것

# 1.4 제어의 역전(IoC : Inversioin of Control)

프로그램 구조에서 각 오브젝트는 프로그램 흐름을 결정하거나 사용할 오브젝트를 구성하는 작업에 능동적으로 참여한다. 모든 오브젝트가 능동적으로 자신이 사용할 클래스를 결정하고, 언제 어떻게 그 오브젝트를 만들지를 스스로 관장한다. 모든 종류의 작업을 사용하는 쪽에서 제어하는 구조이다.

**제어의 역전**이란 이런 제어 흐름의 개념을 거꾸로 뒤집는 것이다.

프로젝트가 자신이 사용할 오브젝트를 스스로 선택하지 않는다. 모든 제어 권한을 다른 대상에게 위임하여 자신도 어떻게 만들어지고 어디서 사용되는지 알 수 없다.

### 장점

-   설계가 깔끔해짐
-   유연성 증가
-   확장성 증가

## 1.4.1 오브젝트 팩토리

UserDao가 어떤 ConnectionMaker 구현 클래스를 사용할지 결정하는 기능을 분리 (`DaoFactory`)

-   같은 커넥션을 사용하는 DAO가 여러개 일 때 활용 가능한 코드

    ```java
    public class DaoFactory{
        public UserDao userDao(){
            return new UserDao(connectionMaker());
        }

        public AccountDao accountDao(){
            return new AccountDao(connectionMaker());
        }

        public MessageDao messageDao(){
            return new MessageDao(connectionMaker());
        }

        public ConnectionMaker connectionMaker(){
            return new DconnectionMaker();
        }
    }
    ```

    ```java
    public class UserDaoTest{
        public static void main(String[] args) throws ClassNotFoundException, SQLException {
            UserDao dao = new DaoFactory().userDao();
            ...
        }
    }

    ```

# 1.5 스프링의 IoC

## 1.5.1 오브젝트 팩토리를 이용한 스프링 IoC

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class DaoFactory {
    @Bean
    public UserDao userDao() {
        ConnectionMaker connectionMaker = new DConnectionMaker();
        UserDao userDao = new UserDao(connectionMaker);
        return userDao;
        return new UserDao(connectionMaker());
    }

    @Bean
    public ConnectionMaker connectionMaker() {
        return new DConnectionMaker();
    }
}
```

-   `@Configuration`: 스프링이 빈 팩토리를 위한 오브젝트 설정을 담당하는 클래스라고 인식함
-   `@Bean`: 오브젝트를 생성하는 메소드에 붙임

```java
public class UserDaoTest{
    public static void main(String[] args) throws ClassNotFoundException, SQLException {
        ApplicationContext context = new AnnotationConfigApplicationContext(DaoFactory.class);
        UserDao userDao = context.getBean("userDao", UserDao.class); //UserDao 클래스 내부 메소드 userDao를 호출한 결과를 리턴
    }
}
```

## 1.5.2 애플리케이션 컨텍스트의 동작 방식

![image](https://user-images.githubusercontent.com/74856502/124291845-f7da0980-db8f-11eb-8503-26ea1e627386.png)

### 애플리케이션 컨텍스트 사용시 장점

-   클라이언트는 구체적인 팩토리 클래스를 알 필요가 없다. 일관된 방식으로 원하는 오브젝트를 가져올 수 있다.
-   애플리케이션 컨텍스트는 종합 IoC 서비스를 제공해준다.
-   애플리케이션 컨텍스트는 빈을 검색하는 다양한 방법을 제공한다.(타입, 빈의 이름, 어노테이션 등으로 검색 가능)

## 1.5.3 스프링 IoC의 용어 정리

-   빈(Bean), 빈 오브젝트
    -   스프링이 IoC 방식으로 관리하는 오브젝트
-   빈 팩토리(Bean Factory)
    -   스프링의 IoC를 담당하는 핵심 컨테이너
    -   빈을 등록/생성/조회/반환하고, 그외에 부가적인 빈을 관리하는 기능을 담당한다.
    -   `Interface: BeanFactory`에 `getBean()`메소드가 정의되어 있고, 이를 구현하여 사용한다.
-   애플리케이션 컨텍스트(Application Context)
    -   빈 팩토리를 확장한 IoC 컨테이너
    -   빈 팩토리의 빈 관리 역할을 제공하고, 스프링이 제공하는 각종 부가 서비스를 추가로 제공한다.
    -   `Interface: BeanFactory`을 상속한다.
    -   `Interface: ApplicationContext`를 구현하여 사용한다.
-   설정정보/메타정보(configuration metadata)
    -   Configuration: 구성정보, 형상정보
    -   IoC 컨테이너에 의해 관리되는 애플리케이션 오브젝트를 생성하고 구성할 때 사용된다.
-   컨테이너, IoC 컨테이너
    -   IoC 방식으로 빈을 관리한다는 의미에서 애플리케이션 컨텍스트나 빈 팩토리를 가리킨다.
    -   컨테이너/스프링 컨테이너
        -   애플리케이션 컨텍스트
    -   IoC 컨테이너
        -   빈 팩토리
-   스프링 프레임워크
    -   IoC 컨테이너, 애플리케이션 컨텍스트를 포함해서 스프링이 제공하는 모든 기능을 통틀어 말한다.

## 1.6 싱글톤 레지스트리와 오브젝트 스코프

## 싱글톤 패턴

한 개의 오브젝트만 만들어 사용하는 것, 서버 환경에서는 수많은 클라이언트 요청이 생길 수 있기 때문에 매번 오브젝트를 생성하면 엄청난 양의 오버헤드가 생긴다. 따라서 서버 환경에서는 싱글톤 사용이 권장된다.

## 싱글톤 구현

-   private 생성자
-   static 필드에 유일한 싱글톤 오브젝트 저장
-   `getInstance()`로 오브젝트 생성/반환 관리

```java
public class UserDao{
    private static UserDao INSTANCE;

    private UserDao(ConnectionMaker connectionMaker){
        this.connectionMaker = connectionMaker;
    }

    public static synchronized UserDao getInstance(){
        if(INSTANCE == null) INSTANCE = new UserDao(...); //생성된 인스턴스가 없다면 새로 생성
        return INSTANCE; //이미 존재한다면 생성된 인스턴스 반환
    }
}
```

## 싱글톤 패턴의 한계

-   private 생성자 때문에 상속 불가능, 다형성 적용이 어렵다.
-   테스트가 힘들다.
-   서버환경에서는 싱글톤이 하나만 만들어지는 것을 보장하지 못한다.
    -   여러 개의 JVM에 분산되어 설치되는 경우 각각 독립적으로 오브젝트가 생성됨
-   싱글톤의 사용은 전역 상태를 만들 수 있기 때문에 객체지향에 맞지 않는다.

## 싱글톤 레지스트리

스프링은 서버 환경에서 싱글톤이 만들어져서 서비스 오브젝트 방식으로 사용되는 것은 적극 지지한다. 또 싱글톤 형태의 오브젝트를 만들고 관리하는 기능인 **싱글톤 레지스트리를** 제공한다.

### 싱글톤 레지스트리 사용의 장점

-   static 메소드와 private 새성자를 사용해야하는 비정상적인 클래스가 아니라 평범한 자바 클래스를 싱글톤으로 사용하게 해 준다.
-   스프링이 지지하는 객체지향적인 설계방식과 원칙, 디자인 패턴을 적용하는 데 제약이 없다.

## 스프링 빈의 스코프

### 빈의 scope

-   스프링이 관리하는 오브젝트인 빈이 생성되고, 존재하고, 적용되는 범위
-   스프링의 기본 스코프는 싱글톤이다.
-   그 외 프로토타입 스코프 / 세션 스코프 / 요청 스코프 등이 있다.

## 1.7 의존 관계 주입(DI : dependency injection)

스프링이 제공하는 IoC 방식의 핵심은 의존관계 주입을 통해 이루어진다.
따라서 스프링을 DI 컨테이너라고 부르기도한다.

DI는 오브젝트 레퍼런스를 외부로부터 제공받고, 이를 통해 다른 오브젝트와 다이나믹하게 의존관계가 만들어지는 것이다.

### 의존관계

-   클래스 A -> 클래스 B
    -   B가 변하면 A가 영향을 받는 관계 = A가 B에 의존하고 있다.
    -   ex) A 클래스 내부에서 B 인스턴스를 사용하고 있는 경우
        -   이 때 사용대상인 B는 **의존 오브젝트**이고, B를 사용하는 A는 **클라이언트 오브젝트**이다.

### 의존관계 주입의 조건

-   클래스 모델이나 코드에는 런타임 시점의 의존관계가 드러나지 않는다. 그러기 위해서는 인터페이스에만 의존하고 있어야한다.

    -   인터페이스에 대해서만 의존 관계를 만들어두면 인터페이스 구현 클래스와의 관계는 느슨해지면서 결합도가 낮아진다.

-   런타임 시점의 의존관계는 컨테이너나 팩토리 같은 제3의 존재가 결정한다.
    -   스프링의 애플리케이션 컨텍스트, 빈 팩토리, IoC 컨테이너 등이 모두 외부에서 오브젝트 사이의 런타임 관계를 맺어주는 책임을 지닌 제3의 존재라고 볼 수 있다.
    -   DI를 원하는 오브젝트는 먼저 자기자신이 컨테이너가 관리하는 빈이 되어야 한다.
-   의존관계는 사용할 오브젝트에 대한 레퍼런스를 외부에서 제공(주입)해줌으로써 만들어진다.

### 의존관계 주입(dependency injection)

런타임에 `UserDao`를 생성하는 오브젝트에서 의존관계를 맺을 오브젝트를 파라미터로 주입시켜 의존관계를 맺는다.

```java
public class UserDao{
    private ConnectionMaker connectionMaker;

    public UserDao(ConnectionMaker connectionMaker){
        this.connectionMaker = connectionMaker;
    }
    ...
}
```

-   생성자를 이용한 주입
-   수정자(Setter) 메소드를 이용한 주입
-   일반 메소드를 이용한 주입

### 의존관계 검색(dependency lookup)

오브젝트 내부에서 `getBean()` 메소드를 이용해 미리 정해놓은 이름에 해당하는 오브젝트를 찾아 요청한다.

```java
public UserDao() {
    AnnotationConfigAppcationContext context = new AnnotationConfigAppcationContext(DaoFactory.class);
    this.connectionMaker = context.getBean("connectionMaker", ConnectionMaker.class);
}
```

-   클래스 분리가 깔끔하지 못하고 클래스간 역할 분리가 완벽하지 않음
    -   오브젝트 팩토리 클래스나 스프링 API가 기존 코드 안에 사용되기 때문
-   등록된 빈을 직접 가져와야 하는 경우, 가져오는 대상이 빈에 등록되지 않는 경우에 주로 사용한다.

## 1.8 XML을 이용한 설정

DI 의존관계 설정정보를 XML로 정의할 수 있다.

### XML

-   단순한 텍스트 파일이기 때문에 다루기 쉽고, 이해하기 쉽다.
-   컴파일과 같은 별도의 빌드 작업이 없다.
-   스키마나 DTD를 이용해서 정해진 포맷을 따라 작성됐는지 확인할 수 있다.
-   오브젝트 사이의 관계를 빠르게 변경사항을 반영할 수 있다.

```java
@Bean --------------------------------. <bean
public ConnectionMaker connectionMaker() { --------> id='connectionMaker'
    return new DConnectionMaker(); ----------------> class='springbook ... DConnectionMaker' />
}
```

-   클래스 설정 -> XML 설정
    -   `@Configuration` -> `<beans>`
    -   `@Bean methodName()` -> `<bean id="methodName" `
    -   `return new BeanClass();` -> `clsss="a.b.c...BeanClass>`

```java
userDao.setConnectionMaker(connectionMaker());
<property name="connectionMaker" ref="connectionMaker" />
```

-   수정자 메소드 -> <property name="setter 대상 프로퍼티" ref="주입할 오브젝트 빈 이름">

```xml
    <beans>
    <bean id="connectionMaker"
        class="springbook. ... .DConnectionMaker" />
    <bean id="userDao" class="springbook. ... .UserDao">
        <property name="connectionMaker" ref="connectionMaker" />
    </bean>
</beans>
```
