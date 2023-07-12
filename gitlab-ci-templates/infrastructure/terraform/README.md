# Terraform template

[[_TOC_]]

## Description

Templates available and description :

| File            | Function                            | Description                                                                                                                                                                                                                                                                                            |
| --------------- | ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| compliance.yaml | terraform:compliance:fmt            | Check all terraform files are correctly formatted.                                                                                                                                                                                                                                                     |
|                 | terraform:compliance:best-practices | All terraform files must be in lowercase and use underscore as word separator.                                                                                                                                                                                                                         |
|                 | terraform:compliance:validate       | Validate runs checks that verify whether a configuration is syntactically valid and internally consistent, regardless of any provided variables or existing state. It is thus primarily useful for general verification of reusable modules, including correctness of attribute names and value types. |
|                 | terraform:compliance:lint           | Find possible errors (like illegal instance types) for Major Cloud providers (AWS/Azure/GCP). |
|                 | terraform:compliance:test           | (only for reusable module) Test the module using https://www.terraform.io/language/modules/testing-experiment |
| run.yaml        | terraform:run:plan                  | Perform terraform plan.                                                                                                                                                                                                                                                |
|                 | terraform:run:show-plan             | Perform terraform plan and save plan into json for audit purpose.                                                                                                                                                                                                                 |
| audit.yaml      | terraform:audit                     | Perform an audit with confest. Needs terraform:run:show-plan.                                                                                                                                                                                                                                               |
| registry.yaml   | terraform:registry:upload           | Publish Terraform module into the MdM Gitlab Registry

## Prerequisite

To use this template you must provide these variables:

```yaml
variables:
  TERRAFORM_VERSION: <terraform_version> # 0.14.11 | 1.1.3
  TERRAFORM_GCP_SA_KEY: $TERRAFORM_KEY_FILE # This is a Gitlab CI/CD variable which contains service account key
  GOOGLE_PROJECT_ID: <gcp_project_id>
```

2 Terraform versions are availables:
- 0.14.11
- 1.1.5

## compliance.yaml

```yaml
include:
  - project: 'common/gitlab-ci-templates'
    ref: <tag or branch name>
    file:
      - /infrastructure/terraform/compliance.yaml
```

### terraform:compliance:fmt

Check if your terraform files are well formatted

```yaml
terraform:fmt:
  stage: test
  extends: template:terraform:compliance:fmt
  variables:
    TF_ROOT_FILE_PATH: <root path of your terraform files if different than ./>
```
### terraform:compliance:best-practices

Check if your terraform files respect best-practices

```yaml
terraform:best-practices:
  stage: test
  extends: .template:terraform:compliance:best-practices
  variables:
    TF_ROOT_FILE_PATH: <root path of your terraform files if different than ./>
```

### terraform:compliance:validate

Validate the configuration files in a directory, referring only to the
configuration and not accessing any remote services such as remote state,
provider APIs, etc.
Validate runs checks that verify whether a configuration is syntactically
valid and internally consistent, regardless of any provided variables or
existing state. It is thus primarily useful for general verification of
reusable modules, including correctness of attribute names and value types.

Note: You must create a GCP Service Account in your GCP Project and give it `roles/storage.objectViewer` to `mdm-tfstate` project

```yaml
terraform:validate:
  stage: test
  extends: .template:terraform:compliance:validate
```

### terraform:compliance:lint

Find possible errors (like illegal instance types) for Major Cloud providers (AWS/Azure/GCP).
Warn about deprecated syntax, unused declarations.
Enforce best practices, naming conventions.

```yaml
terraform:lint:
  stage: lint
  extends: .template:terraform:compliance:lint
```

**Links**:

- Tflint: https://github.com/terraform-linters/tflint

### terraform:compliance:test

Test your reusable module

```yaml
terraform:test:
  stage: test
  extends: .template:terraform:compliance:test
```

**Links**:

- https://www.terraform.io/language/modules/testing-experiment

## terraform:registry:upload

Upload the module into the Gitlab Infrastructure Registry of the repository.

```yaml
terraform:registry:upload:
  stage: upload
  extends: .template:terraform:registry:upload
```

## terraform:run

### terraform:run:plan

```yaml
terraform:plan:
  stage: Plan
  extends: .template:terraform:run:plan
```

### terraform:run:show-plan

```yaml
tf:show-plan:
  stage: Plan
  extends: .template:terraform:run:show-plan
```

## terraform:audit

The audit is performed by [Confest](https://www.conftest.dev/).

Hereafter an exemple of use in a MdM context.

```yaml
tf:show-plan:
  stage: Plan
  extends: .template:terraform:run:show-plan

.tf:audit:
  stage: Audit
  extends: .template:terraform:audit
  variables:
    OWNER: infrastructure #  adapt according to your context
    ENTITY: common # adapt according to your context
    ENVIRONMENT: common # staging|preproduction|production
  needs:
    - "tf:show-plan"

tf:audit:debt:
  extends: .tf:audit
  variables:
    AUDIT_MODE: FULL
  allow_failure: true

tf:audit:regression:
  extends: .tf:audit
  variables:
    AUDIT_MODE: DIFF
```

**Links**:

- MdM OPA rules: https://git.maisonsdumonde.net/common/opa-rules
- Conftest: https://www.conftest.dev/

## Full example

```
variables:
  TERRAFORM_VERSION: 1.0.10
  TERRAFORM_GCP_SA_KEY: $TERRAFORM_KEY_FILE
  GOOGLE_PROJECT_ID: mdm-hub-network

include:
  - project: 'common/gitlab-ci-templates'
    ref: 1.21.0
    file:
      - /infrastructure/terraform/compliance.yaml
      - /infrastructure/terraform/registry.yaml
      - /infrastructure/terraform/run.yaml
      - /infrastructure/terraform/audit.yaml
      - /infrastructure/security/kics.yaml

stages:
  - Security
  - Test
  - Plan
  - Audit
  - Upload
  - Apply

workflow:
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_REF_NAME == "master"'
    - if: "$CI_COMMIT_TAG"

.if-review: &if_review
  if: '$CI_PIPELINE_SOURCE == "merge_request_event"'

.if-master: &if_master
  if: '$CI_COMMIT_REF_NAME == "master"'

.if-tag: &if_tag
  if: "$CI_COMMIT_TAG"

tf:fmt:
  stage: Test
  extends: .template:terraform:compliance:fmt
  rules:
    - <<: *if_review

tf:best-practices:
  stage: Test
  extends: .template:terraform:compliance:best-practices
  rules:
    - <<: *if_review

tf:validate:
  stage: Test
  extends: .template:terraform:compliance:validate
  rules:
    - <<: *if_review

tf:lint:
  stage: Test
  extends: .template:terraform:compliance:lint
  rules:
    - <<: *if_review

tf:plan:
  stage: Plan
  extends: .template:terraform:run:plan
  environment:
    name: plan-common
  rules:
    - <<: *if_master

tf:show-plan:
  stage: Plan
  extends: .template:terraform:run:show-plan
  environment:
    name: plan-common
  rules:
    - <<: *if_review

.tf:audit:
  stage: Audit
  extends: .template:terraform:audit
  variables:
    OWNER: infrastructure
    ENTITY: common
    ENVIRONMENT: common
  rules:
    - <<: *if_review
  needs:
    - "tf:show-plan"

tf:audit:debt:
  extends: .tf:audit
  variables:
    AUDIT_MODE: FULL
  allow_failure: true

tf:audit:regression:
  extends: .tf:audit
  variables:
    AUDIT_MODE: DIFF

tf:security:kics:
  stage: Security
  extends: .template:security:kics
  tags:
    - medium
  allow_failure: true
  rules:
    - <<: *if_review

tf:upload:
  extends: .template:terraform:registry:upload
  stage: Upload
  rules:
    - <<: *if_tag

tf:apply:
  extends: .template:terraform:run:apply
  stage: Apply
  rules:
    - <<: *if_tag
```
