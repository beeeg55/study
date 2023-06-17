# Section18. AWS의 컨테이너 : ECS, Fargate, ECR 및 EKS

Date: June 11, 2023

도커

- 앱 배포를 위한 소프트웨어 개발 플랫폼
- 컨테이너로 앱이 패키징될때 아무 운영체제에나 실행할수 있다
    
    → 어느 운영체제에든 같은 방식으로 실행 
    
    - 어느 머신이든
    - 호환성문제 x
    - 행위 특성 예측 가능
    - 배포 쉬움
    - 언어, 운영체제, 기술에 상관없이 실행 가능
- 사용 사례 : **마이크로서비스 아키텍처**
- 도커 이미지는 도커 레포지토리에 저장된다
- 도커 레퍼지토리 옵션
    - docker hub
        - public repository
        - 많은 기술에 맞는 기본 이미지 찾을 수 있음(ubuntu, mysql등)
    - private ecr(amazon elastic container registry)
        - private repository
        - public repository 옵션도 있음(amazon ecr public gallery)

도커 vs 가상머신

- 가상 머신
    - 리소스가 호스트와 공유되어 한 서버에서 다수의 컨테이너 공유 가능
    - host os 위에 hypervisor 계층이 있고 그위에 앱과 guest 운영체제(vm)
- 도커
    - 호스트 os(ec2)위에 docker damon계층이 있고 그위에 많은 컨테이너가 존재
    - 네트워킹이나 데이터 공유 가능
    - 가상머신보다는 덜 안전하지만 하나의 서버에 많은 컨테이너를 실행 가능

![스크린샷 2023-06-17 오전 11.38.53.png](Section18%20AWS%E1%84%8B%E1%85%B4%20%E1%84%8F%E1%85%A5%E1%86%AB%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%82%E1%85%A5%20ECS,%20Fargate,%20ECR%20%E1%84%86%E1%85%B5%E1%86%BE%20EK%20907f4e93b80e42c79750a2c61ed6307a/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-06-17_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%258C%25E1%2585%25A5%25E1%2586%25AB_11.38.53.png)

![스크린샷 2023-06-17 오전 11.38.31.png](Section18%20AWS%E1%84%8B%E1%85%B4%20%E1%84%8F%E1%85%A5%E1%86%AB%E1%84%90%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%82%E1%85%A5%20ECS,%20Fargate,%20ECR%20%E1%84%86%E1%85%B5%E1%86%BE%20EK%20907f4e93b80e42c79750a2c61ed6307a/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-06-17_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%258C%25E1%2585%25A5%25E1%2586%25AB_11.38.31.png)

도커 시작하는법

1. dockerfile (도커 컨테이너를 구성하는 파일)작성
2. 빌드하면 도커이미지가 됨
3. 푸시해서 도커 레퍼지토리에 저장

AWS에서 도커 관리

- Amazon ECS(Elastic Container Service)
    - 도커 관리를 위한 amazon의 전용 플랫폼
- Amazon Elastic Kubernetes Service ( Amazon EKS)
    - 쿠버네티스의 관리형 버전으로 오픈소스 프로젝트
- AWS Fargate
    - 서버리스 컨테이너 플랫폼
    - ECS 와 EKS 둘다 함께 작동 가능
- Amazon ECR
    - 컨테이너 이미지를 저장하는데 사용

Amazon ECS(Elastic Container Service)의 시작유형

- AWS에서 컨테이너를 실행하면 ECS 클러스터에 ECS 태스크를 실행
    - task : ECS에서 우리가 생성한 컨테이너를 구동하는 최소 단위
- EC2 launch type
    - ECS 클러스터에  ec2 인스턴스가 들어있는데 이경우에는 인프라를 직접 프로비저닝 하고 유지해야함
        - 클러스터: 컨테이너 인스턴스(EC2)의 논리적인 그룹화
    - ec2는 각각 ecs agent 를 실행해야함
        - ecs agent가 각각의 인스턴스를 ecs 서비스와 지정된 ecs 클러스터에 등록
    - ecs 태스크를 수행하기 시작하면 aws 컨테이너를 시작하거나 멈춤
        - 태스크를 시작하거나 멈추게되면 자동으로 ec2 위치지정
    - 도커는 미리 프로비저닝한 ec2에 존재하게된다
- Fargate Launch Type
    - AWS에 도커 컨테이너를 실행
    - 인프라를 프로비저닝 하지않음 (관리할 ec2 없음)
    - serverless
    - ecs 태스크를 정의하는 태스크 정의만 생성하면 필요한 cpu와 ram에 따라 ecs 태스크를 aws가 대신 실행
    - 새 도커컨테이너를 실행하면 그냥 실행
    - 확장하려면 태스크 수만 늘리면 됨
    - ec2 인스턴스를 관리할 필요가 없음
    - ec2 시작유형보다 관리가 쉬움
    - 선불 종량제

ECS - IAM 역할(차이알기)

- ec2 instance profile(ec2 시작 유형)
    - ecs agent 만이 해당 파일 사용
    - 이 파일을 이용해 api 호출
    - cloudwatch 로그에 api 호출하여 컨테이너 로그를 보내고 ecr로부터 도커 이미지를 가져옴
    - secrets manager 이나 ssm parameter store에서 민감데이터 참고 가능
- ECS TASK ROLE(2가지 유형 모두 해당)
    - 각각의 태스크에 특정역할 만들수있음
    - ecs의 태스크 정의에서 태스크 역할을 정의

ECS - 로드밸런서 integration

- alb 를 사용하여 ecs 태스크에 사용자 직접 연결
- nlb 를 사용하여 높은 성능, aws private link와 사용될때도 권장

ECS - DATA VOLUMES(EFS)

- 데이터 지속성을 위함
- ec2, fargate 시작 유형에서 모두 사용
- EC2 태스크에 파일 시스템 직접 마운트 가능
- 파일 시스템을 통해 다른 태스크와 연결할 수도 있음
- ECS + EFS = SERVERLESS
    - 용도 : 다중 AZ가 공유하는 컨테이너의 영구 스토리지
- s3는 ecs 태스크에서 파일시스템으로 마운트 될 수 없다

ECS Auto Scailing

- 오토 스케일링 지표
    - ECS 서비스 CPU 사용률
    - ECS 서비스 메모리 사용률(RAM )
    - ALB 지표인 타겟당 요청 수
- 오토스케일링 설정
    - 대상 추적 스케일링 - cloudwatch 특정 타겟
    - 단계 스케일링 - cloudwatch 알람
    - 예약 스케일링 - 미리 ecs 서비스 확장 설정
- EC2 시작 유형이라면 태스크 레벨에서의 ECS 서비스 확장이 EC2 인스턴스 클러스터의 확장과는 다르다
- EC2인스턴스가 백엔드에 없다면 Fargate 를 사용하는 것이 오토스케일링에 좋음
    - EC2 시작 유형의 경우 EC2를 오토스케일링 할 수 있는 방법은?
        - 방법1. ASG → CPU 사용량에 따라 EC2 확장
        - 방법2. EC2 클러스터 용량 공급자 (EC2 cluster capacity provider)- 새 태스크를 실행할 용량(ram 이나 cpu)이 부족하면 자동 확장(asg와 함께 사용) →  이방법이 good

ECS 솔루션 아케텍트

- event bridge
    - 예시
        1. s3 업로드
        2.  이벤트브릿지에서 태스크를 생성하라고  ECS에 전달
        3.  태스크수행(DB에 결과저장)
- eventbridge schedule
    - 예시
        - 이벤트 브릿지가 한시간마다 ECS에 새로운 태스크 생성
        - S3등 무언가 처리 (serverless batch)
- SQS Queue
    - 
    - 
    - QUEUE에 ECS 를 연결시키면 queue에 메세지가 많아질수록 오토스케일링하여 확장

Amazon ECR(Elastic Container Registry)

- 도커이미지를 저장하고 관리
    - 이미지 취약점 스캐닝, 버저닝 태그, 수명주기확인 등을 지원
- Docker hub 와 같은 역할
- 옵션
    - private : 계정에 한해 이미지를 비공개 저장(여러 계정 설정 가능)
    - public(ecr publie gallery)
    - ECS와 통합되어 있음
    - 이미지 s3에 저장
    - IAM에서 권한 관리

Amazon EKS(Amazon Elastic Kuberbetes Service)

- AWS에 관리형 쿠버네티스 클러스터를 실행할 수 있는 서비스
- ECS와 비슷하나 API가 다르다
    - ECS는 오픈 소스가 아닌 반면 쿠버네티스는 오픈소스이고 여러 클라우드 제공자가 사용하므로 표준화 기대
- EKS 실행모드
    - EC2 시작모드 : EC2 인스턴스에서처럼 작업자 모드를 배포할때 사용
    - Fargate 모드 : EKS 클러스터에 서버리스 컨테이너를 배포할때 사용
- 사용사례
    - 이미 쿠버네티스나 쿠버네티스 api를 사용하고 있을때 쿠버네티스 클러스터를 관리하기 위해 사용
- cloud-agnostic(모든 클라우드에서 지원)
    - 클라우드 또는 컨테이너 간 마이그레이션을 실행하는 경우 간단한 솔루션이 될수 있음
- 로드밸런서와 연결 필요
- node 유형
    - Managed Node Groups(관리형 노드 그룹)
        - aws로 노드(EC2 인스턴스)를 생성하고 관리
        - 노드는 EKS 서비스로 관리되는 ASG의 일부
        - 온디맨드인스턴스와 스팟 인스턴스 지원
    - self-managed nodes(자체 관리형 노드)
        - 사용자 지정 사항이 많고 제어 대상이 많은 경우 직접 노드를 생성하고 EKS 클러스터에 등록한 다음 ASG로 관리
        - EKS Optimized AMI를 사용하면 시간 절약 가능
        - 온디맨드인스턴스와 스팟 인스턴스 지원
    - AWS Fargage
        - 유지 관리할 필요없음( 노드 관리 안해도됨)
- EKS - Data Volumes
    - EKS 클러스터에 스토리지 클래스 매니패스트를 지정해야함
    - 컨테이너 스토리지 인터페이스(CSI)라는 규격 드라이버 활용
    - 지원
        - EBS
        - EFS(Fargate 와 동작)
        - Amazon fsx for lustre
        - amazon fsx for netapp ontap

AWS App Runner 

- 완전 관리형 서비스로 규모에 따라 웹 애플리케이션, API 배포를 도움
- 인프라, 컨테이너, 소스 코드 등을 알필요 없이 아무나 배포 가능하게해줌
- 소스코드, 컨테이너 이미지로 시작 시에 기본설정을 해둠(vcpu, 컨테이너 메모리 크기, 오토스케일링 확인 등)
- app runner 서비스가 웹 앱을 빌드하고 배포함
- 장점 : 오토스케일링 가능, 가용성 높음 , 로드밸런싱 , 암호화
- 컨테이너(애플리케이션)가 VPC에 액세스 가능
- 데이터베이스, 캐시, 메세지 대기열 서비스에 연결 가능
- 사용 사례(신속한 배포가 필요한 경우)
    - 웹앱
    - API
    - 마이크로서비스