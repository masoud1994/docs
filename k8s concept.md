# k8s concept

ReplicasSet , statefulSet, DaemonSet , service , configMap
test clair with docker
and create helm-chart for test
create helm-chart for clair

* https://kubernetes.io/docs/concepts/workloads/pods/
* https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/
* https://kubernetes.io/docs/concepts/storage/persistent-volumes/
https://kubernetes.io/docs/concepts/services-networking/service/#headless-services
* https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/
* https://kubernetes.io/docs/concepts/configuration/overview/
* https://kubernetes.io/docs/reference/kubectl/
* https://kubernetes.io/docs/reference/using-api/
* https://kubernetes.io/docs/tasks/manage-kubernetes-objects/imperative-command/
* https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/
* https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/#add-a-label-to-a-node
* https://kubernetes.io/docs/reference/labels-annotations-taints/
* https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/
* https://kubernetes.io/docs/tasks/configure-pod-container/enforce-standards-namespace-labels/
* https://kubernetes.io/blog/2021/06/21/writing-a-controller-for-pod-labels/
* https://kubernetes.io/docs/concepts/overview/kubernetes-api/
* https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/
* https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/#updating-statefulsets
* https://kubernetes.io/docs/concepts/security/linux-kernel-security-constraints/#privileged-containers
* https://kubernetes.io/docs/concepts/security/linux-kernel-security-constraints/
* https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
* https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/
* https://kubernetes.io/docs/concepts/workloads/management/#canary-deployments




```

# In a namespace
kubectl api-resources --namespaced=true

# Not in a namespace
kubectl api-resources --namespaced=false
```

Pod templates in Daemonset, Deployment, Job

Whereas 

StatefulSets currently require a Headless Service to be responsible for the network identity of the Pods. You are responsible for creating this Service.