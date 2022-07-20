# fargatecli로 컨테이너 만들기

<!--fargatecli 는 [제1회 AWS Fargate 간단 배포 선수권](https://speakerdeck.com/toricls/the-very-first-aws-fargate-easy-deployment-tooling-championship) 에서 소개되고 있는, Fargate에 바보만큼 간단하게 배포할 수 있는 CLI 툴. 혼(하나호지)으로 방치하고 있었습니다만, 문득 움직여 보았습니다.-->

[awslabs/fargatecli](https://github.com/awslabs/fargatecli)

## 준비
fargatecli는 무엇이든 만들 수 없다. 약간 귀찮게 해줄 필요가 있습니다.

## VPC 준비
Default VPC 를 사용합니다. 사용할 Subnet ID 를 확인합니다.
<!--fargatecli는 VPC 세트를 만들지 않습니다. 클라메소 씨가 최강의 공부용 VPC를 만드는 CloudFormation을 준비해 주므로 이제 VPC를 준비합니다. 지역을 좋아하는 것 사용하십시오.
[필요할 때만 생성하는 NAT-Gareway](https://dev.classmethod.jp/articles/create_nat_gateway/)
Frontend Subnet1, 2의 ID와 ApplicationSubnet1, 2의 ID를 기록해 둡니다.-->

## ALB에 붙이는 보안 그룹 준비
fargatecli는 보안 그룹을 만들지 않습니다. 만들기. 여러분은 ALB 80에 액세스 할 수 있도록 열어 둡니다. 이것은 무리하게 인터넷으로부터 접속을 허가할 필요는 없고, 자택의 IP주소로부터만이라든지 열어 두면 충분합니다.

## 컨테이너에 추가할 보안 그룹 준비
fargatecli는 보안 그룹을 만들지 않습니다. 만들기. ALB의 보안 그룹 ID에서 컨테이너 80에 액세스할 수 있도록 열어 둡니다.

## ECS에 클러스터 만들기
fargatecli는 ECS 클러스터를 만들지 않습니다. 만들기.

```
$ aws ecs create-cluster --cluster-name nginx-cluster
```

## fargatecli 설치
go get은 fargatecli라는 명령 이름으로 바이너리를 생성합니다.

```
$ go get github.com/awslabs/fargatecli
$ which fargatecli
```

## fargatecli 사용
fargatecli를 사용하면 oneshot적인 Task, 서있는 Service, 외부에 공개하는 LB, LB에 붙는 Certificate를 관리할 수 있습니다. 이곳에서는 NGINX를 확실히 세우는 것으로, 빌어 먹을 간단한 분위기를 봐 주세요.

## fargatecli로 ALB 만들기
lb 명령으로 ALB를 구성할 수 있습니다.

```
$ fargatecli lb create nginx-lb \
      --port 80 --scheme internet-facing \
      --subnet-id subnet-0807a5c3c2e826742,subnet-0d6b6251136c18fa0 \
      --security-group-id sg-0b21077fb303c2e2a
[i] Created load balancer nginx-lb
```

성공하면 ALB 생성이 시작됩니다. STATUS가 Active가 될 때까지 기다립니다.

```
$ fargatecli lb list
NAME		TYPE		STATUS		DNS NAME					PORTS
nginx-lb	Application	Provisioning	nginx-lb-845291675.us-east-1.elb.amazonaws.com	HTTP:80
```

이상한 곳에 아무것도없는 ALB입니다. AWS CLI와 AWS 매니콘에서도 보통 ALB로 존재합니다.

```
$ aws elbv2 describe-load-balancers --names nginx-lb
```

## fargatecli로 컨테이너 만들기
드디어 컨테이너 생성에 있습니다.

CLI 매개변수가 많습니다. ECS 클러스터 이름, 컨테이너를 설정하는 VPC 서브넷 ID, 컨테이너에 추가할 보안 그룹 ID, 머신 리소스, 시작할 컨테이너 수, 컨테이너 이미지, 열려는 포트, 연결할 ALB 이름을 지정합니다.

```
$ fargatecli service create nginx-service \
    --cluster nginx-cluster \
    --subnet-id subnet-0f53ebfa6d0c7a323,subnet-0b37fdb04471d9495 \
    --cpu 256 --memory 512 --num 2 \
    --image nginx:stable --port 80 \
    --security-group-id sg-04834761e4cb48b21 \
    --lb nginx-lb
[i] Created service nginx-service
```

cluster 지정으로 아래와 같이 기동 상황을 볼 수 있습니다.

```
$ fargatecli service list --cluster nginx-cluster
NAME		IMAGE		CPU	MEMORY	LOAD BALANCER	DESIRED	RUNNING	PENDING	
nginx-service	nginx:stable	256	512	nginx-lb	2	0	2	
```

RUNNING이 되면 ALB URL에 액세스하여 NGINX의 초기 페이지를 볼 수 있습니다. 이번에는 HTTPS를 사용하지 않으므로 HTTP로 액세스하십시오.

## fargatecli로 컨테이너 이미지 교체
직장에서 사용하면 새로운 버전의 컨테이너 이미지를 배포할 수 있습니다. nginx:stable 을, 아무래도 와자라고 합니다만 nginx:latest 로 바꿉니다.

```
$ fargatecli service deploy nginx-service \
      --cluster nginx-cluster \
      --image nginx:latest
[i] Deployed nginx:latest to service nginx-service
```

```
$ fargatecli service list --cluster nginx-cluster
NAME		IMAGE		CPU	MEMORY	LOAD BALANCER	DESIRED	RUNNING	PENDING	
nginx-service	nginx:latest	256	512	nginx-lb	2	2	0
```

이런 식으로 쉽게 배포할 수 있습니다.

## fargatecli로 정리
서비스는 먼저 제로대에 떨어지고, 제로대가 되면 삭제할 수 있습니다.

```
$ fargatecli service scale nginx-service "0" \
    --cluster nginx-cluster
[i] Scaled service nginx-service to 0
```

```
$ fargatecli service destroy nginx-service \
    --cluster nginx-cluster
[i] Destroyed service nginx-service
```

ALB도 삭제할 수 있습니다.

```
$ fargatecli lb destroy nginx-lb
[i] Destroyed load balancer nginx-lb
```
<!--
## 어느 정도 사용해도 좋은가?
이상과 같이, ECS도 Fargate도 아무것도 모르는 곳에서, 어쨌든 컨테이너를 세울 수 있는 것이 최고입니다. 컨테이너 이미지 배포 외에도 README.md를 읽으면 Fargate의 스케일 업, 스케일 아웃, 재시작도 가능하거나 일련의 운영 시나리오를 커버 할 수있을 것 같습니다.

그러나 fargatecli의 커밋 로그를 바라 보면, 굉장히 활발하게 개발되고 있는 것도 없을 것 같고, 긴 눈에서는 조금 불안이 있습니다. 일로 귀찮은 제품의 CD 파이프 라인에 꽉 끼워넣으면 갑자기 움직이지 않을 것 같은 생각이되지 않습니다. 거기에서 Fork하고 일련 托生하는 근성이 있다면 사용해 좋을 것 같습니다.

공부 용도나 신규 제품 출시 시 정도에 편리하게 사용할 수 있는 지금 중 사용하여 ecspresso, docker-compose, Copilot 등으로 시작할 수 있도록 도망치는 정도가 좋은 것일지도 모릅니다. 

