# 스프링 시큐리티와 OAuth2.0으로 로그인 기능 구현하기-2

## 세션 저장소로 데이터베이스 사용하기

- 내장 톰캣의 메모리에 세션이 저장되면 애플리케이션 재실행시 로그인이 풀림
	- 배포할때마다 톰캣이 재시작이 된다.
	- 해결방안 ⇒ 토캣마다 세션 동기화 설정을 해야함
	- 실제 현업에서 세션 저장소 사용에 대한 설정 방법
		- 톰캣 세션 사용하기
			- WAS 2 구동시 톰캣(WAS)간 세션 공유 설정 추가 필요
			- 일반적으로 별다른 설정하지 않을때 기본적으로 선택됨
		- MySQL과 같은 데이터베이스를 세션 저장소로 사용
			- 여러 WAS간 공용 세션 사용시 가장 쉬운방법
			- DB IO 발생해서 성능상 이슈가 생길 수도 있음
		- Redis, Memcached와 같은 메모리 DB를 세션 저장소로 사용
			- B2C 서비스( business to consumer)

## 네이버 로그인
- 스프링 시큐리티에서는 하위 필드를 명시할 수 없음 -> 최상위 필드들만 user_name으로 지정이 가능함 (response의 하위 내용들이 하위 필드)
	- 네이버 생성자에서 response의 id를 user_name으로 지정

```JAVA
private static OAuthAttributes ofNaver(String userNameAttributeName, Map<String, Object> attributes) {
    Map<String, Object> response = (Map<String, Object>) attributes.get("response"); // 해당 부분에서 attributes에서 response를 가져옴

    return OAuthAttributes.builder()
            .name((String) response.get("name"))
            .email((String) response.get("email"))
            .picture((String) response.get("profile_image"))
            .attributes(response)
            .nameAttributeKey(userNameAttributeName)
            .build();
}

```

## 기존 테스트에 시큐리티 적용하기
- 시큐리티 옵션이 활성화되면서 API 테스트들이 모두 인증에 대한 권한을 받지 못함으로 테스트 코드마다 인증한 사용자가 호출한것 처럼 수정을 해야함.

### 문제들
1. CustomerOAuth2UserService를 찾을 수 없음
	- 소셜 로그인관련 설정값들이 없기 때문에 발생
	- `src/test`와 `src/main`의 환경이 차이 난다.
		- `test`에서 들고오는 프로퍼티는 `application.properties`파일까지임 →`application-oauth.properties`는 안들고옴.
2. 302 status code 
	- 인증되지 않은 사용자의 요청은 이동시킴
	- spring-security-test추가하여 사용
3. @WebMvcTest에서 CustomOAuth2UserService을 찾을 수 없음
	- @WebMvcTestsms CustomerOAuthUserService를 스캔하지 않음
		- @Repository, @Service, @Component는 스캔 대상이 아님
	- 단, SecurityConfig는 스캔했는데 SecurityConfig를 생성하기 위해서 필요한 CustomOAuth2UserService를 못찾아서 생긴 문제
		- SecurityConfig를 스캔하지 않는다. 
4. java.lang.IllegalArgumentException:At least one JPA meta model must be present!
	- @EnableJpaAuding로 인해 발생
	- @EnableJpaAuding -> 하나의 @Entity클래스 필요
		- @EnableJpaAuding 제거 
