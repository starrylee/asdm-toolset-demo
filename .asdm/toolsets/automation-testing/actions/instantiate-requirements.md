# Instructions for instantiate-requirements action

## Metadata

```json
{
  "guid": "8c7d6e5f-4a3b-4c2d-8e9f-0a1b2c3d4e5f",
  "name": "instantiate-requirements",
  "displayName": "需求实例化",
  "description": "分析 User Story 的验收标准，识别业务规则、边界值与数据变体，以实例化需求（SBE）方式生成 Gherkin .feature 文件与覆盖追溯矩阵",
  "toolset": {
    "guid": "9e8b7a6c-5d4f-4c3b-9a2e-1f0d8c7b6e5a",
    "id": "automation-testing",
    "name": "自动化测试工具集",
    "version": "0.0.1"
  },
  "scenario": "需求实例化"
}
```

## Purpose

This instruction guides the AI model to 分析 User Story 的验收标准并生成 Gherkin `.feature` 文件，把抽象需求转化为具体、可验证的示例。It applies Specification by Example (SBE) to produce scenario files with full traceability to acceptance criteria, while maximizing reuse of existing Step expressions.

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

Before 实例化需求，the AI model should read the toolset context:

### Context Files to Read (Required)

1. **Toolset README.md** (Required)
   - Path: `.asdm/toolsets/automation-testing/README.md`
   - Purpose: 理解工具集共享约定（Step 注册表、可追溯性、人工确认点）

2. **Gherkin 编写规范** (Required)
   - Path: `.asdm/toolsets/automation-testing/spec/gherkin-writing-guide.md`
   - Purpose: 场景结构、措辞、Background / Scenario Outline / 标签的使用规则

3. **Step 注册表** (Required)
   - Path: `.asdm/workspace/automation-testing/step-registry.md`
   - Purpose: 优先复用已有 Step 表达式，保持措辞一致

### Progressive Context Loading Strategy

1. 先读 README.md 与 `spec/gherkin-writing-guide.md`。
2. 再读 Step 注册表，标记可复用的 Step 表达式。
3. 补充材料（业务规则、原型、接口说明）按用户提供情况读取。

## Steps to Instantiate Requirements

### 1. 收集输入

向用户收集：

- **User Story 文档**（必填）：含验收标准（AC 编号或清单）。
- **补充材料**（可选）：业务规则文档、原型图、接口说明、既有 `.feature` 示例。
- **目标路径**：`.feature` 草稿的工作区特性目录（默认按 feature-id 推导，见 Output Summary）。

### 2. 拆解验收标准

逐条拆解验收标准，输出到 `.asdm/workspace/<feature-id>-<feature-name>/requirements/acceptance-criteria-analysis.md`：

- 识别每条 AC 的：业务规则、前置条件、主流程、边界值、异常流、数据变体。
- 标注歧义、遗漏与依赖项。
- **歧义与遗漏必须向用户澄清**（以问题清单形式集中提问），澄清结果回写分析文档。

### 3. 查询 Step 注册表并规划复用

- 读取 Step 注册表，为每条 AC 初步匹配可复用的 Step 表达式。
- 输出复用 / 新增 Step 的初步清单，保持措辞与已有场景一致。

### 4. 实例化场景

按 SBE 思路编写场景草稿（先写入工作区，不直接进测试仓库）：

- 用**具体数据**表达期望行为，不使用"显示正确""操作成功"等模糊描述。
- 合理使用 `Background`（公共前置）、`Scenario Outline` + `Examples`（数据变体）、数据表。
- 合理打标签：`@smoke` / `@regression` / `@ui` / `@api` 等。
- **Then 步骤必须是可验证的具体断言**。
- 场景与 AC 一一对应或显式标注多对一关系。
- 优先复用注册表中已有 Step 表达式。

### 5. 生成覆盖追溯矩阵

生成 `.asdm/workspace/<feature-id>-<feature-name>/requirements/coverage-matrix.md`：

| 验收标准 | 场景名 | .feature 文件 | 覆盖类型（正向/边界/异常） |
|---|---|---|---|

- 保证每条 AC 至少被一个场景覆盖；无法覆盖的 AC 显式列出并说明原因。

### 6. 人工确认后写入测试仓库

- 向用户展示：场景清单摘要、覆盖矩阵、复用 / 新增 Step 清单。
- **用户确认后**，将 `.feature` 文件写入测试仓库 `features/` 目录（按 `spec/gherkin-writing-guide.md` 的文件命名规则）。
- 若用户要求修改，调整后再次确认。

### 7. 更新 Step 注册表

将本次场景中**新增**的 Step 表达式登记到 Step 注册表（标记"仅场景引用，实现待 generate-test-code"）。

## Execution Guidelines

### When to Use This Action

Use this action when:
- 拿到一个新的 User Story，需要转化为自动化测试场景。
- 需求变更后需要增量更新已有 `.feature` 文件（按幂等方式：已确认场景不覆盖，仅追加 / 修订差异）。

### 场景编写指南

- 一个场景只验证一个业务行为点；步骤数建议不超过 8 步。
- 避免在 Given 中隐含被测行为；用 API 步骤准备数据时打 `@api` 标签说明。
- 场景名使用业务语言，格式建议"主语 + 条件 + 预期"。

### 错误处理

- User Story 缺少验收标准：停止生成，向用户索取或协助补写 AC。
- AC 之间存在冲突：列入澄清问题清单，不在场景中静默取舍。

## Usage

To use this instruction, the AI model should:
1. 读取 README.md、`spec/gherkin-writing-guide.md` 与 Step 注册表。
2. 收集 User Story 与补充材料。
3. 拆解验收标准，澄清歧义。
4. 查注册表规划 Step 复用。
5. 实例化场景草稿并生成覆盖矩阵。
6. 经用户确认后写入测试仓库 `features/`。
7. 更新 Step 注册表。

## Output Summary

After completing the action, the following artifacts will be generated:

- `.feature` 场景文件（确认后写入测试仓库 `features/`，草稿保留在 `.asdm/workspace/<feature-id>-<feature-name>/features/`）
- 验收标准分析文档：`.asdm/workspace/<feature-id>-<feature-name>/requirements/acceptance-criteria-analysis.md`
- 覆盖追溯矩阵：`.asdm/workspace/<feature-id>-<feature-name>/requirements/coverage-matrix.md`
- 更新后的 Step 注册表：`.asdm/workspace/automation-testing/step-registry.md`
