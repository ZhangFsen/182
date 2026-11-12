# 秒记账 v6.23.3（versionCode 55）

单文件 WebView 记账 App：功能全在 `app/src/main/assets/index.html`（约 1900 行，含 HTML / CSS / JS），Android 壳只负责本地存储、自动更新、悬浮球、通知与文件导出。

## 本版更新

1. **修复「点账单条目 → 编辑/删除」没反应**：`recordActionPanel`、`recordViewPanel`、`billDetailPanel`、`yearPanel`、`textPanel`、`shareAppPanel` 六个弹层的 HTML 在早前重构中被误删（JS 与 CSS 都在，只有 DOM 没了，`$('recordActionPanel')` 取到 null 直接抛错）。已从 v6.21 原包原样找回 19 个节点，编辑/删除、账单详情、年份选择、备份恢复、分享海报全部恢复。
2. **修复「分享应用」点了没反应**：同因——`shareAppPanel` 与 `sharePosterCanvas` 缺失，`openShareApp()` 第一行就报错。现已恢复海报生成与「复制链接 / 保存海报 / 分享海报」。
3. **修复分类管理弹层滚动时标题跟着走**：弹层改为纵向 flex（头尾固定），只有 `#catManageList` 内部滚动；「分类管理」标题与支出/收入切换始终钉在顶部，底部「新增分类 / 恢复默认分类」常驻。
4. **回退 6.23.0 的「备注编辑态重排」**：删掉「完成备注 / 改金额」按钮条，备注框、建议标签、数字键盘回到 6.22.1 的原版排布，弹层高度不再切换。上一版输入备注后只剩一个按钮、还得再点一次才能记账，反而更绕。
5. **软键盘改为整块盖住页面**：`AndroidManifest.xml` 给 `MainActivity` 加 `android:windowSoftInputMode="adjustPan"`。点备注 → 系统输入法弹出 → 页面整体上移，备注框悬在键盘上方，自绘数字键盘被完整盖住（原来是未指定模式，系统可能压缩页面导致数字键盘露出半截）。
6. **数字键盘随时可用（点它即接管）**：不再置灰禁用。备注输入中直接点下方数字键，会先结束备注编辑、收起系统输入法，再把这下按键算进金额——一次点击完成，不会出现「在备注里打数字却进了金额」的冲突（软键盘弹出时自绘键盘本来就被 adjustPan 整块盖住）。另用 `visualViewport` 兜底：软键盘收起时自动退出备注编辑态。
7. **收入分类管理补上「其他」**：原来固定「其他」行只在支出分支追加，收入页少一格。现已改为两类共用（不可拖拽、不可编辑删除、显示「固定分类」徽标），与支出完全同构；新增/改名时「其他」在两种类型下都拦截。
8. **焦点唯一化保留**：金额框假光标停闪、橙色高亮退灰，备注框拿到唯一橙描边——解决「金额框和备注框都有光标」的错觉。
9. **输入备注后直接完成**：软键盘上的「完成/回车」（`enterkeyhint=done`）→ 直接保存这笔记账，无需额外按钮。
10. **账单列表扁平化（去掉按天分卡）**：原来每天一张白卡（描边 + 圆角 + 阴影）套在外层白卡里，一天一道线；日期行还带浅橙底色带和底部横线，视觉上被顶成了「表头」，和账单项同级。现在日分组只做透明容器，靠留白分隔；日期压成 11px/500 浅灰小字、去底色带、去分割线、去掉「·」前缀；日合计退到 `#c2b8ad`。同时把金额提到 15px/800、分类名 14px/700，行间改用 `rgba(120,95,70,.13)` 的 0.5px hairline（0.07 在白底上只差 10 个色阶、几乎看不见；调深只改 alpha 不加粗，避免重新「分段化」）。账单页表格同步：`td` 0.13、`th` 0.16（表头必须比行线深一档）。宽松密度同步等比放大（金额 16px、行高 50px）。
11. **启动页改为静态单层（支付宝式）**：早前是「原生启动图画一次图标 + HTML 启动层再画一次」，两层之间夹着淡出与上滑动画，真机上无论如何对齐都会被看成「两个启动页 + 跳一下」。现在改成静止画面的一次硬切——原生启动图**留空**（`windowSplashScreenAnimatedIcon` 指向 1×1 全透明 drawable `splash_transparent.xml`，只保留 `windowSplashScreenBackground` 的 #FF7100），图标、品牌名「秒记账」、标语「记录每一笔小账」、底部 `ZhangFsen` / `v6.23.3` 全部只由 HTML 启动层画一次，**且没有任何入场动画**（元素一律首帧可见，连光斑都不淡入）。原理：静止画面的瞬时替换人眼无感，一旦有元素在动，大脑就会判定换了一屏。节奏为「原生图退场（cue）后静止展示 900ms → 260ms 淡出进首页」，峰值约 1.2s；原生退场只做 120ms alpha 淡出、**不做任何位移**（`translationY` 已删除，那 3% 上滑正是之前「跳一下」的直接来源）。因为原生不再画图标，HTML 层也就不必再对齐尺寸与位置：白块回到 95px、图标组用 47%（50% 是几何中心，但下方挂文字组会显得重心偏沉）。WebView 底色在启动期间设为橙色（原生图若比 WebView 首帧早退，露出的不会是米色），`setHtmlSplash(false)` 时再还原米色并恢复浅色系统栏。**仅冷启动播放**，从后台切回直接进首页；点击可跳过；`HOLD+1200` 兜底移除。**系统栏的坑仍需保留处理**：系统启动图退场会应用 `postSplashScreenTheme(AppTheme)` 把系统栏重置成米色，若 HTML 启动层还在就会看到橙色画面被两条米色栏夹住，所以要在退场后立即 + 60ms + 250ms 三次压回橙色。底部两行署名：`ZhangFsen`（11px/600/白 .82）+ 版本号（10px/400/白 .45，读 `nativeVersionInfo()` 不硬编码）；小字号英文**不要加字距**（1.1px 在 11px 上会让字母散开像水印），靠 600 字重保证不发虚。
12. 云函数无需改动，只需更新 Gitee `mjy-update` 的 `update.json`。

本文件只保留当前版本的更新说明，历史版本记录不随包分发。

## 构建

**本地（Android Studio）**

```
Android Studio 直接打开本目录，或 gradlew assembleRelease
JDK 17；compileSdk 35 / minSdk 23（见 app/build.gradle）
签名用的是内置 miaojizhang-stable-signing.jks，口令与别名写在 app/build.gradle 的 signingConfigs 里
```

**GitHub Actions（推送到 main 或手动触发即自动打包）**

`.github/workflows/build-apk.yml` 会装 JDK 17 + Android SDK 35 + Gradle 8.10.2，跑 `assembleDebug assembleRelease`，产物上传到 Actions 的 Artifacts（名字带构建序号）。仓库里不需要 gradlew，也不需要本地 SDK。

> 注意：`app/build.gradle` 里的签名口令是明文。仓库若公开，等于公开签名钥匙。

## 发布更新

把新 APK 传到 Gitee `mjy-update` 仓库对应的下载地址，再把该仓库的 `update.json` 改成：

```json
{
  "versionCode": 55,
  "versionName": "6.23.3",
  "url": "https://…/MiaoJiZhang-v6.23.3.apk",
  "notes": ["本版更新说明，一条一行"]
}
```

云函数无需改动。

## 目录

| 路径 | 作用 |
|---|---|
| `app/src/main/assets/index.html` | **唯一功能文件**：全部界面、样式、业务逻辑 |
| `app/src/main/res/drawable/splash_transparent.xml` | 原生启动图用的 1×1 全透明 drawable：启动图只显示纯橙背景，图标交给 HTML 层画 |
| `app/src/main/res/values/styles.xml` | `AppStartingTheme`：原生启动底色 `#FF7100`，改这里要同步改 HTML 层的 `.mj-splash` |
| `app/src/main/assets/qrcode.bundle.js` | 分享海报用的二维码生成库 |
| `app/src/main/java/.../MainActivity.java` | WebView 壳、本地文件读写、返回键、状态栏 |
| `app/src/main/java/.../AppUpdater.java` | 检查更新 / 下载安装 APK |
| `app/src/main/java/.../FloatingWindowService.java` | 记账悬浮球 |
| `app/src/main/java/.../ReminderReceiver.java` | 记账提醒通知 |
| `app/src/main/AndroidManifest.xml` | 软键盘模式 `adjustPan`、权限、Activity 配置 |
| `miaojizhang-stable-signing.jks` | 固定签名文件，**删了已发布版本就无法覆盖安装** |
| `.github/workflows/build-apk.yml` | GitHub Actions 打包脚本 |
| `.gitignore` | 排除 `build/`、`.gradle/`、`local.properties`、`.idea/` 等构建产物 |
