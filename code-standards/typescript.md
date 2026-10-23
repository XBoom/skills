# TypeScript 规范

适用于 DeepWiki 前端（`frontend-service`）和 Requira 全仓。通用约定见 [common.md](common.md)。两个仓库的格式配置不同，都是已决议的差异，落地文件分别在 [repos/deepwiki.md](repos/deepwiki.md) 和 [repos/requira.md](repos/requira.md)。

<a id="format"></a>

## 1. 格式（Prettier）

排版只用 Prettier。ESLint 必须继承 `eslint-config-prettier`，避免规则互相打架 [src: deepwiki-microservices:sdd/engineering-baseline/STANDARD.md §2]。

两套配置都用行宽 100、尾逗号 `all`、缩进 2。引号和分号按仓库现有写法保留。

| 仓库 | 分号 | 引号 | 尾逗号 | 行宽 | 状态 |
|---|---|---|---|---|---|
| DeepWiki `frontend-service` | 有 | 双引号 | all | 100 | 维持现有 `.prettierrc.json` [src: frontend-service/.prettierrc.json] |
| Requira | 无（`semi: false`） | 单引号（`singleQuote: true`） | all | 100 | 决议 1、决议 2。另加 `tabWidth: 2`、`arrowParens: always` |

Requira 在 2026-10-21 还没有格式化配置。抽样 26 个文件里，以 `;` 结尾的行是 0 行，import 单引号 102 处、双引号 0 处 [实测]。行宽 100 的一次性排版大约改 25% 的行，单独成 `style` PR，并写入 `.git-blame-ignore-revs`。模板见 [repos/requira.md](repos/requira.md#prettier)。

<a id="lint"></a>

## 2. Lint（ESLint）

基线是 typescript-eslint `recommended` 加 `eslint-config-prettier`。Next 应用再加 `eslint-config-next` [src: https://typescript-eslint.io/getting-started/][src: https://nextjs.org/docs/app/guides/upgrading/version-16]。

- 使用 flat config（`eslint.config.mjs`）。`next lint` 不作为长期入口：Next 16 已经移除它 [src: https://nextjs.org/docs/app/api-reference/config/eslint]。
- DeepWiki 前端当前的 `.eslintrc.json` 只继承 `next/core-web-vitals`，没有继承 `prettier`，与 STANDARD 不一致 [src: frontend-service/.eslintrc.json；STANDARD.md §3]。P1 换成差异页里的 flat config。
- Requira 在 2026-10-21 没有 ESLint。抽样 26 个文件只有 2 条问题（1 条 `no-console`，1 条 `no-unused-vars`）[实测]。P0 加入配置，CI 只报告；P1 起 error 阻断。
- `no-console` 为 `warn`。CLI、scripts、测试目录不检查这条。见 Requira 差异页的忽略范围。
- 业务代码禁止直接 import DeepSeek、MiniMax SDK 或 `openai`，必须走 `ICodeGenLLM`。用 `no-restricted-imports` 落实，在 `llm-gateway/src/adapters/**` 放开 [src: Requira:.cursorrules]。

<a id="types"></a>

## 3. 类型

- Requira 各包维持 `strict: true`，CI 继续跑 `pnpm typecheck` [src: Requira:.github/workflows/ci.yml]。
- `packages/saas-api/tsconfig.json` 把 `**/*.test.ts` 排除在 tsc 之外，测试文件不会被类型检查（Vitest 默认也不做类型检查）[src: Requira:packages/saas-api/tsconfig.json]。P2 补上：增加 `tsconfig.test.json`，或用 `vitest --typecheck`。两种里选一种，全仓统一。
- DeepWiki 前端在 CI 里已经跑 `tsc --noEmit`，但没有 `typecheck` 脚本，`check.sh` 里直接写了 `npx tsc` [src: sdd/engineering-baseline/README.md, scripts/check.sh]。P1 在 `package.json` 增加 `"typecheck": "tsc --noEmit"`，`check.sh` 改为调用 `npm run typecheck`。

<a id="tests"></a>

## 4. 测试

一个仓库内只用一种测试运行器。

- DeepWiki 前端保持 `node:test`（现有 12 个测试文件）。STANDARD 里写死 Vitest 的那一行，在 P0 改为“Vitest 或 node:test，仓库内只用一种” [src: .github/workflows/ci.yml][src: deepwiki-microservices:sdd/engineering-baseline/STANDARD.md §3]。
- Requira 保持 Vitest（`*.test.ts` 与源码放在一起）和 Playwright e2e [src: git tree]。

<a id="errors"></a>

## 5. 错误处理

- 用带类型的错误类，例如 `LlmGatewayError` 和 `writers/errors.ts`。不 `throw` 字符串 [src: Requira:packages/llm-gateway/src/ICodeGenLLM.ts, packages/spec-writer/src/writers/errors.ts]。
- Requira 对外错误码以 `outputs/02-设计/15-error-codes.md` 为准 [src: Requira:.cursorrules]。
- `catch (e)` 时把 `e` 当作 `unknown`，先收窄类型再用（`strict` 下 TS 默认就是 `useUnknownInCatchVariables`）。

<a id="logging"></a>

## 6. 日志

- Requira 的关键事件走 `ITelemetry`，业务代码不用 `console.log` [src: Requira:.cursorrules]。ESLint 的 `no-console: warn` 用来提醒。CLI、scripts、测试目录关闭这条规则。
- 两仓都禁止在日志里打印密钥、token 或完整的用户输入。Requira CI 已经有 `check-secrets.mjs` [src: Requira:.github/workflows/ci.yml]。

<a id="docs"></a>

## 7. 注释

导出的函数、类型用 `/** */`（TSDoc）写一行说明。`packages/llm-gateway/src/gateway.ts` 里的 `selectProvider` 就是这样写的 [src: Requira:packages/llm-gateway/src/gateway.ts]。

注释写为什么。标识符只用 ASCII [src: https://google.github.io/styleguide/tsguide.html#identifiers]。`TODO` / `FIXME` 的写法见 [common.md](common.md#comments)。

<a id="naming"></a>

## 8. 文件名

命名表在 [common.md](common.md#naming)。和 TypeScript 直接相关的约定：

- 普通源文件用 `kebab-case.ts`，React 组件用 `PascalCase.tsx`，hook 用 `useXxx.ts`。
- 接口默认不加 `I` 前缀。例外只有 `.cursorrules` 点名的 `IAuthProvider`、`IBillingProvider`、`ICodeGenLLM`、`ITelemetry`，文件名照旧（`ICodeGenLLM.ts`）[src: Requira:.cursorrules][src: https://google.github.io/styleguide/tsguide.html#naming-style]。
- Requira 的存量文件名有两处草案数字，v1.0 不另行清点：284 个 src 文件里除 6 个 PascalCase 和 4 个 camelCase 外都是 kebab-case；附录点名 `registerSpecWriter.ts` 等 camelCase，以及 `ICodeGenLLM.ts` 等端口接口文件。改名规则见 [repos/requira.md](repos/requira.md#names)。
