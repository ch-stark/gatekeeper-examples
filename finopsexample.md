# Technical Specification: FinOps GPU Admission Controller (2026)

## 1. Executive Summary
With the rapid scaling of AI workloads on NVIDIA H100/H200 infrastructure, unallocated cloud spend has become a primary operational risk. This specification outlines the implementation of a **Gatekeeper Policy** to enforce "Cost-Aware Admission" across the OpenShift fleet.

## 2. Problem Statement
- **Current State:** Developers can request high-cost GPU resources without providing billing attribution.
- **Impact:** Financial "black holes" in monthly cloud reports and inability to perform accurate ROI analysis on AI models.
- **Target Metric:** 100% of GPU-enabled workloads must have a valid `project-code` label.

## 3. Implementation Details

### 3.1 ConstraintTemplate (The Logic)
This Rego policy intercepts Pod creation requests and validates the presence of a billing label if GPUs are requested.

```yaml
apiVersion: templates.gatekeeper.sh/v1
kind: ConstraintTemplate
metadata:
  name: k8sgpufinops
spec:
  crd:
    spec:
      names:
        kind: K8sGPUFinOps
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8sgpufinops

        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          gpu_request := container.resources.limits["[nvidia.com/gpu](https://nvidia.com/gpu)"]
          
          # Logic: If GPU > 0 and Label is missing
          to_number(gpu_request) > 0
          not input.review.object.metadata.labels["project-code"]
```

### 3.2 Constraint (The Scope)
We apply this to all production-grade AI namespaces.

```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sGPUFinOps
metadata:
  name: enforce-gpu-billing-codes
spec:
  enforcementAction: deny
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Pod"]
    namespaces:
      - "ai-prod"
      - "llm-training"
```

## 4. User Experience (UX)

### Success Scenario
* **Action:** Developer deploys with `project-code: "ALPHA-2026"`.
* **Result:** Pod admitted; costs are tracked in KubeCost/FinOps dashboards.

### Failure Scenario
* **Action:** Developer deploys without the label.
* **Result:** Instant CLI error: `admission webhook denied the request: DENIED: GPU request in <training-ctr> requires a 'project-code' label.`

## 5. Roadmap & Future Improvements

* **Phase 2:** Integrate with **RHACM** to push this policy globally across AWS, Azure, and On-prem.
* **Phase 3:** Automated "Warn" period for existing legacy workloads to ensure a smooth transition.
* **Phase 4:** Expand to **OpenShift Virtualization** (VMs) to catch "Shadow IT" GPU usage.




          msg := sprintf("DENIED: GPU request in <%v> requires a 'project-code' label.", [container.name])
        }
