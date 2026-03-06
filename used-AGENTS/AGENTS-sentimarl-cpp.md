# Repository Guidelines

## 0) Agent 工作总则（先看）
- 优先遵循本文件，其次参考根目录与子模块文档；不要仅凭经验改动 `J.RTS/vendor/` 与其他 vendored 代码。
- 本仓库是 MARLlib fork（Python 3.10 + Ray 2.2 + Gymnasium），兼容层较多；未经明确需求，不要随意升级 Ray/Gym/PyTorch 或移除兼容补丁。
- 对跨模块变更（环境注册、算法接入、配置项）要保持“代码 + 配置 + 文档”同步。
- 代码修改**不要进行commit**
- 每次工作可以修改的文件**仅限项目目录内**
- 每次工作**基本流程**
  - 阅读该文档
  - 根据AGENTS.md文档确定自己任务下一步需要看哪个部分并仔细阅读
  - 进行修改
  - 执行必要的测试
  - 反复迭代以上步骤，直到达到目标
  - 将以上过程记录在一个md文件内，命名规则<日期>_<session-id>_<概括任务信息>，放在**report/目录**下
    - 并且生成一份给其他agent读，方便你的其他“同事”了解工作情况的文档，放在**report/coop/目录**下，命名规则同上
- 同一个会话中一般有多次工作，每次工作**如有必要的话**要**更新**对应的report和coop下的文件，如：1. 做了新的修改，在report中的内容补充对当次工作的描述；2. coop下的内容经过当次工作修改后已不适用，需要修改；......


## 1) 文档检索优先级（修改前建议阅读）

### 根目录核心文档
- `README.md`：当前 fork 的依赖基线、运行入口与历史背景。
- `CONTRIBUTING.md`：测试与 PR 期望。（暂时先不用管pytest的部分，**pytest还有些问题**）
- `docs/upgrade_ray2_gymnasium.md`：Ray2/Gymnasium 迁移背景与兼容约束（高优先级；为其他agent生成，仅供参考）。
- `docs/MARLLIB_MARL_GUIDE_CN.md`：`marllib/marl` 结构与接入流程说明（参考用；为其他agent生成，仅供参考）。

### J.RTS 子模块文档
- `J.RTS/README.md`：J.RTS 目录结构、开发流程、提交与测试 SOP。
- `J.RTS/doc/*.md`：系统设计和行为说明（如 topology/toggle/fine-tune）。
- `J.RTS/clang-format`、`J.RTS/clang-tidy`：C++ 格式与静态检查规则。
- `J.RTS/.gitmessage.txt`（与根目录同构）：提交信息模板。

### 文档检索范围建议
- 优先检索：根目录、`docs/`、`J.RTS/README.md`、`J.RTS/doc/`。
- 谨慎检索：`J.RTS/vendor/**` 文档仅在排查第三方行为时参考，不作为本项目编码规范依据。

## 2) Project Structure & Module Organization
核心代码位于 `marllib/`：
- `marllib/envs/base_env/`：环境适配器；`config/` 内为环境 YAML。
- `marllib/envs/global_reward_env/`：全合作奖励包装（`*_fcoop.py`），内部环境继承自base_env，**大多**写法为重载了step返回奖励的部分，具体细节请**阅读代码排查**
- `marllib/marl/algos/`：算法入口、核心逻辑、超参数。
- `marllib/marl/models/`：模型与配置。
- `marllib/patch/`：外部环境/RLlib 兼容补丁，**目前没有作用**，本fork项目的README中环境配置部分已经不需要patch部分的补丁

支持目录：
- `examples/`：样例配置与 checkpoint。
- `J.RTS/`：sentry 环境依赖的外部引擎子模块。
- `third_party/` 与 `J.RTS/vendor/`：第三方代码，非必要不改。

## 3) Build, Test, and Development Commands
- `uv sync`：安装固定依赖环境（Python 3.10）。
- `python exp_script/run_sentry.py`：运行 sentry 训练（依赖 `J.RTS/build` 中 sentinel 模块）。
- `python exp_script/run_mate.py` / `python exp_script/run_pettingzoo.py`：样例训练/渲染流程。
- `python tests/test_sentinel_smoke.py --zero-action`：sentinel 绑定最小冒烟（reset/step/close）。
- `python tests/test_sentry_wrapper_smoke.py`：Sentry Python 包装层最小冒烟。
- `python -m py_compile exp_script/*.py`：脚本级语法检查（快速、无运行时副作用）。
- `pytest -v --cov=marllib --cov-report html`：完整测试。
- `pytest -v tests/test_env.py -k mpe`：快速环境定向测试。
- `make -C docs html`：构建文档（先 `pip install -r docs/requirements.txt`）。

J.RTS 相关：
- 修改J.RTS内C++代码需要确保编译通过
- 同时由J.RTS环境binding出的sentinel模块也应该可以被SentiMARL项目正常使用
- 构建/测试优先参考 `J.RTS/README.md` 中 SOP 与 `script/` 脚本。
- 涉及 C++ 改动时，优先确保最小可复现编译通过，再运行对应测试脚本。

## 4) Coding Style & Naming Conventions

### Python（marllib 主体）
- 4 空格缩进，遵循 PEP 8；命名采用 `snake_case` / `PascalCase` / `UPPER_CASE`。
- 新增环境时，保持 `base_env/*.py` 与 `base_env/config/*.yaml` 的名称和 key 对齐。
- 尽量小步改动，不把功能改动与无关重构混在一起。

### C++（J.RTS）
- 遵循 `J.RTS/clang-format` 与 `J.RTS/clang-tidy` 约束。
- 命名风格以现有代码为准（类名 CamelCase，常量大写，局部/成员按现有文件风格延续）。
- 不在同一提交中混入大规模格式化与功能改动。

### 注释与文档字符串（结合现有“人工+工具”习惯）
- 注释目标是解释“为什么/约束/边界条件”，避免解释显而易见的“做了什么”。
- 保持注释风格与周边一致：已有中英混合时可延续，但单次改动尽量统一语言。
- 对历史注释块（如运行脚本顶部说明、模块级 docstring、分隔注释）优先保留，不做无意义重写。
- 避免批量生成冗长注释；新增注释应短小、可验证、与代码同步更新。
- 特别要注意**J.RTS子模块内**的代码注释为doxygen风格，在进行注释时使用的author名字为m1zu

## 5) Testing Guidelines
- 测试框架以 `pytest` 为主，部分用例采用 `unittest.TestCase` 组织。
- 已有的测试：test_algorithm.py test_env.py test_grid_search.py test_model.py等有一定问题，不作为代码是否通过测试的参考
- 新增测试放在 `tests/`，文件名 `test_*.py`，方法名 `test_*`。
- 开发时先跑与改动相关的最小测试，再跑更大范围回归。
- 若改动涉及 `J.RTS` 绑定/接口，需要 Python 侧能正常加载并完成基本交互。
- 对 `exp_script/` 下脚本改动，至少执行一次 `python -m py_compile exp_script/*.py` 作为低成本语法校验。
- 对 Sentry 绑定链路改动，建议补跑：
  - `python tests/test_sentinel_smoke.py --zero-action`
  - `python tests/test_sentry_wrapper_smoke.py`

## 6) Commit & Pull Request Guidelines
- 提交信息遵循 Conventional Commits：`feat(scope): subject`、`fix(scope): subject` 等。
- scope 可参考 `.gitmessage.txt` 约定（如 `env`/`algo`/`system`/`binding`/`docs`）。
- subject 建议小写开头、简洁聚焦单一变更。

PR 建议：
- 描述行为变化、影响模块、验证命令与结果。
- 关联 issue；必要时在标题标注 `#major` 或 `#minor`，否则按 patch 处理。
- 涉及兼容层（Ray/Gymnasium）时，明确说明是否影响旧 checkpoint 或现有训练脚本。

## 7) 常见任务执行清单（Checklist）

### A. 新增环境接入（Environment Onboarding）
- [ ] **先对齐文档与约束**：阅读 `README.md`、`docs/upgrade_ray2_gymnasium.md`、`marllib/envs/base_env/` 现有同类环境实现，确认是否涉及兼容层。
- [ ] **新增环境代码**：在 `marllib/envs/base_env/` 增加环境适配器，遵循 `MultiAgentEnv` 现有接口风格与返回结构。
- [ ] **新增环境配置**：在 `marllib/envs/base_env/config/` 增加对应 YAML，名称与代码文件/`map_name` key 对齐。
- [ ] **注册与导出检查**：确认环境在 `marllib` 入口可被 `marl.make_env(...)` 正常发现与注册。
- [ ] **协作/奖励逻辑一致性**：如支持 `force_coop`，确认 `global_reward_env` 包装行为与预期一致。
- [ ] **依赖与路径检查**：若依赖 `J.RTS` 或其他外部模块，确认构建产物路径、运行时 `sys.path`/动态库可见性正确。
- [ ] **最小可运行验证**：至少跑一次最小训练或 reset/step 冒烟（例如 `python run_sentry.py` 或对应最小脚本）。
- [ ] **测试补充**：在 `tests/` 增加或更新环境相关测试（`test_*.py`），优先覆盖注册、reset/step、空间定义。
- [ ] **文档同步**：同步更新环境说明（必要时 `README.md`/`docs/`/脚本注释），写明依赖、入口、已知限制。

### B. 新增算法接入（Algorithm Onboarding）
- [ ] **先梳理接入链路**：参考 `agent_generated/MARLLIB_MARL_GUIDE_CN.md`，确定算法属于 IL/CC/VD 哪条运行入口。
- [ ] **补齐算法核心实现**：在 `marllib/marl/algos/core/` 添加策略/损失/Trainer 逻辑，尽量复用已有 utils。
- [ ] **补齐脚本层入口**：在 `marllib/marl/algos/scripts/` 增加 `run_xxx`，包含模型注册、参数解析、`tune.run(...)` 组装。
- [ ] **补齐注册映射**：更新算法注册表与 manager 暴露接口，保证可通过 `marl.algos.xxx(...)` 调用。
- [ ] **补齐超参数配置**：在 `marllib/marl/algos/hyperparams/` 新增 `common/test`（必要时 finetuned）配置。
- [ ] **补齐模型兼容性**：确认 `marl.build_model(...)` 对应架构可用（mlp/gru/lstm 或算法特定模型）。
- [ ] **兼容层开关核对**：保持 Ray2 旧执行栈兼容设置，不要误切到新 API stack（除非任务明确要求）。
- [ ] **最小训练验证**：跑一次小规模训练（小步数、小 batch）验证训练链路完整（初始化、采样、反传、checkpoint）。
- [ ] **测试与回归**：补充算法最小测试或集成冒烟；至少覆盖“能启动训练 + 关键配置生效”。
- [ ] **文档同步**：补充算法使用示例、参数来源与限制，必要时更新 `README.md`/`docs/`。

### C. 提交前统一检查（Definition of Done）
- [ ] 改动不包含 `J.RTS/vendor/`、`third_party/` 的无关修改。
- [ ] 代码、配置、文档三者已同步；命名与 key 对齐。
- [ ] 已执行与改动相关的最小验证命令，并记录结果用于 PR 描述。
- [ ] 提交信息符合 Conventional Commits，scope 与实际影响模块一致。
