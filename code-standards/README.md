# 代码规范 v1.0

- 定稿日期：2026-10-23（Asia/Shanghai）
- 起草：2026-10-21，草案 v0.1。本文件即该草案的 v1.0 定稿。
- 状态：定稿。五条决议已写入正文。
- 适用仓库：DeepWiki 主仓 [XBoom/deepwiki-microservices](https://github.com/XBoom/deepwiki-microservices)、Requira [XBoom/Requira](https://github.com/XBoom/Requira)。本仓（[XBoom/skills](https://github.com/XBoom/skills)）存放规范正文。
- 证据格式沿用草案：`[src: URL]` 指外部来源；`[src: 仓名:路径]` 指仓内文件（基于 2026-10-21 的 `main`）；`[实测]` 指起草时在 box 上抽样跑工具得到的结果（见 [定稿依据](#evidence)）。决议本身来自 Eric 于 2026-10-23 的拍板，不另造来源链接。

<a id="map"></a>

## 文档地图

| 文件 | 内容 |
|---|---|
| [README.md](README.md) | 本页：范围、决议、基线、落地阶段、抽样依据 |
| [common.md](common.md) | 两仓共用：三条线、命名、注释、测试、提交、分支、PR |
| [python.md](python.md) | Python：Ruff、mypy、pytest、错误与日志 |
| [typescript.md](typescript.md) | TypeScript：两套 Prettier、ESLint、类型、错误与日志 |
| [repos/deepwiki.md](repos/deepwiki.md) | DeepWiki 主仓只放的链接，以及该仓差异和落地文件 |
| [repos/requira.md](repos/requira.md) | Requira 只放的链接，以及该仓差异和落地文件 |

<a id="decisions"></a>

## 决议记录（Eric，2026-10-23）

| # | 决议 | 写入位置 |
|---|---|---|
| 1 | Requira 的 TypeScript 保持无分号、单引号 | [typescript.md](typescript.md#format)、[repos/requira.md](repos/requira.md) |
| 2 | Requira 行宽 100，与 DeepWiki 和 `.editorconfig` 相同 | [typescript.md](typescript.md#format)、[common.md](common.md#editor) |
| 3 | Requira 统一 squash 合并，禁止直接 push 到 `main` | [common.md](common.md#pr)、[repos/requira.md](repos/requira.md) |
| 4 | mypy 按服务先报告，该服务清零后再阻断 | [python.md](python.md#mypy)、[repos/deepwiki.md](repos/deepwiki.md) |
| 5 | 规范正文只放本仓。DeepWiki 与 Requira 只放链接和各自差异 | 本页、[repos/deepwiki.md](repos/deepwiki.md)、[repos/requira.md](repos/requira.md) |

同期看过、未采用的方案（草案 §6 的另一侧，此处封存，不再打开）：

1. 把 Requira 改成与 DeepWiki 前端一样的分号加双引号。抽样里 26 个文件会全部改写 [实测]。
2. Requira 行宽 120。抽样改动更少（约 12% 的行），两仓行宽会从此不同 [实测]。
3. Requira 继续混用 merge commit 和直推 `main`。
4. 等 mypy 存量一次修完再统一阻断。抽样估计全仓约 50 到 60 个错误 [实测]。
5. 两仓各存一份规范正文。

<a id="where"></a>

## 正文放在哪里

规范正文在本仓 `code-standards/`。DeepWiki 的 `docs/`、`sdd/` 和 Requira 的 `docs/` 只放链接，差异说明以本仓 `repos/` 为准，不在项目仓再抄一份。

与 DeepWiki 现有 `sdd/engineering-baseline/STANDARD.md` 冲突时，以本仓 v1.0 为准。`STANDARD.md` 继续作为该仓已落地的三条线说明，并在 P0 加上指向本仓的链接 [src: deepwiki-microservices:sdd/engineering-baseline/STANDARD.md]。

<a id="scope"></a>

## 仓库与范围

| 项目 | 仓库 | 用途 | 本规范是否覆盖 |
|---|---|---|---|
| DeepWiki（主仓） | https://github.com/XBoom/deepwiki-microservices | 微服务 monorepo：9 个 Python 服务、Next.js 前端、MCP | 覆盖，Python 与前端都以它的现有配置为基线 |
| DeepWiki（上游跟踪仓） | https://github.com/XBoom/Deepwiki | 跟踪 AsyncFuncAI/deepwiki-open 的私有部署版（`api/` + Next 前端），有长期 `work-mode` 分支 [src: /workspace/pstack-deepwiki-requira.md] | 只读对照。为了少和上游冲突，不对它加规范 |
| DeepWiki 早期拆分仓 | XBoom/deepwiki-api、deepwiki-call、deepwiki-web、deepwiki-contracts、deepwiki-research（submodule 聚合） | 2026-06 以后基本不再更新（pushed_at 停在 2026-06-10 到 06-22） | 不覆盖 |
| DeepWiki 评测 | XBoom/wiki-accuracy-eval | 准确率评测脚本 | 不覆盖 |
| Requira | https://github.com/XBoom/Requira | pnpm monorepo：apps（saas-web、dsh-client、requira-cli）和 packages（saas-api、spec-writer、llm-gateway…） | 覆盖 |

<a id="status"></a>

## 现状对比（2026-10-21）

下表是定稿时的事实，不是目标。目标见各专项和 `repos/`。

| 维度 | deepwiki-microservices | Requira |
|---|---|---|
| 语言占比（按字节，不含 md/图片） | Python 约 2/3（266 个 .py，1.2 MB），TS/TSX/CSS 约 1/3（124 个 TS/TSX，在 frontend-service）[src: git tree] | 代码几乎全是 TypeScript（320 个 .ts 加 31 个 .tsx）；另有 38 个 .html（Stitch 设计稿）和少量 .mjs/.sh [src: git tree] |
| 框架 | FastAPI、SQLAlchemy、httpx；Next.js 15.1 + React 18 [src: deepwiki-microservices:frontend-service/package.json] | Next.js 14、Express、Drizzle、Zod、pnpm workspace [src: Requira:apps/saas-web/package.json, packages/saas-api/package.json, .cursorrules] |
| 格式化 | Python 用 **Ruff format**（双引号，行宽 100）；前端用 **Prettier**（分号、双引号、尾逗号 all、行宽 100）[src: pyproject.toml, frontend-service/.prettierrc.json] | 当时没有任何格式化配置。实际写法是不加分号、单引号（抽样 26 个文件：以 `;` 结尾的行 0 行，import 用单引号 102 处，双引号 0 处）[实测]。v1.0 把这套写法定为该仓格式 |
| Lint | Ruff：`E F W I N UP B C4 SIM`，ignore `E501 B008 B904 E402 N818`；前端 `next lint`（.eslintrc 只继承 `next/core-web-vitals`） | 当时没有 ESLint/Biome。`requira-skills/hooks/pre-commit-lang-lint.sh` 发现没配置就直接跳过 [src: Requira:requira-skills/hooks/pre-commit-lang-lint.sh] |
| 类型检查 | mypy 已写进配置，当时没进 CI（README 写着“全绿之前不进钩子和 CI”）；前端在 CI 里跑 `tsc --noEmit` [src: sdd/engineering-baseline/README.md, scripts/check.sh] | 各包 `tsc --noEmit`，全部开 `strict: true`，CI 跑 `pnpm typecheck` [src: Requira:.github/workflows/ci.yml] |
| 测试 | pytest（按服务分别跑 matrix），前端用 `node --test`；测试文件 60 个（其中前端 12 个） [src: .github/workflows/ci.yml] | Vitest（`*.test.ts` 和源码放在一起，共 136 个）、Playwright e2e [src: git tree] |
| CI | ruff format --check、ruff check、9 服务 pytest、前端 format/lint/tsc/test/build、compose 健康检查、all-pass 汇总 job | 检查密钥、build、typecheck、test。当时没有 lint 和格式检查 |
| 提交钩子 | `.githooks/pre-commit`，执行 `scripts/check.sh static` | 只有 requira-skills 里的示例脚本，没接上 |
| 已有成文规范 | 有：`sdd/engineering-baseline/STANDARD.md`（三条线、工具分工、分档加严、禁止项） | `.cursorrules` 写了技术栈、接口隔离、禁止直连 SDK、禁止 console.log 关键事件 |
| 最近 30 条 commit 的格式 | 28 条符合 Conventional Commits，2 条 merge，0 条不合规；PR 合并后形如 `type(scope): 中文描述 (#N)`，即 squash | 18 条合规，5 条 merge，7 条没有 type（例如 `用户应用按 k8s 驱动发到本机 kind (#26)`） |
| 最近 30 个 PR 的标题 | 30 个全部合规 | 只有 15 个合规 |
| 分支命名 | `cursor/<slug>-<4位hash>` 19 个，`dependabot/...` 10 个，`feature/...` 1 个，全部合入 `main` | `cursor/<slug>-<hash>` 29 个，`evidence/...` 1 个，有堆叠 PR（#4→#8） |

<a id="baseline"></a>

## 选用的基线

| 领域 | 基线 | 为什么选它 |
|---|---|---|
| **总纲** | **Eric 自己的 `sdd/engineering-baseline/STANDARD.md`**，从 DeepWiki 提升为两仓共用，正文放在本仓 | 已经写好“格式 / 扫描 / 测试三条线、一种职责只用一个工具、分档加严”，DeepWiki 已按它落地，CI 也是绿的 [src: deepwiki-microservices:sdd/engineering-baseline/STANDARD.md] |
| Python | **PEP 8，由 Ruff 执行。规则组参照 FastAPI 官方仓** | FastAPI 自己的 ruff 配置是 `E W F I B C4 UP`，ignore `E501 B008 C901`，和 DeepWiki 现有配置几乎一样（DeepWiki 多开了 `N SIM`）[src: https://raw.githubusercontent.com/fastapi/fastapi/master/pyproject.toml][src: https://peps.python.org/pep-0008/][src: https://docs.astral.sh/ruff/settings/] |
| Python 文档字符串 | Google Python Style Guide 的 docstring 写法，只作约定，不交给 lint 卡 | 抽样开 `D`（pydocstyle）会冒出 271 条 [实测][src: https://google.github.io/styleguide/pyguide.html#38-comments-and-docstrings] |
| TypeScript 命名 | **Google TypeScript Style Guide 的命名部分** | 标识符大小写规则清楚。它说接口不加 `I` 前缀，“除非在所处环境里是惯用法”，正好给 Requira 的 `IAuthProvider` 这类端口接口留了例外 [src: https://google.github.io/styleguide/tsguide.html#naming-style] |
| TS lint | **typescript-eslint `recommended` + `eslint-config-prettier`**；Next 应用再加 `eslint-config-next` | 官方推荐的 flat config 入口。Airbnb 规则集还停在旧版 eslintrc 体系，而 Next 16 已经移除 `next lint` 并默认 flat config [src: https://typescript-eslint.io/getting-started/][src: https://nextjs.org/docs/app/guides/upgrading/version-16] |
| TS 格式 | **Prettier**。DeepWiki 前端维持分号、双引号、行宽 100。Requira 维持无分号、单引号，行宽 100 | 格式化器只管排版。顺着各仓现有写法定，见决议 1、2 [src: frontend-service/.prettierrc.json][实测] |
| 提交信息 | **Conventional Commits 1.0.0**（type 用英文，描述可以写中文） | DeepWiki 已经 100% 在用，Requira 有六成在用 [src: https://www.conventionalcommits.org/zh-hans/v1.0.0/] |
| 分支 | **Conventional Branch 1.1.0** | 1.1.0 收录了 `cursor/` 这类 AI agent 前缀，和两仓现有的 `cursor/<slug>-<hash>` 一致 [src: https://conventional-branch.github.io/] |
| PR | Google Engineering Practices（小改动、描述写清楚），PR 标题用 `amannn/action-semantic-pull-request` 校验 | squash 合并时 PR 标题就是 main 上的提交信息 [src: https://google.github.io/eng-practices/review/developer/small-cls.html][src: https://github.com/amannn/action-semantic-pull-request] |

<a id="phases"></a>

## 落地阶段

阶段按草案的 P0→P3 保留。v1.0 定稿已经落在本仓，P0 里“定稿规范”这一项视为完成；其余动作发生在 DeepWiki 与 Requira，清单见各自差异页。本仓这一版没有改那两个仓库。

| 阶段 | 内容 | 门禁强度 |
|---|---|---|
| **P0** | 本规范已定稿。两仓加 `.editorconfig`；DeepWiki 修 Makefile、PR 模板、`STANDARD.md` 的不一致，并加上指向本仓的链接；Requira 加 Prettier、ESLint 配置和脚本，并加上指向本仓的链接；两仓 CI 加 PR 标题检查；Requira CI 加 `format:check`、`lint` | **只报告**（`continue-on-error`），不阻断 |
| **P1** | Requira 做一次全仓 `prettier --write`（单独 PR，类型为 `style`，行宽 100、无分号、单引号，并登记到 `.git-blame-ignore-revs`）；修完 ESLint 的 error；DeepWiki 前端迁移到 flat config | Requira 的 format:check 和 lint error **阻断**；warning 不阻断，但用 `--max-warnings <当前值>` 卡住，只许减少不许增加；PR 标题**阻断** |
| **P2** | DeepWiki mypy 按服务接进 CI，先报告，清零的服务改为阻断；先修 `scheduler_proxy.py` 的重复定义；B904 存量清零后取消 ignore；Requira 给测试文件补上类型检查 | mypy 已清零的服务**阻断**，其余继续只报告 |
| **P3** | 在 `main` 上开启分支保护（必须有 PR、必须 squash、必须通过 required checks，从而禁止直推 `main`）；Requira 加 Dependabot；按 STANDARD 第 3 档出 `S`、pip-audit、pnpm audit 报告 | 分支保护按本行执行。`S`、pip-audit、pnpm audit 按照报告结果再决定是否阻断，v1.0 先只要求出报告 |

<a id="evidence"></a>

## 定稿依据（抽样）

起草时在 box 上只读抽样：用 cursor-github 读取文件内容，在本地跑工具，没有 clone，也没有改任何仓库。DeepWiki 抽的是 `task-service` 的 20 个源文件和测试文件（另外补了 8 个空的 `__init__.py`），约占全仓 Python 的 11%。Requira 抽的是 `packages/spec-writer` 和 `packages/llm-gateway` 的 26 个 .ts 文件，约占全仓 TS 的 7%。全仓数字由抽样按比例外推，只是估计，可能有 ±50% 的偏差。

| 检查项 | 抽样结果 | 全仓估计 | 结论 |
|---|---|---|---|
| DeepWiki `ruff format --check`（现有配置） | 0 个文件需要改 | 0 | 已是绿的，和 CI 一致 |
| DeepWiki `ruff check`（现有配置） | 0 | 0 | 已是绿的 |
| DeepWiki mypy（task-service） | 6 个错误，分布在 3 个文件（其中 1 个是重复定义函数的真缺陷） | 约 50 到 60 | 按决议 4：P2 先只报告，按服务逐个清零后再阻断 |
| DeepWiki 取消 ignore `B904` | 2 | 约 20 | 新代码必须遵守，存量随手修，清零后从 ignore 删除 |
| DeepWiki 加 `S`（bandit） | 3（`S105` 硬编码默认密钥 ×2、`S104` 监听 0.0.0.0 ×1） | 约 30 | 先出报告，多数是开发期默认值，确认后用 `noqa` 加理由 |
| DeepWiki 加 `ANN` | 6 | 约 60 | 只在新文件上要求（STANDARD 第 4 档） |
| DeepWiki 加 `D`（docstring） | 271 | 约 2500 | 不开 |
| DeepWiki 加 `PT/RUF/PTH/TRY/ERA` | 466（其中 RUF001-003 中文标点误报 379 条，PT009 unittest 断言 51 条） | — | v1.0 不开 |
| Requira ESLint（recommended） | 2（1 条 `no-console`，1 条 `no-unused-vars`） | 约 30 | P1 可以直接阻断 error 级别 |
| Requira Prettier（无分号、单引号、行宽 100） | 26 个文件里 18 个要改，1941 行中改动 481 行（约 25%，主要是折行） | 约 4000 到 5000 行 | **采用**。一次性排版，单独成 PR |
| Requira Prettier（无分号、单引号、行宽 120） | 26 个文件里 12 个要改，改动 227 行（约 12%） | 约 2000 到 2500 行 | **不采用**。决议 2 取行宽 100 |
| Requira Prettier（分号、双引号） | 26 个文件全部要改 | 几乎每一行 | **不采用**。决议 1 保留无分号、单引号 |

草案记载的复现材料在起草环境的 `/workspace/code-standards/sample/`（`dwm/`、`requira/`；工具装在 `.venv` 和 `tsl/node_modules`）。那批抽样文件不属于本仓，本仓不收录。

## 阅读顺序

1. 本页的决议和落地阶段。
2. [common.md](common.md) 的提交、分支和 PR。两仓从下一次 PR 起就按 squash 和 Conventional Commits 执行。
3. 按语言读 [python.md](python.md) 或 [typescript.md](typescript.md)。
4. 落地到具体仓库时只打开对应的 `repos/` 页，按里面的文件清单改项目仓，并把链接段落贴进项目仓。
