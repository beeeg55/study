# Section15.  CloudFront 및 AWS 글로벌 액셀러레이터

Date: June 10, 2023

AWS CloudFront

- 글로벌서비스
- CDN(contents delivery network) : 컨텐츠 전송 네트워크
- 웹사이트의 컨텐츠를 서로 다른 엣지 로케이션에 미리 캐싱하여 읽기 성능을 높이는 것
- 낮은 지연시간으로 사용자 경험 증대
- 총 216개의 전세계 엣지 로케이션을 통해 구성되어 있음
- 컨텐츠가 분산되게 되어 ddos 공격(동시에 모든 서버가 공격받는 방식)에서 보호 가능 +  shield, 방화벽 가능
- 지리적 제한 기능
    - 사용자들의 지역에 따라 여러분들의 배포 객체 접근 제한 가능
    - 접근가능 혹은 불가능 국가 목록 을 만들어 설정 가능
    - 사용 : 컨텐츠 저작권법
- 원본제공 방식
    - S3 버킷
        - cloudfront 를 통해 파일을 분산하고 캐싱할 수 있게 해줌
        - 해당 버킷에는 cloudfront 만 접근 가능 : origin access control(oac 원본 접근 제어)
        - oai(origin access identity) 를 대체
        - cloudfront를 통해 데이터를 버킷에 보내기 가능(=ingress)
    - Custom Origin(HTTP)
        - 어플리케이션 로드 밸런서, EC2 인스턴스, s3 웹사이트 등 아무 http 백엔드 가능
        - 방법 1. 엣지로케이션, ec2 모두 public으로 설정 필요함
            - ec2 보안그룹에는 모든 엣지 ip에 대한 설정이 필요
        - 방법 2. 로드밸런서 public으로 설정
            - ec2는 private 가능
            - 로드밸런서 보안그룹에는 모든 엣지 ip에 대한 설정이 필요
- 요금
    - 엣지 로케이션마다 데이터 전송 비용이 다름
    - 더 많은데이터가 오갈수록 싸짐
    - 가격을 줄이는 방법
        - 엣지 로케이션 수를 줄임
            - price class all : 모든 엣지 사용 ,최상의 성능
            - price class 200 : 가장 비싼 리전 제외함
            - price class 100 : 가장 싼 리전만 사용
- 캐시 무효화
    - cloudfront는 백엔드 오리진의 업데이트 상황을 ttl 전까지 모르므로 전체 혹은 특정경로를 캐시 새로고침할 수 있게함

cloudfront vs crr(교차 리전 복제)

- cloudfront : 전세계 대상 정적 콘텐츠에 용이
    - 전세계 엣지 사용 및 캐싱함
- crr - 일부지역대상으로 동적 컨텐츠를 낮은 지연시간으로 보내고싶을때
    - 원하는 리전 지정
    - 실시간 업데이트
    - 캐싱 사용 x
    - 읽기전용
    

AWS Global Accelerator

- 애니캐스트 Ip 개념으로 동작
    - 유니캐스트 ip : 1개의 서버가 1개의 ip를 가짐
    - 애니캐스트 ip : 모든 서버가 동일한 Ip를 가지며 클라이언트는 가장 가까운 서버로 라우팅됨
- 내부 aws 네트워크를 사용 (훨씬 지연시간이 적고 안정적)
- 사용자와 가까운 엣지 로케이션으로 트래픽을 직접 전송
- 전 세계의 유저들에게 두개의 고정ip주소를 제공
- ip, ec2, alb, nlb, public or private 와 함께 작동
- 캐시 x
- 상태 확인 - 장애조치 자동
- ip도 2개 뿐이라 화이트리스트에도 안전
- ddos 공격 막아줌(shield덕분)

Cloudfront vs aws global accelerator

- 둘다 글로벌하며 엣지 로케이션을사용
- 둘다 aws shield를 통합하여 ddos 보호 가능
- cloudfront
    - 캐시가능하여 지연속도 개선, 동적 내용 모두에 대해 성능을 향상
    - 대부분 엣지로케이션부터 내용 제공받음
- global accelerator
    - tcp, udp 상 다양한 애플리케이션 성능 향상
    - 패킷은 어플리케이션으로부터 하나 이상의 aws 리전에서 실행되는 애플리케이션으로 프록시된다.
    - 모든 요청이 애플리케이션으로 들어옴
    - 비 http 를 사용하는 경우에 적합(iot등)
    - 글로벌하게 고정 Ip가 요구되는곳에도 유용함
    - 결정적, 신속 리전 장애 조치