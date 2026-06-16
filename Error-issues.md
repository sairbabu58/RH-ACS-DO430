## Error and Issues

**-> If SecuredCluster instance created 1st before InitBundle.yaml apply. 1st applyed Initbundle.yaml - Delete securedcluster instance and recreate securedcluster**
```
[student@workstation ~]$ oc describe securedcluster managed-cluster
...output omitted...
Status:
  Cluster Name:  managed-cluster
  Conditions:
    Last Transition Time:  2024-10-30T07:18:49Z
    Status:                False
    Type:                  Deployed
    Last Transition Time:  2024-10-30T07:18:49Z
    Status:                True
    Type:                  Initialized
    Last Transition Time:  2024-10-30T07:18:49Z
    Message:               3 errors occurred:
                           * init-bundle secret "sensor-tls" does not exist in namespace "stackrox", please make sure you have downloaded init-bundle secrets (from UI or with roxctl) and created corresponding resources in the correct namespace: secrets "sensor-tls" not found
                           * init-bundle secret "admission-control-tls" does not exist in namespace "stackrox", please make sure you have downloaded init-bundle secrets (from UI or with roxctl) and created corresponding resources in the correct namespace: secrets "admission-control-tls" not found
                           * init-bundle secret "collector-tls" does not exist in namespace "stackrox", please make sure you have downloaded init-bundle secrets (from UI or with roxctl) and created corresponding resources in the correct namespace: secrets "collector-tls" not found
...output omitted...
```

**-> If Central Hostname misconfigure**
```
[student@workstation ~]$ oc logs sensor-f8cdbffdd-9hhbh
...output omitted...
common/compliance: 2024/10/30 09:22:03.099664 node_inventory_handler_impl.go:138: Warn: Received NodeInventory but Central is not reachable. Requesting Compliance to resend NodeInventory later
common/sensor: 2024/10/30 09:24:54.883643 sensor.go:433: Info: Attempting connection setup (client reconciliation = true)
common/sensor: 2024/10/30 09:24:54.883881 sensor.go:493: Info: Central communication stopped: connection couldn't be re-established: checking central status failed: pinging Central: calling https://central:443/v1/ping: Get "https://central:443/v1/ping": dial tcp: lookup central on 172.31.0.10:53: no such host. Retrying after 2m42s...
common/centralclient: 2024/10/30 09:24:54.891489 grpc_connection.go:85: Error: checking central status failed: pinging Central: calling https://central:443/v1/ping: Get "https://central:443/v1/ping": dial tcp: lookup central on 172.31.0.10:53: no such host
```

**-> Storage Class Mismatch or storage issues**

```
[student@workstation ~]$ oc describe pod scanner-v4-db-7c56466b57-72zbh
...output omitted...
Events:
  Type     Reason            Age                 From               Message
  ----     ------            ----                ----               -------
  Warning  FailedScheduling  24m                 default-scheduler  0/1 nodes are available: pod has unbound immediate PersistentVolumeClaims. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.
  Warning  FailedScheduling  16m (x3 over 24m)   default-scheduler  0/1 nodes are available: pod has unbound immediate PersistentVolumeClaims. preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.
```
