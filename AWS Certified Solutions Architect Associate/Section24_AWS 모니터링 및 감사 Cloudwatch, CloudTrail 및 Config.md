# Section24. AWS 모니터링 및 감사 : Cloudwatch, CloudTrail 및 Config

Date: June 22, 2023

Cloudwatch 지표

- 모든 서비스에 대한 지표 제공
- 계정에서 일어나는 모든일 모니터링 가능
- 지표 : 모니터링 할 변수 (s3: 버킷 사이즈 등)
- 지표는 namespace에 속함
- dimension : 지표 지수(특정 인스턴스id, 특정 환경 등)
- 지표당 최대 10개 dimension
- 지표는 타임스탬프 필수
- 지표가 많아지면 대시보드로 한번에 볼 수 있음
- 통합 cloudwatch 에이전트를 이용해 cloudwatch 사용자 지정 지표를 만들수 있음(제공 지표대신 자체 지포 생성 가능)
    - 예시 : ec2인스턴스로부터 메모리 사용량 추출
- 외부로 스트리밍 가능
    - 원하는 대상으로 지속적으로 스트리밍하면 거의 실시간 전송되고 지연시간 짧아짐
        - amazon kinesis data firehose
        - 타사서비스 datadog, dynatrace, new relic, splunk, sumo logic
    - 필터링이 가능하여 필터랑한 지표의 서브셋만 보낼 수 있다

Cloudwatch Logs

- 로그들을 로그그룹으로 그룹화
- 로그 그룹이름 : 어플리케이션
- 로그 스트림
    - 로그그룹내 위치
    - 애플리케이션 내 인스턴스나 다양한 로그 파일명 , 컨테이너 등을 나타냄.
    - 로그만료일 설정가능
    - 유료
    - s3, lambda, elasticsearch, kinesis data streams , kinesis data firehose 등으로 보내기 가능
- sdk, cloudwatch logs agent, cloudwatch unified agent를 통해 로글 보낼 수 있음
- elastic beanstalk : 어플리케이션의 로그를 cloudwatch 로그 전송
- ecs : 컨테이너의 로그를 cloudwatch 로그 전송
- lambda : 함수 자체에서 로그 전송
- vpc flow logs: vpc 메타데이터 네트워크 트래픽 로그
- api gateway : 받은 모든 요청
- cloud trail : 필터링 해 로그를 보낼 수 있음
- route53 : 모든 dns 쿼리를 로그로 저장
- kms 암호화 가능

지표 필터와 인사이트 정의 

- 필터표현식 사용 가능
    - 로그 내 특정 ip ckwdma, ‘error’ 문구 가 포함된 모든 로그 등
- cloudwatch 경보로 연동 가능
- cloud watch logs insights 기능
    - 로그를 쿼리하고 이 쿼리를 대시보드에 추가 가능
    - 자주 쓰이는 쿼리들 저장
- cloudwatch → s3 export api name : createexporttask (실시간 아님)
    
    → 이방법대신 구독 필터를 사용하는 것이 빠름
    
- 여러 계정과 리전간 로그를 집계 가능 kinesisdatastreams에서 여러 구독필터를 구독하여 모으면 됨

cloudwatch logs for ec2

- Cloudwatch agent를 이용해서 ec2 인스턴스에서 로그와 지표를 받아 표시하는 방법
- 기본적으로는 어떤 로그도 ec2에서 옮겨지지않음
- ec2에 프로그램에서 프로그램을 설치 및 실행하여 log file을 푸쉬시키는것
- iam 권한 필요
- 온프레미스 환경에서도 셋업 가능
- 두가지 종류 agent
    - 공통: 가상서버를 위한 것
    - cloudwatch logs agent
        - 더 오래됨
        - cloudwatch logs로 로그만 보냄
    - 통합 agent(cloudwatch unified agent)
        - 프로세스나 ram 같은 추가적인 시스템 단계 지표 수집
        - 지표와 로그 두가지 모두 사용
            - ec2, 리눅스서버에 설치하면 지표 수집가능
            - 기본 ec2 모니터링보다 훨씬 세분화된 지표를 가져올 수 있음
        - cloudwatch logs로 로그만 보냄
        - ssm parameter store로 쉽게 환경 구성함

Cloudwatch alarms

- 지표에서 알림을 트리거 할때 사용
- 다양한 옵션으로 복잡한 경보 정의 가능
- 3가지 상태
    - ok - 트리거 되지 앟음
    - insufficient_Data : 데이터를 결정할 데이터가 부족
    - alarm : 임계값이 위반되어 알림이 보내지는 상태
- 기간 : 경보가 지표를 평가하는 기간
    - 짧게, 길게 모두 가능
    - 고해상도 사용자 지표 : 10,30초,60 초 배수
- 경보 주 대상
    - ec2인스턴스 동작 : 멈추거나 삭제, 복제 등
    - 오토 스케일링 : 스케일인, 아웃
    - sns : lambda 함수로 연결해 위반된 알람에 작업을 수행할 수 있게함
- cloudwatch logs 지표에 기반해 알람생성 기능 있음(set-alarm-state cli 호출)

EC2 인스턴스 복구 

- 상태 점검 → 위반됐을경우 알람을 가게 할 수 있음
    - 인스턴스 상태 = ec2 vm 체크
    - 시스템 상태 = 하드웨어 체크
- ec2 - cloudwatch - sns 구조

![Untitled](Section24%20AWS%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%86%E1%85%B5%E1%86%BE%20%E1%84%80%E1%85%A1%E1%86%B7%E1%84%89%E1%85%A1%20Cloudwatch,%20Clou%2008e61e209fe7425b9894c53c3ba3d7c2/Untitled.png)

Eventbridge

- cron job 기능(예시: 매 한시간마다 람다 호출 )
- 이벤트 규칙
    - 특정 작업을 수행하는 서비스에 반응
    - 예시 : 콘솔 iam root 사용자 로그인 이벤트에 반응
- 대상이 다양한 경우 lambda 함수를 트리거
- json으로 결과 반환
- 이벤트 버스 역할
    - 기본 이벤트 버스 : aws 서비스
    - 파트너 이벤트 버스 : 파트너사 서비스와도 이벤트 전송 가능함
    - 커스텀 이벤트 버스 : 커스텀 app(자체 생성 알람)
    - 이벤트 버스는 리소스 기반 정책을 사용하면 다른계정에서도 액세스 가능
        - 리소스기반 정책
            - 특정 이벤트 버스의 권한 관리 가능
            - 사용사례 : aws organization 중앙에 이벤트 버스를 두고 모든 이벤트를 모음
    - 이벤트 아카이빙 가능 ( 보존기간 - 무기한 or 특정기간)
    - 이를 통해 아카이브된 이벤트 재생 가능(버그 수정 시 사용 가능 - 디버깅)
- 버스의 이벤트를 분석하고 스키마를 추론하는 능력
- 스키마 레지스트리의 스키마를 사용하면 애플리케이션의 코드를 생성할 수 있고 이벤트 버스의 데이터가 어떻게 정형화되는지 미리 알 수 있음
- 스키마 버저닝 가능

Cloudwatch 서비스 insights 유형 

- container insights
    - 컨테이너로부터 지표와 로그를 수집,집계, 요약하는 서비스
    - 사용가능한 컨테이너
        - ecs
        - eks
        - ec2의 kubernetes 플랫폼에 직접 실행하는 컨테이너
        - ecs와 eks의 fargate에 배포된 컨테이너
    - 세분화된 대시보드를 만들 수 있음
    - eks, kubernetes에서 사용할 경우 컨테이너화된 버전의 cloudwatch agent를 사용해야 컨테이너를 찾을 수 있음
- lambda insights
    - lambda에서 실행되는 서버리스 어플리케이션을 위한 모니터링과 트러블 슈팅 솔루션
    - cpu시간, 메모리 디스크, 네트워크, 콜드 스타트, lambda 작업자 종료와 같은 정보를 포함한 시스템 수준의 지표를 수집, 집계, 요약함
    - lambda함수를 위해 lambda 계층에서 실행(람다 인사이트라는 대시보드를 생성해 람다 함수 성능을 측정함)
    - 서버리스 애플리케이션의 세부 모니터링 필요할 때 사용
- contributor insights
    - contributor 데이터를 표시하는 시계열 데이터를 생성하고 로그를 분석하는 서비스
        - 상위 n개 contributor
        - 전체 contributor 수 및 사용량
        - 네트워크의 상위 대화자를 찾고 시스템 성능에 영향을 미치는 대상을 파악 가능
        - vpc 로그, dns 로그 등 aws가 생성한 모든 로그에서 작동
            - 불량 호스트 식별 가능
            - 오류가 가장 많이 발생하는 url 찾을 수 있음
        - 규칙 직접 생성 or aws 생성 규칙 활용
        - 백그라운드에서는 cloudwatch logs 활용
        - 내장된 규칙이 있어 다른 aws 서비스에서 가져온 지표도 분석 가능
    - application insights
        - 모니터링 하는 어플리케이션의 잠재적인 문제와 진행 중인 문제를 분리할 수 있도록 자동화된 대시보드를 제공
        - java, .net, microsoft IIS 웹 서버 ,데이터베이스를  선택해 선택한 기술로만 어플리케이션을 실행 가능
        - ebs, rds, elb, asg, lambda, sqs, dynamodb, s3 bucket, ecs, eks, sns, api gateway 등등과 연결
        - 어플리케이션에 문제가 있는경우 자동으로 대시보드를 생성하여 서비스의 잠재적인 문제를 보여줌
            - 백그라운드에서는 sagemaker라는 머신러닝 서비스 사용됨
        - 이를 통해 어플리케이션 상태 가시성을 높여 트러블 슈팅이나 어플리케이션 보수 시간 줄어듬
        - 문제, 알림 모두 eventbridge, opscenter로 전달되어 알람받을 수 있음

cloudtrail

- aws계정의 거버넌스, 감사 및 규정 준수를 도움
- 기본적으로 활성화
- 콘솔,sdk,cli뿐아니라 aws 기타 서비스에서 발생한 aws 계정 내의 모든 이벤트 및 api 호출 기록을 받아볼 수 있다
- 모든 로그가 cloudtrail에 표시되며 cloudwatch logs or s3로 이동 가능
- 전체 또는 단일 리전에 적용되는 트레일을 생성해 모든 리전에 걸친 이벤트 기록을 s3등 한곳에 모을 수 있음
- cloudtrail을 사용하면 누군가가 aws에서 무언가를 삭제했을 때 대처할 수 있음
    - 예시 ) ec2를 누가 종료했는지 알고싶은 경우 cloudtrail을 보면됨
- 3종류의 이벤트
    - 관리 이벤트
        - aws계정의 리소스에서 수행하는 작업
        - 예시
            - 보안 설정을 누군가 구성함
            - 서브넷을 생성함
            - 로깅을 설정함
        - 트레일은 기본적으로 관리 이벤트를 구성
        - 읽기 이벤트, 쓰기 이벤트로 나눌 수 있음
    - 데이터 이벤트
        - 고볼륨 작업으로 기본적으로 로깅되지않음
        - 예시
            - s3버킷이벤트에서 get, delete, put 등
        - 읽기 이벤트, 쓰기 이벤트로 나눌 수 있음
        - 람다 함수 실행 작업
    - cloudtrail insights 이벤트
        - 모든 유형의 서비스에 걸쳐 많은 이벤트가 있고 계정에서 다수의 api가 매우 빠르게 발생하여 무엇이 이상하거나 특이한지 파악하기 어려울 때 활용
        - 비용 지불 필요
        - 이벤트 분석하여 특이사항 탐지
            - 특이사항 예시
                - 부정확한 리소스 프로비저닝
                - 서비스 한도 도달
                - aws iam 작업 버스트
                - 주기적인 유지 관리 작업 부재
        - cloudtrail의 관리 활동을 분석해서 기준선을 생성한 다음 무언가 변경, 변경시도하는 모든 쓰기 유형의 이벤트를 지속 분석하여 탐지하여 인사이트 이벤트 탐지
        - 인사이트 이벤트를 cloudtrail console, s3, eventbridge 이벤트로 넘길 수 있음
    - 이벤트 보존 기간 : 기본적으로 90일 저장되고 이후 삭제
        - 감사목적으로 더 오래 저장할땐 s3에 전송하여 기록하고 athena를 통해 분석
- eventbridge와의 통합
    - api call을 가로챌수 있게 해줌

![Untitled](Section24%20AWS%20%E1%84%86%E1%85%A9%E1%84%82%E1%85%B5%E1%84%90%E1%85%A5%E1%84%85%E1%85%B5%E1%86%BC%20%E1%84%86%E1%85%B5%E1%86%BE%20%E1%84%80%E1%85%A1%E1%86%B7%E1%84%89%E1%85%A1%20Cloudwatch,%20Clou%2008e61e209fe7425b9894c53c3ba3d7c2/Untitled%201.png)

AWS Config

- aws 내 리소스에 대한 감사와 규정준수여부를 기록해주는 서비스
- 설정된 규칙에 기반에 구성과 구성의 시간에 따른 변화를 기록하며 필요한경우 롤백 가능
- 예시
    - 보안 그룹에 제한되지 않은 ssh 접근있는지
    - 버킷에 공용 액세스가 있는지
    - 시간이 지나며 변화한 alb 구성이 있는지
- 변화가 생길때 sns알림 받을 수 있음
- 리전 별 서비스
- 데이터 중앙화를 위해 리전과 계정간 데이터 통합 가능
- 모든 리소스의 구성을 s3에 저장해 나중에 분석가능(athena 등으로)

Config rules

- aws의 config rule
- 사용자 커스텀 config rule(람다 이용)
    - 예시
        - ebs 볼륨이 gp2유형인지
        - ec2인스턴스가 t2.micro 유형인지
- 규칙들은 구성이 별할때마다 트리거, 평가(and, or)
- 미리 예방하지는 못함(차단 불가)
    - aws config 수정을 통해 예방 가능
        - ssm 자동화 문서를 사용하면 규정을 준수하지 않는 리소스를 수정 가능함(수정작업을 트리거 할 수 있음)
        - aws관리형문서나 본인 자동화 문서를 사용해 규정을 준수하지 앟는 리소스를 수정 가능
        - 람다함수로 원하는 작업 수행도 가능
        - 수정행위를 재시도도 가능(최대 5번)
- 결제 필요하며 비쌈(리전당 기록된 구성 항목 별로 3센트를 지불해야하고 리전당 config 규칙 평가별로 0.1 센트를 내야함)
- 규정 준수여부을 시간별로 볼수 있음
- 리소스 구성을 시간별로 볼수 있음
- 위 두개를 cloudtrail과 연결해 리소스에 대한 api 호출을 볼 수 있음
- notifications
    - eventbridge를 사용해 미준수 했을때 알림 보낼 수 있다
    - config에서 sns로 보내 알림을 보내기도함

Cloudwatch vs cloudtrail vs config

- cloudwatch
    - 지표, cpu, 네트워크 등의 성능 모니터링과 대시보드를 만드는데 사용
    - 이벤트와 알림을 받을 수 있음
    - 로그 집계 및 분석 도구로 사용 가능
    - 예시 : 로드밸런서의 성능을 모니터링
- cloudtrail
    - 계정 내에서 만든 api에 대한 모든 호출을 기록
    - 특정 리소스에 대한 추적 정의
    - 글로벌 서비스
    - 예시 : 누가 로드밸런서 api를 호출했는지 추적
- config
    - 구성 변경을 기록
    - 규정 준수 규칙에 따라 리소스를 평가
    - 변경과 규정 준수에 대한 타임라인 ui 제공
    - 예시 : ssl 인증서 할당 할당되있도록 규정