# 6. Log Management
Ref: https://docs.datadoghq.com/logs/


You can collect, analyze, and explore logs from everywhere (service, container, cloud) and can view them in one dashboard.

Also, you can __correlate__ logs with metrics (CPU, memory, disk, network etc) and tracing. (AWS Cloud Watch Logs and Metrics can __NOT__ correlate each other)

![alt text](../imgs/correlate_metrics_to_logs.gif "")




## 6.1 Enable Kubernetes Log Collection
Refs: 
- https://github.com/DataDog/helm-charts/tree/master/charts/datadog#enabling-log-collection
- https://docs.datadoghq.com/agent/kubernetes/log/?tab=helm

The Datadog Agent can collect logs directly from __container stdout/stderr__ without using a logging driver. 

Datadog Agent deployed by Helm chart can collect logs from containers, and config can be set in yaml.

![alt text](../imgs/logs_get_started_eks.png "")

```yaml
datadog:
  ## @param logs - object - required
  ## Enable logs agent and provide custom configs
  #
  logs:
    ## @param enabled - boolean - optional - default: false
    ## Enables this to activate Datadog Agent log collection.
    #
    enabled: true

    ## @param containerCollectAll - boolean - optional - default: false
    ## Enable this to allow log collection for all containers.
    #
    containerCollectAll: true
```

Upgrade the chart
```sh
helm upgrade datadog \
  -f overrides.yaml \
  datadog/datadog \
  --set datadog.apiKey=YOUR_API_KEY
```

![alt text](../imgs/logs_get_started_eks2.png "")


Starting with Agent v6.19+/v7.19+, __HTTPS transport__ is the default transport used. 



## 6.2 Filter, Aggregate, Visualize Logs

You can filter by K8s service name, name space, pod name, status, etc from left pane (facets panel, more on later).

Logs Live Tail:
![alt text](../imgs/logs.png "")

Log Side Panel show details, correlation with infra/container/metrics:
![alt text](../imgs/logs_side_panel.gif "")


![alt text](../imgs/logs_pattern.png "")



### NOTE: Querying requires from the beginning of words after space or comma

Based on this, you want to search logs with `Incrementing metric` but want to exclude `acceptance-test`
```
Incrementing metric notification.send with meta notification:type:push,notification:api:acceptance-test
```

Then you need to use this query:
```sh
# 1. for more than one word, use double qoute to wrap
# 2. no space after NOT expresison '-'
# 3. need to query the whole word from the beginning
# 4. escape special character ":" with \:
"Incrementing metric" AND -notification\:api\:acceptance-test
```



## 6.3 Scrub Sensitive Info in Logs before Pushed to Datadog

Ref: https://docs.datadoghq.com/agent/logs/advanced_log_collection/?tab=kubernetes#scrub-sensitive-data-from-your-logs


You can use regex to find patterns (email, credit card, social security #, etc) and srub them from logs.

You specify Datadog log config as k8s annotations at `spec.template.medatada.annotations`:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: cardpayment
spec:
  selector:
    matchLabels:
      app: cardpayment
  template:
    metadata:
      annotations: # <----- spec.template.medatada.annotations
        ad.datadoghq.com/cardpayment.logs: >-
          [{
            "source": "java",
            "service": "cardpayment",
            "log_processing_rules": [{
              "type": "mask_sequences",
              "name": "mask_credit_cards",
              "replace_placeholder": "[masked_credit_card]",
              "pattern" : "(?:4[0-9]{12}(?:[0-9]{3})?|[25][1-7][0-9]{14}|6(?:011|5[0-9][0-9])[0-9]{12}|3[47][0-9]{13}|3(?:0[0-5]|[68][0-9])[0-9]{11}|(?:2131|1800|35\\d{3})\\d{11})"   // <---- Regex pattern
            }]
          }]          
      labels:
        app: cardpayment
      name: cardpayment
    spec:
      containers:
        - name: cardpayment
          image: cardpayment:latest
```



## 6.4 Multi-line Aggregation to Reduce Log Records and Save Cost

Ref: https://docs.datadoghq.com/agent/logs/advanced_log_collection/?tab=kubernetes#multi-line-aggregation

Given that java log entry begins with timestamp like below
```log
2018-01-03T09:24:24.983Z UTC Exception in thread "main" java.lang.NullPointerException
        at com.example.myproject.Book.getTitle(Book.java:16)
        at com.example.myproject.Author.getBookTitles(Author.java:25)
        at com.example.myproject.Bootstrap.main(Bootstrap.java:14)
2018-01-03T09:26:24.365Z UTC starting upload of /my/file.gz
```

you can use below log confing to detect and group them
```yaml
apiVersion: apps/v1
kind: Deploymet
metadata:
  name: postgres
spec:
  template:
    metadata:
      annotations:
        ad.datadoghq.com/postgres.logs: >-
          [{
            "source": "postgresql",
            "service": "database",
            "log_processing_rules": [{
              "type": "multi_line",    // <--------- specify type
              "name": "log_start_with_date",
              "pattern" : "\\d{4}-(0?[1-9]|1[012])-(0?[1-9]|[12][0-9]|3[01])"
            }]
          }]     
```


More commonly used log processing rules can be found here: https://docs.datadoghq.com/agent/faq/commonly-used-log-processing-rules/


## 6.5  Exclude Namespaces and Images from Log Collection

Refer to ch2.5 (Best Practice) How to Exclude Containers from Logs & Metrics Collection


## 6.6 Correlate Logs with Metrics (Navigate seamlessly)
Ref: https://docs.datadoghq.com/logs/guide/correlate-logs-with-metrics/

Log Explorer > Click Logs under Content Column > Click Metrics tab 

![alt text](../imgs/log-explorer-metrics-tab.png "")


Dashboard > Graph > click > View Related Logs
![alt text](../imgs/dashboard_to_logs.gif "")




## 6.7 Configure Logs Pipeline

### 6.7.1 What is Log Pipeline
Ref: https://docs.datadoghq.com/logs/log_configuration/pipelines/?tab=source#overview


With __pipelines__, logs are __parsed and enriched__ by chaining them sequentially through processors. This __extracts__ meaningful __attributes__ from semi-structured text to reuse as __facet__. 

![alt text](../imgs/logs_pipeline.png "")


__Preprocessing of JSON logs__ occurs before logs enter pipeline processing. Preprocessing runs a series of operations based on reserved attributes, such as timestamp, status, host, service, and message.

Here is a log processed and status mapped to DD status:
![alt text](../imgs/log_post_processing.png "")




### 6.7.2 Log Integration Pipeline Library
Ref: https://docs.datadoghq.com/logs/log_configuration/pipelines/?tab=source#integration-pipelines

![alt text](../imgs/logs_integration_pipeline_library1.png "")

![alt text](../imgs/logs_integration_pipeline_library2.png "")




## 6.8 Index Logs
Ref: https://docs.datadoghq.com/logs/log_configuration/indexes

### 6.6.1 What is Log Index

> Log Indexes provide fine-grained control over your Log Management budget by allowing you to segment data into value groups for differing retention, quotas, usage monitoring, and billing


Index config is about how long to retain what kind of logs based on tags and attributes (i.e. you want to keep logs with `status:error` for 15 days)


![alt text](../imgs/logs_configure_index.png "")


### 6.6.2 (Best Practice) Use Exclusion Filter for Log Index
Ref: https://docs.datadoghq.com/logs/log_configuration/indexes#exclusion-filters

It's critical to exclude some unimportant logs. 

For example, if you are using Istio service mesh in K8s cluster, then all the traffic going in and out of pods are intercepted by istio sidecar proxy container. Those istio proxy logs can grow gigantic, so it's important to exclude logs from `service:proxyv2` when indexing.

![alt text](../imgs/logs_index_exclusion_filter.png "")



## 6.9 Archive Logs
Ref: https://docs.datadoghq.com/logs/log_configuration/archives/?tab=awss3

Configure your Datadog account to __forward all the logs ingested - whether indexed or not - to a cloud storage__. Keep your logs in a storage-optimized archive for longer periods of time and meet __compliance requirements__ while also keeping __auditability__ for ad hoc investigations, with Rehydration.

![alt text](../imgs/logs_archive.png "")



## 6.10 Use Facets to Filter Logs
Ref: https://docs.datadoghq.com/logs/explo

![alt text](../imgs/facets_in_explorer.gif "")


### 6.10.1 What is Facets

__Facets__ are __user-defined tags and attributes__ from your __indexed logs__ to:
- Search upon your logs
- Define log patterns
- Perform Log analytics 



### 6.10.2 Facet Panel

Shows count of logs matching each of them:

![alt text](../imgs/apm_facets.png "")

You can scope the search query by clicking on values.



### 6.10.3 Manage Facets
Ref: https://docs.datadoghq.com/logs/explorer/facets/#manage-facets

__Out-of-the-box facets__: Host, Service, URL Path, or Duration

The easiest way to create a facet is to add it from the __log side panel__, where most of the facet details—such as the field name or the underlying type of data—are pre-filled and it’s only a matter of double-checking:

![alt text](../imgs/logs_create_facet_from_attribute.png "")





