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
