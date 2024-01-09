# 5. Infrastructure
Track hosts, containers, processes, pods, and serverless functions. Quickly visualize your environment, identify outliers, detect usage patterns



## 5.1 Infra list
Ref: https://docs.datadoghq.com/infrastructure/list/

Infrastructure list shows all of your hosts monitored by Datadog with activity during the last 2 hours(default) and up to 1 week. Search your hosts, or group them by tags

![alt text](../imgs/infra_list.png "")


## 5.2 Hostmap
Ref: https://docs.datadoghq.com/infrastructure/hostmap/

Host Maps visualize hosts together on one screen, with metrics made comprehensible through color and shape:

- Host maps enable you to see __distributions of machines in each of your availability zones__ (AZ)
- color of each host is set to represent the percentage of __CPU usage__ on that host, where the color ranges from green (0% utilized) to __orange__ (100% utilized)

![alt text](../imgs/infra_hostmap.png "")

__Filter by__ limits the Host Map to a specific subset of an infrastructure. The filter input bar in the top left enables filtering of the Host Map by tags as well as Datadog-provided attributes.


** Note: there is a distinction between filtering for tag:value and "tag:value"—filtering for tag:value strictly matches the tag, while filtering for "tag:value" performs a search on that text.



## 5.3 Container Map
Ref: https://docs.datadoghq.com/infrastructure/containermap/

You can see all of your containers together on one screen with customized groupings and filters, as well as metrics made instantly comprehensible through color and shape.

![alt text](../imgs/container_map.png "")



## 5.4 Live Container
Ref: https://docs.datadoghq.com/infrastructure/livecontainers/?tab=helm

Datadog Live Containers enables real-time visibility into all containers across your environment.

- monitor the state of pods, deployments
- view resource specifications
- correlate node activity with related logs

![alt text](../imgs/infra_containers.png "")


Kubernetes resources for Live Containers requires Agent version >= 7.27.0 and Cluster Agent version >= 1.11.0 prior to the configurations below.


### Containers View
Ref: https://docs.datadoghq.com/infrastructure/livecontainers/?tab=helm#containers-view

The Containers view includes Scatter Plot and Timeseries views, and a table to better organize your container data by fields such as container name, status, and start time.

![alt text](../imgs/live_containers_scatterplot.jpeg "")


### Kubernetes resources view
Ref: https://docs.datadoghq.com/infrastructure/livecontainers/?tab=helm#kubernetes-resources-view

This view can show you K8s object's YAML definition, tags, annotations, object status, etc without having to use `kubectl get` or `kubectl describe` commands.

![alt text](../imgs/containers_resource_view.png "")


### GROUP BY FUNCTIONALITY AND FACETS
Ref: https://docs.datadoghq.com/infrastructure/livecontainers/?tab=helm#group-by-functionality-and-facets

You can also leverage facets on the left hand side of the page to quickly group resources or filter for resources you care most about, such as pods with a CrashLoopBackOff pod status.

![alt text](../imgs/containers_filter_by_facets.gif "")


### Container logs
Ref: https://docs.datadoghq.com/infrastructure/livecontainers/?tab=helm#container-logs

View streaming logs for any container like `docker logs -f` or `kubectl logs -f` in Datadog.

