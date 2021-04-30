## 세션 저장소로 데이터베이스 사용하기

-   기본적으로 세션은 실행되는 WAS의 메모리에서 저장되고 호출된다.
-   내장 톰캣처럼 애플리케이션 실행 시 실행되는 구조에서는 항상 세션이 초기화된다. (배포할 때 마다 톰캣이 재시작됨)
-   2대 이상의 서버에서 서비스 하고 있다면 톰캣마다 세션 동기화 설정을 해야한다.

## 세션 저장소

### 1. 톰캣 세션을 사용

-   일반적으로 별다른 설정을 하지 않을 때 기본적으로 선택되는 방식
-   톰캣(WAS)에 세션이 저장되므로 2대 이상의 WAS가 구동되는 환경에서는 톰캣들 간의 세션 공유를 위한 추가 설정이 필요하다.

### 2. MySQL과 같은 데이터베이스를 세션 저장소로 사용

-   여러 WAS 간의 공용 세션을 사용할 수 있는 가장 쉬운 방법
-   많은 설정이 필요 없지만 결국 로그인 요청마다 DB IO가 발생하여 성능 이슈가 발생할 수 있다.- 보통 로그인 요청이 많이 없는 백오피스, 사내 시스템 용도로 사용한다.
-   설정이 간단하고 사용자가 많이 없는 서비스에 적합하다.

    -   spring-session-jdbc 등록

        -   build.gradle에 의존성 추가

        ```
        compile('org.springframework.session:spring-session-jdbc')
        ```

        -   application.properties에 세션 저장소 설정 추가

        ```
        spring.session.store-type=jdbc
        ```

        -   JPA로 인해 세션 테이블(`SPRING_SESSION`, `SPRING_SESSION_ARRIBUTE`)이 자동 생성된다.
        -   로그인하면 `SPRING_SESSION`에 세션이 등록되지만 스프링을 재실행하면 H2 데이터베이스도 재실행되므로 세션이 풀린다. 이후 AWS로 배포하게 되면 AWS의 데이터베이스 서비스인 RDS를 사용하게 되므로 이때 부터 세션이 풀리지 않게된다.

### 3. Redis, Memcached와 같은 메모리 DB를 세션 저장소로 사용

-   B2C 서비스에서 가장 많이 사용하는 방식
-   실제 서비스로 사용하기 위해서는 Embedded Rdis와 같은 방식이 아닌 외부 메모리 서버가 필요하다.(비용이 든다.)

## 네이버 로그인 등록

-   네이버 서비스 등록 후,
-   allication-oauth.properties에 정보 작성

    -   네이버에서는 공식으로 스프링 시큐리티를 지원하지 않아서, Common-OAuth2Provider에서 등록해주던 값들을 수동으로 작성해야 한다.

    ```
    # registration
    spring.security.oauth2.client.registration.naver.client-id=GJHtTqFCOWIagvn2moBo
    spring.security.oauth2.client.registration.naver.client-secret=0Kb9fiMI6G
    spring.security.oauth2.client.registration.naver.redirect-uri={baseUrl}/{action}/oauth2/code/{registrationId}
    spring.security.oauth2.client.registration.naver.authorization-grant-type=authorization_code
    spring.security.oauth2.client.registration.naver.scope=name,email,profile_image
    spring.security.oauth2.client.registration.naver.client-name=Naver

    # provider
    spring.security.oauth2.client.provider.naver.authorization-uri=https://nid.naver.com/oauth2.0/authorize
    spring.security.oauth2.client.provider.naver.token-uri=https://nid.naver.com/oauth2.0/token
    spring.security.oauth2.client.provider.naver.user-info-uri=https://openapi.naver.com/v1/nid/me
    spring.security.oauth2.client.provider.naver.user-name-attribute=response
    ```

-   네이버 오픈 API의 로그인 회원 결과

    ```json
    {
        "resultCode": "00",
        "message": "success",
        "response": {
            "email": ...,
            "nickname": ...,
            "profile_image": ...,
            "age": ...,
            "gender": ...,
            "id": ...,
            "name": ...,
            "birthday": ...,
            ...
        }
    }
    ```

    -   스프링 시큐리티에서는 하위 필드를 명시할 수 없기 때문에 최상위 필드들만 user_name으로 지정할 수 있다.
    -   네이버의 응답값 최상위 필드는 resultCode, message, response이다.

-   OAuthAttribute 확장

```java
    public static OAuthAttributes of(String registrationId, String userNameAttributeName, Map<String ,Object> attributes){
        if("naver".equals(registrationId)){
            return ofNaver("id", attributes);
        }
        return ofGoogle(userNameAttributeName, attributes);
    }

    private static OAuthAttributes ofGoogle(String userNameAttributeName, Map<String, Object> attributes) {
        return OAuthAttributes.builder()
                .name((String) attributes.get("name"))
                .email((String) attributes.get("email"))
                .picture((String) attributes.get("picture"))
                .attributes(attributes)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }
    private static OAuthAttributes ofNaver(String userNameAttributeName, Map<String, Object> attributes){
        Map<String, Object> response = (Map<String, Object>) attributes.get("response"); //최상위 필드 response를 받아옴

        return OAuthAttributes.builder()
                .name((String) response.get("name"))
                .email((String) response.get("email"))
                .picture((String) response.get("profile_image"))
                .attributes(response)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }

```

-   index.mustache에 네이버 로그인 버튼 추가

```html
{{^userName}}
<a
    href="/oauth2/authorization/google"
    class="btn btn-success active"
    role="button"
    >Google Login</a
>
<a
    href="/oauth2/authorization/naver"
    class="btn btn-secondary active"
    role="button"
    >Naver Login</a
>
{{/userName}}
```

## 기존 테스트에서 시큐리티 적용하기

-   시큐리티 적용으로 발생하는 문제
    -   기존에는 바로 API를 호출할 수 있어 테스트 코드 역시 바로 API를 호출하도록 구성했으나, 시큐리티 옵션이 활성화되면 인증된 사용자만 API를 호출 할 수 있다.
    -   기존 API 테스트 코드 모두 인증에 대한 권한을 받지 못했으므로, 테스트 코드마다 인증한 사용자가 호출한 것 처럼 작동하도록 수정해야한다.

### 테스트 코드 수정

-   1. CustomOAuth2UserService를 찾을 수 없음

    -   src/main과 src/test의 환경 차이 때문에 발생한다.
    -   main의 `application.properties` 설정은 test에서 가져와 사용하지만 `application-oauth.properties` 설정은 공유되지 않는다.
    -   가짜 설정값을 src/test에 별도로 등록해줘야한다.
    -   src/test/resources/application.properites

    ```
    spring.jpa.show_sql=true
    spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
    spring.h2.console.enabled=true
    spring.session.store-type=jdbc

    # Test OAuth

    spring.security.oauth2.client.registration.google.client-id=test
    spring.security.oauth2.client.registration.google.client-secret=test
    spring.security.oauth2.client.registration.google.scope=profile,email
    ```

-   2. 302 Status Code **(302 리다이렉션 응답)**

    -   스프링 시큐리티 설정으로 인해 인증되지 않은 사용자의 요청은 이동시킨다.
    -   임의로 인증된 사용자를 추가하여 API만 테스트 해야한다.

        -   의존성 추가

        ```
        testCompile('org.springframework.security:spring-security-test')
        ```

        -   PostsApiControllerTest의 @Test 메소드에 사용자 인증 추가 어노테이션 추가

        ```java
        @Test
        @WithMockUser(roles = "USER")
        public void Posts_등록된다() throws Exception {
            ...
        }
        @Test
        @WithMockUser(roles = "USER")
        public void Posts_수정된다() throws Exception {
            ...
        }
        ```

-   3. @WebMvcTest에서 CustomOAuth2UserService를 찾을 수 없음
    -   @WebMvcTest에서 CustomOAuth2UserService을 스캔하지 않기 때문에 발생
    -   @WebMvcTest의 스캔 대상에서 SecurityConfig를 제거해야한다.
    ```java
    @RunWith(SpringRunner.class)
    @WebMvcTest(controllers = HelloController.class, excludeFilters = {
            @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = SecurityConfig.class)
    }) // 스캔 대상에서 SecurityConfig 제거
    public class HelloControllerTest { ... }
    ```
    -   HelloControllerTest의 @Test 메소드에도 `@WithMockUser(roles = "USER")`를 이용하여 가짜로 인증된 사용자를 생성해야한다.
    -   @EnableJpaAuditing으로 인한 에러
        -   @EnableJpaAuditing을 사용하려면 최소 하나의 @Entiy 클래스가 필요하다.
        -   `Application.java`에서 @EnableJpaAuditing을 제거하고, `config/JpaConfig.java`를 생성하여 @EnableJpaAuditing을 추가한다.
