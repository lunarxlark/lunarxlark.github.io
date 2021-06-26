+++
title = "ElasticsearchのIndexをAPIで操作"
date = 2019-01-14
tags = ["ElasticSearch"]
draft = false
+++


## GET Indecies

```bash
curl -X GET http://<es-url>/<index-name>-* | \
  jq -r ".[].settings.index.provided_name" | sort
authapi_access_log-2018.10.31
authapi_access_log-2018.11.01
authapi_access_log-2018.11.05
```

## DELETE indecies

```bash
curl -X DELETE http://<es-url>/<index-name>-2018.11.06
```
