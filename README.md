# Class 20250915
Q and A during the class

## Question: Pod restart after config change?
> A question about Kubernetes. If I have a running pod, and one container inside, and I change a related manifest, e.g. the pod manifest or deployment manifest. Under which manifest change circumstances is the pod restarted, and when is it not restarted?

## Answer:
> In Kubernetes, pod restart behavior depends on what fields you modify in the manifest:

### Pod is restarted when you change:

Pod spec fields (immutable after creation):

• Container image (spec.containers[].image)

• Container command/args (spec.containers[].command, spec.containers[].args)

• Environment variables (spec.containers[].env)

• Volume mounts (spec.containers[].volumeMounts)

• Resource requests/limits (spec.containers[].resources)

• Security context (spec.securityContext, spec.containers[].securityContext)

• Service account (spec.serviceAccountName)

• Node selector (spec.nodeSelector)

• Tolerations (spec.tolerations)

• Affinity rules (spec.affinity)

For Deployments specifically:
• Any change to spec.template triggers a rolling update, creating new pods and terminating old ones

### Pod is NOT restarted when you change:

Metadata fields:

• Labels (metadata.labels) - except those used by selectors

• Annotations (metadata.annotations)

Deployment fields outside the pod template:

• Replica count (spec.replicas) - scales up/down without restarting existing pods

• Deployment strategy (spec.strategy)

• Progress deadline (spec.progressDeadlineSeconds)

Service and other resource changes:

• Service port changes

• ConfigMap/Secret content (unless pod has restartPolicy configured to watch them)

### Key distinction:

• **Direct Pod edits**: Most spec fields are immutable, so you'll get an error instead of a restart

• **Deployment edits**: Changes to spec.template trigger controlled rolling updates

• **ConfigMap/Secret updates**: Pods don't automatically restart unless you use tools like Reloader or change
the ConfigMap/Secret name in the pod spec

## Verify if a pod was restarted or not: check pod age and restart count:
```
kubectl get pods -o wide -A
```
Look at the AGE column - if it's recent, the pod was recreated. The RESTARTS column shows container restarts within the
same pod.

The general rule: if it affects how the container runs (image, environment, resources), the pod restarts. If
it's just metadata or scaling, it doesn't.

## Question: When a node is tainted, are services also affected?

When I taint a worker node in such a way that all running pods are evicted, is there any other resource which gets evicted as well? For example, a service?

## Answer:
> No, tainting a node only evicts pods. Other Kubernetes resources are not directly affected by node taints.

## What gets evicted:
• **Pods only** - those without matching tolerations

## What does NOT get evicted:

• **Services** - they're cluster-scoped abstractions, not tied to specific nodes

• **ConfigMaps** - stored in etcd, not on nodes

• **Secrets** - stored in etcd, not on nodes

• **PersistentVolumes** - storage resources independent of nodes

• **Ingress** - routing rules, not node-specific

• **NetworkPolicies** - cluster networking rules

• **ServiceAccounts** - authentication resources

## However, indirect effects occur:

Services become unreachable if:

• All pods backing the service are evicted

• No other nodes can schedule replacement pods (due to tolerations/constraints)

Persistent workloads may be affected if:

• StatefulSets can't reschedule pods elsewhere

• PersistentVolumes are node-local (hostPath, local storage)
