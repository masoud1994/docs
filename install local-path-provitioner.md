# install local-path-provitioner
* https://github.com/rancher/local-path-provisioner
* https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/#changing-the-default-storageclass

In this setup, the directory /var/volumes will be used across all the nodes as the path for provisioning (a.k.a, store the persistent volume data). The provisioner will be installed in local-path-storage namespace by default.
```
wget https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.30/deploy/local-path-storage.yaml
```

## Set the default StorageClass 
add storageclass.kubernetes.io/is-default-class: "true" as a annotation in the local-path-storage.yaml
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-path
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: rancher.io/local-path
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
```

change default directory to /var/volumes for create pvc
```
kind: ConfigMap
apiVersion: v1
metadata:
  name: local-path-config
  namespace: local-path-storage
data:
  config.json: |-
    {
            "nodePathMap":[
            {
                    "node":"DEFAULT_PATH_FOR_NON_LISTED_NODES",
                    "paths":["/var/volumes"]
            }
            ]
    }
```

## insatll local-path
```

kubectl apply -f local-path-storage.yaml 
```
after installation, you should see something like the following:
```
$ kubectl -n local-path-storage get pod
NAME                                     READY     STATUS    RESTARTS   AGE
local-path-provisioner-d744ccf98-xfcbk   1/1       Running   0          7m
```

# Changing the default StorageClass 
List the StorageClasses in your cluster:
```
kubectl get storageclass
The output is similar to this:

NAME                 PROVISIONER               AGE
standard (default)   kubernetes.io/gce-pd      1d
gold                 kubernetes.io/gce-pd      1d
```
The default StorageClass is marked by (default).

Mark the default StorageClass as non-default:

The default StorageClass has an annotation storageclass.kubernetes.io/is-default-class set to true. Any other value or absence of the annotation is interpreted as false.

To mark a StorageClass as non-default, you need to change its value to false:
```
kubectl patch storageclass standard -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'
```
where standard is the name of your chosen StorageClass.

Mark a StorageClass as default:

Similar to the previous step, you need to add/set the annotation storageclass.kubernetes.io/is-default-class=true.
```
kubectl patch storageclass gold -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```
Please note that at most one StorageClass can be marked as default. If two or more of them are marked as default, a PersistentVolumeClaim without storageClassName explicitly specified cannot be created.

Verify that your chosen StorageClass is default:
```
kubectl get storageclass
The output is similar to this:

NAME             PROVISIONER               AGE
standard         kubernetes.io/gce-pd      1d
gold (default)   kubernetes.io/gce-pd      1d
```