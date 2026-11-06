# ESP32-S3 交换机 Console

一块 ESP32-S3 连接现场 Wi-Fi，浏览器可通过局域网或选用 Tailscale 隧道访问板载终端，交换机一侧使用 UART1。网页及其终端库均包含在固件中。

## 首次烧录

预编译目标是 **ESP32-S3，16 MB Flash，8 MB 八线 PSRAM（N16R8）**，默认以 DevKitC 类开发板引脚为准。确认板型后，用 esptool 4.x 将合并镜像从地址 0 写入（替换 PORT 为板子的 USB 串口）。

```sh
python -m esptool --chip esp32s3 -p PORT write_flash 0x0 firmware/switch-console-s3-n16r8.bin
```

合并镜像用于初装，会覆盖镜像范围内的原固件、分区表和 NVS。后续在相同分区布局下用 ESP-IDF 的 `idf.py -p PORT flash` 分段写入，可保留 Wi-Fi、节点身份和串口配置。

1. 用密码 `12345678` 连接 `raykwok-rmt-console` 热点，打开 http://192.168.4.1/，输入配网密码 `12345678` 和现场 2.4 GHz Wi-Fi 信息。
2. 使用局域网时，取消勾选 **Use Tailscale**。如要远程访问则勾选它，首次启用需填 Tailscale auth key。
3. 点 **Test and save**，页面会显示本地 IP。设备重启后也可重新连热点查看终端地址，联网成功约 45 秒后热点关闭。在本地模式，从现场网络访问显示的 IP；在 Tailscale 模式，从同一 tailnet 访问设备的 100.x 地址。
4. 首次登录终端或由旧固件升级后，终端密码为 `12345678`。登录后打开 **Change console password** 即可修改，保存后以新密码重新登录。
5. 换网、注册密钥填错或需重新配网时，**板子已正常启动后**按住 BOOT（GPIO0）至少 4 秒再松开，等待设备重启。重新配网时可将注册密钥留空以保留旧值；要替换失效密钥则填新值。不要在按住 BOOT 时按 RESET，那是芯片的下载模式。

## 接线与使用

| S3 端 | 连接 |
|---|---|
| GPIO17，UART1 TX | 接 3.3 V TTL 对端 RX，或 MAX3232 模块的 TTL 发送输入 |
| GPIO18，UART1 RX | 接 3.3 V TTL 对端 TX，或 MAX3232 模块的 TTL 接收输出 |
| GND | 信号地 |

交换机的 RS-232 Console 需要 3.3 V 兼容的电平转换模块。按交换机手册核对 Console 针脚、信号方向和地线，切勿将 RS-232 直接接 GPIO。此版面向 UART/RS-232；USB Console 需要另行实现对应驱动。S3 的调试日志使用独立的系统调试串口。调试交换机重启或网络配置时，请使用独立供电及不经过该交换机的 Wi-Fi 上行。

网页默认 **9600 8N1**。可输入 300–921600 的波特率，选择 5–8 个数据位、奇偶校验以及 1、1.5 或 2 个停止位，按 **Apply**后生效并保存。三线连接不使用硬件流控。回车、Tab、方向键和 Ctrl+C 均通过终端传递。最近 32 KiB 的串口输出保存在 RAM，可在网页重连时回看，板子断电会清空。默认一次由一人操作同一个物理串口。

## 从源码编译

已用 ESP-IDF **v5.4.3** 编译。激活该版本的 ESP-IDF 环境后，在本目录执行：

```sh
idf.py build
idf.py -p PORT flash monitor
```

GPIO 定义在 main/serial.c。MicroLink 固定于 [216da33](https://github.com/CamM2325/microlink/tree/216da3300f0493b0860247d43f7af5ce29df63a5) 的组件源码，终端为 [xterm.js 5.5.0](https://github.com/xtermjs/xterm.js/releases/tag/5.5.0)。第三方许可证随源码保留。

## 验收

本次已验证固件编译、分区大小和网页交互（Chrome，模拟 HTTP 设备）。实机验收可先在断开交换机时短接 GPIO17 与 GPIO18，通过 Tailscale 网页输入字符，确认原样返回，再撤去短接、接上转换器与交换机。测试启动日志、登录、Tab 和 Ctrl+C，改变波特率并重启 S3 验证设置保留。最后验证 Wi-Fi 密码错误可重试，长按 BOOT 可重配，普通局域网 IP 无法操作串口。

[English quick start](README.en.md)
