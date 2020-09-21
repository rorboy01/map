# Monitor routers with prometheus

Following are instructions for using prometheus / grafana to monitor a Kubernetes cluster 
that is running [Apache Qpid Dispatch Router](https://qpid.apache.org/components/dispatch-router/index.html).

#### Notes
These examples are provided for developers to quickly get started with monitoring the dispatch router and are not intended for use in production environments.

These instruction will only setup monitoring of the routers in the cluster. It will not monitor any user applications that connect to those routers.

## Prerequisites / assumptions

You will need access to a Kubernetes cluster running a deployment of the router. See [qdr-operator](https://github.com/interconnectedcloud/qdr-operator) for instructions on how to install the qdr-operator into your cluster.

This guide assumes you are using a project named 'default'. If not, change all namespace definitions to your current project name.

## Router network

After the qdr-operator is installed, the router network used in this example is deployed using:

```console
$ kubectl apply -f mesh-3.yaml
```

Check to ensure the router network is available:

```console
$ kubectl rollout status deployment/example-interconnect -w -n default
```

## Deploy prometheus / grafana

### 
All of the commands needed to install prometheus/grafana monitoring are in the deploy-monitoring script: 

```console
$ ./deploy-monitoring.sh
```

If you recieve any errors, you can run the individual command separately:

### Create the prometheus deployment and alertmanager

```console
$ kubectl apply -f ./monitoring/alerting-interconnect.yaml -n default
$ kubectl apply -f ./monitoring/prometheus.yaml -n default
$ kubectl apply -f ./monitoring/alertmanager.yaml -n default
```

### Wait for Prometheus server to be ready

```console
$ kubectl rollout status deployment/prometheus -w -n default
$ kubectl rollout status deployment/alertmanager -w -n default
$ kubectl create -f ./monitoring/route-alertmanager.yaml -n default
$ kubectl create -f ./monitoring/route-prometheus.yaml -n default
```

### Prepare Grafana datasource and dashboards

```console
$ kubectl create configmap grafana-config \
    --from-file=datasource.yaml=./monitoring/dashboards/datasource.yaml \
    --from-file=grafana-dashboard-provider.yaml=./monitoring/grafana-dashboard-provider.yaml \
    --from-file=interconnect-dashboard.json=./monitoring/dashboards/interconnect-raw.json \
    --from-file=interconnect-dashboard-delayed.json=./monitoring/dashboards/interconnect-delayed.json \
    -n default

```

### Deploy grafana

```console
$ kubectl apply -f ./monitoring/grafana.yaml -n default
```

### Wait for Grafana server to be ready

```console
$ kubectl rollout status deployment/grafana -w -n default
$ kubectl create -f ./monitoring/route-grafana.yaml -n default
```
