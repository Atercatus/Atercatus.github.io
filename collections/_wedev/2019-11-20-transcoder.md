---
title: "AWS Elastic Transcoder를 이용해 MPEG-DASH manifest 파일 생성"
excerpt: "AWS lambda 와 AWS Elastic Transcoder 이용하여 MPEG-DASH manifest 파일 생성해보자"
last_modified_at: 2019-11-20T20:34:00
---

## AWS Elastic Transcoder

MPEG-DASH 를 이용해 비디오를 스트리밍 하기 위해서는 MPEG-DASH에 맞는 manifest 파일을 생성해야합니다.
우선 사용자가 S3에 비디오를 업로드하면 이를 퀄리티별로 인코딩한 후, 이에 맞는 manifest 파일을 생성해주어야합니다.
AWS Elastic Transcoder를 사용하면 이를 한번에 수행할 수 있습니다.

이에 대한 과정을 좀 더 명확하게 알고 싶다면 ffmpeg로 인코딩 한 후, packager의 도움을 받아 manifest파일을 생성하는 작업을 직접 로컬해서 할 수 있습니다.

## 이벤트 트리거

S3 에서 버킷을 선택하고 [관리] 탭을 들어가게 되면 이벤트라는 항목을 볼 수 있습니다.

![image](https://user-images.githubusercontent.com/32104982/69223297-71d99c80-0bbe-11ea-8260-69ecc0195c07.png)

![image](https://user-images.githubusercontent.com/32104982/69223280-68e8cb00-0bbe-11ea-93cf-d7d0064dcaec.png)

버킷에 비디오가 업로드 될 때 이벤트를 발생 시킬 것이므로 PUT 이벤트에 대해서 이벤트가 트리거 되도록 설정합니다.
그리고 트리거 됐을 때, 응답할 람다 함수를 선택합니다.

![image](https://user-images.githubusercontent.com/32104982/69223357-8fa70180-0bbe-11ea-8614-eb48a35ee7cd.png)

## Lambda

할당이 제대로 되었다면 아까 할당한 람다 함수 상세화면에서 아래와 같은 장면을 보실 수 있습니다.

![image](https://user-images.githubusercontent.com/32104982/69223474-c3822700-0bbe-11ea-89d0-bbf3f27d6e93.png)

### 소스 코드

```javascript
require("dotenv").config();
const AWS = require("aws-sdk");

AWS.config.update({
  accessKeyId: process.env.ACCESS_KEY_ID,
  secretAccessKey: process.env.SECRET_ACCESS_KEY,
  region: process.env.BUCKET_REGION
});

const transcoder = new AWS.ElasticTranscoder();

const presets = [
  process.env.PRESET_ID_AUDIO_128k,
  process.env.PRESET_ID_VIDEO_600k,
  process.env.PRESET_ID_VIDEO_1200k,
  process.env.PRESET_ID_VIDEO_2400k,
  process.env.PRESET_ID_VIDEO_4800k
];
const outputKeys = ["audio", "600K", "1200K", "2400K", "4800K"];
const segmentDuration = "5";
const thumbPattern = "thumbnail_{count}";

function makeOutputs() {
  return outputKeys.reduce((acc, outputKey, index) => {
    const output = {
      Key: outputKey,
      PresetId: presets[index],
      SegmentDuration: segmentDuration
    };

    if (outputKey !== "audio") {
      output.ThumbnailPattern = thumbPattern + outputKey;
    }

    return [...acc, output];
  }, []);
}

exports.handler = async (event, context, callback) => {
  const { Records } = event;

  const s3 = Records[0].s3;
  const bucket = s3.bucket;
  const object = s3.object;
  const inputKey = object.key;

  const extensionRemovedPath = inputKey.split(".")[0].split("/");
  const fileName = extensionRemovedPath[extensionRemovedPath.length - 1];

  const path = inputKey.split("/");
  path.splice(path.length - 1, 1);
  const outputKey = path.reduce((acc, cur) => {
    return `${acc}/${cur}`;
  });

  const playlistName = fileName;

  const params = {
    PipelineId: process.env.PIPELINE_ID,
    Input: {
      Key: inputKey,
      AspectRatio: "auto",
      FrameRate: "auto",
      Resolution: "auto",
      Container: "auto",
      Interlaced: "auto"
    },
    OutputKeyPrefix: `${outputKey}/`,
    Outputs: makeOutputs(),
    Playlists: [
      {
        Name: playlistName,
        Format: "MPEG-DASH",
        OutputKeys: outputKeys
      }
    ]
  };

  console.log(params);

  const request = transcoder.createJob(params);
  request.on("success", function(response) {
    console.log("success");
    console.log(response);
  });
  request.on("error", function(err, response) {
    if (err) {
      console.log("error");
      console.log(err);
      console.log(response);
    }
  });

  request.send();

  callback(null, true);
};
```

[참조](https://github.com/aws-samples/aws-sdk-js-sample-video-transcoder/blob/master/console-app/app.js)
[참조](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/ElasticTranscoder.html#createJob-property)

## 결과 확인

![image](https://user-images.githubusercontent.com/32104982/69224046-2d9acc00-0bbf-11ea-9eea-d521a970e47c.png)
