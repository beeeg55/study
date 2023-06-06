# Section10. Route53

Date: June 6, 2023

DNS 

- domain registrar: 도메인 이름을 등록하는곳 - ex. route53, godaddy 등
- dns record : a,aaaa,cname  등
- zone file : 모든 dns 레코드를 포함
- name server : dns 쿼리를 실제로 해결하는 서버
- top level domain : com, us 등
- second level domain : amazon.com, google.com 등

DNS 동작

1. local dns server
2. root dns server
3. tld dns server
4. sld dns server

route 53 

- 고가용성, 확장성을 갖춘, 완전히 관리되며 권한있는 dns
    - 관리 : 사용자가 dns 레코드를 업데이트 가능함
- Domain Registrar 역할
- 리소스 상태 체크
- 100% SLA 가용성을 제공하는 유일한 aws 서비스
- 레코드를 통해 특정 도메인으로 라우팅하는 방법을 정의
- 레코드 포함값
    - 도메인 네임
    - 레코드 타입
    - value
    - routing policy : route53 쿼리 응답 방식
    - ttl  : 레코드 캐시 시간 설정
- dns 레코트 종류 : a,aaaa,cname, ns
    - A : 호스트 네임과 ipv4 를 매핑
    - AAAA : 호스트 네임과 ipv6 를 매핑
    - CNAME : 호스트네임을 다른 호스트이름과 매핑
        - dns namespace or zone paex의 상위노드에 대한 cnames는 생성 불가
    - NS : 호스팅 존의 네임 서버 ( 서버의 dns이름 또는 Ip )
        - 트래픽이 도메인으로 라우팅 되는 방식을 제어

Hosted Zones

- 레코드의 컨테이너
- 도메인과 서브도메인으로 가는 트래픽의 라우팅 방식 정의
    - 퍼블릭 호스팅 존 : 공개
    - 프리이빗 포스팅 존 : 해당 VPC에서만 접근 가능
- 호스트존마다 월에 50센트 지불

TTL(time to leave)

- ttl은 클라이언트에게 응답결과를 N초동안 캐시하도록 요청
- ttl 시간을 높게 설정한 경우
    - route53 트래픽 낮아짐
    - 오래된 레코드로 접근하는 상황 발생 가능
- ttl 시간을 짧게 설정한 경우
    - route53 트래픽 높아져 비용 많이 발생
    - 레코드 변경 빠름
- alias 레코드를 제외하고는 필수로 ttl 보유해야함

CNAME VS ALIAS

- CNAME
    - HOSTNAME TO HOSTNAME
    - 루트도메인은 안됨!
    - dns namespace 상위노드로 사용불가
- ALIAS
    - HOSTNAME TO AWS RESOURCE
    - 루트도 적용 가능
    - 무료
    - 자체 상태확인 가능
    - DNS 확장 기능
    - dns namespace 상위노드로 사용가능
    - TTL 사용 불가

ALIAS RECORD

- AWS 리소스를 위한 별칭 레코드의 타입은 항상 a나 aaaa다
- elb, cloudfront, api gateway, beanstalk, s3, vpc interface endpoints, global accelerator, route53 record 등 타겟
- EC2 DNS이름에 대해서는 설정불가

라우팅 정책 

- Route 53 가 dns 쿼리에 응답하는것을 도움
- route53 지원 라우팅정책
    - 단순
        - 트래픽을 단일 리소스로 보내는 방식
        - 다중 값 받은경우 클라이언트가 하나 아무거나 고름
        - 별칭레코드 사용시 하나의 AWS 리소스만을 대상으로 지정할 수 있음
        - 상태확인 불가
    - 가중치 기반
        - 가중치를 활용해 요청의 일부 비율을 특정 리소스로 보낼수있도록 제어가능
        - 가중치는 상대적인것 ( 합이 100이 아니어도됨)
        - 상태 확인 가능
        - 레코드를 가중치 0설정하면 트래픽 중단 가능. 이때 다른 레코드는 동일한 가중치를 갖게됨
        - 사례 : 서로 다른 리전에 걸쳐 로드밸런싱 할때, 적은양 트래픽으로 새 어플리케이션 테스트할때
    - 장애 조치
        - 메인인스턴스와 상태확인 필수연결
        - 상태확인인 비정상인경우 보조 인스턴스로 장애조치
        - 기본과 보조 하나씩만!
    - 지연시간 기반
        - 지연시간이 가장 짧은, 가장 가까운 리소스로 리다이렉팅 하는 정책
        - 기준 : 가장 가까운 리전에 걸리는 시간
        - 사례 : 지연시간이 중요한 어플리케이션
        - 상태 확인 가능
    - 지리적
        - 사용자의 실제위치 기반으로 가장 가까운 곳으로 라우팅
        - 기본레코드 생성 필수(위치에 해당하는곳 없을때 필요)
        - 사례 : 콘텐츠 분산 제한, 로드밸런싱 등을 실행하는 웹사이트 현지화
        - 상태확인 가능
    - 다중 값 응답
        - 다중 리소스로 라우팅할때 사용. 멀티값 및 리소스 반환
        - 상태확인 가능하며 이를 사용하여 정상만 리턴
        - 클라이언트 측면의 로드밸런서 느낌
        - 
    - 지리 근접
        - 리소스의 지리적위치를 기반으로 편향값을 설정하여 트래픽을 리소스로 라우팅하도록함
        - 다른 리전으로 더 많은 트래픽을 이동시키려는 목적
        - 편향값을 지정( 더많은 트래픽원하면 양수, 더적게는 음수)
        - 지역은 aws자원이면 리전, 아니면 위도경도로 설정
        - 편향 활용을 위해 traffic flow 사용
        - 사례 : 특정리전에 더많은 트래픽을 보내고싶은 경우
    - 라우팅 정책
    

상태확인(health check)

- 주로 공용리소스에 대한 상태확인
- DNS의 장애 조치를 자동화하기 위한 작업
    - 공용 엔드 포인트를 모니터링 (어플리케이션, 서버 등)
        - 15개 global health checkers가  endpoint의 헬스체크를 함. 18%이상의 엔드포인트가 정상이라 판단하면 정상
        - 임계값, 시간 설정
        - HTTP, HTTPS,TCP 등 지원
        - 로드밸런서로부터 2xx,3xx 코드 받아야 통과
        - 텍스트 기반 응답인경우 응답의 최상단 5120 바이트를 확인
        - 상태확인을 위한 방화벽 확인 필요
    - calculated health checks
        - 여러개의 상태확인 결과를 하나로 합쳐주는기능
    - cloudwatch 경보의 상태 모니터링 - 개인 리소스를 위함
        - private endpoint (private vpc나 온프레미스)를 사용하는경우 접근이 불가하기 때문에 cloudwatch로 지표를 만들어 할당
- cloudwatch 지표에서도 확인 가능

Domain Registar VS DNS Service

- Domain Registar
    - 원하는 도메인 이름 구매
    - 매년 비용 지불
    - 일반적으로 구매하면 DNS레코드 관리를 위한 DNS 서비스 제공
    - 예> godaddy, amazon registrar
        - 제 3의 registrar와 route53 dnsservice 조합하여 사용 가능
            - 사용방법
            1. route53에서 공용 호스팅 영역 생성
            2. registrar에서 route53로 name server 업데이트