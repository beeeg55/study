# Section11. 클래식 솔루션 아키텍처 토론

Date: June 6, 2023

1. [whatsthetime.com](http://whatsthetime.com) app
- elastic ip 의 한계 : ip를 사용자가 알고있어야하며, 한계정에서 리전마다 5개까지만 eip 가능
    - Route 53 을 설정 : 도메인 레코드 및 TTL 설정
- 인스턴스 삭제/추가 시 한계 : TTL 때문에 삭제된 인스턴스를 바라보고 있음 + 다운타임 생김
    - ELB + Healthcheck 로 인스턴스를 연결 및 보안그룹 설정 변경(보안 good)하고 alias record로 route53 - elb 연결
    - 오토 스케일링 그룹 : 확장/축소 자동화 + 비용절감
- 1개의 가용영역 한계 : 재난 발생 시 모두 다운
    - 다중 az를 사용하여 가용성 높임
- 비용발생이 많이됨
    - 용량 예약

1. Myclothes.com
- 사용자가 session 유지를 할 수 없음
    - ELB Stickiness 활성화 - 해결은 되나 인스턴스가 종료될경우 정보 사라짐
    - Elasticcache : 세션 ID 저장
- 사용자 데이터 저장 필요
    - Amazon rds - 사용자 데이터 장기 저장, 다중 az기능으로 가용성good
- 사용자가 상품 읽는데 많은 트래픽이듬
    - RDS 읽기 복제본 생성
    - elastic cache + rds = 정보를 캐시로 저장
- 약한보안
    - 보안그룹을 서로 연결된 것만 허용
    
1. MyWordPress.com
- DB 확장 필요
    - aurora로 교체 - 다중az, 읽기전용복제본, 글로벌데이터베이스 사용 가능, 연산 줄일수있음
- 이미지 저장 필요
    - EBS Volume 생성
    - 여러개의 EBS 볼륨이 생기게 되면 다중 AZ에서는 문제가 생김 → 각각 ENI를 연결짓고 이를 EFS 로 통합  (비용은 좀더발생)

애플리케이션을 빠르게 인스턴스화 하기

- GOLDEN AMI : os 종속성 등 설치되고 구성된 전체 소프트웨어를 포함한 이미지이기 때문에, 향후 이 AMI로부터 EC2 인스턴스를 빠르게 부팅할 수 있음
- bootstrap : 사용자 데이터 사용
- hybrid : golden ami + bootstrap (elastic beanstalk)
- rds database - snapshot을 사용하여 시간 축소
- ebs volumes - snapshot을 사용하여 시간 축소

Beanstalk

- AWS에서 애플리케이션을 배포하는 개발자 중심의 관점
- 구성요소들을 재사용하여 배포하는 관리 서비스
- 용량 프로비저닝, 로드밸런서의 구성, 확장, 애플리케이션 상태 모니터링 , 인스턴스 구성등을 처리해줌
- 무료 서비스
- 애플리케이션 버전 : 애플리케이션 코드의 반복
- 환경 : 리소스의 모음
    - 티어 : 웹서버환경티어, 작업자 환경티어(SQS 대기열 사용)
- 내부에서 cloudformation을 활용해 애플리케이션을 배포함
- 프로세스
    1. 어플리케이션 생성
    2. 버전 업로드
    3. 환경 실행
    4. 환경 수명주기 관리
    5. 다시 2부터 반복