# platform-addons

Cluster add-ons: Helm values and Kustomize overlays. Delivered to the cluster by
**Flux**, not by anything in this repository.

**Waves:** 2–6

## Ownership

Six workstreams share this repository without merge contention because none of
them touch the same directories. `CODEOWNERS` enforces it.

| Path | Owner | Contains |
|---|---|---|
| `core/` | `@infra` | vpc-cni, ebs-csi, coredns, kube-proxy, metrics-server |
| `scaling/` | `@scaling` | Karpenter NodePools, KEDA, HPA defaults |
| `utils/` | `@utils` | cert-manager, external-dns, sealed-secrets |
| `velero/` | `@velero` | Backup schedules, restore configuration |
| `rancher/` | `@rancher` | Rancher server, downstream cluster registration |
| `istio/` | `@istio` | istiod, gateways, Kiali, mesh configuration |

Need something in another workstream's directory? Open an issue against them.
Editing it and requesting review inverts ownership.

## How an add-on reaches the cluster

```
platform-addons/  →  gitops-flux/  →  cluster
   values             HelmRelease       running
```

This repository holds **desired configuration**. `gitops-flux` holds the Flux
objects that reference it. Nothing here applies itself, and there is no
`kubectl apply` in any workflow — if you find yourself reaching for one, the
add-on is not wired into Flux yet.

## Adding an add-on

1. Create `<component>/` with `values.yaml` and per-environment overlays.
2. Add a `CODEOWNERS` entry for the path in the same pull request.
3. Open a task against `@flux` to wire the `HelmRelease` and its place in the
   dependency graph.
4. State the monthly cost delta. Every add-on consumes node capacity; several in
   this repository consume a lot.

## Layout convention

```
<component>/
├── values.yaml            base, environment-agnostic
├── values-aws.yaml        AWS-only assumptions live HERE and nowhere else
├── values-dev.yaml
├── values-prod.yaml
└── README.md              what it does, what it depends on, what it costs
```

The `values-aws.yaml` split is not cosmetic. IRSA annotations, ALB annotations
and EBS storage classes must stay isolated so an add-on can move to the home k3s
cluster without rewriting its base configuration.

## Cost discipline

The add-on set in this repository is the single largest line in the platform
bill — larger than the workloads it supports. Before adding anything:

- What does it request in CPU and memory, times replicas, times AZs?
- Does it need persistent volumes, and with what retention?
- Does it add cross-AZ traffic? Istio will, unless topology-aware routing is on.

## Standards

[Engineering Handbook](https://github.com/Ubuntu-25c-market-prep/ops-program/blob/main/docs/engineering-handbook.md) ·
[CONVENTIONS](https://github.com/Ubuntu-25c-market-prep/ops-program/blob/main/CONVENTIONS.md)
