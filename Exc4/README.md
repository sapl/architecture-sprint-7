# Задание 4. Защита доступа к кластеру Kubernetes


### Роли и группы

| Роль               | Права                                                         | Группа пользователей                   |
|--------------------|---------------------------------------------------------------|----------------------------------------|
| Admin              | Полный доступ ко всем ресурсам кластера                       | Привилегированная группа Admin         |
| View               | Доступ только для просмотра ресурсов кластера                 | Группа Monitoring (сервисные аккаунты) |
| Developer-Project1 | Просмотр всех ресурсов в namespace `project1`                 | Группа Developer Project1              |
| Developer-Project2 | Просмотр всех ресурсов в namespace `project2`                 | Группа Developer Project2              |
| TechLead-Project1  | Управление ресурсами (включая секреты) в namespace `project1` | Группа TechLead Project1               |
| TechLead-Project2  | Управление ресурсами (включая секреты) в namespace `project2` | Группа TechLead Project2               


### Создание ролей

- Start minikube:
```shell
minikube start --memory=2048 --cpus=2 
```

- Создать namespace
```shell
kubectl apply -f namespaces/namespace-project1.yaml
kubectl apply -f namespaces/namespace-project2.yaml
```

- `admin` и `view` роль уже есть по умолчанию (так и называются) 
можно их использовать, не создавать свои


- Создание ролей разработчиков и техлидов
```shell
kubectl apply -f roles/developer-project1-role.yaml
kubectl apply -f roles/developer-project2-role.yaml
kubectl apply -f roles/techlead-project1-role.yaml
kubectl apply -f roles/techlead-project2-role.yaml
```

- Привязка ролей
```shell
kubectl create serviceaccount developer1-account -n project1
kubectl create serviceaccount techlead1-account -n project1
kubectl apply -f bindings/monitoring-binding.yaml
kubectl apply -f bindings/developer-project1-binding.yaml
kubectl apply -f bindings/developer-project2-binding.yaml
kubectl apply -f bindings/techlead-project1-binding.yaml
kubectl apply -f bindings/techlead-project2-binding.yaml
```

### Проверка 

- Создать секреты и поднять поды по одному в каждом namespace

```shell
kubectl apply -f secrets/secret1.yaml
kubectl apply -f secrets/secret2.yaml
kubectl apply -f deployments/project1-app.yaml
kubectl apply -f deployments/project2-app.yaml
```

- Утановить Dashboard чтобы там проверять что видно
```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
kubectl proxy
```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

- Получить токен для developer1
```shell
kubectl -n project1 get secret \
  "$(kubectl -n project1 get sa developer1-account -o jsonpath='{.secrets[0].name}')" \
  -o jsonpath='{.data.token}' \
  | base64 --decode
```
Ему будуте видны только поды из project1
и для них не видны секреты

- Получить токен для techlead1
```shell
kubectl -n project1 get secret \
  "$(kubectl -n project1 get sa techlead1-account -o jsonpath='{.secrets[0].name}')" \
  -o jsonpath='{.data.token}' \
  | base64 --decode
```

аналогично для teamlead1 уже будут видны и секреты
и есть права на изменение

