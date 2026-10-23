# 通用约定

本页适用于代码规范 v1.0 覆盖的全部仓库。语言细节见 [python.md](python.md) 和 [typescript.md](typescript.md)，仓库差异见 [repos/](repos/)。

<a id="pipeline"></a>

## 1. 三条线与工具分工

直接沿用 DeepWiki 的 `sdd/engineering-baseline/STANDARD.md`。

- 本地和 CI 都按“**格式 → 扫描（lint，然后类型检查）→ 单元测试**”的顺序跑，前一步失败就停。CI 只用 `--check`，不改文件 [src: deepwiki-microservices:sdd/engineering-baseline/STANDARD.md §1]。
- 一种职责只用一个工具：Python 排版只用 Ruff format，同时不装 Black；import 排序用 Ruff `I`；TypeScript 排版用 Prettier，ESLint 必须继承 `prettier`，避免规则互相打架 [src: deepwiki-microservices:sdd/engineering-baseline/STANDARD.md §2]。
- 一次只加严一组规则，修到全绿再加下一组。禁止直接打开 `ALL`。禁止整文件写不带规则码的 `noqa` 或 `ts-ignore` [src: deepwiki-microservices:sdd/engineering-baseline/STANDARD.md §4、§7]。

<a id="editor"></a>

## 2. 编辑器

编辑器统一用仓库根目录的 `.editorconfig`：UTF-8、LF、文件末尾换行、空格缩进；Python 缩进 4，其他 2；行宽 100 [src: deepwiki-microservices:sdd/engineering-baseline/STANDARD.md §1；模板 sdd/engineering-baseline/templates/.editorconfig]。

两仓根目录在 2026-10-21 都还没有这个文件（模板只放在 DeepWiki 的 `sdd/` 下）[src: git tree]。P0 把模板复制到两个仓库的根目录。行宽 100 同时覆盖 Requira（决议 2）。

<a id="naming"></a>

## 3. 命名

| 对象 | Python（PEP 8） | TypeScript（Google TS Guide） |
|---|---|---|
| 变量、函数、方法 | `snake_case` | `lowerCamelCase` |
| 类、类型、枚举 | `PascalCase` | `UpperCamelCase` |
| 常量 | `UPPER_SNAKE_CASE`（例如 `JWT_SECRET`、`ACCOUNT_SERVICE_URL`）[src: deepwiki-microservices:task-service/api/deps.py] | 模块级常量用 `CONSTANT_CASE` |
| 私有成员 | 前缀 `_` | 用 `private` 关键字，不加 `_` 前缀 |
| 缩写 | — | 缩写按普通单词处理：`loadHttpUrl`，不写 `loadHTTPURL` |
| 文件 | `snake_case.py` | **`kebab-case.ts`**（Requira 在 2026-10-21 的 284 个 src 文件里，除 6 个 PascalCase 和 4 个 camelCase 外都是这种）；React 组件用 `PascalCase.tsx`；hook 用 `useXxx.ts` |
| 目录 | 服务目录用 `kebab-case`（`task-service`），包目录用 `snake_case` | `kebab-case`；Requira 的 workspace 包名统一为 `@requira/<kebab>` |
| 接口 `I` 前缀 | — | 默认不加。**例外**：`.cursorrules` 指定的 SaaS 端口接口（`IAuthProvider`、`IBillingProvider`、`ICodeGenLLM`、`ITelemetry`）保留前缀，它们的文件名也照旧（`ICodeGenLLM.ts`）[src: Requira:.cursorrules][src: https://google.github.io/styleguide/tsguide.html#naming-style] |

规则依据：Ruff `N`（pep8-naming）在 DeepWiki 已开启并且是绿的。TypeScript 文件命名靠 code review 把关，v1.0 不额外引入 lint 插件。

<a id="layout"></a>

## 4. 目录

目录是仓库差异，不在本页再写一套。

- DeepWiki Python 服务的分层见 [python.md](python.md#layout)，仓库级路径见 [repos/deepwiki.md](repos/deepwiki.md)。
- Requira 的 `apps/*` 与 `packages/*` 见 [repos/requira.md](repos/requira.md)。
- 文档：DeepWiki 放 `docs/` 和 `sdd/`，Requira 放 `docs/` 和 `outputs/`。这是两仓已有的约定，`.cursor/rules` 里都有写 [src: deepwiki-microservices:.cursor/rules/docs-location.mdc；Requira:.cursorrules]。规范正文不放进这两个目录，只放链接，见 [README.md](README.md#where)。

<a id="comments"></a>

## 5. 注释与文档

- 注释、docstring、提交描述可以写中文，两仓现在都是这样。标识符只用 ASCII [src: https://google.github.io/styleguide/tsguide.html#identifiers]。
- 注释写“为什么”，不复述代码在做什么。不留注释掉的代码（开 `ERA` 规则时抽样只有 1 处，可以随手清掉）[实测]。
- `TODO`、`FIXME` 后面写上负责人或 issue：`# TODO(eric, #123): ...`。DeepWiki 的 PR 模板已经要求“无新增 TODO/FIXME” [src: deepwiki-microservices:.github/pull_request_template.md]。
- Python docstring 的写法见 [python.md](python.md#docstring)。TypeScript 导出符号的 TSDoc 见 [typescript.md](typescript.md#docs)。

<a id="tests"></a>

## 6. 测试

- 断言对外可见的结果；只 mock 边界（网络、外部进程、模型客户端）；一个测试只说明一个失败原因；新模块要在同一个 PR 里带上测试 [src: deepwiki-microservices:sdd/engineering-baseline/STANDARD.md §5]。
- 默认测试集不调用真实的 LLM，也不访问外网。Python 用 `slow` 和 `network` 两个 marker 把这类测试排除 [src: deepwiki-microservices:sdd/engineering-baseline/STANDARD.md §3]。
- 不把覆盖率百分比当作门禁。
- 测试框架按仓库已有选择，一个仓库里同一种职责只用一种：DeepWiki Python 用 pytest，DeepWiki 前端用 `node:test`，Requira 用 Vitest。见各语言页。

<a id="commits"></a>

## 7. 提交信息（Conventional Commits 1.0.0）

```
<type>(<scope>): <描述，中文或英文，不加句号>

[可选正文：为什么改，有什么影响]

[可选脚注：Refs: #123 / BREAKING CHANGE: ...]
```

- `type` 只能取：`feat` `fix` `docs` `style` `refactor` `perf` `test` `build` `ci` `chore` `revert` [src: https://www.conventionalcommits.org/zh-hans/v1.0.0/]。
- `scope` 填服务名或包名。DeepWiki 用 `wiki`、`parser`、`analysis`、`mcp`、`frontend`、`auth`（已在用）；Requira 用 `saas-web`、`saas-api`、`spec-writer`、`k8s`、`deploy`、`ci`（已在用）[src: 两仓最近 30 条 commit]。
- 好例子（来自现有历史）：`fix(wiki): label module pages from dict fields (#32)`、`fix(spec-writer): 对话写不出需求文档时再向模型要一次`。
- 反例：`用户应用按 k8s 驱动发到本机 kind (#26)`。这条没有 type，应改为 `feat(deploy): 用户应用按 k8s 驱动发到本机 kind`。

squash 合并后，main 上的提交信息就是 PR 标题，所以标题用同一套格式。见 [PR](#pr)。

<a id="branches"></a>

## 8. 分支（Conventional Branch 1.1.0）

- 主干只有 `main`。两仓所有 PR 的 base 都是 `main`，属于主干开发，不采用 git-flow 的 develop 分支。`XBoom/Deepwiki` 的 git-flow 规则只在那个上游跟踪仓有效，本规范不覆盖那个仓 [src: 两仓 PR 列表；XBoom/Deepwiki:.cursor/rules/git-flow.mdc]。
- 短期分支写成 `<type>/<kebab-描述>`。type 可以是 `feature`、`feat`、`fix`、`bugfix`、`hotfix`、`chore`、`release`，也可以是 AI agent 前缀 `cursor/`、`ai/` 等。允许追加短 hash，例如 `cursor/strip-llm-think-tags-e9db` [src: https://conventional-branch.github.io/]。
- 另外允许 `dependabot/`（机器人）和 `evidence/`（Requira 的验收证据分支）[src: Requira PR #30]。
- 只用小写字母、数字和连字符，不用下划线或空格。
- 分支名写入本页，v1.0 不加分支名门禁。草案里提到的 commit-check-action 不接入 CI [src: https://conventional-branch.github.io/]。

<a id="pr"></a>

## 9. PR

决议 3：本规范覆盖的仓库统一 squash 合并，禁止直接 push 到 `main`，禁止 force-push 到 `main`。DeepWiki 的合并结果已经是 `type(scope): 描述 (#N)`。Requira 改为同样的做法。GitHub 分支保护在 P3 打开；保护打开之前，这条已经是规范，由评审执行。

Requira 改为 squash 的直接原因：2026-10-15 `main` 上 CI 连续失败了 3 次，其中有直接 push 的 `fix(saas): 场景1冒烟三阻塞…` 和手工 merge 的 `merge: 合入 PR #27…` [src: Requira workflow runs / commits]。

- PR 标题必须是合规的 Conventional Commit。squash 之后，标题就是 main 上的提交信息。
- 一个 PR 只做一件事。参考 Google 的建议，改动行数（不含生成文件和 lock）尽量控制在几百行以内；超过就拆 PR，或者做堆叠 PR（Requira #4 到 #8 已经这样做过）[src: https://google.github.io/eng-practices/review/developer/small-cls.html]。
- 两仓都用同一类 PR 说明：改了什么、关联任务或文档、验证命令及结果、风险和回滚方式、截图。DeepWiki 现有模板要修正的两处见 [repos/deepwiki.md](repos/deepwiki.md)。
- 只有 CI 全绿才能合并。纯排版的提交必须单独成为一个 PR，提交信息写明不含行为变化，并把它加进 `.git-blame-ignore-revs` [src: deepwiki-microservices:sdd/engineering-baseline/STANDARD.md §6][src: https://docs.github.com/en/repositories/working-with-files/using-files/viewing-and-understanding-files#ignore-commits-in-the-blame-view]。

<a id="pr-title"></a>

## 10. PR 标题检查

两仓都添加 `.github/workflows/pr-title.yml`。P0 先接上，P1 起标题不合规即阻断 [src: https://github.com/amannn/action-semantic-pull-request]。

```yaml
name: pr-title
on:
  pull_request:
    types: [opened, edited, synchronize, reopened]
permissions:
  pull-requests: read
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: amannn/action-semantic-pull-request@v5
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          types: |
            feat
            fix
            docs
            style
            refactor
            perf
            test
            build
            ci
            chore
            revert
          requireScope: false
          subjectPattern: ^(?![A-Z]).+$   # 不允许英文大写开头；中文不受影响
```

`types` 与本页第 7 节的 type 列表相同。`requireScope` 为 false：scope 有则要写对，没有 scope 的标题也通过。
