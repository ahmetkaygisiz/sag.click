---
title: "017 K8s_commands"
date: 2024-10-26T16:05:06+03:00
draft: true
---

Imperative : 

kubectl run --image=nginx nginx
kubectl create deployment --image=nginx nginx
kubectl expose deployment nginx --port 80
kubectl edit deployment nginx
kubectl scale deployment nginx --replicas=5
kubectl set image deployment nginx nginx=nginx:1.18

kubectl create -f nginx.yaml
kubectl replace -f nginx.yaml
kubectl delete -f nginx.yaml

-------------
Declarative 
kubectl apply -f nginx.yaml

-------------

# create a pod
kubectl run nginx --image=nginx
kubectl run nginx --image=nginx --dry-run=client -o yaml

# deployment
kubectl create deployment --image=nginx nginx
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml

kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
kubectl create deployment nginx --image=nginx --replicas=4
kubectl scale deployment nginx--replicas=4
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml

kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml 
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml

git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
kubectl top node
kubectl top pods

####APP Lifecycle
Rollout Command:

kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment

Recreate -> Down all versions first create new ones
RollingUpdate -> Partial down, create new pods
 
kubectl rollout undo deployment/myapp-deployment
kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1


kubectl taint nodes controlplane node-role.kubernetes.io/control-plane:NoSchedule-