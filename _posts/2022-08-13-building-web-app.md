# AWS 서비스로 웹 애플리케이션 만들기

![/assets/img/2022-08-13-building-web-app/Untitled.png](/assets/img/2022-08-13-building-web-app/Untitled.png)

웹애플리케이션 - 인터넷을 통해, 웹 브라우저에서 이용할 수 있는 응용프로그램

사용자가 주소창을 사용하여 서비스를 요청하면 서버쪽에서 요청을 처리하여 그 결과를 사용자의 웹 브라우저에 보내주는 형식으로 작동

## AWS에서 제공하는 컴퓨팅 서비스

Amazon EC2 - 가상 서버 서비스

Amazon ECS, EKS and Fargate - 컨테이너 관리 서비스

AWS Lambda - 서버리스 서비스

![/assets/img/2022-08-13-building-web-app/Untitled%201.png](/assets/img/2022-08-13-building-web-app/Untitled%201.png)

멀티티어 웹서비스 아키텍처

lift-and-shift 구조

Route53 - DNS 웹 서비스

도메인 명 등록, 관리 구매

CDN - Amazon CloudFront

짧은 지연시간, 애플리케이션을 사용자에게 안전하게 전송하는 서비스

정적 - S3 (사진, 동영상)

동적 - ELB + EC2 (로직티어) 

탄력적으로 처리하기 위해 오토스케일링

데이터티어 - RDS, DynamoDB 주로 사용

![/assets/img/2022-08-13-building-web-app/Untitled%202.png](/assets/img/2022-08-13-building-web-app/Untitled%202.png)

1. 컴퓨팅 파워의 규모를 자유자재로 변경할 수 있음 - 탄력성
2. CLI, API, 콘솔창에서 작업 수행
3. 270개 이상의 인스턴스 타입 제공
    1. 웹서비스 - 범용 m,t가 적합
4. 다양한 구매옵션
    1.  Spot 최대 90%까지 저렴. Stateless한 서비스에 사용

![/assets/img/2022-08-13-building-web-app/Untitled%203.png](/assets/img/2022-08-13-building-web-app/Untitled%203.png)

오토스케일링 + 로드밸런서 → 요청분산 + 안정성 확보

비정상 인스턴스 교체, 수평적 확장

![/assets/img/2022-08-13-building-web-app/Untitled%204.png](/assets/img/2022-08-13-building-web-app/Untitled%204.png)

On-demand - 사용한 만큼만 지불

Savings plans - 사용량 약정

Reserved Instances - 용량 약정

Spot Instance - 비딩을 통한 큰 할인률

일정한 사용량 RI + 스파이크 On-demand

Fault tolerant, stateless - Spot Instance

![/assets/img/2022-08-13-building-web-app/Untitled%205.png](/assets/img/2022-08-13-building-web-app/Untitled%205.png)

1. AWS가 관련 인프라 및 소프트웨어 관리
2. MySQL, PostgreSQL, MariaDB, MS SQL, Oracle
3. 다중 AZ, Active-standby 동기식 복제
4. 컴퓨팅 엔진을 쉽게 확장
5. 전송, 저장 암호화

![/assets/img/2022-08-13-building-web-app/Untitled%206.png](/assets/img/2022-08-13-building-web-app/Untitled%206.png)

- 대시보드를 사용하여 성능 문제 감지
- 부하를 일으키는 SQL문

![/assets/img/2022-08-13-building-web-app/Untitled%207.png](/assets/img/2022-08-13-building-web-app/Untitled%207.png)

- 객체 스토리지 서비스
- 11/9 내구성
- Static website host

![/assets/img/2022-08-13-building-web-app/Untitled%208.png](/assets/img/2022-08-13-building-web-app/Untitled%208.png)

- S3 - 간단한 정적 웹사이트 호스팅
- EC2 - 웹서버 구성, 관리 제어, 유연성, 엔터프라이즈 급 웹서비스 호스팅

## 컨테이너 기반 웹 애플리케이션

![/assets/img/2022-08-13-building-web-app/Untitled%209.png](/assets/img/2022-08-13-building-web-app/Untitled%209.png)

오케스트레이션 툴 - ECS, EKS

ECS - Docker 컨테이너 지원하는 관리 서비스

EKS - 오픈소스인 쿠버네티스, 컨트롤 플레인 설치와 운영 필요없이 관리

EC2나 Fargate 위에 컨테이너 올림

![/assets/img/2022-08-13-building-web-app/Untitled%2010.png](/assets/img/2022-08-13-building-web-app/Untitled%2010.png)

요청 - Route53, CDN - ALB - k8s pod - EKS worker node (EC2 Instance) - DynamoDB, ECR,  CloudWatch

![/assets/img/2022-08-13-building-web-app/Untitled%2011.png](/assets/img/2022-08-13-building-web-app/Untitled%2011.png)

- 쿠버네티스 툴들을 그대로 사용할 수 있음
- EKS를 온프레미스에 설치할 수 있는 EKS Anywhere

![/assets/img/2022-08-13-building-web-app/Untitled%2012.png](/assets/img/2022-08-13-building-web-app/Untitled%2012.png)

- 동시에 사용가능

![/assets/img/2022-08-13-building-web-app/Untitled%2013.png](/assets/img/2022-08-13-building-web-app/Untitled%2013.png)

- ECR에 저장되어 있는 이미지를 EKS, ECS, Lambda에 바로 배포할 수 있음

# 서버리스 웹 애플리케이션

![/assets/img/2022-08-13-building-web-app/Untitled%2014.png](/assets/img/2022-08-13-building-web-app/Untitled%2014.png)

서버리스 - 서버에 대한 고민없이 사용할 수 있다는 의미

사용자는 구축만 집중

들어오는 서비스 요청에 맞게 알아서 스케일링

![/assets/img/2022-08-13-building-web-app/Untitled%2015.png](/assets/img/2022-08-13-building-web-app/Untitled%2015.png)

- 작은 단위로 쪼개짐
- 마이크로 서비스 단위로 다수 개가 있는 아키텍처
- 정적 콘텐츠 - S3, CloudFront
- 동적 컨텐츠 - API Gateway, Lambda

![/assets/img/2022-08-13-building-web-app/Untitled%2016.png](/assets/img/2022-08-13-building-web-app/Untitled%2016.png)

- 코드 업로드하면 람다에서 코드를 실행하는데 필요한 모든것을 처리
- API GATEWAY가 트리거하면 사용
- AWS 서비스들 자동화
- 컨테이너 이미지 지원
- 1ms 실행시간 과금 세분화

![/assets/img/2022-08-13-building-web-app/Untitled%2017.png](/assets/img/2022-08-13-building-web-app/Untitled%2017.png)

- key - value NoSQL DB
- 서버리스
- 테이블 자동 확장, 축소하는 기능
- 글로벌 테이블 사용 가능
- 스트림 - 업데이트 기능 추적 가능
- Accelerator - 최대 10배의 성능 출력 가능