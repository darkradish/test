# Database

All templates are related to databases (redis...)

# Available scripts

Scripts has to be run as [`reference`](https://docs.gitlab.com/ee/ci/yaml/#reference-tags) in Gitlab CI jobs. They are not intended to be used as full job.

| File name         | scripts               | Description                               |
| ----------------- | --------------------- | ----------------------------------------- |
| redis/switch.yaml | database:redis:switch | Determine which redis db should be used   |
