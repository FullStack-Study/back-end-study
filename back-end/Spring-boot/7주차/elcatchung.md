# 스프링 부트에서 JPA로 데이터베이스를 다뤄보자

## `SQL 매퍼`와 `ORM`
1. `SQL MAPPER`
	- iBatis (MyBatis (현))
	- 각 테이블 마다 기본적인 CRUD sql을 생성해야함 ⇒ SQL 피해갈 수 가 없음
	- 객체지향 프로그래밍 보단 테이블 모델링에 집중함
	- 부작용
		- 패러다임 불일치 : 
			- 관계형 DB : 어떻게 데이터를 저장할지 초점
			- 객체지항 프로그래밍 : 메시지 기반으로 기능과 속성을 한 곳에서 관리
		- 단순 반복작업
		- 관계 파악에 힘듬 ⇒ 객체지향 프로그램ㅇ에 불편
2. `ORM` : Object Relational Mapping
	- `JPA` : 자바 표준 ORM
		- 인터페이스 / 자바 표준 명세서
		- 구현체 필요 : `Hibernate`, `Eclipse Link`등
		- spring boot : Spring Data JPA 사용
			- JPA <- Hibernate <- Spring Data JPA
			- 이점
				- 구현체 교체의 용이성  : Hibernate외의 다른 구현체로 쉽게 교체할 수 있음
				- 저장소 교체의 용이성 : 관계형 데이터베이스 외에 다른 저장소로 쉽게 교체 / Spring Data JPA→ Spring Data MongoDB
	- 객체를 매핑
	- 객체지향 프로그래밍과 관계형 데이터 베이스 사이에서 `패러디암`을 일치 시킴
	- 이점
		- 생산성 향상
		- 유지보수 편함

### 도메인 패키지 
- 도메인을 담은 패키지
- 도메인`Domain`: 소프트웨어에 대한 요구사항 혹은 문제 영역

### 주의 사항
- Entity클래스에서는 Setter 메소드 생성안함
	- 기본적인 생성자를 통해 최종값 채운후 DB삽입 
	- 값변경은 해당 이벤트에 맞는 public 메소드 호출
- `@Builder` 사용시에  어떤 값이 어떤 필드에 채워야 할지 쉽게 알 수 있음
```JAVA
Example.builder()
		.a(a)
		.b(b)
		.build(); 
```
- 주요 어노테이션을 클래스에 가까이둠 ⇒ 코틀린 등 새 언어 전환으로 롬복이 더이상 필요 없을 때 쉽게 삭제 가능
- Repository 인터페이스 <- DB Layer 접근자
- `JpaRespository<Entity클래스, PK type> `상속시 기본 CRUD 메소드 자동 생성
- Entity 클래스와 기본 Entity Repository 는 함게 위치, Entity클래스는 기본 Repository 없이는 제대로 역할 못함

#### API 생성시 3가지 클래스 필요
1. Request 데이터 받을 DTO
2. API 요청 받을 Controller
3. 트랜잭션, 도메인 기능간의 순서 보장하는 서비스

[견고한 node.js 프로젝트 설계하기](https://velog.io/@hopsprings2/%EA%B2%AC%EA%B3%A0%ED%95%9C-node.js-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%95%84%ED%82%A4%ED%85%8D%EC%B3%90-%EC%84%A4%EA%B3%84%ED%95%98%EA%B8%B0)

* 서비스 ≠ 비지니스 로직 처리
* 서비스 == 트랜잭션, 도메인간 순서 보장

## Spring Web 계층 
![스프링 웹 계층](https://blog.kakaocdn.net/dn/bFruEV/btqAUv4HJLQ/H5TVBjqkKc5KBgD4Vdyvkk/img.png)


	