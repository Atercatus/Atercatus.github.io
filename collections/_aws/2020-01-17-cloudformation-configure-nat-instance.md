---
title: "How to configure NAT Instance using AWS CLoudFormation"
excerpt: "How to configure NAT Instance using AWS CLoudFormation / CloudFormation을 사용하여 NAT Instance 를 생성해보자"
last_modified_at: 2020-01-17T01:00:00
---

## NAT Instance 구성하기

VPC 내에 존재하는 리소스들을 외부 인터넷과 연결하려면 Internet gateway 가 필요합니다. VPC 내의 Public Subnet에 위치한 리소스들은 이를 통해 외부와 연결할 수 있습니다.

단, Private Subnet에 위치한 리소스들은 외부와 연결 될 수 없습니다. 이를 가능하게 하기 위해서는 여러가지 방법이 존재합니다. 그중 NAT Gateway를 설정하는 것이 가장 간편하게 할 수 있는 방법입니다만 그 가격이 너무 비싸므로 NAT Instance 즉, EC2를 생성하여 이를 대체하고자 합니다.

![image](https://user-images.githubusercontent.com/17868599/70904095-e54abe80-2043-11ea-94a3-a1dd517a3e06.png)

위는 현재(2020-01-17) 기준으로 wedev 프로젝트에서 사용되는 리소스들의 구조도 입니다.

데이터베이스들과 Lambda를 간단한 EC2 Instance로 대체하여 이를 CloudFormation 으로 생성한 후, Public Subnet에 있는 NAT Instance 및 Bastion 에 SSH 접속을 하고, Bastion 에서 Private Subnet에 존재하는 EC2 Instance에 접속하여 `https://www.google.com` 으로 ping 테스트를 하여 외부 네트워크와 연결되는지 확인해보도록 하겠습니다.

아래에 주어진 순서대로 리소스들을 생성해보도록 하겠습니다.

1. 파라미터 및 맵핑 설정하기
2. VPC 구성하기
3. Public Subnet 만들기
4. Private Subnet 만들기
5. NAT Instance 및 Bastion 생성
6. Internet Gateway 만들기
7. 라우팅 테이블 만들기
8. 라우팅 편집

### 파라미터 및 맵핑 설정하기

#### 파라미터

- 현재 환경 이름

```yaml
EnvironmentName:
  Description: An environment name that is prefixed to resource names
  Type: String
```

- EC2 Instance 생성에 사용할 KeyPair

```yaml
KeyName:
  Description: Name of an existing EC2 KeyPair to enable SSH access to the instance
  Type: AWS::EC2::KeyPair::KeyName
  ConstraintDescription: must be the name of an existing EC2 KeyPair
```

- EC2 Instance type

```yaml
InstanceType:
  Description: NAT Instance
  Type: String
  Default: t2.micro
  AllowedValues:
    [
      t2.nano,
      t2.micro,
      t2.small,
      t2.medium,
      t2.large,
      t2.xlarge,
      t2.2xlarge,
      t3.nano,
      t3.micro,
      t3.small,
      t3.medium,
      t3.large,
      t3.xlarge,
      t3.2xlarge,
      m4.large,
      m4.xlarge,
      m4.2xlarge,
      m4.4xlarge,
      m4.10xlarge,
      m5.large,
      m5.xlarge,
      m5.2xlarge,
      m5.4xlarge,
      c5.large,
      c5.xlarge,
      c5.2xlarge,
      c5.4xlarge,
      c5.9xlarge,
      g3.8xlarge,
      r5.large,
      r5.xlarge,
      r5.2xlarge,
      r5.4xlarge,
      r3.12xlarge,
      i3.xlarge,
      i3.2xlarge,
      i3.4xlarge,
      i3.8xlarge,
      d2.xlarge,
      d2.2xlarge,
      d2.4xlarge,
      d2.8xlarge,
    ]
  ConstraintDescription: must be a valid EC2 instance type
```

### VPC 구성하기

- SSH 연결을 허용할 CIDR Block

```yaml
SSHLocation:
  Description: The IP address range that can be used to SSH to the EC2 instances
  Type: String
  MinLength: 9
  MaxLength: 18
  Default: 0.0.0.0/0
  AllowedPattern: (\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})/(\d{1,2})
  ConstraintDescription: must be a valid IP CIDR range of the form x.x.x.x/x
```

- VPC CIDR Block

```yaml
VpcCidrBlock:
  Description: The IP address range
  Type: String
  MinLength: 9
  MaxLength: 18
  Default: 10.192.0.0/16
  AllowedPattern: (\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})/(\d{1,2})
  ConstraintDescription: must be a valid IP CIDR range of the form x.x.x.x/x
```

- Private Subnet CIDR Blocks

```yaml
PrivateSubnet1CIDR:
  Description: Please enter the IP range (CIDR notation) for the private subnet in the first Availability Zone
  Type: String
  MinLength: 9
  MaxLength: 18
  Default: 10.192.10.0/24
  AllowedPattern: (\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})/(\d{1,2})
  ConstraintDescription: must be a valid IP CIDR range of the form x.x.x.x/x

PrivateSubnet2CIDR:
  Description: Please enter the IP range (CIDR notation) for the private subnet in the second Availability Zone
  Type: String
  MinLength: 9
  MaxLength: 18
  Default: 10.192.11.0/24
  AllowedPattern: (\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})/(\d{1,2})
  ConstraintDescription: must be a valid IP CIDR range of the form x.x.x.x/x

PrivateSubnet3CIDR:
  Description: Please enter the IP range (CIDR notation) for the private subnet in the third Availability Zone
  Type: String
  MinLength: 9
  MaxLength: 18
  Default: 10.192.12.0/24
  AllowedPattern: (\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})/(\d{1,2})
  ConstraintDescription: must be a valid IP CIDR range of the form x.x.x.x/x
```

- Public Subnet CIDR Block

```yaml
PublicSubnet1CIDR:
  Description: Please enter the IP range (CIDR notation) for the public subnet in the first Availability Zone
  Type: String
  MinLength: 9
  MaxLength: 18
  Default: 10.192.20.0/24
  AllowedPattern: (\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})/(\d{1,2})
  ConstraintDescription: must be a valid IP CIDR range of the form x.x.x.x/x
```

- Bastion SSH 연결 허용할 CIDR Block

```yaml
ServerAccess:
  Description: CIDR IP range allowed to login to the NAT instance
  Type: String
  MinLength: 9
  MaxLength: 18
  Default: 0.0.0.0/0
  AllowedPattern: (\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})/(\d{1,2})
  ConstraintDescription: must be a valid IP CIDR range of the form x.x.x.x/x
```

#### 맵핑

- EC2 Instance Image ID

```yaml
NatRegionMap:
  us-east-1:
    "AMI": "ami-184dc970"
  us-west-1:
    "AMI": "ami-a98396ec"
  us-west-2:
    "AMI": "ami-290f4119"
  eu-west-1:
    "AMI": "ami-14913f63"
  eu-central-1:
    "AMI": "ami-ae380eb3"
  sa-east-1:
    "AMI": "ami-8122969c"
  ap-southeast-1:
    "AMI": "ami-6aa38238"
  ap-southeast-2:
    "AMI": "ami-893f53b3"
  ap-northeast-1:
    "AMI": "ami-00d29e4cb217ae06b"
  ap-northeast-2:
    "AMI": "ami-0185fd13b4270de70"
```

### VPC 구성하기

```yaml
VPC:
  Type: AWS::EC2::VPC
  Properties:
    CidrBlock: !Ref VpcCidrBlock
    EnableDnsSupport: true
    EnableDnsHostnames: true
    InstanceTenancy: default
    Tags:
      - Key: Application
        Value: !Ref AWS::StackName
      - Key: Network
        Value: Public
      - Key: Name
        Value: !Sub ${EnvironmentName} VPC
```

### Public Subnet 만들기

```yaml
PublicSubnet1:
  Type: AWS::EC2::Subnet
  Properties:
    AvailabilityZone: !Select
      - 0
      - !GetAZs
        Ref: AWS::Region
    VpcId: !Ref VPC
    CidrBlock: !Ref PublicSubnet1CIDR
    MapPublicIpOnLaunch: true
    Tags:
      - Key: Application
        Value: !Ref AWS::StackName
      - Key: Network
        Value: Public
      - Key: Name
        Value: !Sub ${EnvironmentName} Public Subnet (AZ1)
```

### Private Subnet 만들기

### NAT Instance 및 Bastion 생성

### Internet Gateway 만들기

### 라우팅 테이블 만들기

### 라우팅 편집

## 전체 코드

- [저장소](https://github.com/Atercatus/AWS-Cloudformation-examples/blob/master/EC2%20Examples/NAT-instance.yaml)
