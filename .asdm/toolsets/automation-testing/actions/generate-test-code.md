# Instructions for generate-test-code action

## Metadata

```json
{
  "guid": "2b3c4d5e-6f7a-4b8c-9d0e-1a2b3c4d5e6f",
  "name": "generate-test-code",
  "displayName": "测试代码生成",
  "description": "解析 .feature 文件，查询 Step 注册表检测复用，生成 Page Object Model、Step Definitions、API Service 与测试数据 Builder，并更新 Step 注册表",
  "toolset": {
    "guid": "9e8b7a6c-5d4f-4c3b-9a2e-1f0d8c7b6e5a",
    "id": "automation-testing",
    "name": "自动化测试工具集",
    "version": "0.0.1"
  },
  "scenario": "测试代码生成"
}
```

## Purpose

This instruction guides the AI model to 解析 `.feature` 文件并生成可直接运行的自动化测试代码。It produces Page Object Model classes, Step Definitions, API Services, and test data Builders, reusing existing Steps via the Step Registry, and validates the result with static checks and `behave --dry-run`.

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

Before 生成测试代码，the AI model should read the toolset context:

### Context Files to Read (Required)

1. **Toolset README.md** (Required)
   - Path: `.asdm/toolsets/automation-testing/README.md`
   - Purpose: 理解工具集共享约定（Step 注册表、幂等、人工确认点）

2. **Page Object 规范** (Required - UI 场景)
   - Path: `.asdm/toolsets/automation-testing/spec/page-object-guide.md`
   - Purpose: Page Object 拆分、选择器策略与封装规则

3. **Step Definition 规范** (Required)
   - Path: `.asdm/toolsets/automation-testing/spec/step-definition-guide.md`
   - Purpose: Step 实现的代码风格、结构组织

4. **测试数据 Builder 规范** (Required)
   - Path: `.asdm/toolsets/automation-testing/spec/test-data-builder-guide.md`
   - Purpose: Builder 的默认值、链式覆盖与场景定制规则

5. **命名规范** (Required)
   - Path: `.asdm/toolsets/automation-testing/spec/naming-convention.md`
   - Purpose: 生成的类、方法、文件命名

6. **Step 注册表** (Required)
   - Path: `.asdm/workspace/automation-testing/step-registry.md`
   - Purpose: 检测可复用的 Step 实现，避免重复定义

### Progressive Context Loading Strategy

1. 先读 README.md、Step 注册表与目标 `.feature` 文件。
2. UI 场景再读 `spec/page-object-guide.md`，并探索被测前端源码（页面路由、组件结构）以获取选择器依据。
3. API 场景再读接口文档或后端源码（Controller / DTO）以获取请求契约。

## Steps to Generate Test Code

### 1. 收集输入

向用户收集：

- **目标 `.feature` 文件**（必填）：单个或多个，位于测试仓库 `features/` 下。
- **被测系统信息**（可选，未提供则探索）：前端源码路径（UI 场景）、接口文档或后端源码路径（API 场景）。
- **生成范围**（可选）：默认全部（POM + Steps + Service + Builder）。

### 2. 解析 .feature 文件

- 解析 Feature / Scenario / Scenario Outline / Examples / 数据表 / 标签。
- 提取所有 Step 的类型（Given / When / Then）、表达式与参数。
- 识别涉及 UI 操作的步骤与涉及 API 调用的步骤。

### 3. 查询 Step 注册表检测复用

- 将每个 Step 与注册表比对，输出复用 / 新增决策到 `.asdm/workspace/<feature-id>-<feature-name>/codegen/step-reuse-report.md`。
- 已实现的 Step：直接复用，**不重复生成**。
- 仅被场景引用、未实现的 Step：进入新增生成清单。
- 语义相似但措辞不同的 Step：在报告中标注，向用户确认是复用还是新增。

### 4. 设计 Page Object（UI 场景）

输出 `.asdm/workspace/<feature-id>-<feature-name>/codegen/page-object-plan.md`：

- 按页面 / 组件拆分 Page Object 及其元素与操作。
- **选择器优先级**：`data-testid` > role / aria > label > 稳定 class / id；每个选择器记录来源依据（源码文件与位置），便于追溯与维护。
- 向用户展示设计摘要，**确认后再生成代码**。

### 5. 生成代码

按确认的设计生成源码资产（写入测试仓库对应目录）：

- **Page Object Model** → `pages/`：封装元素定位与操作，遵循 `spec/page-object-guide.md`。
- **Step Definitions** → `features/steps/`：为每个新增 Step 生成 behave 实现（`@given` / `@when` / `@then` 装饰器），调用 Page Object / API Service 完成操作与断言。
- **API Service** → `services/`：为接口调用生成封装（请求构造、鉴权、响应校验）；支持 UI 与 API 混合场景（API 准备数据、UI 验证结果）。
- **测试数据 Builder** → `data/builders/`：为数据表与 Examples 生成 Builder，支持默认值、链式覆盖与按场景定制，不在 Step 中硬编码数据。

所有代码遵循 `spec/naming-convention.md` 与对应 guide，幂等写入：已存在的实现不覆盖，仅新增 / 追加。

### 6. 校验

生成后执行校验，结果写入 `codegen/generation-report.md`：

- 静态检查：Python 语法、导入完整性、Page Object 引用一致。
- Step 歧义检测：新增 Step 表达式与已有 Step 无多重 / 冲突匹配。
- 运行 `behave --dry-run` 确认所有 Step 均已实现（无 undefined step）。
- 校验失败项：能修复的直接修复，不能修复的列入报告待办。

### 7. 更新 Step 注册表

- 将本次生成的所有 Step（含复用确认的新增）登记 / 更新到 Step 注册表：表达式、参数、类型、所在文件、复用说明。
- 注册表更新与代码落盘保持一致（先代码后注册表）。

### 8. 生成报告

生成 `.asdm/workspace/<feature-id>-<feature-name>/codegen/generation-report.md`：

- 新建 / 修改 / 跳过的文件清单。
- 复用与新增 Step 统计。
- 校验结果与待办项。
- 运行指引（`behave features/<file>.feature` 或标签过滤运行）。

## Execution Guidelines

### When to Use This Action

Use this action when:
- `.feature` 文件评审通过，需要生成可执行测试代码。
- 场景或需求变更后需要增量更新测试代码（幂等，不覆盖手工修改）。

### 代码生成指南

- 选择器不得凭空猜测：UI 场景必须从被测前端源码获取依据，无法获取时列为待办并使用最稳定可得的选择器。
- Step 实现薄封装：Step 只做编排与断言，业务操作沉到 Page Object / Service。
- 不在生成代码中硬编码环境信息（base_url、凭据），统一走 `config/` 与环境变量。
- 重复执行本 Action 时，已存在且内容一致的文件跳过；被手工修改的文件不覆盖，差异列入报告。

### 错误处理

- `.feature` 文件语法错误：停止生成，指明错误位置与修正建议。
- 缺少被测系统源码且无法确定选择器：生成代码骨架 + TODO 标注，选择器待办列入报告。

## Usage

To use this instruction, the AI model should:
1. 读取 README.md、`spec/` 相关规范与 Step 注册表。
2. 收集目标 `.feature` 文件与被测系统信息。
3. 解析 `.feature`，查询注册表输出复用决策。
4. 设计 Page Object 并经用户确认。
5. 生成 POM、Step Definitions、API Service、Builder。
6. 执行静态检查、Step 歧义检测与 `behave --dry-run` 校验。
7. 更新 Step 注册表并生成报告。

## Output Summary

After completing the action, the following artifacts will be generated:

- Page Object：测试仓库 `pages/`
- Step Definitions：测试仓库 `features/steps/`
- API Service：测试仓库 `services/`
- 测试数据 Builder：测试仓库 `data/builders/`
- 更新后的 Step 注册表：`.asdm/workspace/automation-testing/step-registry.md`
- 过程产物：`.asdm/workspace/<feature-id>-<feature-name>/codegen/` 下的 `page-object-plan.md`、`step-reuse-report.md`、`generation-report.md`
