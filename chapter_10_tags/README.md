# 10. Tags
Refs:
- https://docs.datadoghq.com/getting_started/tagging/

## 10.1 Tags Overview

> Tags are a way of adding dimensions to Datadog telemetries so they can be filtered, aggregated, and compared in Datadog visualizations.

> [The Agent can create and assign tags to all metrics, traces, and logs emitted by a Pod, based on its labels or annotations.](https://docs.datadoghq.com/agent/kubernetes/tag/?tab=containerizedagent)


Tag is key-value pair separated by `:`, as in `<TAG_KEY>:<TAG_VALUE>` (`env:staging`).

Example tag values are:
- `env:staging:east` (key: `env`, value: `staging:east`)
- `env_staging:east` (key: `env_staging`, value: `east`)

Some reserved tags are:
- __host__:	Correlation between metrics, traces, processes, and logs
- __device__:	Segregation of metrics, traces, processes, and logs by device or disk
- __source__:	Span filtering and automated pipeline creation for log management
- __service__:	Scoping of application specific data across metrics, traces, and logs
- __env__:	Scoping of application specific data across metrics, traces, and logs
- __version__:	Scoping of application specific data across metrics, traces, and logs



Why Tags?

> it’s more helpful to look at CPU usage across a __collection of hosts__ that represents a service, rather than CPU usage for __server A or server B__ separately.


## 10.2 Default Out-of-the-Box Tags Created by Datadog Agent
Ref: https://docs.datadoghq.com/agent/kubernetes/tag/?tab=containerizedagent#out-of-the-box-tags

![alt text](../imgs/tags_default.png "")

- __container_id__
- __display_container_name__
- __pod_name__
- __kube_ownerref_name__
- __kube_job__
- __kube_replica_set__
- __kube_service__
- __kube_daemon_set__
- __kube_container_name__
- __kube_namespace__
- __kube_app_name__
- __kube_app_instance__
- __kube_app_version__
- __kube_app_component__
- __kube_app_part_of__
- __kube_app_managed_by__
- __pod_phase__
- __environment__
- __kube_ownerref_kind__
- __kube_deployment__
- __kube_stateful_set__
- __persistentvolumeclaim__
- __kube_cronjob__
- __image_name__
- __short_image__
- __image_tag__


## 10.3 How to Assign Tags
Ref: https://docs.datadoghq.com/getting_started/tagging/#assigning-tags

| Method      | ASSIGN TAGS |
| ----------- | ----------- |
| Helm YAML   | Manually in your main Agent or integration configuration files. |
| From UI     | In the Datadog site                        |
| API         | When using Datadog’s API                   |
| DogStatsD SDK | When submitting metrics with DogStatsD   |



## 10.4 How to Enable Unified Service Tagging

Before Enabling Unified Service Tagging
![alt text](../imgs/before_enabling_unified_service_tagging.png "")

After Enabling Unified Service Tagging
![alt text](../imgs/after_enabling_unified_service_tagging.png "")


### 10.4.1 Unified Service Tagging Overview
Ref: https://docs.datadoghq.com/getting_started/tagging/unified_service_tagging/?tab=kubernetes

![alt text](../imgs/unified_service_tagging.gif "")

Unified Service Tagging
> ties all Datadog telemetry together (hence `unified`), including logs, metrics, infra, through the use of three standard tags: `env`, `service`, and `version`. 


With __Unified Service Tagging__, you can 
> View service data __based on environment or version__ in a __unified fashion__ within the Datadog site. Also navigate seamlessly across traces, metrics, and logs with consistent tags

This means you can visualize data based on env or version value without switching views. And __correlating metrics, logs, and traces__ are as easy as __clicking tabs on dashboard or right clicking the value__.


You enable it by simply assigning three reserved tags: `env`, `service`, and `version`.


### 10.4.2 Configure Unified Service Tagging in k8s
Ref: https://docs.datadoghq.com/getting_started/tagging/unified_service_tagging/?tab=kubernetes#configuration-1


These tags will be set as k8s object's (i.e. deployment) environment variables or labels.

In K8s deployment yaml or helm chart's template yaml, add environment variables and labels like below:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    # this part is for Kubernetes State Metrics
    tags.datadoghq.com/env: "<ENV>"  # <---------- three tags as labels
    tags.datadoghq.com/service: "<SERVICE>"
    tags.datadoghq.com/version: "<VERSION>"
...
template:
  metadata:
    labels:
      # this part is for Pod-level metrics. Ref: https://docs.datadoghq.com/getting_started/tagging/unified_service_tagging/?tab=kubernetes#partial-configuration
      tags.datadoghq.com/env: "<ENV>"  
      tags.datadoghq.com/service: "<SERVICE>"
      tags.datadoghq.com/version: "<VERSION>"
  containers:
  -  ...
     env:
          # this part is for APM tracer and StatsD client
          - name: DD_ENV
            valueFrom:
              fieldRef:
                fieldPath: metadata.labels['tags.datadoghq.com/env']  # <---------- injecting env values from labels
          - name: DD_SERVICE
            valueFrom:
              fieldRef:
                fieldPath: metadata.labels['tags.datadoghq.com/service']
          - name: DD_VERSION
            valueFrom:
              fieldRef:
                fieldPath: metadata.labels['tags.datadoghq.com/version']
```


If using istio and therefore have istio sidecar proxy container inside a pod, then you should specify container name
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    # this part is for Kubernetes State Metrics
    tags.datadoghq.com/env: "<ENV>"  # <---------- three tags as labels
    tags.datadoghq.com/service: "<SERVICE>"
    tags.datadoghq.com/version: "<VERSION>"
...
template:
  metadata:
    labels:
      # this part is for Pod-level metrics. Ref: https://docs.datadoghq.com/getting_started/tagging/unified_service_tagging/?tab=kubernetes#partial-configuration
      tags.datadoghq.com/CONTAINER_NAME.env: "<ENV>"  # <---- in case multiple containers in pod
      tags.datadoghq.com/CONTAINER_NAME.service: "<SERVICE>"
      tags.datadoghq.com/CONTAINER_NAME.version: "<VERSION>"
...
```


## 10.5 How to Use Tags to Filter on Datadog Dashboard
Ref: https://docs.datadoghq.com/getting_started/tagging/#using-tags

![alt text](../imgs/tag_dashboard.png "")


|    AREA     |	USE TAGS TO
| ----------- | ----------- |
| Events	    | Example: `tags:service:coffee-house` ![alt text](../imgs/tag_event.png "")| 
|__Dashboards__| Example: `service:coffee-house` in __FROM__ textbox. ![alt text](../imgs/tag_dashboard.png "")| 
| Infrastructure| Example: `service:coffee-house`. ![alt text](../imgs/tag_container.png "")| 
| Monitors	  | Example: `tags:service:coffee-house` ![alt text](../imgs/tag_event.png "")| 
| __Metrics__	| Example: `service:coffee-house` in __Over__ textbox ![alt text](../imgs/tag_metrics.png "")| 
| Integrations| Optionally limit metrics for AWS, Google Cloud, and Azure| 
| __APM	Filter__ | Example: `service:coffee-house`. ![alt text](../imgs/tag_apm.png "")| 
| __Logs__    | Example: `service:coffee-house`. ![alt text](../imgs/tag_logs.png "")| 


By the way, here are conditional expressions you can use in dashboard (https://docs.datadoghq.com/getting_started/tagging/using_tags/?tab=assignment#dashboards)
- `NOT`, `!`
- `AND`, `,`
- `OR`
- `key IN (tag_value1, tag_value2,...)`
- `key NOT IN (tag_value1, tag_value2,...)`

![alt text](../imgs/tag_and_or.png "")

