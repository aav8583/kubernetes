# Выполнено ДЗ № 9

 - [x] Основное ДЗ

## В процессе сделано:
###Подготовка Kubernetes кластера
Остановлен имевшийся кластер, чтобы не тратил деньги. Делается командой

    gcloud container clusters resize CLUSTER_NAME --size=0

    WARNING: The --size flag is now deprecated. Please use `--num-nodes` instead.
    Pool [default-pool] for [CLUSTER_NAME] will be resized to 0.
    
    Do you want to continue (Y/n)?  y
    
    Resizing CLUSTER_NAME...done.
    Updated [https://container.googleapis.com/v1/projects/PROJECT_NAME/zones/CLUSTER_ZONE/clusters/CLUSTER_NAME].
    
    
    Updates are available for some Cloud SDK components.  To install them,
    please run:
    $ gcloud components update

Новый кластер развернут с двумя пулами нод, по инструкции в презентации.

Инициализация через gcloud и настройка kubectl на использование нового контекста:

    gcloud container clusters get-credentials cluster-logging --zone us-central1-c --project PROJECT_NAME
    Fetching cluster endpoint and auth data.
    kubeconfig entry generated for cluster-logging.

    kubectl config current-context
    gke_third-crossing-301118_us-central1-c_cluster-logging

    kubectl get nodes
    NAME                                             STATUS   ROLES    AGE     VERSION
    gke-cluster-logging-default-pool-e5144c9f-s2ch   Ready    <none>   5m42s   v1.17.14-gke.1600
    gke-cluster-logging-infra-pool-8170fcc6-2xvf     Ready    <none>   5m44s   v1.17.14-gke.1600
    gke-cluster-logging-infra-pool-8170fcc6-3tmw     Ready    <none>   5m42s   v1.17.14-gke.1600
    gke-cluster-logging-infra-pool-8170fcc6-lxnq     Ready    <none>   5m43s   v1.17.14-gke.1600

###Установка HipsterShop
Установка знакомого проекта с помощью локального манифеста:

    kubectl create ns microservices-demo
    kubectl apply -f hipster-shop.yml -n microservices-demo
    deployment.apps/emailservice created
    service/emailservice created
    deployment.apps/checkoutservice created
    service/checkoutservice created
    deployment.apps/recommendationservice created
    service/recommendationservice created
    deployment.apps/frontend created
    service/frontend created
    service/frontend-external created
    deployment.apps/paymentservice created
    service/paymentservice created
    deployment.apps/productcatalogservice created
    service/productcatalogservice created
    deployment.apps/cartservice created
    service/cartservice created
    deployment.apps/loadgenerator created
    deployment.apps/currencyservice created
    service/currencyservice created
    deployment.apps/shippingservice created
    service/shippingservice created
    deployment.apps/redis-cart created
    service/redis-cart created
    deployment.apps/adservice created
    service/adservice created

Все поды должны развернуться в default-pool: 

    kubectl get pods -n microservices-demo -o wide
    NAME                                     READY   STATUS    RESTARTS   AGE     IP           NODE                                             NOMINATED NODE   READINESS GATES
    adservice-cb695c556-ntmqt                1/1     Running   0          4m41s   10.28.1.19   gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    cartservice-f4677b75f-wlt56              1/1     Running   2          4m45s   10.28.1.14   gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    checkoutservice-664f865b9b-bsvvc         1/1     Running   0          4m51s   10.28.1.9    gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    currencyservice-bb9d998bd-2pwgl          1/1     Running   0          4m44s   10.28.1.16   gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    emailservice-6756967b6d-qw888            1/1     Running   0          4m52s   10.28.1.8    gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    frontend-766587959d-txzhs                1/1     Running   0          4m49s   10.28.1.11   gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    loadgenerator-9f854cfc5-g5f2s            1/1     Running   4          4m44s   10.28.1.15   gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    paymentservice-57c87dc78b-hmqkz          1/1     Running   0          4m47s   10.28.1.12   gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    productcatalogservice-9f5d68b54-4l4s7    1/1     Running   0          4m46s   10.28.1.13   gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    recommendationservice-57c49756fd-z2w9f   1/1     Running   0          4m50s   10.28.1.10   gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    redis-cart-5f75fbd9c7-s94b5              1/1     Running   0          4m42s   10.28.1.18   gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>
    shippingservice-689c6457cd-6v9nd         1/1     Running   0          4m43s   10.28.1.17   gke-cluster-logging-default-pool-e5144c9f-s2ch   <none>           <none>

###Установка EFK стека | Helm charts
Добавляем helm репозиторий EFK стека (ElasticSearch, Fluent Bit, Kibana)

    helm repo add elastic https://helm.elastic.co

Слайд 8

## Как запустить проект:


## Как проверить работоспособность:


## PR checklist:
 - [x] Выставлен label с темой домашнего задания