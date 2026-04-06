# Synology Surveillance Station 配置

建议群晖只负责录像，不要再直接连摄像头 RTSP / ONVIF。

## 推荐方式

在 Surveillance Station 中手动添加自定义 RTSP 摄像机：

- 品牌：`用户自定义`
- 类型：`RTSP`
- 传输协议：`TCP`

## 录像地址

通道 1：

```text
rtsp://GO2RTC_RTSP_USERNAME:GO2RTC_RTSP_PASSWORD@GO2RTC_IP:8554/tplink_ch1_main
```

通道 2：

```text
rtsp://GO2RTC_RTSP_USERNAME:GO2RTC_RTSP_PASSWORD@GO2RTC_IP:8554/tplink_ch2_main
```

## 说明

- 不建议让群晖继续使用 ONVIF
- 不建议让群晖继续直连摄像头原始 RTSP
- 这样可以减少 TP-Link 摄像头并发占用

## 如果录像失败

优先检查：

1. `go2rtc` 的 RTSP Server 是否启用了账号密码
2. 群晖是否填了 `TCP`
3. 群晖连接的是 `go2rtc` 地址，而不是摄像头地址
