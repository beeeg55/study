# Section20. 서버리스 솔루션 아키텍처 토론

Date: June 18, 2023

모바일 어플리케이션(my todo list)

1. rest api 노출(https)
    
    → api gateway 사용
    
2. serverless 아케틱처
    
    → api gateway + lambda
    
3. 사용자 원하는경우 스스로 데이터 관리(s3 폴더와 직접 상호작용 가능)
    
    → cognito 가 aws sts를 통해 임사 자격증명을 제공하여 클라이언트에게 반환. 이를통해 s3접근
    
    (aws 사용자 자격 증명을 모바일 클라이언트에 저장한다라고 하면 오답임!!)
    
4. 관리자가 서버리스 서비스로 인증
    
    → cognito
    
5. 읽기를 많이하므로 관련 성능 필요
    
    → 캐싱 계층으로 dynomodb dax 사용 or apigateway cache 사용(응답이 잘 바뀌지 않는 경우)
    
6. 데이터베이스 계층은 확장 가능하며  읽기 처리량 높여야함
    
    → dynamodb : serverless + 확장 잘됨
    

서버리스 웹사이트 (myblog.com)

1. 글로벌 스케일 아웃 가능
    
    → 전세계 cdn 서비스 cloudfront를 이용 
    
2. 읽는 빈도가 더 높음
    
    → lambda와 dynamodb 사이에 dax를 놓음 + dynamodb의 글로벌데이터베이스를 사용
    
3. 대부분 정적파일로 구성되고 일부는 동적인 rest api로 구성
    
    → rest api - apigateway+ lambda 로 구성
    
4. 캐시를 적용해서 비용을 절감하고 응답속도도 높이고 싶음
5. 처음 방문하는 사람에겐 환영 이메일 보내고싶음(서버리스)
    
    → dynamodb stream사용 + 람다를 호출하고 람다는 iam 역할을 부여받아 amazon simple email service 사용
    
6. 블로그에 업로드되는 사진은 섬네일 생성 원함(서버리스)
    
    → 사용자는 cloudfront global distribution에 올리고 s3에 전달됨(s3 transfer acceleration방식)
    
    파일 추가되면 람다 트리거되어 섬네일 생성하고 s3에 업로드(옵션으로 sqs와 sns를 호출가능)
    

마이크로서비스 아키텍처 

- 많은 서비스가 restapi로 소통
- 사용 이유 : 서비스 개발 수명 주기를 줄이고자함
    - 각 서비스가 독립적 확장
    - 개별 코드 리포지토리
- 각 서비스를 원하는대로 맘껏 설계
- 동기식 패턴 : api gateway, load balancers
    - 다른 마이크로서비스에 https 호출을 보내기 적합함
- 비동기식 패턴 : sql, kinesis, sns, lambda triggers(s3)
- 단점
    - 새로운 마이크로서비스를 생성할때마다 오버헤드가 발생
    - 서버 밀도나 사용률을 최적화하는데 어려움
    - 여러 버전의 마이크로 서비스를 동시에 가동하려면 복잡
    - 여러 서비스와 통합하려다 클라이언트 코드 요구사항이 급증하기도함
    
    → 어느정도 서버리스 패턴으로 해결 가능
    
    - api gateway, lambda 자동 확장 가능(사용률 신경 안써두됨)
    - api gateway에서 api를 쉽게 복제하고 재생산
    - api gateway를 위한 swagger 통합으로 클라이언트 sdk 생성 가능
    

소프트웨어 업데이트 오프로딩

- ec2에서 작동하는 어플리케이션이 있고 종종 소프트웨어 업데이트를 배포함
- cloudfront를 사용하면 비용 절감 가능
    - 소프트웨어 파일은 정적파일임
    - 서버리스라 확장 가능
    - asg는 많이 확장하지 않아 ec2, 네트워크, efs 비용 절감
    - 가용성 확보
    - 기존 애플리케이션의 확장성을 높이고 비용을 절감하는데 용이