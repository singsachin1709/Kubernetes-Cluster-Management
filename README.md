# Deploying a Scalable Web Application on AWS EKS with Monitoring
This project demonstrates how to deploy and manage a web application (`mywebd`) on an **AWS EKS (Elastic Kubernetes Service)** cluster.  
It includes:  
- Cluster creation  
- Application deployment  
- Exposing the app with a LoadBalancer  
- Enabling horizontal pod autoscaling (HPA)  
- Setting up **Prometheus + Grafana** for monitoring

## 🚀 Steps

### 1. Configure AWS CLI
Set up your AWS CLI credentials:
```
aws configure
```
### 2. Create an EKS Cluster

Create a cluster in ap-south-1 using eksctl:
```
eksctl create cluster mycluster --region=ap-south-1
```
Verify:
```
kubectl get nodes
```
### 3. Create Deployment

Deploy the PHP webserver image:
```
kubectl create deployment mywebd --image=vimal13/apache-webserver-php \
  --dry-run=client -o yaml > mydeployment.yml

kubectl apply -f mydeployment.yml
```
Check pods:
```
kubectl get pods
```
### 4. Expose Deployment with a LoadBalancer

Expose the app to the internet:
```
kubectl expose deployment mywebd --type=LoadBalancer --port=80 \
  --dry-run=client -o yaml > mysvc.yml

kubectl apply -f mysvc.yml
```
Verify service:
```
kubectl get svc mywebd
```
Note the EXTERNAL-IP and open it in a browser:
```
curl http://<EXTERNAL-IP>
```
### 5. Enable Horizontal Pod Autoscaler (HPA)

Scale the deployment automatically based on CPU usage:
```
kubectl autoscale deployment mywebd \
  --cpu-percent=50 --min=1 --max=10 \
  --dry-run=client -o yaml > myhorizontalpodautoscaler.yml

kubectl apply -f myhorizontalpodautoscaler.yml
```
Check HPA:
```
kubectl get hpa
```
### 6. Setup Monitoring with Prometheus and Grafana

We use the kube-prometheus-stack Helm chart which includes:
- Prometheus
- Grafana
- Alertmanager
- Kubernetes dashboards

#### 6.1 Add Helm repos
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```
#### 6.2 Install kube-prometheus-stack
```
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace
```
Verify:
```
kubectl get pods -n monitoring
```
#### 6.3 Access Grafana
1. Get Grafana admin credentials:
```
kubectl get secret monitoring-grafana -n monitoring -o jsonpath="{.data.admin-user}" | base64 --decode; echo
kubectl get secret monitoring-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode; echo
```
2. Port-forward Grafana service:
```
kubectl port-forward svc/monitoring-grafana -n monitoring 3000:80
```
3. Open http://localhost:3000

   Login with the above credentials.

You will see default dashboards for:

- Kubernetes / Nodes
- Pods
- Deployments
- Cluster health
- API server metrics

### 📂 Files Generated
- **mydeployment.yml** → Deployment manifest for mywebd
- **mysvc.yml** → Service manifest (LoadBalancer)
- **myhorizontalpodautoscaler.yml** → HPA configuration

### 🧹 Cleanup

To delete all resources:
```
kubectl delete -f myhorizontalpodautoscaler.yml
kubectl delete -f mysvc.yml
kubectl delete -f mydeployment.yml
helm uninstall monitoring -n monitoring
eksctl delete cluster --name mycluster --region=ap-south-1
```
### ✅ Summary
- ✅ Created an EKS cluster on AWS
- ✅ Deployed a web server application (vimal13/apache-webserver-php)
- ✅ Exposed it using a LoadBalancer Service
- ✅ Configured HPA for scalability
- ✅ Installed Prometheus + Grafana for monitoring cluster health and performance
