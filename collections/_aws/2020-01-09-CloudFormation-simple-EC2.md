---
title: "AWS CloudFormation 으로 EC2 instance 생성"
excerpt: "AWS CloutFormation 으로 EC2 instance 를 생성해보자"
last_modified_at: 2020-01-09T01:00:00
---

## CloudFormation 으로 EC2 Instance 생성하기

이번에는 간단하게 보안그룹 생성 및 EC2 인스턴스를 생성해보겠습니다.

### `Parameters` 설정

```yaml
KeyName:
  Description: Name of an existing EC2 KeyPair to enable SSH access to the instance
  Type: AWS::EC2::KeyPair::KeyName # AWS 고유 파라미터 유형, Amazon EC2 키 페어 이름.
  ConstraintDescription: must be the name of an existing EC2 KeyPair.
InstanceType:
  Description: WebServer EC2 instance type
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
  ConstraintDescription: must be a valid EC2 instance type.
SSHLocation:
  Description: The IP address range that can be used to SSH to the EC2 instances
  Type: String
  MinLength: 9
  MaxLength: 18
  Default: 0.0.0.0/0
  AllowedPattern: (\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})/(\d{1,2})
  ConstraintDescription: must be a valid IP CIDR range of the form x.x.x.x/x.
LatestAmiId:
  Type: "AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>"
  Default: "/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2"
```

- `KeyName`

  - EC2 Instance 생성에 사용될 KeyPair의 이름입니다.

- `InstanceType`

  - EC2 Instance 의 타입입니다.

- `SSHLocation`

  - 보안 그룹 생성 때 사용할 SSH 연결 허가 CIDR Block 입니다.

- `LatestAmiId`
  - AMI ID 입니다.

### EC2 Instance 생성

```yaml
EC2Instance:
  Type: AWS::EC2::Instance
  Properties:
    InstanceType: !Ref "InstanceType"
    SecurityGroups: [!Ref "InstanceSecurityGroup"]
    KeyName: !Ref "KeyName"
    ImageId: !Ref "LatestAmiId"
```

### 보안 그룹 생성

```yaml
InstanceSecurityGroup:
  Type: AWS::EC2::SecurityGroup
  Properties:
    GroupDescription: Enable SSH access via port 22
    SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 22
        ToPort: 22
        CidrIp: !Ref "SSHLocation"
```

## 전체 코드

- [저장소](https://github.com/Atercatus/AWS-Cloudformation-examples/blob/master/EC2%20Examples/EC2InstanceWithSecurityGroupSample.yaml)
