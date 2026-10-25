# 安全护盾 ShieldGuard

> 包名：`com.shieldguard.app` ｜ Kotlin 2.x + Jetpack Compose (Material3) + 单 Activity + Navigation Compose + OkHttp + kotlinx.serialization + DataStore + Room + WorkManager
> minSdk 26 ｜ compileSdk/targetSdk 36 ｜ 中文 UI · 深浅色自适应 · 全局液态玻璃视觉

## 构建运行

1. 用 Android Studio（Ladybug+，AGP 8.9 / Gradle 8.11）打开本目录（工程会自动同步依赖）；
2. 连接设备或启动模拟器（建议 Android 8.0+），点 Run 即可。

无需任何本地配置；公益 VirusTotal Key 已内置。

## 云端自动构建（GitHub Actions）

工程内置 `.github/workflows/build-apk.yml`，无需本地安装任何环境即可拿到 APK：

1. 把本目录推送到 GitHub 仓库（默认分支 `main` 或 `master`）；
2. push 后自动触发构建；也可在仓库 **Actions → Build Debug APK → Run workflow** 手动触发；
3. 构建完成后在本次运行页面底部 **Artifacts** 下载 `ShieldGuard-debug-apk`（保留 30 天）。

云端使用 JDK 17 (Temurin) + Gradle 8.11.1（自动缓存依赖），全程约 3~5 分钟。

## 功能地图

| 模块 | 说明 |
|---|---|
| 首页 · 实时防护 | ForegroundService 常驻通知（监控时长/今日扫描数）；动态注册 `PACKAGE_ADDED/REPLACED`；新装 APK → multipart 上传 `/files` → 轮询 `/analyses/{id}` → `stats.malicious>0` 时悬浮窗（已授权 `SYSTEM_ALERT_WINDOW`）或高优先级通知 + 告警日志；令牌桶 4 次/分钟 + 指数退避 + WorkManager 落库重试 |
| 首页 · 应用扫描 | 分段控制器（玻璃质感）；「已安装应用」搜索/多选/按安装时间与大小排序/忽略系统应用开关/上次扫描徽标；「选择安装包」走 `ACTION_OPEN_DOCUMENT` 多选 + `takePersistableUriPermission` 持久化；公共流程：SHA-256 → 24h 缓存 →（隐私模式哈希查询 / 上传文件带进度）→ 分组展示（安全绿/可疑黄/报毒红）；报毒安装按钮置灰 + 二次确认解锁 |
| 首页 · 网页防护 | `VpnService` 本地 Dummy TUN 仅转发 DNS（只路由虚拟 DNS 地址，其余流量零劫持）；启用前 UDP 53 探测，超时回退默认 52.80.60.30 并警告；状态/会话时长/当前 DNS/拦截计数占位；连接有红→绿水波动画 |
| 设置 | 公益 Key（标注限速失效提示）/ 自定义 Key（AES-GCM + AndroidKeystore 加密存 DataStore，经 `/users/me` 校验）；DNS 自定义；「减弱动效」开关（同时尊重系统"移除动画"）；QQ 群 418916519（mqq 唤起 → 失败复制群号 Toast → 复制链接备选） |

## 液态玻璃实现（5.x）

- **GlassCard**：半透明底（alpha 0.12~0.18）+ RenderEffect 后景模糊近似（API31+，同源背景模糊重建；API<31 自动降级静态半透明）；左上 0.5 → 右下 0.08 渐变描边；顶部白色内发光 0.15→0；彩色环境光阴影（ambient/spotColor）；圆角 24~28dp 按压时与缩放同步 spring(dampingRatio 0.6)。
- **Liquid Layer**：3 个径向渐变光斑沿 Lissajous 路径漂移（周期 8~15s，BlendMode.Screen，alpha 0.06~0.12）；拖拽时向触摸点弹簧跟随、松手缓归；状态变更触发 300ms 涟漪（EmphasizedDecelerate）。
- **卡片特有**：实时防护开启 = 绿色流动荧光描边（dash offset 循环）；勾选 = 液滴吸附弹性形变（scale 1.15→1.0）；网页防护连通 = underline 水波呼吸由红转绿扩散。
- **告警弹窗**：中心膨胀入场（spring 0.6）+ scrim 0.5 + 报毒条目红色 shimmer 自上而下 1.2s。
- **性能**：动画值全部在 `drawWithContent`/`graphicsLayer`/Canvas draw 阶段读取（draw-phase invalidation，不触发每帧 recompose）；API<31 或"减弱动效"降级静态玻璃 + 简单 alpha。

## 工程规范（6）

- 网络 `Dispatchers.IO` + `AppResult` 统一封装；拦截器处理 429（Retry-After）与 401（Key 轮换/停用 + 明确文案）。
- `CountingRequestBody` 上传进度回调；单文件上限 64MB，超限自动回退 `/files/{sha256}` 哈希查询。
- `SgLog` Release 禁用日志；proguard 保留 kotlinx.serialization 模型。
- UI 三态：`EmptyState` / `GlassSkeleton`（骨架屏 shimmer）/ `ErrorState`（可重试），扫描中心对失败任务提供重试。
- 中文注释以"为什么"解释关键取舍。

## 已知边界

- 公益节点限速且可能失效（UI 已标注），建议在设置页换成自己的 Key。
- DNS VPN 为"仅 DNS 转发"的轻量实现（IPv4/UDP/53，A/AAAA 记录场景），不代理其他流量、不做内容过滤；拦截计数为占位。
- 悬浮窗告警需用户在系统设置授予"显示在其他应用上层"。
