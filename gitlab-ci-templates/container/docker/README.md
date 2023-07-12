# Docker templates

[[_TOC_]]

## Description

Templates available and description:

| File       | Function                  | Description                                                      |
| ---------- | ------------------------- | -------------------------------------------------------------    |
| auth.yaml  | docker:auth:credential    | docker authentication with docker-credential-gcr                 |
| build.yaml | docker:build:kaniko       | standard build with Kaniko for Google Artifact Registry          |
|            | docker:build:artifactory  | standard build with Kaniko for Artifactory                       |
| test.yaml  | docker:test:sizing        | use dive to test sizing of a docker                              |
|            | docker:test:security      | use trivy to test vulnerability of a docker                      |
|            | docker:test:lint          | use hadolint for lint dockerfile                                 |

## auth.yaml

### auth:credential

Login to docker registry with docker-credential-gcr:

https://cloud.google.com/artifact-registry/docs/docker/authentication#standalone-helper

parameters :

| VARIABLE   | DESCRIPTION       | DEFAULT VALUE              |
| ---------- | ----------------- | -------------------------- |
| DOCKER_REGISTRY | registry to log in | value comes from gitlab ci |

Example
```yaml
include:
  - project: 'common/gitlab-ci-templates'
    ref: <tag or branch name>
    file:
      - '/container/docker/auth.yaml

job:
  script:
     - !reference[ .templates:docker:auth:credential , script ]
```

## build.yaml

```yaml
include:
  - project: 'common/gitlab-ci-templates'
    ref: <tag or branch name>
    file:
      - '/container/docker/build.yaml'
```

### docker:build:kaniko

Build with Kaniko and push docker image to Google Artifact Registry

For reference: 
* https://docs.gitlab.com/ee/ci/docker/using_kaniko.html
* https://github.com/GoogleContainerTools/kaniko

| VARIABLE                | DESCRIPTION                                                                                                                              | DEFAULT VALUE                         |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------- |
| REGISTRY_PATH           | Path where docker image will be uploaded in Google Artifact Registry                                                                     | apps/${CI_PROJECT_NAME}               |
| REGISTRY_NAME           | Artifact Registry root folder where image will be stored                                                                                 | mdm-docker-registry                   |
| KANIKO_VERSION          | Kaniko version used to build Docker images <br> See more here https://console.cloud.google.com/gcr/images/kaniko-project/global/executor | 1.8.1                                 |
| KANIKO_CACHE            | Set this flag to opt into caching with kaniko                                                                                            | false                                 |
| KANIKO_OPTS             | Kaniko options to pass to the build command                                                                                              | <empty>                               |
| IMAGE_TAG               | Tag of the image to build                                                                                                                | $CI_COMMIT_SHORT_SHA                  |
| IMAGE_NAME              | Name of the image to build                                                                                                               | $CI_PROJECT_NAME                      |
| DOCKERFILE_NAME         | The Docker file name                                                                                                                     | Dockerfile                            |
| REGISTRY_PROJECT        | The project of registry                                                                                                                  | $GOOGLE_ARTIFACT_REGISTRY_PROJECT |
| REGISTRIES_DESTINATIONS | Complete paths of Docker Image in Google Artifact Registry and/or other registry                                                         | europe-west1-docker.pkg.dev/${REGISTRY_PROJECT}/${REGISTRY_NAME}/${REGISTRY_PATH}/${IMAGE_NAME}:${IMAGE_TAG} |
| KUBERNETES_SERVICE_ACCOUNT_OVERWRITE    | K8s Service Account used to execute the build                                                                            | $KUBERNETES_SERVICE_ACCOUNT_GAR |

`$IMAGE_NAME` variable must be defined at project level

`$KUBERNETES_SERVICE_ACCOUNT_GAR` and `$GOOGLE_ARTIFACT_REGISTRY_PROJECT` are set in the Gitlab Groups cicd variables (`/core` and `/web`)

If you want to enable Kaniko caching feature-s, you'll have to do it at project level using `KANIKO_OPTS` variable as described below:

```
build:docker:
  extends: .template:docker:build:kaniko
  stage: Build Docker
  variables:
    IMAGE_NAME: my_docker_image_name
    KANIKO_OPTS: "--cache=true --cache-ttl=168h"
```
### docker:build:kaniko:artifactory

Build with Kaniko and push docker image to GCR
for reference: 
* https://docs.gitlab.com/ee/ci/docker/using_kaniko.html
* https://github.com/GoogleContainerTools/kaniko

| VARIABLE                        | DESCRIPTION                                                          | DEFAULT VALUE                                                         |
| ------------------------------- | -------------------------------------------------------------------- | --------------------------------------------------------------------- |
| ARTIFACTORY_REGISTRY_URL        | The Artifactory registry URL                                         | <empty>                                                               |
| ARTIFACTORY_REGISTRY_USERNAME   | The Artifactory registry username                                    | <empty>                                                               |
| ARTIFACTORY_REGISTRY_PASSWORD   | The Artifactory registry password                                    | <empty>                                                               |
| ARTIFACTORY_PATH                | The relative path where docker image will be uploaded in Artifactory | <empty>                                                               |
| REGISTRIES_DESTINATIONS         | Docker image registry                                                | \$ARTIFACTORY_REGISTRY_URL/\$ARTIFACTORY_PATH/\$IMAGE_NAME:$IMAGE_TAG |


`$IMAGE_NAME` variable must be defined at project level

If you want to enable Kaniko caching feature-s, you'll have to do it at project level using `KANIKO_OPTS` variable as described below:

```
build:docker:
  extends: .docker:build:kaniko:artifactory
  stage: Build Docker
  variables:
    ARTIFACTORY_REGISTRY_URL: my_artifactory_registry_url
    ARTIFACTORY_REGISTRY_USERNAME: my_artifactory_registry_username
    ARTIFACTORY_REGISTRY_PASSWORD: my_artifactory_registry_password
    ARTIFACTORY_PATH: my_artifactory_relative_path
    IMAGE_TAG: my_image_tag
    IMAGE_NAME: my_docker_image_name
    KANIKO_OPTS: "--cache=true --cache-ttl=168h"
```

## test.yaml

```yaml
include:
  - project: 'common/gitlab-ci-templates'
    ref: <tag or branch name>
    file:
      - '/container/docker/test.yaml'
```

### test:security

Reference : https://github.com/aquasecurity/trivy

| VARIABLE           | DESCRIPTION                      | DEFAULT VALUE                  |
| ------------------ | -------------------------------- | ------------------------------ |
| DOCKER_REGISTRY    | the docker registry              | "europe-west1-docker.pkg.dev"                    |
| DOCKER_IMAGE       | name of docker image             |
| DOCKER_TAG         | tag of the image                 |
| DOCKER_TLS_CERTDIR | ****                             | ""                             |
| DOCKER_DRIVER      |                                  | overlay                        |
| DOCKER_HOST        |                                  | tcp://localhost:2375           |

### test:sizing

Reference : https://github.com/wagoodman/dive

| VARIABLE           | DESCRIPTION                      | DEFAULT VALUE                  |
| ------------------ | -------------------------------- | ------------------------------ |
| DOCKER_REGISTRY    | the docker registry              | "europe-west1-docker.pkg.dev"                    |
| DOCKER_IMAGE       | name of docker image             |
| DOCKER_TAG         | tag of the image                 |
| DOCKER_TLS_CERTDIR |                                  | ""                             |
| DOCKER_DRIVER      |                                  | overlay                        |
| DOCKER_HOST        |                                  | tcp://localhost:2375           |

### test:lint

Reference : https://github.com/hadolint/hadolint

| VARIABLE     | DESCRIPTION                                             | DEFAULT VALUE |
| ------------ | ------------------------------------------------------- | ------------- |
| DOCKER_FILE  | the Dockerfile to be linted                             | Dockerfile    |
| IGNORE_CODES | list of hadolint code to ignore. refer to documentation | ""            |
