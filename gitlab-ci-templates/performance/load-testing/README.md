# Helm templates

[[_TOC_]]

## Description

Templates available and description :

| File    | Function        | Description                                 |
| ------- | --------------- | ------------------------------------------- |
| k6.yaml | load-testing:k6 | perform load performances test with k6 tool |

## test

### load-testing:k6

Perform load testing with k6. Based on original template from Gitlab: [Load-Performance-Testing.gitlab-ci.yml](https://gitlab.com/gitlab-org/gitlab/-/blob/master/lib/gitlab/ci/templates/Jobs/Load-Performance-Testing.gitlab-ci.yml)

Variables:

```yaml
K6_TEST_FILE: "" # JS file which contains k6 tests
K6_OPTIONS: "" # Arg to pass to k6 such as environment variables
# influx configuration for grafana dashboard
K6_INFLUXDB_USERNAME: $COMMON_INFLUXDB_USERNAME 
K6_INFLUXDB_PASSWORD: $COMMON_INFLUXDB_PASSWORD
K6_INFLUXDB_SERVER: http://$COMMON_INFLUXDB_SERVER
K6_INFLUXDB_DB: "perf"
K6_INFLUXDB_INSECURE: "true"
```

Usage:

```yaml
load-testing:k6:
  stage: Load Testing
  extends: .template:load-testing:k6
  variables:
    K6_TEST_FILE: scenario.js
    SCENARIO: large_scenario
    K6_OPTIONS: -e=ENV_URL=$ENV_URL -e=SCENARIO=$SCENARIO
    
```

> NOTE: If you need the URL of the depoyed env, use $ENV_URL. This variable is available after a successful deployment.

## Usage with protected public address

eg : mdm-staging.com, mdm-preprod.com

---

 /!\ **Do not change user-agent header**

---

add constant in your k6 config :

```javascript
const requestHeaders = {
  'mdm-gate' : __ENV.MDM_GATE_HASH
}
```

and pass it to request

More inforamation : <https://k6.io/docs/javascript-api/k6-http/params/#example-of-custom-http-headers-and-tags>
