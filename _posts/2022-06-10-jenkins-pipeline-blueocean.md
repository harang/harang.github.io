---
title: "Jenkins Pipeline 구성"
categories:
  - AWS
  - Cloud
---

젠킨스를 처음 사용하는 개발자들이라면
투박한 UI에 당황한 경험이 다들 있을 것이다.
게다가 플러그인은 어찌 이리 많은지,
설정들은 여기저기 숨어있어
젠킨스가 접근성이 좋은 툴은 아니라고 생각했었다.

하지만 Blue Ocean이라는 플러그인을 이용하면
손쉽게 시각화된 파이프라인을 확인 및 설정할 수 있게됐다.

이번에는 harbor에서 내려받은 도커 이미지를 blue ocean 통해서 eks에 배포하는 과정을 살펴볼 것이다.


### Blue Ocean UI 상 설정

![/assets/img/2022-06-10-jenkins-pipeline-blueocean/Untitled.png](/assets/img/2022-06-10-jenkins-pipeline-blueocean/Untitled.png)

Blue Ocean을 이용하면 파이프라인을 이런 식으로 볼 수 있게된다.
다음은 해당 파이프라인의 코드이다.


### JenkinsFile 코드 예시

```jsx
pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        echo 'Build'
      }
    }

    stage('Test') {
      steps {
        echo 'Test'
      }
    }

    stage('Deploy') {
      steps {
        echo 'Deploy'
      }
    }

  }
}
```
Blue Ocean을 이용해도 기존과 동일하게 JenkinsFile을 통해 설정할 수 있다.


### Jenkins에 SAM 사용해서 AWS Credential 설정
AWS에 배포하려면 리소스들이 올라갈 AWS 계정의 크레덴셜을 이용해 젠킨스와 연동해주어야 한다.

- 참조 문서
    
    [https://aws.amazon.com/ko/blogs/compute/building-a-jenkins-pipeline-with-aws-sam/](https://aws.amazon.com/ko/blogs/compute/building-a-jenkins-pipeline-with-aws-sam/)
    

Pipeline: AWS Steps 플러그인 설치

[http://젠킨스URL/credentials/store/system/domain/_/](http://3.37.129.39:8080/credentials/store/system/domain/_/)

![/assets/img/2022-06-10-jenkins-pipeline-blueocean/Untitled%201.png](/assets/img/2022-06-10-jenkins-pipeline-blueocean/Untitled%201.png)

### 파이프라인 구성 예시

withAWS로 젠킨스에 연결한 AWS ID를 사용하는 예시이다.

```jsx
pipeline {
  agent any
 
  stages {
    stage('Install sam-cli') {
      steps {
        sh 'python3 -m venv venv && venv/bin/pip install aws-sam-cli'
        stash includes: '**/venv/**/*', name: 'venv'
      }
    }
    stage('Build') {
      steps {
        unstash 'venv'
        sh 'venv/bin/sam build'
        stash includes: '**/.aws-sam/**/*', name: 'aws-sam'
      }
    }
    stage('beta') {
      environment {
        STACK_NAME = 'sam-app-beta-stage'
        S3_BUCKET = 'sam-jenkins-demo-us-west-2-user1'
      }
      steps {
        withAWS(credentials: 'awsjenkins', region: 'ap-northeast-2') {
          unstash 'venv'
          unstash 'aws-sam'
          sh 'venv/bin/sam deploy --stack-name $STACK_NAME -t template.yaml --s3-bucket $S3_BUCKET --capabilities CAPABILITY_IAM'
          dir ('hello-world') {
            sh 'npm ci'
            sh 'npm run integ-test'
          }
        }
      }
    }
    stage('prod') {
      environment {
        STACK_NAME = 'sam-app-prod-stage'
        S3_BUCKET = 'sam-jenkins-demo-us-east-1-user1'
      }
      steps {
        withAWS(credentials: 'awsjenkins', region: 'ap-northeast-2') {
          unstash 'venv'
          unstash 'aws-sam'
          sh 'venv/bin/sam deploy --stack-name $STACK_NAME -t template.yaml --s3-bucket $S3_BUCKET --capabilities CAPABILITY_IAM'
        }
      }
    }
  }
}
```

### withCredentials 사용해서 설정

1.  `your jenkins server url/pipeline-syntax/` 로 접속해서 파이프라인 툴 열기
2.  `withCredentials: Bind credentials to variables` 선택 
3. Bindings의 Add 버튼 누르고 팝업 옵션에서 `AWS access key and secret` 선택
4.  `Crendentials` 메뉴에서 선택
5.  `Generate pipeline script` 버튼 클릭

```groovy
steps {
						withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: '', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
						    // some block
						}
      }
```


##다음은 젠킨스 파이프라인의 예시 코드이다.

### EKS 클러스터 설정 방법

```groovy
//EKS 설정
aws eks update-kubeconfig \--region ap-northeast-2 \--name cluster-name
```

### 도커 이미지 실행

```groovy
//도커 이미지 다운
docker pull nginx
//다운받은 이미지 확인
docker images nginx

//도커 이미지 실행
docker run -d --restart always nginx
//실행중인 컨테이너 확인
docker ps

```

### 배포

```bash
//네임스페이스 생성
kubectl create namespace <my-namespace>
//Deployment 생성
kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
//Deployment 확인
kubectl get deployments
//Deployment rollout 현황 확인
kubectl rollout status deployment/nginx-deployment

```

