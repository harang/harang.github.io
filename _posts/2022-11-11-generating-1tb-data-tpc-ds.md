# TPC-DS 툴을 이용해 S3에 랜덤 데이터 생성하기

데이터 웨어하우스 성능 테스트를 하려고 할 때 

대량 데이터 샘플이 필요해 어려움을 겪었을 수 있다.

이럴 때 TPC-DS 라는 툴을 이용하면 되는데,

대량 랜덤 데이터를 생성해주는 툴이다.

이번 글에서는 [Hive Testbench](https://github.com/hortonworks/hive-testbench) 라는 오픈소스 툴을 이용할 것이다.

깃헙으로 접속해 README 파일을 읽어보면

Hadoop과 Hive 그리고 gcc 설치가 선행되어야 한다는 것을 알 수 있다.

번거롭게 EC2에 각각 설치하지 않고

EMR을 올려서 돌려보기로 결정했다.

## 작업 순서

1. EMR 설치
2. S3 버킷 생성
3. EMR 접속
4. hive-benchmark 실행
5. S3 버킷 확인

## EMR 설치

[EMR 콘솔](https://ap-northeast-2.console.aws.amazon.com/elasticmapreduce/home?region=ap-northeast-2#cluster-list:) 접속

EMR 클러스터 생성을 먼저 해줄것이다.

![Untitled](/assets/img/2022-11-11-generating-1tb-data-tpc-ds/Untitled.png)

클러스터 이름은 hive-benchmark로 설정하고 다른값들은 기본값을 그대로 뒀다.

![Untitled](/assets/img/2022-11-11-generating-1tb-data-tpc-ds/Untitled%201.png)

hive-benchmark 툴로 1TB 데이터를 생성할 것이기 때문에

빠르게 끝내기 위해서 r5.8xlarge 스펙을 이용했고

클러스터 스케일링 옵션도 켜주었다. (Min 2, Max 10)

자동으로 인스턴스를 삭제해주는 Auto-termination 옵션도 15분으로 설정해주었다.

![Untitled](/assets/img/2022-11-11-generating-1tb-data-tpc-ds/Untitled%202.png)

가지고 있는 EC2 키페어를 지정해주었다.

나중에 SSH로 클러스터에 접속할 때 필요하기 때문에, 꼭 액세스 가능한 키페어로 설정하기 바란다.

EMR role, EC2 인스턴스 롤 모두 디폴트를 사용했다.

### S3 버킷 생성

다음은 TPC-DS 데이터를 담을 S3 버킷을 생성해주었다.

![Untitled](/assets/img/2022-11-11-generating-1tb-data-tpc-ds/Untitled%203.png)

### EMR 접속

MASTER 보안그룹 수정

![Untitled](/assets/img/2022-11-11-generating-1tb-data-tpc-ds/Untitled%204.png)

기본 보안그룹에는 SSH 포트가 열려있지 않기 때문에

생성된 EMR 클러스터의 마스터 노드 보안그룹의 인바운드 포트에서 22포트를 My IP로 열어준다.

![Untitled](/assets/img/2022-11-11-generating-1tb-data-tpc-ds/Untitled%205.png)

클러스터 서머리의 Master 퍼블릭 DNS를 이용하여 SSH로 노드에 접속해준다.

Mobaxterm으로 ssh 접속 성공했을 때의 화면이다.

![Untitled](/assets/img/2022-11-11-generating-1tb-data-tpc-ds/Untitled%206.png)

![Untitled](/assets/img/2022-11-11-generating-1tb-data-tpc-ds/Untitled%207.png)

접속에 성공했다면 다음 커멘드로 버전 확인을 해준다.

```bash
#하둡 버전 확인
hadoop version

#하이브 버전 확인
hive --version
```

![Untitled](/assets/img/2022-11-11-generating-1tb-data-tpc-ds/Untitled%208.png)

hive-benchmark 빌드에 필요한 gcc와

소스코드를 내려받을 때 필요한 git도 설치해준다.

```bash
sudo yum update -y
sudo yum install gcc -y
sudo yum install git -y

git clone https://github.com/hortonworks/hive-testbench.git

```

![Untitled](/assets/img/2022-11-11-generating-1tb-data-tpc-ds/Untitled%209.png)

### 하둡 파일시스템 설정

이제 S3버킷을 설정해 줄 차례이다.

하둡 config에서 하둡 파일시스템을 S3 버킷으로 변경해 줄 것이다.

```bash
sudo vim /etc/hadoop/conf/core-site.xml
```

![Untitled](/assets/img/2022-11-11-generating-1tb-data-tpc-ds/Untitled%2010.png)

fs.defaultFS를 S3 경로로 변경해준다.

![Untitled](/assets/img/2022-11-11-generating-1tb-data-tpc-ds/Untitled%2011.png)

s3://버킷이름 으로 설정해주면 끝이다.

### hive-testbench 빌드

다음은 hive-testbench를 빌드해 줄 차례이다.

```bash
cd hive-testbench
./tpcds-build.sh
```

코드를 실행해주면 빌드가 시작된다.

![Untitled](/assets/img/2022-11-11-generating-1tb-data-tpc-ds/Untitled%2012.png)

### hive-testbench 실행

각자 원하는 데이터 량을 선택해줘야 할 차례이다.

나는 1TB 데이터를 생성할 것이라 1000이라는 스케일 값을 입력해주었고

parquet 포멧의 데이터가 필요해서 데이터 포멧도 지정해주었다.

```bash
./tpcds-setup.sh 1000 FORMAT=parquet
```

![Untitled](/assets/img/2022-11-11-generating-1tb-data-tpc-ds/Untitled%2013.png)

코드를 실행시키면 매핑을 시작한다.

20분정도 기다리면 프로그램이 잘 돌아가는걸 확인할 수 있다.

![Untitled](/assets/img/2022-11-11-generating-1tb-data-tpc-ds/Untitled%2014.png)

![Untitled](/assets/img/2022-11-11-generating-1tb-data-tpc-ds/Untitled%2015.png)

Data Loaded라고 뜨면 완료된 것이다.

### S3 버킷사이즈 확인

이제 완료되었으니 데이터가 잘 들어갔는지 확인할 차례이다.

AWS CLI커멘드를 이용하면 쉽게 확인할 수 있다.

```bash
aws s3 ls --summarize --human-readable --recursive s3://버킷이름
```

![Untitled](/assets/img/2022-11-11-generating-1tb-data-tpc-ds/Untitled%2016.png)

EMR을 이용해 손쉽게 대량 샘플 데이터 생성에 성공했다.