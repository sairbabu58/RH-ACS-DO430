## Error and Issues

**-> If SecuredCluster instance created 1st before InitBundle.yaml applied. 1st applyed Initbundle.yaml - Delete securedcluster instance and recreate securedcluster**
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

-> 
