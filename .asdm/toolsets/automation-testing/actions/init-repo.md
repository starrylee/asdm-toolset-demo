# Instructions for init-repo action

## Metadata

```json
{
  "guid": "3f2a1b0c-9d8e-4f7a-b6c5-4e3d2c1b0a9f",
  "name": "init-repo",
  "displayName": "仓库初始化",
  "description": "搭建 behave + Playwright 测试仓库基础设施骨架，生成配置、hooks、核心基础设施代码、Step 注册表与标准目录结构",
  "toolset": {
    "guid": "9e8b7a6c-5d4f-4c3b-9a2e-1f0d8c7b6e5a",
    "id": "automation-testing",
    "name": "自动化测试工具集",
    "version": "0.0.1"
  },
  "scenario": "仓库初始化"
}
```

## Purpose

This instruction guides the AI model to 搭建测试仓库的基础设施骨架，为后续场景编写（`instantiate-requirements`）与测试代码生成（`generate-test-code`）提供统一的落脚点。It generates the behave + Playwright configuration, hooks, core infrastructure code, Step Registry, and standard directory skeleton in the target test repository.

## Language Setting

默认使用**中文（简体中文）**作为输出语言。所有生成的文件、注释和文档均使用中文，并遵循中文写作规范。

如需切换语言，可在工作区根目录创建 `.asdm/config.json` 配置：
```json
{
  "language": "zh"
}
```
支持的语言：`zh`（中文，默认）、`en`（英文）。

## Context Injection

Before 初始化仓库，the AI model should read the toolset context:

### Context Files to Read (Required)

1. **Toolset README.md** (Required)
   - Path: `.asdm/toolsets/automation-testing/README.md`
   - Purpose: 理解工具集整体定位、共享约定（Step 注册表、幂等、人工确认点）与目录规划

2. **仓库结构规范** (Required)
   - Path: `.asdm/toolsets/automation-testing/spec/repo-structure.md`
   - Purpose: 获取标准目录结构、各目录职责与文件命名规则

3. **命名规范** (Required)
   - Path: `.asdm/toolsets/automation-testing/spec/naming-convention.md`
   - Purpose: 生成的配置、类、模块命名遵循统一规范

4. **Step 注册表规范** (Required)
   - Path: `.asdm/toolsets/automation-testing/spec/step-registry.md`
   - Purpose: 初始化 Step 注册表文件时遵循其格式与维护规则

### Progressive Context Loading Strategy

1. 先读 README.md 与 `spec/repo-structure.md`，确定骨架形态。
2. 再读 `spec/naming-convention.md` 与 `spec/step-registry.md`，确定命名与注册表格式。
3. 项目上下文 `.asdm/contexts/index.md` 若存在则一并读取，了解被测项目信息。

## Steps to Init Repo

### 1. 收集初始化参数

向用户收集以下信息（未提供时逐项询问）：

- **目标路径**：测试仓库的根目录（默认为当前工作区）。
- **项目信息**：项目名称、简述（用于配置文件与 README）。
- **被测系统类型**：UI / API / 混合（决定生成的基础设施范围：纯 API 仓库可不生成浏览器工厂与截图留存）。
- **目标环境**：需要支持的环境列表（如 dev / test / staging），默认 dev。
- **运行模式**：全新搭建 or 已有仓库规范化改造（后者进入"检查并补齐"模式，见步骤 7）。

### 2. 环境检查

检查目标环境：

- Python 3.9+ 与 pip 是否可用（`python3 --version`）。
- 是否为 Git 仓库（非必须，仅提示）。
- 记录检查结果，异常项写入初始化报告的"待办项"。

### 3. 生成目录骨架

按 `spec/repo-structure.md` 创建标准目录：

- `features/`、`features/steps/`
- `pages/`（UI / 混合类型）
- `services/`
- `data/builders/`
- `config/`
- `reports/`（加入 `.gitignore`）
- 根目录放置 `requirements.txt`（或 `pyproject.toml`，按用户偏好）

每个目录附带简短 README 或注释说明其用途与放置规则。

### 4. 生成配置层

创建以下配置：

- **behave 配置**：`behave.ini` 或 `pyproject.toml` 的 behave 段（默认参数、格式化器、junit/allure 输出路径指向 `reports/`）。
- **多环境配置**：`config/` 下按环境分文件（如 `config/dev.yaml` / `config/test.yaml`），含被测系统 base_url、超时、凭据引用（凭据不落明文，走环境变量）。
- **Playwright 依赖**：`requirements.txt` 中包含 `behave`、`playwright`、`pytest`（可选）等版本锁定；提示用户执行 `playwright install` 安装浏览器。

### 5. 生成 hooks 与核心基础设施代码

- **`features/environment.py`**：
  - `before_all` / `after_all`：全局配置加载、Playwright 实例与浏览器生命周期。
  - `before_scenario` / `after_scenario`：上下文隔离；失败时截图 / trace / 视频留存到 `reports/`。
  - `before_tag`：标签驱动逻辑（如 `@api` 跳过浏览器启动）。
- **核心基础设施**（按被测系统类型裁剪）：
  - 基础 Page 基类（封装通用等待、断言辅助、截图）。
  - 浏览器与上下文工厂（支持 headless 配置）。
  - API 客户端封装（请求构造、鉴权、响应校验基础方法）。
  - 测试数据 Builder 基类。
  - 日志与报告集成。

所有代码遵循 `spec/naming-convention.md`，关键处附中文注释。

### 6. 初始化 Step 注册表

- 路径：`.asdm/workspace/automation-testing/step-registry.md`（项目级共享）。
- 按 `spec/step-registry.md` 的格式创建空表（含列：Step 表达式、参数、类型 Given/When/Then、所在文件、复用说明）。
- 表头附加维护规则摘要。

### 7. 幂等与改造模式处理

- **全新搭建**：直接落盘。
- **改造模式**：先扫描已有文件，仅补齐缺失项；已存在的文件一律**不覆盖**，差异项列入报告由用户决定。重复执行本 Action 时同样按"检查并补齐"处理。

### 8. 生成初始化报告并确认

生成 `.asdm/workspace/automation-testing/repo-init-report.md`：

- 已创建 / 已存在的文件清单。
- 环境检查结果与待办项（如需手动安装浏览器、配置凭据环境变量）。
- 下一步指引（`/instantiate-requirements`）。

向用户展示报告摘要，确认后收尾。

## Execution Guidelines

### When to Use This Action

Use this action when:
- 全新搭建 behave + Playwright 测试仓库。
- 已有测试仓库需要规范化改造（检查并补齐，不覆盖已有文件）。

### 约束与最佳实践

- 不安装任何依赖或浏览器到用户环境，仅生成配置并给出安装命令提示。
- 被测系统类型为纯 API 时跳过浏览器工厂、Page 基类与截图留存。
- 所有产物可重复执行幂等：已存在文件跳过并记录。
- 凭据、token 等敏感信息只允许出现在环境变量或本地忽略文件模板中，不写入仓库文件。

## Usage

To use this instruction, the AI model should:
1. 读取 README.md 与 `spec/` 下的相关规范。
2. 收集目标路径、项目信息、被测系统类型、目标环境与运行模式。
3. 执行环境检查。
4. 按规范生成目录骨架、配置、hooks、核心基础设施与 Step 注册表。
5. 以幂等方式落盘（改造模式只补齐不覆盖）。
6. 生成初始化报告并向用户确认。

## Output Summary

After completing the action, the following artifacts will be generated:

- 目录骨架：`features/`、`features/steps/`、`pages/`、`services/`、`data/builders/`、`config/`、`reports/`
- 配置文件：behave 配置、多环境配置、`requirements.txt`（或 `pyproject.toml`）
- 基础设施代码：`features/environment.py`、Page 基类、浏览器 / 上下文工厂、API 客户端封装、Builder 基类
- Step 注册表：`.asdm/workspace/automation-testing/step-registry.md`
- 初始化报告：`.asdm/workspace/automation-testing/repo-init-report.md`

源码资产保存在目标测试仓库内，过程产物（注册表、报告）保存在 `.asdm/workspace/automation-testing/`。
