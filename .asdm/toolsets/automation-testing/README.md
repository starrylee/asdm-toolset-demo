# ASDM Toolset - 自动化测试工具集

toolset-id: automation-testing
toolset-name: 自动化测试工具集
version: 0.0.1
updated-date: 2026-11-12
toolset-description: 基于 behave + Playwright 的自动化测试工程脚手架与测试代码生成工具集

## Overview

[开发者补充：2-3 段描述本工具集的功能、目标用户与解决的问题]

## Features

### Common features

[开发者补充]

### Feature 1: 仓库初始化
[开发者补充：搭建测试仓库基础设施骨架（behave + Playwright 配置、environment.py hooks、核心基础设施代码、Step 注册表）]

### Feature 2: 需求实例化
[开发者补充：分析 User Story 验收标准，生成 Gherkin .feature 文件]

### Feature 3: 测试代码生成
[开发者补充：解析 .feature 文件，生成 Page Object Model、Step Definitions、API Service 与测试数据 Builder]

[开发者按需增加更多 Feature]

## Toolset Installation Process

`INSTALL.md` will setup the toolset with the following steps:

[开发者简要描述安装过程，详见 INSTALL.md]

## Toolset Workflow

Once 自动化测试工具集 is installed, user can use the following commands:

[开发者列出可用的命令/工作流]

## Toolset Structure

The 自动化测试工具集 toolset has the following structure:

```
.asdm/
└── toolsets/
    └── automation-testing/
        ├── INSTALL.md
        ├── README.md
        ├── actions/
        │   ├── <action-name>.md
        │   └── ...
        └── spec/
            ├── <spec-name>.md
            └── ...
```

## Toolset Workspace

The 自动化测试工具集 toolset has the following workspace structure:

> 路径约定：feature 相关产物第一级路径为 `.asdm/workspace/<feature-id>-<feature-name>/`（特性目录，其下再按用途分子目录）；`.asdm/workspace/<workspace-type>/` 仅用于跨 feature 的项目级产物。

```
.asdm/workspace/<workspace-type>/
├── <files>
└── <directories>
```

[开发者按工具集需要补充]

## Copyright & License

Copyright (c) 2026 LeansoftX.com & iSoftStone. All rights reserved.

Licensed under the PROPRIETARY SOFTWARE LICENSE. See [LICENSE](LICENSE) in the project root for license information.
