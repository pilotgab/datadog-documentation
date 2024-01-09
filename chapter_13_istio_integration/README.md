# 13. (Advanced) Enable Istio Service Mesh Integration with Datadog

Refs:
- https://www.datadoghq.com/blog/istio-datadog/#set-up-the-istio-integration
- https://docs.datadoghq.com/integrations/istio/
- https://www.datadoghq.com/blog/istio-datadog/?_gl=1*842vkn*_ga*NzE4NjU2NDg0LjE2MzEwMzg0NTg.*_ga_KN80RDFSQK*MTYzNDQ4NTU1My40My4xLjE2MzQ0ODc0MzMuMA..
- https://docs.datadoghq.com/agent/kubernetes/integrations/?tab=kubernetes


> Datadog monitors every aspect of your __Istio__ environment, so you can:
> - Assess the health of Envoy and the Istio control plane with logs.
> - Break down the __performance__ of your service mesh with request, bandwidth, and resource consumption metrics (see below).
  ![alt text](/imgs/dd_istio_monitoring.png "")
> - Map network communication between containers, pods, and services over the mesh with - __Network Performance Monitoring__.
> - Drill into __distributed traces__ for applications transacting over the mesh with __APM__.




## 13.1 Disable sidecar injection for Datadog Agent pods
Refs: 
- https://docs.datadoghq.com/integrations/istio/#disable-sidecar-injection-for-datadog-agent-pods
- https://www.datadoghq.com/blog/istio-datadog/?_gl=1*842vkn*_ga*NzE4NjU2NDg0LjE2MzEwMzg0NTg.*_ga_KN80RDFSQK*MTYzNDQ4NTU1My40My4xLjE2MzQ0ODc0MzMuMA..#disable-automatic-sidecar-injection-for-datadog-agent-pods

If you are installing the Datadog Agent in a container, Datadog recommends that you first disable Istio’s sidecar injection.


Either set `sidecar.istio.io/inject: "false"` in daemonset's annotations 
```sh
...
spec:
   ...
  template:
    metadata:
      annotations:
        sidecar.istio.io/inject: "false"
```

by patching like below
```sh
kubectl patch daemonset datadog-agent -p '{"spec":{"template":{"metadata":{"annotations":{"sidecar.istio.io/inject":"false"}}}}}'
```


or set it from overrides.yaml
```yaml
agent:
  podAnnotations: 
    sidecar.istio.io/inject: "false"

clusterAgent:
  podAnnotations:
    sidecar.istio.io/inject: "false"
```


Upgrade datadog helm chart
```sh
helm upgrade datadog \
  -f overrides.yaml \
  datadog/datadog \
  --set datadog.apiKey=XYZ \
  --set dadadog.appKey=XYZ
```

Describe datadog agent pod:
```sh
$ k describe po datadog-dfglr
Name:         datadog-dfglr
Namespace:    default
...
Annotations:  
              container.apparmor.security.beta.kubernetes.io/system-probe: unconfined
              container.seccomp.security.alpha.kubernetes.io/system-probe: localhost/system-probe
              kubernetes.io/psp: eks.privileged
              sidecar.istio.io/inject: false  # <--------- annotation applied
```


## 13.2 Configure and Parse Istio Logs

There are two ways to configure Datadog to correctly parse istio logs:
- from DD log pipeline
- from Helm chart

### 13.2.1 From Log Pipeline

Add third party pipeline library for istio:
![alt text](/imgs/log_pipeline_istio.png "")

Duplicate it and add another source `source:proxyv2` (logs from sidecar envoy proxy) in addition to `source:istio` (logs from standalone envoy in ingress gateway etc):
![alt text](/imgs/dd_log_pipeline_istio_library.gif "")


### 13.2.2 From Helm Chart

```yaml
​  confd:
    istio.yaml: |-
      ad_identifiers:
        - istio
      init_config:
      instances:
        - <INSTANCE_CONFIG>
      logs:
        - source: istio
        - service: istio
```



## 13.3 Integrate Istio with Datadog APM
Refs:
- https://docs.datadoghq.com/tracing/setup_overview/proxy_setup/?tab=istio


### 13.3.1 Setup Receiving traces
Ref: https://www.datadoghq.com/blog/istio-datadog/?_gl=1*842vkn*_ga*NzE4NjU2NDg0LjE2MzEwMzg0NTg.*_ga_KN80RDFSQK*MTYzNDQ4NTU1My40My4xLjE2MzQ0ODc0MzMuMA..#receiving-traces

Given that you already enabled DD APM in overrides.yaml as below
```yaml
datadog:
  apm:
    enabled: true
```

This will:
- Deploy the Trace Agent within the node-based Agent pod, which allows node-based Agents to receive traces and report them to Datadog
- Set up a Kubernetes NetworkPolicy that allows APM traces to reach node-based Agent pods
- Use a Kubernetes Service to direct trace data to an available node-based Agent pod


With this config, the node-based Agents should be able to __receive traces from Envoy proxies__ throughout your cluster. 


### 13.3.2 Setup Sending traces
Refs: 
- https://docs.datadoghq.com/tracing/setup_overview/proxy_setup/?tab=istio#istio-configuration-and-installation
- https://www.datadoghq.com/blog/istio-datadog/?_gl=1*842vkn*_ga*NzE4NjU2NDg0LjE2MzEwMzg0NTg.*_ga_KN80RDFSQK*MTYzNDQ4NTU1My40My4xLjE2MzQ0ODc0MzMuMA..#sending-traces


Istio has __built-in support for distributed tracing__ using several possible backends, including Datadog. You need to configure tracing by setting three options:
- `pilot.traceSampling` is the percentage of requests that Istio will record as traces. Set this to 100.00 to send all traces to Datadog.
- `global.proxy.tracer` instructs Istio to use a particular tracing backend, in our case datadog.
- `tracing.enabled` instructs Istio to record traces of requests within your service mesh.


Upgrade Istio to send traces automatically to Datadog:
```sh
istioctl upgrade \
  -f overrides.yaml \
  --set values.global.proxy.tracer=datadog \
  --set values.pilot.traceSampling=100.0 \
  --set values.tracing.enabled=true
```

This might fail if `jaeger` is installed
```sh
✔ Istio core installed                                                                                     ✔ Istiod installed                                                                                    ✔ Egress gateways installed                                                                                     ✔ Ingress gateways installed                                                                                          
✘ Addons encountered an error: failed to wait for resource: resources not ready after 5m0s: timed out waiting for the condition
Deployment/istio-system/istio-tracing
- Pruning removed resources                                                                                           Error: failed to apply manifests: errors occurred during operation
```

So delete `istio-tracing` deployment first
```sh
kubectl delete deploy istio-tracing  -n istio-system
```

Then reinstall istio
```sh
istioctl install \
  -f overrides.yaml \
  --set values.global.proxy.tracer=datadog \
  --set values.pilot.traceSampling=100.0 \
  --set values.tracing.enabled=true              
```




### 13.3.3 Visualize mesh requests with flame graphs
Ref: https://www.datadoghq.com/blog/istio-datadog/?_gl=1*842vkn*_ga*NzE4NjU2NDg0LjE2MzEwMzg0NTg.*_ga_KN80RDFSQK*MTYzNDQ4NTU1My40My4xLjE2MzQ0ODc0MzMuMA..#visualize-mesh-requests-with-flame-graphs

![alt text](/imgs/istio-apm-trace-view.png "")

Once you set up APM in your Istio mesh, you can inspect individual request traces using __flame graphs__. A flame graph is a visualization that displays the __service calls__ that were executed to fulfill a request. The __duration of each service call__ is represented by the __width of the span__, and in the sidebar, you can see the services called and the percent of time spent on each. You can click any span to see further information, such as metadata and error messages.


Note that in several spans, `envoy.proxy` precedes the name of the resource (which is the specific endpoint to which the call is addressed, e.g., `main-app.apm-demo.svc.cluster.local:80`). This is because __Envoy proxies all requests within an Istio mesh__. This architecture also explains why envoy.proxy spans are generated in pairs: the first span is created by the sidecar proxying the outgoing request, and the matching second span is from the sidecar that receives it.



## 13.4 Exclude Logs from Istio (standalone and proxy images)

In overrides.yaml,
```yaml
agent:
  env: 
    # can't mix DD_CONTAINER_EXCLUDE (global) with DD_CONTAINER_EXCLUDE_METRICS|LOGS (local).
    - name: DD_CONTAINER_EXCLUDE_METRICS # ref: https://docs.datadoghq.com/agent/guide/autodiscovery-management/?tab=containerizedagent#exclude-containers
      value: "image:docker.io/istio/proxyv2 kube_namespace:kube-system kube_namespace:istio-system kube_namespace:prometheus kube_namespace:grafana kube_namespace:kubernetes-dashboard kube_namespace:jenkins kube_namespace:kube-node-lease kube_namespace:kube-public kube_namespace:default
    - name: DD_CONTAINER_EXCLUDE_LOGS
      value: "image:docker.io/istio/proxyv2 kube_namespace:kube-system kube_namespace:istio-system kube_namespace:prometheus kube_namespace:grafana kube_namespace:kubernetes-dashboard kube_namespace:jenkins kube_namespace:kube-node-lease kube_namespace:kube-public kube_namespace:default
    - name: DD_CONTAINER_INCLUDE_LOGS # Inclusion always takes precedence. Ref: https://docs.datadoghq.com/agent/guide/autodiscovery-management/?tab=containerizedagent#inclusion-and-exclusion-behavior
      value: "image:us.gcr.io/k8s-artifacts-prod/autoscaling/cluster-autoscaler"
```

Upgrade datadog helm chart
```sh
helm upgrade datadog \
  -f overrides.yaml \
  datadog/datadog \
  --set datadog.apiKey=XYZ \
  --set dadadog.appKey=XYZ
```

