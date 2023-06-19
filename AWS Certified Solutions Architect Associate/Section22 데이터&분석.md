# Section22. 데이터&분석

Date: June 18, 2023

Amazon Athena

- S3 버킷에 저장된 데이터 분석에 사용하는 서버리스 쿼리 서비스
- 데이터를 분석하려면 표준 sql언어로 파일을 쿼리해야함(?)
- sql언어를 사용하는 presto 엔진에 빌드
- s3 버킷의 데이터를 이동시키지 않고 바로 분석
- 지원 형식 : csv, json, orc, avro, parquet 등
- 가격 책정 : 스캔된 데이터의 tb 당 고정 가격 지불
- 서버리스여서 데이터베이스 프로비저닝 할 필요없음
- amazon quicksight라는 도구와 함께 사용하는 일이 많음
    - 보고서와 대시보드 생성
    - s3 버킷 - athena - quicksight 순 배치
- 사용사례 : 임시쿼리 수행, 비즈니스 인텔리전스 분석 및 보고, aws 서비스에서 발생하는 모든 로그를 쿼리하고 분석 가능(vpc 흐름 로그, 로드 밸런스 로그 cloudtrail 추적 등)
- 서버리스 sql 엔진을 사용한 amazon s3 데이터 분석

Amazon Athena 성능 향상

- 스캔된 데이터의 tb 당 가격 지불하므로 데이터를 적게 스캔할 유형의 데이터를 사용 → 열기반 데이터 사용(필요한 열만 스캔)
    - apache parquet과 orc 추천(glue를 사용해야함)
        - glue
            - etl(적재) 작업으로 csv와 parquet간의 데이터를 변환하는데 매우 유용
- 더 적은 데이터를 스캔해야 하므로 데이터를 압축해 더 적게 검색
    - 필요 메커니즘 : bzip2, gzip, lz4, snappy, zlip, zstd
- 특정 열을 항상 쿼리한다면 데이터 세트를 분할(s3 버킷 전체경로를 슬래시로 분할)
- 큰 파일을 사용해서 오버헤드를 최소화하면 성능 향상 가능
    - s3에 작은 파일이 너무 많으면 큰 파일이 있을때보다 성능 떨어짐
    - 파일이 클수록 스캔과 검색이 쉬우므로 128mb 이상의 파일 사용

Amazon Athena - 연합쿼리

- s3 뿐 아니라 어떤 곳의 데이터도 쿼리 가능
- 관계형 데이터베이스, 비관계형 데이터베이스, 객체, 사용자 지정 데이터 원본 쿼리 가능
- AWS나 온프레미스에서 어떻게 사용할까?
    - 데이터 원본 커넥터 사용
        - 데이터 원본 커넥터는 lambda함수로 다른 서비스에서 연합 쿼리를 실행함(cloudwatch logs, dynamodb, rds 등에서 실행)

Amazon Redshift

- 데이터베이스이자 분석엔진
- postgresql 기술 기반이지만 온라인 트랜잭션 처리(oltp)에 사용되지는 않음
- 온라인분석처리를 의미하는 olap 유형의 데이터베이스로 분석과 데이터 웨어하우징에 사용함
- 데이터웨어하우징보다 성능이 10배 이상 좋고 데이터가 pb규모로 확장되므로 모든 데이터를 redshift에 로드하면 빠르게 분석 가능
- 성능향상이 가능 - 열 기반 데이터 스토리지를 사용하기 때문
    - 행 기반이 아니라 병렬 쿼리 엔진이 있음
- 프로비저닝한 인터페이스에 대한 비용만 지불
- 쿼리 수행시 sql문 사용 가능
- amazon quicksight와 같은 bi 도구나 tableau 같은 도구도 redshift와 통합 가능
- 2가지 노드 - 노드크기 미리 프로비저닝 해야함(비용절감하려면 얘약 인스턴스 사용)
    - 리더 노드 : 쿼리 계획 및 집계
    - 컴퓨팅 노드 : 쿼리 실행 및 결과를 리더노드에 보냄
- vs athena
    - redshift
        - redshift는 s3에서 모든 데이터를 redshift에 로드해야함
        - 로드 후에는 redshift의 쿼리가 더 빠르고 조인, 통합 훨씬 빠름
            - 인덱스가 있어서!
        - 쿼리가 많고 복잡, 조인, 집계 등 집중적인 데이터웨어하우스인 경우 굳
    - athena
        - s3가 임시쿼리라면 Athena가 좋은 사용사례

Redshfit snapshots & 재해복구

- 다중 az 모드가 없음(클러스터가 한개의 가용영역에 존재)
    
    → 재해복구전략 스냅샷 적용 
    
- 클러스터의 지정시간 백업으로 s3 내부에 저장되며 증분함
- 변경된 사항만 저장되므로 공간절약
- 새로운 redshift 클리스터에 스냅샷 복원 가능
- 2가지모드
    - 수동 사용
        - 직접 삭제하기전까지 유지
    - 자동화
        - 8시간마다 or 5gb마다
        - 스냅샷 보존기간 설정 가능
    - 자동이든 수동이든 다른 리전에 자동으로 복사하도록 redshift 구성하여 재해복구전략 가능

Redshift 데이터 삽입 방법 3가지

- amazon kinesis data firehose(s3 copy후에 redshift 사용)
- s3에 데이터를 로드하고 redshift에서 복사 명령
    - 인터넷 혹은 vpc(private) 통해 복사 - 향상된 vpc 라우팅
- jdbc 드라이버 인스턴스로 삽입
    - 인스턴스가 있는경우
    - 배치를 통해 삽입하는것이 효율적

Redshift Spectrum

- s3 데이터를 redshift를 사용해 분석은 하지만 로드는 하지 앟음
- 사용하려면 쿼리를 시작할 수 있는 redshift 클러스터가 있어야함
- 쿼리를 시작하면 s3에 있는 데이터에 쿼리를 실행할 수천개의 redshift spectrum 노드에 쿼리가 제출
- 클러스테에서 프로비저닝한것보다 더 많은 처리능력 활용

Amazon OpenSearch Service

- amazon elasticsearch 의 후속서비스
- 기본 키나 데이터베이스의 인덱스로만 데이터를 쿼리할 수 있는 dynamodb와 비교해보면 부분적으로 일치하는 필드를 포함해 모든 필드 검색 가능
- 검색 기능을 제공할 때 많이 사용.. 다른 데이터베이스를 보완하는데 사용
- 쿼리 분석도 가능
    - opensearch dashboard로 데이터 시각화 가능
- 사용하려면 인스턴스의 클러스터 생성필요(서버리스 아님)
- 자체 쿼리 언어가 있어 sql 지원 x
- kinesis data firehose, aws iot, cloudwatch logs, 사용자 지정 애플리케이션의 데이터 주입 가능
- 보안 - cognito, iam과 통합, kms, tls 가능
- 사용패턴
    - 1차로 opensearch에서 검색 후 그 데이터로 full data는 dynamo에서 찾기
    - ccloudwatch logs 주입
    - kindesis data streams or kinesis data firehose 로 커스텀하여 저장

EMR(Elastic MapReduce)

- **빅데이터 작업을 위한 하둡 클러스터 생성**에 사용 → 방대한 양의 데이터 분석 및 처리가 가능함
- 하둡 클러스터는 프로비저닝 필요
- 클러스터는 수백개의 ec2 인스턴스로 구성 가능
- apache spartk hbase, presto, apache flink 등 설정이 어려운데 알아서 프로비저닝과 구성을 대신 처리해줌
- 전체 클러스터 자동 확장되고 스팟인스턴스와 통합되어 가격할인해택 가능
- 노드 유형
    - 마스터 노드 : 클러스터 관리 ( 다른 노드 상태 조정) 하여 장기 실행되야함
    - 코어 노드 : 태스크 실행 및 데이터 저장하며 장기 실행
    - 태스크 노드(옵션) : 태스크 실행 . 스팟인스턴스 사용
    - 구매 옵션
        - 온디맨드 : ec2인스턴스 유형 사용하면 신뢰가능, 예측가능 워크로드, 절대 종료되지 않음
        - 예약(최소 1년) : 절약 가능. 가능한 경우 emr이 자동 예약 인스턴스 사용 - 마스터노드, 코어노드에 적합(장기실행)
        - 스팟인스턴스 : 종료될수 있어 신뢰도 떨어지나 저렴. 태스크 노드에 활용
- 배포 시 장기 실행 클러스터에서  예약 인스턴스를 사용하거나 임시 클러스터를 사용해 특정 작업 수행하고 삭제 가능
- 사용사례 - 모든 작업은 하둡, spark, hbase, presto, flink와 같은 빅데이터 관련 기술 사용
    - 데이터 처리
    - 기계 학습
    - 웹 인덱싱
    - 빅 데이터

Quicksight 

- 서버리스 머신러닝 기반 비즈니스 인텔리전스 서비스
- 대시보드를 생성하고 소유한 데이터 소스와 연결 가능
- 
- 세션 당 비용 지불
- 사용사례
    - 비즈니스 분석
    - 시각화 구현
    - 시각화된 정보를 통한 임시 분석 수행
    - 데이터를 활용한 비즈니스 인사이트 획득
- spice 엔진 : 인 메모리 연산 엔진으로 quicksight로 데이터를 직접 가져올 때 사용되며 quicksight가 다른 db에 연결되어 있을때는 작동하지 않음
- 훌륭한 사용자 수준의 기능 제공
- 엔터프라이즈에딧션 - 액세스 권한이 없는 사용자에게 일부 열 표시 제한(열수준보안 CLS)
- 대시보드, 사용자(iam과는 별도), 분석
    - 스탠다드 버전 : 사용자 정의
    - 엔터프라이즈 버전 : 그룹 사용
    - 대시보드
        - 읽기전용 스냅샷이며 분석 결과 공유 가능
        - 분석의 구성(필터, 매개변수 제어, 정렬 옵션 등)을 저장함
    - 특정 사용자 또는 그룹과 분석 결과나 대시보드를 공유 가능 (대시보드부터 게시해야함)
    - 액세스 권한이 있는 사용자는 기본 데이터도 볼 수 있음

quicksight integration

- aws service - rds, aurora, athena, redshift, s3 등 다양한 데이터 소스에 연결 가능
- data source(saas) - salesforce, jira 등
- 타사 데이터베이스 - 온프레미스 데이터베이스, teradata 등
- excel, csv, json등 import

AWS GLUE

- 추출, 변환, 로드 서비스 관리(ETL 이라고도함)
- 분석을 위해 데이터를 준비하고 변환하는데 매우 유용
- full serverless service
- 사용사례
    - redshift전 glue를 사용해 s3, rds 데이터 필터링, 열추가등 변형 가능
    - 데이터를 parquet 형식(열기반 데이터형식)으로 변환
        - athena는 열기반을 더 잘 분석하므로 athena 전 glue etl로 변환해서 s3에 저장해줌
        - lambda를 결합하면 자동화 가능
- glue data catalog
    - 데이터 세트의 카탈로그
    - glue 데이터 크롤러를 실행해 데이터베이스를 크롤링하고 모든 메타데이터를 카탈로그에 기록함
    
    → gllue 작업에 사용될 모든 데이터베이스 , 테이블 메타 데이터를 갖게됨 
    
    - athena, redshift spectrum, emr는 백그라운드에서 카탈로그를 활용함
- glue 작업 북마크 기능
    - 새 etl 작업을 실행할 때 이전 데이터의 재처리 방지
- glue elastic views 기능
    - sql을 사용해 여러 데이터 스토어의 데이터를 결합하고 복제함
    - 커스텀코드를 지원하지 않으며, glue가 원본 데이터의 변경 사항을 모니터링함
    - 서버리스 서비스
    - 여러 데이터 스토어에 분산된 구체화된 뷰인 가상 테이블 생성 가능
- glue databrew
    - 사전 빌드된 변환을 사용해 데이터를 정리하고 정규화함
- glue studio
    - glue에서 etl 작업을 생성, 실행 및 모니터링하는 gui
- glue streaming etl
    - apache spark structured streaming 위에 빌드되며 etl 작업을 배치 작업이 아니라 스트리밍 작업으로 실행 가능
        - kinesis data streaming, kafka, msk등에서 사용 가능

AWS Lake formation

- 데이터레이크 생성을 도움을 주는 완전 관리형 서비스
    - 데이터레이크 : 데이터분석을 위해 모든 데이터를 한곳으로 모아주는 중앙 집중식 저장소
- 수개월씩 걸린는 작업을 며칠만에 완료 가능
- 데이터 레이크에서의 데이터 검색, 정제, 변환 주입을 도움
- 데이터 수집, 정제나 카탈로깅, 복제 같은 복잡한 수작업을 자동화하고 기계 학습 변환 기능으로 중복 제거를 수행
- 정형 데이터와 비정형 데이터 소스를 결합 가능
- 블루 프린트 제공 : 내장된 블루프린트는 데이터를 데이터 레이크로 이전하는것을 도와주며 s3, rds, 관계형 데이터베이스, nosql 데이터베이스 등에서 지원
- Lake formation에 연결된 어플리케이션에서 행, 열 수준의 세분화된 액세스 제어 가능
- glue위에서 빌드되나 glue와 직접 상호작용하진 않음
- 사용목적 : 중앙화된 권한
    - 한곳에서 보안 관리 가능
    - 여러 iam, s3 버킷 정책등 이 복잡하게 얽인 보안을 한곳에서 관리하게 함

Kerasis Data Analytics for sql applications

2가지 종류

- sql 어플리케이션용
    - 완전 관리형 서비스. 서버프로비저닝 x
    - 오토 스케일링 가능
    - 전송된 데이터만큼 비용 지불
    - 도식
        - kinesis data streams, **kinesis data firehose**로부터 받아 s3와 조인 or sql문 적용
        - 그 후
            - kinesis data streams 으로가면 lambda, application등 스트리밍 데이터 실시간 처리
            - kinesis data firehose로 가면 s3, redshift 등으로 보냄
    - 사용사례
        - 시계열 분석
        - 실시간 대시보드
        - 실시간 지표
- apache flink용
    - apache flink를 사용하면 java, scala, sql로 애플리케이션을 작성하고 스트리밍 데이터를 처리, 분석 가능함
    - kinesis data streams, msk, kafka인 msk 같은 서비스로부터 데이터 읽기 능력 필요 시 사용
        - kinesis data firehose 데이터는 읽지 못함
    - 표준 sql보다 flink는 훨씬 강력함하여 고급쿼리능력 필요시 사용
    - 컴퓨팅 리소스 자동 프로비저닝
    - 병렬연산 오토스케일링 가능
    - 체크포인트와 스냅샷으로 구현되는 애플리케이션 백업 가능

Amazon Managed Streaming for Apache Kafka(Amazon MSK)

- 완전관리형 apache kafka용 아마존 관리형 스트리밍 서비스
    - 그때그때 클러스터를 생성, 업데이트, 삭제함
    - 클러스터 내 브로커 노드와 zookeeper 브로커 노드를 생성 및 관리
    - 고가용성을 위해 vpc의 클러스터를 최대 3개의 다중 az 전역에 배포
    - kafka 장애를 자동 복구하는 기능이 있으며 ebs 볼륨에 데이터를 저장할 수 있음
- msk severless 사용 가능
    - msk에서 kafka를 실행하지만 서버 프로비저닝 용량관리 필요없음
    - msk가 리소스를 자동으로 프로비저닝하고 컴퓨팅과 스토리지를 스케일링함
- kinesis data streams vs amazon msk
    - 공통점
        - 데이터를 스트리밍함
        - kms 암호화
    - 차이점 :
        - kinesis data streams
            - 1mb 메세지 크기제한
            - 샤드로 데이터 스트리밍
            - 용량 확장 시 샤드 분할, 축소하려면 샤드 병합
            - tls 암호화 기능
        - amazon msk
            - 1mb가 기본값이며 더 큰 메시지 보존을 위해 10mb 설정
            - 파티션을 이용한 kafka 주제 사용
            - 파티션 추가로 주제 확장만 가능(제거 불가)
            - 평문과 tls 암호화 기능
            - 1년이상 데이터 보관 가능(ebs 이용)
- amazon msk consumers
    - msk에 데이터를 생산하려면 kafka 생산자를 생성해야함
    - 예 : kinesis data analytics for apache flink, aws glue, lambda, ecs 등

빅 데이터 수집 파이프라인 

- 애플리케이션 수집 파이프라인이 완전히 서버리스이면서 aws가 100% 관리
- 실시간으로 데이터 수집
- 데이터를 변형
- 변형된 데이터를 sql을 통해 요청
- 쿼리를 사용해 생성한 보고서가 s3에 저장
- 데이터를 데이터웨어하우스에 등재해 대시보드 생성
- 사용 서비스
    - iot core: iot 장치관리를 도움
    - kinesis data streams : 실시간 데이터 수집
    - kinesis data firehose : 실시간에 가깝게 s3로 데이터를 운반(최저간격 1분)
    - lambda : firehose를 도와 데이터를 변형
    - 이후 s3가 sqs, sns, lambda 알림을 실행
    - lambda는 sqs를 구독할 수 있음(s3 람다에 연결하기도 가능)
    - athena는 서버리스 sql 서비스로 s3에 직접 저장
    - 보고 버킷은 분석된 데이터를 보관
        - 추가로 분석이 필요하면 시각화에 쓰이는 quicksight
        - redshift 보고 도구 사용