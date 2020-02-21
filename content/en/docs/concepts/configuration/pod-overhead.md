---
reviewers:
- dchen1107
- egernst
- tallclair
title: Pod Overhead
content_template: templates/concept
weight: 20
---

{{% capture overview %}}

{{< feature-state for_k8s_version="v1.18" state="beta" >}}


When you run a Pod on a Node, the Pod itself takes an amount of system resources. These
resources are additional to the resources needed to run the container(s) inside the Pod.
_Pod Overhead_ is a feature for accounting for the resources consumed by the pod infrastructure
on top of the container requests & limits.


{{% /capture %}}


{{% capture body %}}

## Pod Overhead

In Kubernetes, the pod's overhead is set at
[admission](/docs/reference/access-authn-authz/extensible-admission-controllers/#what-are-admission-webhooks)
time according to the overhead associated with the pod's
[RuntimeClass](/docs/concepts/containers/runtime-class/).

When Pod Overhead is enabled, the overhead is considered in addition to the sum of container
resource requests when scheduling a pod. Similarly, Kubelet will include the pod overhead when sizing
the pod cgroup, and when carrying out pod eviction ranking.

### Set Up

You need to make sure that the `PodOverhead`
[feature gate](/docs/reference/command-line-tools-reference/feature-gates/) is enabled (it is on by default as of 1.18)
across your cluster. This means:

- in {{< glossary_tooltip text="kube-scheduler" term_id="kube-scheduler" >}}
- in {{< glossary_tooltip text="kube-apiserver" term_id="kube-apiserver" >}}
- in the {{< glossary_tooltip text="kubelet" term_id="kubelet" >}} on each Node
- in any custom API servers that use feature gates

{{< note >}}
Users who can write to RuntimeClass resources are able to have cluster-wide impact on
workload performance. You can limit access to this ability using Kubernetes access controls.
See [Authorization Overview](/docs/reference/access-authn-authz/authorization/) for more details.
{{< /note >}}


### Usage example

To use the `PodOverhead` feature, a `RuntimeClass` must be utilized which defines the `Overhead` field. As an example,
if you were to use Kata Containers, you could register a RuntimeClass as follows:

```yaml
---
kind: RuntimeClass
apiVersion: node.k8s.io/v1beta1
metadata:
    name: kata-fc
handler: kata-fc
overhead:
    podFixed:
        memory: "120Mi"
	cpu: "250m"
```

Workloads which are created which specify the `kata-fc` RuntimeClass handler will take the memory and
cpu overheads into account for resource quota calculations, node scheduling, as well as pod cgroup sizing.

For the given example workload,

```yaml
```

More specifically, at admission time the RuntimeClass admission controller will update the workload's PodSpec to
include the Overhead field. If the PodSpec already has this field defined, the pod will be rejected.

 If a namespace resource quota is defined, the sum of container requests as well as the
overhead field are counted.

When kube-scheduler runs, the overhead as well as sum of containe requests are taken into account when identifying
an appropriate node binding.

Once scheduled for the node, as part of the pod creation process, Kubelet will create a Pod cgroup (for non burstable QoS)
and this will be sized as he sum of container (requests or limits) plus the overhead defined in the PodSpec. 

### Observability

Metrics are available in [kube-state-metrics](link) to help identify when PodOverhead is being utilized and to help observe
stability of workloads running with a defined Overhead.

[ ] and [ ].



{{% /capture %}}

{{% capture whatsnext %}}

* [RuntimeClass](/docs/concepts/containers/runtime-class/)
* [PodOverhead Design](https://github.com/kubernetes/enhancements/blob/master/keps/sig-node/20190226-pod-overhead.md)

{{% /capture %}}
