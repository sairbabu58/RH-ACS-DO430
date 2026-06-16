## RHACM operator installation
## ACS Central configuration - Customization
## How to get default username/password
## How to access ACS webui

## Install Secured Cluster
## Secure Cluster - custom resources
## How to generate InitBundles
## How to add hub cluster to RHACM 


**Central**

The number of concurrent console users, which mostly affects its overall CPU usage and might require an increase in CPU requests.

RHACS Central services consist of two components, Central and Scanner; the latter is also known as StackRox Scanner. 

**Scanner**

Many unique images, which is addressed by increasing the number of replicas of the Scanner application. Scanner supports autoscaling by default.

**Sensor and Admission Controller**

These services must run in each secured cluster and their memory usage is mostly affected by the number of deployments.

**Collector**

Collector is affected only by the amount of process and network activity on each compute node. However, Collector's default sizing should be appropriate for most use cases. When the amount of active workload increases, 
the number of compute nodes in the cluster also increases. Because each node runs one replica of a Collector pod, the maximum load on a single Collector pod remains manageable.

**Central Components:**

- Central 
- Scanner

**Managed Cluster Components:**

- Sensor
- Admission Controller
- Collector


## Install RHACS Operator

```
-> Operator HuB
-> Redhat ACS
-> Install a per the defaault setting
```

### Create Central Cluster

```
-> oc new-project stackrox
-> Installed Opeator
-> Central
-> Create Central
-> From View - then yaml view
```
**Minimal configuration**
```
apiVersion: platform.stackrox.io/v1alpha1
kind: Central
metadata:
  name: central
  namespace: stackrox
spec:
  central:
    exposure:
      route:
        enabled: true           
  scanner:
    analyzer:
      scaling:
        autoScaling: Enabled    
        minReplicas: 1
        maxReplicas: 3
  scannerV4:
    indexer:
      scaling:
        autoScaling: Enabled    
        minReplicas: 1
        maxReplicas: 3
    matcher:
      scaling:
        autoScaling: Disabled   
        replicas: 2
    scannerComponent: Enabled
```
The default deployment mode for the RHACS control plane is Online, which means that it works with external services from Red Hat to periodically download and update vulnerability definitions and collector eBPF modules. To deploy it in offline mode, you must set the connectivityPolicy attribute to Offline.
```
spec:
  egress:
    connectivityPolicy: Offline
  ...output omitted...
  central:
    telemetry:
      enabled: false
```
You can enable OpenShift monitoring integration by setting the monitoring attribute and configuring monitoring endpoints of any services to expose.
```
spec:
  monitoring:
    openshift:
      enabled: true
  ...output omitted...
  central:
    monitoring:
      exposeEndpoint: Enabled
  ...output omitted...
  scanner:
    monitoring:
      exposeEndpoint: Enabled
  ...output omitted...
  scannerV4:
    monitoring:
      exposeEndpoint: Disabled
```
Customize Central Hostname

```
spec:
  central:
    exposure:
      route:
        enabled: true
        host: custom-hostname.my.cluster.com
```
DB Storage 
```
spec:
  central:
    db:
      persistence:
        persistentVolumeClaim:
          claimName: central-db-claim     
          size: 200Gi                     
          storageClassName: ssd-storage 
```
#### Access Central URL WebUI
```
-> oc get route -n strackrox
-> Over Browser https://url
```

#### Default Password for Central Login 

```
[user@host ~]$ oc -n stackrox extract secrets/central-htpasswd \
  --keys password --to -
# password
y3sIavTsas5sgdsgdgsy568567856LSmdG8
```


#### Central-config.yaml

```
---
apiVersion: platform.stackrox.io/v1alpha1
kind: Central
metadata:
  name: central
  namespace: stackrox
spec:
  monitoring:
    openshift:
      enabled: true
  network:
    policies: Enabled
  central:
    notifierSecretsEncryption:
      enabled: false
    exposure:
      loadBalancer:
        enabled: false
        port: 443
      nodePort:
        enabled: false
      route:
        enabled: true
    resources:
      requests:
        memory: 512Mi
        cpu: 750m
      limits:
        memory: 2Gi
        cpu: 2
    telemetry:
      enabled: true
    db:
      isEnabled: Default
      persistence:
        persistentVolumeClaim:
          claimName: central-db
          size: 10Gi
      resources:
        requests:
          memory: 512Mi
          cpu: 750m
        limits:
          memory: 4Gi
          cpu: 2
  egress:
    connectivityPolicy: Online
  scannerV4:
    db:
      resources:
        requests:
          memory: 512Mi
          cpu: 500m
        limits:
          memory: 4Gi
          cpu: 2
      persistence:
        persistentVolumeClaim:
          storageClassName: lvms-vg1
          size: 35Gi
    indexer:
      scaling:
        autoScaling: Disabled
        replicas: 1
      resources:
        requests:
          memory: 512Mi
          cpu: 500m
        limits:
          memory: 1Gi
          cpu: 1
    matcher:
      scaling:
        autoScaling: Disabled
        replicas: 1
      resources:
        requests:
          memory: 512Mi
          cpu: 500m
        limits:
          memory: 4Gi
          cpu: 4
    scannerComponent: Enabled
  scanner:
    analyzer:
      scaling:
        autoScaling: Disabled
        replicas: 1
      resources:
        requests:
          memory: 512Mi
          cpu: 500m
        limits:
          memory: 2Gi
          cpu: 2
    db:
      resources:
        requests:
          memory: 1Gi
          cpu: 500m
        limits:
          memory: 4Gi
          cpu: 2
...
```
```
[student@workstation ~]$ oc get po,pvc,deploy,svc,route
NAME                                      READY   STATUS    RESTARTS   AGE
pod/central-66457d6f9d-wqvc4              1/1     Running   0          9m17s
pod/central-db-6dc57777f7-l7799           1/1     Running   0          9m17s
pod/config-controller-775f7bbb47-vgh9c    1/1     Running   0          9m17s
pod/scanner-db-5ff649f95-vdgrp            1/1     Running   0          9m17s
pod/scanner-ffdbc688b-jwcpl               1/1     Running   0          9m17s
pod/scanner-v4-db-74c48ddb77-fbttg        1/1     Running   0          9m17s
pod/scanner-v4-indexer-5d8fd89b96-vqxv4   1/1     Running   0          9m17s
pod/scanner-v4-matcher-55f45577cb-vg4wm   1/1     Running   0          9m17s

NAME                                  STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/central-db      Bound    pvc-f56c18b6-4ebe-4085-90cd-2b8990786213   10Gi       RWO            nfs-storage    <unset>                 9m28s
persistentvolumeclaim/scanner-v4-db   Bound    pvc-c8f07cee-3ff7-45c6-9aa4-c2364430c0ea   35Gi       RWO            lvms-vg1       <unset>                 9m17s

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/central              1/1     1            1           9m17s
deployment.apps/central-db           1/1     1            1           9m17s
deployment.apps/config-controller    1/1     1            1           9m17s
deployment.apps/scanner              1/1     1            1           9m17s
deployment.apps/scanner-db           1/1     1            1           9m17s
deployment.apps/scanner-v4-db        1/1     1            1           9m17s
deployment.apps/scanner-v4-indexer   1/1     1            1           9m17s
deployment.apps/scanner-v4-matcher   1/1     1            1           9m17s

NAME                         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
service/central              ClusterIP   172.30.250.221   <none>        443/TCP,9091/TCP    9m17s
service/central-db           ClusterIP   172.30.147.209   <none>        5432/TCP            9m17s
service/scanner              ClusterIP   172.30.201.18    <none>        8080/TCP,8443/TCP   9m17s
service/scanner-db           ClusterIP   172.30.75.55     <none>        5432/TCP            9m17s
service/scanner-v4-db        ClusterIP   172.30.12.59     <none>        5432/TCP            9m17s
service/scanner-v4-indexer   ClusterIP   None             <none>        8443/TCP,9091/TCP   9m17s
service/scanner-v4-matcher   ClusterIP   None             <none>        8443/TCP,9091/TCP   9m17s

NAME                                    HOST/PORT                                PATH   SERVICES   PORT    TERMINATION   WILDCARD
route.route.openshift.io/central        central-stackrox.apps.ocp4.example.com          central    https   passthrough   None
route.route.openshift.io/central-mtls   central.stackrox                                central    https   passthrough   None
[student@workstation ~]$ 
```

#### Get the Central console username and password

```
$ oc extract secret/central-htpasswd --keys password --to -
$ oc get route -n stackrox
$ https://url - over browser
```


## Secure Cluster Process

#### How to generate INITBundles
```
-> open the RHACS WebUI
  -> Cluster
  -> Create Init Bundle
    -> Name: Install-import-cluster
    -> Platform of Secure cluster:
        - OpenShift
    -> Download
$ oc apply -f file.yaml
```

#### Now Install Secure Cluster
```
-> Go to OperatorHuB
-> (Install RHACS Operator) RHACS Operator
-> Secure Cluster
-> Form View / Yaml view
-> Add the Endpoints for your Central cluster https://central-stackrox.apps.ocp4.example.com:443 [## https optional]
```
#### Custom Secure Cluster yaml
```
---
apiVersion: platform.stackrox.io/v1alpha1
kind: SecuredCluster
metadata:
  name: managed-cluster
  namespace: stackrox
spec:
  centralEndpoint: "central-stackrox.apps.ocp4.example.com:443"
  clusterName: central-cluster
  clusterLabels:
    cluster: central-cluster
  monitoring:
    openshift:
      enabled: true
  auditLogs:
    collection: Auto
  sensor:
    resources:
      requests:
        memory: 512Mi
        cpu: 250m
      limits:
        memory: 4Gi
        cpu: 2
  admissionControl:
    listenOnCreates: true
    listenOnUpdates: true
    listenOnEvents: true
    replicas: 1
    bypass: BreakGlassAnnotation
    contactImageScanners: ScanIfMissing
    timeoutSeconds: 10
    resources:
      requests:
        memory: 100Mi
        cpu: 50m
      limits:
        memory: 500Mi
        cpu: 500m
  scannerV4:
    db:
      persistence:
        persistentVolumeClaim:
          claimName: scanner-v4-db
      resources:
        requests:
          memory: 256Mi
          cpu: 200m
        limits:
          memory: 4Gi
          cpu: 2
    indexer:
      scaling:
        autoScaling: Disabled
        replicas: 1
      resources:
        requests:
          memory: 512Mi
          cpu: 250m
        limits:
          memory: 6Gi
          cpu: 4
    scannerComponent: AutoSense
  scanner:
    analyzer:
      scaling:
        autoScaling: Disabled
        replicas: 1
      resources:
        requests:
          memory: 512Mi
          cpu: 250m
        limits:
          memory: 2Gi
          cpu: 1
    db:
      resources:
        requests:
          memory: 256Mi
          cpu: 200m
        limits:
          memory: 2Gi
          cpu: 1
  perNode:
    collector:
      collection: CORE_BPF
      resources:
        requests:
          memory: 200Mi
          cpu: 60m
        limits:
          memory: 1Gi
          cpu: 750m
    compliance:
      resources:
        requests:
          memory: 10Mi
          cpu: 10m
        limits:
          memory: 512Mi
          cpu: 1
    nodeInventory:
      resources:
        requests:
          memory: 10Mi
          cpu: 10m
        limits:
          memory: 512Mi
          cpu: 1
...
```
