# Flutter Build

一个功能强大的 GitHub Actions 工作流，用于自动化构建 Flutter 应用并支持多平台发布。

## 功能特性

- 🚀 **多平台构建支持**：支持 Web、macOS、iOS、Windows、Linux、Android 六大平台
- 🔄 **自动版本管理**：自动生成版本号和构建报告
- 📦 **产物打包发布**：自动打包构建产物并创建 GitHub Release
- 🔧 **灵活的 Flutter 版本管理**：支持自动检测或手动指定 Flutter 版本
- 🛠️ **智能环境配置**：自动检测 JDK 版本、安装依赖
- 📊 **构建报告生成**：生成详细的构建信息报告
- ✅ **平台支持检测**：自动跳过不支持的平台构建

## 支持的平台

| 平台 | 运行环境 | 输出格式 |
|------|---------|---------|
| Web | ubuntu-24.04 | web |
| macOS | macos-14 | .app (tar.gz) |
| iOS | macos-14 | .app (tar.gz, 无签名) |
| Windows | windows-2022 | .zip |
| Linux | ubuntu-24.04 | tar.gz |
| Android | ubuntu-24.04 | APK (split-per-abi) |

## 使用方法

### 前置条件

1. 将本工作流文件复制到您的 Flutter 项目的 `.github/workflows/` 目录下
2. 确保您的 GitHub 仓库具有相应的工作流权限

### 手动触发工作流

1. 进入 GitHub 仓库的 **Actions** 标签页
2. 选择 **Flutter Build** 工作流
3. 点击 **Run workflow** 按钮
4. 填写以下参数：

#### 输入参数说明

| 参数 | 必填 | 默认值 | 说明 |
|------|------|--------|------|
| `repository_url` | ✅ 是 | - | Git 仓库 URL |
| `checkout_options` | ❌ 否 | `--depth=1` | Git 检出参数（如 `-b branch_name --depth=1`） |
| `artifacts_pattern` | ❌ 否 | `*.tar.gz,*.zip,build/app/outputs/flutter-apk/*.apk` | 上传的产物匹配模式 |
| `flutter_version_name` | ❌ 否 | `auto` | Flutter 版本（auto 或指定版本） |
| `flutter_channel_name` | ❌ 否 | `stable` | Flutter 通道（stable/beta/dev/master/any） |

#### Flutter 版本选项

- `auto` - 从 pubspec.yaml 自动检测
- `3.x`, `2.x` - 主版本
- `3.41.x`, `3.38.x`, `3.22.x`, `3.19.x`, `3.16.x`, `3.13.x`, `3.10.x`, `3.7.x`, `3.3.x`, `2.10.x`, `2.8.x`, `2.5.x`, `2.2.x` - 具体版本

### 示例配置

#### 示例 1：使用默认配置

```
repository_url: https://github.com/username/flutter-app.git
```

#### 示例 2：指定分支和 Flutter 版本

```
repository_url: https://github.com/username/flutter-app.git
checkout_options: -b develop --depth=1
flutter_version_name: 3.22.x
flutter_channel_name: stable
```

#### 示例 3：使用最新的 dev 通道

```
repository_url: https://github.com/username/flutter-app.git
flutter_version_name: auto
flutter_channel_name: dev
```

## 工作流程

1. **检出代码** - 从指定的仓库 URL 克隆代码
2. **平台支持检测** - 检查项目是否支持目标平台
3. **环境配置** - 自动设置 JDK、Flutter 环境
4. **依赖安装** - 运行 `flutter pub get`
5. **应用构建** - 根据平台执行对应的构建命令
6. **产物打包** - 将构建产物压缩打包
7. **版本生成** - 生成版本信息（版本名称、版本代码、提交哈希）
8. **发布 Release** - 创建 GitHub Release 并上传产物

## 版本命名规则

构建产物的版本命名遵循以下格式：

- **版本名称**: `{日期时间}-{提交哈希}` (例如: `20260226-143022-a1b2c3d`)
- **版本代码**: GitHub 运行编号
- **版本哈希**: Git 短提交哈希
- **Release 标签**: `v{运行编号}_{仓库名}` (例如: `v123_flutter-app`)

## 构建报告

每次构建都会生成一份详细的构建报告 `build-report.md`，包含：

- 平台信息
- JDK 版本（Android 平台）
- 版本名称和代码
- 仓库 URL 和检出参数
- 提交哈希
- GitHub Actions 运行链接

## 项目要求

### 平台目录结构

工作流会根据项目根目录下的平台文件夹是否存在来判断是否支持该平台：

```
project-root/
├── web/          # Web 平台支持
├── macos/        # macOS 平台支持
├── ios/          # iOS 平台支持
├── windows/      # Windows 平台支持
├── linux/        # Linux 平台支持
└── android/      # Android 平台支持
```

### Android 平台要求

- 项目需包含 `android/gradle/wrapper/gradle-wrapper.properties`
- 工作流会自动根据 Gradle 版本选择合适的 JDK 版本：
  - Gradle 9+: JDK 21
  - Gradle 8: JDK 17
  - Gradle 7: JDK 11

### Linux 平台要求

- 自动安装 `ninja-build` 和 `libgtk-3-dev` 依赖

## 注意事项

1. **iOS 构建无签名** - iOS 平台的构建使用 `--no-codesign` 选项，不包含代码签名
2. **Android APK 分离** - Android 平台使用 `--split-per-abi` 生成针对不同 CPU 架构的 APK
3. **并发控制** - 工作流设置为不取消进行中的任务，允许多个平台并行构建
4. **错误继续** - 单个平台构建失败不会中断其他平台的构建（`continue-on-error: true`）

## License

MIT
