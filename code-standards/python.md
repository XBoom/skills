# Python 规范

适用于 DeepWiki 主仓 [deepwiki-microservices](https://github.com/XBoom/deepwiki-microservices) 的 Python 服务。通用约定见 [common.md](common.md)。该仓还要改的文件和与前端相关的差异见 [repos/deepwiki.md](repos/deepwiki.md)。

现有 Ruff 配置保持不动，v1.0 只做三件事：修正互相矛盾的配置、按档加严、把类型检查接上。互相矛盾的配置（Makefile、PR 模板）在差异页列出。

<a id="format"></a>

## 1. 格式

Ruff format，双引号，行宽 100。CI 在 2026-10-21 是绿的，抽样 28 个文件全部已格式化 [实测]。

排版只用 Ruff format，同时不装 Black [src: deepwiki-microservices:sdd/engineering-baseline/STANDARD.md §2]。`Makefile` 里仍调用 `black --check` 的部分在 P0 删掉，见差异页。

<a id="lint"></a>

## 2. Lint

保留选择集 `E F W I N UP B C4 SIM`，ignore `E501 B008 B904 E402 N818`。

- `B904`：存量继续豁免。新代码必须在 `except` 里写 `raise ... from err`。抽样有 2 处，全仓估计约 20 处 [实测]。存量清零后，把 `B904` 从 ignore 里删掉。
- 规则一次加一组，修到全绿再加下一组。v1.0 不开 `D`、`PT`、`RUF`、`PTH`、`TRY`、`ERA`、`ALL`。抽样依据见 [README.md 的定稿依据](README.md#evidence)。
- `ANN` 只在新文件上要求，对应 STANDARD 第 4 档。抽样 6 处，全仓估计约 60 [实测][src: deepwiki-microservices:sdd/engineering-baseline/STANDARD.md §4]。
- `S`（bandit）先出报告，不在 v1.0 阻断。抽样 3 处（`S105` 硬编码默认密钥 ×2、`S104` 监听 0.0.0.0 ×1），全仓估计约 30，多数是开发期默认值，确认后用带规则码的 `noqa` 加理由 [实测]。

<a id="mypy"></a>

## 3. 类型检查（mypy）

决议 4：mypy 按服务逐个检查。先只报告，不阻断；某个服务的错误清零之后，将该服务改为阻断。其余服务继续只报告。

不把“等全部服务一次修完再统一阻断”当作路径。`task-service` 抽样有 6 个错误，分布在 3 个文件，全仓估计约 50 到 60 [实测]。

抽样里有一处真实缺陷：`services/scheduler_proxy.py` 第 119 行和第 174 行定义了两次 `_sync_trigger_evaluation`，后一个会覆盖前一个（mypy `no-redef`）。另一处是 `api/routes/tasks.py:183` 的返回值类型和注解不一致 [实测]。Ruff 现有规则抓不到重复定义，因为 F811 只在名字被使用之前重定义时才报。P2 先修这两处，再把 mypy 接进 CI。

报告模式和阻断名单的接法见 [repos/deepwiki.md](repos/deepwiki.md#mypy)。类型检查只用 mypy，不并行再引入第二种 Python 类型检查器。

<a id="tests"></a>

## 4. 测试

pytest。新测试写 pytest 风格的普通 `assert`。存量的 `unittest.TestCase` 不强制迁移。抽样中 `PT009` 有 51 处，所以 v1.0 不开 `PT` 规则 [实测]。

`Makefile` 里 `unittest discover`、且只覆盖 6 个服务的目标，P0 改为与 CI 相同的 `scripts/check.sh test`（pytest，9 个服务）。见差异页 [src: Makefile；scripts/check.sh；STANDARD.md §2]。

默认测试集不调用真实 LLM、不访问外网，用 `slow` 和 `network` marker 排除 [src: deepwiki-microservices:sdd/engineering-baseline/STANDARD.md §3]。

<a id="sys-path"></a>

## 5. `sys.path.insert`

这是 `E402` 被 ignore 的原因（`app/main.py` 和测试文件都有）[实测]。属于项目结构问题，短期接受。长期每个服务以包的形式安装，或者用 pytest 的 `pythonpath` 配置。v1.0 不把这件事放进 P0–P3 的阻断项。

<a id="punctuation"></a>

## 6. 中文标点

以后如果开 `RUF` 规则，要先把全角标点（`，`、`：`、`（`、`）` 等）加进 `allowed-confusables`。否则抽样 28 个文件就有 379 条 `RUF001/002/003` 误报 [实测][src: https://docs.astral.sh/ruff/settings/#lint_allowed-confusables]。v1.0 不开 `RUF`。

<a id="errors"></a>

## 7. 错误处理

- 只捕获能处理的异常。`except Exception` 只允许出现在边界上（后台线程、轮询、对外 HTTP 调用），而且必须写日志，并在行尾写带规则码的 `# noqa: BLE001`。现有代码就是这么做的（`scheduler_proxy.py` 里 `except Exception as exc:  # noqa: BLE001` 后面跟 `logger.debug(...)`）[src: deepwiki-microservices:task-service/services/scheduler_proxy.py]。抽样 28 个文件里有 30 处 `except Exception`，没有一处是吞掉异常后直接 `pass` [实测]。
- 在 except 里重新抛异常时，新代码必须写 `raise ... from err`（B904）。存量豁免见上文 [Lint](#lint)。
- `HTTPException` 只在 `api/` 层抛。`services/` 层抛领域异常，由路由层转换成 HTTP 响应。

<a id="logging"></a>

## 8. 日志

每个模块写 `logger = logging.getLogger(__name__)`（抽样 11 个模块都这么写）。用 `%s` 惰性格式化，不用 f-string 拼日志。库代码里不用 `print` [src: https://docs.python.org/3/howto/logging.html][src: deepwiki-microservices:sdd/engineering-baseline/STANDARD.md §4 第 4 档][实测]。

日志里不打印密钥、token 或完整的用户输入。

<a id="docstring"></a>

## 9. 文档字符串

模块顶部写一段模块 docstring，说明这个文件负责什么。`task-service/api/deps.py` 是现成的范例。公开函数和类的 docstring 按 Google 格式写（摘要一行，需要时加 `Args:`、`Returns:`、`Raises:`）[src: https://google.github.io/styleguide/pyguide.html#38-comments-and-docstrings]。

不开 pydocstyle 的 `D` 规则。抽样 28 个文件有 271 条，全仓估计约 2500 [实测]。

<a id="layout"></a>

## 10. 服务目录

DeepWiki 服务统一分层：

| 路径 | 内容 |
|---|---|
| `api/` | 路由、依赖注入 |
| `app/` | 应用入口、ASGI、CORS |
| `models/` | schema、ORM |
| `services/` | 业务 |
| `db/` | 数据库 |
| `utils/` | 工具 |
| `tests/` | 测试 |

运行依赖放该服务的 `requirements.txt`，开发工具统一由根目录 `pyproject.toml` 管 [src: deepwiki-microservices:task-service/*, pyproject.toml]。新服务照抄 `task-service` 的结构。

前端目录和 CI 入口见 [repos/deepwiki.md](repos/deepwiki.md)。
