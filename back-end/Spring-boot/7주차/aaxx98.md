# JPA

객체지향과 관계형 데이터베이스의 패러다임 불일치 문제를 해결하기 위해 등장  
객체 중심으로 개발할 수 있게 하기 때문에 생산성이 향상되고, 유지보수가 용이해진다.

-   ### 객체지향 코드

    ```java
    User user = findUser();
    Group group = user.getGroup();
    ```

    User-Group의 부모-자식 관계가 명확하다.

-   ### 데이터베이스 조회

    ```java
    User user = userDao.findUser();
    Group group = groupDao.findGroup(user.getGroupId());
    ```

    두 객체의 관계와 상관 없이 따로 조회해야한다. SQL에 종속된 개발을 하게 된다.

## Spring Data JPA

JPA는 인터페이스로서 자바 표준 명세서이다. 따라서 사용하려면 Hibernate, Eclipse Link와 같은 구현체(인터페이스를 구현한 클래스)가 필요하다.

그러나 직접 구현체를 사용하지 않고, 구현체를 추상화 시킨 Spring Data JPA 모듈을 이용한다.

-   JPA <- Hibernate(구현체) <- Spring Data JPA

    -   구현체 교체의 용이성: Hibernate가 아닌 다른 구현체로 교체하기 쉽다. Spring Data JPA 내부에서 구현체 매핑을 지원해준다.
    -   저장체 교체의 용이성: 관계형 데이터베이스 외에 다른 저장소로 교체하기 쉽다.

## Posts.java (Entity Class)

```java
package com.jojoldu.book.springboot.domain.posts;

import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Getter //JPA 어노테이션
@NoArgsConstructor //롬복 어노테이션
@Entity //롬복 어노테이션
public class Posts {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 500, nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;

    private String author;

    @Builder
    public Posts(String title, String content, String author){
        this.title=title;
        this.content=content;
        this.author=author;
    }
}

```

-   `@Entity`: 테이블과 링크될 클래스를 나타냄, 클래스이름(CamelCase)->테이블이름(underscore_naming)
-   `@id`: 해당 테이블의 Primary Key 필드
-   `@GeneratedValue`: PK 생성 규칙 - 스프링부트 2.0에서는 GenerationType.IDENTITY 옵션을 추가해야 auto_increment가 된다.
-   `@Column`: 테이블의 칼럼을 나타낸다. 굳이 선언하지 않아도 해당 클래스의 필드가 모두 칼럼이 되지만 기본값(ex: 문자열의 경우 VARCHAR(255)가 기본값)을 변경하여 추가하고 싶은 옵션이 있을 때 사용한다.
-   `@NoArgsConstructor`: 기본 생성자 자동 추가
    ```java
    public Posts() {}
    ```
-   `@Getter`: 클래스 내 모든 필드의 Getter 함수 자동 생성
-   `@Builder`: 해당 클래스의 빌더 패턴 클래스 생성, 생성자 상단에 선언 시 생성자에 포함된 필드만 빌더에 포함

    -   생성자를 통해 최종값을 채운 후, DB에 삽입한다면 인자 위치를 알고있어야한다.

    ```java
    public Example(String a, String b){
        this.a=a;
        this.b=b;
    }

    new Example(b,a) //문제 발생
    ```

    -   Builder를 사용하면 어느 필드에 어떤 값을 채워야할지 명확하게 알 수 있다.

    ```java
    Example.builder()
        .a(a)
        .b(b)
        .build();

    ```

-   **Entity 클래스에서는 절대 Setter 메소드를 만들지 않는다.** 해당 필드의 값 변경이 필요하면 목적, 의도가 나타나는 메소드를 추가한다.

    ### 주문 취소 메소드의 예시

    ```java
    //잘못 된 사용
    public class Order{
        boolean status;
        public void setStatus(boolean status){
            this.status = status;
        }
    }

    public void 주문서비스의_취소이벤트(){
        order.setStatus(false);
    }
    ```

    ```java
    // 올바른 사용
    public class Order{
        boolean status;
        public void cancelOrder(){
            this.status = false;
        }
    }

    public void 주문서비스의_취소이벤트(){
        order.cancelOrder();
    }
    ```

## PostsRepository (Entity Repository)

```java
package com.jojoldu.book.springboot.domain.posts;
import org.springframework.data.jpa.repository.JpaRepository;

public interface PostsRepository extends JpaRepository<Posts,Long>{
}
```

-   `JpaRepository<Entity 클래스, PK 타입>`을 상속받음

## Spring 웹 계층

-   Web Layer
    -   @Controller (컨트롤러)
    -   JSP/Freemarker 등 뷰 템플릿
    -   외부 요청과 응답에 대한 전반적인 영역
-   Service Layer
    -   @Service (서비스영역)
    -   Controller와 Dao의 중간 영역에서 사용됨
    -   @Transactional
-   Repository Layer
    -   데이터 저장소에 접근하는 영역
    -   DAO(Data Acess Object)의 역할
-   Dtos
    -   DTO(Data Transfer Object), 계층간 데이터 교환을 위한 객체들의 영역
-   Domain Model
    -   @Entity
