---
title: "MSA - Microservice Architecture"
categories:
  - AWS
  - Cloud
---

![/_posts/img/2022-05-10-MSA-Architecture/Untitled.png](/assets/img/2022-05-10-MSA-Architecture/Untitled.png)

## 1. 모놀리식 아키텍쳐

- 소프트웨어의 모든 구성요소가 한 프로젝트에 통합되어 있는 형태
- 웹 개발을 예로 들면 웹 프로그램을 개발하기 위해 모듈별로 개발을 하고, 개발이 완료된 웹 어플리케이션을 하나의 결과물로 패키징하여 배포되는 형태
- 주로 소규모 프로젝트에서 사용된다.

**장점**

- 모든 서비스가 같은 개발환경을 사용
- 쉽게 고가용성 서버 환경을 만들 수 있다.
- End-to-End 테스트가 용이하다.

**단점**

- 한 프로젝트의 덩치가 너무 커져서 어플리케이션 구동시간이 늘어나고 빌드,배포 시간도 길어짐
- 조그마한 수정사항이 있어도 전체를 다시 빌드하고 배포를 해야 함
- 부분 장애가 전체 서비스의 장애로 확대될 수 있음
- 서비스 별로 알맞는 기술, 언어, 프레임워크를 선택하기가 까다로움
- 애플리케이션의 한 프로세스에 대한 수요가 급증하면 해당 아키텍처 전체를 확장해야 함

## 2. 마이크로서비스 아키텍쳐

- 작고, 독립적으로 배포 가능한 각각의 기능을 수행하는 서비스로 구성된 프레임워크
- 각 애플리케이션 프로세스가 서비스로 실행됨
- 사용량 단위로 과금되는 클라우드 환경에서는 모놀리식보다 마이크로서비스 아키텍쳐가 적합

**장점**

- 각각의 서비스는 모듈화가 되어있으며 이러한 MSA는 각각 개별의 서비스 개발을 빠르게 하며,
유지보수도 쉽게할 수 있도록 함
- 팀 단위로 적절한 수준에서 기술 스택을 다르게 가져갈 수 있다. 회사가 java의 spring 기반이라도 MSA를 적용하면 node.js로 블록체인 개발 모듈을 연동함에 무리가 없음
- 마이크로서비스는 서비스별로 독립적 배포가 가능하다. 따라서 지속적인 배포 CD도 모놀로식에 비해서 가볍게 할 수 있음
- 마이크로서비스는 각각 서비스의 부하에 따라 개별적으로 scale-out이 가능하다. 메모리, CPU적으로 상당부분 이득이 됨

**단점**

- 관리가 힘들다. 작은 여러 서비스들이 분산되어있기 때문에 모니터링이 힘듦
- 서로를 호출하여 전체 서비스가 이루어지기 때문에 무조건 다른 서비스를 호출하는 코드가 추가되는데 이부분이 모놀리식 아키텍쳐의 개발보다 조금 까다로움
- 통신관련 오류가 잦을 수 있음. 마이크로 서비스들 끼리 계속 서로 통신을 하다보니 모놀리식 아키텍쳐에 비해 통신관련 오류가 잦음
- 테스트가 불편함. 예로 End-to-End 테스트를 위해 UI, Gateway 등등 여러개의 마이크로 서비스를 구동시켜야 함
- 실제 운영환경에 대해서 배포하는 것이 쉽지 않음. 다른 서비스들과의 연계가 정상적으로 이루어지고 있는지도 확인해야 함

**구조**

![MSA%20-%20Microservice%20Architecture%2094937a875b144820b919d53b3bfbae7c/Untitled%201.png](/assets/img/2022-05-10-MSA-Architecture/Untitled%201.png)

- API를 사용하여 타 서비스와 통신
- API를 사용하면 구현 방식을 알지 못해도 제품 또는 서비스가 서로 커뮤니케이션할 수 있으며 애플리케이션 개발을 간소화하여 시간과 비용을 절약할 수 있음

**AWS에서의 통신 방식**

1. DNS 기반 서비스 검색
    1. ECS에는 컨테이너형 서비스를 쉽게 검색하고 서로 연결할 수 있는 통합 서비스 검색이 포함
    2. ECS - Route53 Auto Naming API를 사용하여 서비스 이름의 레지스트리를 생성하고 관리
    3. 정상 서비스 엔드포인트만 반환
    4. Kubernetes 통합 서비스 검색
    5. AWS Cloud Map - IP, URL, ARN의 서비스 레지스트리를 제공
2. 서비스 매쉬 (Service Meshes) - AWS App Mesh
    1. 마이크로서비스 아키텍쳐에서 트래픽을 모니터링하고 제어하는 서비스 간 통신을 처리하기 위한 추가 계층
    2. 데이터 평면 - 이크로 서비스 간의 모든 네트워크 통신을 차단하는 특수 사이드카 프록시로 애플리케이션 코드와 함께 배포되는 지능형 프록시 세트로 구성
    3. 제어 평면 - 프록시와의 통신을 담당
    4. 서비스의 통신 방식을 표준화하여 엔드 투 엔드 가시성을 제공하고 애플리케이션의 고가용성을 보장
    5. 코드 변경이 필요없음

**예시**

![MSA%20-%20Microservice%20Architecture%2094937a875b144820b919d53b3bfbae7c/Untitled%202.png](/assets/img/2022-05-10-MSA-Architecture/Untitled%202.png)

- 웹서비스를 예시로 들면 애플리케이션 로직을 분리해서 여러 개의 애플리케이션으로 나눠서 서비스화하고, 각 서비스별로 톰캣을 분산 배치한 것이 핵심
- 배포 구조관점에서도 각 서비스는 독립된 서버로 타 컴포넌트와의 의존성이 없이 독립적으로 배포
- 사용자 관리 서비스와 상품관리 서비스가 분리되어 독립적인 war파일로 개발되어, 독립된 톰캣 인스턴스에 배치
- 특정 서비스가 배치된 톰캣 인스턴스만 스케일아웃이 가능하고, 앞단에 로드 밸런서를 배치하여 서비스간의 로드를 분산할 수 있음

![MSA%20-%20Microservice%20Architecture%2094937a875b144820b919d53b3bfbae7c/Untitled%203.png](/assets/img/2022-05-10-MSA-Architecture/Untitled%203.png)

- 서비스 별로 별도의 데이터 베이스를 사용
- 다른 서비스 컴포넌트에 대한 의존성이 없이 서비스를 독립적으로 개발 및 배포/운영할 수 있다는 장점을 가지고 있으나, 다른 컴포넌트의 데이타를 API 통신을 통해서만 가지고 와야 하기 때문에 성능상 문제를 야기할 수 있고, 또한 이 기종 데이타 베이스간의 트렌젝션을 묶을 수 없는 문제점을 가지고 있다.

## 3. AWS에서의 마이크로서비스 아키텍쳐

### 자주 사용되는 서비스

![MSA%20-%20Microservice%20Architecture%2094937a875b144820b919d53b3bfbae7c/Untitled%204.png](/assets/img/2022-05-10-MSA-Architecture/Untitled%204.png)

- 코드 업로드만 하면 실행과 규모 조정을 자동으로 해주는 서비스
- 서버리스 서비스라 관리할 필요가 없음

![MSA%20-%20Microservice%20Architecture%2094937a875b144820b919d53b3bfbae7c/Untitled%205.png](/assets/img/2022-05-10-MSA-Architecture/Untitled%205.png)

- ECS (컨테이너), EKS (쿠버네티스) 서비스
- 설치, 운영, 규모 조정 관리를 자동으로 해줌

![MSA%20-%20Microservice%20Architecture%2094937a875b144820b919d53b3bfbae7c/Untitled%206.png](/assets/img/2022-05-10-MSA-Architecture/Untitled%206.png)

- 서버리스 컨테이너 관리 서비스
- Amazon Elastic Container Service(ECS) 및 Amazon Elastic Kubernetes Service(EKS)에서 모두 작동
- 서버를 프로비저닝하고 관리할 필요가 없음
- 애플리케이션별로 리소스를 지정하고 관련 비용을 지불할 수 있음
- 계획적으로 애플리케이션을 격리함으로써 보안 성능을 향상시킬 수 있음

![MSA%20-%20Microservice%20Architecture%2094937a875b144820b919d53b3bfbae7c/Untitled%207.png](/assets/img/2022-05-10-MSA-Architecture/Untitled%207.png)

- 애플리케이션 수준의 네트워킹을 통해 서비스에서 여러 유형의 컴퓨팅 인프라와 원활하게 통신할 수 있도록 하는 서비스 메시
- 모든 서비스에 일관된 가시성 및 네트워크 트래픽 제어 기능을 제공하여 서비스를 쉽게 실행
- 통신을 모니터링하고 제어하는 데 필요한 로직을 모든 서비스 옆에서 실행되는 프록시로 분리
- 오류의 정확한 위치를 신속하게 찾아내고 오류가 있거나 코드 변경 사항을 배포해야 하는 경우 네트워크 트래픽을 자동으로 다시 라우팅할 수 있음

![MSA%20-%20Microservice%20Architecture%2094937a875b144820b919d53b3bfbae7c/Untitled%208.png](/assets/img/2022-05-10-MSA-Architecture/Untitled%208.png)

- AWS의 API관리 서비스
- API를 손쉽게 생성, 게시, 유지 관리, 모니터링 및 보안 유지할 수 있도록 하는 완전관리형 서비스
- RESTful API 및 WebSocket API 지원
- 컨테이너식 서버리스 워크로드 및 웹 애플리케이션을 지원

![MSA%20-%20Microservice%20Architecture%2094937a875b144820b919d53b3bfbae7c/Untitled%209.png](/assets/img/2022-05-10-MSA-Architecture/Untitled%209.png)

- 완전관리형 publisher/subscriber 메시징, SMS, 이메일 및 모바일 푸시 알림 서비스
- 게시자에서 구독자에게 메시지 전송을 제공하는 관리형 서비스
- 클라이언트는 SNS 주제를 구독하고 Amazon Kinesis Data Firehose SQS, AWS Lambda, HTTP, 이메일, 모바일 푸시 알림 및 SMS (모바일 문자 메시지) 와 같은 지원되는 엔드포인트 유형을 사용하여 게시된 메시지를 수신할 수 있음

![MSA%20-%20Microservice%20Architecture%2094937a875b144820b919d53b3bfbae7c/Untitled%2010.png](/assets/img/2022-05-10-MSA-Architecture/Untitled%2010.png)

- 메시지 대기열 서비스
- 표준 대기열은 최대 처리량, 최선 노력 순서, 최소 1회 전달을 제공
    - 실시간 사용자 요청을 집중적 백그라운드 작업과 분리
    - 작업을 여러 작업자 노드에 할당
    - 장래 처리를 위한 메시지 배치
- SQS FIFO (선입선출) 대기열은 메시지가 전송된 정확한 순서대로 정확히 한 번 처리되도록 설계
    - 사용자가 입력한 명령이 올바른 순서로 실행되도록 해야 하는 경우
    - 가격 수정을 올바른 순서로 전송하여 제품 가격을 정확히 표시해야 하는 경우
    - 계정에 등록하기 전에 학생이 과정에 등록되는 것을 방지해야 하는 경우
    

**간단한 마이크로서비스 아키텍쳐**

![MSA%20-%20Microservice%20Architecture%2094937a875b144820b919d53b3bfbae7c/Untitled%2011.png](/assets/img/2022-05-10-MSA-Architecture/Untitled%2011.png)

- 가로로 나눈 서비스
- UI – 정적 컨텐츠는 S3와 클라우드 프론트 이용
제일 가까운 엣지 로케이션이나 캐시를 통함
- 마이크로서비스
    - 애플리케이션 로드 밸런서를 앞단에 두고
    - 서비스단위 컨테이너 사용
- 데이터베이스는 서비스에 따라 다른 걸 사용

**API 적용**

![MSA%20-%20Microservice%20Architecture%2094937a875b144820b919d53b3bfbae7c/Untitled%2012.png](/assets/img/2022-05-10-MSA-Architecture/Untitled%2012.png)

- 사용자들은 모바일 장치, 웹 사이트 또는 기타 백엔드 서비스를 통해 요청을 보내고
클라우드프론트가 그 요청을 받아서 API Gateway로 넘김
- API Gateway 이용
    - 클라우드와치를 통해 서비스들 모니터링도 가능
    - IAM을 통해 권한관리도 한꺼번에 가능
    - 캐시를 이용하면 로딩시간을 줄일 수 있음

**서버리스 마이크로서비스 아키텍쳐**

![MSA%20-%20Microservice%20Architecture%2094937a875b144820b919d53b3bfbae7c/Untitled%2013.png](/assets/img/2022-05-10-MSA-Architecture/Untitled%2013.png)

- API Gateway에서 Lambda로 동기식 호출을 할 수 있어 완전한 서버리스 애플리케이션을 만들 수 있음
- 확장성과 고가용성을 위해 따로 아키텍팅할 필요가 없고 서버리스라 모니터링할 필요도 없음

![MSA%20-%20Microservice%20Architecture%2094937a875b144820b919d53b3bfbae7c/Untitled%2014.png](/assets/img/2022-05-10-MSA-Architecture/Untitled%2014.png)

- Docker 컨테이너는 AWS Fargate와 함께 사용되므로 기본 인프라에 신경 쓸 필요가 없음
- Amazon DynamoDB 외에도 Amazon Aurora Serverless가 사용되며, 이는 Amazon Aurora(MySQL 호환 버전)용 온디맨드 자동 확장 구성
- 이 구성에서는 애플리케이션의 필요에 따라 데이터베이스를 자동으로 시작, 종료 및 용량 스케일업 또는 스케일다운

**CQRS (Command Query Responsibility Segregation) Pattern**

명령 쿼리 책임 분리

![MSA%20-%20Microservice%20Architecture%2094937a875b144820b919d53b3bfbae7c/Untitled%2015.png](/assets/img/2022-05-10-MSA-Architecture/Untitled%2015.png)

- 데이터 저장소에 대한 읽기 및 업데이트 작업을 구분하는 패턴인 명령과 쿼리의 역할 분리
    - 커맨드 발생 시 발생된 이벤트를 비동기 방식으로 전달하여 데이터를 복제할 수 있도록 하는 패턴
- 빈번한 조회 서비스의 성능에 맞는 DB를 선택하여 조회용 DB를 별도로 구성하고 변경이 중요한 서비스는 RDB를 통해 구현
- 이벤트 소싱 - 데이터는 로그성으로 관리를 하여 트랜잭션 문제 발생 시 검증이나, 추적 시에도 활용

**비동기 메시징 및 이벤트 전달**

![MSA%20-%20Microservice%20Architecture%2094937a875b144820b919d53b3bfbae7c/Untitled%2016.png](/assets/img/2022-05-10-MSA-Architecture/Untitled%2016.png)

- 메세지를 교환해서 통신하는 아키텍쳐
- 서비스 검색 기능이 필요하지 않으며 서비스가 느슨하게 결합되어 있다는 것이 장점
- SQS 대기열을 SNS 항목에 가입하면 해당 주제에 대한 메시지를 게시할 수 있고 아마존 SNS는 가입된 SQS 대기열에 메시지를 보냄
- 메시지에는 JSON 형식의 메타데이터 정보와 함께 항목에 게시된 제목 및 메시지가 포함
- Amazon SQS는 사용자 지정 API를 표시, AWS로 마이그레이션하려는 기존 애플리케이션이 있는 경우 코드를 변경해야 함
- 기존 소프트웨어가 JMS, NMS, AMQP, STOMP, MQTT, WebSocket 등 개방형 표준 API와 프로토콜을 사용하는 경우 Amazon MQ를 사용하면 코드를 변경하지 않아도 됨


참고문서: [Implementing Microservices on AWS](https://d1.awsstatic.com/whitepapers/microservices-on-aws.pdf) 백서