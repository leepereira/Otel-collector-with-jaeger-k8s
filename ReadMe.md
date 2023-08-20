# Instrumenting Java Metrics & Traces with Otel Collector & Jaeger backend 
---
The goal of this documentation is to showcase auto-instrumenting a Java application with the OpenTelemetry Java agent to pass metrics and traces to a backend like Jaeger. 

### Architecture Pattern
---
### Getting Started
---
#### Download the OpenTelemetry Java agent can be downloaded from : 

https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/latest/download/opentelemetry-javaagent.jar

We are using an existing application jar that can be found in the existing repo that will be used as the java application that would be instrumented

#### Deploy the K8s Cluster

We are using a AKS cluster to host this setup

#### Deploy the Jaeger Backend 

```yaml
helm repo add jaeger-all-in-one https://raw.githubusercontent.com/hansehe/jaeger-all-in-one/master/helm/charts
```

```yaml
helm install jaeger jaeger-all-in-one/jaeger-all-in-one
```

jaeger-jaeger-all-in-one-0 pod will be deployed as a single pod
```yaml
NAME                         READY   STATUS    RESTARTS   AGE
jaeger-jaeger-all-in-one-0   1/1     Running   0          47s
```

Service for the jaeger backend will also be deployed 
```yml
NAMESPACE           NAME                                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                                                                       AGE
default             jaeger-jaeger-all-in-one            ClusterIP   10.0.148.119   <none>        6831/UDP,6832/UDP,5775/UDP,5778/TCP,16686/TCP,14250/TCP,14268/TCP,14269/TCP   3m52s
default             jaeger-jaeger-all-in-one-headless   ClusterIP   None           <none>        6831/UDP,6832/UDP,5775/UDP,5778/TCP,16686/TCP,14250/TCP,14268/TCP,14269/TCP   3m52s
```

Optionally you can validate that the jaeger service is up by running a port-forward
```yml
kubectl port-forward svc/jaeger-jaeger-all-in-one 16686:16686    
```
```yml
open http://127.0.0.1:16686
```

### Deploy Otel collector

```yml
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm install k8s-opentelemetry-collector . --set mode=daemonset
```

Otel collector will be deployed as a daemonset

```yml
k8s-opentelemetry-collector-agent-525g7   1/1     Running   0          25m
k8s-opentelemetry-collector-agent-t7gf5   1/1     Running   0          25m
k8s-opentelemetry-collector-agent-xs2p4   1/1     Running   0          25m

NAME                                STATUS   ROLES   AGE   VERSION
aks-agentpool-12743558-vmss000000   Ready    agent   97m   v1.26.6
aks-agentpool-12743558-vmss000001   Ready    agent   97m   v1.26.6
aks-agentpool-12743558-vmss000002   Ready    agent   97m   v1.26.6
```

Only changes to the values.yaml in the otel collector deployment 

```yaml
config:
  exporters:
    logging: {}
    jaeger:
      endpoint: jaeger-jaeger-all-in-one:14250 # service at which the jaeger service is running
      tls:
        insecure: true


service:
#service for the otel collector is published a LoadBalancer instead of ClusterIP
  enabled: true

  type: LoadBalancer
```

Record the LoadBalancer IP address : \<external_ip\>

```yml
default             k8s-opentelemetry-collector         LoadBalancer   10.0.171.231   <external_ip>   6831:31844/UDP,14250:30039/TCP,14268:32484/TCP,4317:32495/TCP,4318:32059/TCP,9411:30297/TCP   32m
```

#### Start the application

Setup the following environment variables on the session from where the application jar will be launched
```yml
export OTEL_TRACES_EXPORTER=otlp
export OTEL_METRICS_EXPORTER=otlp
export OTEL_EXPORTER_OTLP_ENDPOINT=http://<external_ip>:4317
export OTEL_RESOURCE_ATTRIBUTES=service.name=course-api,service.version=1.0
export OTEL_EXPORTER_OTLP_INSECURE=true
```

Enable the instrumentation agent using the -javaagent flag to the JVM.

```yaml
java -javaagent:opentelemetry-javaagent.jar -Dotel.service.name=course-api -jar course-api-0.0.1.jar
```

Configuration parameters are passed as Java system properties (-D flags) or as environment variables.
Documentation for list of configuration items : https://opentelemetry.io/docs/instrumentation/java/automatic/agent-config/

Validate that the application has started :
```yml
curl http://localhost:8080/hello   
```

Navigate to the jaeger endpoint and validate that you can see the traces being generated
```yml
http://localhost:16686/
```

course-api should show up in the services drop-downs

