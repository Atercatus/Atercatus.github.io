---
title: "NCLOUD API 사용 설명서"
excerpt: "NCLOUD API 사용 설명서"
last_modified_at: 2019-11-13T19:00:00
---

## NCLOUD API

### 준비하기

- 인증키 생성

  - `Access Key ID` , `Secret Key` 를 확인합니다.
  - 포털의 [마이페이지 > 계정관리 > 인증키 관리](https://www.ncloud.com/mypage/manage/auth) 메뉴에서 '신규 API 인증키 생성'을 통해서 `Access Key ID`, `Secret Key`를 생성합니다.
  - sub account의 경우, 콘솔의 [Sub Account > Sub Accounts > 서브 계정 상세](https://console.ncloud.com/iam/users) 메뉴에서 'API Key' 탭에서 생성한 `Access Key ID`, `Secret Key`를 사용할 수도 있습니다.

- API Key 생성
  - `API Key`를 확인합니다.
  - 콘솔의 [API Gateway > API Key](https://console.ncloud.com/apigw/apiKeys) 메뉴에서 'API Key 생성' 을 통해서 `API Key`를 생성합니다.

### AUTHPARAMS 생성하기

- API를 사용하기 위해선 인증정보가 필요합니다.
- 각 요청에 아래와 같은 내용을 요청메시지 헤더에 첨부하여야 인증을 할 수 있다.

| Header                   | Description                                                                                                                                                          |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| x-ncp-apigw-timestamp    | 1970년 1월 1일 00:00:00 협정 세계시(UTC)부터의 경과 시간을 밀리초(millisecond)로 나타낸 것이다. APIGW 서버와 시간차가 5분 이상 나는 경우 유효하지 않은 요청으로 간주 |
| x-ncp-apigw-api-key      | API Gateway에서 발급받은 키 (primary key 또는 secondary key)                                                                                                         |
| x-ncp-iam-access-key     | 포털 또는 sub account에서 발급받은 Access Key ID                                                                                                                     |
| x-ncp-apigw-signature-v2 | 위 예제의 Body를 Access Key Id와 맵핑되는 SecretKey로 암호화한 서명 HMAC 암호화 알고리즘은 HmacSHA256 사용                                                           |

```
curl -i -X GET \
   -H "x-ncp-apigw-timestamp:1505290625682" \
   -H "x-ncp-apigw-api-key:cstWXuw4wqp1EfuqDwZeMz5fh0epaTykRRRuy5Ra" \
   -H "x-ncp-iam-access-key:D78BB444D6D3C84CA38A" \
   -H "x-ncp-apigw-signature-v1:WTPItrmMIfLUk/UyUIyoQbA/z5hq9o3G8eQMolUzTEo=" \
 'https://ncloud.apigw.ntruss.com/petStore/v1/pets'
```

### x-ncp-apigw-signature-v2

요청에 맞게 StringToSign를 생성하고 SecretKey로 HmacSHA256 알고리즘으로 암호화한 후 Base64로 인코딩합니다. 이 값을 x-ncp-apigw-signature-v1로 사용합니다.

- JavaScript 예제 코드

```javascript
function makeSignature({ secretKey, method, url, timestamp, accessKey }) {
  const space = " ";
  const newLine = "\n";

  const hmac = CryptoJS.algo.HMAC.create(CryptoJS.algo.SHA256, secretKey);

  hmac.update(method); // "GET"
  hmac.update(space);
  hmac.update(url); // "/api/v2/jobs"
  hmac.update(newLine);
  hmac.update(timestamp); // Date.now().toString()
  hmac.update(newLine);
  hmac.update(accessKey);

  const hash = hmac.finalize();

  return hash.toString(CryptoJS.enc.Base64);
}
```

## VOD Transcoder

[서비스 설명](https://www.ncloud.com/product/media/vodTranscoder)

[API 설명](https://apidocs.ncloud.com/ko/media/vod_transcoder/)

[Node.js HTTPS](https://nodejs.org/api/https.html)

[Example Source](https://github.com/Atercatus/ncloud-example)

[SWAGGER](https://vodtranscoder.apigw.ntruss.com/apigw/swagger-ui?productId=68t7ynk7rs&apiId=0wkol9llac&stageId=ppn5jlcb3p#/v2/post_jobs)

### 공통 설정

#### VOD Transcoder API URL

```
https://vodtranscoder.apigw.ntruss.com/api/v2
```

#### 요청 헤더

| 헤더 명                  | 설명                                                                                                                                                                                                     |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| x-ncp-apigw-timestamp    | 1970년 1월 1일 00:00:00 협정 세계시(UTC)부터의 경과 시간을 밀리초(Millisecond)로 나타내며 API Gateway 서버와 시간차가 5분 이상 나는 경우 유효하지 않은 요청으로 간주 `x-ncp-apigw-timestamp:{Timestamp}` |
| x-ncp-apigw-api-key      | API Gateway에서 발급받은 키 `x-ncp-apigw-api-key:{API Gateway API Key}`                                                                                                                                  |
| x-ncp-iam-access-key     | 네이버 클라우드 플랫폼 포털에서 발급받은 Access Key ID `x-ncp-iam-access-key:{Sub Account Access Key}`                                                                                                   |
| x-ncp-apigw-signature-v2 | Access Key ID 와 맵핑되는 Secret Key로 암호화한 서명 HMAC 암호화 알고리즘은 HmacSHA256 사용 `x-ncp-apigw-signature-v2:{API Gateway Signature}`                                                           |
| Content-Type             | Request body가 포함된 요청의 경우 Content type을 application/json 으로 요청 `Content-Type: application/json`                                                                                             |

#### 인증 헤더

VOD Transcoder API를 사용하기 위해서는 API Gateway 인증이 필요합니다.

상세한 API Gateway 인증 관련 가이드는 [NAVER CLOUD PLATFORM API](https://apidocs.ncloud.com/ko/common/ncpapi/) 참고 부탁드립니다.

#### HELPER 함수 작성

```javascript
const CryptoJS = require("crypto-js");

function makeOption({
  host,
  method,
  path,
  timestamp,
  apiKey,
  accessKey,
  secretKey
}) {
  const signature = makeSignature({
    secretKey,
    method,
    url: path,
    timestamp,
    accessKey
  });

  return {
    host,
    method,
    path,
    headers: {
      "Content-Type": "application/json",
      "x-ncp-apigw-timestamp": timestamp,
      "x-ncp-apigw-api-key": apiKey,
      "x-ncp-iam-access-key": accessKey,
      "x-ncp-apigw-signature-v2": signature
    }
  };
}

function makeSignature({ secretKey, method, url, timestamp, accessKey }) {
  const space = " ";
  const newLine = "\n";

  const hmac = CryptoJS.algo.HMAC.create(CryptoJS.algo.SHA256, secretKey);

  hmac.update(method);
  hmac.update(space);
  hmac.update(url);
  hmac.update(newLine);
  hmac.update(timestamp);
  hmac.update(newLine);
  hmac.update(accessKey);

  const hash = hmac.finalize();

  return hash.toString(CryptoJS.enc.Base64);
}

module.exports = {
  makeSignature,
  makeOption
};
```

### JOB 조회

#### 예제 코드

```javascript
require("dotenv").config();
const https = require("https");

const { makeOption } = require("../ncloud/helper");

const vodTranscoder = { host: "vodtranscoder.apigw.ntruss.com" };
const now = Date.now().toString();

const option = makeOption({
  host: vodTranscoder.host,
  method: "GET",
  path: "/api/v2/jobs",
  timestamp: now,
  apiKey: process.env.API_KEY_PRI,
  accessKey: process.env.ACCESS_KEY_ID,
  secretKey: process.env.SECRET_KEY
});

const req = https.request(option, res => {
  console.log(`STATUS: ${res.statusCode}`);
  console.log(`HEADERS: ${JSON.stringify(res.headers)}`);
  res.setEncoding("utf8");
  res.on("data", chunk => {
    console.log(`BODY: ${chunk}`);
  });
  res.on("end", () => {
    console.log("No more data in response.");
  });
});

req.on("error", e => {
  console.error(`problem with request: ${e.message}`);
});

// req.write();

req.end();
```

### JOB 작성

#### Notification URL

Job 생성시 Notification URL 을 등록해두면 Job 상태 변경(Progressing, Success, Failed)에 대한 콜백을 받을 수 있습니다.

선택 옵션이므로 반드시 설정할 필요는 없으며, 이벤트가 발생하면 아래와 같은 형태로 callback 이 발송됩니다. (HTTP POST 로 발송)

```json
{
  "jobId": "JOB_ID",
  "status": "FAILED|PROGRESSING|SUCCESS"
}
```

#### 예제 코드

```javascript
require("dotenv").config();
const https = require("https");

const { makeOption } = require("../ncloud/helper");

const vodTranscoder = { host: "vodtranscoder.apigw.ntruss.com" };
const now = Date.now().toString();

const fileName = "sample";
const ext = "png";
const body = JSON.stringify({
  jobName: "apitest3",
  storageType: "object", // object storage
  notificationUrl: "http://101.101.163.58:4040/test", // 완료 응답 받을 서버의 주소
  inputs: [
    {
      inputBucketName: process.env.BUCKET_NAME, // bucket의 이름
      inputFilePath: "/sample.mp4"
    }
  ],
  output: {
    outputBucketName: process.env.BUCKET_NAME,
    outputFilePath: "/dest",
    thumbnailOn: "false",
    // thumbnailBucketName: "api-guide",
    // thumbnailFilePath: "/vodtr/",
    // thumbnailFileFormat: "PNG",
    // thumbnailAccessControl: "PRIVATE",
    outputFiles: [
      {
        presetId: process.env.GENERIC_360P_16_9_ID, // presetID
        outputFileName: `${fileName}360`,
        accessControl: "private"
      },
      {
        presetId: process.env.GENERIC_480P_16_9_ID,
        outputFileName: `${fileName}480`,
        accessControl: "private"
      },
      {
        presetId: process.env.GENERIC_720P_ID,
        outputFileName: `${fileName}720`,
        accessControl: "private"
      },
      {
        presetId: process.env.GENERIC_1080_ID,
        outputFileName: `${fileName}1080`,
        accessControl: "private"
      }
    ]
  }
});

const option = makeOption({
  host: vodTranscoder.host,
  method: "POST",
  path: "/api/v2/jobs",
  timestamp: now,
  apiKey: process.env.API_KEY_PRI,
  accessKey: process.env.ACCESS_KEY_ID,
  secretKey: process.env.SECRET_KEY
});

const req = https.request(option, res => {
  console.log(`STATUS: ${res.statusCode}`);
  console.log(`HEADERS: ${JSON.stringify(res.headers)}`);
  res.setEncoding("utf8");
  res.on("data", chunk => {
    console.log(`BODY: ${chunk}`);
  });
  res.on("end", () => {
    console.log("No more data in response.");
  });
});

req.on("error", e => {
  console.error(`problem with request: ${e.message}`);
});

req.write(body);

req.end();
```
