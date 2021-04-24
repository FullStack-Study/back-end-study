# 스프링 시큐리티와 OAuth2.0으로 로그인 기능 구현하기

OAuth를 써도 구현해야하는 것 제외하고 로그인 구현시 구현해야하는 것
1. 로그인 보안
2. 회원가입시 이메일 혹은 전화번호 인증
3. 비밀번호 찾기
4. 비밀번호 변경
5. 회원정보 변경

### 스프링 부트 1.5 vs 스프링 부트 2.0
- 스프링 부트 1.5
	- url 명시
- 스프링 부트 2.0
	- client 인증 정보
- OAuth2 연도 방법 변화
- `spring-security-oauth2-autoconfiure` 라이브러리 덕에 차이가 없음

- `Spring Security Oauth2 Client`사용이유
1. spring-security-oauth는 유지 상태로 변경
2. 스프링 부트용 라이브러리 출시
3. 신규라이브러리는 확장 포인트를 고려해서 설계해야함

- 승인된 url 방식(스프링 부트 2)
	- {도메인}/login/oauth2/code/{소셜서비스코드}

### properites file
`application-xxx.properties`
- xxx이름의 profile 생성
- `profile=xxx`로 호출하여 해당 properties의 설정을 가져올 수 있음


### domain/user/User.class
```java

package com.eclat.study.domain.user;

import com.eclat.study.domain.BaseTimeEntity;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.EnumType;
import javax.persistence.Enumerated;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Getter
@NoArgsConstructor
@Entity
public class User extends BaseTimeEntity
{
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String email;

    @Column
    private String picture;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private Role role;


    @Builder
    public User(String name,String email, String picture, Role role){
        this.name = name;
        this.email = email;
        this.picture = picture;
        this.role = role;
    }

    public User update(String name, String picture){
        this.name = name;
        this.picture = picture;

        return this;
    }

    public String getRoleKey(){
        return this.role.getKey();
    }

}


```

### domain/user/Role.enum
```java

package com.eclat.study.domain.user;

import lombok.Getter;
import lombok.RequiredArgsConstructor;

@Getter
@RequiredArgsConstructor
public enum Role {

    GUEST("ROLE_GUEST", "손님"),
    USER("ROLE_USER","일반 사용자");

    private final String key;
    private final String title;
}


```

스프링 시큐리티에서는 권한 코드에 항상 `ROLE_`이 앞에 있어야 한다.

### config.auth.SecurityConfig.class
```java
package com.eclat.study.config.auth;

import com.eclat.study.domain.user.Role;
import lombok.RequiredArgsConstructor;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@RequiredArgsConstructor
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final CustomOAuth2UserService customOAuth2UserService;

    @Override
    protected void configure(HttpSecurity http) throws Exception{
        http
                .csrf().disable()
                .headers().frameOptions().disable()
                .and()
                    .authorizeRequests()
                    .antMatchers("/","/css/**","/images/**","/js/**","/h2-console/**").permitAll()
                    .antMatchers("/api/v1/**").hasRole(Role.USER.name())
                    .anyRequest().authenticated()
                .and()
                    .logout()
                        .logoutSuccessUrl("/")
                .and()
                    .oauth2Login()
                    .userInfoEndpoint()
                    .userService(customOAuth2UserService);
    }
}
```

#### `@ EnableWebSecurity`
	- spring security 설정들을 활성화 시켜줌
#### `.csrf().disable().headers().frameOptions().disable()`
	- h2-console 화면을 사용하기 위해서 해당 옵션 disable
#### `authorizeRequests()`
	- URL별 권한 관리를 설정하는 시작점
	- 해당 authorizeRequests()이 선언되어야 `antMatchers`옵션 사용가능
#### `antMatchers`
	- 권한 관리 대상지정
	- URL, HTTP메소드 별로 관리 가능
	- permitAll()	 전체 열람 권한
	- hasRole() 해당 권한을 가진 사람만 가능
#### `anyRequest`
	- 설정된 값 이외의 나머지 URL
	- authenticated() : 인증된 사용자들에게만 허용 ⇒ 로그인한 사용자
#### `.logout().logoutSuccessUrl("/")`
	- 로그아웃 기능에 대한 여러 설정의 진입점
	- 로그아웃 설정시 /주소로 이동

### config.auth.dto.SessionUser.java
```java

package com.eclat.study.config.auth.dto;

import com.eclat.study.domain.user.User;
import lombok.Getter;

import java.io.Serializable;

@Getter
public class SessionUser implements Serializable {
    private String name;
    private String email;
    private String picture;

    public SessionUser(User user) {
        this.name = user.getName();
        this.email = user.getEmail();
        this.picture = user.getPicture();
    }
}

```

#### Serializable : 직렬화
- 자바 직렬화
	- 자바 시스템 내부에서 사용되는 객체 또는 데이터를 외부의 자바 시스템에서도 사용할 수 있도록 바이트(byte)형태로 데이터 변혼하는 기술과 바이트로 변환된 데이터를 다시 객체로 변환하는 기술(역직렬화)를 말한다.
- 조건
	- `java.io.Serializable`인터페이스를 상속받은 객체

### config.auth.dto.LoginUser.java
```java

package com.eclat.study.config.auth;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser {
}
```

#### `@interface`
- 어노테이션 클래스로 지정