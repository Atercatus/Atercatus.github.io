---
title: "How to deploy lambda using AWS CloudFormation"
excerpt: "AWS CloutFormation 으로 Lambda를 생성해서 Api Gateway로 배포해보자"
last_modified_at: 2020-01-17T01:00:00
---

## 개요

Lambda를 생성하여 배포하려면 크게 3가지의 흐름으로 이루어진다고 볼 수 있습니다.

1. Lambda Function 생성
2. API 생성
3. API 배포

## Lambda Function 생성

### Lambda Function 생성

```yaml
LambdaFunction:
  Type: AWS::Lambda::Function
  Properties:
    Code:
      ZipFile: | # Lambda 를 생성할 때 unzip 수행
        exports.myHandler = async event => {
          // TODO implement
          const response = {
            statusCode: 200,
            body: JSON.stringify("Hello from Lambda!")
          };
          return response;
        };

    Description: My Function
    FunctionName: !Ref LambdaName
    Handler: index.myHandler # filename.functionName
    MemorySize: 128
    Role: !GetAtt LambdaIAMRole.Arn
    Runtime: nodejs12.x
    Timeout: 300
```

- `ZipFile`

  - `Code` 에서 `ZipFile`을 하는 이유는 소스코드를 압축을 해야 Lambda를 생성할 때 이를 `UnZip` 할 수 있기 때문입니다.

- `Handler`

  - filename.functionName 의 형태로 Lambda가 함수를 실행하기 위해 호출하는 코드 내 메서드 이름입니다.

- `MemorySize`

  - 함수에서 액세스 할 수 있는 메모리의 양입니다. 함수의 메모리를 늘리면 CPU 할당도 늘어납니다. 기본값은 128MB 입니다. 값은 64MB의 배수여야 합니다.

- `Role`

  - 함수의 실행 역할의 Amazon 리소스 이름(ARN) 입니다. 위의 예제에서는 CloudWatch Logs 사용에 필요한 권한을 정의했습니다.
  - 이 정책에는 로그 그룹 및 로그 스트림을 생성하고 로그 스트림에 로그 이벤트를 업로드할 수 있는 권한을 부여하는 명령문이 하나 포함되어 있습니다.

- `Timeout`
  - Lambda가 함수를 중지하기 전에 실행을 허용하는 시간입니다. 기본값은 3초 이며 최소 1초에서 최대 900초 까지 가능합니다.

### 권한 생성

```yaml
LambdaIAMRole:
  Type: AWS::IAM::Role
  Properties:
    AssumeRolePolicyDocument:
      Version: 2012-10-17
      Statement:
        - Action:
            - sts:AssumeRole # Returns a set of temporary security credentials that you can use to access AWS resources that you might not normally have access to.
          Effect: Allow
          Principal: # specify the principal that is allowed or denied access to a resource.
            Service:
              - lambda.amazonaws.com
    Policies:
      - PolicyDocument:
          Version: 2012-10-17
          Statement:
            - Action:
                - logs:CreateLogGroup
                - logs:CreateLogStream
                - logs:PutLogEvents
                - logs:DescribeLogStreams
              Effect: Allow
              Resource:
                - !GetAtt LambdaLogGroup.Arn
        PolicyName: lambda
```

- [설명](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html)

- `AssumeRolePolicyDocument`

  - 이 역할과 연결된 신뢰 정책입니다. 어느 개체가 역할을 수임할 수 있는지 정의합니다.

- `Policies`
  - 지정된 IAM 역할에 포함된 인라인 정책 문서를 추가 또는 업데이트합니다. 위의 예제에서는 Amazon CloudWatch Logs 의 역할을 명세했습니다.

### Log Group 구성

```yaml
LambdaLogGroup:
  Type: AWS::Logs::LogGroup
  Properties:
    LogGroupName: !Sub /aws/lambda/${LambdaName}
    RetentionInDays: 90
```

## API 생성

### API Gateway 생성

```yaml
ApiGateway:
  Type: AWS::ApiGateway::RestApi # 프로토콜을 REST 로 설정합니다
  Properties:
    Name: my-api
```

### API Gateway Resource 생성

```yaml
ApiGatewayResource:
  Type: AWS::ApiGateway::Resource
  Properties:
    ParentId: !GetAtt ApiGateway.RootResourceId
    PathPart: "{proxy+}"
    RestApiId: !Ref ApiGateway
```

root(/) 경로에 lambda-proxy 리소스를 구성합니다. lambda-proxy 에 대한 자세한 정보는 [포스트](https://medium.com/@lakshmanLD/lambda-proxy-vs-lambda-integration-in-aws-api-gateway-3a9397af0e6d)를 참고해주세요.

### Method 생성

```yaml
ApiGatewayRootMethod:
  Type: AWS::ApiGateway::Method
  Properties:
    AuthorizationType: NONE
    HttpMethod: "GET"
    Integration:
      Type: AWS_PROXY
      IntegrationHttpMethod: POST
      Uri: !Sub
        - "arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${lambdaArn}/invocations"
        - lambdaArn: !GetAtt "LambdaFunction.Arn"
    ResourceId: !Ref ApiGatewayResource
    RestApiId: !Ref ApiGateway
```

- `Integration`

  - 요청을 받았을 때 호출될 백엔드 시스템을 정의합니다.

- `AWS_PROXY`

  - [타입 설명](https://docs.aws.amazon.com/apigateway/api-reference/resource/integration/)
  - for integrating the API method request with the Lambda function-invoking action with the client request passed through as-is. This integration is also referred to as the Lambda proxy integration.

- `IntegrationHttpMethod`

  - [`IntegrationHttpMethod` must be `POST`](https://stackoverflow.com/questions/55454744/how-to-trigger-aws-lambda-from-api-gateway-with-multiple-methods/55457240#55457240)

- `Uri`
  - Specifies Uniform Resource Identifier (URI) of the integration endpoin

## API 배포

### API Gateway Lambda 호출 권한 부여

```yaml
LambdaApiGatewayInvoke:
  Type: AWS::Lambda::Permission
  Properties:
    Action: lambda:InvokeFunction
    FunctionName: !GetAtt LambdaFunction.Arn
    Principal: apigateway.amazonaws.com
    SourceArn: !Sub arn:aws:execute-api:${AWS::Region}:${AWS::AccountId}:${ApiGateway}/*/GET/*
```

API Gateway 가 해당 Lambda를 호출 할 수 있는 권한을 부여해야 합니다.

### 배포

```yaml
ApiGatewayDeployment:
  Type: AWS::ApiGateway::Deployment
  DependsOn:
    - ApiGatewayRootMethod
  Properties:
    RestApiId: !Ref ApiGateway
    StageName: !Ref ApiGatewayStageName
```

## 전체 코드

- [소스 코드](https://github.com/Atercatus/AWS-Cloudformation-examples/tree/master/Lambda%20Examples/Simple%20Deploy)

## 참조 링크

- [TUTORIAL: Build a Hello World API with Lambda Proxy Integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-create-api-as-simple-proxy-for-lambda.html#api-gateway-proxy-integration-lambda-function-python)

- [Source](https://bl.ocks.org/magnetikonline/c314952045eee8e8375b82bc7ec68e88)

- [Understand API Gateway Lambda Proxy Integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html)

- [`IntegrationHttpMethod` must be `POST`](https://stackoverflow.com/questions/55454744/how-to-trigger-aws-lambda-from-api-gateway-with-multiple-methods/55457240#55457240)

- [AWS::Lambda::Permission](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-permission.html)

- [AWS::IAM::Role](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/aws-resource-iam-role.html)

- [CloudFormation: Invalid permissions on Lambda function](https://stackoverflow.com/questions/41817075/cloudformation-invalid-permissions-on-lambda-function)

- [단계 변수를 사용해 Amazon API Gateway의 AWS Lambda 통합을 정의했습니다. API에서 "내부 서버 오류"라는 메세지 500 상태 코드를 반환하는 이유는 무엇입니까?](https://aws.amazon.com/ko/premiumsupport/knowledge-center/500-error-lambda-api-gateway/)

- [Control Access for Invoking an API](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-control-access-using-iam-policies-to-invoke-api.html#api-gateway-iam-policy-resource-format-for-executing-api)

- [API Gateway에서 Lambda 프록시 통합 설정](https://docs.aws.amazon.com/ko_kr/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html)

- [Can't get AWS Lambda function to log (text output) to CloudWatch](https://stackoverflow.com/questions/37382889/cant-get-aws-lambda-function-to-log-text-output-to-cloudwatch)

- [Resources and Conditions for Lambda Actions](https://docs.aws.amazon.com/lambda/latest/dg/lambda-api-permissions-ref.html)
