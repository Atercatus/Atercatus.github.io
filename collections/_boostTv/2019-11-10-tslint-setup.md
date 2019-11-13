---
title: "MPEG DASH"
excerpt: "MPEG DASH에 대해서 알아보자"
last_modified_at: 2019-11-10T17:00:00
---

## DASH

![image](https://user-images.githubusercontent.com/32104982/68280016-05d44000-00b8-11ea-98d2-b2bcb2bac93c.png)
(이미지출처: http://abt.net/multimedia.php)

[Dynamic Adaptive Streaming over HTTP](https://en.wikipedia.org/wiki/Dynamic_Adaptive_Streaming_over_HTTP) 또는, MPEG-DASH는 web server에서 media content를 제공할 때 client의 네트워크 상황에 따라 그에 맞는 quality의 media content를 제공하는 것을 의미합니다.
애플의 HLS(HTTP Live Streaming)과 유사합니다.

### 프로세스

#### 비디오 인코딩

[ffmpeg](https://www.ffmpeg.org/)를 이용하여 client 에서 입력한 원본 동영상을 여러 해상도의 동영상으로 인코딩합니다.

```bash
wget --no-check-certificate "https://docs.google.com/uc?export=download&id=17leZjmhYETsCl67vXBlc3_FO0W0UElyc" -O sample2.mp4

ffmpeg -y -i ./sample/sample.mp4 -c:a aac -ac 2 -ab 256k -ar 48000 -c:v libx264 -x264opts 'keyint=24:min-keyint=24:no-scenecut' -b:v 1500k -maxrate 1500k -bufsize 1000k -vf scale=-1:720 -strict -2 ./sample/sample720.mp4
ffmpeg -y -i ./sample/sample.mp4 -c:a aac -ac 2 -ab 128k -ar 44100 -c:v libx264 -x264opts 'keyint=24:min-keyint=24:no-scenecut' -b:v 800k -maxrate 800k -bufsize 500k -vf scale=-1:540 -strict -2  ./sample/sample540.mp4 -strict -2
ffmpeg -y -i ./sample/sample.mp4 -c:a aac -ac 2 -ab 64k -ar 22050 -c:v libx264 -x264opts 'keyint=24:min-keyint=24:no-scenecut' -b:v 400k -maxrate 400k -bufsize 400k -vf scale=-1:360 -strict -2  ./sample/sample360.mp4 -strict -2

```

#### MANIFEST 파일 생성

```bash
./shaka_packager/src/out/Release/packager \
  input=./sample/sample720.mp4,stream=audio,output=./dest/sample720_audio.mp4 \
  input=./sample/sample720.mp4,stream=video,output=./dest/sample720_video.mp4 \
  input=./sample/sample360.mp4,stream=audio,output=./dest/sample360_audio.mp4 \
  input=./sample/sample360.mp4,stream=video,output=./dest/sample360_video.mp4 \
--profile on-demand \
--mpd_output ./manifest/sample-manifest-full.mpd \
--min_buffer_time 3 \
--segment_duration 3

./shaka_packager/src/out/Release/packager \
  input=./sample/sample720.mp4,stream=audio,output=./dest/sample720_audio.mp4 \
  input=./sample/sample720.mp4,stream=video,output=./dest/sample720_video.mp4 \
--profile on-demand \
--mpd_output ./manifest/sample-720-full.mpd \
--min_buffer_time 3 \
--segment_duration 3
```

[Shaka packager](https://google.github.io/shaka-packager/html/) 를 이용하여 각각 다른 품질의 비디오 파일과 오디오 파일을 생성합니다. Shaka packager는 이를 토대로 manifest 파일을 생성합니다.

[MDN-DASH](https://developer.mozilla.org/en-US/docs/Web/HTML/DASH_Adaptive_Streaming_for_HTML_5_Video)에 이에 대한 정확한 흐름이 명시되어 있습니다.

[Structure of an MPEG-DASH MPD](https://www.brendanlong.com/the-structure-of-an-mpeg-dash-mpd.html)

---

#### Client

Client에서는 [Shaka player](https://shaka-player-demo.appspot.com/docs/api/tutorial-welcome.html)를 이용하여 사용자의 네트워크 상태에 따른 동영상 요청을 실시합니다.

##### 예제 코드

[Git hub 주소](https://github.com/Atercatus/simple-shaka-example)

- index.js
  shaka-player.compiled.js 는 shaka player 설치를 하면 받을 수 있습니다.

```javascript
<!DOCTYPE html>
<html>
  <head>
    <script src="/js/shaka-player.compiled.js"></script>
    <script>console.log("test");</script>
  </head>
  <body>
    <video id="video" width="640"
          controls autoplay"></video>

    <script src="/js/myapp.js"></script>
  </body>
</html>
```

- myapp.js

```javascript
const manifestUrl = "/manifest";

function initApp() {
  shaka.polyfill.installAll();

  if (shaka.Player.isBrowserSupported()) {
    initPlayer();
  } else {
    console.error("Browser not supported!");
  }
}

function initPlayer() {
  const video = document.getElementById("video");
  const player = new shaka.Player(video);

  window.player = player;

  player.addEventListener("error", onErrorEvent);

  player
    .load(manifestUrl)
    .then(function() {
      console.log("The video has now been loaded!");
    })
    .catch(onError);
}

function onErrorEvent(event) {
  onError(event.detail);
}

function onError(error) {
  console.error("Error code", error.code, "object", error);
}

document.addEventListener("DOMContentLoaded", initApp);
```

- sample-manifest-full.mpd

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--Generated with https://github.com/google/shaka-packager version 1bcaf0a-release-->
<MPD xmlns="urn:mpeg:dash:schema:mpd:2011" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="urn:mpeg:dash:schema:mpd:2011 DASH-MPD.xsd" profiles="urn:mpeg:dash:profile:isoff-on-demand:2011" minBufferTime="PT3S" type="static" mediaPresentationDuration="PT27.33333396911621S">
  <Period id="0">
    <AdaptationSet id="0" contentType="video" maxWidth="1278" maxHeight="720" frameRate="12288/512" subsegmentAlignment="true" par="16:9">
      <Representation id="0" bandwidth="430715" codecs="avc1.64001e" mimeType="video/mp4" sar="639:640" width="640" height="360">
        <BaseURL>./dest/sample360_video.mp4</BaseURL>
        <SegmentBase indexRange="875-1026" timescale="12288">
          <Initialization range="0-874"/>
        </SegmentBase>
      </Representation>
      <Representation id="1" bandwidth="1576446" codecs="avc1.64001f" mimeType="video/mp4" sar="1:1" width="1278" height="720">
        <BaseURL>./dest/sample720_video.mp4</BaseURL>
        <SegmentBase indexRange="855-1006" timescale="12288">
          <Initialization range="0-854"/>
        </SegmentBase>
      </Representation>
    </AdaptationSet>
    <AdaptationSet id="1" contentType="audio">
      <Representation id="2" bandwidth="79354" codecs="mp4a.40.2" mimeType="audio/mp4" audioSamplingRate="22050">
        <AudioChannelConfiguration schemeIdUri="urn:mpeg:dash:23003:3:audio_channel_configuration:2011" value="2"/>
        <BaseURL>./dest/sample360_audio.mp4</BaseURL>
        <SegmentBase indexRange="789-940" timescale="22050">
          <Initialization range="0-788"/>
        </SegmentBase>
      </Representation>
      <Representation id="3" bandwidth="260620" codecs="mp4a.40.2" mimeType="audio/mp4" audioSamplingRate="48000">
        <AudioChannelConfiguration schemeIdUri="urn:mpeg:dash:23003:3:audio_channel_configuration:2011" value="2"/>
        <BaseURL>./dest/sample720_audio.mp4</BaseURL>
        <SegmentBase indexRange="789-940" timescale="48000">
          <Initialization range="0-788"/>
        </SegmentBase>
      </Representation>
    </AdaptationSet>
  </Period>
</MPD>
```

#### Server

##### 예제 코드

```javascript
const express = require("express");
const fs = require("fs");
const path = require("path");
const app = express();
const PORT = process.env.PORT || 4040;

app.use("/manifest/dest", express.static("public/videos"));
app.use("/js", express.static("public/js"));

app.use(express.json());
app.use(express.urlencoded({ extended: false }));

app.get("/", (req, res) => {
  console.log("test");
  // const html = fs.readFileSync(path.resolve(__dirname, "./view/index.html"));
  res.sendFile(path.join(__dirname, "/view/index.html"));
});

app.get("/manifest", (req, res) => {
  console.log("manifest");

  res.sendFile(
    path.join(__dirname, "/public/manifest/sample-manifest-full.mpd")
  );
});

app.post("/test", (req, res) => {
  console.log(req.body);
  console.log("TEST");
  res.send("OK");
});

app.listen(PORT, () => {
  console.log(`Listening on ${PORT}...........`);
});
```
