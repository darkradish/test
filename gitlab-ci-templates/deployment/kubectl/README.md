# kubectl

All templates are related to kubectl tool

# Available scripts

Scripts has to be run as [`reference`](https://docs.gitlab.com/ee/ci/yaml/#reference-tags) in Gitlab CI jobs. They are not intended to be used as full job.

| File name  | scripts               | Description                                     |
| ---------- | --------------------- | ----------------------------------------------- |
| job.yaml   | kubectl:job:delete    | Delete all kubernetes jobs from deployment      |
| clean.yaml | kubectl:clean:objects | Delete all kubernetes objects from release name |
