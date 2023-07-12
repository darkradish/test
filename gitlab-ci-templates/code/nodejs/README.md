# NODEJS templates

[[_TOC_]]

## Description

Templates available and description :

| File             |  Function           | Description         |
|------------------|---------------------|---------------------|
| install.yaml     | install:yarn          | use yarn to install deps |
| build.yaml       | build:yarn          | use yarn to build node |
| test.yaml        | test:audit          | auditing with yarn |
|                  | test:unit           | run yarn test      |
|                  | test:lint           | use eslint         |

## build.yaml

### build:yarn

run yarn build with gitlab cache

parameters :

| VARIABLE | DESCRIPTION | DEFAULT VALUE |
|----------|-------------|---------------|
| NODE_VERSION | version of nodejs | lts |
| IMAGE_TYPE | image type | slim |
| YARN_VERSION | version of yarn | lts |
| NODE_ENV | environment to use node | production |
| BUILD_DIR | directory where app is built | dist/ |
| BUILD_TARGET | target for the build | '' |

Example usage :
```yaml
build:
    extends: .template:nodejs:build:yarn
    stage: Build
```

## install.yaml

### install:yarn

run yarn install with gitlab cache

parameters :

| VARIABLE | DESCRIPTION | DEFAULT VALUE |
|----------|-------------|---------------|
| NODE_VERSION | version of nodejs | lts |
| IMAGE_TYPE | image type | slim |
| YARN_VERSION | version of yarn | lts |
| NODE_ENV | environment to use node | development |

## test.yaml

### test:audit

use improved-yarn-audit to audit code

parameters :

| VARIABLE | DESCRIPTION | DEFAULT VALUE |
|----------|-------------|---------------|
| NODE_VERSION | version of nodejs | lts |
| IMAGE_TYPE | image type | slim |
| NODE_ENV | environment to use node | development |
| AUDIT_LEVEL | audit level minimum (info, low, moderate, high, critical) | low |

### test:lint

run yarn eslint, generate a report (artifact)

parameters :

| VARIABLE | DESCRIPTION | DEFAULT VALUE |
|----------|-------------|---------------|
| NODE_VERSION | version of nodejs | lts |
| IMAGE_TYPE | image type | slim |
| NODE_ENV | environment to use node | development |


### test:unit

run yarn test, must generate a junit.xml

parameters :

| VARIABLE | DESCRIPTION | DEFAULT VALUE |
|----------|-------------|---------------|
| NODE_VERSION | version of nodejs | lts |
| IMAGE_TYPE | image type | slim |
| NODE_ENV | environment to use node | development |
| ENV_CI | specify ci env | true |
