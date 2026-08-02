# mermaid-rust-kmp

`mermaid-rust-kmp` 为 [mermaid-rs-renderer](https://crates.io/crates/mermaid-rs-renderer) 提供 Kotlin Multiplatform 绑定。它通过 Rust 原生库（Android/JVM）或 WebAssembly（Wasm/JS）将 Mermaid 文本渲染为 SVG 字符串。

## 支持平台

| 平台 | 状态 |
| --- | --- |
| Android | 支持 |
| JVM / Desktop | 支持 |
| Kotlin/Wasm 浏览器 | 支持 |
| Kotlin/JS | 当前未支持原生渲染 |
| iOS | 当前未支持原生渲染 |

## 添加依赖

发布的 Kotlin Multiplatform 库坐标为：

```kotlin
implementation("io.github.storytellerf:mermaidffi-kmp:<version>")
```

在 Kotlin Multiplatform 项目中，将依赖添加到需要使用渲染器的平台 source set。例如，Android 和 JVM 可共享同一个 source set：

```kotlin
kotlin {
    sourceSets {
        val jvmAndAndroidMain by creating {
            dependencies {
                implementation("io.github.storytellerf:mermaidffi-kmp:<version>")
            }
        }
    }
}
```

Kotlin/Wasm 目标还需要使用随库发布的 npm 包：

```kotlin
wasmJsMain.dependencies {
    implementation("io.github.storytellerf:mermaidffi-kmp:<version>")
    implementation(npm("mermaid-ffi-wasm", "<version>"))
}
```

## 使用

导入 `com.storyteller_f.mermaid_kmp` 下的函数，并传入 Mermaid 语法；返回值是 SVG 文本。

```kotlin
import com.storyteller_f.mermaid_kmp.renderMermaid

val svg = renderMermaid(
    """flowchart LR
        A[Start] --> B{Decision}
        B -->|Yes| C[OK]
        B -->|No| D[Cancel]
    """.trimIndent(),
)
```

可用 API：

- `renderMermaid(input)`：使用默认渲染选项。
- `renderMermaidWithSpacing(input, nodeSpacing, rankSpacing)`：调整节点和层级间距。
- `renderMermaidClassicTheme(input)`：使用 Mermaid 默认（classic）主题。

输入不被底层渲染器支持时，调用会抛出异常；请在 UI 或服务边界捕获并向用户显示错误。

## 示例应用

[`sample/`](./sample) 是一个 Compose Multiplatform 示例，包含 Android、Desktop、Web（JS/Wasm）和 iOS 入口。它提供 Mermaid 编辑框和 SVG 输出预览文本。

```sh
cd sample

# Android APK
./gradlew :androidApp:assembleDebug

# Desktop 应用
./gradlew :desktopApp:run

# Kotlin/Wasm 浏览器应用
./gradlew :webApp:wasmJsBrowserDevelopmentRun
```

示例中的 iOS 与 Kotlin/JS 目标会显示“尚未支持原生渲染”的提示；Wasm、Android 和 JVM 目标会调用该库。

## 从源码构建

构建需要 Rust stable、Android NDK/SDK、JDK 21、Node.js/npm、TypeScript、`wasm-opt` 和 BoltFFI CLI 0.26.1。CI 使用以下流程生成绑定并验证生成的 KMP 项目：

```sh
cargo install boltffi_cli --version 0.26.1
cargo install wasm-opt
npm install -g typescript

cd mermaid-ffi
boltffi pack kmp --release --experimental
boltffi pack wasm --release
cd ..
./patch-ffi.sh

cd mermaid-ffi/dist/kotlin-multiplatform
./gradlew build -Pversion=0.0.0-local -Pgroup=io.github.storytellerf
```

`patch-ffi.sh` 会为生成的 KMP 工程补充 Maven 发布设置、Kotlin/Wasm 配置和 Wasm 实现。若需要让生成的 KMP 工程使用已发布的 npm 包，可传入版本号：`./patch-ffi.sh <version>`。

## 发布

推送形如 `v<version>` 的 Git tag 会触发 GitHub Actions：发布 `mermaid-ffi-wasm` 到 npm，并发布 `mermaidffi-kmp` 到 Maven Central。
