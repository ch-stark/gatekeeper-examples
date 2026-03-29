# Upstream Contribution Guide: AI Admission Policies in Kubernetes (2026)

This guide explores how to contribute to **Validating Admission Policies (VAP)** and **OPA Gatekeeper**, specifically focusing on enhancements for AI/ML workloads (GPUs, LLM serving, and model governance).

---

## 1. Where to Contribute Upstream

### Validating Admission Policies (VAP)
VAP is the native Kubernetes engine. Contributions here affect the core orchestration layer.
* **Repository:** [kubernetes/kubernetes](https://github.com/kubernetes/kubernetes)
* **SIG:** [SIG API Machinery](https://github.com/kubernetes/community/tree/master/sig-api-machinery)
* **Primary Tooling:** **CEL (Common Expression Language)**.
* **2026 Focus:** Performance for high-scale inference and **Dynamic Resource Allocation (DRA)** for GPUs/TPUs.

### OPA Gatekeeper
Gatekeeper is the CNCF management plane for policy. It is more flexible for organizational guardrails.
* **Repository:** [open-policy-agent/gatekeeper](https://github.com/open-policy-agent/gatekeeper)
* **Library:** [gatekeeper-library](https://github.com/open-policy-agent/gatekeeper-library)
* **Primary Tooling:** **Rego** (though it now supports CEL for native VAP generation).

---

## 2. Enhancing for AI: VAP vs. Gatekeeper

| Feature | Validating Admission Policy (VAP) | OPA Gatekeeper |
| :--- | :--- | :--- |
| **Best Use Case** | Hardware-level resource validation. | High-level AI safety and compliance. |
| **Logic Engine** | CEL (Native, In-process). | Rego (Extensible, Webhook). |
| **AI Strength** | **Ultra-low latency** for inference pods. | **Context-aware** (can query model registries). |
| **Contribution Bar** | High (Requires K8s KEP process). | Medium (CNCF ecosystem). |

### The Verdict:
* **Enhance VAP** if you want to optimize how Kubernetes handles **AI hardware** (GPUs/TPUs).
* **Enhance Gatekeeper** if you want to build **operational guardrails** (safety, cost control, and model signatures).

---

## 3. Implementation Notation Examples

### Validating Admission Policy (CEL)
*Use this for high-performance hardware checks (e.g., ensuring AI pods target the correct GPU nodes).*

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingAdmissionPolicy
metadata:
  name: "ai-gpu-node-validation"
spec:
  failurePolicy: Fail
  matchConstraints:
    resourceRules:
    - apiGroups: [""]
      apiVersions: ["v1"]
      operations: ["CREATE"]
      resources: ["pods"]
  validations:
    - expression: "object.spec.containers.all(c, !c.resources.limits.exists(r, r == '[nvidia.com/gpu](https://nvidia.com/gpu)') || object.spec.nodeSelector['gpu-type'] in ['h100', 'a100'])"
      message: "Pods requiring GPUs must target high-performance nodes (H100/A100)."
