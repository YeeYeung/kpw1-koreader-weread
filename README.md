# Kindle Paperwhite 1 (2012) 复活记：越狱 + KOReader + 微信读书

一台 2012 年的 Kindle Paperwhite 一代，深度亏电放了几年。现在它跑着最新版 KOReader，在线读微信读书，本地读自己的 epub。

本仓库记录完整流程和踩坑。只有文档，不含任何固件或越狱文件，下载源全部指向官方渠道。

## 最终效果

- KOReader v2026.07.1（最新版，PW1 完全可用）
- 微信读书插件：扫码登录、进度同步、章节缓存
- 本地 epub/mobi 阅读，排版全面碾压原生系统
- USBNetwork 已装（可选，shell 直连）
- 固件停留在 5.6.1.1（PW1 最终版，无 OTA 威胁）

## 硬件前提

| 项目 | 说明 |
|---|---|
| 设备 | Kindle Paperwhite 1 (2012)，256MB RAM |
| 固件 | 5.6.1.1（出厂最高版本） |
| 电量 | 刷机前充到 50% 以上 |
| 线材 | 必须是数据线，不能是纯充电线 |

深度亏电的机器先插墙充（5V/1A 老充电头最好）充 4 小时以上，再长按电源键 40 秒硬重启。

## 核心认知（先看这个，少走弯路）

1. **5.6.1.1 无法直接越狱**。PW1 的越狱漏洞只存在于 5.0–5.4.4.2。
2. **降级只是手段，不是归宿**。降到老固件种下越狱后，要升回 5.6.1.1 再打 hotfix。
3. **不要在老固件上装 KUAL/MRPI**——这是本次最大的坑。现代工具链（2024+ 的 KUAL Booklet、MRPI、USBNetwork 包）是为「新固件 + hotfix」环境构建的，在 5.3.3 上会遇到：
   - KUAL azw2 报「未经授权开发商签名」
   - Update Your Kindle 对所有社区包置灰
   - 放到根目录的安装包被开机流程静默删除
   - `;log mrpi` 搜索栏命令无效
4. 越狱本体（文件名注入漏洞）不受签名限制，任何时候都能重跑。
5. 5.6.1.1 是 PW1 最终固件，升回去之后亚马逊没有更新可推，不需要担心 OTA。

## 全流程

### 0. 备份

USB 挂载后把 `documents/` 整个拷到电脑。

### 1. 降级 5.6.1.1 → 5.3.3（脏拔插法）

正常途径 5.6.1.1 拒绝降级，需要强制触发：

1. 设备开飞行模式
2. USB 连接电脑，把官方 `update_kindle_5.3.3.bin` 放到根目录
3. 等 2 分钟让系统扫描到文件
4. **USB 保持连接**，长按电源键 15–20 秒强制重启
5. 重启后自动进入固件安装，进度条走完前不拔线

固件下载（亚马逊官方 S3）：
```
https://s3.amazonaws.com/G7G_FirmwareUpdates_WebDownloads/update_kindle_5.3.3.bin
```

### 2. 安装越狱

用 NiLuJe 的 K5 越狱（适用 5.0–5.4.4.2）：

1. 从 [Snapshots 帖](https://www.mobileread.com/forums/showthread.php?t=225030) 下载 `kindle-jailbreak-1.16.N` 包
2. 解出内层 `kindle-5.4-jailbreak.zip`，内容全部解压到 Kindle 根目录
3. 设备上：Settings → 菜单 → Update Your Kindle
4. 屏幕出现 `**** JAILBREAK ****` 即成功

### 3. 升级回 5.6.1.1

越狱做好后能在官方升级中存活（前提：越狱是最新版）。

1. 官方 `update_kindle_5.6.1.1.bin` 放根目录
2. Settings → 菜单 → Update Your Kindle
3. 约 10 分钟，完成后确认固件版本 5.6.1.1

```
https://s3.amazonaws.com/G7G_FirmwareUpdates_WebDownloads/update_kindle_5.6.1.1.bin
```

### 4. 打越狱 hotfix

1. 越狱包里的 `Update_jailbreak_hotfix_1.16.N_install.bin` 放根目录
2. Settings → 菜单 → Update Your Kindle

hotfix 就是为「老固件越狱后升到 5.6.x」的场景设计的，装完越狱在新固件上稳定存活。

### 5. 安装 KUAL + MRPI + USBNetwork

1. 从 Snapshots 帖下载 KUAL、MRPI（kual-mrinstaller）、USBNetwork 三个包
2. MRPI 的 `extensions/` 目录拷到 Kindle 根目录
3. 根目录建 `mrpackages/` 文件夹，放入：
   - `Update_KUALBooklet_v2.7.37_install.bin`
   - `Update_usbnet_0.22.N_install_touch_pw.bin`（PW1 用 touch_pw 变体）
4. 设备主界面搜索栏输入 `;log mrpi` 回车
5. MRPI 安装画面出现，自动装完队列里的包

### 6. 安装 KOReader + 微信读书插件

1. [KOReader releases](https://github.com/koreader/koreader/releases) 下载 `koreader-kindle-v*.zip`（PW1 用常规 kindle 版，不是 legacy 也不是 pw2/hf 版），`koreader/` 和 `extensions/` 解压合并到根目录
2. [weread.koplugin releases](https://github.com/finlater/weread.koplugin/releases) 下载插件 zip，`weread.koplugin/` 解压到 `koreader/plugins/`
3. KUAL → Start KOReader（选标准启动，停掉亚马逊框架腾内存，256MB 机器必要）

### 7. 微信读书登录

1. 手机微信读书 App：我 → 设置 → 微信读书 Skill → 开通并获取 API Key
2. KOReader：工具 → 微信读书 → 微信扫码登录
3. 手机扫码确认（如出现四位验证码，在 KOReader 输入）

## 踩坑记录

| 现象 | 原因 | 解法 |
|---|---|---|
| 插电只显示需要充电，开不了机 | 锂电深度亏电，保护板截止 | 墙充预充 4h+，长按 40s 硬重启 |
| 老固件上 KUAL azw2 报签名错误 | Kindlet 证书体系需要 MKK，现代包不再适配老固件 | 不修。升回 5.6.1.1 走 hotfix 路线 |
| Update Your Kindle 对社区包置灰 | 更新器预检拒收，老固件缺现代越狱的密钥环境 | 同上 |
| 根目录安装包开机后消失 | 开机流程验签失败后静默删除 | 同上 |
| `;log mrpi` 无反应 | 搜索栏钩子在老固件上未挂载 | 升回 5.6.1.1 后即生效 |
| macOS 弹不出 Kindle 卷 | Spotlight 索引占用 | `diskutil unmount force`，或 sync 后强制弹出 |

## 微信读书的 epub 缓存

插件按章下载正文，在 KOReader 缓存目录拼成完整 epub。看完一本书，设备里就有一份 epub 源文件。**仅限个人使用，不要分发**——分发是侵权，批量抓取还可能触发风控封号。

## 致谢

- [NiLuJe](https://www.mobileread.com/forums/member.php?u=27057) — K5 越狱、KUAL、MRPI、USBNetwork 的维护者
- [koreader/koreader](https://github.com/koreader/koreader) — 开源阅读器本体
- [finlater/weread.koplugin](https://github.com/finlater/weread.koplugin) — 微信读书插件
- [MobileRead 论坛](https://www.mobileread.com/forums/) — 所有前人踩坑
- [oakreef 的 PW1 越狱记录](https://oakreef.ie/bog/kindle-jailbreaking) — 点醒「升回去打 hotfix」路线的关键文章

## 免责声明

刷机有风险。本文流程在一台 PW1 上完整验证成功，但任何操作请自行承担后果。中途断电、拔线、刷错文件都可能导致设备无法启动。
