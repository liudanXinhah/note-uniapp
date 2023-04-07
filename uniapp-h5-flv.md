## ios手机上面没法通过flv.js实现拉流? Ios没法使用flv.js直播如何解决?

### 1.原因：
>1.1:由于flv.js原理利用浏览器里的Media Source Extensions(MSE)转化flv，但是由于苹果拒绝支持MSE，只在iPad的iOS13里支持。

### 2.解决方法：
>2.1:采用 EasyPlayer.js网页全终端播放器

>>2.1.1:集 rtmp, hls, flv, websocket 于一身的网页直播/点播播放器, 使用简单, 功能强大  

>>2.1.2:属性(Property):  
  * video-url 视频流地址 String default ‘’
  * video-title 视频右上角显示的标题 String default ‘’
  * poster 视频封面图片 String default ‘’
  * autoplay 自动播放 Boolean default true
  * loop 是否循环播放 Boolean default false
  * live 是否直播, 标识要不要显示进度条 Boolean default false
  * alt 视频流地址没有指定情况下, 视频所在区域显示的文字, 相当于 html img 标签的 alt 属性String * default ‘无信号’
  * muted 是否静音 Boolean default false
  * aspect 视频显示区域的宽高比 String default ‘16:9’
  * loading 指示加载状态, 支持 sync 修饰符
  * fluent 流畅模式, Boolean default true
  * stretch 是否拉伸, Boolean default false
  * timeout m3u8 加载超时(秒) Number default 20
  * show-custom-button 是否在工具栏显示自定义按钮(极速/流畅, 拉伸/标准), Boolean defaulttrue

>>2.1.3:方法(Medthod):  
  * getCurrentTime 获取当前播放时间进度, 同步返回播放时间进度数据
  * snap 外部 API 方式获取快照, 快照获取成功后, 触发 snapOutside Event

>>2.1.4:事件(Event)：  
  * message 触发通知消息, 参数: { type: ‘’, message: ‘’}
  * ended 播放结束, 参数: 无
  * timeupdate 进度更新, 参数: 当前时间进度
  * pause 暂停, 参数: 当前时间进度
  * play 播放, 参数: 当前时间进度,
  * snapOutside 外部快照回调, 参数: 快照 Base64 数据
  * snapInside 内部快照回调, 由控制栏快照按钮触发, 参数: 快照 Base64 数据

### 3.使用方法:
  >3.1:安装：  
  * npm install easy-player

  >3.2:在 Vue 中使用:
  * copy node_modules/easy-player/dist/component/easy-player.swf 到 www 根目录
  * copy node_modules/easy-player/dist/component/crossdomain.xml 到 www 根目录
  * copy node_modules/easy-player/dist/component/easy-player-lib.min.js 到 www 根目录
  * 以上 copy 操作通过 webpack 完成, 编辑你的 webpack.config.js
  ```
    ......
        // copy js lib and swf files to dist dir
        new CopyWebpackPlugin([
            { from: 'node_modules/easy-player/dist/component/crossdomain.xml'},
            { from: 'node_modules/easy-player/dist/component/easy-player.swf'},
            { from: 'node_modules/easy-player/dist/component/easy-player-lib.min.js', to: 'js/'}
        ]),
    ......
  ```
  * 在 html 中引用 www 根目录 easy-player-lib.min.js
  * 编辑你的 Vue 组件
  ```
    ......
    <EasyPlayer :videoUrl="videoUrl" fluent autoplay live stretch></EasyPlayer>
    ......
    import EasyPlayer from 'easy-player'
    ......
      components: {
        EasyPlayer
      }
    ......

  ```
>3.3脱离 Vue 使用:
* 在 html 中引用 www 根目录 easy-player-element.min.js
* HTML 集成 Demo
```
  <!DOCTYPE HTML>
  <html>
      <head>
          <title>easy-player</title>
          <meta charset="utf-8">
          <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
          <meta content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no" name="viewport">
      <body>      
          <easy-player video-url="rtmp://live.hkstv.hk.lxdns.com/live/hks" live="true" stretch="true"></easy-player>
          <easy-player video-url="http://live.hkstv.hk.lxdns.com/live/hks/playlist.m3u8" live="false" stretch="true"></easy-player>
          <easy-player video-url="http://live.hkstv.hk.lxdns.com/flv/hks.flv" live="true" stretch="true"></easy-player>
          <easy-player video-url="ws://192.168.1.65:3000/play?stream=rtsp://username:password@192.168.1.64:5504/Streaming/Channels/102"></easy-player>
      <script type="text/javascript" src="easy-player-element.min.js"></script></body>
  </html>
```

### 4.常见问题
 > 在html中集成Deme时在本地环境能正常播放flv，但是上到服务器后就黑屏。  
 >> 必须将 EasyPlayer.wasm 文件放在服务器根目录下，比如访问的路劲为 https://zhgd.zghxsjy.com/qr/flv_test/flv.html ,那么就应该放在 https://zhgd.zghxsjy.com 根目录下。因为在 easy-player-element.min.js 中获取 EasyPlayer.wasm 的方式为从根目录获取。