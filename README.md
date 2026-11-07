# YouTube Enhance - Shadowrocket 适配版

由 Loon 插件 [xzavier-amico/modules](https://github.com/xzavier-amico/modules) 的 `youtube-enhance.plugin` 转换而来（原始逻辑来自 Maasea，Apache-2.0）。功能：屏蔽播放器/信息流/Shorts 广告、后台播放、画中画、自动高清（最高 4320）、最高 4 倍速、隐藏 Premium 推广等，开关取原插件默认值（Shorts 入口默认保留、竖屏直播默认保留）。

## 小火箭导入

模块地址（小火箭 → 配置 → 模块 → + → 从 URL 添加）：

```
https://raw.githubusercontent.com/lyc-000001/youtube-shadowrocket/main/youtube-enhance.module
```

前提：小火箭 MITM 已开启且证书已安装并设为信任。导入后彻底杀掉 YouTube App 重新打开再验证。

## 适配说明

原脚本 `youtube.js` 逐字节未改，仅在前面加了一层 Shadowrocket 兼容头：

1. **TextEncoder / TextDecoder 补全**：原脚本在顶层就 `new TextEncoder()` / `new TextDecoder()`，在缺少这两个全局的脚本环境里会直接崩溃、所有规则失效（已用无 TextEncoder 的模拟环境复现原版崩溃、适配版通过）。
2. **body / status 归一化**：`bodyBytes` → `body`、`statusCode` → `status`；请求 body 保证为 ArrayBuffer 视图（脚本明确要求 binary-body-mode 的视图形式）。
3. **开关默认值注入**：原插件有 26 个开关，小火箭模块无法可靠传递，兼容头在宿主没有给出可用 `$argument` 时注入原插件声明的默认值。

已通过的模拟测试（无 TextEncoder/TextDecoder 的沙箱）：`player/ad_break` 返回空 protobuf 拦截广告、`log_event` 热更新哈希头被剥离、`browse` 响应在 ArrayBuffer 与 bodyBytes 两种交付形式下均正常处理。

注意：YouTube 的会员状态是服务端校验，本模块实现的是去广告、后台播放、画中画、分辨率解锁等客户端功能，**不包含** Premium 的离线下载、YouTube Music Premium 等服务端权益。YouTube App 更新后脚本可能失效，属正常现象。

## 文件

- `youtube_shadowrocket.js`：兼容头 + 原脚本
- `youtube-enhance.module`：小火箭模块
