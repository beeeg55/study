# Section4 : IAM 및 AWS CLI

Date: May 30, 2023

**IAM(Identity and Access Management)**

- 사용자를 만들고, 생성 그룹에 배치
- 그룹에는 **사용자만**을 배치한다. 그룹안에 그룹이 배치되지는 않는다
- 하나의 사용자가 여러개의 그룹에 속할수는 있다
- 글로벌 서비스다 ( 리전 제한 x)

권한을 부여하기 위해 화면이나 JSON을 통해 설정

AWS는 최소한의 권한 법칙을 가지고있다

개인에게 권한부여 정책은 인라인 정책이라고 함

**iam 정책 구조 (시험출제 - 정확히 아래 4개 이해하고 있어야함 )**

- **effect : allow, deny 등 허용거부 표기**
- **principal: 정책이 적용될 사용자, 계정, 역할 등**
- **action : 허용 및 거부되는 API 호출 목록**
- **resource : 정책 적용 될 리소스(버킷 등)**

**보안 방어 메커니즘 - 계정보호**

1. 비밀번호 정책정의 - 최소길이, 특정유형문자, 비밀번호 변경 허용/금지, 일정기간 내 변경 등 
2. **다요소 인증 (MFA) - 필수를 권장**
    - MFA : 비밀번호 + 보안장치 ( 비밀번호만으로는 계정 침해가 불가함)
    - MFA 종류
        - Virtual MFA Device (Google Authenticator - phone only, Authy - multi device )
        - U2F 보안키 - 물리적 장치 yubikey
        - Hardware key FOB mfa device (GEMALTO 회사 제공)
        - Hardware key FOB mfa device FOR AWS GovCloud(SUREPASSID회사 제공)
        

AWS 에 접근하는 방법

1. 콘솔
2. CLI - 액세스 키로 보호 
    1. 터미널에서 작업하는 개념
    2. 스크립트를 통해 일부 자동화
3. SDK - 액세스 키로 보호
    1. 애플리케이션에 access를 심어둔 개념
    2. 언어별로 각각 존재

access 키는 사용자가 직접관리하며 콘솔로 발급받는다

(accesskey id 는 username, secret access key는 비밀번호와 대응되는 개념이다)

**aws cloudshell** 

- AWS 클라우드에서 무료로 사용 가능한 터미널 같은 개념
- 전체 저장소가 있음
- 화면 구성 가능
- 파일 업로드, 다운로드 가능
- 특정 리전에서만 가능

**IAM Role(역할)**

- AWS 서비스 역시 사용자와 마찬가지로 권한이 필요함 → 이때 사용하는것! (사용자에게 주는 권한이 아닌 서비스에 주는것임)
- EC2 인스턴스 롤, 롤다함수 롤, Cloudformation 롤 등

**IAM 보안도구**

- IAM 자격 증명 보고서(전체) - 계정수준에서 가능
- IAM 액세스 관리자(특정사용자) - 사용자에게 부여된 서비스 권한과, 해당서비스에 마지막 접근한 시간이 보임 → 해당사용자가 어느 권한을 안쓰는지 찾고 권한을 줄여 최소권한의 원칙을 지킬 수 있음

**IAM 가이드라인 & 좋은 예시**

- 루트계정은 AWS 계정 설정할때만 사용
- 사용자는 그룹 권장, 권한을 할당하여 관리
- 강력한 비밀번호 원칙, 다요소인증(MFA)사용
- AWS SERVICE에 권한을 줄때마다 Role 만들기
- AWS CLI OR SDK(프로그래밍 방식) 사용할 경우 액세스 키를 만들어야하며 기밀유지
- 계정의 권한 감시할때는 보안도구 2개 사용