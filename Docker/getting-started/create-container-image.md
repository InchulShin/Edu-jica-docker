# Amazon ECS에서 사용할 컨테이너 이미지 생성

Amazon ECS는 작업 정의에 Docker 이미지를 사용하여 컨테이너를 시작합니다. Docker는 사용자가 컨테이너에서 분산 애플리케이션을 구축, 실행, 테스트 및 배포할 수 있는 도구를 제공합니다. Docker는 Amazon ECS에 컨테이너를 배포하는 방법을 단계별로 제공합니다. 자세한 내용은 Amazon ECS에 Docker 컨테이너 배포를 참조하세요.

여기에 설명된 단계의 목적은 첫 번째 Docker 이미지를 생성하고 해당 이미지를 Amazon ECS 작업 정의에 사용하기 위해 컨테이너 레지스트리인 Amazon ECR에 푸시하는 과정을 안내하는 것입니다. 이 과정에서는 여러분이 Docker가 무엇인지 및 작동 방식에 대해 기본적인 이해를 하고 있다고 가정합니다. Docker에 대한 자세한 내용은 Docker란 무엇인가? 및 Docker 개요를 참조하세요.

    중요
    AWS와 Docker가 협력하여 Docker 도구로 Amazon ECS에 직접 컨테이너를 배포하고 관리할 수 있는 단순화된 개발자 환경을 만들었습니다. 이제 Docker Desktop 및 Docker Compose를 사용하여 컨테이너를 로컬로 구축하고 테스트한 다음 Fargate의 Amazon ECS에 배포할 수 있습니다. Amazon ECS와 Docker의 통합을 시작하려면 Docker Desktop을 다운로드하고 필요에 따라 Docker ID에 가입합니다. 자세한 내용은 Docker Desktop과 Docker ID 가입을 참조하세요.

## 사전 조건
시작하기 전에 다음 사전 요구 사항을 충족하는지 확인합니다.

- Amazon ECR 설정 단계를 완료했는지 확인합니다.. 자세한 내용은 Amazon Elastic Container Registry 사용 설명서의 Amazon ECR에 대한 설정을 참조하세요.

- 사용자에게 Amazon ECR 서비스에 액세스하고 사용하는 데 필요한 IAM 권한이 있습니다. 자세한 내용은 Amazon ECR 관리형 정책을 참조하세요.

- 도커가 설치되어 있습니다. Amazon Linux 2의 Docker 설치 단계에 대한 자세한 내용은 Amazon Linux 2에 Docker 설치을(를) 참조하세요. 다른 모든 운영 체제의 경우 Docker Desktop 개요에서 Docker 설명서를 참조하세요.

- AWS CLI를 설치하고 구성합니다. 자세한 내용은 AWS Command Line Interface 사용 설명서에서 AWS Command Line Interface 설치를 참조하세요.

로컬 개발 환경이 없거나 필요하지 않고 Amazon EC2 인스턴스를 사용하여 Docker를 사용하려는 경우 다음 단계를 수행하여 Amazon Linux 2를 사용하는 Amazon EC2 인스턴스를 시작하고 Docker Engine과 Docker CLI를 설치합니다.

## Amazon Linux 2에 Docker 설치
<details>
<summary>Amazon Linux 2에 Docker 설치</summary>
<div markdown="1">

Docker Desktop은 설치가 간단한 Mac 또는 Windows 환경용 애플리케이션으로 컨테이너화된 애플리케이션 및 마이크로 서비스를 구축하고 공유할 수 있습니다. Docker Desktop에는 Docker Engine, Docker CLI 클라이언트, Docker Compose 및 Amazon ECS에서 Docker를 사용할 때 유용한 기타 도구가 포함되어 있습니다. 기본 운영 체제에 Docker Desktop을 설치하는 방법에 대한 자세한 내용은 Docker Desktop 개요를 참조하세요.

Amazon EC2 인스턴스에 Docker 설치

Amazon Linux 2 AMI 인스턴스를 시작합니다. 자세한 내용은 Linux 인스턴스용 Amazon EC2 사용 설명서의 인스턴스 시작을 참조하세요.

인스턴스에 연결합니다. 자세한 내용은 Linux 인스턴스용 Amazon EC2 사용 설명서에서 Linux 인스턴스에 연결을 참조하세요.

인스턴스에 설치한 패키지 및 패키지 캐시를 업데이트합니다.

sudo yum update -y
최신 Docker Engine 패키지를 설치합니다.

sudo amazon-linux-extras install docker
중요
이 단계에서는 인스턴스에 Amazon Linux 2 AMI를 사용하고 있다고 가정합니다. 다른 모든 운영 체제의 경우 Docker Desktop 개요를 참조하세요.

Docker 서비스를 시작합니다.

sudo service docker start
(선택 사항) 시스템이 재부팅될 때마다 Docker 데몬이 시작되도록 하려면 다음 명령을 실행합니다.

sudo systemctl enable docker
sudo를 사용하지 않고도 Docker 명령을 실행할 수 있도록 docker 그룹에 ec2-user를 추가합니다.

sudo usermod -a -G docker ec2-user
로그아웃하고 다시 로그인해서 새 docker 그룹 권한을 선택합니다. 이를 위해 현재 SSH 터미널 창을 닫고 새 창에서 인스턴스를 다시 연결할 수 있습니다. 새 SSH 세션은 해당되는 docker 그룹 권한을 갖게 됩니다.

sudo 없이도 Docker 명령을 실행할 수 있는지 확인합니다.

docker info
참고
경우에 따라서는 ec2-user가 Docker 데몬에 액세스할 수 있는 권한을 제공하기 위해 인스턴스를 재부팅해야 할 수도 있습니다. 다음 오류가 표시될 경우 인스턴스를 재부팅합니다.

Cannot connect to the Docker daemon. Is the docker daemon running on this host?

</div>
</details>

## Docker 이미지 생성
Amazon ECS 태스크 정의는 Docker 이미지를 사용하여 클러스터의 컨테이너 인스턴스에서 컨테이너를 시작합니다. 이 섹션에서는 간단한 웹 애플리케이션의 Docker 이미지를 생성하여 이를 로컬 시스템이나 Amazon EC2 인스턴스에서 테스트한 다음, Amazon ECR 컨테이너 레지스트리에 푸시하여 Amazon ECS 작업 정의에서 사용할 수 있도록 합니다.

### 간단한 웹 애플리케이션의 Docker 이미지를 생성하려면

1. Dockerfile이라는 파일을 생성합니다. Dockerfile은 Docker 이미지에 사용할 기본 이미지 및 이를 설치하고 실행할 항목을 설명하는 매니페스트입니다. Dockerfile에 대한 자세한 내용은 Dockerfile 참조를 참조하세요.

```
touch Dockerfile
```

2. 방금 만든 Dockerfile을 수정하고 다음 내용을 추가합니다.

```
FROM ubuntu:18.04

# Install dependencies
RUN apt-get update && \
 apt-get -y install apache2

# Install apache and write hello world message
RUN echo 'Hello World!' > /var/www/html/index.html

# Configure apache
RUN echo '. /etc/apache2/envvars' > /root/run_apache.sh && \
 echo 'mkdir -p /var/run/apache2' >> /root/run_apache.sh && \
 echo 'mkdir -p /var/lock/apache2' >> /root/run_apache.sh && \ 
 echo '/usr/sbin/apache2 -D FOREGROUND' >> /root/run_apache.sh && \ 
 chmod 755 /root/run_apache.sh

EXPOSE 80

CMD /root/run_apache.sh
```

이 Dockerfile은 Ubuntu 18.04 이미지를 사용합니다. RUN 지침은 패키지 캐시를 업데이트하고, 웹 서버의 일부 소프트웨어 패키지를 설치하고, "Hello World!" 내용을 웹 서버의 문서 루트에 씁니다. EXPOSE 지침은 컨테이너에 포트 80을 노출하고 CMD 지침은 웹 서버를 시작합니다.

3. Dockerfile에서 Docker 이미지를 빌드합니다.

    참고
    아래의 명령에서 Docker의 일부 버전에서는 아래 보이는 상대 경로 대신에 Dockerfile의 전체 경로가 필요할 수 있습니다.

```
docker build -t hello-world .
```

docker images를 실행하여 이미지가 올바로 생성되었는지 확인합니다.

```
docker images --filter reference=hello-world

Output:

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              e9ffedc8c286        4 minutes ago       241MB
```

5. 새로 빌드된 이미지를 실행합니다. -p 80:80 옵션은 컨테이너에 있는 노출된 포트 80을 호스트 시스템에 있는 포트 80에 매핑합니다. docker run에 대한 자세한 내용을 보려면 Docker 실행 참조를 참조하세요.

```
docker run -t -i -p 80:80 hello-world
```

    참고
    Apache 웹 서버로부터의 출력이 터미널 창에 표시됩니다. "Could not reliably determine the server's fully qualified domain name" 메시지는 무시해도 됩니다.

6. 브라우저를 열고 Docker를 실행하고 컨테이너를 호스팅하고 있는 서버를 가리킵니다.

- EC2 인스턴스를 사용하고 있는 경우 서버의 Public DNS 값이며, 이는 SSH로 인스턴스에 연결할 때 사용하는 주소와 동일합니다. 인스턴스의 보안 그룹에서 포트 80에 인바운드 트래픽을 허용해야 합니다.

- Docker를 로컬에서 실행하고 있는 경우, 브라우저에서 http://localhost/를 가리킵니다.

- Windows 또는 Mac 컴퓨터에서 docker-machine을 사용하는 경우, docker-machine ip 명령을 사용하여 Docker를 호스팅하고 있는 VirtualBox VM의 IP 주소를 찾고, 사용하고 있는 Docker 머신의 이름으로 machine-name을 바꿉니다.

```
docker-machine ip machine-name
```

"Hello, World!" 문이 있는 웹 페이지가 표시됩니다.

7. Ctrl + c를 입력하여 Docker 컨테이너를 중지합니다.

## Amazon Elastic 컨테이너 레지스트리에 이미지 푸시
Amazon ECR은 관리형 AWS Docker 레지스트리 서비스입니다. Docker CLI를 사용하여 Amazon ECR 리포지토리에서 이미지를 푸시, 풀링 및 관리할 수 있습니다. Amazon ECR 제품 세부 정보, 주요 고객 사례 연구 및 FAQ에 대해서는 Amazon Elastic 컨테이너 레지스트리 제품 세부 정보 페이지를 참조하세요.

### 이미지에 태그를 지정하고 Amazon ECR에 푸시하려면

1. hello-world 이미지를 저장할 Amazon ECR 리포지토리를 생성합니다. 출력의 repositoryUri를 참고합니다.

region을 AWS 리전(예: us-east-1)으로 바꿉니다.

```
aws ecr create-repository --repository-name hello-repository --region region
```

Output:

```
{
    "repository": {
        "registryId": "aws_account_id",
        "repositoryName": "hello-repository",
        "repositoryArn": "arn:aws:ecr:region:aws_account_id:repository/hello-repository",
        "createdAt": 1505337806.0,
        "repositoryUri": "aws_account_id.dkr.ecr.region.amazonaws.com/hello-repository"
    }
}
```

2. hello-world 이미지를 이전 단계의 repositoryUri 값으로 태그 지정합니다.

```
docker tag hello-world aws_account_id.dkr.ecr.region.amazonaws.com/hello-repository
```

3. aws ecr get-login-password 명령을 실행합니다. 인증할 레지스트리 URI를 지정합니다. 자세한 내용은 Amazon Elastic Container Registry 사용 설명서의 레지스트리 권한을 참조하세요.

```
aws ecr get-login-password | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.region.amazonaws.com
```

출력:

```
Login Succeeded
```

    중요
    오류가 발생하면 최신 버전의 AWS CLI를 설치하거나 업그레이드합니다. 자세한 내용은 AWS Command Line Interface 사용 설명서에서 AWS Command Line Interface 설치를 참조하세요.

4. 이전 단계의 repositoryUri 값을 사용하여 Amazon ECR로 이미지를 푸시합니다.

```
docker push aws_account_id.dkr.ecr.region.amazonaws.com/hello-repository
```

## 정리
Amazon ECS 작업 정의를 생성하고 컨테이너 이미지로 작업을 시작하려면 다음 단계(으)로 넘어가세요. Amazon ECR 이미지 실험이 끝나면 이미지 저장 비용을 물지 않도록 리포지토리를 삭제할 수 있습니다.

```
aws ecr delete-repository --repository-name hello-repository --region region --force
```
