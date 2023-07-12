# DNS template

[[_TOC_]]

## Description

Templates available and description :

| File             |  Function           | Description         |
|------------------|---------------------|---------------------|
|ns1.yaml  | add:cname                   | add CNAME in domain |
|         | add:cname:digital:review | add CNAME in domain for digital review env|
|         | delete:cname | delete CNAME in domain |
|         | delete:cname:digital:review | delete CNAME in domain for digital review env|


## NS1

This template communicates with NS1 dns provider


### Prerequisite

To use this template you must provide these variables:

```yaml
variables:
    NS1_API_KEY : #the api key with CNAME PUT and DELETE auth
```

### add cname

Variables:

```yaml
variables:
    NS1_ZONE: # DNS zone
    NS1_DOMAIN: # Domain to be added
    NS1_ANSWER: # CNAME response
```

### delete cname

Variables:

```yaml
variables:
    NS1_ZONE: # DNS zone
    NS1_DOMAIN: # Domain to be added
```
