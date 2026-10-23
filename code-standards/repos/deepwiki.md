# DeepWiki 差异

主仓：https://github.com/XBoom/deepwiki-microservices

规范正文在共用规范仓，不在本仓复制。本页只记录这个仓库相对 [通用约定](../common.md)、[Python 规范](../python.md)、[TypeScript 规范](../typescript.md) 的差异，以及要落进去的文件。

上游跟踪仓 https://github.com/XBoom/Deepwiki 不覆盖。早期拆分仓和 `XBoom/wiki-accuracy-eval` 也不覆盖。范围表见 [代码规范 v1.0](../README.md#scope)。

<a id="link"></a>

## 项目仓里只放这段链接

P0 在 `sdd/engineering-baseline/STANDARD.md` 顶部加入下面这一段。链接指向 `XBoom/skills` 的 `main`，本页合并进 `main` 之后再贴到项目仓。`STANDARD.md` 仍保留该仓已经落地的三条线说明。与 v1.0 冲突时，以规范仓为准。

```markdown
# 工程规范

正文在共用规范仓，不在本仓复制：

- 代码规范 v1.0：https://github.com/XBoom/skills/blob/main/code-standards/README.md
- 本仓差异：https://github.com/XBoom/skills/blob/main/code-standards/repos/deepwiki.md

与本文件冲突时，以规范仓 v1.0 为准。
```

`docs/` 如需入口，放同一段链接，不复制 `common.md` / `python.md` / `typescript.md`。

<a id="delta"></a>

## 相对通用规范的差异

| 项 | 本仓做法 |
|---|---|
| 语言 | Python 服务用 [python.md](../python.md)；`frontend-service` 用 [typescript.md](../typescript.md) 里 DeepWiki 那一列 |
| 格式 | Python：Ruff format，双引号，行宽 100。前端 Prettier：有分号、双引号、尾逗号 all、行宽 100 [src: pyproject.toml, frontend-service/.prettierrc.json] |
| 类型 | mypy 按服务先报告，清零后阻断（决议 4）。前端维持 `tsc --noEmit` |
| 测试 | Python 用 pytest。前端用 `node:test`，不迁到 Vitest |
| 检查入口 | `sh scripts/check.sh [static\|test\|frontend]`，这是唯一入口 [src: deepwiki-microservices:scripts/check.sh] |
| 钩子 | `.githooks/pre-commit` 加 `git config core.hooksPath .githooks`，执行 `scripts/check.sh static` |
| 包管理 | Python 运行依赖用各服务的 `requirements.txt`，开发工具用 Poetry；前端用 npm |
| 依赖更新 | Dependabot 已开启 [src: .github/dependabot.yml] |
| 合并 | 已经是 squash。继续禁止直接 push 和 force-push 到 `main`。P3 用分支保护把这条钉住 |
| 文档目录 | `docs/` 和 `sdd/` [src: deepwiki-microservices:.cursor/rules/docs-location.mdc] |
| 提交 scope | `wiki`、`parser`、`analysis`、`mcp`、`frontend`、`auth` [src: 两仓最近 30 条 commit] |

<a id="files"></a>

## P0：要改的文件

### 1. `Makefile`

现在的 `make lint` 跑的是 `ruff + black --check`，`make test` 用 `unittest discover`，而且只覆盖 6 个服务。CI 和 `check.sh` 实际是 Ruff format 加 pytest、覆盖 9 个服务。这同时违反了 STANDARD 的“不并行装 Black”和“不用 unittest discover”两条 [src: Makefile；scripts/check.sh；STANDARD.md §2]。改为：

```make
lint: ## 格式与扫描（与 CI 相同）
	sh scripts/check.sh static
format: ## 自动修排版和可修 lint
	poetry run ruff format && poetry run ruff check --fix
test: ## 全部 Python 服务的 pytest（与 CI 相同）
	sh scripts/check.sh test
fe-lint: ## 前端格式、lint、类型、测试
	sh scripts/check.sh frontend
```

### 2. `.github/pull_request_template.md`

把 `make lint`、`make test` 写成和 CI 一致的说法。修正错别字 `Dokcer`→`Docker` [src: .github/pull_request_template.md]。

### 3. 根目录 `.editorconfig`

直接复制 `sdd/engineering-baseline/templates/.editorconfig` [src: deepwiki-microservices:sdd/engineering-baseline/STANDARD.md §1；模板 sdd/engineering-baseline/templates/.editorconfig]。条文见 [common.md](../common.md#editor)：UTF-8、LF、文件末尾换行、空格缩进、Python 4 空格、其他 2 空格、行宽 100。草案没有内联模板正文，落地时以仓内那份模板为准。

### 4. `sdd/engineering-baseline/STANDARD.md`

- 顶部加上 [本页的链接段落](#link)。
- §2 前端测试一行改为“Vitest 或 node:test，仓库内只用一种”。本仓继续用 `node:test`。

### 5. `.github/workflows/pr-title.yml`

使用 [common.md](../common.md#pr-title) 里的那份 workflow，类型列表保持一致。

<a id="mypy"></a>

## P2：mypy 按服务先报告，清零后阻断

在 `scripts/check.sh` 增加只报告的一段：

```sh
python_types() {
    for service in $services; do
        (cd "$root" && poetry run mypy "$service") || echo "mypy: $service 未通过（报告模式）"
    done
}
# case 中增加：types) python_types ;;
```

CI 增加：

```yaml
- run: sh scripts/check.sh types
  continue-on-error: true
```

某个服务错误清零之后，把它移出这段报告循环，单独运行 `poetry run mypy <service>`，失败即失败，并拿掉该服务对应 job 的 `continue-on-error`。草案没有给出阻断名单的第二种脚本，落地时就改 `check.sh` 的这一处，不另加第二种类型检查器。

先修抽样里的真实缺陷，再谈阻断：

- `task-service/services/scheduler_proxy.py` 第 119 行和第 174 行重复定义 `_sync_trigger_evaluation`（mypy `no-redef`）[实测][src: deepwiki-microservices:task-service/services/scheduler_proxy.py]。
- `task-service/api/routes/tasks.py:183` 返回值类型和注解不一致 [实测]。

`B904` 的存量（抽样 2 处，全仓估计约 20 处）清零后，从 Ruff ignore 里删除 `B904`。新代码在清零之前就已经必须写 `raise ... from err` [实测]。

<a id="frontend"></a>

## P1：前端 flat config

`frontend-service/eslint.config.mjs` 替换 `.eslintrc.json`。可以和 Dependabot 升级 eslint 的 PR #26 一起做。

```js
import { defineConfig, globalIgnores } from 'eslint/config'
import nextVitals from 'eslint-config-next/core-web-vitals'
import prettier from 'eslint-config-prettier'

export default defineConfig([
  ...nextVitals,
  prettier,
  globalIgnores(['.next/**', 'out/**', 'build/**', 'next-env.d.ts']),
])
```

`package.json`：`"lint": "eslint ."`，再加 `"typecheck": "tsc --noEmit"`。`check.sh` 改为调用 `npm run typecheck`。

<a id="inconsistencies"></a>

## 现存不一致（2026-10-21）

1. `Makefile` 说 lint 用 black、测试用 unittest、覆盖 6 个服务；`scripts/check.sh` 和 `.github/workflows/ci.yml` 实际用 Ruff format 和 pytest、覆盖 9 个服务。`.github/pull_request_template.md` 却要求跑 `make lint`。P0 按上面的 Makefile 和模板修正。
2. `frontend-service/.eslintrc.json` 只继承 `next/core-web-vitals`，没有继承 `prettier`。`package.json` 的测试用 `node --test`。测试运行器按 v1.0 保留 `node:test`，并改 STANDARD 的表述；ESLint 在 P1 补上 `prettier` [src: frontend-service/.eslintrc.json；STANDARD.md §3]。
3. `task-service/services/scheduler_proxy.py` 第 119 行和第 174 行重复定义 `_sync_trigger_evaluation`。P2 先修，再接 mypy。
4. 上游跟踪仓 `XBoom/Deepwiki` 的 `.github/workflows/backend-ci.yml` 会跑 `mypy` 和 `ruff`，但 `api/pyproject.toml` 的开发依赖只有 `pytest`，而且这个 CI 只能手动触发。`.cursor/rules/git-flow.mdc` 要求以 develop 为基线，默认分支却是 `main`。只记录，不在 v1.0 处理 [src: /workspace/pstack-deepwiki-requira.md]。

<a id="phases"></a>

## 本仓落地对照

| 阶段 | 本仓动作 | 门禁 |
|---|---|---|
| P0 | 链接段落、`.editorconfig`、Makefile、PR 模板、STANDARD 的测试表述、`pr-title.yml` | 只报告，不阻断 |
| P1 | 前端 `eslint.config.mjs`、`lint` / `typecheck` 脚本。可以和 Dependabot 升级 eslint 的 PR #26 一起做 | PR 标题阻断。前端原有的 format / lint / tsc / test 继续由现有 CI 阻断 |
| P2 | `check.sh` 的 mypy 报告段；修重复定义；清零的服务改为阻断；B904 清零后取消 ignore | 已清零的服务阻断，其余只报告 |
| P3 | `main` 分支保护：必须有 PR、必须 squash、必须通过 required checks | 禁止直推 `main` 由分支保护执行。`S` 与 pip-audit 先出报告 |

阶段定义的全文在 [README.md](../README.md#phases)。
