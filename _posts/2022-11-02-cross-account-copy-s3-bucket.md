---
title: "Cross Account S3 bucket copy"
categories:
  - AWS
  - Cloud
---

# 다른 계정에 있는 S3 버킷 복제하기

여러 AWS 계정을 이용할 경우, 데이터가 분산 저장되어 있을 수 있는데

하나의 계정으로 데이터를 통합하기 위해

타 계정에 있는 S3 버킷의 데이터를 복제하는 방법을 알아볼 것이다.

[AWS 공식 문서](https://aws.amazon.com/ko/premiumsupport/knowledge-center/copy-s3-objects-account/)를 참고한 내용이다.

작업 순서는 다음과 같다

1. 환경 세팅
    1. 로컬 컴퓨터에 AWS CLI 설치
    2. 소스 버킷에서 ACL 비활성화
2. 소스 버킷이 있는 계정
    1. IAM User 생성
    2. 정책 생성
    3. 생성한 IAM User와 연결
3. 대상 계정
    1. S3 버킷 생성 (ACL 비활성화)
    2. 버킷 정책 설정
4. 복제 수행
    1. 2.b에서 생성한 정책 수정
    2. AWS Configure로 다운로드한 Access Key 이용해서 소스 계정의  IAM User로 로그인
    3. aws s3 sync 커멘드 이용해서 버킷 복제

# 작업 순서

## 사전준비

### 로컬 컴퓨터에 AWS CLI 설치

먼저 로컬 컴퓨터에 AWS CLI를 설치한다.

AWS CLI는 커멘드 라인 인터페이스 (Command Line Interface)의 약자로

AWS 서비스를 터미널에서 제어하고 스크립트를 통해 자동화할 수 있도록 해준다.

로컬 컴퓨터의 OS에 따라 설치하는 방법이 다른데,

윈도우 사용자의 경우

[https://awscli.amazonaws.com/AWSCLIV2.msi](https://awscli.amazonaws.com/AWSCLIV2.msi)

해당 링크를 클릭해 다운로드 받아지는 파일로 설치를 진행하면 된다.

맥OS는

[https://awscli.amazonaws.com/AWSCLIV2.pkg](https://awscli.amazonaws.com/AWSCLIV2.pkg)

pkg 링크 이용하면 동일하게 GUI에서 설치가 가능하다.

### 소스 버킷에서 ACL 비활성화

다음으로 해줘야 하는 작업은 소스 버킷의 ACL 비활성화이다.

ACL을 비활성화 하게 되면, 소스 버킷의 모든 객체의 소유자를 계정 소유자로 강제할 수 있다.

소스 버킷이 있는 AWS 계정으로 콘솔 로그인 한뒤, 

AWS S3 콘솔에서 소스 버킷 - 권한 - 객체 소유권 부분을 확인해주면 된다.

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled.png)

해당 옵션이 활성화 되어있다면 편집에 들어가 수정해주면 된다.

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled%201.png)

ACL 비활성화됨(권장) 옵션을 선택한 후 변경 사항 저장해준다.

**추후에 필요하기 때문에 메모장에 미리 소스 버킷의 이름을 적어두는게 좋다

## S3 소스 버킷(복제 대상)이 있는 계정

다음은 소스 버킷이 있는 계정에서 해주어야하는 작업들이다.

### IAM User 생성

콘솔이 한국어로 설정되어 있다면 User가 사용자라고 표기될 것이다.

[IAM 사용자 콘솔 링크](https://us-east-1.console.aws.amazon.com/iamv2/home#/users)로 접속한 뒤,

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled%202.png)

우측 상단의 사용자 추가를 선택한다.

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled%203.png)

원하는 사용자 이름을 적어주면 되는데

알아보기 쉽게 s3-cross-account-copy-user 이라는 이름을 선택했다.

앞 단계에서 설치한 AWS CLI를 통해서만 이용할 유저이기 때문에 암호 선택하지 않고, 액세스 키만 선택해준다.

아직 권한 정책 생성을 하지 않았기 때문에, 권한 설정은 일단 스킵하고 유저를 생성해준다.

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled%204.png)

스킵하면 무서운 경고가 뜨지만 일단 무시해주면 된다.

사용자 만들기를 누르면 사용자 생성이 완료됐다는 안내창이 뜬다.

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled%205.png)

여기서 꼭 잊지 말고 .csv 다운로드를 선택해서 Access Key .csv 파일을 다운로드 해두어야 한다.

그리고 다시 IAM 사용자 콘솔로 돌아가서, 방금 생성한 사용자를 선택하면

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled%206.png)

이런 화면이 뜰텐데

아까 소스 버킷을 적어둔 메모장에 꼭꼭 사용자 ARN을 적어두는걸 추천한다.

### 정책 생성

다음은 방금 생성한 사용자에게 부여할 정책을 생성해 줄 것이다.

이전과 동일하게 IAM 콘솔에서 작업해주면 된다.

왼쪽 메뉴에서 액세스 관리 - 정책을 선택하거나 [링크](https://us-east-1.console.aws.amazon.com/iamv2/home#/policies)를 통해 접속한다.

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled%207.png)

우측 상단의 정책 생성 선택한 뒤 

JSON 탭을 선택해 정책을 입력해준다.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetObject",
        "s3:GetObjectTagging"
      ],
      "Resource": [
        "arn:aws:s3:::source-DOC-EXAMPLE-BUCKET",
        "arn:aws:s3:::source-DOC-EXAMPLE-BUCKET/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:PutObject",
        "s3:PutObjectAcl",
        "s3:PutObjectTagging"
      ],
      "Resource": [
        "arn:aws:s3:::destination-DOC-EXAMPLE-BUCKET",
        "arn:aws:s3:::destination-DOC-EXAMPLE-BUCKET/*"
      ]
    }
  ]
}
```

코드의 source-DOC-EXAMPLE-BUCKET 부분에 아까 메모장에 적어둔 소스 버킷의 이름으로 수정해준다. 

동일하게 destination-DOC-EXAMPLE-BUCKET도 수정해주어야 하는데, 추후에 대상 버킷을 생성한 뒤 수정해주도록 할 것이다.

태그 추가는 스킵해주고 정책 검토로 넘어가 적절한 이름을 설정해준다.

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled%208.png)

사용자 이름과 통일성을 주기 위해 s3-cross-account-copy-policy 라는 이름을 선택했다.

정책 생성을 누른 뒤 정책 리스트에서 생성한 정책을 눌러 추가 설정을 해줄 것이다.

### 생성한 IAM User와 연결

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled%209.png)

이런 요약 페이지가 나오는데

연결을 누르고 아까 생성해준 사용자를 선택해주면 연결이 끝난다.

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled%2010.png)

## 대상 버킷의 AWS 계정

이제 복제한 객체들이 들어간 버킷을 생성해 줄 차례이다.

타 브라우저를 이용하거나, shift+ctrl+n을 눌러 시크릿 모드로 AWS 콘솔에 접속해준다.

동일한 브라우저를 이용하게 되면, 기존의 로그인한 정보가 리셋되기 때문에 나는 편의성을 위해 시크릿모드로 접속하는 편이다.

 

### S3 버킷 생성

AWS 콘솔에서 S3를 검색해 접속해주거나 [링크](https://s3.console.aws.amazon.com/s3/buckets?region=ap-northeast-2)를 이용하여 접속한다.

이제 복제를 위한 버킷을 생성해 줄 차례이다.

해당 콘솔에서

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled%2011.png)

버킷 만들기를 선택하면 손쉽게 버킷을 생성할 수 있다.

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled%2012.png)

버킷 이름은 AWS의 모든 리전의 모든 사용자가 중복으로 사용할 수 없게 설정되어 있기 때문에

절대 중복되지 않을만한 값으로 설정해 주어야 한다.

아까와 동일하게 메모장에 버킷 이름을 적어준다.

정책 수정할 때 필요함!

여기서의 핵심은 객체 소유권에서 소스 버킷과 동일하게 ACL 비활성화됨을 선택해주어야 한다는 것이다.

다른 옵션들은 스킵하고 버킷을 생성해준다.

추가 설정이 필요하다면 S3 버킷에 복제를 완료하고 난 뒤에 해도 되기 때문에 현 단계에선 스킵하도록 한다.

### 버킷 정책 설정

S3 콘솔에서 생성한 버킷 선택 - 권한 - 버킷 정책 옵션에서

버킷 정책을 편집해줄 차례이다.

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled%2013.png)

```json
{
  "Version": "2012-10-17",
  "Id": "Policy1611277539797",
  "Statement": [
    {
      "Sid": "Stmt1611277535086",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::222222222222:user/Jane"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::destination-DOC-EXAMPLE-BUCKET/*",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-acl": "bucket-owner-full-control"
        }
      }
    },
    {
      "Sid": "Stmt1611277877767",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::222222222222:user/Jane"
      },
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::destination-DOC-EXAMPLE-BUCKET"
    }
  ]
}
```

해당 정책에서 "arn:aws:iam::222222222222:user/Jane" 부분을 메모장에 적어둔 ARN으로 전부 수정하고

destination-DOC-EXAMPLE-BUCKET 부분도 방금 생성하면서 적어둔 대상 버킷 이름으로 수정해준다.

## 복제 수행

이제 마지막 단계이다.

### 소스 계정의 정책 수정

소스 계정의 AWS 콘솔로 돌아가서 [정책](https://us-east-1.console.aws.amazon.com/iamv2/home#/policies)을 다시 선택한 뒤 destination-DOC-EXAMPLE-BUCKET 이름을 수정해줄 것이다.

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled%2014.png)

먼저 정책 편집을 눌러주고

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled%2015.png)

해당 부분의 destination-DOC-EXAMPLE-BUCKET 을 새로 생성한 버킷이름으로 수정해준다.

### AWS CLI에 로그인

AWS Configure로 다운로드한 Access Key이용해서 소스 계정의  IAM User로 로그인해 줄 것이다.

익숙한 터미널 툴을 이용해 진행해주면 되는데, 이번에 파워쉘을 이용할 것이다.

먼저 AWS CLI가 잘 설치되어있는지 확인한다.

```json
aws --version
```

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled%2016.png)

이제 앞서 말했던 AWS Configure이라는 커멘드를 통해 로그인해줄 것인데,

아까 다운받은 Access Key .csv 파일을 활용할 차례이다.

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled%2017.png)

차례대로 

AWS Access Key ID와 Secret Access Key는 .csv 파일에서 복사 붙여넣기 해주면 되고,

region name은 AWS 리전 선택을 해주는 것인데 서울 리전은 ap-northeast-2라고 적어주면 된다.

output format은 json이라고 입력해주면

AWS CLI에 로그인이 끝났다.

### aws s3 sync 커멘드 이용해서 버킷 복제

이제 드디어 버킷 복제를 할 차례이다.

```json
aws s3 sync s3://source-DOC-EXAMPLE-BUCKET s3://destination-DOC-EXAMPLE-BUCKET --acl bucket-owner-full-control
```

아까와 동일하게 버킷 이름들을 수정해준 뒤 터미널에 입력해주면 끝이다.

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled%2018.png)

에러메세지가 뜨지 않고 copy:로 시작하는 메세지가 잔뜩 뜨면 복제에 성공한 것이다.

소스 버킷

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled%2019.png)

대상 버킷

![Untitled](/assets/img/2022-11-02-cross-account-copy-s3-bucket/Untitled%2020.png)

동일하게 복제가 완료된 것을 확인할 수 있다.