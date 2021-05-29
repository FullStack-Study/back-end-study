# 배포하기
## 깃설치하기
```shell
sudo yum install git

//버전 확인및 설치 확인
git --version

//디렉토리 생성
mkdir ~/app && mkdir ~/app/step1

//이동
cd ~/app/step1

//클론
git clone 깃주소

//테스트 검증
./gradlew test
```
### 검증 실패시 : 권한 문제
```shell

chmod +x ./gradlew
```
[Linux/기본명령어/chmod](http://www.incodom.kr/Linux/%EA%B8%B0%EB%B3%B8%EB%AA%85%EB%A0%B9%EC%96%B4/chmod)
#### `chmod (옵션) (reference) (operator) (modes) 파일`
- 옵션
	- - R : 하위 파일과 디렉토리 모든 권한을 변경
	- -v  : 실행되고 있는 모든 파일을 나열한다.
	- - c : 권한이 변경된 파일 내용을 출력한다.
- 모드 
	- 문자열 모드
		- reference
			- u : 유저의 권한
			- g : 그룹의 권한
			- o : other의 권한
			- a : all 의 권한
		- operator
			- + : 추가한다
			- - : 제고한다
			- = : 권한을 설정한대로 변경
		- modes
			- r : 읽기권한
			- w : 쓰기 권한
			- x : execute 실행권한
## 배포 스크립트 만들기
- 포괄하는 의미
	- git clone / git pull을 통해 새 버전의 프로젝트 받음
	- gradle이나 maven을 통해 프로젝트 테스트와 빌드
	- EC2서버에서 해당 프로젝트 실행 및 재실행
### `pgrep`
- `ps` 명령어와 `grep`명령어를 합쳐서 하나의 명령어로 사용해서 원하는 정보를 편하게 출력하는 명렁어
- 옵션
	- -c : 조건에 맞는 프로세스 수를 출력한다
	- -d : PID를 구분하는 문자열을 설정
	- - f: -l옵션과 함께 사용하면 명령어의 경로를 출력
		- 프로세스 이름으로 검색 
### `ls`
- `list directory contents`
- 해당 디렉토리 내에 있는 디렉토리 및 파일을 화면에 출력함.
### `nohup`
- 리눅스에서 프로세스를 실행한 터미널의 세션 연결이 끊어지더라도 지속적으로 동작 할 수 있게 해주는 명령어


### 배포 스크립트 실행
1. 실행권한 추가 `chmod +x 파일.sh`
2. 실행하기 `파일.sh`


글을 씁시다. 그래