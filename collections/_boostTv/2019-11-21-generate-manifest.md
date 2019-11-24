---
title: "manifest 생성하기"
excerpt: "manifest 생성하기"
last_modified_at: 2019-11-21T22:34:00
---

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

app.use(bodyParser.text()); // 구독 이후에 사용됨
app.use(bodyParser.json({ limit: "50mb" }));
app.use(bodyParser.urlencoded({ limit: "50mb", extended: false }));

app.post("/", (req, res) => {
  //  처음 구독인증을 위해 필요한 소스
  //  const options = {
  //    TopicArn: req.headers["x-amz-sns-topic-arn"],
  //    Token: req.body.Token
  //  };

  const { state, outputKeyPrefix, outputs, playlists } = req.body;
  const playlistName = playlists[0].name;

  const manifestUrl = `https://${process.env.BUCKET_NAME}-${process.env.REGION}.amazonaws.com/${outputKeyPrefix}${playlistName}.mpd`;
  console.log(manifestUrl);

  console.log(req.body);

  // sns.confirmSubscription(options);
  res.send();
});

app.listen(PORT, () => {
  console.log(`Listening on ${PORT}.............`);
});
```
