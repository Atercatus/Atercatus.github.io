---
title: "AWS SNS 구독 확인 하는 방법"
excerpt: "Node.js 로 AWS SNS 구독 확인 하는 방법에 대해서 알아보자"
last_modified_at: 2019-11-20T22:34:00
---

## AWS SNS

AWS Elastic Transcoder를 통한 작업이 모두 완료되었을 경우, 그에 대한 응답을 우리의 서버로 받아야할 필요가 있습니다. 정상적으로 인코딩 되었다면, DB에 해당 비디오의 정보를 저장해야 하기 때문입니다.

AWS SNS 서비스를 이용하면 이에 대한 응답을 받을 수 있습니다.
해당 서비스를 구독하기 위해서는 응답을 받고자 하는 서버가 제대로된 서버인지 인증을 거쳐야합니다.

Node.js 로 구현한 서버에서 이에 대한 응답을 받는 방법에 대해서 알아보겠습니다.

## 소스코드

```javascript
require("dotenv").config();
const express = require("express");
const AWS = require("aws-sdk");
const bodyParser = require("body-parser");

const PORT = 8080;

const app = express();

console.log(process.env.ACCESS_KEY_ID);

AWS.config.update({
  accessKeyId: process.env.ACCESS_KEY_ID,
  secretAccessKey: process.env.SECRET_ACCESS_KEY,
  region: process.env.BUCKET_REGION
});

const sns = new AWS.SNS();

app.use((req, res, next) => {
  if (req.get("x-amz-sns-message-type")) {
    req.headers["content-type"] = "application/json";
  }
  next();
});
app.use(bodyParser.json({ limit: "50mb" }));
app.use(bodyParser.urlencoded({ limit: "50mb", extended: false }));

app.post("/", (req, res) => {
  const options = {
    TopicArn: req.headers["x-amz-sns-topic-arn"],
    Token: req.body.Token
  };

  console.log(options);
  sns.confirmSubscription(options);
  res.send();
});

app.listen(PORT, () => {
  console.log(`Listening on ${PORT}.............`);
});
```

aws-sdk를 이용해서 구현했습니다. 이때 `express.json` 과 `express.urlencoded`를 사용하면 `req.body` 내에 Token의 값이 `undefined`인 경우가 발생합니다.

왜 인지는 모르겠으나 body-parser 모듈을 사용해야만이 body에 값이 존재하는 것을 볼 수 있습니다. 그리고, header 에 Content-Type이 정의되어 있지 않아서 임의로 넣어주어야 합니다.

구독 확인은 입력한 url에 앤드포인트는 루트인채 POST 요청으로 날라오게 됩니다.

![image](https://user-images.githubusercontent.com/32104982/69246670-774ddb80-0bec-11ea-84c6-13fffee25258.png)
