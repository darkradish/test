# gitlab-ci template

[[_TOC_]]

## Description

Templates available and description :

| File      | Function | Description             |
|-----------|----------|-------------------------|
| lint.yaml | yamllint | A linter for YAML files |


## lint.yaml

### yamllint

https://github.com/adrienverge/yamllint

```yaml
include:
  - project: 'common/gitlab-ci-templates'
    ref: <tag or branch name>
    file: 
      - '/devops/gitlab-ci/lint.yaml'
```

Usage

```yaml
test:lint:yaml:
  extends: .template:gitlab-ci:yamllint
  stage: Test
```

## merge-request.yaml

### merge-request:create

Create a merge request. 

Default value for variables:

```yaml
SOURCE_BRANCH: master
TARGET_BRANCH: develop
LABELS: "" # list of labels comma seperated. if empty, it will not add labels to merge request
DESCRIPTION: "" # allow to override default value which it is `This merge request has been generated automatically.`
TITLE: "" # allow to override default value which it is `Merge $MERGE_BACK_SOURCE_BRANCH to $MERGE_BACK_TARGET_BRANCH`
```

```yaml
include:
  - project: 'common/gitlab-ci-templates'
    ref: <tag or branch name>
    file: 
      - '/devops/gitlab-ci/merge-request.yaml'
```

Usage

```yaml
create-merge-request:
  extends: .template:gitlab-ci:merge-request:create
```
