# Now in Android 项目指南

[English](AGENTS.md) | [简体中文](AGENTS.zh-CN.md)

Now in Android 是一款使用 Kotlin 编写的原生 Android 移动应用程序。它提供关于 Android 开发领域的定期新闻动态。用户可以选择关注特定话题、在有新内容发布时接收通知，以及将内容加入书签收藏。

## 架构

该项目是一款现代 Android 应用程序，遵循 Google 官方架构指南。它是一个采用单 Activity 架构的响应式应用，使用了以下技术栈与设计：

-   **UI 层：** 完全使用 Jetpack Compose 构建，包含 Material 3 组件，并针对不同屏幕尺寸提供自适应布局。
-   **状态管理：** 使用 Kotlin 协程和 `Flow` 实现单向数据流（UDF）。`ViewModel` 作为状态持有者，将 UI 状态以数据流的形式对外暴露。
-   **依赖注入：** 全应用统一使用 Hilt 进行依赖注入，简化了依赖管理并提高可测试性。
-   **导航：** 使用 Jetpack Navigation 2 for Compose 处理导航，提供声明式且类型安全的方式在不同页面间导航。
-   **数据层：** 采用 Repository（仓库）模式实现数据层。
    -   **本地数据：** 使用 Room 和 DataStore 进行本地数据持久化。
    -   **远程数据：** 使用 Retrofit 和 OkHttp 从网络获取数据。
-   **后台处理：** 使用 WorkManager 处理可延迟执行的后台任务。

## 模块结构

- 主应用模块位于 `app/` 目录。
- 功能业务模块位于 `feature/` 目录。
- 核心基础设施与共享模块位于 `core/` 目录。

## 构建与测试命令

应用及 Android 依赖库包含两个产品变种（Product Flavors）：`demo` 和 `prod`，以及两个构建类型（Build Types）：`debug` 和 `release`。

- 构建打包：`./gradlew assemble{Variant}`，通常为 `assembleDemoDebug`。
- 修复代码格式/代码风格：`./gradlew spotlessApply`
- 运行本地测试：`./gradlew {variant}Test`
- 运行单个测试类：`./gradlew {variant}Test --tests "com.example.myapp.MyTestClass"`
- 运行本地截图测试：`./gradlew verifyRoborazziDemoDebug`

### 插桩测试（Instrumented tests）

- 使用 Gradle 托管设备（GMD）运行设备端测试：`./gradlew pixel6api31aospDebugAndroidTest`，此外还支持 `pixel4api30aospatdDebugAndroidTest` 和 `pixelcapi30aospatdDebugAndroidTest`。

### 编写测试规范

#### 插桩测试（UI 测试）

- UI 功能测试必须使用 `ComposeTestRule` 配合 `ComponentActivity`。
- 范围更大的集成测试存放在 `:app` 模块中，此类测试可以启动如 `MainActivity` 等完整 Activity。

#### 本地测试（单元测试）

- 使用 [kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines) 处理大多数协程断言
- 使用 [cashapp/turbine](https://github.com/cashapp/turbine) 进行复杂的数据流（Flow）与协程测试
- 使用 [google/truth](https://github.com/google/truth) 进行断言

## 持续集成（CI）

- CI 工作流定义在 `.github/workflows/*.yaml` 中，包含多项自动化检查。
- 截图测试由 CI 统一生成与比对，切勿将本地开发机生成的截图文件提交到仓库中。

## 版本控制与代码托管

- 本项目使用 Git 进行版本管理，托管在：https://github.com/android/nowinandroid
