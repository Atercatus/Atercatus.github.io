---
title: "AWS CLI로 VPC 구성하기"
excerpt: "AWS CLI를 이용하여 VPC 를 구성해보자"
last_modified_at: 2020-01-03T01:00:00
---

## AWS CLI로 VPC 구성하기

### VPC

VPC(Virtual Private Cloud)에서는 사용자가 정의한 가장 네트워크로 AWS 리소스를 시작할 수 있습니다. 이 가상 네트워크는 AWS 의 확장 가능한 인프라를 사용한다는 이점과 함께 고객의 자체 데이터 센터에서 운영하는 기존 네트워크와 매우 유사합니다.

Amazon VPC는 Amazon EC2의 네트워크 계층입니다. 아래는 VPC의 핵심 개념입니다.

- VPC는 사용자의 AWS 계정 전용 가상 네트워크입니다.
- 서브넷은 VPC의 IP 주소 범위 입니다.
- 라우팅 테이블에는 네트워크 트래픽을 전달할 위치를 결정하는데 사용되는 라우팅이라는 규칙 집합이 포함되어 있습니다.
- 인터넷 게이트웨이는 수평 확장되고 가용성이 높은 중복 VPC 구성 요소로, VPC의 인스턴스와 인터넷 간에 통신할 수 있게 해줍니다. 따라서 네트워크 트래픽에 가용성 위험이나 대역폭 제약 조건이 발생하지 않습니다.
- VPC 엔드포인트를 통해 인터넷 게이트웨이, NAT 디바이스, VPN 연결 또는 AWS Direct Connect 연결을 필요로 하지 않고 PrivateLink 구동 지원 AWS 서비스 및 VPC 엔드포인트 서비스에 비공개로 연결할 수 있습니다.
- VPC의 인스턴스는 서비스의 리소스와 통신하는 데 퍼블릭 IP 주소를 필요로 하지 않습니다. VPC와 기타 서비스 간의 트래픽은 Amazon 네트워크를 벗어나지 않습니다.

### VPC 조회

VPC에 대한 조회는 `aws ec2 describe-vpc`로 수행할 수 있습니다. 필터를 사용하여 특정 VPC 조회가 가능합니다.

- 기본 VPC 조회

```
aws ec2 describe-vpcs --filter Name=isDefault,Values=true
```

- 실행결과

```
{
    "Vpcs": [
        {
            "CidrBlock": "172.31.0.0/16",
            "DhcpOptionsId": "dopt-ef79b684",
            "State": "available",
            "VpcId": "vpc-01234567",
            "OwnerId": "<OwenerId>",
            "InstanceTenancy": "default",
            "CidrBlockAssociationSet": [
                {
                    "AssociationId": "vpc-cidr-assoc-01234567",
                    "CidrBlock": "172.31.0.0/16",
                    "CidrBlockState": {
                        "State": "associated"
                    }
                }
            ],
            "IsDefault": true
        }
    ]
}
```

IsDefault 값으로 현재 조회된 VPC가 기본 VPC임을 알 수 있습니다.

> 주의: 결과는 IsDefault 로 나오지만 `--filer`를 이용해 검색 할 시에는 isDefault로 첫글자를 소문자로 작성해야합니다. // `--filter Name=isDefault,Values=true`

### Subnet 조회

서브넷은 ec2 describe-subnets 명령어로 조회가 가능합니다.

- VPC ID로 필터링하여 조회

```
aws ec2 describe-subnets --filter Name=vpcId,Values=vpc-12341234
```

- 실행결과

```
{
    "Subnets": [
        {
            "AvailabilityZone": "ap-northeast-2c",
            "AvailabilityZoneId": "apne2-az3",
            "AvailableIpAddressCount": 251,
            "CidrBlock": "10.0.0.0/24",
            "DefaultForAz": false,
            "MapPublicIpOnLaunch": false,
            "State": "available",
            "SubnetId": "subnet-12341234",
            "VpcId": "vpc-12341234",
            "OwnerId": "12341234",
            "AssignIpv6AddressOnCreation": false,
            "Ipv6CidrBlockAssociationSet": [],
            "SubnetArn": "arn:aws:ec2:ap-northeast-2:12341234:subnet/subnet-12341234"
        },
        {
            "AvailabilityZone": "ap-northeast-2c",
            "AvailabilityZoneId": "apne2-az3",
            "AvailableIpAddressCount": 251,
            "CidrBlock": "10.0.1.0/24",
            "DefaultForAz": false,
            "MapPublicIpOnLaunch": false,
            "State": "available",
            "SubnetId": "subnet-12341234",
            "VpcId": "vpc-12341234",
            "OwnerId": "12341234",
            "AssignIpv6AddressOnCreation": false,
            "Ipv6CidrBlockAssociationSet": [],
            "SubnetArn": "arn:aws:ec2:ap-northeast-2:12341234:subnet/subnet-12341234"
        }
    ]
}
```

### Security group

보안그룹은 EC2 대시보드와 VPC 대시보드 둘 다 존재합니다. 이 둘은 같습니다. 시큐리피 그룹은 VPC에 속해있습니다. 따라서 다수의 VPC가 정의되어 있는 경우 같은 조건이어도 VPC 별로 다른 보안 그룹을 생성해야합니다.
보안 그룹은 `ec2 describe-security-group` 로 조회 할 수 있습니다.

- 보안 그룹 조회

```
aws ec2 describe-security-groups --filter Name=vpc-id,Values=vpc-12341234
```

- 실행 결과

```
{
    "SecurityGroups": [
        {
            "Description": "default VPC security group",
            "GroupName": "default",
            "IpPermissions": [
                {
                    "IpProtocol": "-1",
                    "IpRanges": [],
                    "Ipv6Ranges": [],
                    "PrefixListIds": [],
                    "UserIdGroupPairs": [
                        {
                            "GroupId": "sg-1234",
                            "UserId": "1234"
                        }
                    ]
                }
            ],
            "OwnerId": "12341234",
            "GroupId": "sg-12341234",
            "IpPermissionsEgress": [
                {
                    "IpProtocol": "-1",
                    "IpRanges": [
                        {
                            "CidrIp": "0.0.0.0/0"
                        }
                    ],
                    "Ipv6Ranges": [],
                    "PrefixListIds": [],
                    "UserIdGroupPairs": []
                }
            ],
            "VpcId": "vpc-12341234"
        }
    ]
}
```

#### IpPermissionsEgress

아웃바운드 규칙 목록을 의미합니다. 이 규칙들은 인스턴스에서 외부로 나가는 트래픽을 모든 포트(`"IpProtocol": "-1"`)와 모든 IP 에 대해서(`"CidrIp": "0.0.0.0/0"`) 허용함을 의미합니다.

#### IpPermissions

인바운드 규칙 목록을 의미합니다. 이 규칙들은 외부에서 이 보안 그룹을 가진 리소스에 접근할 때 사용되는 규칙입니다. 모든 포트("IpProtocol": "-1")에 대해서 `"sg-1234"` 의 보안그룹을 가지는 리소스의 트래픽만을 허용한 다는 것을 의미합니다.

## AWS CLI를 사용하여 IPv4 VPC 및 서브넷 생성

아래의 예제는 [AWS 사용 설명서](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/vpc-subnets-commands-example.html) 를 옮긴 것 입니다.

### VPC와 서브넷 만들기

1. CIDR 블록이 `10.0.0.0/16`인 VPC 를 만듭니다.

```
aws ec2 create-vpc --cidr-block 10.0.0.0/16
```

2. 이전 단계에서 생성된 VPC ID를 사용하여 CIDR 블록이 `10.0.1.0/24`인 서브넷을 만듭니다.

```
aws ec2 create-subnet --vpc-id vpc-2f09a348 --cidr-block 10.0.1.0/24
```

3. CIDR 블록이 `10.0.0.0/24`인 VPC에 두 번째 서브넷을 만듭니다.

```
aws ec2 create-subnet --vpc-id vpc-2f09a348 --cidr-block 10.0.0.0/24
```

### 서브넷을 퍼블릭으로 만들기

VPC와 서브넷을 만든 후, VPC에 인터넷 게이트웨이를 연결하고, 사용자 지정 라우팅 테이블을 만들고, 서브넷이 인터넷 게이트웨이로 라우팅되로록 구성하여 서브넷 중 하나를 퍼블릭 서브넷으로 만들 수 있습니다.

**서브넷을 퍼블릭 서브넷으로 만들려면**

1. 인터넷 게이트웨이 생성.

```
aws ec2 create-internet-gateway
```

반환된 출력에 표시된 인터넷 게이트웨이 ID를 메모해둡니다.

```
{
    "InternetGateway": {
        "Attachments": [],
        "InternetGatewayId": "igw-1ff7a07b",
        "Tags": []
    }
}
```

2. 이전 단계에서 메모해 둔 ID를 사용하여 VPC에 인터넷 게이트웨이를 연결합니다.

```
aws ec2 attach-internet-gateway --vpc-id vpc-2f09a348 --internet-gateway-id igw-1ff7a07b
```

3. VPC에 대해 사용자 지정 라우팅 테이블을 만듭니다.

```
aws ec2 create-route-table --vpc-id vpc-2f09a348
```

반환된 출력에 표시된 라우팅 테이블 ID를 메모해 둡니다.

```
{
    "RouteTable": {
        "Associations": [],
        "PropagatingVgws": [],
        "RouteTableId": "rtb-c1c8faa6",
        "Routes": [
            {
                "DestinationCidrBlock": "10.0.0.0/16",
                "GatewayId": "local",
                "Origin": "CreateRouteTable",
                "State": "active"
            }
        ],
        "Tags": [],
        "VpcId": "vpc-2f09a348",
        "OwnerId": ""
    }
}
```

4. 라우팅 테이블에 모든 트래픽(`0.0.0.0/0`) 이 인터넷 게이트웨이를 가리키는 경로를 만듭니다.

```
aws ec2 create-route --route-table-id rtb-c1c8faa6 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-1ff7a07b
```

5. 경로가 만들어졌고 활성 상태인지 확인하기 위해, 라우팅 테이블을 설명하고 그 결과를 볼 수 있습니다.

```
aws ec2 describe-route-tables --route-table-id rtb-c1c8faa6
```

```
{
    "RouteTables": [
        {
            "Associations": [],
            "RouteTableId": "rtb-c1c8faa6",
            "VpcId": "vpc-2f09a348",
            "PropagatingVgws": [],
            "Tags": [],
            "Routes": [
                {
                    "GatewayId": "local",
                    "DestinationCidrBlock": "10.0.0.0/16",
                    "State": "active",
                    "Origin": "CreateRouteTable"
                },
                {
                    "GatewayId": "igw-1ff7a07b",
                    "DestinationCidrBlock": "0.0.0.0/0",
                    "State": "active",
                    "Origin": "CreateRoute"
                }
            ]
        }
    ]
}
```

6. 라우팅 테이블이 현재 서브넷과 연결되지 않았습니다. 서브넷에서 보내는 트래픽이 인터넷 게이트웨이로 라우팅되도록 라우팅 테이블을 VPC의 해당 서브넷과 연결해야 합니다. 먼저 `describe-subnets` 명령을 사용하여 서브넷 ID를 가져옵니다. `--filter` 옵션을 사용하여 새 VPC의 서브넷만 반환하고, `--query` 옵션을 사용하여 서브넷 ID와 그 CIDR 블록만 반환할 수 있습니다.

```
aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-2f09a348" --query 'Subnets[*].{ID:SubnetId,CIDR:CidrBlock}'
```

```
[
    {
        "CIDR": "10.0.1.0/24",
        "ID": "subnet-b46032ec"
    },
    {
        "CIDR": "10.0.0.0/24",
        "ID": "subnet-a46032fc"
    }
]

```

7. 사용자 지정 라우팅 테이블과 연결할 서브넷을 선택할 수 있습니다(예: `subnet-b46032ec)`. 이 서브넷이 퍼블릭 서브넷이 됩니다.

```
aws ec2 associate-route-table  --subnet-id subnet-b46032ec --route-table-id rtb-c1c8faa6
```

8. 또는 서브넷에서 시작된 인스턴스가 퍼블릭 IP 주소를 자동으로 받도록 서브넷의 퍼블릭 IP 주소 지정 동작을 수정할 수 있습니다. 그렇지 않은 경우, 시작 후 인터넷에서 연결할 수 있도록 탄력적 IP 주소를 인스턴스와 연결해야 합니다.

```
aws ec2 modify-subnet-attribute --subnet-id subnet-b46032ec --map-public-ip-on-launch
```

### 서브넷에서 인스턴스 시작

서브넷이 퍼블릭이고 인터넷을 통해 해당 서브넷의 인스턴스에 액세스할 수 있는지 테스트하려면, 먼저 인스턴스와 연결할 보안 그룹과 인스턴스에 연결하는 데 사용할 키 페어를 만들어야 합니다. 보안 그룹에 대한 자세한 내용은 [VPC의 보안그룹](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/VPC_SecurityGroups.html) 단원을 참조하십시오. 키 페어에 대한 자세한 내용은 Linux 인스턴스용 Amazon EC2 사용 설명서의 [Amazon EC2 키 페어](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)를 참조하십시오.

**퍼블릭 서브넷에서 인스턴스를 시작하고 연결하려면**

1. 키 페어를 만든 다음 `--query` 옵션과 `--output` 텍스트 옵션을 사용하여 확장자가 `.pem`인 파일에 직접 프라이빗 키를 파이프합니다.

```
aws ec2 create-key-pair --key-name MyKeyPair --query 'KeyMaterial' --output text > MyKeyPair.pem
```

> 만약 PowerShell 을 사용할 경우 `> file` 이 기본적으로 UTF-8 인코딩을 수행하므로 SSH client에서 사용할 수 없습니다. 따라서 output을 변경해 주어야합니다. `out-file` 명령을 통해 output 을 변경하고 `ascii`를 통해 명확한 인코딩을 지정해 줍니다
>
> ```
> aws ec2 create-key-pair --key-name MyKeyPair --query 'KeyMaterial' --output text | out-file -encoding ascii -filepath MyKeyPair.pem
> ```

이 예에서는 Amazon Linux 인스턴스를 시작합니다. Linux 또는 Mac OS X 운영 체제에서 SSH 클라이언트를 사용하여 인스턴스에 연결하려면, 사용자만 프라이빗 키 파일을 읽을 수 있도록 다음 명령으로 해당 권한을 설정합니다.

```
chmod 400 MyKeyPair.pem
```

2. VPC에 보안 그룹을 만든 다음, 어디서나 SSH 액세스를 허용하는 규칙을 추가합니다.

```
aws ec2 create-security-group --group-name SSHAccess --description "Security group for SSH access" --vpc-id vpc-2f09a348
```

```
{
    "GroupId": "sg-e1fb8c9a"
}
```

```
aws ec2 authorize-security-group-ingress --group-id sg-e1fb8c9a --protocol tcp --port 22 --cidr 0.0.0.0/0
```

> 0.0.0.0/0을 사용하면 모든 IPv4 주소에서 SSH를 사용하여 인스턴스에 액세스할 수 있습니다. 예제에서 잠시 사용하는 것은 괜찮지만 프로덕션 환경에서는 특정 IP 주소 또는 주소 범위에 대해서만 승인하십시오.

3. 생성한 보안 그룹과 키 페어를 사용하여 퍼블릭 서브넷에서 인스턴스를 시작합니다. 출력에 표시된 인스턴스의 인스턴스 ID를 메모해 둡니다.

```
aws ec2 run-instances --image-id ami-a4827dc9 --count 1 --instance-type t2.micro --key-name MyKeyPair --security-group-ids sg-e1fb8c9a --subnet-id subnet-b46032ec
```

> 이 예에서 AMI는 미국 동부(버지니아 북부) 리전의 Amazon Linux AMI입니다. 다른 리전에 있는 경우, 해당 리전에 적합한 AMI의 AMI ID가 필요합니다. 자세한 내용은 Linux 인스턴스용 Amazon EC2 사용 설명서의 [Linux AMI 찾기](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/finding-an-ami.html) 단원을 참조하십시오.

4. 인스턴스에 연결하려면 인스턴스가 `running` 상태에 있어야 합니다. 인스턴스를 설명하고 인스턴스의 상태를 확인한 다음, 해당 퍼블릭 IP 주소를 메모해 둡니다.

```
aws ec2 describe-instances --instance-id i-0146854b7443af453
```

```
{
    "Reservations": [
        {
            ...
            "Instances": [
                {
                    ...
                    "State": {
                        "Code": 16,
                        "Name": "running"
                    },
                    ...
                    "PublicIpAddress": "52.87.168.235",
                    ...
                }
            ]
        }
    ]
}
```

5. 인스턴스가 실행 상태에 있는 경우, 다음 명령을 사용하여 Linux 또는 Mac OS X 컴퓨터에서 SSH 클라이언트를 통해 인스턴스에 연결할 수 있습니다.

```
ssh -i "MyKeyPair.pem" ec2-user@52.87.168.235
```

### 정리

인스턴스에 연결할 수 있는지 확인한 후, 인스턴스가 더 이상 필요하지 않은 경우 종료할 수 있습니다. 이렇게 하려면 [terminate-instances](https://docs.aws.amazon.com/cli/latest/reference/ec2/terminate-instances.html) 명령을 사용합니다. 이 예에서 만든 리소스를 삭제하려면 다음 명령을 나열된 순서대로 사용합니다.

1. 보안 그룹 삭제:

```
aws ec2 delete-security-group --group-id sg-e1fb8c9a

```

2. 서브넷 삭제:

```
aws ec2 delete-subnet --subnet-id subnet-b46032ec
```

3. 사용자 지정 라우팅 테이블 삭제:

```
aws ec2 delete-route-table --route-table-id rtb-c1c8faa6
```

4. VPC에서 인터넷 게이트웨이 분리:

```
aws ec2 detach-internet-gateway --internet-gateway-id igw-1ff7a07b --vpc-id vpc-2f09a348
```

5. 인터넷 게이트웨이 삭제:

```
aws ec2 delete-internet-gateway --internet-gateway-id igw-1ff7a07b
```

6. VPC 삭제:

```
aws ec2 delete-vpc --vpc-id vpc-2f09a348
```

## 외부 API Call 확인해보기

### Nodejs 설치

- [Amazon linux 에 Nodejs 설치](https://docs.aws.amazon.com/ko_kr/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html)

### HTTPS 요청

Node.js 의 https 모듈을 이용해 네이버페이지를 요청한 다음 반환하는 서버를 만들어 봤습니다. 아래의 코드를 통해 서버를 실행할 경우, 클라이언트에서 이 서버에 요청을 하면 네이버 페이지가 나옵니다.

```javascript
const express = require("express");
const app = express();
const https = require("https");

const PORT = 4000;

const options = {
  host: "www.naver.com",
  path: "/"
};

function httpRequest() {
  return new Promise((resolve, reject) => {
    const req = https.get("https://www.naver.com", res => {
      let body = "";

      res.on("data", chunk => {
        body += chunk;
      });
      res.on("end", () => {
        resolve(body);
      });
    });

    req.end();
  });
}

const callback = res => {
  let body = "";
  res.on("data", data => {
    body += data;
  });

  res.on("end", () => {
    console.log(body);
    return body;
  });
};

app.get("/", async (req, res) => {
  const ret = await httpRequest(options);

  console.log(ret);

  res.send(ret);
});
```

## 참조 링크

- [AWS 계정 관리](https://www.44bits.io/ko/post/publishing_and_managing_aws_user_access_key)

- [AWS CLI 기초](https://www.44bits.io/ko/post/aws_command_line_interface_basic)

- [AWS CLI 로 VPC 만들기](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/vpc-subnets-commands-example.html)

- [AWS CLI 매뉴얼](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/aws-cli.pdf)

- [AWS CLI 를 이용해 VPC 탐색하기](https://www.44bits.io/ko/post/inspect_resources_in_default_vpc_by_aws_cli)

- [AWS VPC](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/what-is-amazon-vpc.html)
