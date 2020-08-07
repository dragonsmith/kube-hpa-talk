# Kubernetes HPA talk sandbox

## Preparations

### Sandbox cluster

I'm using Docker for Mac default Kubernetes cluster.

To switch to its context:

```shell
kubectl config use-context docker-desktop
```

### Metrics server

```shell
helm install metrics-server stable/metrics-server -n kube-system --set 'args={--kubelet-insecure-tls}'
```

### Prometheus

We will use Prometheus Operator

Install Prometheus Operator:

```shell
helm install prometheus-operator \
  stable/prometheus-operator \
  -n monitoring \
  --create-namespace \
  -f values/prometheus-operator.yaml
```

This example provides absolutely stripped values - just for testing.
Do not use provided values for any production setup.

### Nginx Ingress controller

```shell
helm install ingress \
  stable/nginx-ingress \
  -n ingress \
  --create-namespace
```

### Demo application deploy

The following cmd will populate Deployment, Service and Ingress controller for our test app.
The app & its Deployment were taken from the [official example](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) 
I've added only Ingress resource.

```shell
kubectl apply -f app.yaml
```

## Example walkthrough

### Install & configure Prometheus Adapter

```shell
helm install prometheus-adapter \
  stable/prometheus-adapter \
  -n monitoring \
  -f values/prometheus-adapter.yaml
```

### Apply HPA resource manifest

```shell
kubectl apply -f hpa.yaml
```

### Get HPA configuration status

```shell
kubectl describe hpa.v2beta2.autoscaling php-apache
```

### Load our app with requests

```shell
while true; do
  curl -H 'Host: apache.example.com' http://localhost
done
```

### Watch how new pods appear or disappear

```shell
kubectl get po -w
```
