# AWS RDB

## 데이터 베이스
### 쿼리 튜닝이란
- SQL문을 최적화하여 빠른 시간 내에 원하는 결과값을 얻기 위한 작업
#### 접근방법
- 부하의 감소 : 동일한 부하를 보다 효율적인 방법으로 수행
- 부하의 조정 : 부하의 정도에 따라 업무를 조정한느 접근 방법
- 부하의 병렬 수행 : 부하가 많이 걸리는 부분에 병렬 서비스를 실행 
[SQL 튜닝](https://velog.io/@gillog/SQL-%ED%8A%9C%EB%8B%9D)

## 관리형 서비스 RDS
- AWS에서 지원한는 클라우드 기반 관계형 데이터 베이스 
- 하드웨어 프로비저닝, 데이터베이스 설정, 패치 및 백업과 같은 운영 작업을 자동화 
### MariaDB
- MySQL기반의 MariaDB
- 장점
	- 도일 하드웨어 사양으로 MySQL보다 향상된 성능
	- 좀 더 활성화된 커뮤니티
	- 다양한 기능
	- 다양한 스토리지 엔진 

### 설정
- 타임존
- Character Set ⇒ `utf8mb4`로 해야지 이모티콘 저장 가능 
- Max Connection ( [DB Connection Pool에 대한 이야기 · 안녕 프로그래밍](https://www.holaxprogramming.com/2013/01/10/devops-how-to-manage-dbcp/))

### 접속
- RDB의 보안 그룹에서 EC2의 보안 그룹의 그룹ID을 사용하면 EC2 연결 / 내. IP 입력시 내 컴퓨터 연결 