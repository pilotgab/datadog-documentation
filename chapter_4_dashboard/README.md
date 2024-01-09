# 4. Dashboard

## 4.1 Use Out-of-box Dashboards
Dashboards > Dashboards list



## 4.2 See Detailed View of Widget by Maximizing

![alt text](../imgs/dashboard_widgets.png "")


## 4.3 Copy Query from Widget

![alt text](../imgs/dashboard_query.png "")

```
[{"style":{"palette":"dog_classic","type":"solid","width":"normal"},"type":"line","q":"1000*avg:aws.elb.latency{*,*,*} by {availability-zone}","preTemplateQuery":"1000*avg:aws.elb.latency{$region,$account,$scope} by {availability-zone}"}]
```


## 4.4 Copy Metrics Query from Widget

Get Metrics Query from Widget

![alt text](../imgs/dashboard_metrics_query.png "")

```sh
# ELB latency query (you can use this query to setup alert later)
1000*avg:aws.elb.latency{*,*,*} by {availability-zone}
```
