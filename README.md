
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
