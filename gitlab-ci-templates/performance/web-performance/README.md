# Helm templates

[[_TOC_]]

## Description

Templates available and description :

| File         | Function                  | Description                            |
| ------------ | ------------------------- | -------------------------------------- |
| browser.yaml | web-performance:sitespeed | perform web performance with sitespeed |

## test

### web-performance:sitespeed

Perform web performance with sitespeed. Based on original template from Gitlab: [Browser-Performance](https://gitlab.com/gitlab-org/gitlab/-/blob/master/lib/gitlab/ci/templates/Verify/Browser-Performance.gitlab-ci.yml)

Variables:

```yaml
SITESPEED_IMAGE: "sitespeedio/sitespeed.io"
SITESPEED_VERSION: "14.1.0"
SITESPEED_OPTIONS: ""
ENV_URL: "" # template helm deploy propagate ENV_URL
CUSTOM_URIS: "" (ex: CUSTOM_URIS:|
                        /test
                        /test2)
# for grafana graphing option
SITESPEED_INFLUXDB_USERNAME: $COMMON_INFLUXDB_USERNAME
SITESPEED_INFLUXDB_PASSWORD: $COMMON_INFLUXDB_PASSWORD
SITESPEED_INFLUXDB_SERVER: $COMMON_INFLUXDB_SERVER
SITESPEED_INFLUXDB_PORT: 80
SITESPEED_INFLUXDB_DB: "perf"
```

## Usage with protected public address

eg : mdm-staging.com, mdm-preprod.com

---

/!\ **Do not change user-agent header**
