Architecture Pattern : 
https://www.jaegertracing.io/docs/1.47/architecture/#with-opentelemetry

OpenTelemetry Collectors will be placed between the SDKs and the Jaeger Collectors.

The OpenTelemetry Collectors can be run as an application sidecar, as a host agent / daemon, or as a central cluster.

We are leaning towards deploying OpenTelemetry Collectors as a daemon set

Installing Jaeger : 
https://github.com/jaegertracing/helm-charts


$ helm repo add jaegertracing https://jaegertracing.github.io/helm-charts

If installing Jaeger with the operator : 
Jaeger Operator uses webhooks to validate Jaeger custom resources (CRs). This requires an installed version of the cert-manager

Cert Manager install
$ kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.6.3/cert-manager.yaml

kubectl create namespace observability # <1>
kubectl create -f https://github.com/jaegertracing/jaeger-operator/releases/download/v1.47.0/jaeger-operator.yaml -n observability # <2>

```yml
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: simplest
```

```yml
kubectl apply -f simplest.yaml
```

