# JumpServer Docs

- https://docs.jumpserver.org

[![Python3](https://img.shields.io/badge/python-3.8-green.svg?style=plastic)](https://www.python.org/)

## 开始

```shell
git clone --depth=1 https://github.com/jumpserver/docs
cd docs
pip install -r requirements/requirements.txt
```

## 运行

```bash
mkdocs serve
```

默认服务中文文档；英文文档使用 `/en/` 路径访问。

The default site serves Chinese documentation. English documentation is available under the `/en/` path.

## 编译

```bash
mkdocs build
```

构建会同时生成中文站点和英文站点：`site/` 与 `site/en/`。中英文页面按同目录的 `.md` 与 `.en.md` 文件配对，维护规则见 [双语文档规范](I18N_GUIDE.md)。

The build produces both locales: `site/` and `site/en/`. Chinese and English pages are paired as `.md` and `.en.md` files in the same directory. See the [Bilingual Documentation Guide](I18N_GUIDE.md) for maintenance rules.

## 帮助

```bash
mkdocs --help
```
