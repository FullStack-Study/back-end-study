# 자동화 2
 

## AWS
### `IAM`

- 사용자
	- AWS 서비스에만 할당 할 수 있는 권한
	- 로컬 PC, IDC 서버
- 역할
	- AWS서비스외에 사용할 수 있는 권한
	- EC2, CodeDeploy, SQS 등


### `Code Deploy`
`배포`
- `Code Commit`
	- 깃허브와 같은 코드 저장소의 역할을 한다.
	- 프라이빗 기능을 지원한다는 강점이 있지만, 현재 깃허브에서 무료로 프라이빗 지원을 하고 있어서 거의 사용되지 않는다.
- `Code Build`
	- Travis CI와 같이 `빌드용 서비스`
	- 멀티 모듈을 배포해야하는 경우 사용해볼만 하지만, 규모 있는 서비스는 젠킨스/팀시티를 이용함
- `CodeDeploy`
	- AWS의 `배포 서비스`
	- 대체제가 없음
	- 오토 스케일링 그룹 배포, 블루 그린 배포, 롤링 배포, EC2 배포 등 많은 기능 지원


![블루-그린배포](https://user-images.githubusercontent.com/45676906/114496301-9b371480-9c5a-11eb-8a83-e1cc0cf2066d.jpeg)

[AWS 블루/그린(Blue/Green) 배포 방식이란? :: Gyun’s 개발일지](https://devlog-wjdrbs96.tistory.com/300)

Traivs CO는 S3로 특정 파일만 업로드가 안되고 `디렉토리 단위로만` 업로드 할 수가 있음 ⇒ 항상 before-deploy  디렉토리 생성해야함.

**배포하는 동안은 서비스를 이용할 수가 없음 -> 무중단 배포가 아님**

[CodeDeploy EC2/온프레미스 배포에 대한 로그 데이터 보기 - AWS CodeDeploy](https://docs.aws.amazon.com/ko_kr/codedeploy/latest/userguide/deployments-view-logs.html)