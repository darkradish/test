## notification

These templates are meant to be referenced by existing scripts in gitlab CI and they cannot be used alone.

In all cases of notification you have to create a file /tmp/isJobSuccessful to not send the critical alerts.yaml.
If you don't do it the notification will be sent.

You have to set action and env variables togive information about which script sent the alert.

before using it you have to include the files :


```yaml
include:
  project: common/gitlab-ci-templates
  ref: <TAG-number>
  file:
    - /infrastructure/notification/pagerduty.yaml
```
### notification:pager-duty

```yaml
sample_not_send_alert:
  script:
    - touch /tmp/isJobSuccessful
  after_script:
  - action="test action"
  - env="environment"
  - summary="summary of action"
  - source="pipeline name"
  - component="component name"
  - severity="critical"
  - if [ -e /tmp/isJobSuccessful ] then exit 0 fi
  - !reference [".template:notification:pager-duty", "script"]
```

```yaml
sample_send_alert:
  after_script:
    - action="test action"
    - env="environment"
    - summary="summary of action"
    - source="pipeline name"
    - component="component name"
    - severity="critical"
    - if [ -e /tmp/isJobSuccessful ] then exit 0 fi
    - !reference [".template:notification:pager-duty", "script"]
```