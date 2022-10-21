# Elastic Beanstalk애플리케이션 배포 옵션

## 종류

- All at once: 모든 인스턴스에 동시에 새 버전 배포
- Rolling: 배치 단위로 새 버전 배포
- Rolling with additional batch: 배치 단위로 새 버전 배포, +1 추가 배치
- Immutable: 새로운 인스턴스 그룹에 배포
- Traffic Splitting: 새 인스턴스 그룹을 배포하지만 트래픽 조절 가능
- Blue/Green

다음은 각 배포 방식에 대한 자세한 설명이다.

## All at once

![버전 업데이트 전의 모습](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled.png)

버전 업데이트 전의 모습

![배포가 진행되는 모습](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%201.png)

배포가 진행되는 모습

배포 진행중에 잠깐의 다운타임이 발생하게 된다.(모든 인스턴스가 업데이트 과정을 진행하기 때문)

![Untitled](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%202.png)

잠깐의 다운타임이 있지만 가장 빠른 배포 방식이다.

## Rolling

![Untitled](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%203.png)

 Rolling 배포 방식을 선택하게 되면 배치 크기에 대한 비율과 고정 선택지가 나온다. 예를 들어 비율 방식을 선택하고 비율 50%를 선택하면, 인스턴스가 4개 있을 때 2개를 먼저 버전 업데이트를 하고, 완료되면 나머지 2개를 업데이트하는 방식이다. 고정 방식에서는 인스턴스 개수를 설정하면 된다.(1개 2개..)

![버전 업데이트 전 모습](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%204.png)

버전 업데이트 전 모습

![비율이 50%일 때 Rolling 업데이트
](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%205.png)

비율이 50%일 때 Rolling 업데이트

![Untitled](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%206.png)

 버전 2 업데이트가 완료된 모습이다. 하지만 이때에 사용자가 접속하게 된다면 어떤 사용자는 버전 2의 모습을 보고, 또 다른 사용자는 버전 1의 모습을 보게 되는 현상이 나타나게 된다. 또한 롤링 업데이트의 단점은 순간적으로 2개의 인스턴스가 다운타임이 발생하기 때문에 사용자들이 사용할 수 있는 컴퓨터 자원은 절반이 된다. 이때 트래픽이 몰린다면 사용자는 서버 이용에 불편함을 느낄 수 있다.

## Rolling with Additional Batch

 롤링 업데이트의 경우 순간적으로 다운타임이 발생하기 때문에 컴퓨터 자원을 일부밖에 쓰지 못한다. Rolling with Additional Batch의 경우 이를 방지할 수 있다.

 

![버전 업데이트 전 모습 ](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%207.png)

버전 업데이트 전 모습 

![Untitled](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%208.png)

 버전 업데이트가 이루어진다면, 추가적으로 상위버전의 어플리케이션이 배포된 인스턴스가 추가적으로 배포가 된다.

![Untitled](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%209.png)

 배포가 완료되면 인스턴스의 개수가 기존의 100%를 넘어가게 되어 컴퓨터 자원의 문제는 발생하지 않게 된다.

![Untitled](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%2010.png)

 배포가 계속 진행되지만, Rolling과 마찬가지로 사용자들은 서로 다른 경험을 할 수 있다.

![Untitled](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%2011.png)

 모든 업데이트가 완료되면 추가적으로 생성된 인스턴스를 삭제해준다.

## Immutable

 Immutable은 불변이라는 뜻으로 여기서는 기존의 환경의 어떤 변경없이 배포를 한다는 의미이다.

![배포전의 모습](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%2012.png)

배포전의 모습

![Untitled](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%2013.png)

 Auto Scaling 그룹을 새로 만들고, 우선 하나의 인스턴스에 버전 2의 어플리케이션을 올린다.

![Untitled](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%2014.png)

 먼저 업데이트한 인스턴스에서 이상이 없다면 이전 버전의 그룹의 인스턴스 개수와 맞게 올려준다. (만약 버전2의 인스턴스가 문제가 있다면 버전 2의 그룹은 삭제되고, 이전버전의 Auto Scaling 그룹으로 Load Balancing 된다.)

## Traffic Splitting

 트래픽 분할은 기존 Auto Scaling 그룹과 같은 사이즈의 인스턴스를 한 번에 올린 다음 정해진 트래픽 비율에 따라서 트래픽이 분산된다. 정상 배포 후 Traffic splitting evaluation time(트래픽 분할 평가 시간)에 따라 대기한 후 트래픽이 업데이트된 인스턴스로 분산되게 된다.

 이런 테스트를 Canary 테스트라고 하는데, 안정적인 버전을 release하기 전에 테스트 버전을 일부 사용자에게 배포하는 것을 의미한다. 카나리 버전에 심각한 버그가 발생하더라도 사용하는 사용자가 적기 때문에 피해를 최소화할 수 있다.

![Untitled](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%2015.png)

 Traffic Splitting을 선택하면 위의 그림 옵션을 선택할 수 있는데 트래픽 분할은 새 버전에 대한 트래픽 비율을 어느 정도로 할지를 결정하고, “새 어플리케이션에 대한” 시간의 경우 새 어플리케이션으로 전환하기 전에 초기 정상 배포 후에 대기하는 시간(분)이다.

![배포 전의 모습](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%2016.png)

배포 전의 모습

 

![Untitled](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%2017.png)

 새로운 Auto Scaling 그룹이 생성되고 정해진 비율에 따라 트래픽이 분산된다. 여기서 새 어플리케이션에 문제가 생긴다면 이전 버전으로 롤백 된다.

![Untitled](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%2018.png)

 정상배포가 완료되면 일정 시간 후 이전버전의 그룹이 사라지고, 새 어플리케이션 그룹으로 대체된다.

## Blue/Green

 Immutable 방식과 유사하나 도메인을 스와프 시켜줘야 하는 방법에서 차이가 있다.

![Untitled](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%2019.png)

 

기존 환경에서 작업-환경복제를 한다.

![Untitled](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%2020.png)

 도메인을 새로 지정해주고 복제해준다.

![Untitled](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%2021.png)

 복제가 완료되면, 완료된 환경에 새 어플리케이션을 배포한다.

![Untitled](/assets/img/2022-07-13-elastic-beanstalk-deployment-options/Untitled%2022.png)

 배포가 완료되면 환경 URL 교체를 클릭한다.

## Reference

[https://www.youtube.com/watch?v=AfRnvsRxZ_0&t=1176s](https://www.youtube.com/watch?v=AfRnvsRxZ_0&t=1176s)

[https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/Welcome.html](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/Welcome.html)