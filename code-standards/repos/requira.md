# Requira 差异

仓库：https://github.com/XBoom/Requira

规范正文在共用规范仓，不在本仓复制。本页只记录 Requira 相对 [通用约定](../common.md) 和 [TypeScript 规范](../typescript.md) 的差异，以及要落进去的文件。

<a id="link"></a>

## 项目仓里只放这段链接

P0 在 `docs/` 增加一页（或在现有文档入口顶部加入），内容就是下面这一段。链接指向 `XBoom/skills` 的 `main`，本页合并进 `main` 之后再贴到项目仓。`.cursorrules` 继续保留技术栈、端口接口和“禁止直连 SDK”等架构约束，不把规范正文贴进去。

```markdown
# 工程规范

正文在共用规范仓，不在本仓复制：

- 代码规范 v1.0：https://github.com/XBoom/skills/blob/main/code-standards/README.md
- 通用约定：https://github.com/XBoom/skills/blob/main/code-standards/common.md
- TypeScript：https://github.com/XBoom/skills/blob/main/code-standards/typescript.md
- 本仓差异：https://github.com/XBoom/skills/blob/main/code-standards/repos/requira.md
```

<a id="delta"></a>

## 相对通用规范的差异

| 项 | 本仓做法 |
|---|---|
| 格式 | 无分号、单引号、尾逗号 all、行宽 100（决议 1、决议 2）[实测] |
| Lint | P0 起用仓库根目录的 ESLint flat config。2026-10-21 还没有 ESLint/Biome，`requira-skills/hooks/pre-commit-lang-lint.sh` 发现没配置就跳过 [src: Requira:requira-skills/hooks/pre-commit-lang-lint.sh] |
| 类型 | 各包 `strict: true`，CI 跑 `pnpm typecheck` [src: Requira:.github/workflows/ci.yml]。测试文件的类型检查见 P2 |
| 测试 | Vitest，`*.test.ts` 与源码放在一起；另有 Playwright e2e [src: git tree] |
| 检查入口 | 根脚本 `pnpm check`，依次跑 `format:check`、`lint`、`typecheck`、`test` |
| 钩子 | `.githooks/pre-commit` 调用 `pnpm check:static`，配合 `git config core.hooksPath .githooks`。不用 Husky，与 STANDARD “不同时启用 pre-commit 框架”一致 [src: deepwiki-microservices:sdd/engineering-baseline/STANDARD.md] |
| 包管理 | pnpm@10.34.6，只用这一个 [src: Requira:package.json] |
| 依赖更新 | 2026-10-21 没有 Dependabot。P3 照 DeepWiki 的 `.github/dependabot.yml` 加一份 [src: .github/dependabot.yml] |
| 合并 | 统一 squash。禁止直接 push 到 `main`，禁止 force-push 到 `main`（决议 3）。P3 用分支保护执行 |
| 目录 | `apps/*` 是可部署应用，`packages/*` 是库。每个包有 `src/`，`dist/` 不提交 [src: Requira:pnpm-workspace.yaml, packages/*/tsconfig.json] |
| 文档目录 | `docs/` 和 `outputs/` [src: Requira:.cursorrules] |
| 错误码 | 对外错误码以 `outputs/02-设计/15-error-codes.md` 为准 [src: Requira:.cursorrules] |
| 端口接口 | `IAuthProvider`、`IBillingProvider`、`ICodeGenLLM`、`ITelemetry` 保留 `I` 前缀和现有文件名 [src: Requira:.cursorrules] |
| 提交 scope | `saas-web`、`saas-api`、`spec-writer`、`k8s`、`deploy`、`ci` [src: 两仓最近 30 条 commit] |

框架是 Next.js 14、Express、Drizzle、Zod、pnpm workspace [src: Requira:apps/saas-web/package.json, packages/saas-api/package.json, .cursorrules]。

<a id="prettier"></a>

## P0：格式与 lint 文件

下面这些文件是 v1.0 的落地模板。P0 放进仓库，CI 只报告；P1 做一次全仓排版之后，`format:check` 和 lint error 改为阻断。

### `.editorconfig`

与 DeepWiki 用同一份，复制 DeepWiki 的 `sdd/engineering-baseline/templates/.editorconfig`。条文见 [common.md](../common.md#editor)。行宽 100。

### `.prettierrc.json`

```json
{
  "semi": false,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2,
  "arrowParens": "always"
}
```

`semi: false` 与 `singleQuote: true` 是决议 1。`printWidth: 100` 是决议 2。抽样在这套配置下，26 个文件里 18 个要改，1941 行中改动 481 行（约 25%，主要是折行）[实测]。

### `.prettierignore`

```
node_modules
dist
.next
pnpm-lock.yaml
outputs
stitch_custom_project_workspace
**/*.md
```

### `eslint.config.mjs`（仓库根目录）

```js
// @ts-check
import js from '@eslint/js'
import tseslint from 'typescript-eslint'
import prettier from 'eslint-config-prettier'
import { defineConfig, globalIgnores } from 'eslint/config'

export default defineConfig([
  globalIgnores(['**/dist/**', '**/.next/**', '**/node_modules/**', 'outputs/**', 'stitch_custom_project_workspace/**']),
  {
    files: ['**/*.{ts,tsx,mts,js,mjs}'],
    extends: [js.configs.recommended, tseslint.configs.recommended],
    rules: {
      'no-console': 'warn',
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_', varsIgnorePattern: '^_' }],
      'no-restricted-imports': ['error', {
        patterns: [{ group: ['deepseek*', '@deepseek-ai/*', 'openai'], message: '业务代码走 ICodeGenLLM（@requira/llm-gateway）' }],
      }],
    },
  },
  { files: ['packages/llm-gateway/src/adapters/**'], rules: { 'no-restricted-imports': 'off' } },
  { files: ['scripts/**', 'apps/requira-cli/**', '**/*.test.ts', '**/e2e/**'], rules: { 'no-console': 'off' } },
  prettier,
])
```

saas-web 另外放一份 `apps/saas-web/eslint.config.mjs`，展开 `eslint-config-next/core-web-vitals`。Next 14 对应的 `eslint-config-next` 14 只提供 eslintrc 格式，接进 flat config 要借助 `@eslint/eslintrc` 的 FlatCompat，或者等 saas-web 升到 Next 16（原生 flat config）[src: https://nextjs.org/docs/app/guides/upgrading/version-16][src: https://nextjs.org/docs/app/api-reference/config/eslint]。在升到 Next 16 之前，saas-web 只跑根配置的 typescript-eslint。

抽样 26 个文件在 recommended 下只有 2 条问题（1 条 `no-console`，1 条 `no-unused-vars`），全仓估计约 30 [实测]。

### 根 `package.json` scripts（追加）

```json
{
  "lint": "eslint .",
  "format": "prettier --write .",
  "format:check": "prettier --check .",
  "check:static": "pnpm format:check && pnpm lint",
  "check": "pnpm check:static && pnpm typecheck && pnpm test"
}
```

devDependencies 加在根目录：`eslint`、`@eslint/js`、`typescript-eslint`、`eslint-config-prettier`、`prettier`（用 `pnpm add -Dw` 安装）。

### `.github/workflows/ci.yml`

在 `pnpm install` 之后插入：

```yaml
      - run: pnpm format:check
        continue-on-error: true   # P0 只报告；P1 一次性排版后删掉这行
      - run: pnpm lint
        continue-on-error: true   # P1 后改为阻断
```

P1 起 warning 不阻断，用 `--max-warnings <当前值>` 卡住，只许减少不许增加。

### `.githooks/pre-commit`

```sh
#!/bin/sh
set -eu
cd "$(git rev-parse --show-toplevel)"
pnpm check:static
```

### `.github/workflows/pr-title.yml`

使用 [common.md](../common.md#pr-title) 的 workflow。P0 接上，P1 起阻断。

<a id="merge"></a>

## 合并

决议 3：PR 只用 squash，标题用 [Conventional Commits](../common.md#commits)，squash 之后标题就是 `main` 上的提交信息。禁止直接 push 到 `main`，禁止 force-push 到 `main`。

2026-10-15 `main` 上 CI 连续失败了 3 次，其中有直接 push 的 `fix(saas): 场景1冒烟三阻塞…` 和手工 merge 的 `merge: 合入 PR #27…` [src: Requira workflow runs / commits]。分支保护在 P3 打开；打开之前这条已经是规范。

最近 30 条 commit 里有 7 条没有 type，例如 `用户应用按 k8s 驱动发到本机 kind (#26)`，应写成 `feat(deploy): 用户应用按 k8s 驱动发到本机 kind` [src: 两仓最近 30 条 commit]。

<a id="types"></a>

## P2：测试文件的类型检查

`packages/saas-api/tsconfig.json` 把 `**/*.test.ts` 排除在 tsc 之外 [src: Requira:packages/saas-api/tsconfig.json]。P2 加 `tsconfig.test.json`，或改用 `vitest --typecheck`。全仓只用其中一种。

<a id="names"></a>

## 文件名

2026-10-21 的统计有两处数字，都来自草案，v1.0 不另行清点：

- 284 个 src 文件里，除 6 个 PascalCase 和 4 个 camelCase 外都是 kebab-case。
- 附录点名的对象是 `registerSpecWriter.ts` 等 4 个 camelCase，以及 `ICodeGenLLM.ts` 等 PascalCase 接口文件。

新文件用 kebab-case。已有 camelCase 在修改该文件的 PR 里改成 kebab-case，不为改名单独做阻断。`.cursorrules` 点名的端口接口文件保持 PascalCase 和 `I` 前缀 [src: Requira:.cursorrules][src: https://google.github.io/styleguide/tsguide.html#naming-style]。

<a id="inconsistencies"></a>

## 现存不一致（2026-10-21）

1. 根目录没有 Prettier 或 ESLint 配置。`requira-skills/hooks/pre-commit-lang-lint.sh` 遇到没配置就跳过，所以当时没有任何风格检查。P0 放入本页的配置和 `.githooks/pre-commit`。
2. `packages/saas-api/tsconfig.json` 把测试排除在类型检查之外。P2 补上。
3. 文件名里有 camelCase。处理方式见上一节。`ICodeGenLLM.ts` 这类端口接口文件保留。
4. 合并方式混有 merge commit 和直推 `main`。自 v1.0 起改为 squash，并禁止直推。

<a id="phases"></a>

## 本仓落地对照

| 阶段 | 本仓动作 | 门禁 |
|---|---|---|
| P0 | 链接段落、`.editorconfig`、Prettier、ESLint、根 scripts、`.githooks/pre-commit`、CI 的 `format:check` 与 `lint`、`pr-title.yml` | 只报告（`continue-on-error`），不阻断 |
| P1 | 一次全仓 `prettier --write`（单独 PR，`style`，无分号、单引号、行宽 100），登记 `.git-blame-ignore-revs`；修完 ESLint error | `format:check` 与 lint error 阻断；warning 用 `--max-warnings` 只减不增；PR 标题阻断 |
| P2 | 测试文件补上类型检查（`tsconfig.test.json` 或 `vitest --typecheck` 二选一） | 与现有 `pnpm typecheck` 一样阻断 |
| P3 | `main` 分支保护（必须有 PR、必须 squash、必须通过 required checks）；加 Dependabot | 禁止直推 `main` 由分支保护执行。pnpm audit 先出报告 |

阶段定义的全文在 [README.md](../README.md#phases)。
