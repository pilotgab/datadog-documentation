# 8. Metrics
Ref: https://docs.datadoghq.com/metrics/


## 8.1 What is StatsD?

![alt text](../imgs/dogstatsd.png "")

> The easiest way to get your custom application metrics into Datadog is to send them to DogStatsD, a metrics aggregation service bundled with the Datadog Agent



## 8.2 Enable StatsD to collect custom metrics
https://docs.datadoghq.com/developers/dogstatsd/?tab=helm

Configure Datadog Helm chart's overrides.yaml:
```sh
dogstatsd:
    port: 8125
    useHostPort: true
    nonLocalTraffic: true
```

Then helm upgrade.


## 8.3 DogstatsD Client setup
Ref: https://docs.datadoghq.com/developers/dogstatsd/?tab=helm#code

You need to install DogStatsD client library to your code (python, Java, .Net, Go, PHP, etc)

For Python,
```sh
pip install datadog
```

Then
```py
from datadog import initialize, statsd

options = {
    'statsd_host':'127.0.0.1',
    'statsd_port':8125
}

initialize(**options)
```


For [Nodejs](https://docs.datadoghq.com/integrations/node/),
```sh
npm install hot-shots
```

Then in app code,
```js
var StatsD = require('hot-shots');
var dogstatsd = new StatsD();

// Increment a counter.
dogstatsd.increment('page.views')
```


## 8.4 Configure Pod to be able to connect to Datadog agent pod through DD_AGENT_HOST

Either inject `DD_AGENT_HOST` env to pod's yaml manually like below,
```yaml
    env:
    - name: DD_AGENT_HOST
      valueFrom:
          fieldRef:
              fieldPath: status.hostIP
```

or in the later chapter (ch 11.2.2 Confiure your application pods to pull the host IP in order to communicate with the Datadog Agent), __Automatically__ let [Datadog Admission Controller](https://docs.datadoghq.com/agent/cluster_agent/admission_controller/) inject `DD_AGENT_HOST` and `DD_ENTITY_ID` envs to container to configure DogStatsD and APM tracer libraries into the user’s application containers.





# But...What Metrics in K8s Cluster to Monitor?

Refs: 
- https://www.datadoghq.com/blog/monitoring-101-collecting-data/
- https://www.datadoghq.com/blog/eks-cluster-metrics/
- https://www.datadoghq.com/blog/monitoring-kubernetes-performance-metrics/#toc-resource-utilization6



Summary:

- [Cluster state metrics](https://www.datadoghq.com/blog/eks-cluster-metrics/#cluster-state-metrics)
  ![alt text](../imgs/cluster_state_metrics.png "")
  - node status
    - OutOfDisk
    - Ready
    - MemoryPressure (node memory is too low)
    - PIDPressure (too many running processes)
    - DiskPressure (remaining disk capacity is too low)
    - NetworkUnavailable
  - Desired Pods
  - Current Pods
  - Pod capacity
  - Available Pods
  - Unavailable Pods
- Container and node resource metrics
  ![alt text](../imgs/resource_metrics.png "")
  - Memory requests	
  - Memory limits	
  - Allocatable memory	
  - Memory utilization	
  - CPU requests	
  - CPU limits
  - Allocatable CPU
  - CPU utilization	
  - Disk utilization		
- AWS service metrics