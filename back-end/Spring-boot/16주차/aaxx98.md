## 4. Travis CI와 AWS S3, CodeDeploy 연동

### EC2에 IAM 역할 추가

EC2가 CodeDeploy를 연동받을 수 있도록 EC2에서 사용할 역할을 생성한다.
`AmazonEC2RoleforAWSCodeDeploy`

-   역할
    -   AWS 서비스에만 할당할 수 있는 권한
    -   EC2, CodeDeploy, SQS 등
-   사용자
    -   AWS 서비스 외에 사용할 수 있는 권한
    -   로컬 PC, IDC 서버 등

### EC2에 CodeDeploy 에이전트 설치

`CodeDeploy 에이전트 설치`

> aws s3 cp s3://aws-codedeploy-ap-northeast-2/latest/install . --region ap-northeast-2

`루비 설치`

> sudo yum install ruby

`실행 권한 추가`

> chmod +x ./install

`설치`

> sudo ./install auto

`Agent 상태 검사`

> sudo service codedeploy-agent status

### CodeDeploy를 위한 권한 생성

### CodeDeploy 생성

-   AWS 배포
    -   1. Code Commit
        -   `깃허브`와 같은 코드 저장소의 역할
        -   프라이빗 기능 지원(현재 깃허브에서 무료로 프라이빗 지원을 하고 있어서 거의 사용되지 않음)
    -   2. Code Build
        -   `Travis CI`와 마찬가지로 빌드용 서비스
        -   멀티 모듈을 배포해야 하는 경우 사용해 볼만 하지만, 규모가 있는 서비스에서는 젠킨스/팀시티등을 많이 사용함
    -   3. CodeDeploy
        -   AWS 배포 서비스
        -   대체재가 없음
        -   오토스케일링 그룹 배포, 블루 그린 배포, 롤링 배포, EC2 단독 배포 등 많은 기능 지원

### Travis CI, S3, CodeDeploy 연동

-   AWS CodeDeploy 설정: `appspec.yml`

    ```yml
    version: 0.0
    os: linux
    files:
        - source: /
        destination: /home/ec2-user/app/step2/zip/
        overwrite: yes
    ```

-   `.travis.yml`

    ```yml
    ---
    deploy:
    ---
    - provider: codedeploy
      access_key_id: $AWS_ACCESS_KEY
      secret_access_key: $AWS_SECRET_KEY

      bucket: swchoi-springboot-build
      key: springboot-webservice.zip

      bundle_type: zip
      application: springboot-webservice

      deplyment_group: springboot-webservice-group
      region: ap-northeast-2
      wait-until-deployed: true
    ```

## 5. 배포 자동화 구성

jar 배포 후 실행

### `deploy.sh` 파일 추가

```sh
#!/bin/bash

REPOSITORY=/home/ec2-user/app/step2
PROJECT_NAME=springboot-webservice

echo "> Build 파일 복사"

cp $REPOSITORY/zip/*.jar $REPOSITORY/

echo "> 현재 구동중인 애플리케이션 pid 확인"

CURRENT_PID=$(pgrep -fl $PROJECT_NAME | grep jar | awk '{print $1}')

echo "현재 구동 중인 애플리케이션 pid: $CURRENT_PID"

if [ -z "$CURRENT_PID" ]; then
   echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
else
   echo "> kill -15 $CURRENT_PID"
   kill -15 $CURRENT_PID
   sleep 5
fi

echo "> 새 어플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR Name: $JAR_NAME"

echo "> $JAR_NAME 에 실행권한 추가"

chmod +x $JAR_NAME

echo "> $JAR_NAME 실행"

nohup java -jar \
  -Dspring.config.location=classpath:/application.properties,classpath:/application-real.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \
  -Dspring.profiles.active=real \
  $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
```

### `.travis.yml` 파일 수정

before_deploy 수정

```yml
before_deploy:
    - mkdir -p before-deploy
    - cp scripts/*.sh before-deploy/
    - cp appspec.yml before-deploy/
    - cp build/libs/*.jar before-deploy/
    - cd before-deploy && zip -r before-deploy *
    - cd ../ && mkdir -p deploy
    - mv before-deploy/before-deploy.zip deploy/springboot-webservice.zip
```

### `appspec.yml` 파일 수정

```yml
version: 0.0
os: linux
files:
    - source: /
      destination: /home/ec2-user/app/step2/zip/
      overwrite: yes

permissions:
    - object: /
      pattern: "**"
      owner: ec2-user
      group: ec2-user

hooks:
    ApplicationStart:
        - location: deploy.sh
          timeout: 60
          runas: ec2-user
```

## 6. 테스트, 빌드, 배포 자동화 완료

-   작업이 끝난 내용을 Master 브랜치에 푸시하면 자동으로 EC2에 배포된다.
-   그렇지만 배포하는 동안 스프링 부트 프로젝트는 종료 상태가 되어 서비스를 이용할 수 없어진다.
-   무중단 배포가 필요하다.
