# Build template

[[_TOC_]]

## Description

Templates available and description :

| File        | Function                                  | Description                                                                        |
| ----------- | ----------------------------------------- | ----------------------------------------------------------------                   |
| config.yaml | config:set-project                        | Set the project ID of the Cloud Platform project to operate on                     |
| tag.yaml    | tag-image                                 | Generic Tag for Google Artifact Registry                                           |
| pubsub.yaml | gcloud:pubsub:topics:create               |                                                                                    |
|             | gcloud:pubsub:topics:create:review        |                                                                                    |
|             | gcloud:pubsub:subscriptions:create        |                                                                                    |
|             | gcloud:pubsub:subscriptions:create:review |                                                                                    |
|             | gcloud:pubsub:topics:delete               |                                                                                    |
|             | gcloud:pubsub:topics:delete:review        |                                                                                    |
|             | gcloud:pubsub:subscriptions:delete        |                                                                                    |
|             | gcloud:pubsub:subscriptions:delete:review |                                                                                    |


## config.yaml

### config:set-project

```yaml
include:
  - project: 'common/gitlab-ci-templates'
    ref: <tag or branch name>
    file: 
      - '/devops/gcloud/config.yaml'
```

Usage

```yaml
script:
- !reference [".template:gcloud:config:set-project", "script"]
```

## tag.yaml

### gcloud:tag-image

| VARIABLE                             | DESCRIPTION                                    | DEFAULT VALUE      |
| ------------------------------------ | ---------------------------------------------- | ------------------ |
| KUBERNETES_SERVICE_ACCOUNT_OVERWRITE | The service account used to execute the tag    | \$KUBERNETES_SERVICE_ACCOUNT_GAR  |
| GOOGLE_CLOUD_PROJECT_FROM            | The Google Project ID source image             | \GOOGLE_ARTIFACT_REGISTRY_FROM_PROJECT                |
| GOOGLE_CLOUD_PROJECT_TO              | The Google Project ID destination image        | \GOOGLE_ARTIFACT_REGISTRY_PROJECT                |
| REGISTRY_IMAGE_PATH                  | path in registry                               | apps/${CI_PROJECT_NAME}                   |
| REGISTRY_DOMAIN                      | domain of the registry                         | europe-west1-docker.pkg.dev          |
| IMAGE_NAME                           | name of the image                              |                    |
| IS_MERGE_FF                          | Fostforward merge before tag                   | true               |
| IMAGE_TAG                            | image tag name                                 |                    |

`$IMAGE_TAG` must be set in the project as follow:

```yaml
tag:docker:
  extends: .template:gcloud:tag-image
  variables:
    IMAGE_NAME: image-name
  rules:
    - if: $CI_COMMIT_TAG
      variables:
        IMAGE_TAG: $CI_COMMIT_TAG
```

You can specify environments for the tag job:

Example:

```yaml
tag:docker:digital:
   extends: .tag:docker
   parallel:
     matrix:
       - GOOGLE_ARTIFACT_REGISTRY_TAG_ENV: [preproduction, production]
   environment:
     name: $GOOGLE_ARTIFACT_REGISTRY_TAG_ENV
```

The following variables are set in the Gitlab groups (/core and /web). You don't need to set them in the project:

- `$KUBERNETES_SERVICE_ACCOUNT_GAR`
- `$GOOGLE_ARTIFACT_REGISTRY_PROJECT`
- `$GOOGLE_ARTIFACT_REGISTRY_FROM_PROJECT`

## pubsub.yaml

### gcloud:pubsub:topics:create

Create topics based on `TOPIC_LIST` variable.

| VARIABLE                  | DESCRIPTION                                              | DEFAULT VALUE                                                   |
| ------------------------- | -------------------------------------------------------- | --------------------------------------------------------------- |
| PUBSUB_GCLOUD_CREDENTIALS | Gcloud key authorized to create topics in target project |                                                                 |
| GOOGLE_PROJECT_ID         | The Google Project ID where the build will run           |                                                                 |
| TOPIC_PREFIX              | Prefix topic with a specific string                      |                                                                 |
| PUBLISHER_SERVICE_ACCOUNT | Service account name of app which will publish to topic  |                                                                 |
| LABELS                    | Labels on topic                                          | `owner=$CI_PROJECT_NAME,managed-by=gitlab-ci,entity=digital`    |
| FILTER_TOPIC_EXISTS       | Filter expression when lookup for existing topic         | `labels.managed-by=gitlab-ci AND labels.owner:$CI_PROJECT_NAME` |
| TOPIC_LIST                | List of topic to create. Seperated with space.           |                                                                 |

### gcloud:pubsub:topics:create:review

Create topics on review environment based on `TOPIC_LIST` variable.

It adds review environment name as prefix of each topic id.

| VARIABLE                  | DESCRIPTION                                              | DEFAULT VALUE                                                                                       |
| ------------------------- | -------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| PUBSUB_GCLOUD_CREDENTIALS | Gcloud key authorized to create topics in target project | `$PUBSUB_EDITOR_DIGITAL_STAGING_KEY_SECRET_FILE`                                                    |
| GOOGLE_PROJECT_ID         | The Google Project ID where the build will run           | `$GOOGLE_CLOUD_DIGITAL_STAGING_PROJECT`                                                             |
| TOPIC_PREFIX              | Prefix topic with a specific string                      | `${CI_ENVIRONMENT_SLUG}-`                                                                           |
| PUBLISHER_SERVICE_ACCOUNT | Service account name of app which will publish to topic  |                                                                                                     |
| LABELS                    | Labels on topic                                          | `owner=$CI_PROJECT_NAME,managed-by=gitlab-ci,env=$CI_ENVIRONMENT_SLUG,entity=digital`               |
| FILTER_TOPIC_EXISTS       | Filter expression when lookup for existing topic         | `labels.managed-by=gitlab-ci AND labels.env:$CI_ENVIRONMENT_SLUG AND labels.owner:$CI_PROJECT_NAME` |
| TOPIC_LIST                | List of topic to create. Seperated with space.           |                                                                                                     |

### gcloud:pubsub:subscriptions:create

Create subscriptions based on `SUBSCRIPTION_LIST` variable.

| VARIABLE                   | DESCRIPTION                                                      | DEFAULT VALUE                                                   |
| -------------------------- | ---------------------------------------------------------------- | --------------------------------------------------------------- |
| PUBSUB_GCLOUD_CREDENTIALS  | Gcloud key authorized to create subscriptions in target project  |                                                                 |
| GOOGLE_PROJECT_ID          | The Google Project ID where the build will run                   |                                                                 |
| SUBSCRIPTION_PREFIX        | Prefix subscription with a specific string                       |                                                                 |
| SUBSCRIBER_SERVICE_ACCOUNT | Service account name of app which will subscribe to subscription |                                                                 |
| LABELS                     | Labels on subscription                                           | `owner=$CI_PROJECT_NAME,managed-by=gitlab-ci,entity=digital`    |
| FILTER_SUBSCRIPTION_EXISTS | Filter expression when lookup for existing subscription          | `labels.managed-by=gitlab-ci AND labels.owner:$CI_PROJECT_NAME` |
| SUBSCRIPTION_LIST          | List of subscription to create. Seperated with space.            |                                                                 |
| DEFAULT_TOPIC              | Bind subscription to a default topic                             | `default`                                                       |
| BIND_TO_STAGING_TOPIC      | Bind custom subscription to staging topics                       | `false`                                                         |
| SUBSCRIPTION_EXPIRE_IN     | Fix an expiry time for a subscription                            | `90d`                                                           |
### gcloud:pubsub:subscriptions:create:review

Create subscriptions on review environment based on `SUBSCRIPTION_LIST` variable.

It adds review environment name as prefix of each topic id.

| VARIABLE                   | DESCRIPTION                                                      | DEFAULT VALUE                                                                                       |
| -------------------------- | ---------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| PUBSUB_GCLOUD_CREDENTIALS  | Gcloud key authorized to create subscriptions in target project  | `$PUBSUB_EDITOR_DIGITAL_STAGING_KEY_SECRET_FILE`                                                    |
| GOOGLE_PROJECT_ID          | The Google Project ID where the build will run                   | `$GOOGLE_CLOUD_DIGITAL_STAGING_PROJECT`                                                             |
| SUBSCRIPTION_PREFIX        | Prefix subscription with a specific string                       | `${CI_ENVIRONMENT_SLUG}-`                                                                           |
| SUBSCRIBER_SERVICE_ACCOUNT | Service account name of app which will subscribe to subscription |                                                                                                     |
| LABELS                     | Labels on subscription                                           | `owner=$CI_PROJECT_NAME,managed-by=gitlab-ci,entity=digital`                                        |
| FILTER_SUBSCRIPTION_EXISTS | Filter expression when lookup for existing subscription          | `labels.managed-by=gitlab-ci AND labels.env:$CI_ENVIRONMENT_SLUG AND labels.owner:$CI_PROJECT_NAME` |
| SUBSCRIPTION_LIST          | List of subscription to create. Seperated with space.            |                                                                                                     |
| DEFAULT_TOPIC              | Bind subscription to a default topic                             | `${CI_ENVIRONMENT_SLUG}-default`                                                                    |
| BIND_TO_STAGING_TOPIC      | Bind custom subscription to staging topics                       | `false`                                                                                             |

### gcloud:pubsub:topics:delete

Delete all topics which match `FILTERS`.

| VARIABLE                  | DESCRIPTION                                              | DEFAULT VALUE                 |
| ------------------------- | -------------------------------------------------------- | ----------------------------- |
| PUBSUB_GCLOUD_CREDENTIALS | Gcloud key authorized to delete topics in target project |                               |
| GOOGLE_PROJECT_ID         | The Google Project ID where the build will run           |                               |
| FILTERS                   | Filter expression when lookup for existing topic         | `labels.managed-by=gitlab-ci` |

### gcloud:pubsub:topics:delete:review

Delete all topics which match `FILTERS`.

| VARIABLE                  | DESCRIPTION                                              | DEFAULT VALUE                                                     |
| ------------------------- | -------------------------------------------------------- | ----------------------------------------------------------------- |
| PUBSUB_GCLOUD_CREDENTIALS | Gcloud key authorized to delete topics in target project | `$PUBSUB_EDITOR_DIGITAL_STAGING_KEY_SECRET_FILE`                  |
| GOOGLE_PROJECT_ID         | The Google Project ID where the build will run           | `$GOOGLE_CLOUD_DIGITAL_STAGING_PROJECT`                           |
| FILTERS                   | Filter expression when lookup for existing topic         | `labels.managed-by=gitlab-ci AND labels.env=$CI_ENVIRONMENT_SLUG` |

### gcloud:pubsub:subscriptions:delete

Delete all subscriptions which match `FILTERS`.

| VARIABLE                  | DESCRIPTION                                                     | DEFAULT VALUE                 |
| ------------------------- | --------------------------------------------------------------- | ----------------------------- |
| PUBSUB_GCLOUD_CREDENTIALS | Gcloud key authorized to delete subscriptions in target project |                               |
| GOOGLE_PROJECT_ID         | The Google Project ID where the build will run                  |                               |
| FILTERS                   | Filter expression when lookup for existing subscription         | `labels.managed-by=gitlab-ci` |


### gcloud:pubsub:subscriptions:delete:review

Delete all subscriptions which match `FILTERS`.

| VARIABLE                  | DESCRIPTION                                                     | DEFAULT VALUE                                                     |
| ------------------------- | --------------------------------------------------------------- | ----------------------------------------------------------------- |
| PUBSUB_GCLOUD_CREDENTIALS | Gcloud key authorized to delete subscriptions in target project | `$PUBSUB_EDITOR_DIGITAL_STAGING_KEY_SECRET_FILE`                  |
| GOOGLE_PROJECT_ID         | The Google Project ID where the build will run                  | `$GOOGLE_CLOUD_DIGITAL_STAGING_PROJECT`                           |
| FILTERS                   | Filter expression when lookup for existing subscription         | `labels.managed-by=gitlab-ci AND labels.env=$CI_ENVIRONMENT_SLUG` |
