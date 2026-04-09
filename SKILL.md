---
name: test-case-generator
description: Generate comprehensive test cases from requirement documents using systematic testing methodologies. Use this skill when user asks to "generate test cases", "create test cases from requirements", "design test scenarios", "analyze testing points", or provides a requirement document and wants test coverage analysis. Supports 10 testing dimensions including functionality, input validation, boundary testing, business rules, state transitions, exceptions, concurrency/idempotency, fault tolerance, security, and performance testing. Includes comprehensive API testing coverage with 7 dimensions. **NEW v2.3**: Added Excel generation best practices, Python error prevention guidelines, and output file validation to prevent common syntax/runtime errors. **v2.2**: Enhanced document reading capability supporting text (with images), URLs (with auto-auth browser), and complete visual content extraction from UI mockups, diagrams, and configuration tables.
---

# Test Case Generator (Enhanced)

## Overview

Generate comprehensive, high-coverage, non-redundant test cases from requirement documents using systematic testing methodologies. This skill transforms textual requirements into structured test points and test cases across **10 testing dimensions**, ensuring thorough coverage while avoiding duplication.

**Enhanced Features:**
- ✅ **10 Testing Dimensions** (vs. 6 in basic version)
- ✅ **API Testing 7 Dimensions** (idempotency, concurrency, fault tolerance)
- ✅ **Self-Check Mechanism** (5 critical questions)
- ✅ **External Dependencies Analysis** (APIs, services, cache, DB)
- ✅ **Risk Point Identification** (high-risk logic)

## When to Use

Use this skill when:
- User provides a requirement document and asks for test cases
- User asks to "analyze testing points" from a specification
- User wants to "generate test scenarios" for a feature
- User needs "test coverage analysis" for requirements
- User asks about test design methodologies
- **User needs API/interface test cases**
- **User wants comprehensive security/performance testing**
- **User provides requirement document in text format (may include images)** - Must extract information from both text and images
- **User provides requirement document URL (e.g., internal doc system)** - Must open browser to read online documentation

## Workflow

### Step 0: Document Reading and Analysis (NEW - Critical Enhancement)

**CRITICAL**: Before analyzing requirements, read the source document completely - DO NOT skip images or online content.

#### 0.1 Document Input Format Recognition

Identify the input format:

| Input Type | Detection Method | Processing Strategy |
|-----------|------------------|-------------------|
| **Text content** | Check if input is markdown, text, or file path | - If file path: Read with appropriate tool<br>- If text/markdown: Direct analysis<br>- **CRITICAL**: Extract ALL information including images/screenshots |
| **URL** | Check if input starts with http://, https://, or is a known doc system URL | - Use opencode browser (preferred - auto-auth)<br>- Fallback: playwright (wait for user login) |

#### 0.2 Document Reading Strategies

**Strategy A: Text/Markdown Input (with possible images)**

```python
# Workflow for text input
1. If file path provided:
   - Read file using appropriate tool (docx, read, etc.)
   - Check if file contains embedded images

2. Extract ALL content:
   - Text content (requirements, descriptions)
   - Tables (configuration items, parameters)
   - Images/Charts (UI mocks, workflows, data models)
   - Screenshots (example interfaces, edge cases)

3. For images/charts/screenshots:
   - Use look_at tool to analyze visual content
   - Extract: UI elements, data fields, workflows, constraints
   - Example: If showing NPS card UI screenshot → extract score range, button states, form fields
```

**Strategy B: URL Input (online documentation)**

```bash
# Preferred: opencode browser (auto-auth with cookies)
# Fallback: playwright (wait for user to login)

# Step 1: Open URL with opencode browser
# Step 2: Read complete page content (all sections, tables, images)
# Step 3: Extract requirements from:
#   - Text descriptions
#   - Configuration tables/parameters
#   - UI screenshots/diagrams
#   - Workflow charts
# Step 4: Proceed to Step 1 (Analyze Requirements)
```

**Critical Rules:**

1. **NEVER skip visual content** - Images/charts/screenshots contain critical requirements
2. **Read complete document** - Not just summary, all sections and details
3. **Extract from visual elements**:
   - UI mockups: Fields, buttons, labels, validation rules
   - Workflow diagrams: States, transitions, edge cases
   - Data models: Fields, types, constraints, relationships
   - Configuration tables: All rows and columns
4. **Verify no missing information**:
   - Cross-check text vs visual content
   - Ensure all UI panels/sections are covered
   - Check all configuration options are captured

**Output Format after Document Reading:**

```markdown
### 需求文档完整解析

**文档来源**: [file path or URL]
**文档格式**: [docx/markdown/URL]

**文本内容提取:**
- 功能点列表
- 输入参数表
- 业务规则
- 状态流转

**视觉内容提取（如适用）:**
- UI界面截图分析: [从图片中提取的UI元素]
- 工作流程图: [从图表中提取的流程]
- 配置项表: [从截图/表格中提取的详细配置]
- 数据模型: [从图表中提取的字段关系]

**完整度自检:**
- [ ] 所有文本需求已提取
- [ ] 所有图片/图表内容已分析
- [ ] 视觉内容与文本内容一致性验证
- [ ] 无遗漏的关键信息
```

### Step 1: Analyze Requirements (Enhanced)

Read and understand the requirement document thoroughly:

#### 1.1 Core Requirements
1. **Identify functional requirements** - What the system should do
2. **Identify non-functional requirements** - Performance, security, compatibility
3. **Identify constraints** - Limitations, boundaries, rules
4. **Identify business rules** - Logic, conditions, workflows
5. **Identify data requirements** - Input/output specifications, data types

#### 1.2 Enhanced Analysis (NEW)
6. **Identify external dependencies**
   - External APIs/services
   - Cache systems (Redis, Memcached)
   - Databases (MySQL, MongoDB, etc.)
   - Third-party integrations
   - File systems/storage

7. **Identify risk points**
   - High-risk business logic (payment, permission changes)
   - Data consistency critical operations
   - Security-sensitive operations
   - Performance-critical paths
   - Complex conditional logic

8. **Identify ALL API endpoints (CRITICAL - NEW)**
   - List every REST API/GraphQL/WebSocket endpoint mentioned in requirements
   - Group by module (user-facing, admin backend, external services)
   - For each endpoint, capture: HTTP method, path, main purpose, key parameters
   - Example:
     ```
     ### API接口清单

     **用户端接口:**
     1. GET /api/nps/card - 获取NPS卡片数据
     2. POST /api/nps/submit - 提交用户评分
     3. POST /api/nps/exposure - 曝光上报
     4. POST /api/nps/close - 关闭卡片

     **后台管理接口:**
     1. GET /api/admin/scenes - 场景列表查询
     2. POST /api/admin/scenes - 创建场景
     3. PUT /api/admin/scenes/{id} - 编辑场景
     4. DELETE /api/admin/scenes/{id} - 删除场景
     5. POST /api/admin/mutex - 创建互斥配置
     6. POST /api/admin/blacklist - 黑名单配置
     ```

**Output Format:**
```markdown
### 需求解析

1. **功能点列表**
   - 功能1: [描述]
   - 功能2: [描述]

2. **输入参数**
   | 字段名 | 类型 | 必填 | 约束条件 | 说明 |
   |--------|------|------|----------|------|
   | field1 | String | 是 | 1-50字符 | 用户名 |

3. **业务规则**
   - 规则1: [条件 -> 结果]
   - 规则2: [条件 -> 结果]

4. **状态流转**
   [状态A] --事件1--> [状态B] --事件2--> [状态C]

5. **外部依赖**
   - 依赖1: [服务名称/接口]
   - 依赖2: [数据库/缓存]

6. **风险点识别**
   - 风险1: [高危逻辑]
   - 风险2: [数据一致性]
```

### Step 2: Extract Test Points (Enhanced)

Extract testable elements from requirements across **10 testing dimensions**:

#### Testing Dimensions Checklist

| # | 测试维度 | 测试点 | 覆盖度 |
|---|---------|--------|--------|
| **1. 功能测试** | | | |
| | 正常流程 | 主流程、分支流程 | |
| | 业务规则 | 条件组合、逻辑冲突 | |
| **2. 输入测试** | | | |
| | 合法输入 | 有效值、格式正确 | |
| | 非法输入 | 无效值、格式错误 | |
| | 空值/null | null、空字符串、缺失字段 | |
| | 超长/极值 | 最大长度、超大数值 | |
| | 特殊字符 | 转义字符、表情、HTML | |
| **3. 边界测试** | | | |
| | 临界值 | 上界、下界、边界值 | |
| | 边界外值 | 上界+1、下界-1 | |
| **4. 业务规则测试** | | | |
| | 多条件组合 | 判定表、决策树 | |
| | 逻辑冲突 | 矛盾条件、互斥规则 | |
| **5. 状态流转测试** | | | |
| | 正常流转 | 合法状态转换 | |
| | 非法跳转 | 跳过中间状态、回退 | |
| **6. 异常测试** | | | |
| | 参数缺失 | 必填参数为空 | |
| | 数据不存在 | 查询不存在记录 | |
| | 服务异常 | 外部服务不可用 | |
| **7. 并发与幂等性** | | | |
| | 重复请求 | 幂等性验证 | |
| | 并发请求 | 多用户同时操作 | |
| | 数据一致性 | 并发数据完整性 | |
| **8. 容错与稳定性** | | | |
| | 超时 | 请求超时处理 | |
| | 重试 | 失败重试机制 | |
| | 降级 | 服务降级策略 | |
| | 部分失败 | 部分成功处理 | |
| **9. 安全测试** | | | |
| | SQL注入 | 恶意SQL语句 | |
| | XSS | 跨站脚本攻击 | |
| | 越权访问 | 权限绕过 | |
| | Token校验 | 认证失效 | |
| **10. 性能测试** | | | |
| | 高并发 | 大量并发请求 | |
| | 大数据量 | 大数据集处理 | |

### Step 3: Select Testing Methods (Enhanced)

Choose appropriate testing design methods based on the nature of the requirements. See `references/testing-methods.md` for detailed explanations of each method.

#### Testing Method Selection Matrix

| Scenario | Primary Methods | Secondary Methods | Coverage Focus |
|----------|-----------------|-------------------|----------------|
| Input validation with ranges | Equivalence Partitioning + Boundary Value Analysis | Error Guessing | Input testing (维度2,3) |
| Complex business rules | Decision Table + Cause-Effect Graphing | Scenario Testing | Business rules (维度4) |
| Multiple configuration options | Orthogonal Array Testing + Pairwise | Error Guessing | Combinatorial testing |
| Workflow/State-based system | State Transition Testing + Scenario Testing | Error Guessing | State transitions (维度5) |
| API/Interface testing | All methods + Concurrency + Security | Fault tolerance | All 10 dimensions |
| General coverage | Combine multiple methods | Self-check mechanism | All dimensions |

#### Enhanced Method Application

**1. Scenario Testing (NEW)**
- **When to use**: User workflows, end-to-end testing
- **How to apply**:
  - Identify user personas and goals
  - Map user journeys through the system
  - Create scenarios covering happy path + error paths
  - Example: "用户登录 -> 浏览商品 -> 加入购物车 -> 下单支付"

**2. Error Guessing (Enhanced)**
- **When to use**: All scenarios, especially high-risk areas
- **How to apply**:
  - Leverage historical bug data
  - Consider common pitfalls (off-by-one, null pointer, race conditions)
  - Think about edge cases from user perspective
  - Example: "用户快速点击提交按钮2次"

### Step 4: Generate Test Cases (Enhanced)

Create test cases following the enhanced format:

#### 4.1 Functional Test Cases

```markdown
| 用例ID | 用例标题 | 前置条件 | 测试步骤 | 输入数据 | 预期结果 | 用例类型 | 优先级 |
|--------|---------|---------|---------|---------|---------|---------|--------|
| TC-FUNC-001 | 功能测试-正常流程 | [前置条件] | 1. 步骤1<br>2. 步骤2 | [输入数据] | [预期结果] | 功能测试 | P0 |
```

**用例类型 (Test Type):**
- 功能测试 (FUNC)
- 输入测试 (INPUT)
- 边界测试 (BOUND)
- 业务规则 (BIZ)
- 状态流转 (STATE)
- 异常测试 (EX)
- 并发测试 (CONC)
- 容错测试 (FAULT)
- 安全测试 (SEC)
- 性能测试 (PERF)

#### 4.2 API Test Cases (NEW - Enhanced)

For API/interface testing, use dedicated format covering 7 dimensions:

```markdown
| 用例ID | 接口名称 | 测试场景 | 请求方法 | 请求参数 | 预期结果 | 测试维度 | 优先级 |
|--------|---------|---------|---------|---------|---------|---------|--------|
| TC-API-001 | /api/endpoint | 参数校验-必填参数缺失 | POST | {} | HTTP 400<br>{"code":"PARAM_MISSING"} | 1.参数校验 | P0 |
```

**API Testing 7 Dimensions:**

1. **参数校验 (Parameter Validation)**
   - 必填参数缺失
   - 类型错误
   - 长度/范围限制
   - 非法值/特殊字符

2. **功能验证 (Functional Validation)**
   - 正常请求返回正确数据
   - 业务逻辑正确执行
   - 数据正确存储/更新

3. **异常处理 (Exception Handling)**
   - 错误码正确性
   - 异常返回格式
   - 错误信息准确性

4. **幂等性 (Idempotency)**
   - 重复调用结果一致
   - 唯一标识正确处理
   - 防重复提交机制

5. **并发 (Concurrency)**
   - 多请求竞争
   - 数据一致性
   - 锁机制正确性

6. **性能 (Performance)**
   - 响应时间达标
   - QPS满足要求
   - 资源消耗合理

7. **容错 (Fault Tolerance)**
   - 超时处理
   - 重试机制
   - 服务降级
   - 部分失败处理

**Priority Guidelines:**
- **P0**: Core functionality, critical paths, must-pass scenarios, security vulnerabilities
- **P1**: Important features, edge cases, common error scenarios, concurrency issues
- **P2**: Nice-to-have, rare scenarios, cosmetic issues, performance optimization

### Step 5: Coverage Analysis (Enhanced)

Verify test coverage across all dimensions:

#### 5.1 Coverage Checklist (NEW)

```markdown
### 测试覆盖度检查

#### 测试设计方法覆盖
- [x] 等价类划分 (Equivalence Partitioning)
- [x] 边界值分析 (Boundary Value Analysis)
- [x] 判定表 (Decision Table)
- [x] 状态迁移 (State Transition)
- [x] 场景法 (Scenario Testing)
- [x] 错误推测 (Error Guessing)
- [x] 正交/组合测试 (Orthogonal/Pairwise)

#### 测试维度覆盖
- [x] 1. 功能测试 (Functionality)
- [x] 2. 输入测试 (Input)
- [x] 3. 边界测试 (Boundary)
- [x] 4. 业务规则 (Business Rules)
- [x] 5. 状态流转 (State Transition)
- [x] 6. 异常测试 (Exceptions)
- [x] 7. 并发与幂等性 (Concurrency & Idempotency)
- [x] 8. 容错与稳定性 (Fault Tolerance)
- [x] 9. 安全测试 (Security)
- [x] 10. 性能测试 (Performance)

#### API测试覆盖（如适用）
- [x] 1. 参数校验
- [x] 2. 功能验证
- [x] 3. 异常处理
- [x] 4. 幂等性
- [x] 5. 并发
- [x] 6. 性能
- [x] 7. 容错
```

#### 5.2 Coverage Matrix

```markdown
| 测试设计方法 | 覆盖场景 | 用例数量 | 覆盖度评估 |
|------------|---------|---------|-----------|
| 等价类划分 | 输入验证、数据类型 | XX | 高/中/低 |
| 边界值分析 | 数值范围、长度限制 | XX | 高/中/低 |
| 判定表 | 多条件组合 | XX | 高/中/低 |
| 状态迁移 | 工作流、状态变更 | XX | 高/中/低 |
| 场景法 | 用户路径 | XX | 高/中/低 |
| 错误推测 | 异常情况 | XX | 高/中/低 |
```

### Step 6: Self-Check Mechanism (NEW)

**CRITICAL**: Before finalizing test cases, answer these 5 questions:

#### Self-Check Questions

1. **是否覆盖所有功能点？**
   - [ ] 所有需求点都有对应测试用例
   - [ ] 正常流程和分支流程都覆盖
   - [ ] 遗漏功能点: [列出]

2. **是否遗漏边界情况？**
   - [ ] 输入边界值测试完整
   - [ ] 时间边界（开始/结束时间）
   - [ ] 数值边界（最大/最小值）
   - [ ] 遗漏边界: [列出]

3. **是否覆盖异常流程？**
   - [ ] 参数异常（缺失、错误、超限）
   - [ ] 业务异常（数据不存在、状态错误）
   - [ ] 系统异常（超时、服务不可用）
   - [ ] 遗漏异常: [列出]

4. **是否存在遗漏的高风险场景？**
   - [ ] 并发场景（重复提交、竞争条件）
   - [ ] 安全场景（注入、越权）
   - [ ] 数据一致性场景
   - [ ] 遗漏高风险: [列出]

5. **是否有冗余或重复用例？**
   - [ ] 检查重复测试点
   - [ ] 合并相似用例
   - [ ] 冗余用例: [列出]

6. **是否覆盖所有API接口？（NEW - CRITICAL）**
   - [ ] 每个识别出的API接口都有对应测试用例
   - [ ] 每个接口至少覆盖参数校验、功能验证2个维度（P0级）
   - [ ] 关键接口覆盖幂等性、并发、异常处理（P0/P1级）
   - [ ] 遗漏接口: [列出接口名称]
   - [ ] 遗漏接口的测试维度: [列出维度]

**Action**: If any question has "遗漏" items, **补充测试用例** before proceeding.

**Additional Action for API Coverage:**
- If question 6 has "遗漏接口", you MUST create test cases for those interfaces immediately
- Use the 7-dimension API testing framework for each missing endpoint
- Minimum coverage: 参数校验 + 功能验证 (P0)
- Recommended coverage: All 7 dimensions for critical endpoints

### Step 7: Review and Refine

Final review checklist:
- [ ] No redundant test cases
- [ ] All requirements have corresponding test cases
- [ ] Test cases are independent and executable
- [ ] Expected results are clear and verifiable
- [ ] Priority assignment is reasonable
- [ ] **Self-check mechanism completed**
- [ ] **All 10 testing dimensions considered**
- [ ] **API testing 7 dimensions covered (if applicable)**
- [ ] **ALL API endpoints identified in Step 1 have test cases (NEW)**

## Testing Design Methods Reference

This skill supports the following testing design methods. Load `references/testing-methods.md` for detailed methodology explanations:

| Method | Best For | Token Cost | Coverage Dimension |
|--------|----------|------------|-------------------|
| Equivalence Partitioning | Input validation, data type testing | Low | 维度2 (输入测试) |
| Boundary Value Analysis | Numeric ranges, date ranges, length limits | Low | 维度3 (边界测试) |
| Decision Table | Complex conditional logic, business rules | Medium | 维度4 (业务规则) |
| Cause-Effect Graphing | Input-output relationships, system behavior | Medium | 维度4 (业务规则) |
| Orthogonal Array Testing | Configuration combinations, multi-factor testing | Low | 组合测试 |
| State Transition | Workflows, status changes, lifecycle testing | Medium | 维度5 (状态流转) |
| Scenario Testing | User workflows, end-to-end testing | Medium | 维度1 (功能测试) |
| Error Guessing | Common bugs, historical issues, edge cases | Low | 维度6 (异常测试) |
| Pairwise Testing | Combinatorial testing with reduced test count | Low | 组合测试 |

**Usage:** Reference the detailed methodology in `references/testing-methods.md` when applying specific testing techniques.

## Output Format (Enhanced)

### 1. 测试点分析 (Test Points Analysis)

```markdown
### 测试点分析

**功能点:**
- 功能1: [描述]
- 功能2: [描述]

**API接口清单（CRITICAL - 必须）：**
**用户端接口:**
- 接口1: GET/POST /api/path1 - [功能描述]
- 接口2: GET/POST /api/path2 - [功能描述]

**后台管理接口:**
- 接口3: GET/POST /admin/path3 - [功能描述]
- 接口4: GET/POST /admin/path4 - [功能描述]

**输入参数:**
| 字段名 | 类型 | 必填 | 约束条件 |
|--------|------|------|----------|
| ... | ... | ... | ... |

**业务规则:**
- 规则1: [条件 -> 结果]

**状态流:**
[状态图或文字描述]

**外部依赖:**
- 依赖1: [描述]

**风险点:**
- 风险1: [高危逻辑]
```

### 2. 功能测试用例 (Functional Test Cases)

```markdown
### 功能测试用例

| 用例ID | 用例标题 | 前置条件 | 测试步骤 | 输入数据 | 预期结果 | 用例类型 | 优先级 |
|--------|---------|---------|---------|---------|---------|---------|--------|
| TC-FUNC-001 | ... | ... | ... | ... | ... | 功能测试 | P0 |
```

### 3. 接口测试用例 (API Test Cases) (NEW)

```markdown
### 接口测试用例

| 用例ID | 接口名称 | 测试场景 | 请求方法 | 请求参数 | 预期结果 | 测试维度 | 优先级 |
|--------|---------|---------|---------|---------|---------|---------|--------|
| TC-API-001 | /api/xxx | 参数校验-必填参数缺失 | POST | {} | HTTP 400 | 1.参数校验 | P0 |
```

### 4. 覆盖说明 (Coverage Report)

```markdown
### 覆盖说明

**测试设计方法:**
- [x] 等价类划分 ✔
- [x] 边界值分析 ✔
- [x] 判定表 ✔
- [x] 状态迁移 ✔
- [x] 场景法 ✔
- [x] 错误推测 ✔
- [x] 并发 ✔
- [x] 安全 ✔

**测试维度:**
- [x] 1. 功能测试
- [x] 2. 输入测试
- [x] 3. 边界测试
- [x] 4. 业务规则
- [x] 5. 状态流转
- [x] 6. 异常测试
- [x] 7. 并发与幂等性
- [x] 8. 容错与稳定性
- [x] 9. 安全测试
- [x] 10. 性能测试
```

### 5. 自检与补充 (Self-Check) (NEW)

```markdown
### 自检与补充

1. **是否覆盖所有功能点？**
   ✅ 所有需求点已覆盖
   - [列出已覆盖的功能点]

2. **是否遗漏边界情况？**
   ✅ 边界情况已完整覆盖
   - [列出边界测试用例]

3. **是否覆盖异常流程？**
   ✅ 异常流程已系统化覆盖
   - [列出异常测试用例]

4. **是否存在遗漏的高风险场景？**
   ✅ 高风险场景已覆盖
   - 并发场景: TC-CONC-001 ~ TC-CONC-005
   - 安全场景: TC-SEC-001 ~ TC-SEC-008

5. **是否有冗余或重复用例？**
   ✅ 已检查无冗余用例
   - [说明去重情况]

**补充用例:** (如有遗漏)
- [补充的测试用例]
```

### 6. 测试总结报告 (Summary Report)

```markdown
### 测试总结

**用例统计:**
- 总用例数: XXX条
- P0用例: XX条 (XX%)
- P1用例: XX条 (XX%)
- P2用例: XX条 (XX%)

**覆盖度评估:**
- 功能覆盖: 100%
- 输入覆盖: 100%
- 边界覆盖: 100%
- 异常覆盖: 100%
- 并发覆盖: 100%
- 安全覆盖: 100%

**风险点识别:**
- 风险1: [描述]
- 风险2: [描述]

**建议:**
- 建议1: [描述]
- 建议2: [描述]
```

## Resources

### references/testing-methods.md

Comprehensive documentation of testing design methodologies including:
- Equivalence Partitioning (等价类划分法)
- Boundary Value Analysis (边界值分析法)
- Decision Table Testing (判定表驱动法)
- Cause-Effect Graphing (因果图法)
- Orthogonal Array Testing (正交实验法)
- State Transition Testing (状态迁移图法)
- Scenario Testing (场景法) (NEW)
- Error Guessing (错误猜测法)
- Pairwise Testing (成对测试法)

Load this reference when detailed methodology guidance is needed.

## Best Practices

1. **Start with requirements analysis** - Don't jump to test cases without understanding the system
2. **Use multiple methods** - No single method provides complete coverage
3. **Prioritize ruthlessly** - Focus on high-risk, high-value scenarios
4. **Avoid redundancy** - Each test case should test something unique
5. **Make tests independent** - Each test should be executable in isolation
6. **Specify expected results clearly** - Ambiguous expected results lead to unreliable tests
7. **Consider non-functional aspects** - Performance, security, compatibility
8. **Think about edge cases** - Boundaries, nulls, empty strings, maximums
9. **Include negative tests** - Not just happy path
10. **Document assumptions** - Make implicit requirements explicit
11. **Always run self-check** - Ensure completeness before finalizing (NEW)
12. **Cover all 10 dimensions** - Don't skip concurrency, security, fault tolerance (NEW)
13. **API testing must cover 7 dimensions** - Especially idempotency and concurrency (NEW)
14. **Identify risk points early** - Focus on high-risk areas (NEW)
15. **Excel generation best practices** - Avoid common Python/openpyxl errors (NEW):
    - Always check data types before comparison (`isinstance(value, (int, float))`)
    - Handle MergedCell exceptions when iterating columns
    - Use proper imports (`from openpyxl.cell.cell import MergedCell`)
    - Test code incrementally - don't generate all data at once
    - Verify file creation with `ls -lh` before declaring success
16. **Code quality in generation scripts** - Prevent syntax errors (NEW):
    - Verify all parentheses are matched
    - Use proper string escaping for newlines in cell values
    - Test simple version first, then add complexity
    - Use type checking before operations (`isinstance`, `type()`)
    - Handle exceptions gracefully with try-except
17. **Output file validation** - Always verify generated files (NEW):
    - Check file exists and size > 0
    - Validate sheet count and structure
    - Test opening file with pandas/openpyxl
    - Verify data integrity (row counts, priorities)
    - Provide summary statistics to user

## Example Usage

**User Input:**
```
分析一下这个需求文档，生成测试用例
[Requirement document content]
```

**Expected Output:**
1. ✅ 需求解析（6个方面）
2. ✅ 测试点分析（10个维度）
3. ✅ 功能测试用例（表格）
4. ✅ 接口测试用例（表格，如适用）
5. ✅ 覆盖说明（清单）
6. ✅ 自检与补充（5个问题）
7. ✅ 测试总结报告

## Changelog

### v2.1 (Critical Fix)
- ✅ Added API endpoint identification in Step 1.2 (MANDATORY)
- ✅ Added API interface list output format
- ✅ Added 6th self-check question: "是否覆盖所有API接口？"
- ✅ Added automatic API coverage validation mechanism
- ✅ Added minimum API testing requirements (P0: 参数校验 + 功能验证)
- ✅ Added final checklist item: "ALL API endpoints have test cases"

**Fix Issue:** v2.0 lacked explicit API endpoint identification, leading to missing interfaces in actual test case generation (e.g., only 2/6 APIs covered for NPS system). v2.1 enforces API identification and coverage validation.

### v2.3 (Code Quality & Output Validation) - NEW
- ✅ Added Excel generation best practices (MergedCell handling, type checking)
- ✅ Added Python script error prevention guidelines (parentheses matching, type validation)
- ✅ Added output file validation checklist (file existence, structure verification)
- ✅ Added incremental testing approach (test simple, then complex)
- ✅ Added summary statistics reporting (sheet counts, priority distribution)

**Fix Issue:** v2.2 lacked guidance on Excel generation error prevention, leading to multiple Python syntax errors (type errors, MergedCell attributes) when generating test cases. v2.3 adds best practices to prevent common openpyxl/Python errors and mandates output file verification before declaring success.

### v2.2 (Document Reading Enhancement)
- ✅ Added Step 0: Document Reading and Analysis (NEW)
- ✅ Enhanced document input format recognition (text with images, URLs)
- ✅ Added visual content extraction strategy (UI mockups, diagrams, screenshots)
- ✅ Added opencode browser support (auto-auth with cookies) for URL inputs
- ✅ Added playwright fallback (wait for user login) for URL inputs
- ✅ Added complete content extraction rules (text + visual elements)
- ✅ Added completeness verification checklist (text + visual consistency)

**Fix Issue:** v2.1 assumed text-only requirement documents, missing critical information from images/charts/screenshots embedded in documents. v2.2 ensures comprehensive reading of ALL content including visual elements like UI mockups, workflow diagrams, configuration tables, and online documentation via browser.

### v2.0 (Enhanced)
- ✅ Added 10 testing dimensions (vs. 6 in v1.0)
- ✅ Added API testing 7 dimensions
- ✅ Added self-check mechanism (5 questions)
- ✅ Added external dependencies analysis
- ✅ Added risk point identification
- ✅ Enhanced scenario testing method
- ✅ Separated functional and API test case tables
- ✅ Added test type classification
- ✅ Enhanced coverage checklist
- ✅ Improved output format with structured sections
