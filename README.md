# JumpServer API 文档

本分支仅维护 JumpServer API 文档及其构建资源。

## 文档结构

- 中文 API 文档：`docs/dev/api/**/*.md`
- 英文 API 文档：`docs/dev/api/**/*.en.md`
- OpenAPI 定义：`swagger.yml`
- MkDocs 配置：`mkdocs.yml`

## 安装依赖

```shell
pip install -r requirements/requirements.txt
```

## 本地预览

```shell
mkdocs serve
```

中文文档使用根路径访问，英文文档使用 `/en/` 路径访问。

## 构建

```shell
mkdocs build --clean
```

构建结果默认输出到 `site/`，英文站点位于 `site/en/`。
