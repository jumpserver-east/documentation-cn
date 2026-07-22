# JumpServer 双语文档规范 / Bilingual Documentation Guide

本文档规定中文与英文文档的配对方式、术语和构建要求。每条规则同时提供英文说明，便于中英文维护者使用。

This guide defines how Chinese and English documentation is paired, written, and validated. Each rule is stated in both languages so that Chinese and English maintainers can follow the same process.

## 文件配对 / File Pairing

- 中文文档使用稳定的基础文件名，例如 `manual/maintenance/env.md`。
- 英文文档在同一目录使用 `.en.md` 后缀，例如 `manual/maintenance/env.en.md`。
- 新增或删除中文页面时，必须在同一个变更中新增或删除对应的英文页面；如果英文翻译暂时不可用，应保留中文页面作为插件回退内容，并在 PR 中说明。
- 文档中的图片、代码和配置文件路径使用与中文页面相同的相对路径。

- Chinese pages use the unsuffixed filename, for example `manual/maintenance/env.md`.
- English pages use the `.en.md` suffix in the same directory, for example `manual/maintenance/env.en.md`.
- Adding or removing a Chinese page requires adding or removing its English pair in the same change. If the translation is not ready, keep the Chinese page as the plugin fallback and explain the gap in the PR.
- Images, commands, and configuration paths must use the same relative paths in both languages.

## 目录与标题 / Navigation and Headings

- `mkdocs.yml` 的 `nav` 只维护中文路径；英文标题通过 i18n 插件的 `nav_translations` 映射。
- `nav_translations` 的键必须与导航中的中文标题完全一致，值使用产品界面中的权威英文。
- 中英文页面的标题层级和编号必须保持一致，这样自动生成的锚点和页面结构才不会分叉。
- 链接到同一文档时使用相对链接（例如 `[参数说明](env.md)`）；i18n 插件会根据当前语言解析对应页面。

- Maintain Chinese paths in `mkdocs.yml`; translate navigation labels through the i18n plugin's `nav_translations` map.
- Keys in `nav_translations` must exactly match the Chinese labels in the navigation. Values must use the product UI's canonical English.
- Keep heading levels and section numbers identical in both languages so generated anchors and page structure remain aligned.
- Use relative links for pages in the same documentation set (for example, `[Configuration Parameters](env.md)`); the i18n plugin resolves the localized page.

## 翻译与术语 / Translation and Terminology

- 优先使用 JumpServer 产品界面中的英文标签；同一中文术语在整套文档中只能对应一个首选英文表达。
- 产品名、组件名、环境变量、命令、文件名、协议名和版本号保持原样，不做音译或大小写改写。
- `asset`、`account`、`system user`、`session`、`ticket`、`node`、`component`、`configuration` 等术语按产品语义使用，不使用泛化的同义词替换。
- 中文页面中的提示、警告、表格列和步骤必须在英文页面中完整呈现，不得因翻译省略安全限制或前置条件。
- 英文句子使用简洁的技术写作风格；按钮、菜单和参数名称使用代码格式或粗体，且与产品界面一致。

- Prefer the English labels used by the JumpServer product UI. A Chinese term must have one preferred English rendering across the documentation set.
- Keep product names, component names, environment variables, commands, filenames, protocol names, and version numbers unchanged. Do not transliterate them or alter their casing.
- Use product-specific terms such as `asset`, `account`, `system user`, `session`, `ticket`, `node`, `component`, and `configuration` consistently; do not replace them with generic synonyms.
- Every note, warning, table field, and prerequisite in the Chinese page must be represented in the English page. Do not omit operational or security constraints during translation.
- Use concise technical English. Format button labels, menu items, and parameter names as code or bold text and keep them consistent with the product UI.

### 核心术语 / Core Glossary

| 中文 | 首选英文 / Canonical English |
| :--- | :--- |
| 资产 | asset |
| 账号 | account |
| 系统用户 | system user |
| 会话 | session |
| 工单 | ticket |
| 节点 | node |
| 组件 | component |
| 配置 | configuration |
| 授权 | permission |
| 会话录像 | session recording |
| 作业 | job |
| 终端 | terminal |
| 改密 | secret change |

Use the canonical English term in the table above unless the product UI or a protocol name requires a more specific expression.

## 代码与示例 / Code and Examples

- 代码块中的命令、变量名、路径、端口和占位符保持可执行格式；仅翻译注释和说明文字。
- 示例中的密钥、令牌和密码必须使用掩码或明显的占位符，禁止提交真实凭据。
- 修改命令或配置示例前，必须确认中文和英文版本的行为仍然等价。

- Keep commands, variable names, paths, ports, and placeholders executable inside code blocks; translate comments and explanatory text only.
- Mask keys, tokens, and passwords in examples or use unmistakable placeholders. Never commit real credentials.
- Before changing a command or configuration example, verify that the Chinese and English versions still describe equivalent behavior.

## 校验 / Validation

在提交前执行以下检查：

Run these checks before submitting a change:

```bash
# Install the documentation dependencies
pip install -r requirements/requirements.txt

# Build both locales from a clean output directory
mkdocs build --clean
```

本分支只维护运维文档。构建必须成功，且不得出现缺失页面、断链或未挂载页面。当前运维导航没有单独的 `index.md`，因此 i18n 插件会为中文和英文各输出一条无首页提示。

This branch maintains operations documentation only. The build must succeed without missing pages, broken links, or unlisted pages. The operations navigation has no separate `index.md`, so the i18n plugin reports one missing-homepage notice for each locale.

检查清单：

Validation checklist:

- 每个导航页面都有对应的 `.en.md` 文件。
- `nav_translations` 中没有失配或重复的中文键。
- 中英文页面的标题层级、图片引用、代码围栏和表格列数一致。
- 英文页面不包含未翻译的中文说明（产品名、命令和专有名词除外）。
- 构建输出在 `site/` 生成默认中文站点、在 `site/en/` 生成英文站点，且搜索索引可以正常生成。

- Every page in the navigation has an `.en.md` counterpart.
- `nav_translations` contains no unmatched or duplicate Chinese keys.
- Heading levels, image references, code fences, and table column counts match between locales.
- English pages contain no untranslated Chinese explanations, except for product names, commands, and proper nouns.
- The build output contains the default Chinese site in `site/`, the English site in `site/en/`, and valid search indexes for both locales.
