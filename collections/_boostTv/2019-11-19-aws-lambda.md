---
title: "Pre-signed url 을 이용한 S3 업로드"
excerpt: "AWS lambda 와 API Gateway를 이용하여 pre-signed url 을 반환하고, 이를 이용하여 브라우저에서 S3 버킷에 업로드를 해보자"
last_modified_at: 2019-11-17T20:34:00
---

## Pre-signed url

브라우저에서 우리가 구현할 서버에 파일을 업로드하고 다시 서버에서 S3 버킷으로 업로드 하는 과정은 자원이 많이 소모되고 비효율적이라고 판단되어 브라우저에서 S3 버킷으로 다이렉트로 접근하도록 설계했습니다.

이를 위해선 브라우저에서 S3에 접근권한을 갖기 위한 동작이 필요한데, credential을 브라우저에서 담고 있기에는 위험하기 때문에 lambda에서 pre-signed url을 반환하는 함수를 작성해서 이용하기로 했습니다.

전체적인 구조로는 브라우저가 동영상을 업로드 하기 위해서는 우선 lambda에 `putObject`에 해당하는 pre-signed url을 요청하고 이를 응답으로 받습니다. 이후, 이 url을 이용하여 S3 버킷에 접근하여 바로 동영상을 업로드하고, 그것이 완료되면 구현한 서버로 이를 알려주면 서버에선 이에 대한 정보를 DB에 저장하는 방식으로 이루어 질 것입니다.

### lambda 함수 생성

![image](https://user-images.githubusercontent.com/32104982/69133121-b864c380-0af8-11ea-95f5-350469a957d6.png)

### lambda 코드

```javascript
require("dotenv").config();
const AWS = require("aws-sdk");

const s3 = new AWS.S3({
  accessKeyId: process.env.ACCESS_KEY_ID,
  secretAccessKey: process.env.SECRET_ACCESS_KEY,
  region: process.env.BUCKET_REGION,
  signatureVersion: "v4"
});

const bucket = process.env.BUCKET_NAME;

exports.handler = async (event, context, callback) => {
  try {
    const { fileName } = event;

    console.log(bucket);

    const params = {
      Bucket: bucket,
      Key: fileName,
      Expires: 60 * 30,
      ContentType: "video/*",
      ACL: "public-read"
    };

    const url = await s3.getSignedUrlPromise("putObject", params);

    callback(null, url);
  } catch (err) {
    callback(err);
  }
};
```

파일이 크므로 .zip으로 만들어 업로드 lambda에 업로드 했습니다.
ACL을 접근하기 위해서 아래와 같이 버킷 설정을 했습니다.

![image](https://user-images.githubusercontent.com/32104982/69133237-efd37000-0af8-11ea-8090-3be9c3bfb24c.png)

### 버킷 정책

```
{
    "Version": "2012-10-17",
    "Id": "Policy",
    "Statement": [
        {
            "Sid": "Stmt",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:PutObjectAcl",
                "s3:List*"
            ],
            "Resource": "arn:aws:s3:::NAME/*"
        }
    ]
}
```

### CORS 구성

```
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
<CORSRule>
    <AllowedOrigin>*</AllowedOrigin>
    <AllowedMethod>POST</AllowedMethod>
    <AllowedMethod>GET</AllowedMethod>
    <AllowedMethod>PUT</AllowedMethod>
    <AllowedMethod>DELETE</AllowedMethod>
    <AllowedMethod>HEAD</AllowedMethod>
    <AllowedHeader>*</AllowedHeader>
</CORSRule>
</CORSConfiguration>


```

## API Gateway

위에서 작성한 lambda를 배포하기 위해서는 API Gateway를 사용해야합니다.

![image](https://user-images.githubusercontent.com/32104982/69133570-75572000-0af9-11ea-8b61-6449722042b4.png)

## CORS 설정

![image](https://user-images.githubusercontent.com/32104982/69140645-39768780-0b06-11ea-83ce-0551b533092c.png)

브라우저에서 접근하기 위해서는 반드시 CORS 설정을 해주어야한다.
그리고 **반드시! 반드시!** 재배포 해야한다.

![image](https://user-images.githubusercontent.com/32104982/69140677-54e19280-0b06-11ea-87e0-dfe6955b11e5.png)

이작업을 하지 않으면 적용이 되지 않는다. 이 문제 때문에 총 3시간을 썼다.

## 테스트

```tsx
import React from "react";

const VideoUpload: React.FunctionComponent = () => {
  const fileInput = React.createRef<HTMLInputElement>();

  const uploadVideo = async e => {
    e.preventDefault();

    const file = fileInput.current.files[0];
    const fileName = file.name;

    const preSignedUrl = await getPreSignedUrl(fileName);
    uploadToBucket(preSignedUrl, file);
  };

  return (
    <>
      <form onSubmit={uploadVideo}>
        <input type="file" ref={fileInput} />
        <input type="submit" />
      </form>
    </>
  );
};

const getPreSignedUrl = async fileName => {
  const response = await fetch(process.env.AUTH_LAMBDA_HOST, {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({ fileName })
  });

  const url = await response.json();
  return url;
};

const uploadToBucket = async (preSignedUrl, file) => {
  const option = {
    method: "PUT",
    body: file
  };

  await fetch(preSignedUrl, option);
};

export default VideoUpload;
```

![image](https://user-images.githubusercontent.com/32104982/69140746-7a6e9c00-0b06-11ea-9959-8d21de743907.png)

![image](https://user-images.githubusercontent.com/32104982/69140762-82c6d700-0b06-11ea-88d7-b064b195cf70.png)

확인 완료 ^^
