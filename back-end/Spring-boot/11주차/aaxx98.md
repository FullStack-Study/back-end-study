# AWS 서버 환경을 만들어보자 - EC2

외부에서 본인이 만든 서비스에 접근하려면 24시간 작동하는 서버가 필요하다.

## 24시간 작동하는 서버

-   집에 PC를 24시간 구동시키기
-   호스팅 서비스(Cafe24, 코리아호스팅) 이용
-   클라우드 서비스(AWS, AZURE, GCP) 이용

비용은 호스팅 서비스나 집 PC를 이용하는 것이 저렴하다.

만약 특정 시간에만 트래픽이 몰린다면 유동적으로 사양을 늘릴 수 있는 클라우드가 유리하다.

## 클라우드 서비스

클라우드 서비스는 인터넷(클라우드)를 통해 서버, 스토리지, 데이터베이스, 네트워크, 소프트웨어, 모니터링 등 컴퓨팅 서비스를 제공하는 것이다.

사용자 및 개발자에게 시스템 구현의 내부 사항을 추상화시켜 제공한다.

-   추상화(Abstraction)

    -   응용프로그램은 지정되지 않은 물리적 시스템에서 실행된다.
    -   데이터는 알 수 없는 위치에 저장되고, 시스템 관리는 다른 사람에게 아웃소싱되며 사용자가 액세스 가능해진다.

AWS의 EC2는 서버 장비를 대여하는 것이지만, 로그관리, 모니터링, 하드웨어 교체, 네트워크 관리 등을 기본적으로 제공한다.

### 클라우드 서비스의 형태

1. Infrastructure as a Service (IaaS)
    - 기존 물리 장비를 미들웨어와 함께 묶어둔 추상화 서비스
    - 가상머신, 스토리지, 네트워크, 운영체제 등의 IT 인프라를 대여해 주는 서비스
    - AWS의 EC2, S3 등
2. Platform as a Service(PaaS)
    - IaaS에서 한 번 더 추상화한 서비스
    - 한 번 더 추상화 했기 때문에 많은 기능이 자동화 되어있다.
    - AWS의 Beanstalk, Heroku 등
3. Software as a Service(SaaS)
    - 소프트웨어 서비스
    - 구글 드라이브, 드랍박스, 와탭 등

## AWS

### EC2 인스턴스

-   Elastic Compute Cloud
-   성능, 용량을 유동적으로 사용할 수 있는 서버 (가상머신)

### Amazon Machine Image

-   EC2 인스턴스를 시작하는 데 필요한 정보를 이미지로 만들어 둔 것
-   가상 머신에 운영체제를 설치할 수 있도록 구워 넣은 이미지
-   Amazon Linux 1 OS가 인스턴스에 설치된다.

        -   ## 설정

            -   Java 8 설치

            ```
            sudo yum install -y java-1.8.0-openjdk-devel.x86_64
            ```

            -   타임존 변경

            ```
            sudo rm /etc/localtime
            sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
            ```

            -   호스트네임 변경

            ```
            sudo vi /etc/cloud/cloud.cfg
            # 아래와 같이 내용 수정
            preserve_hostname: true


            sudo hostnamectl set-hostname [HOSTNAME]
            ```

### EIP(Elastic IP)

-   탄력적 IP, AWS의 고정 IP
