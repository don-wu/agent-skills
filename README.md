# agent-skills

don-wu 的 Claude Code 自定义 Skill 合集。

## 安装

```bash
npx skills add don-wu/agent-skills
```

---

## Skills 列表

### `ios-android-diff-review`

**iOS/Android 平台差异测试用例审查与补全**

#### 功能描述

当你有一份测试用例，想检查是否遗漏了 iOS/Android 双端的平台专项差异场景时使用。Skill 会：

1. 解析输入的测试用例，识别功能域
2. 对照内置的 **82 条平台差异规则（R01–R82）** 进行第一轮检查，输出差异报告
3. 为每个差异点补充符合标准格式的新用例
4. 进行第二轮自检（查遗漏、去重、检查 P0/P1 覆盖率）
5. 迭代直到自检报告显示「新增遗漏：无」
6. 输出完整用例集（原始用例保留 + 补充用例追加）

#### 触发方式

```
/ios-android-diff-review [飞书文档 URL 或 token（可选）]
```

#### 两种输入模式

| 模式 | 说明 |
|------|------|
| **文档模式** | 提供飞书文档 URL / token，自动读取用例；结果可选择写回文档 |
| **文本模式** | 直接粘贴测试用例文本，结果输出到对话 |

#### 内置差异规则覆盖范围（R01–R82）

| 分类 | 规则编号 | 主要覆盖点 |
|------|---------|-----------|
| 渲染/图形 | R01–R03 | Metal vs Vulkan、Shader 缓存、GPU 驱动差异 |
| 支付体系 | R04–R07 | Apple IAP 强制要求、订阅机制、沙盒环境 |
| 账号认证 | R08–R11 | Sign in with Apple 强制、Game Center、生物识别 |
| 推送通知 | R12–R16 | APNs vs FCM、通知权限 5 态 vs 2 态、Doze 模式 |
| 权限系统 | R17–R21 | 相册分级授权、ATT/IDFA、剪贴板隐私提示 |
| 生命周期 | R22–R25 | 后台挂起、Force Quit、BGTask vs WorkManager |
| 网络 | R26–R28 | HTTP/3 QUIC、ATS 强制 HTTPS |
| 安全 | R29–R31 | 越狱/Root 检测、Keychain vs Keystore、SSL Pinning |
| 广告归因 | R32–R33 | SKAdNetwork vs GAID、ATT 隐私限制 |
| WebView | R34–R35 | WKWebView vs Chromium、JS Bridge 注入方式 |
| 存储 | R36–R39, R80–R81 | 沙盒隔离、iCloud vs Google Play 云存档、Core Data vs Room |
| 触觉反馈 | R40 | Core Haptics vs VibrationEffect |
| AR | R41 | ARKit TrueDepth/LiDAR vs ARCore |
| Unity/引擎 | R42–R43 | IL2CPP 强制、Metal vs Vulkan 渲染后端 |
| 热更新合规 | R44–R45 | iOS 禁止动态代码下发、Privacy Manifest |
| 分发/审核 | R46–R48 | TestFlight vs Play 轨道、包格式、审核周期 |
| 音频 | R49–R50 | AVAudioSession vs AudioFocus、硬件静音键 |
| 崩溃稳定性 | R51–R54 | Watchdog/Jetsam vs ANR/OOM、启动崩溃恢复 |
| 功耗性能 | R55–R57 | 冷启动基准、触控采样率（60Hz vs 480Hz） |
| 本地化 | R58–R59 | RTL 布局、VoiceOver vs TalkBack |
| 系统交互 | R60–R64 | 深链接、录屏保护、键盘遮挡、灵动岛、iPad 多任务 |
| 视频媒体 | R65–R66 | HEVC 硬解率、音视频中断处理 |
| 蓝牙/手柄 | R67 | MFi vs 标准 HID |
| 社交分享 | R68 | UIActivityViewController vs Intent |
| 自动化 | R69 | XCUITest vs Espresso |
| 防沉迷 | R70 | Screen Time vs Digital Wellbeing |
| 应用评分 | R71 | SKStoreReviewController 3次/年 vs In-App Review 1次/月 |
| 数据埋点 | R72 | IDFV/IDFA vs GAID |
| 机型兼容 | R73–R74 | iOS 有限机型 vs Android 碎片化 |
| 内存管理 | R75 | ARC 确定性释放 vs GC 批量回收 |
| 新手引导 | R76 | 权限弹窗时机差异 |
| 客服举报 | R77 | PHPickerViewController vs Intent |
| Shader 编译 | R78 | MTLBinaryArchive vs Vulkan Pipeline Cache |
| 深链接补充 | R79 | 未安装时降级行为 |
| 网络安全 | R82 | ATS 豁免 vs network_security_config |

#### 补充用例格式

- 用例 ID：`TC-DIFF-[模块]-[序号]`，如 `TC-DIFF-PAY-001`
- 包含字段：基本信息、测试设计、测试环境、前置条件、测试数据表、测试步骤（含 iOS 预期 / Android 预期双列）、预期结果、风险评估
- 优先级：P0（核心差异）/ P1（重要差异）/ P2（一般差异）/ P3（边缘差异）

#### 所需工具

- `lark-cli`（读写飞书文档时需要）

#### 使用示例

```
# 文档模式
/ios-android-diff-review https://xxx.feishu.cn/docx/EXEpd8uqUoMs5FxyuWdcWzl1nNd

# 文本模式（触发后直接粘贴用例）
/ios-android-diff-review
```
