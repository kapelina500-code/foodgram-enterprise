# Сеть в K8s (minikube)

Схема: `клиент -> ingress-nginx -> Ingress -> Service -> Pod`.
Хост `foodgram.test`: `/` идёт на `frontend`, а `/api`, `/admin`, `/media`, `/s` - на `backend`.
(На macOS лучше `.test`, а не `.local` — `.local` занят Bonjour и часто ломает DNS.)

## Запуск

```bash
minikube start --driver=qemu2 --network=socket_vmnet --cni=calico
minikube addons enable ingress
kubectl get pods -n ingress-nginx

kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.21.2/cert-manager.yaml
kubectl get pods -n cert-manager  

kubectl apply -f k8s/foodgram/namespace.yaml
kubectl apply -R -f k8s/foodgram/postgres
kubectl apply -R -f k8s/foodgram/backend
kubectl apply -R -f k8s/foodgram/frontend
kubectl apply -f k8s/cert-manager/cluster-issuer.yaml
kubectl apply -f k8s/foodgram/ingress
```

## Локальный домен

```bash
echo "$(minikube ip) foodgram.test" | sudo tee -a /etc/hosts
minikube tunnel   # в отдельном терминале (для QEMU нужен socket_vmnet)
```

## Проверка

```bash
kubectl get ingress -n foodgram
kubectl get certificate -n foodgram  
curl -k https://foodgram.test/api/recipes/
```

Сертификат self-signed, поэтому браузер покажет предупреждение.