# Deploying Monitoring Stack using Prometheus, Grafana, and Prometheus Operator

To make good and reasonable autoscaling decisions, we need observability and
monitoring on our infrastructure. In this section, you will be deploying a
monitoring stack composed of [Prometheus](https://prometheus.io/), [Grafana](https://grafana.com/), and [Prometheus Exporters](https://prometheus.io/docs/instrumenting/exporters/)
all deployed using the Prometheus Operator.

You can check out the following videos
for an introduction to prometheus, how it works, and how it can be deployed.
However, note that the installation procedure in the video is outdated, so
use the deployment outlined in this tutorial to deploy the monitoring infrastructure.

- [How Prometheus Monitoring works - Prometheus Architecture explained](https://youtu.be/h4Sl21AKiDg)
- [Setup Prometheus Monitoring on Kubernetes using Helm and Prometheus Operator](https://youtu.be/QoDqxm7ybLc)


## Helm Installation

In this step, we will be using `helm` (previously installed) to deploy the prometheus monitoring
stack on our cluster. **Run the following commands on your master node:**

```sh
# add helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
# helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm repo update
# create monitoring namespace
kubectl create ns monitoring
# install operator (it takes some time)
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring \
    --set prometheus.prometheusSpec.scrapeInterval=10s
# patch service to make it accessible
kubectl patch svc/prometheus-kube-prometheus-prometheus -n monitoring --patch "$(curl -sSL https://raw.githubusercontent.com/hamzehkhazaei/EECS4222_Project_1/master/files/prom-svc.yaml)"
# patch grafana service to change it to port 3000
kubectl patch svc/prometheus-grafana -n monitoring --type='json' -p='[{"op": "replace", "path": "/spec/ports/0/port", "value": 3000}]'
# patch grafana service to change the service to load balancer type
kubectl patch svc/prometheus-grafana -n monitoring --patch "$(curl -sSL https://raw.githubusercontent.com/hamzehkhazaei/EECS4222_Project_1/master/files/grafana-svc.yaml)"
```

Now, let's check the status of the deployment:

```console
$ kubectl get pods -n monitoring
NAME                                                     READY   STATUS    RESTARTS      AGE
prometheus-prometheus-node-exporter-f47r9                1/1     Running   0             51m
prometheus-prometheus-node-exporter-mxqwt                1/1     Running   0             51m
prometheus-kube-prometheus-operator-86cbd94f79-xgw8r     1/1     Running   0             51m
prometheus-kube-state-metrics-6db866c85b-xbh9t           1/1     Running   0             51m
prometheus-prometheus-kube-prometheus-prometheus-0       2/2     Running   0             51m
prometheus-grafana-6b78466b4-99nbh                       3/3     Running   0             31m
prometheus-prometheus-node-exporter-t79xb                1/1     Running   1 (17m ago)   51m
alertmanager-prometheus-kube-prometheus-alertmanager-0   2/2     Running   2 (17m ago)   51m
```

Wait until the `STATUS` for all pods turns into `Running`. We should be able
to see a list of services and see the `TYPE` for `prometheus-kube-prometheus-prometheus`
and `prometheus-grafana` shown as `LoadBalancer`.

```console
$ kubectl get svc -n monitoring
NAME                                      TYPE           CLUSTER-IP      EXTERNAL-IP                                 PORT(S)                         AGE
prometheus-kube-prometheus-operator       ClusterIP      10.43.7.37      <none>                                      443/TCP                         52m
prometheus-kube-prometheus-alertmanager   ClusterIP      10.43.36.233    <none>                                      9093/TCP,8080/TCP               52m
prometheus-kube-state-metrics             ClusterIP      10.43.148.252   <none>                                      8080/TCP                        52m
prometheus-prometheus-node-exporter       ClusterIP      10.43.209.132   <none>                                      9100/TCP                        52m
alertmanager-operated                     ClusterIP      None            <none>                                      9093/TCP,9094/TCP,9094/UDP      52m
prometheus-operated                       ClusterIP      None            <none>                                      9090/TCP                        52m
prometheus-kube-prometheus-prometheus     LoadBalancer   10.43.142.29    192.168.0.101,192.168.0.102,192.168.0.103   9090:32634/TCP,8080:31882/TCP   52m
prometheus-grafana                        LoadBalancer   10.43.239.4     192.168.0.101,192.168.0.102,192.168.0.103   3000:31189/TCP                  2m8s
```

Now, you should be able to open Prometheus on `http://MASTER_IP:9090` and Grafana on `http://MASTER_IP:3000`. The
default username and password for Grafana is `admin` and `prom-operator`. Next, we will try out some queries
on Prometheus to get several metrics from our deployment. 

[Next Step](06-monitoring-interaction.md) -->

## References

- [Prometheus Installation Steps](https://gitlab.com/nanuchi/youtube-tutorial-series/-/blob/master/prometheus-exporter/install-prometheus-commands.md)
- [Artifacthub: Kubernetes Prometheus Stack](https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack)
- [Github: kube-prometheus-stack helm chart](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)
- [Instructions on using Alert Manager and other CRDs](https://github.com/prometheus-operator/kube-prometheus)
- [Prometheus API Python Client](https://pypi.org/project/prometheus-api-client/)
