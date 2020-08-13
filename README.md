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

### Let's check current pods for our application

```shell
kubetl get po
```

Only one pod should exist.

### Load our app with requests

Let's load our app with some requests for new Ingress metrics to appear

```shell
while true; do
  curl -H 'Host: apache.example.com' http://localhost
done
```

### Install & configure Prometheus Adapter

```shell
helm install prometheus-adapter \
  stable/prometheus-adapter \
  -n monitoring \
  -f values/prometheus-adapter.yaml
```

### Check if new metrics appeared inside Metrics API

```shell
kubectl get --raw '/apis/custom.metrics.k8s.io/v1beta1' | jq
```

At this moment, if you are not familiar with the Metrics API itself you do not know how to debug further. It is not a problem at all, let's make K8s to show us an example.

### Apply HPA resource manifest

K8s' HPA controller will start to run API requests to Metrics API as soon as we will apply our first HPA resource.

```shell
kubectl apply -f hpa.yaml
```

### Get HPA configuration status

```shell
kubectl describe hpa.v2beta2.autoscaling php-apache
```

### Check Prometheus Adapter logs

We will immediately see the actual requests inside Prometheus Adapter logs.

```shell
kubectl logs -n monitoring -l app=prometheus-adapter -f
```

### Let's try to run them manually too

```shell
kubectl get --raw '/apis/custom.metrics.k8s.io/v1beta1/namespaces/default/ingresses.networking.k8s.io/php-apache/requests-per-second' | jq
```

### Watch how new pods appear or disappear

If there are no errors in the configuration you will see your result soon:

```shell
kubectl get po -w
```
