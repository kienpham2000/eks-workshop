---
title: "Deploy Grafana"
date: 2018-10-14T20:54:13-04:00
weight: 15
draft: true
---

#### Deploy Grafana

We are now going to install Grafana. For this example, we are primarily using the Grafana defaults,
but we are overriding several parameters. As with Prometheus, we are setting the storage class
to gp2, admin password, configuring the datasource to point to Prometheus and creating an
[external load](https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/)
balancer for the service.

```
kubectl create namespace grafana
helm install stable/grafana \
    --name grafana \
    --namespace grafana \
    --set persistence.storageClassName="gp2" \
    --set adminPassword="EKS!sAWSome" \
    --set datasources."datasources\.yaml".apiVersion=1 \
    --set datasources."datasources\.yaml".datasources[0].name=Prometheus \
    --set datasources."datasources\.yaml".datasources[0].type=prometheus \
    --set datasources."datasources\.yaml".datasources[0].url=http://prometheus-server.prometheus.svc.cluster.local \
    --set datasources."datasources\.yaml".datasources[0].access=proxy \
    --set datasources."datasources\.yaml".datasources[0].isDefault=true \
    --set service.type=LoadBalancer
```

Run the following command to check if Grafana is deployed properly:

```sh
kubectl get all -n grafana
```
You should see similar results. They should all be Ready and Available

```text
NAME                          READY     STATUS    RESTARTS   AGE
pod/grafana-b9697f8b5-t9w4j   1/1       Running   0          2m

NAME              TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)        AGE
service/grafana   LoadBalancer   10.100.49.172   abe57f85de73111e899cf0289f6dc4a4-1343235144.us-west-2.elb.amazonaws.com   80:31570/TCP   3m


NAME                      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/grafana   1         1         1            1           2m

NAME                                DESIRED   CURRENT   READY     AGE
replicaset.apps/grafana-b9697f8b5   1         1         1         2m
```

You can get Grafana ELB URL using this command. Copy & Paste the value into browser to access Grafana web UI.

```sh
export ELB=$(kubectl get svc -n grafana grafana -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')

echo "http://$ELB"
```

When logging in, use the username **admin** and get the password hash by running the following:

```
kubectl get secret --namespace grafana grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```


{{% notice tip %}}
It can take several minutes before the ELB is up, DNS is propagated and the nodes are registered.
{{% /notice %}}

