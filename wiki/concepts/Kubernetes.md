---
type: concept
title: "Kubernetes"
created: 2026-06-28
updated: 2026-06-28
aliases:
  - k8s
  - k3s
  - kubectl
tags:
  - kubernetes
  - k8s
  - containers
  - orchestration
  - devops
  - homelab
status: developing
related:
  - "[[Docker]]"
  - "[[CICD Workflows]]"
  - "[[Terraform]]"
  - "[[Ansible]]"
  - "[[Self-Hosted Services]]"
  - "[[Proxmox]]"
---

# Kubernetes

Kubernetes (k8s) is a container orchestrator: you declare the desired state of a
workload and the cluster continuously works to make reality match. It turns a pool
of machines into one programmable platform that schedules containers, heals
failures, scales, and wires up networking/storage/config.

> [!key-insight]
> Kubernetes is a **declarative, reconciling control loop**. You don't run
> commands that imperatively "start 3 containers" — you submit an object
> ("I want 3 replicas") to the API, and controllers loop forever to make and keep
> it true. Every feature is a variation on this: watch desired state → observe
> actual state → act to close the gap. Internalise the control loop and the rest
> of k8s falls into place.

## When to reach for it (and when not)

| Use Kubernetes when… | Stick with [[Docker]] Compose when… |
|---|---|
| many services across multiple nodes | a handful of containers on one host |
| need self-healing, rolling updates, autoscaling | a single box you can babysit |
| a platform team / multiple teams share infra | one person, one app |
| portability across clouds/on-prem matters | the homelab is small and static |
| you've outgrown a single host's resources | Compose already does the job |

> [!warning] Kubernetes has real operational cost — upgrades, networking, storage,
> RBAC, and a steep learning curve. For a small homelab, **Compose or a single
> [[Docker]] host is usually the right call**; reach for k8s when you genuinely
> need multi-node scheduling or are learning it deliberately. Don't cargo-cult it.

## Architecture

A cluster = a **control plane** + one or more **worker nodes**.

### Control plane

- **kube-apiserver** — the front door. Every read/write goes through the REST API;
  `kubectl`, controllers, and kubelets all talk to it. The only component that
  talks to etcd.
- **etcd** — distributed key-value store holding *all* cluster state (the single
  source of truth). Back this up; lose it and you lose the cluster.
- **kube-scheduler** — assigns unscheduled Pods to nodes based on resources,
  affinity/anti-affinity, taints/tolerations, and constraints.
- **kube-controller-manager** — runs the built-in control loops (Deployment,
  ReplicaSet, Node, Job, …) that drive actual → desired.
- **cloud-controller-manager** — integrates with a cloud provider (load
  balancers, nodes, routes) when running managed/cloud.

### Worker nodes

- **kubelet** — node agent; takes Pod specs from the API and makes the container
  runtime run them, reports health back.
- **kube-proxy** — programs node networking (iptables/IPVS) so Service virtual IPs
  route to the right Pods.
- **container runtime** — runs containers via the CRI (containerd, CRI-O). Docker
  Engine itself was dropped as a runtime (dockershim removed in 1.24); images are
  still standard OCI, see [[Docker]].

## Core objects

The nouns you actually write YAML for:

| Object | What it is |
|---|---|
| **Pod** | smallest unit — one or more co-located containers sharing network/storage. Usually not created directly |
| **ReplicaSet** | keeps N identical Pods running (managed by Deployments) |
| **Deployment** | declarative stateless apps: rolling updates, rollbacks, scaling |
| **StatefulSet** | stable identity + ordered rollout + per-Pod storage (databases) |
| **DaemonSet** | one Pod per node (log shippers, agents, CNI) |
| **Job / CronJob** | run-to-completion / scheduled batch work |
| **Service** | stable virtual IP + DNS load-balancing across a set of Pods |
| **Ingress** | HTTP(S) routing into the cluster (host/path → Service) |
| **ConfigMap / Secret** | non-secret / secret config injected as env or files |
| **Namespace** | logical partition for multi-tenant/scoping + quotas |
| **PV / PVC / StorageClass** | persistent storage + claims + dynamic provisioning |

### A minimal Deployment + Service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels: { app: web }
  template:
    metadata:
      labels: { app: web }
    spec:
      containers:
        - name: web
          image: ghcr.io/example/web:1.2.3
          ports: [{ containerPort: 8080 }]
          resources:
            requests: { cpu: 100m, memory: 128Mi }
            limits:   { cpu: 500m, memory: 256Mi }
---
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  selector: { app: web }
  ports:
    - port: 80
      targetPort: 8080
```

`kubectl apply -f` this and the Deployment controller creates a ReplicaSet → 3
Pods; the Service gives them one stable in-cluster address.

## Networking

The model: **every Pod gets its own IP** and all Pods can reach each other without
NAT. A **CNI plugin** (Calico, Cilium, Flannel) implements that flat network.

- **Service types:**
  - `ClusterIP` (default) — internal-only virtual IP.
  - `NodePort` — exposes a port on every node.
  - `LoadBalancer` — provisions an external LB (cloud) or via **MetalLB** on bare
    metal/homelab.
- **Ingress** — an **Ingress controller** (ingress-nginx, Traefik) terminates
  HTTP(S) and routes by host/path to Services; this is where TLS lands. (Gateway
  API is the newer successor to Ingress.)
- **CoreDNS** — in-cluster DNS so Services resolve by name
  (`web.default.svc.cluster.local`).
- **NetworkPolicy** — firewall rules between Pods (default is wide open until you
  add policies). Cilium (eBPF) is a popular high-perf CNI + policy engine.

## Storage

- **Volumes** outlive container restarts within a Pod.
- **PersistentVolume (PV)** — a piece of storage; **PersistentVolumeClaim (PVC)**
  — a workload's request for some.
- **StorageClass** — enables **dynamic provisioning** (a PVC auto-creates a PV via
  a CSI driver: cloud disks, Ceph/Rook, Longhorn, NFS).
- **StatefulSets** give each replica its own stable PVC (e.g. a 3-node database).

## Config & secrets

- **ConfigMap** — non-secret config as env vars or mounted files.
- **Secret** — base64-encoded by default (NOT encrypted at rest unless you enable
  etcd encryption). For real secret management use **Sealed Secrets**, **External
  Secrets Operator**, or pull from [[Self-Hosted Services|Vaultwarden]]/Vault.
- **RBAC** — Roles/ClusterRoles + bindings gate who/what can touch the API;
  ServiceAccounts give Pods scoped API identity.

## kubectl — the daily driver

```bash
kubectl get pods -A                 # everything, all namespaces
kubectl describe pod <name>         # events + state (first stop when debugging)
kubectl logs -f deploy/web          # stream logs
kubectl apply -f manifests/         # declarative create/update
kubectl rollout status deploy/web   # watch a rolling update
kubectl rollout undo deploy/web     # roll back
kubectl scale deploy/web --replicas=5
kubectl exec -it <pod> -- sh        # shell into a container
kubectl get events --sort-by=.lastTimestamp
```

> [!note] Prefer `kubectl apply -f` (declarative) over imperative
> `kubectl create/run` for anything you keep — it's the same desired-state model
> the rest of k8s uses, and it's diff-able in git (see GitOps below).

## Packaging & deployment

- **Helm** — the de facto package manager; **charts** are templated manifests with
  `values.yaml` overrides. `helm install`/`upgrade`/`rollback`. Great for
  installing off-the-shelf apps; templating can get gnarly for your own.
- **Kustomize** — template-free overlays (built into `kubectl -k`); base manifests
  + per-environment patches. Cleaner for your own apps.
- **GitOps** — git is the source of truth; **Argo CD** or **Flux** continuously
  reconcile the cluster to a repo. Pairs naturally with the k8s control loop and
  with [[CICD Workflows]] (CI builds images, GitOps deploys them).

## Distributions & where to run it

| Flavor | Fit |
|---|---|
| **k3s** | lightweight CNCF-certified single binary — ideal for homelab/edge/VMs |
| **k0s** / **RKE2** | lightweight/hardened alternatives (RKE2 is FIPS/security-focused) |
| **kubeadm** | the reference DIY bootstrapper for vanilla clusters |
| **Talos Linux** | immutable, API-managed OS purpose-built for k8s |
| **kind / minikube** | throwaway local clusters for dev/CI |
| **EKS / GKE / AKS** | managed control plane in the cloud (no etcd babysitting) |

> [!note] Homelab pattern: **k3s** across a few [[Proxmox]] VMs (one server + a
> couple agents), **MetalLB** for LoadBalancer IPs, **Traefik/ingress-nginx** for
> ingress, **Longhorn** for storage, reached over [[Tailscale]]. Provision the VMs
> with [[Terraform]], configure with [[Ansible]] / cloud-init. For most homelabs a
> single [[Docker]] host is still simpler — run k3s when you want the platform or
> the practice. Catalog: [[Self-Hosted Services]].

## Strengths / trade-offs

**Strengths**
- Self-healing, rolling updates/rollbacks, horizontal autoscaling out of the box.
- One declarative API for compute, networking, storage, config — and it's
  portable across clouds and bare metal.
- Enormous ecosystem (Helm charts, operators, CNCF tooling) and a strong GitOps
  story.

**Trade-offs**
- Steep learning curve and real operational burden (upgrades, etcd, networking,
  storage, RBAC).
- Easy to over-adopt — a lot of workloads are better served by [[Docker]] Compose
  or a single host.
- Many moving parts to secure: Secrets aren't encrypted by default, NetworkPolicy
  is opt-in, RBAC needs deliberate design.
