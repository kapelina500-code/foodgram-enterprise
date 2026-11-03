# Безопасность Foodgram

## Calico

```bash
minikube start --driver=qemu2 --network=socket_vmnet --cni=calico
kubectl get pods -n calico-system
```

## Apply security manifests

```bash
kubectl apply -f k8s/foodgram/security/pod-watcher-rbac.yaml
kubectl apply -f k8s/foodgram/security/ntfy-secret.yaml
kubectl apply -f k8s/foodgram/security/pod-watcher-code-configmap.yaml
kubectl apply -f k8s/foodgram/security/pod-watcher-deployment.yaml
kubectl apply -f k8s/foodgram/security/networkpolicy-postgres.yaml
kubectl apply -f k8s/foodgram/security/student/developers-rolebinding.yaml
kubectl apply -f k8s/foodgram/security/student/student-sa-headlamp.yaml
```

## Frontend non-root image

Frontend использует `nginxinc/nginx-unprivileged` и порт `8080`.
Образ: `elvinka505/foodgram-frontend:0.1.1`.

Сборка и загрузка в minikube:

```bash
docker build -f frontend/Dockerfile.nginx -t elvinka505/foodgram-frontend:0.1.1 frontend
minikube image load elvinka505/foodgram-frontend:0.1.1
# или: docker push elvinka505/foodgram-frontend:0.1.1
kubectl apply -R -f k8s/foodgram/frontend
kubectl rollout restart deploy/frontend -n foodgram
```

## Проверки

```bash
kubectl get networkpolicy -n foodgram
kubectl exec -n foodgram deploy/backend -- id
kubectl exec -n foodgram deploy/frontend -- id
kubectl exec -n foodgram statefulset/postgres -- id
kubectl create token student -n foodgram --duration=24h
```
