# Задание 5. Управление трафиком внутри кластера Kubertnetes




### Создание сервисов

- Start minikube:
```shell
minikube start --memory=2048 --cpus=2 --network-plugin=cni --cni=calico
```

- Запуск сервисов:
```shell
kubectl run front-end-app --image=nginx --labels role=front-end --expose --port=80
kubectl run back-end-api-app --image=nginx --labels role=back-end-api --expose --port=80
kubectl run admin-front-end-app --image=nginx --labels role=admin-front-end --expose --port=80
kubectl run admin-back-end-api-app --image=nginx --labels role=admin-back-end-api --expose --port=80
```

- Применить политики:
```shell
kubectl apply -f non-admin-api-allow.yaml
```

- Проверяем доступность
```shell
 kubectl run test-$RANDOM --image=alpine --labels role=front-end --restart=Never -it --rm -- sh
# wget -qO- --timeout=2 http://back-end-api-app     # Должно получиться
# wget -qO- --timeout=2 http://admin-back-end-api-app  # Должно быть "timeout" или "refused"
```