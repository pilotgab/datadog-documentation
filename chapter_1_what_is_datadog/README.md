# Monitor, Collect Logs & Metrics, and Alert using Datadog
Refs: 
- https://docs.datadoghq.com/agent/kubernetes/?tab=helm
- https://github.com/DataDog/helm-charts/tree/master/charts/datadog



# 1. What is Datadog?

Multi-cloud and k8s and serverless solutions for __monitoring metrics and events__, __aggregating logs__, __visualizing data__ in dashboards, and __alerting__. 

![alt text](../imgs/dd_overview.png "")

1. __Integration__: Integrate with Cloud Providers for their infra metrics (i.e. EC2, ELB, RDS metrics etc)
2. __Dashboard__: Live and high-resolution dashboard for metrics (CPU & memory & disk utilization) and events (docker & k8s events etc). Synchronous mousing across all graphs
![alt text](../imgs/dashboard_aws.png "")
![alt text](../imgs/dashboard_docdb.png "")
![alt text](../imgs/dashboard_elb.png "")
![alt text](../imgs/dashboard_k8s.png "")
3. __Infrastructure__: Track hosts, containers, processes, pods, and serverless functions. Quickly visualize your environment, identify outliers
, detect usage patterns
![alt text](../imgs/infra_hostmap.png "")
![alt text](../imgs/infra_list.png "")
![alt text](../imgs/infra_containers.png "")
![alt text](../imgs/infra_network.png "")
4. __Log Managenet__: Aggregate, search/filter, and visualize logs
![alt text](../imgs/logs.png "")
![alt text](../imgs/logs_pattern.png "")
5. __Events__: You can monitor events from AWS, GCP, Azure, Docker, K8s, etc. You can filter by user, source, tag, host, status, priority, and incident
6. __Monitor & Alerts__: Alert based on custom metrics (# of k8s host & pod, resource utilization, existance/absense of certain keywords in logs or metrics)
![alt text](../imgs/monitor_alert1.png "")
![alt text](../imgs/monitor_alert2.png "")
![alt text](../imgs/events_asg.png "")
![alt text](../imgs/events_cw_alarm.png "")
![alt text](../imgs/events_docker.png "")
7. __APM (App Performance Monitoring or tracing)__: Scan and monitor application performance (latency, error rates, etc) by tracing requests
![alt text](../imgs/apm_services.png "")
![alt text](../imgs/apm_traces.png "")
![alt text](../imgs/apm_traces_details.png "")
8. __Synthetics__: Monitor and test public endpoints for SLAs and track performance issues
![alt text](../imgs/synthetics_test.png "")
![alt text](../imgs/synthetics_test_details.png "")
9. __Network Monitoring__: Visualize network traffic flow 
![alt text](../imgs/network_map.png "")



Datadog agent is open source project: https://github.com/DataDog/dd-agent
