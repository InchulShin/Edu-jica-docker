# AWS Copilot을 사용하여 Amazon ECS 시작하기

Amazon ECS 애플리케이션을 배포하여 AWS Copilot으로 Amazon ECS를 시작합니다.

## 사전 조건
시작하기 전에 다음 사전 조건을 충족하는지 확인합니다.

- AWS 계정을 설정합니다. 자세한 내용은 Amazon ECS 사용 설정 섹션을 참조하세요.

- AWS Copilot CLI를 설치합니다. 릴리스는 현재 Linux 및 macOS 시스템을 지원합니다. 자세한 내용은 AWS Copilot CLI 설치 섹션을 참조하세요.

- AWS CLI를 설치하고 구성합니다. 자세한 내용은 AWS 명령줄 인터페이스를 참조하세요.

- aws configure를 실행하여 애플리케이션 및 서비스를 관리하기 위해 AWS Copilot CLI에서 사용할 기본 프로필을 설정합니다.

- Docker를 설치 및 실행합니다. 자세한 내용은 Docker 시작하기를 참조하세요.

## 하나의 명령을 사용하여 애플리케이션 배포
AWS 명령줄 도구를 설치하고 시작하기 전에 이미 `aws configure`를 실행하였는지 확인합니다.

다음 명령을 사용하여 애플리케이션을 배포합니다.

```
git clone https://github.com/aws-samples/amazon-ecs-cli-sample-app.git demo-app && \ 
cd demo-app &&                               \
copilot init --app demo                      \
  --name api                                 \
  --type 'Load Balanced Web Service'         \
  --dockerfile './Dockerfile'                \
  --port 80                                  \
  --deploy
```

## 단계별 애플리케이션 배포
### 1단계: 자격 증명 구성
`aws configure`를 실행하여 애플리케이션 및 서비스를 관리하기 위해 AWS Copilot CLI에서 사용하는 기본 프로필을 설정합니다.

```
aws configure
```

### 2단계: 데모 앱 복제
간단한 Flask 애플리케이션 및 Dockerfile을 복제합니다.

```
git clone https://github.com/aws-samples/amazon-ecs-cli-sample-app.git demo-app
```

### 3단계: 애플리케이션 설정
데모 앱 디렉터리 내에서 init 명령을 실행합니다.

```
copilot init
```

AWS Copilot은 다음 단계로 시작하는 일련의 터미널 프롬프트로 사용자의 첫 애플리케이션 및 서비스 설정을 안내합니다. 이미 AWS Copilot을 사용하여 애플리케이션을 배포한 경우, 애플리케이션 이름 목록에서 하나를 선택하라는 메시지가 표시됩니다.

애플리케이션 이름을 지정합니다.

```
What would you like to name your application? [? for help]
```

Enter *demo*.

### 4단계: “데모” 애플리케이션에서 ECS 서비스 설정
1. 서비스 유형을 선택하라는 메시지가 표시됩니다. 작은 API를 제공하는 간단한 Flask 애플리케이션을 만들고 있습니다.

```
Which service type best represents your service's architecture? [Use arrows to move, type to filter, ? for more help]
   > Load Balanced Web Service
     Backend Service
     Scheduled Job
```

Choose *Load Balanced Web Service*.

2. 서비스의 이름을 제공합니다.

```
What do you want to name this Load Balanced Web Service? [? for help]
```

서비스 이름으로 *api*을 입력합니다(Enter).

3. Dockerfile을 선택합니다.

```
Which Dockerfile would you like to use for api? [Use arrows to move, type to filter, ? for more help]
   > ./Dockerfile
     Use an existing image instead
```

**Choose** *Dockerfile*.

4. 포트를 정의합니다.

```
Which port do you want customer traffic sent to? [? for help] (80)
```

*80*을 입력하거나(Enter) 기본값을 그대로 사용합니다.

5. 생성 중인 애플리케이션 리소스를 보여주는 로그가 표시됩니다.

```
Creating the infrastructure to manage services under application demo.
```

6. 애플리케이션 리소스를 만든 후 테스트 환경을 배포합니다.

```
Would you like to deploy a test environment? [? for help] (y/N)
```

Enter *y*.

```
Proposing infrastructure changes for the test environment.
```

7. 애플리케이션 배포 상태를 표시하는 로그가 표시됩니다.

```
Note: It's best to run this command in the root of your Git repository.
Welcome to the Copilot CLI! We're going to walk you through some questions
to help you get set up with an application on ECS. An application is a collection of
containerized services that operate together.

Use existing application: No
Application name: demo
Workload type: Load Balanced Web Service
Service name: api
Dockerfile: ./Dockerfile
no EXPOSE statements in Dockerfile ./Dockerfile
Port: 80
Ok great, we'll set up a Load Balanced Web Service named api in application demo listening on port 80.

✔ Created the infrastructure to manage services under application demo.

✔ Wrote the manifest for service api at copilot/api/manifest.yml
Your manifest contains configurations like your container size and port (:80).

✔ Created ECR repositories for service api.

All right, you're all set for local development.
Deploy: Yes

✔ Created the infrastructure for the test environment.
- Virtual private cloud on 2 availability zones to hold your services     [Complete]
- Virtual private cloud on 2 availability zones to hold your services     [Complete]
  - Internet gateway to connect the network to the internet               [Complete]
  - Public subnets for internet facing services                           [Complete]
  - Private subnets for services that can't be reached from the internet  [Complete]
  - Routing tables for services to talk with each other                   [Complete]
- ECS Cluster to hold your services                                       [Complete]
✔ Linked account aws_account_id and region region to application demo.

✔ Created environment test in region region under application demo.                         
Environment test is already on the latest version v1.0.0, skip upgrade.
[+] Building 0.8s (7/7) FINISHED
 => [internal] load .dockerignore                                                                                  0.1s
 => => transferring context: 2B                                                                                    0.0s
 => [internal] load build definition from Dockerfile                                                               0.0s
 => => transferring dockerfile: 37B                                                                                0.0s
 => [internal] load metadata for docker.io/library/nginx:latest                                                    0.7s
 => [internal] load build context                                                                                  0.0s
 => => transferring context: 32B                                                                                   0.0s
 => [1/2] FROM docker.io/library/nginx@sha256:aeade65e99e5d5e7ce162833636f692354c227ff438556e5f3ed0335b7cc2f1b     0.0s
 => CACHED [2/2] COPY index.html /usr/share/nginx/html                                                             0.0s
 => exporting to image                                                                                             0.0s
 => => exporting layers                                                                                            0.0s
 => => writing image sha256:3ee02fd4c0f67d7bd808ed7fc73263880649834cbb05d5ca62380f539f4884c4                       0.0s
 => => naming to aws_account_id.dkr.ecr.region.amazonaws.com/demo/api:cee7709                                      0.0s
WARNING! Your password will be stored unencrypted in /home/user/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
The push refers to repository [aws_account_id.dkr.ecr.region.amazonaws.com/demo/api]
592a5c0c47f1: Pushed
6c7de695ede3: Pushed
2f4accd375d9: Pushed
ffc9b21953f4: Pushed
cee7709: digest: sha_digest

✔ Deployed api, you can access it at http://demo-Publi-1OQ8VMS2VC2WG-561733989.region.elb.amazonaws.com.
```

### 5단계: 애플리케이션이 실행 중인지 확인
다음 명령을 사용하여 애플리케이션의 상태를 확인합니다.

AWS Copilot 애플리케이션을 모두 나열합니다.

```
copilot app ls
```

애플리케이션의 환경 및 서비스에 대한 정보를 표시합니다.

```
copilot app show
```

환경에 대한 정보를 표시합니다.

```
copilot env ls
```

엔드포인트, 용량 및 관련 리소스를 포함하여 서비스에 대한 정보를 표시합니다.

```
copilot svc show
```

애플리케이션의 모든 서비스 목록입니다.

```
copilot svc ls
```

배포된 서비스의 로그를 표시합니다.

```
copilot svc logs
```

서비스 상태를 표시합니다.

```
copilot svc status
```

사용 가능한 명령 및 옵션을 나열합니다.

```
copilot --help
```
```
copilot init --help
```

### 6단계. CI/CD 파이프라인을 생성하는 방법 알아보기
지침은 ECS 워크숍에서 확인할 수 있습니다. AWS Copilot을 사용하여 CI/CD 파이프라인 및 git 워크플로를 완전히 자동화하는 방법을 자세히 설명합니다.

### 7단계: 정리
다음 명령을 실행하여 모든 리소스를 삭제하고 정리합니다.

```
copilot app delete
```
