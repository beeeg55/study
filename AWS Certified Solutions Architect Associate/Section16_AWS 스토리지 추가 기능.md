# Section16. AWS 스토리지 추가 기능

Date: June 10, 2023

AWS Snow family

- 보안성이 뛰어난 휴대용 장치의 모음
- 사용 사례
    - 엣지에서 데이터를 수집하고 처리하기위해 사용
        - snowcone, snowball edge
    - AWS 안팎으로 데이터를 마그리에이션 할때
        - snowcone, snowball edge, snowmobil
        

**data migration with aws snow family**

- 오프라인에서 데이터 마이그레이션을 실행하는 장치
- aws가 물리적 장치를 보내주면 그 장치에 데이터를 끌어오고 다시 aws로 전송
- 기존 네트워크에 많은 양의 데이터를 전송하려면 많은 지연시간, 대역폭 발생
- 작동
    - snow ball local에서 데이터를 직접 가져옴
    - 그 장치를 aws 시설의 aws로 다시 배송하면 자체적인 인프라에 연결하여 데이터를 import
    - 장치
        - snowball edge
            - tb or pb 크기의 데이터를 aws 안팎으로 전송
            - 데이터 전송건마다 비용청구
            - 인터페이스는 블록 스토리지를 제공하거나 amozon s3 호환 객체 스토리지 제공
            - 2가지 옵션
                - snowball edge storage optimized
                    - 블록 볼륨으로 사용할 수 있도록  80tb의 하드웨어 디스크 용량 제공하거나  s3호환 객체 스토리지
                - snowball edge compute optimized
                    - 42TB 하드웨어 디스크 용량 제공
            - 데이터 센터 폐쇄을 위한 대량의 데이터 클라우드 마이그레이션이나 AWS 에 데이터를 백업함으로써 재해복구를 하는 경우
        - aws snowcone
            - 크기가 더 작음, 휴대가능
            - 환경에 강함(사막,물, 드론)  - 공간의 제약을 받는 환경에서자주사용
            - 엣지컴퓨팅, 스토리지, 데이터전송에 사용
            - 8tb 저장
            - snowball edge storage optimized
            - 비교하면 10배는 적은 용량
            - 배터리, 케이블 직접 준비
            - 오프라인으로 다시전송 혹은 aws datasync 를 사용해 데이터를 온라인 재전송
        - snow mobile
            - 용량 : 100pb
            - 보안성이 뛰어나고 온도조절 가능
            - gps 추적 및 연중무휴 비디오 감시
            - 10PB 이상의 데이터를 전송하려면 snowball보댜 좋은 방법
- snow 제품 사용방법
    1. 배송을 위해 콘솔에서 장치를 요청
    2. snowball 클라이언트나 aws opshub를 서버에 설치
    3. snowball을  서버에 연결 및 클라이언트를 사용하여 파일을 복사
    4. 배송보내기
    5. s3버킷에 해당 데이터를 불러들임
    6. snowball reset 
    

edge computing

- 데이터가 엣지 로케이션에서 생성될 때 실시간으로 처리하는 방식
- 여기서 엣지로케이션은 연결에 제한이 있거나 인터넷 엑세스가없거나 컴퓨팅을 할 수 없는곳
- 사용사례 : 엣지로케이션에서 데이터전처리, 머신러닝, 미디어스트림
- 사용 후 오프라인으로 다시 배송하면 aws로 데이터 보내기 가능
- 기종1. snowcone(smaller)
    - cpu 2개, 4gb 메모리, 유무선 액세스(wifi), usb-c or 선택적 배터리로 작동
- 기종 2. snowball edge - compute optimized
    - 52개의 vcpu를 가짐
    - 200RB RAM
    - optional gpu (영상처리, 머신러닝 하는 경우)
    - 사용 가능한 스토리지: 42tb
- 기종 3. snowball edge - storage optimized
    - 40개 cpu와 80gb ram
    - 객체 스토리지 클러스터링 가능
    - aws iot greengress라는 서비스를 통하여 모든 장치들은 내부 ec2 인스턴스나 람다함수 실행 가능
    - 오래 사용해야하는 경우에는 장기 배포 옵션 선택 가능 - 1~3년 빌리면 가격 할인

aws opshub

- 기존 cli에서 복잡하게 명령어 쳐야하는 불편함을 없애기 위함
- 컴퓨터나 노트북에 설치하는 소프트웨어(클라우드 사용없이 컴퓨터에 다운로드하여 사용)
- 연결 후 그래픽 인터페이스 통해 snow 장치에 연결해서 구성 및 사용
    - 단일 장치와 클러스터링 장치를 잠금 해제 및 구성 가능
    - 파일전송가능, snow 장치에서 실행되는 ec2인스턴스 시작 및 관리 가능
    - 장치 메트릭 모니터링과 aws 호환 서비스 실행 가능(ec2 인스턴스, datasync, 네트워크 파일 시스템 )

snow ball into glacier

- snowball은 gracier에 데이터를 직접 끌어올 수는 없다.
- s3를 사용해서 수명주기 정책을 생성 → glacier 객체로 전환

amazon fsx 

- 완전 관리형 서비스로 타사 고성능 파일 시스템을 실행
- rds에서 mysql을 사용하는 개념과 비슷함
- 4가지 차이 알아야함
- amazon fsx for windows file server
    - 완전 관리형 windows 파일 서버 공유 드라이브
    - smb 프로토콜, windows ntfs 지원
    - microsoft active directory 통합 지원하여 사용자 보안 추가 가능, acl로 사용자 할당량을 추가해 액세스 제어가능
    - 윈도우 뿐 아니라 linux ec2인스턴스에서도 마운트 가능
    - 기존 온프레미스에 파일서버가 있는 경우 분산파일시스템(dfs) 기능을 사용하여 파일 시스템 그룹화 가능
        
        → 온프레미스의 윈도우 파일 서버와 fsx for windows file server 결합 가능
        
    - 초당 10gb, 100pb까지 확장 가능
    - 스토리지 옵션
        - ssd - 지연 시간이 짧아야 하는 워크로드 저장 가능(데이터베이스, 미디어 처리, 데이터 분석 등)
        - hdd - 더 비용이 쌈. 넓은 스펙트럼의 워크로드 저장(홈 디렉터리, cms)
    - 프라이빗 연결로 온프라미스 인프라에서 엑세스 가능
    - 고가용성(다중 az 에 대해 fsx 구성 가능 )
    - 모든 데이터는 재해 복구 목적으로 s3에 매일 백업
- amazon fsx for lustre
    - lustre : 분산파일 시스템. 대행 연산에 사용
    - lunux + cluster 합친단어로 머신러닝과 hpc, 고성능 연산에 사용
        - 동영상 처리, 금용모델링, 전자설계자동화 등의 애플리케이션 등
    - 확장성이 높다
    - 초당 100gb~수백만 1iops 로 확장되고 밀리초보다 짧은 지연시간
    - 스토리지 옵션
        - ssd - 낮은 지연시간. 워크로드가 많거나 크기가 작은 무작위 파일작업이 많은경우 iops도 사용가능. 더 비쌈
        - hdd - 처리량이 많은 워크로드나 크기가 큰 시퀀스 파일 작업
    - seamless(무결절성 통합)이 가능함
        - fsx로 s3를 파일 시스템 처럼 읽어들일 수 있음
        - fsx의 연산 출력값을 다시 s3에 쓰기 가능
    - vpn 과 직접 연결을 통해 온프레미스 서버에서 사용 가능
    - fsx 파일시스템 배포 옵션
    - 스크래치 파일 시스템
        - 임시 스토리지로 복제 불가 (데이터 복제가 없어 비용은 최적화 가능)
        - 기저 서버가 오작동하면 파일 모두 유실
        - 최적화로 초과 버스트 사용 가능(영구 파일 시스템보다 성능 6배 높일 수 있음) - tib 처리량당 초당 200mb 속도
        - 단기 처리데이터에서 사용
    - 영구 파일 시스템
        - 장기 스토리지
        - 동일한 가용영역에 데이터가 복제(단일 az)
        - 거저서버가 오작동 시 몇분내 파일 대체
        - 민감한 데이터의 장기처리 및 스토리지
        - 데이터 복제본 있음
- amazon fsx for netapp ontap
    - nfs, smb, isci 프로토콜과 호환 가능
    - 온프레미스의 ontap 이나 nas에서 실행 중인 워크로드를 aws로 옮길수 잇음
    - 다양한 운영체제에서 사용가능 (호환 가능 폭이 넓음)
        - linux
        - windws
        - macos
        - vmware cloud on aws
        - amazon workspaces & appstream 2.0
        - amazon ec2, ecs and eks
    - 스토리지 자동 확장 축소 - 오토스케일링
    - 복제, 스냅샷 기능 지원
    - 적은 비용
    - 데이터 압축, 데이터 중복제거 가능
    - 지정시간 복제 기능 - 워크로드 테스트시 도움
- amazon fsx for openzfs
    - 여러버전에서의 nfs 프로토콜과 호환 가능
    - 주로 zfs에서 실행되는 워크로드를 내부적으로 aws로 옮길때 사용
    - 다양한 운영체제에서 사용가능 (호환 가능 폭이 넓음)
        - linux
        - windws
        - macos
        - vmware cloud on aws
        - amazon workspaces & appstream 2.0
        - amazon ec2, ecs and eks
    - 좋은 성능 (백만 iops까지 확장 가능하고 지연시간은 0.5 밀리초 이하)
    - 스냅샷 압축 지원
    - 적은 비용
    - 데이터 중복제거 기능은 없음
    - 지정시간 동시 복제기능 - 워크로드 테스트에 적합

hybrid cloud for storage

- 일부는 클라우드, 일부는 온프레미스에 두는것
- 하이브리드 쓰는 이유
    - 클라우드 마이그레이션 오래걸림
    - 보안
    - 규정 준수 요건
    - 전략
- s3 데이터를 온프레미스에 두려면..?
    
    → aws storage gateway가 s3와 온프레미스 인프라를 이어주는 역할
    

aws의 스토리지 클라우드 네이티브 옵션

- 블록 스토리지 (ebs, ec2 인스턴스스토어)
- 파일 시스템 - amazon efs, amazon fsx
- 객체 스토리지 - amazon s3, amazon gracier

aws storage gateway

- aws storage gateway가 s3와 온프레미스 인프라를 이어주는 역할
- 게이트웨이는 회사 데이터센터 내에서 운영
    - 온프레미스에 서버가 없는 경우 aws의 하드웨어 사용 가능(storage gateway 하드웨어 어플라이언스)
        - 물리적 설치 필요
- 사용사례
    - 재해복구목적 : 온프레미스 데이터를 클라우드에 백업
    - 백업 및 복구 목적 : 클라우드 마이그레이션
    - 스토리지 확장 (클라우드에는 콜드데이터를 둠)
    - 기본은 aws로 저장하고 지연시간을 줄이기 위해 온프레미스 storage gateway를 캐시로 사용
- 유형
    - s3 file gateway
        - s3는 glacier 제외하고 다 지원
        - 온프레미스서버와 s3 file gateway는 온프레미스에 위치하며, 이들간에는 nfs,smb 프로토콜을 사용한다
            - smb를 사용하는 경우 사용자 인증을 위해 active directory와 통합 필요
        - s3 file gateway는 s3에 https 요청으로 변환시켜 버킷으로 데이터 전송
        - 아카이브 원하는 경우 수명주기정책사용하여 glacier로 전송
        - 게이트웨이마다 버킷 접근을 위한 iam 역할 생성
    - fsx file gateway
        - fsx는 이미 온프레미스 접근이 가능하지만 사용하는 이유는 자주 액세스하는 데이터의 로컬 캐시 확보 가능
        - fsx는 주기적으로 s3에 백업
        - smb, ntfs, active directory 호환 가능
        - 그룹파일 공유, 온프레미스 연결할 홈 디렉터리로 사용 가능함
    - volume gateway
        - s3가 백업하는 iscsi 프로토콜 사용
        - 볼륨이 ebs 스냅샷으로 저장되며 필요에 따라 온프레미스 볼륨 복구 가능
        - 온프레미스 서버가 볼륨게이트웨이와 iscsi로 연결, gateway가 s3 bucket가 https로연결, s3가 ebs 스냅샷을 생성
        - 2가지 유형
            - 캐시 볼륨 - 액세스 시 지연시간이 낮음
            - 저장 볼륨 - 저장 데이터 세트가 온프레미스에 있으며 주기적 s3 백업이 따름
    - tape gateway
        - 물리적으로 테이프를 사용하는 백업 시스템이 있는 회사가 백업에 테이프 대신 클라우드를 활용한 백업을 하게함
        - 가상 테이프 라이브러리(vtl)은 s3와 glacier을 이용
        - 테이프 기반 프로세스의 기존 백업 데이터를 iscsi 인터페이스를 사용하여 백업
        - 벤더가 사용하는 서비스

AWS 전용 제품군(AWS Transfer Family)

- s3, efs의 데이터를 전송하고 싶을때 s3 api, efs 네트워크 파일 시스템을 사용하고 않고 ftp 사용하고 싶을때
- 지원 프로토콜
    - ftp
    - ftps(암호화)
    - sftp(암호화)
- 완전 관리형 인프라로 확장성, 안정성, 가용성 높음
- 가격 책정 - 시간당 프로비저닝된 엔드포인트 비용 +  전송된 데이터의 gb 요금
- 자격증명 관리 및 저장 가능
- 기존 인증 시스템과 통합 가능(microsoft active directory, ldap, okta, amazon cognigo, custom)
- 사용사례 : sharing files, public datasets, crm, erp 등

datasync

- 일정시간에 맞춰 데이터 동기화하고 싶을때
- 양방향 동기화 가능
- 용량 부족할 경우 aws snowcone 함께 사용(사전에 datasync agent 설치되어 있기 때문)
- 데이터를 동기화하며 대용량의 데이터를 한곳에서 다른곳으로 옮길 수 있음
    - 온프레미스나 aws의 다른 클라우드로 데이터 옮기기 가능
        - nfs, smb, hdfs 등 프로토콜에 서버를 연결해야함
        - 옮길위치인 온프레미스나 다른클라우드에 **에이전트**가 있어야함(nfs, smb 서버에 연결필요하므로)
    - aws서비스에서 다른 aws 서비스로 데이터 옮기기 가능 - 에이전트 필요없음
- 동기화 가능한 곳
    - amazon s3(glacier 포함)
    - efs
    - fsx
- 복제작업은 계속 이뤄지지 않고 일정 시간 맞춰서함
- 파일권한, 메타데이터 저장 기능 - 보안 준수 (nfs posix, smb)
- 에이전트 하나의 태스크가 초당 10gb 까지 사용 가능하며 네트워크 성능을 초과하고 싶지 않은 경우 대역 폭에 제한걸 수 있음