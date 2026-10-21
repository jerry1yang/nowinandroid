![Now in Android](docs/images/nia-splash.jpg "Now in Android")

<a href="https://play.google.com/store/apps/details?id=com.google.samples.apps.nowinandroid"><img src="https://play.google.com/intl/en_us/badges/static/images/badges/en_badge_web_generic.png" height="70"></a>

Now in Android 应用
==================

[English](README.md) | [简体中文](README.zh-CN.md)

**在[设计案例研究](https://goo.gle/nia-figma)、[架构学习路线](docs/ArchitectureLearningJourney.md)和[模块化学习路线](docs/ModularizationLearningJourney.md)中了解此应用的设计与构建过程。**

这是 [Now in Android](https://developer.android.com/series/now-in-android) 应用的代码仓库。该项目仍在**积极开发中** 🚧。

**Now in Android** 是一款完全使用 Kotlin 和 Jetpack Compose 构建的功能完备的 Android 应用。它遵循 Android 设计与开发最佳实践，旨在为开发者提供实用的参考。作为一个实际运行的应用，它通过定期提供新闻更新，帮助开发者紧跟 Android 开发领域的最新动态。

该应用目前处于开发阶段。`prodRelease` 变体已在 [Google Play 商店上架](https://play.google.com/store/apps/details?id=com.google.samples.apps.nowinandroid)。

# 功能特性

**Now in Android** 展示来自 [Now in Android](https://developer.android.com/series/now-in-android) 系列的内容。用户可以浏览近期视频、文章等内容的链接；还可以关注感兴趣的话题，并在有符合所关注兴趣的新内容发布时收到通知。

## 屏幕截图

![展示“为你推荐”界面、兴趣界面和话题详情界面的屏幕截图](docs/images/screenshots.png "展示“为你推荐”界面、兴趣界面和话题详情界面的屏幕截图")

# 开发环境

**Now in Android** 使用 Gradle 构建系统，可直接导入 Android Studio（请确保使用[此处](https://developer.android.com/studio)提供的最新稳定版本）。

将运行配置（Run Configuration）切换为 `app`：

![image](https://user-images.githubusercontent.com/873212/210559920-ef4a40c5-c8e0-478b-bb00-4879a8cf184a.png)

可以构建并运行 `demoDebug` 和 `demoRelease` 构建变体（`prod` 变体使用的后端服务器目前尚未公开）：

![image](https://user-images.githubusercontent.com/873212/210560507-44045dc5-b6d5-41ca-9746-f0f7acf22f8e.png)

成功运行后，您可以参考下方的学习路线，以更深入地了解所使用的库与工具、UI、测试、架构等方案背后的设计考量，以及项目的各个部分是如何组合在一起构成一个完整的应用。

# 架构

**Now in Android** 应用遵循[官方架构指南](https://developer.android.com/topic/architecture)，并在[架构学习路线](docs/ArchitectureLearningJourney.md)中进行了详细说明。

# 模块化

**Now in Android** 应用已实现完全模块化，您可以在[模块化学习路线](docs/ModularizationLearningJourney.md)中找到详细指南以及所使用的模块化策略说明。

# 构建

该应用包含常规的 `debug` 和 `release` 构建变体。

此外，`app` 的 `benchmark` 变体用于测试启动性能并生成基准配置文件（Baseline Profile，详情请参阅下文）。

`app-nia-catalog` 是一个独立的应用程序，用于展示针对 **Now in Android** 风格定制的组件列表。

该应用还使用 [Product Flavors（产品变种）](https://developer.android.com/studio/build/build-variants#product-flavors)来控制应用内容的加载来源：

- `demo` flavor 使用本地静态数据，以便能够立即构建和体验 UI。
- `prod` flavor 向后端服务器发起真实的真实网络请求，提供最新内容。目前该后端尚未公开。

日常开发请使用 `demoDebug` 变体。进行 UI 性能测试请使用 `demoRelease` 变体。

# 测试

为方便组件测试，**Now in Android** 使用 [Hilt](https://developer.android.com/training/dependency-injection/hilt-android) 进行依赖注入。

大多数数据层组件都被定义为接口。随后绑定具体的实现（带有各种依赖项），为应用中的其他组件提供这些接口。
在测试中，**Now in Android** 明确**不**使用任何 Mock 库。
相反，可以在测试中使用 Hilt 的测试 API（或在 `ViewModel` 测试中通过手动构造函数注入）将生产实现替换为测试替身（Test Doubles）。

这些测试替身实现了与生产代码相同的接口，通常提供了一个简化但依然贴近真实的实现，并附带额外的测试钩子（Test-only hooks）。
这样可以减少测试的脆弱性，并覆盖更多的生产代码，而不是仅仅验证针对 Mock 对象的特定调用。

示例：
- 在插桩测试（Instrumentation Tests）中，使用临时文件夹来存储用户偏好设置，并在每次测试后清空。这允许使用真实的 `DataStore` 并运行所有相关代码，而不是去模拟数据更新流。
- 每个 Repository 都有对应的 `Test` 实现，它们不仅实现了完整的 Repository 接口，还提供了仅供测试使用的钩子。`ViewModel` 测试使用这些 `Test` Repository，因此可以通过测试钩子操作 Repository 的状态并验证后续行为，而无需校验是否调用了特定的 Repository 方法。

执行以下 Gradle 任务以运行测试：

- `testDemoDebug`：针对 `demoDebug` 变体运行所有本地单元测试。截图测试会失败（原因见下文）。为避免此问题，请在运行单元测试之前先执行 `recordRoborazziDemoDebug`。
- `connectedDemoDebugAndroidTest`：针对 `demoDebug` 变体运行所有插桩测试。

> [!NOTE]
> 请勿运行 `./gradlew test` 或 `./gradlew connectedAndroidTest`，因为这会针对**所有**构建变体执行测试，这不仅没有必要，而且会导致失败，因为目前仅支持 `demoDebug` 变体。其他变体均未编写任何测试（后续可能会有所改变）。

## 截图测试（Screenshot tests）

截图测试会截取应用中的屏幕或 UI 组件的屏幕截图，并将其与之前录制的已知渲染正确的屏幕截图进行比对。

例如，Now in Android 包含[截图测试](https://github.com/android/nowinandroid/blob/main/app/src/testDemo/kotlin/com/google/samples/apps/nowinandroid/ui/NiaAppScreenSizesScreenshotTests.kt)，用于验证在不同屏幕尺寸下导航栏是否正确显示（[已知正确的截图](https://github.com/android/nowinandroid/tree/main/app/src/testDemo/screenshots)）。

Now In Android 使用 [Roborazzi](https://github.com/takahirom/roborazzi) 对特定屏幕和 UI 组件执行截图测试。在进行截图测试时，以下 Gradle 任务非常有用：

- `verifyRoborazziDemoDebug`：运行所有截图测试，将当前截图与已知正确的截图进行比对验证。
- `recordRoborazziDemoDebug`：录制新的“已知正确”截图。当您对 UI 进行了更改并手动验证其渲染正确后，请使用此命令。截图将保存在 `modulename/src/test/screenshots` 目录下。
- `compareRoborazziDemoDebug`：在失败的测试截图与已知正确的截图之间生成对比图像。这些图像也可以在 `modulename/src/test/screenshots` 中找到。

> [!NOTE]
> **关于截图测试失败的说明**  
> 此代码仓库中保存的已知正确截图是在 CI 上使用 Linux 环境录制的。其他平台可能会（而且很可能会）生成略有差异的图像，从而导致截图测试失败。在非 Linux 平台上进行开发时，一种解决办法是在开始工作之前先在 `main` 分支上运行 `recordRoborazziDemoDebug`。做出更改后，`verifyRoborazziDemoDebug` 就只会识别真正合理的更改。

有关截图测试的更多信息，请[观看此演讲视频](https://www.droidcon.com/2023/11/15/easy-screenshot-testing-with-compose/)。

# UI

该应用采用 [Material 3 设计规范](https://m3.material.io/)进行设计。若要详细了解设计流程并获取设计源文件，请参阅 [Now in Android Material 3 案例研究](https://goo.gle/nia-figma)（设计资源[亦提供 PDF 版本](docs/Now-In-Android-Design-File.pdf)）。

所有屏幕和 UI 元素均完全采用 [Jetpack Compose](https://developer.android.com/jetpack/compose) 构建。

应用支持两种主题：

- 动态取色（Dynamic color）—— 基于[用户当前的系统配色主题](https://material.io/blog/announcing-material-you)提取颜色（若设备支持）
- 默认主题（Default theme）—— 当不支持动态取色时使用预定义的配色方案

两种主题均支持深色模式（Dark mode）。

该应用采用自适应布局来[支持不同的屏幕尺寸](https://developer.android.com/guide/topics/large-screens/support-different-screen-sizes)。

欢迎[在此了解更多有关 UI 架构的信息](docs/ArchitectureLearningJourney.md#ui-layer)。

# 性能

## 基准测试（Benchmarks）

在 `benchmarks` 模块中可以找到所有使用 [`Macrobenchmark`](https://developer.android.com/topic/performance/benchmarking/macrobenchmark-overview) 编写的测试。该模块还包含用于生成 Baseline Profile 的测试。

## 基准配置文件（Baseline profiles）

此应用的 Baseline Profile 文件位于 [`app/src/main/baseline-prof.txt`](app/src/main/baseline-prof.txt)。
它包含了一系列规则，可在应用启动期间的关键用户路径上启用 AOT（Ahead-Of-Time）预编译。
有关 Baseline Profile 的更多信息，请阅读[此文档](https://developer.android.com/studio/profile/baselineprofiles)。

> [!NOTE]
> 对于修改了会影响应用启动代码的 Release 构建，需要重新生成 Baseline Profile。

如需生成 Baseline Profile，请选择 `benchmark` 构建变体，并在 AOSP Android 模拟器上运行 `BaselineProfileGenerator` 基准测试。
然后将模拟器中生成的 Baseline Profile 复制到 [`app/src/main/baseline-prof.txt`](app/src/main/baseline-prof.txt)。

## Compose 编译器指标（Compose compiler metrics）

运行以下命令以获取并分析 Compose 编译器指标：

```bash
./gradlew assembleRelease -PenableComposeCompilerMetrics=true -PenableComposeCompilerReports=true
```

报告文件将生成在 [build/compose-reports](build/compose-reports) 目录下。指标文件也将生成在 [build/compose-metrics](build/compose-metrics) 目录下。

有关 Compose 编译器指标的更多信息，请参阅[这篇博文](https://medium.com/androiddevelopers/jetpack-compose-stability-explained-79c10db270c8)。

# 开源协议

**Now in Android** 基于 Apache License (Version 2.0) 协议分发。详情请参阅 [LICENSE](LICENSE)。
