
# Node Affinity

👉 Controls which nodes a pod can run on

“Run my backend only on app nodes”

# Pod Anti-Affinity

👉 Ensures pods do not run on the same node

“Spread replicas across nodes for high availability”

labels--
kubectl label node ip-192-168-22-233.ap-south-1.compute.internal role=app

✅ Meaning

You are adding a label to a worker node:

role = app

kubectl get node 


1️⃣#  TAINT & TOLERATION — FULL WORKING EXAMPLE
Scenario

We want only backend pods to run on a specific node.

# STEP 1 — TAINT A NODE
kubectl taint node ip-192-168-22-233.ap-south-1.compute.internal \
env=prod:NoSchedule

What this does

Node repels all pods

Only pods with toleration can run

Verify:

kubectl describe node ip-192-168-22-233.ap-south-1.compute.internal

# STEP 2 — ADD TOLERATION IN DEPLOYMENT
tolerations:
  - key: "env"
    operator: "Equal"
    value: "prod"
    effect: "NoSchedule"

Where it goes (IMPORTANT)
spec:
  template:
    spec:
      tolerations:
        - key: "env"
          operator: "Equal"
          value: "prod"
          effect: "NoSchedule"

Result

✔ Pod can now run on tainted node
❌ Other pods blocked

## 

“Taints protect nodes. Tolerations grant permission.”

# 2️⃣ CORDON — STOP NEW PODS
Command
kubectl cordon ip-192-168-22-233.ap-south-1.compute.internal

What happens

Node marked Unschedulable

Existing pods continue running

New pods ❌ blocked

Check:

kubectl get nodes

When to use cordon

Node maintenance

Kernel patching

Pre-drain step

3️⃣ DRAIN — SAFELY EVICT PODS
Command
kubectl drain ip-192-168-22-233.ap-south-1.compute.internal \
--ignore-daemonsets \
--delete-emptydir-data

What happens

Node cordoned automatically

Pods gracefully terminated

Pods recreated on other nodes

IMPORTANT FLAGS EXPLAINED
Flag	Purpose
--ignore-daemonsets	Keep kube-proxy, CNI
--delete-emptydir-data	Required for local storage
--force	For unmanaged pods
# 4️⃣ UNCordon — BRING NODE BACK
kubectl uncordon ip-192-168-22-233.ap-south-1.compute.internal

# 5️⃣ COMPLETE FLOW (REAL PRODUCTION)
cordon → drain → maintain → uncordon

# HELM
What is Helm?
✅ Definition (Simple)

Helm is a package manager for Kubernetes.

Just like:

apt → Linux

yum → Linux

npm → Node.js

👉 Helm manages Kubernetes applications as packages called Charts.

🔹 Why Do We Need Helm?

Without Helm, managing Kubernetes apps becomes:

❌ Too many YAML files
❌ Manual updates
❌ Hard rollbacks
❌ Configuration duplication
❌ Error-prone deployments

Helm solves all of this.

🔹 Problem WITHOUT Helm (Reality)

Example: Installing Prometheus manually

You must create:

Deployment YAML

Service YAML

ConfigMaps

RBAC

CRDs

Alertmanager configs

Grafana dashboards

💥 50+ YAML files

🔹 Solution WITH Helm
helm install monitoring prometheus-community/kube-prometheus-stack


✔ Everything installed
✔ Correct versions
✔ Best practices
✔ Production-ready

# Download helm 
https://helm.sh/docs/intro/install/

# DaemonSet
A DaemonSet defines Pods that provide node-local facilities. These might be fundamental to the operation of your cluster, such as a networking helper tool, or be part of an add-on.
A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created.

Some typical uses of a DaemonSet are:

running a cluster storage daemon on every node
running a logs collection daemon on every node
running a node monitoring daemon on every node


# Setup grafana , Prom 

# Check cluster access
kubectl get nodes

Install Helm (if not installed)
brew install helm


Verify:

helm version

# STEP 2 — CREATE A MONITORING NAMESPACE
kubectl create namespace monitoring


Why?

Keeps monitoring workloads isolated from applications

# STEP 3 — ADD PROMETHEUS HELM REPO
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# STEP 4 — INSTALL PROMETHEUS + GRAFANA (KUBE-PROMETHEUS-STACK)
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring

What this installs automatically

✅ Prometheus
✅ Grafana
✅ Alertmanager
✅ Node Exporter
✅ kube-state-metrics


# STEP 5 — VERIFY PODS
kubectl get pods -n monitoring


You should see:

prometheus-*
grafana-*
alertmanager-*
node-exporter-*

# STEP 6 — ACCESS GRAFANA
Get Grafana service
kubectl get svc -n monitoring


Grafana service name:

monitoring-grafana

OPTION 1️⃣ — Port Forward (BEST FOR DEMO)
kubectl port-forward svc/monitoring-grafana 3000:80 -n monitoring


Open browser:

http://localhost:3000

# OPTION 2️⃣ — LoadBalancer (EKS PRODUCTION)
kubectl patch svc monitoring-grafana -n monitoring \
  -p '{"spec":{"type":"LoadBalancer"}}'


Then:

kubectl get svc -n monitoring

# STEP 7 — GRAFANA LOGIN
Get admin password
kubectl get secret monitoring-grafana -n monitoring \
  -o jsonpath="{.data.admin-password}" | base64 --decode
