# JumpServer API Documentation

This branch maintains only the JumpServer API documentation and its build assets.

## Documentation Structure

- Chinese API documentation: `docs/dev/api/**/*.md`
- English API documentation: `docs/dev/api/**/*.en.md`
- OpenAPI schema: `swagger.yml`
- MkDocs configuration: `mkdocs.yml`

## Install Dependencies

```shell
pip install -r requirements/requirements.txt
```

## Local Preview

```shell
mkdocs serve
```

The Chinese documentation is served from the root path, and the English documentation is available under `/en/`.

## Build

```shell
mkdocs build --clean
```

The build output is written to `site/` by default, with the English site under `site/en/`.
