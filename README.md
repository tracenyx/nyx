<div align="center">

<img src="docs/assets/mainlogo.png" alt="Nyx" width="120"/>

# Nyx

**AI-powered Kubernetes network security & observability**

Zero trust from the kernel up. One policy model for Linux and Windows.

[![License](https://img.shields.io/badge/license-Proprietary-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-Linux%20%7C%20Windows-brightgreen.svg)]()
[![Kubernetes](https://img.shields.io/badge/kubernetes-1.26%2B-blue.svg)]()
[![Early Access](https://img.shields.io/badge/status-Early%20Access-orange.svg)](https://tracenyx.ai)

[Website](https://tracenyx.ai) · [Docs](https://docs.tracenyx.ai) · [Blog](https://blog.tracenyx.ai) · [Get Started Free](https://tracenyx.ai#contact)

</div>

---

## The Problem

Most Kubernetes network security tools work on Linux only. Your Windows nodes run unprotected — different tool, different policy model, no unified visibility.

Even on Linux, the default Kubernetes NetworkPolicy is limited. No deny rules. No FQDN-based egress control. No observability. No AI. Just basic allow rules and hope.

| | Calico | Cilium | **Nyx** |
|---|---|---|---|
| Linux enforcement | ✅ | ✅ | ✅ |
| Windows enforcement | Partial | ❌ | ✅ |
| FQDN egress filtering | ✅ | ✅ | ✅ |
| Unified Linux + Windows UI | ❌ | ❌ | ✅ |
| AI observability | ❌ | ❌ | ✅ |
| Natural language policy | ❌ | ❌ | ✅ |
| Cluster footprint | 20+ components | 15+ components | **2 components** |

---

## How It Works

Nyx enforces policy at the kernel level — eBPF on Linux, Windows Filtering Platform on Windows. No sidecars. No proxies. No Envoy dependency.

```
kubectl apply -f policy.yaml
        │
        ▼
Nyx Control Plane (1 Deployment)
        │
        ├──────────────────────────┐
        ▼                          ▼
Linux Nodes                  Windows Nodes
eBPF / TC hooks              WFP callout driver
Kernel-level enforcement     Kernel-level enforcement
        │                          │
        └──────────┬───────────────┘
                   ▼
        Tracenyx Cloud (your region)
        Flow logs · AI analysis · Dashboard · Alerts
```

Two components run in your cluster — one DaemonSet for enforcement, one Deployment for the admission webhook. That's it.

---

## Quick Start

### Prerequisites

- Kubernetes 1.26+
- Linux nodes: kernel 5.15+
- Windows nodes: Windows Server 2022+

### Install

```bash
helm repo add tracenyx https://charts.tracenyx.ai
helm repo update

helm install nyx tracenyx/nyx \
  --namespace nyx-system \
  --create-namespace \
  --set license.key=YOUR_SCOUT_KEY
```

Get your free Scout key → [tracenyx.ai](https://tracenyx.ai)

### Verify

```bash
kubectl get pods -n nyx-system

# Expected output:
# nyx-daemonset-xxxxx    Running   (one per node)
# nyx-gatekeeper-xxxxx   Running   (one deployment)
```

---

## Your First Policy

### Default deny — cross-namespace traffic 

```yaml
apiVersion: nyx.tracenyx.io/v1alpha1
kind: NyxClusterNetworkPolicy
metadata:
  name: deny-cross-namespace
spec:
  priority: 50
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - decision: Deny
    fromNamespaceSelector:
      matchExpressions:
      - key: kubernetes.io/metadata.name
        operator: NotIn
        values: ["payment-ns"]
  egress:
  - decision: Deny
    toNamespaceSelector:
      matchExpressions:
      - key: kubernetes.io/metadata.name
        operator: NotIn
        values: ["payment-ns"]
```

### Allow specific service traffic

```yaml
apiVersion: nyx.tracenyx.io/v1alpha1
kind: NyxNetworkPolicy
metadata:
  name: allow-frontend-to-api
  namespace: cloudmart
spec:
  podSelector:
    matchLabels:
      app: api-gateway
  priority: 100
  policyTypes:
  - Ingress
  ingress:
  - decision: Allow
    fromPodSelector:
      matchLabels:
        app: frontend
    toPorts:
      port: 8080
      protocol: TCP
```

### FQDN egress control — prevent data exfiltration

```yaml
apiVersion: nyx.tracenyx.io/v1alpha1
kind: NyxNetworkPolicy
metadata:
  name: payments-egress
  namespace: cloudmart-payments
spec:
  podSelector:
    matchLabels:
      app: payments
  policyTypes:
  - Egress
  egress:
  - decision: Allow
    toFqdn:
      matchName: api.stripe.com
    toPorts:
      port: 443
  - decision: Allow
    toFqdn:
      matchName: approved-storage.blob.core.windows.net
    toPorts:
      port: 443
  - decision: Deny
    toFqdn:
      matchPattern: "*.blob.core.windows.net"
```

> Traditional firewalls see an IP address. Nyx sees a hostname. For services like Azure Blob Storage where multiple tenants share the same IP range, that difference is everything.

See more examples → [github.com/tracenyx/nyx-examples](https://github.com/tracenyx/nyx-examples)

---

## Platform Support

| Platform | Enforcement | Status |
|---|---|---|
| Linux (eBPF / TC) | Kernel-level | ✅ GA |
| Windows Server 2022 (WFP) | Kernel-level | ✅ GA |
| AKS (Azure) | ✅ | Tested |
| EKS (AWS) | ✅ | Tested |
| GKE (Google Cloud) | ✅ | Tested |
| On-premise | ✅ | Tested |
| Mixed Linux + Windows | ✅ | Tested |

---

## Policy Modes

| Mode | Behaviour | Use case |
|---|---|---|
| `enforce` | Blocks traffic, writes to kernel | Production enforcement |
| `audit` | Allows traffic, logs violations | Compliance validation |
| `dry-run` | No blocking, logs what would be denied | Safe policy testing |

---

## Tiers

| Feature | Scout | Sentinel | Aegis |
|---|---|---|---|
| **Price** | Free | Per namespace | Custom ACV |
| **Namespaces** | 3 | 20 | Unlimited |
| **Clusters** | 1 | 3 | Unlimited |
| **Linux eBPF** | ✅ | ✅ | ✅ |
| **Windows WFP** | 30-day trial | ✅ | ✅ |
| **AI anomaly detection** | 50/day | 500/day | Unlimited |
| **AI policy generation** | 10/day | 100/day | Unlimited |
| **Flow log retention** | 7 days | 90 days | 180 days |
| **Observability dashboard** | Basic | Full | Full |
| **Support** | GitHub Issues | Email | Dedicated SLA |
| **Data region** | Auto (AU East) | Auto | Preferred |
| **SSO / RBAC** | ❌ | ❌ | ✅ |
| **Git policy integration** | ❌ | ❌ | Roadmap |

[Start free with Scout →](https://tracenyx.ai)

---

## AI Features

Nyx analyses your actual flow logs and Kubernetes metadata — not generic security advice.

- **Anomaly detection** — surfaces unusual traffic patterns before they become incidents
- **Natural language policy** — describe what you want in plain English, Nyx generates the YAML
- **Policy conflict detection** — flags contradictions before you apply them
- **Compliance gap analysis** — maps your policy coverage against SOC2, PCI-DSS, HIPAA (Aegis)
- **"Why was this blocked?"** — AI explains any policy decision in plain language

> All AI processing happens within Tracenyx infrastructure. We send only anonymised, aggregated traffic patterns to our AI provider — never pod names, IP addresses, or raw flow records. Aegis customers can enable Private AI mode for fully isolated processing.

---

## Try It — CloudMart Demo

CloudMart is a sample microservices application (Linux + Windows) that demonstrates what your cluster looks like without proper network security — and how Nyx fixes it step by step.

```bash
helm install cloudmart tracenyx/nyx-demo
```

Follow the tutorial → [github.com/tracenyx/nyx-demo](https://github.com/tracenyx/nyx-demo)

---

## Roadmap

### v0.1 — May 2026 · Current
- Zero trust policy enforcement (Linux + Windows)
- FQDN-based egress filtering
- AI anomaly detection and observability dashboard
- Natural language policy generation
- Scout free tier

### v0.2 — Q3 2026
- Sentinel paid tier
- ARM64 support
- Extended AI history and trend analysis
- Private AI mode (Aegis)

### v0.3 — Q4 2026
- **Pod-to-pod traffic encryption**
  Transparent encryption between pods — no application changes required. Works across Linux and Windows nodes in the same cluster.
- **Process-level visibility**
  See which process inside a pod made a network call. Detect shell spawns, unexpected binaries, and lateral movement attempts at the process level — on both Linux and Windows nodes.
- **Git-based policy management (Aegis)**
  Policy-as-code via GitHub, GitLab, or Azure DevOps. PR-based review workflow, drift detection, and audit trail tied to Git commits.

### v1.0 — Q1 2027
- **Security validation suite**
  Test whether your policies actually block what you think they block. Built-in attack simulation covering lateral movement, data exfiltration, and privilege escalation — mapped to MITRE ATT&CK for Kubernetes.
- **Ingress protection**
  Rate limiting and connection throttling per namespace and per pod — enforced at the kernel level. Protect your services without additional infrastructure.
- **L7 observability**
  HTTP status code tracking, error rate analysis, and latency percentiles per service — with no sidecar required.

### Future
- Fine-tuned Nyx Security Model (trained on Kubernetes network patterns)
- Bring-your-own-model for air-gapped environments
- Full on-premise deployment option
- SPIFFE/SPIRE workload identity integration

---

## Documentation

- [Getting Started](https://docs.tracenyx.ai/getting-started)
- [Policy Model](https://docs.tracenyx.ai/policy-model)
- [Architecture](https://docs.tracenyx.ai/architecture)
- [CRD Reference](https://docs.tracenyx.ai/api-reference)
- [CloudMart Tutorial](https://github.com/tracenyx/nyx-demo)

---

## Security

Found a vulnerability? Please do not open a public GitHub issue.

→ Email: [security@tracenyx.ai](mailto:security@tracenyx.ai)

We aim to respond to all security disclosures within 24 hours.

---

## License

Nyx is proprietary software. The Scout tier is free to use under the [Scout License Terms](https://tracenyx.ai/legal/scout-license). Commercial use requires a license key.

© 2026 Tracenyx Pty Ltd. All rights reserved.

---

<div align="center">

Built in Melbourne, Australia 🇦🇺

[tracenyx.ai](https://tracenyx.ai) · [blog.tracenyx.ai](https://blog.tracenyx.ai) · [@tracenyx](https://x.com/tracenyx)

</div>
