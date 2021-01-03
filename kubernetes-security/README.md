# Выполнено ДЗ № 3

 - [x] Основное ДЗ

## В процессе сделано:
 #### Задание 1: 
 - 01-bob.yaml - добавление ServiceAccount bob
 - 02-crb-bob.yaml - биндинг ServiceAccount bob к роли admin namespace'а default
 - 03-dave.yaml - добавление ServiceAccount dave без доступа к кластеру

 #### Задание 2:
 - 01-namespace.yaml - создание namespace prometheus
 - 02-carol.yaml - создание ServiceAccount carol в namespace prometheus
 - 03-namespace-role-reader.yaml - создание роли pod-reader
 - 04-role-binder.yaml - биндинг роли pod-reader ко всем ServiceAccount в namespace prometheus 

 #### Задание 3:
 - 01-namespace-dev.yaml - создание namespace dev 
 - 02-jane.yaml - создание ServiceAccount jane в namespace dev
 - 03-admin-jane.yaml - биндинг роли admin к ServiceAccount jane
 - 04-ken.yaml - создание ServiceAccount jane в namespace dev
 - 05-view-ken.yaml - биндинг роли view к ServiceAccount jane

## Как запустить проект:
    minikube start
    kubectl apply -f kubernetes-security/task01/
    kubectl apply -f kubernetes-security/task02/
    kubectl apply -f kubernetes-security/task03/
    
## Как проверить работоспособность:
    kubectl auth can-i get pods -n kube-system --as system:serviceaccount:default:bob
    yes
    kubectl auth can-i get pods -n kube-system --as system:serviceaccount:default:dave
    no
    kubectl auth can-i get deployments --as system:serviceaccount:prometheus:carol
    no
    kubectl auth can-i list pods --as system:serviceaccount:prometheus:carol -n prometheus
    yes
    kubectl auth can-i list pods --as system:serviceaccount:prometheus:carol
    yes
    kubectl auth can-i get deployments --as system:serviceaccount:dev:jane
    no
    kubectl auth can-i create deployments --as system:serviceaccount:dev:jane -n dev
    yes
    kubectl auth can-i get deployments --as system:serviceaccount:dev:ken
    no
    kubectl auth can-i get deployments --as system:serviceaccount:dev:ken -n dev
    yes
    kubectl auth can-i create deployments --as system:serviceaccount:dev:ken -n dev
    no

## PR checklist:
 - [x] Выставлен label с темой домашнего задания (kubernetes-security)