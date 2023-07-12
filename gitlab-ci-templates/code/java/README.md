# Java templates

[[_TOC_]]

## Description

Templates available and description :

| File             |  Function           | Description         |
|------------------|---------------------|---------------------|
| build.yaml       | build:maven         | use maven to build java |
| test.yaml        | test:maven          | maven test phase |

## build.yaml

### compile:maven

Run maven compile (test-compile phase).

parameters :

| VARIABLE | DESCRIPTION | DEFAULT VALUE |
|----------|-------------|---------------|
|MAVEN_IMAGE_TAG|Maven Docker image tag used to build|3.8-openjdk-17-slim|
|MAVEN_OPTS|This variable contains parameters used to start up the JVM running Maven and can be used to supply additional options to it. E.g. Repository Maven could be defined with the value -Dmaven.repo.local=.m2| |

Example usage :

```yaml
build:
    extends: .template:java:build:maven
    stage: Build
```

## test.yaml

### test:maven:jacoco

Run maven tests (test phase), with JaCoCo report output.

parameters :

| VARIABLE | DESCRIPTION | DEFAULT VALUE |
|----------|-------------|---------------|
|MAVEN_IMAGE_TAG|Maven Docker image tag used to run tests|3.8-openjdk-17-slim|

Example usage :

```yaml
build:
    extends: .template:java:test:maven:jacoco
    stage: Test
```
## package.yaml

### spring-boot-package:maven

The Spring Boot Maven Plugin provides Spring Boot support in Maven, letting you package executable jar archives.

parameters :

| VARIABLE | DESCRIPTION | DEFAULT VALUE |
|----------|-------------|---------------|
|MAVEN_IMAGE_TAG|Maven Docker image tag used to build|3.8-openjdk-17-slim|
|MAVEN_OPTS|This variable contains parameters used to start up the JVM running Maven and can be used to supply additional options to it. E.g. Repository Maven could be defined with the value -Dmaven.repo.local=.m2| |


Example usage :

```yaml
package:
    extends: .template:java:spring-boot-package:maven
    stage: Package
```
