---
wrapper_template: "base-docs.html"
markdown_includes:
  nav: "shared/_side-navigation.md"
context:
  title: "Scaling"
  description: Learn how various components of CDK can be horizontally scaled to meet demand or increase reliability.
keywords: juju, scaling, add-unit, ha, high availability
tags: [operating]
sidebar: k8smain-sidebar
permalink: scaling.html
layout: [base, ubuntu-com]
toc: False
---

The **Canonical Distribution of Kubernetes<sup>&reg;</sup>** has been designed to be flexible enough to efficiently run your workloads. Various components of **CDK** can be horizontally scaled to meet demand or to increase reliability, as detailed below.

## kubernetes-master

The kubernetes-master nodes act as the control plane for the cluster. **CDK** was designed with separate master nodes so that these nodes can be scaled independently of the worker units, to give better efficiency and flexibility.

Additional units can be added like so:

```bash
juju add-unit kubernetes-master
```

To add multiple units, you can also specify a numerical value

```bash
juju add-unit kubernetes-master -n 3
```

## kubernetes-worker

As the **Kubernetes** worker nodes handle the actual workloads, they usually run on machines with more resources. The resource profile of units is maintained by **Juju** using [constraints][juju-constraints].

You can check the current constraints with the command:

```bash
juju get-constraints kubernetes-worker
```

Which will return the current settings, for example:

```no-highlight
cores=4 mem=4096M root-disk=16384M
```

To create an additional kubernetes-worker unit with this resource profile, you can simply run:

```bash
juju add-unit kubernetes-worker
```

To add multiple units, you can specify a number (for example 2):

```bash
juju add-unit kubernetes-worker -n 2
```

To add additional units with specific new resource constraints, these may also be specified in the command. For example:

```bash
juju add-unit kubernetes-worker -n 2 --constraints "mem=6G cores=2"
```

... will cause two new kubernetes-worker units to be added, with at least 2 cores, 6G of memory and 16G of storage (as the existing application constraints are inherited).

To change the constraints for all _future_ units of kubernetes-worker, you can set different constraints like so:

```bash
juju set-constraints kubernetes-worker cores=2 mem=6G root-disk=16G
```

Note that in this case, any constraints you supply will **_replace all the
exisiting constraints_**, so in this example, we also include the existing
`root-disk` requirement.

<div class="p-notification--information">
  <p markdown="1" class="p-notification__response">
    <span class="p-notification__status">Note:</span>
Constraints are designed to supply the <i>minimum</i> of what is requested. This can result in the actual instances far exceeding these values, depending on the backing cloud.
  </p>
</div>

### Scaling down kubernetes-worker

Should workloads reduce, it is also possible to scale down the number of worker nodes. In order to do this safely, the node to be removed can be paused.

```bash
juju run-action kubernetes-worker/3 pause --wait
```

Pausing the worker will indicate to **Kubernetes** that it is out of service. Any workloads will be migrated to alternative worker units. You can verify this with the command:

```bash
kubectl get pod -o wide
```

The individual unit (in this example, number 3) can then be safely removed:

```bash
juju remove-unit kubernetes-worker/3
```

Note that due to the numbering system used by **Juju**, if you subsequently add additional units of this application, the numbers of any previously deleted units _will_not_ be re-used.

## etcd

The **Canonical Distribution of Kubernetes<sup>&reg;</sup>** installs a three machine cluster for etcd, which provides tolerence for a single failure. Should you wish to extend the fault tolerance, you can add additional units of etcd.

```bash
juju add-unit etcd -n 2
```

The recommended cluster-size for etcd is three, five or seven machines. Adding large numbers of additional units has a negative effect on performance due to synchronising data.

## Juju high availability

On a default deployment of the **Canonical Distribution of
Kubernetes<sup>&reg;</sup>**, there is only one controller instance, which isn't desirable for critical applications. It is possible to scale out the controller itself to prevent a single point of failure.

**Juju** supports a high availability mode to run multiple controllers with an automatic failover.

A single command will automatically create and maintain high availability for **Juju**:

```bash
juju enable-ha
```

You can verify the additional machines have been added by listing the machines in the controller model:

```bash
juju machines -m controller
```

For a more detailed guide, please refer to the [Juju high availability documentation][juju-ha].

<!-- LINKS -->

[juju-ha]: https://docs.jujucharms.com/stable/en/controllers-ha
[juju-constraints]: https://docs.jujucharms.com/stable/en/reference-constraints