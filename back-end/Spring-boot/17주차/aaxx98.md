# 무중단 배포

배포하는 동안, 새로운 Jar가 실행되기 전까지는 기존 Jar을 종료시켜놓기 때문에 서비스가 종료된다. 서비스를 정지하지 않고 배포하는 것이 무중단 배포이다.

### 무중단 배포 방식

-   AWS에서 블루그린(Blue-Green) 무중단 배포
-   도커를 이용한 웹서비스 무중단 배포
-   L4 스위치를 이용한 무중단 배포(비용이 많이 든다.)
-   **엔진엑스를 이용한 무중단 배포**

### 엔진엑스

웹서버, 리버스 프록시, 캐싱, 로드 밸런싱, 미디어 스트리밍 등을 위한 오픈소스 웹서버이다.

-   리버스 프록시
    -   엔진엑스가 외부의 요청을 받아 백엔드 서버로 요청을 전달하는 행위
    -   리버스 프록시를 통해 무중단 배포 환경 구축

### 구조

하나의 EC2(혹은 리눅스) 서버에 엔진엑스 1대와 스프링부트 Jar 2대를 사용함

-   엔진엑스는 80(http), 443(https) 포트를 할당한다.
-   스프링 부트1은 8081 포트로 실행한다.
-   스프링 부트2는 8082 포트로 실행한다.

### 운영 과정

1. 사용자는 서비스 주소로 접속한다.(80 or 443 포트)
2. 엔진엑스는 사용자의 요청을 받아 현재 연결된 스프링 부트로 요청을 전달한다.

    - 스프링 부트 1(8081 포트)로 요청을 전달한다고 가정

        `사용자->엔진엑스->스프링부트1`

3. 스프링 부트2는 엔진엑스와 연결된 상태가 아니니 요청받지 못한다.

**신규 배포가 필요하면, 엔진엑스와 연결되지 않은 스프링 부트2(8082 포트)로 배포한다.**

1. 배포하는 동안에도 서비스는 중단되지 않는다.

    - 엔진엑스는 스프링 부트1을 바라보기 때문이다.

2. 배포가 끝나고 스프링 부트2가 정상적으로 구동중인지 확인한다.

3. 스프링 부트2가 정상 구동중이면 `nginx reload` 명령을 통해 8081 대신 8082를 바라보게 한다.

    `사용자->엔진엑스->스프링부트2`

4. `nginx reload`는 0.1초 이내에 완료된다.

## 엔진엑스 설치와 스프링 부트 연동

-   엔진엑스 설치

    > sudo amazon-linux-extras install nginx1

    -   엔진엑스 실행
        > sudo service nginx start
    -   재실행
        > sudo service nginx restartd
    -   확인

        > ps -ef | grep nginx

    -   종료
        > sudo service nginx stop

-   보안 그룹 추가

    -   엔진엑스의 포트번호 80

-   리다이렉션 주소 추가

    -   네이버 로그인, 구글 클라우드에 포트번호 80 등록

-   엔진엑스와 스프링부트 연동

    -   `/etc/nginx/nginx.conf` (엔진엑스 설정)
        ```conf
        location / {
            proxy_pass http://localhost:8080; --- 요청이 들어오면 여기로 전달
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
        }
        ```

-   무중단 배포 스크립트

    -   `ProfileController.java`

        ```java
        @RequiredArgsConstructor
        @RestController
        public class ProfileController {
            private final Environment env;

            @GetMapping("/profile")
            public String profile() {
                List<String> profiles = Arrays.asList(env.getActiveProfiles());
                List<String> realProfiles = Arrays.asList("real", "real1", "real2");
                String defaultProfile = profiles.isEmpty()? "default" : profiles.get(0);

                return profiles.stream()
                        .filter(realProfiles::contains)
                        .findAny()
                        .orElse(defaultProfile);
            }
        }
        ```

-   `real1.properties`, `real2.properties` 추가

    ```properties
    server.port=8081
    spring.profiles.include=oauth,real-db
    spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
    spring.session.store-type=jdbc
    ```

    -   real2 에서는 포트번호 8082

-   엔진엑스 설정 수정

    -   `/etc/nginx/conf.d/service-url.inc` 파일 생성
        > sudo vim /etc/nginx/conf.d/service-url.inc
        ```inc
        set $service_url http://127.0.0.1:8080;
        ```
    -   `etc/nginx/nginx.conf` 내용 수정

        ```
        # Load configuration files for the default server block.
        include /etc/nginx/conf.d/service-url.inc;

        location / {
                proxy_pass $service_url;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
        }
        ```

## 배포 스크립트 작성

-   스크립트

    -   `stop.sh`: 기존 엔진엑스에 연결되어 잇진 않지만 실행중이던 스프링 부트 종료
    -   `start.sh`: 배포할 신규 버전 스프링 부트 프로젝트를 `stop.sh`로 종료한 'profile'로 실행
    -   `health.sh`: `start.sh`로 실행시킨 프로젝트가 정상적으로 실행되었는지 체크
    -   `switch.sh`: 엔진엑스가 바라보는 스프링 부트를 최신 버전으로 변경
    -   `profile.sh`: 앞선 4개 스크립트 파일에서 공용으로 사용할 'profile'과 포트 체크 로직

-   `appspec.yml`에 스크립트 설정
    ```yml
    hooks:
        AfterInstall:
            - location: stop.sh # 엔진엑스와 연결되어 있지 않은 스프링 부트를 종료
            timeout: 60
            runas: ec2-user
        ApplicationStart:
            - location: start.sh # 엔진엑스와 연결되어 있지 않은 Port로 새 버전의 스프링 부트를 시작
            timeout: 60
            runas: ec2-user
        ValidateService:
            - location: health.sh # 새 스프링 부트가 정상적으로 실행됐는지 확인
            timeout: 60
            runas: ec2-user
    ```

## 자동으로 버전값 변경

-   `build.gradle`의 version
    ```gradle
    version '1.0.4-SNAPSHOT-'+new Date().format("yyyyMMddHHmmss")
    ```

## 무중단 배포 테스트

-   CodeDeploy 로그 확인

    > tail -f /opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log

-   스프링 부트 로그 확인

    > vim ~/app/step3/nohup.out

-   자바 애플리케이션 실행 여부 확인
    > ps -ef | grep java
    -   real1, real2가 실행되고 있는 것을 확인할 수 있다.
