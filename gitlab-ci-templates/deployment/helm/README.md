# Helm templates

[[_TOC_]]

## Description

Templates available and description :

| File        | Function                                  | Description                                 |
| ----------- | ----------------------------------------- | ------------------------------------------- |
| auth.yaml   | helm:auth:gcs                             | Authenticate helm to repo gcs               |
| build.yaml  | helm:build:package                        | Package helm                                |
|             | helm:build:package:gcs                    | Package helm and send into gcs              |
|             | helm:send:package                         | send package into gitlab repo               |
| test.yaml   | helm:test:lint:simple                     | Linting charts                              |
|             | helm:test:lint                            | Linting charts with gcs auth                |
|             | helm:test:render-template                 | Render templates for every env availables   |
| deploy.yaml | helm:deploy:install                       | Standard install chart on kube              |
|             | helm:deploy:install:review                | Review env install chart on kube            |
|             | helm:deploy:install:review:digital        | Review env install chart on kube for digit  |
|             | helm:deploy:install:staging               | Staging env install chart on kube           |
|             | helm:deploy:install:staging:digital       | Staging env install chart on kube for digit |
|             | helm:deploy:install:preproduction:digital | Staging env install chart on kube for digit |
|             | helm:deploy:install:production:digital    | Staging env install chart on kube for digit |
|             | helm:deploy:uninstall                     | Standard install chart on kube              |
|             | helm:deploy:uninstall:review              | Review env install chart on kube            |
| setup.yaml  | .template:helm:setup                      | setup hel for other jobs                    |

## auth

### helm:auth:gcs

Authenticate to Google and add GCS bucket as helm repository via a helm plugin.

Variables :

```yaml
HELM_GCS_BUCKET: "gs://${DIGITAL_HELM_CHART_REPOSITORY_BUCKET}/repo"
HELM_CREDENTIALS: $DIGITAL_HELM_CHART_REPO_CREATOR_KEY
HELM_REPO: "mdm"
HELM_DIR: "chart/"
HELM_GCS_ENABLED: "true"
```

## build

### build:package

Build a generic helm chart

Variables :

```yaml
HELM_REPO: "mdm"
HELM_DIR: "chart/"
```

### build:package:gcs

Same as `build:package` and it send helm chart to GCS bucket.

Variables :

```yaml
HELM_FORCE: false # force push chart to gcs bucket
HELM_REPO: "mdm"
HELM_DIR: "chart/"
```

## test

### test:lint:simple

Linting charts

Variables:

```yaml
HELM_REPO: "mdm"
HELM_DIR: "chart/"
HELM_OPTS: ""
HELM_SEARCH: true
HELM_SEARCH_MAXDEPTH: 2
```

### test:lint

Linting charts

Variables:

```yaml
HELM_REPO: "mdm"
HELM_DIR: "chart/"
HELM_OPTS: ""
HELM_SEARCH_MAXDEPTH: 2
```

### test:render-template

Template charts before deploying them

Variables:

```yaml
HELM_REPO: "mdm"
HELM_DIR: "chart/"
HELM_OPTS: ""
```

## deploy

### deploy:install

Generic template to install any helm chart

Variables:

```yaml
HELM_OPTS: ""
RELEASE_NAME: $CI_ENVIRONMENT_SLUG
RELEASE_ENV_TYPE: $CI_ENVIRONMENT_SLUG
RELEASE_HOST: ""
RELEASE_PATH: ""
HELM_CREDENTIALS: $DIGITAL_HELM_CHART_REPO_VIEWER_KEY
HELM_REPO: "mdm"
HELM_DIR: "chart"
HELM_CUSTOM_KEYS: ""
IMAGE_TAG: $CI_COMMIT_SHORT_SHA
RELEASE_URI_SCHEMA: "https"
RUNDECK_EXPORT: "false"
RUNDECK_HELM_VALUES: ""
HELM_TIMEOUT: "600s"
```

### deploy:install:{environment}

Install helm chart in specific environment.

Possible value of `{environment}` are `review, staging, preproduction, production`.

Variables:

```yaml
RELEASE_HOST: "$CI_ENVIRONMENT_SLUG-$CI_PROJECT_NAME.$RELEASE_DOMAIN"
RELEASE_ENV_TYPE: "review"
```

### deploy:install:{environment}:digital

Install helm chart in specific environment AND in `digital-*` GCP project.

Possible value of `{environment}` are `review, staging, preproduction, production`.

Variables:

```yaml
RELEASE_DOMAIN: "mdm-staging.com"
GOOGLE_PROJECT_ID: "${GOOGLE_CLOUD_DIGITAL_STAGING_PROJECT}"
```

### deploy:uninstall

Uninstall chart

Variables:

```yaml
RELEASE_NAME: $CI_ENVIRONMENT_SLUG
RELEASE_HOST: ""
GIT_STRATEGY: none
HELM_VERSION: 3.5.3
HELM_REPO: "mdm"
HELM_DIR: "chart/"
```

### deploy:uninstall:review

Uninstall chart in review environment.

Variables:

```yaml
CUSTOM_DOMAIN: false
```

If you used template `dns:ns1:add:cname:digital:review` in your gitlab pipeline, you can enable option
`CUSTOM_DOMAIN: true` to automatically uninstall CNAME entry.
