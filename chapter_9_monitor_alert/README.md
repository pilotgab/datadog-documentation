# 9. Monitor & Alerts Based on Metrics & Events
Refs: 
- https://docs.datadoghq.com/getting_started/monitors/#define-the-metric
- https://www.datadoghq.com/blog/monitoring-101-alerting/?_gl=1*qkmczj*_ga*NzE4NjU2NDg0LjE2MzEwMzg0NTg.*_ga_KN80RDFSQK*MTYzMjM5NzQ4NC4yMC4xLjE2MzIzOTg4MzguMA..



## 9.1 Define Metrics

Metrics can be fetched from existing Dashboard's widgets, or Datadog integration documents (`Data collected` section lists all kinds of metrics)

![alt text](../imgs/metric_query.png "")


## 9.2 Set Alert Conditions

![alt text](../imgs/alert_thresholds.png "")


## 9.3 Configure Alert Message

![alt text](../imgs/message_notify.png "")


## 9.4 Create SLO (Service Level Objectives)
Ref: https://docs.datadoghq.com/monitors/service_level_objectives/


### 9.4.1 Terminologies
Ref: https://docs.datadoghq.com/monitors/service_level_objectives/#key-terminology

- Service Level Indicator (SLI)
- Service Level Objective (SLO)
- Service Level Agreement (SLA)
- Error Budget




## 9.5 Sample Alert's Queries


```sh
# prod worker node's CPU usage
100 - avg:system.cpu.idle{k8s_namespace:prod} by {host}

# prod worker node's memory usage
1 - avg:system.mem.pct_usable{k8s_namespace:prod} by {host}

# the number of pods running per node
top(sum:kubernetes.pods.running{*} by {host}, 50, 'mean', 'desc')

# the number of app pods running
sum:kubernetes.pods.running{kube_namespace:prod,pod_phase:running,kube_app_name:YOUR_APP_NAME}

# Pod terminated
`Killing: Stopping container APP_NAME` from Kubernetes

# no logs from pod
count(kube_namespace:prod pod_name:POD_NAME*)

# high memory usage by containers
sum:kubernetes.memory.usage{kube_namespace:prod} by {kube_container_name,container_id}

# worker node whose status is NotReady
sum:kubernetes_state.nodes.by_condition{status:true,kube_cluster_name:eks-ue1-prod-APP-api-infra,!condition:ready}

# container OOM killed
sum:kubernetes.containers.state.terminated{kube_namespace:staging,reason:oomkilled} by {pod_name}

# container in CrashLoopBack state
sum:kubernetes.containers.state.waiting{kube_namespace:staging,reason:crashloopbackoff} by {pod_name}
```