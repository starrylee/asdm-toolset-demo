# ASDM Toolset - 自动化测试工具集

toolset-id: automation-testing
toolset-name: 自动化测试工具集
version: 0.0.1
updated-date: 2026-11-12
toolset-description: 基于 behave + Playwright 的自动化测试工程脚手架与测试代码生成工具集

## Overview

自动化测试工具集面向采用 BDD（行为驱动开发）实践的测试工程师、测试开发工程师与全栈工程师，提供一条从"需求验收标准"到"可执行自动化测试代码"的 AI 辅助流水线。工具集以 Python 生态的 **behave**（Gherkin 执行引擎）与 **Playwright**（UI / API 自动化驱动）为技术底座，把"搭脚手架、写场景、写代码"三件重复性高、规范性强的工作沉淀为可复用的 Action。

传统自动化测试落地常见三类问题：测试仓库结构因人而异、基础设施（浏览器管理、环境配置、报告、hooks）反复重造；验收标准与测试场景之间依赖人工转写，容易遗漏、歧义或粒度不一；Page Object、Step Definition 与测试数据的编写风格不统一，Step 重复定义、选择器无法追溯。本工具集通过"统一骨架 + 统一场景格式 + 统一代码生成规则"来解决这些问题，使不同团队、不同项目产出的测试资产结构一致、可维护、可审查。

工具集由三个核心 Feature 串联而成：**仓库初始化**一次性生成测试工程骨架；**需求实例化**把 User Story 的验收标准转化为 Gherkin `.feature` 文件；**测试代码生成**解析 `.feature` 文件并生成 Page Object Model、Step Definitions、API Service 与测试数据 Builder。三者既可顺序使用，也可单独使用（例如在已有测试仓库中仅使用后两个 Feature）。

## Features

### Common features

所有 Feature 共享以下约定与能力：

- **统一技术栈**：测试执行基于 behave + Playwright（Python），不引入其他 BDD 或 UI 自动化框架。
- **Step 注册表（Step Registry）**：全局登记已有 Step 的表达式、参数、所在文件与复用说明；生成新场景或新代码前先查表复用，避免 Step 重复定义与歧义匹配。
- **可追溯性**：每个 `.feature` 场景可追溯到 User Story 与验收标准编号；每个 Step 可追溯到场景；每个 Page Object 的选择器可追溯到页面 / 源码依据。
- **增量与幂等**：重复执行同一 Action 不会覆盖开发者手工修改的内容；已存在的文件按"合并 / 追加 / 跳过"策略处理，并在结果中列出变更清单。
- **规范先行**：生成规则统一沉淀在 `spec/` 目录（命名规范、目录规范、Gherkin 编写规范、代码风格规范），Action 只负责流程编排，不内嵌规则细节。
- **人工确认点**：关键产物（场景清单、Page Object 设计、Step 复用决策）在写入仓库前先给出摘要，由用户确认后再落盘。
- **工作区隔离**：中间产物与分析记录统一放在 `.asdm/workspace/` 下，不污染测试仓库源码目录。

### Feature 1: 仓库初始化

搭建测试仓库基础设施骨架，为后续场景编写与代码生成提供统一的落脚点。

- **配置层**：生成 behave 配置（`behave.ini` / `pyproject.toml` 中的 behave 段）、Playwright 依赖与浏览器安装配置、多环境配置（如 dev / test / staging）及 `requirements.txt` / `pyproject.toml`。
- **hooks 层**：生成 `features/environment.py`，包含 `before_all` / `after_all`（全局配置加载、浏览器与 Playwright 实例生命周期）、`before_scenario` / `after_scenario`（上下文隔离、失败截图 / trace / 视频留存）、`before_tag` 等标签驱动逻辑。
- **核心基础设施代码**：基础 Page 基类、浏览器与上下文工厂、API 客户端封装、测试数据 Builder 基类、日志与报告（如 Allure / HTML 报告）集成、通用等待与断言工具。
- **Step 注册表**：初始化 Step 注册表文件及其维护规则，后续每次生成 Step 后自动更新。
- **目录骨架**：生成 `features/`、`features/steps/`、`pages/`、`services/`、`data/builders/`、`config/`、`reports/` 等标准目录及 README 说明。
- **适用场景**：全新测试仓库搭建；已有仓库的规范化改造（以"检查并补齐"模式运行，不覆盖已有文件）。

### Feature 2: 需求实例化

分析 User Story 的验收标准，生成 Gherkin `.feature` 文件，把抽象需求转化为具体、可验证的示例。

- **输入**：User Story 文档（含验收标准）、可选的业务规则 / 原型 / 接口说明等补充材料。
- **分析**：逐条拆解验收标准，识别业务规则、边界值、异常流与数据变体，检查歧义与遗漏并向用户澄清。
- **示例化**：采用"实例化需求（SBE）"思路，用具体数据而非抽象描述表达期望行为；合理使用 `Background`、`Scenario Outline` + `Examples`、数据表与标签（如 `@smoke` `@regression` `@ui` `@api`）。
- **输出**：符合 Gherkin 编写规范的 `.feature` 文件，以及"验收标准 → 场景"的覆盖追溯矩阵；Then 步骤必须是可验证的具体断言，不使用"显示正确""成功"等模糊描述。
- **复用**：生成时优先复用 Step 注册表中已有的 Step 表达式，保持措辞一致。

### Feature 3: 测试代码生成

解析 `.feature` 文件，生成可直接运行的自动化测试代码。

- **Page Object Model**：按页面 / 组件拆分 Page Object，封装元素定位与操作；选择器优先使用稳定属性（如 `data-testid`、role、label），并记录选择器的来源依据以便追溯与维护。
- **Step Definitions**：为每个 Step 生成 behave 步骤实现，调用 Page Object / API Service 完成操作与断言；对照 Step 注册表复用已有实现，仅为新增 Step 生成新代码，并同步更新注册表。
- **API Service**：为涉及接口调用的场景生成 API Service 封装（请求构造、鉴权、响应校验），支持 UI 与 API 混合场景（如通过 API 准备数据、UI 验证结果）。
- **测试数据 Builder**：为场景中的数据表与 Examples 生成 Builder，支持默认值、链式覆盖与按场景定制，避免在 Step 中硬编码数据。
- **校验**：生成后执行静态检查（语法、导入、Step 歧义检测）与 `behave --dry-run`，确认所有 Step 均已实现。

## Toolset Installation Process

`INSTALL.md` will setup the toolset with the following steps:

1. **环境检查**：确认已安装 Python 3.9+、pip 与 Node.js（用于 Playwright 浏览器安装，如需要）；检查当前目录是否为 Git 仓库。
2. **目录创建**：在项目中创建 `.asdm/toolsets/automation-testing/` 与 `.asdm/workspace/` 目录。
3. **文件部署**：将 `README.md`、`actions/`、`spec/` 部署到工具集目录。
4. **IDE 集成**：为所使用的 AI 助手（如 VS Code / IDEA 中的 AI 插件）注册各 Action 的调用入口。
5. **依赖提示**：提示安装 `behave`、`playwright` 等依赖及浏览器（实际安装由"仓库初始化"Action 在目标测试仓库中完成）。
6. **安装验证**：检查目录结构与文件完整性，输出安装结果与后续使用指引。

安装完成后无需额外配置即可使用，详细步骤与各 IDE 的差异说明详见 `INSTALL.md`。

## Toolset Workflow

Once 自动化测试工具集 is installed, user can use the following commands:

| 命令（Action） | 对应 Feature | 说明 | 主要输入 | 主要输出 |
|---|---|---|---|---|
| `init-repo` | 仓库初始化 | 搭建 behave + Playwright 测试仓库骨架 | 项目信息、被测系统类型（UI / API / 混合）、目标环境 | 配置文件、`environment.py`、核心基础设施代码、Step 注册表 |
| `instantiate-requirements` | 需求实例化 | 将 User Story 验收标准转化为 Gherkin | User Story 文档 | `.feature` 文件、验收标准覆盖矩阵 |
| `generate-test-code` | 测试代码生成 | 由 `.feature` 生成 POM、Steps、Service、Builder | `.feature` 文件 | Page Object、Step Definitions、API Service、数据 Builder、更新后的 Step 注册表 |

推荐工作流：

```
init-repo  →  instantiate-requirements  →  generate-test-code  →  运行 behave 验证
 (一次性)        (每个 User Story)          (每个 .feature)         (本地 / CI)
```

1. 新项目先执行 `init-repo` 完成骨架搭建（每个测试仓库仅需一次）。
2. 每拿到一个 User Story，执行 `instantiate-requirements` 生成并评审 `.feature` 文件。
3. 评审通过后执行 `generate-test-code` 生成测试代码，并运行 `behave` 验证。
4. 场景或需求变更时，重新执行第 2、3 步，工具集按增量方式更新产物。

## Toolset Structure

The 自动化测试工具集 toolset has the following structure:

```
.asdm/
└── toolsets/
    └── automation-testing/
        ├── INSTALL.md
        ├── README.md
        ├── actions/
        │   ├── init-repo.md
        │   ├── instantiate-requirements.md
        │   └── generate-test-code.md
        └── spec/
            ├── repo-structure.md
            ├── gherkin-writing-guide.md
            ├── naming-convention.md
            ├── page-object-guide.md
            ├── step-definition-guide.md
            ├── step-registry.md
            └── test-data-builder-guide.md
```

目录说明：

- `actions/`：每个 Action 一个文件，描述触发条件、输入输出、执行步骤与人工确认点。
- `spec/`：规则与规范文件，供各 Action 引用，包括仓库目录规范、Gherkin 编写规范、命名规范、Page Object / Step Definition / 数据 Builder 编写规范，以及 Step 注册表格式与维护规则。

## Toolset Workspace

The 自动化测试工具集 toolset has the following workspace structure:

> 路径约定：feature 相关产物第一级路径为 `.asdm/workspace/<feature-id>-<feature-name>/`（特性目录，其下再按用途分子目录）；`.asdm/workspace/<workspace-type>/` 仅用于跨 feature 的项目级产物。

```
.asdm/workspace/
├── <feature-id>-<feature-name>/          # 特性目录（feature 级产物）
│   ├── requirements/                      # 需求实例化的分析过程产物
│   │   ├── acceptance-criteria-analysis.md   # 验收标准拆解与澄清记录
│   │   └── coverage-matrix.md                # 验收标准 → 场景 覆盖追溯矩阵
│   ├── features/                          # 评审中的 .feature 草稿（确认后写入测试仓库）
│   │   └── <feature-name>.feature
│   └── codegen/                           # 测试代码生成的过程产物
│       ├── page-object-plan.md               # Page Object 设计与选择器依据
│       ├── step-reuse-report.md              # Step 复用 / 新增决策清单
│       └── generation-report.md              # 本次生成的文件变更清单与校验结果
└── automation-testing/                    # 项目级产物（<workspace-type> = automation-testing）
    ├── repo-init-report.md                   # 仓库初始化结果与待办项
    └── step-registry.md                      # 跨 feature 共享的 Step 注册表
```

说明：

- 最终的 `.feature` 文件、Page Object、Step Definitions 等**源码资产**写入测试仓库对应目录（`features/`、`pages/`、`services/` 等），不放在 workspace 中；workspace 仅保存分析、设计、报告等过程产物。
- 项目级的 Step 注册表为跨 feature 共享，因此放在 `automation-testing/` 项目级目录下，而非某个特性目录内。
- 过程产物建议随代码一并纳入版本管理，以保留需求到测试的追溯链路。

## Copyright & License

Copyright (c) 2026 LeansoftX.com & iSoftStone. All rights reserved.

Licensed under the PROPRIETARY SOFTWARE LICENSE. See [LICENSE](LICENSE) in the project root for license information.
