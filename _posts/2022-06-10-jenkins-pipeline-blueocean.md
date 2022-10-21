---
title: "Jenkins Pipeline 구성"
categories:
  - AWS
  - Cloud
---

- 도커 이미지 blue ocean 통해서 eks에 배포
- harbor 에서 이미지 받았다고 가정

### Blue Ocean UI 상 설정

![/assets/img/2022-06-10-jenkins-pipeline-blueocean/Untitled.png](/assets/img/2022-06-10-jenkins-pipeline-blueocean/Untitled.png)

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

### Jenkins에 SAM 사용해서 AWS Credential 설정

- 참조 문서
    
    [https://aws.amazon.com/ko/blogs/compute/building-a-jenkins-pipeline-with-aws-sam/](https://aws.amazon.com/ko/blogs/compute/building-a-jenkins-pipeline-with-aws-sam/)
    

Pipeline: AWS Steps 플러그인 설치

[http://젠킨스URL/credentials/store/system/domain/_/](http://3.37.129.39:8080/credentials/store/system/domain/_/)

![/assets/img/2022-06-10-jenkins-pipeline-blueocean/Untitled%201.png](/assets/img/2022-06-10-jenkins-pipeline-blueocean/Untitled%201.png)

### 파이프라인 구성 예시

withAWS 사용해서 젠킨스에 설정한 ID사용

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

