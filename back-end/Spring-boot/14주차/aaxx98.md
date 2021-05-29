# EC2 서버에 프로젝트를 배포해보자

## 1. EC2에 프로젝트 Clone

-   EC2에 git 설치
    > sudo yum install git
-   git 버전 확인
    > git --version
-   프로젝트를 저장할 디렉토리 생성
    > mkdir ~/app && mkdir ~/app/step1
-   디렉토리로 이동
    > cd ~/app/step1
-   git clone

    > git clone [레파지토리 주소]

-   ### 코드 테스트
    -   프로젝트 파일로 이동 후
        > ./gradlew test
        -   `bash: ./gradlew: Permission denied` 발생
            -   실행 권한을 추가해야함
                > chmod +x ./gradlew

## 2. 배포 스크립트 만들기

### 배포

작성한 코드를 실제 서버에 반영하는 것

-   git clone 혹은 git pull을 통해 새 버전의 프로젝트 받음
-   Gradle이나 Maven을 통해 프로젝트 테스트와 빌드
-   EC2 서버에서 해당 프로젝트 실행 및 재실행

배포할 때 마다 배포 명령어를 실행하는 것은 불편하므로 쉘 스크립트(`.sh`)로 작성

### vim으로 `deploy.sh` 작성

[vim 사용법](https://github.com/johngrib/simple_vim_guide/blob/master/md/for_beginners.md)

-   파일 생성
    > vim ~/app/step1/deploy.sh
-   `deploy.sh` 작성

    ```shell
    #!/bin/bash

    REPOSITORY=/home/ec2-user/app/step1
    PROJECT_NAME=spring

    cd $REPOSITORY/$PROJECT_NAME/

    echo "> Git Pull"

    git pull

    echo "> 프로젝트 Build 시작"

    ./gradlew build

    echo "> step1 디렉토리로 이동"

    cd $REPOSITORY

    echo "> Build 파일 복사"

    cp $REPOSITORY/$PROJECT_NAME/build/libs/*.jar $REPOSITORY/

    echo "> 현재 구동중인 애플리케이션 pid 확인"

    CURRENT_PID=$(pgrep -f ${PROJECT_NAME}*.jar)

    echo "> 현재 구동중인 애플리케이션 PID: $CURRENT_PID"

    if [ -z "$CUREENT_PID" ]; then
            echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."
    else
            echo "> kill -15 $CUREENT_PID"
            kill -15 $CURRENT_PID
            sleep 5
    fi
    echo "> 새 애플리케이션 배포"

    JAR_NAME=$(ls -tr $REPOSITORY/|grep *.jar|tail -n 1)

    echo "> JAR Name: $JAR_NAME"

    nohup java -jar $REPOSITORY/$JAR_NAME 2>&1 &

    ```

## \* Tomcat 구동 실패: 8080 포트 사용중인 프로세스 찾아서 kill하기

-   process id 찾기
    > netstat -ntlp | grep :8080
-   kill
    > kill [process id]
