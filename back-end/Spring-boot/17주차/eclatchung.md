# 무중단 배포

이전에 배운 배포 -> 무중단 배포가 아님
새로운 `JAR`가 실행되기 전까지 기존 `JAR`를 종료 시켜 놓기 때문.

### 무중단 배포 
1. 엔진엑스를 이용한 무중단 배포
2. 블루- 그린 무중단 배포
3. 도커를 이용한 웹서비스 무중단 배포
4. L4 스위치 무중단 배포
	- 로드 밸런싱을 처리하는 장비

## 엔진 엑스
- 웹서버, 리버스 프록시, 캐싱, 로드 밸런싱, 미디어 스트리밍을 위한 오픈 소스 소프트웨어

- 리버스 프록시
	- 외부의 요청을 받아 백엔드 서버로 요청을 전달


- 하나의 EC2 / 리눅스 서버 -> 엔진 엑스 1대와 스프링 부트 Jar 2대

## 연동 방법
1. 엔진 엑스 설치
2. 보안 그룹 추가
3. 리다이렉션 주소 추가하기
4. 엔진엑스와 스프링 부트 연동
	-  엔진 엑스 설정


```
location / {
	proxy_pass http://localhost:8080;
	proxy_set_header X-Real-IP $remote_addr;
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	proxy_set_header Host $http_host;
}
```

## 배포스크립트 
- `stop.sh` : 기존 엔진엑스에 연결되어 있지 않지만, 실행중이던 스프링 부트 종료
- `start.sh` : 배포할 신규 버전 스프링 부트 프로젝트를 `stop.sh`로 종료한 `profile`로 실행
- `health.sh` : `start.sh`로 실행시킨 프로젝트가 정상적으로 실행됐는지 변경
- `switch.sh` : 엔진엑스가 바라보는 스프링 부트를 최신 버전으로 변경
- `profile.sh` : 앞선 4개의 스크립트 파일에서 공용으로 사용할 `profile`과 포트 체크 로직

### `stop.sh`
```shell

#!/usr/bin/env bash



ABSPATH=$(readlink -f $0)

ABSDIR=$(dirname $ABSPATH)

source ${ABSDIR}/profile.sh



IDLE_PORT=$(find_idle_port)



echo "> $IDLE_PORT 에서 구동중인 애플리케이션 pid 확인"

IDLE_PID=$(lsof -ti tcp:${IDLE_PORT})



if [ -z ${IDLE_PID} ]

then

  echo "> 현재 구동중인 애플리케이션이 없으므로 종료하지 않습니다."

else

  echo "> kill -15 $IDLE_PID"

  kill -15 ${IDLE_PID}

  sleep 5

fi

``` 

### `start.sh`
```shell

#!/usr/bin/env bash



ABSPATH=$(readlink -f $0)

ABSDIR=$(dirname $ABSPATH)

source ${ABSDIR}/profile.sh



REPOSITORY=/home/ec2-user/app/step3

PROJECT_NAME=freelec-springboot2-webservice



echo "> Build 파일 복사"

echo "> cp $REPOSITORY/zip/*.jar $REPOSITORY/"



cp $REPOSITORY/zip/*.jar $REPOSITORY/



echo "> 새 어플리케이션 배포"

JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)



echo "> JAR Name: $JAR_NAME"



echo "> $JAR_NAME 에 실행권한 추가"



chmod +x $JAR_NAME



echo "> $JAR_NAME 실행"



IDLE_PROFILE=$(find_idle_profile)



echo "> $JAR_NAME 를 profile=$IDLE_PROFILE 로 실행합니다."

nohup java -jar \

    -Dspring.config.location=classpath:/application.properties,classpath:/application-$IDLE_PROFILE.properties,/home/ec2-user/app/application-oauth.properties,/home/ec2-user/app/application-real-db.properties \

    -Dspring.profiles.active=$IDLE_PROFILE \

    $JAR_NAME > $REPOSITORY/nohup.out 2>&1 &

```
- `IDLE_PROFILE`을 통해 `properties`파일을 가져오고, `active profile`을 지정하는 것이 `deploy.sh`와 다

### `health.sh`
```shell
#!/usr/bin/env bash



ABSPATH=$(readlink -f $0)

ABSDIR=$(dirname $ABSPATH)

source ${ABSDIR}/profile.sh

source ${ABSDIR}/switch.sh



IDLE_PORT=$(find_idle_port)



echo "> Health Check Start!"

echo "> IDLE_PORT: $IDLE_PORT"

echo "> curl -s http://localhost:$IDLE_PORT/profile "

sleep 10



for RETRY_COUNT in {1..10}

do

  RESPONSE=$(curl -s http://localhost:${IDLE_PORT}/profile)

  UP_COUNT=$(echo ${RESPONSE} | grep 'real' | wc -l)



  if [ ${UP_COUNT} -ge 1 ]

  then # $up_count >= 1 ("real" 문자열이 있는지 검증)

      echo "> Health check 성공"

      switch_proxy

      break

  else

      echo "> Health check의 응답을 알 수 없거나 혹은 실행 상태가 아닙니다."

      echo "> Health check: ${RESPONSE}"

  fi



  if [ ${RETRY_COUNT} -eq 10 ]

  then

    echo "> Health check 실패. "

    echo "> 엔진엑스에 연결하지 않고 배포를 종료합니다."

    exit 1

  fi



  echo "> Health check 연결 실패. 재시도..."

  sleep 10

done

```

### `switch.sh`
```shell
#!/usr/bin/env bash



ABSPATH=$(readlink -f $0)

ABSDIR=$(dirname $ABSPATH)

source ${ABSDIR}/profile.sh



function switch_proxy() {

    IDLE_PORT=$(find_idle_port)



    echo "> 전환할 Port: $IDLE_PORT"

    echo "> Port 전환"

    echo "set \$service_url http://127.0.0.1:${IDLE_PORT};" | sudo tee /etc/nginx/conf.d/service-url.inc



    echo "> 엔진엑스 Reload"

    sudo service nginx reload

}

```


```shell
   echo "set \$service_url http://127.0.0.1:${IDLE_PORT};" | sudo tee /etc/nginx/conf.d/service-url.inc
```

위의 부분에서 포트 변경 후 `service-url.inc`에 덮어쓰여진다. 
