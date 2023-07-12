# gitlab api template

[[_TOC_]]

## Description

Templates available and description :

| File               | Function                        | Description                                                              |
| ------------------ | ------------------------------- | ------------------------------------------------------------------------ |
| merge-request.yaml | merge-request:create            | Create merge request                                                     |
|                    | merge-request:create-merge-back | Create merge request which merge master to develop with label MERGE-BACK |
|                    | merge-request:notes             | Create a note in merge request                                           |
| env.yaml           | env:get-url:branch              | get an url from project PROJECT_URL_SRC and branch BRANCH_URL_SRC        |
|                    | env:get-urls:branches           | get an url from projects PROJECTS_URL_SRC and branches BRANCHES_URL_SRC  |
| label.yaml           | label:get-branches                     | function for getting branch labl in labels tag                                                               |
|           | label:get-branches                     | function for getting branch name in labels                                                                |
|          | label:get-urls                     | getting branch url in labels branch name                                                                |
|          | label:get-review-env                     |  set RELEASE_ENV_TYPE with label REVIEW_ENV value                                                               |
## merge-request.yaml

### merge-request:create-merge-back

Create a merge request from `$MERGE_BACK_SOURCE_BRANCH` to `$MERGE_BACK_TARGET_BRANCH` with label `MERGE-BACK`.

Rules are defined by developer in gitlab-ci file of project. In example below, we suggest an usage.

Generally, we want to create a merge request to merge master into develop when pushing/merging changes to master branch.

Default value for variables:

```yaml
SOURCE_BRANCH: master
TARGET_BRANCH: develop
LABELS: "MERGE-BACK" # list of labels comma seperated. if empty, it will not add labels to merge request
DESCRIPTION: "" # allow to override default value which is `This merge request has been generated automatically.`
TITLE: "" # allow to override default value which is `Merge $MERGE_BACK_SOURCE_BRANCH to $MERGE_BACK_TARGET_BRANCH`
```

```yaml
include:
  - project: 'common/gitlab-ci-templates'
    ref: <tag or branch name>
    file:
      - '/gitlab/gitlab-api/merge-request.yaml'
```

Usage

```yaml
git-flow:merge-back:
  extends: .template:gitlab-ci:merge-request:create-merge-back
  stage: Merge Back
  rules:
    - if: $CI_PIPELINE_SOURCE == "push" && $CI_COMMIT_REF_NAME == "master"
```

### merge-request:notes

Create a note in merge request

```yaml
.template:gitlab-api:merge-request:notes:
  stage: .post
  variables:
    MR_MESSAGE: ""
    GITLAB_API_TOKEN: $WEB_DEV_DEVELOPER_ACCESS_TOKEN
```

MR_MESSAGE is the body, it is formatted with br for back to line html.

## env.yaml

get env information from gitlab api

### env:get-url:branch

get environment url from a project name binding with a branch

return var env_url

Branch of target project **must be deployed in review environment**. Otherwise, template will not be able to get url.

Default value for variables:

```yaml
  GITLAB_API_TOKEN: $WEB_DEV_DEVELOPER_ACCESS_TOKEN
  BRANCH_URL_SRC: $CI_COMMIT_REF_NAME
  PROJECT_URL_SRC: ""
  URL_TYPE: "internal" # support internal(http://), service(k8s), domain, other or null(full url)
```

Description of variables

| VARIABLE         | DESCRIPTION                                                                                                                                                                             | DEFAULT VALUE                   |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------- |
| GITLAB_API_TOKEN | Gitlab token to access gitlab api                                                                                                                                                       | $WEB_DEV_DEVELOPER_ACCESS_TOKEN |
| BRANCH_URL_SRC   | Branch name of target project to get url. Branch must be deployed in review environment                                                                                                 | $CI_COMMIT_REF_NAME             |
| PROJECT_URL_SRC  | Path to project in gitlab. For example `web/dev/api/mercure` for mercure project.                                                                                                       |                                 |
| URL_TYPE         | Type of url to get. It can be internal (url defined in istio virtual service), service (url to kubernetes service *.*.svc.cluster.local), domain (host part of url) or other (full url) | internal                        |

```yaml
include:
  - project: 'common/gitlab-ci-templates'
    ref: <tag or branch name>
    file:
      - '/devops/gitlab-ci/merge-request.yaml'
```

### env:get-urls:branches

Loop over list of branches and projects. It uses template `env:get-url:branch`.

Default values for variables:

```yaml
  GITLAB_API_TOKEN: $WEB_DEV_DEVELOPER_ACCESS_TOKEN
  PROJECT_URL_SRC: ""
  URL_TYPE: "internal" # support internal(http://), service(k8s), domain, other or null(full url)
  PROJECTS_URL_SRC: ""
  BRANCHES_URL_SRC: |
      $CI_COMMIT_REF_NAME
      $CI_MERGE_REQUEST_TARGET_BRANCH_NAME
```

Description of variables

| VARIABLE         | DESCRIPTION                                                                                                                                                                             | DEFAULT VALUE                                            |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| GITLAB_API_TOKEN | Gitlab token to access gitlab api                                                                                                                                                       | $WEB_DEV_DEVELOPER_ACCESS_TOKEN                          |
| BRANCHES_URL_SRC | List of branches in target project seperated by space.                                                                                                                                  | $CI_COMMIT_REF_NAME $CI_MERGE_REQUEST_TARGET_BRANCH_NAME |
| PROJECTS_URL_SRC | List of paths to project in gitlab seperated by space. For example `web/dev/api/mercure corvus`.                                                                                        |                                                          |
| URL_TYPE         | Type of url to get. It can be internal (url defined in istio virtual service), service (url to kubernetes service *.*.svc.cluster.local), domain (host part of url) or other (full url) | internal                                                 |

result example :

```bash
urls_list["api/mercure"]="http://mercure.mdm-staging.com"
urls_list["app-shop"]="http://shop.mdm-staging.com"
```

## label.yaml

Get info from Gitlab MR labels
### label:get-branches

Loop over a list of project to extract branch name from Gitlab MR scoped labels.

Default values for variables:

```yaml
  PROJECTS_URL_SRC: ""
```


| Variable         | Description                                                                                                             | Default |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------- | ------- |
| PROJECTS_URL_SRC | Space separated list containing project name. <br> These projects name must preceed branch name in Gitlab scoped labels | <empty> |

### label:get-urls

Use the `env:get-url:branch` before_script. While parsing MR scoped labels, find environnement URLs tied to the branch in the scoped label.
