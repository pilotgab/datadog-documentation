# 11. APM (App Performance Monitoring aka tracing)

## 11.1 What is APM?
Refs: 
- https://docs.datadoghq.com/tracing/#terminology
- https://docs.datadoghq.com/tracing/

End-to-end request tracing from frontend to database for errors and latency.

Some examples are:
- [Service Map](https://docs.datadoghq.com/tracing/#service-map): visualize service topology
![alt text](../imgs/apm_service_map.png "")
- [Continuous Profiler](https://docs.datadoghq.com/tracing/#continuous-profiler): visualize and trace requests (AWS X-Ray or Istio Kiali dashboard kinda request tracing)
![alt text](../imgs/apm_request_tracing_breakdown.png "")
- [Service Performance Dashboards](https://docs.datadoghq.com/tracing/#service-performance-dashboards): profile requests, latency, errors
![alt text](../imgs/apm_profile_requests.png "")
- [Live Search](https://docs.datadoghq.com/tracing/#live-search): live real-time tracing
![alt text](../imgs/apm_live_search_requests.png "")
- correlate, pivot from traces to metrics, logs, infra seamlessly
![alt text](../imgs/apm_correlate_logs_metrics_infra.png "")
![alt text](../imgs/apm_correlate_logs_metrics_infra2.png "")




## 11.2 How to Setup APM for K8s 
Refs:
- https://docs.datadoghq.com/agent/kubernetes/apm/?tab=helm

### 11.2.1 Configure the Datadog Agent to Accept Traces

Configurations are in Helm overrides.yaml.

![alt text](../imgs/apm_setup1.png "")

```yaml
datadog:
  ## @param apm - object - required
  ## Enable apm agent and provide custom configs
  #
  apm:
    ## @param enabled - boolean - optional - default: false
    ## Enable this to enable APM and tracing, on port 8126
    #
    enabled: true
```


Notice the use of tags (`service`, `env`, `version`) as labels in K8s deployment yaml (Refer to ch9.3 How to Enable Unified Service Tagging)
![alt text](../imgs/apm_setup2.png "")

![alt text](../imgs/apm_setup3.png "")



### 11.2.2 Confiure your application pods to pull the host IP in order to communicate with the Datadog Agent

There are two ways:
- automatically with the __Datadog Admission Controller__, or
- manually; the application container needs the `DD_AGENT_HOST` environment variable that points to `status.hostIP`


__Automatically__ let [Datadog Admission Controller](https://docs.datadoghq.com/agent/cluster_agent/admission_controller/) inject `DD_AGENT_HOST` and `DD_ENTITY_ID` envs to container to configure DogStatsD and APM tracer libraries into the user’s application containers.

DAC will also inject Datadog standard tags or unified service tags (`env`, `service`, `version`) from application labels into the container environment variables.

```yaml
clusterAgent:
[...]
  ## @param admissionController - object - required
  ## Enable the admissionController to automatically inject APM and
  ## DogStatsD config and standard tags (env, service, version) into
  ## your pods
  #
  admissionController:
    enabled: true # <---------

    ## @param mutateUnlabelled - boolean - optional
    ## Enable injecting config without having the pod label:
    ## admission.datadoghq.com/enabled="true"
    #
    mutateUnlabelled: true  # <---------
```



### 11.2.3 Configure your application tracers to emit traces
Ref: https://docs.datadoghq.com/tracing/setup_overview/setup/nodejs/?tab=containers

Tracers are supposed for nodejs, python, java, Ruby, Go, C++, .NET Core, and .NET framework.

Below is app-level config for __nodejs__ app:
```sh
npm install dd-trace --save
```


Adding tracer in code (https://docs.datadoghq.com/tracing/setup_overview/setup/nodejs/?tab=containers#adding-the-tracer-in-code):
```js
// This line must come before importing any instrumented module.
const tracer = require('dd-trace').init();
```



## 11.3 Live View Search for 15 minutes
Ref: https://docs.datadoghq.com/tracing/trace_search_and_analytics/#live-search-for-15-minutes

![alt text](../imgs/apm_live_view.png "")


With the __APM Live Search__ you can:
- Monitor that a new deployment went smoothly by filtering on `version_id` of all tags.
- View outage-related information in real time by searching 100% of ingested traces for a particular org_id or customer_id that is associated with a problematic child span.
- Check if a process has correctly started by typing process_id and autocompleting the new process ID as a tag on child spans.
- Monitor load test and performance impact on your endpoints by filtering on the duration of a child resource.
- Run one-click search queries on any span or tag directly from the trace panel view.
- Add, remove, and sort columns from span tags for a customized view.



## 11.4 Search Bar in APM

Ref: https://docs.datadoghq.com/tracing/trace_search_and_analytics/query_syntax

### 11.4.1 Conditional Operators

- `AND` (`authentication AND failure`)
- `OR` (`authentication OR failure`)
- `-` (`authentication - failure`)


### 11.4.2 Query/Search Methods

There two kinds of search using:
- tags
- facets


Your traces inherit __tags__ from hosts and integrations that generate them. They can be used in the search and as facets as well.

Examples:
```sh
# All traces with the tag #env:prod or the tag #test
"env:prod" OR test

# All traces that contain tags #service:srvA or #service:srvB.
service:srvA OR service:srvB 

# All traces that contain #env:prod and that do not contain #version:beta
"env:prod" AND -"version:beta"

# using wildcards
service:web*
```


__Facets__

A Facet displays all the distinct attribute values (`service`, `namespace`, `status`, etc) or a tag. 

Use Facets to filter on your Traces. To search on a specific facet you must add it as a facet first then add @ to specify you are searching on a facet.

The search bar and url automatically reflect your selections.

Facet panel
![alt text](../imgs/apm_facets.png "")


```sh
# retrieve all traces that have a response time over 100ms 
@http.response_time:>100


# retrieve all your 4xx errors
@http.status_code:[400 TO 499]
```



## 11.5 How to Configure Retention Filters 
Ref: https://docs.datadoghq.com/tracing/#trace-retention-and-ingestion


### 11.5.1 Create Custom Retention Filter
The lifecycle of ingested traces is below:
![alt text](../imgs/apm_traces_injested_indexed_archived_lifecycle.png "")


Ingested and kept for __15 mins__ for live search > Indexed using custom tags in retention filter for __15 days__


Can filter traces using tags to retain:
![alt text](../imgs/retention_filters.png "")


To ensure all __production errors__ are retained and available for search and analytics for 15 days, create a 100 percent retention filter scoped to `env:prod` and `status:error`:

![alt text](../imgs/apm_custom_retention_filter.png "")



### 11.5.2 Intelligent Retention Filter
Ref: https://docs.datadoghq.com/tracing/trace_retention_and_ingestion/#datadog-intelligent-retention-filter

For 30 days, intelligent retention retains:
- A representative selection of __errors__, ensuring error diversity (for example, response code 400s, 500s).
- __High Latency__ in the different quartiles p75, p90, p95.
- All Resources with any traffic will have associated Traces in the past for any time window selection.
- True maximum duration trace for each time window.




## 11.6 Deployment Tracking using `version` tag
Ref: https://docs.datadoghq.com/tracing/deployment_tracking/

Since we have setup unified service tagging earlier, `version` tag is added to k8s deployment yaml file. 

This enables tracking app performance, latency, error rate by versions.

![alt text](../imgs/apm_service_by_version.png "")




## 11.7 How to Connect Logs and Traces side-by-side
Refs:
- https://docs.datadoghq.com/tracing/#connect-logs-and-distributed-traces
- https://docs.datadoghq.com/tracing/connect_logs_and_traces/

Correlation between Datadog APM and Datadog Log Management is improved by the injection of __trace IDs, span IDs__, env, service, and version as attributes in your logs.

![alt text](../imgs/apm_correlate_logs_metrics_infra.png "")
![alt text](../imgs/apm_correlate_logs_metrics_infra2.png "")


### 11.7.1 Automatic Injection (for Nodejs)
Ref: https://docs.datadoghq.com/tracing/connect_logs_and_traces/nodejs/

By default, APM traces aren't connected/correlated with logs:

![alt text](../imgs/apm_connect_trace_with_logs.png "")


Enable injection with the environment variable `DD_LOGS_INJECTION=true` in k8s deployment yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    tags.datadoghq.com/env: "<environment>"
    tags.datadoghq.com/service: "<service>"
    tags.datadoghq.com/version: "<version>"
spec:
  template:
    metadata:
      labels:
        tags.datadoghq.com/env: "<environment>"
        tags.datadoghq.com/service: "<service>"
        tags.datadoghq.com/version: "<version>"
    spec:
      containers:
        - env:
            - name: DD_AGENT_HOST
              valueFrom:
                fieldRef:
                  fieldPath: status.hostIP
            - name: DD_ENV
              valueFrom:
                fieldRef:
                  fieldPath: metadata.labels['tags.datadoghq.com/env']
            - name: DD_VERSION
              valueFrom:
                fieldRef:
                  fieldPath: metadata.labels['tags.datadoghq.com/version']
            - name: DD_LOGS_INJECTION  # <--- set this env to inject trace & span IDs to container. Ref: https://docs.datadoghq.com/tracing/connect_logs_and_traces/nodejs/
              value: "true"
```

or configuring the tracer directly

```js
const tracer = require('dd-trace').init({
    logInjection: true
});
```


Then you will be able to correlate APM traces with logs:

![alt text](../imgs/apm_trace_logs_correlation.png "")



## 11.8 How to Connect Synthetic Tests with APM Tracing
Refs:
- https://docs.datadoghq.com/tracing/#connect-synthetic-test-data-and-traces
- https://docs.datadoghq.com/synthetics/apm/

Jump from a test run that potentially failed to the root cause of the issue by looking at the trace generated by that very test run.

![alt text](../imgs/apm_synthetics_tests.gif "")


