# `rtsp`推流

海康大华等品牌的摄像头所推的流大部分是`rtsp`，通过`video.src`是播放不了实时视频的，需要转一遍



## 解决方案

**`RTSP` 流 -> `Web` 能接收的流（`Websocket` 或 `WebRTC`） -> Web Video**



有两部分是需要处理的：

- `RTSP` 流 -> `WebSocket` / `WebRTC`
- `WebSocket` / `WebRTC` -> `WebVideo`



## 方案1

基于 `ffmpeg` 的 Node 后端推流方案 + 基于 `jspmeg` / `flvjs` 的前端视频展示

后端推流方案有很多，这里选用`node-rtsp-stream`，这一步是用来将`rtsp`转成`WebSocket`

前端`jspmeg` / `flvjs`将`websocket`转成`webVideo`



### 核心代码解析

`server`

```js
new Stream({
  name: 'socket',
  streamUrl: 'rtsp://localhost:554/test', // 这个就是本地的rtsp流，这个需要工具生成，后面会说
  wsPort: 9999,
  ffmpegOptions: {
    '-stats': '',
    '-r': 20,
    '-s': '1280 720'
  }
});
```

`fe`

```tsx
export default function FlvPlayer() {
  const flvRef = useRef(null);
  useEffect(() => {
    // 根据你后端 RTSP 推流服务转的 WebSocket 地址修改
    const videoElement = document.getElementById('videoElement');
    if (flvjs.isSupported()) {
      const flvPlayer = flvjs.createPlayer({
        type: 'flv',
        isLive: true,
        url: 'ws://localhost:9998', // 连接websocket
      });
      flvPlayer.attachMediaElement(videoElement);
      flvPlayer.load();
      flvRef.current = flvPlayer;
      // flvPlayer.play();
    }
  }, []);

  return (
    <div>
      <h1>Flv.js Player</h1>
      <div style={{ width: '100%', display: 'flex', justifyContent: 'center', alignItems: 'center' }}>
        <video id="videoElement" controls></video>
        <button onClick={() => {flvRef?.current?.play()}}>Play</button>
      </div>
    </div>
  );
}
```





## 方案2

`WebRtc`的推流延迟更低，但是需要安装软件[webrtc-streamer](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fmpromonet%2Fwebrtc-streamer)、直接安装就行；该方案虽不需要后端支持，但浏览器有版本限制



```html
<h2 id="message"></h2>
	<webrtc-streamer id="stream" options="rtptransport=tcp&timeout=60&width=0&height=0&bitrate=0&rotation=0" style="display:none"></webrtc-streamer>
	<p id="hint">默认从url连接上获取 video || audio 参数进行播放，本示例如果没有会选择本地 RTSP 推流</p>
	<p style="color: red">【注意】：WebRTC 播放 RTSP 流需要前置启动 webrtc-streamer 客户端，否则推流无法播放</p>
	<script>     
		let messageElement = document.getElementById("message"); 
        customElements.whenDefined('webrtc-streamer').then(() => {
            let streamElement = document.getElementById("stream");

			var params = new URLSearchParams(location.search);
			if (params.has("options")) {
				streamElement.setAttribute('options', params.get("options"));
			}
			let url = {
				video:params.get("video") || 'rtsp://localhost:554/test',// 本地的rtsp流
				//audio:params.get("audio")
			};
			streamElement.setAttribute('url', JSON.stringify(url));
			streamElement.style.display = "block"
		}).catch( (e) => {
			messageElement.innerText = "webrtc-streamer webcomponent fails to initialize error:" + e
		})
	</script>
```



## 创建本地`rtsp`流

https://juejin.cn/post/7221858081485963324#heading-11



如果有线上的rtsp流，需要先拉取下

1. 安装`ffmpeg`
2. `ffplay rtsp`



## 代码

https://github.com/wczy-ao/web-rtsp-video