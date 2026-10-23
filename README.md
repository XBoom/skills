# skills

Eric 所有项目共用的规范仓。规范正文放在本仓；各项目仓库只放指向这里的链接，以及本仓 `code-standards/repos/` 里写明的差异。

## 代码规范

| 规范 | 版本 | 入口 |
|---|---|---|
| 代码规范 | v1.0（2026-10-23 定稿） | [code-standards/README.md](code-standards/README.md) |

v1.0 覆盖 DeepWiki 主仓 [deepwiki-microservices](https://github.com/XBoom/deepwiki-microservices) 与 [Requira](https://github.com/XBoom/Requira)。之后接入的项目沿用 [通用约定](code-standards/common.md)，并在 `code-standards/repos/` 增加一页差异说明。

## 怎么读

1. 从 [代码规范 v1.0](code-standards/README.md) 进入，看范围、五条决议和落地阶段。
2. 写代码时读 [通用约定](code-standards/common.md)，再读 [Python](code-standards/python.md) 或 [TypeScript](code-standards/typescript.md)。
3. 改某一个仓库时打开对应差异页：[DeepWiki](code-standards/repos/deepwiki.md)、[Requira](code-standards/repos/requira.md)。
4. 项目仓里只粘贴差异页给出的链接段落。正文留在本仓，避免两份规范各自漂移。
