---
title: "kops로 EKS 배포하기"
categories:
  - AWS
  - Cloud
---


1. 환경 준비
    - 마스터 머신에 kops, 쿠버네티스 설치

1. 도메인 준비
    - Route 53 도메인 준비
        - [sample-dev.com](http://sample-dev.com) 구매
        - 서브도메인 설정법
            
            [dev.sample-dev.com](http://dev.sample-dev.com) 을 사용하려면 해줘야하는 설정 
            
            (실제로 설정하진 않았음, 3번으로 넘어가서 sample-dev.com 사용했음)
            
            ```bash
            aws route53 create-hosted-zone --name [dev.sample-dev.com](http://dev.sample-dev.com/) --caller-reference 1
            #subdomain dev.sample-dev.com 생성
            ```
            
            Partent hosted zone id 메모
            
            ```bash
            # Note: This example assumes you have jq installed locally.
            aws route53 list-hosted-zones | jq '.HostedZones[] | select(.Name=="example.com.") | .Id'
            ```
            
            Subdomain의 NS value (subdomain.json)
            
            ```json
            {
              "Comment": "Create a subdomain NS record in the parent domain",
              "Changes": [
                {
                  "Action": "CREATE",
                  "ResourceRecordSet": {
                    "Name": "subdomain.example.com",
                    "Type": "NS",
                    "TTL": 300,
                    "ResourceRecords": [
                      {
                        "Value": "ns-1.<example-aws-dns>-1.co.uk"
                      },
                      {
                        "Value": "ns-2.<example-aws-dns>-2.org"
                      },
                      {
                        "Value": "ns-3.<example-aws-dns>-3.com"
                      },
                      {
                        "Value": "ns-4.<example-aws-dns>-4.net"
                      }
                    ]
                  }
                }
              ]
            }
            ```
            
            Subdomain NS 레코드 적용
            
            ```bash
            aws route53 change-resource-record-sets \
             --hosted-zone-id <parent-zone-id> \
             --change-batch file://subdomain.json
            ```
            
        
2. 클러스터 상태 저장용 S3 버킷 생성
    
    ```bash
    #S3 버킷 생성
    aws s3 mb s3://clusters.dev.sample-dev.com
    
    #kops는 이 위치를 기본값으로 인식
    #이 부분을 bash profile등에 넣어두는것을 권장
    export KOPS_STATE_STORE=s3://clusters.dev.sample-dev.com 
    ```
    

1. 클러스터 설정 구성

```bash
export NAME=northeast2.sample-dev.com

kops create cluster \
#AWS Availability Zone 설정
--zones=ap-northeast-2a,ap-northeast-2b,ap-northeast-2c \
#클러스터 이름 설정
--name=${NAME} \
#특정 VPC 및 서브넷 설정 (resource id 추가)
--vpc=vpc-0f0718f641fb72bf1 \
--subnets=subnet-01eda348d14f748bc,subnet-007a52b216e03a900,subnet-04529fe54dcc9f281 \
#aws 서비스 사용하려면 pub 키 생성 후 설정해줘야 함
--ssh-public-key ./.ssh/id_rsa.pub

#클러스터 보기
kops get cluster

#노드그룹 보기
kops get ig --name=${NAME}

#생성 후 잉여 노드그룹 삭제
kops delete ig --name=${NAME} nodes-ap-northeast-2b,nodes-ap-northeast-2c

#노드그룹 설정 (instance size, max, min node #)
kops edit ig --name=${NAME} nodes-ap-northeast-2a

#마스터 노드 그룹 설정
kops edit ig --name=${NAME} master-ap-northeast-2a

#클러스터 설정
kops edit cluster ${NAME}

```

1. AWS에 클러스터 생성 및 리소스 배포

```bash
kops update cluster ${NAME} --yes --admin
```

Note: 클러스터 삭제하기

```bash
kops delete cluster ${NAME} --yes
```

---

### Kubernetes 설정하는 법

- 쿠버네티스 크레덴셜을 따로 설정해 주지 않았기 때문에 참고해야 할 사항

Exported kubecfg with no user authentication; use --admin, --user or --auth-plugin flags with `kops export kubecfg`

Cluster is starting.  It should be ready in a few minutes.

Suggestions:

- validate cluster: kops validate cluster --wait 10m
- list nodes: kubectl get nodes --show-labels
- ssh to the master: ssh -i ~/.ssh/id_rsa [ubuntu@api.northeast2.sample-dev.com](mailto:ubuntu@api.northeast2.sample-dev.com)
- the ubuntu user is specific to Ubuntu. If not using Ubuntu please use the appropriate user based on your OS.
--> Debian은 admin으로 로그인
- read about installing addons at: [https://kops.sigs.k8s.io/operations/addons](https://kops.sigs.k8s.io/operations/addons).