# Spotless 版权头配置 (`spotless`)

本目录存放 Now in Android 项目在代码格式化与合规性检查时所使用的 **Apache 2.0 开源许可证版权头（License / Copyright Header）模板**。

配合项目中的 Gradle 插件 [Spotless](https://github.com/diffplug/spotless) 以及 `build-logic` 约定插件，该配置确保了全工程源码文件在开源合规性、格式排版以及规范性上的一致。

---

## 目录结构

```text
spotless/
├── copyright.kt     # 用于 Kotlin 源码文件（.kt）的版权头模板
├── copyright.kts    # 用于 Gradle 构建脚本（.kts）的版权头模板
└── copyright.xml    # 用于 Android 清单与资源文件（.xml）的版权头模板
```

---

## 模板说明与工作机制

### 1. 动态年份占位符 (`$YEAR`)
每个模板文件中均包含 `$YEAR` 占位符。Spotless 在处理文件时：
* 对已有文件：保留该文件原有的创建年份；
* 对新创建的文件：自动注入当前年份。

### 2. 匹配规则（定义于 `build-logic/.../Spotless.kt`）

* **Kotlin 源码 (`src/**/*.kt`)**：
  * 应用 `spotless/copyright.kt` 头部模板；
  * 使用 `ktlint` 按根目录 `.editorconfig` 配置进行代码格式化；
  * 强制文件末尾保留空行（`endWithNewline()`）。
* **Gradle 构建脚本 (`*.kts`)**：
  * 应用 `spotless/copyright.kts` 头部模板；
  * 正则匹配首行非注释内容 `(^(?![\\/ ]\\*).*$)` 进行插入。
* **Android XML 资源与清单 (`src/**/*.xml`)**：
  * 应用 `spotless/copyright.xml` 头部模板；
  * 正则匹配 `(<[^!?])`，确保版权头精确插入在 `<?xml version="1.0" ... ?>` 声明之后、首个 XML 标签之前。

---

## 常用 Gradle 命令与开发实战

### 1. 自动格式化与补全版权头（日常最常用）
```bash
./gradlew spotlessApply
```
* **自动排版**：按照 `ktlint` 与根目录 `.editorconfig` 规范排版所有代码；
* **自动注入协议头**：如果新建了代码（`.kt`）或配置文件（`.xml` / `.kts`），**自动在顶部插入对应的 Apache 2.0 版权头**，无需手动复制粘贴；
* **自动修正空行**：确保文件末尾包含规范的结束换行。

### 2. 精准针对单模块提速
当项目包含数十个子模块时，为节省时间，可仅对当前正在开发的单个模块执行格式化：
```bash
# 仅格式化 feature:foryou:impl 模块
./gradlew :feature:foryou:impl:spotlessApply

# 仅格式化 app 模块
./gradlew :app:spotlessApply

# 仅格式化核心数据层
./gradlew :core:data:spotlessApply
```

### 3. 检查合规性（CI 自动化门禁）
```bash
./gradlew spotlessCheck
```
* 检查全工程是否有文件缺少版权头或存在排版违规；
* 若有违规，任务将失败报错并指出具体文件与行号，防止不合规代码合并入主分支。

---

## 开发工作流与实用技巧

### 1. Android Studio 侧边栏图形化运行
在 Android Studio 右侧展开 **Gradle** 工具面板：
`项目根目录 -> Tasks -> verification -> spotlessApply`，双击即可一键执行。

### 2. 结合 Git Hooks 自动化（彻底免人工操心）
可配置本地 Git 预提交钩子，在每次 `git commit` 时自动格式化有变动的文件：
在本地 `.git/hooks/pre-commit` 中添加：
```bash
#!/bin/bash
./gradlew spotlessApply
git add -u
```
赋予执行权限后，每次执行 `git commit` 都会自动先跑格式化并更新暂存区，彻底杜绝因格式或版权头遗漏导致远程 CI 挂掉。

### 3. 常见避坑说明
* **CI 报错 “Missing license header”**：通常是因为新建了文件但提交前未运行 `spotlessApply`，在本地补跑一次并提交即可修复；
* **Compose 尾随逗号（Trailing Comma）**：Spotless 格式化后可能在函数参数末尾保留逗号，这是 [`.editorconfig`](../.editorconfig) 专为 Compose 声明式 UI 开启的优化，能有效减少 Git Diff 冲突，属正常表现。
