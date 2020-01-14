---
title: "AWS CloudFormation - 1"
excerpt: "AWS CloutFormation 의 개념 및 작동방식"
last_modified_at: 2020-01-08T01:00:00
---

해당 글은 [AWS CloudFormation 사용설명서](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html)를 옮긴 것 입니다.

## CloudFormation 의 개념

템플릿 및 스택을 생성하여 AWS 리소스와 해당 속성에 대해 명세합니다. AWS CloudFormation에서 스택을 생성할 때마다 템플릿에 명세된 리소스를 프로비저닝 합니다.

### 템플릿

JSON 또는 YAML 형식의 텍스트 파일입니다. AWS 리소스 구축을 위한 블루프린트로 사용합니다.

- JSON Example

```
{
  "AWSTemplateFormatVersion" : "2010-09-09",
  "Description" : "A sample template",
  "Resources" : {
    "MyEC2Instance" : {
      "Type" : "AWS::EC2::Instance",
      "Properties" : {
        "ImageId" : "ami-0ff8a91507f77f867",
        "InstanceType" : "t2.micro",
        "KeyName" : "testkey",
        "BlockDeviceMappings" : [
          {
            "DeviceName" : "/dev/sdm",
            "Ebs" : {
              "VolumeType" : "io1",
              "Iops" : "200",
              "DeleteOnTermination" : "false",
              "VolumeSize" : "20"
            }
          }
        ]
      }
    }
  }
}
```

단일 템플릿에서 여러 리소스를 지정하고 해당 리소스를 함께 작동하도록 구성할 수도 있습니다. 예를 들어, 다음 예제와 같이 탄력적 IP(EIP)를 포함하도록 이전 템플릿을 수정한 후 Amazon EC2 인스턴스와 연결할 수 있습니다.

- JSON Example

```
{
  "AWSTemplateFormatVersion" : "2010-09-09",
  "Description" : "A sample template",
  "Resources" : {
    "MyEC2Instance" : {
      "Type" : "AWS::EC2::Instance",
      "Properties" : {
        "ImageId" : "ami-0ff8a91507f77f867",
        "InstanceType" : "t2.micro",
        "KeyName" : "testkey",
        "BlockDeviceMappings" : [
          {
            "DeviceName" : "/dev/sdm",
            "Ebs" : {
              "VolumeType" : "io1",
              "Iops" : "200",
              "DeleteOnTermination" : "false",
              "VolumeSize" : "20"
            }
          }
        ]
      }
    },
    "MyEIP" : {
      "Type" : "AWS::EC2::EIP",
      "Properties" : {
        "InstanceId" : {"Ref": "MyEC2Instance"}
      }
    }
  }
}
```

### 스택

AWS CloudFormation을 사용할 경우 스택이라는 하나의 단위로 관련 리소스를 관리합니다. 리소스를 생성하려면 생성한 템플릿을 제출하여 스택을 생성합니다. 그러면 AWS CloudFormation에서 모든 리소스를 자동으로 프로비저닝합니다. 콘솔, API 또는 AWS CLI를 사용하여 스택으로 작업할 수 있습니다.

### 변경 세트

스택에서 실행 중인 리소스를 변경해야 하는 경우 스택을 업데이트 합니다. 리소스를 변경하기 전에 제안된 변경 사항이 요약된 변경 세트를 생성할 수 있습니다. 변경 세트를 사용하면 변경 사항을 구현하기 이전에 해당 변경이 실행 중인 리소스 특히, 중요 리소스에 미치는 영향을 확인할 수 있습니다.

## AWS CloudFormation 작동 방식

스택을 생성할 때 AWS CloudFormation 에서는 AWS에 대한 기본 서비스 호출을 실행해 리소스를 프로비저닝 및 구성합니다. AWS CloudFormation 에서는 수행 권한이 있는 작업만 수행할 수 있습니다. 이는 AWS Identity and Access Management(IAM)를 사용하여 관리합니다.

![](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/images/create-stack-diagram.png)

1. AWS CloudFormation 템플릿(JSON 또는 YAML 형식 문서)은 AWS CloudFormation Designer에서 디자인하거나 텍스트 편집기에서 작성할 수 있습니다. 또한 제공된 템플릿을 사용하도록 선택할 수도 있습니다. 이 템플릿에서는 필요한 리소스와 해당 설정을 설명합니다.

   - JSON Example

   ```
   {
     "AWSTemplateFormatVersion" : "2010-09-09",
     "Description" : "A simple EC2 instance",
     "Resources" : {
       "MyEC2Instance" : {
         "Type" : "AWS::EC2::Instance",
         "Properties" : {
           "ImageId" : "ami-0ff8a91507f77f867",
           "InstanceType" : "t1.micro"
         }
       }
     }
   }
   ```

2. 템플릿을 로컬에 또는 S3 버킷에 저장합니다. 템플릿을 생성한 경우 `.json`,`.yaml` 또는 `.txt` 등과 같은 파일 확장명으로 저장합니다.

3. 템플릿 파일의 위치(예: 로컬 컴퓨터의 경로 또는 Amazon S3 URL)를 지정하여 AWS CloudFormation 스택을 생성합니다. 템플릿에 파라미터가 포함되어 있는 경우 스택을 생성할 때 입력값을 지정할 수 있습니다.
   AWS CloudFormation 콘솔, API 또는 AWS CLI를 사용하여 스택을 생성할 수 있습니다.

AWS CloudFormation에서는 템플릿에 설명된 AWS 서비스를 호출하여 리소스를 프로비저닝 및 구성합니다.

모든 리소스가 생성된 후 AWS CloudFormation에서는 스택이 생성되었음을 보고합니다. 그러면 스택의 리소스를 사용할 수 있습니다. 스택 생성에 실패하면 AWS CloudFormation에서는 생성한 리소스를 삭제하여 변경 사항을 롤백합니다.

### 변경 세트로 스택 업데이트

스택의 리소스를 업데이트해야 하는 경우 스택의 템플릿을 수정할 수 있습니다. 새 스택을 만들고 이전 스택은 삭제할 필요가 없습니다. 스택을 업데이트하려면 원본 스택 템플릿의 수정된 버전, 다른 입력파라미터 또는 둘 다를 제출하여 변경세트를 생성합니다. AWS CloudFormation에서는 수정된 템플릿을 원본 템플릿과 비교해 변경 세트를 생성합니다. 변경 세트에는 제안된 변경 사항이 나열되어 있습니다. 변경 사항을 검토한 후 변경 세트를 실행해 스택을 업데이트하거나 새 변경 세트를 생성할 수 있습니다. 다음 다이어그램은 스택 업데이트 워크플로우를 요약해서 보여줍니다.

![](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/images/update-stack-diagram.png)

### 스택 삭제

스택을 삭제하는 경우 삭제할 스택을 지정하면 AWS CloudFormation에서는 해당 스택과 스택 내 모든 리소스를 삭제합니다. AWS CloudFormation 콘솔, API 또는 AWS CLI를 사용하여 스택을 삭제할 수 있습니다.

스택은 삭제하지만 해당 스택 내 일부 리소스는 보존하려는 경우 삭제 정책을 사용하여 해당 리소스는 보존할 수 있습니다.

## AWS Identity and Access Management을 통한 액세스 제어

AWS Identity and Access Management(IAM)를 사용하여 AWS 계정에서 어떤 사용자가 어느 리소스에 액세스 가능한지를 제어할 수 있는 IAM 사용자를 생성할 수 있습니다. IAM을 AWS CloudFormation과 함께 사용하여 사용자가 AWS CloudFormation에서 수행할 수 있는 작업(예: 스택 탬플릿을 보거나, 스택을 생성하거나 삭제할 수 있는지 여부)을 제어할 수 있습니다.

### AWS CloudFormation 작업

AWS 계정에서 그룹 또는 IAM 사용자를 생성할 경우 부여할 권한을 지정하는 IAM 정책을 해당 그룹 또는 사용자와 연결할 수 있습니다. 예를 들어, 엔트리 레벨 개발자의 그룹이 있다고 가정하겠습니다. 모든 엔트리 레벨 개발자를 포함하는 `Junior application developers` 그룹을 생성할 수 있습니다. 그런 다음 사용자에게 AWS CloudFormation 스택 보기만 허용하는 정책을 해당 그룹에 연결합니다.

- 스택 보기 권한을 부여하는 샘플 정책

```
{
    "Version":"2012-10-17",
    "Statement":[{
        "Effect":"Allow",
        "Action":[
            "cloudformation:DescribeStacks",
            "cloudformation:DescribeStackEvents",
            "cloudformation:DescribeStackResource",
            "cloudformation:DescribeStackResources"
        ],
        "Resource":"*"
    }]
}
```

### AWS CloudFormation 콘솔별 작업

AWS CloudFormation 콘솔을 사용하는 IAM 사용자는 AWS CLI 또는 AWS CloudFormation API를 사용하는 데 필요하지 않은 추가 권한이 필요합니다.

다음 필수 작업은 AWS CloudFormation 콘솔에서만 사용되며 API 참조에 명시되지 않습니다. 사용자가 작업을 통해 템플릿을 Amazon S3 버킷에 업로드할 수 있습니다.

```
cloudformation:CreateUploadBucket
```

템플릿을 업로드할 때 다음과 같은 Amazon S3 권한이 필요합니다.

```
s3:PutObject
s3:ListBucket
s3:GetObject
s3:CreateBucket
```

[AWS 특정 파라미터 유형](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/parameters-section-structure.html#aws-specific-parameter-types)을 가진 템플릿의 경우 해당 Describe API를 호출할 수 있는 권한이 필요합니다. 예를 들어, 템플릿에 `AWS::EC2::KeyPair::KeyName` 파라미터 유형이 포함되어 있는 경우 EC2 `DescribeKeyPairs` 작업을 호출할 수 있는 권한이 필요합니다. 이러한 방법으로 콘솔에서 파라미터 드롭다운 목록에 대한 값을 가져옵니다. 다음 예제에서는 다른 파라미터 유형에 필요한 작업을 보여줍니다.

```
ec2:DescribeSecurityGroups (for the AWS::EC2::SecurityGroup::Id parameter type)
ec2:DescribeSubnets (for the Subnet::Id parameter type)
ec2:DescribeVpcs (for the AWS::EC2::VPC::Id parameter type)
```

## AWS CloudTrail을 사용하여 AWS CloudFormation API 호출 로깅

AWS CloudFormation은 AWS CloudFormation의 사용자, 역할 또는 AWS 서비스가 수행한 작업에 대한 레코드를 제공하는 서비스인 AWS CloudTrail과 통합됩니다. CloudTrail은 AWS CloudFormation 콘솔의 호출 및 AWS CloudFormation API 코드 호출 등 AWS CloudFormation에 대한 모든 API 호출을 이벤트로 캡처합니다. 추적을 생성하면 CloudTrail 이벤트를 비롯하여 AWS CloudFormation 이벤트를 Amazon S3 버킷으로 지속적으로 배포할 수 있습니다. 추적을 구성하지 않은 경우 **Event history(이벤트 기록)**에서 CloudTrail 콘솔의 최신 이벤트를 볼 수도 있습니다. CloudTrail에서 수집하는 정보를 사용하여 AWS CloudFormation에 수행된 요청, 요청이 수행된 IP 주소, 요청을 수행한 사람, 요청이 수행된 시간 및 추가 세부 정보를 확인할 수 있습니다.

## AWS CloudFormation에 대한 VPC 엔드포인트 설정

인터페이스 VPC 엔드포인트를 사용하도록 AWS CloudFormation을 구성하여 VPC의 보안 태세를 향상시킬 수 있습니다. 인터페이스 엔드포인트는 프라이빗 IP 주소를 사용하여 AWS CloudFormation API에 비공개로 액세스할 수 있는 기술인 PrivateLink로 구동됩니다. PrivateLink는 VPC 및 AWS CloudFormation 간의 모든 네트워크 트래픽을 Amazon 네트워크로 제한합니다. 또한 인터넷 게이트웨이, NAT 디바이스 또는 가상 프라이빗 게이트웨이가 필요 없습니다.

---

## 참조 링크

-[AWS CloudFormation 사용설명서](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/cfn-whatis-concepts.html)
