# 스프링 시큐리티와 스프링 시큐리티 Oauth2 클라이언트

-   스프링 시큐리티

    -   스프링 기반 애플리케이션에서 보안을 위한 표준 프레임워크
    -   보안 기능을 직접 구현하는 것 보다는 스프링 시큐리티를 통해 구현하는 것을 적극 권장(서비스 개발에 집중)
    -   구글, 네이버, 페이스북 등에서 다음과 같은 OAuth API를 제공함
        -   로그인 시 보안
        -   회원가입 시, 이메일 또는 전화번호 인증
        -   비밀번호 찾기
        -   비밀번호 변경
        -   회원정보 변경

## 구글 로그인 등록

-   구글 서비스 등록 후,
-   application-oauth.properties에 정보 작성
    ```
    spring.security.oauth2.client.registration.google.client-id=[클라이언트ID]
    spring.security.oauth2.client.registration.google.client-secret=[클라이언트보안PW]
    spring.security.oauth2.client.registration.google.scope=profile,email
    ```
-   src/main/java/.../domain/user/User.java
    -   사용자 정보를 담당할 도메인
    -   id, name, email, picture, role
-   src/main/java/.../domain/user/Role.java
    -   각 사용자의 권한을 관리하는 Enum 클래스
-   src/main/java/.../domain/user/UserRepository.java
    -   User의 CRUD 담당
-   src/main/java/.../config/auth/SecurityConfig.java

    -   URL별 권한 관리, 로그인 성공 시 이동할 위치 설정

    ```java
    package com.jojoldu.book.springboot.config.auth;

    ...

    @RequiredArgsConstructor
    @EnableWebSecurity
    public class SecurityConfig extends WebSecurityConfigurerAdapter {
    private final CustomOAth2UserService customOAth2UserService;

        @Override
        protected void configure(HttpSecurity http) throws Exception {
            http
                    .csrf().disable()
                    .headers().frameOptions().disable()
                    .and()
                        .authorizeRequests() // 권한 관리
                        .antMatchers("/", "/css/**","/images/**","/js/**","/h2-console/**").permitAll() // 모든 사용자 허용
                        .antMatchers("/api/v1/**").hasRole(Role.USER.name()) // role이 USER인 사람만 허용
                        .anyRequest().authenticated() //나머지 경로는인증된 사용자에게만 허용
                    .and()
                        .logout()
                            .logoutSuccessUrl("/") //로그아웃 성공시 "/"로 이동
                    .and()
                        .oauth2Login()
                            .userInfoEndpoint()
                                .userService(customOAth2UserService);

        }

    }

    ```

-   src/main/java/.../config/auth/CustomOAuth2UserService.java

    -   가입/정보수정/세션저장 등 기능 작성

-   src/main/java/.../config/auth/dto/SessionUser.java

    -   인증된 사용자 정보
    -   name, email, picture를 User 객체로 부터 받아옴
    -   User 클래스는 엔티티이기 때문에 다른 클래스와 관계가 형성될 수 있다.(직렬화 과정에서 성능 이슈, 부수 효과가 발생함) 따라서 직렬화 기능을 가진 세션 Dto를 별도로 만들어 사용한다.
        -   직렬화(Serialization)
            -   객체를 스트림 형식으로 파일에 저장하는 방법
            -   Serialization 인터페이스를 구현해야한다.

-   index.mustache

    ```
    <!--로그인 기능 영역-->
    <div class="row">
        <div class="col-md-6">
            <a href="/posts/save" role="button" class="btn btn-primary"
                >글 등록</a
            >
            {{#userName}}
            Logged in as: <span id="user">{{userName}}</span>
            <a href="/logout" role="button" class="btn btn-info active">Logout</a>
            {{/userName}}
            {{^userName}}
            <a
                href="/oauth2/authorization/google"
                class="btn btn-success active"
                role="button"
                >Google Login</a
            >
            {{/userName}}
        </div>
    </div>
    <br />
    ```

    -   `{{#userName}}`~`{{/userName}}` : `userName` attribute가 있을 때 노출되는 내용
        -   "/logout" : 스프링 시큐리티에서 기본적으로 제공하는 로그아웃 URL
    -   `{{^userName}}`~`{{/userName}}` : `userName`attribute가 없을 때 노출되는 내용
        -   "/oauth2/authorization/google" : 스프링 시큐리티에서 기본적으로 제공하는 로그인 URL

## 어노테이션 기반으로 개선하기

-   문제점: 여러 컨트롤러와 메소드에서 세션 값을 가져 올 때 마다 같은 코드 반복
    -   `SessionUser user = (SessionUser) httpSession.getAttribute("user");`
-   어노테이션 생성

```java
package com.jojoldu.book.springboot.config.auth;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.PARAMETER) // 어노테이션 생성 위치 지정
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser {
}
```

-   src/main/java/.../config/auth/LoginUserArgumentResolver.java

    -   조건에 맞는 경우 메소드가 있다면 HandlerMethodArgumentResolver의 구현체가 지정한 값으로 해당 메소드의 파라미터로 넘길 수 있다.
    -   아래 코드는 `@LoginUser` 어노테이션이 붙은 파라미터에 자동으로 httpSession에 저장되어있는 user를 넘겨주도록 한다.

    ```java
    package com.jojoldu.book.springboot.config.auth;

    ...

    @RequiredArgsConstructor
    @Component
    public class LoginUserArgumentResolver implements HandlerMethodArgumentResolver {
        private final HttpSession httpSession;

        @Override
        public boolean supportsParameter(MethodParameter parameter){ // 특정 파라미터를 지원하는지 판단
            boolean isLoginUserAnnotaion = parameter.getParameterAnnotation(LoginUser.class) != null;
            boolean isUserClass = SessionUser.class.equals(parameter.getParameterType());
            // 파라미터에 @LoginUser 어노테이션이 붙어있고, 파라미터 클래스 타입이 SessionUser.class인 경우
            return isLoginUserAnnotaion && isUserClass;
        }

        @Override
        public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
            // 파라미터에 전달할 객체를 세션으로부터 받아옴
            return httpSession.getAttribute("user");
        }
    }
    ```

-   IndexController.java 코드 개선

    ```java
    @GetMapping("/")
    public String index(Model model, @LoginUser SessionUser user){
        model.addAttribute("posts", postsService.findAllDesc());
        if(user!=null){
            model.addAttribute("userName", user.getName());
        }
        return "index";
    }
    ```
