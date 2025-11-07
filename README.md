
# Linux 内核构建 Actions

本项目包含了一套 GitHub Actions 工作流，用于构建不同用途的 Linux 内核，涵盖了从最小化的内建配置到功能齐全、可调试的开发版本。

## 工作流概览

仓库中提供了四个独立的工作流，每个都为特定的场景量身定制。你可以从 GitHub 仓库的 "Actions" 标签页手动触发它们。

| 工作流文件 | 主要目的 | 配置方式 | 调试支持 |
| :--- | :--- | :--- | :--- |
| `build-kernel-builtin.yaml` | **快速最小化构建** | 自动生成 (`defconfig`) | ❌ 否 |
| `build-kernel-module.yaml` | **定制化模块化构建** | 依赖 `build-kernel/` 目录下的外部文件 | ❌ 否 |
| `build-kernel-builtin-debug.yaml`| **通用的可调试构建** | 自动生成 + 自动开启调试选项 | ✅ **是** (自动开启调试选项) |
| `build-kernel-module-debug.yaml`| **可调试的模块化构建** | 依赖 `build-kernel/` 目录下的外部文件 | ✅ **是** (需开启 `DEBUG_INFO`) |

---

## 如何使用

1.  在你的 GitHub 仓库中，导航到 **Actions** 标签页。
2.  从左侧列表中选择你想要运行的工作流。
3.  点击 **"Run workflow"** (运行工作流) 按钮。
4.  填写所需的输入参数，然后开始构建。

### 通用输入参数

-   `repo`: Linux 内核的来源仓库 (例如, `torvalds/linux`, `amaindex/linux`)。
-   `branch`: 需要检出和构建的 git 分支 (例如, `master`, `v6.5`)。
-   `config_name`: (仅用于基于模块的工作流) 位于本项目 `build-kernel/` 目录下的配置文件名 (例如, `config-debian-12-edited`)。

---

## 工作流详解

### 1. 构建最小化内建内核 (`build-kernel-builtin.yaml`)

-   **描述**: 创建一个最小化的内核，所有功能都被直接编译进内核二进制文件 (`bzImage`) 中，不生成可加载的模块。
-   **配置**: 自动运行 `make defconfig` 来生成一份默认配置。
-   **适用场景**: 快速的编译测试、基础功能验证，或者不需要内核模块的环境。

### 2. 使用本地配置构建虚拟机内核 (`build-kernel-module.yaml`)

-   **描述**: 基于一个你提供的配置文件，构建一个模块化的内核。这适用于为虚拟机或生产系统创建一个功能丰富的内核。
-   **配置**: 需要一个放置在 `build-kernel/` 目录下的自定义配置文件。你需要通过 `config_name` 输入来指定文件名。
-   **适用场景**: 为特定的目标系统构建一个定制化的、生产级别的内核。

### 3. 构建最小化内建可调试内核 (`build-kernel-builtin-debug.yaml`)

-   **描述**: 这是获取可调试内核最便捷的方式。它会自动生成一份默认配置，然后在此基础上启用最关键的调试选项。
-   **配置**: 从 `make defconfig` 开始，然后自动启用 `CONFIG_DEBUG_INFO`, `CONFIG_FRAME_POINTER`, 和 `CONFIG_GDB_SCRIPTS`。
-   **适用场景**: 内核学习、练习 GDB 调试，或者在一个通用的、可调试的内核上快速验证一个 bug。

### 4. 构建可调试的虚拟机内核 (`build-kernel-module-debug.yaml`)

-   **描述**: 与“使用本地配置构建虚拟机内核”类似，但主要为**开发和调试**而设计。它会强制要求配置中启用调试信息，并提供 GDB 等调试器所需的 `vmlinux` 文件。
-   **配置**: 需要 `build-kernel/` 目录下的自定义配置文件，并且该文件**必须**开启 `CONFIG_DEBUG_INFO=y`。
-   **适用场景**: 内核开发、驱动调试、性能分析。

---

## 调试版本配置指南

对于 `build-kernel-module-debug.yaml` 工作流，你必须提供一个自定义的配置文件。为确保最佳的调试体验，请在你的配置文件中启用以下选项：

-   `CONFIG_DEBUG_INFO=y`: **(强制)** 将 DWARF 调试符号嵌入到 `vmlinux` 文件中。
-   `CONFIG_FRAME_POINTER=y`: **(强烈推荐)** 确保在调试器中能获得可靠的函数调用栈。
-   `CONFIG_GDB_SCRIPTS=y`: **(推荐)** 启用 GDB 辅助脚本，简化调试过程。

---

## 输出产物 (Artifacts)

运行成功后，工作流会上传一个或两个产物文件，这些文件会被保留90天。

-   **部署包 (`linux-kernel-package-*.tar.gz`)**: 一个 `tar.gz` 压缩包，包含了可启动的内核 (`vmlinuz`)、模块、头文件和配置。用于将内核部署到目标系统。
-   **调试符号 (`vmlinux-debug-*.zip`)**: 一个只包含未压缩的 `vmlinux` 二进制文件的 zip 包。这个文件通常非常大，供 GDB 等调试器使用，**运行内核本身并不需要它**。
