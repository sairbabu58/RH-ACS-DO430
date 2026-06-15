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
