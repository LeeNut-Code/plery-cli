# plery-cli

管理 192.168.43.1 随身 WiFi（UFI，ZTE 风格 `/reqproc` 接口）的命令行工具。单文件 Python，纯标准库，无依赖。

## 安装

```bash
chmod +x plery
ln -sf "$(pwd)/plery" ~/.local/bin/plery
```

## 配置

`~/.config/plery/config.json`（自动创建，权限 0600）：

```json
{
  "host": "192.168.43.1",
  "password": "admin"
}
```

- `login --password XXX` 会把密码写入 config；不传参数则使用已存的密码
- 临时换设备：`--host IP` 全局 flag

## 命令

| 命令 | 说明 |
|---|---|
| `plery --version` | 版本（当前 1.0） |
| `plery login [--password XXX]` | 登录（密码持久化到 config） |
| `plery logout` | 登出 |
| `plery status` | 状态一览：信号/网络/SIM/WiFi/电池 |
| `plery traffic` | 月用量 + 在线时长 |
| `plery devices` | 已连设备列表（MAC/IP/主机名/类型） |
| `plery net on\|off` | 连接/断开网络（实测可用） |
| `plery macfilter show` | MAC 过滤模式 + 黑白名单 |
| `plery macfilter add MAC [--mode 1\|2]` | 加入名单（默认按当前模式：白1/黑2） |
| `plery macfilter del MAC` | 从黑白名单移除 |
| `plery macfilter mode 0\|1\|2` | 设过滤模式（0=关 1=白名单 2=黑名单） |
| `plery limit show` | 查看流量限额 |
| `plery limit set --size 205 [--alert 90] [--off]` | 设置限额（GB/月，默认启用） |
| `plery wifi show` | WiFi 状态（SSID/加密/频段） |
| `plery wifi password NEWPASS` | 改 WiFi 密码（WPA2PSK/AES） |
| `plery wifi coverage low\|medium\|high` | 发射功率 |
| `plery wifi sleep --minutes N` | 闲置休眠分钟数（0=不休眠） |
| `plery device info` | 电池/省电状态 |
| `plery device reboot` | 重启设备 |
| `plery device shutdown` | 关机 |
| `plery tsw show` | 定时开关机状态 |
| `plery tsw set --on HH:MM --off HH:MM` | 设置定时开关机 |
| `plery tsw off` | 关闭定时开关机 |
| `plery raw 'cmd=字段&multi_data=1'` | 直接 GET（逃生舱，`cmd=` 开头 verbatim） |
| `plery raw --post 'goformId=XXX&参数'` | 直接 POST |

全局 `--json`：输出原始 API JSON（供脚本/自动化复用）。

## 机制

- **自动重登录**：会话过期（查询返回空串）时自动重新 LOGIN 再重试，用户无感知
- **错误处理**：设备不可达 / 登录失败 / result 非 0 → stderr 报错，exit 1

## 验证

读命令与无害等幂写命令（`limit set --size 205`、`wifi coverage medium`、`wifi sleep --minutes 10`、`tsw set/off`、`macfilter add/del/mode` 回环后恢复原状）已对真机实测通过；`net on/off` 断网/重连实测可用（ppp_status 确认）；`device reboot/shutdown`、`wifi password` 为破坏性操作，只实现未执行。

## 已知限制

- WiFi 密码固定写 WPA2PSK/AES（设备唯一加密模式）
- 定时开关机只到分钟精度（设备格式 `HH;MM`）
- 短信功能固件停用（`HAS_SMS:false`），未实现
