# ATDD 文档模板

## 用途
本模板定义了ATDD（验收测试驱动开发）文档的结构。ATDD文档分为两个阶段：
- **初稿阶段**（阶段二）：在编码之前，根据BDD场景规划验收测试用例，供用户审阅确认。
- **定稿阶段**（阶段五）：在自验证完成后，将实际验证结果和证据填入文档。

ATDD文档与BDD场景直接对应和追溯。

## 文档结构

每个ATDD文档**必须**遵循以下结构：
```markdown
# [功能名称]

## BDD参考
[BDD文档路径，如 PROJECT_STATE/BDD/user-login-email-validation.md]

## 验证策略
- 日期：[验证日期]
- 策略：[一种或多种：运行测试框架、调用工具(MCP/skill)、执行校验命令、编写测试脚本、构建/运行项目...]
- 方式：[一种或多种：检查测试报告、工具调用结果、命令/脚本运行结果、项目构建产物、项目运行状态...]

## 测试用例

### 测试用例 1：[测试名称]
**Given（前置条件）**：[具体的测试设置——精确的数据、状态、命令]
**When（操作）**：[执行的具体操作——命令、API调用、调用工具]
**Then（预期结果）**：[预期结果——精确的断言、预期的输出]

**测试所用命令**（如果有的话）：[具体命令]
**测试所用的工具**（如果有的话）：[工具名称]

**结果**：通过 / 失败（初稿阶段留空）
**证据**：[确认结果的内容——测试输出、日志片段、截图引用等]（初稿阶段留空）

---

### 测试用例 2：[测试名称]
...

## 汇总

| BDD场景 | 测试用例 | 结果 |
|---|---|---|
| [场景名称] | TC-1 | （初稿阶段留空） |
| ... | ... | ... |

**场景总数**：[N]
**通过数**：[N]（初稿阶段留空）
**失败数**：[N]（初稿阶段留空）
**备注**：[任何限制条件、跳过的场景或无法完全验证的条件]
```

## 非文字证据

自验证过程中产生的非文字证据（截图、数据文件等）应保存到
`PROJECT_STATE/ATDD/<特性名称>/`目录下，使用有意义的文件名（如 `evidence-tc1.png`、`evidence-tc2.log`）。

在acceptance.md的证据字段中使用相对路径引用：
```
详见 [证据截图](./evidence-tc1.png)
```

支持的证据文件类型：
- 截图：`.png`、`.jpg`、`.gif`
- 数据文件：`.json`、`.csv`、`.xml`
- 其他：任何能说明验证结果的文件

## 编写指南

### 初稿阶段（阶段二）
1. 汇总表中**必须**明确每个测试用例所对应的BDD场景。
2. Given/When/Then 应详细描述**计划**的测试设置、操作和预期结果。
3. 包含**计划使用**的测试命令或工具（即使尚未编写测试代码，也应描述预期的命令格式）。
4. 结果和证据字段留空，标注"（待验证）"。
5. 汇总表中的结果列留空。

### 定稿阶段（阶段五）
1. 根据实际自验证结果填写每个测试用例的**结果**（通过/失败）。
2. 填写每个测试用例的**证据**（测试输出、日志片段/数据、截图等）。
3. 如有非文字证据文件，在证据字段中使用相对路径引用（如 `详见 [截图](./evidence-tc1.png)`）。
4. 填写汇总表。
5. 如实记录结果。如果某个场景无法通过自动化测试验证的，如实说明并描述检查了哪些内容。
6. 在备注中说明任何限制条件。

## 完整示例（定稿后）

```markdown
# 带邮箱验证的用户登录

## BDD参考
PROJECT_STATE/BDD/user-login-email-validation.md

## 验证策略
- 日期：2026-04-09
- 策略：测试框架-pytest (v8.1)
- 方式：自动化单元测试 + API调用结果检查

## 测试用例

### 测试用例 1：有效凭据返回JWT令牌
**Given（前置条件）**：测试数据库中已插入用户 alice@example.com（密码哈希对应 "SecurePass123!"）
**When（操作）**：POST /api/login，请求体为 {"email": "alice@example.com", "password": "SecurePass123!"}
**Then（预期结果）**：响应状态码 200，响应体包含 "token" 字段，令牌为有效JWT且有效期为24小时

**测试所用命令**：`pytest tests/test_login.py::test_valid_login -v`

**结果**：通过
**证据**：测试输出显示 "assert response.status_code == 200" 断言通过。JWT解码后exp声明正确。详见 [测试输出截图](./evidence-tc1.png)

---

### 测试用例 2：无效密码返回401
**Given（前置条件）**：测试数据库中已插入用户 alice@example.com
**When（操作）**：POST /api/login，请求体为 {"email": "alice@example.com", "password": "WrongPassword"}
**Then（预期结果）**：响应状态码 401，响应体包含 "Invalid credentials"，无 token 字段

**测试所用命令**：`pytest tests/test_login.py::test_invalid_password -v`

**结果**：通过
**证据**：测试输出确认状态码为 401 且错误信息匹配。

---

### 测试用例 3：格式错误的邮箱返回400
**Given（前置条件）**：无需特定设置
**When（操作）**：POST /api/login，请求体为 {"email": "not-an-email", "password": "AnyPassword"}
**Then（预期结果）**：响应状态码 400，响应体包含 "Invalid email format"

**测试所用命令**：`pytest tests/test_login.py::test_malformed_email -v`

**结果**：通过
**证据**：验证层在数据库查询之前即拒绝了请求。

---

### 测试用例 4：SQL注入被净化处理
**Given（前置条件）**：测试数据库按常规方式插入数据
**When（操作）**：POST /api/login，请求体为 {"email": "admin@example.com' OR '1'='1", "password": "anything"}
**Then（预期结果）**：响应状态码 400，未使用原始输入执行数据库查询

**测试所用命令**：`pytest tests/test_login.py::test_sql_injection_sanitized -v`
**测试所用的工具**：数据库查询日志分析

**结果**：通过
**证据**：使用了参数化查询。输入在到达数据库层之前已被邮箱格式验证拒绝。详见 [查询日志](./evidence-tc4.log)

---

### 测试用例 5：数据库不可达返回503
**Given（前置条件）**：通过 `docker stop auth-db` 停止数据库容器
**When（操作）**：使用有效凭据发送 POST /api/login 请求
**Then（预期结果）**：响应状态码 503，错误信息为 "Service temporarily unavailable"

**测试所用的工具**：curl 命令行工具

**结果**：通过
**证据**：停止数据库后，通过curl发送请求。收到503响应及正确的错误信息体。测试后重启数据库。详见 [响应截图](./evidence-tc5.png)

---

## 汇总

| BDD场景 | 测试用例 | 结果 |
|---|---|---|
| 使用有效凭据成功登录 | TC-1 | 通过 |
| 使用无效密码登录 | TC-2 | 通过 |
| 使用格式错误的邮箱登录 | TC-3 | 通过 |
| SQL注入攻击尝试登录 | TC-4 | 通过 |
| 数据库不可达时登录 | TC-5 | 通过 |

**场景总数**：5
**通过数**：5
**失败数**：0
```

## 完整示例（初稿状态）

初稿阶段，结果和证据字段留空：

```markdown
# 带邮箱验证的用户登录

## BDD参考
PROJECT_STATE/BDD/user-login-email-validation.md

## 验证策略
- 日期：2026-04-09
- 策略：测试框架-pytest (v8.1)
- 方式：自动化单元测试 + API调用结果检查

## 测试用例

### 测试用例 1：有效凭据返回JWT令牌
**Given（前置条件）**：测试数据库中插入用户 alice@example.com（密码哈希对应 "SecurePass123!"）
**When（操作）**：POST /api/login，请求体为 {"email": "alice@example.com", "password": "SecurePass123!"}
**Then（预期结果）**：响应状态码 200，响应体包含 "token" 字段，令牌为有效JWT且有效期为24小时

**测试所用命令**（计划）：`pytest tests/test_login.py::test_valid_login -v`

**结果**：（待验证）
**证据**：（待验证）

---

### 测试用例 2：无效密码返回401
**Given（前置条件）**：测试数据库中插入用户 alice@example.com
**When（操作）**：POST /api/login，请求体为 {"email": "alice@example.com", "password": "WrongPassword"}
**Then（预期结果）**：响应状态码 401，响应体包含 "Invalid credentials"，无 token 字段

**测试所用命令**（计划）：`pytest tests/test_login.py::test_invalid_password -v`

**结果**：（待验证）
**证据**：（待验证）

---

### 测试用例 3：格式错误的邮箱返回400
**Given（前置条件）**：无需特定设置
**When（操作）**：POST /api/login，请求体为 {"email": "not-an-email", "password": "AnyPassword"}
**Then（预期结果）**：响应状态码 400，响应体包含 "Invalid email format"

**测试所用命令**（计划）：`pytest tests/test_login.py::test_malformed_email -v`

**结果**：（待验证）
**证据**：（待验证）

---

### 测试用例 4：SQL注入被净化处理
**Given（前置条件）**：测试数据库按常规方式插入数据
**When（操作）**：POST /api/login，请求体为 {"email": "admin@example.com' OR '1'='1", "password": "anything"}
**Then（预期结果）**：响应状态码 400，未使用原始输入执行数据库查询

**测试所用命令**（计划）：`pytest tests/test_login.py::test_sql_injection_sanitized -v`

**结果**：（待验证）
**证据**：（待验证）

---

### 测试用例 5：数据库不可达返回503
**Given（前置条件）**：通过 `docker stop auth-db` 停止数据库容器
**When（操作）**：使用有效凭据发送 POST /api/login 请求
**Then（预期结果）**：响应状态码 503，错误信息为 "Service temporarily unavailable"

**测试所用的工具**（计划）：curl 命令行工具

**结果**：（待验证）
**证据**：（待验证）

---

## 汇总

| BDD场景 | 测试用例 | 结果 |
|---|---|---|
| 使用有效凭据成功登录 | TC-1 | （待验证） |
| 使用无效密码登录 | TC-2 | （待验证） |
| 使用格式错误的邮箱登录 | TC-3 | （待验证） |
| SQL注入攻击尝试登录 | TC-4 | （待验证） |
| 数据库不可达时登录 | TC-5 | （待验证） |

**场景总数**：5
**通过数**：（待验证）
**失败数**：（待验证）
```
