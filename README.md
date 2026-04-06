# TP-Link 双目摄像头接入 Home Assistant / go2rtc / 群晖

这个仓库整理了一套已经落地验证的方案，用于把中国版 TP-Link 双目 IPC 接入到 Home Assistant、go2rtc 和群晖 Surveillance Station。

目标是同时实现：

- 双目实时画面
- Home Assistant 内的 PTZ 控制
- 基于 `MULTITRANS` 的双向对讲
- 群晖只负责稳定录像
- 更现代化的 HA 弹出卡界面

## 方案概览

推荐拓扑：

- 摄像头 -> `go2rtc`
- `Home Assistant` -> `go2rtc` 获取画面与对讲
- `Home Assistant` -> 摄像头 `ONVIF` 负责 PTZ
- `Synology Surveillance Station` -> `go2rtc` RTSP 转发地址负责录像

这样可以尽量避开 TP-Link 摄像头对连接数、双 ONVIF 客户端和多端直连取流的限制。

## 仓库内容

- [`tplink-go2rtc-ha/README.md`](./tplink-go2rtc-ha/README.md)
  - 完整方法、排错过程和注意事项
- [`tplink-go2rtc-ha/examples/go2rtc.yaml`](./tplink-go2rtc-ha/examples/go2rtc.yaml)
  - go2rtc 示例配置
- [`tplink-go2rtc-ha/examples/ha-door-popup.yaml`](./tplink-go2rtc-ha/examples/ha-door-popup.yaml)
  - Home Assistant 弹出卡示例
- [`tplink-go2rtc-ha/examples/synology-surveillance-station.md`](./tplink-go2rtc-ha/examples/synology-surveillance-station.md)
  - 群晖 Surveillance Station 接入说明

## 核心思路

1. 双目视频继续走 RTSP：
   - `stream1&channel=1`
   - `stream1&channel=2`
2. 双向对讲走 `multitrans://`
3. PTZ 只保留一条 `ONVIF` 链路给 Home Assistant
4. 群晖不再直连摄像头，而是录制 `go2rtc` 的转发 RTSP
5. HA 前端用 `WebRTC Camera` + `button-card` + `browser_mod` 做弹出卡和长按对讲

## 已验证的能力

- 中国版 TP-Link IPC 的 `MULTITRANS` 能力识别
- go2rtc 双目取流与双向音频
- Home Assistant 内双目弹窗卡
- PTZ 方向控制
- 长按麦克风进入对讲，松手回监听
- 群晖通过 go2rtc RTSP 录像

## 使用前提

- 摄像头已开启 RTSP / ONVIF
- 摄像头为支持 `MULTITRANS` 的中国版 TP-Link IPC
- Home Assistant 已启用 HTTPS
- 已安装这些前端组件：
  - `WebRTC Camera`
  - `button-card`
  - `browser_mod`
  - `card-mod`

## 注意事项

- 仓库示例已做脱敏，请替换成你自己的 IP、账号、密码和实体名
- 预置位按钮里的 `preset` token 需要按你的摄像头实际值调整
- 当前“监听 / 对讲”如果通过切流实现，视频会有一次短暂重连黑屏，这是 WebRTC 重建连接导致的

## 后续可继续优化

- 把“按住说话”从切换流改成单连接内切换麦克风发送状态，进一步减少黑屏
- 读取真实 ONVIF preset token，完善回预置位按钮
- 继续优化弹窗在手机和高缩放页面下的布局
