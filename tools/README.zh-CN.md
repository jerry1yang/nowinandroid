# 开发工具集 (`tools`)

本目录包含用于本地开发环境初始化、代码格式规范以及 Git 提交/推送阶段自动化质量检查的工具脚本与配置方案。

## 目录结构

```text
tools/
├── setup.sh                     # 开发者环境一键初始化脚本
├── pre-push                     # Git pre-push 客户端钩子脚本
└── nowinandroid-codestyle.xml   # Android Studio / IntelliJ IDEA 代码风格方案
```

---

## 工具与配置文件详解

### 1. `setup.sh`（环境一键初始化脚本）

新克隆仓库后，推荐在项目根目录下首先执行此脚本：

```bash
./tools/setup.sh
```

**脚本主要功能：**
* **自动安装 `commit-msg` 钩子**：从 Google Gerrit 官方下载标准的 `commit-msg` 脚本并写入 `.git/hooks/commit-msg`，自动在提交信息中生成标准的 `Change-Id` 标识。
* **自动安装 `pre-push` 钩子**：将 `tools/pre-push` 复制至 `.git/hooks/pre-push` 并赋予执行权限。
* **输出推荐的 IDE 配置建议**：
  * **删除修改行的末尾空格**：*Settings -> Editor -> General -> Remove trailing spaces on: Modified lines*
  * **保证文件以换行符结尾**：*Settings -> Editor -> General -> Ensure every saved file ends with a line break*
  * **保存时自动优化导包**：*Settings -> Editor -> General -> Auto Import -> Optimize imports on the fly*

---

### 2. `pre-push`（Git 推送前置检查钩子）

该脚本在开发者执行 `git push` 时、提交推送到远端服务器前自动触发运行。

**核心检查逻辑：**
1. **草稿提交拦截**：
   * 自动扫描待推送的 Commit 信息。
   * 如果 Commit 标题以 `WIP`（Work In Progress，开发中草稿）或 `stash` 开头，直接阻断推送，防止不完整的草稿代码误推送到远端。
2. **本地预提交检查（Presubmit）**：
   * 当检测到向 Gerrit 评审分支（如 `refs/for/*`）推送时，自动在本地触发执行完整质量检查：
     ```bash
     ./gradlew check
     ```
   * 包含编译、代码风格检查以及单元测试，一旦失败立即中止 Push，在本地提前发现问题。

---

### 3. `nowinandroid-codestyle.xml`（IDE 代码风格方案）

适用于 **Android Studio** 或 **IntelliJ IDEA** 的官方统一代码格式化方案。

**导入方法：**
1. 打开 Android Studio 设置：*Settings* (macOS 上为 *Preferences*) -> *Editor* -> *Code Style*。
2. 点击 *Scheme* 右侧的齿轮图标 ⚙️，选择 **Import Scheme...** -> **IntelliJ IDEA code style XML**。
3. 选择本项目中的 `tools/nowinandroid-codestyle.xml` 文件并确认应用。

**主要规范规则：**
* **单行最大字符数**：限制为 100 字符（`RIGHT_MARGIN = 100`）。
* **禁止通配符导包（No Star Imports）**：强制显式导入每个类（`NAME_COUNT_TO_USE_STAR_IMPORT = 99`），杜绝类似 `import com.example.*` 的通配符导入。
* **统一导包排序**：规范定义了 Android、Google、第三方库与 Java 标准库的导入分组及空行分割。
* **适配 Kotlin 与 Compose 缩进**：关闭多余的参数列表与链式调用二次缩进，与 Jetpack Compose 声明式 UI 代码风格深度契合。
