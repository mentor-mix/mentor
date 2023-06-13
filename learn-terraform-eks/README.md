# terraform 으로 EKS 클러스터 프로비너닝 하기

- 참고
  - [Provision an EKS Cluster learn guide](https://learn.hashicorp.com/terraform/kubernetes/provision-eks-cluster)
  - https://learn.hashicorp.com/tutorials/terraform/eks
- 아키텍처
[![arch](./terraform-eks-arch.png?raw=true)]()

## 1. bastion 생성
- 최소 사양으로 생성

### 1.1 환경 구성

- root 권한 변경
```sh
sudo -i
```

#### 1.1.1. git 구성
```sh
yum update
yum install git
```

#### 1.1.2. terraform 구성

https://learn.hashicorp.com/tutorials/terraform/install-cli

```sh
yum install -y yum-utils
yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
yum -y install terraform
```

#### 1.1.3. docker 구성

https://learn.hashicorp.com/tutorials/terraform/install-cli
하단 내용 필요하면 docker 설치후 해보기

#### 1.1.4 AWS CLI 구성

https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

```sh
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
```

```sh
$ aws configure
AWS Access Key ID [None]: 
AWS Secret Access Key [None]: 
Default region name [None]: us-east-1
Default output format [None]:
```

## 2. terraform 으로 EKS 클러스터 배포

### 2-1. terraform 명령어 수행

- terraform init
- terraform plan 
- terraform apply

### 2-2. kubeconfig

```sh
aws eks --region $(terraform output -raw region) update-kubeconfig --name $(terraform output -raw cluster_name)
```

## 3. 애플리케이션 배포

```yaml
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx
spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nginx
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: default
  labels:
    app: nginx
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
```


## 4. Clean Up

```sh
terraform destroy
```

## 5. NEXT ?

https://github.com/aws-ia/terraform-aws-eks-blueprints