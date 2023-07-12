# rundecl

All templates are related to kubectl tool

# Available scripts

Scripts has to be run as [`reference`](https://docs.gitlab.com/ee/ci/yaml/#reference-tags) in Gitlab CI jobs. They are not intended to be used as full job.

| File name   | scripts        | Description                                                                           |
| ----------- | -------------- | ------------------------------------------------------------------------------------- |
| export.yaml | rundeck:export | Export Kubernetes cronjobs defined in `chart/templates/cronjob` by default to Rundeck |
