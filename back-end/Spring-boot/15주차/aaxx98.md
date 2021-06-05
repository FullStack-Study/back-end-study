# Travis CI 배포 자동화

## 1. CI & CD 소개

### CI(Continuous Integration): 지속적인 통합

-   코드 버전관리를 하는 VCS(Git, SVN 등)에 PUSH가 되면 자동으로 테스트와 빌드가 수행되어 안정적인 배포파일을 만드는 과정

### CD(Continuous Deployment): 지속적인 배포

-   빌드 결과를 자동으로 운영 서버에 무중단 배포까지 진행하는 과정

## 2. Travis CI 연동하기

> https://www.travis-ci.org/ 가 아닌 https://www.travis-ci.com/ 로 접속하자...

[Travis CI](https://www.travis-ci.com/) > 깃허브로 로그인 > Settings에서 저장소 활성화

### 프로젝트 설정

-   Travis CI의 상세 설정은 프로젝트 내의 `.travis.yml` 파일로 할 수 있다.

    -   `.yml`: YAML(야믈) 파일, JSON에서 괄호를 제거한 형태

    ```yml
    language: java
    jdk:
        - openjdk8

    branches:
    only:
        - master

    # Travis CI 서버의 Home
    cache:
    directories:
        - "$HOME/.m2/repository"
        - "$HOME/.gradle"

    before_install:
        - chmod +x gradlew

    script: "./gradlew clean build"

    # CI 실행 완료 시 메일로 알람
    notifications:
    email:
    recipients:
        - 본인 메일 주소
    ```

    -   branches: 어느 브랜치가 푸시될 때 Travis CI를 실행할 지 결정
    -   cache: 그레이들을 통해 의존성을 받게 되면 이를 해당 디렉토리에 캐시하여 같은 의존성은 다음 배포 때 부터 다시 받지 않도록 설정
    -   script: master 브랜치에 푸시되었을 때 수행하는 명령어
    -   notifications: Travis CI 실행 완료 시 자동으로 알람이 가도록 설정

## 3. Travis CI와 AWS S3 연동하기

**S3** : AWS에서 제공하는 파일 서버

-   AWS Key 발급
    -   AWS 서비스에 외부 서비스가 접근할 수 없다. `IAM(Identity and Access Management)`로 접근 가능한 권한을 가진 Key를 생성해야 한다.
-   S3 버킷 생성

    -   Travis CI의 레파지토리 Setting에서 환경변수 설정
        -   AWS_ACCESS_KEY : 엑세스 키
        -   AWS_SECRET_KEY : 비밀 엑세스 키
    -   `.travis.yml` 추가

    ```yml
    before_deploy:
        - zip -r springboot-webservice ./*
        - mkdir -p deploy
        - mv springboot-webservice.zip deploy/springboot-webservice.zip

    deploy:
        - provider: s3
    access_key_id: $AWS_ACCESS_KEY ## Travis CI 환경변수
    secret_access_key: $AWS_SECRET_KEY ## Travis CI 환경변수

    bucket: swchoi-springboot-build-aaxx98 ## S3버킷
    region: ap-northeast-2
    skip_cleanup: true
    acl: private # zip 파일 접근을 private으로
    local_dir: deploy # before_deploy에서 생성한 디렉토리
    wait_until_deployed: true
    ```
