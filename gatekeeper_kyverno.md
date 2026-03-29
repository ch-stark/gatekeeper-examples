# When Gatekeeper Beats Kyverno: 7 Use Cases That Matter

In 2026, the "Gatekeeper vs. Kyverno" debate has settled into a clear division: Kyverno is for operational simplicity, while Gatekeeper is for complex, data-driven governance.

While Kyverno's move to CEL (Common Expression Language) makes it fast for simple checks, it lacks the programmable depth of OPA's Rego. Here are 7 specific use cases where Gatekeeper is the superior — or only — choice.

---

## 1. Recursive Dependency Validation

**The Problem:** A Pod can only be created if its Namespace has a specific label, and that label must match a value inside a specific ConfigMap in a different administrative namespace.

**Why Gatekeeper wins:** Rego is a query language that handles "joins" across different data types naturally. CEL (used by Kyverno and native K8s) is non-Turing complete and does not support the recursive lookups or complex data-walking required to traverse three different resource types in one go.

---

## 2. Real-Time "Security Oracle" (gRPC External Data)

**The Problem:** Blocking a container image immediately if a vulnerability scanner (like Snyk or Wiz) flags a "Critical" CVE, even if the image was already scanned once.

**Why Gatekeeper wins:** Gatekeeper's External Data Provider allows it to call external gRPC/HTTP "Oracles" during the admission process. It can send the image digest to your security tool and get a live go/no-go decision. Kyverno's external lookups are more limited in their error handling and protocol support.

---

## 3. Complex Set Operations (Intersection/Union)

**The Problem:** "Allow this Pod only if its list of `imagePullSecrets` is a subset of the authorized secrets allowed for this specific Namespace's 'Security Group' (defined in an annotation)."

**Why Gatekeeper wins:** Rego treats sets as first-class citizens. You can perform `set_diff`, `intersection`, and `union` in a single line. Doing "subset" logic in Kyverno's YAML/CEL requires long, unreadable logic strings that are prone to bugs.

---

## 4. Multi-System Policy Portability

**The Problem:** You want to use the exact same security logic to validate your Kubernetes manifests, your Terraform plans, and your Envoy/Istio service mesh authorization.

**Why Gatekeeper wins:** OPA is universal. Write the Rego once and run it in Gatekeeper (K8s), Conftest (CI/CD/Terraform), and Envoy (AuthZ). Kyverno is strictly Kubernetes-only, forcing you to rewrite security logic in different languages for different tools.

---

## 5. Granular "Fail Open" vs. "Fail Closed"

**The Problem:** If your external vulnerability scanner is down, you want to warn (allow) for "Dev" namespaces but block (deny) for "Prod" namespaces.

**Why Gatekeeper wins:** Gatekeeper allows you to define different `failurePolicy` behaviors within the Rego logic itself. You can catch the error from an external call and decide the outcome based on the request's context — for example, Namespace labels.

---

## 6. Aggregation & Quota Enforcement (Beyond ResourceQuotas)

**The Problem:** "The sum of all GPU requests in a Namespace cannot exceed a dynamic value fetched from an external Billing API."

**Why Gatekeeper wins:** This requires calculating a sum across many objects and comparing it to an external variable. Rego handles this map/reduce style logic effortlessly. Kyverno is designed to evaluate objects one-by-one, making aggregate math across a namespace difficult.

---

## 7. Algorithmic Decision Trees

**The Problem:** A complex compliance rule where the "allow" decision depends on a nested tree of conditions — for example:

```
PCI-compliant=true AND Network=Private AND Encryption=AES256
OR (Override=true AND Manager-Approval=signed)
```

**Why Gatekeeper wins:** This is essentially a small program. Rego is a logic-programming language specifically designed for these decision trees. Representing this in Kyverno's YAML structure leads to indentation hell that is nearly impossible for a security auditor to verify.

---

## Strategic Recommendation

| Use Case | Choose |
|---|---|
| Label hygiene, `:latest` tag blocking, resource limits | **Kyverno** |
| Cross-resource validation, external security integrations, multi-cloud compliance | **Gatekeeper** |

Choose **Kyverno** if you want to quickly fix hygiene issues across your cluster fleet.

Choose **Gatekeeper** if you are building an enterprise platform that requires programmable, auditable, multi-system policy logic.
