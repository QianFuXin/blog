---
tags: ["srs", "flv", "obs-studio"]
---

# 使用srs_flv_js_obs-studio实现直播

## srs服务部署

```shell
docker run --rm -it -p 1935:1935 -p 1985:1985 -p 8080:8080 \
        -p 8000:8000/udp -p 10080:10080/udp ossrs/srs:5
```

## obs-studio推流

```shell
rtmp://qianfuxin.com/live
livestream
```

## flv.js获取流

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>FLV Stream Player</title>
  </head>
  <body>
    <video
      id="videoElement"
      controls
      style="width: 100%; height: auto;"
    ></video>
    <script src="https://cdn.jsdelivr.net/npm/flv.js@1.5.0/dist/flv.min.js"></script>
    <script>
      if (flvjs.isSupported()) {
        var videoElement = document.getElementById("videoElement");
        var flvPlayer = flvjs.createPlayer({
          type: "flv",
          url: "http://qianfuxin.com:8080/live/livestream.flv ",
        });
        flvPlayer.attachMediaElement(videoElement);
        flvPlayer.load();
        flvPlayer.play();
      }
    </script>
  </body>
</html>
```
