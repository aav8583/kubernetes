# Выполнено ДЗ № 1

 - [x] Основное ДЗ
 - [x] Задание со *

## В процессе сделано:
 #### Задание 1: 
 ответить на вопрос "Разберитесь почему все pod в namespace kube-system восстановились после удаления. Укажите причину в описании PR"
 #### Ответ:
 Потому что системные компоненты должны перезапускаться всегда и у них критический приоритет:  
 
    Restart Policy: Always  
    Priority: 2000000000  
    Priority Class Name: system-cluster-critical  

 #### Задание 2:
 Выясните причину, по которой pod frontend находится в статусе Error
 #### Ответ:
 В логе контейнера вывелся алерт:  
    
    panic: environment variable "PRODUCT_CATALOG_SERVICE_ADDR" not set
 Добавил переменную окружения в манифест и под запустился

## Как запустить проект:
    minikube start
    kubectl apply -f web-pod.yaml
    kubectl apply -f frontend-pod-healthy.yaml
    
## Как проверить работоспособность:
Прокинуть порт хост-машины на порт пода "web":

    kubectl port-forward --address 0.0.0.0 pod/web 8000:8000

## PR checklist:
 - [x] Выставлен label с темой домашнего задания