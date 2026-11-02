# 面向 AI 编码助手的组织操作与安全交互指南 (AGENTS_GUIDE)

> **版本**：v1.0  
> **适用对象**：在 `scuec-newthread-db` 组织仓库内运行的所有 AI 编程助手（Claude Code, Codex, Pi, Cursor, GitHub Copilot 等）及人类开发者  
> **文件定位**：本规范定义了 AI Agent 在参与本组织代码编写、重构、测试、Git 提交与发布时的行为准则与安全边界。

---

## 1. 组织身份与交互基准

1. **组织归属**：本项目属于中南民族大学计算机科学学院新思路实验室 DB 组（SCUEC DB Lab）。
2. **交互语言**：面向开发者的日常汇报、交互与 Issue/PR 沟通默认使用简体中文；代码标识符、注释、构建脚本、API 及 Commit Message 遵循英文规范。
3. **极简充分原则**：优先使用仓库既有抽象与依赖，严禁随意引入庞大且非必要的外部第三方库；重构代码前先理解完整调用链。

---

## 2. 代码开发与构建验证规范

### 2.1 C/C++ 工程准则
- 遵循 C++20 / C++17 现代标准，严禁使用已被废弃的不安全 C 风格内存操作函数；
- 必须保证 CMake 配置可跨平台复现（支持 Ubuntu Linux 与 macOS），禁止在 CMakeLists.txt 中硬编码个人绝对路径；
- 改动代码后，必须在本地运行并验证：`cmake -B build && cmake --build build && ctest --test-dir build`，确保测试用例全部通过。

### 2.2 Python 工程准则
- 依赖管理统一基于 `pyproject.toml`，推荐使用 `uv` 工具链；
- 包含硬件检测逻辑时，必须支持对 CUDA、Apple Silicon MPS 及 CPU 的优雅降级探测，不得将 PyTorch 作为不可选的硬依赖；
- 改动代码后，必须在本地运行并验证：`pytest`，确保所有用例通过。

---

## 3. Git 协作与提交门禁

1. **分支操作规范**：
   - AI 严禁直接向默认主分支（`main` 或 `master`）提交代码；
   - 必须引导人类创建特性分支（`feat/*`、`fix/*`、`exp/*`）并通过 Pull Request 合并；
2. **Commit Message 规范**：
   遵循 Conventional Commits 格式：
   - `feat: add prefix range scan to storage engine`
   - `fix: resolve race condition in buffer pool eviction`
   - `test: add unit tests for learned index model`
   - `docs: update CMake build instructions in README`
3. **不可逆高危操作熔断 (严禁执行)**：
   - 严禁自主执行 `git push --force` 或 `git push -f` 覆盖远程主干；
   - 严禁自主执行 `git reset --hard` 清除未暂存的工作成果；
   - 严禁执行任何仓库删除（Delete repository）、可见性更改（Make public/private）或权限剥夺操作。

---

## 4. 安全、隐私与数据防爆红线

1. **敏感凭据零容忍**：
   - 绝对禁止在代码、配置、测试用例或注释中硬编码任何真实密码、数据库连接串、私有服务器 IP、SSH 私钥或 API Token；
   - 敏感配置一律通过系统环境变量读取，或从 `.env` 动态载入；仓库中仅允许提交 `.env.example` 占位模板；
2. **大文件防爆拦截**：
   - 单文件体积超过 **50 MiB** 时，严禁直接执行 `git commit`；
   - 必须主动拦截并提醒人类开发者配置 Git LFS 托管，或将大文件存入实验室网盘并在 Markdown 中附带下载链接；
   - 严禁将编译产物（`.o`, `.so`, `build/`）、虚拟环境（`.venv/`）、测试日志及大模型权重（`.pt`, `.onnx`）提交进 Git 暂存区；
3. **学术数据保密**：
   - 凡涉及未正式录用的学术论文实验代码、企业横向合作真实业务数据，严禁通过任何外部未经许可的公开通道广播。

---

## 5. 机器自检清单 (Pre-flight Checklist)

AI Agent 在完成任务并交付代码前，必须自查以下 4 项：
- [ ] 本地单元测试已实际运行并全部通过（CTest 或 Pytest）；
- [ ] 运行 `git diff` 核对修改范围，确认未污染与任务无关的文件；
- [ ] 确认没有任何明文敏感密码或私有密钥被暂存；
- [ ] 确认未引入超过 50MB 的二进制大文件。
