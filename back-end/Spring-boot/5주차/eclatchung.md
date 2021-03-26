# SpringBoot


## `Gradle`
- 프로젝트를 위한 범용 빌드 도구
- `Groovy`를 기반으로 한 빌드 도구
- `Ant`와 `Maven`같은 이전 세대의 빌드 도구의 단점을 보완하고 장점을 취합하여 만든 오픈 소스로 공개된 빌드 도구
### `Maven`
- ArtifactId : 프로젝트 이름
		- 버전 정보를 생락한 `jar`파일의 이름
		- 지하철 1호선
- groupId : 프로젝트를 모든 프로젝트 사이에어서 고유하게 식별하게 해주는 것
	- 도메인 네임이여야한다.
	- package 명명 규칙을 따르도록한다.
	- 지하철 노선 전체

## `build.gradle`
`ext` : `build.gradle` 에서 사용하는 전역변수를 설정하겠다는 의미
`repositories` : 각종 의존성(라이브러리)들을 어떤 원격 저장소에서 받을 것인지 정함.
`dependencies`  : 프로젝트 개발에 필요한 의존성을 선언

## 스프링
- 스프링 프레임워크는 자바 기반 엔터프라이즈 애플리케이션 개발을 위한 포괄적인 인프라를 제공해주는 플랫폼
- 2002년 개발
- 엔터프라이즈 애플리케이션은 `JavaEE`(Java Enterprise Edition)을 통해 개발되었다.
- `JavaEE`는 `JavaSE`(Java Standard Edition)에서 서버츨 개발을 위해서 기능이 더해진 자바 버전
- 스프링은 JavaEE를 대체하기 위해 개발되었음
- `EJB` (Enterprise Java Beans): JavaEE에서 핵심적인 역할을함
	- EJB는 엔터프라이즈 애플리케이션을 쉽게 작성할 수 있도록 목표를 가지고 있지만 개발하기 어려움
	- 기업 환경의 시스템을 구현하기 위한 서버측 컴포넌트 모델
	
![스프링구조](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fuk9Fa%2FbtqxWdng3pc%2FQZmG37Lu36BD6ROHxBlUV1%2Fimg.png)

## 스프링 부트
- 스르링 기반의 상용화가 가능한 애플리케이션을 쉽게 만들기 위해 단독 실행을 가능하게 해주는 스프링 프로젝트
- 스프링을 쉽게 사용할 수 있도록 필요한 설정을 대부분 미리 세팅해놈
- `Tomcat`, `Jetty`, `Undertow`를 내장
	-  `jetty` : 자바 HTTP 서버, 자바 서블릿 컨테이너 
	- `톰캣`: 웹서버와 연동하여 실행할 수 있는 자바 환경을 제공
	- `Undertow` : 웹서버 , 서블릿 컨테이너 기능을 제공
- 기본 설정이 되어있는 starter컴포넌트를 제공
- 설정을 자동으로 실행