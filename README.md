# 小米电视去广告

路由器 hosts 层去广告方案：干掉开机广告、屏保广告、推荐信息流、统计上报，同时保留小爱语音、米家联动、小米账号登录、系统升级、投屏。

## 文件说明

- `blocklist.txt` — 屏蔽列表（30 个域名），写入路由器 `/etc/hosts`，格式 `127.0.0.1 <域名>`
- `whitelist.txt` — 放行列表（12 个域名），**绝不能屏蔽**，否则小爱语音、米家控制、账号登录直接失效

## 适用

路由器 hosts、AdGuard、OpenWrt、旁路由。不要直接在电视内修改 hosts（新版系统锁文件）。

建议路由器单独给电视 IP 应用这条规则，不要全屋统一拦截，避免家里其他小米设备异常。

## 使用方法（OpenWrt）

```sh
# 备份
cp /etc/hosts /etc/hosts.bak
# 追加屏蔽列表
cat blocklist.txt >> /etc/hosts
# 重载 dnsmasq（无需重启路由）
/etc/init.d/dnsmasq restart
```

## 屏蔽范围

- 小米广告主域名（ad.mi.com、ad.xiaomi.com 等）
- PatchWall 广告、画报、推荐流（pandora.xiaomi.com 系）
- 广告统计、埋点上报（mistat、kuyun、umeng 等）
- 第三方 OTT 广告（hpplay 等）

## 放行（关键）

- 米家 IoT：`api.io.mi.com`、`device.io.mi.com`、`mqtt.io.mi.com`
- 小爱语音：`xiaoai.mi.com`、`ai.xiaomi.com`、`assistant.pandora.xiaomi.com`
- 账号登录：`account.xiaomi.com`、`passport.xiaomi.com`、`sso.xiaomi.com`
- 系统升级：`update.miui.com`、`otacdn.xiaomi.net`、`cdn.otacdn.xiaomi.com`

## 配合的 ADB 精简（可选）

```sh
# 停用（可逆）
for p in com.miui.tv.analytics com.mitv.screensaver com.xiaomi.mitv.health \
  com.xiaomi.mitv.healthdiagnosis com.xiaomi.screenrecorder com.mitv.overseaservice \
  com.xm.webcontent com.xiaomi.mitv.upgrade com.xiaomi.mitv.tvmanager; do
  adb shell "pm disable-user --user 0 $p"
done
# 卸载画报
adb shell "pm uninstall --user 0 com.mitv.gallery"
adb shell "pm uninstall --user 0 com.xiaomi.tv.gallery"
```

## 已知问题

部分新版 HyperOS 电视开机广告是本地预存视频，hosts 无法消除，只能屏蔽联网推送的在线广告、画报、推荐流。

**重要**：实测发现开机时「hosts 有屏蔽 + 上述 9 个包被停用」组合会导致小爱语音报 `network not available`（错误码 40010007）。规避方法：重启电视前先清空 hosts，重启确认小爱正常后再加回屏蔽；或保持 9 个包启用只用 hosts 屏蔽。

## 故障排查

- 小爱无法唤醒 / 不能控制米家设备：检查 `api.io.mi.com` / `mqtt.io.mi.com` 是否被误拦
- 电视退出小米账号：放行 `account.xiaomi.com`
- 无法系统升级：放行 `update.miui.com`
- 小米系统版本更新后广告域名可能新增，广告复现时需补充新域名
