# test infra template

[[_TOC_]]

## Description

Templates available and description :

| File            | Function          | Description                                                                                                                                                                                                                                                                                            |
|-----------------|-------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| kics.yaml | security:kics           | use kics to scan infrastructure files                                                                                                                                            |

### security:kics

Authenticate docker with a docker key string

parameters :

| VARIABLE   | DESCRIPTION       | DEFAULT VALUE              |
| ---------- | ----------------- | -------------------------- |
| KICS_FAIL_ON | which kind of results should return an exit code ([high,medium,low,info]) | high,medium |
| FOLDER_PATH  | Path of analyze | "." |
| KICS_OPTS    | add opts to kics | "" |
