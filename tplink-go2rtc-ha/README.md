# TP-Link 双目摄像头接入 Home Assistant / go2rtc / 群晖

这份文档整理了今天完成的方案，目标是把中国版 TP-Link 双目 IPC 接入：

- `go2rtc`：统一取流、双向音频转发
- `Home Assistant`：双目画面、现代化弹出卡、PTZ、长按对讲
- `Synology Surveillance Station`：只负责录像

适用前提：

- 摄像头为中国版 TP-Link IPC，支持 `MULTITRANS`
- 已开启 RTSP / ONVIF
- Home Assistant 已可通过 HTTPS 访问
- 已安装 `WebRTC Camera`、`button-card`、`browser_mod`、`card-mod`

## 一、最终拓扑

推荐拓扑如下：

1. 摄像头只给 `go2rtc` 提供 RTSP 和 `MULTITRANS`
2. `Home Assistant` 从 `go2rtc` 播放视频和双向音频
3. `Home Assistant` 单独直连摄像头 `ONVIF` 用于 PTZ
4. `Synology Surveillance Station` 只连 `go2rtc` 的 RTSP 转发地址，不再直连摄像头

这样可以避开 TP-Link 摄像头对并发连接、双 ONVIF 客户端的限制。

## 二、今天完成的方法步骤

1. 确认摄像头型号为 TP-Link 双目 IPC，并验证双目 RTSP 规则：
   - `stream1&channel=1`
   - `stream1&channel=2`
2. 确认摄像头 RTSP `OPTIONS` 返回包含 `MULTITRANS`，证明支持中国版 TP-Link 双向音频协议。
3. 在 `go2rtc` 中建立三路流：
   - `tplink_ch1_main`：通道 1 画面
   - `tplink_ch2_main`：通道 2 画面
   - `tplink_ch2_talk`：通道 2 画面 + `MULTITRANS` 双向音频
4. 在 `go2rtc` 中开启 RTSP Server 认证，给群晖使用。
5. 在 Home Assistant 中保留一个 ONVIF 集成，仅用于 `camera.tp_ipc_mainstream` 的 PTZ 控制。
6. 在 HA 主界面的 `main-ui.yaml` 中，把“大门监控”入口改成现代化弹窗，整合：
   - 云台镜头
   - 固定镜头
   - PTZ 十字键
   - 长按说话麦克风按钮
   - 回预置位按钮
7. 把右侧固定镜头从常驻“对讲模式”改成：
   - 默认监听
   - 长按麦克风 250ms 切到对讲
   - 松手立即切回监听
   - 切流后强制取消静音，避免回到静音状态
8. 把群晖改成只连接 `go2rtc` 的 RTSP 地址录制，不再使用摄像头原始 RTSP / ONVIF。

## 三、关键结论

- 中国版 TP-Link 双向音频应使用 `multitrans://`
- 双目视频继续走 RTSP
- PTZ 最稳的方式是 `HA -> ONVIF -> 摄像头`
- 群晖只录像时最稳，建议 `群晖 -> go2rtc RTSP`
- 由于 TP-Link 摄像头对连接数较敏感，不建议让多个客户端同时直接连接摄像头

## 四、文件说明

- `examples/go2rtc.yaml`
  - go2rtc 示例配置
- `examples/ha-door-popup.yaml`
  - Home Assistant 弹出卡示例
- `examples/synology-surveillance-station.md`
  - 群晖 Surveillance Station 配置方法

## 五、需要按你环境替换的内容

上传到 GitHub 的示例已经脱敏，请把这些占位符替换成你自己的值：

- `CAMERA_IP`
- `CAMERA_USERNAME`
- `CAMERA_PASSWORD`
- `GO2RTC_IP`
- `GO2RTC_RTSP_USERNAME`
- `GO2RTC_RTSP_PASSWORD`
- `camera.tp_ipc_mainstream`

## 六、预置位说明

示例里的“回预置位”按钮当前使用：

- `move_mode: GotoPreset`
- `preset: "1"`

如果按钮没有反应，通常不是布局问题，而是摄像头的预置位 token 不是 `1`。此时需要按实际预置位 token 调整。

## 七、排错建议

1. `go2rtc` 中画面正常，但没有对讲：
   - 检查是否使用了 `multitrans://`
   - 检查 HA 是否为 `HTTPS`
   - 检查浏览器麦克风权限
2. 切换监听/对讲后变成静音：
   - 已在当前方案中通过前端按钮逻辑强制取消静音
   - 如果未来改卡片结构，需要保留这段逻辑
3. 群晖无法录像：
   - 确认群晖连接的是 `go2rtc` 的 RTSP 地址，而不是摄像头原始地址
4. PTZ 无法回预置位：
   - 优先检查 `preset` token

## 八、当前完成状态

今天已经完成：

- 双目视频接入
- go2rtc 双向音频
- HA 对讲
- HA PTZ
- 现代化弹出卡
- 长按说话逻辑
- 群晖录制链路整理

后续如果要继续优化，建议优先做：

1. 读取摄像头真实预置位 token，修正“回预置位”按钮
2. 继续微调弹窗响应式布局
3. 如有需要，把“固定镜头 / 云台镜头”进一步做成更完整的状态条卡片
