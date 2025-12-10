**Central**
The number of concurrent console users, which mostly affects its overall CPU usage and might require an increase in CPU requests.

**Scanner**
Many unique images, which is addressed by increasing the number of replicas of the Scanner application. Scanner supports autoscaling by default.

**Sensor and Admission Controller**
These services must run in each secured cluster and their memory usage is mostly affected by the number of deployments.

**Collector**
Collector is affected only by the amount of process and network activity on each compute node. However, Collector's default sizing should be appropriate for most use cases. When the amount of active workload increases, 
the number of compute nodes in the cluster also increases. Because each node runs one replica of a Collector pod, the maximum load on a single Collector pod remains manageable.
