# Sonarqube template

[[_TOC_]]

## Description

Templates available and description :

| File             |  Fonction           | Description         |
|------------------|---------------------|---------------------|
| scanner.yaml     | code-analyze        | Analyse project code|

## scanner.yaml - Basic Usage

Allow running sonar-scanner on full project with default values 

> NOT RECOMMENDED FOR LARGE PROJECTS (over 300k lines)

```yaml
include:
- project: 'common/gitlab-ci-templates'
    ref: <tag or branch>
    file: 'code/sonarqube/scanner.yaml'

sonarqube:analyze-code:
  stage: Test
  extends: .template:sonarqube:code-analyze
  rules:
    - if: '$CI_COMMIT_BRANCH == "master"'
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'

```

## scanner.yaml - Using `sonar-project.properties` File

Use the exact same syntax as in [basic usage](https://git.maisonsdumonde.net/common/gitlab-ci-templates/-/tree/master/code/sonarqube#scanneryaml-basic-usage) chapter for the `.gitlab-ci.yml` but put a `sonar-project.properties` file at the project's root. It allow running sonar-scanner with custom parameters.

For further information about how to construct this file, have a look [HERE](https://docs.sonarqube.org/latest/analysis/analysis-parameters/)

## Default values

| VARIABLE          | DESCRIPTION | DEFAULT VALUE
| ---               | ---   |---
| SONARQUBE_URL    | Sonarqube URL | https://sonarqube.maisonsdumonde.net
| CI_PROJECT_NAME  | Project name where CI runs | CI GENERATED
| CI_PROJECT_DIR   | Dir where project is cloned - also default path to analysis if `sonar.sources` is not specified in config file | CI GENERATED
| SONARQUBE_TOKEN  | Sonarqube token used to authenticate against Sonarqube backend | Set in gitlab CI/CD vars
| SEVERITY | Verbosity level of sonar-scanner job | INFO
| GIT_DEPTH | Tells git to fetch all the branches of the project, required by the analysis task | 0
| SONARQUBE_FLAG | Allow CORE to know which entity manage this project | Set in gitlab CI/CD vars
| SONARQUBE_EXTRA_OPTS | Sonarqube custom opts, not in sonar-project.properties, eg: -Dsonar.sources="xxx" | ""
