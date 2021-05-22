# AWS RDS

-   직접 데이터베이스를 설치하여 다루게 된다면 모니터링, 알람, 백업, HA 구성 등을 모두 직접 해야한다.
-   AWS에서는 위 작업을 모두 지원하는 관리형 서비스 RDS(Relational Database Service)를 제공한다.
-   하드웨어 프로비저닝, 데이터베이스 설정, 패치 및 백업과 같이 잦은 운영 작업을 자동화한다.

    데이터 베이스 구축 및 EC2 서버와 연동

## Maria DB

-   상용 데이터베이스인 MSSQL, 오라클 보다 동일 사양 대비 가격이 더 저렴함
-   Amazon Aurora 교체 용이성
    -   Amazon Aurora: AWS에서 MySQL과 PostgreSQL을 클라우드 기반에 맞게 재구성한 데이터베이스로 RDS MySQL 대비 5배, RDS postgreSQL 대비 3배의 성능의 유료서비스
    -   AWS에서 직접 엔지니어링 하고 있기 때문에 발전가능성이 높음
-   MySQL 대비 장점
    -   동일 하드웨어 사양으로 MySQL보다 향상된 성능
    -   좀 더 활성화된 커뮤니티
    -   다양한 기능
    -   다양한 스토리 엔진
    -   그외 : [MYSQL 에서 MARIADB 로 마이그레이션 해야할 10가지 이유](https://xdhyix.wordpress.com/2016/03/24/mysql-%EC%97%90%EC%84%9C-mariadb-%EB%A1%9C-%EB%A7%88%EC%9D%B4%EA%B7%B8%EB%A0%88%EC%9D%B4%EC%85%98-%ED%95%B4%EC%95%BC%ED%95%A0-10%EA%B0%80%EC%A7%80-%EC%9D%B4%EC%9C%A0/)

## RDS 설정

-   타임존
-   Character Set
-   Max Connection

## 내 PC에서 RDS로 접속

1. MySQL 설치

    > sudo yum install mysql

2. RDS에 접속
    > mysql -u [마스터이름] -p -h [엔드포인트]
