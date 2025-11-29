

#  **Falcon Operator Test Module**

A lightweight module for deploying **CrowdStrike Falcon Operator** custom resources across Kubernetes clusters (EKS, AKS, GKE, Bottlerocket, Minikube, KinD).

This repo provides a **minimal, end-to-end example** of how to install and test:

* **FalconNodeSensor**
* **FalconContainer**
* **FalconAdmission**
* **FalconImageAnalyzer**
* **FalconDeployment** (unified CR)

It is designed for **learning**, **testing**, and **CI/CD validation** before rolling out Falcon Operator in real environments.

---

# **Repository Structure**

```
.
├── charts/
│   └── falcon-operator/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           └── falcondeployment.yaml
│
├── manifests/
│   ├── falconadmission.yaml
│   ├── falconcontainer.yaml
│   ├── falcondeployment.yaml
│   ├── falconimageanalyzer.yaml
│   └── falconnodesensor.yaml
│
└── README.md
```

---

#  **CrowdStrike Credentials & API Scope Requirements**

Falcon Operator CRDs **require valid CrowdStrike API keys** with specific scopes.

##  **Required API Scopes**

At minimum, your API Client must have:

### **For Node Sensor + Container Sensor**

* **Sensor Download: Read**
* **Container Sensor Download: Read** (if applicable)
* **Kubernetes Protection: Write**
* **Kubernetes Admission Control: Write**
* **Image Assessment: Write**
* **Falcon Images: Read**

### **For FalconDeployment CR**

* Same as above — full K8s protection CRUD permissions.

If using Falcon Container with private registry automation:

* **Registry Credentials: Write**

> ⚠️ If your credentials do not have these scopes, the Operator will fail to reconcile the CR.

---

# 🔧 **Setting Environment Variables (Local Development)**

Before applying CRDs or Helm charts, set these:

```bash
export FALCON_CLIENT_ID="your_client_id"
export FALCON_CLIENT_SECRET="your_client_secret"
```

You can then reference them directly in your `values.yaml` or Kubernetes secrets:

```yaml
falcon:
  client_id: ${FALCON_CLIENT_ID}
  client_secret: ${FALCON_CLIENT_SECRET}
```

---

#  **Secure Credential Management (Production)**

For **production** workloads, **NEVER** hardcode Falcon API keys into:

✘ values.yaml
✘ GitHub repos
✘ Kubernetes manifests
✘ Terraform variables without encryption

Instead, use a Secrets Manager:

### **Recommended Approaches for Real Environments**

| Platform  | Recommended Secret Store                    |
| --------- | ------------------------------------------- |
| AWS EKS   | AWS Secrets Manager + IRSA                  |
| Azure AKS | Azure Key Vault + Managed Identity          |
| GKE       | Google Secret Manager + Workload Identity   |
| Any K8s   | External Secrets Operator OR Sealed Secrets |

### **Example: Storing Falcon Keys in AWS Secrets Manager**

```bash
aws secretsmanager create-secret \
  --name falcon-operator-creds \
  --secret-string "{\"client_id\":\"YOUR_ID\",\"client_secret\":\"YOUR_SECRET\"}"
```

Using External Secrets Operator:

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: falcon-operator-creds
spec:
  secretStoreRef:
    name: aws-secretsmanager
    kind: SecretStore
  target:
    name: falcon-api-creds
  data:
    - secretKey: client_id
      remoteRef:
        key: falcon-operator-creds
        property: client_id
    - secretKey: client_secret
      remoteRef:
        key: falcon-operator-creds
        property: client_secret
```

Your CRs then reference:

```yaml
falcon_api:
  client_id: valueFromSecret
```

---

# 📦 **Install CRDs Manually**

To apply example manifests:

```bash
kubectl apply -f manifests/
```

This deploys all test CRs using the credentials inserted into each file.

---

# 🚀 **Deploy with Helm (Recommended)**

### **Install**

```bash
helm install falcon-operator-test charts/falcon-operator \
  --set falcon.client_id="$FALCON_CLIENT_ID" \
  --set falcon.client_secret="$FALCON_CLIENT_SECRET"
```

### **Upgrade**

```bash
helm upgrade falcon-operator-test charts/falcon-operator \
  --reuse-values
```

---

#  **Supported Platforms**

This test module works on:

* Amazon **EKS**
* Azure **AKS**
* Google **GKE**
* **Bottlerocket** (EKS-optimized)
* Minikube
* KinD / Local Docker clusters
* Rancher RKE2 / K3s

---

# 📘 **What This Module Is For**

* Learning Falcon Operator CRDs
* Testing Kubernetes Admission / Node / Container sensors
* CI/CD testing before production rollout
* Validating multi-cloud operator behavior
* Building GitOps patterns around CrowdStrike deployment
* Demonstrating Kubernetes security skills in a portfolio

---

# 🔥 **Next Steps**

You can extend this module to include:

✔ GitOps automation
✔ Flux or ArgoCD deployment
✔ External Secrets integration
✔ Multi-environment Helm values
✔ Bottlerocket-specific node sensor configs

If you'd like me to generate any of these, just tell me:

**“Add production version”**
or
**“Add GitOps version”**

---

# 📫 **Connect**

If you’re exploring Kubernetes security, CNAPP, Falon Operator, or multi-cloud hardening — feel free to reach out.

-