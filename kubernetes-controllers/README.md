# Выполнено ДЗ № 2

 - [x] Основное ДЗ
 - [x] Задание со *

## В процессе сделано:

#### Задание 1: ReplicaSet, Deployment 
Поднят кластер kind. 
        
Создан ReplicaSet для микросервиса frontend, поочередно созданы три реплики каждой из двух версий.
При обновлении и последующем обновлении манифеста версия образа контейнеров не обновилась.

#### Вопрос: 
почему обновление ReplicaSet не повлекло обновление запущенных pod?
#### Ответ:
Replicaset не умеет обновлять некоторые свои свойства, в частности шаблоны контейнеров. Шаблоны используются только при создании объекта ReplicaSet.

Рассмотрены способы масштабирования подов из командной строки и посредством ReplicaSet и Deployment.
Произведен откат Deployment сервиса paymentservice.
Выполнено задание со *.

#### Задание 2: Probes, DaemonSet
Развернут frontend с deploument, указан readinessProbe.

С помощью DaemonSet развернуты  Node Exporter, в т.ч. на мастер нодах.

## Как запустить проект:
    kind create cluster --config kind-config.yaml
Конфиг kind-config.yaml:

    kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4
    nodes:
        - role: control-plane
        - role: control-plane
        - role: control-plane
        - role: worker
        - role: worker
        - role: worker
    
## Как проверить работоспособность:
- Прокинуть порт хост-машины на порт пода "web":

    kubectl port-forward --address 0.0.0.0 pod/web 8000:8000
    
- Проверить корректность применения манифестов можно командой:
 ```bash
 kubectl get pods -l app=frontend
 ```
```bash
 kubectl get pods -l app=paymentservice
 ```
 - Проверить сервис node-exporter можно командой:
 ```bash
 curl localhost:9100/metrics
```

## PR checklist:
 - [x] Выставлен label с темой домашнего задания (kubernetes-controllers)