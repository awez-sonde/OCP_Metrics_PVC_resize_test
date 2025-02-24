# OCP Metrics PVC Resize test
This project is to verify if PVC resizing has any effect on Openshift metrics


## Adding persistent storage to openshift metrics

```
[root@test ipi]# oc project openshift-monitoring
Now using project "openshift-monitoring" on server "https://api.asonde-acm-ipi.vmware.tamlab.rdu2.redhat.com:6443".

[root@test ipi]# oc edit cm cluster-monitoring-config -o yaml

```


```
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  config.yaml: |
    enableUserWorkload: true
    prometheusK8s:
      volumeClaimTemplate:
       spec:
         storageClassName: thin-csi
         volumeMode: Filesystem
         resources:
           requests:
             storage: 40Gi
kind: ConfigMap
metadata:
  creationTimestamp: "2024-12-22T12:37:33Z"
  name: cluster-monitoring-config
  namespace: openshift-monitoring
  resourceVersion: "89472349"
  uid: a8312d33-eec6-4401-b75f-a268909cd444


```

Addition of storage was around 11.45 AM approx.

Checking the dashboard, we can see that the data before 11.45 AM hours is gone, this is because new data is written on the new PVC.

<img width="2716" alt="image" src="https://github.com/user-attachments/assets/62900651-cdb4-4d2c-add1-0140c5794123" />

Checking the PVC that got created

```
[root@test ipi]# oc get pvc
NAME                                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
prometheus-k8s-db-prometheus-k8s-0   Bound    pvc-4908dc87-e171-4531-bab6-68efd705fbc5   40Gi       RWO            thin-csi       4h5m
prometheus-k8s-db-prometheus-k8s-1   Bound    pvc-c15c98b4-5cba-4e17-805c-340f1e3b2e25   40Gi       RWO            thin-csi       4h5m
```


## Expanding the PVC

### Expanding the first PVC from 40GiB to 60 GiB

<img width="2536" alt="image" src="https://github.com/user-attachments/assets/23d788b5-6e98-41f5-9a1c-0780e6f6d91b" />


We need to restart the pod, as can be seen in the below Message

<img width="2257" alt="image" src="https://github.com/user-attachments/assets/8c1c6a9b-612e-4b9f-9df5-482d176bba03" />

<img width="2714" alt="image" src="https://github.com/user-attachments/assets/9b263176-19a3-495e-ae54-b3bac429e261" />

Once the pod is restarted , the changes can be seen in Prometheus Pod directly

<img width="874" alt="image" src="https://github.com/user-attachments/assets/1215aec8-d856-4fd7-9b80-408cd0409204" />


#### Verifying the dashboard after first PVC expansion

We can still see the metrics from 11.45AM

<img width="2498" alt="image" src="https://github.com/user-attachments/assets/057ceccb-789a-4cd9-9b24-5c71bed44ce7" />


### Expanding the second PVC

<img width="2694" alt="image" src="https://github.com/user-attachments/assets/d4e6020f-c92b-4e75-92c2-854a84402ac2" />

Restarting the pod

```
[root@test ipi]# oc delete po prometheus-k8s-1
pod "prometheus-k8s-1" deleted
```

```
[root@test ipi]# oc get po
NAME                                                     READY   STATUS     RESTARTS   AGE
alertmanager-main-0                                      6/6     Running    12         103d
alertmanager-main-1                                      6/6     Running    12         103d
cluster-monitoring-operator-7d6ffc9477-w55fw             1/1     Running    2          109d
kube-state-metrics-c7c8f4657-c6bvq                       3/3     Running    6          103d
monitoring-plugin-794b464cf4-bgpq6                       1/1     Running    2          103d
monitoring-plugin-794b464cf4-vmlh6                       1/1     Running    2          103d
node-exporter-2pbnq                                      2/2     Running    4          103d
node-exporter-4566x                                      2/2     Running    4          103d
node-exporter-88fpn                                      2/2     Running    4          103d
node-exporter-f97z6                                      2/2     Running    4          103d
node-exporter-jxgrw                                      2/2     Running    4          103d
node-exporter-ntwlc                                      2/2     Running    4          103d
node-exporter-rgtc2                                      2/2     Running    10         103d
node-exporter-v97j4                                      2/2     Running    4          103d
node-exporter-xz6hv                                      2/2     Running    4          103d
openshift-state-metrics-7f8ff767bd-vfj94                 3/3     Running    6          103d
prometheus-adapter-d87677bf9-rxhtr                       1/1     Running    0          2d5h
prometheus-adapter-d87677bf9-v8fgq                       1/1     Running    0          2d5h
prometheus-k8s-0                                         6/6     Running    0          4h12m
prometheus-k8s-1                                         0/6     Init:0/1   0          4s
prometheus-operator-5d6d65fbb8-vmr6p                     2/2     Running    4          103d
prometheus-operator-admission-webhook-565688df86-4drbz   1/1     Running    2          109d
prometheus-operator-admission-webhook-565688df86-js5fs   1/1     Running    2          109d
telemeter-client-85f44f68cf-t2xrs                        3/3     Running    6          103d
thanos-querier-64b6c675cd-dlr8l                          6/6     Running    6          63d
thanos-querier-64b6c675cd-llzwl                          6/6     Running    6          63d



[root@test ipi]# oc get po
NAME                                                     READY   STATUS    RESTARTS   AGE
alertmanager-main-0                                      6/6     Running   12         103d
alertmanager-main-1                                      6/6     Running   12         103d
cluster-monitoring-operator-7d6ffc9477-w55fw             1/1     Running   2          109d
kube-state-metrics-c7c8f4657-c6bvq                       3/3     Running   6          103d
monitoring-plugin-794b464cf4-bgpq6                       1/1     Running   2          103d
monitoring-plugin-794b464cf4-vmlh6                       1/1     Running   2          103d
node-exporter-2pbnq                                      2/2     Running   4          103d
node-exporter-4566x                                      2/2     Running   4          103d
node-exporter-88fpn                                      2/2     Running   4          103d
node-exporter-f97z6                                      2/2     Running   4          103d
node-exporter-jxgrw                                      2/2     Running   4          103d
node-exporter-ntwlc                                      2/2     Running   4          103d
node-exporter-rgtc2                                      2/2     Running   10         103d
node-exporter-v97j4                                      2/2     Running   4          103d
node-exporter-xz6hv                                      2/2     Running   4          103d
openshift-state-metrics-7f8ff767bd-vfj94                 3/3     Running   6          103d
prometheus-adapter-d87677bf9-rxhtr                       1/1     Running   0          2d5h
prometheus-adapter-d87677bf9-v8fgq                       1/1     Running   0          2d5h
prometheus-k8s-0                                         6/6     Running   0          4h12m
prometheus-k8s-1                                         5/6     Running   0          15s
prometheus-operator-5d6d65fbb8-vmr6p                     2/2     Running   4          103d
prometheus-operator-admission-webhook-565688df86-4drbz   1/1     Running   2          109d
prometheus-operator-admission-webhook-565688df86-js5fs   1/1     Running   2          109d
telemeter-client-85f44f68cf-t2xrs                        3/3     Running   6          103d
thanos-querier-64b6c675cd-dlr8l                          6/6     Running   6          63d
thanos-querier-64b6c675cd-llzwl                          6/6     Running   6          63d
```

Checking the second pod for changes ( this time using CLI)

```
[root@test ipi]# oc rsh prometheus-k8s-1 df -h 
Filesystem      Size  Used Avail Use% Mounted on
overlay         120G   18G  102G  16% /
tmpfs            64M     0   64M   0% /dev
shm              64M     0   64M   0% /dev/shm
tmpfs           1.6G   72M  1.5G   5% /etc/hostname
/dev/sdb         59G  983M   55G   2% /prometheus
/dev/sda4       120G   18G  102G  16% /etc/hosts
tmpfs           6.7G  228K  6.7G   1% /etc/prometheus/config_out
tmpfs           6.7G   12K  6.7G   1% /etc/prometheus/certs
tmpfs           6.7G  4.0K  6.7G   1% /etc/prometheus/secrets/kube-rbac-proxy
tmpfs           6.7G  8.0K  6.7G   1% /etc/prometheus/secrets/prometheus-k8s-thanos-sidecar-tls
tmpfs           6.7G  4.0K  6.7G   1% /etc/prometheus/web_config/web-config.yaml
tmpfs           6.7G  4.0K  6.7G   1% /etc/prometheus/secrets/prometheus-k8s-proxy
tmpfs           6.7G  8.0K  6.7G   1% /etc/prometheus/secrets/prometheus-k8s-tls
tmpfs           6.7G  8.0K  6.7G   1% /etc/prometheus/secrets/metrics-client-certs
tmpfs           6.7G   20K  6.7G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs           3.9G     0  3.9G   0% /proc/acpi
tmpfs           3.9G     0  3.9G   0% /proc/scsi
tmpfs           3.9G     0  3.9G   0% /sys/firmware
```

#### Verifying the dashboard after second PVC expansion

We can still see the data from 11.45 AM

<img width="2733" alt="image" src="https://github.com/user-attachments/assets/683b86fa-6b35-453d-8524-1786cc4abd27" />


The above shows that extending the PVC should NOT effect the prometheus data.
The PVC's were extended at around 4.00PM but the data from 11.45AM to 4.00PM is intact.







