Step 1: Create a GKE Cluster

Provision a Google Kubernetes Engine (GKE) cluster to host the monitoring stack.

Step 2: Create a Dedicated Monitoring Namespace (Best Practice)

Create a separate namespace named monitor to isolate Prometheus and Grafana components.

kubectl create ns monitor


Step 3: Navigate to the Official Helm Repository for Prometheus & Grafana

The Helm chart source code for Prometheus and Grafana (kube-prometheus-stack) is available at:

https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack

Step 4: Add the Prometheus Community Helm Repository

Add the official Helm repository and update it locally.

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update


Step 5: Install Prometheus and Grafana Using Helm

Deploy the monitoring stack into the monitor namespace.

helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack --namespace monitor


Explanation:

kube-prometheus-stack → Release name (customizable)

prometheus-community/kube-prometheus-stack → Chart name

monitor → Namespace where the operator will be deployed

Part 2 – Verification & Access

Step 6: Verify the Installation

Check whether all monitoring pods are running successfully.

kubectl get pods -n monitor


Step 7: Create a Service for Grafana (Expose on Port 3000)

Expose Grafana as a LoadBalancer service on port 3000.

kubectl expose deployment kube-prometheus-stack-grafana \
  --port=3000 \
  --target-port=3000 \
  --name=grafana \
  --type=LoadBalancer \
  -n monitor


Step 8: Verify Service and Access Grafana

Check the created service:

kubectl get svc -n monitor


Default Login Credentials:

Username: admin

Password: prom-operator (or retrieve from secret)

Retrieve admin password:

kubectl --namespace monitor get secrets kube-prometheus-stack-grafana \
-o jsonpath="{.data.admin-password}" | base64 -d ; echo


Step 9: Explore Auto-Configured Grafana Dashboards

After logging in to Grafana, navigate to:

General / Kubernetes / Compute Resources / Cluster

General / Kubernetes / Compute Resources / Namespace (Pods)

These dashboards are automatically provisioned by kube-prometheus-stack.

Step 10: Delete the Helm Release (Cleanup)

To uninstall the monitoring stack:

helm uninstall kube-prometheus-stack --namespace monitor

Installation Output Example
NAME: kube-prometheus-stack
LAST DEPLOYED: Wed Feb 18 03:47:29 2026
NAMESPACE: monitor
STATUS: deployed
REVISION: 1
DESCRIPTION: Install complete


Check deployment status:

kubectl get po -n monitor

Access Grafana Locally (Port Forward)
export POD_NAME=$(kubectl --namespace monitor get pod \
-l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=kube-prometheus-stack" -oname)

kubectl --namespace monitor port-forward $POD_NAME 3000


Access in browser:

http://localhost:3000
