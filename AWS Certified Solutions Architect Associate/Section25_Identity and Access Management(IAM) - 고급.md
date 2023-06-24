# Identity and Access Management(IAM) - 고급

Date: June 24, 2023

AWS Organizations

- 글로벌 서비스로 다수의 aws 계정을 동시에 관리할 수 있게 해줌
- 조직을 생서하면 조직의 메인계정이 관리계정이 됨
- 기타 계정이나 조직에서 생성한 계정은 멤버계정이라고함
    - 멤버 계정은 한 조직에만 속함
- 모든 계정의 비용을 통합 결제할 수 있게 해줌
    - 관리계정에 하나의 지불방법을 설정해주면 됨
- 조직 내 모든계정에 대해 집계된 사용량에 기반해 할인 가능
- 계정간에 예약 인스턴스와 , saving plans 할인 가능 ( 조직 전체에 걸쳐 적용)
- 계정 생성을 자동화 할 수 있는 api 가 있어 계정 생성이 쉬움
- root organizitional unit(ou)안에 여러 ou를 마음대로 만들수 있고 그 안에 member account를 생성
- 활용 예시 - 비즈니스 따라, 환경따라, 프로젝트 따라
- 장점
    - 다수의 계정을 가지는것이 다수의 vpc를 단일 계정으로 갖는것보다 보안이 뛰어남(더 독립적)
    - 청구 목적의 태그 기준을 적용 가능
    - 한번에 모든 걔정에 대해 cloudtrail을 활성화하여 모든 로그를 하나의 중앙 s3 계정에 모을 수 있음
    - 중앙 로깅 계정으로 전송가능
    - 관리 목적의 계정 간 역할을 수립할 수 있어 관리 계정에서 모든 멤버 계정을 관리 가능
- 보안측면 : Service Control Polices(SCP) 서비스제어정책
    - 특정 ou 또는 iam 계정에 적용되는 iam 정책으로 해당 사용자와 역할 모두가 계정내에서 할 수 있는 일을 제한
    - 모든 대상에 적용되나 관리계정에는 적용되지 않음
    - 기본적으로 아무것도 허용하지 않음
    - SCP가 허용되어있더라도 계정 정책에 deny가 하나라도 포함되면 거부됨
    - 차단 목록과 허용목록 전략
        - 차단 목록 : 계정이 사용하지 못하게 할 서비스 지정
        - 허용 목록 : 명시적으로 적은 서비스만 허용하게 함

IAM 조건(CONDITION)

IAM 내부의 정책에 적용

- aws: SourceIp
    - api 호출이 생성되는 클라이언트 ip를 제한하는데 사용되는 조건
- aws:requestedregion
    - api 호출 리전을 제한
- ec2:resourcetag
    - ec2인스턴스 태그에 사용됨
- aws: principalTag
    - 사용자 태그에 사용됨
- aws:multifactorauthpresent
    - 멀티팩터 인증을 강제함

IAM FOR S3

- 버킷 목록 특정해야함
- 객체 수준 권한
    - 리소스 칸에 모든 객체를 나타내는 버킷명/* 이 들어감

리소스 정책 & aws:principalOrgID

- aws:principalOrgID 조건
    - aws조직의 멤버 계정에만 리소스 정책에 제한되도록함(조직 외부 계정은 거부)

IAM 역할 vs 리소스 기반 정책

- 교차계정
    - 리소스 기반 정책
        - 리소스에 리소스 기반 정책을 붙이기 (예시 : s3 bucket policy )
        - 리소스는 권한을 포기할 필요가 없다
        - 지원 : s3, sns topics, sqs queue 등등
    - 역할을 사용
        - 역할을 맡으면 기존의 권한을 포기하고 부여된것만 사용할 수 있음
- eventbridge 사용 시 타겟에 대한 권한 필요
    - 리소스 기반 정책 : 람다, sns, sqs, cloudwatch logs, api gateway
    - iam role : kinesis stream, systems manager run command, ecs task,,,

IAM 권한 경계

- 사용자와 역할만 지원하고 그룹은 지원하지 않음
- IAM 개체의 최대 권한을 정의하는 고급 기능
- IAM 정책을 통해 IAM 권한을 지정해주는 작업도 해야함
- **IAM 권한과 IAM 권한경계와 SCP 조건의 교집합 권한만 사용 가능함**
- AWS Organization SCP와 함께 사용 가능
- 예시
    - 관리자가 아닌 사용자에게 관리자를 위임하기 위해 권한 경계내에서 새 iam 사용자를 생성
    - 개발자 스스로 관리 권한 부여시 스스로 관리자를 못하도록
    - 계정에 SCP를 적용해서 계정의 모든 사용자를 제한하지않고 조직의 특정 사용자만 제한
- 

Amazon Cognito

- 사용자에게 웹 및 모바일 앱과 상호 작용할 수 있는 자격 증명을 부여
- 일반적으로 이 사용자들은 aws 계정 외부에 있음
- 모르는 사용자들에게 자격 증명을 부여해 사용자를 인식
- 2종류의 서비스
    - Cognito 사용자 풀
        - 앱 사용자에게 가입 기능을 제공
        - API gateway 및 어플리케이션 로드 밸런서와 원활히 통합
        - api gateway와 alb와 통합하여 로그인을 한곳에서 검증할 수 있음
    - Cognito Identity Pools(Federated Identity)
        - 앱에 등록된 사용자에게 임시 aws 자격 증명을 제공해서 일부 aws 리소스에 직접 액세스하게 해줌
        - cognito 사용자 풀과 통합
        - 임시 aws 자격 증명을 통해 aws 계정에 직접 access
        - 자격 증명에 적용되는 iam 정책은 cognito 자격증명 풀 서비스에 사전 정의되어 있음
        - user_id를 기반으로 사용자 정의하여 세분화된 제어 가능
        - 기본 iam 역할만을 정의 가능(정의되지 않은 사용자 역시 기본 iam)
        - dynamodb에 행 수준 보안 설정 가능
- 사례 : 수백명의사용자, 모바일 사용자, saml을 통한 인증

cognito user pools(cup) - user features

- 웹 및 모바일 앱을 대상으로 하는 서버리스 사용자 데이터베이스를 생성
- 사용자 이름, 이메일 비밀번호로 간단한 조합으로 간단한 로그인 절차를 정의
- 비밀번호 재서정기능
- mfa
- facebook, google 로그인고ㅓㅏ 통합
- email, 폰 인증

AWS IAM Identity Center

- 한번만 로그인하면 됨
- 아래 서비스를 지원
    - aws 조직
    - 비즈니스 클라우드 어플리케이션(salesforce, box, m365)
    - saml 2.0 enabled applications
    - ec2 windows instance
- 로그인 시 iam identity center가 identity poviders로 자격증명을 얻어 sso 로그인하게 됨
- 조직으로 사용시 사용자를 권한 set을 만들고 권한 셋을 Ou와 연결
- identity poviders
    - iam 자격 증명 센터에 내장된 id 저장소
    - 서드파티 id 공급자 : active directory, onelogin, okta 등
- 세분화된 권한 및 할당
    - 다중계정권한
        - 조직에서 여러 계정에 대한 권한 관리
        - 권한 셋을 사용해서 사용자를 그룹에 할당하는 하나 이상의 iam 정책을 정의
    - 어플리케이션 할당
        - 어떤 사용자 또는 그룹이 어떤 어플리케이션에 액세스 할 수 있는지 정의 가능
        - url, 인증서, 메타데이터 제공
    - 속성 기반 액세스 제어(Attribute-based access control : abac)
        - iam 자격 증명 센터 스토어에 저장된 사용자 속성을 기반으로 세분화된 권한을 갖게됨(태그)
        - 사용자를 비용센터에 할당, 직함, 로케일(특정 리전에만 액세스하도록)
        - 사용사례: iam 권한셋을 한번만 정의하고 이 속성을 활용하여, 사용자 또는 그룹의 aws 액세스만 수정

Microsoft active directory(ad)

- AD 도메인 서비스를 사용하는 모든 windows 서버에 사용되는 소프트웨어
- 객체의 데이터베이스이며 사용자 계정, 컴퓨터, 프런터, 파일 공유, 보안 그룹이 객체가 될 수 있음(microsoft 에서 관리되는 온프레미스의 모든 사용자는 관리 대상이 될 수 있음)
- 중앙 집중식 보안관리로 계정 생성, 권한 할당 등의 작업이 가능함
- 모든 객체는 트리로 구성되며 트리의 그룹을 forest라고 함

AWS Directory Service

- aws에 액티브 디렉토리를 생성하는 서비스
- 액티브 디렉터리를 사용해 windows를 실행할 ec2 인스턴스를 생성하고 이 windows 인스턴스는 네트워크용 도메인 컨트롤러와 결합되어 모든 로그인정보와 자격증명을 공유함
- 3가지 서비스
    - aws 관리형 microsoft ad
        - aws에 자체 액티브 디렉터리를 생성하고 로컬에서 관리할 수 있으며 멀티팩터 인증 지원
        - 독립 실행형 ad로 사용자가 있는 온프레미스 ad와 신뢰관계를 구축
        - 온프레미스 와 aws 액티브 디렉터리 ad 간에 사용자가 공유
    - ad connector
        - 디렉터리 게이트웨이
        - 프록시로 온프레미스 ad에 리다이렉트하며 mfa를 지원
        - 사용자는 온프레미스 ad 에서만 관리
    - simple ad
        - aws의 ad 호환 관리형 디렉토리
        - microsoft 디렉토리를 사용하지 않으며 온프레미스 ad와도 결합되지 않음
        - 온프레미스 ad 디렉토리가 없으나 aws 클라우드에 액티브 디렉터리가 필요한 경우 사용
        

Iam identity center와 액티브 디렉터리 통합 방법

- directory 서비스를 통해 aws 관리형 microsoft ad 연결할 경우
    - Iam identity center를 통합 연결 설정 해주면됨
- directory 서비스를 통해 온프레미스에 있는 자체 관리형 microsoft ad 연결할 경우 (사용자 관리를 어디서 하고싶냐에 따라 다름)
    - 방법 1. aws 관리형 microsoft ad를 통해 양방향 신뢰 관계를 구축
        - directory 서비스를 이용하여 관리형 microsoft ad를 생성하고 온프레미스 ad 와 양방향신뢰관계를 구축한 후 sso 사용
    - 방법 2. ad connector 활용
        - ad connector는 iam identity center와 연결하는 역할
        - ad connector로 연결 후 자체 디렉터리로 프록시

AWS Control Tower

- 모범 사례를 기반으로 안전하고 규정을 준수하는 다중 계정 aws 환경을 쉽게 설정
- aws organiztion을 사용해 계정을 자동 생성함
- 장점
    - 클릭 몇번으로 환경 설정 가능
    - 원하는것 미리 구성가능
    - 가드레일 사용해 자동으로 지속적인 정책 관리
    - 정책 위반을 감지하고 자동 교정
    - 대화형 대시보드를 통해 모든 계정의 전반적인 규정준수를 모니터링함

AWS Control Tower - 가드레일

- 사례
    - 다중 계정 설정 시에 특정 항목에 관해 한번에 모두 제한
    - 특정 유형의 규정 준수 사항을 모니터링
- control tower 환경 내의 모든 계정에 관한 거버넌스를 얻을 수 있음
- 클라우드 환경의 보안과 규정 준수 제어를 위힘
- 2가지 유형
    - 예방 가드레일
        - 계정을 무언가로부터 보호하는 것(제한적)으로 SCP를 사용해 모든 계정에 적용
        - 예시 : 모든 계정이 특정 리전 제한
    - 탐지 가드레일
        - 규정을 준수하지 않는 것을 탐지하며 awsconfg를 사용