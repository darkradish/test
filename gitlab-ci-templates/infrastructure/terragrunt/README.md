# Terragrunt template

[[_TOC_]]

## Description

Templates available and description :

| File            | Function                  | Description                                                |
| --------------- | ------------------------- | ---------------------------------------------------------- |
| compliance.yaml | terragrunt:compliance:fmt | Check if all terragrunt HCL files are correctly formatted. |
| run.yaml        | terragrunt:run:show-plan  | Execute terragrunt run-all plan and produce a report.      |
| run.yaml        | terragrunt:run:apply      | Execute terragrunt run-all apply.                          |
| audit.yaml      | terragrunt:audit          | Execute an andit on the Terraform code.                    |


## Prerequisite

To use this template you must provide these variables:

```yaml
variables:
  TERRAFORM_VERSION: <terraform_version> # 0.13.5 | 0.14.11 | 1.0.10
```

## compliance.yaml

```yaml
include:
  - project: 'common/gitlab-ci-templates'
    ref: <tag or branch name>
    file: 
      - /infrastructure/terragrunt/compliance.yaml
```

### terragrunt:compliance:fmt

Check if all terragrunt HCL files are correctly formatted.

```yaml
terragrunt:fmt:
  stage: test
  extends: .template:terragrunt:compliance:fmt
```

## run.yaml

```yaml
include:
  - project: 'common/gitlab-ci-templates'
    ref: <tag or branch name>
    file: 
      - /infrastructure/terragrunt/run.yaml
```


### terragrunt:run:show-plan

```yaml
terragrunt:show-plan:
  stage: test
  extends: .template:terragrunt:run:show-plan
```

### terragrunt:run:apply

```yaml
terragrunt:plan:
  stage: deploy
  extends: .template:terragrunt:run:apply
```
> note that it needs the plan from the plan job

## audit.yaml

```yaml
include:
  - project: 'common/gitlab-ci-templates'
    ref: <tag or branch name>
    file: 
      - /infrastructure/terragrunt/audit.yaml
```

### terragrunt:run:audit

```yaml
terragrunt:audit:
  stage: audit
  extends: .template:terragrunt:audit
```

## Complete example pipeline

### Structure of the repository

```
.
├── live
│   ├── pp
│   │   ├── infrastructure               # contains the infrastructure root Terraform modules 
│   │   └── projects                     # contains the projects root Terraform modules
│   │   └── terragrunt.hcl
│   ├── staging
│   │   ├── infrastructure               # contains the infrastructure root Terraform modules 
│   │   └── projects                     # contains the projects root Terraform modules
│   │   └── terragrunt.hcl
├── modules
│   ├── infrastructure                  # contains the child Terraform modules sourced by the root modules 
│   ├── projects                        # contains the child Terraform modules sourced by the root modules 
├── pipeline.yml.j2
└── .gitlab-ci.yml

```
### CICD pipeline

This pipeline performs *validation*, *plan* and *audit* during a Merge Request.

It uses the [parent-child pipelines](https://docs.gitlab.com/ee/ci/pipelines/parent_child_pipelines.html) feature of Gitlab.
That feature allows to create a pipeline for each Root Terraform modules in the repository.

```yaml
# .gitlab-ci.yml
workflow:
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH || $CI_PIPELINE_SOURCE == "merge_request_event"

stages:
  - generate
  - trigger
  - deploy

generator:
  stage: generate
  image: python:3-bullseye
  parallel:
    matrix:
      - MODULE_TYPE: [projects, infrastructure]
        MODULE_ENV_PATH: [staging]
        MODULE_ENV_NAME: [staging]
      - MODULE_TYPE: [projects, infrastructure]
        MODULE_ENV_PATH: [pp]
        MODULE_ENV_NAME: [preproduction]
      - MODULE_TYPE: [projects, infrastructure]
        MODULE_ENV_PATH: [prod]
        MODULE_ENV_NAME: [production]
  before_script:
    - pip install j2cli[yaml] yamllint
  script:
    - list_modules=$(find live/${MODULE_ENV_PATH}/${MODULE_TYPE}/ -maxdepth 1 -mindepth 1 -type d | xargs -n1 basename)
    - >
      path=${MODULE_ENV_PATH}
      environment=${MODULE_ENV_NAME}
      type=${MODULE_TYPE}
      modules=${list_modules}
      j2 pipeline.yml.j2 > generated-pipeline.yml
    - >
      yamllint \
        --config-data "{extends: default, rules: {line-length: {max: 160}}}" \
        --strict \
        generated-pipeline.yml
  artifacts:
    paths:
      - generated-pipeline.yml
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH || $CI_PIPELINE_SOURCE == "merge_request_event"
      when: always

.templates: &templates
  project: common/gitlab-ci-templates
  ref: 1.25.0
  file:
    - /infrastructure/terragrunt/compliance.yaml
    - /infrastructure/terraform/compliance.yaml
    - /infrastructure/terragrunt/audit.yaml
    - /infrastructure/terragrunt/run.yaml
    - /infrastructure/security/kics.yaml

staging:
  stage: trigger
  trigger:
    include:
      - <<: *templates
      - artifact: generated-pipeline.yml
        job: "generator: [projects, staging, staging]"
      - artifact: generated-pipeline.yml
        job: "generator: [infrastructure, staging, staging]"
    strategy: depend
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH || $CI_PIPELINE_SOURCE == "merge_request_event"
      changes:
        - "live/staging/**/*"
        - "modules/**/*"
      when: always

pp:
  stage: trigger
  trigger:
    include:
      - <<: *templates
      - artifact: generated-pipeline.yml
        job: "generator: [projects, pp, preproduction]"
      - artifact: generated-pipeline.yml
        job: "generator: [infrastructure, pp, preproduction]"
    strategy: depend
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH || $CI_PIPELINE_SOURCE == "merge_request_event"
      changes:
        - "live/pp/**/*"
        - "modules/**/*"
      when: always

prod:
  stage: trigger
  trigger:
    include:
      - <<: *templates
      - artifact: generated-pipeline.yml
        job: "generator: [projects, prod, production]"
      - artifact: generated-pipeline.yml
        job: "generator: [infrastructure, prod, production]"
    strategy: depend
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH || $CI_PIPELINE_SOURCE == "merge_request_event"
      changes:
        - "live/prod/**/*"
        - "modules/**/*"
      when: always
```

file : `pipeline.yml.j2`

```jinja2
---
variables:
  TF_VAR_gitlab_token: $GITLAB_TERRAFORM_PAT # Personal access token of [Web Maintener SA](https://git.maisonsdumonde.net/webmaintainer), necessary if you use the Terraform Gitlab Provider, cannot be the $CI_JOB_TOKEN
  GOOGLE_PROJECT_ID: $GOOGLE_CLOUD_DIGITAL_{{ environment|upper }}_PROJECT
  KUBERNETES_SERVICE_ACCOUNT_OVERWRITE: ci-tf-digital-{{ environment }}-sa
  ENV: {{ environment }}

.rule-exclude-prod: &rule_exclude_prod
  if: $ENV == "production"
  when: never

stages:
  - Validate
  - Fmt
  - Plan
  - Audit
  - Security
  - Deploy

.tg:validate:
  stage: Validate
  extends: .template:terragrunt:compliance:validate
  tags:
    - "terragrunt"

.tf:fmt:
  stage: Fmt
  extends: .template:terraform:compliance:fmt
  tags:
    - "terraform"

.tg:fmt:
  stage: Fmt
  extends: .template:terragrunt:compliance:fmt
  tags:
    - "terragrunt"

.tg:show-plan:
  stage: Plan
  extends: .template:terragrunt:run:show-plan
  tags:
    - "terragrunt"

.tg:audit:
  stage: Audit
  extends: .template:terragrunt:audit
  tags:
    - "terragrunt"
  variables:
    OWNER: infrastructure
    ENTITY: digit
    ENVIRONMENT: {{ environment }}

.tg:apply:
  stage: Deploy
  extends: .template:terragrunt:run:apply
  tags:
    - "terragrunt"

{% set list = modules.split('\n') -%}
{% for module in list -%}

.changes-{{ module }}: &changes_{{ module }}
  changes:
    - live/{{ path }}/{{ type }}/{{ module }}/**/*
    - modules/{{ type }}/{{ module }}/**/*
  when: always

.rules-{{ module }}-merge: &rules_{{ module }}_merge
  if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
  <<: *changes_{{ module }}

.rules-{{ module }}-review: &rules_{{ module }}_review
  if: $CI_MERGE_REQUEST_ID
  <<: *changes_{{ module }}

{{ module }}:validate:{{ environment }}:
  extends: .tg:validate
  variables:
    TF_ROOT: ${CI_PROJECT_DIR}/live/{{ path }}/{{ type }}/{{ module }}
  rules:
    - <<: *rule_exclude_prod
    - <<: *rules_{{ module }}_review
    - <<: *rules_{{ module }}_merge

{{ module }}:hclfmt:{{ environment }}:
  extends: .tg:fmt
  variables:
    TF_ROOT: ${CI_PROJECT_DIR}/live/{{ path }}/{{ type }}/{{ module }}
  rules:
    - <<: *rules_{{ module }}_review
    - <<: *rules_{{ module }}_merge

{{ module }}:fmt:{{ environment }}:
  extends: .tf:fmt
  variables:
    TF_ROOT_FILE_PATH: ${CI_PROJECT_DIR}/modules/{{ type }}/{{ module }}
  rules:
    - <<: *rules_{{ module }}_review
    - <<: *rules_{{ module }}_merge

{{ module }}:plan:{{ environment }}:
  extends: .tg:show-plan
  variables:
    TF_ROOT: ${CI_PROJECT_DIR}/live/{{ path }}/{{ type }}/{{ module }}
  needs:
    - "{{ module }}:validate:{{ environment }}"
  environment:
    name: plan-{{ environment }}
  rules:
    - <<: *rule_exclude_prod
    - <<: *rules_{{ module }}_review
    - <<: *rules_{{ module }}_merge

{{ module }}:deploy:{{ environment }}:
  extends: .tg:apply
  variables:
    TF_ROOT: ${CI_PROJECT_DIR}/live/{{ path }}/{{ type }}/{{ module }}
  needs:
    - "{{ module }}:plan:{{ environment }}"
  environment:
    name: deploy-{{ environment }}
  rules:
    - <<: *rule_exclude_prod
    - <<: *rules_{{ module }}_merge
      when: manual

.audit:{{ module }}:{{ environment }}:
  extends: .tg:audit
  variables:
    IDENTIFIER: {{ module }}
    TF_ROOT: ${CI_PROJECT_DIR}/live/{{ path }}/{{ type }}/{{ module }}
  needs:
    - "{{ module }}:plan:{{ environment }}"
  rules:
    - <<: *rule_exclude_prod
    - <<: *rules_{{ module }}_review

audit-tech-debt:{{ module }}:{{ environment }}:
  extends: .audit:{{ module }}:{{ environment }}
  variables:
    AUDIT_MODE: FULL
  allow_failure: true

audit-regression:{{ module }}:{{ environment }}:
  extends: .audit:{{ module }}:{{ environment }}
  variables:
    AUDIT_MODE: DIFF

{{ module }}:kics:{{ environment }}:
  stage: Security
  extends: .template:security:kics
  variables:
    FOLDER_PATH: ${CI_PROJECT_DIR}/live/{{ path }}/{{ type }}/{{ module }}
  tags:
    - "medium"
  allow_failure: true
  artifacts:
    reports:
      sast: ${CI_PROJECT_DIR}/live/{{ path }}/{{ type }}/{{ module }}/results/gl-sast-results.json
  rules:
    - <<: *rules_{{ module }}_review
{% endfor -%}
```
