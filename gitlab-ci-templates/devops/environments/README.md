# Environments templates

[[_TOC_]]

## Description

Environments describe where code is deployed.

https://docs.gitlab.com/ee/ci/environments/index.html
https://docs.gitlab.com/ee/ci/yaml/README.html#environment

Templates available and description :

| File         | Function        | Description                                          |
|--------------|-----------------|------------------------------------------------------|
| digital.yaml | digital:review  | Configure review environment where code is deployed  |
|              | digital:staging | Configure staging environment where code is deployed |

## digital.yaml

### digital:review

Configure review environment where code is deployed

Step `deploy:review:stop` must be configured. It is called `on_stop` of environment.

parameters :

| VARIABLE          | DESCRIPTION    | DEFAULT VALUE                            |
|-------------------|----------------|------------------------------------------|
| GOOGLE_PROJECT_ID | GCP project id | $GOOGLE_CLOUD_PROJECT provided by gitlab |

### digital:staging

Configure staging environment where code is deployed

Step `deploy:review:stop` must be configured. It is called `on_stop` of environment.

parameters :

| VARIABLE          | DESCRIPTION    | DEFAULT VALUE                            |
|-------------------|----------------|------------------------------------------|
| GOOGLE_PROJECT_ID | GCP project id | $GOOGLE_CLOUD_PROJECT provided by gitlab |
