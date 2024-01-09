# 7. Events
Ref: https://docs.datadoghq.com/events/

Events are records of notable changes relevant for managing and troubleshooting IT operations, such as code deployments, service health, configuration changes, or monitoring alerts.

For example, if a __docker container__ exits with error, or __K8s pod__ gets terminated due to an error, Datadog will pick up those as events.


There are more than 100 Datadog integrations support events collection, including Kubernetes, Docker, Jenkins, Chef, Puppet, AWS ECS or Autoscaling, Sentry, and Nagios.



You can monitor and alert based on specific events.



## 7.1 Event Streams

The Datadog Events Stream shows an instant view of your infrastructure and services events, to help you troubleshoot issues happening now or in the past.

![alt text](../imgs/event-stream.png "")

